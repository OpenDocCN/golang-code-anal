# `v2ray-core\common\mux\mux_test.go`

```
package mux_test

import (
    "io"
    "testing"

    "github.com/google/go-cmp/cmp"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    . "v2ray.com/core/common/mux" // 导入 v2ray.com/core/common/mux 包，并将其命名为 mux
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/transport/pipe"
)

// 从 buf.Reader 中读取所有数据，返回读取到的数据和可能的错误
func readAll(reader buf.Reader) (buf.MultiBuffer, error) {
    var mb buf.MultiBuffer
    for {
        b, err := reader.ReadMultiBuffer() // 从 buf.Reader 中读取多个缓冲区
        if err == io.EOF { // 如果读取到文件末尾
            break
        }
        if err != nil { // 如果发生错误
            return nil, err
        }
        mb = append(mb, b...) // 将读取到的缓冲区追加到 mb 中
    }
    return mb, nil // 返回读取到的数据和 nil 错误
}

// 测试 Reader 和 Writer 的功能
func TestReaderWriter(t *testing.T) {
    pReader, pWriter := pipe.New(pipe.WithSizeLimit(1024)) // 创建一个新的管道，限制大小为 1024

    dest := net.TCPDestination(net.DomainAddress("v2ray.com"), 80) // 创建一个 TCP 目标地址
    writer := NewWriter(1, dest, pWriter, protocol.TransferTypeStream) // 创建一个新的 Writer 对象

    dest2 := net.TCPDestination(net.LocalHostIP, 443) // 创建一个本地 TCP 目标地址
    writer2 := NewWriter(2, dest2, pWriter, protocol.TransferTypeStream) // 创建一个新的 Writer 对象

    dest3 := net.TCPDestination(net.LocalHostIPv6, 18374) // 创建一个本地 IPv6 TCP 目标地址
    writer3 := NewWriter(3, dest3, pWriter, protocol.TransferTypeStream) // 创建一个新的 Writer 对象

    // 写入数据到 writer 对象
    writePayload := func(writer *Writer, payload ...byte) error {
        b := buf.New() // 创建一个新的缓冲区
        b.Write(payload) // 将数据写入缓冲区
        return writer.WriteMultiBuffer(buf.MultiBuffer{b}) // 将缓冲区写入 writer 对象
    }

    common.Must(writePayload(writer, 'a', 'b', 'c', 'd')) // 写入数据到 writer 对象
    common.Must(writePayload(writer2)) // 写入数据到 writer2 对象

    common.Must(writePayload(writer, 'e', 'f', 'g', 'h')) // 写入数据到 writer 对象
    common.Must(writePayload(writer3, 'x')) // 写入数据到 writer3 对象

    writer.Close() // 关闭 writer 对象
    writer3.Close() // 关闭 writer3 对象

    common.Must(writePayload(writer2, 'y')) // 写入数据到 writer2 对象
    writer2.Close() // 关闭 writer2 对象

    bytesReader := &buf.BufferedReader{Reader: pReader} // 创建一个新的 buf.BufferedReader 对象，其 Reader 为 pReader
}
    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并确保操作成功
    common.Must(meta.Unmarshal(bytesReader))
    # 比较meta和给定的FrameMetadata结构体，如果不同则输出差异
    if r := cmp.Diff(meta, FrameMetadata{
        SessionID:     1,
        SessionStatus: SessionStatusNew,
        Target:        dest,
        Option:        OptionData,
    }); r != "" {
        t.Error("metadata: ", r)
    }

    # 读取bytesReader中的所有数据，并将结果赋值给data和err
    data, err := readAll(NewStreamReader(bytesReader))
    # 确保读取操作成功
    common.Must(err)
    # 将data转换为字符串，并比较其值是否为"abcd"，如果不同则输出差异
    if s := data.String(); s != "abcd" {
        t.Error("data: ", s)
    }
    # 重复上述代码块的操作，但使用不同的FrameMetadata和数据值
    # 重复上述代码块的操作，但使用不同的FrameMetadata和数据值
    # 重复上述代码块的操作，但使用不同的FrameMetadata和数据值
    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并确保没有错误发生
    common.Must(meta.Unmarshal(bytesReader))
    # 比较meta和给定的FrameMetadata结构体，如果有差异则输出错误信息
    if r := cmp.Diff(meta, FrameMetadata{
        SessionID:     1,
        SessionStatus: SessionStatusEnd,
        Option:        0,
    }); r != "" {
        t.Error("meta: ", r)
    }

    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并确保没有错误发生
    common.Must(meta.Unmarshal(bytesReader))
    # 比较meta和给定的FrameMetadata结构体，如果有差异则输出错误信息
    if r := cmp.Diff(meta, FrameMetadata{
        SessionID:     3,
        SessionStatus: SessionStatusEnd,
        Option:        0,
    }); r != "" {
        t.Error("meta: ", r)
    }

    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并确保没有错误发生
    common.Must(meta.Unmarshal(bytesReader))
    # 比较meta和给定的FrameMetadata结构体，如果有差异则输出错误信息
    if r := cmp.Diff(meta, FrameMetadata{
        SessionID:     2,
        SessionStatus: SessionStatusKeep,
        Option:        1,
    }); r != "" {
        t.Error("meta: ", r)
    }
    # 读取bytesReader中的所有数据并存储在data中
    data, err := readAll(NewStreamReader(bytesReader))
    # 确保没有错误发生
    common.Must(err)
    # 比较data的字符串表示和"y"，如果不相等则输出错误信息
    if s := data.String(); s != "y" {
        t.Error("data: ", s)
    }

    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并确保没有错误发生
    common.Must(meta.Unmarshal(bytesReader))
    # 比较meta和给定的FrameMetadata结构体，如果有差异则输出错误信息
    if r := cmp.Diff(meta, FrameMetadata{
        SessionID:     2,
        SessionStatus: SessionStatusEnd,
        Option:        0,
    }); r != "" {
        t.Error("meta: ", r)
    }

    # 关闭pWriter
    pWriter.Close()

    # 创建一个名为meta的FrameMetadata变量
    var meta FrameMetadata
    # 使用bytesReader解析meta，并检查是否有错误发生
    err := meta.Unmarshal(bytesReader)
    # 如果没有错误发生，则输出错误信息
    if err == nil {
        t.Error("nil error")
    }
# 闭合前面的函数定义
```