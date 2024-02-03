# `v2ray-core\common\errors\multi_error.go`

```go
package errors

import (
    "strings"  // 导入 strings 包
)

type multiError []error  // 定义一个类型为 error 切片的 multiError 类型

func (e multiError) Error() string {  // 定义 multiError 类型的方法 Error，返回字符串类型
    var r strings.Builder  // 创建一个 strings.Builder 类型的变量 r
    r.WriteString("multierr: ")  // 将字符串 "multierr: " 写入 r
    for _, err := range e {  // 遍历 multiError 切片
        r.WriteString(err.Error())  // 将每个 error 的 Error 方法返回的字符串写入 r
        r.WriteString(" | ")  // 在每个 error 后面添加 " | "
    }
    return r.String()  // 返回 r 中的字符串
}

func Combine(maybeError ...error) error {  // 定义一个函数 Combine，接收多个 error 类型参数，返回一个 error 类型
    var errs multiError  // 创建一个 multiError 类型的变量 errs
    for _, err := range maybeError {  // 遍历参数中的 error 切片
        if err != nil {  // 如果 error 不为 nil
            errs = append(errs, err)  // 将 error 添加到 errs 中
        }
    }
    if len(errs) == 0 {  // 如果 errs 中没有 error
        return nil  // 返回 nil
    }
    return errs  // 返回 errs
}
```