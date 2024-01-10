# `v2ray-core\proxy\http\client.go`

```
// +build !confonly

package http

import (
    "bufio" // 用于提供缓冲读写功能的包
    "context" // 用于提供上下文功能的包
    "encoding/base64" // 用于提供base64编解码功能的包
    "io" // 用于提供输入输出功能的包
    "net/http" // 用于提供HTTP客户端和服务端功能的包
    "net/url" // 用于提供URL解析功能的包
    "sync" // 用于提供同步功能的包

    "golang.org/x/net/http2" // 提供HTTP/2协议支持的包

    "v2ray.com/core" // V2Ray核心功能包
    "v2ray.com/core/common" // V2Ray通用功能包
    "v2ray.com/core/common/buf" // V2Ray缓冲功能包
    "v2ray.com/core/common/bytespool" // V2Ray字节池功能包
    "v2ray.com/core/common/net" // V2Ray网络功能包
    "v2ray.com/core/common/protocol" // V2Ray协议功能包
    "v2ray.com/core/common/retry" // V2Ray重试功能包
    "v2ray.com/core/common/session" // V2Ray会话功能包
    "v2ray.com/core/common/signal" // V2Ray信号功能包
    "v2ray.com/core/common/task" // V2Ray任务功能包
    "v2ray.com/core/features/policy" // V2Ray策略功能包
    "v2ray.com/core/transport" // V2Ray传输功能包
    "v2ray.com/core/transport/internet" // V2Ray网络传输功能包
    "v2ray.com/core/transport/internet/tls" // V2Ray TLS传输功能包
)

type Client struct {
    serverPicker  protocol.ServerPicker // 服务器选择器
    policyManager policy.Manager // 策略管理器
}

type h2Conn struct {
    rawConn net.Conn // 原始连接
    h2Conn  *http2.ClientConn // HTTP/2客户端连接
}

var (
    cachedH2Mutex sync.Mutex // 缓存HTTP/2连接的互斥锁
    cachedH2Conns map[net.Destination]h2Conn // 缓存的HTTP/2连接
)

// NewClient create a new http client based on the given config.
func NewClient(ctx context.Context, config *ClientConfig) (*Client, error) {
    serverList := protocol.NewServerList() // 创建一个新的服务器列表
    for _, rec := range config.Server {
        s, err := protocol.NewServerSpecFromPB(rec) // 从配置中创建服务器规格
        if err != nil {
            return nil, newError("failed to get server spec").Base(err) // 如果创建失败，则返回错误
        }
        serverList.AddServer(s) // 将服务器添加到服务器列表中
    }
    if serverList.Size() == 0 {
        return nil, newError("0 target server") // 如果服务器列表为空，则返回错误
    }

    v := core.MustFromContext(ctx) // 从上下文中获取核心实例
    return &Client{
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList), // 使用轮询策略创建服务器选择器
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager), // 从核心实例中获取策略管理器
    }, nil
}

// Process implements proxy.Outbound.Process. We first create a socket tunnel via HTTP CONNECT method, then redirect all inbound traffic to that tunnel.
func (c *Client) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    outbound := session.OutboundFromContext(ctx) // 从上下文中获取出站会话
    # 如果出站连接为空或目标无效，则返回新错误
    if outbound == nil || !outbound.Target.IsValid() {
        return newError("target not specified.")
    }
    # 获取目标地址
    target := outbound.Target
    targetAddr := target.NetAddr()

    # 如果目标网络是 UDP，则返回新错误
    if target.Network == net.Network_UDP {
        return newError("UDP is not supported by HTTP outbound")
    }

    # 初始化用户和连接变量
    var user *protocol.MemoryUser
    var conn internet.Connection

    # 从读取器中读取多个缓冲区
    mbuf, _ := link.Reader.ReadMultiBuffer()
    len := mbuf.Len()
    firstPayload := bytespool.Alloc(len)
    mbuf, _ = buf.SplitBytes(mbuf, firstPayload)
    firstPayload = firstPayload[:len]

    # 释放多个缓冲区
    buf.ReleaseMulti(mbuf)
    # 延迟释放第一个有效载荷
    defer bytespool.Free(firstPayload)

    # 使用指数退避算法进行重试
    if err := retry.ExponentialBackoff(5, 100).On(func() error {
        # 选择服务器并获取目标和用户
        server := c.serverPicker.PickServer()
        dest := server.Destination()
        user = server.PickUser()

        # 设置HTTP隧道连接
        netConn, err := setUpHTTPTunnel(ctx, dest, targetAddr, user, dialer, firstPayload)
        if netConn != nil {
            conn = internet.Connection(netConn)
        }
        return err
    }); err != nil {
        return newError("failed to find an available destination").Base(err)
    }

    # 延迟关闭连接
    defer func() {
        if err := conn.Close(); err != nil {
            newError("failed to closed connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
        }
    }()

    # 获取策略
    p := c.policyManager.ForLevel(0)
    if user != nil {
        p = c.policyManager.ForLevel(user.Level)
    }

    # 创建上下文和取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 设置信号超时
    timer := signal.CancelAfterInactivity(ctx, cancel, p.Timeouts.ConnectionIdle)

    # 请求函数
    requestFunc := func() error {
        defer timer.SetTimeout(p.Timeouts.DownlinkOnly)
        return buf.Copy(link.Reader, buf.NewWriter(conn), buf.UpdateActivity(timer))
    }
    # 响应函数
    responseFunc := func() error {
        defer timer.SetTimeout(p.Timeouts.UplinkOnly)
        return buf.Copy(buf.NewReader(conn), link.Writer, buf.UpdateActivity(timer))
    }

    # 响应完成后的任务
    var responseDonePost = task.OnSuccess(responseFunc, task.Close(link.Writer))
    # 如果任务执行时发生错误，将错误赋值给 err
    if err := task.Run(ctx, requestFunc, responseDonePost); err != nil:
        # 返回一个新的错误，基于之前的错误，并添加"connection ends"的描述
        return newError("connection ends").Base(err)
    
    # 如果没有发生错误，返回空值
    return nil
// setUpHTTPTunnel函数将通过HTTP CONNECT方法创建一个套接字隧道
func setUpHTTPTunnel(ctx context.Context, dest net.Destination, target string, user *protocol.MemoryUser, dialer internet.Dialer, firstPayload []byte) (net.Conn, error) {
    // 创建一个HTTP请求
    req := &http.Request{
        Method: http.MethodConnect, // 设置请求方法为CONNECT
        URL:    &url.URL{Host: target}, // 设置请求的URL
        Header: make(http.Header), // 创建一个空的Header
        Host:   target, // 设置请求的目标主机
    }

    // 如果用户不为空且用户账户不为空
    if user != nil && user.Account != nil {
        account := user.Account.(*Account)
        auth := account.GetUsername() + ":" + account.GetPassword()
        req.Header.Set("Proxy-Authorization", "Basic "+base64.StdEncoding.EncodeToString([]byte(auth))) // 设置代理授权信息
    }

    // connectHTTP1函数用于发送HTTP CONNECT请求
    connectHTTP1 := func(rawConn net.Conn) (net.Conn, error) {
        req.Header.Set("Proxy-Connection", "Keep-Alive") // 设置代理连接为保持连接

        err := req.Write(rawConn) // 将请求写入到连接中
        if err != nil {
            rawConn.Close()
            return nil, err
        }

        if _, err := rawConn.Write(firstPayload); err != nil { // 将第一个有效载荷写入连接
            rawConn.Close()
            return nil, err
        }

        resp, err := http.ReadResponse(bufio.NewReader(rawConn), req) // 从连接中读取响应
        if err != nil {
            rawConn.Close()
            return nil, err
        }

        if resp.StatusCode != http.StatusOK { // 如果响应状态码不是200
            rawConn.Close()
            return nil, newError("Proxy responded with non 200 code: " + resp.Status) // 返回错误信息
        }
        return rawConn, nil
    }
}
    // 定义一个名为 connectHTTP2 的匿名函数，接受原始连接和 HTTP/2 客户端连接作为参数，返回一个新的连接和可能的错误
    connectHTTP2 := func(rawConn net.Conn, h2clientConn *http2.ClientConn) (net.Conn, error) {
        // 创建一个管道，用于将数据从请求体传输到 HTTP/2 客户端连接
        pr, pw := io.Pipe()
        // 将管道的写端设置为请求体
        req.Body = pr

        var pErr error
        var wg sync.WaitGroup
        wg.Add(1)

        // 启动一个 goroutine，将第一个有效载荷写入管道，并在完成后标记完成
        go func() {
            _, pErr = pw.Write(firstPayload)
            wg.Done()
        }()

        // 发起 HTTP/2 请求，并获取响应
        resp, err := h2clientConn.RoundTrip(req)
        if err != nil {
            rawConn.Close()
            return nil, err
        }

        // 等待 goroutine 完成，并检查是否有错误发生
        wg.Wait()
        if pErr != nil {
            rawConn.Close()
            return nil, pErr
        }

        // 检查代理响应的状态码，如果不是 200，则关闭原始连接并返回错误
        if resp.StatusCode != http.StatusOK {
            rawConn.Close()
            return nil, newError("Proxy responded with non 200 code: " + resp.Status)
        }
        // 返回一个新的 HTTP/2 连接
        return newHTTP2Conn(rawConn, pw, resp.Body), nil
    }

    // 加锁以访问缓存的 HTTP/2 连接
    cachedH2Mutex.Lock()
    cachedConn, cachedConnFound := cachedH2Conns[dest]
    cachedH2Mutex.Unlock()

    // 如果找到缓存的连接
    if cachedConnFound {
        rc, cc := cachedConn.rawConn, cachedConn.h2Conn
        // 如果缓存的连接可以接受新请求
        if cc.CanTakeNewRequest() {
            // 连接到 HTTP/2 代理，并返回代理连接或错误
            proxyConn, err := connectHTTP2(rc, cc)
            if err != nil {
                return nil, err
            }

            return proxyConn, nil
        }
    }

    // 如果没有找到缓存的连接，则通过拨号器建立新的原始连接
    rawConn, err := dialer.Dial(ctx, dest)
    if err != nil {
        return nil, err
    }

    iConn := rawConn
    // 如果连接是统计连接，则将其转换为基础连接
    if statConn, ok := iConn.(*internet.StatCouterConnection); ok {
        iConn = statConn.Connection
    }

    nextProto := ""
    // 如果连接是 TLS 连接，则进行 TLS 握手，并获取协商的协议
    if tlsConn, ok := iConn.(*tls.Conn); ok {
        if err := tlsConn.Handshake(); err != nil {
            rawConn.Close()
            return nil, err
        }
        nextProto = tlsConn.ConnectionState().NegotiatedProtocol
    }

    // 根据协商的协议类型进行不同的处理
    switch nextProto {
    // 如果协议为空或为 HTTP/1.1，则连接到 HTTP/1.1 代理
    case "", "http/1.1":
        return connectHTTP1(rawConn)
    # 根据协议类型进行不同的处理
    case "h2":
        # 创建一个 HTTP/2 传输对象
        t := http2.Transport{}
        # 使用传输对象创建一个新的客户端连接
        h2clientConn, err := t.NewClientConn(rawConn)
        # 如果创建连接出错，则关闭原始连接并返回错误
        if err != nil {
            rawConn.Close()
            return nil, err
        }

        # 通过 HTTP/2 连接进行代理连接
        proxyConn, err := connectHTTP2(rawConn, h2clientConn)
        # 如果代理连接出错，则关闭原始连接并返回错误
        if err != nil {
            rawConn.Close()
            return nil, err
        }

        # 加锁，以确保并发安全
        cachedH2Mutex.Lock()
        # 如果缓存的 HTTP/2 连接为空，则创建一个新的映射
        if cachedH2Conns == nil {
            cachedH2Conns = make(map[net.Destination]h2Conn)
        }

        # 将目标地址和 HTTP/2 连接映射存储到缓存中
        cachedH2Conns[dest] = h2Conn{
            rawConn: rawConn,
            h2Conn:  h2clientConn,
        }
        # 解锁
        cachedH2Mutex.Unlock()

        # 返回代理连接和可能的错误
        return proxyConn, err
    # 如果协议类型不是 "h2"，则返回错误
    default:
        return nil, newError("negotiated unsupported application layer protocol: " + nextProto)
    }
# 创建一个新的基于HTTP2的连接
func newHTTP2Conn(c net.Conn, pipedReqBody *io.PipeWriter, respBody io.ReadCloser) net.Conn {
    # 返回一个http2Conn对象，包含传入的连接、请求体和响应体
    return &http2Conn{Conn: c, in: pipedReqBody, out: respBody}
}

# 定义http2Conn结构体，包含net.Conn类型的连接、*io.PipeWriter类型的请求体和io.ReadCloser类型的响应体
type http2Conn struct {
    net.Conn
    in  *io.PipeWriter
    out io.ReadCloser
}

# 重写http2Conn的Read方法，从响应体中读取数据
func (h *http2Conn) Read(p []byte) (n int, err error) {
    return h.out.Read(p)
}

# 重写http2Conn的Write方法，将数据写入请求体
func (h *http2Conn) Write(p []byte) (n int, err error) {
    return h.in.Write(p)
}

# 重写http2Conn的Close方法，关闭请求体和响应体
func (h *http2Conn) Close() error {
    h.in.Close()
    return h.out.Close()
}

# 初始化函数，注册ClientConfig类型的配置，并返回一个新的Client对象
func init() {
    common.Must(common.RegisterConfig((*ClientConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewClient(ctx, config.(*ClientConfig))
    }))
}
```