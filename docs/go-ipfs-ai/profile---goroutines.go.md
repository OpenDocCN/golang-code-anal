# `kubo\profile\goroutines.go`

```go
// 定义一个名为 profile 的包
package profile

// 导入所需的包
import (
    "io"
    "runtime"
)

// WriteAllGoroutineStacks 函数用于将所有 goroutine 的堆栈跟踪写入给定的写入器
// 与 Go 提供的方法不同之处在于它不会在 64 MB 后截断
func WriteAllGoroutineStacks(w io.Writer) error {
    // 基于 pprof.writeGoroutineStacks 方法，移除了 64 MB 的限制
    // 创建一个 1 MB 大小的字节切片
    buf := make([]byte, 1<<20)
    // 循环获取每个 goroutine 的堆栈跟踪
    for i := 0; ; i++ {
        // 获取当前 goroutine 的堆栈跟踪，并将结果存储到 buf 中
        n := runtime.Stack(buf, true)
        // 如果获取的堆栈跟踪长度小于 buf 的长度，则截取 buf 到实际长度
        if n < len(buf) {
            buf = buf[:n]
            break
        }
        // 如果获取的堆栈跟踪长度大于等于 buf 的长度，则扩大 buf 的长度为原来的两倍
        buf = make([]byte, 2*len(buf))
    }
    // 将堆栈跟踪写入给定的写入器
    _, err := w.Write(buf)
    return err
}
```