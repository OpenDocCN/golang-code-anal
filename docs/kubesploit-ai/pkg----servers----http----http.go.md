# `kubesploit\pkg\servers\http\http.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd.

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括对适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 包http
// 导入 HTTP2 包，因为 net/http 本身支持该协议
import (
	"crypto/tls" // 导入加密/解密包
	"fmt" // 导入格式化包
	"net/http" // 导入网络通信包
	"path/filepath" // 导入文件路径包
	"strconv" // 导入字符串转换包
	"strings" // 导入字符串处理包
	"time" // 导入时间包

	// X Packages
	"golang.org/x/sync/errgroup" // 导入并发控制包

	// 3rd Party
	"github.com/cretz/gopaque/gopaque" // 导入第三方包

	uuid "github.com/satori/go.uuid" // 导入 UUID 生成包

	// Merlin
	"kubesploit/pkg/api/messages" // 导入 Merlin 包中的消息定义
)
// 导入所需的包
"kubesploit/pkg/core"
"kubesploit/pkg/handlers"
"kubesploit/pkg/servers"
"kubesploit/pkg/util"
)

// Server 是用于 HTTP3 服务器的结构体
type Server struct {
	servers.Server // 继承自 servers 包中的 Server 结构体
	x509Cert string // x.509 证书
	x509Key  string // x.509 私钥
	urls     []string // URL 列表
	ctx      *handlers.HTTPContext // HTTP 上下文
}

// Template 是用于收集创建实例所需信息的结构体，用于 New() 函数
type Template struct {
	servers.Template // 继承自 servers 包中的 Template 结构体
	X509Key  string // 用于 TLS 加密的 x.509 私钥
	X509Cert string // 用于 TLS 加密的 x.509 公钥
// URLS is a string variable that holds a comma separated list of URLs for handling incoming web traffic
URLS     string 

// PSK is a string variable that holds the pre-shared key password used prior to Password Authenticated Key Exchange (PAKE)
PSK      string 

// init function registers this server type with the servers package
func init() {
	// Register Server types for http, https, and http2
	servers.RegisteredServers["http"] = ""
	servers.RegisteredServers["https"] = ""
	servers.RegisteredServers["http2"] = ""
}

// GetOptions function returns a map of configurable server options typically used when creating a listener
func GetOptions(protocol string) map[string]string {
	// Create a map to hold server options
	options := make(map[string]string)
	options["Interface"] = "127.0.0.1"
	// Set the default port based on the protocol
	if protocol == "http" {
		options["Port"] = "80"
	} else {
		options["Port"] = "443"
	}
	}

	// 设置选项中的 PSK 为 "kubesploit"
	options["PSK"] = "kubesploit"
	// 设置选项中的 URLS 为 "/"
	options["URLS"] = "/"

	// 如果协议不是 HTTP，则设置选项中的 X509Cert 和 X509Key
	if strings.ToLower(protocol) != "http" {
		options["X509Cert"] = filepath.Join(string(core.CurrentDir), "data", "x509", "server.crt")
		options["X509Key"] = filepath.Join(string(core.CurrentDir), "data", "x509", "server.key")
	}
	// 返回设置好的选项
	return options
}

