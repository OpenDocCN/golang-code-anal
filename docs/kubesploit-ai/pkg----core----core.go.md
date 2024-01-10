# `kubesploit\pkg\core\core.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package core

import (
    // 标准库
    "bytes"
    "crypto/rsa"
    "encoding/gob"
    "fmt"
    "math/rand"
    "os"
    "time"

    // 第三方库
    "gopkg.in/square/go-jose.v2"

    // Merlin
    "kubesploit/pkg/messages"
)

// Debug 将Merlin置于调试模式，并显示调试消息
var Debug = false

// Verbose 将Merlin置于详细模式，并显示详细消息
var Verbose = false

// CurrentDir 是Merlin执行的当前目录
var CurrentDir, _ = os.Getwd()
var src = rand.NewSource(time.Now().UnixNano())

// 常量
const (
    letterIdxBits = 6                    // 6位表示字母索引
    letterIdxMask = 1<<letterIdxBits - 1 // 所有的1位，与letterIdxBits一样多
    letterIdxMax  = 63 / letterIdxBits   // 63位中适合的字母索引数量
    letterBytes   = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
)

// RandStringBytesMaskImprSrc 生成并返回一个n个字符长的随机字符串
func RandStringBytesMaskImprSrc(n int) string {
    // 创建一个长度为n的字节切片，用于存储生成的随机字符串
    b := make([]byte, n)
    // 使用src.Int63()生成63位随机比特，足够用于生成letterIdxMax个字符
    for i, cache, remain := n-1, src.Int63(), letterIdxMax; i >= 0; {
        // 如果remain为0，则重新生成随机比特，并重置remain为letterIdxMax
        if remain == 0 {
            cache, remain = src.Int63(), letterIdxMax
        }
        // 通过位运算获取随机字符的索引，并将其存储到字节切片中
        if idx := int(cache & letterIdxMask); idx < len(letterBytes) {
            b[i] = letterBytes[idx]
            i--
        }
        // 将cache右移letterIdxBits位，减少remain的值
        cache >>= letterIdxBits
        remain--
    }
    // 将字节切片转换为字符串并返回
    return string(b)
}

// 使用提供的代理密钥解密提供的 JWE 字符串
func DecryptJWE(jweString string, key []byte) (messages.Base, error) {
    var m messages.Base

    // 将 JWE 字符串解析回 JSONWebEncryption 对象
    jwe, errObject := jose.ParseEncrypted(jweString)
    if errObject != nil {
        return m, fmt.Errorf("解析 JWE 字符串为 JSONWebEncryption 对象时出错:\r\n%s", errObject)
    }

    // 解密 JWE
    jweMessage, errDecrypt := jwe.Decrypt(key)
    if errDecrypt != nil {
        return m, fmt.Errorf("解密 JWE 字符串时出错:\r\n%s", errDecrypt.Error())
    }

    // 将 JWE 载荷解码为 messages.Base 结构
    errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
    if errDecode != nil {
        return m, fmt.Errorf("解码代理发送的 JWE 载荷消息时出错:\r\n%s", errDecode.Error())
    }

    return m, nil
}

// GetJWESymetric 获取一个输入，通常是一个经过 gob 编码的 messages.Base，然后使用提供的输入密钥返回一个紧凑序列化的 JWE
func GetJWESymetric(data []byte, key []byte) (string, error) {
    // 使用 AES GCM 的密钥必须遵循 [NIST.800-38D] 第 8.3 节中的约束，该节规定：“认证加密函数的总调用次数不得超过 2^32 次，包括所有 IV 长度和使用给定密钥的认证加密函数的所有实例”。根据此规则，AES GCM 不得使用相同的密钥值超过 2^32 次。== 4294967296
    // TODO 确保不超过 4294967295 次使用相同密钥创建 JWE
    // 使用指定的算法和接收者信息创建加密器，生成一个 per message key，并使用传入的密钥进行加密
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.PBES2_HS512_A256KW, // 使用传入的密钥创建一个 per message key 进行加密
            //Algorithm: jose.DIRECT, // 不创建 per message key
            PBES2Count: 500000, // PBES2 迭代次数
            Key:        key}, // 加密使用的密钥
        nil) // 附加参数为空
    if encErr != nil {
        return "", fmt.Errorf("there was an error creating the JWE encryptor:\r\n%s", encErr) // 如果创建加密器出错，则返回错误信息
    }
    // 使用加密器对数据进行加密
    jwe, errJWE := encrypter.Encrypt(data)
    if errJWE != nil {
        return "", fmt.Errorf("there was an error encrypting the Authentication JSON object to a JWE object:\r\n%s", errJWE.Error()) // 如果加密出错，则返回错误信息
    }

    // 将加密后的数据序列化为紧凑格式
    serialized, errSerialized := jwe.CompactSerialize()
    if errSerialized != nil {
        return "", fmt.Errorf("there was an error serializing the JWE in compact format:\r\n%s", errSerialized.Error()) // 如果序列化出错，则返回错误信息
    }

    // 解析序列化后的数据，确保没有序列化错误
    _, errJWE = jose.ParseEncrypted(serialized)
    if errJWE != nil {
        return "", fmt.Errorf("there was an error parsing the encrypted JWE:\r\n%s", errJWE.Error()) // 如果解析出错，则返回错误信息
    }

    // 返回序列化后的数据和空错误
    return serialized, nil
// GetJWEAsymetric函数接受一个输入，通常是一个gob编码的messages.Base，然后使用提供的RSA公钥返回一个紧凑序列化的JWE
func GetJWEAsymetric(data []byte, key *rsa.PublicKey) (string, error) {
    // TODO 将密钥算法更改为ECDH
    // 创建一个加密器，使用指定的RSA公钥和算法A256GCM
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM, jose.Recipient{Algorithm: jose.RSA_OAEP, Key: key}, nil)
    if encErr != nil {
        return "", fmt.Errorf("创建加密器时出错:\r\n%s", encErr)
    }
    // 将数据加密为JWE对象
    jwe, errJWE := encrypter.Encrypt(data)
    if errJWE != nil {
        return "", fmt.Errorf("将数据加密为JWE对象时出错:\r\n%s", errJWE.Error())
    }

    // 将JWE对象序列化为紧凑格式
    serialized, errSerialized := jwe.CompactSerialize()
    if errSerialized != nil {
        return "", fmt.Errorf("将JWE对象序列化为紧凑格式时出错:\r\n%s", errSerialized.Error())
    }

    // 解析序列化后的JWE对象，确保没有序列化错误
    _, errJWE = jose.ParseEncrypted(serialized)
    if errJWE != nil {
        return "", fmt.Errorf("解析加密的JWE对象时出错:\r\n%s", errJWE.Error())
    }

    return serialized, nil
}
```