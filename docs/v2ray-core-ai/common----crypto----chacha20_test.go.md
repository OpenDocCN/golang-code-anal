# `v2ray-core\common\crypto\chacha20_test.go`

```go
package crypto_test

import (
    "crypto/rand"  // 导入加密随机数生成包
    "encoding/hex"  // 导入十六进制编解码包
    "testing"  // 导入测试包

    "github.com/google/go-cmp/cmp"  // 导入 Google 的比较包

    "v2ray.com/core/common"  // 导入 V2Ray 的通用包
    . "v2ray.com/core/common/crypto"  // 导入 V2Ray 的加密包
)

func mustDecodeHex(s string) []byte {
    b, err := hex.DecodeString(s)  // 使用十六进制编解码包解码字符串
    common.Must(err)  // 检查错误，如果有错误则中断程序
    return b  // 返回解码后的字节数组
}

func TestChaCha20Stream(t *testing.T) {
    var cases = []struct {  // 定义测试用例结构体数组
        key    []byte  // 定义密钥字节数组
        iv     []byte  // 定义初始化向量字节数组
        output []byte  // 定义输出字节数组
    }{
        {
            key: mustDecodeHex("0000000000000000000000000000000000000000000000000000000000000000"),  // 调用解码函数，将十六进制字符串转换为字节数组
            iv:  mustDecodeHex("0000000000000000"),  // 调用解码函数，将十六进制字符串转换为字节数组
            output: mustDecodeHex("76b8e0ada0f13d90405d6ae55386bd28bdd219b8a08ded1aa836efcc8b770dc7" +
                "da41597c5157488d7724e03fb8d84a376a43b8f41518a11cc387b669b2ee6586" +
                "9f07e7be5551387a98ba977c732d080dcb0f29a048e3656912c6533e32ee7aed" +
                "29b721769ce64e43d57133b074d839d531ed1f28510afb45ace10a1f4b794d6f"),  // 调用解码函数，将十六进制字符串转换为字节数组
        },
        {
            key: mustDecodeHex("5555555555555555555555555555555555555555555555555555555555555555"),  // 调用解码函数，将十六进制字符串转换为字节数组
            iv:  mustDecodeHex("5555555555555555"),  // 调用解码函数，将十六进制字符串转换为字节数组
            output: mustDecodeHex("bea9411aa453c5434a5ae8c92862f564396855a9ea6e22d6d3b50ae1b3663311" +
                "a4a3606c671d605ce16c3aece8e61ea145c59775017bee2fa6f88afc758069f7" +
                "e0b8f676e644216f4d2a3422d7fa36c6c4931aca950e9da42788e6d0b6d1cd83" +
                "8ef652e97b145b14871eae6c6804c7004db5ac2fce4c68c726d004b10fcaba86"),  // 调用解码函数，将十六进制字符串转换为字节数组
        },
        {
            key:    mustDecodeHex("0000000000000000000000000000000000000000000000000000000000000000"),  // 调用解码函数，将十六进制字符串转换为字节数组
            iv:     mustDecodeHex("000000000000000000000000"),  // 调用解码函数，将十六进制字符串转换为字节数组
            output: mustDecodeHex("76b8e0ada0f13d90405d6ae55386bd28bdd219b8a08ded1aa836efcc8b770dc7da41597c5157488d7724e03fb8d84a376a43b8f41518a11cc387b669b2ee6586"),  // 调用解码函数，将十六进制字符串转换为字节数组
        },
    }
    # 遍历测试用例数组
    for _, c := range cases {
        # 创建一个新的ChaCha20流加密对象，使用给定的密钥和初始化向量
        s := NewChaCha20Stream(c.key, c.iv)
        # 创建一个与输出长度相同的输入字节切片
        input := make([]byte, len(c.output))
        # 创建一个与输出长度相同的实际输出字节切片
        actualOutout := make([]byte, len(c.output))
        # 使用ChaCha20流加密对象对输入进行加密，结果存储在实际输出字节切片中
        s.XORKeyStream(actualOutout, input)
        # 比较期望输出和实际输出，如果不相等则输出错误信息并终止测试
        if r := cmp.Diff(c.output, actualOutout); r != "" {
            t.Fatal(r)
        }
    }
// 测试ChaCha20解码函数
func TestChaCha20Decoding(t *testing.T) {
    // 创建32字节的密钥
    key := make([]byte, 32)
    // 生成随机密钥
    common.Must2(rand.Read(key))
    // 创建8字节的初始化向量
    iv := make([]byte, 8)
    // 生成随机初始化向量
    common.Must2(rand.Read(iv))
    // 创建ChaCha20流加密器
    stream := NewChaCha20Stream(key, iv)

    // 创建1024字节的载荷
    payload := make([]byte, 1024)
    // 生成随机载荷
    common.Must2(rand.Read(payload))

    // 创建与载荷相同长度的字节切片
    x := make([]byte, len(payload))
    // 使用ChaCha20流加密器对载荷进行解密
    stream.XORKeyStream(x, payload)

    // 创建新的ChaCha20流加密器
    stream2 := NewChaCha20Stream(key, iv)
    // 使用新的ChaCha20流加密器对解密后的数据再次进行解密
    stream2.XORKeyStream(x, x)
    // 检查解密后的数据与原始载荷是否相同
    if r := cmp.Diff(x, payload); r != "" {
        t.Fatal(r)
    }
}
```