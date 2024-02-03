# `kubesploit\pkg\handlers\http.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package handlers

import (
    // 标准库
    "bytes"
    "crypto/sha256"
    "encoding/gob"
    "fmt"
    "io/ioutil"
    "net/http"
    "strings"
    "time"

    // 第三方库
    uuid "github.com/satori/go.uuid"
    "go.dedis.ch/kyber/v3"

    // Merlin
    "kubesploit/pkg/agents"
    messageAPI "kubesploit/pkg/api/messages"
    "kubesploit/pkg/core"
    "kubesploit/pkg/logging"
    "kubesploit/pkg/messages"
    "kubesploit/pkg/util"
)

// HTTPContext包含有关处理程序的上下文信息，例如用于加密/解密的密钥
type HTTPContext struct {
    Context
    PSK       string       // 在密码认证密钥交换（PAKE）之前使用的预共享密钥密码
    JWTKey    []byte       // 服务器用于创建JWT的密码
    OpaqueKey kyber.Scalar // OPAQUE服务器的密钥
}

// AgentHTTP函数负责所有Merlin代理流量
func (ctx *HTTPContext) AgentHTTP(w http.ResponseWriter, r *http.Request) {
    // 如果设置了Verbose标志，则记录接收到的连接信息
    if core.Verbose {
        m := fmt.Sprintf("Received %s %s connection from %s", r.Proto, r.Method, r.RemoteAddr)
        message("warn", m)
        logging.Server(m)
    }

    // 如果设置了Debug标志，则记录HTTP连接的详细信息
    if core.Debug {
        var m string
        m += "HTTP Connection Details:\r\n"
        m += fmt.Sprintf("Host: %s\r\n", r.Host)
        m += fmt.Sprintf("URI: %s\r\n", r.RequestURI)
        m += fmt.Sprintf("Method: %s\r\n", r.Method)
        m += fmt.Sprintf("Protocol: %s\r\n", r.Proto)
        m += fmt.Sprintf("Headers: %s\r\n", r.Header)
        if r.TLS != nil {
            m += fmt.Sprintf("TLS Negotiated Protocol: %s\r\n", r.TLS.NegotiatedProtocol)
            m += fmt.Sprintf("TLS Cipher Suite: %d\r\n", r.TLS.CipherSuite)
            m += fmt.Sprintf("TLS Server Name: %s\r\n", r.TLS.ServerName)
        }
        m += fmt.Sprintf("Content Length: %d", r.ContentLength)

        message("debug", m)
        logging.Server(m)
    }

    // 检查是否有Merlin PRISM活动
    if r.UserAgent() == "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.85 Safari/537.36 " {
        m := fmt.Sprintf("Someone from %s is attempting to fingerprint this Merlin server", r.RemoteAddr)
        message("warn", m)
        logging.Server(m)
    }

    // 确保消息包含JWT
    token := r.Header.Get("Authorization")
    if token == "" {
        if core.Verbose {
            m := "incoming request did not contain an Authorization header"
            message("warn", m)
            logging.Server(m)
        }
        w.WriteHeader(404)
        return
    }

    // 确保Authorization头包含Bearer令牌
    if !strings.Contains(token, "Bearer eyJ") {
        if core.Verbose {
            m := "incoming request did not contain a Bearer token"
            message("warn", m)
            logging.Server(m)
        }
        w.WriteHeader(404)
        return
    }
    // 确保内容类型为：application/octet-stream; charset=utf-8
    if r.Header.Get("Content-Type") != "application/octet-stream; charset=utf-8" {
        // 如果内容类型不符合要求，并且设置了详细输出，则记录警告信息并写入日志
        if core.Verbose {
            m := "incoming request did not contain a Content-Type header of: application/octet-stream; charset=utf-8"
            message("warn", m)
            logging.Server(m)
        }
        // 返回状态码 404
        w.WriteHeader(404)
        return
    }
    // 如果请求方法为 GET，则返回状态码 404
    } else if r.Method == "GET" {
        w.WriteHeader(404)
    } else {
        // 其他情况下返回状态码 404
        w.WriteHeader(404)
    }
    // 如果设置了调试模式，则记录调试信息
    if core.Debug {
        message("debug", "Leaving http2.agentHandler function without error")
    }
// message 用于向所有连接的客户端发送广播消息
func message(level string, message string) {
    // 创建一个 UserMessage 结构体对象
    m := messageAPI.UserMessage{
        Message: message,
        Time:    time.Now().UTC(),
        Error:   false,
    }
    // 根据不同的级别设置消息的级别
    switch level {
    case "info":
        m.Level = messageAPI.Info
    case "note":
        m.Level = messageAPI.Note
    case "warn":
        m.Level = messageAPI.Warn
    case "debug":
        m.Level = messageAPI.Debug
    case "success":
        m.Level = messageAPI.Success
    default:
        m.Level = messageAPI.Plain
    }
    // 发送广播消息
    messageAPI.SendBroadcastMessage(m)
}
```