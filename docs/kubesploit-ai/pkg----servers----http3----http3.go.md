# `kubesploit\pkg\servers\http3\http3.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何担保；包括对适销性或特定用途的隐含担保。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参阅<http://www.gnu.org/licenses/>。

// 包http3
# 导入标准库中的模块
import (
	"crypto/tls"  # 加密传输层安全协议
	"fmt"  # 格式化输出
	"net/http"  # HTTP 客户端和服务器
	"path/filepath"  # 文件路径操作
	"strconv"  # 字符串和基本数据类型之间的转换
	"strings"  # 字符串操作
	"time"  # 时间操作

	// 导入第三方库中的模块
	"golang.org/x/sync/errgroup"  # 并发控制

	// 导入第三方库中的模块
	"github.com/cretz/gopaque/gopaque"  # 加密协议
	"github.com/lucas-clemente/quic-go"  # QUIC 协议
	"github.com/lucas-clemente/quic-go/http3"  # HTTP/3 协议
	uuid "github.com/satori/go.uuid"  # UUID 生成

	// 导入 Merlin 模块
// 导入所需的包
"kubesploit/pkg/api/messages"
"kubesploit/pkg/core"
"kubesploit/pkg/handlers"
"kubesploit/pkg/servers"
"kubesploit/pkg/util"
)

// Server 是用于 HTTP3 服务器的结构体
type Server struct {
	servers.Server  // 继承自 servers 包中的 Server 结构体
	x509Cert string  // x.509 证书
	x509Key  string  // x.509 私钥
	urls     []string  // URL 列表
	ctx      *handlers.HTTPContext  // HTTP 上下文
}

