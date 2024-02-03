# `kubesploit\pkg\api\listeners\listeners.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package listeners

import (
    // 标准库
    "fmt"
    "strings"
    "time"

    // 第三方库
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/api/messages"
    "kubesploit/pkg/listeners"
    "kubesploit/pkg/servers"
)

// Exists函数确定输入的监听器名称是否是一个实例化对象
func Exists(name string) messages.UserMessage {
    // 通过监听器名称获取监听器对象，判断是否存在
    _, err := listeners.GetListenerByName(name)
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    return messages.UserMessage{
        Time:  time.Now().UTC(),
        Error: false,
    }
}

// NewListener函数在服务器上实例化一个新的监听器对象
func NewListener(options map[string]string) (messages.UserMessage, uuid.UUID) {
    // 使用选项实例化一个新的监听器对象
    l, err := listeners.New(options)
    if err != nil {
        return messages.ErrorMessage(err.Error()), uuid.Nil
    }
    // 返回创建监听器的消息和ID
    m := fmt.Sprintf("%s listener was created with an ID of: %s", l.Name, l.ID)
    # 创建一个用户消息对象，包含级别、时间、消息内容和错误信息
    um := messages.UserMessage{
        Level:   messages.Success,  # 设置消息级别为成功
        Time:    time.Now().UTC(),   # 设置消息时间为当前时间的 UTC 时间
        Message: m,                  # 设置消息内容为 m
        Error:   false,              # 设置错误信息为 false
    }
    # 返回用户消息对象和 l 的 ID
    return um, l.ID
// Remove函数用于删除并移除服务器上的监听器
func Remove(name string) messages.UserMessage {
    // 通过名称获取监听器
    l, errL := listeners.GetListenerByName(name)
    // 如果获取监听器出错，则返回错误消息
    if errL != nil {
        return messages.ErrorMessage(errL.Error())
    }
    // 通过监听器ID删除监听器
    err := listeners.RemoveByID(l.ID)
    // 如果删除监听器出错，则返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 返回成功删除监听器的消息
    m := fmt.Sprintf("deleted listener %s:%s", l.Name, l.ID)
    return messages.UserMessage{
        Level:   messages.Success,
        Time:    time.Now().UTC(),
        Message: m,
        Error:   false,
    }
}

// Restart函数用于重启监听器的服务器
func Restart(listenerID uuid.UUID) messages.UserMessage {
    // 通过监听器ID获取监听器
    l, err := listeners.GetListenerByID(listenerID)
    // 如果获取监听器出错，则返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 重启监听器
    errRestart := l.Restart(l.GetConfiguredOptions())
    // 如果重启监听器出错，则返回错误消息
    if errRestart != nil {
        return messages.ErrorMessage(errRestart.Error())
    }

    // TODO 不确定这段代码应该放在这里还是放在listeners包中
    // 启动监听器的服务器
    go func() {
        err := l.Server.Start()
        if err != nil {
            messages.DelayedMessage(messages.ErrorMessage(err.Error()))
        }
    }()
    // 返回成功重启监听器的消息
    return messages.UserMessage{
        Level:   messages.Success,
        Time:    time.Now().UTC(),
        Message: fmt.Sprintf("%s listener was successfully restarted", l.Name),
        Error:   false,
    }
}

// SetOption函数用于设置可配置的监听器选项的值
func SetOption(listenerID uuid.UUID, Args []string) messages.UserMessage {
    // 通过监听器ID获取监听器
    l, err := listeners.GetListenerByID(listenerID)
    // 如果获取监听器出错，则返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    # 检查参数列表长度是否大于等于2
    if len(Args) >= 2 {
        # 遍历已配置的选项
        for k := range l.GetConfiguredOptions() {
            # 如果参数列表中的第二个参数与已配置选项中的某个选项相等
            if Args[1] == k {
                # 将参数列表中第三个参数开始的所有参数连接成一个字符串
                v := strings.Join(Args[2:], " ")
                # 设置选项的值，并返回可能出现的错误
                err := l.SetOption(k, v)
                # 如果出现错误，返回错误消息
                if err != nil {
                    return messages.ErrorMessage(err.Error())
                }
                # 设置成功，返回成功消息
                return messages.UserMessage{
                    Error:   false,
                    Level:   messages.Success,
                    Time:    time.Now().UTC(),
                    Message: fmt.Sprintf("set %s to: %s", k, v),
                }
            }
        }
    }
    # 参数不足，返回错误消息
    return messages.ErrorMessage(fmt.Sprintf("not enough arguments provided for the Listeners SetOption call: %s", Args))
// Start 方法运行 Listener 的服务器
func Start(name string) messages.UserMessage {
    // 通过名称获取 Listener 对象
    l, err := listeners.GetListenerByName(name)
    // 如果获取失败，返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 根据服务器状态进行不同的处理
    switch l.Server.Status() {
    // 如果服务器正在运行，返回消息提示服务器已经在运行
    case servers.Running:
        return messages.UserMessage{
            Error:   false,
            Level:   messages.Note,
            Time:    time.Now().UTC(),
            Message: "the server is already running",
        }
    // 如果服务器已经停止，启动服务器并返回成功消息
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
                l.Server.GetInterface(),
                l.Server.GetPort()),
        }
    // 如果服务器已经关闭，重新启动服务器并返回成功消息
    case servers.Closed:
        if err := l.Restart(l.GetConfiguredOptions()); err != nil {
            return messages.ErrorMessage(err.Error())
        }
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
            Message: fmt.Sprintf("Restarted %s %s listener on %s:%d", l.Name, servers.GetProtocol(l.Server.GetProtocol()),
                l.Server.GetInterface(),
                l.Server.GetPort()),
        }
    // 处理未知的服务器状态
    default:
        return messages.UserMessage{
            Error:   true,
            Level:   messages.Warn,
            Time:    time.Now().UTC(),
            Message: fmt.Sprintf("unhandled server status: %s", servers.GetStateString(l.Server.Status())),
        }
    }
}
// Stop terminates the Listener's server
func Stop(name string) messages.UserMessage {
    // 通过名称获取监听器对象
    l, err := listeners.GetListenerByName(name)
    // 如果获取监听器对象出错，则返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 如果监听器的服务器状态为运行中，则停止服务器
    if l.Server.Status() == servers.Running {
        err := l.Server.Stop()
        // 如果停止服务器出错，则返回错误消息
        if err != nil {
            return messages.ErrorMessage(err.Error())
        }
        // 返回成功消息
        return messages.UserMessage{
            Error:   false,
            Level:   messages.Success,
            Time:    time.Now().UTC(),
            Message: fmt.Sprintf("%s listener was stopped", l.Name),
        }
    }
    // 如果监听器的服务器状态不是运行中，则返回提示消息
    return messages.UserMessage{
        Error:   false,
        Level:   messages.Note,
        Time:    time.Now().UTC(),
        Message: "this listener is not running",
    }
}

