# `kubesploit\pkg\api\listeners\listeners.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发是希望它对增强组织的安全性有所帮助。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括对适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包名为listeners
package listeners
// 导入标准库
import (
	"fmt" // 格式化输出
	"strings" // 字符串操作
	"time" // 时间操作

	// 导入第三方库
	uuid "github.com/satori/go.uuid" // 生成唯一标识符

	// 导入自定义包
	"kubesploit/pkg/api/messages" // 消息处理
	"kubesploit/pkg/listeners" // 监听器操作
	"kubesploit/pkg/servers" // 服务器操作
)

// Exists 函数用于确定输入的监听器名称是否已经实例化
func Exists(name string) messages.UserMessage {
	// 通过监听器名称获取监听器对象，判断是否存在
	_, err := listeners.GetListenerByName(name)
	// 如果存在错误，返回错误消息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 返回一个包含当前时间和错误状态的用户消息对象
	return messages.UserMessage{
		Time:  time.Now().UTC(),
		Error: false,
	}
}

// NewListener 在服务器上实例化一个新的 Listener 对象
func NewListener(options map[string]string) (messages.UserMessage, uuid.UUID) {
	// 使用给定的选项创建一个新的 Listener 对象
	l, err := listeners.New(options)
	// 如果创建过程中出现错误，返回包含错误信息的用户消息对象和空的 UUID
	if err != nil {
		return messages.ErrorMessage(err.Error()), uuid.Nil
	}
	// 根据创建的 Listener 对象生成消息
	m := fmt.Sprintf("%s listener was created with an ID of: %s", l.Name, l.ID)
	// 创建一个包含成功级别、当前时间、消息内容和错误状态的用户消息对象
	um := messages.UserMessage{
		Level:   messages.Success,
		Time:    time.Now().UTC(),
		Message: m,
		Error:   false,
	}
// Remove函数用于删除服务器中的监听器，并返回相应的用户消息
func Remove(name string) messages.UserMessage {
	// 通过监听器名称获取监听器对象
	l, errL := listeners.GetListenerByName(name)
	// 如果获取监听器对象出错，返回相应的错误消息
	if errL != nil {
		return messages.ErrorMessage(errL.Error())
	}
	// 通过监听器ID删除监听器
	err := listeners.RemoveByID(l.ID)
	// 如果删除监听器出错，返回相应的错误消息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 格式化消息，表示成功删除监听器
	m := fmt.Sprintf("deleted listener %s:%s", l.Name, l.ID)
	// 返回成功的用户消息
	return messages.UserMessage{
		Level:   messages.Success,
		Time:    time.Now().UTC(),
		Message: m,
		Error:   false,
	}
}
// Restart函数用于重新启动监听器的服务器
func Restart(listenerID uuid.UUID) messages.UserMessage {
    // 通过监听器ID获取监听器对象
    l, err := listeners.GetListenerByID(listenerID)
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 重新启动监听器，并传入配置选项
    errRestart := l.Restart(l.GetConfiguredOptions())
    if errRestart != nil {
        return messages.ErrorMessage(errRestart.Error())
    }

    // TODO 不确定这段代码应该放在这里还是放在listeners包中
    // 启动监听器的服务器，使用goroutine以异步方式执行
    go func() {
        err := l.Server.Start()
        if err != nil {
            // 异步发送错误消息
            messages.DelayedMessage(messages.ErrorMessage(err.Error()))
        }
    }()
}
	// 返回一个包含成功消息的 UserMessage 结构体
	return messages.UserMessage{
		Level:   messages.Success, // 设置消息级别为成功
		Time:    time.Now().UTC(), // 设置消息时间为当前时间的 UTC 时间
		Message: fmt.Sprintf("%s listener was successfully restarted", l.Name), // 根据监听器名称生成成功消息
		Error:   false, // 设置错误标志为 false
	}
}

