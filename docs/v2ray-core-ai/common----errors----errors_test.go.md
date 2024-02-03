# `v2ray-core\common\errors\errors_test.go`

```go
package errors_test

import (
    "io"
    "strings"
    "testing"

    "github.com/google/go-cmp/cmp"

    . "v2ray.com/core/common/errors"
    "v2ray.com/core/common/log"
)

func TestError(t *testing.T) {
    // 创建一个新的错误对象"TestError"
    err := New("TestError")
    // 检查错误的严重程度是否为信息级别
    if v := GetSeverity(err); v != log.Severity_Info {
        t.Error("severity: ", v)
    }

    // 创建一个新的错误对象"TestError2"，并将其基础错误设置为EOF
    err = New("TestError2").Base(io.EOF)
    // 检查错误的严重程度是否为信息级别
    if v := GetSeverity(err); v != log.Severity_Info {
        t.Error("severity: ", v)
    }

    // 创建一个新的错误对象"TestError3"，并将其基础错误设置为EOF，并将其设置为警告级别
    err = New("TestError3").Base(io.EOF).AtWarning()
    // 检查错误的严重程度是否为警告级别
    if v := GetSeverity(err); v != log.Severity_Warning {
        t.Error("severity: ", v)
    }

    // 创建一个新的错误对象"TestError4"，并将其基础错误设置为EOF，并将其设置为警告级别
    err = New("TestError4").Base(io.EOF).AtWarning()
    // 创建一个新的错误对象"TestError5"，并将其基础错误设置为前一个错误对象
    err = New("TestError5").Base(err)
    // 检查错误的严重程度是否为警告级别
    if v := GetSeverity(err); v != log.Severity_Warning {
        t.Error("severity: ", v)
    }
    // 检查错误消息中是否包含"EOF"
    if v := err.Error(); !strings.Contains(v, "EOF") {
        t.Error("error: ", v)
    }
}

type e struct{}

func TestErrorMessage(t *testing.T) {
    // 创建一个包含错误对象和期望错误消息的数据结构数组
    data := []struct {
        err error
        msg string
    }{
        {
            // 创建一个错误对象"a"，并将其基础错误设置为错误对象"b"，并设置路径对象为e{}
            err: New("a").Base(New("b")).WithPathObj(e{}),
            msg: "v2ray.com/core/common/errors_test: a > b",
        },
        {
            // 创建一个错误对象"a"，并将其基础错误设置为错误对象"b"，并设置路径对象为e{}
            err: New("a").Base(New("b").WithPathObj(e{})),
            msg: "a > v2ray.com/core/common/errors_test: b",
        },
    }

    // 遍历数据结构数组
    for _, d := range data {
        // 检查错误消息是否符合预期
        if diff := cmp.Diff(d.msg, d.err.Error()); diff != "" {
            t.Error(diff)
        }
    }
}
```