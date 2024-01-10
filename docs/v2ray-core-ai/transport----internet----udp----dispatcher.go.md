# `v2ray-core\transport\internet\udp\dispatcher.go`

```
package udp

import (
    "context"  // 上下文包，用于控制请求的生命周期
    "io"  // 输入输出包，提供了基本的输入输出功能
    "sync"  // 同步包，提供了并发安全的锁和等待组
    "time"  // 时间包，提供了时间相关的功能

    "v2ray.com/core/common/signal/done"  // V2Ray框架的信号处理包

    "v2ray.com/core/common"  // V2Ray框架的通用包
    "v2ray.com/core/common/buf"  // V2Ray框架的缓冲区包
    "v2ray.com/core/common/net"  // V2Ray框架的网络包
    "v2ray.com/core/common/protocol/udp"  // V2Ray框架的UDP协议包
    "v2ray.com/core/common/session"  // V2Ray框架的会话包
    "v2ray.com/core/common/signal"  // V2Ray框架的信号包
    "v2ray.com/core/features/routing"  // V2Ray框架的路由功能包
    "v2ray.com/core/transport"  // V2Ray框架的传输包
)

type ResponseCallback func(ctx context.Context, packet *udp.Packet)  // 定义了一个回调函数类型，用于处理响应

type connEntry struct {
    link   *transport.Link  // 传输连接
    timer  signal.ActivityUpdater  // 信号处理器，用于更新活动状态
    cancel context.CancelFunc  // 取消函数，用于取消上下文
}

type Dispatcher struct {
    sync.RWMutex  // 读写锁，用于保护并发访问
    conns      map[net.Destination]*connEntry  // 目标地址到连接条目的映射
    dispatcher routing.Dispatcher  // 路由分发器
    callback   ResponseCallback  // 响应回调函数
}

func NewDispatcher(dispatcher routing.Dispatcher, callback ResponseCallback) *Dispatcher {
    return &Dispatcher{
        conns:      make(map[net.Destination]*connEntry),  // 初始化连接映射
        dispatcher: dispatcher,  // 设置路由分发器
        callback:   callback,  // 设置响应回调函数
    }
}

func (v *Dispatcher) RemoveRay(dest net.Destination) {
    v.Lock()  // 加写锁
    defer v.Unlock()  // 延迟释放写锁
    if conn, found := v.conns[dest]; found {  // 如果目标地址存在连接
        common.Close(conn.link.Reader)  // 关闭连接的读取器
        common.Close(conn.link.Writer)  // 关闭连接的写入器
        delete(v.conns, dest)  // 从连接映射中删除目标地址
    }
}

func (v *Dispatcher) getInboundRay(ctx context.Context, dest net.Destination) *connEntry {
    v.Lock()  // 加写锁
    defer v.Unlock()  // 延迟释放写锁

    if entry, found := v.conns[dest]; found {  // 如果目标地址存在连接
        return entry  // 返回连接条目
    }

    newError("establishing new connection for ", dest).WriteToLog()  // 记录日志，表示正在建立新连接

    ctx, cancel := context.WithCancel(ctx)  // 创建一个新的上下文，并返回取消函数
    removeRay := func() {  // 定义一个移除连接的函数
        cancel()  // 取消上下文
        v.RemoveRay(dest)  // 移除目标地址的连接
    }
    timer := signal.CancelAfterInactivity(ctx, removeRay, time.Second*4)  // 在指定时间内无活动后取消上下文
    link, _ := v.dispatcher.Dispatch(ctx, dest)  // 使用路由分发器分发连接
    entry := &connEntry{  // 创建连接条目
        link:   link,  // 设置连接
        timer:  timer,  // 设置定时器
        cancel: removeRay,  // 设置取消函数
    }
    v.conns[dest] = entry  // 将连接条目添加到连接映射中
    go handleInput(ctx, entry, dest, v.callback)  // 启动处理输入的协程
    return entry  // 返回连接条目
}
func (v *Dispatcher) Dispatch(ctx context.Context, destination net.Destination, payload *buf.Buffer) {
    // 在目标字符串中添加用户信息
    newError("dispatch request to: ", destination).AtDebug().WriteToLog(session.ExportIDToError(ctx))

    // 从上下文和目标地址获取入站连接
    conn := v.getInboundRay(ctx, destination)
    // 获取输出流
    outputStream := conn.link.Writer
    // 如果输出流不为空
    if outputStream != nil {
        // 将数据写入输出流
        if err := outputStream.WriteMultiBuffer(buf.MultiBuffer{payload}); err != nil {
            // 如果写入失败，记录错误并取消连接
            newError("failed to write first UDP payload").Base(err).WriteToLog(session.ExportIDToError(ctx))
            conn.cancel()
            return
        }
    }
}

func handleInput(ctx context.Context, conn *connEntry, dest net.Destination, callback ResponseCallback) {
    // 延迟取消连接
    defer conn.cancel()

    // 获取输入流和定时器
    input := conn.link.Reader
    timer := conn.timer

    for {
        select {
        case <-ctx.Done():
            return
        default:
        }

        // 读取多缓冲区数据
        mb, err := input.ReadMultiBuffer()
        if err != nil {
            // 如果读取失败，记录错误并返回
            newError("failed to handle UDP input").Base(err).WriteToLog(session.ExportIDToError(ctx))
            return
        }
        // 更新定时器
        timer.Update()
        // 遍历多缓冲区数据，调用回调函数处理数据
        for _, b := range mb {
            callback(ctx, &udp.Packet{
                Payload: b,
                Source:  dest,
            })
        }
    }
}

type dispatcherConn struct {
    dispatcher *Dispatcher
    cache      chan *udp.Packet
    done       *done.Instance
}

func DialDispatcher(ctx context.Context, dispatcher routing.Dispatcher) (net.PacketConn, error) {
    // 创建调度连接
    c := &dispatcherConn{
        cache: make(chan *udp.Packet, 16),
        done:  done.New(),
    }

    // 创建调度器并设置回调函数
    d := NewDispatcher(dispatcher, c.callback)
    c.dispatcher = d
    return c, nil
}

func (c *dispatcherConn) callback(ctx context.Context, packet *udp.Packet) {
    select {
    case <-c.done.Wait():
        // 如果完成信号已触发，释放数据包的负载并返回
        packet.Payload.Release()
        return
    case c.cache <- packet:
        // 如果缓存通道可用，将数据包发送到缓存通道
    default:
        // 如果缓存通道已满，释放数据包的负载并返回
        packet.Payload.Release()
        return
    }
}
# 从连接中读取数据到指定的字节切片中
func (c *dispatcherConn) ReadFrom(p []byte) (int, net.Addr, error) {
    # 选择一个可用的 case，如果连接已经关闭，则返回错误
    case <-c.done.Wait():
        return 0, nil, io.EOF
    # 从缓存中读取数据包，将数据包的内容拷贝到指定的字节切片中，并返回读取的字节数和数据包的来源地址
    case packet := <-c.cache:
        n := copy(p, packet.Payload.Bytes())
        return n, &net.UDPAddr{
            IP:   packet.Source.Address.IP(),
            Port: int(packet.Source.Port),
        }, nil
    }
}

# 向连接中写入数据，并指定目标地址
func (c *dispatcherConn) WriteTo(p []byte, addr net.Addr) (int, error) {
    # 创建一个缓冲区
    buffer := buf.New()
    # 扩展缓冲区的大小
    raw := buffer.Extend(buf.Size)
    # 将数据拷贝到缓冲区中
    n := copy(raw, p)
    # 调整缓冲区的大小
    buffer.Resize(0, int32(n))

    # 创建一个上下文
    ctx := context.Background()
    # 调用调度器的 Dispatch 方法，将数据发送到指定的目标地址
    c.dispatcher.Dispatch(ctx, net.DestinationFromAddr(addr), buffer)
    # 返回写入的字节数和空错误
    return n, nil
}

# 关闭连接
func (c *dispatcherConn) Close() error {
    return c.done.Close()
}

# 获取本地地址
func (c *dispatcherConn) LocalAddr() net.Addr {
    return &net.UDPAddr{
        IP:   []byte{0, 0, 0, 0},
        Port: 0,
    }
}

# 设置连接的截止时间
func (c *dispatcherConn) SetDeadline(t time.Time) error {
    return nil
}

# 设置连接的读取截止时间
func (c *dispatcherConn) SetReadDeadline(t time.Time) error {
    return nil
}

# 设置连接的写入截止时间
func (c *dispatcherConn) SetWriteDeadline(t time.Time) error {
    return nil
}
```