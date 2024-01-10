# `kubo\cmd\ipfs\kubo\debug.go`

```
# 导入 kubo 包
package kubo

# 导入 net/http 包
import (
    "net/http"

    # 导入 profile 包
    "github.com/ipfs/kubo/profile"
)

# 初始化函数
func init() {
    # 注册处理函数，当请求路径为 /debug/stack 时执行匿名函数
    http.HandleFunc("/debug/stack",
        # 匿名函数，接收 http.ResponseWriter 和 *http.Request 两个参数
        func(w http.ResponseWriter, _ *http.Request) {
            # 调用 profile 包的 WriteAllGoroutineStacks 方法，将所有 goroutine 的堆栈信息写入 http.ResponseWriter
            _ = profile.WriteAllGoroutineStacks(w)
        },
    )
}
```