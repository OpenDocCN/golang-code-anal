# `kubesploit\pkg\agent\agent_test.go`

```
// Kubesploit 是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参阅<http://www.gnu.org/licenses/>。

package agent

import (
    // 标准库
    "bytes"
    "crypto/sha256"
    "encoding/gob"
    "fmt"
    "net/http"
    "strings"
    "testing"
    "time"

    // 第三方库
    "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/core"
    "kubesploit/pkg/messages"
    "kubesploit/test/testServer"
)

// TestNewHTTPAgent 确保agent.New函数返回一个http代理而不出错
func TestNewHTTPAgent(t *testing.T) {
    _, err := New("http", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false)

    if err != nil {
        t.Error(err)
    }
}

// TestNewHTTPSAgent 确保agent.New函数返回一个https代理而不出错
func TestNewHTTPSAgent(t *testing.T) {
    _, err := New("https", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false)

    if err != nil {
        t.Error(err)
    }
}

// TestNewH2CAgent 确保agent.New函数返回一个HTTP/2明文代理而不出错
func TestNewH2CAgent(t *testing.T) {
    // 使用 New 函数创建一个新的对象，使用 "h2c" 协议，连接到指定的地址
    // 参数分别为 "h2c", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false
    _, err := New("h2c", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false)

    // 如果创建对象时发生错误，输出错误信息
    if err != nil {
        t.Error(err)
    }
// TestNewH2Agent 确保 agent.New 函数返回一个没有错误的 HTTP/2 agent
func TestNewH2Agent(t *testing.T) {
    // 调用 New 函数创建一个 HTTP/2 agent，并检查是否有错误返回
    _, err := New("h2", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false)

    if err != nil {
        t.Error(err)
    }
}

// TestNewHQAgent 确保 agent.New 函数返回一个没有错误的 HTTP/3 agent
func TestNewHQAgent(t *testing.T) {
    // 调用 New 函数创建一个 HTTP/3 agent，并检查是否有错误返回
    _, err := New("http3", "https://127.0.0.1:8080", "", "test", "http://127.0.0.1:8081", "", false, false)

    if err != nil {
        t.Error(err)
    }
}

// TestKillDate 发送一个带有已过期的 kill date 的消息
func TestKillDate(t *testing.T) {
    // 创建一个 HTTP/2 agent，并检查是否有错误返回
    agent, err := New("h2", "https://127.0.0.1:8080", "", "test", "", "", false, false)

    if err != nil {
        t.Error(err)
    }

    // 设置 kill date 为 1560616599
    agent.KillDate = 1560616599

    // 运行 agent，并检查是否有错误返回
    errRun := agent.Run()
    // TODO 除非有错误发生，否则该函数实际上不会返回
    if errRun == nil {
        t.Errorf("当 killdate 超过时，agent 没有退出")
    }
}

// TestFailedCheckin 测试 agent 在失败的 checkin 尝试次数超过 agent 的 MaxRetry 设置后退出
func TestFailedCheckin(t *testing.T) {
    // 创建一个 HTTP/2 agent，并检查是否有错误返回
    agent, err := New("h2", "https://127.0.0.1:8080", "", "test", "", "", false, false)

    if err != nil {
        t.Error(err)
    }

    // 设置 FailedCheckin 为 agent 的 MaxRetry
    agent.FailedCheckin = agent.MaxRetry

    // 运行 agent，并检查是否有错误返回
    errRun := agent.Run()
    if errRun == nil {
        t.Errorf("当失败的 checkin 尝试次数达到最大值时，agent 没有退出")
    }
}

// TestInvalidMessageType 发送一个带有无效 Type 字符串的有效 message.Base
func TestInvalidMessageType(t *testing.T) {
    // 创建一个 HTTP/2 agent，并检查是否有错误返回
    agent, err := New("h2", "https://127.0.0.1:8080", "", "test", "", "", false, false)

    if err != nil {
        t.Error(err)
    }

    // 创建一个有效的 message.Base，但 Type 字符串无效
    m := messages.Base{
        Version: 1.0,
        ID:      agent.ID,
        Type:    "NotReal",
        Token:   agent.JWT,
    }
    // 发送消息，并检查是否有错误返回
    _, errSend := agent.sendMessage("POST", m)
}
    # 如果 errSend 为 nil，则表示代理处理了一个无效的消息类型但没有返回错误
    if errSend == nil:
        # 输出错误信息
        t.Error("agent handler processed an invalid message type without returning an error")
// TestInvalidMessage 发送一个不是有效 message.Base 结构的消息
func TestInvalidMessage(t *testing.T) {
    // 创建一个新的 agent 对象
    agent, err := New("h2", "https://127.0.0.1:8081", "", "test", "", "", false, false)

    // 如果创建 agent 对象时出现错误，输出错误信息
    if err != nil {
        t.Error(err)
    }

    // 用于测试开始/结束信号的通道
    setup := make(chan struct{})
    ended := make(chan struct{})

    // 启动测试服务器
    go testserver.TestServer{}.Start("8081", ended, setup, t)
    // 等待设置完成
    <-setup

    // 定义一个 testMessage 结构
    type testMessage struct {
        Alpha   string
        Number  int64
        Boolean bool
    }

    // 创建一个 testMessage 对象
    m := testMessage{
        Alpha:   "TestString",
        Number:  1337,
        Boolean: false,
    }

    // 无法使用 agent.sendMessage，因为它只接受有效的 message.Base 对象

    // 将消息转换为 gob 格式
    messageBytes := new(bytes.Buffer)
    errGobEncode := gob.NewEncoder(messageBytes).Encode(m)
    if errGobEncode != nil {
        t.Errorf("编码消息时出错:\r\n%s", errGobEncode.Error())
        return
    }

    // 获取 JWE
    jweString, errJWE := core.GetJWESymetric(messageBytes.Bytes(), agent.secret)
    if errJWE != nil {
        t.Errorf("获取 JWE 时出错:\r\n%s", errJWE.Error())
        return
    }

    // 将 JWE 编码为 gob 格式
    jweBytes := new(bytes.Buffer)
    errJWEBuffer := gob.NewEncoder(jweBytes).Encode(jweString)
    if errJWEBuffer != nil {
        t.Errorf("编码 JWE 时出错:\r\n%s", errJWEBuffer.Error())
        return
    }

    // 创建一个新的 POST 请求
    req, reqErr := http.NewRequest("POST", agent.URL, jweBytes)
    if reqErr != nil {
        t.Errorf("发送 POST 请求时出错:\r\n%s", reqErr.Error())
        return
    }

    // 设置请求头部信息
    if req != nil {
        req.Header.Set("User-Agent", agent.UserAgent)
        req.Header.Set("Content-Type", "application/octet-stream; charset=utf-8")
        req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", agent.JWT))
    }

    // 发送请求
}
    // 从 agent.Client 中获取一个客户端实例
    var client = *agent.Client
    // 使用客户端实例发送请求，并接收响应
    resp, err := client.Do(req)
    // 如果发生错误，输出错误信息并返回
    if err != nil {
        t.Errorf("there was an error with the HTTP client while performing a POST:\r\n%s", err.Error())
        return
    }

    // 关闭 ended 通道
    close(ended)

    // 如果响应为空，输出错误信息并返回
    if resp == nil {
        t.Error("the server did not return a response")
        return
    }

    // 如果响应状态码不是 404，输出错误信息
    if resp.StatusCode != 404 {
        t.Error("the merlin server did not return a 404 for an invalid message type")
    }
// TestPSK 确保代理不能使用错误的PSK与服务器成功通信
func TestPSK(t *testing.T) {
    // 创建一个新的代理对象，使用指定的参数
    agent, err := New("h2", "https://127.0.0.1:8080/merlin", "", "test", "", "", false, false)

    // 如果创建代理对象时出现错误，输出错误信息
    if err != nil {
        t.Error(err)
    }
    // 设置等待时间为5000毫秒
    agent.WaitTime = 5000 * time.Millisecond
    // 使用"wrongPassword"进行SHA256哈希，将结果赋值给代理对象的密钥
    k := sha256.Sum256([]byte("wrongPassword"))
    agent.secret = k[:]

    // 用于测试开始/结束的信号通道
    setup := make(chan struct{})
    ended := make(chan struct{})

    // 启动测试服务器
    go testserver.TestServer{}.Start("8080", ended, setup, t)
    // 等待设置完成
    <-setup

    // 创建一个消息对象
    m := messages.Base{
        Version: 1.0,
        ID:      agent.ID,
        Type:    "StatusOk",
        Token:   agent.JWT,
    }

    // 发送加密消息给代理，检查是否会返回错误
    _, errSend := agent.sendMessage("POST", m)
    if errSend == nil {
        t.Error("代理成功使用错误的密钥发送了加密消息")
        return
    }

    // 关闭测试服务器
    close(ended)
}

// TestWrongUUID 使用不同于正在运行代理的UUID向代理发送有效消息
func TestWrongUUID(t *testing.T) {
    // 创建一个新的代理对象，使用指定的参数
    agent, err := New("h2", "https://127.0.0.1:8080", "", "test", "", "", false, false)

    // 如果创建代理对象时出现错误，输出错误信息
    if err != nil {
        t.Error(err)
    }

    // 创建一个消息对象
    m := messages.Base{
        Version: 1.0,
        ID:      uuid.NewV4(),
        Type:    "ServerOk",
        Token:   agent.JWT,
    }

    // 向代理发送带有不匹配UUID的消息，期望代理返回错误
    _, errHandler := agent.messageHandler(m)
    if errHandler == nil {
        t.Error("代理处理了带有错误UUID的消息，但没有返回错误")
    }

    // 如果返回了错误，检查错误信息是否包含特定内容
    if errHandler != nil {
        if !strings.Contains(errHandler.Error(), "the input message UUID did not match this agent's UUID") {
            t.Error(errHandler)
        }
    }
}

// TestInvalidHTTPTrafficPayload 向服务器发送一个gob编码的字符串，以确保它能处理无效的流量
func TestInvalidHTTPTrafficPayload(t *testing.T) {
    // 创建一个新的代理对象，使用 "h2" 协议，连接到指定的地址，使用指定的用户名和密码
    agent, err := New("h2", "https://127.0.0.1:8080", "", "test", "", "", false, false)

    // 如果创建代理对象时发生错误，输出错误信息
    if err != nil {
        t.Error(err)
    }

    // 创建一个消息对象，设置版本号、ID、类型和令牌
    m := messages.Base{
        Version: 1.0,
        ID:      agent.ID,
        Type:    "BadPayload",
        Token:   agent.JWT,
    }

    // 向代理对象发送消息，使用 "POST" 方法，如果出现错误，将错误信息存储在 errHandler 中
    _, errHandler := agent.sendMessage("POST", m)
    // 如果没有出现错误，输出错误信息
    if errHandler == nil {
        t.Error("the agent handled a message with a wrong UUID without returning an error")
    }
// TestAuthentication 验证使用正确的PSK进行成功身份验证
func TestAuthentication(t *testing.T) {
    // 创建新的代理对象，使用指定的参数
    agent, err := New("h2", "https://127.0.0.1:8082/merlin", "", "test", "", "", false, false)

    // 如果创建代理对象时出现错误，输出错误信息并返回
    if err != nil {
        t.Error(err)
        return
    }
    // 设置等待时间
    agent.WaitTime = 5000 * time.Millisecond

    // 用于测试开始/结束的信号通道
    setup := make(chan struct{})
    ended := make(chan struct{})

    // 启动测试服务器
    go testserver.TestServer{}.Start("8082", ended, setup, t)
    // 等待设置完成
    <-setup

    // 进行初始身份验证
    authenticated := agent.initialCheckIn()
    // 如果身份验证失败，输出错误信息
    if authenticated == false {
        t.Error("the agent did not successfully authenticate")
    }
    // 关闭测试服务器
    close(ended)
}

// TestBadAuthentication 验证使用错误的PSK进行身份验证失败
func TestBadAuthentication(t *testing.T) {
    // 创建新的代理对象，使用指定的参数
    agent, err := New("h2", "https://127.0.0.1:8083", "", "neverGonnaGiveYouUp", "", "", false, false)

    // 如果创建代理对象时出现错误，输出错误信息并返回
    if err != nil {
        t.Error(err)
        return
    }
    // 设置等待时间
    agent.WaitTime = 5000 * time.Millisecond

    // 用于测试开始/结束的信号通道
    setup := make(chan struct{})
    ended := make(chan struct{})

    // 启动测试服务器
    go testserver.TestServer{}.Start("8083", ended, setup, t)
    // 等待设置完成
    <-setup

    // 进行初始身份验证
    authenticated := agent.initialCheckIn()
    // 如果身份验证成功，输出错误信息
    if authenticated != false {
        t.Error("the agent successfully authenticated with the wrong PSK")
    }
    // 关闭测试服务器
    close(ended)
}

// Bad content-type header
// TODO test every function of the message handler
```