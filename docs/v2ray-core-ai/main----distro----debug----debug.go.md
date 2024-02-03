# `v2ray-core\main\distro\debug\debug.go`

```go
# 导入 net/http/pprof 包，用于性能分析和调试
import _ "net/http/pprof"
# 导入 net/http 包，用于 HTTP 服务器和客户端功能
import "net/http"

# 在包初始化时启动一个 goroutine，用于监听端口 6060，并提供 nil 处理程序
# 用于性能分析和调试
func init() {
    go func() {
        http.ListenAndServe(":6060", nil)
    }()
}
```