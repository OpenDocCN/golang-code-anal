# `v2ray-core\common\crypto\benchmark_test.go`

```
package crypto_test

import (
    "crypto/cipher"  // 导入加密算法的包
    "testing"  // 导入测试包

    . "v2ray.com/core/common/crypto"  // 导入自定义的加密包
)

const benchSize = 1024 * 1024  // 定义测试数据大小为 1MB

func benchmarkStream(b *testing.B, c cipher.Stream) {
    b.SetBytes(benchSize)  // 设置测试数据大小
    input := make([]byte, benchSize)  // 创建指定大小的输入数据切片
    output := make([]byte, benchSize)  // 创建指定大小的输出数据切片
    b.ResetTimer()  // 重置计时器
    for i := 0; i < b.N; i++ {  // 循环执行测试
        c.XORKeyStream(output, input)  // 使用加密流对输入数据进行加密
    }
}

func BenchmarkChaCha20(b *testing.B) {
    key := make([]byte, 32)  // 创建32字节的密钥
    nonce := make([]byte, 8)  // 创建8字节的随机数
    c := NewChaCha20Stream(key, nonce)  // 创建 ChaCha20 加密流
    benchmarkStream(b, c)  // 执行加密流性能测试
}

func BenchmarkChaCha20IETF(b *testing.B) {
    key := make([]byte, 32)  // 创建32字节的密钥
    nonce := make([]byte, 12)  // 创建12字节的随机数
    c := NewChaCha20Stream(key, nonce)  // 创建 ChaCha20 加密流
    benchmarkStream(b, c)  // 执行加密流性能测试
}

func BenchmarkAESEncryption(b *testing.B) {
    key := make([]byte, 32)  // 创建32字节的密钥
    iv := make([]byte, 16)  // 创建16字节的初始化向量
    c := NewAesEncryptionStream(key, iv)  // 创建 AES 加密流

    benchmarkStream(b, c)  // 执行加密流性能测试
}

func BenchmarkAESDecryption(b *testing.B) {
    key := make([]byte, 32)  // 创建32字节的密钥
    iv := make([]byte, 16)  // 创建16字节的初始化向量
    c := NewAesDecryptionStream(key, iv)  // 创建 AES 解密流

    benchmarkStream(b, c)  // 执行解密流性能测试
}
```