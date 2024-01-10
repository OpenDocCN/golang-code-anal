# `v2ray-core\common\buf\copy.go`

```
package buf

import (
    "io"
    "time"

    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/signal"
)

type dataHandler func(MultiBuffer)

type copyHandler struct {
    onData []dataHandler
}

// SizeCounter is for counting bytes copied by Copy().
type SizeCounter struct {
    Size int64
}

// CopyOption is an option for copying data.
type CopyOption func(*copyHandler)

// UpdateActivity is a CopyOption to update activity on each data copy operation.
func UpdateActivity(timer signal.ActivityUpdater) CopyOption {
    return func(handler *copyHandler) {
        handler.onData = append(handler.onData, func(MultiBuffer) {
            timer.Update()
        })
    }
}

// CountSize is a CopyOption that sums the total size of data copied into the given SizeCounter.
func CountSize(sc *SizeCounter) CopyOption {
    return func(handler *copyHandler) {
        handler.onData = append(handler.onData, func(b MultiBuffer) {
            sc.Size += int64(b.Len())
        })
    }
}

type readError struct {
    error
}

func (e readError) Error() string {
    return e.error.Error()
}

func (e readError) Inner() error {
    return e.error
}

// IsReadError returns true if the error in Copy() comes from reading.
func IsReadError(err error) bool {
    _, ok := err.(readError)
    return ok
}

type writeError struct {
    error
}

func (e writeError) Error() string {
    return e.error.Error()
}

func (e writeError) Inner() error {
    return e.error
}

// IsWriteError returns true if the error in Copy() comes from writing.
func IsWriteError(err error) bool {
    _, ok := err.(writeError)
    return ok
}

func copyInternal(reader Reader, writer Writer, handler *copyHandler) error {
    // 无限循环，读取数据直到出现错误或者读取到空数据
    for {
        // 读取多个缓冲区的数据
        buffer, err := reader.ReadMultiBuffer()
        // 如果缓冲区不为空
        if !buffer.IsEmpty() {
            // 遍历数据处理函数列表，对每个处理函数处理数据
            for _, handler := range handler.onData {
                handler(buffer)
            }
            // 将数据写入到写入器中
            if werr := writer.WriteMultiBuffer(buffer); werr != nil {
                // 如果写入出现错误，返回写入错误
                return writeError{werr}
            }
        }
        // 如果读取数据出现错误，返回读取错误
        if err != nil {
            return readError{err}
        }
    }
// Copy函数将从reader复制所有有效载荷到writer，或者在发生错误时停止。当到达文件末尾时返回nil。
func Copy(reader Reader, writer Writer, options ...CopyOption) error {
    var handler copyHandler
    for _, option := range options {
        option(&handler)
    }
    err := copyInternal(reader, writer, &handler)
    if err != nil && errors.Cause(err) != io.EOF {
        return err
    }
    return nil
}

// ErrNotTimeoutReader是一个新的错误，表示不是TimeoutReader类型
var ErrNotTimeoutReader = newError("not a TimeoutReader")

// CopyOnceTimeout函数将从reader复制一次数据到writer，如果reader不是TimeoutReader类型则返回ErrNotTimeoutReader
func CopyOnceTimeout(reader Reader, writer Writer, timeout time.Duration) error {
    timeoutReader, ok := reader.(TimeoutReader)
    if !ok {
        return ErrNotTimeoutReader
    }
    mb, err := timeoutReader.ReadMultiBufferTimeout(timeout)
    if err != nil {
        return err
    }
    return writer.WriteMultiBuffer(mb)
}
```