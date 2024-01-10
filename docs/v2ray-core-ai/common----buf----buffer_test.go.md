# `v2ray-core\common\buf\buffer_test.go`

```
package buf_test

import (
    "bytes"  // 导入 bytes 包
    "crypto/rand"  // 导入 crypto/rand 包
    "testing"  // 导入 testing 包

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包
    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    . "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包，使用 . 表示可以直接使用 buf 包里的函数和变量
)

func TestBufferClear(t *testing.T) {
    buffer := New()  // 创建一个新的 Buffer 对象
    defer buffer.Release()  // 在函数返回时释放 Buffer 对象

    payload := "Bytes"  // 定义字符串 payload
    buffer.Write([]byte(payload))  // 将 payload 转换为字节数组写入 Buffer 对象
    if diff := cmp.Diff(buffer.Bytes(), []byte(payload)); diff != "" {  // 比较 Buffer 对象的内容和 payload 转换的字节数组是否相同
        t.Error(diff)  // 如果不相同则输出 diff
    }

    buffer.Clear()  // 清空 Buffer 对象
    if buffer.Len() != 0 {  // 如果 Buffer 对象的长度不为 0
        t.Error("expect 0 length, but got ", buffer.Len())  // 输出错误信息
    }
}

func TestBufferIsEmpty(t *testing.T) {
    buffer := New()  // 创建一个新的 Buffer 对象
    defer buffer.Release()  // 在函数返回时释放 Buffer 对象

    if buffer.IsEmpty() != true {  // 如果 Buffer 对象不为空
        t.Error("expect empty buffer, but not")  // 输出错误信息
    }
}

func TestBufferString(t *testing.T) {
    buffer := New()  // 创建一个新的 Buffer 对象
    defer buffer.Release()  // 在函数返回时释放 Buffer 对象

    const payload = "Test String"  // 定义常量 payload
    common.Must2(buffer.WriteString(payload))  // 将 payload 写入 Buffer 对象
    if buffer.String() != payload {  // 如果 Buffer 对象的内容不等于 payload
        t.Error("expect buffer content as ", payload, " but actually ", buffer.String())  // 输出错误信息
    }
}

func TestBufferByte(t *testing.T) {
    {
        buffer := New()  // 创建一个新的 Buffer 对象
        common.Must(buffer.WriteByte('m'))  // 将字节 'm' 写入 Buffer 对象
        if buffer.String() != "m" {  // 如果 Buffer 对象的内容不等于 "m"
            t.Error("expect buffer content as ", "m", " but actually ", buffer.String())  // 输出错误信息
        }
        buffer.Release()  // 释放 Buffer 对象
    }
    {
        buffer := StackNew()  // 创建一个新的 Stack Buffer 对象
        common.Must(buffer.WriteByte('n'))  // 将字节 'n' 写入 Stack Buffer 对象
        if buffer.String() != "n" {  // 如果 Stack Buffer 对象的内容不等于 "n"
            t.Error("expect buffer content as ", "n", " but actually ", buffer.String())  // 输出错误信息
        }
        buffer.Release()  // 释放 Stack Buffer 对象
    }
    {
        buffer := StackNew()  // 创建一个新的 Stack Buffer 对象
        common.Must2(buffer.WriteString("HELLOWORLD"))  // 将字符串 "HELLOWORLD" 写入 Stack Buffer 对象
        if b := buffer.Byte(5); b != 'W' {  // 获取 Stack Buffer 对象中索引为 5 的字节
            t.Error("unexpected byte ", b)  // 输出错误信息
        }

        buffer.SetByte(5, 'M')  // 将 Stack Buffer 对象中索引为 5 的字节设置为 'M'
        if buffer.String() != "HELLOMORLD" {  // 如果 Stack Buffer 对象的内容不等于 "HELLOMORLD"
            t.Error("expect buffer content as ", "n", " but actually ", buffer.String())  // 输出错误信息
        }
        buffer.Release()  // 释放 Stack Buffer 对象
    }
}
func TestBufferResize(t *testing.T) {
    buffer := New()  // 创建一个新的 Buffer 对象
    # 释放缓冲区资源
    defer buffer.Release()

    # 定义并初始化 payload 变量
    const payload = "Test String"
    # 将 payload 写入缓冲区
    common.Must2(buffer.WriteString(payload))
    # 检查缓冲区内容是否与 payload 相同，如果不同则输出错误信息
    if buffer.String() != payload {
        t.Error("expect buffer content as ", payload, " but actually ", buffer.String())
    }

    # 调整缓冲区大小，删除指定范围的内容
    buffer.Resize(-6, -3)
    # 检查缓冲区长度是否为 3，如果不是则输出错误信息
    if l := buffer.Len(); int(l) != 3 {
        t.Error("len error ", l)
    }

    # 获取缓冲区的字符串内容
    if s := buffer.String(); s != "Str" {
        t.Error("unexpect buffer ", s)
    }

    # 调整缓冲区大小，扩展或缩小缓冲区的容量
    buffer.Resize(int32(len(payload)), 200)
    # 检查缓冲区长度是否为 200 减去 payload 的长度，如果不是则输出错误信息
    if l := buffer.Len(); int(l) != 200-len(payload) {
        t.Error("len error ", l)
    }
func TestBufferSlice(t *testing.T) {
    {   // 创建新的缓冲区
        b := New()
        // 向缓冲区写入字节
        common.Must2(b.Write([]byte("abcd")))
        // 从缓冲区中获取指定范围的字节
        bytes := b.BytesFrom(-2)
        // 检查获取的字节与期望的字节是否相同
        if diff := cmp.Diff(bytes, []byte{'c', 'd'}); diff != "" {
            t.Error(diff)
        }
    }

    {
        b := New()
        common.Must2(b.Write([]byte("abcd")))
        bytes := b.BytesTo(-2)
        if diff := cmp.Diff(bytes, []byte{'a', 'b'}); diff != "" {
            t.Error(diff)
        }
    }

    {
        b := New()
        common.Must2(b.Write([]byte("abcd")))
        bytes := b.BytesRange(-3, -1)
        if diff := cmp.Diff(bytes, []byte{'b', 'c'}); diff != "" {
            t.Error(diff)
        }
    }
}

func TestBufferReadFullFrom(t *testing.T) {
    // 创建一个长度为1024的字节数组
    payload := make([]byte, 1024)
    // 从随机源中读取随机字节填充到payload中
    common.Must2(rand.Read(payload))

    // 创建一个从payload中读取数据的reader
    reader := bytes.NewReader(payload)
    // 创建一个新的缓冲区
    b := New()
    // 从reader中读取1024字节的数据到缓冲区中
    n, err := b.ReadFullFrom(reader, 1024)
    // 检查是否发生错误
    common.Must(err)
    // 检查实际读取的字节数是否为1024
    if n != 1024 {
        t.Error("expect reading 1024 bytes, but actually ", n)
    }

    // 检查缓冲区中的字节与期望的payload是否相同
    if diff := cmp.Diff(payload, b.Bytes()); diff != "" {
        t.Error(diff)
    }
}

func BenchmarkNewBuffer(b *testing.B) {
    for i := 0; i < b.N; i++ {
        // 创建一个新的缓冲区
        buffer := New()
        // 释放缓冲区
        buffer.Release()
    }
}

func BenchmarkNewBufferStack(b *testing.B) {
    for i := 0; i < b.N; i++ {
        // 创建一个新的缓冲区
        buffer := StackNew()
        // 释放缓冲区
        buffer.Release()
    }
}

func BenchmarkWrite2(b *testing.B) {
    buffer := New()

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        // 向缓冲区写入2个字节的数据
        _, _ = buffer.Write([]byte{'a', 'b'})
        // 清空缓冲区
        buffer.Clear()
    }
}

func BenchmarkWrite8(b *testing.B) {
    buffer := New()

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        // 向缓冲区写入8个字节的数据
        _, _ = buffer.Write([]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'})
        // 清空缓冲区
        buffer.Clear()
    }
}

func BenchmarkWrite32(b *testing.B) {
    buffer := New()
    // 创建一个长度为32的随机字节数组
    payload := make([]byte, 32)
    rand.Read(payload)

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        // 向缓冲区写入32个字节的数据
        _, _ = buffer.Write(payload)
        // 清空缓冲区
        buffer.Clear()
    }
}
# 定义一个性能测试函数，测试向缓冲区写入两个字节的性能
func BenchmarkWriteByte2(b *testing.B) {
    # 创建一个新的缓冲区
    buffer := New()

    # 重置计时器
    b.ResetTimer()
    # 循环执行 b.N 次，向缓冲区写入两个字节，并清空缓冲区
    for i := 0; i < b.N; i++ {
        _ = buffer.WriteByte('a')
        _ = buffer.WriteByte('b')
        buffer.Clear()
    }
}

# 定义一个性能测试函数，测试向缓冲区写入八个字节的性能
func BenchmarkWriteByte8(b *testing.B) {
    # 创建一个新的缓冲区
    buffer := New()

    # 重置计时器
    b.ResetTimer()
    # 循环执行 b.N 次，向缓冲区写入八个字节，并清空缓冲区
    for i := 0; i < b.N; i++ {
        _ = buffer.WriteByte('a')
        _ = buffer.WriteByte('b')
        _ = buffer.WriteByte('c')
        _ = buffer.WriteByte('d')
        _ = buffer.WriteByte('e')
        _ = buffer.WriteByte('f')
        _ = buffer.WriteByte('g')
        _ = buffer.WriteByte('h')
        buffer.Clear()
    }
}
```