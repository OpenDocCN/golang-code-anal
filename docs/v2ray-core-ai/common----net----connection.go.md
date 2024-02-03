# `v2ray-core\common\net\connection.go`

```go
// +build !confonly

package net

import (
    "io"
    "net"
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/signal/done"
)

// ConnectionOption 定义连接选项的函数类型
type ConnectionOption func(*connection)

// ConnectionLocalAddr 设置连接的本地地址
func ConnectionLocalAddr(a net.Addr) ConnectionOption {
    return func(c *connection) {
        c.local = a
    }
}

// ConnectionRemoteAddr 设置连接的远程地址
func ConnectionRemoteAddr(a net.Addr) ConnectionOption {
    return func(c *connection) {
        c.remote = a
    }
}

// ConnectionInput 设置连接的输入流
func ConnectionInput(writer io.Writer) ConnectionOption {
    return func(c *connection) {
        c.writer = buf.NewWriter(writer)
    }
}

// ConnectionInputMulti 设置连接的多重输入流
func ConnectionInputMulti(writer buf.Writer) ConnectionOption {
    return func(c *connection) {
        c.writer = writer
    }
}

// ConnectionOutput 设置连接的输出流
func ConnectionOutput(reader io.Reader) ConnectionOption {
    return func(c *connection) {
        c.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
    }
}

// ConnectionOutputMulti 设置连接的多重输出流
func ConnectionOutputMulti(reader buf.Reader) ConnectionOption {
    return func(c *connection) {
        c.reader = &buf.BufferedReader{Reader: reader}
    }
}

// ConnectionOutputMultiUDP 设置连接的多重 UDP 输出流
func ConnectionOutputMultiUDP(reader buf.Reader) ConnectionOption {
    return func(c *connection) {
        c.reader = &buf.BufferedReader{
            Reader:  reader,
            Spliter: buf.SplitFirstBytes,
        }
    }
}

// ConnectionOnClose 设置连接关闭时的操作
func ConnectionOnClose(n io.Closer) ConnectionOption {
    return func(c *connection) {
        c.onClose = n
    }
}

// NewConnection 创建一个新的连接
func NewConnection(opts ...ConnectionOption) net.Conn {
    c := &connection{
        done: done.New(),
        local: &net.TCPAddr{
            IP:   []byte{0, 0, 0, 0},
            Port: 0,
        },
        remote: &net.TCPAddr{
            IP:   []byte{0, 0, 0, 0},
            Port: 0,
        },
    }

    for _, opt := range opts {
        opt(c)
    }

    return c
}

// connection 定义连接结构
type connection struct {
    reader  *buf.BufferedReader
    writer  buf.Writer
    done    *done.Instance
    onClose io.Closer
    local   Addr
    remote  Addr
}
// Read 方法实现了从连接中读取数据，并将其写入到字节切片中
func (c *connection) Read(b []byte) (int, error) {
    return c.reader.Read(b)
}

// ReadMultiBuffer 方法实现了从连接中读取多个缓冲区的数据
func (c *connection) ReadMultiBuffer() (buf.MultiBuffer, error) {
    return c.reader.ReadMultiBuffer()
}

// Write 方法实现了向连接中写入数据
func (c *connection) Write(b []byte) (int, error) {
    // 如果连接已关闭，则返回错误
    if c.done.Done() {
        return 0, io.ErrClosedPipe
    }

    // 计算字节切片的长度
    l := len(b)
    // 创建一个多缓冲区，用于存储字节切片的数据
    mb := make(buf.MultiBuffer, 0, l/buf.Size+1)
    // 将字节切片的数据合并到多缓冲区中
    mb = buf.MergeBytes(mb, b)
    // 将多缓冲区中的数据写入到连接中
    return l, c.writer.WriteMultiBuffer(mb)
}

// WriteMultiBuffer 方法实现了向连接中写入多个缓冲区的数据
func (c *connection) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 如果连接已关闭，则释放多缓冲区并返回错误
    if c.done.Done() {
        buf.ReleaseMulti(mb)
        return io.ErrClosedPipe
    }

    // 将多缓冲区中的数据写入到连接中
    return c.writer.WriteMultiBuffer(mb)
}

// Close 方法实现了关闭连接
func (c *connection) Close() error {
    // 强制关闭连接
    common.Must(c.done.Close())
    // 中断读取操作
    common.Interrupt(c.reader)
    // 关闭写入操作
    common.Close(c.writer)
    // 如果存在关闭回调函数，则执行关闭回调函数
    if c.onClose != nil {
        return c.onClose.Close()
    }

    return nil
}

// LocalAddr 方法实现了获取本地地址
func (c *connection) LocalAddr() net.Addr {
    return c.local
}

// RemoteAddr 方法实现了获取远程地址
func (c *connection) RemoteAddr() net.Addr {
    return c.remote
}

// SetDeadline 方法实现了设置连接的截止时间
func (c *connection) SetDeadline(t time.Time) error {
    return nil
}

// SetReadDeadline 方法实现了设置连接的读取截止时间
func (c *connection) SetReadDeadline(t time.Time) error {
    return nil
}

// SetWriteDeadline 方法实现了设置连接的写入截止时间
func (c *connection) SetWriteDeadline(t time.Time) error {
    return nil
}
```