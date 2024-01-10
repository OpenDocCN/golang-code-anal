# `v2ray-core\common\buf\writer_test.go`

```
package buf_test

import (
    "bufio"  // 导入 bufio 包，提供了缓冲读写功能
    "bytes"  // 导入 bytes 包，提供了操作字节切片的函数
    "crypto/rand"  // 导入 crypto/rand 包，提供了加密强随机数生成器
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "testing"  // 导入 testing 包，提供了编写测试的支持

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包，提供了比较函数

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    . "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包，并将其导出的所有函数和变量都导入当前包
    "v2ray.com/core/transport/pipe"  // 导入 v2ray.com/core/transport/pipe 包
)

func TestWriter(t *testing.T) {
    lb := New()  // 创建一个新的 Buffer 对象
    common.Must2(lb.ReadFrom(rand.Reader))  // 从随机数生成器中读取数据到 lb 中

    expectedBytes := append([]byte(nil), lb.Bytes()...)  // 复制 lb 中的字节切片到 expectedBytes 中

    writeBuffer := bytes.NewBuffer(make([]byte, 0, 1024*1024))  // 创建一个新的字节缓冲区

    writer := NewBufferedWriter(NewWriter(writeBuffer))  // 创建一个新的缓冲写入器，并将其包装在一个新的写入器中
    writer.SetBuffered(false)  // 设置缓冲写入器的缓冲状态为 false
    common.Must(writer.WriteMultiBuffer(MultiBuffer{lb}))  // 将 lb 中的数据写入到缓冲写入器中
    common.Must(writer.Flush())  // 刷新缓冲写入器

    if r := cmp.Diff(expectedBytes, writeBuffer.Bytes()); r != "" {  // 比较 expectedBytes 和 writeBuffer.Bytes() 的差异
        t.Error(r)  // 输出差异信息
    }
}

func TestBytesWriterReadFrom(t *testing.T) {
    const size = 50000  // 定义常量 size 为 50000
    pReader, pWriter := pipe.New(pipe.WithSizeLimit(size))  // 创建一个新的管道，设置大小限制为 size
    reader := bufio.NewReader(io.LimitReader(rand.Reader, size))  // 创建一个新的带缓冲的读取器，限制从随机数生成器中读取的数据大小为 size
    writer := NewBufferedWriter(pWriter)  // 创建一个新的缓冲写入器，并将其包装在一个新的写入器中
    writer.SetBuffered(false)  // 设置缓冲写入器的缓冲状态为 false
    nBytes, err := reader.WriteTo(writer)  // 从 reader 中读取数据并写入到 writer 中
    if nBytes != size {  // 如果实际写入的字节数不等于 size
        t.Fatal("unexpected size of bytes written: ", nBytes)  // 输出错误信息并终止测试
    }
    if err != nil {  // 如果发生错误
        t.Fatal("expect success, but actually error: ", err.Error())  // 输出错误信息并终止测试
    }

    mb, err := pReader.ReadMultiBuffer()  // 从管道中读取多个缓冲区
    common.Must(err)  // 如果发生错误，终止程序
    if mb.Len() != size {  // 如果读取的缓冲区大小不等于 size
        t.Fatal("unexpected size read: ", mb.Len())  // 输出错误信息并终止测试
    }
}

func TestDiscardBytes(t *testing.T) {
    b := New()  // 创建一个新的 Buffer 对象
    common.Must2(b.ReadFullFrom(rand.Reader, Size))  // 从随机数生成器中读取 Size 大小的数据到 b 中

    nBytes, err := io.Copy(DiscardBytes, b)  // 将 b 中的数据拷贝到 DiscardBytes 中
    common.Must(err)  // 如果发生错误，终止程序
    if nBytes != Size {  // 如果拷贝的字节数不等于 Size
        t.Error("copy size: ", nBytes)  // 输出错误信息
    }
}

func TestDiscardBytesMultiBuffer(t *testing.T) {
    const size = 10240*1024 + 1  // 定义常量 size 为 10240*1024 + 1
    buffer := bytes.NewBuffer(make([]byte, 0, size))  // 创建一个新的字节缓冲区
    common.Must2(buffer.ReadFrom(io.LimitReader(rand.Reader, size)))  // 从随机数生成器中读取数据到缓冲区中

    r := NewReader(buffer)  // 创建一个新的缓冲读取器
    nBytes, err := io.Copy(DiscardBytes, &BufferedReader{Reader: r})  // 将缓冲读取器中的数据拷贝到 DiscardBytes 中
    common.Must(err)  // 如果发生错误，终止程序
    if nBytes != size {  // 如果拷贝的字节数不等于 size
        t.Error("copy size: ", nBytes)  // 输出错误信息
    }
}
func TestWriterInterface(t *testing.T) {
    {
        // 声明一个接口变量，其类型为 *BufferToBytesWriter
        var writer interface{} = (*BufferToBytesWriter)(nil)
        // 判断接口变量的类型
        switch writer.(type) {
        // 如果类型是 Writer、io.Writer 或 io.ReaderFrom，则执行下面的代码
        case Writer, io.Writer, io.ReaderFrom:
        // 如果类型不是上述类型，则执行下面的代码
        default:
            t.Error("BufferToBytesWriter is not Writer, io.Writer or io.ReaderFrom")
        }
    }

    {
        // 声明一个接口变量，其类型为 *BufferedWriter
        var writer interface{} = (*BufferedWriter)(nil)
        // 判断接口变量的类型
        switch writer.(type) {
        // 如果类型是 Writer、io.Writer、io.ReaderFrom 或 io.ByteWriter，则执行下面的代码
        case Writer, io.Writer, io.ReaderFrom, io.ByteWriter:
        // 如果类型不是上述类型，则执行下面的代码
        default:
            t.Error("BufferedWriter is not Writer, io.Writer, io.ReaderFrom or io.ByteWriter")
        }
    }
}
```