# `v2ray-core\transport\pipe\writer.go`

```go
// 引入名为 "pipe" 的包和 "v2ray.com/core/common/buf" 包
package pipe

// 定义一个结构体 "Writer"，它是一个 buf.Writer，用于将数据写入管道
type Writer struct {
    pipe *pipe
}

// 实现 buf.Writer 接口的 WriteMultiBuffer 方法
func (w *Writer) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 调用管道的 WriteMultiBuffer 方法将数据写入管道
    return w.pipe.WriteMultiBuffer(mb)
}

// 实现 io.Closer 接口的 Close 方法。关闭管道后，写入管道将返回 io.ErrClosedPipe，而读取将返回 io.EOF。
func (w *Writer) Close() error {
    // 调用管道的 Close 方法关闭管道
    return w.pipe.Close()
}

// 实现 common.Interruptible 接口的 Interrupt 方法
func (w *Writer) Interrupt() {
    // 调用管道的 Interrupt 方法中断操作
    w.pipe.Interrupt()
}
```