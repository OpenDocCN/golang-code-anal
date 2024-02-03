# `trojan-go\tunnel\simplesocks\client.go`

```go
package simplesocks

import (
    "context"

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包
    "github.com/p4gefau1t/trojan-go/log"     // 导入 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 tunnel 包
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"  // 导入 trojan 包
)

const (
    Connect   tunnel.Command = 1   // 定义 Connect 常量为 tunnel.Command 类型的值 1
    Associate tunnel.Command = 3   // 定义 Associate 常量为 tunnel.Command 类型的值 3
)

type Client struct {
    underlay tunnel.Client  // 定义一个名为 underlay 的字段，类型为 tunnel.Client
}

func (c *Client) DialConn(addr *tunnel.Address, t tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := c.underlay.DialConn(nil, &Tunnel{})  // 使用 underlay 字段调用 DialConn 方法，返回连接和错误
    if err != nil {  // 如果错误不为空
        return nil, common.NewError("simplesocks failed to dial using underlying tunnel").Base(err)  // 返回一个错误
    }
    return &Conn{  // 返回一个 Conn 对象
        Conn:       conn,  // 设置 Conn 字段为 conn
        isOutbound: true,  // 设置 isOutbound 字段为 true
        metadata: &tunnel.Metadata{  // 设置 metadata 字段为一个 tunnel.Metadata 对象
            Command: Connect,  // 设置 Command 字段为 Connect
            Address: addr,     // 设置 Address 字段为 addr
        },
    }, nil  // 返回 Conn 对象和空错误
}

func (c *Client) DialPacket(t tunnel.Tunnel) (tunnel.PacketConn, error) {
    conn, err := c.underlay.DialConn(nil, &Tunnel{})  // 使用 underlay 字段调用 DialConn 方法，返回连接和错误
    if err != nil {  // 如果错误不为空
        return nil, common.NewError("simplesocks failed to dial using underlying tunnel").Base(err)  // 返回一个错误
    }
    metadata := &tunnel.Metadata{  // 创建一个 tunnel.Metadata 对象
        Command: Associate,  // 设置 Command 字段为 Associate
        Address: &tunnel.Address{  // 设置 Address 字段为一个 tunnel.Address 对象
            DomainName:  "UDP_CONN",  // 设置 DomainName 字段为 "UDP_CONN"
            AddressType: tunnel.DomainName,  // 设置 AddressType 字段为 tunnel.DomainName
        },
    }
    if err := metadata.WriteTo(conn); err != nil {  // 如果写入 metadata 到 conn 出错
        return nil, common.NewError("simplesocks failed to write udp associate").Base(err)  // 返回一个错误
    }
    return &PacketConn{  // 返回一个 PacketConn 对象
        PacketConn: trojan.PacketConn{  // 设置 PacketConn 字段为 trojan.PacketConn 对象
            Conn: conn,  // 设置 Conn 字段为 conn
        },
    }, nil  // 返回 PacketConn 对象和空错误
}

func (c *Client) Close() error {
    return c.underlay.Close()  // 调用 underlay 字段的 Close 方法，返回错误
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    log.Debug("simplesocks client created")  // 打印日志信息
    return &Client{  // 返回一个 Client 对象
        underlay: underlay,  // 设置 underlay 字段为传入的参数 underlay
    }, nil  // 返回 Client 对象和空错误
}
```