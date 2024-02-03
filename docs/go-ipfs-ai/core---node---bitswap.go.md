# `kubo\core\node\bitswap.go`

```go
package node

import (
    "context"
    "time"

    "github.com/ipfs/boxo/bitswap"
    "github.com/ipfs/boxo/bitswap/network"
    blockstore "github.com/ipfs/boxo/blockstore"
    exchange "github.com/ipfs/boxo/exchange"
    "github.com/ipfs/kubo/config"
    irouting "github.com/ipfs/kubo/routing"
    "github.com/libp2p/go-libp2p/core/host"
    "go.uber.org/fx"

    "github.com/ipfs/kubo/core/node/helpers"
)

// Docs: https://github.com/ipfs/kubo/blob/master/docs/config.md#internalbitswap
const (
    DefaultEngineBlockstoreWorkerCount = 128
    DefaultTaskWorkerCount             = 8
    DefaultEngineTaskWorkerCount       = 8
    DefaultMaxOutstandingBytesPerPeer  = 1 << 20
    DefaultProviderSearchDelay         = 1000 * time.Millisecond
)

type bitswapOptionsOut struct {
    fx.Out

    BitswapOpts []bitswap.Option `group:"bitswap-options,flatten"`
}

// BitswapOptions creates configuration options for Bitswap from the config file
// and whether to provide data.
func BitswapOptions(cfg *config.Config, provide bool) interface{} {
    // 从配置文件创建 Bitswap 的配置选项，包括提供数据的设置
    # 返回一个 bitswapOptionsOut 结构体
    return func() bitswapOptionsOut {
        # 定义一个内部的 bitswap 配置
        var internalBsCfg config.InternalBitswap
        # 如果外部传入的 bitswap 配置不为空，则使用外部传入的配置
        if cfg.Internal.Bitswap != nil {
            internalBsCfg = *cfg.Internal.Bitswap
        }

        # 定义 bitswap 的选项列表
        opts := []bitswap.Option{
            # 设置提供数据的功能是否启用
            bitswap.ProvideEnabled(provide),
            # 设置提供者搜索延迟，使用内部 bitswap 配置的值，如果没有则使用默认值
            bitswap.ProviderSearchDelay(internalBsCfg.ProviderSearchDelay.WithDefault(DefaultProviderSearchDelay)), // See https://github.com/ipfs/go-ipfs/issues/8807 for rationale
            # 设置引擎块存储工作线程数，使用内部 bitswap 配置的值，如果没有则使用默认值
            bitswap.EngineBlockstoreWorkerCount(int(internalBsCfg.EngineBlockstoreWorkerCount.WithDefault(DefaultEngineBlockstoreWorkerCount))),
            # 设置任务工作线程数，使用内部 bitswap 配置的值，如果没有则使用默认值
            bitswap.TaskWorkerCount(int(internalBsCfg.TaskWorkerCount.WithDefault(DefaultTaskWorkerCount))),
            # 设置引擎任务工作线程数，使用内部 bitswap 配置的值，如果没有则使用默认值
            bitswap.EngineTaskWorkerCount(int(internalBsCfg.EngineTaskWorkerCount.WithDefault(DefaultEngineTaskWorkerCount))),
            # 设置每个对等节点的最大未完成字节数，使用内部 bitswap 配置的值，如果没有则使用默认值
            bitswap.MaxOutstandingBytesPerPeer(int(internalBsCfg.MaxOutstandingBytesPerPeer.WithDefault(DefaultMaxOutstandingBytesPerPeer))),
        }

        # 返回 bitswapOptionsOut 结构体，包含 bitswap 的选项列表
        return bitswapOptionsOut{BitswapOpts: opts}
    }
// 结构体定义，包含了在线交换所需的各种属性和依赖
type onlineExchangeIn struct {
    fx.In

    Mctx        helpers.MetricsCtx  // 指标上下文
    Host        host.Host           // 主机
    Rt          irouting.ProvideManyRouter  // 路由器
    Bs          blockstore.GCBlockstore  // 块存储
    BitswapOpts []bitswap.Option `group:"bitswap-options"`  // bitswap 选项
}

// OnlineExchange 函数创建一个基于 LibP2P 的块交换 (BitSwap)。
// 可以通过 "bitswap-options" 组提供给 bitswap.New 的额外选项。
func OnlineExchange() interface{} {
    return func(in onlineExchangeIn, lc fx.Lifecycle) exchange.Interface {
        // 从 IPFS 主机和路由器创建 bitswap 网络
        bitswapNetwork := network.NewFromIpfsHost(in.Host, in.Rt)

        // 创建新的 bitswap 实例，传入指标上下文、bitswap 网络、块存储和 bitswap 选项
        exch := bitswap.New(helpers.LifecycleCtx(in.Mctx, lc), bitswapNetwork, in.Bs, in.BitswapOpts...)
        
        // 在生命周期结束时执行的钩子，关闭 bitswap 实例
        lc.Append(fx.Hook{
            OnStop: func(ctx context.Context) error {
                return exch.Close()
            },
        })
        // 返回 bitswap 实例
        return exch
    }
}
```