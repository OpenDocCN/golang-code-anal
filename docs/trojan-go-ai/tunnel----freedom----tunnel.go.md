# `trojan-go\tunnel\freedom\tunnel.go`

```
package freedom

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "FREEDOM"  // 定义常量Name为FREEDOM

type Tunnel struct{}  // 定义Tunnel结构体

func (*Tunnel) Name() string {  // 定义Tunnel结构体的Name方法，返回字符串类型
    return Name  // 返回常量Name
}

func (*Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的NewClient方法，接收上下文和tunnel.Client类型参数，返回tunnel.Client和error类型
    return NewClient(ctx, client)  // 调用NewClient函数，返回结果
}

func (*Tunnel) NewServer(ctx context.Context, client tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的NewServer方法，接收上下文和tunnel.Server类型参数，返回tunnel.Server和error类型
    panic("not supported")  // 抛出异常，表示不支持该操作
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel结构体
}
```