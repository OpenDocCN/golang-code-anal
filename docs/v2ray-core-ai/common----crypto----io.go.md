# `v2ray-core\common\crypto\io.go`

```go
package crypto

import (
    "crypto/cipher"  // 导入加密算法的包
    "io"  // 导入输入输出的包

    "v2ray.com/core/common/buf"  // 导入自定义的缓冲区包
)

type CryptionReader struct {
    stream cipher.Stream  // 定义加密流
    reader io.Reader  // 定义输入流
}

func NewCryptionReader(stream cipher.Stream, reader io.Reader) *CryptionReader {
    return &CryptionReader{
        stream: stream,  // 初始化加密流
        reader: reader,  // 初始化输入流
    }
}

func (r *CryptionReader) Read(data []byte) (int, error) {
    nBytes, err := r.reader.Read(data)  // 从输入流读取数据
    if nBytes > 0 {
        r.stream.XORKeyStream(data[:nBytes], data[:nBytes])  // 对读取的数据进行解密
    }
    return nBytes, err  // 返回读取的字节数和可能的错误
}

var (
    _ buf.Writer = (*CryptionWriter)(nil)  // 确保 CryptionWriter 实现了 buf.Writer 接口
)

type CryptionWriter struct {
    stream    cipher.Stream  // 定义加密流
    writer    io.Writer  // 定义输出流
    bufWriter buf.Writer  // 定义缓冲区写入器
}

// NewCryptionWriter creates a new CryptionWriter.
func NewCryptionWriter(stream cipher.Stream, writer io.Writer) *CryptionWriter {
    return &CryptionWriter{
        stream:    stream,  // 初始化加密流
        writer:    writer,  // 初始化输出流
        bufWriter: buf.NewWriter(writer),  // 初始化缓冲区写入器
    }
}

// Write implements io.Writer.Write().
func (w *CryptionWriter) Write(data []byte) (int, error) {
    w.stream.XORKeyStream(data, data)  // 对数据进行加密

    if err := buf.WriteAllBytes(w.writer, data); err != nil {  // 将加密后的数据写入输出流
        return 0, err  // 如果出现错误，返回错误信息
    }
    return len(data), nil  // 返回写入的字节数和无错误
}

// WriteMultiBuffer implements buf.Writer.
func (w *CryptionWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    for _, b := range mb {
        w.stream.XORKeyStream(b.Bytes(), b.Bytes())  // 对多个缓冲区的数据进行加密
    }

    return w.bufWriter.WriteMultiBuffer(mb)  // 将加密后的多个缓冲区写入输出流
}
```