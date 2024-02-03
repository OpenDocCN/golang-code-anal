# `trojan-go\common\error.go`

```go
// common 包，包含了一些通用的函数和类型
package common

// 引入 fmt 包，用于格式化输入输出
import (
    "fmt"
)

// Error 结构体，包含错误信息
type Error struct {
    info string
}

// Error 方法，返回错误信息
func (e *Error) Error() string {
    return e.info
}

// Base 方法，将当前错误信息与传入的错误信息连接起来
func (e *Error) Base(err error) *Error {
    if err != nil {
        e.info += " | " + err.Error()
    }
    return e
}

// NewError 函数，创建一个新的 Error 对象
func NewError(info string) *Error {
    return &Error{
        info: info,
    }
}

// Must 函数，如果错误不为空，则打印错误信息并抛出异常
func Must(err error) {
    if err != nil {
        fmt.Println(err)
        panic(err)
    }
}

// Must2 函数，如果错误不为空，则打印错误信息并抛出异常
func Must2(_ interface{}, err error) {
    if err != nil {
        fmt.Println(err)
        panic(err)
    }
}
```