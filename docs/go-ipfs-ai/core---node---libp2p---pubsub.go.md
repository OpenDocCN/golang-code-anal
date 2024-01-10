# `kubo\core\node\libp2p\pubsub.go`

```
package libp2p

import (
    pubsub "github.com/libp2p/go-libp2p-pubsub"  // 导入 pubsub 包
    "github.com/libp2p/go-libp2p/core/discovery"  // 导入 discovery 包
    "github.com/libp2p/go-libp2p/core/host"  // 导入 host 包
    "go.uber.org/fx"  // 导入 fx 包

    "github.com/ipfs/kubo/core/node/helpers"  // 导入 helpers 包
)

func FloodSub(pubsubOptions ...pubsub.Option) interface{} {
    // 返回一个函数，该函数接受一些参数并返回 pubsub.PubSub 和 error
    return func(mctx helpers.MetricsCtx, lc fx.Lifecycle, host host.Host, disc discovery.Discovery) (service *pubsub.PubSub, err error) {
        // 使用 pubsub.NewFloodSub 创建一个 FloodSub 服务
        return pubsub.NewFloodSub(helpers.LifecycleCtx(mctx, lc), host, append(pubsubOptions, pubsub.WithDiscovery(disc))...)
    }
}

func GossipSub(pubsubOptions ...pubsub.Option) interface{} {
    // 返回一个函数，该函数接受一些参数并返回 pubsub.PubSub 和 error
    return func(mctx helpers.MetricsCtx, lc fx.Lifecycle, host host.Host, disc discovery.Discovery) (service *pubsub.PubSub, err error) {
        // 使用 pubsub.NewGossipSub 创建一个 GossipSub 服务
        return pubsub.NewGossipSub(helpers.LifecycleCtx(mctx, lc), host, append(
            pubsubOptions,
            pubsub.WithDiscovery(disc),
            pubsub.WithFloodPublish(true))...,
        )
    }
}
```