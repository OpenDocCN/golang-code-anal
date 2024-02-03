# `kubesploit\pkg\api\messages\messages.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途适用性的暗示保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package messages

import (
    // 标准库
    "fmt"
    "time"

    // 第三方库
    uuid "github.com/satori/go.uuid"
)

const (
    // Info消息仅用于通知用户，并不表示需要采取行动
    Info int = 0
    // Note消息类似于详细消息，用于让用户了解最新情况
    Note int = 1
    // Warn消息用于通知用户发生错误或某些事情未按计划进行
    Warn int = 2
    // Debug消息在调试程序时显示，并显示大量信息
    Debug int = 3
    // Success消息用于通知用户某个操作已经顺利完成
    Success int = 4
    // Plain消息没有颜色或其他格式，并用于边缘情况
    Plain int = 5
)

// messageChannel是所有注册客户端的映射，消息可以在通道中排队等待客户端
var messageChannel = make(map[uuid.UUID]chan UserMessage)
// UserMessage 是一个结构体，用于保存将要展示给用户的消息
type UserMessage struct {
    Level   int       // 消息级别（例如信息，调试，详细信息）
    Message string    // 要显示的文本
    Time    time.Time // 消息生成的时间
    Error   bool      // 消息是否是错误的结果
}

// Register 通知服务器客户端存在，并且可以接收消息
func Register(clientID uuid.UUID) UserMessage {
    // 如果消息通道中不存在该客户端，则创建一个消息通道
    if _, ok := messageChannel[clientID]; !ok {
        messageChannel[clientID] = make(chan UserMessage)
        return UserMessage{
            Level:   Success,
            Message: "successfully registered client for user message channel with server",
            Time:    time.Now().UTC(),
            Error:   false,
        }
    }
    // 如果消息通道中已经存在该客户端，则返回警告消息
    return UserMessage{
        Level:   Warn,
        Message: fmt.Sprintf("client %s already registered for user a user message channel with server", clientID),
        Time:    time.Now().UTC(),
        Error:   true,
    }
}

// SendBroadcastMessage 将输入消息添加到所有注册客户端的消息通道中
func SendBroadcastMessage(message UserMessage) {
    // 遍历消息通道，将消息发送给所有注册的客户端
    for c := range messageChannel {
        messageChannel[c] <- message
    }
}

// GetMessageForClient 用于客户端接收其队列中的任何消息
func GetMessageForClient(clientID uuid.UUID) UserMessage {
    // 如果消息通道中存在该客户端，则从通道中接收消息
    if _, ok := messageChannel[clientID]; ok {
        return <-messageChannel[clientID]
    }
    // 如果消息通道中不存在该客户端，则返回警告消息
    return UserMessage{
        Level:   Warn,
        Message: fmt.Sprintf(" could not get messages for client %s because it does not exist", clientID),
        Time:    time.Now().UTC(),
        Error:   true,
    }
}

// ErrorMessage 返回一个预格式化的错误消息
func ErrorMessage(message string) UserMessage {
    return UserMessage{
        Error:   true,
        Level:   Warn,
        Time:    time.Now().UTC(),
        Message: message,
    }
}
// JobMessage 返回一个消息，显示代理作业已成功创建
func JobMessage(agentID uuid.UUID, jobID string) UserMessage {
    // 格式化消息，包括作业ID、代理ID和当前时间
    m := fmt.Sprintf("Created job %s for agent %s at %s", jobID, agentID, time.Now().UTC().Format(time.RFC3339))
    // 返回用户消息对象
    return UserMessage{
        Error:   false,
        Level:   Note,
        Message: m,
        Time:    time.Now().UTC(),
    }
}

// DelayedMessage 用于返回延迟或推迟的消息，比如用于go协程的情况
func DelayedMessage(message UserMessage) {
    // 发送延迟消息给客户端
    SendBroadcastMessage(message)
}
```