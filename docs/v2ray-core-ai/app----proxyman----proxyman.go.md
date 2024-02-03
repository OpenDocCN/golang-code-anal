# `v2ray-core\app\proxyman\proxyman.go`

```go
// Package proxyman 定义了用于管理入站和出站代理的应用程序。
package proxyman

import (
    "context"

    "v2ray.com/core/common/session"
)

// ContextWithSniffingConfig 是 session.ContextWithContent 的包装器。
// 已弃用。直接使用 session.ContextWithContent。
func ContextWithSniffingConfig(ctx context.Context, c *SniffingConfig) context.Context {
    // 从上下文中获取内容
    content := session.ContentFromContext(ctx)
    // 如果内容为空，则创建一个新的内容并将其添加到上下文中
    if content == nil {
        content = new(session.Content)
        ctx = session.ContextWithContent(ctx, content)
    }
    // 设置嗅探请求的配置
    content.SniffingRequest.Enabled = c.Enabled
    content.SniffingRequest.OverrideDestinationForProtocol = c.DestinationOverride
    // 返回更新后的上下文
    return ctx
}
```