# `trojan-go\tunnel\transport\conn.go`

```go
# 导入 net 包，用于网络通信
import (
    "net"
    # 导入 tunnel 包，用于隧道传输
    "github.com/p4gefau1t/trojan-go/tunnel"
)

# 定义 Conn 结构体，包含 net.Conn 类型的字段
type Conn struct {
    net.Conn
}

# 定义 Conn 结构体的 Metadata 方法，返回 tunnel.Metadata 类型指针
func (c *Conn) Metadata() *tunnel.Metadata {
    # 返回空指针
    return nil
}
```