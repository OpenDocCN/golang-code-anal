# `kubesploit\cmd\prism\main.go`

```
/*
Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
这个文件是Kubesploit的一部分。
版权所有©2021 CyberArk Software Ltd。

Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

Kubesploit的分发希望能够有助于增强组织的安全性。
Kubesploit不得以任何恶意方式使用。
Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅GNU通用公共许可证以获取更多详细信息。

您应该已经收到了GNU通用公共许可证的副本。
如果没有，请参阅<http://www.gnu.org/licenses/>。
*/
package main

import (
	// Standard
	"bytes"  // 导入处理字节的包
	"crypto/sha256"  // 导入 SHA256 加密算法包
	"encoding/gob"  // 导入 Gob 编码/解码包
	"encoding/json"  // 导入 JSON 编码/解码包
	"flag"  // 导入命令行参数解析包
	"fmt"  // 导入格式化输出包
	"io"  // 导入输入输出包
	"net/http"  // 导入 HTTP 包
	"os"  // 导入操作系统功能包
	"strings"  // 导入字符串处理包
	"time"  // 导入时间包

	// 3rd Party
	"github.com/cretz/gopaque/gopaque"  // 导入第三方包
	"github.com/fatih/color"  // 导入第三方包
	"github.com/satori/go.uuid"  // 导入第三方包
)
// 导入加密库中的 pbkdf2 模块
"golang.org/x/crypto/pbkdf2"
// 导入 go-jose.v2 库
"gopkg.in/square/go-jose.v2"
// 导入 go-jose.v2 库中的 jwt 模块
"gopkg.in/square/go-jose.v2/jwt"

// 导入自定义包 agent
"kubesploit/pkg/agent"
// 导入自定义包 core
"kubesploit/pkg/core"
// 导入自定义包 messages
"kubesploit/pkg/messages"
)

// 全局变量
// 定义 URL 地址
var url = "https://127.0.0.1:443"
// 定义预共享密钥
var psk = "merlin"
// 定义代理
var proxy = ""
// 定义密钥的字节流
var secret []byte
// 定义是否显示详细信息的标志
var verbose = false
// 定义是否显示调试信息的标志
var debug = false
// 定义 Merlin 的 JWT 字符串
var merlinJWT string
// 定义主机地址
var host string
// 定义 JA3 字符串
var ja3 = ""
# 定义一个接口类型merlinClient，包含了Do、Get、Head、Post四个方法
type merlinClient interface {
	Do(req *http.Request) (*http.Response, error)  # 执行HTTP请求
	Get(url string) (resp *http.Response, err error)  # 发送GET请求
	Head(url string) (resp *http.Response, err error)  # 发送HEAD请求
	Post(url, contentType string, body io.Reader) (resp *http.Response, err error)  # 发送POST请求
}

# 主函数
func main() {
	# 定义命令行参数
	flag.BoolVar(&verbose, "verbose", false, "Enable verbose output")  # 定义是否启用详细输出的命令行参数
	flag.BoolVar(&debug, "debug", false, "Enable debug output")  # 定义是否启用调试输出的命令行参数
	flag.StringVar(&url, "url", url, "Full URL for agent to connect to")  # 定义代理连接的完整URL的命令行参数
	flag.StringVar(&psk, "psk", psk, "Pre-Shared Key used to encrypt initial communications")  # 定义用于加密初始通信的预共享密钥的命令行参数
	protocol := flag.String("proto", "h2", "Protocol for the agent to connect with [https (HTTP/1.1), h2 (HTTP/2), hq (QUIC or HTTP/3.0)]")  # 定义代理连接的协议的命令行参数
	flag.StringVar(&proxy, "proxy", proxy, "Hardcoded proxy to use for http/1.1 traffic only that will override host configuration")  # 定义用于仅http/1.1流量的硬编码代理的命令行参数，将覆盖主机配置
	flag.StringVar(&host, "host", host, "HTTP Host header")  # 定义HTTP Host头的命令行参数
	flag.StringVar(&ja3, "ja3", ja3, "JA3 signature string (not the MD5 hash). Overrides -proto flag")  # 定义JA3签名字符串的命令行参数（不是MD5哈希），覆盖-proto标志
	flag.Usage = usage  # 设置自定义的用法函数
	flag.Parse()  # 解析命令行参数
}
// 声明一个错误变量
var err error

// 设置并运行代理
// 使用 agent.New 函数创建一个代理对象，并检查是否有错误发生
a, errNew := agent.New(*protocol, url, host, psk, ja3, proxy, verbose, debug)
if errNew != nil {
    // 如果有错误发生，输出警告信息并退出程序
    message("warn", errNew.Error())
    os.Exit(1)
}

// 计算 PSK 的 SHA256 哈希值作为密钥
k := sha256.Sum256([]byte(psk))
secret = k[:]

// 设置初始的 JWT
// 调用 getJWT 函数获取 JWT，并检查是否有错误发生
merlinJWT, err = getJWT(a.ID)
if err != nil {
    // 如果有错误发生，输出警告信息并退出程序
    message("warn", err.Error())
    os.Exit(1)
}

// 检查是否为 v0.7.0 或更早的版本
// 发送信息到服务器，检查是否为 Merlin 服务器版本 v0.7.0.BETA 或更早
message("info", fmt.Sprintf("Connecting to %s checking for Merlin server version v0.7.0.BETA or earlier", url))
// 发送预 8 版本消息
err = sendPre8Message(a)
// 如果出现错误
if err != nil {
    // 如果设置了详细模式
    if verbose {
        // 输出警告信息
        message("warn", err.Error())
    }
    // 输出提示信息，说明不是 Merlin 服务器
    message("note", fmt.Sprintf("%s is not a Merlin server", url))
} else {
    // 如果没有错误，退出程序
    os.Exit(0)
}

// 发送消息到服务器，检查是否为 Merlin 服务器版本 v0.8.0.BETA 或更高
message("info", fmt.Sprintf("Connecting to %s checking for Merlin server version v0.8.0.BETA or greater", url))
// 发送不透明注册消息
err = opaqueRegister(a)
// 如果出现错误
if err != nil {
    // 如果设置了详细模式
    if verbose {
        // 输出警告信息
        message("warn", err.Error())
    }
    // 输出提示信息，说明不是 Merlin 服务器，使用预共享密钥
    message("note", fmt.Sprintf("%s is not a Merlin server using \"%s\" as a pre-shared key", url, psk))
}
}

// opaqueAuthenticate用于使用OPAQUE密码认证密钥交换（PAKE）协议对代理进行身份验证
func opaqueRegister(a agent.Agent) error {

	// 生成一个随机密码，并通过5000次PBKDF2迭代
	x := core.RandStringBytesMaskImprSrc(30)
	pwdU := pbkdf2.Key([]byte(x), a.ID.Bytes(), 5000, 32, sha256.New)

	// 1 - 创建一个NewUserRegister
	userReg := gopaque.NewUserRegister(gopaque.CryptoDefault, a.ID.Bytes(), nil)
	userRegInit := userReg.Init(pwdU)

	userRegInitBytes, errUserRegInitBytes := userRegInit.ToBytes()
	if errUserRegInitBytes != nil {
		return fmt.Errorf("将OPAQUE用户注册初始化消息编组为字节时出错：\r\n%s", errUserRegInitBytes.Error())
	}

	// 要发送到服务器的消息
	regInitBase := messages.Base{
// 定义一个结构体，包含版本、ID、类型、载荷和填充字段
Version: 1.0,
ID:      a.ID,
Type:    "RegInit",
Payload: userRegInitBytes,
Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),

// 将a.Client转换为merlinClient类型
var client merlinClient = *a.Client

// 发送POST请求，发送regInitBase地址和client信息，接收regInitResp和errRegInitResp
regInitResp, errRegInitResp := sendMessage("POST", regInitBase, client)

// 如果发送消息出现错误，则返回错误信息
if errRegInitResp != nil {
    return fmt.Errorf("there was an error sending the agent OPAQUE user registration initialization message:\r\n%s", errRegInitResp.Error())
}

// 如果接收到的消息类型不是"RegInit"，则返回错误信息
if regInitResp.Type != "RegInit" {
    return fmt.Errorf("invalid message type %s in resopnse to OPAQUE user registration initialization", regInitResp.Type)
}

// 如果程序执行到这一步，表示已经成功使用正确的psk加密发送消息到服务器，并且收到了响应
# 在消息中输出成功信息，格式化输出 Merlin 服务器版本号
message("success", fmt.Sprintf("Verified Merlin server v0.8.0.BETA or greater instance at %s", url))

# 如果 verbose 为真，则在消息中输出解密后的 Merlin 消息
if verbose:
    message("note", fmt.Sprintf("Decrypted Merlin message:\r\n%+v", regInitResp))

# 创建一个名为 serverRegInit 的变量，类型为 gopaque.ServerRegisterInit
var serverRegInit gopaque.ServerRegisterInit

# 将 regInitResp.Payload 转换为字节数组，并解析到 serverRegInit 中
errServerRegInit := serverRegInit.FromBytes(gopaque.CryptoDefault, regInitResp.Payload.([]byte))
if errServerRegInit != nil:
    return fmt.Errorf("there was an error unmarshalling the OPAQUE server register initialization message from bytes:\r\n%s", errServerRegInit.Error())

# 如果 a.Verbose 为真，则在消息中输出 OPAQUE Alpha、Beta、V 和 PubS 的值
if a.Verbose:
    message("info", fmt.Sprintf("OPAQUE Alpha:\t%s", userRegInit.Alpha))
    message("info", fmt.Sprintf("OPAQUE Beta:\t\t%s", serverRegInit.Beta))
    message("info", fmt.Sprintf("OPAQUE V:\t\t%s", serverRegInit.V))
    message("info", fmt.Sprintf("OPAQUE PubS:\t\t%s", serverRegInit.ServerPublicKey))
	// 返回空值
	return nil
}

// 发送预先8版本的消息
func sendPre8Message(a agent.Agent) error {
	// 创建消息对象
	g := messages.Base{
		Version: 1.0,
		ID:      a.ID,
		Type:    "StatusCheckIn",
		Padding: core.RandStringBytesMaskImprSrc(a.PaddingMax),
	}

	// 创建字节缓冲区
	b := new(bytes.Buffer)
	// 编码消息对象为 JSON 格式并写入缓冲区
	errJ := json.NewEncoder(b).Encode(g)
	// 检查是否有错误
	if errJ != nil {
		// 返回 JSON 编码错误
		return fmt.Errorf("there was an error encoding the JSON message:\r\n%s", errJ.Error())
	}

	// 创建 HTTP 请求
	req, reqErr := http.NewRequest("POST", url, b)
	// 检查是否有错误
	if reqErr != nil {
		// 返回 HTTP 请求创建错误
		return fmt.Errorf("there was an error creating a new HTTP request:\r\n%s", reqErr)
	}

	// 设置请求头中的 User-Agent
	req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36 ")
	// 设置请求头中的 Content-Type
	req.Header.Set("Content-Type", "application/json; charset=utf-8")

	// 发起 HTTP 请求
	var client merlinClient = *a.Client
	resp, err := client.Do(req)

	// 如果发起请求时出现错误，返回错误信息
	if err != nil {
		return fmt.Errorf("there was an error sending the request:\r\n%s", err.Error())
	}

	// 解析响应中的 JSON 数据
	var payload json.RawMessage
	j := messages.Base{
		Payload: &payload,
	}

	errDecode := json.NewDecoder(resp.Body).Decode(&j)
	// 如果解析 JSON 数据时出现错误，处理错误
	if errDecode != nil {
		// 返回一个格式化的错误信息，指示解码消息响应为 JSON 消息时出错
		return fmt.Errorf("there was an error decoding the message response into a JSON message:\r\n%s", errDecode.Error())
	}

	// 如果消息类型为 "AgentControl"
	if j.Type == "AgentControl" {
		// 输出成功消息，指示验证了 Merlin 服务器 v0.7.0.BETA 或更早版本的实例
		message("success", fmt.Sprintf("Verified Merlin server v0.7.0.BETA or earlier instance at %s", url))
		// 如果 verbose 为真，输出附加信息
		if verbose {
			message("note", fmt.Sprintf("Merlin message:\r\n%+v", j))
		}
	} else {
		// 返回一个格式化的错误信息，指示接收到的 JSON 消息不包含 AgentControl 类型的消息
		return fmt.Errorf("received JSON message did not contain an message type of AgentControl")
	}
	// 返回空错误，表示没有错误发生
	return nil
}

// sendMessage 是一个通用函数，用于接收一个 messages.Base 结构体，对其进行编码、加密，并发送到服务器
// 响应消息将被解密、解码，并返回一个 messages.Base 结构体
func sendMessage(method string, m messages.Base, client merlinClient) (messages.Base, error) {
	// 如果 debug 为真，输出调试信息
	if debug {
		message("debug", "Entering into agent.sendMessage")
	}
```

