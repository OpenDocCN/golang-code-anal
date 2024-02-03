# `v2ray-core\common\crypto\chacha20.go`

```go
// 导入 crypto/cipher 包
import (
    "crypto/cipher"

    "v2ray.com/core/common/crypto/internal"
)

// NewChaCha20Stream 根据给定的密钥和 IV 创建一个新的 ChaCha20 加密/解密流
// 调用者必须确保密钥的长度为 32 字节，IV 的长度为 8 或 12 字节
func NewChaCha20Stream(key []byte, iv []byte) cipher.Stream {
    // 调用 internal 包中的 NewChaCha20Stream 函数创建 ChaCha20 流
    return internal.NewChaCha20Stream(key, iv, 20)
}
```