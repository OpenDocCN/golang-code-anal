# `trojan-go\tunnel\websocket\tunnel.go`

```go
package websocket

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "WEBSOCKET"  // 定义常量Name为"WEBSOCKET"

type Tunnel struct{}  // 定义Tunnel结构体

func (*Tunnel) Name() string {  // 定义Tunnel结构体的Name方法
    return Name  // 返回常量Name
}

func (*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的NewServer方法
    return NewServer(ctx, underlay)  // 返回NewServer函数的结果
}

func (*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的NewClient方法
    return NewClient(ctx, underlay)  // 返回NewClient函数的结果
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel结构体
}
```