# `kubesploit\pkg\listeners\listeners.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其中包括许可证的第3版或任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包声明
package listeners
// 导入标准库中的 fmt 和 strings 包
import (
	"fmt"
	"strings"
	
	// 导入第三方库中的 uuid 包
	uuid "github.com/satori/go.uuid"
	
	// 导入自定义的 Merlin 包中的服务器相关模块
	"kubesploit/pkg/servers"
	"kubesploit/pkg/servers/http"
	"kubesploit/pkg/servers/http2"
	"kubesploit/pkg/servers/http3"
)

// Listeners 是一个包含所有已实例化的 Listener 对象的映射
var Listeners = make(map[uuid.UUID]*Listener)

// Listener 是一个用于创建 Merlin 监听器的结构，包含一个嵌入的 Server 对象
type Listener struct {
// ID: 用于存储 Listener 对象的唯一标识符
// Name: 存储监听器的名称
// Description: 存储监听器的描述信息
// Server: 与服务器对象交互的接口

// New: 实例化一个 Listener 对象
func New(options map[string]string) (*Listener, error) {
    var listener Listener
    var err error

    // 确保提供了监听器名称
    listener.Name = options["Name"]
    if listener.Name == "" {
        return &listener, fmt.Errorf("必须提供监听器名称")
    }
    // 确保不存在具有此名称的监听器
    if Exists(listener.Name) {
        return &listener, fmt.Errorf("具有此名称的监听器已存在")
    }
}
// 根据传入的协议类型创建一个新的服务器对象
switch strings.ToLower(options["Protocol"]) {
    // 如果协议为空，则返回错误信息
    case "":
        return &listener, fmt.Errorf("a listener protocol must be provided")
    // 如果协议为 http、https 或 http2，则创建一个新的 http 服务器对象
    case "http", "https", "http2":
        listener.Server, err = http.New(options)
    // 如果协议为 h2c，则创建一个新的 http2 服务器对象
    case "h2c":
        listener.Server, err = http2.New(options)
    // 如果协议为 http3，则创建一个新的 http3 服务器对象
    case "http3":
        listener.Server, err = http3.New(options)
    // 如果协议类型不在上述范围内，则返回错误信息
    default:
        err = fmt.Errorf("invalid listener server type: %s", options["Protocol"])
}

// 如果创建服务器对象时发生错误，则返回错误信息
if err != nil {
    return &listener, err
}
// 为 listener 对象生成一个新的 UUID
listener.ID = uuid.NewV4()
// 从 options 参数中获取描述信息并赋值给 listener 对象的 Description 属性
listener.Description = options["Description"]

// 将 listener 对象存储到全局的 Listeners 列表中
Listeners[listener.ID] = &listener

// 返回 listener 对象和空错误
return &listener, nil
}

// GetConfiguredOptions 返回服务器当前配置的选项，这些选项可以由用户设置
func (l *Listener) GetConfiguredOptions() map[string]string {
// 从服务器对象中获取已配置的选项
options := l.Server.GetConfiguredOptions()
// 将 listener 对象的名称、描述和 ID 转换为字符串并存储到 options 中
options["Name"] = l.Name
options["Description"] = l.Description
options["ID"] = l.ID.String()
// 返回包含配置选项的 map
return options
}

