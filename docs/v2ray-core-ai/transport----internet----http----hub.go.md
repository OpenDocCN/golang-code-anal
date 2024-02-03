# `v2ray-core\transport\internet\http\hub.go`

```go
// +build !confonly

// 声明 http 包，引入所需的依赖包
package http

import (
    "context"
    "io"
    "net/http"
    "strings"
    "time"

    "golang.org/x/net/http2"
    "golang.org/x/net/http2/h2c"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    http_proto "v2ray.com/core/common/protocol/http"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/common/session"
    "v2ray.com/core/common/signal/done"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
)

// Listener 结构体定义
type Listener struct {
    server  *http.Server
    handler internet.ConnHandler
    local   net.Addr
    config  *Config
}

// Addr 方法返回本地地址
func (l *Listener) Addr() net.Addr {
    return l.local
}

// Close 方法关闭服务器
func (l *Listener) Close() error {
    return l.server.Close()
}

// flushWriter 结构体定义
type flushWriter struct {
    w io.Writer
    d *done.Instance
}

// Write 方法实现了 io.Writer 接口
func (fw flushWriter) Write(p []byte) (n int, err error) {
    if fw.d.Done() {
        return 0, io.ErrClosedPipe
    }

    n, err = fw.w.Write(p)
    if f, ok := fw.w.(http.Flusher); ok {
        f.Flush()
    }
    return
}

// ServeHTTP 方法处理 HTTP 请求
func (l *Listener) ServeHTTP(writer http.ResponseWriter, request *http.Request) {
    host := request.Host
    // 验证请求的主机是否有效
    if !l.config.isValidHost(host) {
        writer.WriteHeader(404)
        return
    }
    path := l.config.getNormalizedPath()
    // 验证请求的路径是否有效
    if !strings.HasPrefix(request.URL.Path, path) {
        writer.WriteHeader(404)
        return
    }

    // 设置响应头
    writer.Header().Set("Cache-Control", "no-store")
    writer.WriteHeader(200)
    // 刷新响应
    if f, ok := writer.(http.Flusher); ok {
        f.Flush()
    }

    remoteAddr := l.Addr()
    // 解析请求的远程地址
    dest, err := net.ParseDestination(request.RemoteAddr)
    if err != nil {
        newError("failed to parse request remote addr: ", request.RemoteAddr).Base(err).WriteToLog()
    } else {
        remoteAddr = &net.TCPAddr{
            IP:   dest.Address.IP(),
            Port: int(dest.Port),
        }
    }

    // 解析 X-Forwarded-For 头部，获取转发的地址
    forwardedAddrs := http_proto.ParseXForwardedFor(request.Header)
}
    # 检查 forwardedAddrs 数组的长度是否大于 0，并且第一个元素的地址族是否为 IP 地址
    if len(forwardedAddrs) > 0 && forwardedAddrs[0].Family().IsIP() {
        # 如果条件成立，将远程地址的 IP 设置为第一个 forwardedAddrs 的 IP 地址
        remoteAddr.(*net.TCPAddr).IP = forwardedAddrs[0].IP()
    }

    # 创建一个新的完成信号
    done := done.New()
    # 创建一个新的网络连接
    conn := net.NewConnection(
        # 设置连接的输出为请求的 Body
        net.ConnectionOutput(request.Body),
        # 设置连接的输入为 flushWriter 结构体的实例，其中包含了 writer 和 done 信号
        net.ConnectionInput(flushWriter{w: writer, d: done}),
        # 设置连接关闭时的操作为同时关闭 done 信号和请求的 Body
        net.ConnectionOnClose(common.ChainedClosable{done, request.Body}),
        # 设置连接的本地地址为 l 的地址
        net.ConnectionLocalAddr(l.Addr()),
        # 设置连接的远程地址为 remoteAddr
        net.ConnectionRemoteAddr(remoteAddr),
    )
    # 调用连接处理函数
    l.handler(conn)
    # 等待完成信号的完成
    <-done.Wait()
}

// Listen函数用于创建一个监听器，接受传入的连接请求
func Listen(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, handler internet.ConnHandler) (internet.Listener, error) {
    // 从streamSettings中获取http协议的配置
    httpSettings := streamSettings.ProtocolSettings.(*Config)
    // 创建一个Listener对象
    listener := &Listener{
        handler: handler,
        local: &net.TCPAddr{
            IP:   address.IP(),
            Port: int(port),
        },
        config: httpSettings,
    }

    var server *http.Server
    // 根据streamSettings创建TLS配置
    config := tls.ConfigFromStreamSettings(streamSettings)
    if config == nil {
        h2s := &http2.Server{}
        // 创建一个不带TLS的http.Server对象
        server = &http.Server{
            Addr:              serial.Concat(address, ":", port),
            Handler:           h2c.NewHandler(listener, h2s),
            ReadHeaderTimeout: time.Second * 4,
        }
    } else {
        // 创建一个带TLS的http.Server对象
        server = &http.Server{
            Addr:              serial.Concat(address, ":", port),
            TLSConfig:         config.GetTLSConfig(tls.WithNextProto("h2")),
            Handler:           listener,
            ReadHeaderTimeout: time.Second * 4,
        }
    }

    // 将server对象赋值给listener的server属性
    listener.server = server
    // 启动一个goroutine来监听传入的连接请求
    go func() {
        // 在指定的地址和端口上创建TCP监听器
        tcpListener, err := internet.ListenSystem(ctx, &net.TCPAddr{
            IP:   address.IP(),
            Port: int(port),
        }, streamSettings.SocketSettings)
        if err != nil {
            // 如果监听失败，则记录错误日志
            newError("failed to listen on", address, ":", port).Base(err).WriteToLog(session.ExportIDToError(ctx))
            return
        }
        if config == nil {
            // 如果没有TLS配置，则使用server.Serve方法来处理连接请求
            err = server.Serve(tcpListener)
            if err != nil {
                newError("stoping serving H2C").Base(err).WriteToLog(session.ExportIDToError(ctx))
            }
        } else {
            // 如果有TLS配置，则使用server.ServeTLS方法来处理连接请求
            err = server.ServeTLS(tcpListener, "", "")
            if err != nil {
                newError("stoping serving TLS").Base(err).WriteToLog(session.ExportIDToError(ctx))
            }
        }
    }()

    // 返回创建的listener对象
    return listener, nil
}

// 初始化函数
func init() {
    # 注册传输监听器，当指定协议的传输发生时调用指定的监听函数
    common.Must(internet.RegisterTransportListener(protocolName, Listen))
# 闭合前面的函数定义
```