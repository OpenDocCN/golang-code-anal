# `trojan-go\tunnel\shadowsocks\tunnel.go`

```go
package shadowsocks
// 导入必要的包
import (
    "context"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

// 定义常量 Name 为 "SHADOWSOCKS"
const Name = "SHADOWSOCKS"

// 定义类型 Tunnel
type Tunnel struct{}

// 实现 Name 方法，返回常量 Name
func (t *Tunnel) Name() string {
    return Name
}

// 实现 NewClient 方法，返回新的客户端
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
    return NewClient(ctx, client)
}

// 实现 NewServer 方法，返回新的服务器
func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
    return NewServer(ctx, server)
}

// 初始化函数
func init() {
    // 注册隧道类型为 Name，并指定对应的 Tunnel 对象
    tunnel.RegisterTunnel(Name, &Tunnel{})
}
```