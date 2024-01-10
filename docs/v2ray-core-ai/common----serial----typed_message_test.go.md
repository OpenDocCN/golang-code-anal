# `v2ray-core\common\serial\typed_message_test.go`

```
// 定义包名为 serial_test
package serial_test

// 导入 testing 包
import (
    "testing"

    // 导入 v2ray.com/core/common/serial 包，并使用 . 符号代替该包的引用
    . "v2ray.com/core/common/serial"
)

// 定义测试函数 TestGetInstance，参数为 t *testing.T
func TestGetInstance(t *testing.T) {
    // 调用 GetInstance 函数，返回 p 和 err 两个值
    p, err := GetInstance("")
    // 如果 p 不为 nil，则输出错误信息
    if p != nil {
        t.Error("expected nil instance, but got ", p)
    }
    // 如果 err 为 nil，则输出错误信息
    if err == nil {
        t.Error("expect non-nil error, but got nil")
    }
}

// 定义测试函数 TestConvertingNilMessage，参数为 t *testing.T
func TestConvertingNilMessage(t *testing.T) {
    // 调用 ToTypedMessage 函数，传入参数为 nil，返回值赋给 x
    x := ToTypedMessage(nil)
    // 如果 x 不为 nil，则输出错误信息
    if x != nil {
        t.Error("expect nil, but actually not")
    }
}
```