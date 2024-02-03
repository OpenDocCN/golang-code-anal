# `v2ray-core\app\commander\outbound.go`

```go
// +build !confonly

package commander

import (
    "context"
    "sync"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/signal/done"
    "v2ray.com/core/transport"
)

// OutboundListener is a net.Listener for listening gRPC connections.
type OutboundListener struct {
    buffer chan net.Conn  // 用于缓存连接的通道
    done   *done.Instance  // 用于通知连接关闭的实例
}

func (l *OutboundListener) add(conn net.Conn) {
    select {
    case l.buffer <- conn:  // 将连接放入缓存通道
    case <-l.done.Wait():   // 等待通道关闭
        conn.Close() // nolint: errcheck  // 关闭连接
    default:
        conn.Close() // nolint: errcheck  // 关闭连接
    }
}

// Accept implements net.Listener.
func (l *OutboundListener) Accept() (net.Conn, error) {
    select {
    case <-l.done.Wait():  // 等待通道关闭
        return nil, newError("listen closed")  // 返回错误信息
    case c := <-l.buffer:  // 从缓存通道中取出连接
        return c, nil  // 返回连接
    }
}

// Close implement net.Listener.
func (l *OutboundListener) Close() error {
    common.Must(l.done.Close())  // 关闭通知连接关闭的实例
L:
    for {
        select {
        case c := <-l.buffer:  // 从缓存通道中取出连接
            c.Close() // nolint: errcheck  // 关闭连接
        default:
            break L  // 跳出循环
        }
    }
    return nil  // 返回空错误
}

// Addr implements net.Listener.
func (l *OutboundListener) Addr() net.Addr {
    return &net.TCPAddr{  // 返回 TCP 地址
        IP:   net.IP{0, 0, 0, 0},  // IP 地址
        Port: 0,  // 端口号
    }
}

// Outbound is a outbound.Handler that handles gRPC connections.
type Outbound struct {
    tag      string  // 标签
    listener *OutboundListener  // 外发监听器
    access   sync.RWMutex  // 读写锁
    closed   bool  // 是否关闭
}

// Dispatch implements outbound.Handler.
func (co *Outbound) Dispatch(ctx context.Context, link *transport.Link) {
    co.access.RLock()  // 加读锁

    if co.closed {  // 如果已关闭
        common.Interrupt(link.Reader)  // 中断读取
        common.Interrupt(link.Writer)  // 中断写入
        co.access.RUnlock()  // 解锁
        return  // 返回
    }

    closeSignal := done.New()  // 创建关闭信号
    c := net.NewConnection(net.ConnectionInputMulti(link.Writer), net.ConnectionOutputMulti(link.Reader), net.ConnectionOnClose(closeSignal))  // 创建新连接
    co.listener.add(c)  // 添加连接
    co.access.RUnlock()  // 解锁
    <-closeSignal.Wait()  // 等待关闭信号
}
// Tag 方法实现了 outbound.Handler 接口，返回 Outbound 结构体的 tag 属性
func (co *Outbound) Tag() string {
    return co.tag
}

// Start 方法实现了 common.Runnable 接口，用于启动 Outbound 结构体
func (co *Outbound) Start() error {
    // 获取锁
    co.access.Lock()
    // 将 closed 属性设置为 false
    co.closed = false
    // 释放锁
    co.access.Unlock()
    // 返回 nil，表示启动成功
    return nil
}

// Close 方法实现了 common.Closable 接口，用于关闭 Outbound 结构体
func (co *Outbound) Close() error {
    // 获取锁
    co.access.Lock()
    // 延迟释放锁
    defer co.access.Unlock()
    // 将 closed 属性设置为 true
    co.closed = true
    // 调用 listener 的 Close 方法关闭连接
    return co.listener.Close()
}
```