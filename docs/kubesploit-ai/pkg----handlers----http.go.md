# `kubesploit\pkg\handlers\http.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其中包括许可证的第3版或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包处理程序
package handlers
# 导入标准库
import (
	"bytes  # 用于操作字节的标准库
	"crypto/sha256  # 提供 SHA-256 哈希算法的标准库
	"encoding/gob  # 用于编码和解码数据结构的标准库
	"fmt  # 提供格式化输入输出的标准库
	"io/ioutil  # 用于读取文件内容的标准库
	"net/http  # 提供 HTTP 客户端和服务端的标准库
	"strings  # 提供字符串操作的标准库
	"time  # 提供时间操作的标准库

	# 第三方库
	uuid "github.com/satori/go.uuid"  # 用于生成 UUID 的第三方库
	"go.dedis.ch/kyber/v3"  # 提供密码学算法的第三方库

	# Merlin
	"kubesploit/pkg/agents"  # 导入自定义的 agents 模块
	messageAPI "kubesploit/pkg/api/messages"  # 导入自定义的 messages 模块
	"kubesploit/pkg/core"  # 导入自定义的 core 模块
	"kubesploit/pkg/logging"  # 导入自定义的 logging 模块
)
// 导入所需的包
import (
	"kubesploit/pkg/messages"
	"kubesploit/pkg/util"
)

// HTTPContext 包含有关处理程序的上下文信息，例如用于加密/解密的密钥
type HTTPContext struct {
	Context
	PSK       string       // 在密码认证密钥交换（PAKE）之前使用的预共享密钥密码
	JWTKey    []byte       // 服务器用于创建JWT的密码
	OpaqueKey kyber.Scalar // OPAQUE服务器的密钥
}

