# `v2ray-core\proxy\vmess\encoding\auth.go`

```go
package encoding

import (
    "crypto/md5"  // 导入 md5 加密算法
    "encoding/binary"  // 导入二进制编码包
    "hash/fnv"  // 导入 Fnv 哈希算法包

    "v2ray.com/core/common"  // 导入 v2ray 核心通用包

    "golang.org/x/crypto/sha3"  // 导入 sha3 加密算法包
)

// Authenticate authenticates a byte array using Fnv hash.
// 使用 Fnv 哈希算法对字节数组进行认证
func Authenticate(b []byte) uint32 {
    fnv1hash := fnv.New32a()  // 创建一个 Fnv32a 哈希对象
    common.Must2(fnv1hash.Write(b))  // 写入字节数组到哈希对象
    return fnv1hash.Sum32()  // 返回哈希值
}

type NoOpAuthenticator struct{}  // 定义一个空操作认证结构体

func (NoOpAuthenticator) NonceSize() int {  // 实现 NonceSize 方法
    return 0  // 返回 0
}

func (NoOpAuthenticator) Overhead() int {  // 实现 Overhead 方法
    return 0  // 返回 0
}

// Seal implements AEAD.Seal().
// Seal 实现了 AEAD.Seal() 方法
func (NoOpAuthenticator) Seal(dst, nonce, plaintext, additionalData []byte) []byte {
    return append(dst[:0], plaintext...)  // 返回拷贝的明文
}

// Open implements AEAD.Open().
// Open 实现了 AEAD.Open() 方法
func (NoOpAuthenticator) Open(dst, nonce, ciphertext, additionalData []byte) ([]byte, error) {
    return append(dst[:0], ciphertext...), nil  // 返回拷贝的密文和空错误
}

// FnvAuthenticator is an AEAD based on Fnv hash.
// FnvAuthenticator 是基于 Fnv 哈希的 AEAD
type FnvAuthenticator struct {
}

// NonceSize implements AEAD.NonceSize().
// NonceSize 实现了 AEAD.NonceSize() 方法
func (*FnvAuthenticator) NonceSize() int {
    return 0  // 返回 0
}

// Overhead impelements AEAD.Overhead().
// Overhead 实现了 AEAD.Overhead() 方法
func (*FnvAuthenticator) Overhead() int {
    return 4  // 返回 4
}

// Seal implements AEAD.Seal().
// Seal 实现了 AEAD.Seal() 方法
func (*FnvAuthenticator) Seal(dst, nonce, plaintext, additionalData []byte) []byte {
    dst = append(dst, 0, 0, 0, 0)  // 在目标切片末尾添加四个 0
    binary.BigEndian.PutUint32(dst, Authenticate(plaintext))  // 使用大端序将认证值写入目标切片
    return append(dst, plaintext...)  // 返回拷贝的明文
}

// Open implements AEAD.Open().
// Open 实现了 AEAD.Open() 方法
func (*FnvAuthenticator) Open(dst, nonce, ciphertext, additionalData []byte) ([]byte, error) {
    if binary.BigEndian.Uint32(ciphertext[:4]) != Authenticate(ciphertext[4:]) {  // 如果密文前四个字节的认证值不等于后面的明文的认证值
        return dst, newError("invalid authentication")  // 返回目标切片和错误信息
    }
    return append(dst, ciphertext[4:]...), nil  // 返回拷贝的密文和空错误
}

// GenerateChacha20Poly1305Key generates a 32-byte key from a given 16-byte array.
// GenerateChacha20Poly1305Key 从给定的 16 字节数组生成一个 32 字节的密钥
func GenerateChacha20Poly1305Key(b []byte) []byte {
    key := make([]byte, 32)  // 创建一个 32 字节的切片
    t := md5.Sum(b)  // 对给定的字节数组进行 MD5 加密
    copy(key, t[:])  // 将加密结果拷贝到密钥切片的前 16 个字节
    t = md5.Sum(key[:16])  // 对密钥切片的前 16 个字节再进行 MD5 加密
    copy(key[16:], t[:])  // 将再次加密的结果拷贝到密钥切片的后 16 个字节
    return key  // 返回生成的密钥
}

type ShakeSizeParser struct {
}
    # 定义一个变量 shake，用于存储 sha3.ShakeHash 对象
    shake  sha3.ShakeHash
    # 定义一个变量 buffer，用于存储长度为2的字节数组
    buffer [2]byte
// 创建一个新的 ShakeSizeParser 对象，根据给定的随机数初始化 SHA3 Shake128 对象
func NewShakeSizeParser(nonce []byte) *ShakeSizeParser {
    // 创建 SHA3 Shake128 对象
    shake := sha3.NewShake128()
    // 将随机数写入 SHA3 对象
    common.Must2(shake.Write(nonce))
    // 返回初始化后的 ShakeSizeParser 对象指针
    return &ShakeSizeParser{
        shake: shake,
    }
}

// 返回对象的大小，固定为 2
func (*ShakeSizeParser) SizeBytes() int32 {
    return 2
}

// 读取下一个 16 位无符号整数
func (s *ShakeSizeParser) next() uint16 {
    // 从 SHA3 对象中读取数据到缓冲区
    common.Must2(s.shake.Read(s.buffer[:]))
    // 将缓冲区中的数据解析为大端字节序的 16 位无符号整数并返回
    return binary.BigEndian.Uint16(s.buffer[:])
}

// 解码给定的字节切片，返回解码后的结果和错误信息
func (s *ShakeSizeParser) Decode(b []byte) (uint16, error) {
    // 读取下一个掩码值
    mask := s.next()
    // 将字节切片中的数据解析为大端字节序的 16 位无符号整数
    size := binary.BigEndian.Uint16(b)
    // 返回掩码值和解析后的数据异或的结果，以及空的错误信息
    return mask ^ size, nil
}

// 编码给定的 16 位无符号整数，返回编码后的字节切片
func (s *ShakeSizeParser) Encode(size uint16, b []byte) []byte {
    // 读取下一个掩码值
    mask := s.next()
    // 将掩码值和给定的大小进行异或操作，并将结果以大端字节序写入字节切片
    binary.BigEndian.PutUint16(b, mask^size)
    // 返回编码后的字节切片的前两个字节
    return b[:2]
}

// 返回下一个填充长度，取余 64
func (s *ShakeSizeParser) NextPaddingLen() uint16 {
    // 返回下一个掩码值对 64 取余的结果
    return s.next() % 64
}

// 返回最大填充长度，固定为 64
func (s *ShakeSizeParser) MaxPaddingLen() uint16 {
    // 返回固定的最大填充长度 64
    return 64
}
```