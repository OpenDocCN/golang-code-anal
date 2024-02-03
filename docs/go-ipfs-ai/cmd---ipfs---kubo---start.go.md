# `kubo\cmd\ipfs\kubo\start.go`

```go
// cmd/ipfs/kubo 实现了 kubo 的主要 CLI 二进制文件
package kubo

import (
    "bytes" // 导入 bytes 包，用于操作字节切片
    "context" // 导入 context 包，用于跟踪请求的上下文
    "encoding/json" // 导入 json 包，用于 JSON 数据的编解码
    "errors" // 导入 errors 包，用于创建错误
    "fmt" // 导入 fmt 包，用于格式化输入输出
    "io" // 导入 io 包，用于进行 I/O 操作
    "net" // 导入 net 包，用于网络操作
    "net/http" // 导入 http 包，用于 HTTP 客户端和服务端的实现
    "os" // 导入 os 包，用于操作系统功能
    "runtime/pprof" // 导入 pprof 包，用于性能分析
    "strings" // 导入 strings 包，用于字符串操作
    "time" // 导入 time 包，用于时间操作

    "github.com/blang/semver/v4" // 导入第三方库，用于语义化版本控制
    "github.com/google/uuid" // 导入第三方库，用于生成 UUID
    u "github.com/ipfs/boxo/util" // 导入自定义包，用于一些工具函数
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 IPFS 命令行包
    "github.com/ipfs/go-ipfs-cmds/cli" // 导入 IPFS 命令行包的 CLI 实现
    cmdhttp "github.com/ipfs/go-ipfs-cmds/http" // 导入 IPFS 命令行包的 HTTP 实现
    logging "github.com/ipfs/go-log" // 导入日志包
    ipfs "github.com/ipfs/kubo" // 导入 kubo 包
    "github.com/ipfs/kubo/client/rpc/auth" // 导入客户端认证包
    "github.com/ipfs/kubo/cmd/ipfs/util" // 导入 IPFS 工具包
    oldcmds "github.com/ipfs/kubo/commands" // 导入旧的 IPFS 命令包
    config "github.com/ipfs/kubo/config" // 导入配置包
    "github.com/ipfs/kubo/core" // 导入核心功能包
    corecmds "github.com/ipfs/kubo/core/commands" // 导入核心命令包
    "github.com/ipfs/kubo/core/corehttp" // 导入核心 HTTP 包
    "github.com/ipfs/kubo/plugin/loader" // 导入插件加载器包
    "github.com/ipfs/kubo/repo" // 导入存储库包
    "github.com/ipfs/kubo/repo/fsrepo" // 导入文件系统存储库包
    "github.com/ipfs/kubo/tracing" // 导入跟踪包
    ma "github.com/multiformats/go-multiaddr" // 导入多地址包
    madns "github.com/multiformats/go-multiaddr-dns" // 导入多地址 DNS 包
    manet "github.com/multiformats/go-multiaddr/net" // 导入多地址网络包
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp" // 导入 OpenTelemetry HTTP 包
    "go.opentelemetry.io/contrib/propagators/autoprop" // 导入 OpenTelemetry 自动传播包
    "go.opentelemetry.io/otel" // 导入 OpenTelemetry 包
    "go.opentelemetry.io/otel/attribute" // 导入 OpenTelemetry 属性包
    "go.opentelemetry.io/otel/codes" // 导入 OpenTelemetry 状态码包
    "go.opentelemetry.io/otel/trace" // 导入 OpenTelemetry 跟踪包
)

// log 是命令日志记录器
var (
    log    = logging.Logger("cmd/ipfs") // 创建命令日志记录器
    tracer trace.Tracer // 创建跟踪器
)

// 为测试目的声明为变量
var dnsResolver = madns.DefaultResolver // 设置 DNS 解析器为默认解析器

const (
    EnvEnableProfiling = "IPFS_PROF" // 环境变量，用于启用性能分析
    cpuProfile         = "ipfs.cpuprof" // CPU 性能分析文件名
    heapProfile        = "ipfs.memprof" // 堆内存性能分析文件名
)

type PluginPreloader func(*loader.PluginLoader) error // 插件预加载器函数类型

func loadPlugins(repoPath string, preload PluginPreloader) (*loader.PluginLoader, error) {
    plugins, err := loader.NewPluginLoader(repoPath) // 创建插件加载器
    if err != nil {
        return nil, fmt.Errorf("error loading plugins: %s", err) // 如果创建插件加载器出错，返回错误
    }
    // 如果预加载函数不为空，则执行预加载函数
    if preload != nil {
        // 如果预加载函数返回错误，则返回加载插件错误
        if err := preload(plugins); err != nil {
            return nil, fmt.Errorf("error loading plugins (preload): %s", err)
        }
    }
    // 初始化插件
    if err := plugins.Initialize(); err != nil {
        return nil, fmt.Errorf("error initializing plugins: %s", err)
    }
    // 注入插件
    if err := plugins.Inject(); err != nil {
        return nil, fmt.Errorf("error initializing plugins: %s", err)
    }
    // 返回插件和空错误
    return plugins, nil
# 打印错误信息到标准错误输出，并返回错误码
func printErr(err error) int {
    # 使用格式化输出将错误信息写入标准错误输出
    fmt.Fprintf(os.Stderr, "Error: %s\n", err.Error())
    # 返回错误码 1
    return 1
}

# 生成一个新的 UUID，并将其封装成 logging.Metadata 对象
func newUUID(key string) logging.Metadata {
    # 初始化 ids 变量
    ids := "#UUID-ERROR#"
    # 生成一个新的 UUID，并将其转换成字符串
    if id, err := uuid.NewRandom(); err == nil {
        ids = id.String()
    }
    # 返回包含 key 和 ids 的 logging.Metadata 对象
    return logging.Metadata{
        key: ids,
    }
}

# 构建默认的环境
func BuildDefaultEnv(ctx context.Context, req *cmds.Request) (cmds.Environment, error) {
    # 调用 BuildEnv 函数，并传入 nil 作为参数
    return BuildEnv(nil)(ctx, req)
}

# 创建一个用于 kubo CLI 的环境
# 注意：插件预加载器应该只调用与预加载插件相关的函数（例如 Load）
func BuildEnv(pl PluginPreloader) func(ctx context.Context, req *cmds.Request) (cmds.Environment, error) {
    // 定义一个函数，接收上下文和请求参数，返回环境和错误
    return func(ctx context.Context, req *cmds.Request) (cmds.Environment, error) {
        // 检查请求是否为调试模式
        checkDebug(req)
        // 获取仓库路径和可能的错误
        repoPath, err := getRepoPath(req)
        // 如果获取仓库路径出错，返回空和错误
        if err != nil {
            return nil, err
        }
        // 打印配置路径信息
        log.Debugf("config path is %s", repoPath)

        // 加载插件
        plugins, err := loadPlugins(repoPath, pl)
        // 如果加载插件出错，返回空和错误
        if err != nil {
            return nil, err
        }

        // 设置初始化节点的函数，以便可以延迟构造节点
        return &oldcmds.Context{
            ConfigRoot: repoPath,
            ReqLog:     &oldcmds.ReqLog{},
            Plugins:    plugins,
            ConstructNode: func() (n *core.IpfsNode, err error) {
                // 如果请求为空，返回错误
                if req == nil {
                    return nil, errors.New("constructing node without a request")
                }

                // 打开仓库路径
                r, err := fsrepo.Open(repoPath)
                // 如果打开仓库路径出错，返回错误
                if err != nil { // repo is owned by the node
                    return nil, err
                }

                // 一切正常，设置节点所有权并返回节点
                n, err = core.NewNode(ctx, &core.BuildCfg{
                    Repo: r,
                })
                // 如果创建节点出错，返回错误
                if err != nil {
                    return nil, err
                }

                // 返回节点和空错误
                return n, nil
            },
        }, nil
    }
// Start函数的入口，接受一个构建环境的函数，返回退出码
func Start(buildEnv func(ctx context.Context, req *cmds.Request) (cmds.Environment, error)) (exitCode int) {
    // 创建一个带有日志的上下文
    ctx := logging.ContextWithLoggable(context.Background(), newUUID("session"))

    // 创建一个追踪器提供程序
    tp, err := tracing.NewTracerProvider(ctx)
    if err != nil {
        return printErr(err) // 如果出错，打印错误信息并返回
    }
    defer func() {
        if err := tp.Shutdown(ctx); err != nil {
            exitCode = printErr(err) // 延迟关闭追踪器提供程序，并在出错时打印错误信息
        }
    }()
    otel.SetTracerProvider(tp) // 设置追踪器提供程序
    otel.SetTextMapPropagator(autoprop.NewTextMapPropagator()) // 设置文本映射传播器
    tracer = tp.Tracer("Kubo-cli") // 设置追踪器

    stopFunc, err := profileIfEnabled() // 如果启用了配置文件，进行配置文件处理
    if err != nil {
        return printErr(err) // 如果出错，打印错误信息并返回
    }
    defer stopFunc() // 尽可能晚地执行配置文件处理

    intrh, ctx := util.SetupInterruptHandler(ctx) // 设置中断处理程序
    defer intrh.Close() // 延迟关闭中断处理程序

    // 处理 `ipfs version` 或 `ipfs help`
    if len(os.Args) > 1 {
        // 处理 `ipfs --version'
        if os.Args[1] == "--version" {
            os.Args[1] = "version"
        }

        // 处理 `ipfs help` 和 `ipfs help <sub-command>`
        if os.Args[1] == "help" {
            if len(os.Args) > 2 {
                os.Args = append(os.Args[:1], os.Args[2:]...)
                // 处理 `ipfs help --help`
                // 当命令不是 `ipfs help --help` 时，追加 `--help`
                if os.Args[1] != "--help" {
                    os.Args = append(os.Args, "--help")
                }
            } else {
                os.Args[1] = "--help"
            }
        }
    } else if insideGUI() { // 如果没有传递参数，并且在 GUI 环境中
        // 启动守护进程而不是启动一个幽灵窗口
        os.Args = append(os.Args, "daemon", "--init")
    }

    // 输出取决于在 os.Args 中传递的可执行文件名
}
    // 确保命令行参数中的可执行文件名稳定
    os.Args[0] = "ipfs"

    // 运行 CLI 命令，传入上下文、根命令、命令行参数、标准输入、标准输出、标准错误、构建环境和执行器
    err = cli.Run(ctx, Root, os.Args, os.Stdin, os.Stdout, os.Stderr, buildEnv, makeExecutor)
    // 如果出现错误，返回状态码 1
    if err != nil {
        return 1
    }

    // 一切都比预期的要好 :)
    // 返回状态码 0
    return 0
}

