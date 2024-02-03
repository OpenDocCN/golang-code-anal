# `v2ray-core\transport\internet\kcp\listener.go`

```go
// +build !confonly

package kcp

import (
    "context"
    "crypto/cipher"
    gotls "crypto/tls"
    "sync"

    goxtls "github.com/xtls/go"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
    "v2ray.com/core/transport/internet/udp"
    "v2ray.com/core/transport/internet/xtls"
)

type ConnectionID struct {
    Remote net.Address
    Port   net.Port
    Conv   uint16
}

// Listener defines a server listening for connections
type Listener struct {
    sync.Mutex
    sessions   map[ConnectionID]*Connection  // 保存连接的映射表
    hub        *udp.Hub  // UDP 通信的中心
    tlsConfig  *gotls.Config  // TLS 配置
    xtlsConfig *goxtls.Config  // XTLS 配置
    config     *Config  // KCP 配置
    reader     PacketReader  // 数据包读取器
    header     internet.PacketHeader  // 网络数据包头部
    security   cipher.AEAD  // 加密算法
    addConn    internet.ConnHandler  // 连接处理器
}

func NewListener(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, addConn internet.ConnHandler) (*Listener, error) {
    kcpSettings := streamSettings.ProtocolSettings.(*Config)  // 获取 KCP 协议配置
    header, err := kcpSettings.GetPackerHeader()  // 获取数据包头部
    if err != nil {
        return nil, newError("failed to create packet header").Base(err).AtError()  // 如果获取失败，返回错误
    }
    security, err := kcpSettings.GetSecurity()  // 获取加密算法
    if err != nil {
        return nil, newError("failed to create security").Base(err).AtError()  // 如果获取失败，返回错误
    }
    l := &Listener{
        header:   header,
        security: security,
        reader: &KCPPacketReader{
            Header:   header,
            Security: security,
        },
        sessions: make(map[ConnectionID]*Connection),  // 初始化连接映射表
        config:   kcpSettings,  // 保存 KCP 配置
        addConn:  addConn,  // 保存连接处理器
    }

    if config := tls.ConfigFromStreamSettings(streamSettings); config != nil {
        l.tlsConfig = config.GetTLSConfig()  // 获取并保存 TLS 配置
    }
    if config := xtls.ConfigFromStreamSettings(streamSettings); config != nil {
        l.xtlsConfig = config.GetXTLSConfig()  // 获取并保存 XTLS 配置
    }
}
    // 使用给定的上下文、地址、端口、流设置和 UDP Hub 容量创建 UDP 连接
    hub, err := udp.ListenUDP(ctx, address, port, streamSettings, udp.HubCapacity(1024))
    // 如果创建过程中出现错误，则返回 nil 和错误信息
    if err != nil {
        return nil, err
    }
    // 上锁以确保对 l.hub 的安全访问
    l.Lock()
    // 将创建的 UDP Hub 赋值给 l.hub
    l.hub = hub
    // 解锁
    l.Unlock()
    // 记录日志，表示正在监听给定地址和端口
    newError("listening on ", address, ":", port).WriteToLog()

    // 启动处理数据包的协程
    go l.handlePackets()

    // 返回 l 和 nil，表示监听成功
    return l, nil
}


func (l *Listener) handlePackets() {
    // 从 hub 接收数据包
    receive := l.hub.Receive()
    // 遍历接收到的数据包
    for payload := range receive {
        // 调用 OnReceive 处理接收到的数据包
        l.OnReceive(payload.Payload, payload.Source)
    }
}

func (l *Listener) OnReceive(payload *buf.Buffer, src net.Destination) {
    // 通过 reader 读取 payload 数据包的段
    segments := l.reader.Read(payload.Bytes())
    // 释放 payload 数据包
    payload.Release()

    // 如果没有读取到有效的段，则记录日志并丢弃数据包
    if len(segments) == 0 {
        newError("discarding invalid payload from ", src).WriteToLog()
        return
    }

    // 获取第一个段的会话和命令
    conv := segments[0].Conversation()
    cmd := segments[0].Command()

    // 构建 ConnectionID 结构
    id := ConnectionID{
        Remote: src.Address,
        Port:   src.Port,
        Conv:   conv,
    }

    // 加锁
    l.Lock()
    defer l.Unlock()

    // 从 sessions 中查找对应的连接
    conn, found := l.sessions[id]

    // 如果找不到对应的连接
    if !found {
        // 如果命令是 CommandTerminate，则直接返回
        if cmd == CommandTerminate {
            return
        }
        // 构建 Writer 结构
        writer := &Writer{
            id:       id,
            hub:      l.hub,
            dest:     src,
            listener: l,
        }
        // 构建远程地址和本地地址
        remoteAddr := &net.UDPAddr{
            IP:   src.Address.IP(),
            Port: int(src.Port),
        }
        localAddr := l.hub.Addr()
        // 创建新的连接
        conn = NewConnection(ConnMetadata{
            LocalAddr:    localAddr,
            RemoteAddr:   remoteAddr,
            Conversation: conv,
        }, &KCPPacketWriter{
            Header:   l.header,
            Security: l.security,
            Writer:   writer,
        }, writer, l.config)
        var netConn internet.Connection = conn
        // 如果存在 TLS 配置，则使用 TLS 进行加密
        if l.tlsConfig != nil {
            netConn = gotls.Server(conn, l.tlsConfig)
        } else if l.xtlsConfig != nil {
            netConn = goxtls.Server(conn, l.xtlsConfig)
        }

        // 添加连接到 Listener 中
        l.addConn(netConn)
        // 将新建的连接加入 sessions 中
        l.sessions[id] = conn
    }
    // 将接收到的段输入到连接中
    conn.Input(segments)
}

func (l *Listener) Remove(id ConnectionID) {
    // 加锁
    l.Lock()
    // 从 sessions 中删除指定的连接
    delete(l.sessions, id)
    l.Unlock()
}

// Close 停止监听 UDP 地址，已接受的连接不会被关闭
func (l *Listener) Close() error {
    // 关闭 hub
    l.hub.Close()

    // 加锁
    l.Lock()
    defer l.Unlock()
}
    # 遍历会话列表中的每个连接
    for _, conn := range l.sessions {
        # 启动一个新的 goroutine 来终止连接
        go conn.Terminate()
    }
    # 返回空值
    return nil
// ActiveConnections 返回当前活跃连接数
func (l *Listener) ActiveConnections() int {
    // 加锁
    l.Lock()
    // 延迟解锁
    defer l.Unlock()

    // 返回当前连接数
    return len(l.sessions)
}

// Addr 返回监听器的网络地址，返回的地址是共享的，不要修改它
func (l *Listener) Addr() net.Addr {
    // 返回监听器的网络地址
    return l.hub.Addr()
}

// Writer 结构体定义
type Writer struct {
    id       ConnectionID
    dest     net.Destination
    hub      *udp.Hub
    listener *Listener
}

// Write 方法用于向目标地址写入数据
func (w *Writer) Write(payload []byte) (int, error) {
    return w.hub.WriteTo(payload, w.dest)
}

// Close 方法用于关闭连接
func (w *Writer) Close() error {
    // 从监听器中移除连接
    w.listener.Remove(w.id)
    return nil
}

// ListenKCP 用于监听 KCP 连接
func ListenKCP(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, addConn internet.ConnHandler) (internet.Listener, error) {
    return NewListener(ctx, address, port, streamSettings, addConn)
}

// init 函数用于注册 KCP 监听器
func init() {
    common.Must(internet.RegisterTransportListener(protocolName, ListenKCP))
}
```