// 如果 verbose 为 true，则打印发送消息的信息
if verbose {
    message("note", fmt.Sprintf("Sending %s message to %s", m.Type, url))
}

// 创建一个空的 messages.Base 类型变量 returnMessage
var returnMessage messages.Base

// 将 messages.Base 类型转换为 gob 编码
messageBytes := new(bytes.Buffer)
errGobEncode := gob.NewEncoder(messageBytes).Encode(m)
if errGobEncode != nil {
    // 如果编码出错，则返回错误信息
    return returnMessage, fmt.Errorf("there was an error encoding the %s message to a gob:\r\n%s", m.Type, errGobEncode.Error())
}

// 获取 JWE
jweString, errJWE := core.GetJWESymetric(messageBytes.Bytes(), secret)
if errJWE != nil {
    // 如果获取 JWE 出错，则返回错误信息
    return returnMessage, errJWE
}

// 将 JWE 编码为 gob
	// 创建一个字节流缓冲区，用于存储 JWE 字符串的编码结果
	jweBytes := new(bytes.Buffer)
	// 使用 gob 编码器将 JWE 字符串编码到字节流缓冲区中
	errJWEBuffer := gob.NewEncoder(jweBytes).Encode(jweString)
	// 如果编码过程中出现错误，则返回错误信息
	if errJWEBuffer != nil {
		return returnMessage, fmt.Errorf("there was an error encoding the %s JWE string to a gob:\r\n%s", m.Type, errJWEBuffer.Error())
	}

	// 根据请求方法构建 HTTP 请求
	switch strings.ToLower(method) {
	case "post":
		// 创建一个 POST 请求
		req, reqErr := http.NewRequest("POST", url, jweBytes)
		// 如果构建请求过程中出现错误，则返回错误信息
		if reqErr != nil {
			return returnMessage, fmt.Errorf("there was an error building the HTTP request:\r\n%s", reqErr.Error())
		}

		// 如果请求对象不为空，则设置请求头信息
		if req != nil {
			req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36 ")
			req.Header.Set("Content-Type", "application/octet-stream; charset=utf-8")
			req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", merlinJWT))
		}

		// 发送请求
		// 使用 HTTP 客户端发送请求，并接收响应
		resp, err := client.Do(req)
		// 如果发生错误，返回错误信息
		if err != nil {
			return returnMessage, fmt.Errorf("there was an error with the HTTP client while performing a POST:\r\n%s", err.Error())
		}
		// 如果开启了调试模式，打印 HTTP 响应信息
		if debug {
			message("debug", fmt.Sprintf("HTTP Response:\r\n%+v", resp))
		}
		// 如果响应状态码不是 200，返回错误信息
		if resp.StatusCode != 200 {
			return returnMessage, fmt.Errorf("there was an error communicating with the server:\r\n%d", resp.StatusCode)
		}

		// 获取响应的 Content-Type 头部信息
		contentType := resp.Header.Get("Content-Type")
		// 如果没有 Content-Type 头部信息，返回错误信息
		if contentType == "" {
			return returnMessage, fmt.Errorf("the response did not contain a Content-Type header")
		}

		// 检查响应是否包含 application/octet-stream Content-Type 头部信息
		isOctet := false
		for _, v := range strings.Split(contentType, ",") {
			if strings.ToLower(v) == "application/octet-stream" {
		// 设置一个变量来标记是否为八进制数据
		isOctet = true
	}

}

// 如果不是八进制数据，则返回错误信息
if !isOctet {
	return returnMessage, fmt.Errorf("the response message did not contain the application/octet-stream Content-Type header")
}

// 检查响应消息是否包含数据
if resp.ContentLength == 0 {
	return returnMessage, fmt.Errorf("the response message did not contain any data")
}

var jweString string

// 将服务器响应中的GOB解码为JWE
errD := gob.NewDecoder(resp.Body).Decode(&jweString)
if errD != nil {
	return returnMessage, fmt.Errorf("there was an error decoding the gob message:\r\n%s", errD.Error())
}
// 解密 JWE 为 messages.Base
respMessage, errDecrypt := core.DecryptJWE(jweString, secret)
// 如果解密出错，返回错误
if errDecrypt != nil {
    return returnMessage, errDecrypt
}
// 返回解密后的消息
return respMessage, nil
// 默认情况下，返回一个错误，说明发送消息的方法无效
default:
    return returnMessage, fmt.Errorf("%s is an invalid method for sending a message", method)
}