// 检查是否在 GUI 环境中
func insideGUI() bool {
    return util.InsideGUI()
}

// 检查是否启用调试模式
func checkDebug(req *cmds.Request) {
    // 检查用户是否想要调试。选项或环境变量。
    debug, _ := req.Options["debug"].(bool)
    if debug || os.Getenv("IPFS_LOGGING") == "debug" {
        u.Debug = true
        logging.SetDebugLogging()
    }
    if u.GetenvBool("DEBUG") {
        u.Debug = true
    }
}

// 获取 API 地址选项
func apiAddrOption(req *cmds.Request) (ma.Multiaddr, error) {
    apiAddrStr, apiSpecified := req.Options[corecmds.ApiOption].(string)
    if !apiSpecified {
        return nil, nil
    }
    return ma.NewMultiaddr(apiAddrStr)
}

// encodedAbsolutePathVersion 是 multipart 请求中绝对路径头部的 %-encoded 版本。在此版本之前，其以原始形式发送。
var encodedAbsolutePathVersion = semver.MustParse("0.23.0-dev")

// 创建执行器
func makeExecutor(req *cmds.Request, env interface{}) (cmds.Executor, error) {
    exe := tracingWrappedExecutor{cmds.NewExecutor(req.Root)}
    cctx := env.(*oldcmds.Context)

    // 检查命令是否被禁用
    if req.Command.NoLocal && req.Command.NoRemote {
        return nil, fmt.Errorf("command disabled: %v", req.Path)
    }

    // 是否可以在本地运行？
    if !req.Command.NoLocal {
        if doesNotUseRepo, ok := corecmds.GetDoesNotUseRepo(req.Command.Extra); doesNotUseRepo && ok {
            return exe, nil
        }
    }

    // 从命令行获取 API 选项
    apiAddr, err := apiAddrOption(req)
    if err != nil {
        return nil, err
    }

    // 当传递 API 标志时，要求在守护进程上运行此命令（除非我们尝试 _运行_ 守护进程）。
    daemonRequested := apiAddr != nil && req.Command != daemonCmd

    // 如果需要，在客户端上运行此命令。
}
    // 如果命令中指定了不使用远程执行，则判断是否为守护进程请求，如果是则返回错误信息
    if req.Command.NoRemote {
        if daemonRequested {
            // 用户请求在守护进程上运行命令，但我们无法执行
            // 注意：对于 `ipfs daemon` 命令，我们忽略这个检查
            return nil, errors.New("api flag specified but command cannot be run on the daemon")
        }
        return exe, nil
    }

    // 最后，在存储库中查找 API 文件
    if apiAddr == nil {
        var err error
        apiAddr, err = fsrepo.APIAddr(cctx.ConfigRoot)
        switch err {
        case nil, repo.ErrApiNotRunning:
        default:
            return nil, err
        }
    }

    // 仍然没有指定 API 地址？在客户端上运行或失败
    if apiAddr == nil {
        if req.Command.NoLocal {
            return nil, fmt.Errorf("command must be run on the daemon: %v", req.Path)
        }
        return exe, nil
    }

    // 解析 API 地址
    apiAddr, err = resolveAddr(req.Context, apiAddr)
    if err != nil {
        return nil, err
    }
    network, host, err := manet.DialArgs(apiAddr)
    if err != nil {
        return nil, err
    }

    // 构造执行器
    opts := []cmdhttp.ClientOpt{
        cmdhttp.ClientWithAPIPrefix(corehttp.APIPath),
    }

    // 如果（a）有存储库并且（b）没有强制使用守护进程，则回退到本地执行器
    if !daemonRequested && fsrepo.IsInitialized(cctx.ConfigRoot) {
        opts = append(opts, cmdhttp.ClientWithFallback(exe))
    }

    var tpt http.RoundTripper
    switch network {
    case "tcp", "tcp4", "tcp6":
        tpt = http.DefaultTransport
    case "unix":
        path := host
        host = "unix"
        tpt = &http.Transport{
            DialContext: func(_ context.Context, _, _ string) (net.Conn, error) {
                return net.Dial("unix", path)
            },
        }
    default:
        return nil, fmt.Errorf("unsupported API address: %s", apiAddr)
    }

    apiAuth, specified := req.Options[corecmds.ApiAuthOption].(string)
    // 如果指定了授权信息
    if specified {
        // 将 API 授权信息转换为授权对象
        authorization := config.ConvertAuthSecret(apiAuth)
        // 使用授权对象创建授权的 RoundTripper
        tpt = auth.NewAuthorizedRoundTripper(authorization, tpt)
    }

    // 创建一个 HTTP 客户端
    httpClient := &http.Client{
        // 设置客户端的传输层
        Transport: otelhttp.NewTransport(tpt),
    }
    // 将 HTTP 客户端添加到选项中
    opts = append(opts, cmdhttp.ClientWithHTTPClient(httpClient))

    // 获取远程版本，因为某些功能的兼容性可能会根据它而改变
    remoteVersion, err := getRemoteVersion(tracingWrappedExecutor{cmdhttp.NewClient(host, opts...)})
    if err != nil {
        // 如果出现错误，则返回空和错误
        return nil, err
    }
    // 将远程版本添加到选项中
    opts = append(opts, cmdhttp.ClientWithRawAbsPath(remoteVersion.LT(encodedAbsolutePathVersion)))

    // 返回包装了追踪的执行器的 HTTP 客户端
    return tracingWrappedExecutor{cmdhttp.NewClient(host, opts...)}, nil