// New 创建一个新的 HTTP 服务器对象并返回指针
// 所有参数都作为字符串传入，并进行转换/验证
func New(options map[string]string) (*Server, error) {
	var s Server
	var certificates *tls.Certificate
	var err error
	// 将协议转换为小写
	proto := strings.ToLower(options["Protocol"])
// 验证协议是否匹配，如果不是http、https或http2，则返回错误
if proto != "http" && proto != "https" && proto != "http2" {
    return &s, fmt.Errorf("server protocol mismatch, expected: http, https, or http2 got: %s", proto)
}
// 根据协议类型设置服务器的协议属性
switch proto {
case "http":
    s.Protocol = servers.HTTP
case "https":
    s.Protocol = servers.HTTPS
case "http2":
    s.Protocol = servers.HTTP2
}

// 将端口从字符串转换为整数
s.Port, err = strconv.Atoi(options["Port"])
if err != nil {
    return &s, fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
}
// 验证 X509 密钥文件是否存在并且可以被解析
if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
    // 获取 TLS 证书和密钥
    certificates, err = util.GetTLSCertificates(options["X509Cert"], options["X509Key"])
    if err != nil {
        // 如果证书不存在，发送广播消息并创建内存中的 x.509 证书用于当前会话
        m := fmt.Sprintf("Certificate was not found at: %s\r\n", options["X509Cert"])
        m += "Creating in-memory x.509 certificate used for this session only"
        messages.SendBroadcastMessage(messages.UserMessage{
            Level:   messages.Note,
            Message: m,
            Time:    time.Now().UTC(),
            Error:   false,
        })
        // 生成内存中的证书
        certificates, err = util.GenerateTLSCert(nil, nil, nil, nil, nil, nil, true)
        if err != nil {
            return &s, err
        }
        // 留空以强制使用服务器的 TLS 配置
        s.x509Cert = ""
        s.x509Key = ""
    }
}
		} else {
			// 如果不是自签名证书，从选项中获取 X509 证书和密钥
			s.x509Cert = options["X509Cert"]
			s.x509Key = options["X509Key"]
		}
		// 检查是否存在不安全的指纹
		insecure, errI := util.CheckInsecureFingerprint(*certificates)
		// 如果存在错误，返回错误信息
		if errI != nil {
			return &s, errI
		}
		// 如果存在不安全的指纹
		if insecure {
			// 创建警告消息
			m := fmt.Sprintf("Insecure publicly distributed Merlin x.509 testing certificate in use for %s server on %s:%s\r\n", proto, options["Interface"], options["Port"])
			m += "Additional details: https://merlin-c2.readthedocs.io/en/latest/server/x509.html"
			// 发送广播消息
			messages.SendBroadcastMessage(messages.UserMessage{
				Level:   messages.Warn,
				Message: m,
				Time:    time.Now().UTC(),
				Error:   false,
			})
		}
	}
	// 创建一个新的 ServeMux 实例，用于处理 HTTP 请求的多路复用器
	mux := http.NewServeMux()

	// 解析 URL
	if options["URLS"] == "" {
		// 如果未提供 URL，则默认为根路径
		s.urls = []string{"/"}
	} else {
		// 如果提供了 URL，则按逗号分隔并存储在 s.urls 中
		s.urls = strings.Split(options["URLS"], ",")
	}

	// 为每个 URL 添加代理处理程序
	if options["PSK"] == "" {
		// 如果未提供预共享密钥 (PSK)，则返回错误
		return &s, fmt.Errorf("a Pre-Shared Key (PSK) password must be provided")
	}
	// 生成用于签名和加密 JWT 的密钥
	jwtKey := []byte(core.RandStringBytesMaskImprSrc(32))
	// 创建一个默认的不透明密钥
	opaqueKey := gopaque.CryptoDefault.NewKey(nil)
	// 为处理程序设置上下文
	s.ctx = &handlers.HTTPContext{PSK: options["PSK"], JWTKey: jwtKey, OpaqueKey: opaqueKey}

	// 使用上下文添加处理程序
	for _, url := range s.urls {
		// 为每个 URL 添加处理程序
		mux.HandleFunc(url, s.ctx.AgentHTTP)
	}
	}

	// 设置服务器的传输参数
	s.Transport = &http.Server{
		Addr:           options["Interface"] + ":" + options["Port"], // 设置服务器监听的地址和端口
		Handler:        mux, // 设置服务器的处理程序
		ReadTimeout:    10 * time.Second, // 设置读取超时时间
		WriteTimeout:   10 * time.Second, // 设置写入超时时间
		MaxHeaderBytes: 1 << 20, // 设置最大头部字节数
	}

	// 如果使用 TLS，则添加 X.509 证书
	if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
		s.Transport.(*http.Server).TLSConfig = &tls.Config{Certificates: []tls.Certificate{*certificates}} // 设置 TLS 配置
		// #nosec G402 TLS version is not configured to facilitate dynamic JA3 configurations
	}

	s.Interface = options["Interface"] // 设置服务器的接口
	s.ID = uuid.NewV4() // 生成服务器的唯一标识
	s.State = servers.Stopped // 设置服务器状态为停止

	return &s, nil // 返回服务器对象和空错误
// Renew 生成一个新的 Server 对象并保留原始的加密密钥
func Renew(ctx handlers.ContextInterface, options map[string]string) (*Server, error) {
    // 使用传入的选项生成一个新的 Server 对象
    tempServer, err := New(options)
    if err != nil {
        return tempServer, err
    }

    // 保留服务器原始的 JWT 密钥，用于签名和加密授权 JWT
    tempServer.ctx.JWTKey = ctx.(handlers.HTTPContext).JWTKey

    // 保留服务器原始的 OPAQUE 密钥，用于 OPAQUE 注册/授权
    tempServer.ctx.OpaqueKey = ctx.(handlers.HTTPContext).OpaqueKey

    return tempServer, nil
}

// GetConfiguredOptions 返回用户可以设置的服务器当前配置选项
func (s *Server) GetConfiguredOptions() map[string]string {
    // 实现代码省略
}
// 创建一个空的字符串到字符串的映射
options := make(map[string]string)
// 将接口信息添加到选项映射中
options["Interface"] = s.Interface
// 将端口信息添加到选项映射中
options["Port"] = fmt.Sprintf("%d", s.Port)
// 将协议信息添加到选项映射中
options["Protocol"] = s.GetProtocolString()
// 将预共享密钥信息添加到选项映射中
options["PSK"] = s.ctx.PSK
// 将URL信息添加到选项映射中，并用逗号分隔多个URL
options["URLS"] = strings.Join(s.urls, ",")

// 如果协议不是HTTP，则将X509证书和密钥信息添加到选项映射中
if s.Protocol != servers.HTTP {
    options["X509Cert"] = s.x509Cert
    options["X509Key"] = s.x509Key
}
// 返回选项映射
return options
}

