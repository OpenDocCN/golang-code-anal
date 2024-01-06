# `kubesploit\pkg\core\core.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit 是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发是希望它对增强组织安全有用。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包声明
package core
// 导入标准库中的模块
import (
	"bytes"  // 用于操作字节流
	"crypto/rsa"  // 提供了 RSA 加密算法的实现
	"encoding/gob"  // 用于编码和解码数据结构
	"fmt"  // 用于格式化输出
	"math/rand"  // 生成随机数
	"os"  // 提供了对操作系统功能的访问
	"time"  // 提供了时间相关的功能

	// 导入第三方库中的模块
	"gopkg.in/square/go-jose.v2"  // 提供了 JOSE（JSON Object Signing and Encryption）规范的实现

	// 导入 Merlin 应用中的模块
	"kubesploit/pkg/messages"  // 导入了自定义的消息模块
)

// Debug 变量用于控制 Merlin 是否处于调试模式，并显示调试信息
var Debug = false  // 默认为关闭调试模式
// Verbose用于将Merlin置于详细模式，并显示详细消息
var Verbose = false

// CurrentDir是Merlin执行时的当前目录
var CurrentDir, _ = os.Getwd()
var src = rand.NewSource(time.Now().UnixNano())

// 常量
const (
	letterIdxBits = 6                    // 用6位表示一个字母索引
	letterIdxMask = 1<<letterIdxBits - 1 // 所有的1位，与letterIdxBits一样多
	letterIdxMax  = 63 / letterIdxBits   // 在63位中适合的字母索引数量
	letterBytes   = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
)

// RandStringBytesMaskImprSrc生成并返回一个长度为n的随机字符串
func RandStringBytesMaskImprSrc(n int) string {
	// http://stackoverflow.com/questions/22892120/how-to-generate-a-random-string-of-a-fixed-length-in-golang
	b := make([]byte, n)
	// 一个src.Int63()生成63个随机位，足够生成letterIdxMax个字符！
// 使用循环生成随机字符串，参数 n 为字符串长度
for i, cache, remain := n-1, src.Int63(), letterIdxMax; i >= 0; {
    // 如果剩余字符数为 0，则重新生成随机数
    if remain == 0 {
        cache, remain = src.Int63(), letterIdxMax
    }
    // 从随机数中取出一个字符的索引，将其转换为字母并存入结果数组
    if idx := int(cache & letterIdxMask); idx < len(letterBytes) {
        b[i] = letterBytes[idx]
        i--
    }
    // 将随机数右移，减少剩余字符数
    cache >>= letterIdxBits
    remain--
}
// 将结果数组转换为字符串并返回
return string(b)
}