// AgentHTTP 函数负责所有 Merlin 代理流量
func (ctx *HTTPContext) AgentHTTP(w http.ResponseWriter, r *http.Request) {
	// 如果设置了详细模式，则记录接收到的连接信息
	if core.Verbose {
		m := fmt.Sprintf("Received %s %s connection from %s", r.Proto, r.Method, r.RemoteAddr)
		message("warn", m) // 发送警告消息
		logging.Server(m) // 记录服务器日志
	}
# 如果调试模式开启
if core.Debug:
    # 创建一个空字符串用于存储 HTTP 连接的详细信息
    var m string
    # 添加 HTTP 连接的详细信息到字符串中
    m += "HTTP Connection Details:\r\n"
    m += fmt.Sprintf("Host: %s\r\n", r.Host)
    m += fmt.Sprintf("URI: %s\r\n", r.RequestURI)
    m += fmt.Sprintf("Method: %s\r\n", r.Method)
    m += fmt.Sprintf("Protocol: %s\r\n", r.Proto)
    m += fmt.Sprintf("Headers: %s\r\n", r.Header)
    # 如果存在 TLS 连接
    if r.TLS != nil:
        m += fmt.Sprintf("TLS Negotiated Protocol: %s\r\n", r.TLS.NegotiatedProtocol)
        m += fmt.Sprintf("TLS Cipher Suite: %d\r\n", r.TLS.CipherSuite)
        m += fmt.Sprintf("TLS Server Name: %s\r\n", r.TLS.ServerName)
    m += fmt.Sprintf("Content Length: %d", r.ContentLength)

    # 输出调试信息
    message("debug", m)
    # 记录服务器日志
    logging.Server(m)

# 检查 Merlin PRISM 活动
	// 检查用户代理是否为特定值，如果是则记录警告信息并进行日志记录
	if r.UserAgent() == "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36 " {
		m := fmt.Sprintf("Someone from %s is attempting to fingerprint this Merlin server", r.RemoteAddr)
		message("warn", m)
		logging.Server(m)
	}

	// 确保请求头中包含 JWT
	token := r.Header.Get("Authorization")
	if token == "" {
		// 如果请求头中不包含 Authorization 头，则记录警告信息并进行日志记录
		if core.Verbose {
			m := "incoming request did not contain an Authorization header"
			message("warn", m)
			logging.Server(m)
		}
		// 返回 404 状态码
		w.WriteHeader(404)
		return
	}

	// 确保 Authorization 头包含 bearer token
	if !strings.Contains(token, "Bearer eyJ") {
		// 如果 Authorization 头不包含 bearer token，则...
// 如果设置了Verbose标志，打印警告消息并记录到日志中
if core.Verbose {
    m := "incoming request did not contain a Bearer token"
    message("warn", m)
    logging.Server(m)
}
// 返回状态码404
w.WriteHeader(404)
// 结束函数执行
return

// 确保内容类型为：application/octet-stream; charset=utf-8
if r.Header.Get("Content-Type") != "application/octet-stream; charset=utf-8" {
    // 如果设置了Verbose标志，打印警告消息并记录到日志中
    if core.Verbose {
        m := "incoming request did not contain a Content-Type header of: application/octet-stream; charset=utf-8"
        message("warn", m)
        logging.Server(m)
    }
    // 返回状态码404
    w.WriteHeader(404)
    // 结束函数执行
    return
}
# 如果请求方法为 POST
if r.Method == http.MethodPost:

    # 定义返回消息和错误变量
    var returnMessage messages.Base
    var err error
    var key []byte

    # 读取请求消息直到 EOF
    requestBytes, errRequestBytes := ioutil.ReadAll(r.Body)
    if errRequestBytes != nil:
        # 如果读取请求消息出错，记录错误信息并返回
        m := fmt.Sprintf("There was an error reading a POST message sent by an "+
            "agent:\r\n%s", errRequestBytes)
        message("warn", m)
        return

    # 解码 gob 格式的数据为 JWE 字符串
    var jweString string
    errDecode := gob.NewDecoder(bytes.NewReader(requestBytes)).Decode(&jweString)
    if errDecode != nil:
        # 如果解码出错，记录错误信息
        message("warn", fmt.Sprintf("there was an error decoding JWE payload message sent by an agent:\r\n%s", errDecode.Error()))
// 如果没有返回值，直接返回
return
}

// 验证 JWT 并获取声明
var agentID uuid.UUID
var errValidate error

// 设置返回头部
//w.Header().Set("Content-Type", "application/octet-stream")

// 使用 HTTP 接口 JWT 密钥验证 JWT；由服务器提供给经过身份验证的代理
agentID, errValidate = util.ValidateJWT(strings.Split(token, " ")[1], ctx.JWTKey)
// 如果返回了 agentID，则消息包含了使用 HTTP 接口密钥加密的 JWT
if (errValidate != nil) && (agentID == uuid.Nil) { // 未经身份验证的代理
if core.Verbose {
message("warn", errValidate.Error())
message("note", "trying again with interface PSK")
}
// 使用接口 PSK 验证 JWT；未经身份验证的代理使用
hashedKey := sha256.Sum256([]byte(ctx.PSK))
			// 复制 hashedKey 到 key
			key = hashedKey[:]
			// 使用 util 包中的 ValidateJWT 函数验证 JWT，并返回代理ID和验证错误
			agentID, errValidate = util.ValidateJWT(strings.Split(token, " ")[1], key)
			// 如果验证错误不为空
			if errValidate != nil {
				// 检查请求是否匹配可能是孤立代理的流量
				// 孤立代理将使用服务器的JWT密钥加密的JWT，而不是PSK
				if len(token) == 402 && r.ContentLength > 409 {
					// 记录检测到的孤立代理请求，并指示代理进行OPAQUE身份验证
					m := fmt.Sprintf("Orphaned agent request detected from %s, instructing agent to OPAQUE authenticate", r.RemoteAddr)
					message("note", m)
					logging.Server(m)
					// 返回401状态码
					w.WriteHeader(401)
					return
				}
				// 如果核心设置为详细模式
				if core.Verbose {
					// 记录验证错误
					message("warn", errValidate.Error())
				}
				// 返回404状态码
				w.WriteHeader(404)
				return
			}
			// 如果核心设置为调试模式
			if core.Debug {
				// 记录未经身份验证的JWT
				message("info", "Unauthenticated JWT")
			}
			}

			// 使用接口 PSK 解密 HTTP 负载，即 JWE
			k, errDecryptPSK := util.DecryptJWE(jweString, key)
			// 成功使用接口 PSK 解密 JWE
			if errDecryptPSK == nil {
				// 如果调试模式开启，打印 POST 数据
				if core.Debug {
					message("debug", fmt.Sprintf("[DEBUG]POST DATA: %v", k))
				}
				// 如果详细模式开启，打印接收到的消息类型和使用接口 PSK 解密 JWE
				if core.Verbose {
					message("note", fmt.Sprintf("Received %s message, decrypted JWE with interface PSK", k.Type))
				}

				messagePayloadBytes := new(bytes.Buffer)

				// 根据消息类型执行相应操作，允许使用 PSK 签名的 JWT 和 PSK 加密的 JWT
				switch k.Type {
				case "AuthInit":
					// 执行 OPAQUEAuthenticateInit 函数进行服务器认证初始化
					serverAuthInit, err := agents.OPAQUEAuthenticateInit(k)
					if err != nil {
# 记录错误信息到日志服务器
logging.Server(err.Error())
# 发送警告消息
message("warn", err.Error())
# 设置 HTTP 响应状态码为 404
w.WriteHeader(404)
# 返回
return
# 如果 serverAuthInit.Type 为 "ReRegister"，则记录消息并发送通知
if serverAuthInit.Type == "ReRegister" {
    m := fmt.Sprintf("Un-Registered agent %s sent OPAQUE authentication, instructing agent to OPAQUE register", agentID)
    message("note", m)
    logging.Server(m)
} else {
    # 否则记录消息
    logging.Server(fmt.Sprintf("Received new agent OPAQUE authentication from %s", agentID))
}

# 将返回消息编码为 gob 格式
errAuthInit := gob.NewEncoder(messagePayloadBytes).Encode(serverAuthInit)
# 如果编码过程中出现错误，记录错误信息到日志服务器并发送警告消息，设置 HTTP 响应状态码为 404
if errAuthInit != nil {
    m := fmt.Sprintf("there was an error encoding the return message into a gob:\r\n%s", errAuthInit.Error())
    logging.Server(m)
    message("warn", m)
    w.WriteHeader(404)
}
# 返回空值
return
# 如果消息类型是 "RegInit"，则执行以下操作
case "RegInit":
    # 调用 agents 包中的 OPAQUERegistrationInit 函数，初始化 agent 的 OPAQUE 用户注册
    serverRegInit, err := agents.OPAQUERegistrationInit(k, ctx.OpaqueKey)
    # 如果出现错误，记录日志并返回错误消息
    if err != nil:
        logging.Server(err.Error())
        message("warn", err.Error())
        w.WriteHeader(404)
        return
    # 记录日志，表示接收到来自 agentID 的新 agent OPAQUE 用户注册初始化
    logging.Server(fmt.Sprintf("Received new agent OPAQUE user registration initialization from %s", agentID))
    # 将返回消息编码为 gob 格式
    errRegInit := gob.NewEncoder(messagePayloadBytes).Encode(serverRegInit)
    # 如果编码过程出现错误，记录日志并返回错误消息
    if errRegInit != nil:
        m := fmt.Sprintf("there was an error encoding the return message into a gob:\r\n%s", errRegInit.Error())
        logging.Server(m)
        message("warn", m)
        w.WriteHeader(404)
        return
				// 如果消息类型是 "RegComplete"，则处理 OPAQUE 注册完成的情况
				case "RegComplete":
					// 调用 agents.OPAQUERegistrationComplete 函数处理 OPAQUE 注册完成的情况
					serverRegComplete, err := agents.OPAQUERegistrationComplete(k)
					// 如果出现错误，记录日志并返回错误消息
					if err != nil {
						logging.Server(err.Error())
						message("warn", err.Error())
						w.WriteHeader(404)
						return
					}
					// 记录日志，表示接收到新的 OPAQUE 用户注册完成的消息
					logging.Server(fmt.Sprintf("Received new agent OPAQUE user registration complete from %s", agentID))

					// 将返回消息编码为 gob 格式
					errRegInit := gob.NewEncoder(messagePayloadBytes).Encode(serverRegComplete)
					// 如果编码过程中出现错误，记录日志并返回错误消息
					if errRegInit != nil {
						m := fmt.Sprintf("there was an error encoding the return message into a gob:\r\n%s", errRegInit.Error())
						logging.Server(m)
						message("warn", m)
						w.WriteHeader(404)
						return
					}
				// 默认情况下，如果 JWT 未经身份验证，则打印警告消息并返回 404
				default:
					message("warn", fmt.Sprintf("invalid message type: %s for unauthenticated JWT", k.Type))
					w.WriteHeader(404)
					return
				}
				// 获取 JWE
				jwe, errJWE := core.GetJWESymetric(messagePayloadBytes.Bytes(), key)
				// 如果获取 JWE 出现错误，则记录日志并返回 404
				if errJWE != nil {
					logging.Server(errJWE.Error())
					message("warn", errJWE.Error())
					w.WriteHeader(404)
					return
				}

				// 设置返回头信息
				w.Header().Set("Content-Type", "application/octet-stream")

				// 将 JWE 编码为 gob 格式
				errJWEBuffer := gob.NewEncoder(w).Encode(jwe)
				// 如果编码出现错误，则...
				if errJWEBuffer != nil {
// 创建一个格式化的错误信息，描述写入 HTTP 流的响应消息时发生的错误
m := fmt.Errorf("there was an error writing the %s response message to the HTTP stream:\r\n%s", k.Type, errJWEBuffer.Error())
// 记录错误日志
logging.Server(m.Error())
// 记录警告消息
message("warn", m.Error())
// 设置 HTTP 响应状态码为 404
w.WriteHeader(404)
// 结束函数执行
return

// 如果 core.Verbose 为真，则记录一条消息
if core.Verbose {
    message("note", "Unauthenticated JWT w/ Authenticated JWE agent session key")
}
// 使用代理会话密钥解密 HTTP 负载（JWE）
j, errDecrypt := util.DecryptJWE(jweString, agents.GetEncryptionKey(agentID))
// 如果解密过程中发生错误，则记录警告消息，设置 HTTP 响应状态码为 404，并结束函数执行
if errDecrypt != nil {
    message("warn", errDecrypt.Error())
    w.WriteHeader(404)
    return
}
// 如果调试模式开启，打印 POST 数据的调试信息
if core.Debug {
    message("debug", fmt.Sprintf("[DEBUG]POST DATA: %v", j))
}
// 如果详细模式开启，打印接收到的消息类型、ID和时间信息
if core.Verbose {
    message("info", fmt.Sprintf("Received %s message from %s at %s", j.Type, j.ID, time.Now().UTC().Format(time.RFC3339)))
}

// 根据消息类型进行不同的处理
// 如果消息类型为 "AuthComplete"，调用 agents.OPAQUEAuthenticateComplete 处理消息，并记录日志
switch j.Type {
case "AuthComplete":
    returnMessage, err = agents.OPAQUEAuthenticateComplete(j)
    if err != nil {
        logging.Server(fmt.Sprintf("Received new agent OPAQUE authentication from %s", agentID))
    }
    m := fmt.Sprintf("New authenticated agent checkin for %s from %s at %s", j.ID.String(), r.RemoteAddr, time.Now().UTC().Format(time.RFC3339))
    message("success", m)
    logging.Server(m)
default:
    // 如果消息类型不匹配，打印警告信息，并返回 404 状态码
    message("warn", fmt.Sprintf("Invalid Activity: %s", j.Type))
    w.WriteHeader(404)
}
				return
			}
		} else { // Authenticated Agents
			// 如果不使用PSK，代理先前已经进行了身份验证
			if core.Debug {
				message("info", "Authenticated JWT")
			}

			// 解密JWE
			key = agents.GetEncryptionKey(agentID)

			j, errDecrypt := util.DecryptJWE(jweString, key)
			if errDecrypt != nil {
				message("warn", errDecrypt.Error())
				w.WriteHeader(404)
				return
			}

			if core.Debug {
				message("debug", fmt.Sprintf("[DEBUG]POST DATA: %v", j))
```

			}
			// 如果设置了Verbose标志，则输出认证的JWT和认证的JWE代理会话密钥的消息
			if core.Verbose {
				message("note", "Authenticated JWT w/ Authenticated JWE agent session key")
				message("info", fmt.Sprintf("Received %s message from %s at %s", j.Type, j.ID, time.Now().UTC().Format(time.RFC3339)))
			}

			// 如果同时返回了agentID和error，则声明可能有问题，需要代理重新认证
			if (errValidate != nil) && (agentID != uuid.Nil) {
				message("warn", fmt.Sprintf("Agent %s connected with expired JWT. Instructing agent to re-authenticate", agentID))
				j.Type = "ReAuthenticate"
			}

			// 认证和授权的消息类型
			switch j.Type {
			case "KeyExchange":
				returnMessage, err = agents.KeyExchange(j)
			case "StatusCheckIn":
				returnMessage, err = agents.StatusCheckIn(j)
			case "CmdResults":
				err = agents.JobResults(j)
		// 根据消息类型进行不同的处理
		switch j.Type {
			// 如果消息类型为AgentInfo，则调用agents.UpdateInfo方法
			case "AgentInfo":
				err = agents.UpdateInfo(j)
			// 如果消息类型为FileTransfer，则调用agents.FileTransfer方法
			case "FileTransfer":
				err = agents.FileTransfer(j)
			// 如果消息类型为ReAuthenticate，则调用agents.OPAQUEReAuthenticate方法，并返回消息和错误
			case "ReAuthenticate":
				returnMessage, err = agents.OPAQUEReAuthenticate(agentID)
			// 如果消息类型不在上述三种情况中，则返回错误信息
			default:
				err = fmt.Errorf("invalid message type: %s", j.Type)
			}
		}

		// 如果处理过程中出现错误，则记录错误信息并返回404状态码
		if err != nil {
			m := fmt.Sprintf("There was an error during while handling a message from agent %s:\r\n%s", agentID, err.Error())
			logging.Server(m)
			message("warn", m)
			w.WriteHeader(404)
			return
		}

		// 如果返回消息的类型为空
		if returnMessage.Type == "" {
		// 设置返回消息的类型为 "ServerOk"
		returnMessage.Type = "ServerOk"
		// 设置返回消息的ID为agentID
		returnMessage.ID = agentID
	}

	// 如果core.Verbose为真，则打印发送消息类型到代理的消息
	if core.Verbose {
		message("note", fmt.Sprintf("Sending "+returnMessage.Type+" message type to agent"))
	}

	// 获取JWT并添加到消息的Base中，除了重新认证消息之外的所有消息
	if returnMessage.Type != "ReAuthenticate" {
		// 获取JWT
		jsonWebToken, errJWT := util.GetJWT(agentID, ctx.JWTKey)
		// 如果获取JWT出错，则打印警告信息并返回404
		if errJWT != nil {
			message("warn", errJWT.Error())
			w.WriteHeader(404)
			return
		}
		// 将JWT添加到返回消息的Token中
		returnMessage.Token = jsonWebToken
	}

	// 将消息的Base编码为gob格式
	returnMessageBytes := new(bytes.Buffer)
		// 使用 GOB 编码器将 returnMessage 编码成字节流，存储到 returnMessageBytes 中
		errReturnMessageBytes := gob.NewEncoder(returnMessageBytes).Encode(returnMessage)
		// 如果编码过程中出现错误，记录错误信息并返回
		if errReturnMessageBytes != nil {
			m := fmt.Sprintf("there was an error encoding the %s return message for agent %s into a GOB:\r\n%s", returnMessage.Type, agentID, errReturnMessageBytes.Error())
			logging.Server(m)
			message("warn", m)
			return
		}

		// 获取代理的加密密钥
		key = agents.GetEncryptionKey(agentID)
		// 使用密钥对 returnMessageBytes 进行 JWE 加密
		jwe, errJWE := core.GetJWESymetric(returnMessageBytes.Bytes(), key)
		// 如果加密过程中出现错误，记录错误信息
		if errJWE != nil {
			logging.Server(errJWE.Error())
			message("warn", errJWE.Error())
		}

		// 设置返回头部的 Content-Type
		w.Header().Set("Content-Type", "application/octet-stream")

		// 将 JWE 编码成 GOB，并发送给代理
		// 使用gob编码器将jwe编码并写入到w中
		errEncode := gob.NewEncoder(w).Encode(jwe)
		// 如果编码过程中出现错误，记录日志并返回
		if errEncode != nil {
			m := fmt.Sprintf("There was an error encoding the server AuthComplete GOB message:\r\n%s", errEncode.Error())
			logging.Server(m)
			message("warn", m)
			return
		}

		// 在成功发送kill消息后，从服务器中移除agent
		if returnMessage.Type == "AgentControl" {
			if returnMessage.Payload.(messages.AgentControl).Command == "kill" {
				// 从agents中移除指定agent
				err := agents.RemoveAgent(agentID)
				// 如果移除过程中出现错误，记录日志并返回
				if err != nil {
					message("warn", err.Error())
					return
				}
				// 记录agent被移除的日志
				message("info", fmt.Sprintf("Agent %s was removed from the server", agentID))
				return
			}
		}
// 如果请求方法为 GET，则返回 404 状态码
} else if r.Method == "GET" {
    w.WriteHeader(404)
// 如果请求方法不为 GET，则返回 404 状态码
} else {
    w.WriteHeader(404)
}
// 如果调试模式开启，则打印调试信息
if core.Debug {
    message("debug", "Leaving http2.agentHandler function without error")
}

// message 用于向所有连接的客户端发送广播消息
func message(level string, message string) {
    // 创建消息对象
    m := messageAPI.UserMessage{
        Message: message,
        Time:    time.Now().UTC(),
        Error:   false,
    }
    // 根据消息级别进行处理
    switch level {
    case "info":
# 根据不同的输入字符串设置消息的级别
		# 如果输入字符串为 "info"，则将消息级别设置为 Info
		m.Level = messageAPI.Info
		# 如果输入字符串为 "note"，则将消息级别设置为 Note
		case "note":
		m.Level = messageAPI.Note
		# 如果输入字符串为 "warn"，则将消息级别设置为 Warn
		case "warn":
		m.Level = messageAPI.Warn
		# 如果输入字符串为 "debug"，则将消息级别设置为 Debug
		case "debug":
		m.Level = messageAPI.Debug
		# 如果输入字符串为 "success"，则将消息级别设置为 Success
		case "success":
		m.Level = messageAPI.Success
		# 如果输入字符串不匹配以上任何情况，则将消息级别设置为 Plain
		default:
		m.Level = messageAPI.Plain
	}
	# 发送设置好级别的消息
	messageAPI.SendBroadcastMessage(m)
}
```