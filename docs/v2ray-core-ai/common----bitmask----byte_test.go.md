# `v2ray-core\common\bitmask\byte_test.go`

```go
package bitmask_test

import (
    "testing"  // 导入测试包
    . "v2ray.com/core/common/bitmask"  // 导入位掩码包，并使用点操作符简化调用
)

func TestBitmaskByte(t *testing.T) {
    b := Byte(0)  // 创建一个字节类型的位掩码，并初始化为0
    b.Set(Byte(1))  // 设置位掩码中的第1位
    if !b.Has(1) {  // 如果位掩码中不包含第1位
        t.Fatal("expected ", b, " to contain 1, but actually not")  // 输出错误信息
    }

    b.Set(Byte(2))  // 设置位掩码中的第2位
    if !b.Has(2) {  // 如果位掩码中不包含第2位
        t.Fatal("expected ", b, " to contain 2, but actually not")  // 输出错误信息
    }
    if !b.Has(1) {  // 如果位掩码中不包含第1位
        t.Fatal("expected ", b, " to contain 1, but actually not")  // 输出错误信息
    }

    b.Clear(Byte(1))  // 清除位掩码中的第1位
    if !b.Has(2) {  // 如果位掩码中不包含第2位
        t.Fatal("expected ", b, " to contain 2, but actually not")  // 输出错误信息
    }
    if b.Has(1) {  // 如果位掩码中包含第1位
        t.Fatal("expected ", b, " to not contain 1, but actually did")  // 输出错误信息
    }

    b.Toggle(Byte(2))  // 切换位掩码中的第2位
    if b.Has(2) {  // 如果位掩码中包含第2位
        t.Fatal("expected ", b, " to not contain 2, but actually did")  // 输出错误信息
    }
}
```