# `kubo\plugin\plugins\nopfs\nopfs.go`

```
package nopfs

import (
    "os"  // 导入操作系统相关的包
    "path/filepath"  // 导入处理文件路径的包

    "github.com/ipfs-shipyard/nopfs"  // 导入 nopfs 包
    "github.com/ipfs-shipyard/nopfs/ipfs"  // 导入 nopfs 的 ipfs 子包
    "github.com/ipfs/kubo/config"  // 导入 kubo 的配置包
    "github.com/ipfs/kubo/core"  // 导入 kubo 的核心包
    "github.com/ipfs/kubo/core/node"  // 导入 kubo 的节点包
    "github.com/ipfs/kubo/plugin"  // 导入 kubo 的插件包
    "go.uber.org/fx"  // 导入 fx 包
)

// Plugins sets the list of plugins to be loaded.
var Plugins = []plugin.Plugin{  // 定义插件列表
    &nopfsPlugin{},  // 添加 nopfsPlugin 插件
}

// fxtestPlugin is used for testing the fx plugin.
// It merely adds an fx option that logs a debug statement, so we can verify that it works in tests.
type nopfsPlugin struct{}  // 定义 nopfsPlugin 结构体

var _ plugin.PluginFx = (*nopfsPlugin)(nil)  // 确保 nopfsPlugin 实现了 PluginFx 接口

func (p *nopfsPlugin) Name() string {  // 实现 nopfsPlugin 的 Name 方法
    return "nopfs"  // 返回插件名称
}

func (p *nopfsPlugin) Version() string {  // 实现 nopfsPlugin 的 Version 方法
    return "0.0.10"  // 返回插件版本
}

func (p *nopfsPlugin) Init(env *plugin.Environment) error {  // 实现 nopfsPlugin 的 Init 方法
    return nil  // 初始化插件，返回空错误
}

// MakeBlocker is a factory for the blocker so that it can be provided with Fx.
func MakeBlocker() (*nopfs.Blocker, error) {  // 定义 MakeBlocker 函数，用于创建 blocker
    ipfsPath, err := config.PathRoot()  // 获取 IPFS 路径
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空 blocker 和错误
    }

    defaultFiles, err := nopfs.GetDenylistFiles()  // 获取默认的拒绝列表文件
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空 blocker 和错误
    }

    kuboFiles, err := nopfs.GetDenylistFilesInDir(filepath.Join(ipfsPath, "denylists"))  // 获取指定目录中的拒绝列表文件
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空 blocker 和错误
    }

    files := append(defaultFiles, kuboFiles...)  // 将默认文件和 kubo 文件合并

    return nopfs.NewBlocker(files)  // 返回新创建的 blocker
}

// PathResolvers returns wrapped PathResolvers for Kubo.
func PathResolvers(fetchers node.FetchersIn, blocker *nopfs.Blocker) node.PathResolversOut {  // 定义 PathResolvers 函数
    res := node.PathResolverConfig(fetchers)  // 获取路径解析器配置
    return node.PathResolversOut{  // 返回路径解析器输出
        IPLDPathResolver:          ipfs.WrapResolver(res.IPLDPathResolver, blocker),  // 使用 blocker 封装 IPLDPathResolver
        UnixFSPathResolver:        ipfs.WrapResolver(res.UnixFSPathResolver, blocker),  // 使用 blocker 封装 UnixFSPathResolver
        OfflineIPLDPathResolver:   ipfs.WrapResolver(res.OfflineIPLDPathResolver, blocker),  // 使用 blocker 封装 OfflineIPLDPathResolver
        OfflineUnixFSPathResolver: ipfs.WrapResolver(res.OfflineUnixFSPathResolver, blocker),  // 使用 blocker 封装 OfflineUnixFSPathResolver
    }
}
# 定义 nopfsPlugin 结构体的 Options 方法，接收一个 FXNodeInfo 参数，返回 fx.Option 切片和 error
func (p *nopfsPlugin) Options(info core.FXNodeInfo) ([]fx.Option, error) {
    # 如果环境变量 IPFS_CONTENT_BLOCKING_DISABLE 不为空，则返回原始的 FXOptions 和 nil
    if os.Getenv("IPFS_CONTENT_BLOCKING_DISABLE") != "" {
        return info.FXOptions, nil
    }

    # 否则，创建一个新的 opts 切片，包含原始的 FXOptions 和一些额外的选项
    opts := append(
        info.FXOptions,
        fx.Provide(MakeBlocker),  # 提供 MakeBlocker 函数
        fx.Decorate(ipfs.WrapBlockService),  # 装饰 WrapBlockService 函数
        fx.Decorate(ipfs.WrapNameSystem),  # 装饰 WrapNameSystem 函数
        fx.Decorate(PathResolvers),  # 装饰 PathResolvers 函数
    )
    # 返回新的 opts 切片和 nil
    return opts, nil
}
```