// 定义一个结构体，包含一个 cmds.Executor 类型的字段
type tracingWrappedExecutor struct {
    exec cmds.Executor
}

// 实现 cmds.Executor 接口的 Execute 方法
func (twe tracingWrappedExecutor) Execute(req *cmds.Request, re cmds.ResponseEmitter, env cmds.Environment) error {
    // 使用追踪器开始一个 span，并设置 span 名称和属性
    ctx, span := tracer.Start(req.Context, "cmds."+strings.Join(req.Path, "."), trace.WithAttributes(attribute.StringSlice("Arguments", req.Arguments)))
    // 延迟执行 span.End()，确保 span 在函数结束时结束
    defer span.End()
    req.Context = ctx

    // 调用内部的 exec 对象的 Execute 方法
    err := twe.exec.Execute(req, re, env)
    // 如果执行出错，设置 span 的状态为错误
    if err != nil {
        span.SetStatus(codes.Error, err.Error())
    }
    return err
}

// 获取仓库路径的函数
func getRepoPath(req *cmds.Request) (string, error) {
    // 从请求的选项中获取仓库路径
    repoOpt, found := req.Options[corecmds.RepoDirOption].(string)
    // 如果找到并且不为空，则返回该路径
    if found && repoOpt != "" {
        return repoOpt, nil
    }

    // 否则，获取最佳已知路径作为仓库路径
    repoPath, err := fsrepo.BestKnownPath()
    if err != nil {
        return "", err
    }
    return repoPath, nil
}

