# `v2ray-core\proxy\vmess\aead\authid.go`

```
package aead

import (
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "crypto/aes"  // 导入 aes 包，用于实现高级加密标准
    "crypto/cipher"  // 导入 cipher 包，用于实现加密和解密
    rand3 "crypto/rand"  // 导入 crypto/rand 包，并重命名为 rand3，用于生成随机数
    "encoding/binary"  // 导入 binary 包，用于进行二进制数据的读写
    "errors"  // 导入 errors 包，用于处理错误
    "hash/crc32"  // 导入 crc32 包，用于计算 CRC-32 校验和
    "io"  // 导入 io 包，用于进行 I/O 操作
    "math"  // 导入 math 包，用于数学计算
    "time"  // 导入 time 包，用于处理时间
    "v2ray.com/core/common"  // 导入 common 包，用于处理通用功能
    antiReplayWindow "v2ray.com/core/common/antireplay"  // 导入 v2ray.com/core/common/antireplay 包，并重命名为 antiReplayWindow，用于处理防重放攻击
)

func CreateAuthID(cmdKey []byte, time int64) [16]byte {
    buf := bytes.NewBuffer(nil)  // 创建一个新的字节缓冲区
    common.Must(binary.Write(buf, binary.BigEndian, time))  // 将时间以大端序写入字节缓冲区
    var zero uint32  // 声明一个无符号 32 位整数变量 zero
    common.Must2(io.CopyN(buf, rand3.Reader, 4))  // 从随机数生成器中读取 4 个字节写入字节缓冲区
    zero = crc32.ChecksumIEEE(buf.Bytes())  // 计算字节缓冲区的 CRC-32 校验和
    common.Must(binary.Write(buf, binary.BigEndian, zero))  // 将 zero 以大端序写入字节缓冲区
    aesBlock := NewCipherFromKey(cmdKey)  // 使用 cmdKey 创建一个新的加密块
    if buf.Len() != 16 {  // 如果字节缓冲区长度不等于 16
        panic("Size unexpected")  // 抛出异常
    }
    var result [16]byte  // 声明一个长度为 16 的字节数组 result
    aesBlock.Encrypt(result[:], buf.Bytes())  // 使用加密块对字节缓冲区的内容进行加密，并将结果写入 result
    return result  // 返回加密后的结果
}

func NewCipherFromKey(cmdKey []byte) cipher.Block {
    aesBlock, err := aes.NewCipher(KDF16(cmdKey, KDFSaltConst_AuthIDEncryptionKey))  // 使用 KDF16 函数和 KDFSaltConst_AuthIDEncryptionKey 创建一个新的 AES 加密块
    if err != nil {  // 如果发生错误
        panic(err)  // 抛出异常
    }
    return aesBlock  // 返回 AES 加密块
}

type AuthIDDecoder struct {
    s cipher.Block  // 声明一个 cipher.Block 类型的字段 s
}

func NewAuthIDDecoder(cmdKey []byte) *AuthIDDecoder {
    return &AuthIDDecoder{NewCipherFromKey(cmdKey)}  // 返回一个使用 cmdKey 创建的 AuthIDDecoder 实例
}

func (aidd *AuthIDDecoder) Decode(data [16]byte) (int64, uint32, int32, []byte) {
    aidd.s.Decrypt(data[:], data[:])  // 使用字段 s 对数据进行解密
    var t int64  // 声明一个 int64 类型的变量 t
    var zero uint32  // 声明一个 uint32 类型的变量 zero
    var rand int32  // 声明一个 int32 类型的变量 rand
    reader := bytes.NewReader(data[:])  // 创建一个从 data 切片读取数据的 Reader
    common.Must(binary.Read(reader, binary.BigEndian, &t))  // 从 Reader 中以大端序读取数据到变量 t
    common.Must(binary.Read(reader, binary.BigEndian, &rand))  // 从 Reader 中以大端序读取数据到变量 rand
    common.Must(binary.Read(reader, binary.BigEndian, &zero))  // 从 Reader 中以大端序读取数据到变量 zero
    return t, zero, rand, data[:]  // 返回 t、zero、rand 和 data 切片
}

func NewAuthIDDecoderHolder() *AuthIDDecoderHolder {
    return &AuthIDDecoderHolder{make(map[string]*AuthIDDecoderItem), antiReplayWindow.NewAntiReplayWindow(120)}  // 返回一个新的 AuthIDDecoderHolder 实例，包含一个空的 map 和一个新的 AntiReplayWindow
}

type AuthIDDecoderHolder struct {
    aidhi map[string]*AuthIDDecoderItem  // 声明一个 map 类型的字段 aidhi，键为字符串，值为 AuthIDDecoderItem 指针
    apw   *antiReplayWindow.AntiReplayWindow  // 声明一个 *antiReplayWindow.AntiReplayWindow 类型的字段 apw
}

type AuthIDDecoderItem struct {
    dec    *AuthIDDecoder  // 声明一个 *AuthIDDecoder 类型的字段 dec
    ticket interface{}  // 声明一个 interface{} 类型的字段 ticket
}
# 创建一个新的 AuthIDDecoderItem 对象，使用给定的密钥和票据
func NewAuthIDDecoderItem(key [16]byte, ticket interface{}) *AuthIDDecoderItem {
    return &AuthIDDecoderItem{
        dec:    NewAuthIDDecoder(key[:]),  # 使用密钥创建一个新的 AuthIDDecoder 对象
        ticket: ticket,  # 设置票据
    }
}

# 向 AuthIDDecoderHolder 中添加用户
func (a *AuthIDDecoderHolder) AddUser(key [16]byte, ticket interface{}) {
    a.aidhi[string(key[:])] = NewAuthIDDecoderItem(key, ticket)  # 使用密钥和票据创建一个新的 AuthIDDecoderItem 对象，并添加到 aidhi 中
}

# 从 AuthIDDecoderHolder 中移除用户
func (a *AuthIDDecoderHolder) RemoveUser(key [16]byte) {
    delete(a.aidhi, string(key[:]))  # 从 aidhi 中删除指定密钥对应的 AuthIDDecoderItem 对象
}

# 在 AuthIDDecoderHolder 中查找匹配的 AuthID
func (a *AuthIDDecoderHolder) Match(AuthID [16]byte) (interface{}, error) {
    for _, v := range a.aidhi {  # 遍历 aidhi 中的所有 AuthIDDecoderItem 对象

        t, z, r, d := v.dec.Decode(AuthID)  # 使用 AuthIDDecoder 对象解码 AuthID
        if z != crc32.ChecksumIEEE(d[:12]) {  # 检查校验和是否匹配
            continue  # 如果不匹配，则继续下一个循环
        }

        if t < 0:  # 如果 t 小于 0
            continue  # 继续下一个循环

        if math.Abs(math.Abs(float64(t))-float64(time.Now().Unix())) > 120:  # 如果时间差大于 120 秒
            continue  # 继续下一个循环

        if !a.apw.Check(AuthID[:]) {  # 如果 AuthID 不通过检查
            return nil, ErrReplay  # 返回错误信息
        }

        _ = r  # 忽略 r

        return v.ticket, nil  # 返回票据和空错误信息

    }
    return nil, ErrNotFound  # 如果没有找到匹配的 AuthID，则返回错误信息
}

# 定义错误信息：用户不存在
var ErrNotFound = errors.New("user do not exist")

# 定义错误信息：重复请求
var ErrReplay = errors.New("replayed request")
```