// SetOption 设置可配置监听器选项的值
func SetOption(listenerID uuid.UUID, Args []string) messages.UserMessage {
	// 根据监听器ID获取监听器信息
	l, err := listeners.GetListenerByID(listenerID)
	if err != nil {
		// 如果出现错误，返回包含错误消息的 UserMessage 结构体
		return messages.ErrorMessage(err.Error())
	}
	// 如果参数数量大于等于2
	if len(Args) >= 2 {
		// 遍历已配置选项
		for k := range l.GetConfiguredOptions() {
			// 如果参数中的选项名称与已配置选项匹配
			if Args[1] == k {
				// 将参数值连接成字符串
				v := strings.Join(Args[2:], " ")
				// 设置选项的值
				err := l.SetOption(k, v)
				// 如果出现错误
				if err != nil {
// 返回一个错误消息对象，其中包含特定错误信息
return messages.ErrorMessage(err.Error())

// 返回一个用户消息对象，表示操作成功，并包含相关信息
return messages.UserMessage{
    Error:   false,
    Level:   messages.Success,
    Time:    time.Now().UTC(),
    Message: fmt.Sprintf("set %s to: %s", k, v),
}

// 返回一个错误消息对象，表示参数不足的错误信息
return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Listeners SetOption call: %s", Args))

// 启动监听器的服务器，并返回一个用户消息对象
func Start(name string) messages.UserMessage {
    // 通过名称获取监听器对象
    l, err := listeners.GetListenerByName(name)
    // 如果出现错误，返回一个错误消息对象
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
# 根据服务器的状态进行不同的操作
switch l.Server.Status() {
    # 如果服务器正在运行，返回一个用户消息，表示服务器已经在运行
    case servers.Running:
        return messages.UserMessage{
            Error:   false,
            Level:   messages.Note,
            Time:    time.Now().UTC(),
            Message: "the server is already running",
        }
    # 如果服务器已经停止，启动一个协程来启动服务器，并返回一个用户消息
    case servers.Stopped:
        go func() {
            err := l.Server.Start()
            if err != nil {
                messages.DelayedMessage(messages.ErrorMessage(err.Error()))
            }
        }()
        return messages.UserMessage{
            Error: false,
            Level: messages.Success,
            Time:  time.Now().UTC(),
            Message: fmt.Sprintf("Started %s listener on %s:%d", servers.GetProtocol(l.Server.GetProtocol()),
            # ...
        }
}
// 获取服务器接口和端口，并将其作为参数传递给 Restart 方法
l.Server.GetInterface(),
l.Server.GetPort()),
}

// 如果服务器状态为 Closed，则尝试重新启动服务器，并返回相应的消息
case servers.Closed:
if err := l.Restart(l.GetConfiguredOptions()); err != nil {
    return messages.ErrorMessage(err.Error())
}

// 启动服务器，并在启动过程中处理可能出现的错误
go func() {
    err := l.Server.Start()
    if err != nil {
        messages.DelayedMessage(messages.ErrorMessage(err.Error()))
    }
}()

// 返回成功的用户消息，包括重启的监听器的信息
return messages.UserMessage{
    Error: false,
    Level: messages.Success,
    Time:  time.Now().UTC(),
    Message: fmt.Sprintf("Restarted %s %s listener on %s:%d", l.Name, servers.GetProtocol(l.Server.GetProtocol()),
        l.Server.GetInterface(),
        l.Server.GetPort()),
		}
	default:
		// 如果服务器状态未被处理，返回错误消息
		return messages.UserMessage{
			Error:   true,
			Level:   messages.Warn,
			Time:    time.Now().UTC(),
			Message: fmt.Sprintf("unhandled server status: %s", servers.GetStateString(l.Server.Status())),
		}
	}
}

// Stop terminates the Listener's server
// 停止 Listener 的服务器
func Stop(name string) messages.UserMessage {
	// 通过名称获取 Listener 对象
	l, err := listeners.GetListenerByName(name)
	// 如果获取失败，返回错误消息
	if err != nil {
		return messages.ErrorMessage(err.Error())
	}
	// 如果服务器正在运行，停止服务器
	if l.Server.Status() == servers.Running {
		err := l.Server.Stop()
		// 如果停止失败，返回错误消息
		if err != nil {
		// 如果发生错误，返回错误消息
		return messages.ErrorMessage(err.Error())
		// 如果没有发生错误，返回成功消息
		}
		return messages.UserMessage{
			Error:   false,
			Level:   messages.Success,
			Time:    time.Now().UTC(),
			Message: fmt.Sprintf("%s listener was stopped", l.Name),
		}
	}
	// 如果没有找到监听器，返回提示消息
	return messages.UserMessage{
		Error:   false,
		Level:   messages.Note,
		Time:    time.Now().UTC(),
		Message: "this listener is not running",
	}
}

// GetListenerStatus 返回监听器的服务器状态
func GetListenerStatus(listenerID uuid.UUID) messages.UserMessage {
	// 通过监听器ID获取监听器信息
	l, err := listeners.GetListenerByID(listenerID)
// 如果发生错误，返回错误消息
if err != nil {
    return messages.ErrorMessage(err.Error())
}
// 返回用户消息对象，包含消息级别、消息内容、时间和错误状态
return messages.UserMessage{
    Level:   messages.Plain,
    Message: servers.GetStateString(l.Server.Status()),
    Time:    time.Time{},
    Error:   false,
}
}

// 通过名称获取已实例化监听器的唯一标识符
func GetListenerByName(name string) (messages.UserMessage, uuid.UUID) {
    // 根据名称获取监听器对象
    l, err := listeners.GetListenerByName(name)
    // 如果发生错误，返回错误消息和空的 UUID
    if err != nil {
        return messages.ErrorMessage(err.Error()), uuid.Nil
    }
    // 创建用户消息对象，包含错误状态和当前时间
    um := messages.UserMessage{
        Error: false,
        Time:  time.Now().UTC(),
	}
	return um, l.ID
}

// GetListenerConfiguredOptions enumerates all of a Listener's settings and returns them
// 根据监听器ID获取监听器的配置选项
func GetListenerConfiguredOptions(listenerID uuid.UUID) (messages.UserMessage, map[string]string) {
	// 通过监听器ID获取监听器信息
	l, err := listeners.GetListenerByID(listenerID)
	// 如果出现错误，返回错误消息和空的配置选项
	if err != nil {
		return messages.ErrorMessage(err.Error()), nil
	}
	// 初始化用户消息
	um := messages.UserMessage{
		Message: "",
		Time:    time.Now().UTC(),
		Error:   false,
	}
	// 返回用户消息和监听器的配置选项
	return um, l.GetConfiguredOptions()
}

// GetListeners returns a list of an instantiated Listeners
// 获取已实例化的监听器列表
func GetListeners() []listeners.Listener {
// 返回所有的监听器
func GetListeners() {
	return listeners.GetListeners()
}

// 返回可用于监听器的服务器协议类型
func GetListenerTypes() []string {
	return listeners.GetListenerTypes()
}

// 根据提供的协议类型返回未实例化监听器的所有可配置选项
func GetListenerOptions(protocol string) map[string]string {
	return listeners.GetListenerOptions(protocol)
}

// TODO 将自动完成器移动到 CLI 包中

// 返回可用监听器的 CLI tab 自动完成器
func GetListenerNamesCompleter() func(string) []string {
	return listeners.GetList()
}
// GetListenerOptionsCompleter 返回支持的 Listener 服务器协议的 CLI tab 补全器
func GetListenerOptionsCompleter(protocol string) func(string) []string {
    return listeners.GetListenerOptionsCompleter(protocol)
}

// GetListenerTypesCompleter 返回可用的 Listener 类型的 CLI tab 补全器
func GetListenerTypesCompleter() func(string) []string {
    return listeners.GetListenerTypesCompleter()
}
```