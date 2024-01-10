# `v2ray-core\common\mux\reader.go`

```
package mux

import (
    "io"

    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/crypto"
    "v2ray.com/core/common/serial"
)

// PacketReader is an io.Reader that reads whole chunk of Mux frames every time.
type PacketReader struct {
    reader io.Reader  // 用于读取数据的 io.Reader 接口
    eof    bool       // 标记是否已经到达文件末尾
}

// NewPacketReader creates a new PacketReader.
func NewPacketReader(reader io.Reader) *PacketReader {
    return &PacketReader{
        reader: reader,  // 初始化 PacketReader 结构体的 reader 字段
        eof:    false,   // 初始化 PacketReader 结构体的 eof 字段
    }
}

// ReadMultiBuffer implements buf.Reader.
func (r *PacketReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    if r.eof {  // 如果已经到达文件末尾
        return nil, io.EOF  // 返回空值和文件末尾错误
    }

    size, err := serial.ReadUint16(r.reader)  // 从 reader 中读取一个 uint16 类型的数据
    if err != nil {  // 如果读取过程中出现错误
        return nil, err  // 返回空值和读取错误
    }

    if size > buf.Size {  // 如果读取的数据大小超过了 buf.Size
        return nil, newError("packet size too large: ", size)  // 返回空值和自定义的错误信息
    }

    b := buf.New()  // 创建一个新的缓冲区
    if _, err := b.ReadFullFrom(r.reader, int32(size)); err != nil {  // 从 reader 中读取指定大小的数据到缓冲区
        b.Release()  // 释放缓冲区
        return nil, err  // 返回空值和读取错误
    }
    r.eof = true  // 标记已经到达文件末尾
    return buf.MultiBuffer{b}, nil  // 返回包含缓冲区的 MultiBuffer 和空值
}

// NewStreamReader creates a new StreamReader.
func NewStreamReader(reader *buf.BufferedReader) buf.Reader {
    return crypto.NewChunkStreamReaderWithChunkCount(crypto.PlainChunkSizeParser{}, reader, 1)  // 使用 crypto 包创建一个新的 StreamReader
}
```