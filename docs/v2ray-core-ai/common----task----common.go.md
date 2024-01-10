# `v2ray-core\common\task\common.go`

```
// 声明一个名为 task 的包
package task

// 导入 v2ray.com/core/common 包
import "v2ray.com/core/common"

// Close 函数接收一个接口类型的参数 v，并返回一个闭包函数，用于关闭参数 v
func Close(v interface{}) func() error {
    // 返回一个闭包函数，用于关闭参数 v
    return func() error {
        // 调用 common 包中的 Close 函数关闭参数 v，并返回可能出现的错误
        return common.Close(v)
    }
}
```