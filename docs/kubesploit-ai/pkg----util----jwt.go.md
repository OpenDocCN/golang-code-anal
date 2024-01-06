# `kubesploit\pkg\util\jwt.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括对特定用途的适销性或适用性的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参阅<http://www.gnu.org/licenses/>。

// 包名为util
package util
# 导入标准库中的模块
import (
	"bytes"  # 用于操作字节的模块
	"encoding/gob"  # 用于编码和解码数据的模块
	"fmt"  # 用于格式化输出的模块
	"time"  # 用于处理时间的模块

	# 导入第三方库中的模块
	"github.com/fatih/color"  # 用于在终端中输出带颜色的文本
	uuid "github.com/satori/go.uuid"  # 用于生成和解析 UUID 的模块
	"gopkg.in/square/go-jose.v2"  # 用于处理 JOSE 标准的模块
	"gopkg.in/square/go-jose.v2/jwt"  # 用于处理 JWT 的模块

	# 导入自定义的模块
	"kubesploit/pkg/agents"  # 用于处理代理的模块
	"kubesploit/pkg/core"  # 用于处理核心功能的模块
	"kubesploit/pkg/logging"  # 用于记录日志的模块
	"kubesploit/pkg/messages"  # 用于处理消息的模块
)
// GetJWT函数根据提供的代理和JWT密钥返回一个JSON Web Token
func GetJWT(agentID uuid.UUID, key []byte) (string, error) {
    // 如果核心调试模式开启，打印进入agents.GetJWT函数的调试信息
    if core.Debug {
        message("debug", "Entering into agents.GetJWT function")
    }

    // 创建JWE加密器
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.DIRECT,
            Key:       key},
        (&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
    // 如果创建JWE加密器出现错误，返回错误信息
    if encErr != nil {
        return "", fmt.Errorf("there was an error creating the JWE encryptor:\r\n%s", encErr)
    }

    // 创建JWT签名器
    signer, errSigner := jose.NewSigner(jose.SigningKey{
        Algorithm: jose.HS256,
        Key:       key},
        (&jose.SignerOptions{}).WithType("JWT"))
    // 如果创建JWT签名器出现错误，返回错误信息
    if errSigner != nil {
		// 如果创建 JWT 签名者时出现错误，返回错误信息
		return "", fmt.Errorf("there was an error creating the JWT signer:\r\n%s", errSigner.Error())
	}

	// 获取代理的生命周期
	lifetime, errLifetime := agents.GetLifetime(agentID)
	// 如果获取生命周期出现错误并且错误信息不是 "agent WaitTime is equal to zero"，返回错误信息
	if errLifetime != nil && errLifetime.Error() != "agent WaitTime is equal to zero" {
		return "", errLifetime
	}

	// 当服务器尚未收到 AgentInfo 结构并且不知道代理的生命周期或者休眠时间设置为零时使用
	if lifetime == 0 {
		// 设置默认生命周期为 30 秒
		lifetime = time.Second * 30
	}

	// TODO 添加 JWT 声明的其余信息
	// 创建 JWT 声明
	cl := jwt.Claims{
		ID:        agentID.String(),
		NotBefore: jwt.NewNumericDate(time.Now()),
		IssuedAt:  jwt.NewNumericDate(time.Now()),
		Expiry:    jwt.NewNumericDate(time.Now().Add(lifetime)),
	}
// 使用签名者和加密者对声明进行签名和加密，返回JWT字符串和可能的错误
agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
if err != nil {
    return "", fmt.Errorf("there was an error serializing the JWT:\r\n%s", err.Error())
}

// 解析JWT以检查是否有错误
_, errParse := jwt.ParseEncrypted(agentJWT)
if errParse != nil {
    return "", fmt.Errorf("there was an error parsing the encrypted JWT:\r\n%s", errParse.Error())
}

// 记录创建了经过身份验证的JWT
logging.Server(fmt.Sprintf("Created authenticated JWT for %s", agentID))

// 如果处于调试模式，则记录发送给代理的JWT字符串
if core.Debug {
    message("debug", fmt.Sprintf("Sending agent %s an authenticated JWT with a lifetime of %v:\r\n%v",
        agentID.String(), lifetime, agentJWT))
}

// 返回JWT字符串和无错误
return agentJWT, nil
// ValidateJWT函数用于验证提供的JSON Web Token
func ValidateJWT(agentJWT string, key []byte) (uuid.UUID, error) {
    var agentID uuid.UUID
    // 如果核心调试模式开启，则输出调试信息
    if core.Debug {
        message("debug", "Entering into jwt.ValidateJWT")
        message("debug", fmt.Sprintf("Input JWT: %v", agentJWT))
    }

    // 创建一个jwt.Claims结构体实例
    claims := jwt.Claims{}

    // 解析JWT，确保它是一个有效的JWT
    nestedToken, err := jwt.ParseSignedAndEncrypted(agentJWT)
    if err != nil {
        return agentID, fmt.Errorf("there was an error parsing the JWT:\r\n%s", err.Error())
    }

    // 解密JWT
    token, errToken := nestedToken.Decrypt(key)
    if errToken != nil {
        return agentID, fmt.Errorf("there was an error decrypting the JWT:\r\n%s", errToken.Error())
	// 反序列化声明并验证签名
	errClaims := token.Claims(key, &claims)
	// 如果在反序列化 JWT 声明时出现错误，返回错误信息
	if errClaims != nil {
		return agentID, fmt.Errorf("there was an deserializing the JWT claims:\r\n%s", errClaims.Error())
	}

	// 从声明中获取代理 ID
	agentID = uuid.FromStringOrNil(claims.ID)

	// 获取代理的等待时间字段值
	AgentWaitTime, errWait := agents.GetAgentFieldValue(agentID, "WaitTime")
	// 在 OPAQUE 注册和认证期间会返回错误
	if errWait != nil {
		// 如果处于调试模式，记录获取代理等待时间时出现的错误
		if core.Debug {
			message("debug", fmt.Sprintf("there was an error getting the agent's wait time:\r\n%s", errWait.Error()))
		}
	}
	// 如果代理等待时间为空，设置默认值为 "60s"
	if AgentWaitTime == "" {
		AgentWaitTime = "60s"
	}
// 将代理等待时间解析为持续时间
WaitTime, errParse := time.ParseDuration(AgentWaitTime)
if errParse != nil {
    return agentID, fmt.Errorf("there was an error parsing the agent's wait time into a duration:\r\n%s", errParse.Error())
}

// 验证声明；默认的余地是1分钟；将其设置为代理的等待时间设置的1倍
errValidate := claims.ValidateWithLeeway(jwt.Expected{
    Time: time.Now(),
}, WaitTime)

// 如果验证失败，返回错误信息
if errValidate != nil {
    if core.Verbose {
        message("warn", fmt.Sprintf("The JWT claims were not valid for %s", agentID))
        message("note", fmt.Sprintf("JWT Claim Expiry: %s", claims.Expiry.Time()))
        message("note", fmt.Sprintf("JWT Claim Issued: %s", claims.IssuedAt.Time()))
    }
    return agentID, errValidate
}

// 如果调试模式开启，输出代理ID信息
if core.Debug {
    message("debug", fmt.Sprintf("agentID: %s", agentID.String()))
}
	// 打印调试信息，表示离开 jwt.ValidateJWT 函数且没有错误发生
	message("debug", "Leaving jwt.ValidateJWT without error")
	}

	// TODO 我需要验证其他内容，比如令牌的年龄/过期时间
	return agentID, nil
}

// DecryptJWE 函数接收 JWE 字符串和代理密钥，对 JWE 字符串进行解密
func DecryptJWE(jweString string, key []byte) (messages.Base, error) {
	if core.Debug {
		// 打印调试信息，表示进入 jwt.DecryptJWE 函数
		message("debug", "Entering into jwt.DecryptJWE function")
		// 打印调试信息，表示输入的 JWE 字符串
		message("debug", fmt.Sprintf("Input JWE String: %s", jweString))
	}

	var m messages.Base

	// 将 JWE 字符串解析为 JSONWebEncryption 对象
	jwe, errObject := jose.ParseEncrypted(jweString)
	if errObject != nil {
		// 如果解析出错，返回错误信息
		return m, fmt.Errorf("there was an error parseing the JWE string into a JSONWebEncryption object:\r\n%s", errObject)
	}
// 如果调试模式开启，打印解析后的 JWE 对象
if core.Debug {
    message("debug", fmt.Sprintf("Parsed JWE:\r\n%+v", jwe))
}

// 解密 JWE
jweMessage, errDecrypt := jwe.Decrypt(key)
// 如果解密出错，返回错误信息
if errDecrypt != nil {
    return m, fmt.Errorf("there was an error decrypting the JWE:\r\n%s", errDecrypt.Error())
}

// 将 JWE 载荷解码为 messages.Base 结构
errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
// 如果解码出错，返回错误信息
if errDecode != nil {
    return m, fmt.Errorf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error())
}

// 如果调试模式开启，打印离开 jwt.DecryptJWE 函数时没有错误的消息
if core.Debug {
    message("debug", "Leaving jwt.DecryptJWE function without error")
    message("debug", fmt.Sprintf("Returning message base: %+v", m))
}
// message 函数用于在命令行打印消息
func message(level string, message string) {
    // 根据不同的消息级别，使用不同的颜色打印消息
    switch level {
    case "info":
        color.Cyan("[i]" + message)
    case "note":
        color.Yellow("[-]" + message)
    case "warn":
        color.Red("[!]" + message)
    case "debug":
        color.Red("[DEBUG]" + message)
    case "success":
        color.Green("[+]" + message)
    default:
        color.Red("[_-_]Invalid message level: " + message)
    }
}
这是一个代码块的结束标记，表示前面的函数或者循环的结束。
```