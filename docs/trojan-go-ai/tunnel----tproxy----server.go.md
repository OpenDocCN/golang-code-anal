# `trojan-go\tunnel\tproxy\server.go`

```go
// 仅在构建目标为 Linux 时才编译该文件
// 构建标签为 linux
package tproxy

import (
    "context"  // 导入 context 包
    "io"  // 导入 io 包
    "net"  // 导入 net 包
    "sync"  // 导入 sync 包
    "time"  // 导入 time 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 tunnel 包
)

const MaxPacketSize = 1024 * 8  // 定义最大数据包大小为 1024 * 8

type Server struct {  // 定义 Server 结构体
    tcpListener net.Listener  // TCP 监听器
    udpListener *net.UDPConn  // UDP 连接
    packetChan  chan tunnel.PacketConn  // 通道，用于传输数据包
    timeout     time.Duration  // 超时时间
    mappingLock sync.RWMutex  // 映射锁
    mapping     map[string]*PacketConn  // 映射关系
    ctx         context.Context  // 上下文
    cancel      context.CancelFunc  // 取消函数
}

func (s *Server) Close() error {  // 关闭函数
    s.cancel()  // 取消函数
    s.tcpListener.Close()  // 关闭 TCP 监听器
    return s.udpListener.Close()  // 返回关闭 UDP 连接的结果
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {  // 接受连接函数
    conn, err := s.tcpListener.Accept()  // 接受 TCP 连接
    if err != nil {  // 如果出现错误
        select {  // 选择
        case <-s.ctx.Done():  // 如果上下文已经结束
        default:  // 默认情况
            log.Fatal(common.NewError("tproxy failed to accept connection").Base(err))  // 记录错误日志
        }
        return nil, common.NewError("tproxy failed to accept conn")  // 返回错误
    }
    dst, err := getOriginalTCPDest(conn.(*net.TCPConn))  // 获取原始 TCP 目的地
    if err != nil {  // 如果出现错误
        return nil, common.NewError("tproxy failed to obtain original address of tcp socket").Base(err)  // 返回错误
    }
    address, err := tunnel.NewAddressFromAddr("tcp", dst.String())  // 从地址创建新的地址
    common.Must(err)  // 必须成功
    log.Info("tproxy connection from", conn.RemoteAddr().String(), "metadata", dst.String())  // 记录信息日志
    return &Conn{  // 返回连接
        metadata: &tunnel.Metadata{  // 元数据
            Address: address,  // 地址
        },
        Conn: conn,  // 连接
    }, nil  // 返回连接和空错误
}

func (s *Server) packetDispatchLoop() {  // 数据包分发循环函数
    type tproxyPacketInfo struct {  // 定义 tproxyPacketInfo 结构体
        src     *net.UDPAddr  // 源地址
        dst     *net.UDPAddr  // 目的地地址
        payload []byte  // 负载数据
    }
    packetQueue := make(chan *tproxyPacketInfo, 1024)  // 创建通道，用于传输数据包信息
    # 创建一个匿名的 goroutine 函数
    go func() {
        # 无限循环，读取 UDP 数据包
        for {
            # 创建一个大小为 MaxPacketSize 的字节切片
            buf := make([]byte, MaxPacketSize)
            # 从 UDP 监听器中读取数据包，并返回读取的字节数、源地址、目标地址和可能的错误
            n, src, dst, err := ReadFromUDP(s.udpListener, buf)
            # 如果出现错误
            if err != nil {
                # 检查是否上下文已经被取消
                select {
                case <-s.ctx.Done():
                default:
                    # 如果上下文未被取消，则记录错误并终止程序
                    log.Fatal(common.NewError("tproxy failed to read from udp socket").Base(err))
                }
                # 关闭连接
                s.Close()
                return
            }
            # 记录 UDP 数据包的来源地址、元数据和大小
            log.Debug("udp packet from", src, "metadata", dst, "size", n)
            # 将数据包信息发送到 packetQueue 通道
            packetQueue <- &tproxyPacketInfo{
                src:     src,
                dst:     dst,
                payload: buf[:n],
            }
        }
    }()
}
// 接受传入的数据包，并返回一个数据包连接和可能的错误
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    // 选择一个 case 执行
    select {
    // 从 packetChan 通道中接收一个连接
    case conn := <-s.packetChan:
        // 记录日志，表示接受了 tproxy 数据包连接
        log.Info("tproxy packet conn accepted")
        // 返回连接和 nil 错误
        return conn, nil
    // 当 s.ctx 被取消时，返回一个 io.EOF 错误
    case <-s.ctx.Done():
        return nil, io.EOF
    }
}

// 创建一个新的 tproxy 服务器
func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 从配置中获取本地地址
    listenAddr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
    // 解析本地地址的 IP
    ip, err := listenAddr.ResolveIP()
    // 如果解析失败，取消上下文并返回错误
    if err != nil {
        cancel()
        return nil, common.NewError("invalid tproxy local address").Base(err)
    }
    // 监听 TCP 地址
    tcpListener, err := ListenTCP("tcp", &net.TCPAddr{
        IP:   ip,
        Port: cfg.LocalPort,
    })
    // 如果监听失败，取消上下文并返回错误
    if err != nil {
        cancel()
        return nil, common.NewError("tproxy failed to listen tcp").Base(err)
    }

    // 监听 UDP 地址
    udpListener, err := ListenUDP("udp", &net.UDPAddr{
        IP:   ip,
        Port: cfg.LocalPort,
    })
    // 如果监听失败，取消上下文并返回错误
    if err != nil {
        cancel()
        return nil, common.NewError("tproxy failed to listen udp").Base(err)
    }

    // 创建一个新的 tproxy 服务器对象
    server := &Server{
        tcpListener: tcpListener,
        udpListener: udpListener,
        ctx:         ctx,
        cancel:      cancel,
        timeout:     time.Duration(cfg.UDPTimeout) * time.Second,
        mapping:     make(map[string]*PacketConn),
        packetChan:  make(chan tunnel.PacketConn, 32),
    }
    // 启动数据包分发循环
    go server.packetDispatchLoop()
    // 记录日志，表示 tproxy 服务器正在监听
    log.Info("tproxy server listening on", tcpListener.Addr(), "(tcp)", udpListener.LocalAddr(), "(udp)")
    // 记录调试日志，表示 tproxy 服务器已创建
    log.Debug("tproxy server created")
    // 返回 tproxy 服务器对象和 nil 错误
    return server, nil
}
```