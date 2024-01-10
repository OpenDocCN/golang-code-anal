# `v2ray-core\app\proxyman\inbound\worker.go`

```
package inbound

import (
    "context"  // 引入上下文包，用于控制goroutine的生命周期
    "sync"  // 引入同步包，用于实现并发控制
    "sync/atomic"  // 引入原子操作包，用于实现原子操作
    "time"  // 引入时间包，用于处理时间相关操作

    "v2ray.com/core/app/proxyman"  // 引入代理管理包
    "v2ray.com/core/common"  // 引入通用包
    "v2ray.com/core/common/buf"  // 引入缓冲区包
    "v2ray.com/core/common/net"  // 引入网络包
    "v2ray.com/core/common/serial"  // 引入序列化包
    "v2ray.com/core/common/session"  // 引入会话包
    "v2ray.com/core/common/signal/done"  // 引入信号包
    "v2ray.com/core/common/task"  // 引入任务包
    "v2ray.com/core/features/routing"  // 引入路由包
    "v2ray.com/core/features/stats"  // 引入统计包
    "v2ray.com/core/proxy"  // 引入代理包
    "v2ray.com/core/transport/internet"  // 引入网络传输包
    "v2ray.com/core/transport/internet/tcp"  // 引入TCP传输包
    "v2ray.com/core/transport/internet/udp"  // 引入UDP传输包
    "v2ray.com/core/transport/pipe"  // 引入管道包
)

type worker interface {
    Start() error  // 定义接口方法Start，用于启动worker
    Close() error  // 定义接口方法Close，用于关闭worker
    Port() net.Port  // 定义接口方法Port，用于获取worker监听的端口
    Proxy() proxy.Inbound  // 定义接口方法Proxy，用于获取worker的代理信息
}

type tcpWorker struct {
    address         net.Address  // 定义worker的地址
    port            net.Port  // 定义worker的端口
    proxy           proxy.Inbound  // 定义worker的代理
    stream          *internet.MemoryStreamConfig  // 定义内存流配置
    recvOrigDest    bool  // 定义是否接收原始目的地
    tag             string  // 定义标签
    dispatcher      routing.Dispatcher  // 定义调度器
    sniffingConfig  *proxyman.SniffingConfig  // 定义嗅探配置
    uplinkCounter   stats.Counter  // 定义上行计数器
    downlinkCounter stats.Counter  // 定义下行计数器

    hub internet.Listener  // 定义网络监听器

    ctx context.Context  // 定义上下文
}

func getTProxyType(s *internet.MemoryStreamConfig) internet.SocketConfig_TProxyMode {
    if s == nil || s.SocketSettings == nil {  // 如果内存流配置为空或者套接字设置为空
        return internet.SocketConfig_Off  // 返回套接字配置为关闭
    }
    return s.SocketSettings.Tproxy  // 返回套接字配置的TProxy模式
}

func (w *tcpWorker) callback(conn internet.Connection) {
    ctx, cancel := context.WithCancel(w.ctx)  // 使用worker的上下文创建一个新的上下文和取消函数
    sid := session.NewID()  // 生成一个新的会话ID
    ctx = session.ContextWithID(ctx, sid)  // 将会话ID添加到上下文中
    # 如果接收到原始目标地址信息
    if w.recvOrigDest {
        # 声明一个目标地址变量
        var dest net.Destination
        # 根据传入流的 TProxy 类型进行判断
        switch getTProxyType(w.stream) {
        # 如果是 Redirect 类型
        case internet.SocketConfig_Redirect:
            # 获取原始目标地址
            d, err := tcp.GetOriginalDestination(conn)
            # 如果获取失败，记录错误日志
            if err != nil {
                newError("failed to get original destination").Base(err).WriteToLog(session.ExportIDToError(ctx))
            } else {
                # 否则将目标地址赋值为获取到的地址
                dest = d
            }
        # 如果是 TProxy 类型
        case internet.SocketConfig_TProxy:
            # 将目标地址设置为本地地址
            dest = net.DestinationFromAddr(conn.LocalAddr())
        }
        # 如果目标地址有效
        if dest.IsValid() {
            # 则在上下文中设置出站信息
            ctx = session.ContextWithOutbound(ctx, &session.Outbound{
                Target: dest,
            })
        }
    }
    # 在上下文中设置入站信息
    ctx = session.ContextWithInbound(ctx, &session.Inbound{
        Source:  net.DestinationFromAddr(conn.RemoteAddr()),
        Gateway: net.TCPDestination(w.address, w.port),
        Tag:     w.tag,
    })
    # 创建一个会话内容对象
    content := new(session.Content)
    # 如果存在嗅探配置
    if w.sniffingConfig != nil {
        # 设置嗅探请求的相关属性
        content.SniffingRequest.Enabled = w.sniffingConfig.Enabled
        content.SniffingRequest.OverrideDestinationForProtocol = w.sniffingConfig.DestinationOverride
    }
    # 在上下文中设置会话内容
    ctx = session.ContextWithContent(ctx, content)
    # 如果存在上行或下行计数器
    if w.uplinkCounter != nil || w.downlinkCounter != nil {
        # 使用统计计数器连接包装原始连接
        conn = &internet.StatCouterConnection{
            Connection:   conn,
            ReadCounter:  w.uplinkCounter,
            WriteCounter: w.downlinkCounter,
        }
    }
    # 使用代理处理连接
    if err := w.proxy.Process(ctx, net.Network_TCP, conn, w.dispatcher); err != nil {
        # 如果处理出错，记录错误日志
        newError("connection ends").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }
    # 取消操作
    cancel()
    # 关闭连接
    if err := conn.Close(); err != nil {
        # 如果关闭连接出错，记录错误日志
        newError("failed to close connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }
// 返回代理
func (w *tcpWorker) Proxy() proxy.Inbound {
    return w.proxy
}

// 启动 TCP Worker
func (w *tcpWorker) Start() error {
    // 创建一个后台上下文
    ctx := context.Background()
    // 监听 TCP 连接
    hub, err := internet.ListenTCP(ctx, w.address, w.port, w.stream, func(conn internet.Connection) {
        // 异步执行回调函数
        go w.callback(conn)
    })
    if err != nil {
        // 如果监听失败，返回错误信息
        return newError("failed to listen TCP on ", w.port).AtWarning().Base(err)
    }
    // 将监听到的 hub 赋值给 w.hub
    w.hub = hub
    return nil
}

// 关闭 TCP Worker
func (w *tcpWorker) Close() error {
    var errors []interface{}
    if w.hub != nil {
        // 如果 hub 不为空，关闭 hub
        if err := common.Close(w.hub); err != nil {
            errors = append(errors, err)
        }
        // 如果代理不为空，关闭代理
        if err := common.Close(w.proxy); err != nil {
            errors = append(errors, err)
        }
    }
    // 如果有错误发生，返回错误信息
    if len(errors) > 0 {
        return newError("failed to close all resources").Base(newError(serial.Concat(errors...)))
    }
    // 没有错误发生，返回 nil
    return nil
}

// 返回端口号
func (w *tcpWorker) Port() net.Port {
    return w.port
}

// UDP 连接结构体
type udpConn struct {
    lastActivityTime int64 // in seconds
    reader           buf.Reader
    writer           buf.Writer
    output           func([]byte) (int, error)
    remote           net.Addr
    local            net.Addr
    done             *done.Instance
    uplink           stats.Counter
    downlink         stats.Counter
}

// 更新活动时间
func (c *udpConn) updateActivity() {
    atomic.StoreInt64(&c.lastActivityTime, time.Now().Unix())
}

// 读取多缓冲区数据
func (c *udpConn) ReadMultiBuffer() (buf.MultiBuffer, error) {
    mb, err := c.reader.ReadMultiBuffer()
    if err != nil {
        return nil, err
    }
    // 更新活动时间
    c.updateActivity()

    // 如果上行计数器不为空，增加上行数据量
    if c.uplink != nil {
        c.uplink.Add(int64(mb.Len()))
    }

    return mb, nil
}

// 读取数据
func (c *udpConn) Read(buf []byte) (int, error) {
    // 抛出未实现异常
    panic("not implemented")
}

// 写入数据
func (c *udpConn) Write(buf []byte) (int, error) {
    // 调用输出函数，获取写入数据量和错误信息
    n, err := c.output(buf)
    // 如果下行计数器不为空，增加下行数据量
    if c.downlink != nil {
        c.downlink.Add(int64(n))
    }
    // 如果没有错误，更新活动时间
    if err == nil {
        c.updateActivity()
    }
}
    # 返回变量 n 和 err
    return n, err
# 关闭 UDP 连接
func (c *udpConn) Close() error:
    # 强制关闭连接
    common.Must(c.done.Close())
    # 强制关闭写入
    common.Must(common.Close(c.writer))
    # 返回空错误
    return nil

# 获取远程地址
func (c *udpConn) RemoteAddr() net.Addr:
    # 返回连接的远程地址
    return c.remote

# 获取本地地址
func (c *udpConn) LocalAddr() net.Addr:
    # 返回连接的本地地址
    return c.local

# 设置连接的截止时间
func (*udpConn) SetDeadline(time.Time) error:
    # 返回空错误
    return nil

# 设置读取数据的截止时间
func (*udpConn) SetReadDeadline(time.Time) error:
    # 返回空错误
    return nil

# 设置写入数据的截止时间
func (*udpConn) SetWriteDeadline(time.Time) error:
    # 返回空错误
    return nil

# 定义连接标识结构
type connID struct:
    src  net.Destination
    dest net.Destination

# 定义 UDP 工作器结构
type udpWorker struct:
    sync.RWMutex
    proxy           proxy.Inbound
    hub             *udp.Hub
    address         net.Address
    port            net.Port
    tag             string
    stream          *internet.MemoryStreamConfig
    dispatcher      routing.Dispatcher
    uplinkCounter   stats.Counter
    downlinkCounter stats.Counter
    checker         *task.Periodic
    activeConn      map[connID]*udpConn

# 获取连接
func (w *udpWorker) getConnection(id connID) (*udpConn, bool):
    # 加锁
    w.Lock()
    # 延迟解锁
    defer w.Unlock()
    # 如果找到活跃连接并且连接未关闭
    if conn, found := w.activeConn[id]; found && !conn.done.Done():
        # 返回连接和 true
        return conn, true
    # 创建管道读写器
    pReader, pWriter := pipe.New(pipe.DiscardOverflow(), pipe.WithSizeLimit(16*1024))
    # 创建 UDP 连接
    conn := &udpConn{
        reader:   pReader,
        writer:   pWriter,
        output:   func(b []byte) (int, error):
            return w.hub.WriteTo(b, id.src),
        remote: &net.UDPAddr:
            IP:   id.src.Address.IP(),
            Port: int(id.src.Port),
        local: &net.UDPAddr:
            IP:   w.address.IP(),
            Port: int(w.port),
        done:     done.New(),
        uplink:   w.uplinkCounter,
        downlink: w.downlinkCounter,
    }
    # 将连接添加到活跃连接中
    w.activeConn[id] = conn
    # 更新连接活动时间
    conn.updateActivity()
    # 返回连接和 false
    return conn, false

# 回调函数
func (w *udpWorker) callback(b *buf.Buffer, source net.Destination, originalDest net.Destination):
    # 创建连接标识
    id := connID:
        src: source
    // 如果 originalDest 是有效的，则将其赋值给 id.dest
    if originalDest.IsValid() {
        id.dest = originalDest
    }
    // 调用 w.getConnection 方法获取连接和是否存在的标志
    conn, existing := w.getConnection(id)

    // 如果管道已满，则 payload 将被丢弃
    conn.writer.WriteMultiBuffer(buf.MultiBuffer{b}) // nolint: errcheck

    // 如果连接不存在
    if !existing {
        // 必须启动 checker
        common.Must(w.checker.Start())

        // 启动一个 goroutine
        go func() {
            // 创建一个空的上下文
            ctx := context.Background()
            // 创建一个新的会话 ID
            sid := session.NewID()
            // 将会话 ID 添加到上下文中
            ctx = session.ContextWithID(ctx, sid)

            // 如果 originalDest 是有效的，则将其添加到出站信息中
            if originalDest.IsValid() {
                ctx = session.ContextWithOutbound(ctx, &session.Outbound{
                    Target: originalDest,
                })
            }
            // 将入站信息添加到上下文中
            ctx = session.ContextWithInbound(ctx, &session.Inbound{
                Source:  source,
                Gateway: net.UDPDestination(w.address, w.port),
                Tag:     w.tag,
            })
            // 使用代理处理连接
            if err := w.proxy.Process(ctx, net.Network_UDP, conn, w.dispatcher); err != nil {
                newError("connection ends").Base(err).WriteToLog(session.ExportIDToError(ctx))
            }
            // 关闭连接
            conn.Close() // nolint: errcheck
            // 移除连接
            w.removeConn(id)
        }()
    }
# 从活跃连接中移除指定ID的连接
func (w *udpWorker) removeConn(id connID) {
    # 加锁，保证操作的原子性
    w.Lock()
    # 从活跃连接中删除指定ID的连接
    delete(w.activeConn, id)
    # 解锁
    w.Unlock()
}

# 处理接收到的数据包
func (w *udpWorker) handlePackets() {
    # 从消息中心接收数据包
    receive := w.hub.Receive()
    # 遍历接收到的数据包
    for payload := range receive {
        # 调用回调函数处理数据包的内容、来源和目标
        w.callback(payload.Payload, payload.Source, payload.Target)
    }
}

# 清理过期的连接
func (w *udpWorker) clean() error {
    # 获取当前时间的秒数
    nowSec := time.Now().Unix()
    # 加锁，保证操作的原子性
    w.Lock()
    # 延迟解锁
    defer w.Unlock()

    # 如果活跃连接数为0，则返回错误
    if len(w.activeConn) == 0 {
        return newError("no more connections. stopping...")
    }

    # 遍历活跃连接
    for addr, conn := range w.activeConn {
        # 如果当前时间与连接的最后活动时间相差超过8秒
        if nowSec-atomic.LoadInt64(&conn.lastActivityTime) > 8 { //TODO Timeout too small
            # 从活跃连接中删除过期的连接
            delete(w.activeConn, addr)
            # 关闭连接
            conn.Close() // nolint: errcheck
        }
    }

    # 如果活跃连接数为0，则重新创建一个容量为16的活跃连接map
    if len(w.activeConn) == 0 {
        w.activeConn = make(map[connID]*udpConn, 16)
    }

    return nil
}

# 启动UDP工作器
func (w *udpWorker) Start() error {
    # 初始化活跃连接map
    w.activeConn = make(map[connID]*udpConn, 16)
    # 创建一个后台上下文
    ctx := context.Background()
    # 监听UDP连接
    h, err := udp.ListenUDP(ctx, w.address, w.port, w.stream, udp.HubCapacity(256))
    if err != nil {
        return err
    }

    # 创建一个定时任务，每16秒执行一次清理操作
    w.checker = &task.Periodic{
        Interval: time.Second * 16,
        Execute:  w.clean,
    }

    # 设置消息中心
    w.hub = h
    # 启动处理数据包的协程
    go w.handlePackets()
    return nil
}

# 关闭UDP工作器
func (w *udpWorker) Close() error {
    # 加锁，保证操作的原子性
    w.Lock()
    # 延迟解锁
    defer w.Unlock()

    # 存储所有可能出现的错误
    var errors []interface{}

    # 如果消息中心不为空，则关闭消息中心
    if w.hub != nil {
        if err := w.hub.Close(); err != nil {
            errors = append(errors, err)
        }
    }

    # 如果定时任务不为空，则关闭定时任务
    if w.checker != nil {
        if err := w.checker.Close(); err != nil {
            errors = append(errors, err)
        }
    }

    # 关闭代理
    if err := common.Close(w.proxy); err != nil {
        errors = append(errors, err)
    }

    # 如果存在错误，则返回错误信息
    if len(errors) > 0 {
        return newError("failed to close all resources").Base(newError(serial.Concat(errors...)))
    }
    return nil
}

# 获取UDP工作器的端口
func (w *udpWorker) Port() net.Port {
    return w.port
}

# 获取UDP工作器的代理
func (w *udpWorker) Proxy() proxy.Inbound {
    return w.proxy
}
```