# `v2ray-core\app\dispatcher\stats_test.go`

```go
package dispatcher_test

import (
    "testing"

    . "v2ray.com/core/app/dispatcher"  // 导入 v2ray.com/core/app/dispatcher 包，使用 . 表示可以直接使用其中的函数和变量
    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包
)

type TestCounter int64  // 定义 TestCounter 类型为 int64

func (c *TestCounter) Value() int64 {  // 定义 TestCounter 类型的方法 Value，返回 int64 类型
    return int64(*c)  // 返回 TestCounter 类型的值转换为 int64 类型
}

func (c *TestCounter) Add(v int64) int64 {  // 定义 TestCounter 类型的方法 Add，参数为 int64 类型，返回 int64 类型
    x := int64(*c) + v  // 将 TestCounter 类型的值转换为 int64 类型后与参数相加，赋值给 x
    *c = TestCounter(x)  // 将 x 转换为 TestCounter 类型后赋值给 c
    return x  // 返回 x
}

func (c *TestCounter) Set(v int64) int64 {  // 定义 TestCounter 类型的方法 Set，参数为 int64 类型，返回 int64 类型
    *c = TestCounter(v)  // 将 v 转换为 TestCounter 类型后赋值给 c
    return v  // 返回 v
}

func TestStatsWriter(t *testing.T) {  // 定义测试函数 TestStatsWriter，参数为 *testing.T 类型
    var c TestCounter  // 声明一个 TestCounter 类型的变量 c
    writer := &SizeStatWriter{  // 声明一个 SizeStatWriter 类型的变量 writer，使用 & 表示取其地址
        Counter: &c,  // 将 c 的地址赋值给 Counter
        Writer:  buf.Discard,  // 将 buf.Discard 赋值给 Writer
    }

    mb := buf.MergeBytes(nil, []byte("abcd"))  // 调用 buf.MergeBytes 函数，将 nil 和 []byte("abcd") 合并成 MultiBuffer 类型的变量 mb
    common.Must(writer.WriteMultiBuffer(mb))  // 调用 writer 的 WriteMultiBuffer 方法，参数为 mb

    mb = buf.MergeBytes(nil, []byte("efg"))  // 调用 buf.MergeBytes 函数，将 nil 和 []byte("efg") 合并成 MultiBuffer 类型的变量 mb
    common.Must(writer.WriteMultiBuffer(mb))  // 调用 writer 的 WriteMultiBuffer 方法，参数为 mb

    if c.Value() != 7 {  // 如果 c 的值不等于 7
        t.Fatal("unexpected counter value. want 7, but got ", c.Value())  // 输出错误信息
    }
}
```