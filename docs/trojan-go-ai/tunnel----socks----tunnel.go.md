# `trojan-go\tunnel\socks\tunnel.go`

```go
package socks

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "SOCKS"  // 定义常量Name为"SOCKS"

type Tunnel struct{}  // 定义类型Tunnel结构体

func (*Tunnel) Name() string {  // 定义Tunnel类型的方法Name，返回字符串类型
    return Name  // 返回常量Name
}

func (*Tunnel) NewClient(context.Context, tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel类型的方法NewClient，接收上下文和tunnel.Client类型参数，返回tunnel.Client和error类型
    panic("not supported")  // 抛出异常，表示不支持该方法
}

func (*Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel类型的方法NewServer，接收上下文和tunnel.Server类型参数，返回tunnel.Server和error类型
    return NewServer(ctx, server)  // 调用NewServer函数，返回结果
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel类型到tunnel包中
}
```