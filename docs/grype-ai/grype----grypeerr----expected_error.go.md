# `grype\grype\grypeerr\expected_error.go`

```
// 导入 grypeerr 包和 fmt 包
package grypeerr

import (
	"fmt"
)

// ExpectedErr 表示 grype 可能产生的一类预期错误
type ExpectedErr struct {
	Err error  // Err 字段用于存储错误信息
}

// New 生成一个新的 ExpectedErr 对象
func NewExpectedErr(msgFormat string, args ...interface{}) ExpectedErr {
	// 使用 fmt.Errorf 格式化错误信息，并存储在 Err 字段中
	return ExpectedErr{
		Err: fmt.Errorf(msgFormat, args...),
	}
}

// Error 返回表示底层错误条件的字符串
func (e ExpectedErr) Error() string {
	// 实现 error 接口的 Error 方法，返回错误信息的字符串表示形式
# 返回错误对象的错误信息。
```