# `grype\grype\grypeerr\expected_error.go`

```go
// 定义 grypeerr 包
package grypeerr

// 导入 fmt 包
import (
    "fmt"
)

// ExpectedErr 表示 grype 可能产生的一类预期错误
type ExpectedErr struct {
    Err error  // 错误对象
}

// New 生成一个新的 ExpectedErr 对象
func NewExpectedErr(msgFormat string, args ...interface{}) ExpectedErr {
    return ExpectedErr{
        Err: fmt.Errorf(msgFormat, args...),  // 使用 fmt.Errorf 格式化错误信息
    }
}

// Error 返回表示底层错误条件的字符串
func (e ExpectedErr) Error() string {
    return e.Err.Error()  // 返回错误对象的字符串表示
}
```