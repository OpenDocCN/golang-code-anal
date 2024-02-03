# `v2ray-core\transport\internet\kcp\cryptreal.go`

```go
# 导入所需的包
package kcp

import (
    "crypto/aes"  # 导入 AES 加密算法
    "crypto/cipher"  # 导入加密算法接口
    "crypto/sha256"  # 导入 SHA256 哈希算法
    "v2ray.com/core/common"  # 导入通用工具包
)

# 根据种子生成基于 AEAD AES GCM 的加密算法
func NewAEADAESGCMBasedOnSeed(seed string) cipher.AEAD:
    # 对种子进行 SHA256 哈希运算
    HashedSeed := sha256.Sum256([]byte(seed))
    # 使用前16个字节创建 AES 加密块
    aesBlock := common.Must2(aes.NewCipher(HashedSeed[:16])).(cipher.Block)
    # 使用 AES 加密块创建 GCM 模式的加密算法
    return common.Must2(cipher.NewGCM(aesBlock)).(cipher.AEAD)
```