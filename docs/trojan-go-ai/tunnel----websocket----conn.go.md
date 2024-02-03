# `trojan-go\tunnel\websocket\conn.go`

```go
package websocket

import (
    "context"  // 导入上下文包
    "net"      // 导入网络包

    "golang.org/x/net/websocket"  // 导入 websocket 包

    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义的 tunnel 包
)

type OutboundConn struct {
    *websocket.Conn  // 定义 OutboundConn 结构体包含一个 websocket 连接
    tcpConn net.Conn  // 定义 OutboundConn 结构体包含一个 TCP 连接
}

func (c *OutboundConn) Metadata() *tunnel.Metadata {
    return nil  // 返回空的元数据
}

func (c *OutboundConn) RemoteAddr() net.Addr {
    // 重写 RemoteAddr 方法，返回 TCP 连接的远程地址
    return c.tcpConn.RemoteAddr()
}

type InboundConn struct {
    OutboundConn  // InboundConn 结构体包含 OutboundConn 结构体的所有字段和方法
    ctx    context.Context  // 定义上下文
    cancel context.CancelFunc  // 定义取消函数
}

func (c *InboundConn) Close() error {
    c.cancel()  // 调用取消函数
    return c.Conn.Close()  // 关闭连接并返回错误
}
```