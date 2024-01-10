# `trojan-go\tunnel\freedom\client.go`

```
package freedom

import (
    "context"
    "net"

    "github.com/txthinking/socks5"
    "golang.org/x/net/proxy"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

type Client struct {
    preferIPv4   bool  // 是否优先使用 IPv4
    noDelay      bool  // 是否禁用 Nagle 算法
    keepAlive    bool  // 是否启用 TCP keep-alive
    ctx          context.Context  // 上下文对象
    cancel       context.CancelFunc  // 取消函数
    forwardProxy bool  // 是否使用转发代理
    proxyAddr    *tunnel.Address  // 代理地址
    username     string  // 用户名
    password     string  // 密码
}

func (c *Client) DialConn(addr *tunnel.Address, _ tunnel.Tunnel) (tunnel.Conn, error) {
    // forward proxy
    if c.forwardProxy {  // 如果使用转发代理
        var auth *proxy.Auth
        if c.username != "" {  // 如果用户名不为空
            auth = &proxy.Auth{  // 创建代理认证对象
                User:     c.username,  // 设置用户名
                Password: c.password,  // 设置密码
            }
        }
        dialer, err := proxy.SOCKS5("tcp", c.proxyAddr.String(), auth, proxy.Direct)  // 创建 SOCKS5 代理拨号器
        if err != nil {
            return nil, common.NewError("freedom failed to init socks dialer")  // 返回错误信息
        }
        conn, err := dialer.Dial("tcp", addr.String())  // 通过 SOCKS5 代理拨号器拨号目标地址
        if err != nil {
            return nil, common.NewError("freedom failed to dial target address via socks proxy " + addr.String()).Base(err)  // 返回错误信息
        }
        return &Conn{  // 返回连接对象
            Conn: conn,
        }, nil
    }
    network := "tcp"  // 默认网络类型为 TCP
    if c.preferIPv4 {  // 如果优先使用 IPv4
        network = "tcp4"  // 设置网络类型为 TCP4
    }
    dialer := new(net.Dialer)  // 创建网络拨号器
    tcpConn, err := dialer.DialContext(c.ctx, network, addr.String())  // 通过上下文和网络拨号器拨号目标地址
    if err != nil {
        return nil, common.NewError("freedom failed to dial " + addr.String()).Base(err)  // 返回错误信息
    }

    tcpConn.(*net.TCPConn).SetKeepAlive(c.keepAlive)  // 设置 TCP keep-alive
    tcpConn.(*net.TCPConn).SetNoDelay(c.noDelay)  // 设置禁用 Nagle 算法
    return &Conn{  // 返回连接对象
        Conn: tcpConn,
    }, nil
}

func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    // 如果配置了使用代理，则创建一个 SOCKS5 客户端
    if c.forwardProxy {
        // 使用 SOCKS5 协议创建客户端
        socksClient, err := socks5.NewClient(c.proxyAddr.String(), c.username, c.password, 0, 0)
        // 检查错误
        common.Must(err)
        // 进行 SOCKS5 协商
        if err := socksClient.Negotiate(&net.TCPAddr{}); err != nil {
            return nil, common.NewError("freedom failed to negotiate socks").Base(err)
        }
        // 解析一个无用的地址，用于测试
        a, addr, port, err := socks5.ParseAddress("1.1.1.1:53") // useless address
        common.Must(err)
        // 发送 UDP 请求到 SOCKS5 代理
        resp, err := socksClient.Request(socks5.NewRequest(socks5.CmdUDP, a, addr, port))
        if err != nil {
            return nil, common.NewError("freedom failed to dial udp to socks").Base(err)
        }
        // 创建一个 UDP 数据包连接
        packetConn, err := net.ListenPacket("udp", "127.0.0.1:0")
        if err != nil {
            return nil, common.NewError("freedom failed to listen udp").Base(err)
        }
        // 解析 SOCKS5 返回的 UDP 地址
        socksAddr, err := net.ResolveUDPAddr("udp", resp.Address())
        if err != nil {
            return nil, common.NewError("freedom recv invalid socks bind addr").Base(err)
        }
        // 返回 SOCKS5 数据包连接
        return &SocksPacketConn{
            PacketConn:  packetConn,
            socksAddr:   socksAddr,
            socksClient: socksClient,
        }, nil
    }
    // 如果没有配置使用代理，则根据配置选择 UDP 或 UDP4 网络
    network := "udp"
    if c.preferIPv4 {
        network = "udp4"
    }
    // 创建 UDP 或 UDP4 数据包连接
    udpConn, err := net.ListenPacket(network, "")
    if err != nil {
        return nil, common.NewError("freedom failed to listen udp socket").Base(err)
    }
    // 返回 UDP 或 UDP4 数据包连接
    return &PacketConn{
        UDPConn: udpConn.(*net.UDPConn),
    }, nil
// 关闭客户端连接，取消相关操作
func (c *Client) Close() error {
    // 取消操作
    c.cancel()
    // 返回空错误
    return nil
}

// 创建新的客户端连接
func NewClient(ctx context.Context, _ tunnel.Client) (*Client, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 根据配置信息创建地址
    addr := tunnel.NewAddressFromHostPort("tcp", cfg.ForwardProxy.ProxyHost, cfg.ForwardProxy.ProxyPort)
    // 创建新的上下文，并返回取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 返回新的客户端对象和空错误
    return &Client{
        ctx:          ctx,
        cancel:       cancel,
        noDelay:      cfg.TCP.NoDelay,
        keepAlive:    cfg.TCP.KeepAlive,
        preferIPv4:   cfg.TCP.PreferIPV4,
        forwardProxy: cfg.ForwardProxy.Enabled,
        proxyAddr:    addr,
        username:     cfg.ForwardProxy.Username,
        password:     cfg.ForwardProxy.Password,
    }, nil
}
```