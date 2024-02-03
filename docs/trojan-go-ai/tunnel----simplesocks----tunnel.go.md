# `trojan-go\tunnel\simplesocks\tunnel.go`

```go
package simplesocks

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "SIMPLESOCKS"  // 定义常量Name为"SIMPLESOCKS"

type Tunnel struct{}  // 定义类型Tunnel结构体

func (*Tunnel) Name() string {  // 定义Tunnel类型的方法Name，返回字符串类型
    return Name  // 返回常量Name
}

func (*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel类型的方法NewServer，接收上下文和tunnel.Server类型参数，返回tunnel.Server和error类型
    return NewServer(ctx, underlay)  // 返回NewServer函数的结果
}

func (*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel类型的方法NewClient，接收上下文和tunnel.Client类型参数，返回tunnel.Client和error类型
    return NewClient(ctx, underlay)  // 返回NewClient函数的结果
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel类型到tunnel包中
}
```