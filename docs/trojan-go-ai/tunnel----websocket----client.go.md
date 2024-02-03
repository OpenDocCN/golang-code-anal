# `trojan-go\tunnel\websocket\client.go`

```go
package websocket

import (
    "context"  // 导入上下文包
    "strings"  // 导入字符串处理包

    "golang.org/x/net/websocket"  // 导入 websocket 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义 tunnel 包
)

type Client struct {
    underlay tunnel.Client  // 定义 underlay 字段为 tunnel.Client 类型
    hostname string  // 定义 hostname 字段为字符串类型
    path     string  // 定义 path 字段为字符串类型
}

func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := c.underlay.DialConn(nil, &Tunnel{})  // 使用 underlay 对象拨号连接
    if err != nil {
        return nil, common.NewError("websocket cannot dial with underlying client").Base(err)  // 如果出错，返回错误信息
    }
    url := "wss://" + c.hostname + c.path  // 构建 WebSocket 连接的 URL
    origin := "https://" + c.hostname  // 构建 WebSocket 连接的来源
    wsConfig, err := websocket.NewConfig(url, origin)  // 创建 WebSocket 配置
    if err != nil {
        return nil, common.NewError("invalid websocket config").Base(err)  // 如果出错，返回错误信息
    }
    wsConn, err := websocket.NewClient(wsConfig, conn)  // 创建 WebSocket 客户端连接
    if err != nil {
        return nil, common.NewError("websocket failed to handshake with server").Base(err)  // 如果出错，返回错误信息
    }
    return &OutboundConn{
        Conn:    wsConn,  // 返回 WebSocket 连接
        tcpConn: conn,  // 返回 TCP 连接
    }, nil
}

func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    return nil, common.NewError("not supported by websocket")  // 返回不支持的错误信息
}

func (c *Client) Close() error {
    return c.underlay.Close()  // 关闭 underlay 连接
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    cfg := config.FromContext(ctx, Name).(*Config)  // 从上下文中获取配置信息
    if !strings.HasPrefix(cfg.Websocket.Path, "/") {
        return nil, common.NewError("websocket path must start with \"/\"")  // 如果路径不以 "/" 开头，返回错误信息
    }
    if cfg.Websocket.Host == "" {
        cfg.Websocket.Host = cfg.RemoteHost  // 如果 WebSocket 主机名为空，使用远程主机名
        log.Warn("empty websocket hostname")  // 记录警告日志
    }
    log.Debug("websocket client created")  // 记录调试日志
    return &Client{
        hostname: cfg.Websocket.Host,  // 返回新创建的 WebSocket 客户端对象
        path:     cfg.Websocket.Path,  // 返回新创建的 WebSocket 客户端对象
        underlay: underlay,  // 返回新创建的 WebSocket 客户端对象
    }, nil
}
```