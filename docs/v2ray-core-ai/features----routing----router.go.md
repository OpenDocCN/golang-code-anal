# `v2ray-core\features\routing\router.go`

```go
package routing

import (
    "v2ray.com/core/common"  // 导入v2ray核心common包
    "v2ray.com/core/features"  // 导入v2ray核心features包
)

// Router is a feature to choose an outbound tag for the given request.
//
// v2ray:api:stable
type Router interface {
    features.Feature  // Router接口继承features.Feature接口

    // PickRoute returns a route decision based on the given routing context.
    PickRoute(ctx Context) (Route, error)  // PickRoute方法根据给定的路由上下文返回路由决策
}

// Route is the routing result of Router feature.
//
// v2ray:api:stable
type Route interface {
    // A Route is also a routing context.
    Context  // Route接口继承Context接口

    // GetOutboundGroupTags returns the detoured outbound group tags in sequence before a final outbound is chosen.
    GetOutboundGroupTags() []string  // GetOutboundGroupTags方法返回最终选择之前的顺序中的出站组标签

    // GetOutboundTag returns the tag of the outbound the connection was dispatched to.
    GetOutboundTag() string  // GetOutboundTag方法返回连接分派到的出站标签
}

// RouterType return the type of Router interface. Can be used to implement common.HasType.
//
// v2ray:api:stable
func RouterType() interface{} {
    return (*Router)(nil)  // 返回Router接口的类型
}

// DefaultRouter is an implementation of Router, which always returns ErrNoClue for routing decisions.
type DefaultRouter struct{}  // DefaultRouter是Router的实现，始终返回路由决策的ErrNoClue

// Type implements common.HasType.
func (DefaultRouter) Type() interface{} {
    return RouterType()  // 实现common.HasType接口，返回RouterType的类型
}

// PickRoute implements Router.
func (DefaultRouter) PickRoute(ctx Context) (Route, error) {
    return nil, common.ErrNoClue  // 实现Router接口的PickRoute方法，返回nil和common.ErrNoClue
}

// Start implements common.Runnable.
func (DefaultRouter) Start() error {
    return nil  // 实现common.Runnable接口的Start方法，返回nil
}

// Close implements common.Closable.
func (DefaultRouter) Close() error {
    return nil  // 实现common.Closable接口的Close方法，返回nil
}
```