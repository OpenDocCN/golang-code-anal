# `kubo\core\node\libp2p\host.go`

```
package libp2p

import (
    "context"

    "github.com/libp2p/go-libp2p"
    record "github.com/libp2p/go-libp2p-record"
    "github.com/libp2p/go-libp2p/core/host"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/libp2p/go-libp2p/core/peerstore"
    "github.com/libp2p/go-libp2p/core/routing"
    routedhost "github.com/libp2p/go-libp2p/p2p/host/routed"

    "github.com/ipfs/kubo/core/node/helpers"
    "github.com/ipfs/kubo/repo"

    "go.uber.org/fx"
)

type P2PHostIn struct {
    fx.In

    Repo          repo.Repo
    Validator     record.Validator
    HostOption    HostOption
    RoutingOption RoutingOption
    ID            peer.ID
    Peerstore     peerstore.Peerstore

    Opts [][]libp2p.Option `group:"libp2p"`
}

type P2PHostOut struct {
    fx.Out

    Host    host.Host
    Routing routing.Routing `name:"initialrouting"`
}

func Host(mctx helpers.MetricsCtx, lc fx.Lifecycle, params P2PHostIn) (out P2PHostOut, err error) {
    opts := []libp2p.Option{libp2p.NoListenAddrs}
    for _, o := range params.Opts {
        opts = append(opts, o...)
    }

    ctx := helpers.LifecycleCtx(mctx, lc)
    cfg, err := params.Repo.Config()
    if err != nil {
        return out, err
    }
    bootstrappers, err := cfg.BootstrapPeers()
    if err != nil {
        return out, err
    }

    routingOptArgs := RoutingOptionArgs{
        Ctx:                           ctx,
        Datastore:                     params.Repo.Datastore(),
        Validator:                     params.Validator,
        BootstrapPeers:                bootstrappers,
        OptimisticProvide:             cfg.Experimental.OptimisticProvide,
        OptimisticProvideJobsPoolSize: cfg.Experimental.OptimisticProvideJobsPoolSize,
    }
    // 将 libp2p.NoListenAddrs 选项添加到 opts 中
    opts = append(opts, libp2p.Routing(func(h host.Host) (routing.PeerRouting, error) {
        args := routingOptArgs
        args.Host = h
        r, err := params.RoutingOption(args)
        out.Routing = r
        return r, err
    }))
}
    // 使用参数中的 ID、Peerstore 和 opts 构建 Host 对象，并将错误赋值给 err
    out.Host, err = params.HostOption(params.ID, params.Peerstore, opts...)
    // 如果 err 不为空，则返回空的 P2PHostOut 对象和 err
    if err != nil {
        return P2PHostOut{}, err
    }

    // 将 routingOptArgs.Host 设置为 out.Host
    routingOptArgs.Host = out.Host

    // 以下代码仅用于测试：模拟网络构建
    // 忽略实际构建路由的 libp2p 构造选项！
    // 如果 out.Routing 为空，则使用参数中的 routingOptArgs 构建 Routing 对象，并将错误赋值给 err
    if out.Routing == nil {
        r, err := params.RoutingOption(routingOptArgs)
        // 如果 err 不为空，则返回空的 P2PHostOut 对象和 err
        if err != nil {
            return P2PHostOut{}, err
        }
        // 将 r 赋值给 out.Routing
        out.Routing = r
        // 将 out.Host 和 out.Routing 封装成 routedhost 对象，并重新赋值给 out.Host
        out.Host = routedhost.Wrap(out.Host, out.Routing)
    }

    // 将 out.Host.Close() 函数作为 OnStop 回调函数添加到 lc 中
    lc.Append(fx.Hook{
        OnStop: func(ctx context.Context) error {
            return out.Host.Close()
        },
    })

    // 返回 out 和 err
    return out, err
# 闭合前面的函数定义
```