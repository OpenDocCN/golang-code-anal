# `kubesploit\pkg\util\jwt.go`

```
// Kubesploit 是一个基于 Russel Van Tuyl 的 Merlin 框架构建的后渗透命令和控制框架。
// 本文件是 Kubesploit 的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit 是自由软件：您可以根据 GNU 通用公共许可证的条款重新分发或修改它，
// 无论是许可证的第 3 版还是任何以后的版本。

// Kubesploit 分发的目的是为了增强组织的安全性。
// Kubesploit 不得以任何恶意方式使用。
// Kubesploit 按原样分发，没有任何保证；包括适销性或特定用途适用性的暗示保证。请参阅
// GNU 通用公共许可证以获取更多详细信息。

// 您应该已经收到 GNU 通用公共许可证的副本。
// 如果没有，请参阅 <http://www.gnu.org/licenses/>。

package util

import (
    // 标准库
    "bytes"
    "encoding/gob"
    "fmt"
    "time"

    // 第三方库
    "github.com/fatih/color"
    uuid "github.com/satori/go.uuid"
    "gopkg.in/square/go-jose.v2"
    "gopkg.in/square/go-jose.v2/jwt"

    // Merlin
    "kubesploit/pkg/agents"
    "kubesploit/pkg/core"
    "kubesploit/pkg/logging"
    "kubesploit/pkg/messages"
)

// GetJWT 使用 JWT 密钥接口为提供的代理返回 JSON Web 令牌
func GetJWT(agentID uuid.UUID, key []byte) (string, error) {
    if core.Debug {
        message("debug", "Entering into agents.GetJWT function")
    }

    // 创建 JWE 加密器
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.DIRECT,
            Key:       key},
        (&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
    if encErr != nil {
        return "", fmt.Errorf("there was an error creating the JWE encryptor:\r\n%s", encErr)
    }
    // 使用给定的密钥和算法创建一个新的签名者
    signer, errSigner := jose.NewSigner(jose.SigningKey{
        Algorithm: jose.HS256,
        Key:       key},
        (&jose.SignerOptions{}).WithType("JWT"))
    // 如果创建签名者时出现错误，则返回错误信息
    if errSigner != nil {
        return "", fmt.Errorf("there was an error creating the JWT signer:\r\n%s", errSigner.Error())
    }

    // 获取代理的生命周期
    lifetime, errLifetime := agents.GetLifetime(agentID)
    // 如果获取生命周期时出现错误，并且错误信息不是"agent WaitTime is equal to zero"，则返回错误信息
    if errLifetime != nil && errLifetime.Error() != "agent WaitTime is equal to zero" {
        return "", errLifetime
    }

    // 当服务器尚未收到AgentInfo结构并且不知道代理的生命周期，或者睡眠时间设置为零时使用
    if lifetime == 0 {
        lifetime = time.Second * 30
    }

    // 创建JWT声明信息
    cl := jwt.Claims{
        ID:        agentID.String(),
        NotBefore: jwt.NewNumericDate(time.Now()),
        IssuedAt:  jwt.NewNumericDate(time.Now()),
        Expiry:    jwt.NewNumericDate(time.Now().Add(lifetime)),
    }

    // 使用签名者和加密器对声明进行签名和加密
    agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
    // 如果在序列化JWT时出现错误，则返回错误信息
    if err != nil {
        return "", fmt.Errorf("there was an error serializing the JWT:\r\n%s", err.Error())
    }

    // 解析JWT以检查错误
    _, errParse := jwt.ParseEncrypted(agentJWT)
    // 如果解析加密JWT时出现错误，则返回错误信息
    if errParse != nil {
        return "", fmt.Errorf("there was an error parsing the encrypted JWT:\r\n%s", errParse.Error())
    }
    // 记录创建的经过身份验证的JWT
    logging.Server(fmt.Sprintf("Created authenticated JWT for %s", agentID))
    // 如果处于调试模式，则记录发送给代理的JWT信息
    if core.Debug {
        message("debug", fmt.Sprintf("Sending agent %s an authenticated JWT with a lifetime of %v:\r\n%v",
            agentID.String(), lifetime, agentJWT))
    }

    // 返回经过身份验证的JWT和空错误
    return agentJWT, nil
// ValidateJWT 验证提供的 JSON Web Token
func ValidateJWT(agentJWT string, key []byte) (uuid.UUID, error) {
    var agentID uuid.UUID
    if core.Debug {
        message("debug", "Entering into jwt.ValidateJWT")
        message("debug", fmt.Sprintf("Input JWT: %v", agentJWT))
    }

    claims := jwt.Claims{}

    // 解析以确保它是有效的 JWT
    nestedToken, err := jwt.ParseSignedAndEncrypted(agentJWT)
    if err != nil {
        return agentID, fmt.Errorf("there was an error parsing the JWT:\r\n%s", err.Error())
    }

    // 解密 JWT
    token, errToken := nestedToken.Decrypt(key)
    if errToken != nil {
        return agentID, fmt.Errorf("there was an error decrypting the JWT:\r\n%s", errToken.Error())
    }

    // 反序列化声明并验证签名
    errClaims := token.Claims(key, &claims)
    if errClaims != nil {
        return agentID, fmt.Errorf("there was an deserializing the JWT claims:\r\n%s", errClaims.Error())
    }

    agentID = uuid.FromStringOrNil(claims.ID)

    AgentWaitTime, errWait := agents.GetAgentFieldValue(agentID, "WaitTime")
    // 在 OPAQUE 注册和认证期间将返回错误
    if errWait != nil {
        if core.Debug {
            message("debug", fmt.Sprintf("there was an error getting the agent's wait time:\r\n%s", errWait.Error()))
        }
    }
    if AgentWaitTime == "" {
        AgentWaitTime = "60s"
    }

    WaitTime, errParse := time.ParseDuration(AgentWaitTime)
    if errParse != nil {
        return agentID, fmt.Errorf("there was an error parsing the agent's wait time into a duration:\r\n%s", errParse.Error())
    }
    // 验证声明；默认 Leeway 为 1 分钟；将其设置为代理的 WaitTime 设置的 1 倍
    errValidate := claims.ValidateWithLeeway(jwt.Expected{
        Time: time.Now(),
    }, WaitTime)
}
    # 如果 JWT 验证出错
    if errValidate != nil:
        # 如果设置了详细输出
        if core.Verbose:
            # 输出警告信息
            message("warn", fmt.Sprintf("The JWT claims were not valid for %s", agentID))
            # 输出提示信息，JWT Claim Expiry
            message("note", fmt.Sprintf("JWT Claim Expiry: %s", claims.Expiry.Time()))
            # 输出提示信息，JWT Claim Issued
            message("note", fmt.Sprintf("JWT Claim Issued: %s", claims.IssuedAt.Time()))
        # 返回代理ID和验证错误
        return agentID, errValidate
    # 如果设置了调试输出
    if core.Debug:
        # 输出调试信息，代理ID
        message("debug", fmt.Sprintf("agentID: %s", agentID.String()))
        # 输出调试信息，离开 jwt.ValidateJWT 函数，没有错误
        message("debug", "Leaving jwt.ValidateJWT without error")
    # 返回代理ID和空错误
    # TODO 我需要验证其他内容，比如令牌的年龄/过期时间
    return agentID, nil
// DecryptJWE函数接受提供的JWE字符串，并使用每个代理密钥对其进行解密
func DecryptJWE(jweString string, key []byte) (messages.Base, error) {
    // 如果核心调试模式开启，则打印进入jwt.DecryptJWE函数的调试信息
    if core.Debug {
        message("debug", "Entering into jwt.DecryptJWE function")
        message("debug", fmt.Sprintf("Input JWE String: %s", jweString))
    }

    var m messages.Base

    // 将JWE字符串解析回JSONWebEncryption对象
    jwe, errObject := jose.ParseEncrypted(jweString)
    if errObject != nil {
        return m, fmt.Errorf("there was an error parseing the JWE string into a JSONWebEncryption object:\r\n%s", errObject)
    }

    // 如果核心调试模式开启，则打印解析后的JWE对象信息
    if core.Debug {
        message("debug", fmt.Sprintf("Parsed JWE:\r\n%+v", jwe))
    }

    // 解密JWE
    jweMessage, errDecrypt := jwe.Decrypt(key)
    if errDecrypt != nil {
        return m, fmt.Errorf("there was an error decrypting the JWE:\r\n%s", errDecrypt.Error())
    }

    // 将JWE载荷解码为messages.Base结构
    errDecode := gob.NewDecoder(bytes.NewReader(jweMessage)).Decode(&m)
    if errDecode != nil {
        return m, fmt.Errorf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error())
    }

    // 如果核心调试模式开启，则打印离开jwt.DecryptJWE函数的调试信息和返回的消息基础信息
    if core.Debug {
        message("debug", "Leaving jwt.DecryptJWE function without error")
        message("debug", fmt.Sprintf("Returning message base: %+v", m))
    }
    return m, nil
}

// message用于在命令行打印消息
func message(level string, message string) {
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
```