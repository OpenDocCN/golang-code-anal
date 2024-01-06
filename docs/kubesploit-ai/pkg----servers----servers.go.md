# `kubesploit\pkg\servers\servers.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括适销性或特定用途的隐含担保。有关更多详细信息，请参阅GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

// 服务器包
package servers
// 导入第三方库
import (
	// 使用 github.com/satori/go.uuid 库生成唯一标识符
	uuid "github.com/satori/go.uuid"

	// 导入自定义包
	"kubesploit/pkg/handlers"
)

// 定义常量
const (
	// 支持的协议

	// HTTP 是 HTTP/1.1 明文协议
	HTTP int = 1
	// HTTPS 是 HTTP/1.1 安全协议（通过 SSL/TLS）
	HTTPS int = 2
	// H2C 是 HTTP/2.0 明文协议
	H2C int = 3
	// HTTP2 是 HTTP/2.0 安全协议（通过 SSL/TLS）
	HTTP2 int = 4
	// HTTP3 是 HTTP/2.0 安全协议（通过 Quick UDP Internet Connection (QUIC)）
	HTTP3 int = 5
)
	// 定义常量 HTTP3，赋值为 5
	HTTP3 int = 5
	// 定义常量 DNS，赋值为 6，表示域名服务协议
	DNS int = 6

	// 服务器状态

	// 停止状态，当服务器从未启动时的状态
	Stopped int = 0
	// 运行状态，表示服务器正在接受连接并提供内容
	Running int = 1
	// 错误状态，表示服务器操作发生错误
	Error int = 2
	// 关闭状态，表示服务器曾经运行但已经停止，无法再次重用
	Closed int = 3
)

// RegisteredServers 包含注册的监听器类型的数组
var RegisteredServers = make(map[string]string) // TODO not sure what to do with the value just yet, might change type

// ServerInterface 用于提供服务器模块必须支持的一组标准方法，以便与 Merlin 协作
// ServerInterface 定义了服务器接口的方法
type ServerInterface interface {
    // GetConfiguredOptions 返回配置选项的映射
    GetConfiguredOptions() map[string]string
    // GetContext 返回服务器的上下文接口
    GetContext() handlers.ContextInterface
    // GetInterface 返回服务器的网络接口
    GetInterface() string
    // GetProtocol 返回服务器的协议
    GetProtocol() int
    // GetProtocolString 返回服务器的协议字符串
    GetProtocolString() string
    // GetPort 返回服务器监听的端口
    GetPort() int
    // SetOption 设置服务器的选项
    SetOption(string, string) error
    // Start 启动服务器
    Start() error
    // Status 返回服务器的状态
    Status() int
    // Stop 停止服务器
    Stop() error
}

// Server 结构体用于提供服务器模块必须支持的一组标准字段，以便与 Merlin 一起使用
type Server struct {
    ServerInterface
    ID        uuid.UUID   // 服务器对象的唯一标识符
    Transport interface{} // 用于发送和接收流量的服务器或传输
    Interface string      // 服务器将监听的网络适配器接口
    Port      int         // 服务器将监听的端口
}
// Protocol 是服务器将使用的协议（即 HTTP/2 或 HTTP/3）的常量，来自服务器包
// State 是服务器的状态
type Server struct {
    Protocol  int         // 服务器将使用的协议（即 HTTP/2 或 HTTP/3）的常量，来自服务器包
    State     int
}

// Template 是用于收集创建新服务器实例所需信息的结构
type Template struct {
    Interface string  // 服务器接口
    Port      string  // 服务器端口
    Protocol  string  // 服务器协议
}

// GetProtocol 用于将服务器协议常量转换为字符串，以便在书面消息或日志中使用
func GetProtocol(protocol int) string {
    switch protocol {
    case HTTP:
        return "HTTP"
    case HTTPS:
        return "HTTPS"
    case H2C:
        return "H2C"
    }
}
// 根据常量值返回对应的协议名称字符串
func GetProtocolString(protocol int) string {
    switch protocol {
    case HTTP2:
        return "HTTP2"
    case HTTP3:
        return "HTTP3"
    case DNS:
        return "DNS"
    default:
        return "invalid protocol"
    }
}

// GetStateString 用于将服务器状态常量转换为字符串，以便在书面消息或日志中使用
func GetStateString(state int) string {
    switch state {
    case Stopped:
        return "Stopped"
    case Running:
        return "Running"
    case Error:
        return "Error"
    }
}
# 根据枚举类型的不同值返回不同的字符串
case Closed:
    # 如果枚举值为Closed，返回字符串"Closed"
    return "Closed"
default:
    # 如果枚举值为其他值，返回字符串"Undefined"
    return "Undefined"
}
```