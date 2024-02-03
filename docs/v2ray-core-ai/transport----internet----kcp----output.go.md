# `v2ray-core\transport\internet\kcp\output.go`

```go
// +build !confonly

package kcp

import (
    "io" // 导入io包，用于实现输入输出功能
    "sync" // 导入sync包，用于实现同步功能

    "v2ray.com/core/common/retry" // 导入retry包，用于实现重试功能

    "v2ray.com/core/common/buf" // 导入buf包，用于实现缓冲功能
)

// SegmentWriter 定义了一个接口，用于写入Segment
type SegmentWriter interface {
    Write(seg Segment) error
}

// SimpleSegmentWriter 实现了SegmentWriter接口
type SimpleSegmentWriter struct {
    sync.Mutex // 互斥锁，用于保护共享资源
    buffer *buf.Buffer // 缓冲区，用于存储数据
    writer io.Writer // io.Writer接口，用于写入数据
}

// NewSegmentWriter 创建一个新的SegmentWriter
func NewSegmentWriter(writer io.Writer) SegmentWriter {
    return &SimpleSegmentWriter{
        writer: writer,
        buffer: buf.New(),
    }
}

// Write 实现了SegmentWriter接口的Write方法
func (w *SimpleSegmentWriter) Write(seg Segment) error {
    w.Lock() // 加锁
    defer w.Unlock() // 延迟解锁

    w.buffer.Clear() // 清空缓冲区
    rawBytes := w.buffer.Extend(seg.ByteSize()) // 扩展缓冲区，以便存储数据
    seg.Serialize(rawBytes) // 将Segment序列化到缓冲区
    _, err := w.writer.Write(w.buffer.Bytes()) // 将缓冲区中的数据写入到io.Writer中
    return err // 返回错误信息
}

// RetryableWriter 实现了SegmentWriter接口
type RetryableWriter struct {
    writer SegmentWriter // SegmentWriter接口，用于写入Segment
}

// NewRetryableWriter 创建一个新的RetryableWriter
func NewRetryableWriter(writer SegmentWriter) SegmentWriter {
    return &RetryableWriter{
        writer: writer,
    }
}

// Write 实现了SegmentWriter接口的Write方法
func (w *RetryableWriter) Write(seg Segment) error {
    return retry.Timed(5, 100).On(func() error { // 使用retry包中的Timed方法进行重试
        return w.writer.Write(seg) // 调用writer的Write方法写入Segment
    })
}
```