// 开始 CPU 分析，并返回一个停止函数
func startProfiling() (func(), error) {
    // 尽早开始 CPU 分析
    ofi, err := os.Create(cpuProfile)
    if err != nil {
        return nil, err
    }
    err = pprof.StartCPUProfile(ofi)
    if err != nil {
        ofi.Close()
        return nil, err
    }
    // 启动一个 goroutine 定时写入堆分析
    go func() {
        for range time.NewTicker(time.Second * 30).C {
            err := writeHeapProfileToFile()
            if err != nil {
                log.Error(err)
            }
        }
    }()

    // 返回一个停止函数，用于停止 CPU 分析并关闭文件
    stopProfiling := func() {
        pprof.StopCPUProfile()
        ofi.Close() // 由闭包捕获
    }
    return stopProfiling, nil
}

// 将堆分析写入文件
func writeHeapProfileToFile() error {
    mprof, err := os.Create(heapProfile)
    if err != nil {
        return err
    }
    defer mprof.Close() // 写入堆分析后关闭文件
    return pprof.WriteHeapProfile(mprof)
}

// 如果启用了分析，则执行分析
func profileIfEnabled() (func(), error) {
    // FIXME 这是一个临时的 hack，以便正确执行异步操作的分析
    # 检查环境变量 EnvEnableProfiling 是否存在，如果存在则进行性能分析
    if os.Getenv(EnvEnableProfiling) != "":
        # 启动性能分析，并返回停止性能分析的函数
        stopProfilingFunc, err := startProfiling() // TODO maybe change this to its own option... profiling makes it slower.
        # 如果启动性能分析出错，则返回错误
        if err != nil:
            return nil, err
        # 返回停止性能分析的函数
        return stopProfilingFunc, nil
    # 如果环境变量 EnvEnableProfiling 不存在，则返回一个空函数和空值
    return func() {}, nil
# 解析地址，返回解析后的地址和可能的错误
func resolveAddr(ctx context.Context, addr ma.Multiaddr) (ma.Multiaddr, error) {
    # 使用带有超时的上下文
    ctx, cancelFunc := context.WithTimeout(ctx, 10*time.Second)
    # 延迟调用取消函数
    defer cancelFunc()

    # 解析地址
    addrs, err := dnsResolver.Resolve(ctx, addr)
    if err != nil {
        return nil, err
    }

    # 如果解析后的地址列表为空，返回错误
    if len(addrs) == 0 {
        return nil, errors.New("non-resolvable API endpoint")
    }

    # 返回解析后的第一个地址
    return addrs[0], nil
}

# 实现一个空的写入器
type nopWriter struct {
    io.Writer
}

# 关闭方法的实现
func (nw nopWriter) Close() error {
    return nil
}

# 获取远程版本信息
func getRemoteVersion(exe cmds.Executor) (*semver.Version, error) {
    # 使用带有截止时间的上下文
    ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(time.Second*30))
    # 延迟调用取消函数
    defer cancel()

    # 创建一个请求
    req, err := cmds.NewRequest(ctx, []string{"version"}, nil, nil, nil, Root)
    if err != nil {
        return nil, err
    }

    # 创建一个缓冲区
    var buf bytes.Buffer
    # 创建一个写入响应的响应发射器
    re, err := cmds.NewWriterResponseEmitter(nopWriter{&buf}, req)
    if err != nil {
        return nil, err
    }

    # 执行请求
    err = exe.Execute(req, re, nil)
    if err != nil {
        return nil, err
    }

    # 解码 JSON 数据到版本信息结构
    var out ipfs.VersionInfo
    dec := json.NewDecoder(&buf)
    if err := dec.Decode(&out); err != nil {
        return nil, err
    }

    # 返回解析后的版本信息
    return semver.New(out.Version)
}
```