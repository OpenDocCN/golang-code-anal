# `v2ray-core\features\routing\session\context.go`

```go
package session

import (
    "context"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/features/routing"
)

// Context is an implementation of routing.Context, which is a wrapper of context.context with session info.
type Context struct {
    Inbound  *session.Inbound  // 入站会话信息
    Outbound *session.Outbound // 出站会话信息
    Content  *session.Content  // 会话内容信息
}

// GetInboundTag implements routing.Context.
func (ctx *Context) GetInboundTag() string {
    if ctx.Inbound == nil {
        return ""
    }
    return ctx.Inbound.Tag // 返回入站标签
}

// GetSourceIPs implements routing.Context.
func (ctx *Context) GetSourceIPs() []net.IP {
    if ctx.Inbound == nil || !ctx.Inbound.Source.IsValid() {
        return nil
    }
    dest := ctx.Inbound.Source
    if dest.Address.Family().IsDomain() {
        return nil
    }

    return []net.IP{dest.Address.IP()} // 返回源 IP 地址
}

// GetSourcePort implements routing.Context.
func (ctx *Context) GetSourcePort() net.Port {
    if ctx.Inbound == nil || !ctx.Inbound.Source.IsValid() {
        return 0
    }
    return ctx.Inbound.Source.Port // 返回源端口
}

// GetTargetIPs implements routing.Context.
func (ctx *Context) GetTargetIPs() []net.IP {
    if ctx.Outbound == nil || !ctx.Outbound.Target.IsValid() {
        return nil
    }

    if ctx.Outbound.Target.Address.Family().IsIP() {
        return []net.IP{ctx.Outbound.Target.Address.IP()} // 返回目标 IP 地址
    }

    return nil
}

// GetTargetPort implements routing.Context.
func (ctx *Context) GetTargetPort() net.Port {
    if ctx.Outbound == nil || !ctx.Outbound.Target.IsValid() {
        return 0
    }
    return ctx.Outbound.Target.Port // 返回目标端口
}

// GetTargetDomain implements routing.Context.
func (ctx *Context) GetTargetDomain() string {
    if ctx.Outbound == nil || !ctx.Outbound.Target.IsValid() {
        return ""
    }
    dest := ctx.Outbound.Target
    if !dest.Address.Family().IsDomain() {
        return ""
    }
    return dest.Address.Domain() // 返回目标域名
}

// GetNetwork implements routing.Context.
// GetNetwork 返回上下文中的网络信息
func (ctx *Context) GetNetwork() net.Network {
    // 如果出站信息为空，则返回未知网络
    if ctx.Outbound == nil {
        return net.Network_Unknown
    }
    // 返回出站目标的网络信息
    return ctx.Outbound.Target.Network
}

// GetProtocol 返回上下文中的协议信息
func (ctx *Context) GetProtocol() string {
    // 如果内容为空，则返回空字符串
    if ctx.Content == nil {
        return ""
    }
    // 返回内容的协议信息
    return ctx.Content.Protocol
}

// GetUser 返回上下文中的用户信息
func (ctx *Context) GetUser() string {
    // 如果入站信息为空或者用户信息为空，则返回空字符串
    if ctx.Inbound == nil || ctx.Inbound.User == nil {
        return ""
    }
    // 返回入站用户的邮箱信息
    return ctx.Inbound.User.Email
}

// GetAttributes 返回上下文中的属性信息
func (ctx *Context) GetAttributes() map[string]string {
    // 如果内容为空，则返回空
    if ctx.Content == nil {
        return nil
    }
    // 返回内容的属性信息
    return ctx.Content.Attributes
}

// AsRoutingContext 从上下文中创建一个带有会话信息的上下文
func AsRoutingContext(ctx context.Context) routing.Context {
    // 从上下文中获取入站、出站和内容信息，创建上下文对象
    return &Context{
        Inbound:  session.InboundFromContext(ctx),
        Outbound: session.OutboundFromContext(ctx),
        Content:  session.ContentFromContext(ctx),
    }
}
```