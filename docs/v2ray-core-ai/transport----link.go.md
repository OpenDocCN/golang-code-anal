# `v2ray-core\transport\link.go`

```go
// 导入名为 "buf" 的包，该包位于 "v2ray.com/core/common" 下
import "v2ray.com/core/common/buf"

// Link 是一个连接工具，用于连接入站和出站代理处理程序之间的通信
type Link struct {
    // Reader 是一个用于读取数据的缓冲区
    Reader buf.Reader
    // Writer 是一个用于写入数据的缓冲区
    Writer buf.Writer
}
```