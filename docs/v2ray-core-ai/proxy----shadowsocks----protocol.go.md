# `v2ray-core\proxy\shadowsocks\protocol.go`

```
// +build !confonly

package shadowsocks

import (
    "crypto/hmac"  // 导入加密哈希消息认证码包
    "crypto/rand"  // 导入加密随机数包
    "crypto/sha256"  // 导入SHA-256哈希算法包
    "hash"  // 导入哈希包
    "hash/crc32"  // 导入CRC32校验包
    "io"  // 导入输入输出包
    "io/ioutil"  // 导入I/O实用工具包
    "v2ray.com/core/common/dice"  // 导入v2ray的公共工具包

    "v2ray.com/core/common"  // 导入v2ray的公共包
    "v2ray.com/core/common/buf"  // 导入v2ray的缓冲区包
    "v2ray.com/core/common/net"  // 导入v2ray的网络包
    "v2ray.com/core/common/protocol"  // 导入v2ray的协议包
)

const (
    Version = 1  // 定义版本号为1
)

var addrParser = protocol.NewAddressParser(  // 创建地址解析器
    protocol.AddressFamilyByte(0x01, net.AddressFamilyIPv4),  // IPv4地址族
    protocol.AddressFamilyByte(0x04, net.AddressFamilyIPv6),  // IPv6地址族
    protocol.AddressFamilyByte(0x03, net.AddressFamilyDomain),  // 域名地址族
    protocol.WithAddressTypeParser(func(b byte) byte {  // 使用地址类型解析器
        return b & 0x0F  // 返回地址类型
    }),
)

// ReadTCPSession reads a Shadowsocks TCP session from the given reader, returns its header and remaining parts.
func ReadTCPSession(user *protocol.MemoryUser, reader io.Reader) (*protocol.RequestHeader, buf.Reader, error) {
    account := user.Account.(*MemoryAccount)  // 获取用户账户信息

    hashkdf := hmac.New(func() hash.Hash { return sha256.New() }, []byte("SSBSKDF"))  // 创建HMAC哈希函数
    hashkdf.Write(account.Key)  // 写入账户密钥

    behaviorSeed := crc32.ChecksumIEEE(hashkdf.Sum(nil))  // 计算行为种子

    behaviorRand := dice.NewDeterministicDice(int64(behaviorSeed))  // 创建确定性骰子
    BaseDrainSize := behaviorRand.Roll(3266)  // 基础排水大小
    RandDrainMax := behaviorRand.Roll(64) + 1  // 随机排水最大值
    RandDrainRolled := dice.Roll(RandDrainMax)  // 随机排水值
    DrainSize := BaseDrainSize + 16 + 38 + RandDrainRolled  // 排水大小
    readSizeRemain := DrainSize  // 剩余读取大小

    buffer := buf.New()  // 创建缓冲区
    defer buffer.Release()  // 延迟释放缓冲区

    ivLen := account.Cipher.IVSize()  // 获取IV长度
    var iv []byte  // 定义IV字节切片
    if ivLen > 0 {  // 如果IV长度大于0
        if _, err := buffer.ReadFullFrom(reader, ivLen); err != nil {  // 从读取器中读取IV
            readSizeRemain -= int(buffer.Len())  // 更新剩余读取大小
            DrainConnN(reader, readSizeRemain)  // 排水连接
            return nil, nil, newError("failed to read IV").Base(err)  // 返回错误
        }

        iv = append([]byte(nil), buffer.BytesTo(ivLen)...)  // 将IV添加到字节切片中
    }

    r, err := account.Cipher.NewDecryptionReader(account.Key, iv, reader)  // 创建解密读取器
    // 如果发生错误，减去已读取的数据大小，然后清空剩余数据并返回错误
    if err != nil {
        readSizeRemain -= int(buffer.Len())
        DrainConnN(reader, readSizeRemain)
        return nil, nil, newError("failed to initialize decoding stream").Base(err).AtError()
    }
    // 创建一个带有缓冲的读取器
    br := &buf.BufferedReader{Reader: r}

    // 创建一个请求头对象，并设置版本号、用户、命令
    request := &protocol.RequestHeader{
        Version: Version,
        User:    user,
        Command: protocol.RequestCommandTCP,
    }

    // 减去已读取的数据大小，然后清空缓冲区
    readSizeRemain -= int(buffer.Len())
    buffer.Clear()

    // 从缓冲区和读取器中解析出地址和端口
    addr, port, err := addrParser.ReadAddressPort(buffer, br)
    // 如果解析出错，减去已读取的数据大小，然后清空剩余数据并返回错误
    if err != nil {
        readSizeRemain -= int(buffer.Len())
        DrainConnN(reader, readSizeRemain)
        return nil, nil, newError("failed to read address").Base(err)
    }

    // 将解析出的地址和端口设置到请求头对象中
    request.Address = addr
    request.Port = port

    // 如果请求头中的地址为空，减去已读取的数据大小，然后清空剩余数据并返回错误
    if request.Address == nil {
        readSizeRemain -= int(buffer.Len())
        DrainConnN(reader, readSizeRemain)
        return nil, nil, newError("invalid remote address.")
    }

    // 返回请求头对象和读取器，以及空的错误
    return request, br, nil
// DrainConnN 从给定的 reader 中读取 n 个字节并丢弃，返回可能出现的错误
func DrainConnN(reader io.Reader, n int) error {
    // 使用 io.CopyN 函数从 reader 中读取 n 个字节并丢弃，返回读取时可能出现的错误
    _, err := io.CopyN(ioutil.Discard, reader, int64(n))
    return err
}

// WriteTCPRequest 将 Shadowsocks 请求写入给定的 writer，并返回一个用于写入 body 的 writer
func WriteTCPRequest(request *protocol.RequestHeader, writer io.Writer) (buf.Writer, error) {
    user := request.User
    account := user.Account.(*MemoryAccount)

    var iv []byte
    // 如果加密算法需要 IV，则创建一个随机的 IV
    if account.Cipher.IVSize() > 0 {
        iv = make([]byte, account.Cipher.IVSize())
        common.Must2(rand.Read(iv))
        // 将 IV 写入 writer，如果出现错误则返回相应的错误
        if err := buf.WriteAllBytes(writer, iv); err != nil {
            return nil, newError("failed to write IV")
        }
    }

    // 使用账户的加密算法创建一个新的加密写入流
    w, err := account.Cipher.NewEncryptionWriter(account.Key, iv, writer)
    if err != nil {
        return nil, newError("failed to create encoding stream").Base(err).AtError()
    }

    // 创建一个新的缓冲区用于存储请求头部
    header := buf.New()

    // 将请求的地址和端口写入请求头部，如果出现错误则返回相应的错误
    if err := addrParser.WriteAddressPort(header, request.Address, request.Port); err != nil {
        return nil, newError("failed to write address").Base(err)
    }

    // 将请求头部写入加密写入流，如果出现错误则返回相应的错误
    if err := w.WriteMultiBuffer(buf.MultiBuffer{header}); err != nil {
        return nil, newError("failed to write header").Base(err)
    }

    return w, nil
}

// ReadTCPResponse 从给定的 reader 中读取 Shadowsocks 响应，返回一个读取器和可能出现的错误
func ReadTCPResponse(user *protocol.MemoryUser, reader io.Reader) (buf.Reader, error) {
    account := user.Account.(*MemoryAccount)

    var iv []byte
    // 如果加密算法需要 IV，则从 reader 中读取 IV
    if account.Cipher.IVSize() > 0 {
        iv = make([]byte, account.Cipher.IVSize())
        if _, err := io.ReadFull(reader, iv); err != nil {
            return nil, newError("failed to read IV").Base(err)
        }
    }

    // 使用账户的解密算法创建一个新的解密读取器
    return account.Cipher.NewDecryptionReader(account.Key, iv, reader)
}

// WriteTCPResponse 将 Shadowsocks 响应写入给定的 writer，返回一个用于写入的 writer 和可能出现的错误
func WriteTCPResponse(request *protocol.RequestHeader, writer io.Writer) (buf.Writer, error) {
    user := request.User
    account := user.Account.(*MemoryAccount)

    var iv []byte
    # 如果账户的加密算法需要初始化向量（IV）
    if account.Cipher.IVSize() > 0:
        # 创建一个与 IV 大小相等的字节切片
        iv = make([]byte, account.Cipher.IVSize())
        # 从系统随机源中读取随机字节填充 IV
        common.Must2(rand.Read(iv))
        # 将 IV 写入到 writer 中
        if err := buf.WriteAllBytes(writer, iv); err != nil:
            # 如果写入 IV 失败，则返回错误
            return nil, newError("failed to write IV.").Base(err)
    
    # 返回一个新的加密写入器，使用账户的密钥和 IV，写入到指定的 writer 中
    return account.Cipher.NewEncryptionWriter(account.Key, iv, writer)
}
// EncodeUDPPacket 函数用于对 UDP 数据包进行编码
func EncodeUDPPacket(request *protocol.RequestHeader, payload []byte) (*buf.Buffer, error) {
    // 获取用户信息
    user := request.User
    // 获取用户账户信息
    account := user.Account.(*MemoryAccount)

    // 创建一个新的缓冲区
    buffer := buf.New()
    // 获取初始化向量的长度
    ivLen := account.Cipher.IVSize()
    // 如果初始化向量的长度大于 0，则从随机读取器中读取相应长度的数据
    if ivLen > 0 {
        common.Must2(buffer.ReadFullFrom(rand.Reader, ivLen))
    }

    // 将地址和端口写入缓冲区
    if err := addrParser.WriteAddressPort(buffer, request.Address, request.Port); err != nil {
        return nil, newError("failed to write address").Base(err)
    }

    // 将有效载荷写入缓冲区
    buffer.Write(payload)

    // 使用账户的密钥对缓冲区中的数据进行加密
    if err := account.Cipher.EncodePacket(account.Key, buffer); err != nil {
        return nil, newError("failed to encrypt UDP payload").Base(err)
    }

    // 返回加密后的缓冲区
    return buffer, nil
}

// DecodeUDPPacket 函数用于对 UDP 数据包进行解码
func DecodeUDPPacket(user *protocol.MemoryUser, payload *buf.Buffer) (*protocol.RequestHeader, *buf.Buffer, error) {
    // 获取用户账户信息
    account := user.Account.(*MemoryAccount)

    var iv []byte
    // 如果不是 AEAD 加密方式，并且初始化向量的长度大于 0，则从 payload 中读取初始化向量
    if !account.Cipher.IsAEAD() && account.Cipher.IVSize() > 0 {
        // 保存初始化向量，因为它将在 DecodePacket 中从 payload 中移除
        iv = make([]byte, account.Cipher.IVSize())
        copy(iv, payload.BytesTo(account.Cipher.IVSize()))
    }

    // 使用账户的密钥对 payload 中的数据进行解密
    if err := account.Cipher.DecodePacket(account.Key, payload); err != nil {
        return nil, nil, newError("failed to decrypt UDP payload").Base(err)
    }

    // 创建一个新的请求头
    request := &protocol.RequestHeader{
        Version: Version,
        User:    user,
        Command: protocol.RequestCommandUDP,
    }

    // 将 payload 的第一个字节的低 4 位清零
    payload.SetByte(0, payload.Byte(0)&0x0F)

    // 从 payload 中读取地址和端口
    addr, port, err := addrParser.ReadAddressPort(nil, payload)
    if err != nil {
        return nil, nil, newError("failed to parse address").Base(err)
    }

    // 设置请求头的地址和端口
    request.Address = addr
    request.Port = port

    // 返回请求头、payload 和 nil
    return request, payload, nil
}

// UDPReader 结构用于读取 UDP 数据
type UDPReader struct {
    Reader io.Reader
    User   *protocol.MemoryUser
}

// ReadMultiBuffer 方法用于读取多个缓冲区
func (v *UDPReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 创建一个新的缓冲区
    buffer := buf.New()
    // 从读取器中读取数据到缓冲区
    _, err := buffer.ReadFrom(v.Reader)
    # 如果错误不为空，则释放缓冲区并返回空和错误
    if err != nil:
        buffer.Release()
        return nil, err
    # 解码 UDP 数据包，获取用户和数据，如果出现错误则释放缓冲区并返回空和错误
    _, payload, err := DecodeUDPPacket(v.User, buffer)
    if err != nil:
        buffer.Release()
        return nil, err
    # 返回包含数据的多缓冲区和空错误
    return buf.MultiBuffer{payload}, nil
// UDPWriter 结构体定义，包含一个 io.Writer 接口和一个 protocol.RequestHeader 指针
type UDPWriter struct {
    Writer  io.Writer
    Request *protocol.RequestHeader
}

// Write 方法实现了 io.Writer 接口
func (w *UDPWriter) Write(payload []byte) (int, error) {
    // 使用请求头和有效载荷编码 UDP 数据包
    packet, err := EncodeUDPPacket(w.Request, payload)
    if err != nil {
        return 0, err
    }
    // 将 UDP 数据包写入 UDPWriter 的 Writer 接口
    _, err = w.Writer.Write(packet.Bytes())
    // 释放 UDP 数据包的资源
    packet.Release()
    // 返回有效载荷的长度和可能的错误
    return len(payload), err
}
```