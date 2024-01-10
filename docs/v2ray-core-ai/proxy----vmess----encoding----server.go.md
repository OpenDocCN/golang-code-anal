# `v2ray-core\proxy\vmess\encoding\server.go`

```
package encoding

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "crypto/aes" // 导入 aes 包，用于对称加密
    "crypto/cipher" // 导入 cipher 包，用于加密和解密
    "crypto/md5" // 导入 md5 包，用于计算 MD5 哈希值
    "crypto/sha256" // 导入 sha256 包，用于计算 SHA-256 哈希值
    "encoding/binary" // 导入 binary 包，用于二进制数据的读写
    "hash/fnv" // 导入 fnv 包，用于实现非加密哈希函数
    "io" // 导入 io 包，用于实现 I/O 操作
    "io/ioutil" // 导入 ioutil 包，用于实现 I/O 操作
    "sync" // 导入 sync 包，用于实现同步原语
    "time" // 导入 time 包，用于处理时间

    "golang.org/x/crypto/chacha20poly1305" // 导入 chacha20poly1305 包，用于实现 ChaCha20-Poly1305 加密算法
    "v2ray.com/core/common" // 导入 common 包，用于通用功能
    "v2ray.com/core/common/bitmask" // 导入 bitmask 包，用于位掩码操作
    "v2ray.com/core/common/buf" // 导入 buf 包，用于缓冲区操作
    "v2ray.com/core/common/crypto" // 导入 crypto 包，用于加密相关功能
    "v2ray.com/core/common/dice" // 导入 dice 包，用于生成随机数
    "v2ray.com/core/common/net" // 导入 net 包，用于网络操作
    "v2ray.com/core/common/protocol" // 导入 protocol 包，用于协议相关功能
    "v2ray.com/core/common/task" // 导入 task 包，用于任务调度
    "v2ray.com/core/proxy/vmess" // 导入 vmess 包，用于 Vmess 代理
    vmessaead "v2ray.com/core/proxy/vmess/aead" // 导入 vmessaead 包，用于 Vmess AEAD 加密
)

type sessionId struct {
    user  [16]byte // 用户 ID
    key   [16]byte // 密钥
    nonce [16]byte // 随机数
}

// SessionHistory keeps track of historical session ids, to prevent replay attacks.
type SessionHistory struct {
    sync.RWMutex // 读写锁
    cache map[sessionId]time.Time // 存储 sessionId 和时间的映射
    task  *task.Periodic // 定时任务
}

// NewSessionHistory creates a new SessionHistory object.
func NewSessionHistory() *SessionHistory {
    h := &SessionHistory{
        cache: make(map[sessionId]time.Time, 128), // 初始化缓存
    }
    h.task = &task.Periodic{
        Interval: time.Second * 30, // 设置定时任务执行间隔
        Execute:  h.removeExpiredEntries, // 执行过期条目清理函数
    }
    return h
}

// Close implements common.Closable.
func (h *SessionHistory) Close() error {
    return h.task.Close() // 关闭定时任务
}

func (h *SessionHistory) addIfNotExits(session sessionId) bool {
    h.Lock() // 加锁

    if expire, found := h.cache[session]; found && expire.After(time.Now()) {
        h.Unlock() // 解锁
        return false
    }

    h.cache[session] = time.Now().Add(time.Minute * 3) // 添加新的 session
    h.Unlock() // 解锁
    common.Must(h.task.Start()) // 启动定时任务
    return true
}

func (h *SessionHistory) removeExpiredEntries() error {
    now := time.Now() // 获取当前时间

    h.Lock() // 加锁
    defer h.Unlock() // 延迟解锁

    if len(h.cache) == 0 {
        return newError("nothing to do") // 如果缓存为空，返回错误
    }

    for session, expire := range h.cache {
        if expire.Before(now) {
            delete(h.cache, session) // 删除过期的 session 条目
        }
    }
}
    # 如果缓存中的元素数量为0
    if len(h.cache) == 0 {
        # 则初始化一个容量为128的sessionId到time.Time的映射，并赋值给缓存
        h.cache = make(map[sessionId]time.Time, 128)
    }

    # 返回空值
    return nil
// ServerSession 结构体用于在 VMess 服务器中保存会话信息
type ServerSession struct {
    userValidator   *vmess.TimedUserValidator  // 用户验证器
    sessionHistory  *SessionHistory            // 会话历史记录
    requestBodyKey  [16]byte                   // 请求体密钥
    requestBodyIV   [16]byte                   // 请求体初始化向量
    responseBodyKey [16]byte                   // 响应体密钥
    responseBodyIV  [16]byte                   // 响应体初始化向量
    responseWriter  io.Writer                  // 响应写入器
    responseHeader  byte                       // 响应头

    isAEADRequest   bool                        // 是否是 AEAD 请求

    isAEADForced    bool                        // 是否强制使用 AEAD
}

// NewServerSession 根据给定的 UserValidator 创建一个新的 ServerSession 实例
// ServerSession 实例不会拥有验证器的所有权
func NewServerSession(validator *vmess.TimedUserValidator, sessionHistory *SessionHistory) *ServerSession {
    return &ServerSession{
        userValidator:  validator,               // 用户验证器
        sessionHistory: sessionHistory,          // 会话历史记录
    }
}

// parseSecurityType 根据字节解析出安全类型
func parseSecurityType(b byte) protocol.SecurityType {
    if _, f := protocol.SecurityType_name[int32(b)]; f {
        st := protocol.SecurityType(b)
        // 为了向后兼容
        if st == protocol.SecurityType_UNKNOWN {
            st = protocol.SecurityType_LEGACY
        }
        return st
    }
    return protocol.SecurityType_UNKNOWN
}

// DecodeRequestHeader 从输入流中解码并返回（如果成功）一个 RequestHeader
func (s *ServerSession) DecodeRequestHeader(reader io.Reader) (*protocol.RequestHeader, error) {
    buffer := buf.New()
    behaviorRand := dice.NewDeterministicDice(int64(s.userValidator.GetBehaviorSeed()))
    BaseDrainSize := behaviorRand.Roll(3266)
    RandDrainMax := behaviorRand.Roll(64) + 1
    RandDrainRolled := dice.Roll(RandDrainMax)
    DrainSize := BaseDrainSize + 16 + 38 + RandDrainRolled
    readSizeRemain := DrainSize
}
    drainConnection := func(e error) error {
        // 定义一个函数，用于在发生错误时读取一定长度的数据，然后关闭连接，以偏移填充读取模式
        readSizeRemain -= int(buffer.Len())  // 减去已读取数据的长度
        if readSizeRemain > 0 {
            err := s.DrainConnN(reader, readSizeRemain)  // 读取指定长度的数据，然后关闭连接
            if err != nil {
                return newError("failed to drain connection DrainSize = ", BaseDrainSize, " ", RandDrainMax, " ", RandDrainRolled).Base(err).Base(e)  // 返回连接关闭失败的错误信息
            }
            return newError("connection drained DrainSize = ", BaseDrainSize, " ", RandDrainMax, " ", RandDrainRolled).Base(e)  // 返回连接已关闭的信息
        }
        return e  // 返回原始错误信息
    }

    defer func() {
        buffer.Release()  // 在函数返回之前释放缓冲区
    }()

    if _, err := buffer.ReadFullFrom(reader, protocol.IDBytesLen); err != nil {
        return nil, newError("failed to read request header").Base(err)  // 如果读取请求头失败，则返回错误信息
    }

    var decryptor io.Reader  // 定义一个解密器
    var vmessAccount *vmess.MemoryAccount  // 定义一个 Vmess 内存账户

    user, foundAEAD, errorAEAD := s.userValidator.GetAEAD(buffer.Bytes())  // 从缓冲区中获取用户信息和 AEAD 加密信息

    var fixedSizeAuthID [16]byte  // 定义一个固定大小的认证 ID
    copy(fixedSizeAuthID[:], buffer.Bytes())  // 将缓冲区中的数据复制到固定大小的认证 ID 中

    if foundAEAD {
        vmessAccount = user.Account.(*vmess.MemoryAccount)  // 如果找到 AEAD，则获取 Vmess 内存账户
        var fixedSizeCmdKey [16]byte  // 定义一个固定大小的命令密钥
        copy(fixedSizeCmdKey[:], vmessAccount.ID.CmdKey())  // 将 Vmess 账户的命令密钥复制到固定大小的命令密钥中
        aeadData, shouldDrain, errorReason, bytesRead := vmessaead.OpenVMessAEADHeader(fixedSizeCmdKey, fixedSizeAuthID, reader)  // 打开 Vmess AEAD 头部
        if errorReason != nil {
            if shouldDrain {
                readSizeRemain -= bytesRead  // 减去已读取数据的长度
                return nil, drainConnection(newError("AEAD read failed").Base(errorReason))  // 如果需要关闭连接，则返回 AEAD 读取失败的错误信息
            } else {
                return nil, drainConnection(newError("AEAD read failed, drain skiped").Base(errorReason))  // 如果不需要关闭连接，则返回 AEAD 读取失败，但跳过关闭连接的错误信息
            }
        }
        decryptor = bytes.NewReader(aeadData)  // 将解密后的数据存储到解密器中
        s.isAEADRequest = true  // 设置 AEAD 请求标志为 true
    # 如果不强制使用 AEAD 并且错误为未找到 AEAD，则执行以下操作
    } else if !s.isAEADForced && errorAEAD == vmessaead.ErrNotFound {
        # 从缓冲区中获取用户信息、时间戳、有效性和用户验证错误
        userLegacy, timestamp, valid, userValidationError := s.userValidator.Get(buffer.Bytes())
        # 如果用户信息无效或者用户验证错误不为空，则返回空值和错误信息
        if !valid || userValidationError != nil {
            return nil, drainConnection(newError("invalid user").Base(userValidationError))
        }
        # 将用户信息设置为 userLegacy
        user = userLegacy
        # 使用时间戳生成初始化向量
        iv := hashTimestamp(md5.New(), timestamp)
        # 获取 vmessAccount 对象
        vmessAccount = userLegacy.Account.(*vmess.MemoryAccount)

        # 创建 AES 解密流
        aesStream := crypto.NewAesDecryptionStream(vmessAccount.ID.CmdKey(), iv[:])
        # 创建解密器
        decryptor = crypto.NewCryptionReader(aesStream, reader)
    } else {
        # 如果不满足上述条件，则返回错误信息
        return nil, drainConnection(newError("invalid user").Base(errorAEAD))
    }

    # 减去已读取的数据大小
    readSizeRemain -= int(buffer.Len())
    # 清空缓冲区
    buffer.Clear()
    # 从解密器中读取指定大小的数据
    if _, err := buffer.ReadFullFrom(decryptor, 38); err != nil {
        return nil, newError("failed to read request header").Base(err)
    }

    # 创建请求头对象
    request := &protocol.RequestHeader{
        User:    user,
        Version: buffer.Byte(0),
    }

    # 复制缓冲区中指定范围的数据到请求体 IV（16 字节）
    copy(s.requestBodyIV[:], buffer.BytesRange(1, 17))   // 16 bytes
    # 复制缓冲区中指定范围的数据到请求体密钥（16 字节）
    copy(s.requestBodyKey[:], buffer.BytesRange(17, 33)) // 16 bytes
    # 创建会话 ID 对象
    var sid sessionId
    # 复制用户 ID 到会话 ID 对象
    copy(sid.user[:], vmessAccount.ID.Bytes())
    # 设置会话 ID 对象的密钥和随机数
    sid.key = s.requestBodyKey
    sid.nonce = s.requestBodyIV
    # 如果会话历史中不存在该会话 ID，则执行以下操作
    if !s.sessionHistory.addIfNotExits(sid) {
        # 如果不是 AEAD 请求，则执行以下操作
        if !s.isAEADRequest {
            # 尝试烧毁用户验证器的保险丝
            drainErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
            # 如果烧毁失败，则返回错误信息
            if drainErr != nil {
                return nil, drainConnection(newError("duplicated session id, possibly under replay attack, and failed to taint userHash").Base(drainErr))
            }
            # 返回错误信息
            return nil, drainConnection(newError("duplicated session id, possibly under replay attack, userHash tainted"))
        } else {
            # 返回错误信息
            return nil, newError("duplicated session id, possibly under replay attack, but this is a AEAD request")
        }

    }

    # 设置响应头的值为缓冲区中指定位置的字节（1 字节）
    s.responseHeader = buffer.Byte(33)             // 1 byte
    // 设置请求的选项，使用位掩码从缓冲区的第34个字节中读取1个字节
    request.Option = bitmask.Byte(buffer.Byte(34)) // 1 byte
    // 从缓冲区的第35个字节中读取1个字节，右移4位得到填充长度
    padingLen := int(buffer.Byte(35) >> 4)
    // 从缓冲区的第35个字节中读取1个字节，使用位运算获取安全类型
    request.Security = parseSecurityType(buffer.Byte(35) & 0x0F)
    // 1个字节保留
    request.Command = protocol.RequestCommand(buffer.Byte(37))

    // 根据请求命令类型进行不同的处理
    switch request.Command {
    case protocol.RequestCommandMux:
        // 如果是多路复用命令，设置地址为v1.mux.cool，端口为0
        request.Address = net.DomainAddress("v1.mux.cool")
        request.Port = 0
    case protocol.RequestCommandTCP, protocol.RequestCommandUDP:
        // 如果是TCP或UDP命令，使用地址解析器读取地址和端口
        if addr, port, err := addrParser.ReadAddressPort(buffer, decryptor); err == nil {
            request.Address = addr
            request.Port = port
        }
    }

    // 如果填充长度大于0，从解密器中读取指定长度的填充数据
    if padingLen > 0 {
        if _, err := buffer.ReadFullFrom(decryptor, int32(padingLen)); err != nil {
            // 如果不是AEAD请求，进行用户验证器的处理
            if !s.isAEADRequest {
                burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
                if burnErr != nil {
                    return nil, newError("failed to read padding, failed to taint userHash").Base(burnErr).Base(err)
                }
                return nil, newError("failed to read padding, userHash tainted").Base(err)
            }
            return nil, newError("failed to read padding").Base(err)
        }
    }

    // 从解密器中读取4个字节的数据，用于校验和
    if _, err := buffer.ReadFullFrom(decryptor, 4); err != nil {
        // 如果不是AEAD请求，进行用户验证器的处理
        if !s.isAEADRequest {
            burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
            if burnErr != nil {
                return nil, newError("failed to read checksum, failed to taint userHash").Base(burnErr).Base(err)
            }
            return nil, newError("failed to read checksum, userHash tainted").Base(err)
        }
        return nil, newError("failed to read checksum").Base(err)
    }

    // 计算数据的哈希值，并与预期的哈希值进行比较
    fnv1a := fnv.New32a()
    common.Must2(fnv1a.Write(buffer.BytesTo(-4)))
    actualHash := fnv1a.Sum32()
    expectedHash := binary.BigEndian.Uint32(buffer.BytesFrom(-4))
    # 如果实际哈希值不等于期望哈希值
    if actualHash != expectedHash {
        # 如果不是 AEAD 请求
        if !s.isAEADRequest {
            # 创建一个认证错误
            Autherr := newError("invalid auth, legacy userHash tainted")
            # 尝试清除受损的用户哈希值
            burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
            # 如果清除失败
            if burnErr != nil {
                # 更新认证错误信息
                Autherr = newError("invalid auth, can't taint legacy userHash").Base(burnErr)
            }
            # 返回空值和认证错误，可能是受到 https://github.com/v2ray/v2ray-core/issues/2523 中描述的攻击
            return nil, drainConnection(Autherr)
        } else {
            # 返回空值和新的认证错误，表示无效的认证，但这是一个 AEAD 请求
            return nil, newError("invalid auth, but this is a AEAD request")
        }

    }

    # 如果请求的地址为空
    if request.Address == nil {
        # 返回空值和新的错误，表示无效的远程地址
        return nil, newError("invalid remote address")
    }

    # 如果请求的安全类型是未知或自动
    if request.Security == protocol.SecurityType_UNKNOWN || request.Security == protocol.SecurityType_AUTO {
        # 返回空值和新的错误，表示未知的安全类型
        return nil, newError("unknown security type: ", request.Security)
    }

    # 返回请求和空值，表示请求有效
    return request, nil
// DecodeRequestBody函数返回一个Reader，调用者可以从中获取解密后的请求体
func (s *ServerSession) DecodeRequestBody(request *protocol.RequestHeader, reader io.Reader) buf.Reader {
    // 创建一个crypto.ChunkSizeDecoder类型的变量sizeParser，并初始化为crypto.PlainChunkSizeParser类型
    var sizeParser crypto.ChunkSizeDecoder = crypto.PlainChunkSizeParser{}
    // 如果请求头中包含protocol.RequestOptionChunkMasking选项，则将sizeParser初始化为NewShakeSizeParser类型
    if request.Option.Has(protocol.RequestOptionChunkMasking) {
        sizeParser = NewShakeSizeParser(s.requestBodyIV[:])
    }
    // 创建一个crypto.PaddingLengthGenerator类型的变量padding
    var padding crypto.PaddingLengthGenerator
    // 如果请求头中包含protocol.RequestOptionGlobalPadding选项，则将padding初始化为sizeParser的crypto.PaddingLengthGenerator类型
    if request.Option.Has(protocol.RequestOptionGlobalPadding) {
        padding = sizeParser.(crypto.PaddingLengthGenerator)
    }

    // 根据请求头中的Security类型进行不同的处理
    switch request.Security {
    case protocol.SecurityType_NONE:
        // 如果请求头中包含protocol.RequestOptionChunkStream选项
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            // 如果请求头中的命令传输类型为protocol.TransferTypeStream，则返回一个crypto.NewChunkStreamReader类型的Reader
            if request.Command.TransferType() == protocol.TransferTypeStream {
                return crypto.NewChunkStreamReader(sizeParser, reader)
            }
            // 创建一个AEADAuthenticator类型的auth，并初始化其属性
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(NoOpAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            // 返回一个crypto.NewAuthenticationReader类型的Reader
            return crypto.NewAuthenticationReader(auth, sizeParser, reader, protocol.TransferTypePacket, padding)
        }
        // 返回一个buf.NewReader类型的Reader
        return buf.NewReader(reader)
    // 如果安全类型是传统的加密方式
    case protocol.SecurityType_LEGACY:
        // 创建一个新的AES解密流
        aesStream := crypto.NewAesDecryptionStream(s.requestBodyKey[:], s.requestBodyIV[:])
        // 创建一个新的加密读取器
        cryptionReader := crypto.NewCryptionReader(aesStream, reader)
        // 如果请求选项包含分块流
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            // 创建一个新的AEAD认证器
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(FnvAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            // 返回一个新的认证读取器
            return crypto.NewAuthenticationReader(auth, sizeParser, cryptionReader, request.Command.TransferType(), padding)
        }
        // 返回一个新的缓冲读取器
        return buf.NewReader(cryptionReader)
    // 如果安全类型是AES128_GCM
    case protocol.SecurityType_AES128_GCM:
        // 创建一个新的AES GCM加密器
        aead := crypto.NewAesGcm(s.requestBodyKey[:])
        // 创建一个新的AEAD认证器
        auth := &crypto.AEADAuthenticator{
            AEAD:                    aead,
            NonceGenerator:          GenerateChunkNonce(s.requestBodyIV[:], uint32(aead.NonceSize())),
            AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
        }
        // 返回一个新的认证读取器
        return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
    // 如果安全类型是CHACHA20_POLY1305
    case protocol.SecurityType_CHACHA20_POLY1305:
        // 创建一个新的CHACHA20_POLY1305加密器
        aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(s.requestBodyKey[:]))
        // 创建一个新的AEAD认证器
        auth := &crypto.AEADAuthenticator{
            AEAD:                    aead,
            NonceGenerator:          GenerateChunkNonce(s.requestBodyIV[:], uint32(aead.NonceSize())),
            AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
        }
        // 返回一个新的认证读取器
        return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
    // 如果安全类型未知
    default:
        // 抛出异常
        panic("Unknown security type.")
    }
// EncodeResponseHeader 将编码后的响应头写入给定的写入器中
func (s *ServerSession) EncodeResponseHeader(header *protocol.ResponseHeader, writer io.Writer) {
    var encryptionWriter io.Writer
    // 如果不是 AEAD 请求，则对请求体密钥和 IV 进行 MD5 加密
    if !s.isAEADRequest {
        s.responseBodyKey = md5.Sum(s.requestBodyKey[:])
        s.responseBodyIV = md5.Sum(s.requestBodyIV[:])
    } else {
        // 如果是 AEAD 请求，则对请求体密钥和 IV 进行 SHA256 加密，并截取前16个字节
        BodyKey := sha256.Sum256(s.requestBodyKey[:])
        copy(s.responseBodyKey[:], BodyKey[:16])
        BodyIV := sha256.Sum256(s.requestBodyIV[:])
        copy(s.responseBodyIV[:], BodyIV[:16])
    }

    // 创建 AES 加密流
    aesStream := crypto.NewAesEncryptionStream(s.responseBodyKey[:], s.responseBodyIV[:])
    // 创建加密写入器
    encryptionWriter = crypto.NewCryptionWriter(aesStream, writer)
    s.responseWriter = encryptionWriter

    // 创建用于存储 AEAD 加密后的头部的缓冲区
    aeadEncryptedHeaderBuffer := bytes.NewBuffer(nil)

    // 如果是 AEAD 请求，则将加密写入器指向 AEAD 加密后的头部缓冲区
    if s.isAEADRequest {
        encryptionWriter = aeadEncryptedHeaderBuffer
    }

    // 将响应头和选项写入加密写入器
    common.Must2(encryptionWriter.Write([]byte{s.responseHeader, byte(header.Option)}))
    // 将命令编组后写入加密写入器
    err := MarshalCommand(header.Command, encryptionWriter)
    if err != nil {
        // 如果出现错误，则写入空字节
        common.Must2(encryptionWriter.Write([]byte{0x00, 0x00}))
    }
}
    # 如果请求是 AEAD 请求
    if s.isAEADRequest {

        # 使用密钥派生函数生成 AEAD 响应头长度加密密钥
        aeadResponseHeaderLengthEncryptionKey := vmessaead.KDF16(s.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderLenKey)
        # 使用密钥派生函数生成 AEAD 响应头长度加密 IV
        aeadResponseHeaderLengthEncryptionIV := vmessaead.KDF(s.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderLenIV)[:12]

        # 使用 AES 加密算法创建 AEAD 响应头长度加密密钥的块
        aeadResponseHeaderLengthEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderLengthEncryptionKey)).(cipher.Block)
        # 使用 GCM 模式创建 AEAD 响应头长度加密算法
        aeadResponseHeaderLengthEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderLengthEncryptionKeyAESBlock)).(cipher.AEAD)

        # 创建一个空的字节缓冲区
        aeadResponseHeaderLengthEncryptionBuffer := bytes.NewBuffer(nil)

        # 解密响应头长度的二进制数据
        decryptedResponseHeaderLengthBinaryDeserializeBuffer := uint16(aeadEncryptedHeaderBuffer.Len())

        # 将解密后的响应头长度写入到 AEAD 响应头长度加密缓冲区
        common.Must(binary.Write(aeadResponseHeaderLengthEncryptionBuffer, binary.BigEndian, decryptedResponseHeaderLengthBinaryDeserializeBuffer))

        # 使用 AEAD 响应头长度加密算法对数据进行加密，并将结果写入到 writer
        AEADEncryptedLength := aeadResponseHeaderLengthEncryptionAEAD.Seal(nil, aeadResponseHeaderLengthEncryptionIV, aeadResponseHeaderLengthEncryptionBuffer.Bytes(), nil)
        common.Must2(io.Copy(writer, bytes.NewReader(AEADEncryptedLength)))

        # 使用密钥派生函数生成 AEAD 响应头载荷加密密钥
        aeadResponseHeaderPayloadEncryptionKey := vmessaead.KDF16(s.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadKey)
        # 使用密钥派生函数生成 AEAD 响应头载荷加密 IV
        aeadResponseHeaderPayloadEncryptionIV := vmessaead.KDF(s.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadIV)[:12]

        # 使用 AES 加密算法创建 AEAD 响应头载荷加密密钥的块
        aeadResponseHeaderPayloadEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderPayloadEncryptionKey)).(cipher.Block)
        # 使用 GCM 模式创建 AEAD 响应头载荷加密算法
        aeadResponseHeaderPayloadEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderPayloadEncryptionKeyAESBlock)).(cipher.AEAD)

        # 使用 AEAD 响应头载荷加密算法对数据进行加密，并将结果写入到 writer
        aeadEncryptedHeaderPayload := aeadResponseHeaderPayloadEncryptionAEAD.Seal(nil, aeadResponseHeaderPayloadEncryptionIV, aeadEncryptedHeaderBuffer.Bytes(), nil)
        common.Must2(io.Copy(writer, bytes.NewReader(aeadEncryptedHeaderPayload)))
    }
// EncodeResponseBody 返回一个 Writer，用于自动加密调用者写入的内容。
func (s *ServerSession) EncodeResponseBody(request *protocol.RequestHeader, writer io.Writer) buf.Writer {
    // 创建一个大小解析器，初始值为 PlainChunkSizeParser
    var sizeParser crypto.ChunkSizeEncoder = crypto.PlainChunkSizeParser{}
    // 如果请求选项中包含 ChunkMasking，则将 sizeParser 更新为 NewShakeSizeParser
    if request.Option.Has(protocol.RequestOptionChunkMasking) {
        sizeParser = NewShakeSizeParser(s.responseBodyIV[:])
    }
    // 创建一个填充长度生成器，初始值为 nil
    var padding crypto.PaddingLengthGenerator
    // 如果请求选项中包含 GlobalPadding，则将 padding 更新为 sizeParser.(crypto.PaddingLengthGenerator)
    if request.Option.Has(protocol.RequestOptionGlobalPadding) {
        padding = sizeParser.(crypto.PaddingLengthGenerator)
    }

    // 根据请求的安全类型进行不同的处理
    switch request.Security {
    case protocol.SecurityType_NONE:
        // 如果请求选项中包含 ChunkStream，并且命令的传输类型为 Stream，则返回一个新的 ChunkStreamWriter
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            if request.Command.TransferType() == protocol.TransferTypeStream {
                return crypto.NewChunkStreamWriter(sizeParser, writer)
            }
            // 创建一个 AEADAuthenticator，并返回一个新的 AuthenticationWriter
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(NoOpAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            return crypto.NewAuthenticationWriter(auth, sizeParser, writer, protocol.TransferTypePacket, padding)
        }
        // 返回一个新的 Writer
        return buf.NewWriter(writer)
    case protocol.SecurityType_LEGACY:
        // 如果请求选项中包含 ChunkStream，则创建一个 AEADAuthenticator，并返回一个新的 AuthenticationWriter
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            auth := &crypto.AEADAuthenticator{
                AEAD:                    new(FnvAuthenticator),
                NonceGenerator:          crypto.GenerateEmptyBytes(),
                AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
            }
            return crypto.NewAuthenticationWriter(auth, sizeParser, s.responseWriter, request.Command.TransferType(), padding)
        }
        // 返回一个新的 SequentialWriter
        return &buf.SequentialWriter{Writer: s.responseWriter}
    # 如果安全类型是AES128_GCM，则使用AES-GCM算法创建AEAD对象
    aead := crypto.NewAesGcm(s.responseBodyKey[:])

    # 创建AEAD认证器，包括AEAD对象、Nonce生成器和附加数据生成器
    auth := &crypto.AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateChunkNonce(s.responseBodyIV[:], uint32(aead.NonceSize())),
        AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
    }
    # 返回一个新的身份验证写入器，用于对数据进行身份验证和加密
    return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)

    # 如果安全类型是CHACHA20_POLY1305，则使用CHACHA20_POLY1305算法创建AEAD对象
    aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(s.responseBodyKey[:]))

    # 创建AEAD认证器，包括AEAD对象、Nonce生成器和附加数据生成器
    auth := &crypto.AEADAuthenticator{
        AEAD:                    aead,
        NonceGenerator:          GenerateChunkNonce(s.responseBodyIV[:], uint32(aead.NonceSize())),
        AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
    }
    # 返回一个新的身份验证写入器，用于对数据进行身份验证和加密
    return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)

    # 如果安全类型不是AES128_GCM或CHACHA20_POLY1305，则抛出异常
    default:
        panic("Unknown security type.")
    }
# 定义一个方法，用于从给定的 reader 中读取指定长度的数据并丢弃，避免占用内存
func (s *ServerSession) DrainConnN(reader io.Reader, n int) error:
    # 使用 io.CopyN 方法从 reader 中读取指定长度的数据并丢弃，返回实际读取的字节数和可能出现的错误
    _, err := io.CopyN(ioutil.Discard, reader, int64(n))
    # 返回可能出现的错误
    return err
```