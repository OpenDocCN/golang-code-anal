# `v2ray-core\transport\internet\headers\http\linkedreadRequest.go`

```go
// 导入 http 包
package http

// 导入 bufio 和 net/http 包
import (
    "bufio"
    "net/http"

    _ "unsafe" // required to use //go:linkname
)

// 使用 //go:linkname 需要导入 "unsafe" 包
// 定义 readRequest 函数，使用 //go:linkname 进行链接
func readRequest(b *bufio.Reader, deleteHostHeader bool) (req *http.Request, err error)
```