# `v2ray-core\common\common_test.go`

```
package common_test

import (
    "errors"  // 导入 errors 包，用于创建错误
    "testing"  // 导入 testing 包，用于编写测试函数

    . "v2ray.com/core/common"  // 导入 common 包，并使用 . 符号来省略包名
)

func TestMust(t *testing.T) {
    // 定义一个函数，用于检测是否有 panic 发生
    hasPanic := func(f func()) (ret bool) {
        // 使用 defer 来捕获 panic，并将结果保存到 ret 变量中
        defer func() {
            if r := recover(); r != nil {
                ret = true
            }
        }()
        f()  // 调用传入的函数
        return false
    }

    // 定义测试用例的结构体
    testCases := []struct {
        Input func()  // 定义输入的函数类型
        Panic bool  // 定义是否发生 panic 的标志
    }{
        {
            Panic: true,  // 第一个测试用例，期望发生 panic
            Input: func() { Must(func() error { return errors.New("test error") }()) },  // 调用 Must 函数，传入一个返回错误的函数
        },
        {
            Panic: true,  // 第二个测试用例，期望发生 panic
            Input: func() { Must2(func() (int, error) { return 0, errors.New("test error") }()) },  // 调用 Must2 函数，传入一个返回错误的函数
        },
        {
            Panic: false,  // 第三个测试用例，期望不发生 panic
            Input: func() { Must(func() error { return nil }()) },  // 调用 Must 函数，传入一个返回 nil 的函数
        },
    }

    // 遍历测试用例
    for idx, test := range testCases {
        // 检查是否发生了预期的 panic
        if hasPanic(test.Input) != test.Panic {
            t.Error("test case #", idx, " expect panic ", test.Panic, " but actually not")  // 输出测试结果
        }
    }
}
```