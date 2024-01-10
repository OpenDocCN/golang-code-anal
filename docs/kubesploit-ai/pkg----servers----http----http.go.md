# `kubesploit\pkg\servers\http\http.go`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 其版本由自由软件基金会发布，或者
// 任何以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括隐含的适销性或特定用途的保证。参见
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package http

// HTTP2在此包中，因为net/http本身支持该协议

import (
    "crypto/tls"
    "fmt"
    "net/http"
    "path/filepath"
    "strconv"
    "strings"
    "time"

    // X Packages
    "golang.org/x/sync/errgroup"

    // 3rd Party
    "github.com/cretz/gopaque/gopaque"
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/api/messages"
    "kubesploit/pkg/core"
    "kubesploit/pkg/handlers"
    "kubesploit/pkg/servers"
    "kubesploit/pkg/util"
)

// Server是HTTP3服务器的结构
type Server struct {
    servers.Server
    x509Cert string
    x509Key  string
    urls     []string
    ctx      *handlers.HTTPContext
}

// Template是一个用于收集创建实例所需信息的结构，用于New()函数
type Template struct {
    servers.Template
    X509Key  string // 用于TLS加密的x.509私钥
    X509Cert string // 用于TLS加密的x.509公钥
}
    URLS     string // 用逗号分隔的 URL 列表，用于处理传入的网络流量
    PSK      string // 在密码认证密钥交换（PAKE）之前使用的预共享密钥密码
// init函数用于在服务器包中注册此服务器类型
func init() {
    // 注册服务器类型
    servers.RegisteredServers["http"] = ""
    servers.RegisteredServers["https"] = ""
    servers.RegisteredServers["http2"] = ""
}

// GetOptions函数返回一个可配置的服务器选项映射，通常在创建监听器时使用
func GetOptions(protocol string) map[string]string {
    options := make(map[string]string)
    options["Interface"] = "127.0.0.1"
    if protocol == "http" {
        options["Port"] = "80"
    } else {
        options["Port"] = "443"
    }

    //options["Protocol"] = protocol
    options["PSK"] = "kubesploit"
    options["URLS"] = "/"

    if strings.ToLower(protocol) != "http" {
        options["X509Cert"] = filepath.Join(string(core.CurrentDir), "data", "x509", "server.crt")
        options["X509Key"] = filepath.Join(string(core.CurrentDir), "data", "x509", "server.key")
    }
    return options
}

