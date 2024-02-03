# `trojan-go\tunnel\trojan\server.go`

```go
package trojan

import (
    "context"  // 上下文包，用于控制goroutine的取消、超时等
    "fmt"  // 格式化包，用于格式化输出
    "io"  // 输入输出包，提供了基本的输入输出功能
    "net"  // 网络包，提供了基本的网络功能
    "sync/atomic"  // 原子操作包，提供了原子操作的函数

    "github.com/p4gefau1t/trojan-go/api"  // 引入trojan-go的api包
    "github.com/p4gefau1t/trojan-go/common"  // 引入trojan-go的common包
    "github.com/p4gefau1t/trojan-go/config"  // 引入trojan-go的config包
    "github.com/p4gefau1t/trojan-go/log"  // 引入trojan-go的log包
    "github.com/p4gefau1t/trojan-go/redirector"  // 引入trojan-go的redirector包
    "github.com/p4gefau1t/trojan-go/statistic"  // 引入trojan-go的statistic包
    "github.com/p4gefau1t/trojan-go/statistic/memory"  // 引入trojan-go的memory包
    "github.com/p4gefau1t/trojan-go/statistic/mysql"  // 引入trojan-go的mysql包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 引入trojan-go的tunnel包
    "github.com/p4gefau1t/trojan-go/tunnel/mux"  // 引入trojan-go的mux包
)

// InboundConn is a trojan inbound connection
type InboundConn struct {
    // WARNING: do not change the order of these fields.
    // 64-bit fields that use `sync/atomic` package functions
    // must be 64-bit aligned on 32-bit systems.
    // Reference: https://github.com/golang/go/issues/599
    // Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
    sent uint64  // 发送的字节数，使用uint64类型保证64位对齐
    recv uint64  // 接收的字节数，使用uint64类型保证64位对齐

    net.Conn  // 网络连接
    auth     statistic.Authenticator  // 统计认证信息
    user     statistic.User  // 统计用户信息
    hash     string  // 哈希值
    metadata *tunnel.Metadata  // 隧道元数据
    ip       string  // IP地址
}

func (c *InboundConn) Metadata() *tunnel.Metadata {
    return c.metadata  // 返回隧道元数据
}

func (c *InboundConn) Write(p []byte) (int, error) {
    n, err := c.Conn.Write(p)  // 写入数据到连接
    atomic.AddUint64(&c.sent, uint64(n))  // 原子操作，累加发送字节数
    c.user.AddTraffic(n, 0)  // 用户流量统计，增加发送字节数
    return n, err  // 返回写入的字节数和错误
}

func (c *InboundConn) Read(p []byte) (int, error) {
    n, err := c.Conn.Read(p)  // 从连接读取数据
    atomic.AddUint64(&c.recv, uint64(n))  // 原子操作，累加接收字节数
    c.user.AddTraffic(0, n)  // 用户流量统计，增加接收字节数
    return n, err  // 返回读取的字节数和错误
}

func (c *InboundConn) Close() error {
    log.Info("user", c.hash, "from", c.Conn.RemoteAddr(), "tunneling to", c.metadata.Address, "closed",
        "sent:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.sent)), "recv:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.recv)))  // 记录用户信息、连接地址、隧道地址、发送和接收的流量
    c.user.DelIP(c.ip)  // 删除用户IP
    return c.Conn.Close()  // 关闭连接
}

func (c *InboundConn) Auth() error {
    userHash := [56]byte{}  // 创建长度为56的字节数组
    n, err := c.Conn.Read(userHash[:])  // 从连接读取数据到字节数组
    # 如果发生错误或者读取的字节数不等于56，则返回一个包含错误信息的错误对象
    if err != nil || n != 56 {
        return common.NewError("failed to read hash").Base(err)
    }

    # 验证用户的哈希值是否有效，并获取对应的用户信息
    valid, user := c.auth.AuthUser(string(userHash[:]))
    # 如果哈希值无效，则返回一个包含错误信息的错误对象
    if !valid {
        return common.NewError("invalid hash:" + string(userHash[:]))
    }
    # 将用户哈希值和用户信息保存到连接对象中
    c.hash = string(userHash[:])
    c.user = user

    # 解析连接的远程地址，获取IP地址
    ip, _, err := net.SplitHostPort(c.Conn.RemoteAddr().String())
    # 如果解析失败，则返回一个包含错误信息的错误对象
    if err != nil {
        return common.NewError("failed to parse host:" + c.Conn.RemoteAddr().String()).Base(err)
    }
    # 将IP地址保存到连接对象中，并检查是否添加成功
    c.ip = ip
    ok := user.AddIP(ip)
    # 如果添加IP地址失败，则返回一个包含错误信息的错误对象
    if !ok {
        return common.NewError("ip limit reached")
    }

    # 读取连接中的CRLF标志
    crlf := [2]byte{}
    _, err = io.ReadFull(c.Conn, crlf[:])
    # 如果读取失败，则返回错误对象
    if err != nil {
        return err
    }

    # 创建一个tunnel.Metadata对象，并从连接中读取元数据
    c.metadata = &tunnel.Metadata{}
    if err := c.metadata.ReadFrom(c.Conn); err != nil {
        return err
    }

    # 再次读取连接中的CRLF标志
    _, err = io.ReadFull(c.Conn, crlf[:])
    # 如果读取失败，则返回错误对象
    if err != nil {
        return err
    }
    # 没有发生错误，返回nil
    return nil
// Server 是一个特洛伊隧道服务器
type Server struct {
    auth       statistic.Authenticator  // 用于身份验证的统计信息
    redir      *redirector.Redirector   // 重定向器
    redirAddr  *tunnel.Address          // 隧道地址
    underlay   tunnel.Server            // 底层隧道服务器
    connChan   chan tunnel.Conn         // 隧道连接通道
    muxChan    chan tunnel.Conn         // 多路复用通道
    packetChan chan tunnel.PacketConn   // 数据包连接通道
    ctx        context.Context          // 上下文
    cancel     context.CancelFunc       // 取消函数
}

// Close 关闭服务器
func (s *Server) Close() error {
    // 取消上下文
    s.cancel()
    // 关闭底层隧道服务器
    return s.underlay.Close()
}

// acceptLoop 接受循环
func (s *Server) acceptLoop() {
    // 无限循环，接受传入的连接
    for {
        // 接受传入的连接
        conn, err := s.underlay.AcceptConn(&Tunnel{})
        // 如果出现错误，表示连接关闭
        if err != nil { // Closing
            // 记录错误日志
            log.Error(common.NewError("trojan failed to accept conn").Base(err))
            // 检查是否上下文已经关闭，如果是则返回
            select {
            case <-s.ctx.Done():
                return
            default:
            }
            // 继续下一次循环
            continue
        }
        // 在新的 goroutine 中处理连接
        go func(conn tunnel.Conn) {
            // 创建可回滚的连接
            rewindConn := common.NewRewindConn(conn)
            // 设置缓冲区大小为128
            rewindConn.SetBufferSize(128)
            // 在函数结束时停止缓冲
            defer rewindConn.StopBuffering()

            // 创建入站连接对象
            inboundConn := &InboundConn{
                Conn: rewindConn,
                auth: s.auth,
            }

            // 进行身份验证
            if err := inboundConn.Auth(); err != nil {
                // 回滚连接
                rewindConn.Rewind()
                // 停止缓冲
                rewindConn.StopBuffering()
                // 记录警告日志
                log.Warn(common.NewError("connection with invalid trojan header from " + rewindConn.RemoteAddr().String()).Base(err))
                // 重定向连接
                s.redir.Redirect(&redirector.Redirection{
                    RedirectTo:  s.redirAddr,
                    InboundConn: rewindConn,
                })
                return
            }

            // 停止缓冲
            rewindConn.StopBuffering()
            // 根据不同的命令处理连接
            switch inboundConn.metadata.Command {
            case Connect:
                // 如果是 MUX_CONN，则发送到 muxChan
                if inboundConn.metadata.DomainName == "MUX_CONN" {
                    s.muxChan <- inboundConn
                    log.Debug("mux(r) connection")
                } else {
                    // 否则发送到 connChan
                    s.connChan <- inboundConn
                    log.Debug("normal trojan connection")
                }

            case Associate:
                // 发送到 packetChan
                s.packetChan <- &PacketConn{
                    Conn: inboundConn,
                }
                log.Debug("trojan udp connection")
            case Mux:
                // 发送到 muxChan
                s.muxChan <- inboundConn
                log.Debug("mux connection")
            default:
                // 记录错误日志，表示未知的 trojan 命令
                log.Error(common.NewError(fmt.Sprintf("unknown trojan command %d", inboundConn.metadata.Command)))
            }
        }(conn)
    }
}

// AcceptConn 方法用于接受连接，并根据不同类型的隧道进行处理
func (s *Server) AcceptConn(nextTunnel tunnel.Tunnel) (tunnel.Conn, error) {
    switch nextTunnel.(type) {
    case *mux.Tunnel:
        // 如果是 mux.Tunnel 类型的隧道，则从 muxChan 中接收连接
        select {
        case t := <-s.muxChan:
            return t, nil
        case <-s.ctx.Done():
            return nil, common.NewError("trojan client closed")
        }
    default:
        // 如果是其他类型的隧道，则从 connChan 中接收连接
        select {
        case t := <-s.connChan:
            return t, nil
        case <-s.ctx.Done():
            return nil, common.NewError("trojan client closed")
        }
    }
}

// AcceptPacket 方法用于接受数据包连接
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    // 从 packetChan 中接收数据包连接
    select {
    case t := <-s.packetChan:
        return t, nil
    case <-s.ctx.Done():
        return nil, common.NewError("trojan client closed")
    }
}

// NewServer 方法用于创建新的服务器实例
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)

    // TODO replace this dirty code
    var auth statistic.Authenticator
    var err error
    if cfg.MySQL.Enabled {
        log.Debug("mysql enabled")
        // 如果启用了 MySQL，则创建一个基于 MySQL 的认证器
        auth, err = statistic.NewAuthenticator(ctx, mysql.Name)
    } else {
        log.Debug("auth by config file")
        // 否则，根据配置文件创建认证器
        auth, err = statistic.NewAuthenticator(ctx, memory.Name)
    }
    if err != nil {
        cancel()
        return nil, common.NewError("trojan failed to create authenticator")
    }

    // 如果启用了 API，则在新的 goroutine 中运行 API 服务
    if cfg.API.Enabled {
        go api.RunService(ctx, Name+"_SERVER", auth)
    }

    // 创建一个新的服务器实例
    redirAddr := tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)
    s := &Server{
        underlay:   underlay,
        auth:       auth,
        redirAddr:  redirAddr,
        connChan:   make(chan tunnel.Conn, 32),
        muxChan:    make(chan tunnel.Conn, 32),
        packetChan: make(chan tunnel.PacketConn, 32),
        ctx:        ctx,
        cancel:     cancel,
        redir:      redirector.NewRedirector(ctx),
    }
}
    # 如果未禁用 HTTP 检查
    if !cfg.DisableHTTPCheck {
        # 通过 TCP 连接到重定向地址
        redirConn, err := net.Dial("tcp", redirAddr.String())
        # 如果连接出现错误
        if err != nil {
            # 取消操作
            cancel()
            # 返回错误信息
            return nil, common.NewError("invalid redirect address. check your http server: " + redirAddr.String()).Base(err)
        }
        # 关闭重定向连接
        redirConn.Close()
    }

    # 启动接受循环
    go s.acceptLoop()
    # 输出调试信息
    log.Debug("trojan server created")
    # 返回服务器对象和空错误
    return s, nil
# 闭合前面的函数定义
```