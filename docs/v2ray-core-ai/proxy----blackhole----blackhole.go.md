# `v2ray-core\proxy\blackhole\blackhole.go`

```
// +build !confonly
// 标记该文件不仅仅是配置文件

// Package blackhole is an outbound handler that blocks all connections.
// blackhole 包是一个出站处理程序，它阻止所有连接。

package blackhole
// 包名为 blackhole

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用 go:generate 命令生成错误处理代码

import (
    "context"
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/internet"
)
// 导入所需的包

// Handler is an outbound connection that silently swallow the entire payload.
// Handler 是一个出站连接，它悄悄地吞噬整个有效载荷。

type Handler struct {
    response ResponseConfig
}
// Handler 结构体定义，包含 response 字段

// New creates a new blackhole handler.
// New 创建一个新的 blackhole 处理程序。

func New(ctx context.Context, config *Config) (*Handler, error) {
    response, err := config.GetInternalResponse()
    if err != nil {
        return nil, err
    }
    return &Handler{
        response: response,
    }, nil
}
// New 函数定义，返回一个 Handler 实例

// Process implements OutboundHandler.Dispatch().
// Process 实现了 OutboundHandler.Dispatch()。

func (h *Handler) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    nBytes := h.response.WriteTo(link.Writer)
    if nBytes > 0 {
        // Sleep a little here to make sure the response is sent to client.
        // 在这里稍微休眠一下，以确保响应被发送到客户端。
        time.Sleep(time.Second)
    }
    common.Interrupt(link.Writer)
    return nil
}
// Process 方法定义，实现了 OutboundHandler.Dispatch()

func init() {
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))
    }))
}
// 初始化函数，注册配置并返回处理程序实例
```