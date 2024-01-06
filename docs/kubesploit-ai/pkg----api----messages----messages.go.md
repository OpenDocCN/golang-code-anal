# `kubesploit\pkg\api\messages\messages.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包声明
package messages
# 导入标准库中的 fmt 和 time 模块，以及第三方库中的 uuid 模块
import (
	"fmt"
	"time"
	uuid "github.com/satori/go.uuid"
)

# 定义不同类型的消息级别常量，用于标识消息的重要程度
const (
	# Info 消息用于向用户提供信息，不表示需要采取任何行动
	Info int = 0
	# Note 消息类似于详细消息，用于让用户了解最新情况
	Note int = 1
	# Warn 消息用于通知用户发生错误或某些事情未按计划进行
	Warn int = 2
	# Debug 消息在调试程序时显示大量信息
	Debug int = 3
	# Success 消息用于通知用户某个操作已成功完成
	Success int = 4
)
// Plain messages have no color or other formatting applied and are used for edge cases
// 定义了一个普通消息的级别，用于处理边缘情况，没有颜色或其他格式

// messageChannel is a map of all registered clients where messages can be queued in a channel for the client
// messageChannel是一个包含所有注册客户端的映射，可以在通道中为客户端排队消息

// UserMessage is a structure that holds messages that will be presented to the user
// UserMessage是一个结构，用于保存将呈现给用户的消息
// 包括消息级别（例如Info，Debug，Verbose）、要显示的文本、生成消息的时间、消息是否是错误的标志

// Register lets the sever know that a client exists and that it can be sent messages
// Register函数让服务器知道客户端存在，并且可以发送消息给客户端
// 定义一个结构体 UserMessage，包含 Level（消息级别）、Message（消息内容）、Time（时间戳）、Error（是否出错）字段
type UserMessage struct {
	Level   string
	Message string
	Time    time.Time
	Error   bool
}

// RegisterClient 为指定用户注册客户端消息通道，并返回注册结果
func RegisterClient(clientID string) UserMessage {
	// 如果客户端成功注册，返回成功消息
	if _, ok := registeredClients[clientID]; !ok {
		registeredClients[clientID] = true
		return UserMessage{
			Level:   Success,
			Message: "successfully registered client for user message channel with server",
			Time:    time.Now().UTC(),
			Error:   false,
		}
	}
	// 如果客户端已经注册，返回警告消息
	return UserMessage{
		Level:   Warn,
		Message: fmt.Sprintf("client %s already registered for user a user message channel with server", clientID),
		Time:    time.Now().UTC(),
		Error:   true,
	}
}

// SendBroadcastMessage 将输入消息添加到所有已注册客户端的消息通道中
func SendBroadcastMessage(message UserMessage) {
	// 遍历消息通道中的所有客户端，将消息添加到各自的通道中
	for c := range messageChannel {
		messageChannel[c] <- message
	}
}
// GetMessageForClient用于客户端接收其排队的任何消息
func GetMessageForClient(clientID uuid.UUID) UserMessage {
	// 检查消息通道中是否存在客户端ID对应的消息
	if _, ok := messageChannel[clientID]; ok {
		// 如果存在，则从消息通道中接收消息并返回
		return <-messageChannel[clientID]
	}
	// 如果不存在，则返回一个包含错误信息的UserMessage结构体
	return UserMessage{
		Level:   Warn,
		Message: fmt.Sprintf(" could not get messages for client %s because it does not exist", clientID),
		Time:    time.Now().UTC(),
		Error:   true,
	}
}

// ErrorMessage返回一个预格式化的错误消息，用于输入字符串
func ErrorMessage(message string) UserMessage {
	// 返回一个包含错误信息的UserMessage结构体
	return UserMessage{
		Error:   true,
		Level:   Warn,
		Time:    time.Now().UTC(),
// 创建一个包含消息的 UserMessage 结构体
func JobMessage(agentID uuid.UUID, jobID string) UserMessage {
	// 根据 agentID、jobID 和当前时间创建消息
	m := fmt.Sprintf("Created job %s for agent %s at %s", jobID, agentID, time.Now().UTC().Format(time.RFC3339))
	// 返回一个包含消息的 UserMessage 结构体
	return UserMessage{
		Error:   false,
		Level:   Note,
		Message: m,
		Time:    time.Now().UTC(),
	}
}

// DelayedMessage 用于返回延迟或延迟发送的消息，比如用于 go routines
func DelayedMessage(message UserMessage) {
	// 发送延迟消息给客户端
	SendBroadcastMessage(message)
}
```