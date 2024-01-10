# `v2ray-core\proxy\vless\encoding\addons.go`

```
// +build !confonly
// 声明当前文件不仅仅是配置文件

package encoding
// 声明当前文件属于 encoding 包

import (
    "io"
    // 导入 io 包

    "github.com/golang/protobuf/proto"
    // 导入 protobuf 包

    "v2ray.com/core/common/buf"
    // 导入 buf 包
    "v2ray.com/core/common/protocol"
    // 导入 protocol 包
    "v2ray.com/core/proxy/vless"
    // 导入 vless 包
)

func EncodeHeaderAddons(buffer *buf.Buffer, addons *Addons) error {
    // 定义函数 EncodeHeaderAddons，接收 buf.Buffer 和 Addons 作为参数，返回 error

    switch addons.Flow {
    // 根据 addons 的 Flow 属性进行分支处理
    case vless.XRO, vless.XRD:
        // 如果 Flow 属性为 vless.XRO 或 vless.XRD

        bytes, err := proto.Marshal(addons)
        // 使用 protobuf 序列化 addons 对象

        if err != nil {
            return newError("failed to marshal addons protobuf value").Base(err)
            // 如果序列化失败，返回错误信息
        }
        if err := buffer.WriteByte(byte(len(bytes))); err != nil {
            return newError("failed to write addons protobuf length").Base(err)
            // 将序列化后的长度写入 buffer
        }
        if _, err := buffer.Write(bytes); err != nil {
            return newError("failed to write addons protobuf value").Base(err)
            // 将序列化后的值写入 buffer
        }

    default:
        // 如果 Flow 属性不是 vless.XRO 或 vless.XRD

        if err := buffer.WriteByte(0); err != nil {
            return newError("failed to write addons protobuf length").Base(err)
            // 将长度为 0 写入 buffer
        }

    }

    return nil
    // 返回空值
}

func DecodeHeaderAddons(buffer *buf.Buffer, reader io.Reader) (*Addons, error) {
    // 定义函数 DecodeHeaderAddons，接收 buf.Buffer 和 io.Reader 作为参数，返回 *Addons 和 error

    addons := new(Addons)
    // 创建一个新的 Addons 对象

    buffer.Clear()
    // 清空 buffer

    if _, err := buffer.ReadFullFrom(reader, 1); err != nil {
        return nil, newError("failed to read addons protobuf length").Base(err)
        // 从 reader 中读取 addons 的长度
    }

    if length := int32(buffer.Byte(0)); length != 0 {
        // 如果长度不为 0

        buffer.Clear()
        // 清空 buffer

        if _, err := buffer.ReadFullFrom(reader, length); err != nil {
            return nil, newError("failed to read addons protobuf value").Base(err)
            // 从 reader 中读取 addons 的值
        }

        if err := proto.Unmarshal(buffer.Bytes(), addons); err != nil {
            return nil, newError("failed to unmarshal addons protobuf value").Base(err)
            // 使用 protobuf 反序列化 addons 的值
        }

        // Verification.
        switch addons.Flow {
        default:
            // 对 addons 的 Flow 属性进行验证

        }

    }

    return addons, nil
    // 返回 addons 和空值
}

// EncodeBodyAddons returns a Writer that auto-encrypt content written by caller.
// EncodeBodyAddons 返回一个 Writer，用于自动加密调用者写入的内容
func EncodeBodyAddons(writer io.Writer, request *protocol.RequestHeader, addons *Addons) buf.Writer {
    // 定义函数 EncodeBodyAddons，接收 io.Writer、protocol.RequestHeader 和 Addons 作为参数，返回 buf.Writer

    switch addons.Flow {
    // 根据 addons 的 Flow 属性进行分支处理
    # 如果请求的命令是UDP，则返回一个新的多长度数据包写入器
    default:
        if request.Command == protocol.RequestCommandUDP {
            return NewMultiLengthPacketWriter(writer.(buf.Writer))
        }
    # 否则返回一个新的缓冲区写入器
    }
    return buf.NewWriter(writer)
}

// DecodeBodyAddons函数返回一个Reader，调用者可以从中获取解密后的body。
func DecodeBodyAddons(reader io.Reader, request *protocol.RequestHeader, addons *Addons) buf.Reader {

    switch addons.Flow {
    default:

        if request.Command == protocol.RequestCommandUDP {
            return NewLengthPacketReader(reader)
        }

    }

    return buf.NewReader(reader)

}

// NewMultiLengthPacketWriter函数返回一个MultiLengthPacketWriter指针，其Writer字段为传入的writer。
func NewMultiLengthPacketWriter(writer buf.Writer) *MultiLengthPacketWriter {
    return &MultiLengthPacketWriter{
        Writer: writer,
    }
}

// MultiLengthPacketWriter结构体包含一个buf.Writer字段。
type MultiLengthPacketWriter struct {
    buf.Writer
}

// WriteMultiBuffer方法将多个buf写入到Writer中。
func (w *MultiLengthPacketWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    defer buf.ReleaseMulti(mb)
    mb2Write := make(buf.MultiBuffer, 0, len(mb)+1)
    for _, b := range mb {
        length := b.Len()
        if length == 0 || length+2 > buf.Size {
            continue
        }
        eb := buf.New()
        if err := eb.WriteByte(byte(length >> 8)); err != nil {
            eb.Release()
            continue
        }
        if err := eb.WriteByte(byte(length)); err != nil {
            eb.Release()
            continue
        }
        if _, err := eb.Write(b.Bytes()); err != nil {
            eb.Release()
            continue
        }
        mb2Write = append(mb2Write, eb)
    }
    if mb2Write.IsEmpty() {
        return nil
    }
    return w.Writer.WriteMultiBuffer(mb2Write)
}

// NewLengthPacketWriter函数返回一个LengthPacketWriter指针，其Writer字段为传入的writer，cache字段为长度为0，容量为65536的切片。
func NewLengthPacketWriter(writer io.Writer) *LengthPacketWriter {
    return &LengthPacketWriter{
        Writer: writer,
        cache:  make([]byte, 0, 65536),
    }
}

// LengthPacketWriter结构体包含一个io.Writer字段和一个cache切片。
type LengthPacketWriter struct {
    io.Writer
    cache []byte
}

// WriteMultiBuffer方法将多个buf写入到Writer中。
func (w *LengthPacketWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    length := mb.Len() // none of mb is nil
    //fmt.Println("Write", length)
    if length == 0 {
        return nil
    }
    defer func() {
        w.cache = w.cache[:0]
    }()
    w.cache = append(w.cache, byte(length>>8), byte(length))
}
    # 遍历切片 mb，i 为索引，b 为值
    for i, b := range mb:
        # 将 b 的字节内容追加到 w.cache 中
        w.cache = append(w.cache, b.Bytes()...)
        # 释放 b 占用的内存
        b.Release()
        # 将切片 mb 中的元素置为 nil
        mb[i] = nil
    # 将 w.cache 写入到 w 中
    if _, err := w.Write(w.cache); err != nil:
        # 如果写入出错，返回一个新的错误
        return newError("failed to write a packet").Base(err)
    # 写入成功，返回空值
    return nil
// 创建一个新的LengthPacketReader对象，传入一个io.Reader作为参数，返回指向该对象的指针
func NewLengthPacketReader(reader io.Reader) *LengthPacketReader {
    return &LengthPacketReader{
        Reader: reader,
        cache:  make([]byte, 2),
    }
}

// 定义LengthPacketReader结构体
type LengthPacketReader struct {
    io.Reader
    cache []byte
}

// 读取多个缓冲区
func (r *LengthPacketReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 从Reader中读取两个字节到cache中，如果出错则返回错误
    if _, err := io.ReadFull(r.Reader, r.cache); err != nil { // maybe EOF
        return nil, newError("failed to read packet length").Base(err)
    }
    // 计算包长度
    length := int32(r.cache[0])<<8 | int32(r.cache[1])
    // 创建一个初始容量为length/buf.Size+1的空MultiBuffer
    mb := make(buf.MultiBuffer, 0, length/buf.Size+1)
    // 循环读取数据直到包长度为0
    for length > 0 {
        size := length
        if size > buf.Size {
            size = buf.Size
        }
        length -= size
        // 创建一个新的缓冲区b
        b := buf.New()
        // 从Reader中读取size个字节到缓冲区b中，如果出错则返回错误
        if _, err := b.ReadFullFrom(r.Reader, size); err != nil {
            return nil, newError("failed to read packet payload").Base(err)
        }
        // 将缓冲区b追加到MultiBuffer中
        mb = append(mb, b)
    }
    // 返回读取到的MultiBuffer和nil错误
    return mb, nil
}
```