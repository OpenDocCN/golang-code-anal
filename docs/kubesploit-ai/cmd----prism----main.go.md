# `kubesploit\cmd\prism\main.go`

```go
/*
Kubesploit is a post-exploitation command and control framework built on top of Merlin by Russel Van Tuyl.
This file is part of Kubesploit.
Copyright (c) 2021 CyberArk Software Ltd. All rights reserved.

Kubesploit is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
any later version.

Kubesploit is distributed in the hope that it will be useful for enhancing organizations' security.
Kubesploit shall not be used in any malicious manner.
Kubesploit is distributed AS-IS, WITHOUT ANY WARRANTY; including the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with Kubesploit.  If not, see <http://www.gnu.org/licenses/>.
*/
package main

import (
    // Standard
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "crypto/sha256"  // 导入 crypto/sha256 包，用于 SHA-256 哈希算法
    "encoding/gob"  // 导入 encoding/gob 包，用于 GOB 编码和解码
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 编码和解码
    "flag"  // 导入 flag 包，用于命令行参数解析
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于 I/O 操作
    "net/http"  // 导入 net/http 包，用于 HTTP 客户端和服务端的实现
    "os"  // 导入 os 包，提供了一些与操作系统交互的函数
    "strings"  // 导入 strings 包，用于操作字符串
    "time"  // 导入 time 包，用于时间相关操作

    // 3rd Party
    "github.com/cretz/gopaque/gopaque"  // 导入第三方库 github.com/cretz/gopaque/gopaque
    "github.com/fatih/color"  // 导入第三方库 github.com/fatih/color
    "github.com/satori/go.uuid"  // 导入第三方库 github.com/satori/go.uuid
    "golang.org/x/crypto/pbkdf2"  // 导入第三方库 golang.org/x/crypto/pbkdf2
    "gopkg.in/square/go-jose.v2"  // 导入第三方库 gopkg.in/square/go-jose.v2
    "gopkg.in/square/go-jose.v2/jwt"  // 导入第三方库 gopkg.in/square/go-jose.v2/jwt

    // Merlin
    "kubesploit/pkg/agent"  // 导入自定义包 kubesploit/pkg/agent
    "kubesploit/pkg/core"  // 导入自定义包 kubesploit/pkg/core
    "kubesploit/pkg/messages"  // 导入自定义包 kubesploit/pkg/messages
)

// GLOBAL VARIABLES
var url = "https://127.0.0.1:443"  // 定义全局变量 url，存储目标地址
var psk = "merlin"  // 定义全局变量 psk，存储预共享密钥
var proxy = ""  // 定义全局变量 proxy，存储代理信息
var secret []byte  // 定义全局变量 secret，存储字节切片
var verbose = false  // 定义全局变量 verbose，存储是否启用详细输出的标志
var debug = false  // 定义全局变量 debug，存储是否启用调试模式的标志
var merlinJWT string  // 定义全局变量 merlinJWT，存储 Merlin JWT
var host string  // 定义全局变量 host，存储主机信息
var ja3 = ""  // 定义全局变量 ja3，存储 JA3 指纹

type merlinClient interface {
    Do(req *http.Request) (*http.Response, error)  // 定义接口 merlinClient 包含 Do 方法
    Get(url string) (resp *http.Response, err error)  // 定义接口 merlinClient 包含 Get 方法
    Head(url string) (resp *http.Response, err error)  // 定义接口 merlinClient 包含 Head 方法
    Post(url, contentType string, body io.Reader) (resp *http.Response, err error)  // 定义接口 merlinClient 包含 Post 方法
}

func main() {
    flag.BoolVar(&verbose, "verbose", false, "Enable verbose output")  // 解析命令行参数，设置 verbose 变量
    # 定义一个布尔类型的命令行参数，用于启用调试输出
    flag.BoolVar(&debug, "debug", false, "Enable debug output")
    # 定义一个字符串类型的命令行参数，用于指定代理连接的完整 URL
    flag.StringVar(&url, "url", url, "Full URL for agent to connect to")
    # 定义一个字符串类型的命令行参数，用于指定用于加密初始通信的预共享密钥
    flag.StringVar(&psk, "psk", psk, "Pre-Shared Key used to encrypt initial communications")
    # 定义一个字符串类型的命令行参数，用于指定代理连接的协议
    protocol := flag.String("proto", "h2", "Protocol for the agent to connect with [https (HTTP/1.1), h2 (HTTP/2), hq (QUIC or HTTP/3.0)]")
    # 定义一个字符串类型的命令行参数，用于指定仅用于 http/1.1 流量的硬编码代理，将覆盖主机配置
    flag.StringVar(&proxy, "proxy", proxy, "Hardcoded proxy to use for http/1.1 traffic only that will override host configuration")
    # 定义一个字符串类型的命令行参数，用于指定 HTTP Host 头
    flag.StringVar(&host, "host", host, "HTTP Host header")
    # 定义一个字符串类型的命令行参数，用于指定 JA3 签名字符串（而不是 MD5 哈希），将覆盖 -proto 标志
    flag.StringVar(&ja3, "ja3", ja3, "JA3 signature string (not the MD5 hash). Overrides -proto flag")
    # 设置自定义的用法说明函数
    flag.Usage = usage
    # 解析命令行参数
    flag.Parse()

    var err error

    # 设置并运行代理
    a, errNew := agent.New(*protocol, url, host, psk, ja3, proxy, verbose, debug)
    if errNew != nil {
        message("warn", errNew.Error())
        os.Exit(1)
    }

    # 计算预共享密钥的 SHA256 哈希值，并将结果存储在 secret 变量中
    k := sha256.Sum256([]byte(psk))
    secret = k[:]

    # 设置初始 JWT
    merlinJWT, err = getJWT(a.ID)
    if err != nil {
        message("warn", err.Error())
        os.Exit(1)
    }

    # 检查是否为 v0.7.0 或更早版本
    message("info", fmt.Sprintf("Connecting to %s checking for Merlin server version v0.7.0.BETA or earlier", url))
    err = sendPre8Message(a)
    if err != nil {
        if verbose {
            message("warn", err.Error())
        }
        message("note", fmt.Sprintf("%s is not a Merlin server", url))
    } else {
        os.Exit(0)
    }

    # 向服务器发送消息，并查看是否收到可解密的 RegInit 消息
    message("info", fmt.Sprintf("Connecting to %s checking for Merlin server version v0.8.0.BETA or greater", url))
    err = opaqueRegister(a)
    if err != nil {
        if verbose {
            message("warn", err.Error())
        }
        message("note", fmt.Sprintf("%s is not a Merlin server using \"%s\" as a pre-shared key", url, psk))
    }
// opaqueAuthenticate is used to authenticate an agent leveraging the OPAQUE Password Authenticated Key Exchange (PAKE) protocol
func opaqueRegister(a agent.Agent) error {

    // Generate a random password and run it through 5000 iterations of PBKDF2
    x := core.RandStringBytesMaskImprSrc(30) // 生成一个30位的随机密码
    pwdU := pbkdf2.Key([]byte(x), a.ID.Bytes(), 5000, 32, sha256.New) // 使用PBKDF2算法对密码进行5000次迭代加密

    // 1 - Create a NewUserRegister
    userReg := gopaque.NewUserRegister(gopaque.CryptoDefault, a.ID.Bytes(), nil) // 创建一个新的用户注册对象
    userRegInit := userReg.Init(pwdU) // 使用加密后的密码初始化用户注册对象

    userRegInitBytes, errUserRegInitBytes := userRegInit.ToBytes() // 将用户注册初始化消息转换为字节流
    if errUserRegInitBytes != nil {
        return fmt.Errorf("there was an error marshalling the OPAQUE user registration initialization message to bytes:\r\n%s", errUserRegInitBytes.Error()) // 如果转换过程中出现错误，则返回错误信息
    }

    // message to be sent to the server
    regInitBase := messages.Base{ // 创建要发送到服务器的消息
        Version: 1.0,
        ID:      a.ID,
        Type:    "RegInit",
        Payload: userRegInitBytes,
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    var client merlinClient = *a.Client // 创建一个客户端对象

    regInitResp, errRegInitResp := sendMessage("POST", regInitBase, client) // 发送消息到服务器
    if errRegInitResp != nil {
        return fmt.Errorf("there was an error sending the agent OPAQUE user registration initialization message:\r\n%s", errRegInitResp.Error()) // 如果发送消息过程中出现错误，则返回错误信息
    }

    if regInitResp.Type != "RegInit" {
        return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user registration initialization", regInitResp.Type) // 如果收到的响应消息类型不正确，则返回错误信息
    }

    // If we get this far then it means we sent a message to the server encrypted with the correct psk and it responded
    message("success", fmt.Sprintf("Verified Merlin server v0.8.0.BETA or greater instance at %s", url)) // 发送成功消息

    if verbose {
        message("note", fmt.Sprintf("Decrypted Merlin message:\r\n%+v", regInitResp)) // 如果是详细模式，则打印解密后的消息
    }

    var serverRegInit gopaque.ServerRegisterInit // 创建服务器注册初始化对象
    // 从字节流中解析 OPAQUE 服务器注册初始化消息
    errServerRegInit := serverRegInit.FromBytes(gopaque.CryptoDefault, regInitResp.Payload.([]byte))
    // 如果解析出错，返回错误信息
    if errServerRegInit != nil {
        return fmt.Errorf("there was an error unmarshalling the OPAQUE server register initialization message from bytes:\r\n%s", errServerRegInit.Error())
    }

    // 如果设置了详细输出，打印 OPAQUE Alpha、Beta、V 和 PubS 的值
    if a.Verbose {
        message("info", fmt.Sprintf("OPAQUE Alpha:\t%s", userRegInit.Alpha))
        message("info", fmt.Sprintf("OPAQUE Beta:\t\t%s", serverRegInit.Beta))
        message("info", fmt.Sprintf("OPAQUE V:\t\t%s", serverRegInit.V))
        message("info", fmt.Sprintf("OPAQUE PubS:\t\t%s", serverRegInit.ServerPublicKey))
    }

    // 返回空值
    return nil
// sendPre8Message 函数用于发送早于版本8的消息给代理
func sendPre8Message(a agent.Agent) error {
    // 创建一个消息基类对象
    g := messages.Base{
        Version: 1.0,
        ID:      a.ID,
        Type:    "StatusCheckIn",
        Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
    }

    // 创建一个字节缓冲区
    b := new(bytes.Buffer)
    // 将消息基类对象编码成 JSON 格式并写入字节缓冲区
    errJ := json.NewEncoder(b).Encode(g)
    if errJ != nil {
        return fmt.Errorf("there was an error encoding the JSON message:\r\n%s", errJ.Error())
    }

    // 创建一个 HTTP 请求
    req, reqErr := http.NewRequest("POST", url, b)
    if reqErr != nil {
        return fmt.Errorf("there was an error creating a new HTTP request:\r\n%s", reqErr)
    }

    // 设置请求头部信息
    req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36 ")
    req.Header.Set("Content-Type", "application/json; charset=utf-8")

    // 创建一个 Merlin 客户端对象
    var client merlinClient = *a.Client

    // 发送 HTTP 请求并获取响应
    resp, err := client.Do(req)

    if err != nil {
        return fmt.Errorf("there was an error sending the request:\r\n%s", err.Error())
    }

    // 创建一个 JSON 原始消息对象
    var payload json.RawMessage
    j := messages.Base{
        Payload: &payload,
    }

    // 解码响应消息体并将其存入 JSON 消息对象
    errDecode := json.NewDecoder(resp.Body).Decode(&j)
    if errDecode != nil {
        return fmt.Errorf("there was an error decoding the message response into a JSON message:\r\n%s", errDecode.Error())
    }

    // 检查消息类型是否为 "AgentControl"
    if j.Type == "AgentControl" {
        message("success", fmt.Sprintf("Verified Merlin server v0.7.0.BETA or earlier instance at %s", url))
        if verbose {
            message("note", fmt.Sprintf("Merlin message:\r\n%+v", j))
        }
    } else {
        return fmt.Errorf("received JSON message did not contain an message type of AgentControl")
    }
    return nil
}

// sendMessage 是一个通用函数，用于接收一个 messages.Base 结构体，对其进行编码、加密，并发送到服务器
// 响应消息将被解密、解码，并返回一个 messages.Base 结构体
func sendMessage(method string, m messages.Base, client merlinClient) (messages.Base, error) {
    // 如果 debug 标志为真，则输出进入 agent.sendMessage 的调试信息
    if debug {
        message("debug", "Entering into agent.sendMessage")
    }
    // 如果 verbose 标志为真，则输出发送消息的详细信息
    if verbose {
        message("note", fmt.Sprintf("Sending %s message to %s", m.Type, url))
    }

    // 创建一个空的消息对象
    var returnMessage messages.Base

    // 将 messages.Base 转换为 gob 格式的字节流
    messageBytes := new(bytes.Buffer)
    errGobEncode := gob.NewEncoder(messageBytes).Encode(m)
    // 如果转换过程中出现错误，则返回错误信息
    if errGobEncode != nil {
        return returnMessage, fmt.Errorf("there was an error encoding the %s message to a gob:\r\n%s", m.Type, errGobEncode.Error())
    }

    // 获取 JWE
    jweString, errJWE := core.GetJWESymetric(messageBytes.Bytes(), secret)
    // 如果获取 JWE 过程中出现错误，则返回错误信息
    if errJWE != nil {
        return returnMessage, errJWE
    }

    // 将 JWE 编码为 gob 格式的字节流
    jweBytes := new(bytes.Buffer)
    errJWEBuffer := gob.NewEncoder(jweBytes).Encode(jweString)
    // 如果编码过程中出现错误，则返回错误信息
    if errJWEBuffer != nil {
        return returnMessage, fmt.Errorf("there was an error encoding the %s JWE string to a gob:\r\n%s", m.Type, errJWEBuffer.Error())
    }

    // 根据 method 的值进行不同的处理
    switch strings.ToLower(method) {
    // 如果 method 不在预期范围内，则返回错误信息
    default:
        return returnMessage, fmt.Errorf("%s is an invalid method for sending a message", method)
    }
// getJWT 用于在向服务器发送第一条消息时发送未经身份验证的 JWT
func getJWT(agentID uuid.UUID) (string, error) {
    // 创建加密器
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.DIRECT, // 不会创建每条消息的密钥
            Key:       secret},
        (&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
    if encErr != nil {
        return "", fmt.Errorf("创建 JWT 加密器时出错:\r\n%s", encErr.Error())
    }

    // 创建签名者
    signer, errSigner := jose.NewSigner(jose.SigningKey{
        Algorithm: jose.HS256,
        Key:       secret},
        (&jose.SignerOptions{}).WithType("JWT"))
    if errSigner != nil {
        return "", fmt.Errorf("创建 JWT 签名者时出错:\r\n%s", errSigner.Error())
    }

    // 构建 JWT 声明
    cl := jwt.Claims{
        IssuedAt: jwt.NewNumericDate(time.Now().UTC()),
        ID:       agentID.String(),
    }

    agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
    if err != nil {
        return "", fmt.Errorf("序列化 JWT 时出错:\r\n%s", err)
    }

    // 解析以检查错误
    _, errParse := jwt.ParseSignedAndEncrypted(agentJWT)
    if errParse != nil {
        return "", fmt.Errorf("解析加密的 JWT 时出错:\r\n%s", errParse.Error())
    }

    return agentJWT, nil
}

// message 用于在命令行打印消息
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
        color.Red("[_-_]无效的消息级别: " + message)
    }
}

// usage 打印命令行选项
# 定义一个名为 usage 的函数，用于打印程序名称和参数的默认值，并退出程序
func usage() {
    # 打印程序名称
    fmt.Printf("Merlin PRISM\r\n")
    # 打印参数的默认值
    flag.PrintDefaults()
    # 退出程序
    os.Exit(0)
}
```