// Template 是用于收集创建实例所需信息的结构体，用于 New() 函数
type Template struct {
	servers.Template  // 继承自 servers 包中的 Template 结构体
	X509Key  string  // 用于 TLS 加密的 x.509 私钥
// X509Cert string // 用于 TLS 加密的 x.509 公钥
// PSK string // 在密码认证密钥交换（PAKE）之前使用的预共享密钥密码

// init 函数将此服务器类型注册到 servers 包中
func init() {
    servers.RegisteredServers["http3"] = ""
}

// GetOptions 函数返回一个可配置的服务器选项映射，通常在创建监听器时使用
func GetOptions() map[string]string {
    options := make(map[string]string)
    options["Interface"] = "127.0.0.1"
    options["Port"] = "443"
    options["PSK"] = "merlin"
    options["URLS"] = "/"
    options["X509Cert"] = filepath.Join(core.CurrentDir, "data", "x509", "server.crt")
    options["X509Key"] = filepath.Join(core.CurrentDir, "data", "x509", "server.key")
    return options
}
// New函数创建一个新的HTTP3服务器对象并返回一个指针
// 所有参数都作为字符串传入，并被转换/验证
func New(options map[string]string) (*Server, error) {
	var s Server
	var certificates *tls.Certificate
	var err error

	// 验证协议是否匹配
	if strings.ToLower(options["Protocol"]) != "http3" {
		return &s, fmt.Errorf("服务器协议不匹配，期望: HTTP3 实际: %s", options["Protocol"])
	}
	s.Protocol = servers.HTTP3

	// 将端口从字符串转换为整数
	s.Port, err = strconv.Atoi(options["Port"])
	if err != nil {
		return &s, fmt.Errorf("将端口号转换为整数时出错: %s", err.Error())
	}
// 验证 X509 Key 文件是否存在并且可以被解析
certificates, err = util.GetTLSCertificates(options["X509Cert"], options["X509Key"])
if err != nil {
    // 如果证书文件不存在，发送提示消息并创建一个用于本次会话的内存中的 x.509 证书
    m := fmt.Sprintf("Certificate was not found at: %s\r\n", options["X509Cert"])
    m += "Creating in-memory x.509 certificate used for this session only"
    messages.SendBroadcastMessage(messages.UserMessage{
        Level:   messages.Note,
        Message: m,
        Time:    time.Now().UTC(),
        Error:   false,
    })
    certificates, err = util.GenerateTLSCert(nil, nil, nil, nil, nil, nil, true)
    if err != nil {
        return &s, err
    }
    // 留空以强制使用服务器的 TLS 配置
    s.x509Cert = ""
    s.x509Key = ""
} else {
    // 如果证书文件存在，设置服务器的 x509Cert 为证书文件路径
    s.x509Cert = options["X509Cert"]
}
		# 将 options 字典中的 "X509Key" 值赋给 s.x509Key
		s.x509Key = options["X509Key"]
	}
	# 检查证书是否存在不安全的指纹
	insecure, errI := util.CheckInsecureFingerprint(*certificates)
	# 如果检查出错，返回错误
	if errI != nil {
		return &s, errI
	}

	# 如果存在不安全的指纹
	if insecure {
		# 构建警告消息
		m := fmt.Sprintf("Insecure publicly distributed Merlin x.509 testing certificate in use for HTTP/3 server on %s:%s\r\n", options["Interface"], options["Port"])
		m += "Additional details: https://merlin-c2.readthedocs.io/en/latest/server/x509.html"
		# 发送广播消息
		messages.SendBroadcastMessage(messages.UserMessage{
			Level:   messages.Warn,
			Message: m,
			Time:    time.Now().UTC(),
			Error:   false,
		})
	}

	# 创建一个新的 ServeMux 实例
	mux := http.NewServeMux()
	// 解析URLs
	// 如果没有提供URLS选项，则将默认URL设置为"/"
	if options["URLS"] == "" {
		s.urls = []string{"/"}
	} else {
		// 如果提供了URLS选项，则将其按逗号分隔并存储在urls数组中
		s.urls = strings.Split(options["URLS"], ",")
	}

	// 为每个URL添加代理处理程序
	// 如果没有提供PSK选项，则返回错误
	if options["PSK"] == "" {
		return &s, fmt.Errorf("a Pre-Shared Key (PSK) password must be provided")
	}
	// 生成用于签名和加密JWT的密钥
	jwtKey := []byte(core.RandStringBytesMaskImprSrc(32))
	// 生成用于加密的不透明密钥
	opaqueKey := gopaque.CryptoDefault.NewKey(nil)
	// 创建HTTP上下文对象，包括PSK、JWT密钥和不透明密钥
	s.ctx = &handlers.HTTPContext{PSK: options["PSK"], JWTKey: jwtKey, OpaqueKey: opaqueKey}

	// 为每个URL添加带有上下文的处理程序
	for _, url := range s.urls {
		// 将代理处理程序绑定到相应的URL
		mux.HandleFunc(url, s.ctx.AgentHTTP)
	}
	/* #nosec G402 */
	// G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
	// G402 (CWE-295): TLS MinVersion too low. (Confidence: HIGH, Severity: HIGH)
	// TLS version is not configured to facilitate dynamic JA3 configurations
	// 创建一个 HTTP 服务器对象
	srv := &http.Server{
		// 设置服务器监听的地址和端口
		Addr:           options["Interface"] + ":" + options["Port"],
		// 设置处理请求的处理器
		Handler:        mux,
		// 设置读取请求的超时时间
		ReadTimeout:    10 * time.Second,
		// 设置写入响应的超时时间
		WriteTimeout:   10 * time.Second,
		// 设置最大请求头大小
		MaxHeaderBytes: 1 << 20,
		// 设置 TLS 配置
		TLSConfig:      &tls.Config{Certificates: []tls.Certificate{*certificates}},
	}

	// 设置 HTTP/3 传输对象
	s.Transport = &http3.Server{
		// 设置 HTTP 服务器对象
		Server: srv,
		// 设置 QUIC 配置
		QuicConfig: &quic.Config{
			// 设置最大空闲超时时间
			MaxIdleTimeout: time.Until(time.Now().AddDate(0, 42, 0)),
			// 禁用保持活动连接
			KeepAlive:      false,
		},
	}

	// 设置服务器的接口
	s.Interface = options["Interface"]
	// 生成新的 UUID 作为服务器的 ID
	s.ID = uuid.NewV4()
	// 设置服务器状态为停止
	s.State = servers.Stopped

	// 返回新生成的服务器对象和错误信息
	return &s, nil
}

