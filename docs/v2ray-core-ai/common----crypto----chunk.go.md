# `v2ray-core\common\crypto\chunk.go`

```go
package crypto

import (
    "encoding/binary"  // 导入用于处理二进制数据的包
    "io"  // 导入用于输入输出操作的包

    "v2ray.com/core/common"  // 导入自定义包
    "v2ray.com/core/common/buf"  // 导入自定义包
)

// ChunkSizeDecoder is a utility class to decode size value from bytes.
type ChunkSizeDecoder interface {
    SizeBytes() int32  // 定义接口方法，用于返回大小字节数
    Decode([]byte) (uint16, error)  // 定义接口方法，用于解码字节数据
}

// ChunkSizeEncoder is a utility class to encode size value into bytes.
type ChunkSizeEncoder interface {
    SizeBytes() int32  // 定义接口方法，用于返回大小字节数
    Encode(uint16, []byte) []byte  // 定义接口方法，用于将大小值编码为字节数据
}

type PaddingLengthGenerator interface {
    MaxPaddingLen() uint16  // 定义接口方法，用于返回最大填充长度
    NextPaddingLen() uint16  // 定义接口方法，用于返回下一个填充长度
}

type PlainChunkSizeParser struct{}  // 定义结构体

func (PlainChunkSizeParser) SizeBytes() int32 {  // 实现接口方法，返回大小字节数
    return 2
}

func (PlainChunkSizeParser) Encode(size uint16, b []byte) []byte {  // 实现接口方法，将大小值编码为字节数据
    binary.BigEndian.PutUint16(b, size)  // 将大小值以大端序写入字节数据
    return b[:2]  // 返回编码后的字节数据
}

func (PlainChunkSizeParser) Decode(b []byte) (uint16, error) {  // 实现接口方法，解码字节数据
    return binary.BigEndian.Uint16(b), nil  // 从字节数据中读取大小值并返回
}

type AEADChunkSizeParser struct {
    Auth *AEADAuthenticator  // 定义结构体字段
}

func (p *AEADChunkSizeParser) SizeBytes() int32 {  // 实现接口方法，返回大小字节数
    return 2 + int32(p.Auth.Overhead())  // 返回大小字节数，包括认证开销
}

func (p *AEADChunkSizeParser) Encode(size uint16, b []byte) []byte {  // 实现接口方法，将大小值编码为字节数据
    binary.BigEndian.PutUint16(b, size-uint16(p.Auth.Overhead()))  // 将大小值减去认证开销后以大端序写入字节数据
    b, err := p.Auth.Seal(b[:0], b[:2])  // 使用认证器对字节数据进行加密
    common.Must(err)  // 检查错误并处理
    return b  // 返回加密后的字节数据
}

func (p *AEADChunkSizeParser) Decode(b []byte) (uint16, error) {  // 实现接口方法，解码字节数据
    b, err := p.Auth.Open(b[:0], b)  // 使用认证器对字节数据进行解密
    if err != nil {  // 如果有错误发生
        return 0, err  // 返回错误
    }
    return binary.BigEndian.Uint16(b) + uint16(p.Auth.Overhead()), nil  // 从解密后的字节数据中读取大小值并返回
}

type ChunkStreamReader struct {
    sizeDecoder   ChunkSizeDecoder  // 定义结构体字段
    reader        *buf.BufferedReader  // 定义结构体字段

    buffer        []byte  // 定义结构体字段
    leftOverSize  int32  // 定义结构体字段
    maxNumChunk   uint32  // 定义结构体字段
    numChunk      uint32  // 定义结构体字段
}

func NewChunkStreamReader(sizeDecoder ChunkSizeDecoder, reader io.Reader) *ChunkStreamReader {  // 定义函数，创建新的 ChunkStreamReader 实例
    return NewChunkStreamReaderWithChunkCount(sizeDecoder, reader, 0)  // 调用带有最大块数量参数的函数
}

func NewChunkStreamReaderWithChunkCount(sizeDecoder ChunkSizeDecoder, reader io.Reader, maxNumChunk uint32) *ChunkStreamReader {  // 定义函数，创建新的 ChunkStreamReader 实例
    // 创建一个 ChunkStreamReader 结构体的指针，并初始化 sizeDecoder 和 buffer 字段
    r := &ChunkStreamReader{
        sizeDecoder: sizeDecoder,
        buffer:      make([]byte, sizeDecoder.SizeBytes()),
        maxNumChunk: maxNumChunk,
    }
    // 检查传入的 reader 是否为 buf.BufferedReader 类型，如果是则将其赋值给 r.reader，否则创建一个新的 buf.BufferedReader 并赋值给 r.reader
    if breader, ok := reader.(*buf.BufferedReader); ok {
        r.reader = breader
    } else {
        r.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
    }
    // 返回初始化后的 ChunkStreamReader 结构体指针
    return r
# 读取块大小的方法，返回块大小和可能的错误
func (r *ChunkStreamReader) readSize() (uint16, error) {
    # 从读取器中读取固定大小的数据到缓冲区，如果出错则返回错误
    if _, err := io.ReadFull(r.reader, r.buffer); err != nil {
        return 0, err
    }
    # 使用大小解码器解码缓冲区中的数据，返回解码后的块大小
    return r.sizeDecoder.Decode(r.buffer)
}

# 读取多个缓冲区的方法，返回多个缓冲区和可能的错误
func (r *ChunkStreamReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    # 获取剩余数据的大小
    size := r.leftOverSize
    # 如果剩余数据大小为0
    if size == 0 {
        # 增加块计数
        r.numChunk++
        # 如果最大块数大于0且块计数大于最大块数，则返回空和文件结束错误
        if r.maxNumChunk > 0 && r.numChunk > r.maxNumChunk {
            return nil, io.EOF
        }
        # 读取下一个块的大小
        nextSize, err := r.readSize()
        if err != nil {
            return nil, err
        }
        # 如果下一个块的大小为0，则返回空和文件结束错误
        if nextSize == 0 {
            return nil, io.EOF
        }
        # 将下一个块的大小转换为int32类型
        size = int32(nextSize)
    }
    # 更新剩余数据的大小
    r.leftOverSize = size

    # 从读取器中读取最多指定大小的数据到多个缓冲区，如果缓冲区不为空则更新剩余数据的大小并返回多个缓冲区和空错误，否则返回空和可能的错误
    mb, err := r.reader.ReadAtMost(size)
    if !mb.IsEmpty() {
        r.leftOverSize -= mb.Len()
        return mb, nil
    }
    return nil, err
}

# 块写入器结构体
type ChunkStreamWriter struct {
    sizeEncoder ChunkSizeEncoder
    writer      buf.Writer
}

# 创建新的块写入器实例
func NewChunkStreamWriter(sizeEncoder ChunkSizeEncoder, writer io.Writer) *ChunkStreamWriter {
    return &ChunkStreamWriter{
        sizeEncoder: sizeEncoder,
        writer:      buf.NewWriter(writer),
    }
}

# 写入多个缓冲区的方法，返回可能的错误
func (w *ChunkStreamWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    # 定义切片大小
    const sliceSize = 8192
    # 获取多个缓冲区的总长度
    mbLen := mb.Len()
    # 创建一个空的多个缓冲区，容量为总长度除以缓冲区大小加上总长度除以切片大小再加2
    mb2Write := make(buf.MultiBuffer, 0, mbLen/buf.Size+mbLen/sliceSize+2)

    # 循环处理多个缓冲区
    for {
        # 将多个缓冲区按照切片大小进行分割
        mb2, slice := buf.SplitSize(mb, sliceSize)
        mb = mb2

        # 创建一个新的缓冲区，将切片大小编码后的数据写入其中
        b := buf.New()
        w.sizeEncoder.Encode(uint16(slice.Len()), b.Extend(w.sizeEncoder.SizeBytes()))
        mb2Write = append(mb2Write, b)
        # 将切片数据追加到新的多个缓冲区中
        mb2Write = append(mb2Write, slice...)

        # 如果多个缓冲区为空，则跳出循环
        if mb.IsEmpty() {
            break
        }
    }

    # 将新的多个缓冲区写入到写入器中，返回可能的错误
    return w.writer.WriteMultiBuffer(mb2Write)
}
```