// New函数创建一个新的HTTP服务器对象并返回一个指针
// 所有参数都作为字符串传入，并进行转换/验证
func New(options map[string]string) (*Server, error) {
    var s Server
    var certificates *tls.Certificate
    var err error
    proto := strings.ToLower(options["Protocol"])

    // 验证协议匹配
    if proto != "http" && proto != "https" && proto != "http2" {
        return &s, fmt.Errorf("server protocol mismatch, expected: http, https, or http2 got: %s", proto)
    }
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

    // 验证X509密钥文件是否存在并可以解析
    // 如果协议是 HTTPS 或者 HTTP2
    if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
        // 获取 TLS 证书
        certificates, err = util.GetTLSCertificates(options["X509Cert"], options["X509Key"])
        // 如果获取证书出错
        if err != nil {
            // 生成内存中的 x.509 证书用于当前会话
            m := fmt.Sprintf("Certificate was not found at: %s\r\n", options["X509Cert"])
            m += "Creating in-memory x.509 certificate used for this session only"
            // 发送广播消息
            messages.SendBroadcastMessage(messages.UserMessage{
                Level:   messages.Note,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   false,
            })
            // 生成内存中的证书
            certificates, err = util.GenerateTLSCert(nil, nil, nil, nil, nil, nil, true)
            // 如果生成证书出错
            if err != nil {
                return &s, err
            }
            // 置空以强制使用服务器的 TLSConfig
            s.x509Cert = ""
            s.x509Key = ""
        } else {
            // 设置证书和密钥
            s.x509Cert = options["X509Cert"]
            s.x509Key = options["X509Key"]
        }
        // 检查是否存在不安全的指纹
        insecure, errI := util.CheckInsecureFingerprint(*certificates)
        // 如果检查不安全指纹出错
        if errI != nil {
            return &s, errI
        }
        // 如果存在不安全的指纹
        if insecure {
            m := fmt.Sprintf("Insecure publicly distributed Merlin x.509 testing certificate in use for %s server on %s:%s\r\n", proto, options["Interface"], options["Port"])
            m += "Additional details: https://merlin-c2.readthedocs.io/en/latest/server/x509.html"
            // 发送警告消息
            messages.SendBroadcastMessage(messages.UserMessage{
                Level:   messages.Warn,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   false,
            })
        }
    }

    // 创建一个新的 ServeMux
    mux := http.NewServeMux()

    // 解析 URL
    if options["URLS"] == "" {
        // 如果 URLS 为空，则默认为 "/"
        s.urls = []string{"/"}
    } else {
        // 否则按逗号分隔 URLS
        s.urls = strings.Split(options["URLS"], ",")
    }

    // 为每个 URL 添加代理处理程序
    // 如果选项中的PSK为空，则返回错误
    if options["PSK"] == "" {
        return &s, fmt.Errorf("a Pre-Shared Key (PSK) password must be provided")
    }
    // 生成用于签名和加密JWT的密钥
    jwtKey := []byte(core.RandStringBytesMaskImprSrc(32)) // Used to sign and encrypt JWT
    // 生成用于加密的不透明密钥
    opaqueKey := gopaque.CryptoDefault.NewKey(nil)
    // 设置HTTP上下文，包括PSK、JWT密钥和不透明密钥
    s.ctx = &handlers.HTTPContext{PSK: options["PSK"], JWTKey: jwtKey, OpaqueKey: opaqueKey}

    // 为每个URL添加处理程序和上下文
    for _, url := range s.urls {
        mux.HandleFunc(url, s.ctx.AgentHTTP)
    }

    // 设置HTTP服务器的参数
    s.Transport = &http.Server{
        Addr:           options["Interface"] + ":" + options["Port"],
        Handler:        mux,
        ReadTimeout:    10 * time.Second,
        WriteTimeout:   10 * time.Second,
        MaxHeaderBytes: 1 << 20,
    }

    // 如果使用TLS，则添加X.509证书
    if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
        // 设置TLS配置的证书
        s.Transport.(*http.Server).TLSConfig = &tls.Config{Certificates: []tls.Certificate{*certificates}} // #nosec G402 TLS version is not configured to facilitate dynamic JA3 configurations
    }

    // 设置服务器的接口和ID
    s.Interface = options["Interface"]
    s.ID = uuid.NewV4()
    // 设置服务器状态为停止
    s.State = servers.Stopped

    // 返回服务器和nil作为错误
    return &s, nil
// Renew 生成一个新的 Server 对象并保留原始的加密密钥
func Renew(ctx handlers.ContextInterface, options map[string]string) (*Server, error) {
    // 使用给定的选项生成一个新的 Server 对象
    tempServer, err := New(options)
    // 如果生成过程中出现错误，则返回错误
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
    options := make(map[string]string)
    options["Interface"] = s.Interface
    options["Port"] = fmt.Sprintf("%d", s.Port)
    options["Protocol"] = s.GetProtocolString()
    options["PSK"] = s.ctx.PSK
    options["URLS"] = strings.Join(s.urls, ",")

    if s.Protocol != servers.HTTP {
        options["X509Cert"] = s.x509Cert
        options["X509Key"] = s.x509Key
    }
    return options
}

// GetContext 返回 Server 当前的上下文信息，如加密密钥
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

// GetProtocol 返回服务器的协议作为 servers 包中常量的整数
func (s *Server) GetProtocol() int {
    return s.Protocol
}

