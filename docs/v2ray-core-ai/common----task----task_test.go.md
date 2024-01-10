# `v2ray-core\common\task\task_test.go`

```
package task_test

import (
    "context"  // 导入 context 包，用于处理上下文
    "errors"   // 导入 errors 包，用于处理错误
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"     // 导入 time 包，用于处理时间

    "github.com/google/go-cmp/cmp"  // 导入第三方包，用于比较数据

    "v2ray.com/core/common"  // 导入自定义包
    . "v2ray.com/core/common/task"  // 导入自定义包，并将其所有公共函数和变量导入当前命名空间
)

func TestExecuteParallel(t *testing.T) {
    // 在后台并行执行两个函数，返回错误
    err := Run(context.Background(),
        func() error {
            time.Sleep(time.Millisecond * 200)  // 休眠 200 毫秒
            return errors.New("test")  // 返回一个错误
        }, func() error {
            time.Sleep(time.Millisecond * 500)  // 休眠 500 毫秒
            return errors.New("test2")  // 返回一个错误
        })

    // 比较错误信息，如果不相等则输出错误信息
    if r := cmp.Diff(err.Error(), "test"); r != "" {
        t.Error(r)
    }
}

func TestExecuteParallelContextCancel(t *testing.T) {
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在上下文中并行执行三个函数，返回错误
    err := Run(ctx, func() error {
        time.Sleep(time.Millisecond * 2000)  // 休眠 2000 毫秒
        return errors.New("test")  // 返回一个错误
    }, func() error {
        time.Sleep(time.Millisecond * 5000)  // 休眠 5000 毫秒
        return errors.New("test2")  // 返回一个错误
    }, func() error {
        cancel()  // 取消上下文
        return nil  // 返回空值
    })

    // 获取错误信息
    errStr := err.Error()
    // 如果错误信息中不包含 "canceled"，则输出错误信息
    if !strings.Contains(errStr, "canceled") {
        t.Error("expected error string to contain 'canceled', but actually not: ", errStr)
    }
}

func BenchmarkExecuteOne(b *testing.B) {
    // 定义一个空函数
    noop := func() error {
        return nil
    }
    // 循环执行 b.N 次，每次在后台执行一个空函数
    for i := 0; i < b.N; i++ {
        common.Must(Run(context.Background(), noop))
    }
}

func BenchmarkExecuteTwo(b *testing.B) {
    // 定义一个空函数
    noop := func() error {
        return nil
    }
    // 循环执行 b.N 次，每次在后台并行执行两个空函数
    for i := 0; i < b.N; i++ {
        common.Must(Run(context.Background(), noop, noop))
    }
}
```