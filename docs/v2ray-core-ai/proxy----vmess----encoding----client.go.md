# `v2ray-core\proxy\vmess\encoding\client.go`

```
package encoding

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "context" // 导入 context 包，用于处理上下文
    "crypto/aes" // 导入 aes 包，用于实现高级加密标准
    "crypto/cipher" // 导入 cipher 包，用于实现加密和解密
    "crypto/md5" // 导入 md5 包，用于实现 MD5 哈希算法
    "crypto/rand" // 导入 rand 包，用于生成随机数
    "crypto/sha256" // 导入 sha256 包，用于实现 SHA-256 哈希算法
    "encoding/binary" // 导入 binary 包，用于实现二进制数据和文本数据的转换
    "hash" // 导入 hash 包，用于实现哈希函数
    "hash/fnv" // 导入 fnv 包，用于实现非加密哈希函数
    "io" // 导入 io 包，用于实现 I/O 操作

    "golang.org/x/crypto/chacha20poly1305" // 导入 chacha20poly1305 包，用于实现 ChaCha20-Poly1305 加密算法

    "v2ray.com/core/common" // 导入 common 包，用于通用功能
    "v2ray.com/core/common/bitmask" // 导入 bitmask 包，用于位掩码操作
    "v2ray.com/core/common/buf" // 导入 buf 包，用于缓冲区操作
    "v2ray.com/core/common/crypto" // 导入 crypto 包，用于加密操作
    "v2ray.com/core/common/dice" // 导入 dice 包，用于生成随机数
    "v2ray.com/core/common/protocol" // 导入 protocol 包，用于协议相关操作
    "v2ray.com/core/common/serial" // 导入 serial 包，用于序列化操作
    "v2ray.com/core/proxy/vmess" // 导入 vmess 包，用于 VMess 相关操作
    vmessaead "v2ray.com/core/proxy/vmess/aead" // 导入 vmessaead 包，用于 VMess AEAD 相关操作
)

func hashTimestamp(h hash.Hash, t protocol.Timestamp) []byte {
    common.Must2(serial.WriteUint64(h, uint64(t))) // 将时间戳 t 写入哈希函数 h 中
    common.Must2(serial.WriteUint64(h, uint64(t))) // 将时间戳 t 再次写入哈希函数 h 中
    common.Must2(serial.WriteUint64(h, uint64(t))) // 将时间戳 t 再次写入哈希函数 h 中
    common.Must2(serial.WriteUint64(h, uint64(t))) // 将时间戳 t 再次写入哈希函数 h 中
    return h.Sum(nil) // 返回哈希函数 h 的结果
}

// ClientSession stores connection session info for VMess client.
type ClientSession struct {
    isAEAD          bool // 标识是否使用 AEAD 加密
    idHash          protocol.IDHash // 存储 ID 的哈希值
    requestBodyKey  [16]byte // 请求体的密钥
    requestBodyIV   [16]byte // 请求体的初始化向量
    responseBodyKey [16]byte // 响应体的密钥
    responseBodyIV  [16]byte // 响应体的初始化向量
    responseReader  io.Reader // 响应体的读取器
    responseHeader  byte // 响应头
}

// NewClientSession creates a new ClientSession.
func NewClientSession(isAEAD bool, idHash protocol.IDHash, ctx context.Context) *ClientSession {

    session := &ClientSession{ // 创建 ClientSession 对象
        isAEAD: isAEAD, // 设置是否使用 AEAD 加密
        idHash: idHash, // 设置 ID 的哈希值
    }

    randomBytes := make([]byte, 33) // 创建长度为 33 的随机字节切片
    common.Must2(rand.Read(randomBytes)) // 生成随机字节并写入 randomBytes 中
    copy(session.requestBodyKey[:], randomBytes[:16]) // 将 randomBytes 的前 16 个字节复制到 requestBodyKey 中
    copy(session.requestBodyIV[:], randomBytes[16:32]) // 将 randomBytes 的第 16 到 32 个字节复制到 requestBodyIV 中
    session.responseHeader = randomBytes[32] // 设置响应头为 randomBytes 的第 32 个字节

    if !session.isAEAD { // 如果不使用 AEAD 加密
        session.responseBodyKey = md5.Sum(session.requestBodyKey[:]) // 使用 MD5 哈希算法计算响应体的密钥
        session.responseBodyIV = md5.Sum(session.requestBodyIV[:]) // 使用 MD5 哈希算法计算响应体的初始化向量
    } else {
        # 如果不是第一次握手，则使用 SHA256 对请求体密钥进行哈希，并复制前16个字节到响应体密钥
        BodyKey := sha256.Sum256(session.requestBodyKey[:])
        copy(session.responseBodyKey[:], BodyKey[:16])
        # 使用 SHA256 对请求体初始化向量进行哈希，并复制前16个字节到响应体初始化向量
        BodyIV := sha256.Sum256(session.requestBodyIV[:])
        copy(session.responseBodyIV[:], BodyIV[:16])
    }
    # 返回会话对象
    return session
// EncodeRequestHeader 将请求头部编码为字节流
func (c *ClientSession) EncodeRequestHeader(header *protocol.RequestHeader, writer io.Writer) error {
    // 生成时间戳
    timestamp := protocol.NewTimestampGenerator(protocol.NowTime(), 30)()
    // 获取用户账户信息
    account := header.User.Account.(*vmess.MemoryAccount)
    // 如果不是 AEAD 加密
    if !c.isAEAD {
        // 计算 ID 的哈希值
        idHash := c.idHash(account.AnyValidID().Bytes())
        // 写入 ID 哈希值到 writer
        common.Must2(serial.WriteUint64(idHash, uint64(timestamp)))
        common.Must2(writer.Write(idHash.Sum(nil)))
    }

    // 创建缓冲区
    buffer := buf.New()
    // 在函数返回时释放缓冲区
    defer buffer.Release()

    // 写入协议版本号
    common.Must(buffer.WriteByte(Version))
    // 写入请求体 IV
    common.Must2(buffer.Write(c.requestBodyIV[:]))
    // 写入请求体密钥
    common.Must2(buffer.Write(c.requestBodyKey[:]))
    // 写入响应头部
    common.Must(buffer.WriteByte(c.responseHeader))
    // 写入请求头部选项
    common.Must(buffer.WriteByte(byte(header.Option)))

    // 生成填充长度
    padingLen := dice.Roll(16)
    // 计算安全位和安全类型
    security := byte(padingLen<<4) | byte(header.Security)
    common.Must2(buffer.Write([]byte{security, byte(0), byte(header.Command)}))

    // 如果命令不是 Mux，则写入地址和端口
    if header.Command != protocol.RequestCommandMux {
        if err := addrParser.WriteAddressPort(buffer, header.Address, header.Port); err != nil {
            return newError("failed to writer address and port").Base(err)
        }
    }

    // 如果填充长度大于 0，则填充随机数据
    if padingLen > 0 {
        common.Must2(buffer.ReadFullFrom(rand.Reader, int32(padingLen)))
    }

    // 计算哈希值
    {
        fnv1a := fnv.New32a()
        common.Must2(fnv1a.Write(buffer.Bytes()))
        hashBytes := buffer.Extend(int32(fnv1a.Size()))
        fnv1a.Sum(hashBytes[:0])
    }

    // 如果不是 AEAD 加密
    if !c.isAEAD {
        // 计算 IV
        iv := hashTimestamp(md5.New(), timestamp)
        // 创建 AES 加密流
        aesStream := crypto.NewAesEncryptionStream(account.ID.CmdKey(), iv[:])
        // 对缓冲区进行加密
        aesStream.XORKeyStream(buffer.Bytes(), buffer.Bytes())
        // 将加密后的数据写入 writer
        common.Must2(writer.Write(buffer.Bytes()))
    } else {
        // 复制固定长度的命令密钥
        var fixedLengthCmdKey [16]byte
        copy(fixedLengthCmdKey[:], account.ID.CmdKey())
        // 使用 AEAD 加密算法对数据进行加密
        vmessout := vmessaead.SealVMessAEADHeader(fixedLengthCmdKey, buffer.Bytes())
        // 将加密后的数据写入 writer
        common.Must2(io.Copy(writer, bytes.NewReader(vmessout)))
    }

    return nil
}
func (c *ClientSession) EncodeRequestBody(request *protocol.RequestHeader, writer io.Writer) buf.Writer {
    // 定义一个变量 sizeParser，类型为 crypto.ChunkSizeEncoder，初始值为 crypto.PlainChunkSizeParser{}
    var sizeParser crypto.ChunkSizeEncoder = crypto.PlainChunkSizeParser{}
    // 如果请求选项中包含 protocol.RequestOptionChunkMasking，则将 sizeParser 替换为 NewShakeSizeParser(c.requestBodyIV[:])
    if request.Option.Has(protocol.RequestOptionChunkMasking) {
        sizeParser = NewShakeSizeParser(c.requestBodyIV[:])
    }
    // 定义一个变量 padding，类型为 crypto.PaddingLengthGenerator
    var padding crypto.PaddingLengthGenerator
    // 如果请求选项中包含 protocol.RequestOptionGlobalPadding，则将 padding 替换为 sizeParser.(crypto.PaddingLengthGenerator)
    if request.Option.Has(protocol.RequestOptionGlobalPadding) {
        padding = sizeParser.(crypto.PaddingLengthGenerator)
    }

    // 根据请求的安全类型进行不同的处理
    switch request.Security {
    case protocol.SecurityType_NONE:
        // 如果请求选项中包含 protocol.RequestOptionChunkStream，并且请求的传输类型为 protocol.TransferTypeStream，则返回一个新的 crypto.NewChunkStreamWriter(sizeParser, writer)
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            if request.Command.TransferType() == protocol.TransferTypeStream {
                return crypto.NewChunkStreamWriter(sizeParser, writer)
            }
            // 否则，创建一个 AEAD 认证器，并返回一个新的 crypto.NewAuthenticationWriter(auth, sizeParser, writer, protocol.TransferTypePacket, padding)
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(NoOpAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            return crypto.NewAuthenticationWriter(auth, sizeParser, writer, protocol.TransferTypePacket, padding)
        }
        // 如果不满足上述条件，则返回一个新的 buf.NewWriter(writer)
        return buf.NewWriter(writer)
    case protocol.SecurityType_LEGACY:
        // 创建一个 AES 加密流，并返回一个新的 crypto.NewAesEncryptionStream(c.requestBodyKey[:], c.requestBodyIV[:])
        aesStream := crypto.NewAesEncryptionStream(c.requestBodyKey[:], c.requestBodyIV[:])
        // 创建一个加密写入器，并返回一个新的 crypto.NewCryptionWriter(aesStream, writer)
        cryptionWriter := crypto.NewCryptionWriter(aesStream, writer)
        // 如果请求选项中包含 protocol.RequestOptionChunkStream，则创建一个 AEAD 认证器，并返回一个新的 crypto.NewAuthenticationWriter(auth, sizeParser, cryptionWriter, request.Command.TransferType(), padding)
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(FnvAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            return crypto.NewAuthenticationWriter(auth, sizeParser, cryptionWriter, request.Command.TransferType(), padding)
        }
        // 如果不满足上述条件，则返回一个新的 buf.SequentialWriter{Writer: cryptionWriter}
        return &buf.SequentialWriter{Writer: cryptionWriter}
    # 如果安全类型是AES128_GCM，则使用AES-GCM算法创建一个AEAD对象
    aead := crypto.NewAesGcm(c.requestBodyKey[:])
    # 创建一个AEAD认证器，用于对数据进行认证
    auth := &crypto.AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateChunkNonce(c.requestBodyIV[:], uint32(aead.NonceSize())),
        AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
    }
    # 返回一个新的认证写入器，用于对数据进行认证和加密
    return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
    # 如果安全类型是CHACHA20_POLY1305，则使用CHACHA20_POLY1305算法创建一个AEAD对象
    aead, err := chacha20poly1305.New(GenerateChacha20Poly1305Key(c.requestBodyKey[:]))
    # 检查是否有错误发生
    common.Must(err)
    # 创建一个AEAD认证器，用于对数据进行认证
    auth := &crypto.AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateChunkNonce(c.requestBodyIV[:], uint32(aead.NonceSize())),
        AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
    }
    # 返回一个新的认证写入器，用于对数据进行认证和加密
    return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
    # 如果安全类型不是以上两种情况，则抛出异常
    default:
        panic("Unknown security type.")
    }
}
// 解码响应头部
func (c *ClientSession) DecodeResponseHeader(reader io.Reader) (*protocol.ResponseHeader, error) {
    // 如果不是 AEAD 加密
    if !c.isAEAD {
        // 创建 AES 解密流
        aesStream := crypto.NewAesDecryptionStream(c.responseBodyKey[:], c.responseBodyIV[:])
        // 创建加密流读取器
        c.responseReader = crypto.NewCryptionReader(aesStream, reader)
    }

    // 创建缓冲区
    buffer := buf.StackNew()
    // 在函数返回时释放缓冲区
    defer buffer.Release()

    // 从响应读取器中读取 4 个字节到缓冲区
    if _, err := buffer.ReadFullFrom(c.responseReader, 4); err != nil {
        return nil, newError("failed to read response header").Base(err).AtWarning()
    }

    // 检查响应头部的第一个字节是否符合预期
    if buffer.Byte(0) != c.responseHeader {
        return nil, newError("unexpected response header. Expecting ", int(c.responseHeader), " but actually ", int(buffer.Byte(0)))
    }

    // 创建响应头部对象
    header := &protocol.ResponseHeader{
        Option: bitmask.Byte(buffer.Byte(1)),
    }

    // 如果第二个字节不为 0
    if buffer.Byte(2) != 0 {
        // 读取命令 ID 和数据长度
        cmdID := buffer.Byte(2)
        dataLen := int32(buffer.Byte(3))

        // 清空缓冲区
        buffer.Clear()
        // 从响应读取器中读取指定长度的数据到缓冲区
        if _, err := buffer.ReadFullFrom(c.responseReader, dataLen); err != nil {
            return nil, newError("failed to read response command").Base(err)
        }
        // 反序列化命令
        command, err := UnmarshalCommand(cmdID, buffer.Bytes())
        if err == nil {
            header.Command = command
        }
    }
    // 如果是 AEAD 加密
    if c.isAEAD {
        // 创建 AES 解密流
        aesStream := crypto.NewAesDecryptionStream(c.responseBodyKey[:], c.responseBodyIV[:])
        // 创建加密流读取器
        c.responseReader = crypto.NewCryptionReader(aesStream, reader)
    }
    // 返回响应头部对象和空错误
    return header, nil
}

// 解码响应体
func (c *ClientSession) DecodeResponseBody(request *protocol.RequestHeader, reader io.Reader) buf.Reader {
    // 创建块大小解码器
    var sizeParser crypto.ChunkSizeDecoder = crypto.PlainChunkSizeParser{}
    // 如果请求选项包含块掩码
    if request.Option.Has(protocol.RequestOptionChunkMasking) {
        // 创建 SHAKE 块大小解码器
        sizeParser = NewShakeSizeParser(c.responseBodyIV[:])
    }
    // 创建填充长度生成器
    var padding crypto.PaddingLengthGenerator
    // 如果请求选项包含全局填充
    if request.Option.Has(protocol.RequestOptionGlobalPadding) {
        padding = sizeParser.(crypto.PaddingLengthGenerator)
    }

    // 根据请求安全选项进行不同的处理
    switch request.Security {
    # 如果安全类型为NONE
    case protocol.SecurityType_NONE:
        # 如果请求选项包含分块流
        if request.Option.Has(protocol.RequestOptionChunkStream):
            # 如果请求命令的传输类型为流
            if request.Command.TransferType() == protocol.TransferTypeStream:
                # 返回一个新的块流读取器
                return crypto.NewChunkStreamReader(sizeParser, reader)

            # 创建一个AEAD认证器
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(NoOpAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }

            # 返回一个新的认证读取器
            return crypto.NewAuthenticationReader(auth, sizeParser, reader, protocol.TransferTypePacket, padding)
        
        # 返回一个新的缓冲区读取器
        return buf.NewReader(reader)
    
    # 如果安全类型为LEGACY
    case protocol.SecurityType_LEGACY:
        # 如果请求选项包含分块流
        if request.Option.Has(protocol.RequestOptionChunkStream):
            # 创建一个AEAD认证器
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(FnvAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            # 返回一个新的认证读取器
            return crypto.NewAuthenticationReader(auth, sizeParser, c.responseReader, request.Command.TransferType(), padding)
        
        # 返回一个新的缓冲区读取器
        return buf.NewReader(c.responseReader)
    
    # 如果安全类型为AES128_GCM
    case protocol.SecurityType_AES128_GCM:
        # 创建一个AES GCM加密器
        aead := crypto.NewAesGcm(c.responseBodyKey[:])

        # 创建一个AEAD认证器
        auth := &crypto.AEADAuthenticator{
            AEAD:                    aead,
            NonceGenerator:          GenerateChunkNonce(c.responseBodyIV[:], uint32(aead.NonceSize())),
            AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
        }
        # 返回一个新的认证读取器
        return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
    // 如果安全类型是 CHACHA20_POLY1305，则创建一个 AEAD 对象
    aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(c.responseBodyKey[:]))

    // 创建一个 AEADAuthenticator 对象，用于身份验证
    auth := &crypto.AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateChunkNonce(c.responseBodyIV[:], uint32(aead.NonceSize())),
        AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
    }

    // 返回一个新的身份验证读取器
    return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
    // 如果安全类型未知，则抛出异常
    default:
        panic("Unknown security type.")
    }
# 生成一个用于生成指定大小的随机字节序列的函数
func GenerateChunkNonce(nonce []byte, size uint32) crypto.BytesGenerator {
    # 复制传入的随机字节序列
    c := append([]byte(nil), nonce...)
    # 初始化计数器
    count := uint16(0)
    # 返回一个匿名函数，用于生成随机字节序列
    return func() []byte {
        # 将计数器的值以大端序写入到复制的随机字节序列中
        binary.BigEndian.PutUint16(c, count)
        # 计数器自增
        count++
        # 返回指定大小的随机字节序列
        return c[:size]
    }
}
```