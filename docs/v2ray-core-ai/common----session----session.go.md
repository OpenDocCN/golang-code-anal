# `v2ray-core\common\session\session.go`

```
// Package session provides functions for sessions of incoming requests.
package session // import "v2ray.com/core/common/session"

import (
    "context" // 导入 context 包
    "math/rand" // 导入 math/rand 包

    "v2ray.com/core/common/errors" // 导入错误处理包
    "v2ray.com/core/common/net" // 导入网络通信包
    "v2ray.com/core/common/protocol" // 导入协议包
)

// ID of a session.
type ID uint32 // 定义会话的 ID 类型为无符号 32 位整数

// NewID generates a new ID. The generated ID is high likely to be unique, but not cryptographically secure.
// The generated ID will never be 0.
func NewID() ID {
    for {
        id := ID(rand.Uint32()) // 生成一个随机的 ID
        if id != 0 { // 如果生成的 ID 不为 0
            return id // 返回生成的 ID
        }
    }
}

// ExportIDToError transfers session.ID into an error object, for logging purpose.
// This can be used with error.WriteToLog().
func ExportIDToError(ctx context.Context) errors.ExportOption {
    id := IDFromContext(ctx) // 从上下文中获取会话 ID
    return func(h *errors.ExportOptionHolder) {
        h.SessionID = uint32(id) // 将会话 ID 转换为错误对象，用于日志记录
    }
}

// Inbound is the metadata of an inbound connection.
type Inbound struct {
    // Source address of the inbound connection.
    Source net.Destination // 入站连接的源地址
    // Getaway address
    Gateway net.Destination // 网关地址
    // Tag of the inbound proxy that handles the connection.
    Tag string // 处理连接的入站代理的标签
    // User is the user that authencates for the inbound. May be nil if the protocol allows anounymous traffic.
    User *protocol.MemoryUser // 对于入站连接进行身份验证的用户，如果协议允许匿名流量，则可能为空
}

// Outbound is the metadata of an outbound connection.
type Outbound struct {
    // Target address of the outbound connection.
    Target net.Destination // 出站连接的目标地址
    // Gateway address
    Gateway net.Address // 网关地址
}

// SniffingRequest controls the behavior of content sniffing.
type SniffingRequest struct {
    OverrideDestinationForProtocol []string // 用于协议的目标地址覆盖
    Enabled                        bool // 是否启用内容嗅探
}

// Content is the metadata of the connection content.
type Content struct {
    // Protocol of current content.
    Protocol string // 当前内容的协议

    SniffingRequest SniffingRequest // 嗅探请求控制内容嗅探的行为

    Attributes map[string]string // 属性映射

    SkipRoutePick bool // 是否跳过路由选择
}

// Sockopt is the settings for socket connection.
# 定义了一个结构体 Sockopt，用于表示套接字连接的标记
type Sockopt struct {
    // 套接字连接的标记
    Mark int32
}

# 为 Content 结构体定义了一个方法，用于附加额外的字符串属性到内容中
func (c *Content) SetAttribute(name string, value string) {
    # 如果属性为空，则创建一个新的属性字典
    if c.Attributes == nil {
        c.Attributes = make(map[string]string)
    }
    # 将属性名和属性值添加到属性字典中
    c.Attributes[name] = value
}

# 为 Content 结构体定义了一个方法，用于从内容中检索额外的字符串属性
func (c *Content) Attribute(name string) string {
    # 如果属性为空，则返回空字符串
    if c.Attributes == nil {
        return ""
    }
    # 返回属性字典中对应属性名的属性值
    return c.Attributes[name]
}
```