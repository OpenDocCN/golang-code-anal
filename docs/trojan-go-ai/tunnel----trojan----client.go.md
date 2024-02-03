# `trojan-go\tunnel\trojan\client.go`

```go
package trojan

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于控制goroutine的生命周期
    "net"  // 导入 net 包，用于网络通信
    "sync"  // 导入 sync 包，用于同步操作
    "sync/atomic"  // 导入 sync/atomic 包，用于原子操作
    "time"  // 导入 time 包，用于时间相关操作

    "github.com/p4gefau1t/trojan-go/api"  // 导入 trojan-go 的 api 包
    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 的 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 trojan-go 的 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 trojan-go 的 log 包
    "github.com/p4gefau1t/trojan-go/statistic"  // 导入 trojan-go 的 statistic 包
    "github.com/p4gefau1t/trojan-go/statistic/memory"  // 导入 trojan-go 的 statistic/memory 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 的 tunnel 包
    "github.com/p4gefau1t/trojan-go/tunnel/mux"  // 导入 trojan-go 的 tunnel/mux 包
)

const (
    MaxPacketSize = 1024 * 8  // 定义最大数据包大小
)

const (
    Connect   tunnel.Command = 1  // 定义连接命令
    Associate tunnel.Command = 3  // 定义关联命令
    Mux       tunnel.Command = 0x7f  // 定义多路复用命令
)

type OutboundConn struct {
    // WARNING: do not change the order of these fields.
    // 64-bit fields that use `sync/atomic` package functions
    // must be 64-bit aligned on 32-bit systems.
    // Reference: https://github.com/golang/go/issues/599
    // Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
    sent uint64  // 发送数据计数，使用原子操作保证线程安全
    recv uint64  // 接收数据计数，使用原子操作保证线程安全

    metadata          *tunnel.Metadata  // 元数据
    user              statistic.User  // 用户统计信息
    headerWrittenOnce sync.Once  // 保证头部只写一次的同步锁
    net.Conn
}

func (c *OutboundConn) Metadata() *tunnel.Metadata {
    return c.metadata  // 返回连接的元数据
}

func (c *OutboundConn) WriteHeader(payload []byte) (bool, error) {
    var err error  // 定义错误变量
    written := false  // 初始化写入状态为 false
    c.headerWrittenOnce.Do(func() {  // 使用 sync.Once 确保头部只写一次
        hash := c.user.Hash()  // 获取用户哈希值
        buf := bytes.NewBuffer(make([]byte, 0, MaxPacketSize))  // 创建一个缓冲区
        crlf := []byte{0x0d, 0x0a}  // 定义回车换行符
        buf.Write([]byte(hash))  // 写入用户哈希值
        buf.Write(crlf)  // 写入回车换行符
        c.metadata.WriteTo(buf)  // 将元数据写入缓冲区
        buf.Write(crlf)  // 写入回车换行符
        if payload != nil {  // 如果有有效载荷
            buf.Write(payload)  // 写入有效载荷
        }
        _, err = c.Conn.Write(buf.Bytes())  // 将缓冲区内容写入连接
        if err == nil {  // 如果写入成功
            written = true  // 更新写入状态为 true
        }
    })
    return written, err  // 返回写入状态和错误
}

func (c *OutboundConn) Write(p []byte) (int, error) {
    written, err := c.WriteHeader(p)  // 调用写头部函数
    if err != nil {  // 如果有错误
        return 0, common.NewError("trojan failed to flush header with payload").Base(err)  // 返回错误信息
    }
    # 如果数据已经被写入，则返回写入的数据长度和空错误
    if written:
        return len(p), nil
    # 否则，使用连接对象的Write方法将数据p写入到连接中，并返回写入的字节数和可能的错误
    n, err := c.Conn.Write(p)
    # 将写入的字节数添加到用户的流量统计中
    c.user.AddTraffic(n, 0)
    # 使用原子操作将写入的字节数添加到连接对象的sent属性中
    atomic.AddUint64(&c.sent, uint64(n))
    # 返回写入的字节数和可能的错误
    return n, err
// 读取数据到指定的字节切片中，并返回读取的字节数和可能出现的错误
func (c *OutboundConn) Read(p []byte) (int, error) {
    // 从连接中读取数据到指定的字节切片中
    n, err := c.Conn.Read(p)
    // 将读取的数据量添加到用户的流量统计中
    c.user.AddTraffic(0, n)
    // 原子性地将读取的数据量添加到接收流量统计中
    atomic.AddUint64(&c.recv, uint64(n))
    // 返回读取的字节数和可能出现的错误
    return n, err
}

// 关闭连接并返回可能出现的错误
func (c *OutboundConn) Close() error {
    // 记录连接关闭的日志信息，包括元数据和发送、接收的流量信息
    log.Info("connection to", c.metadata, "closed", "sent:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.sent)), "recv:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.recv)))
    // 关闭连接并返回可能出现的错误
    return c.Conn.Close()
}

// 客户端结构体
type Client struct {
    underlay tunnel.Client
    user     statistic.User
    ctx      context.Context
    cancel   context.CancelFunc
}

// 关闭客户端并返回可能出现的错误
func (c *Client) Close() error {
    // 取消客户端的上下文
    c.cancel()
    // 关闭底层连接并返回可能出现的错误
    return c.underlay.Close()
}

// 拨号连接到指定地址，并返回连接和可能出现的错误
func (c *Client) DialConn(addr *tunnel.Address, overlay tunnel.Tunnel) (tunnel.Conn, error) {
    // 拨号连接到指定地址
    conn, err := c.underlay.DialConn(addr, &Tunnel{})
    if err != nil {
        return nil, err
    }
    // 创建新的出站连接
    newConn := &OutboundConn{
        Conn: conn,
        user: c.user,
        metadata: &tunnel.Metadata{
            Command: Connect,
            Address: addr,
        },
    }
    // 如果覆盖层是多路复用隧道，则修改元数据的命令为多路复用
    if _, ok := overlay.(*mux.Tunnel); ok {
        newConn.metadata.Command = Mux
    }

    // 启动一个 goroutine，用于在100毫秒后刷新 Trojan 头部
    go func(newConn *OutboundConn) {
        // 如果 Trojan 头部在100毫秒后仍然被缓冲，客户端可能期望从服务器接收数据，因此刷新 Trojan 头部
        time.Sleep(time.Millisecond * 100)
        newConn.WriteHeader(nil)
    }(newConn)
    // 返回新的出站连接和可能出现的错误
    return newConn, nil
}

// 拨号数据包连接，并返回数据包连接和可能出现的错误
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    // 创建一个虚假地址
    fakeAddr := &tunnel.Address{
        DomainName:  "UDP_CONN",
        AddressType: tunnel.DomainName,
    }
    // 拨号连接到虚假地址
    conn, err := c.underlay.DialConn(fakeAddr, &Tunnel{})
    if err != nil {
        return nil, err
    }
    // 返回数据包连接和可能出现的错误
    return &PacketConn{
        Conn: &OutboundConn{
            Conn: conn,
            user: c.user,
            metadata: &tunnel.Metadata{
                Command: Associate,
                Address: fakeAddr,
            },
        },
    }, nil
}
func NewClient(ctx context.Context, client tunnel.Client) (*Client, error) {
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 使用内存存储创建一个认证对象
    auth, err := statistic.NewAuthenticator(ctx, memory.Name)
    // 如果创建认证对象出错，则取消上下文并返回错误
    if err != nil {
        cancel()
        return nil, err
    }

    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 如果配置中启用了 API，则启动 API 服务
    if cfg.API.Enabled {
        go api.RunService(ctx, Name+"_CLIENT", auth)
    }

    // 遍历认证对象中的用户列表，取第一个用户作为当前用户
    var user statistic.User
    for _, u := range auth.ListUsers() {
        user = u
        break
    }
    // 如果没有找到有效用户，则取消上下文并返回错误
    if user == nil {
        cancel()
        return nil, common.NewError("no valid user found")
    }

    // 输出调试信息
    log.Debug("trojan client created")
    // 返回一个新的客户端对象
    return &Client{
        underlay: client,
        ctx:      ctx,
        user:     user,
        cancel:   cancel,
    }, nil
}
```