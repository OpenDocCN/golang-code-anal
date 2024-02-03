# `kubesploit\pkg\servers\http3\http3.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 由自由软件基金会发布，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

package http3

import (
    // 标准库
    "crypto/tls"
    "fmt"
    "net/http"
    "path/filepath"
    "strconv"
    "strings"
    "time"

    // X Packages
    "golang.org/x/sync/errgroup"

    // 第三方库
    "github.com/cretz/gopaque/gopaque"
    "github.com/lucas-clemente/quic-go"
    "github.com/lucas-clemente/quic-go/http3"
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

// Template是用于收集创建实例所需信息的结构，可用于New()函数
type Template struct {
    servers.Template
    X509Key  string // 用于TLS加密的x.509私钥
    X509Cert string // 用于TLS加密的x.509公钥
}
    PSK      string // 客户端和服务器之间在进行密码认证密钥交换（PAKE）之前使用的预共享密钥密码
// init函数用于在服务器包中注册此服务器类型
func init() {
    servers.RegisteredServers["http3"] = ""
}

// GetOptions函数返回一个可配置的服务器选项映射，通常在创建监听器时使用
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
// 所有参数都作为字符串传入，并进行转换/验证
func New(options map[string]string) (*Server, error) {
    var s Server
    var certificates *tls.Certificate
    var err error

    // 验证协议匹配
    if strings.ToLower(options["Protocol"]) != "http3" {
        return &s, fmt.Errorf("server protocol mismatch, expected: HTTP3 got: %s", options["Protocol"])
    }
    s.Protocol = servers.HTTP3

    // 将端口从字符串转换为整数
    s.Port, err = strconv.Atoi(options["Port"])
    if err != nil {
        return &s, fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
    }

    // 验证X509密钥文件是否存在并且可以解析
    certificates, err = util.GetTLSCertificates(options["X509Cert"], options["X509Key"])
    // 如果发生错误
    if err != nil {
        // 生成错误消息
        m := fmt.Sprintf("Certificate was not found at: %s\r\n", options["X509Cert"])
        m += "Creating in-memory x.509 certificate used for this session only"
        // 发送广播消息
        messages.SendBroadcastMessage(messages.UserMessage{
            Level:   messages.Note,
            Message: m,
            Time:    time.Now().UTC(),
            Error:   false,
        })
        // 生成临时的 x.509 证书
        certificates, err = util.GenerateTLSCert(nil, nil, nil, nil, nil, nil, true)
        if err != nil {
            return &s, err
        }
        // 置空以强制使用服务器的 TLSConfig
        s.x509Cert = ""
        s.x509Key = ""
    } else {
        // 设置 x.509 证书和密钥
        s.x509Cert = options["X509Cert"]
        s.x509Key = options["X509Key"]
    }
    // 检查是否存在不安全的指纹
    insecure, errI := util.CheckInsecureFingerprint(*certificates)
    if errI != nil {
        return &s, errI
    }

    // 如果存在不安全的指纹，发送警告消息
    if insecure {
        m := fmt.Sprintf("Insecure publicly distributed Merlin x.509 testing certificate in use for HTTP/3 server on %s:%s\r\n", options["Interface"], options["Port"])
        m += "Additional details: https://merlin-c2.readthedocs.io/en/latest/server/x509.html"
        messages.SendBroadcastMessage(messages.UserMessage{
            Level:   messages.Warn,
            Message: m,
            Time:    time.Now().UTC(),
            Error:   false,
        })
    }

    // 创建一个新的 ServeMux
    mux := http.NewServeMux()

    // 解析 URL
    if options["URLS"] == "" {
        // 如果 URLS 为空，则默认为根路径
        s.urls = []string{"/"}
    } else {
        // 否则按逗号分隔 URLS
        s.urls = strings.Split(options["URLS"], ",")
    }

    // 为每个 URL 添加代理处理程序
    if options["PSK"] == "" {
        // 如果没有提供预共享密钥（PSK），则返回错误
        return &s, fmt.Errorf("a Pre-Shared Key (PSK) password must be provided")
    }
    // 生成用于签名和加密 JWT 的密钥
    jwtKey := []byte(core.RandStringBytesMaskImprSrc(32))
    // 生成默认的不透明密钥
    opaqueKey := gopaque.CryptoDefault.NewKey(nil)
    // 设置 HTTP 上下文
    s.ctx = &handlers.HTTPContext{PSK: options["PSK"], JWTKey: jwtKey, OpaqueKey: opaqueKey}

    // 添加带上下文的处理程序
    // 遍历 s.urls 切片中的每个 URL，并将其与 s.ctx.AgentHTTP 绑定，形成路由规则
    for _, url := range s.urls {
        mux.HandleFunc(url, s.ctx.AgentHTTP)
    }

    /* #nosec G402 */
    // G402: TLS InsecureSkipVerify set true. (Confidence: HIGH, Severity: HIGH) Allowed for testing
    // G402 (CWE-295): TLS MinVersion too low. (Confidence: HIGH, Severity: HIGH)
    // TLS version is not configured to facilitate dynamic JA3 configurations
    // 创建一个 HTTP 服务器对象
    srv := &http.Server{
        Addr:           options["Interface"] + ":" + options["Port"],  // 设置服务器监听的地址和端口
        Handler:        mux,  // 设置服务器的路由处理器
        ReadTimeout:    10 * time.Second,  // 设置读取超时时间
        WriteTimeout:   10 * time.Second,  // 设置写入超时时间
        MaxHeaderBytes: 1 << 20,  // 设置最大请求头字节数
        TLSConfig:      &tls.Config{Certificates: []tls.Certificate{*certificates}},  // 设置 TLS 配置
    }

    // 创建一个 HTTP/3 服务器对象
    s.Transport = &http3.Server{
        Server: srv,  // 设置 HTTP/3 服务器对象的底层 HTTP 服务器
        QuicConfig: &quic.Config{
            // 设置最大空闲超时时间，防止客户端发送 HTTP/2 PING 帧
            MaxIdleTimeout: time.Until(time.Now().AddDate(0, 42, 0)),
            KeepAlive:      false,  // 禁用保持活动连接
        },
    }

    // 设置服务器的网络接口
    s.Interface = options["Interface"]
    // 生成一个新的 UUID 作为服务器的 ID
    s.ID = uuid.NewV4()
    // 设置服务器状态为已停止
    s.State = servers.Stopped

    // 返回服务器对象和空错误
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
    options["Protocol"] = servers.GetProtocol(s.Protocol)
    options["PSK"] = s.ctx.PSK
    options["URLS"] = strings.Join(s.urls, " ")
    options["X509Cert"] = s.x509Cert
    options["X509Key"] = s.x509Key

    return options
}

// GetContext 返回 Server 的当前上下文信息，如加密密钥
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

// GetProtocol 返回服务器的协议作为 servers 包中的常量的整数
func (s *Server) GetProtocol() int {
    return s.Protocol
}

// GetProtocolString 函数返回服务器的协议
func (s *Server) GetProtocolString() string {
    switch s.Protocol {
    case servers.HTTP3:
        return "HTTP3"
    default:
        return "UNKNOWN"
    }
}

// SetOption 函数为实例化的服务器对象设置选项
func (s *Server) SetOption(option string, value string) error {
    var err error
    // 检查非字符串选项
    switch strings.ToLower(option) {
    case "interface":
        s.Interface = value
    case "port":
        s.Port, err = strconv.Atoi(value)
        if err != nil {
            return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
        }
    case "protocol":
        return fmt.Errorf("the protocol can not be changed; create a new listener instead")
    case "psk":
        s.ctx.PSK = value
    case "urls":
        s.urls = strings.Split(value, ",")
    case "x509cert":
        s.x509Cert = option
    case "x509key":
        s.x509Key = option
    default:
        return fmt.Errorf("invalid option: %s", option)
    }
    return nil
}

// Start function starts the HTTP3 server
func (s *Server) Start() error {
    var g errgroup.Group

    // 捕获 Panic
    defer func() {
        if r := recover(); r != nil {
            m := fmt.Sprintf("The %s server on %s:%d paniced:\r\n%v+\r\n", servers.GetProtocol(s.Protocol), s.Interface, s.Port, r.(error))
            messages.SendBroadcastMessage(messages.UserMessage{
                Level:   messages.Warn,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   true,
            })
        }
    }()

    g.Go(func() error {
        s.State = servers.Running
        if s.x509Key != "" && s.x509Cert != "" {
            return s.Transport.(*http3.Server).ListenAndServeTLS(s.x509Cert, s.x509Key)
        }
        return s.Transport.(*http3.Server).ListenAndServe()
    })

    if err := g.Wait(); err != nil {
        if !strings.Contains(strings.ToLower(err.Error()), "server closed") {
            s.State = servers.Error
            return fmt.Errorf("there was an error starting the %s server on %s:%d %s", servers.GetProtocol(s.Protocol), s.Interface, s.Port, err.Error())
        }
    }
    return nil
}
// Status函数枚举服务器当前是否正在运行或已停止，并将值作为字符串返回
func (s *Server) Status() int {
    return s.State
}

// Stop函数停止HTTP3服务器
func (s *Server) Stop() error {
    // http3 Close()发送QUIC CONNECTION_CLOSE帧
    err := s.Transport.(*http3.Server).Close()
    // 截至quic-go v0.17.3，CloseGracefully未实现，这意味着不会发送CONNECTION_CLOSE或GOAWAY帧
    // CloseGracefully()不会释放端口，因为它未实现
    //err := s.Transport.(*http3.Server).CloseGracefully(time.Second * 0)
    if err != nil {
        return fmt.Errorf("停止HTTP3服务器时出错：\r\n%s", err.Error())
    }
    s.State = servers.Closed

    return nil
}
```