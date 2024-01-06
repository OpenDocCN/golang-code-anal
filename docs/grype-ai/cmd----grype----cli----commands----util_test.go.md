# `grype\cmd\grype\cli\commands\util_test.go`

```
package commands

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"sync"  // 导入 sync 包，用于实现并发控制
	"sync/atomic"  // 导入 sync/atomic 包，用于原子操作
	"testing"  // 导入 testing 包，用于编写测试函数

	"github.com/hashicorp/go-multierror"  // 导入第三方包，用于处理多个错误
	"github.com/stretchr/testify/require"  // 导入第三方包，用于编写测试断言
)

const lotsaParallel = 100  // 定义常量 lotsaParallel，值为 100

func Test_lotsaLotsaParallel(t *testing.T) {  // 定义测试函数 Test_lotsaLotsaParallel
	funcs := []func() error{}  // 创建一个空的函数切片
	for i := 0; i < lotsaParallel; i++ {  // 循环 lotsaParallel 次
		funcs = append(funcs, func() error {  // 将一个匿名函数添加到 funcs 切片中
			Test_lotsaParallel(t)  // 调用 Test_lotsaParallel 函数
			return nil  // 返回空错误
		})
	}
	err := parallel(funcs...)
	require.NoError(t, err)
}

func Test_lotsaParallel(t *testing.T) {
	for i := 0; i < lotsaParallel; i++ {
		Test_parallel(t)
	}
}

// Test_lotsaParallel函数用于测试并行执行多个函数，循环执行lotsaParallel次Test_parallel函数

// Test_parallel tests the parallel function by executing a set of functions that can only execute in a specific
// order if they are actually running in parallel.
// Test_parallel函数通过执行一组函数来测试并行函数，只有在实际并行运行时，这些函数才能按特定顺序执行。

func Test_parallel(t *testing.T) {
	count := atomic.Int32{}
	count.Store(0)

	wg1 := sync.WaitGroup{}
	wg1.Add(1)
```

// 创建一个等待组 wg2，用于等待一个任务完成
wg2 := sync.WaitGroup{}
wg2.Add(1)

// 创建另一个等待组 wg3，用于等待另一个任务完成
wg3 := sync.WaitGroup{}
wg3.Add(1)

// 创建三个错误对象，分别表示不同的错误
err1 := fmt.Errorf("error-1")
err2 := fmt.Errorf("error-2")
err3 := fmt.Errorf("error-3")

// 初始化一个空字符串，用于记录任务执行的顺序
order := ""

// 调用 parallel 函数并传入一个匿名函数作为参数
got := parallel(
    func() error {
        // 等待 wg1 完成
        wg1.Wait()
        // 增加计数
        count.Add(1)
        // 更新执行顺序
        order = order + "_0"
        return nil
    },
		func() error {  // 定义一个匿名函数，返回一个错误
			wg3.Wait()  // 等待 wg3 完成
			defer wg2.Done()  // 在函数结束时调用 wg2.Done()，表示 wg2 完成
			count.Add(10)  // 将 10 加到 count 上
			order = order + "_1"  // 将 "_1" 添加到 order 上
			return err1  // 返回 err1
		},
		func() error {  // 定义一个匿名函数，返回一个错误
			wg2.Wait()  // 等待 wg2 完成
			defer wg1.Done()  // 在函数结束时调用 wg1.Done()，表示 wg1 完成
			count.Add(100)  // 将 100 加到 count 上
			order = order + "_2"  // 将 "_2" 添加到 order 上
			return err2  // 返回 err2
		},
		func() error {  // 定义一个匿名函数，返回一个错误
			defer wg3.Done()  // 在函数结束时调用 wg3.Done()，表示 wg3 完成
			count.Add(1000)  // 将 1000 加到 count 上
			order = order + "_3"  // 将 "_3" 添加到 order 上
			return err3  // 返回 err3
		},
	)
	// 检查 count.Load() 的返回值是否等于 int32(1111)
	require.Equal(t, int32(1111), count.Load())
	// 检查 order 的值是否等于 "_3_1_2_0"
	require.Equal(t, "_3_1_2_0", order)

	// 将 got 转换为 *multierror.Error 类型，并获取其中的 Errors 字段
	errs := got.(*multierror.Error).Errors

	// 无法直接将 errs 与包含 err1, err2, err3 的切片进行相等性检查，因为上面的函数是并行运行的
	// 例如：在 func()#4 返回并执行了 `wg3.Done()` 后，线程可能会立即暂停
	// 然后剩余的函数先执行，导致 err3 不是列表中的第一个而是最后一个
	require.Contains(t, errs, err1)
	require.Contains(t, errs, err2)
	require.Contains(t, errs, err3)
}

func Test_parallelMapped(t *testing.T) {
	err0 := fmt.Errorf("error-0")
	err1 := fmt.Errorf("error-1")
	err2 := fmt.Errorf("error-2")

	tests := []struct {
	// 这里是测试用例的定义，包括一系列的测试数据和期望的结果
		// 定义结构体字段，包括名称、函数列表和预期结果的映射
		name     string
		funcs    []func() error
		expected map[int]error
	}{
		// 第一个测试用例
		{
			// 测试用例名称
			name: "basic",
			// 函数列表，每个函数返回一个错误
			funcs: []func() error{
				// 第一个函数返回 nil
				func() error {
					return nil
				},
				// 第二个函数返回 err1
				func() error {
					return err1
				},
				// 第三个函数返回 nil
				func() error {
					return nil
				},
				// 第四个函数返回 err2
				func() error {
					return err2
				},
			},
# 定义一个期望的结果，是一个映射，键为整数，值为错误
expected: map[int]error{
    1: err1,  # 键为1，值为err1
    3: err2,  # 键为3，值为err2
},
# 下一个测试用例
{
    name: "no errors",  # 测试用例的名称
    funcs: []func() error{  # 函数列表，每个函数返回一个错误
        func() error {  # 第一个函数
            return nil  # 返回空错误
        },
        func() error {  # 第二个函数
            return nil  # 返回空错误
        },
    },
    expected: map[int]error{},  # 期望的结果是一个空的映射
},
{
    name: "all errors",  # 测试用例的名称
    funcs: []func() error{  # 函数列表，每个函数返回一个错误
# 创建一个匿名函数，返回err0
func() error {
    return err0
},
# 创建一个匿名函数，返回err1
func() error {
    return err1
},
# 创建一个匿名函数，返回err2
func() error {
    return err2
},
# 创建一个包含匿名函数的切片
},
# 创建一个包含测试用例的切片
expected: map[int]error{
    0: err0,
    1: err1,
    2: err2,
},
# 遍历测试用例切片
for _, test := range tests {
    # 使用测试用例的名称创建一个子测试
    t.Run(test.name, func(t *testing.T) {
		// 使用并行映射执行给定的函数列表
		got := parallelMapped(test.funcs...)
		// 断言实际结果与期望结果相等
		require.Equal(t, test.expected, got)
		// 结束当前测试
	})
}
```