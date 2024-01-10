# `v2ray-core\common\session\context.go`

```
package session

import "context"

type sessionKey int

const (
    idSessionKey sessionKey = iota  // 定义一个枚举类型的 sessionKey，用于标识不同的 session
    inboundSessionKey                // 定义另一个 sessionKey
    outboundSessionKey               // 定义另一个 sessionKey
    contentSessionKey                // 定义另一个 sessionKey
    muxPreferedSessionKey            // 定义另一个 sessionKey
    sockoptSessionKey                // 定义另一个 sessionKey
)

// ContextWithID returns a new context with the given ID.
func ContextWithID(ctx context.Context, id ID) context.Context {
    return context.WithValue(ctx, idSessionKey, id)  // 在给定的上下文中设置 ID 对应的值
}

// IDFromContext returns ID in this context, or 0 if not contained.
func IDFromContext(ctx context.Context) ID {
    if id, ok := ctx.Value(idSessionKey).(ID); ok {  // 从上下文中获取 ID 对应的值
        return id
    }
    return 0
}

func ContextWithInbound(ctx context.Context, inbound *Inbound) context.Context {
    return context.WithValue(ctx, inboundSessionKey, inbound)  // 在给定的上下文中设置 inbound 对应的值
}

func InboundFromContext(ctx context.Context) *Inbound {
    if inbound, ok := ctx.Value(inboundSessionKey).(*Inbound); ok {  // 从上下文中获取 inbound 对应的值
        return inbound
    }
    return nil
}

func ContextWithOutbound(ctx context.Context, outbound *Outbound) context.Context {
    return context.WithValue(ctx, outboundSessionKey, outbound)  // 在给定的上下文中设置 outbound 对应的值
}

func OutboundFromContext(ctx context.Context) *Outbound {
    if outbound, ok := ctx.Value(outboundSessionKey).(*Outbound); ok {  // 从上下文中获取 outbound 对应的值
        return outbound
    }
    return nil
}

func ContextWithContent(ctx context.Context, content *Content) context.Context {
    return context.WithValue(ctx, contentSessionKey, content)  // 在给定的上下文中设置 content 对应的值
}

func ContentFromContext(ctx context.Context) *Content {
    if content, ok := ctx.Value(contentSessionKey).(*Content); ok {  // 从上下文中获取 content 对应的值
        return content
    }
    return nil
}

// ContextWithMuxPrefered returns a new context with the given bool
func ContextWithMuxPrefered(ctx context.Context, forced bool) context.Context {
    return context.WithValue(ctx, muxPreferedSessionKey, forced)  // 在给定的上下文中设置 muxPrefered 对应的值
}

// MuxPreferedFromContext returns value in this context, or false if not contained.
func MuxPreferedFromContext(ctx context.Context) bool {
    # 检查上下文中是否存在指定键对应的值，并判断是否成功获取
    if val, ok := ctx.Value(muxPreferedSessionKey).(bool); ok:
        # 如果成功获取，则返回对应的数值
        return val
    # 如果未成功获取，则返回 false
    return false
// ContextWithSockopt 返回一个包含 Socket 配置的新上下文
func ContextWithSockopt(ctx context.Context, s *Sockopt) context.Context {
    // 使用 context.WithValue 将 Socket 配置 s 存入上下文 ctx 中
    return context.WithValue(ctx, sockoptSessionKey, s)
}

// SockoptFromContext 从上下文中返回 Socket 配置，如果不存在则返回 nil
func SockoptFromContext(ctx context.Context) *Sockopt {
    // 从上下文中获取 Socket 配置，如果存在则返回，否则返回 nil
    if sockopt, ok := ctx.Value(sockoptSessionKey).(*Sockopt); ok {
        return sockopt
    }
    return nil
}
```