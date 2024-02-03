# `v2ray-core\proxy\vmess\aead\kdf.go`

```go
package aead

import (
    "crypto/hmac"  // 导入加密哈希消息认证码包
    "crypto/sha256"  // 导入SHA256哈希算法包
    "hash"  // 导入哈希函数包
)

func KDF(key []byte, path ...string) []byte {
    hmacf := hmac.New(func() hash.Hash {  // 创建一个新的HMAC对象
        return sha256.New()  // 返回一个新的SHA256哈希对象
    }, []byte(KDFSaltConst_VMessAEADKDF))  // 使用常量KDFSaltConst_VMessAEADKDF创建HMAC对象

    for _, v := range path {  // 遍历路径参数
        hmacf = hmac.New(func() hash.Hash {  // 创建一个新的HMAC对象
            return hmacf  // 返回之前创建的HMAC对象
        }, []byte(v))  // 使用路径参数v创建HMAC对象
    }
    hmacf.Write(key)  // 将密钥写入HMAC对象
    return hmacf.Sum(nil)  // 返回HMAC对象的哈希值
}

func KDF16(key []byte, path ...string) []byte {
    r := KDF(key, path...)  // 调用KDF函数生成哈希值
    return r[:16]  // 返回哈希值的前16个字节
}
```