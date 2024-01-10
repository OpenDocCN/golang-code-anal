# `kubo\core\node\libp2p\discovery.go`

```
package libp2p

import (
    "context"
    "time"

    "github.com/libp2p/go-libp2p/core/host"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/libp2p/go-libp2p/p2p/discovery/mdns"

    "go.uber.org/fx"

    "github.com/ipfs/kubo/core/node/helpers"
)

const discoveryConnTimeout = time.Second * 30

type discoveryHandler struct {
    ctx  context.Context
    host host.Host
}

func (dh *discoveryHandler) HandlePeerFound(p peer.AddrInfo) {
    log.Info("connecting to discovered peer: ", p)
    // 使用上下文设置超时时间
    ctx, cancel := context.WithTimeout(dh.ctx, discoveryConnTimeout)
    defer cancel()
    // 尝试连接到发现的对等节点
    if err := dh.host.Connect(ctx, p); err != nil {
        log.Warnf("failed to connect to peer %s found by discovery: %s", p.ID, err)
    }
}

func DiscoveryHandler(mctx helpers.MetricsCtx, lc fx.Lifecycle, host host.Host) *discoveryHandler {
    // 返回发现处理程序实例
    return &discoveryHandler{
        ctx:  helpers.LifecycleCtx(mctx, lc), // 使用生命周期上下文创建上下文
        host: host,
    }
}

func SetupDiscovery(useMdns bool) func(helpers.MetricsCtx, fx.Lifecycle, host.Host, *discoveryHandler) error {
    // 返回设置发现服务的函数
    return func(mctx helpers.MetricsCtx, lc fx.Lifecycle, host host.Host, handler *discoveryHandler) error {
        if useMdns {
            // 创建基于 mDNS 的服务
            service := mdns.NewMdnsService(host, mdns.ServiceName, handler)
            // 启动 mDNS 服务
            if err := service.Start(); err != nil {
                log.Error("error starting mdns service: ", err)
                return nil
            }
        }
        return nil
    }
}
```