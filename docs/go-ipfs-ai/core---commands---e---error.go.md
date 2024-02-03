# `kubo\core\commands\e\error.go`

```go
// 导入必要的包
package e

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "runtime/debug"  // 导入 runtime/debug 包，用于获取堆栈信息
)

// TypeErr 函数返回一个带有解释预期错误和实际错误的字符串的错误
func TypeErr(expected, actual interface{}) error {
    return fmt.Errorf("expected type %T, got %T", expected, actual)  // 格式化输出预期类型和实际类型的错误信息
}

// 编译时类型检查，确保 HandlerError 是一个错误类型
var _ error = New(nil)

// HandlerError 结构体为错误添加堆栈跟踪信息
type HandlerError struct {
    Err   error   // 错误信息
    Stack []byte  // 堆栈信息
}

// Error 方法使 HandlerError 实现 error 接口
func (err HandlerError) Error() string {
    return fmt.Sprintf("%s in:\n%s", err.Err.Error(), err.Stack)  // 格式化输出错误信息和堆栈信息
}

// New 函数返回一个新的 HandlerError
func New(err error) HandlerError {
    return HandlerError{Err: err, Stack: debug.Stack()}  // 返回一个包含错误信息和堆栈信息的 HandlerError 结构体
}
```