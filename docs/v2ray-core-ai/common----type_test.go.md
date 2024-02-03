# `v2ray-core\common\type_test.go`

```go
package common_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    . "v2ray.com/core/common"  // 导入 v2ray 核心通用包
)

type TConfig struct {
    value int  // 定义 TConfig 结构体，包含整型 value
}

type YConfig struct {
    value string  // 定义 YConfig 结构体，包含字符串 value
}

func TestObjectCreation(t *testing.T) {
    var f = func(ctx context.Context, t interface{}) (interface{}, error) {  // 定义函数 f，接受上下文和接口类型参数，返回接口类型和错误
        return func() int {  // 返回一个匿名函数，返回整型
            return t.(*TConfig).value  // 返回 TConfig 结构体的 value 值
        }, nil
    }

    Must(RegisterConfig((*TConfig)(nil), f))  // 注册 TConfig 结构体和函数 f
    err := RegisterConfig((*TConfig)(nil), f)  // 注册 TConfig 结构体和函数 f，返回错误
    if err == nil {  // 如果错误为空
        t.Error("expect non-nil error, but got nil")  // 输出错误信息
    }

    g, err := CreateObject(context.Background(), &TConfig{value: 2})  // 创建 TConfig 结构体对象，返回对象和错误
    Must(err)  // 如果有错误，触发 panic
    if v := g.(func() int)(); v != 2 {  // 如果返回值不等于 2
        t.Error("expect return value 2, but got ", v)  // 输出错误信息
    }

    _, err = CreateObject(context.Background(), &YConfig{value: "T"})  // 创建 YConfig 结构体对象，返回对象和错误
    if err == nil {  // 如果错误为空
        t.Error("expect non-nil error, but got nil")  // 输出错误信息
    }
}
```