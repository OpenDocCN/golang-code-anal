# `trojan-go\tunnel\router\tunnel.go`

```
package router

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "ROUTER"  // 定义常量Name为"ROUTER"

type Tunnel struct{}  // 定义Tunnel结构体

func (t *Tunnel) Name() string {  // 定义Tunnel结构体的方法Name，返回字符串类型
    return Name  // 返回常量Name
}

func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的方法NewClient，接收上下文和tunnel.Client类型参数，返回tunnel.Client和error类型
    return NewClient(ctx, client)  // 返回NewClient函数的调用结果
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的方法NewServer，接收上下文和tunnel.Server类型参数，返回tunnel.Server和error类型
    panic("not supported")  // 抛出panic异常，提示不支持该操作
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 调用tunnel包的RegisterTunnel函数，注册Tunnel结构体
}
```