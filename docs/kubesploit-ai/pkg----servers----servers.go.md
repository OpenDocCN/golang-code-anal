# `kubesploit\pkg\servers\servers.go`

```go
// Kubesploit是基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架的一部分。
// 本文件是Kubesploit的一部分。
// 版权所有©2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参阅<http://www.gnu.org/licenses/>。

package servers

import (
    // 第三方
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/handlers"
)

const (
    // 支持的协议

    // HTTP是HTTP/1.1明文协议
    HTTP int = 1
    // HTTPS是HTTP/1.1安全（通过SSL/TLS）协议
    HTTPS int = 2
    // H2C是HTTP/2.0明文协议
    H2C int = 3
    // HTTP2是HTTP/2.0安全（通过SSL/TLS）协议
    HTTP2 int = 4
    // HTTP3是通过Quick UDP Internet Connection（QUIC）的HTTP/2.0安全协议
    HTTP3 int = 5
    // DNS是域名服务协议
    DNS int = 6

    // 服务器状态

    // 停止是服务器从未启动时的状态
    Stopped int = 0
    // 运行表示服务器正在积极接受连接并提供内容
    Running int = 1
    // 错误在操作服务器时出现错误时使用
    Error int = 2
    // 关闭在服务器运行但已停止时使用；它不能再次重用
    Closed int = 3
)
// RegisteredServers包含注册的监听器类型的数组
var RegisteredServers = make(map[string]string) // TODO 目前不确定该值该如何处理，可能会更改类型

// ServerInterface用于提供服务器模块必须支持的一组标准方法，以便与Merlin一起使用
type ServerInterface interface {
    GetConfiguredOptions() map[string]string
    GetContext() handlers.ContextInterface
    GetInterface() string
    GetProtocol() int
    GetProtocolString() string
    GetPort() int
    SetOption(string, string) error
    Start() error
    Status() int
    Stop() error
}

// Server结构用于提供服务器模块必须支持的一组标准字段，以便与Merlin一起使用
type Server struct {
    ServerInterface
    ID        uuid.UUID   // 服务器对象的唯一标识符
    Transport interface{} // 将用于发送和接收流量的服务器或传输
    Interface string      // 服务器将在其上监听的网络适配器接口
    Port      int         // 服务器将监听的端口
    Protocol  int         // 服务器将使用的协议（例如来自servers包的HTTP/2或HTTP/3）
    State     int
}

// Template是一个用于收集创建新服务器实例所需信息的结构
type Template struct {
    Interface string
    Port      string
    Protocol  string
}

// GetProtocol用于将服务器协议常量转换为字符串，以便在书面消息或日志中使用
func GetProtocol(protocol int) string {
    switch protocol {
    case HTTP:
        return "HTTP"
    case HTTPS:
        return "HTTPS"
    case H2C:
        return "H2C"
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

// GetStateString用于将服务器状态常量转换为字符串，以便在书面消息或日志中使用
# 根据状态值返回对应的状态字符串
func GetStateString(state int) string:
    # 使用 switch 语句根据状态值进行匹配
    switch state:
        # 如果状态值为 Stopped，则返回 "Stopped"
        case Stopped:
            return "Stopped"
        # 如果状态值为 Running，则返回 "Running"
        case Running:
            return "Running"
        # 如果状态值为 Error，则返回 "Error"
        case Error:
            return "Error"
        # 如果状态值为 Closed，则返回 "Closed"
        case Closed:
            return "Closed"
        # 如果状态值为其他值，则返回 "Undefined"
        default:
            return "Undefined"
```