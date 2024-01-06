# `kubesploit\pkg\servers\http2\http2.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包http2
// 导入 fmt 包，用于格式化输出
import (
	"fmt"
	// 导入 net/http 包，用于创建 HTTP 客户端或服务器
	"net/http"
	// 导入 strconv 包，用于字符串和数字之间的转换
	"strconv"
	// 导入 strings 包，用于处理字符串
	"strings"
	// 导入 time 包，用于处理时间
	"time"

	// 导入 golang.org/x/net/http2 包，用于支持特定的 HTTP/2 功能
	"golang.org/x/net/http2"
	// 导入 golang.org/x/net/http2/h2c 包，用于支持 HTTP/2 cleartext (h2c) 协议
	"golang.org/x/net/http2/h2c"
	// 导入 golang.org/x/sync/errgroup 包，用于处理并发错误
	"golang.org/x/sync/errgroup"

	// 导入 github.com/cretz/gopaque/gopaque 包，用于处理 gopaque 相关功能
	"github.com/cretz/gopaque/gopaque"
	// 导入 github.com/satori/go.uuid 包，用于生成 UUID
	uuid "github.com/satori/go.uuid"

	// 导入 kubesploit/pkg/api/messages 包，用于处理 API 消息
	"kubesploit/pkg/api/messages"
)
// 导入所需的包
"kubesploit/pkg/core"
"kubesploit/pkg/handlers"
"kubesploit/pkg/servers"
)

// Server 是用于 HTTP/2 clear-text (h2c) 服务器的结构体
type Server struct {
	servers.Server // 继承自 servers 包中的 Server 结构体
	urls []string    // 用于存储 URL 地址的切片
	ctx  *handlers.HTTPContext // 指向 handlers 包中 HTTPContext 结构体的指针
}

// Template 是用于收集创建实例所需信息的结构体，用于 New() 函数
type Template struct {
	servers.Template // 继承自 servers 包中的 Template 结构体
	X509Key  string // 用于 TLS 加密的 x.509 私钥
	X509Cert string // 用于 TLS 加密的 x.509 公钥
	PSK      string // 用于密码认证密钥交换 (PAKE) 前的预共享密钥密码
}
// init函数用于在servers包中注册该服务器类型
func init() {
	// 将"h2c"类型的服务器注册到RegisteredServers字典中
	servers.RegisteredServers["h2c"] = ""
}

// GetOptions函数返回一个可配置的服务器选项的映射，通常在创建监听器时使用
func GetOptions() map[string]string {
	// 创建一个字符串映射类型的options变量
	options := make(map[string]string)
	// 设置默认的服务器选项
	options["Interface"] = "127.0.0.1"
	options["Port"] = "80"
	options["PSK"] = "kubesploit"
	options["URLS"] = "/"
	return options
}

// New函数创建一个新的HTTP2服务器对象并返回一个指针
// 所有参数都作为字符串传入，并进行转换/验证
func New(options map[string]string) (*Server, error) {
	// 声明一个Server类型的变量s和一个error类型的变量err
	var s Server
	var err error
// 验证协议是否匹配，如果不匹配则返回错误
if strings.ToLower(options["Protocol"]) != "h2c" {
    return &s, fmt.Errorf("server protocol mismatch, expected: H2C got: %s", options["Protocol"])
}
// 设置服务器的协议为 H2C
s.Protocol = servers.H2C

// 将端口从字符串转换为整数
s.Port, err = strconv.Atoi(options["Port"])
if err != nil {
    return &s, fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
}

// 创建一个新的 ServeMux 对象
mux := http.NewServeMux()

// 解析 URL
if options["URLS"] == "" {
    // 如果 URLS 为空，则设置默认的 URL 为 "/"
    s.urls = []string{"/"}
} else {
    // 如果 URLS 不为空，则按逗号分隔并存储到 s.urls 中
    s.urls = strings.Split(options["URLS"], ",")
}
	}

	// 为每个 URL 添加代理处理程序
	if options["PSK"] == "" {
		return &s, fmt.Errorf("a Pre-Shared Key (PSK) password must be provided")
	}
	jwtKey := []byte(core.RandStringBytesMaskImprSrc(32)) // 用于签名和加密 JWT
	opaqueKey := gopaque.CryptoDefault.NewKey(nil)
	s.ctx = &handlers.HTTPContext{PSK: options["PSK"], JWTKey: jwtKey, OpaqueKey: opaqueKey}

	// 使用上下文添加处理程序
	for _, url := range s.urls {
		mux.HandleFunc(url, s.ctx.AgentHTTP)
	}

	h2s := &http2.Server{}
	s.Transport = &http.Server{
		Addr:           options["Interface"] + ":" + options["Port"], // 设置服务器地址和端口
		Handler:        h2c.NewHandler(mux, h2s), // 设置处理程序
		ReadTimeout:    10 * time.Second, // 设置读取超时时间
		WriteTimeout:   10 * time.Second,  // 设置写超时时间为10秒
		MaxHeaderBytes: 1 << 20,  // 设置最大请求头大小为1MB
	}

	s.Interface = options["Interface"]  // 从options中获取Interface并赋值给s的Interface属性
	s.ID = uuid.NewV4()  // 生成一个新的UUID并赋值给s的ID属性
	s.State = servers.Stopped  // 设置s的State属性为Stopped

	return &s, nil  // 返回生成的Server对象和nil的错误值
}

