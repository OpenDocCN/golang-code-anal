# `kubo\plugin\loader\loader.go`

```
package loader

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "os"  // 导入操作系统功能包
    "path/filepath"  // 导入文件路径包
    "runtime"  // 导入运行时包
    "strings"  // 导入字符串处理包

    config "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/ipld/go-ipld-prime/multicodec"  // 导入多编解码包

    "github.com/ipfs/kubo/core"  // 导入核心功能包
    "github.com/ipfs/kubo/core/coreapi"  // 导入核心 API 包
    plugin "github.com/ipfs/kubo/plugin"  // 导入插件包
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 导入文件系统仓库包

    logging "github.com/ipfs/go-log"  // 导入日志包
    opentracing "github.com/opentracing/opentracing-go"  // 导入分布式追踪包
)

var preloadPlugins []plugin.Plugin  // 预加载的插件列表

// Preload 添加一个或多个插件到预加载列表。这应该 _只_ 在初始化期间调用。
func Preload(plugins ...plugin.Plugin) {
    preloadPlugins = append(preloadPlugins, plugins...)  // 将插件添加到预加载列表
}

var log = logging.Logger("plugin/loader")  // 创建日志记录器

var loadPluginFunc = func(string) ([]plugin.Plugin, error) {
    return nil, fmt.Errorf("unsupported platform %s", runtime.GOOS)  // 返回不支持的平台错误
}

type loaderState int  // 定义加载器状态类型

const (
    loaderLoading loaderState = iota  // 加载中状态
    loaderInitializing  // 初始化中状态
    loaderInitialized  // 已初始化状态
    loaderInjecting  // 注入中状态
    loaderInjected  // 已注入状态
    loaderStarting  // 启动中状态
    loaderStarted  // 已启动状态
    loaderClosing  // 关闭中状态
    loaderClosed  // 已关闭状态
    loaderFailed  // 失败状态
)

func (ls loaderState) String() string {
    switch ls {
    case loaderLoading:
        return "Loading"  // 返回加载中
    case loaderInitializing:
        return "Initializing"  // 返回初始化中
    case loaderInitialized:
        return "Initialized"  // 返回已初始化
    case loaderInjecting:
        return "Injecting"  // 返回注入中
    case loaderInjected:
        return "Injected"  // 返回已注入
    case loaderStarting:
        return "Starting"  // 返回启动中
    case loaderStarted:
        return "Started"  // 返回已启动
    case loaderClosing:
        return "Closing"  // 返回关闭中
    case loaderClosed:
        return "Closed"  // 返回已关闭
    case loaderFailed:
        return "Failed"  // 返回失败
    default:
        return "Unknown"  // 返回未知状态
    }
}

// PluginLoader 跟踪已加载的插件。
//
// 使用方法：
//  1. 使用 Load 和 LoadDirectory 加载任何所需的插件。预加载的插件将自动加载。
//  2. 调用 Initialize 运行所有初始化逻辑。
// 定义插件加载器结构体，包含状态、插件列表、已启动插件列表、配置和仓库路径
type PluginLoader struct {
    state   loaderState
    plugins []plugin.Plugin
    started []plugin.Plugin
    config  config.Plugins
    repo    string
}

// 创建新的插件加载器
func NewPluginLoader(repo string) (*PluginLoader, error) {
    // 初始化插件加载器，设置插件列表和仓库路径
    loader := &PluginLoader{plugins: make([]plugin.Plugin, 0, len(preloadPlugins)), repo: repo}
    // 如果仓库路径不为空
    if repo != "" {
        // 读取仓库路径下的插件配置文件
        switch plugins, err := readPluginsConfig(repo, config.DefaultConfigFile); {
        case err == nil:
            loader.config = plugins
        case os.IsNotExist(err):
        default:
            return nil, err
        }
    }

    // 遍历预加载的插件列表，加载插件
    for _, v := range preloadPlugins {
        if err := loader.Load(v); err != nil {
            return nil, err
        }
    }

    // 加载仓库路径下的插件目录
    if err := loader.LoadDirectory(filepath.Join(repo, "plugins")); err != nil {
        return nil, err
    }
    return loader, nil
}

// 读取 IPFS 配置文件中的插件配置
func readPluginsConfig(repoRoot string, userConfigFile string) (config.Plugins, error) {
    var cfg struct {
        Plugins config.Plugins
    }

    // 获取配置文件路径
    cfgPath, err := config.Filename(repoRoot, userConfigFile)
    if err != nil {
        return config.Plugins{}, err
    }

    // 打开配置文件
    cfgFile, err := os.Open(cfgPath)
    if err != nil {
        return config.Plugins{}, err
    }
    defer cfgFile.Close()

    // 解析配置文件中的插件配置
    err = json.NewDecoder(cfgFile).Decode(&cfg)
    if err != nil {
        return config.Plugins{}, err
    }

    return cfg.Plugins, nil
}

// 检查插件加载器的状态
func (loader *PluginLoader) assertState(state loaderState) error {
    // 如果加载器的状态与指定状态不符，则返回错误
    if loader.state != state {
        return fmt.Errorf("loader state must be %s, was %s", state, loader.state)
    }
    return nil
}
// transition 方法用于将插件加载器的状态从一个状态转换到另一个状态
func (loader *PluginLoader) transition(from, to loaderState) error {
    // 如果当前状态不是指定的 from 状态，则返回错误
    if err := loader.assertState(from); err != nil {
        return err
    }
    // 将插件加载器的状态设置为指定的 to 状态
    loader.state = to
    return nil
}

// Load 方法用于将一个插件加载到插件加载器中
func (loader *PluginLoader) Load(pl plugin.Plugin) error {
    // 如果当前状态不是 loaderLoading 状态，则返回错误
    if err := loader.assertState(loaderLoading); err != nil {
        return err
    }

    // 获取插件的名称
    name := pl.Name()

    // 遍历已加载的插件列表，检查是否有重复的插件
    for _, p := range loader.plugins {
        if p.Name() == name {
            // 插件已经加载过，返回错误
            return fmt.Errorf(
                "plugin: %s, is duplicated in version: %s, "+
                    "while trying to load dynamically: %s",
                name, p.Version(), pl.Version())
        }
    }

    // 如果插件在配置中被禁用，则不加载
    if loader.config.Plugins[name].Disabled {
        log.Infof("not loading disabled plugin %s", name)
        return nil
    }
    // 将插件添加到插件列表中
    loader.plugins = append(loader.plugins, pl)
    return nil
}

// LoadDirectory 方法用于将一个目录中的插件加载到插件加载器中
func (loader *PluginLoader) LoadDirectory(pluginDir string) error {
    // 如果当前状态不是 loaderLoading 状态，则返回错误
    if err := loader.assertState(loaderLoading); err != nil {
        return err
    }
    // 加载指定目录中的动态插件
    newPls, err := loadDynamicPlugins(pluginDir)
    if err != nil {
        return err
    }

    // 遍历新加载的插件列表，逐个加载到插件加载器中
    for _, pl := range newPls {
        if err := loader.Load(pl); err != nil {
            return err
        }
    }
    return nil
}

// loadDynamicPlugins 方法用于加载动态插件
func loadDynamicPlugins(pluginDir string) ([]plugin.Plugin, error) {
    // 检查指定目录是否存在
    _, err := os.Stat(pluginDir)
    if os.IsNotExist(err) {
        return nil, nil
    }
    if err != nil {
        return nil, err
    }

    var plugins []plugin.Plugin
    # 使用 filepath.Walk 遍历指定目录下的文件和子目录，并对每个文件和目录执行指定的函数
    err = filepath.Walk(pluginDir, func(fi string, info os.FileInfo, err error) error {
        # 如果在遍历过程中出现错误，则返回该错误
        if err != nil {
            return err
        }
        # 如果当前对象是一个目录
        if info.IsDir() {
            # 如果当前目录不是根目录，则记录警告信息
            if fi != pluginDir:
                log.Warnf("found directory inside plugins directory: %s", fi)
            return nil
        }

        # 如果文件不可执行，则记录错误信息并返回
        if info.Mode().Perm()&0o111 == 0:
            # 文件不可执行，防止加载非可执行文件作为插件，比如一些安全设置下的 /tmp 目录
            log.Errorf("non-executable file in plugins directory: %s", fi)
            return nil
        # 尝试加载插件，如果成功则将插件添加到列表中，否则返回加载插件时的错误信息
        if newPlugins, err := loadPluginFunc(fi); err == nil:
            plugins = append(plugins, newPlugins...)
        else:
            return fmt.Errorf("loading plugin %s: %s", fi, err)
        return nil
    })

    # 返回加载的插件列表和遍历过程中的错误
    return plugins, err
// Initialize函数初始化所有已加载的插件。
func (loader *PluginLoader) Initialize() error {
    // 如果在加载器处于加载中状态时进行初始化，则返回错误
    if err := loader.transition(loaderLoading, loaderInitializing); err != nil {
        return err
    }
    // 遍历所有插件，调用其Init方法进行初始化
    for _, p := range loader.plugins {
        err := p.Init(&plugin.Environment{
            Repo:   loader.repo,
            Config: loader.config.Plugins[p.Name()].Config,
        })
        // 如果初始化过程中出现错误，则将加载器状态设置为失败并返回错误
        if err != nil {
            loader.state = loaderFailed
            return err
        }
    }

    return loader.transition(loaderInitializing, loaderInitialized)
}

// Inject函数将所有插件注入到适当的子系统中。
func (loader *PluginLoader) Inject() error {
    // 如果在加载器处于已初始化状态时进行注入，则返回错误
    if err := loader.transition(loaderInitialized, loaderInjecting); err != nil {
        return err
    }

    // 遍历所有插件，根据类型调用相应的注入方法
    for _, pl := range loader.plugins {
        if pl, ok := pl.(plugin.PluginIPLD); ok {
            err := injectIPLDPlugin(pl)
            if err != nil {
                loader.state = loaderFailed
                return err
            }
        }
        if pl, ok := pl.(plugin.PluginTracer); ok {
            err := injectTracerPlugin(pl)
            if err != nil {
                loader.state = loaderFailed
                return err
            }
        }
        if pl, ok := pl.(plugin.PluginDatastore); ok {
            err := injectDatastorePlugin(pl)
            if err != nil {
                loader.state = loaderFailed
                return err
            }
        }
        if pl, ok := pl.(plugin.PluginFx); ok {
            err := injectFxPlugin(pl)
            if err != nil {
                loader.state = loaderFailed
                return err
            }
        }
    }

    return loader.transition(loaderInjecting, loaderInjected)
}

// Start函数启动所有长期运行的插件。
func (loader *PluginLoader) Start(node *core.IpfsNode) error {
    // 如果在加载器处于已注入状态时进行启动，则返回错误
    if err := loader.transition(loaderInjected, loaderStarting); err != nil {
        return err
    }
    // 创建一个核心 API 接口实例，并检查是否有错误发生
    iface, err := coreapi.NewCoreAPI(node)
    if err != nil {
        return err
    }
    // 遍历加载器的插件列表
    for _, pl := range loader.plugins {
        // 如果插件实现了 PluginDaemon 接口，则启动插件
        if pl, ok := pl.(plugin.PluginDaemon); ok {
            err := pl.Start(iface)
            // 如果启动插件时发生错误，则关闭加载器并返回错误
            if err != nil {
                _ = loader.Close()
                return err
            }
            // 将已启动的插件添加到加载器的已启动列表中
            loader.started = append(loader.started, pl)
        }
        // 如果插件实现了 PluginDaemonInternal 接口，则启动插件
        if pl, ok := pl.(plugin.PluginDaemonInternal); ok {
            err := pl.Start(node)
            // 如果启动插件时发生错误，则关闭加载器并返回错误
            if err != nil {
                _ = loader.Close()
                return err
            }
            // 将已启动的插件添加到加载器的已启动列表中
            loader.started = append(loader.started, pl)
        }
    }
    // 调用加载器的 transition 方法，将加载器状态转换为已启动
    return loader.transition(loaderStarting, loaderStarted)
// Close stops all long-running plugins.
func (loader *PluginLoader) Close() error {
    switch loader.state {
    case loaderClosing, loaderFailed, loaderClosed:
        // 如果状态为关闭中、失败或已关闭，则无需执行任何操作，直接返回 nil
        return nil
    }
    // 将状态设置为关闭中
    loader.state = loaderClosing

    var errs []string
    started := loader.started
    loader.started = nil
    // 遍历已启动的插件
    for _, pl := range started {
        // 如果插件实现了 io.Closer 接口
        if closer, ok := pl.(io.Closer); ok {
            // 调用插件的 Close 方法关闭插件
            err := closer.Close()
            // 如果关闭过程中出现错误，则记录错误信息
            if err != nil {
                errs = append(errs, fmt.Sprintf(
                    "error closing plugin %s: %s",
                    pl.Name(),
                    err.Error(),
                ))
            }
        }
    }
    // 如果存在错误，则将状态设置为关闭失败，并返回错误信息
    if errs != nil {
        loader.state = loaderFailed
        return fmt.Errorf(strings.Join(errs, "\n"))
    }
    // 将状态设置为已关闭
    loader.state = loaderClosed
    return nil
}

// 注入数据存储插件
func injectDatastorePlugin(pl plugin.PluginDatastore) error {
    return fsrepo.AddDatastoreConfigHandler(pl.DatastoreTypeName(), pl.DatastoreConfigParser())
}

// 注入 IPLD 插件
func injectIPLDPlugin(pl plugin.PluginIPLD) error {
    return pl.Register(multicodec.DefaultRegistry)
}

// 注入追踪器插件
func injectTracerPlugin(pl plugin.PluginTracer) error {
    log.Warn("Tracer plugins are deprecated, it's recommended to configure an OpenTelemetry collector instead.")
    // 初始化追踪器
    tracer, err := pl.InitTracer()
    if err != nil {
        return err
    }
    // 设置全局追踪器
    opentracing.SetGlobalTracer(tracer)
    return nil
}

// 注入 FX 插件
func injectFxPlugin(pl plugin.PluginFx) error {
    // 注册 FX 选项函数
    core.RegisterFXOptionFunc(pl.Options)
    return nil
}
```