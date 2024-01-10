# `v2ray-core\transport\internet\quic\config.go`

```
// +build !confonly

package quic

import (
    "crypto/aes"  // 导入 AES 加密算法
    "crypto/cipher"  // 导入加密算法接口
    "crypto/sha256"  // 导入 SHA256 哈希算法

    "golang.org/x/crypto/chacha20poly1305"  // 导入 ChaCha20-Poly1305 加密算法
    "v2ray.com/core/common"  // 导入通用工具包
    "v2ray.com/core/common/protocol"  // 导入协议相关包
    "v2ray.com/core/transport/internet"  // 导入网络传输相关包
)

// 根据配置获取认证加密算法
func getAuth(config *Config) (cipher.AEAD, error) {
    // 获取安全类型
    security := config.Security.GetSecurityType()
    // 如果安全类型为 NONE，则返回空
    if security == protocol.SecurityType_NONE {
        return nil, nil
    }

    // 对密钥进行加盐处理
    salted := []byte(config.Key + "v2ray-quic-salt")
    // 使用 SHA256 对加盐后的密钥进行哈希处理，生成密钥
    key := sha256.Sum256(salted)

    // 如果安全类型为 AES128_GCM
    if security == protocol.SecurityType_AES128_GCM {
        // 创建 AES 加密算法实例
        block, err := aes.NewCipher(key[:16])
        // 检查错误
        common.Must(err)
        // 返回 GCM 模式的加密算法
        return cipher.NewGCM(block)
    }

    // 如果安全类型为 CHACHA20_POLY1305
    if security == protocol.SecurityType_CHACHA20_POLY1305 {
        // 返回 ChaCha20-Poly1305 加密算法
        return chacha20poly1305.New(key[:])
    }

    // 如果安全类型不支持，则返回错误
    return nil, newError("unsupported security type")
}

// 根据配置获取数据包头部
func getHeader(config *Config) (internet.PacketHeader, error) {
    // 如果配置中的头部为空，则返回空
    if config.Header == nil {
        return nil, nil
    }

    // 获取头部实例
    msg, err := config.Header.GetInstance()
    // 如果出现错误，则返回错误
    if err != nil {
        return nil, err
    }

    // 创建数据包头部
    return internet.CreatePacketHeader(msg)
}
```