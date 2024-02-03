# `v2ray-core\transport\internet\websocket\hub.go`

```go
// +build !confonly

package websocket

import (
    "context"
    "crypto/tls"
    "net/http"
    "sync"
    "time"

    "github.com/gorilla/websocket"
    "github.com/pires/go-proxyproto"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    http_proto "v2ray.com/core/common/protocol/http"
    "v2ray.com/core/common/session"
    "v2ray.com/core/transport/internet"
    v2tls "v2ray.com/core/transport/internet/tls"
)

type requestHandler struct {
    path string
    ln   *Listener
}

// 创建 WebSocket 升级器
var upgrader = &websocket.Upgrader{
    ReadBufferSize:   4 * 1024,
    WriteBufferSize:  4 * 1024,
    HandshakeTimeout: time.Second * 4,
    CheckOrigin: func(r *http.Request) bool {
        return true
    },
}

// 处理 WebSocket 请求的方法
func (h *requestHandler) ServeHTTP(writer http.ResponseWriter, request *http.Request) {
    // 如果请求的路径不匹配，则返回 404
    if request.URL.Path != h.path {
        writer.WriteHeader(http.StatusNotFound)
        return
    }
    // 尝试将 HTTP 连接升级为 WebSocket 连接
    conn, err := upgrader.Upgrade(writer, request, nil)
    if err != nil {
        newError("failed to convert to WebSocket connection").Base(err).WriteToLog()
        return
    }

    // 解析 X-Forwarded-For 头部，获取转发的地址
    forwardedAddrs := http_proto.ParseXForwardedFor(request.Header)
    remoteAddr := conn.RemoteAddr()
    // 如果存在转发地址且为 IP 地址，则替换远程地址
    if len(forwardedAddrs) > 0 && forwardedAddrs[0].Family().IsIP() {
        remoteAddr.(*net.TCPAddr).IP = forwardedAddrs[0].IP()
    }

    // 将连接添加到监听器中
    h.ln.addConn(newConnection(conn, remoteAddr))
}

// WebSocket 监听器
type Listener struct {
    sync.Mutex
    server   http.Server
    listener net.Listener
    config   *Config
    addConn  internet.ConnHandler
}

// 监听 WebSocket 连接
func ListenWS(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, addConn internet.ConnHandler) (internet.Listener, error) {
    // 使用系统默认的网络协议栈监听 TCP 地址
    listener, err := internet.ListenSystem(ctx, &net.TCPAddr{
        IP:   address.IP(),
        Port: int(port),
    }, streamSettings.SocketSettings)
    if err != nil {
        return nil, newError("failed to listen TCP(for WS) on", address, ":", port).Base(err)
    }
    # 创建一个新的错误消息，记录监听的地址和端口，并将错误信息写入日志
    newError("listening TCP(for WS) on ", address, ":", port).WriteToLog(session.ExportIDToError(ctx))

    # 从流设置中获取 WebSocket 的配置
    wsSettings := streamSettings.ProtocolSettings.(*Config)

    # 如果允许代理协议，则设置代理策略函数，并创建一个带有代理协议的监听器
    if wsSettings.AcceptProxyProtocol {
        policyFunc := func(upstream net.Addr) (proxyproto.Policy, error) { return proxyproto.REQUIRE, nil }
        listener = &proxyproto.Listener{Listener: listener, Policy: policyFunc}
        # 记录接受代理协议的信息，并将其写入日志
        newError("accepting PROXY protocol").AtWarning().WriteToLog(session.ExportIDToError(ctx))
    }

    # 根据流设置创建 TLS 配置
    if config := v2tls.ConfigFromStreamSettings(streamSettings); config != nil {
        if tlsConfig := config.GetTLSConfig(); tlsConfig != nil {
            # 如果 TLS 配置不为空，则使用 TLS 配置创建一个新的监听器
            listener = tls.NewListener(listener, tlsConfig)
        }
    }

    # 创建一个新的监听器对象
    l := &Listener{
        config:   wsSettings,
        addConn:  addConn,
        listener: listener,
    }

    # 配置 HTTP 服务器
    l.server = http.Server{
        Handler: &requestHandler{
            path: wsSettings.GetNormalizedPath(),
            ln:   l,
        },
        ReadHeaderTimeout: time.Second * 4,
        MaxHeaderBytes:    2048,
    }

    # 启动一个 goroutine 来监听并处理连接
    go func() {
        if err := l.server.Serve(l.listener); err != nil {
            # 如果出现错误，则记录失败信息，并将其写入日志
            newError("failed to serve http for WebSocket").Base(err).AtWarning().WriteToLog(session.ExportIDToError(ctx))
        }
    }()

    # 返回监听器对象和错误信息
    return l, err
// Addr 实现了 net.Listener.Addr() 方法，返回监听器的地址信息
func (ln *Listener) Addr() net.Addr {
    return ln.listener.Addr()
}

// Close 实现了 net.Listener.Close() 方法，关闭监听器
func (ln *Listener) Close() error {
    return ln.listener.Close()
}

// init 函数用于注册 WebSocket 监听器
func init() {
    common.Must(internet.RegisterTransportListener(protocolName, ListenWS))
}
```