# `kubo\routing\wrapper.go`

```
package routing

import (
    "context"

    routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"  // 导入routing-helpers包
    "github.com/libp2p/go-libp2p/core/routing"  // 导入core/routing包
)

type ProvideManyRouter interface {
    routinghelpers.ProvideManyRouter  // 定义ProvideManyRouter接口
    routing.Routing  // 定义Routing接口
}

var (
    _ routing.Routing                  = &httpRoutingWrapper{}  // httpRoutingWrapper实现了Routing接口
    _ routinghelpers.ProvideManyRouter = &httpRoutingWrapper{}  // httpRoutingWrapper实现了ProvideManyRouter接口
)

// httpRoutingWrapper is a wrapper needed to construct the routing.Routing interface from
// http delegated routing.
type httpRoutingWrapper struct {
    routing.ContentRouting  // 包含ContentRouting
    routing.PeerRouting  // 包含PeerRouting
    routing.ValueStore  // 包含ValueStore
    routinghelpers.ProvideManyRouter  // 包含ProvideManyRouter
}

func (c *httpRoutingWrapper) Bootstrap(ctx context.Context) error {
    return nil  // 实现Bootstrap方法，返回nil
}
```