# `trojan-go\tunnel\tunnel.go`

```go
package tunnel

import (
    "context"  // 上下文包，用于控制goroutine的生命周期
    "io"  // 输入输出包，提供了基本的输入输出功能
    "net"  // 网络包，提供了基本的网络功能

    "github.com/p4gefau1t/trojan-go/common"  // 引入第三方库

)

// Conn is the TCP connection in the tunnel
type Conn interface {
    net.Conn  // TCP连接接口
    Metadata() *Metadata  // 获取元数据的方法
}

// PacketConn is the UDP packet stream in the tunnel
type PacketConn interface {
    net.PacketConn  // UDP数据包连接接口
    WriteWithMetadata([]byte, *Metadata) (int, error)  // 带元数据写入数据的方法
    ReadWithMetadata([]byte) (int, *Metadata, error)  // 带元数据读取数据的方法
}

// ConnDialer creates TCP connections from the tunnel
type ConnDialer interface {
    DialConn(*Address, Tunnel) (Conn, error)  // 从隧道创建TCP连接的方法
}

// PacketDialer creates UDP packet stream from the tunnel
type PacketDialer interface {
    DialPacket(Tunnel) (PacketConn, error)  // 从隧道创建UDP数据包连接的方法
}

// ConnListener accept TCP connections
type ConnListener interface {
    AcceptConn(Tunnel) (Conn, error)  // 接受TCP连接的方法
}

// PacketListener accept UDP packet stream
// We don't have any tunnel based on packet streams, so AcceptPacket will always receive a real PacketConn
type PacketListener interface {
    AcceptPacket(Tunnel) (PacketConn, error)  // 接受UDP数据包连接的方法
}

// Dialer can dial to original server with a tunnel
type Dialer interface {
    ConnDialer  // TCP连接拨号接口
    PacketDialer  // UDP数据包连接拨号接口
}

// Listener can accept TCP and UDP streams from a tunnel
type Listener interface {
    ConnListener  // TCP连接监听接口
    PacketListener  // UDP数据包连接监听接口
}

// Client is the tunnel client based on stream connections
type Client interface {
    Dialer  // 拨号接口
    io.Closer  // 关闭接口
}

// Server is the tunnel server based on stream connections
type Server interface {
    Listener  // 监听接口
    io.Closer  // 关闭接口
}

// Tunnel describes a tunnel, allowing creating a tunnel from another tunnel
// We assume that the lower tunnels know exatly how upper tunnels work, and lower tunnels is transparent for the upper tunnels
type Tunnel interface {
    Name() string  // 获取隧道名称的方法
    NewClient(context.Context, Client) (Client, error)  // 创建客户端的方法
    NewServer(context.Context, Server) (Server, error)  // 创建服务器的方法
}

var tunnels = make(map[string]Tunnel)  // 创建隧道映射

// RegisterTunnel register a tunnel by tunnel name
func RegisterTunnel(name string, tunnel Tunnel) {
    # 将变量tunnel的值存入字典tunnels中，键为name
    tunnels[name] = tunnel
# 根据给定的名称获取隧道对象，如果存在则返回该对象，否则返回错误信息
func GetTunnel(name string) (Tunnel, error) {
    # 检查名称是否在隧道映射中存在，如果存在则返回对应的隧道对象和空错误
    if t, ok := tunnels[name]; ok {
        return t, nil
    }
    # 如果名称不存在于隧道映射中，则返回空对象和包含错误信息的新错误对象
    return nil, common.NewError("unknown tunnel name " + name)
}
```