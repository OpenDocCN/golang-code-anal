# `v2ray-core\common\buf\reader.go`

```
package buf

import (
    "io"

    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
)

// readOneUDP reads one UDP packet from the given reader and returns a Buffer.
func readOneUDP(r io.Reader) (*Buffer, error) {
    // 创建一个新的 Buffer 对象
    b := New()
    // 循环读取数据，最多尝试 64 次
    for i := 0; i < 64; i++ {
        // 从输入流中读取数据到 Buffer
        _, err := b.ReadFrom(r)
        // 如果 Buffer 不为空，则返回 Buffer 和 nil
        if !b.IsEmpty() {
            return b, nil
        }
        // 如果读取出错，则释放 Buffer 并返回错误
        if err != nil {
            b.Release()
            return nil, err
        }
    }

    // 释放 Buffer 并返回错误
    b.Release()
    return nil, newError("Reader returns too many empty payloads.")
}

// ReadBuffer reads a Buffer from the given reader.
func ReadBuffer(r io.Reader) (*Buffer, error) {
    // 创建一个新的 Buffer 对象
    b := New()
    // 从输入流中读取数据到 Buffer
    n, err := b.ReadFrom(r)
    // 如果成功读取到数据，则返回 Buffer 和错误
    if n > 0 {
        return b, err
    }
    // 释放 Buffer 并返回错误
    b.Release()
    return nil, err
}

// BufferedReader is a Reader that keeps its internal buffer.
type BufferedReader struct {
    // Reader is the underlying reader to be read from
    Reader Reader
    // Buffer is the internal buffer to be read from first
    Buffer MultiBuffer
    // Spliter is a function to read bytes from MultiBuffer
    Spliter func(MultiBuffer, []byte) (MultiBuffer, int)
}

// BufferedBytes returns the number of bytes that is cached in this reader.
func (r *BufferedReader) BufferedBytes() int32 {
    return r.Buffer.Len()
}

// ReadByte implements io.ByteReader.
func (r *BufferedReader) ReadByte() (byte, error) {
    var b [1]byte
    // 读取一个字节到数组中
    _, err := r.Read(b[:])
    return b[0], err
}

// Read implements io.Reader. It reads from internal buffer first (if available) and then reads from the underlying reader.
func (r *BufferedReader) Read(b []byte) (int, error) {
    spliter := r.Spliter
    if spliter == nil {
        spliter = SplitBytes
    }

    // 如果内部缓冲区不为空，则从内部缓冲区读取数据
    if !r.Buffer.IsEmpty() {
        buffer, nBytes := spliter(r.Buffer, b)
        r.Buffer = buffer
        if r.Buffer.IsEmpty() {
            r.Buffer = nil
        }
        return nBytes, nil
    }

    // 从底层 Reader 中读取 MultiBuffer
    mb, err := r.Reader.ReadMultiBuffer()
    if err != nil {
        return 0, err
    }

    // 从 MultiBuffer 中读取数据到数组中
    mb, nBytes := spliter(mb, b)
    # 如果字节流不为空
    if !mb.IsEmpty() {
        # 将字节流赋值给 r.Buffer
        r.Buffer = mb
    }
    # 返回读取的字节数和空指针
    return nBytes, nil
// ReadMultiBuffer 实现了 Reader 接口
func (r *BufferedReader) ReadMultiBuffer() (MultiBuffer, error) {
    // 如果缓冲区不为空，则直接返回缓冲区内容并清空缓冲区
    if !r.Buffer.IsEmpty() {
        mb := r.Buffer
        r.Buffer = nil
        return mb, nil
    }

    // 否则调用底层 Reader 的 ReadMultiBuffer 方法
    return r.Reader.ReadMultiBuffer()
}

// ReadAtMost 返回一个最多包含 size 大小的 MultiBuffer
func (r *BufferedReader) ReadAtMost(size int32) (MultiBuffer, error) {
    // 如果缓冲区为空，则从底层 Reader 读取数据到缓冲区
    if r.Buffer.IsEmpty() {
        mb, err := r.Reader.ReadMultiBuffer()
        if mb.IsEmpty() && err != nil {
            return nil, err
        }
        r.Buffer = mb
    }

    // 从缓冲区中分割出指定大小的数据，并更新缓冲区
    rb, mb := SplitSize(r.Buffer, size)
    r.Buffer = rb
    if r.Buffer.IsEmpty() {
        r.Buffer = nil
    }
    return mb, nil
}

// writeToInternal 将数据写入到内部的 writer 中
func (r *BufferedReader) writeToInternal(writer io.Writer) (int64, error) {
    // 创建一个 MultiBuffer 的写入器
    mbWriter := NewWriter(writer)
    var sc SizeCounter
    // 如果缓冲区不为空，则将缓冲区的数据写入到 writer 中
    if r.Buffer != nil {
        sc.Size = int64(r.Buffer.Len())
        if err := mbWriter.WriteMultiBuffer(r.Buffer); err != nil {
            return 0, err
        }
        r.Buffer = nil
    }

    // 从底层 Reader 中复制数据到 mbWriter 中，并返回复制的字节数
    err := Copy(r.Reader, mbWriter, CountSize(&sc))
    return sc.Size, err
}

// WriteTo 实现了 io.WriterTo 接口
func (r *BufferedReader) WriteTo(writer io.Writer) (int64, error) {
    // 调用内部的 writeToInternal 方法将数据写入到 writer 中
    nBytes, err := r.writeToInternal(writer)
    // 如果返回的错误是 io.EOF，则表示写入完成，返回写入的字节数和 nil
    if errors.Cause(err) == io.EOF {
        return nBytes, nil
    }
    return nBytes, err
}

// Interrupt 实现了 common.Interruptible 接口
func (r *BufferedReader) Interrupt() {
    // 调用 common 包中的 Interrupt 方法中断底层 Reader
    common.Interrupt(r.Reader)
}

// Close 实现了 io.Closer 接口
func (r *BufferedReader) Close() error {
    // 调用 common 包中的 Close 方法关闭底层 Reader
    return common.Close(r.Reader)
}

// SingleReader 是一个每次读取一个 Buffer 的 Reader
type SingleReader struct {
    io.Reader
}

// ReadMultiBuffer 实现了 Reader 接口
func (r *SingleReader) ReadMultiBuffer() (MultiBuffer, error) {
    // 从底层 Reader 中读取一个 Buffer，并返回包含该 Buffer 的 MultiBuffer
    b, err := ReadBuffer(r.Reader)
    return MultiBuffer{b}, err
}

// PacketReader 是一个每次读取一个 Buffer 的 Reader
type PacketReader struct {
    io.Reader
}

// ReadMultiBuffer 实现了 Reader 接口
# 从PacketReader中读取多个缓冲区数据
func (r *PacketReader) ReadMultiBuffer() (MultiBuffer, error):
    # 调用readOneUDP函数从r.Reader中读取数据到b中
    b, err := readOneUDP(r.Reader)
    # 如果读取出错，则返回nil和错误信息
    if err != nil:
        return nil, err
    # 返回包含b的MultiBuffer和nil的错误信息
    return MultiBuffer{b}, nil
```