// GetProtocolString 函数返回服务器的协议
func (s *Server) GetProtocolString() string {
    switch s.Protocol {
    case servers.HTTP:
        return "HTTP"
    case servers.HTTPS:
        return "HTTPS"
    }
    # 如果服务器类型是 HTTP2，则返回字符串 "HTTP2"
    case servers.HTTP2:
        return "HTTP2"
    # 如果服务器类型不是 HTTP2，则返回字符串 "UNKNOWN"
    default:
        return "UNKNOWN"
    }
// SetOption function sets an option for an instantiated server object
func (s *Server) SetOption(option string, value string) error {
    var err error
    // Check non-string options first
    switch strings.ToLower(option) {
    case "interface":
        // 设置服务器对象的接口
        s.Interface = value
    case "port":
        // 将字符串转换为整数类型的端口号
        s.Port, err = strconv.Atoi(value)
        if err != nil {
            return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
        }
    case "protocol":
        // 协议不能被更改，需要创建一个新的监听器
        return fmt.Errorf("the protocol can not be changed; create a new listener instead")
    case "psk":
        // 设置服务器对象的预共享密钥
        s.ctx.PSK = value
    case "urls":
        // 使用逗号分隔的字符串设置服务器对象的 URL 列表
        s.urls = strings.Split(value, ",")
    case "x509cert":
        // 如果协议是 HTTPS 或者 HTTP2，则设置服务器对象的 x509 证书
        if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
            s.x509Cert = option
        }
    case "x509key":
        // 如果协议是 HTTPS 或者 HTTP2，则设置服务器对象的 x509 密钥
        if s.Protocol == servers.HTTPS || s.Protocol == servers.HTTP2 {
            s.x509Key = option
        }
    default:
        // 返回无效选项的错误
        return fmt.Errorf("invalid option: %s", option)
    }
    return nil
}

// Start function starts the HTTP server
func (s *Server) Start() error {
    var g errgroup.Group

    // Catch Panic
    // 捕获 panic，发送警告消息
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
    # 使用 Go 协程启动服务器
    g.Go(func() error {
        # 设置服务器状态为运行中
        s.State = servers.Running
        # 根据协议类型选择启动 HTTP 服务器或者 HTTPS/HTTP2 服务器
        switch s.Protocol:
            # 如果是 HTTP 协议，使用 http.Server 的 ListenAndServe 方法启动服务器
            case servers.HTTP:
                return s.Transport.(*http.Server).ListenAndServe()
            # 如果是 HTTPS 或者 HTTP2 协议，使用 http.Server 的 ListenAndServeTLS 方法启动服务器
            case servers.HTTPS, servers.HTTP2:
                return s.Transport.(*http.Server).ListenAndServeTLS(s.x509Cert, s.x509Key)
            # 如果协议类型不匹配，返回错误信息
            default:
                return fmt.Errorf("could not start HTTP server, invalid protocol %d, %s", s.Protocol, servers.GetStateString(s.Protocol))
        }
    })

    # 等待协程执行完毕，获取可能的错误信息
    if err := g.Wait(); err != nil:
        # 如果错误不是服务器关闭导致的，设置服务器状态为错误，并返回错误信息
        if err != http.ErrServerClosed:
            s.State = servers.Error
            return fmt.Errorf("there was an error with the %s server on %s:%d %s", s.GetProtocolString(), s.Interface, s.Port, err.Error())
    # 如果没有错误，返回空
    return nil
// Status 函数枚举服务器当前的运行状态，返回字符串形式的值
func (s *Server) Status() int {
    return s.State
}

// Stop 函数停止服务器
func (s *Server) Stop() error {
    // 不使用 Shutdown，因为它不会立即释放端口，并且会允许流量继续
    err := s.Transport.(*http.Server).Close()
    if err != nil {
        return fmt.Errorf("there was an error stopping the HTTP server:\r\n%s", err.Error())
    }
    s.State = servers.Closed
    return nil
}
```