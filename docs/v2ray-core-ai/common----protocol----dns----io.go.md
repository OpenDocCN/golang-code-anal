# `v2ray-core\common\protocol\dns\io.go`

```go
package dns

import (
    "encoding/binary"  // 导入二进制编码包
    "sync"  // 导入同步包

    "golang.org/x/net/dns/dnsmessage"  // 导入 DNS 消息包
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/buf"  // 导入缓冲区包
    "v2ray.com/core/common/serial"  // 导入序列化包
)

// PackMessage 将 DNS 消息打包成缓冲区
func PackMessage(msg *dnsmessage.Message) (*buf.Buffer, error) {
    buffer := buf.New()  // 创建新的缓冲区
    rawBytes := buffer.Extend(buf.Size)  // 扩展缓冲区大小
    packed, err := msg.AppendPack(rawBytes[:0])  // 将消息打包到缓冲区
    if err != nil {
        buffer.Release()  // 释放缓冲区
        return nil, err
    }
    buffer.Resize(0, int32(len(packed)))  // 调整缓冲区大小
    return buffer, nil
}

// MessageReader 定义消息读取接口
type MessageReader interface {
    ReadMessage() (*buf.Buffer, error)
}

// UDPReader 实现了消息读取接口
type UDPReader struct {
    buf.Reader  // 缓冲区读取器

    access sync.Mutex  // 互斥锁
    cache  buf.MultiBuffer  // 多缓冲区
}

// readCache 从缓存中读取消息
func (r *UDPReader) readCache() *buf.Buffer {
    r.access.Lock()  // 加锁
    defer r.access.Unlock()  // 延迟解锁

    mb, b := buf.SplitFirst(r.cache)  // 从多缓冲区中分离第一个缓冲区
    r.cache = mb  // 更新缓存
    return b  // 返回缓冲区
}

// refill 重新填充缓存
func (r *UDPReader) refill() error {
    mb, err := r.Reader.ReadMultiBuffer()  // 从读取器中读取多缓冲区
    if err != nil {
        return err
    }
    r.access.Lock()  // 加锁
    r.cache = mb  // 更新缓存
    r.access.Unlock()  // 解锁
    return nil
}

// ReadMessage 实现了消息读取接口
func (r *UDPReader) ReadMessage() (*buf.Buffer, error) {
    for {
        b := r.readCache()  // 从缓存中读取消息
        if b != nil {
            return b, nil
        }
        if err := r.refill(); err != nil {
            return nil, err
        }
    }
}

// Close 实现了通用的关闭接口
func (r *UDPReader) Close() error {
    defer func() {
        r.access.Lock()  // 加锁
        buf.ReleaseMulti(r.cache)  // 释放多缓冲区
        r.cache = nil  // 置空缓存
        r.access.Unlock()  // 解锁
    }()

    return common.Close(r.Reader)  // 关闭读取器
}

// TCPReader 实现了 TCP 消息读取器
type TCPReader struct {
    reader *buf.BufferedReader  // 缓冲区读取器
}

// NewTCPReader 创建一个新的 TCP 消息读取器
func NewTCPReader(reader buf.Reader) *TCPReader {
    return &TCPReader{
        reader: &buf.BufferedReader{
            Reader: reader,
        },
    }
}

// ReadMessage 实现了消息读取接口
func (r *TCPReader) ReadMessage() (*buf.Buffer, error) {
    size, err := serial.ReadUint16(r.reader)  // 从读取器中读取 16 位整数
    if err != nil {
        return nil, err
    }
    # 如果消息大小大于缓冲区大小，则返回错误
    if size > buf.Size:
        return nil, newError("message size too large: ", size)
    # 创建一个新的缓冲区
    b := buf.New()
    # 从读取器中读取指定大小的数据到缓冲区中
    if _, err := b.ReadFullFrom(r.reader, int32(size)); err != nil:
        return nil, err
    # 返回读取的缓冲区和空错误
    return b, nil
# 中断 TCP 读取器的操作
func (r *TCPReader) Interrupt() {
    common.Interrupt(r.reader)
}

# 关闭 TCP 读取器
func (r *TCPReader) Close() error {
    return common.Close(r.reader)
}

# 定义消息写入接口
type MessageWriter interface {
    WriteMessage(msg *buf.Buffer) error
}

# UDP 写入器结构体
type UDPWriter struct {
    buf.Writer
}

# 实现 UDP 写入器的消息写入方法
func (w *UDPWriter) WriteMessage(b *buf.Buffer) error {
    return w.WriteMultiBuffer(buf.MultiBuffer{b})
}

# TCP 写入器结构体
type TCPWriter struct {
    buf.Writer
}

# 实现 TCP 写入器的消息写入方法
func (w *TCPWriter) WriteMessage(b *buf.Buffer) error {
    # 如果消息为空，则返回空
    if b.IsEmpty() {
        return nil
    }

    # 创建一个空的多缓冲区
    mb := make(buf.MultiBuffer, 0, 2)

    # 创建一个新的缓冲区用于存储消息长度，并将消息长度写入其中
    size := buf.New()
    binary.BigEndian.PutUint16(size.Extend(2), uint16(b.Len()))
    # 将消息长度和消息内容添加到多缓冲区中
    mb = append(mb, size, b)
    # 调用写入器的多缓冲区写入方法
    return w.WriteMultiBuffer(mb)
}
```