// GetListenerStatus returns the Listener's server status
func GetListenerStatus(listenerID uuid.UUID) messages.UserMessage {
    // 通过监听器ID获取监听器对象
    l, err := listeners.GetListenerByID(listenerID)
    // 如果获取监听器对象出错，则返回错误消息
    if err != nil {
        return messages.ErrorMessage(err.Error())
    }
    // 返回监听器服务器状态消息
    return messages.UserMessage{
        Level:   messages.Plain,
        Message: servers.GetStateString(l.Server.Status()),
        Time:    time.Time{},
        Error:   false,
    }
}

// GetListenerByName return the unique identifier for an instantiated Listener by its name
func GetListenerByName(name string) (messages.UserMessage, uuid.UUID) {
    // 通过名称获取监听器对象
    l, err := listeners.GetListenerByName(name)
    // 如果获取监听器对象出错，则返回错误消息和空的UUID
    if err != nil {
        return messages.ErrorMessage(err.Error()), uuid.Nil
    }
    // 返回成功消息和监听器ID
    um := messages.UserMessage{
        Error: false,
        Time:  time.Now().UTC(),
    }
    return um, l.ID
}

// GetListenerConfiguredOptions enumerates all of a Listener's settings and returns them
func GetListenerConfiguredOptions(listenerID uuid.UUID) (messages.UserMessage, map[string]string) {
    // 通过监听器ID获取监听器对象
    l, err := listeners.GetListenerByID(listenerID)
    // 如果获取监听器对象出错，则返回错误消息和空的设置选项
    if err != nil {
        return messages.ErrorMessage(err.Error()), nil
    }
    # 创建一个名为um的UserMessage结构体实例，包含空消息、当前时间和错误标志为false
    um := messages.UserMessage{
        Message: "",
        Time:    time.Now().UTC(),
        Error:   false,
    }
    # 返回um实例和l.GetConfiguredOptions()的结果
    return um, l.GetConfiguredOptions()
// GetListeners 返回一个已实例化的 Listeners 列表
func GetListeners() []listeners.Listener {
    return listeners.GetListeners()
}

// GetListenerTypes 返回可用于 Listener 的支持服务器协议列表
func GetListenerTypes() []string {
    return listeners.GetListenerTypes()
}

// GetListenerOptions 返回基于提供的协议类型的未实例化监听器的所有可配置选项
func GetListenerOptions(protocol string) map[string]string {
    return listeners.GetListenerOptions(protocol)
}

// TODO 将 completers 移动到 CLI 包中

// GetListenerNamesCompleter 返回可用 Listeners 的 CLI tab 补全器
func GetListenerNamesCompleter() func(string) []string {
    return listeners.GetList()
}

// GetListenerOptionsCompleter 返回支持的 Listener 服务器协议的 CLI tab 补全器
func GetListenerOptionsCompleter(protocol string) func(string) []string {
    return listeners.GetListenerOptionsCompleter(protocol)
}

// GetListenerTypesCompleter 返回可用 Listener 类型的 CLI tab 补全器
func GetListenerTypesCompleter() func(string) []string {
    return listeners.GetListenerTypesCompleter()
}
```