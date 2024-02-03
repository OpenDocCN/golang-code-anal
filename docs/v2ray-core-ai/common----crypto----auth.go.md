# `v2ray-core\common\crypto\auth.go`

```go
package crypto

import (
    "crypto/cipher"  // 导入加密解密相关的包
    "io"  // 导入输入输出相关的包
    "math/rand"  // 导入随机数生成相关的包

    "v2ray.com/core/common"  // 导入通用功能相关的包
    "v2ray.com/core/common/buf"  // 导入缓冲区相关的包
    "v2ray.com/core/common/bytespool"  // 导入字节池相关的包
    "v2ray.com/core/common/protocol"  // 导入协议相关的包
)

type BytesGenerator func() []byte  // 定义一个函数类型 BytesGenerator，用于生成字节流

func GenerateEmptyBytes() BytesGenerator {  // 生成一个空字节流的函数
    var b [1]byte
    return func() []byte {
        return b[:0]
    }
}

func GenerateStaticBytes(content []byte) BytesGenerator {  // 生成一个固定内容字节流的函数
    return func() []byte {
        return content
    }
}

func GenerateIncreasingNonce(nonce []byte) BytesGenerator {  // 生成一个递增的随机数字节流的函数
    c := append([]byte(nil), nonce...)
    return func() []byte {
        for i := range c {
            c[i]++
            if c[i] != 0 {
                break
            }
        }
        return c
    }
}

func GenerateInitialAEADNonce() BytesGenerator {  // 生成一个初始的 AEAD 随机数字节流的函数
    return GenerateIncreasingNonce([]byte{0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF})
}

type Authenticator interface {  // 定义一个接口 Authenticator
    NonceSize() int  // 返回随机数大小
    Overhead() int  // 返回额外开销
    Open(dst, cipherText []byte) ([]byte, error)  // 解密函数
    Seal(dst, plainText []byte) ([]byte, error)  // 加密函数
}

type AEADAuthenticator struct {  // AEADAuthenticator 结构体
    cipher.AEAD  // AEAD 加密解密对象
    NonceGenerator          BytesGenerator  // 随机数生成器
    AdditionalDataGenerator BytesGenerator  // 额外数据生成器
}

func (v *AEADAuthenticator) Open(dst, cipherText []byte) ([]byte, error) {  // 解密函数
    iv := v.NonceGenerator()  // 生成随机数
    if len(iv) != v.AEAD.NonceSize() {  // 判断随机数大小是否合法
        return nil, newError("invalid AEAD nonce size: ", len(iv))  // 返回错误信息
    }

    var additionalData []byte  // 定义额外数据
    if v.AdditionalDataGenerator != nil {  // 判断是否有额外数据生成器
        additionalData = v.AdditionalDataGenerator()  // 生成额外数据
    }
    return v.AEAD.Open(dst, iv, cipherText, additionalData)  // 调用 AEAD 对象的解密函数
}

func (v *AEADAuthenticator) Seal(dst, plainText []byte) ([]byte, error) {  // 加密函数
    iv := v.NonceGenerator()  // 生成随机数
    if len(iv) != v.AEAD.NonceSize() {  // 判断随机数大小是否合法
        return nil, newError("invalid AEAD nonce size: ", len(iv))  // 返回错误信息
    }

    var additionalData []byte  // 定义额外数据
    if v.AdditionalDataGenerator != nil {  // 判断是否有额外数据生成器
        additionalData = v.AdditionalDataGenerator()  // 生成额外数据
    }
    # 使用AEAD算法对明文进行加密，并将结果封装成密文
    return v.AEAD.Seal(dst, iv, plainText, additionalData), nil
// 定义 AuthenticationReader 结构体，包含各种字段用于身份验证
type AuthenticationReader struct {
    auth         Authenticator
    reader       *buf.BufferedReader
    sizeParser   ChunkSizeDecoder
    sizeBytes    []byte
    transferType protocol.TransferType
    padding      PaddingLengthGenerator
    size         uint16
    paddingLen   uint16
    hasSize      bool
    done         bool
}

// 创建并返回一个新的 AuthenticationReader 对象
func NewAuthenticationReader(auth Authenticator, sizeParser ChunkSizeDecoder, reader io.Reader, transferType protocol.TransferType, paddingLen PaddingLengthGenerator) *AuthenticationReader {
    // 初始化 AuthenticationReader 对象的字段
    r := &AuthenticationReader{
        auth:         auth,
        sizeParser:   sizeParser,
        transferType: transferType,
        padding:      paddingLen,
        sizeBytes:    make([]byte, sizeParser.SizeBytes()),
    }
    // 检查 reader 是否为 *buf.BufferedReader 类型，如果是则直接赋值给 r.reader，否则创建一个新的 *buf.BufferedReader 对象
    if breader, ok := reader.(*buf.BufferedReader); ok {
        r.reader = breader
    } else {
        r.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
    }
    return r
}

// 读取身份验证数据的大小和填充长度
func (r *AuthenticationReader) readSize() (uint16, uint16, error) {
    // 如果已经有大小和填充长度，则直接返回，否则继续读取
    if r.hasSize {
        r.hasSize = false
        return r.size, r.paddingLen, nil
    }
    // 从 reader 中读取 sizeBytes，如果出错则返回错误
    if _, err := io.ReadFull(r.reader, r.sizeBytes); err != nil {
        return 0, 0, err
    }
    var padding uint16
    // 如果存在填充长度生成器，则获取填充长度
    if r.padding != nil {
        padding = r.padding.NextPaddingLen()
    }
    // 解码 sizeBytes 获取大小，并返回大小、填充长度和可能的错误
    size, err := r.sizeParser.Decode(r.sizeBytes)
    return size, padding, err
}

// 定义一个错误变量 errSoft，表示等待更多数据
var errSoft = newError("waiting for more data")

// 从 reader 中读取数据到缓冲区，并进行身份验证解密
func (r *AuthenticationReader) readBuffer(size int32, padding int32) (*buf.Buffer, error) {
    b := buf.New()
    // 从 reader 中读取指定大小的数据到缓冲区 b，如果出错则释放缓冲区并返回错误
    if _, err := b.ReadFullFrom(r.reader, size); err != nil {
        b.Release()
        return nil, err
    }
    size -= padding
    // 对缓冲区中的数据进行身份验证解密，返回解密后的数据和可能的错误
    rb, err := r.auth.Open(b.BytesTo(0), b.BytesTo(size))
    if err != nil {
        b.Release()
        return nil, err
    }
    b.Resize(0, int32(len(rb)))
    return b, nil
}

// 从 reader 中读取数据到多缓冲区，并进行身份验证解密
func (r *AuthenticationReader) readInternal(soft bool, mb *buf.MultiBuffer) error {
    # 如果软错误并且读取器缓冲的字节数小于解析器期望的大小字节数，则返回软错误
    if soft && r.reader.BufferedBytes() < r.sizeParser.SizeBytes() {
        return errSoft
    }

    # 如果已经完成解析，则返回文件结束错误
    if r.done {
        return io.EOF
    }

    # 读取大小和填充信息
    size, padding, err := r.readSize()
    if err != nil {
        return err
    }

    # 如果大小等于认证开销加上填充大小，则标记为完成并返回文件结束错误
    if size == uint16(r.auth.Overhead())+padding {
        r.done = true
        return io.EOF
    }

    # 如果软错误并且大小大于读取器缓冲的字节数，则设置大小、填充长度和标记，并返回软错误
    if soft && int32(size) > r.reader.BufferedBytes() {
        r.size = size
        r.paddingLen = padding
        r.hasSize = true
        return errSoft
    }

    # 如果大小小于等于缓冲区大小，则读取缓冲区，并将结果追加到合并字节切片中
    if size <= buf.Size {
        b, err := r.readBuffer(int32(size), int32(padding))
        if err != nil {
            return nil
        }
        *mb = append(*mb, b)
        return nil
    }

    # 分配大小为size的字节切片，函数结束时释放内存
    payload := bytespool.Alloc(int32(size))
    defer bytespool.Free(payload)

    # 从读取器中读取size大小的数据到payload中
    if _, err := io.ReadFull(r.reader, payload[:size]); err != nil {
        return err
    }

    # 减去填充大小
    size -= padding

    # 使用认证对象对payload进行解密，将结果追加到合并字节切片中
    rb, err := r.auth.Open(payload[:0], payload[:size])
    if err != nil {
        return err
    }

    *mb = buf.MergeBytes(*mb, rb)
    return nil
}

func (r *AuthenticationReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 定义读取的缓冲区大小为16
    const readSize = 16
    // 创建一个初始容量为readSize的多缓冲区
    mb := make(buf.MultiBuffer, 0, readSize)
    // 调用readInternal方法读取数据到多缓冲区
    if err := r.readInternal(false, &mb); err != nil {
        // 如果出现错误，释放多缓冲区并返回错误
        buf.ReleaseMulti(mb)
        return nil, err
    }

    // 循环读取数据到多缓冲区
    for i := 1; i < readSize; i++ {
        // 调用readInternal方法读取数据到多缓冲区
        err := r.readInternal(true, &mb)
        // 如果出现软错误或者到达文件末尾，跳出循环
        if err == errSoft || err == io.EOF {
            break
        }
        // 如果出现其他错误，释放多缓冲区并返回错误
        if err != nil {
            buf.ReleaseMulti(mb)
            return nil, err
        }
    }

    // 返回读取的多缓冲区和nil错误
    return mb, nil
}

type AuthenticationWriter struct {
    auth         Authenticator
    writer       buf.Writer
    sizeParser   ChunkSizeEncoder
    transferType protocol.TransferType
    padding      PaddingLengthGenerator
}

func NewAuthenticationWriter(auth Authenticator, sizeParser ChunkSizeEncoder, writer io.Writer, transferType protocol.TransferType, padding PaddingLengthGenerator) *AuthenticationWriter {
    // 创建一个AuthenticationWriter对象并初始化字段
    w := &AuthenticationWriter{
        auth:         auth,
        writer:       buf.NewWriter(writer),
        sizeParser:   sizeParser,
        transferType: transferType,
    }
    // 如果padding不为nil，设置padding字段
    if padding != nil {
        w.padding = padding
    }
    // 返回创建的AuthenticationWriter对象
    return w
}

func (w *AuthenticationWriter) seal(b []byte) (*buf.Buffer, error) {
    // 计算加密后的数据大小
    encryptedSize := int32(len(b) + w.auth.Overhead())
    var paddingSize int32
    // 如果padding不为nil，计算填充大小
    if w.padding != nil {
        paddingSize = int32(w.padding.NextPaddingLen())
    }

    // 计算总大小
    sizeBytes := w.sizeParser.SizeBytes()
    totalSize := sizeBytes + encryptedSize + paddingSize
    // 如果总大小超过buf.Size，返回错误
    if totalSize > buf.Size {
        return nil, newError("size too large: ", totalSize)
    }

    // 创建一个新的缓冲区
    eb := buf.New()
    // 编码加密后的数据大小
    w.sizeParser.Encode(uint16(encryptedSize+paddingSize), eb.Extend(sizeBytes))
    // 使用auth对数据进行加密
    if _, err := w.auth.Seal(eb.Extend(encryptedSize)[:0], b); err != nil {
        // 如果出现错误，释放缓冲区并返回错误
        eb.Release()
        return nil, err
    }
    // 如果填充大小大于0
    if paddingSize > 0 {
        // 由于块的大小和填充长度已加密，填充内容并不重要。
        // 创建填充字节切片，长度为填充大小
        paddingBytes := eb.Extend(paddingSize)
        // 生成随机填充内容
        common.Must2(rand.Read(paddingBytes))
    }
    // 返回填充后的块和空指针
    return eb, nil
}
// writeStream 方法用于将数据流写入，返回错误信息
func (w *AuthenticationWriter) writeStream(mb buf.MultiBuffer) error {
    // 在函数结束时释放多缓冲区
    defer buf.ReleaseMulti(mb)

    var maxPadding int32
    // 如果存在填充，则获取最大填充长度
    if w.padding != nil {
        maxPadding = int32(w.padding.MaxPaddingLen())
    }

    // 计算有效载荷大小
    payloadSize := buf.Size - int32(w.auth.Overhead()) - w.sizeParser.SizeBytes() - maxPadding
    // 创建一个多缓冲区用于写入
    mb2Write := make(buf.MultiBuffer, 0, len(mb)+10)

    // 创建一个临时缓冲区
    temp := buf.New()
    // 在函数结束时释放临时缓冲区
    defer temp.Release()

    // 从临时缓冲区中扩展指定大小的原始字节
    rawBytes := temp.Extend(payloadSize)

    // 循环处理多缓冲区中的数据
    for {
        // 从多缓冲区中分割指定长度的字节到原始字节中
        nb, nBytes := buf.SplitBytes(mb, rawBytes)
        mb = nb

        // 对原始字节进行加密封装
        eb, err := w.seal(rawBytes[:nBytes])

        // 如果加密出现错误，则释放多缓冲区并返回错误信息
        if err != nil {
            buf.ReleaseMulti(mb2Write)
            return err
        }
        // 将加密后的数据追加到待写入的多缓冲区中
        mb2Write = append(mb2Write, eb)
        // 如果多缓冲区已经为空，则跳出循环
        if mb.IsEmpty() {
            break
        }
    }

    // 将待写入的多缓冲区数据写入到底层写入器中
    return w.writer.WriteMultiBuffer(mb2Write)
}

// writePacket 方法用于将数据包写入，返回错误信息
func (w *AuthenticationWriter) writePacket(mb buf.MultiBuffer) error {
    // 在函数结束时释放多缓冲区
    defer buf.ReleaseMulti(mb)

    // 创建一个多缓冲区用于写入
    mb2Write := make(buf.MultiBuffer, 0, len(mb)+1)

    // 遍历多缓冲区中的数据包
    for _, b := range mb {
        // 如果数据包为空，则继续下一次循环
        if b.IsEmpty() {
            continue
        }

        // 对数据包进行加密封装
        eb, err := w.seal(b.Bytes())
        // 如果加密出现错误，则继续下一次循环
        if err != nil {
            continue
        }

        // 将加密后的数据包追加到待写入的多缓冲区中
        mb2Write = append(mb2Write, eb)
    }

    // 如果待写入的多缓冲区为空，则返回空
    if mb2Write.IsEmpty() {
        return nil
    }

    // 将待写入的多缓冲区数据写入到底层写入器中
    return w.writer.WriteMultiBuffer(mb2Write)
}

// WriteMultiBuffer 方法实现了 buf.Writer 接口
func (w *AuthenticationWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 如果多缓冲区为空，则对空字节进行加密封装
    if mb.IsEmpty() {
        eb, err := w.seal([]byte{})
        common.Must(err)
        // 将加密后的数据写入到底层写入器中
        return w.writer.WriteMultiBuffer(buf.MultiBuffer{eb})
    }

    // 如果传输类型为数据流，则调用 writeStream 方法进行写入
    if w.transferType == protocol.TransferTypeStream {
        return w.writeStream(mb)
    }

    // 否则调用 writePacket 方法进行写入
    return w.writePacket(mb)
}
```