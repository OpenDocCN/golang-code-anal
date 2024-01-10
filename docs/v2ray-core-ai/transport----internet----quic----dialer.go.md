# `v2ray-core\transport\internet\quic\dialer.go`

```
// +build !confonly
// 定义包名为 quic，表示该文件不仅仅是配置文件

package quic

import (
    "context"  // 导入 context 包，用于处理请求的取消和超时
    "sync"  // 导入 sync 包，用于实现同步操作
    "time"  // 导入 time 包，用于处理时间相关的操作

    "github.com/lucas-clemente/quic-go"  // 导入 quic-go 包，用于实现 QUIC 协议
    "v2ray.com/core/common"  // 导入 common 包，用于通用的操作
    "v2ray.com/core/common/net"  // 导入 net 包，用于处理网络相关的操作
    "v2ray.com/core/common/task"  // 导入 task 包，用于处理任务相关的操作
    "v2ray.com/core/transport/internet"  // 导入 internet 包，用于处理网络传输相关的操作
    "v2ray.com/core/transport/internet/tls"  // 导入 tls 包，用于处理 TLS 相关的操作
)

type sessionContext struct {
    rawConn *sysConn  // 定义原始连接的指针
    session quic.Session  // 定义 QUIC 会话
}

var errSessionClosed = newError("session closed")  // 定义一个错误变量，表示会话已关闭

func (c *sessionContext) openStream(destAddr net.Addr) (*interConn, error) {
    if !isActive(c.session) {  // 如果会话不活跃
        return nil, errSessionClosed  // 返回空指针和会话已关闭的错误
    }

    stream, err := c.session.OpenStream()  // 打开一个新的数据流
    if err != nil {  // 如果出现错误
        return nil, err  // 返回空指针和错误
    }

    conn := &interConn{  // 创建一个新的网络连接
        stream: stream,  // 设置数据流
        local:  c.session.LocalAddr(),  // 设置本地地址
        remote: destAddr,  // 设置目标地址
    }

    return conn, nil  // 返回网络连接和空指针
}

type clientSessions struct {
    access   sync.Mutex  // 定义互斥锁
    sessions map[net.Destination][]*sessionContext  // 定义目的地到会话上下文的映射
    cleanup  *task.Periodic  // 定义定时清理任务
}

func isActive(s quic.Session) bool {
    select {
    case <-s.Context().Done():  // 如果会话的上下文已经结束
        return false  // 返回 false
    default:
        return true  // 否则返回 true
    }
}

func removeInactiveSessions(sessions []*sessionContext) []*sessionContext {
    activeSessions := make([]*sessionContext, 0, len(sessions))  // 创建一个空的活跃会话列表
    for _, s := range sessions {  // 遍历所有会话
        if isActive(s.session) {  // 如果会话是活跃的
            activeSessions = append(activeSessions, s)  // 将会话添加到活跃会话列表中
            continue
        }
        if err := s.session.CloseWithError(0, ""); err != nil {  // 如果关闭会话时出现错误
            newError("failed to close session").Base(err).WriteToLog()  // 记录关闭会话失败的错误
        }
        if err := s.rawConn.Close(); err != nil {  // 如果关闭原始连接时出现错误
            newError("failed to close raw connection").Base(err).WriteToLog()  // 记录关闭原始连接失败的错误
        }
    }

    if len(activeSessions) < len(sessions) {  // 如果活跃会话列表长度小于原始会话列表长度
        return activeSessions  // 返回活跃会话列表
    }

    return sessions  // 否则返回原始会话列表
}

func openStream(sessions []*sessionContext, destAddr net.Addr) *interConn {
    # 遍历会话列表中的每个会话
    for _, s := range sessions:
        # 如果会话不活跃，则跳过当前循环，继续下一个会话
        if !isActive(s.session):
            continue

        # 尝试打开到目标地址的流，如果出现错误则跳过当前循环，继续下一个会话
        conn, err := s.openStream(destAddr)
        if err != nil:
            continue

        # 如果成功打开流，则返回连接
        return conn
    # 如果没有找到可用的连接，则返回空
    return nil
// 清理会话，删除不活跃的会话
func (s *clientSessions) cleanSessions() error {
    // 加锁以确保并发安全
    s.access.Lock()
    // 在函数返回时解锁
    defer s.access.Unlock()

    // 如果会话列表为空，则直接返回
    if len(s.sessions) == 0 {
        return nil
    }

    // 创建一个新的会话映射
    newSessionMap := make(map[net.Destination][]*sessionContext)

    // 遍历会话列表，删除不活跃的会话，并将活跃的会话添加到新的会话映射中
    for dest, sessions := range s.sessions {
        sessions = removeInactiveSessions(sessions)
        if len(sessions) > 0 {
            newSessionMap[dest] = sessions
        }
    }

    // 更新会话列表为新的会话映射
    s.sessions = newSessionMap
    return nil
}

// 打开连接，返回一个网络连接和可能的错误
func (s *clientSessions) openConnection(destAddr net.Addr, config *Config, tlsConfig *tls.Config, sockopt *internet.SocketConfig) (internet.Connection, error) {
    // 加锁以确保并发安全
    s.access.Lock()
    // 在函数返回时解锁
    defer s.access.Unlock()

    // 如果会话列表为空，则初始化会话列表
    if s.sessions == nil {
        s.sessions = make(map[net.Destination][]*sessionContext)
    }

    // 从目标地址获取目标信息
    dest := net.DestinationFromAddr(destAddr)

    var sessions []*sessionContext
    // 如果会话列表中存在目标地址对应的会话，则获取该会话
    if s, found := s.sessions[dest]; found {
        sessions = s
    }

    // 如果条件为真，则打开一个流连接
    if true {
        conn := openStream(sessions, destAddr)
        if conn != nil {
            return conn, nil
        }
    }

    // 删除不活跃的会话
    sessions = removeInactiveSessions(sessions)

    // 使用系统包监听 UDP 地址
    rawConn, err := internet.ListenSystemPacket(context.Background(), &net.UDPAddr{
        IP:   []byte{0, 0, 0, 0},
        Port: 0,
    }, sockopt)
    if err != nil {
        return nil, err
    }

    // 配置 QUIC 连接参数
    quicConfig := &quic.Config{
        ConnectionIDLength: 12,
        HandshakeTimeout:   time.Second * 8,
        MaxIdleTimeout:     time.Second * 30,
    }

    // 将系统连接包装成 QUIC 连接
    conn, err := wrapSysConn(rawConn, config)
    if err != nil {
        rawConn.Close()
        return nil, err
    }

    // 使用 QUIC 协议建立连接
    session, err := quic.DialContext(context.Background(), conn, destAddr, "", tlsConfig.GetTLSConfig(tls.WithDestination(dest)), quicConfig)
    if err != nil {
        conn.Close()
        return nil, err
    }

    // 创建会话上下文
    context := &sessionContext{
        session: session,
        rawConn: conn,
    }
    // 将会话上下文添加到会话列表中
    s.sessions[dest] = append(sessions, context)
    // 打开流连接
    return context.openStream(destAddr)
}
// 定义全局变量 client，用于管理会话
var client clientSessions

// 初始化函数，在程序启动时执行
func init() {
    // 初始化会话映射表
    client.sessions = make(map[net.Destination][]*sessionContext)
    // 创建定时清理任务
    client.cleanup = &task.Periodic{
        Interval: time.Minute,
        Execute:  client.cleanSessions,
    }
    // 强制执行定时清理任务
    common.Must(client.cleanup.Start())
}

// Dial 函数用于建立连接
func Dial(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
    // 根据流设置创建 TLS 配置
    tlsConfig := tls.ConfigFromStreamSettings(streamSettings)
    // 如果 TLS 配置为空，则创建一个默认的 TLS 配置
    if tlsConfig == nil {
        tlsConfig = &tls.Config{
            ServerName:    internalDomain,
            AllowInsecure: true,
        }
    }

    var destAddr *net.UDPAddr
    // 如果目标地址是 IP 地址，则直接使用目标地址和端口
    if dest.Address.Family().IsIP() {
        destAddr = &net.UDPAddr{
            IP:   dest.Address.IP(),
            Port: int(dest.Port),
        }
    } else {
        // 如果目标地址是域名，则解析为 UDP 地址
        addr, err := net.ResolveUDPAddr("udp", dest.NetAddr())
        if err != nil {
            return nil, err
        }
        destAddr = addr
    }

    // 获取传输协议配置
    config := streamSettings.ProtocolSettings.(*Config)

    // 调用 client 的 openConnection 方法建立连接
    return client.openConnection(destAddr, config, tlsConfig, streamSettings.SocketSettings)
}

// 初始化函数，注册传输协议的拨号器
func init() {
    common.Must(internet.RegisterTransportDialer(protocolName, Dial))
}
```