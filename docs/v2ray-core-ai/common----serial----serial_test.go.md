# `v2ray-core\common\serial\serial_test.go`

```
package serial_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 google/go-cmp/cmp 包，用于比较数据

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包，用于缓冲区操作
    "v2ray.com/core/common/serial"  // 导入 v2ray.com/core/common/serial 包，用于序列化操作
)

func TestUint16Serial(t *testing.T) {
    b := buf.New()  // 创建新的缓冲区
    defer b.Release()  // 在函数返回前释放缓冲区资源

    n, err := serial.WriteUint16(b, 10)  // 将 uint16 类型的数据写入缓冲区
    common.Must(err)  // 检查错误
    if n != 2 {  // 如果写入的字节数不等于2
        t.Error("expect 2 bytes writtng, but actually ", n)  // 输出错误信息
    }
    if diff := cmp.Diff(b.Bytes(), []byte{0, 10}); diff != "" {  // 比较写入的字节流和期望的字节流
        t.Error(diff)  // 输出错误信息
    }
}

func TestUint64Serial(t *testing.T) {
    b := buf.New()  // 创建新的缓冲区
    defer b.Release()  // 在函数返回前释放缓冲区资源

    n, err := serial.WriteUint64(b, 10)  // 将 uint64 类型的数据写入缓冲区
    common.Must(err)  // 检查错误
    if n != 8 {  // 如果写入的字节数不等于8
        t.Error("expect 8 bytes writtng, but actually ", n)  // 输出错误信息
    }
    if diff := cmp.Diff(b.Bytes(), []byte{0, 0, 0, 0, 0, 0, 0, 10}); diff != "" {  // 比较写入的字节流和期望的字节流
        t.Error(diff)  // 输出错误信息
    }
}

func TestReadUint16(t *testing.T) {
    testCases := []struct {  // 定义测试用例结构体
        Input  []byte  // 输入字节流
        Output uint16  // 期望输出的 uint16 类型数据
    }{
        {
            Input:  []byte{0, 1},  // 输入字节流
            Output: 1,  // 期望输出的 uint16 类型数据
        },
    }

    for _, testCase := range testCases {  // 遍历测试用例
        v, err := serial.ReadUint16(bytes.NewReader(testCase.Input))  // 从输入字节流中读取 uint16 类型数据
        common.Must(err)  // 检查错误
        if v != testCase.Output {  // 如果读取的数据不等于期望的输出
            t.Error("for input ", testCase.Input, " expect output ", testCase.Output, " but got ", v)  // 输出错误信息
        }
    }
}

func BenchmarkReadUint16(b *testing.B) {
    reader := buf.New()  // 创建新的缓冲区
    defer reader.Release()  // 在函数返回前释放缓冲区资源

    common.Must2(reader.Write([]byte{0, 1}))  // 写入字节流
    b.ResetTimer()  // 重置计时器

    for i := 0; i < b.N; i++ {  // 循环执行 b.N 次
        _, err := serial.ReadUint16(reader)  // 从缓冲区中读取 uint16 类型数据
        common.Must(err)  // 检查错误
        reader.Clear()  // 清空缓冲区
        reader.Extend(2)  // 扩展缓冲区大小为2
    }
}

func BenchmarkWriteUint64(b *testing.B) {
    writer := buf.New()  // 创建新的缓冲区
    defer writer.Release()  // 在函数返回前释放缓冲区资源

    b.ResetTimer()  // 重置计时器

    for i := 0; i < b.N; i++ {  // 循环执行 b.N 次
        _, err := serial.WriteUint64(writer, 8)  // 将 uint64 类型的数据写入缓冲区
        common.Must(err)  // 检查错误
        writer.Clear()  // 清空缓冲区
    }
}
```