# `trojan-go\tunnel\dokodemo\server.go`

```go
package dokodemo

import (
    "context"  // 导入上下文包，用于控制请求的取消和超时
    "net"  // 导入网络包，用于网络通信
    "sync"  // 导入同步包，用于实现并发安全的数据访问
    "time"  // 导入时间包，用于处理时间相关的操作

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义的 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义的 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义的 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义的 tunnel 包
)

type Server struct {
    tunnel.Server  // 继承自 tunnel 包中的 Server 结构体
    tcpListener net.Listener  // TCP 监听器
    udpListener net.PacketConn  // UDP 监听器
    packetChan  chan tunnel.PacketConn  // 通道，用于传输数据包
    timeout     time.Duration  // 超时时间
    targetAddr  *tunnel.Address  // 目标地址
    mappingLock sync.Mutex  // 互斥锁，用于保护映射关系
    mapping     map[string]*PacketConn  // 映射关系，用于存储数据包
    ctx         context.Context  // 上下文
    cancel      context.CancelFunc  // 取消函数
}

func (s *Server) dispatchLoop() {
    fixedMetadata := &tunnel.Metadata{  // 创建固定的元数据
        Address: s.targetAddr,  // 设置地址为目标地址
    }
    // 无限循环，接收 UDP 数据包并处理
    for {
        // 创建一个大小为 MaxPacketSize 的字节切片
        buf := make([]byte, MaxPacketSize)
        // 从 UDP 监听器中读取数据包，并返回读取的字节数、发送方地址和可能的错误
        n, addr, err := s.udpListener.ReadFrom(buf)
        // 如果有错误发生
        if err != nil {
            // 检查是否上下文已经被取消
            select {
            case <-s.ctx.Done():
            default:
                // 如果上下文未被取消，则记录错误并终止程序
                log.Fatal(common.NewError("dokodemo failed to read from udp socket").Base(err))
            }
            return
        }
        // 记录接收到的 UDP 数据包的地址
        log.Debug("udp packet from", addr)
        // 加锁以保护映射表
        s.mappingLock.Lock()
        // 如果地址已经存在于映射表中
        if conn, found := s.mapping[addr.String()]; found {
            // 将数据包发送到对应的连接的输入通道
            conn.input <- buf[:n]
            // 解锁映射表并继续下一次循环
            s.mappingLock.Unlock()
            continue
        }
        // 创建一个新的上下文和取消函数
        ctx, cancel := context.WithCancel(s.ctx)
        // 创建一个新的 PacketConn 结构体
        conn := &PacketConn{
            input:      make(chan []byte, 16),
            output:     make(chan []byte, 16),
            metadata:   fixedMetadata,
            src:        addr,
            PacketConn: s.udpListener,
            ctx:        ctx,
            cancel:     cancel,
        }
        // 将新连接添加到映射表中
        s.mapping[addr.String()] = conn
        // 解锁映射表
        s.mappingLock.Unlock()

        // 将数据包发送到新连接的输入通道
        conn.input <- buf[:n]
        // 将新连接发送到 packetChan 通道
        s.packetChan <- conn

        // 启动一个新的 goroutine 处理输出通道的数据
        go func(conn *PacketConn) {
            for {
                select {
                case payload := <-conn.output:
                    // "Multiple goroutines may invoke methods on a Conn simultaneously."
                    // 向发送方地址写入数据包的 payload
                    _, err := s.udpListener.WriteTo(payload, conn.src)
                    // 如果发生错误，则记录错误并终止 goroutine
                    if err != nil {
                        log.Error(common.NewError("dokodemo udp write error").Base(err))
                        return
                    }
                case <-s.ctx.Done():
                    // 如果上下文被取消，则终止 goroutine
                    return
                case <-time.After(s.timeout):
                    // 如果超时，则加锁映射表，删除连接并解锁映射表，关闭连接并记录日志，最后终止 goroutine
                    s.mappingLock.Lock()
                    delete(s.mapping, conn.src.String())
                    s.mappingLock.Unlock()
                    conn.Close()
                    log.Debug("closing timeout packetConn")
                    return
                }
            }
        }(conn)
    }
# 接受传入的连接，并返回连接对象和错误信息
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    # 接受 TCP 连接
    conn, err := s.tcpListener.Accept()
    # 如果出现错误，记录错误信息并终止程序
    if err != nil {
        log.Fatal(common.NewError("dokodemo failed to accept connection").Base(err))
    }
    # 返回连接对象和空错误信息
    return &Conn{
        Conn: conn,
        targetMetadata: &tunnel.Metadata{
            Address: s.targetAddr,
        },
    }, nil
}

# 接受传入的数据包连接，并返回数据包连接对象和错误信息
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    # 使用 select 语句监听多个通道的操作，并返回第一个就绪的通道
    select {
    # 如果有数据包连接就绪，直接返回连接对象和空错误信息
    case conn := <-s.packetChan:
        return conn, nil
    # 如果上下文已经关闭，返回空连接对象和包含错误信息的错误对象
    case <-s.ctx.Done():
        return nil, common.NewError("dokodemo server closed")
    }
}

# 关闭服务器，返回错误信息
func (s *Server) Close() error {
    # 取消上下文
    s.cancel()
    # 关闭 TCP 监听器
    s.tcpListener.Close()
    # 关闭 UDP 监听器
    s.udpListener.Close()
    # 返回空错误信息
    return nil
}

# 创建新的服务器对象，返回服务器对象和错误信息
func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
    # 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    # 创建目标地址对象
    targetAddr := tunnel.NewAddressFromHostPort("tcp", cfg.TargetHost, cfg.TargetPort)
    # 创建监听地址对象
    listenAddr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)

    # 使用 TCP 协议监听指定地址
    tcpListener, err := net.Listen("tcp", listenAddr.String())
    # 如果出现错误，返回空服务器对象和包含错误信息的错误对象
    if err != nil {
        return nil, common.NewError("failed to listen tcp").Base(err)
    }
    # 使用 UDP 协议监听指定地址
    udpListener, err := net.ListenPacket("udp", listenAddr.String())
    # 如果出现错误，返回空服务器对象和包含错误信息的错误对象
    if err != nil {
        return nil, common.NewError("failed to listen udp").Base(err)
    }

    # 创建上下文和取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 创建服务器对象
    server := &Server{
        tcpListener: tcpListener,
        udpListener: udpListener,
        targetAddr:  targetAddr,
        mapping:     make(map[string]*PacketConn),
        packetChan:  make(chan tunnel.PacketConn, 32),
        timeout:     time.Second * time.Duration(cfg.UDPTimeout),
        ctx:         ctx,
        cancel:      cancel,
    }
    # 启动分发循环
    go server.dispatchLoop()
    # 返回服务器对象和空错误信息
    return server, nil
}
```