// Renew generates a new Server object and retains original encryption keys
func Renew(ctx handlers.ContextInterface, options map[string]string) (*Server, error) {
	tempServer, err := New(options)  // 调用New函数生成一个新的Server对象
	if err != nil {
		return tempServer, err  // 如果生成过程中出现错误，则返回错误
	}

	// Retain server's original JWT key used to sign and encrypt authorization JWT
	tempServer.ctx.JWTKey = ctx.(handlers.HTTPContext).JWTKey  // 保留服务器原始的JWT密钥用于签名和加密授权JWT
// 保留服务器在 OPAQUE 注册/授权中使用的原始 OPAQUE 密钥
tempServer.ctx.OpaqueKey = ctx.(handlers.HTTPContext).OpaqueKey

// GetConfiguredOptions 返回用户可以设置的服务器当前配置选项
func (s *Server) GetConfiguredOptions() map[string]string {
    options := make(map[string]string)
    options["Interface"] = s.Interface
    options["Port"] = fmt.Sprintf("%d", s.Port)
    options["Protocol"] = servers.GetProtocol(s.Protocol)
    options["PSK"] = s.ctx.PSK
    options["URLS"] = strings.Join(s.urls, " ")

    return options
}

// GetContext 返回服务器当前的上下文信息，如加密密钥
// GetContext函数返回服务器的上下文接口
func (s *Server) GetContext() handlers.ContextInterface {
	return *s.ctx
}

// GetInterface函数返回服务器绑定的接口
func (s *Server) GetInterface() string {
	return s.Interface
}

// GetPort函数返回服务器绑定的端口
func (s *Server) GetPort() int {
	return s.Port
}

// GetProtocol函数返回服务器的协议作为服务器包中常量的整数
func (s *Server) GetProtocol() int {
	return s.Protocol
}

// GetProtocolString函数返回服务器的协议
// GetProtocolString returns the protocol string based on the server's Protocol value
func (s *Server) GetProtocolString() string {
	// Switch statement to check the value of s.Protocol and return the corresponding protocol string
	switch s.Protocol {
	case servers.H2C:
		return "H2C"
	default:
		return "UNKNOWN"
	}
}

// SetOption sets an option for an instantiated server object
func (s *Server) SetOption(option string, value string) error {
	var err error
	// Check non-string options first
	switch strings.ToLower(option) {
	// Case for "interface" option
	case "interface":
		// Set the server's Interface value to the provided value
		s.Interface = value
	// Case for "port" option
	case "port":
		// Convert the provided value to an integer and set it as the server's Port value
		s.Port, err = strconv.Atoi(value)
		// Check for any errors during the conversion
		if err != nil {
			// Return an error message if there was an error converting the port number to an integer
			return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
		}
		}
	case "protocol":
		// 如果尝试更改协议，则返回错误
		return fmt.Errorf("the protocol can not be changed; create a new listener instead")
	case "psk":
		// 设置预共享密钥
		s.ctx.PSK = value
	case "urls":
		// 将逗号分隔的字符串分割成字符串数组
		s.urls = strings.Split(value, ",")
	default:
		// 如果选项无效，则返回错误
		return fmt.Errorf("invalid option: %s", option)
	}
	// 返回空值
	return nil
}

// Start the HTTP2 server
// 启动 HTTP2 服务器
func (s *Server) Start() error {
	// 定义错误组
	var g errgroup.Group

	// 捕获异常
	defer func() {
		// 如果有异常发生，则执行恢复操作
		if r := recover(); r != nil {
		// 格式化错误信息，包括服务器协议、接口和端口，以及错误信息
		m := fmt.Sprintf("The %s server on %s:%d paniced:\r\n%v+\r\n", servers.GetProtocol(s.GetProtocol()), s.Interface, s.Port, r.(error))
		// 发送广播消息，包括警告级别、消息内容、时间和错误标志
		messages.SendBroadcastMessage(messages.UserMessage{
			Level:   messages.Warn,
			Message: m,
			Time:    time.Now().UTC(),
			Error:   true,
		})
	}()

	// 启动服务器
	g.Go(func() error {
		s.State = servers.Running
		return s.Transport.(*http.Server).ListenAndServe()
	})

	// 等待所有任务完成
	if err := g.Wait(); err != nil {
		// 如果出现错误并且不是服务器关闭引起的错误，则将服务器状态设置为错误，并返回错误信息
		if err != http.ErrServerClosed {
			s.State = servers.Error
			return fmt.Errorf("there was an error with the %s server on %s:%d %s", s.GetProtocolString(), s.Interface, s.Port, err.Error())
		}
	}
	return nil
}

// Status enumerates if the server is currently running or stopped and returns the value as a string
// Status 方法返回服务器当前的状态（运行或停止）并将值作为字符串返回
func (s *Server) Status() int {
	// 返回服务器的状态值
	return s.State
}

// Stop the HTTP2 server
// Stop 方法停止 HTTP2 服务器
func (s *Server) Stop() error {
	// Shutdown gracefully shuts down the server without interrupting any active connections.
	// This will keep the agent checking in since it is an active connection
	//err = s.Transport.(*http.Server).Shutdown(context.Background())
	// Close immediately closes all active net.Listeners and any connections in state StateNew, StateActive, or StateIdle.
	// 关闭所有活动的 net.Listeners 和处于 StateNew、StateActive 或 StateIdle 状态的连接
	err := s.Transport.(*http.Server).Close()
	if err != nil {
		// 如果关闭过程中出现错误，返回错误信息
		return fmt.Errorf("there was an error stopping the HTTP server:\r\n%s", err.Error())
	}
	// 更新服务器状态为 Closed
	s.State = servers.Closed
# 返回空值，表示没有返回任何有效的数据。
```