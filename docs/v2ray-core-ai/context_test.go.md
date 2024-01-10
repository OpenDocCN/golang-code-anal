# `v2ray-core\context_test.go`

```
# 定义包名为 core_test
package core_test

# 导入所需的包
import (
    "context"  # 导入 context 包
    "testing"  # 导入 testing 包

    . "v2ray.com/core"  # 导入 v2ray.com/core 包，并使用 . 符号代替包名
)

# 定义测试函数 TestContextPanic
func TestContextPanic(t *testing.T) {
    # 延迟执行的函数，用于捕获 panic
    defer func() {
        # 恢复 panic，并判断是否为 nil
        r := recover()
        if r == nil {
            t.Error("expect panic, but nil")  # 如果 r 为 nil，则输出错误信息
        }
    }()

    # 从 context 包中获取上下文，如果失败则触发 panic
    MustFromContext(context.Background())
}
```