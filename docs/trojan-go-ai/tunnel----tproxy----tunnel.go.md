# `trojan-go\tunnel\tproxy\tunnel.go`

```go
//go:build linux
// +build linux
// 声明该文件只在 Linux 系统下编译

package tproxy
// 导入必要的包
import (
    "context"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

// 定义常量 Name 为 "TPROXY"
const Name = "TPROXY"

// 定义类型 Tunnel
type Tunnel struct{}

// 实现 Name 方法，返回常量 Name
func (t *Tunnel) Name() string {
    return Name
}

// 实现 NewClient 方法，抛出异常 "not supported"
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
    panic("not supported")
}

// 实现 NewServer 方法，调用 NewServer 函数创建新的服务器
func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
    return NewServer(ctx, server)
}

// 初始化函数
func init() {
    // 注册隧道类型为 Name，对应的实例为 &Tunnel{}
    tunnel.RegisterTunnel(Name, &Tunnel{})
}
```