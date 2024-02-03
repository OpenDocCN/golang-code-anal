# `trojan-go\tunnel\socks\server.go`

```go
package socks

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "context" // 导入 context 包，用于处理上下文
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于进行 I/O 操作
    "io/ioutil" // 导入 ioutil 包，用于进行 I/O 操作
    "net" // 导入 net 包，用于网络操作
    "sync" // 导入 sync 包，用于同步操作
    "time" // 导入 time 包，用于处理时间

    "github.com/p4gefau1t/trojan-go/common" // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/config" // 导入自定义包 config
    "github.com/p4gefau1t/trojan-go/log" // 导入自定义包 log
    "github.com/p4gefau1t/trojan-go/tunnel" // 导入自定义包 tunnel
)

const (
    Connect   tunnel.Command = 1 // 定义常量 Connect，值为 1
    Associate tunnel.Command = 3 // 定义常量 Associate，值为 3
)

const (
    MaxPacketSize = 1024 * 8 // 定义常量 MaxPacketSize，值为 1024 * 8
)

type Server struct {
    connChan         chan tunnel.Conn // 定义通道 connChan，用于传输 tunnel.Conn 类型的数据
    packetChan       chan tunnel.PacketConn // 定义通道 packetChan，用于传输 tunnel.PacketConn 类型的数据
    underlay         tunnel.Server // 定义字段 underlay，类型为 tunnel.Server
    localHost        string // 定义字段 localHost，类型为 string
    localPort        int // 定义字段 localPort，类型为 int
    timeout          time.Duration // 定义字段 timeout，类型为 time.Duration
    listenPacketConn tunnel.PacketConn // 定义字段 listenPacketConn，类型为 tunnel.PacketConn
    mapping          map[string]*PacketConn // 定义字段 mapping，类型为 map[string]*PacketConn
    mappingLock      sync.RWMutex // 定义字段 mappingLock，类型为 sync.RWMutex
    ctx              context.Context // 定义字段 ctx，类型为 context.Context
    cancel           context.CancelFunc // 定义字段 cancel，类型为 context.CancelFunc
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    select {
    case conn := <-s.connChan: // 从 connChan 通道中接收数据，并赋值给 conn
        return conn, nil // 返回 conn 和 nil
    case <-s.ctx.Done(): // 当 s.ctx 完成时
        return nil, common.NewError("socks server closed") // 返回 nil 和 "socks server closed" 的错误
    }
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    select {
    case conn := <-s.packetChan: // 从 packetChan 通道中接收数据，并赋值给 conn
        return conn, nil // 返回 conn 和 nil
    case <-s.ctx.Done(): // 当 s.ctx 完成时
        return nil, common.NewError("socks server closed") // 返回 nil 和 "socks server closed" 的错误
    }
}

func (s *Server) Close() error {
    s.cancel() // 调用 cancel 方法
    return s.underlay.Close() // 返回 underlay 的 Close 方法的结果
}

func (s *Server) handshake(conn net.Conn) (*Conn, error) {
    version := [1]byte{} // 定义长度为 1 的字节数组 version
    if _, err := conn.Read(version[:]); err != nil { // 从 conn 中读取数据到 version，如果出错
        return nil, common.NewError("failed to read socks version").Base(err) // 返回 nil 和读取 socks 版本失败的错误
    }
    if version[0] != 5 { // 如果 version 的第一个元素不等于 5
        return nil, common.NewError(fmt.Sprintf("invalid socks version %d", version[0])) // 返回 nil 和 socks 版本无效的错误
    }
    nmethods := [1]byte{} // 定义长度为 1 的字节数组 nmethods
    if _, err := conn.Read(nmethods[:]); err != nil { // 从 conn 中读取数据到 nmethods，如果出错
        return nil, common.NewError("failed to read NMETHODS") // 返回 nil 和读取 NMETHODS 失败的错误
    }
    # 从连接中读取指定长度的数据并丢弃，如果出现错误则返回错误信息
    if _, err := io.CopyN(ioutil.Discard, conn, int64(nmethods[0])); err != nil:
        return nil, common.NewError("socks failed to read methods").Base(err)
    # 向连接中写入指定的字节，如果出现错误则返回错误信息
    if _, err := conn.Write([]byte{0x5, 0x0}); err != nil:
        return nil, common.NewError("failed to respond auth").Base(err)

    # 从连接中读取指定长度的数据到缓冲区，如果出现错误则返回错误信息
    buf := [3]byte{}
    if _, err := conn.Read(buf[:]); err != nil:
        return nil, common.NewError("failed to read command")

    # 创建一个新的地址对象，并从连接中读取地址信息，如果出现错误则返回错误信息
    addr := new(tunnel.Address)
    if err := addr.ReadFrom(conn); err != nil:
        return nil, err

    # 返回一个连接对象和空的错误信息
    return &Conn{
        metadata: &tunnel.Metadata{
            Command: tunnel.Command(buf[1]),
            Address: addr,
        },
        Conn: conn,
    }, nil
# 服务器端连接函数，向连接写入特定字节流，表示连接成功
func (s *Server) connect(conn net.Conn) error {
    _, err := conn.Write([]byte{0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00})
    return err
}

# 服务器端关联函数，向连接写入特定字节流，表示关联地址
func (s *Server) associate(conn net.Conn, addr *tunnel.Address) error {
    # 创建一个字节缓冲区，写入特定字节流和地址信息
    buf := bytes.NewBuffer([]byte{0x05, 0x00, 0x00})
    common.Must(addr.WriteTo(buf))
    # 将缓冲区中的字节流写入连接
    _, err := conn.Write(buf.Bytes())
    return err
}

# 服务器端数据包分发循环函数
func (s *Server) packetDispatchLoop() {
    # 空函数，用于数据包分发循环
}

# 服务器端接受循环函数
func (s *Server) acceptLoop() {
    # 空函数，用于接受循环
}
    # 无限循环，接受新的连接
    for {
        # 接受新的连接
        conn, err := s.underlay.AcceptConn(&Tunnel{})
        # 如果出现错误，记录错误信息并返回
        if err != nil {
            log.Error(common.NewError("socks accept err").Base(err))
            return
        }
        # 在新的 goroutine 中处理连接
        go func(conn net.Conn) {
            # 进行握手，建立新的连接
            newConn, err := s.handshake(conn)
            # 如果握手失败，记录错误信息并返回
            if err != nil {
                log.Error(common.NewError("socks failed to handshake with client").Base(err))
                return
            }
            # 记录连接信息
            log.Info("socks connection from", conn.RemoteAddr(), "metadata", newConn.metadata.String())
            # 根据不同的命令类型进行处理
            switch newConn.metadata.Command {
            # 处理 Connect 命令
            case Connect:
                # 如果连接失败，记录错误信息并关闭连接
                if err := s.connect(newConn); err != nil {
                    log.Error(common.NewError("socks failed to respond CONNECT").Base(err))
                    newConn.Close()
                    return
                }
                # 将新的连接发送到连接通道
                s.connChan <- newConn
                return
            # 处理 Associate 命令
            case Associate:
                # 延迟关闭连接
                defer newConn.Close()
                # 创建关联地址
                associateAddr := tunnel.NewAddressFromHostPort("udp", s.localHost, s.localPort)
                # 如果关联失败，记录错误信息并返回
                if err := s.associate(newConn, associateAddr); err != nil {
                    log.Error(common.NewError("socks failed to respond to associate request").Base(err))
                    return
                }
                # 读取数据并记录日志
                buf := [16]byte{}
                newConn.Read(buf[:])
                log.Debug("socks udp session ends")
            # 处理其他未知命令
            default:
                # 记录错误信息并关闭连接
                log.Error(common.NewError(fmt.Sprintf("unknown socks command %d", newConn.metadata.Command)))
                newConn.Close()
            }
        }(conn)
    }
// NewServer 函数用于创建一个 SOCKS 服务器
func NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 从底层服务器接受数据包
    listenPacketConn, err := underlay.AcceptPacket(&Tunnel{})
    if err != nil {
        // 如果出现错误，返回错误信息
        return nil, common.NewError("socks failed to listen packet from underlying server")
    }
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 创建一个服务器对象
    server := &Server{
        underlay:         underlay,
        ctx:              ctx,
        cancel:           cancel,
        connChan:         make(chan tunnel.Conn, 32),
        packetChan:       make(chan tunnel.PacketConn, 32),
        localHost:        cfg.LocalHost,
        localPort:        cfg.LocalPort,
        timeout:          time.Duration(cfg.UDPTimeout) * time.Second,
        listenPacketConn: listenPacketConn,
        mapping:          make(map[string]*PacketConn),
    }
    // 启动接受连接的循环
    go server.acceptLoop()
    // 启动数据包分发的循环
    go server.packetDispatchLoop()
    // 输出调试信息
    log.Debug("socks server created")
    // 返回创建的服务器对象和空错误
    return server, nil
}
```