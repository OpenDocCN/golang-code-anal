# `kubesploit\pkg\listeners\listeners.go`

```go
// Kubesploit是基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架的一部分。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package listeners

import (
    // 标准库
    "fmt"
    "strings"

    // 第三方库
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/servers"
    "kubesploit/pkg/servers/http"
    "kubesploit/pkg/servers/http2"
    "kubesploit/pkg/servers/http3"
)

// Listeners包含所有实例化的Listener对象
var Listeners = make(map[uuid.UUID]*Listener)

// Listener是一个带有嵌入式Server对象的Merlin监听器的结构
type Listener struct {
    ID          uuid.UUID               // Listener对象的唯一标识符
    Name        string                  // 监听器的名称
    Description string                  // 监听器的描述
    Server      servers.ServerInterface // 与服务器对象交互的接口
}

// New实例化一个Listener对象
func New(options map[string]string) (*Listener, error) {
    var listener Listener
    var err error

    // 确保提供了监听器名称
    listener.Name = options["Name"]
    // 如果监听器名称为空，则返回错误
    if listener.Name == "" {
        return &listener, fmt.Errorf("a listener name must be provided")
    }
    // 确保不存在具有相同名称的监听器
    if Exists(listener.Name) {
        return &listener, fmt.Errorf("a listener with this name already exists")
    }

    // 为监听器获取一个新的服务器对象
    switch strings.ToLower(options["Protocol"]) {
    case "":
        return &listener, fmt.Errorf("a listener protocol must be provided")
    case "http", "https", "http2":
        listener.Server, err = http.New(options)
    case "h2c":
        listener.Server, err = http2.New(options)
    case "http3":
        listener.Server, err = http3.New(options)

    default:
        err = fmt.Errorf("invalid listener server type: %s", options["Protocol"])
    }

    // 如果出现错误，则返回错误
    if err != nil {
        return &listener, err
    }

    // 为监听器设置新的唯一标识符和描述
    listener.ID = uuid.NewV4()
    listener.Description = options["Description"]

    // 将监听器添加到监听器列表中
    Listeners[listener.ID] = &listener

    // 返回监听器和空错误
    return &listener, nil
// GetConfiguredOptions返回服务器当前配置的选项，这些选项可以由用户设置
func (l *Listener) GetConfiguredOptions() map[string]string {
    // 获取服务器配置的选项
    options := l.Server.GetConfiguredOptions()
    // 设置选项中的名称、描述和ID
    options["Name"] = l.Name
    options["Description"] = l.Description
    options["ID"] = l.ID.String()
    return options
}

// Restart创建一个新的服务器实例，因为停止后的http服务器无法被重用
func (l *Listener) Restart(options map[string]string) error {
    var err error

    // 停止运行中的实例
    if l.Server.Status() == servers.Running {
        if err = l.Server.Stop(); err != nil {
            return err
        }
    }

    // 创建一个新的实例
    switch l.Server.GetProtocol() {
    case servers.HTTP, servers.HTTPS, servers.HTTP2:
        l.Server, err = http.Renew(l.Server.GetContext(), options)
    case servers.H2C:
        l.Server, err = http2.Renew(l.Server.GetContext(), options)
    case servers.HTTP3:
        l.Server, err = http3.Renew(l.Server.GetContext(), options)
    default:
        err = fmt.Errorf("invalid server protocol: %d (%s)", l.Server.GetProtocol(), servers.GetProtocol(l.Server.GetProtocol()))
    }
    return err
}

// SetOption在Listener上设置可配置选项的值
func (l *Listener) SetOption(option string, value string) error {
    switch strings.ToLower(option) {
    case "name":
        l.Name = value
    case "description":
        l.Description = value
    default:
        return l.Server.SetOption(option, value)
    }
    return nil
}

// GetList返回存在的Listener列表，用于命令行选项的自动补全
func GetList() func(string) []string {
    return func(line string) []string {
        listeners := make([]string, 0)
        for listenerUUID := range Listeners {
            listeners = append(listeners, Listeners[listenerUUID].Name)
        }
        return listeners
    }
}
// 通过ID（UUIDv4）查找并返回已实例化的监听器对象的指针
func GetListenerByID(id uuid.UUID) (*Listener, error) {
    // 检查ID是否存在于Listeners映射中
    l, exists := Listeners[id]
    // 如果ID不存在，则返回空的Listener指针和错误信息
    if !exists {
        return &Listener{}, fmt.Errorf(fmt.Sprintf("a listener with an ID of %s does not exist", id))
    }
    // 如果ID存在，则返回对应的Listener指针和nil错误
    return l, nil
}

// 通过名称（字符串）查找并返回已实例化的监听器对象的指针
func GetListenerByName(name string) (*Listener, error) {
    // 检查名称是否存在
    if !Exists(name) {
        return &Listener{}, fmt.Errorf("%s listener does not exist", name)
    }
    var listener *Listener
    // 遍历Listeners映射，找到与名称匹配的监听器对象
    for k, v := range Listeners {
        if name == v.Name {
            listener = Listeners[k]
            break
        }
    }
    // 返回找到的监听器对象的指针和nil错误
    return listener, nil
}

// 获取所有可配置模块选项的映射
func GetListenerOptions(protocol string) map[string]string {
    var options map[string]string
    // 根据协议类型选择相应的选项
    switch strings.ToLower(protocol) {
    case "http", "https", "http2":
        options = http.GetOptions(strings.ToLower(protocol))
    case "h2c":
        options = http2.GetOptions()
    case "http3":
        options = http3.GetOptions()
    default:
        options = make(map[string]string)
    }

    // 设置默认的选项值
    options["Name"] = "Default"
    options["Description"] = "Default listener"
    // 返回选项映射
    return options
}

// 获取用于CLI选项补全的监听器选项数组
func GetListenerOptionsCompleter(protocol string) func(string) []string {
    # 定义一个匿名函数，接受一个字符串参数，返回一个字符串数组
    return func(line string) []string {
        # 声明一个空的字符串到字符串的映射
        var serverOptions map[string]string
        # 声明一个空的字符串数组
        options := make([]string, 0)
        # 根据协议名称的小写形式进行不同的处理
        switch strings.ToLower(protocol) {
        # 如果协议是 http、https 或者 http2，则获取对应的服务器选项
        case "http", "https", "http2":
            serverOptions = http.GetOptions(strings.ToLower(protocol))
        # 如果协议是 h2c，则获取对应的服务器选项
        case "h2c":
            serverOptions = http2.GetOptions()
        # 如果协议是 http3，则获取对应的服务器选项
        case "http3":
            serverOptions = http3.GetOptions()
        # 如果协议不在以上列举的情况中，则创建一个空的服务器选项映射
        default:
            serverOptions = make(map[string]string)
        }
        # 遍历服务器选项映射，将键添加到选项数组中
        for k := range serverOptions {
            options = append(options, k)
        }
        # 添加固定的字符串 "Name" 到选项数组中
        options = append(options, "Name")
        # 添加固定的字符串 "Description" 到选项数组中
        options = append(options, "Description")
        # 返回选项数组
        return options
    }
// GetListenerTypesCompleter 返回 Merlin 支持的 CLI tab 补全的监听器类型列表
func GetListenerTypesCompleter() func(string) []string {
    return func(line string) []string {
        return GetListenerTypes()
    }
}

// GetListenerTypes 返回 Merlin 支持的监听器类型列表
func GetListenerTypes() []string {
    var t []string
    for v := range servers.RegisteredServers {
        t = append(t, v)
    }
    return t
}

// Exists 确定监听器是否已经被实例化
func Exists(name string) bool {
    for _, v := range Listeners {
        if name == v.Name {
            return true
        }
    }
    return false
}

// RemoveByID 根据输入的 UUID 从全局的监听器列表中删除一个监听器
func RemoveByID(id uuid.UUID) error {
    if _, ok := Listeners[id]; ok {
        err := Listeners[id].Server.Stop()
        if err != nil {
            return err
        }
        delete(Listeners, id)
        return nil
    }
    return fmt.Errorf("无法移除监听器：%s，因为它不存在", id)
}

// GetListeners 用于返回一个监听器对象列表，供客户端应用程序使用
func GetListeners() []Listener {
    var listeners []Listener
    for id := range Listeners {
        listeners = append(listeners, *Listeners[id])
    }
    return listeners
}
```