# `kubo\thirdparty\assert\assert.go`

```go
package assert

import "testing"

// 如果错误不为 nil，则使用传入的消息和错误信息终止测试
func Nil(err error, t *testing.T, msgs ...string) {
    if err != nil {
        t.Fatal(msgs, "error:", err)
    }
}

// 如果条件不为真，则使用传入的消息终止测试
func True(v bool, t *testing.T, msgs ...string) {
    if !v {
        t.Fatal(msgs)
    }
}

// 如果条件不为假，则使用传入的消息终止测试
func False(v bool, t *testing.T, msgs ...string) {
    True(!v, t, msgs...)
}

// 如果错误为 nil，则使用传入的消息和错误信息终止测试
func Err(err error, t *testing.T, msgs ...string) {
    if err == nil {
        t.Fatal(msgs, "error:", err)
    }
}
```