// GetContext函数返回服务器当前的上下文信息，如加密密钥
func (s *Server) GetContext() handlers.ContextInterface {
    return *s.ctx
}

// GetInterface函数返回服务器绑定的接口信息
// GetInterface函数返回服务器的接口
func (s *Server) GetInterface() string {
	return s.Interface
}

// GetPort函数返回服务器绑定的端口
func (s *Server) GetPort() int {
	return s.Port
}

// GetProtocol函数返回服务器的协议，以服务器包中的常量整数形式返回
func (s *Server) GetProtocol() int {
	return s.Protocol
}

// GetProtocolString函数返回服务器的协议
func (s *Server) GetProtocolString() string {
	switch s.Protocol {
	case servers.HTTP:
		return "HTTP"
	case servers.HTTPS:
		// 在这里应该有返回HTTPS的逻辑
	}
}
// 根据服务器类型返回相应的协议类型
func (s *Server) GetProtocol() string {
	// 根据服务器类型返回相应的协议类型
	switch s.Type {
	case servers.HTTP:
		return "HTTP"
	case servers.HTTPS:
		return "HTTPS"
	case servers.HTTP2:
		return "HTTP2"
	default:
		return "UNKNOWN"
	}
}

// SetOption函数为实例化的服务器对象设置选项
func (s *Server) SetOption(option string, value string) error {
	var err error
	// 首先检查非字符串选项
	switch strings.ToLower(option) {
	case "interface":
		// 设置服务器的接口
		s.Interface = value
	case "port":
		// 将端口号转换为整数并设置到服务器对象中
		s.Port, err = strconv.Atoi(value)
		if err != nil {
			// 如果转换出错，则返回错误信息
			return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
		}
# 根据不同的选项进行相应的操作
case "protocol":
    # 如果尝试更改协议，则返回错误
    return fmt.Errorf("the protocol can not be changed; create a new listener instead")
case "psk":
    # 设置预共享密钥
    s.ctx.PSK = value
case "urls":
    # 将逗号分隔的字符串分割成多个 URL
    s.urls = strings.Split(value, ",")
case "x509cert":
    # 如果协议是 HTTPS 或 HTTP2，则设置 x509 证书
    if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2:
        s.x509Cert = option
case "x509key":
    # 如果协议是 HTTPS 或 HTTP2，则设置 x509 密钥
    if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2:
        s.x509Key = option
default:
    # 如果选项无效，则返回错误
    return fmt.Errorf("invalid option: %s", option)
}
# 返回操作成功
return nil
// Start function starts the HTTP server
// Start函数启动HTTP服务器

func (s *Server) Start() error {
	// 创建一个错误组
	var g errgroup.Group

	// 捕获Panic
	// 如果发生panic，捕获并发送警告消息
	defer func() {
		if r := recover(); r != nil {
			m := fmt.Sprintf("The %s server on %s:%d paniced:\r\n%v+\r\n", servers.GetProtocol(s.GetProtocol()), s.Interface, s.Port, r.(error))
			messages.SendBroadcastMessage(messages.UserMessage{
				Level:   messages.Warn,
				Message: m,
				Time:    time.Now().UTC(),
				Error:   true,
			})
		}
	}()

	// 启动服务器状态为Running
	g.Go(func() error {
		s.State = servers.Running
		switch s.Protocol {
// 根据服务器的协议类型启动相应的服务
switch s.Protocol {
	case servers.HTTP:
		// 如果是HTTP协议，调用http.Server的ListenAndServe方法启动服务
		return s.Transport.(*http.Server).ListenAndServe()
	case servers.HTTPS, servers.HTTP2:
		// 如果是HTTPS或HTTP2协议，调用http.Server的ListenAndServeTLS方法启动服务，并传入证书和密钥
		return s.Transport.(*http.Server).ListenAndServeTLS(s.x509Cert, s.x509Key)
	default:
		// 如果协议类型不合法，返回错误信息
		return fmt.Errorf("could not start HTTP server, invalid protocol %d, %s", s.Protocol, servers.GetStateString(s.Protocol))
	}
})

// 等待所有的goroutine完成
if err := g.Wait(); err != nil {
	// 如果等待过程中出现错误
	if err != http.ErrServerClosed {
		// 如果错误不是由服务器关闭引起的，将服务器状态设置为错误，并返回错误信息
		s.State = servers.Error
		return fmt.Errorf("there was an error with the %s server on %s:%d %s", s.GetProtocolString(), s.Interface, s.Port, err.Error())
	}
}
// 返回空值
return nil
}

// Status函数用于返回服务器当前的状态
func (s *Server) Status() int {
// 返回服务器的状态
return s.State
}

// Stop 函数停止服务器
func (s *Server) Stop() error {
	// 不使用 Shutdown，因为它不会立即释放端口，并且会允许流量继续
	err := s.Transport.(*http.Server).Close()
	if err != nil {
		return fmt.Errorf("there was an error stopping the HTTP server:\r\n%s", err.Error())
	}
	// 将服务器状态设置为已关闭
	s.State = servers.Closed
	return nil
}
```