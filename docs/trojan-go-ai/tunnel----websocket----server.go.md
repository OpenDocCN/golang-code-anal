# `trojan-go\tunnel\websocket\server.go`

```
package websocket

import (
    "bufio" // 导入 bufio 包，提供了缓冲 I/O 的功能
    "context" // 导入 context 包，提供了跟踪请求的上下文
    "math/rand" // 导入 math/rand 包，提供了生成伪随机数的功能
    "net" // 导入 net 包，提供了网络操作的基本接口
    "net/http" // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "strings" // 导入 strings 包，提供了操作字符串的函数
    "time" // 导入 time 包，提供了时间的功能

    "golang.org/x/net/websocket" // 导入 websocket 包，提供了 WebSocket 的实现

    "github.com/p4gefau1t/trojan-go/common" // 导入 common 包，提供了一些通用的功能
    "github.com/p4gefau1t/trojan-go/config" // 导入 config 包，提供了配置相关的功能
    "github.com/p4gefau1t/trojan-go/log" // 导入 log 包，提供了日志相关的功能
    "github.com/p4gefau1t/trojan-go/redirector" // 导入 redirector 包，提供了重定向相关的功能
    "github.com/p4gefau1t/trojan-go/tunnel" // 导入 tunnel 包，提供了隧道相关的功能
)

// Fake response writer
// Websocket ServeHTTP method uses Hijack method to get the ReadWriter
type fakeHTTPResponseWriter struct {
    http.Hijacker // 嵌入 http.Hijacker 接口
    http.ResponseWriter // 嵌入 http.ResponseWriter 接口

    ReadWriter *bufio.ReadWriter // 读写器
    Conn       net.Conn // 网络连接
}

func (w *fakeHTTPResponseWriter) Hijack() (net.Conn, *bufio.ReadWriter, error) {
    return w.Conn, w.ReadWriter, nil // 返回网络连接和读写器
}

type Server struct {
    underlay  tunnel.Server // 隧道服务器
    hostname  string // 主机名
    path      string // 路径
    enabled   bool // 是否启用
    redirAddr net.Addr // 重定向地址
    redir     *redirector.Redirector // 重定向器
    ctx       context.Context // 上下文
    cancel    context.CancelFunc // 取消函数
    timeout   time.Duration // 超时时间
}

func (s *Server) Close() error {
    s.cancel() // 取消上下文
    return s.underlay.Close() // 关闭隧道服务器
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := s.underlay.AcceptConn(&Tunnel{}) // 接受连接
    if err != nil {
        return nil, common.NewError("websocket failed to accept connection from underlying server") // 返回错误
    }
    if !s.enabled {
        s.redir.Redirect(&redirector.Redirection{
            InboundConn: conn,
            RedirectTo:  s.redirAddr,
        }) // 重定向连接
        return nil, common.NewError("websocket is disabled. redirecting http request from " + conn.RemoteAddr().String()) // 返回错误
    }
    rewindConn := common.NewRewindConn(conn) // 创建重置连接
    rewindConn.SetBufferSize(512) // 设置缓冲区大小
    defer rewindConn.StopBuffering() // 延迟停止缓冲
    rw := bufio.NewReadWriter(bufio.NewReader(rewindConn), bufio.NewWriter(rewindConn)) // 创建读写器
    req, err := http.ReadRequest(rw.Reader) // 读取请求
    # 如果存在错误
    if err != nil:
        # 记录调试信息
        log.Debug("invalid http request")
        # 重置连接
        rewindConn.Rewind()
        # 停止缓冲
        rewindConn.StopBuffering()
        # 重定向连接
        s.redir.Redirect(&redirector.Redirection{
            InboundConn: rewindConn,
            RedirectTo:  s.redirAddr,
        })
        # 返回错误信息
        return nil, common.NewError("not a valid http request: " + conn.RemoteAddr().String()).Base(err)
    # 如果请求头中的 Upgrade 不是 websocket 或者请求路径不匹配
    if strings.ToLower(req.Header.Get("Upgrade")) != "websocket" || req.URL.Path != s.path:
        # 记录调试信息
        log.Debug("invalid http websocket handshake request")
        # 重置连接
        rewindConn.Rewind()
        # 停止缓冲
        rewindConn.StopBuffering()
        # 重定向连接
        s.redir.Redirect(&redirector.Redirection{
            InboundConn: rewindConn,
            RedirectTo:  s.redirAddr,
        })
        # 返回错误信息
        return nil, common.NewError("not a valid websocket handshake request: " + conn.RemoteAddr().String()).Base(err)
    
    # 创建一个用于握手的通道
    handshake := make(chan struct{})
    
    # 设置 WebSocket 的 URL 和来源
    url := "wss://" + s.hostname + s.path
    origin := "https://" + s.hostname
    # 创建 WebSocket 配置
    wsConfig, err := websocket.NewConfig(url, origin)
    # 如果创建配置出错，返回错误信息
    if err != nil:
        return nil, common.NewError("failed to create websocket config").Base(err)
    # 声明 WebSocket 连接和取消函数
    var wsConn *websocket.Conn
    ctx, cancel := context.WithCancel(s.ctx)
    // 创建一个 WebSocket 服务器对象，配置为传入的 wsConfig，处理函数为匿名函数
    wsServer := websocket.Server{
        Config: *wsConfig,
        Handler: func(conn *websocket.Conn) {
            wsConn = conn                              // 在握手后存储 WebSocket 连接
            wsConn.PayloadType = websocket.BinaryFrame // 将其视为二进制 WebSocket

            log.Debug("websocket obtained")
            handshake <- struct{}{}
            // 除非连接结束，否则此函数不应返回，否则 ServeHTTP 方法将关闭 WebSocket
            <-ctx.Done()
            log.Debug("websocket closed")
        },
        Handshake: func(wsConfig *websocket.Config, httpRequest *http.Request) error {
            log.Debug("websocket url", httpRequest.URL, "origin", httpRequest.Header.Get("Origin"))
            return nil
        },
    }

    // 创建一个假的 HTTP 响应写入器对象，包含连接和读写器
    respWriter := &fakeHTTPResponseWriter{
        Conn:       conn,
        ReadWriter: rw,
    }
    // 启动 WebSocket 服务器的 HTTP 服务
    go wsServer.ServeHTTP(respWriter, req)

    // 选择等待握手信号或超时信号
    select {
    case <-handshake:
    case <-time.After(s.timeout):
    }

    // 如果 WebSocket 连接为空，则取消并返回错误
    if wsConn == nil {
        cancel()
        return nil, common.NewError("websocket failed to handshake")
    }

    // 返回一个 InboundConn 对象，包含 TCP 连接、WebSocket 连接、上下文和取消函数
    return &InboundConn{
        OutboundConn: OutboundConn{
            tcpConn: conn,
            Conn:    wsConn,
        },
        ctx:    ctx,
        cancel: cancel,
    }, nil
# 定义 Server 结构体的 AcceptPacket 方法，接收一个隧道对象，返回一个数据包连接和错误
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    # 返回空值和不支持的错误信息
    return nil, common.NewError("not supported")
}

# 定义 NewServer 函数，接收上下文和底层隧道服务器对象，返回一个 Server 对象和错误
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    # 从上下文中获取配置对象
    cfg := config.FromContext(ctx, Name).(*Config)
    # 如果启用了 WebSocket
    if cfg.Websocket.Enabled {
        # 如果 WebSocket 路径不以 "/" 开头，则返回空值和错误信息
        if !strings.HasPrefix(cfg.Websocket.Path, "/") {
            return nil, common.NewError("websocket path must start with \"/\"")
        }
    }
    # 如果远程主机为空，则记录警告并将远程主机设置为 WebSocket 主机
    if cfg.RemoteHost == "" {
        log.Warn("empty websocket redirection hostname")
        cfg.RemoteHost = cfg.Websocket.Host
    }
    # 如果远程端口为 0，则记录警告并将远程端口设置为 80
    if cfg.RemotePort == 0 {
        log.Warn("empty websocket redirection port")
        cfg.RemotePort = 80
    }
    # 创建一个新的上下文和取消函数
    ctx, cancel := context.WithCancel(ctx)
    log.Debug("websocket server created")
    # 返回一个 Server 对象和空值
    return &Server{
        enabled:   cfg.Websocket.Enabled,
        hostname:  cfg.Websocket.Host,
        path:      cfg.Websocket.Path,
        ctx:       ctx,
        cancel:    cancel,
        underlay:  underlay,
        timeout:   time.Second * time.Duration(rand.Intn(10)+5),
        redir:     redirector.NewRedirector(ctx),
        redirAddr: tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort),
    }, nil
}
```