# `trojan-go\tunnel\mux\tunnel.go`

```go
package mux

import (
    "context"

    "github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "MUX"

type Tunnel struct{}

func (*Tunnel) Name() string {
    return Name
}

func (*Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
    return NewClient(ctx, client)
}

func (*Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
    return NewServer(ctx, server)
}

func init() {
    // 注册名为 "MUX" 的隧道类型
    tunnel.RegisterTunnel(Name, &Tunnel{})
}
```