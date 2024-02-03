# `v2ray-core\common\buf\writer.go`

```go
package buf

import (
    "io"
    "net"
    "sync"

    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
)

// BufferToBytesWriter is a Writer that writes alloc.Buffer into underlying writer.
type BufferToBytesWriter struct {
    io.Writer

    cache [][]byte
}

// WriteMultiBuffer implements Writer. This method takes ownership of the given buffer.
func (w *BufferToBytesWriter) WriteMultiBuffer(mb MultiBuffer) error {
    defer ReleaseMulti(mb)  // 释放多个缓冲区

    size := mb.Len()  // 获取缓冲区的长度
    if size == 0 {
        return nil  // 如果长度为0，直接返回
    }

    if len(mb) == 1 {
        return WriteAllBytes(w.Writer, mb[0].Bytes())  // 如果缓冲区长度为1，直接写入数据
    }

    if cap(w.cache) < len(mb) {
        w.cache = make([][]byte, 0, len(mb))  // 如果缓存容量小于缓冲区长度，重新分配缓存
    }

    bs := w.cache  // 将缓存赋值给bs
    for _, b := range mb {
        bs = append(bs, b.Bytes())  // 将缓冲区的数据添加到bs中
    }

    defer func() {
        for idx := range bs {
            bs[idx] = nil  // 延迟释放bs中的数据
        }
    }()

    nb := net.Buffers(bs)  // 创建net.Buffers对象

    for size > 0 {
        n, err := nb.WriteTo(w.Writer)  // 将数据写入底层Writer
        if err != nil {
            return err  // 如果出错，返回错误
        }
        size -= int32(n)  // 更新剩余数据长度
    }

    return nil  // 返回nil
}

// ReadFrom implements io.ReaderFrom.
func (w *BufferToBytesWriter) ReadFrom(reader io.Reader) (int64, error) {
    var sc SizeCounter  // 创建SizeCounter对象
    err := Copy(NewReader(reader), w, CountSize(&sc))  // 复制数据到BufferToBytesWriter中，并统计数据大小
    return sc.Size, err  // 返回数据大小和错误
}

// BufferedWriter is a Writer with internal buffer.
type BufferedWriter struct {
    sync.Mutex
    writer   Writer
    buffer   *Buffer
    buffered bool
}

// NewBufferedWriter creates a new BufferedWriter.
func NewBufferedWriter(writer Writer) *BufferedWriter {
    return &BufferedWriter{
        writer:   writer,
        buffer:   New(),
        buffered: true,
    }
}

// WriteByte implements io.ByteWriter.
func (w *BufferedWriter) WriteByte(c byte) error {
    return common.Error2(w.Write([]byte{c}))  // 写入单个字节数据
}

// Write implements io.Writer.
func (w *BufferedWriter) Write(b []byte) (int, error) {
    if len(b) == 0 {
        return 0, nil  // 如果数据长度为0，直接返回
    }

    w.Lock()  // 加锁
    defer w.Unlock()  // 延迟解锁
    # 如果写入器没有缓冲
    if !w.buffered {
        # 如果写入器实现了io.Writer接口
        if writer, ok := w.writer.(io.Writer); ok {
            # 直接将数据写入到写入器中
            return writer.Write(b)
        }
    }

    # 初始化总字节数
    totalBytes := 0
    # 循环直到所有数据都被写入
    for len(b) > 0 {
        # 如果缓冲区为空
        if w.buffer == nil {
            # 创建一个新的缓冲区
            w.buffer = New()
        }

        # 将数据写入缓冲区
        nBytes, err := w.buffer.Write(b)
        # 更新总字节数
        totalBytes += nBytes
        # 如果写入出错
        if err != nil {
            # 返回错误和已写入的字节数
            return totalBytes, err
        }
        # 如果不需要缓冲或者缓冲区已满
        if !w.buffered || w.buffer.IsFull() {
            # 刷新缓冲区
            if err := w.flushInternal(); err != nil {
                # 如果刷新出错，返回错误和已写入的字节数
                return totalBytes, err
            }
        }
        # 更新剩余未写入的数据
        b = b[nBytes:]
    }

    # 返回已写入的总字节数和nil错误
    return totalBytes, nil
// WriteMultiBuffer 实现了 Writer 接口。它接管了给定的 MultiBuffer。
func (w *BufferedWriter) WriteMultiBuffer(b MultiBuffer) error {
    // 如果 MultiBuffer 为空，则返回空
    if b.IsEmpty() {
        return nil
    }

    // 加锁
    w.Lock()
    // 在函数返回时解锁
    defer w.Unlock()

    // 如果没有缓冲，则直接调用底层 writer 的 WriteMultiBuffer 方法
    if !w.buffered {
        return w.writer.WriteMultiBuffer(b)
    }

    // 创建 MultiBufferContainer 对象
    reader := MultiBufferContainer{
        MultiBuffer: b,
    }
    // 在函数返回时关闭 MultiBufferContainer
    defer reader.Close()

    // 循环读取 MultiBufferContainer 中的数据
    for !reader.MultiBuffer.IsEmpty() {
        // 如果缓冲为空，则创建一个新的缓冲
        if w.buffer == nil {
            w.buffer = New()
        }
        // 从 reader 中读取数据到缓冲中
        common.Must2(w.buffer.ReadFrom(&reader))
        // 如果缓冲已满，则刷新缓冲
        if w.buffer.IsFull() {
            if err := w.flushInternal(); err != nil {
                return err
            }
        }
    }

    return nil
}

// Flush 将缓冲的内容刷新到底层 writer 中。
func (w *BufferedWriter) Flush() error {
    // 加锁
    w.Lock()
    // 在函数返回时解锁
    defer w.Unlock()

    return w.flushInternal()
}

// flushInternal 刷新内部缓冲的内容到底层 writer 中。
func (w *BufferedWriter) flushInternal() error {
    // 如果缓冲为空，则返回空
    if w.buffer.IsEmpty() {
        return nil
    }

    // 将缓冲内容赋值给 b，并清空缓冲
    b := w.buffer
    w.buffer = nil

    // 如果底层 writer 实现了 io.Writer 接口，则直接写入缓冲内容
    if writer, ok := w.writer.(io.Writer); ok {
        err := WriteAllBytes(writer, b.Bytes())
        b.Release()
        return err
    }

    // 否则调用底层 writer 的 WriteMultiBuffer 方法
    return w.writer.WriteMultiBuffer(MultiBuffer{b})
}

// SetBuffered 设置是否使用内部缓冲。如果设置为 false，则调用 Flush() 来清空缓冲。
func (w *BufferedWriter) SetBuffered(f bool) error {
    // 加锁
    w.Lock()
    // 在函数返回时解锁
    defer w.Unlock()

    // 设置是否使用内部缓冲
    w.buffered = f
    // 如果不使用缓冲，则立即刷新缓冲
    if !f {
        return w.flushInternal()
    }
    return nil
}

// ReadFrom 实现了 io.ReaderFrom 接口。
func (w *BufferedWriter) ReadFrom(reader io.Reader) (int64, error) {
    // 如果设置不使用缓冲出现错误，则返回 0 和错误
    if err := w.SetBuffered(false); err != nil {
        return 0, err
    }

    // 创建 SizeCounter 对象
    var sc SizeCounter
    // 从 reader 中复制数据到 w 中，并统计数据大小
    err := Copy(NewReader(reader), w, CountSize(&sc))
    return sc.Size, err
}

// Close 实现了 io.Closable 接口。
func (w *BufferedWriter) Close() error {
    // 如果刷新缓冲出现错误，则返回错误
    if err := w.Flush(); err != nil {
        return err
    }
    // 调用 common 包中的 Close 方法关闭底层 writer
    return common.Close(w.writer)
}
// SequentialWriter 是一个将 MultiBuffer 顺序写入底层 io.Writer 的 Writer。
type SequentialWriter struct {
    io.Writer
}

// WriteMultiBuffer 实现了 Writer 接口。
func (w *SequentialWriter) WriteMultiBuffer(mb MultiBuffer) error {
    // 调用 WriteMultiBuffer 函数将 MultiBuffer 写入底层 io.Writer，并返回写入后的 MultiBuffer 和错误
    mb, err := WriteMultiBuffer(w.Writer, mb)
    // 释放写入后的 MultiBuffer
    ReleaseMulti(mb)
    // 返回错误
    return err
}

// noOpWriter 是一个空操作的 Writer 类型
type noOpWriter byte

// WriteMultiBuffer 实现了 Writer 接口。
func (noOpWriter) WriteMultiBuffer(b MultiBuffer) error {
    // 释放写入的 MultiBuffer
    ReleaseMulti(b)
    // 返回空错误
    return nil
}

// Write 实现了 Write 接口。
func (noOpWriter) Write(b []byte) (int, error) {
    // 返回写入的字节数和空错误
    return len(b), nil
}

// ReadFrom 实现了 ReadFrom 接口。
func (noOpWriter) ReadFrom(reader io.Reader) (int64, error) {
    // 创建一个新的 MultiBuffer
    b := New()
    // 在函数返回时释放 MultiBuffer
    defer b.Release()

    // 初始化总字节数为 0
    totalBytes := int64(0)
    // 循环从 reader 中读取数据到 MultiBuffer 中
    for {
        // 清空 MultiBuffer
        b.Clear()
        // 从 reader 中读取数据到 MultiBuffer 中，并返回读取的字节数和错误
        _, err := b.ReadFrom(reader)
        // 更新总字节数
        totalBytes += int64(b.Len())
        // 如果出现错误
        if err != nil {
            // 如果错误是 io.EOF，则表示读取结束，返回总字节数和空错误
            if errors.Cause(err) == io.EOF {
                return totalBytes, nil
            }
            // 如果不是 io.EOF 错误，则返回总字节数和错误
            return totalBytes, err
        }
    }
}

// Discard 是一个吞噬所有写入内容的 Writer
var (
    Discard Writer = noOpWriter(0)

    // DiscardBytes 是一个吞噬所有写入内容的 io.Writer
    DiscardBytes io.Writer = noOpWriter(0)
)
```