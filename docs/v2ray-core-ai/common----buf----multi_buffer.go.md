# `v2ray-core\common\buf\multi_buffer.go`

```
// buf 包用于处理缓冲区相关操作
package buf

import (
    "io"

    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/serial"
)

// ReadAllToBytes 从读取器中读取所有内容到一个字节数组中，直到 EOF
func ReadAllToBytes(reader io.Reader) ([]byte, error) {
    // 从读取器中读取内容到 MultiBuffer 中
    mb, err := ReadFrom(reader)
    if err != nil {
        return nil, err
    }
    // 如果 MultiBuffer 长度为 0，则返回空
    if mb.Len() == 0 {
        return nil, nil
    }
    // 创建一个与 MultiBuffer 长度相同的字节数组
    b := make([]byte, mb.Len())
    // 将 MultiBuffer 中的内容拷贝到字节数组中
    mb, _ = SplitBytes(mb, b)
    // 释放 MultiBuffer 中的内容
    ReleaseMulti(mb)
    return b, nil
}

// MultiBuffer 是一个 Buffer 列表，Buffer 的顺序很重要
type MultiBuffer []*Buffer

// MergeMulti 将 src 中的内容合并到 dest 中，并返回合并后的 dest 和 src
func MergeMulti(dest MultiBuffer, src MultiBuffer) (MultiBuffer, MultiBuffer) {
    // 将 src 中的内容追加到 dest 中
    dest = append(dest, src...)
    // 将 src 中的每个元素置为 nil
    for idx := range src {
        src[idx] = nil
    }
    return dest, src[:0]
}

// MergeBytes 将给定的字节数组合并到 MultiBuffer 中，并返回合并后的 MultiBuffer
func MergeBytes(dest MultiBuffer, src []byte) MultiBuffer {
    n := len(dest)
    // 如果 dest 中有元素且最后一个元素未满
    if n > 0 && !(dest)[n-1].IsFull() {
        // 将 src 中的内容写入到最后一个元素中
        nBytes, _ := (dest)[n-1].Write(src)
        src = src[nBytes:]
    }

    // 将 src 中的内容逐个写入到新的 Buffer 中，并追加到 dest 中
    for len(src) > 0 {
        b := New()
        nBytes, _ := b.Write(src)
        src = src[nBytes:]
        dest = append(dest, b)
    }

    return dest
}

// ReleaseMulti 释放 MultiBuffer 中的所有内容，并返回一个空的 MultiBuffer
func ReleaseMulti(mb MultiBuffer) MultiBuffer {
    // 释放 MultiBuffer 中每个元素的内容，并将其置为 nil
    for i := range mb {
        mb[i].Release()
        mb[i] = nil
    }
    return mb[:0]
}

// Copy 将 MultiBuffer 的起始部分拷贝到给定的字节数组中
func (mb MultiBuffer) Copy(b []byte) int {
    total := 0
    // 遍历 MultiBuffer 中的每个 Buffer
    for _, bb := range mb {
        // 将 Buffer 中的内容拷贝到字节数组中
        nBytes := copy(b[total:], bb.Bytes())
        total += nBytes
        // 如果拷贝的字节数小于 Buffer 的长度，则停止拷贝
        if int32(nBytes) < bb.Len() {
            break
        }
    }
    return total
}

// ReadFrom 从读取器中读取所有内容直到 EOF
// 从输入的 io.Reader 中读取数据到 MultiBuffer 中
func ReadFrom(reader io.Reader) (MultiBuffer, error) {
    // 创建一个初始容量为 16 的 MultiBuffer
    mb := make(MultiBuffer, 0, 16)
    // 循环读取数据
    for {
        // 创建一个新的 Buffer
        b := New()
        // 从 reader 中读取数据到 Buffer 中
        _, err := b.ReadFullFrom(reader, Size)
        // 如果 Buffer 为空，则释放它
        if b.IsEmpty() {
            b.Release()
        } else {
            // 否则将 Buffer 添加到 MultiBuffer 中
            mb = append(mb, b)
        }
        // 如果出现错误
        if err != nil {
            // 如果是 EOF 或者 ErrUnexpectedEOF，则返回 MultiBuffer 和 nil
            if errors.Cause(err) == io.EOF || errors.Cause(err) == io.ErrUnexpectedEOF {
                return mb, nil
            }
            // 否则返回 MultiBuffer 和错误
            return mb, err
        }
    }
}

// 从 MultiBuffer 中分割指定数量的字节
// 返回剩余的 MultiBuffer 和写入字节切片的字节数
func SplitBytes(mb MultiBuffer, b []byte) (MultiBuffer, int) {
    totalBytes := 0
    endIndex := -1
    // 遍历 MultiBuffer
    for i := range mb {
        // 获取当前 Buffer
        pBuffer := mb[i]
        // 从 Buffer 中读取数据到字节切片中
        nBytes, _ := pBuffer.Read(b)
        // 累加读取的字节数
        totalBytes += nBytes
        // 更新字节切片
        b = b[nBytes:]
        // 如果 Buffer 不为空
        if !pBuffer.IsEmpty() {
            endIndex = i
            break
        }
        // 释放 Buffer
        pBuffer.Release()
        mb[i] = nil
    }

    // 根据 endIndex 更新 MultiBuffer
    if endIndex == -1 {
        mb = mb[:0]
    } else {
        mb = mb[endIndex:]
    }

    return mb, totalBytes
}

// 从 MultiBuffer 中分割第一个 Buffer，并将其内容复制到给定的切片中
func SplitFirstBytes(mb MultiBuffer, p []byte) (MultiBuffer, int) {
    // 分割第一个 Buffer
    mb, b := SplitFirst(mb)
    // 如果 b 为空，则返回原始的 MultiBuffer 和 0
    if b == nil {
        return mb, 0
    }
    // 复制 Buffer 的内容到给定的切片中
    n := copy(p, b.Bytes())
    // 释放 Buffer
    b.Release()
    return mb, n
}

// 合并给定 MultiBuffer 中所有内容，返回新的 MultiBuffer
func Compact(mb MultiBuffer) MultiBuffer {
    // 如果 MultiBuffer 为空，则直接返回
    if len(mb) == 0 {
        return mb
    }

    // 创建一个新的 MultiBuffer
    mb2 := make(MultiBuffer, 0, len(mb))
    // 获取第一个 Buffer
    last := mb[0]

    // 遍历 MultiBuffer 中的每个 Buffer
    for i := 1; i < len(mb); i++ {
        // 获取当前的 Buffer
        curr := mb[i]
        // 如果当前 Buffer 和上一个 Buffer 的长度之和大于 Size
        if last.Len()+curr.Len() > Size {
            // 将上一个 Buffer 添加到新的 MultiBuffer 中
            mb2 = append(mb2, last)
            // 更新上一个 Buffer 为当前的 Buffer
            last = curr
        } else {
            // 否则将当前 Buffer 的内容读取到上一个 Buffer 中
            common.Must2(last.ReadFrom(curr))
            // 释放当前 Buffer
            curr.Release()
        }
    }
    # 将 last 添加到 mb2 中
    mb2 = append(mb2, last)
    # 返回更新后的 mb2
    return mb2
// SplitFirst 从 MultiBuffer 的开头分割出第一个 Buffer。
func SplitFirst(mb MultiBuffer) (MultiBuffer, *Buffer) {
    // 如果 MultiBuffer 为空，则返回原 MultiBuffer 和 nil
    if len(mb) == 0 {
        return mb, nil
    }

    // 获取第一个 Buffer
    b := mb[0]
    // 将第一个 Buffer 置为 nil
    mb[0] = nil
    // 从 MultiBuffer 中移除第一个 Buffer
    mb = mb[1:]
    // 返回剩余的 MultiBuffer 和被分割出的 Buffer
    return mb, b
}

// SplitSize 将 MultiBuffer 的开头分割成另一个 MultiBuffer，最多包含 size 字节。
func SplitSize(mb MultiBuffer, size int32) (MultiBuffer, MultiBuffer) {
    // 如果 MultiBuffer 为空，则返回原 MultiBuffer 和 nil
    if len(mb) == 0 {
        return mb, nil
    }

    // 如果第一个 Buffer 的长度大于 size
    if mb[0].Len() > size {
        // 创建一个新的 Buffer
        b := New()
        // 将第一个 Buffer 的前 size 字节复制到新的 Buffer 中
        copy(b.Extend(size), mb[0].BytesTo(size))
        // 将第一个 Buffer 的指针向后移动 size 字节
        mb[0].Advance(size)
        // 返回剩余的 MultiBuffer 和包含新 Buffer 的 MultiBuffer
        return mb, MultiBuffer{b}
    }

    // 计算 MultiBuffer 中总共的字节数
    totalBytes := int32(0)
    var r MultiBuffer
    endIndex := -1
    for i := range mb {
        // 如果加上当前 Buffer 的长度后超过了 size
        if totalBytes+mb[i].Len() > size {
            endIndex = i
            break
        }
        // 累加当前 Buffer 的长度
        totalBytes += mb[i].Len()
        // 将当前 Buffer 添加到新的 MultiBuffer 中
        r = append(r, mb[i])
        // 将当前 Buffer 置为 nil
        mb[i] = nil
    }
    // 如果 endIndex 为 -1，表示没有 Buffer 被添加到新的 MultiBuffer 中
    if endIndex == -1 {
        // 为了重用 mb 数组，将其长度置为 0
        mb = mb[:0]
    } else {
        // 否则，将 mb 中 endIndex 之后的 Buffer 作为剩余的 MultiBuffer
        mb = mb[endIndex:]
    }
    // 返回剩余的 MultiBuffer 和被分割出的 MultiBuffer
    return mb, r
}

// WriteMultiBuffer 逐个将 MultiBuffer 中的所有 Buffer 写入到 Writer 中，如果有错误则返回错误和剩余的 MultiBuffer。
func WriteMultiBuffer(writer io.Writer, mb MultiBuffer) (MultiBuffer, error) {
    // 循环处理 MultiBuffer 中的每个 Buffer
    for {
        // 分割出第一个 Buffer
        mb2, b := SplitFirst(mb)
        mb = mb2
        // 如果没有 Buffer 了，则退出循环
        if b == nil {
            break
        }

        // 将 Buffer 的内容写入到 Writer 中
        _, err := writer.Write(b.Bytes())
        // 释放 Buffer
        b.Release()
        // 如果有错误，则返回剩余的 MultiBuffer 和错误
        if err != nil {
            return mb, err
        }
    }

    // 写入完成后，返回 nil 和 nil
    return nil, nil
}

// Len 返回 MultiBuffer 中所有 Buffer 的总字节数。
func (mb MultiBuffer) Len() int32 {
    // 如果 MultiBuffer 为空，则返回 0
    if mb == nil {
        return 0
    }

    // 计算 MultiBuffer 中所有 Buffer 的总字节数
    size := int32(0)
    for _, b := range mb {
        size += b.Len()
    }
    return size
}

// IsEmpty 如果 MultiBuffer 没有内容，则返回 true。
func (mb MultiBuffer) IsEmpty() bool {
    // 遍历 MultiBuffer 中的每个 Buffer
    for _, b := range mb {
        // 如果有任何一个 Buffer 不为空，则返回 false
        if !b.IsEmpty() {
            return false
        }
    }
    // 如果所有的 Buffer 都为空，则返回 true
    return true
}
// String方法将MultiBuffer中的内容转换为字符串并返回
func (mb MultiBuffer) String() string {
    // 创建一个与MultiBuffer长度相同的接口切片
    v := make([]interface{}, len(mb))
    // 遍历MultiBuffer，将每个元素存入接口切片
    for i, b := range mb {
        v[i] = b
    }
    // 调用serial.Concat方法将接口切片中的内容连接成字符串并返回
    return serial.Concat(v...)
}

// MultiBufferContainer是对MultiBuffer的ReadWriteCloser包装
type MultiBufferContainer struct {
    MultiBuffer
}

// Read实现了io.Reader接口
func (c *MultiBufferContainer) Read(b []byte) (int, error) {
    // 如果MultiBuffer为空，则返回0和io.EOF
    if c.MultiBuffer.IsEmpty() {
        return 0, io.EOF
    }
    // 调用SplitBytes方法从MultiBuffer中读取数据到b中
    mb, nBytes := SplitBytes(c.MultiBuffer, b)
    // 更新MultiBuffer为剩余的数据
    c.MultiBuffer = mb
    return nBytes, nil
}

// ReadMultiBuffer实现了Reader接口
func (c *MultiBufferContainer) ReadMultiBuffer() (MultiBuffer, error) {
    // 将MultiBuffer存入mb中
    mb := c.MultiBuffer
    // 清空MultiBuffer
    c.MultiBuffer = nil
    return mb, nil
}

// Write实现了io.Writer接口
func (c *MultiBufferContainer) Write(b []byte) (int, error) {
    // 调用MergeBytes方法将b中的数据合并到MultiBuffer中
    c.MultiBuffer = MergeBytes(c.MultiBuffer, b)
    return len(b), nil
}

// WriteMultiBuffer实现了Writer接口
func (c *MultiBufferContainer) WriteMultiBuffer(b MultiBuffer) error {
    // 调用MergeMulti方法将b中的数据合并到MultiBuffer中
    mb, _ := MergeMulti(c.MultiBuffer, b)
    c.MultiBuffer = mb
    return nil
}

// Close实现了io.Closer接口
func (c *MultiBufferContainer) Close() error {
    // 调用ReleaseMulti方法释放MultiBuffer中的数据
    c.MultiBuffer = ReleaseMulti(c.MultiBuffer)
    return nil
}
```