// Renew 生成一个新的服务器对象并保留原始的加密密钥
func Renew(ctx handlers.ContextInterface, options map[string]string) (*Server, error) {
	// 创建一个临时服务器对象
	tempServer, err := New(options)
	// 如果创建过程中出现错误，则返回错误信息
	if err != nil {
		return tempServer, err
	}

	// 保留服务器原始的 JWT 密钥，用于签名和加密授权 JWT
	tempServer.ctx.JWTKey = ctx.(handlers.HTTPContext).JWTKey

	// 保留服务器原始的 OPAQUE 密钥，用于 OPAQUE 注册和授权
// 将上下文中的不透明密钥赋值给临时服务器的不透明密钥
tempServer.ctx.OpaqueKey = ctx.(handlers.HTTPContext).OpaqueKey

// 返回用户可以设置的服务器当前配置选项的映射
func (s *Server) GetConfiguredOptions() map[string]string {
    // 创建一个空的字符串映射
    options := make(map[string]string)
    // 设置接口选项
    options["Interface"] = s.Interface
    // 设置端口选项
    options["Port"] = fmt.Sprintf("%d", s.Port)
    // 设置协议选项
    options["Protocol"] = servers.GetProtocol(s.Protocol)
    // 设置预共享密钥选项
    options["PSK"] = s.ctx.PSK
    // 将服务器的 URLS 列表连接成一个字符串，并设置为 URLS 选项
    options["URLS"] = strings.Join(s.urls, " ")
    // 设置 X509 证书选项
    options["X509Cert"] = s.x509Cert
    // 设置 X509 密钥选项
    options["X509Key"] = s.x509Key

    // 返回配置选项映射
    return options
}

// 返回服务器当前上下文信息，如加密密钥
// GetContext 函数返回服务器的上下文接口
func (s *Server) GetContext() handlers.ContextInterface {
	return *s.ctx
}

// GetInterface 函数返回服务器绑定的接口
func (s *Server) GetInterface() string {
	return s.Interface
}

// GetPort 函数返回服务器绑定的端口
func (s *Server) GetPort() int {
	return s.Port
}

// GetProtocol 函数返回服务器的协议作为服务器包中常量的整数
func (s *Server) GetProtocol() int {
	return s.Protocol
}

// GetProtocolString 函数返回服务器的协议
// GetProtocolString function returns the protocol string based on the server's Protocol value
func (s *Server) GetProtocolString() string {
	// Switch statement to check the value of s.Protocol and return the corresponding protocol string
	switch s.Protocol {
	case servers.HTTP3:
		return "HTTP3"
	default:
		return "UNKNOWN"
	}
}

// SetOption function sets an option for an instantiated server object
func (s *Server) SetOption(option string, value string) error {
	var err error
	// Check non-string options first
	switch strings.ToLower(option) {
	case "interface":
		// Set the server's Interface value to the provided value
		s.Interface = value
	case "port":
		// Convert the provided value to an integer and set it as the server's Port value
		s.Port, err = strconv.Atoi(value)
		// Check for any errors during the conversion
		if err != nil {
			// Return an error message if there was an error converting the port number to an integer
			return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
		}
```

		}
	case "protocol":
		// 如果尝试更改协议，则返回错误
		return fmt.Errorf("the protocol can not be changed; create a new listener instead")
	case "psk":
		// 设置预共享密钥
		s.ctx.PSK = value
	case "urls":
		// 将逗号分隔的字符串分割成多个 URL
		s.urls = strings.Split(value, ",")
	case "x509cert":
		// 设置 X.509 证书
		s.x509Cert = option
	case "x509key":
		// 设置 X.509 密钥
		s.x509Key = option
	default:
		// 如果选项无效，则返回错误
		return fmt.Errorf("invalid option: %s", option)
	}
	// 返回空值
	return nil
}

// Start function starts the HTTP3 server
func (s *Server) Start() error {
	// 定义错误组
	var g errgroup.Group
// 捕获 panic，防止程序崩溃
defer func() {
    if r := recover(); r != nil {
        // 生成错误信息
        m := fmt.Sprintf("The %s server on %s:%d paniced:\r\n%v+\r\n", servers.GetProtocol(s.Protocol), s.Interface, s.Port, r.(error))
        // 发送广播消息
        messages.SendBroadcastMessage(messages.UserMessage{
            Level:   messages.Warn,
            Message: m,
            Time:    time.Now().UTC(),
            Error:   true,
        })
    }
}()

// 启动一个 goroutine
g.Go(func() error {
    // 设置服务器状态为运行中
    s.State = servers.Running
    // 如果配置了证书和密钥，则使用 TLS 协议启动服务器
    if s.x509Key != "" && s.x509Cert != "" {
        return s.Transport.(*http3.Server).ListenAndServeTLS(s.x509Cert, s.x509Key)
    }
    // 否则使用普通 HTTP 协议启动服务器
    return s.Transport.(*http3.Server).ListenAndServe()
	})

	// 等待所有任务完成，如果有错误则返回
	if err := g.Wait(); err != nil {
		// 如果错误信息中不包含 "server closed"，则将服务器状态设置为错误，并返回错误信息
		if !strings.Contains(strings.ToLower(err.Error()), "server closed") {
			s.State = servers.Error
			return fmt.Errorf("there was an error starting the %s server on %s:%d %s", servers.GetProtocol(s.Protocol), s.Interface, s.Port, err.Error())
		}
	}
	// 返回空值
	return nil
}

// Status 函数枚举服务器当前是否正在运行或已停止，并将值作为字符串返回
func (s *Server) Status() int {
	return s.State
}

// Stop 函数停止 HTTP3 服务器
func (s *Server) Stop() error {
	// 调用 http3 的 Close() 方法发送一个 QUIC CONNECTION_CLOSE 帧
	err := s.Transport.(*http3.Server).Close()
// 在 quic-go v0.17.3 版本中，CloseGracefully 方法未实现，这意味着不会发送 CONNECTION_CLOSE 或 GOAWAY 帧
// CloseGracefully() 方法不会释放端口，因为它未实现
//err := s.Transport.(*http3.Server).CloseGracefully(time.Second * 0)  // 调用 CloseGracefully 方法关闭 HTTP3 服务器，如果出错则返回错误信息
if err != nil {  // 如果出错，则返回错误信息
    return fmt.Errorf("there was an error stopping the HTTP3 server:\r\n%s", err.Error())
}
s.State = servers.Closed  // 修改服务器状态为 Closed
return nil  // 返回空值
}
```