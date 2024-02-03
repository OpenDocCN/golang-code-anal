# `trojan-go\tunnel\http\tunnel.go`

```go
package http

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入trojan-go的tunnel包，用于建立隧道连接
)

const Name = "HTTP"  // 定义常量Name为"HTTP"

type Tunnel struct{}  // 定义Tunnel结构体

func (t *Tunnel) Name() string {  // 定义Tunnel结构体的方法Name，返回字符串类型
    return Name  // 返回常量Name
}

func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的方法NewClient，接收上下文和客户端对象，返回客户端对象和错误
    panic("not supported")  // 抛出异常，表示不支持该操作
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的方法NewServer，接收上下文和服务器对象，返回服务器对象和错误
    return NewServer(ctx, server)  // 调用NewServer函数，返回服务器对象和错误
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel结构体到tunnel包中，使用常量Name作为标识
}
```