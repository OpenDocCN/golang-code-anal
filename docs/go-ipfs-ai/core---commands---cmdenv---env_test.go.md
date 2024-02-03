# `kubo\core\commands\cmdenv\env_test.go`

```go
package cmdenv

import (
    "strconv"  // 导入 strconv 包，用于字符串转换
    "testing"  // 导入 testing 包，用于测试
)

func TestEscNonPrint(t *testing.T) {
    b := []byte("hello")  // 创建一个字节数组，存储字符串"hello"的字节表示
    b[2] = 0x7f  // 修改字节数组中索引为2的字节为十六进制0x7f
    s := string(b)  // 将字节数组转换为字符串
    if !needEscape(s) {  // 如果字符串不需要转义
        t.Fatal("string needs escaping")  // 输出错误信息
    }
    if !hasNonPrintable(s) {  // 如果字符串中包含不可打印字符
        t.Fatal("expected non-printable")  // 输出错误信息
    }
    if hasNonPrintable(EscNonPrint(s)) {  // 如果经过转义后的字符串中包含不可打印字符
        t.Fatal("escaped string has non-printable")  // 输出错误信息
    }
    if EscNonPrint(`hel\lo`) != `hel\\lo` {  // 如果转义后的字符串不符合预期
        t.Fatal("backslash not escaped")  // 输出错误信息
    }

    s = `hello`  // 将字符串变量 s 赋值为"hello"
    if needEscape(s) {  // 如果字符串需要转义
        t.Fatal("string does not need escaping")  // 输出错误信息
    }
    if EscNonPrint(s) != s {  // 如果转义后的字符串与原字符串不相等
        t.Fatal("string should not have changed")  // 输出错误信息
    }
    s = `"hello"`  // 将字符串变量 s 赋值为`"hello"`
    if EscNonPrint(s) != s {  // 如果转义后的字符串与原字符串不相等
        t.Fatal("string should not have changed")  // 输出错误信息
    }
    if EscNonPrint(`"hel\"lo"`) != `"hel\\"lo"` {  // 如果转义后的字符串不符合预期
        t.Fatal("did not get expected escaped string")  // 输出错误信息
    }
}

func hasNonPrintable(s string) bool {
    for _, r := range s {  // 遍历字符串 s 中的每个字符
        if !strconv.IsPrint(r) {  // 如果字符不可打印
            return true  // 返回 true
        }
    }
    return false  // 返回 false
}
```