// DecryptJWE 使用提供的 JWE 字符串和代理密钥解密
func DecryptJWE(jweString string, key []byte) (messages.Base, error) {
    var m messages.Base

    // 将 JWE 字符串解析为 JSONWebEncryption 对象
    jwe, errObject := jose.ParseEncrypted(jweString)
```

// 如果解析 JWE 字符串为 JSONWebEncryption 对象时出现错误，返回错误信息
if errObject != nil {
    return m, fmt.Errorf("there was an error parseing the JWE string into a JSONWebEncryption object:\r\n%s", errObject)
}

// 解密 JWE
jweMessage, errDecrypt := jwe.Decrypt(key)
// 如果解密 JWE 字符串时出现错误，返回错误信息
if errDecrypt != nil {
    return m, fmt.Errorf("there was an error decrypting the JWE string:\r\n%s", errDecrypt.Error())
}

// 将 JWE 载荷解码为 messages.Base 结构
errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
// 如果解码 JWE 载荷时出现错误，返回错误信息
if errDecode != nil {
    return m, fmt.Errorf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error())
}

// 返回解码后的消息和空错误
return m, nil
}

// GetJWESymetric 接受一个输入，通常是一个经过 gob 编码的 messages.Base，返回一个紧凑序列化的 JWE
// 使用对称加密算法对数据进行加密，并返回加密后的字符串和可能出现的错误
func GetJWESymetric(data []byte, key []byte) (string, error) {
	// AES GCM算法的密钥必须遵循[NIST.800-38D]第8.3节中的约束，即“认证加密函数的总调用次数不得超过2^32次，包括所有IV长度和使用给定密钥的认证加密函数的所有实例”。根据这个规则，AES GCM算法的密钥值不能超过2^32次。== 4294967296
	// TODO 确保不超过4294967295次使用相同密钥创建JWE
	// 创建一个加密器，使用AES GCM算法，生成一个消息密钥并使用传入的密钥进行加密
	encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
		jose.Recipient{
			Algorithm: jose.PBES2_HS512_A256KW, // 使用传入的密钥创建一个消息密钥并进行加密
			//Algorithm: jose.DIRECT, // 不创建消息密钥
			PBES2Count: 500000, // PBES2算法的迭代次数
			Key:        key}, // 加密所使用的密钥
		nil)
	if encErr != nil {
		return "", fmt.Errorf("创建JWE加密器时出错：\r\n%s", encErr)
	}
	// 对数据进行加密
	jwe, errJWE := encrypter.Encrypt(data)
// 如果加密认证 JSON 对象到 JWE 对象时出现错误，返回错误信息
if errJWE != nil {
    return "", fmt.Errorf("there was an error encrypting the Authentication JSON object to a JWE object:\r\n%s", errJWE.Error())
}

// 将 JWE 对象序列化为紧凑格式
serialized, errSerialized := jwe.CompactSerialize()
// 如果序列化 JWE 对象时出现错误，返回错误信息
if errSerialized != nil {
    return "", fmt.Errorf("there was an error serializing the JWE in compact format:\r\n%s", errSerialized.Error())
}

// 解析序列化后的 JWE 对象，确保没有错误
_, errJWE = jose.ParseEncrypted(serialized)
// 如果解析加密的 JWE 对象时出现错误，返回错误信息
if errJWE != nil {
    return "", fmt.Errorf("there was an error parsing the encrypted JWE:\r\n%s", errJWE.Error())
}

// 返回序列化后的 JWE 对象
return serialized, nil
}

// GetJWEAsymetric 接受一个输入，通常是 gob 编码的 messages.Base，使用提供的 RSA 公钥返回紧凑序列化的 JWE
// 使用非对称加密算法将数据加密成 JWE 对象，并返回序列化后的字符串
func GetJWEAsymetric(data []byte, key *rsa.PublicKey) (string, error) {
	// TODO change key algorithm to ECDH
	// 创建一个新的加密器，使用 RSA_OAEP 算法和指定的公钥
	encrypter, encErr := jose.NewEncrypter(jose.A256GCM, jose.Recipient{Algorithm: jose.RSA_OAEP, Key: key}, nil)
	if encErr != nil {
		return "", fmt.Errorf("there was an error creating the agent encryptor:\r\n%s", encErr)
	}
	// 将数据加密成 JWE 对象
	jwe, errJWE := encrypter.Encrypt(data)
	if errJWE != nil {
		return "", fmt.Errorf("there was an error encrypting the data into a JWE object:\r\n%s", errJWE.Error())
	}

	// 将 JWE 对象序列化成紧凑格式的字符串
	serialized, errSerialized := jwe.CompactSerialize()
	if errSerialized != nil {
		return "", fmt.Errorf("there was an error serializing the JWE in compact format:\r\n%s", errSerialized.Error())
	}

	// 解析序列化后的 JWE 字符串，确保没有错误
	_, errJWE = jose.ParseEncrypted(serialized)
	if errJWE != nil {
		return "", fmt.Errorf("there was an error parsing the encrypted JWE:\r\n%s", errJWE.Error())
	}
这部分代码缺少上下文，无法确定具体的作用和功能，无法添加注释。
```