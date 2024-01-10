# `trojan-go\tunnel\shadowsocks\client.go`

```
package shadowsocks

import (
    "context"  // 导入 context 包，用于处理上下文
    "github.com/shadowsocks/go-shadowsocks2/core"  // 导入 shadowsocks 核心包
    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 公共包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 trojan-go 配置包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 trojan-go 日志包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 隧道包
)

type Client struct {
    underlay tunnel.Client  // 定义一个名为 underlay 的隧道客户端
    core.Cipher  // 使用 shadowsocks2 核心包中的 Cipher 结构体
}

func (c *Client) DialConn(address *tunnel.Address, tunnel tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := c.underlay.DialConn(address, &Tunnel{})  // 使用 underlay 客户端进行连接
    if err != nil {
        return nil, err  // 如果连接出错，返回错误
    }
    return &Conn{  // 返回一个新的连接
        aeadConn: c.Cipher.StreamConn(conn),  // 使用 Cipher 结构体中的 StreamConn 方法进行加密
        Conn:     conn,  // 返回连接
    }, nil
}

func (c *Client) DialPacket(tunnel tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")  // 抛出错误，表示不支持该操作
}

func (c *Client) Close() error {
    return c.underlay.Close()  // 关闭 underlay 客户端
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    cfg := config.FromContext(ctx, Name).(*Config)  // 从上下文中获取配置信息
    cipher, err := core.PickCipher(cfg.Shadowsocks.Method, nil, cfg.Shadowsocks.Password)  // 选择加密方法
    if err != nil {
        return nil, common.NewError("invalid shadowsocks cipher").Base(err)  // 如果选择加密方法出错，返回错误
    }
    log.Debug("shadowsocks client created")  // 记录日志，表示创建了 shadowsocks 客户端
    return &Client{  // 返回一个新的客户端
        underlay: underlay,  // 设置 underlay 客户端
        Cipher:   cipher,  // 设置加密方法
    }, nil
}
```