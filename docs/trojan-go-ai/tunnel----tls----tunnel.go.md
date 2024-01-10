# `trojan-go\tunnel\tls\tunnel.go`

```
package tls

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入tunnel包
)

const Name = "TLS"  // 定义常量Name为"TLS"

type Tunnel struct{}  // 定义Tunnel结构体

func (t *Tunnel) Name() string {  // 定义Tunnel结构体的Name方法
    return Name  // 返回常量Name
}

func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {  // 定义Tunnel结构体的NewClient方法
    return NewClient(ctx, client)  // 返回NewClient方法的结果
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {  // 定义Tunnel结构体的NewServer方法
    return NewServer(ctx, server)  // 返回NewServer方法的结果
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册Tunnel结构体
}
```