// Restart 创建一个新的服务器实例，因为 HTTP 服务器在停止后无法被重用
func (l *Listener) Restart(options map[string]string) error {
// 声明一个错误变量
var err error
// 停止运行中的实例
if l.Server.Status() == servers.Running {
    // 如果实例正在运行，则停止它
    if err = l.Server.Stop(); err != nil {
        return err
    }
}

// 创建一个新的实例
switch l.Server.GetProtocol() {
    // 根据服务器协议类型选择相应的 Renew 方法进行实例的更新
    case servers.HTTP, servers.HTTPS, servers.HTTP2:
        l.Server, err = http.Renew(l.Server.GetContext(), options)
    case servers.H2C:
        l.Server, err = http2.Renew(l.Server.GetContext(), options)
    case servers.HTTP3:
        l.Server, err = http3.Renew(l.Server.GetContext(), options)
    // 如果协议类型不在已知范围内，则返回错误
    default:
        err = fmt.Errorf("invalid server protocol: %d (%s)", l.Server.GetProtocol(), servers.GetProtocol(l.Server.GetProtocol()))
}
// 返回可能存在的错误
return err
// SetOption函数用于设置Listener的可配置选项的值
func (l *Listener) SetOption(option string, value string) error {
    // 使用switch语句根据option的值进行不同的操作
    switch strings.ToLower(option) {
    case "name":
        // 如果option为"name"，则设置Listener的Name属性为value
        l.Name = value
    case "description":
        // 如果option为"description"，则设置Listener的Description属性为value
        l.Description = value
    default:
        // 如果option不是"name"或"description"，则调用Server的SetOption方法进行处理
        return l.Server.SetOption(option, value)
    }
    // 返回nil表示操作成功
    return nil
}

// GetList函数返回一个用于命令行自动补全的Listener列表
func GetList() func(string) []string {
    return func(line string) []string {
        // 创建一个空的Listener列表
        listeners := make([]string, 0)
        // 遍历Listeners中的所有listenerUUID
        for listenerUUID := range Listeners {
// 将监听器的名称添加到监听器列表中
listeners = append(listeners, Listeners[listenerUUID].Name)

// 返回监听器列表
return listeners
}

// 通过ID（UUIDv4）查找并返回指向已实例化监听器对象的指针
func GetListenerByID(id uuid.UUID) (*Listener, error) {
	// 检查监听器是否存在
	l, exists := Listeners[id]
	if !exists {
		// 如果监听器不存在，则返回空的监听器对象和错误信息
		return &Listener{}, fmt.Errorf(fmt.Sprintf("a listener with an ID of %s does not exist", id))
	}
	// 如果监听器存在，则返回监听器对象和空的错误信息
	return l, nil
}

// 通过名称（字符串）查找并返回指向已实例化监听器对象的指针
func GetListenerByName(name string) (*Listener, error) {
	// 检查监听器是否存在
	if !Exists(name) {
		// 如果监听器不存在，则返回空的监听器对象和错误信息
		return &Listener{}, fmt.Errorf("%s listener does not exist", name)
	}
	// 如果监听器存在，则返回监听器对象和空的错误信息
	}
	// 声明一个指向 Listener 结构体的指针变量
	var listener *Listener
	// 遍历 Listeners 切片
	for k, v := range Listeners {
		// 如果输入的协议名与 Listeners 中的协议名匹配，则将对应的 Listener 赋值给 listener，并跳出循环
		if name == v.Name {
			listener = Listeners[k]
			break
		}
	}
	// 返回找到的 listener 和空错误
	return listener, nil
}

// GetListenerOptions 获取所有可配置模块选项的映射
func GetListenerOptions(protocol string) map[string]string {
	// 声明一个字符串到字符串的映射
	var options map[string]string
	// 根据协议名进行不同的操作
	switch strings.ToLower(protocol) {
	case "http", "https", "http2":
		// 如果协议是 http、https 或 http2，则获取对应协议的选项
		options = http.GetOptions(strings.ToLower(protocol))
	case "h2c":
		// 如果协议是 h2c，则获取 h2c 的选项
		options = http2.GetOptions()
	case "http3":
	// 创建一个http3的选项对象
	options = http3.GetOptions()
	// 默认情况下，创建一个空的map作为选项对象
	default:
		options = make(map[string]string)
	}

	// 设置选项对象的Interface属性为"10.240.0.20"
	options["Interface"] = "10.240.0.20"
	// 设置选项对象的Name属性为"Default"
	options["Name"] = "Default"
	// 设置选项对象的Description属性为"Default listener"
	options["Description"] = "Default listener"
	// 返回设置好的选项对象
	return options
}

// GetListenerOptionsCompleter函数用于获取用于CLI选项补全的监听器选项数组
func GetListenerOptionsCompleter(protocol string) func(string) []string {
	// 返回一个匿名函数，该函数接受一个字符串参数并返回一个字符串数组
	return func(line string) []string {
		// 声明一个serverOptions的map
		var serverOptions map[string]string
		// 声明一个空的字符串数组options
		options := make([]string, 0)
		// 根据协议类型设置serverOptions
		switch strings.ToLower(protocol) {
		case "http", "https", "http2":
			serverOptions = http.GetOptions(strings.ToLower(protocol))
		case "h2c":
// 获取服务器选项，根据不同的协议类型调用对应的GetOptions函数
serverOptions = http2.GetOptions()
// 根据不同的协议类型调用对应的GetOptions函数
case "http3":
    serverOptions = http3.GetOptions()
// 默认情况下，创建一个空的map
default:
    serverOptions = make(map[string]string)
}
// 遍历serverOptions，将键添加到options切片中
for k := range serverOptions {
    options = append(options, k)
}
// 将"Name"和"Description"添加到options切片中
options = append(options, "Name")
options = append(options, "Description")
// 返回options切片
return options
}

// GetListenerTypesCompleter返回一个函数，该函数返回Merlin支持的CLI选项的列表
func GetListenerTypesCompleter() func(string) []string {
    return func(line string) []string {
        // 调用GetListenerTypes函数返回监听器类型列表
        return GetListenerTypes()
    }
}
// GetListenerTypes 返回 Merlin 支持的监听器类型列表
func GetListenerTypes() []string {
    // 声明一个字符串切片
    var t []string
    // 遍历已注册的服务器，将监听器类型添加到切片中
    for v := range servers.RegisteredServers {
        t = append(t, v)
    }
    // 返回监听器类型切片
    return t
}

// Exists 确定监听器是否已经被实例化
func Exists(name string) bool {
    // 遍历监听器列表，如果名称匹配则返回 true
    for _, v := range Listeners {
        if name == v.Name {
            return true
        }
    }
    // 如果没有匹配的监听器名称，则返回 false
    return false
}
// RemoveByID 根据输入的 UUID 从全局的 Listeners 列表中删除一个 Listener
func RemoveByID(id uuid.UUID) error {
	// 检查是否存在指定的 UUID 对应的 Listener
	if _, ok := Listeners[id]; ok {
		// 停止对应的 Server
		err := Listeners[id].Server.Stop()
		if err != nil {
			return err
		}
		// 从 Listeners 列表中删除指定的 UUID 对应的 Listener
		delete(Listeners, id)
		return nil
	}
	// 如果不存在指定的 UUID 对应的 Listener，则返回错误信息
	return fmt.Errorf("could not remove listener: %s because it does not exist", id)
}

// GetListeners 用于返回一个 Listener 对象列表，以供客户端应用程序使用
func GetListeners() []Listener {
	var listeners []Listener
	// 遍历 Listeners 列表，将每个 Listener 对象添加到返回的列表中
	for id := range Listeners {
		listeners = append(listeners, *Listeners[id])
	}
}
# 返回变量 listeners 的值。
```