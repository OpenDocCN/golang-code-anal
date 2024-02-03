# `v2ray-core\common\crypto\chunk_test.go`

```go
package crypto_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "io"  // 导入 io 包，用于实现 I/O 操作
    "testing"  // 导入 testing 包，用于编写测试函数

    "v2ray.com/core/common"  // 导入 common 包
    "v2ray.com/core/common/buf"  // 导入 buf 包
    . "v2ray.com/core/common/crypto"  // 导入 crypto 包，并将其所有公开的符号导入当前包的命名空间
)

func TestChunkStreamIO(t *testing.T) {
    cache := bytes.NewBuffer(make([]byte, 0, 8192))  // 创建一个容量为 8192 的字节缓冲区

    writer := NewChunkStreamWriter(PlainChunkSizeParser{}, cache)  // 使用 PlainChunkSizeParser 创建一个新的 ChunkStreamWriter 对象
    reader := NewChunkStreamReader(PlainChunkSizeParser{}, cache)  // 使用 PlainChunkSizeParser 创建一个新的 ChunkStreamReader 对象

    b := buf.New()  // 创建一个新的缓冲区
    b.WriteString("abcd")  // 向缓冲区写入字符串 "abcd"
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))  // 使用 writer 将缓冲区内容写入到 MultiBuffer 中

    b = buf.New()  // 创建一个新的缓冲区
    b.WriteString("efg")  // 向缓冲区写入字符串 "efg"
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{b}))  // 使用 writer 将缓冲区内容写入到 MultiBuffer 中

    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))  // 使用 writer 写入一个空的 MultiBuffer

    if cache.Len() != 13 {  // 如果缓冲区的长度不等于 13
        t.Fatalf("Cache length is %d, want 13", cache.Len())  // 输出错误信息
    }

    mb, err := reader.ReadMultiBuffer()  // 从 reader 中读取 MultiBuffer
    common.Must(err)  // 如果出现错误，则触发 panic

    if s := mb.String(); s != "abcd" {  // 如果 MultiBuffer 转换成字符串后不等于 "abcd"
        t.Error("content: ", s)  // 输出错误信息
    }

    mb, err = reader.ReadMultiBuffer()  // 从 reader 中读取 MultiBuffer
    common.Must(err)  // 如果出现错误，则触发 panic

    if s := mb.String(); s != "efg" {  // 如果 MultiBuffer 转换成字符串后不等于 "efg"
        t.Error("content: ", s)  // 输出错误信息
    }

    _, err = reader.ReadMultiBuffer()  // 从 reader 中读取 MultiBuffer
    if err != io.EOF {  // 如果错误不是 io.EOF
        t.Error("error: ", err)  // 输出错误信息
    }
}
```