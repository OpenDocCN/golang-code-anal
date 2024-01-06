# `grype\cmd\grype\cli\commands\util.go`

```
// 导入 commands 包
package commands

// 导入 fmt、os、strings、sync、github.com/hashicorp/go-multierror 和 golang.org/x/exp/maps 包
import (
	"fmt"
	"os"
	"strings"
	"sync"

	"github.com/hashicorp/go-multierror"
	"golang.org/x/exp/maps"
)

// stderrPrintLnf 函数用于向标准错误输出流打印格式化的消息，并在消息结尾添加换行符
func stderrPrintLnf(message string, args ...interface{}) error {
	// 如果消息不以换行符结尾，则在消息结尾添加换行符
	if !strings.HasSuffix(message, "\n") {
		message += "\n"
	}
	// 使用 fmt.Fprintf 将格式化的消息输出到标准错误输出流，并返回可能出现的错误
	_, err := fmt.Fprintf(os.Stderr, message, args...)
	return err
}
// parallel函数接受一组函数并并行运行它们，捕获所有返回的错误，并返回其中一个并行函数返回的单个错误，或者如果有多个错误，则返回一个包含所有错误的multierror.Error
func parallel(funcs ...func() error) error {
    // 用于存储并行函数返回的错误的切片
    errs := parallelMapped(funcs...)
    // 如果有错误返回
    if len(errs) > 0 {
        // 获取所有错误的值
        values := maps.Values(errs)
        // 如果只有一个错误
        if len(values) == 1 {
            return values[0]
        }
        // 如果有多个错误，将它们添加到multierror中并返回
        return multierror.Append(nil, values...)
    }
    // 如果没有错误返回，则返回nil
    return nil
}

// parallelMapped函数接受一组函数并并行运行它们，捕获所有返回的错误，并将每个函数的错误映射到一个map中
func parallelMapped(funcs ...func() error) map[int]error {
    // 用于存储每个函数返回的错误的map
    errs := map[int]error{}
    // 用于保护并发访问errs的互斥锁
    errorLock := &sync.Mutex{}
# 创建一个等待组，用于等待所有 goroutine 完成
wg := &sync.WaitGroup{}
# 将等待组的计数器设置为 funcs 的长度
wg.Add(len(funcs))
# 遍历 funcs 切片，为每个函数创建一个 goroutine
for i, fn := range funcs {
    go func(i int, fn func() error) {
        # 在函数执行完毕后，将等待组的计数器减一
        defer wg.Done()
        # 执行函数并获取返回的错误
        err := fn()
        # 如果有错误发生，将错误添加到 errs 切片中
        if err != nil {
            # 加锁以确保多个 goroutine 同时修改 errs 切片时不会出现竞争条件
            errorLock.Lock()
            defer errorLock.Unlock()
            errs[i] = err
        }
    }(i, fn)
}
# 等待所有 goroutine 完成
wg.Wait()
# 返回所有收集到的错误
return errs
}

# 将多个错误合并为一个错误
func appendErrors(errs error, err ...error) error {
    # 如果 errs 为空，直接返回 err 中的第一个错误
    if errs == nil {
        switch len(err) {
# 根据不同情况返回不同的值
case 0:
    # 如果情况为0，返回空值
    return nil
case 1:
    # 如果情况为1，返回err数组的第一个元素
    return err[0]
# 如果情况不是0或1，执行下面的语句
}
# 将err数组中的所有元素添加到errs数组中
return multierror.Append(errs, err...)
```