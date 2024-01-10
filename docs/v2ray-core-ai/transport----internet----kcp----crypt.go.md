# `v2ray-core\transport\internet\kcp\crypt.go`

```
// +build !confonly  // 标记此文件不仅仅是配置文件

package kcp  // 声明包名为 kcp

import (  // 导入需要的包
    "crypto/cipher"  // 导入加密解密包
    "encoding/binary"  // 导入二进制编码包
    "hash/fnv"  // 导入哈希函数包

    "v2ray.com/core/common"  // 导入通用工具包
)

// SimpleAuthenticator is a legacy AEAD used for KCP encryption.  // SimpleAuthenticator 是用于 KCP 加密的传统 AEAD
type SimpleAuthenticator struct{}  // 定义 SimpleAuthenticator 结构体

// NewSimpleAuthenticator creates a new SimpleAuthenticator  // 创建一个新的 SimpleAuthenticator
func NewSimpleAuthenticator() cipher.AEAD {  // 定义函数 NewSimpleAuthenticator，返回一个 AEAD 加密对象
    return &SimpleAuthenticator{}  // 返回 SimpleAuthenticator 对象的指针
}

// NonceSize implements cipher.AEAD.NonceSize().  // NonceSize 实现了 cipher.AEAD.NonceSize()
func (*SimpleAuthenticator) NonceSize() int {  // 定义 NonceSize 方法，返回一个整数
    return 0  // 返回 0
}

// Overhead implements cipher.AEAD.NonceSize().  // Overhead 实现了 cipher.AEAD.NonceSize()
func (*SimpleAuthenticator) Overhead() int {  // 定义 Overhead 方法，返回一个整数
    return 6  // 返回 6
}

// Seal implements cipher.AEAD.Seal().  // Seal 实现了 cipher.AEAD.Seal()
func (a *SimpleAuthenticator) Seal(dst, nonce, plain, extra []byte) []byte {  // 定义 Seal 方法，对明文进行加密
    dst = append(dst, 0, 0, 0, 0, 0, 0) // 4 bytes for hash, and then 2 bytes for length  // 在 dst 后追加 6 个字节的 0
    binary.BigEndian.PutUint16(dst[4:], uint16(len(plain)))  // 将明文长度编码为 2 字节的大端整数，放入 dst 中
    dst = append(dst, plain...)  // 将明文追加到 dst 中

    fnvHash := fnv.New32a()  // 创建一个新的 32 位哈希对象
    common.Must2(fnvHash.Write(dst[4:]))  // 将 dst 中从第 4 个字节开始的内容写入哈希对象
    fnvHash.Sum(dst[:0])  // 计算哈希值并存入 dst

    dstLen := len(dst)  // 获取 dst 的长度
    xtra := 4 - dstLen%4  // 计算需要追加的字节数
    if xtra != 4 {  // 如果需要追加的字节数不为 4
        dst = append(dst, make([]byte, xtra)...)  // 在 dst 后追加 xtra 个字节的 0
    }
    xorfwd(dst)  // 对 dst 进行异或操作
    if xtra != 4 {  // 如果需要追加的字节数不为 4
        dst = dst[:dstLen]  // 截取 dst，保留原始长度
    }
    return dst  // 返回加密后的密文
}

// Open implements cipher.AEAD.Open().  // Open 实现了 cipher.AEAD.Open()
func (a *SimpleAuthenticator) Open(dst, nonce, cipherText, extra []byte) ([]byte, error) {  // 定义 Open 方法，对密文进行解密
    dst = append(dst, cipherText...)  // 将密文追加到 dst 中
    dstLen := len(dst)  // 获取 dst 的长度
    xtra := 4 - dstLen%4  // 计算需要追加的字节数
    if xtra != 4 {  // 如果需要追加的字节数不为 4
        dst = append(dst, make([]byte, xtra)...)  // 在 dst 后追加 xtra 个字节的 0
    }
    xorbkd(dst)  // 对 dst 进行异或操作
    if xtra != 4 {  // 如果需要追加的字节数不为 4
        dst = dst[:dstLen]  // 截取 dst，保留原始长度
    }

    fnvHash := fnv.New32a()  // 创建一个新的 32 位哈希对象
    common.Must2(fnvHash.Write(dst[4:]))  // 将 dst 中从第 4 个字节开始的内容写入哈希对象
    if binary.BigEndian.Uint32(dst[:4]) != fnvHash.Sum32() {  // 如果计算的哈希值与密文中的哈希值不相等
        return nil, newError("invalid auth")  // 返回错误信息
    }

    length := binary.BigEndian.Uint16(dst[4:6])  // 从密文中获取明文长度
    if len(dst)-6 != int(length) {  // 如果密文长度减去 6 不等于明文长度
        return nil, newError("invalid auth")  // 返回错误信息
    }

    return dst[6:], nil  // 返回解密后的明文和空错误信息
}
```