// getJWT 用于在向服务器发送第一条消息时发送一个未经身份验证的 JWT
func getJWT(agentID uuid.UUID) (string, error) {
    // 创建加密器
    encrypter, encErr := jose.NewEncrypter(jose.A256GCM,
        jose.Recipient{
            Algorithm: jose.DIRECT, // 不创建每条消息的密钥
```

	// 创建加密器
	enc, encErr := jose.NewEncrypter(jose.ContentEncryption("A128GCM"), jose.Recipient{Algorithm: jose.A128GCM, Key: secret}, (&jose.EncrypterOptions{}).WithType("JWT").WithContentType("JWT"))
	if encErr != nil {
		return "", fmt.Errorf("there was an error creating the JWT encryptor:\r\n%s", encErr.Error())
	}

	// 创建签名器
	signer, errSigner := jose.NewSigner(jose.SigningKey{Algorithm: jose.HS256, Key: secret}, (&jose.SignerOptions{}).WithType("JWT"))
	if errSigner != nil {
		return "", fmt.Errorf("there was an error creating the JWT signer:\r\n%s", errSigner.Error())
	}

	// 构建 JWT 声明
	cl := jwt.Claims{
		IssuedAt: jwt.NewNumericDate(time.Now().UTC()), // 设置签发时间为当前时间
		ID:       agentID.String(), // 设置 ID 为 agentID 的字符串表示
	}
```
// 使用签名者和加密者对声明进行签名和加密，返回JWT字符串和可能的错误
agentJWT, err := jwt.SignedAndEncrypted(signer, encrypter).Claims(cl).CompactSerialize()
if err != nil {
    return "", fmt.Errorf("there was an error serializing the JWT:\r\n%s", err)
}

// 解析JWT以检查是否有错误
_, errParse := jwt.ParseSignedAndEncrypted(agentJWT)
if errParse != nil {
    return "", fmt.Errorf("there was an error parsing the encrypted JWT:\r\n%s", errParse.Error())
}

// 返回JWT字符串和nil，表示没有错误
return agentJWT, nil
}

// message 用于在命令行打印消息
func message(level string, message string) {
    switch level {
    case "info":
        // 打印信息级别的消息
        color.Cyan("[i]" + message)
// 根据消息级别打印不同颜色的消息
switch level {
    // 如果消息级别为 "note"，则打印黄色的消息
    case "note":
        color.Yellow("[-]" + message)
    // 如果消息级别为 "warn"，则打印红色的消息
    case "warn":
        color.Red("[!]" + message)
    // 如果消息级别为 "debug"，则打印红色的消息
    case "debug":
        color.Red("[DEBUG]" + message)
    // 如果消息级别为 "success"，则打印绿色的消息
    case "success":
        color.Green("[+]" + message)
    // 如果消息级别不在以上四种情况，则打印红色的消息，提示消息级别无效
    default:
        color.Red("[_-_]Invalid message level: " + message)
    }
}

// usage 函数用于打印命令行选项的使用方法
func usage() {
    // 打印程序名称
    fmt.Printf("Merlin PRISM\r\n")
    // 打印命令行选项的默认值
    flag.PrintDefaults()
    // 退出程序
    os.Exit(0)
}
```