# `grype\cmd\grype\cli\commands\util.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于访问操作系统功能
    "strings"  // 导入 strings 包，用于处理字符串
    "sync"  // 导入 sync 包，用于实现并发控制

    "github.com/hashicorp/go-multierror"  // 导入第三方包，用于处理多个错误
    "golang.org/x/exp/maps"  // 导入第三方包，用于处理 map 数据结构
)

func stderrPrintLnf(message string, args ...interface{}) error {
    // 如果消息不以换行符结尾，则添加换行符
    if !strings.HasSuffix(message, "\n") {
        message += "\n"
    }
    // 将格式化后的消息输出到标准错误流，并返回可能的错误
    _, err := fmt.Fprintf(os.Stderr, message, args...)
    return err
}

// parallel 函数接受一组函数，并并行运行它们，捕获所有返回的错误，并返回其中一个并行函数返回的单个错误，或者如果有多个错误，则返回一个包含所有错误的 multierror.Error
func parallel(funcs ...func() error) error {
    // 并行运行函数并捕获所有错误
    errs := parallelMapped(funcs...)
    // 如果有错误，则返回其中一个错误，如果有多个错误，则返回一个包含所有错误的 multierror.Error
    if len(errs) > 0 {
        values := maps.Values(errs)
        if len(values) == 1 {
            return values[0]
        }
        return multierror.Append(nil, values...)
    }
    return nil
}

// parallelMapped 函数接受一组函数，并并行运行它们，捕获所有返回的错误，并在一个 map 中指示每个函数的索引返回了哪个错误
func parallelMapped(funcs ...func() error) map[int]error {
    errs := map[int]error{}  // 初始化一个空的错误 map
    errorLock := &sync.Mutex{}  // 创建一个互斥锁
    wg := &sync.WaitGroup{}  // 创建一个 WaitGroup
    wg.Add(len(funcs))  // 设置 WaitGroup 的计数器
    for i, fn := range funcs {
        // 并行运行函数
        go func(i int, fn func() error) {
            defer wg.Done()  // 函数执行完毕后减少 WaitGroup 的计数器
            err := fn()  // 执行函数并获取返回的错误
            if err != nil {
                errorLock.Lock()  // 加锁
                defer errorLock.Unlock()  // 解锁
                errs[i] = err  // 将错误存入 map
            }
        }(i, fn)
    }
    wg.Wait()  // 等待所有函数执行完毕
    return errs  // 返回捕获的所有错误
}

func appendErrors(errs error, err ...error) error {
    if errs == nil {
        switch len(err) {
        case 0:
            return nil
        case 1:
            return err[0]
        }
    }
    return multierror.Append(errs, err...)  // 将新的错误追加到已有的错误中
}
```