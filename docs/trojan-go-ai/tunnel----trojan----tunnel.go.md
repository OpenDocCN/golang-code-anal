# `trojan-go\tunnel\trojan\tunnel.go`

```go
package trojan

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "TROJAN"  // 定义常量Name为"TROJAN"

type Tunnel struct{}  // 定义Tunnel结构体

func (c *Tunnel) Name() string {  // 定义Tunnel结构体的方法Name，返回字符串类型
    return Name  // 返回常量Name
}

func (c *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的方法NewClient，接收上下文和tunnel.Client类型参数，返回tunnel.Client和error类型
    return NewClient(ctx, client)  // 调用NewClient函数并返回结果
}

func (c *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的方法NewServer，接收上下文和tunnel.Server类型参数，返回tunnel.Server和error类型
    return NewServer(ctx, server)  // 调用NewServer函数并返回结果
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel结构体
}
```