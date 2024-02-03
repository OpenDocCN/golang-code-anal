# `kubesploit\pkg\servers\http2\http2.go`

```go
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途适用性的暗示保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

package http2

// 该包用于net/http包中不可用的特定HTTP/2功能

import (
    "fmt"
    "net/http"
    "strconv"
    "strings"
    "time"

    // X Packages
    "golang.org/x/net/http2"
    "golang.org/x/net/http2/h2c"
    "golang.org/x/sync/errgroup"

    // 3rd Party
    "github.com/cretz/gopaque/gopaque"
    uuid "github.com/satori/go.uuid"

    // Merlin
    "kubesploit/pkg/api/messages"
    "kubesploit/pkg/core"
    "kubesploit/pkg/handlers"
    "kubesploit/pkg/servers"
)

// Server是HTTP/2明文（h2c）服务器的结构
type Server struct {
    servers.Server
    urls []string
    ctx  *handlers.HTTPContext
}

// Template是用于收集创建New()函数实例所需信息的结构
type Template struct {
    servers.Template
    X509Key  string // 用于TLS加密的x.509私钥
    X509Cert string // 用于TLS加密的x.509公钥
    PSK      string // 定义一个字符串类型的变量 PSK，用于存储在密码认证密钥交换（PAKE）之前使用的预共享密钥密码
// 注册该服务器类型到 servers 包中
func init() {
    servers.RegisteredServers["h2c"] = ""
}

// GetOptions 返回一个可配置的服务器选项映射，通常在创建监听器时使用
func GetOptions() map[string]string {
    options := make(map[string]string)
    options["Interface"] = "127.0.0.1"
    options["Port"] = "80"
    options["PSK"] = "kubesploit"
    options["URLS"] = "/"
    return options
}

// New 创建一个新的 HTTP2 服务器对象并返回一个指针
// 所有参数都作为字符串传入，并进行转换/验证
func New(options map[string]string) (*Server, error) {
    var s Server
    var err error

    // 验证协议匹配
    if strings.ToLower(options["Protocol"]) != "h2c" {
        return &s, fmt.Errorf("server protocol mismatch, expected: H2C got: %s", options["Protocol"])
    }
    s.Protocol = servers.H2C

    // 将端口从字符串转换为整数
    s.Port, err = strconv.Atoi(options["Port"])
    if err != nil {
        return &s, fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
    }

    mux := http.NewServeMux()

    // 解析 URL
    if options["URLS"] == "" {
        s.urls = []string{"/"}
    } else {
        s.urls = strings.Split(options["URLS"], ",")
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
}
    # 设置服务器的传输方式为 HTTP
    s.Transport = &http.Server{
        # 设置服务器监听的地址和端口
        Addr:           options["Interface"] + ":" + options["Port"],
        # 设置处理器为 H2C 协议的处理器
        Handler:        h2c.NewHandler(mux, h2s),
        # 设置读取超时时间为 10 秒
        ReadTimeout:    10 * time.Second,
        # 设置写入超时时间为 10 秒
        WriteTimeout:   10 * time.Second,
        # 设置最大头部字节数为 1MB
        MaxHeaderBytes: 1 << 20,
    }

    # 设置服务器的接口
    s.Interface = options["Interface"]
    # 为服务器生成一个新的唯一标识
    s.ID = uuid.NewV4()
    # 设置服务器状态为停止
    s.State = servers.Stopped

    # 返回服务器对象和空错误
    return &s, nil
// Renew函数生成一个新的Server对象，并保留原始的加密密钥
func Renew(ctx handlers.ContextInterface, options map[string]string) (*Server, error) {
    // 使用给定的选项生成一个新的Server对象
    tempServer, err := New(options)
    // 如果生成过程中出现错误，则返回错误
    if err != nil {
        return tempServer, err
    }

    // 保留服务器原始的JWT密钥，用于签名和加密授权JWT
    tempServer.ctx.JWTKey = ctx.(handlers.HTTPContext).JWTKey

    // 保留服务器原始的OPAQUE密钥，用于OPAQUE注册/授权
    tempServer.ctx.OpaqueKey = ctx.(handlers.HTTPContext).OpaqueKey

    return tempServer, nil
}

// GetConfiguredOptions函数返回用户可以设置的服务器当前配置选项
func (s *Server) GetConfiguredOptions() map[string]string {
    options := make(map[string]string)
    options["Interface"] = s.Interface
    options["Port"] = fmt.Sprintf("%d", s.Port)
    options["Protocol"] = servers.GetProtocol(s.Protocol)
    options["PSK"] = s.ctx.PSK
    options["URLS"] = strings.Join(s.urls, " ")

    return options
}

// GetContext函数返回服务器当前的上下文信息，如加密密钥
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
func (s *Server) GetProtocolString() string {
    switch s.Protocol {
    case servers.H2C:
        return "H2C"
    default:
        return "UNKNOWN"
    }
}

// SetOption为实例化的服务器对象设置选项
func (s *Server) SetOption(option string, value string) error {
    var err error
    // 检查非字符串选项
    switch strings.ToLower(option) {
    case "interface":
        // 设置接口
        s.Interface = value
    case "port":
        // 设置端口，将字符串转换为整数
        s.Port, err = strconv.Atoi(value)
        if err != nil {
            // 如果转换出错，返回错误信息
            return fmt.Errorf("there was an error converting the port number to an integer: %s", err.Error())
        }
    case "protocol":
        // 协议不能被更改，需要创建一个新的监听器
        return fmt.Errorf("the protocol can not be changed; create a new listener instead")
    case "psk":
        // 设置预共享密钥
        s.ctx.PSK = value
    case "urls":
        // 设置 URL 列表，通过逗号分隔的字符串
        s.urls = strings.Split(value, ",")
    default:
        // 无效选项，返回错误信息
        return fmt.Errorf("invalid option: %s", option)
    }
    // 没有错误，返回 nil
    return nil
// Start the HTTP2 server
func (s *Server) Start() error {
    var g errgroup.Group

    // 捕获 Panic
    defer func() {
        if r := recover(); r != nil {
            // 生成错误信息
            m := fmt.Sprintf("The %s server on %s:%d paniced:\r\n%v+\r\n", servers.GetProtocol(s.GetProtocol()), s.Interface, s.Port, r.(error))
            // 发送广播消息
            messages.SendBroadcastMessage(messages.UserMessage{
                Level:   messages.Warn,
                Message: m,
                Time:    time.Now().UTC(),
                Error:   true,
            })
        }
    }()

    // 启动 HTTP2 服务器
    g.Go(func() error {
        s.State = servers.Running
        return s.Transport.(*http.Server).ListenAndServe()
    })

    // 等待所有 goroutine 完成
    if err := g.Wait(); err != nil {
        if err != http.ErrServerClosed {
            // 设置服务器状态为错误
            s.State = servers.Error
            return fmt.Errorf("there was an error with the %s server on %s:%d %s", s.GetProtocolString(), s.Interface, s.Port, err.Error())
        }
    }
    return nil
}

// Status enumerates if the server is currently running or stopped and returns the value as a string
// 返回服务器当前的状态，以整数形式
func (s *Server) Status() int {
    return s.State
}

// Stop the HTTP2 server
// 停止 HTTP2 服务器
func (s *Server) Stop() error {
    // 优雅地关闭服务器，不中断任何活动连接
    // 这将保持代理的检查，因为它是一个活动连接
    //err = s.Transport.(*http.Server).Shutdown(context.Background())
    // 立即关闭所有活动的 net.Listeners 和任何处于 StateNew、StateActive 或 StateIdle 状态的连接
    err := s.Transport.(*http.Server).Close()
    if err != nil {
        return fmt.Errorf("there was an error stopping the HTTP server:\r\n%s", err.Error())
    }
    // 设置服务器状态为关闭
    s.State = servers.Closed
    return nil
}
```