# `v2ray-core\common\crypto\aes.go`

```
package crypto

import (
    "crypto/aes"  // 导入 AES 加密算法
    "crypto/cipher"  // 导入密码接口

    "v2ray.com/core/common"  // 导入通用包
)

// NewAesDecryptionStream creates a new AES encryption stream based on given key and IV.
// Caller must ensure the length of key and IV is either 16, 24 or 32 bytes.
func NewAesDecryptionStream(key []byte, iv []byte) cipher.Stream {
    return NewAesStreamMethod(key, iv, cipher.NewCFBDecrypter)  // 创建基于给定密钥和 IV 的 AES 解密流
}

// NewAesEncryptionStream creates a new AES description stream based on given key and IV.
// Caller must ensure the length of key and IV is either 16, 24 or 32 bytes.
func NewAesEncryptionStream(key []byte, iv []byte) cipher.Stream {
    return NewAesStreamMethod(key, iv, cipher.NewCFBEncrypter)  // 创建基于给定密钥和 IV 的 AES 加密流
}

func NewAesStreamMethod(key []byte, iv []byte, f func(cipher.Block, []byte) cipher.Stream) cipher.Stream {
    aesBlock, err := aes.NewCipher(key)  // 创建 AES 加密算法的密钥块
    common.Must(err)  // 检查错误
    return f(aesBlock, iv)  // 返回基于给定密钥块和 IV 的密码流
}

// NewAesCTRStream creates a stream cipher based on AES-CTR.
func NewAesCTRStream(key []byte, iv []byte) cipher.Stream {
    return NewAesStreamMethod(key, iv, cipher.NewCTR)  // 创建基于 AES-CTR 的流密码
}

// NewAesGcm creates a AEAD cipher based on AES-GCM.
func NewAesGcm(key []byte) cipher.AEAD {
    block, err := aes.NewCipher(key)  // 创建 AES 加密算法的密钥块
    common.Must(err)  // 检查错误
    aead, err := cipher.NewGCM(block)  // 创建基于 AES-GCM 的 AEAD 密码
    common.Must(err)  // 检查错误
    return aead  // 返回 AEAD 密码
}
```