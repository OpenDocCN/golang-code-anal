# `trojan-go\tunnel\adapter\server.go`

```
package adapter

import (
    "context"  // 导入上下文包
    "net"  // 导入网络包
    "sync"  // 导入同步包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/http"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/socks"  // 导入自定义包
)

type Server struct {
    tcpListener net.Listener  // TCP监听器
    udpListener net.PacketConn  // UDP监听器
    socksConn   chan tunnel.Conn  // 用于传输socks连接的通道
    httpConn    chan tunnel.Conn  // 用于传输http连接的通道
    socksLock   sync.RWMutex  // 读写锁
    nextSocks   bool  // 下一个连接是否为socks连接
    ctx         context.Context  // 上下文
    cancel      context.CancelFunc  // 取消函数
}

func (s *Server) acceptConnLoop() {
    for {
        conn, err := s.tcpListener.Accept()  // 接受TCP连接
        if err != nil {
            select {
            case <-s.ctx.Done():  // 如果上下文结束
                log.Debug("exiting")  // 输出调试信息
                return  // 退出循环
            default:
                continue  // 继续下一次循环
            }
        }
        rewindConn := common.NewRewindConn(conn)  // 创建新的重置连接
        rewindConn.SetBufferSize(16)  // 设置缓冲区大小
        buf := [3]byte{}  // 创建长度为3的字节数组
        _, err = rewindConn.Read(buf[:])  // 读取数据到buf
        rewindConn.Rewind()  // 重置连接
        rewindConn.StopBuffering()  // 停止缓冲
        if err != nil {
            log.Error(common.NewError("failed to detect proxy protocol type").Base(err))  // 输出错误信息
            continue  // 继续下一次循环
        }
        s.socksLock.RLock()  // 读取锁
        if buf[0] == 5 && s.nextSocks {  // 如果协议类型为5且下一个连接为socks连接
            s.socksLock.RUnlock()  // 释放锁
            log.Debug("socks5 connection")  // 输出调试信息
            s.socksConn <- &freedom.Conn{  // 将连接放入socks连接通道
                Conn: rewindConn,  // 连接为重置连接
            }
        } else {
            s.socksLock.RUnlock()  // 释放锁
            log.Debug("http connection")  // 输出调试信息
            s.httpConn <- &freedom.Conn{  // 将连接放入http连接通道
                Conn: rewindConn,  // 连接为重置连接
            }
        }
    }
}

func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
    # 检查 overlay 是否为 http.Tunnel 类型，如果是则执行以下代码块
    if _, ok := overlay.(*http.Tunnel); ok:
        # 从 http 连接通道中获取连接
        select:
        case conn := <-s.httpConn:
            # 返回获取的连接和空错误
            return conn, nil
        # 如果上下文已经结束，则返回空和适配器关闭的错误
        case <-s.ctx.Done():
            return nil, common.NewError("adapter closed")
    
    # 如果 overlay 不是 http.Tunnel 类型，则检查是否为 socks.Tunnel 类型，如果是则执行以下代码块
    else if _, ok := overlay.(*socks.Tunnel); ok:
        # 加锁以确保线程安全
        s.socksLock.Lock()
        # 设置下一个 socks 连接标志为 true
        s.nextSocks = true
        # 解锁
        s.socksLock.Unlock()
        # 从 socks 连接通道中获取连接
        select:
        case conn := <-s.socksConn:
            # 返回获取的连接和空错误
            return conn, nil
        # 如果上下文已经结束，则返回空和适配器关闭的错误
        case <-s.ctx.Done():
            return nil, common.NewError("adapter closed")
    
    # 如果 overlay 既不是 http.Tunnel 类型也不是 socks.Tunnel 类型，则抛出异常
    else:
        panic("invalid overlay")
# 定义 Server 结构体的 AcceptPacket 方法，接收一个隧道对象，并返回一个隧道数据包连接和一个错误
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    # 返回一个 freedom.PacketConn 对象，其中 UDPConn 属性为 Server 结构体的 udpListener 属性转换为 net.UDPConn 类型
    return &freedom.PacketConn{
        UDPConn: s.udpListener.(*net.UDPConn),
    }, nil
}

# 定义 Server 结构体的 Close 方法，关闭服务器并返回一个错误
func (s *Server) Close() error {
    # 调用 cancel 方法取消上下文
    s.cancel()
    # 关闭 tcpListener
    s.tcpListener.Close()
    # 返回关闭 udpListener 的结果
    return s.udpListener.Close()
}

# 定义 NewServer 函数，创建一个新的服务器实例并返回该实例和一个错误
func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
    # 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    # 声明一个取消函数
    var cancel context.CancelFunc
    # 使用 WithCancel 方法创建一个新的上下文，并将取消函数赋值给 cancel
    ctx, cancel = context.WithCancel(ctx)

    # 根据配置信息创建 TCP 地址
    addr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
    # 监听 TCP 地址，返回一个 TCP 监听器和一个错误
    tcpListener, err := net.Listen("tcp", addr.String())
    # 如果监听失败，则调用 cancel 方法取消上下文，并返回一个错误
    if err != nil {
        cancel()
        return nil, common.NewError("adapter failed to create tcp listener").Base(err)
    }
    # 监听 UDP 地址，返回一个 UDP 连接和一个错误
    udpListener, err := net.ListenPacket("udp", addr.String())
    # 如果监听失败，则调用 cancel 方法取消上下文，并返回一个错误
    if err != nil {
        cancel()
        return nil, common.NewError("adapter failed to create tcp listener").Base(err)
    }
    # 创建一个 Server 实例，并初始化其属性
    server := &Server{
        tcpListener: tcpListener,
        udpListener: udpListener,
        socksConn:   make(chan tunnel.Conn, 32),
        httpConn:    make(chan tunnel.Conn, 32),
        ctx:         ctx,
        cancel:      cancel,
    }
    # 打印服务器监听的 TCP/UDP 地址信息
    log.Info("adapter listening on tcp/udp:", addr)
    # 启动接受连接的循环
    go server.acceptConnLoop()
    # 返回创建的 Server 实例和一个空的错误
    return server, nil
}
```