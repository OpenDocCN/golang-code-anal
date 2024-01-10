# `v2ray-core\transport\pipe\reader.go`

```
// 导入必要的包
package pipe

import (
    "time"

    "v2ray.com/core/common/buf"
)

// Reader 是一个从管道中读取内容的 buf.Reader
type Reader struct {
    pipe *pipe
}

// ReadMultiBuffer 实现了 buf.Reader 接口
func (r *Reader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    return r.pipe.ReadMultiBuffer()
}

// ReadMultiBufferTimeout 在给定的时间内从管道中读取内容，否则返回 buf.ErrTimeout
func (r *Reader) ReadMultiBufferTimeout(d time.Duration) (buf.MultiBuffer, error) {
    return r.pipe.ReadMultiBufferTimeout(d)
}

// Interrupt 实现了 common.Interruptible 接口
func (r *Reader) Interrupt() {
    r.pipe.Interrupt()
}
```