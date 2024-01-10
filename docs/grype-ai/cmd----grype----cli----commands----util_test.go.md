# `grype\cmd\grype\cli\commands\util_test.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "sync"  // 导入 sync 包，用于同步操作
    "sync/atomic"  // 导入 sync/atomic 包，用于原子操作
    "testing"  // 导入 testing 包，用于测试

    "github.com/hashicorp/go-multierror"  // 导入第三方包，用于处理多个错误
    "github.com/stretchr/testify/require"  // 导入第三方包，用于测试断言
)

const lotsaParallel = 100  // 定义常量 lotsaParallel 为 100

func Test_lotsaLotsaParallel(t *testing.T) {
    funcs := []func() error{}  // 创建一个空的函数切片
    for i := 0; i < lotsaParallel; i++ {  // 循环 lotsaParallel 次
        funcs = append(funcs, func() error {  // 将一个匿名函数添加到 funcs 中
            Test_lotsaParallel(t)  // 调用 Test_lotsaParallel 函数
            return nil  // 返回空错误
        })
    }
    err := parallel(funcs...)  // 调用 parallel 函数并传入 funcs 切片中的函数
    require.NoError(t, err)  // 使用 require 包检查错误是否为空
}

func Test_lotsaParallel(t *testing.T) {
    for i := 0; i < lotsaParallel; i++ {  // 循环 lotsaParallel 次
        Test_parallel(t)  // 调用 Test_parallel 函数
    }
}

// Test_parallel tests the parallel function by executing a set of functions that can only execute in a specific
// order if they are actually running in parallel.
func Test_parallel(t *testing.T) {
    count := atomic.Int32{}  // 创建一个 int32 类型的原子变量 count
    count.Store(0)  // 将 count 设置为 0

    wg1 := sync.WaitGroup{}  // 创建一个 WaitGroup wg1
    wg1.Add(1)  // 向 wg1 中添加一个计数

    wg2 := sync.WaitGroup{}  // 创建一个 WaitGroup wg2
    wg2.Add(1)  // 向 wg2 中添加一个计数

    wg3 := sync.WaitGroup{}  // 创建一个 WaitGroup wg3
    wg3.Add(1)  // 向 wg3 中添加一个计数

    err1 := fmt.Errorf("error-1")  // 创建一个错误 err1
    err2 := fmt.Errorf("error-2")  // 创建一个错误 err2
    err3 := fmt.Errorf("error-3")  // 创建一个错误 err3

    order := ""  // 创建一个空字符串 order

    got := parallel(  // 调用 parallel 函数并传入多个匿名函数
        func() error {
            wg1.Wait()  // 等待 wg1 完成
            count.Add(1)  // 原子操作，count 加 1
            order = order + "_0"  // 将 "_0" 添加到 order 中
            return nil  // 返回空错误
        },
        func() error {
            wg3.Wait()  // 等待 wg3 完成
            defer wg2.Done()  // 在函数返回时调用 wg2.Done
            count.Add(10)  // 原子操作，count 加 10
            order = order + "_1"  // 将 "_1" 添加到 order 中
            return err1  // 返回 err1 错误
        },
        func() error {
            wg2.Wait()  // 等待 wg2 完成
            defer wg1.Done()  // 在函数返回时调用 wg1.Done
            count.Add(100)  // 原子操作，count 加 100
            order = order + "_2"  // 将 "_2" 添加到 order 中
            return err2  // 返回 err2 错误
        },
        func() error {
            defer wg3.Done()  // 在函数返回时调用 wg3.Done
            count.Add(1000)  // 原子操作，count 加 1000
            order = order + "_3"  // 将 "_3" 添加到 order 中
            return err3  // 返回 err3 错误
        },
    )
    require.Equal(t, int32(1111), count.Load())  // 使用 require 包检查 count 的值是否为 1111
    require.Equal(t, "_3_1_2_0", order)  // 使用 require 包检查 order 的值是否为 "_3_1_2_0"

    errs := got.(*multierror.Error).Errors  // 获取并保存 got 的错误信息
}
    // 不能使用 err1, err2, err3 的切片来检查是否包含在 errs 中，因为上面的函数是并行运行的，例如：
    // 在 func()#4 返回并且 `wg3.Done()` 执行完之后，线程可能会立即暂停
    // 然后剩下的函数先执行，导致 err3 不是列表中的第一个，而是最后一个
    require.Contains(t, errs, err1)
    require.Contains(t, errs, err2)
    require.Contains(t, errs, err3)
func Test_parallelMapped(t *testing.T) {
    // 创建三个错误对象
    err0 := fmt.Errorf("error-0")
    err1 := fmt.Errorf("error-1")
    err2 := fmt.Errorf("error-2")

    // 定义测试用例
    tests := []struct {
        name     string
        funcs    []func() error
        expected map[int]error
    }{
        // 第一个测试用例
        {
            name: "basic",
            funcs: []func() error{
                // 定义四个匿名函数，分别返回不同的错误
                func() error {
                    return nil
                },
                func() error {
                    return err1
                },
                func() error {
                    return nil
                },
                func() error {
                    return err2
                },
            },
            // 期望的结果是一个包含错误索引和对应错误的映射
            expected: map[int]error{
                1: err1,
                3: err2,
            },
        },
        // 第二个测试用例
        {
            name: "no errors",
            funcs: []func() error{
                // 定义两个匿名函数，都返回nil
                func() error {
                    return nil
                },
                func() error {
                    return nil
                },
            },
            // 期望的结果是一个空的映射
            expected: map[int]error{},
        },
        // 第三个测试用例
        {
            name: "all errors",
            funcs: []func() error{
                // 定义三个匿名函数，分别返回不同的错误
                func() error {
                    return err0
                },
                func() error {
                    return err1
                },
                func() error {
                    return err2
                },
            },
            // 期望的结果是一个包含错误索引和对应错误的映射
            expected: map[int]error{
                0: err0,
                1: err1,
                2: err2,
            },
        },
    }

    // 遍历测试用例
    for _, test := range tests {
        // 运行子测试
        t.Run(test.name, func(t *testing.T) {
            // 调用 parallelMapped 函数，获取结果
            got := parallelMapped(test.funcs...)
            // 使用断言检查结果是否符合预期
            require.Equal(t, test.expected, got)
        })
    }
}
```