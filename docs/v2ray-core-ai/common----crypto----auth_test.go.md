# `v2ray-core\common\crypto\auth_test.go`

```go
package crypto_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "crypto/aes"  // 导入 aes 包，用于实现高级加密标准（Advanced Encryption Standard）算法
    "crypto/cipher"  // 导入 cipher 包，用于实现加密和解密
    "crypto/rand"  // 导入 rand 包，用于生成随机数
    "io"  // 导入 io 包，用于实现 I/O 操作
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/google/go-cmp/cmp"  // 导入 cmp 包，用于比较两个值是否相等

    "v2ray.com/core/common"  // 导入 common 包
    "v2ray.com/core/common/buf"  // 导入 buf 包
    . "v2ray.com/core/common/crypto"  // 导入 crypto 包，并将其中的所有公共函数和变量导入当前包
    "v2ray.com/core/common/protocol"  // 导入 protocol 包
)

func TestAuthenticationReaderWriter(t *testing.T) {
    key := make([]byte, 16)  // 创建一个长度为 16 的字节切片 key
    rand.Read(key)  // 生成随机数填充 key
    block, err := aes.NewCipher(key)  // 使用 key 创建一个 AES 加密算法的实例
    common.Must(err)  // 检查是否有错误发生，如果有则触发 panic

    aead, err := cipher.NewGCM(block)  // 使用 block 创建一个 GCM（Galois/Counter Mode）加密算法的实例
    common.Must(err)  // 检查是否有错误发生，如果有则触发 panic

    const payloadSize = 1024 * 80  // 定义 payloadSize 常量为 1024 * 80
    rawPayload := make([]byte, payloadSize)  // 创建一个长度为 payloadSize 的原始数据字节切片 rawPayload
    rand.Read(rawPayload)  // 生成随机数填充 rawPayload

    payload := buf.MergeBytes(nil, rawPayload)  // 将 rawPayload 合并为一个 buf.MultiBuffer 对象 payload

    cache := bytes.NewBuffer(nil)  // 创建一个新的字节缓冲区 cache
    iv := make([]byte, 12)  // 创建一个长度为 12 的初始化向量 iv
    rand.Read(iv)  // 生成随机数填充 iv

    writer := NewAuthenticationWriter(&AEADAuthenticator{
        AEAD:                    aead,  // 设置 AEADAuthenticator 结构体的 AEAD 字段为 aead
        NonceGenerator:          GenerateStaticBytes(iv),  // 设置 AEADAuthenticator 结构体的 NonceGenerator 字段为生成的静态字节 iv
        AdditionalDataGenerator: GenerateEmptyBytes(),  // 设置 AEADAuthenticator 结构体的 AdditionalDataGenerator 字段为生成的空字节
    }, PlainChunkSizeParser{}, cache, protocol.TransferTypeStream, nil)  // 创建一个新的认证写入器 writer

    common.Must(writer.WriteMultiBuffer(payload))  // 使用 writer 写入 payload 到缓冲区
    if cache.Len() <= 1024*80 {  // 如果缓冲区长度小于等于 1024*80
        t.Error("cache len: ", cache.Len())  // 输出错误信息
    }
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))  // 使用 writer 写入空的 buf.MultiBuffer 到缓冲区

    reader := NewAuthenticationReader(&AEADAuthenticator{
        AEAD:                    aead,  // 设置 AEADAuthenticator 结构体的 AEAD 字段为 aead
        NonceGenerator:          GenerateStaticBytes(iv),  // 设置 AEADAuthenticator 结构体的 NonceGenerator 字段为生成的静态字节 iv
        AdditionalDataGenerator: GenerateEmptyBytes(),  // 设置 AEADAuthenticator 结构体的 AdditionalDataGenerator 字段为生成的空字节
    }, PlainChunkSizeParser{}, cache, protocol.TransferTypeStream, nil)  // 创建一个新的认证读取器 reader

    var mb buf.MultiBuffer  // 声明一个 buf.MultiBuffer 变量 mb

    for mb.Len() < payloadSize {  // 当 mb 的长度小于 payloadSize 时循环
        mb2, err := reader.ReadMultiBuffer()  // 从 reader 中读取 buf.MultiBuffer 到 mb2
        common.Must(err)  // 检查是否有错误发生，如果有则触发 panic

        mb, _ = buf.MergeMulti(mb, mb2)  // 将 mb2 合并到 mb
    }

    if mb.Len() != payloadSize {  // 如果 mb 的长度不等于 payloadSize
        t.Error("mb len: ", mb.Len())  // 输出错误信息
    }

    mbContent := make([]byte, payloadSize)  // 创建一个长度为 payloadSize 的字节切片 mbContent
    buf.SplitBytes(mb, mbContent)  // 将 mb 中的数据拆分到 mbContent
    if r := cmp.Diff(mbContent, rawPayload); r != "" {  // 使用 cmp 包比较 mbContent 和 rawPayload
        t.Error(r)  // 输出错误信息
    }

    _, err = reader.ReadMultiBuffer()  // 从 reader 中读取 buf.MultiBuffer
    # 如果错误不是文件结束错误
    if err != io.EOF:
        # 输出错误信息
        t.Error("error: ", err)
    # 结束条件判断
func TestAuthenticationReaderWriterPacket(t *testing.T) {
    // 生成一个16字节的随机密钥
    key := make([]byte, 16)
    common.Must2(rand.Read(key))
    // 使用密钥创建一个AES加密块
    block, err := aes.NewCipher(key)
    common.Must(err)

    // 使用加密块创建一个GCM模式的AEAD对象
    aead, err := cipher.NewGCM(block)
    common.Must(err)

    // 创建一个缓冲区
    cache := buf.New()
    // 生成一个12字节的随机初始化向量
    iv := make([]byte, 12)
    rand.Read(iv)

    // 创建一个认证写入器，使用AEADAuthenticator进行认证
    writer := NewAuthenticationWriter(&AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateStaticBytes(iv),
        AdditionalDataGenerator: GenerateEmptyBytes(),
    }, PlainChunkSizeParser{}, cache, protocol.TransferTypePacket, nil)

    // 创建一个多缓冲区payload，写入数据
    var payload buf.MultiBuffer
    pb1 := buf.New()
    pb1.Write([]byte("abcd"))
    payload = append(payload, pb1)

    pb2 := buf.New()
    pb2.Write([]byte("efgh"))
    payload = append(payload, pb2)

    // 写入payload到缓冲区
    common.Must(writer.WriteMultiBuffer(payload))
    // 检查缓冲区长度是否为0
    if cache.Len() == 0 {
        t.Error("cache len: ", cache.Len())
    }

    // 写入空的多缓冲区到缓冲区
    common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{}))

    // 创建一个认证读取器，使用AEADAuthenticator进行认证
    reader := NewAuthenticationReader(&AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateStaticBytes(iv),
        AdditionalDataGenerator: GenerateEmptyBytes(),
    }, PlainChunkSizeParser{}, cache, protocol.TransferTypePacket, nil)

    // 从缓冲区读取数据到多缓冲区
    mb, err := reader.ReadMultiBuffer()
    common.Must(err)

    // 从多缓冲区中分离出第一个缓冲区，并检查其内容
    mb, b1 := buf.SplitFirst(mb)
    if b1.String() != "abcd" {
        t.Error("b1: ", b1.String())
    }

    // 从多缓冲区中分离出第二个缓冲区，并检查其内容
    mb, b2 := buf.SplitFirst(mb)
    if b2.String() != "efgh" {
        t.Error("b2: ", b2.String())
    }

    // 检查多缓冲区是否为空
    if !mb.IsEmpty() {
        t.Error("not empty")
    }

    // 从读取器中读取多缓冲区，预期返回io.EOF
    _, err = reader.ReadMultiBuffer()
    if err != io.EOF {
        t.Error("error: ", err)
    }
}
```