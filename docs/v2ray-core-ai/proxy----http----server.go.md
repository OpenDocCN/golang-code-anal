# `v2ray-core\proxy\http\server.go`

```
// +build !confonly

package http

import (
    "bufio" // 导入 bufio 包，提供了缓冲读取功能
    "context" // 导入 context 包，提供了跟踪请求的上下文
    "encoding/base64" // 导入 base64 包，提供了 base64 编码和解码功能
    "io" // 导入 io 包，提供了基本的 I/O 接口
    "net/http" // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "strings" // 导入 strings 包，提供了操作字符串的函数
    "time" // 导入 time 包，提供了时间的表示和计算

    "v2ray.com/core" // 导入 v2ray 核心包
    "v2ray.com/core/common" // 导入 v2ray 核心通用包
    "v2ray.com/core/common/buf" // 导入 v2ray 核心通用缓冲包
    "v2ray.com/core/common/errors" // 导入 v2ray 核心通用错误包
    "v2ray.com/core/common/log" // 导入 v2ray 核心通用日志包
    "v2ray.com/core/common/net" // 导入 v2ray 核心通用网络包
    "v2ray.com/core/common/protocol" // 导入 v2ray 核心通用协议包
    http_proto "v2ray.com/core/common/protocol/http" // 导入 v2ray 核心 HTTP 协议包
    "v2ray.com/core/common/session" // 导入 v2ray 核心会话包
    "v2ray.com/core/common/signal" // 导入 v2ray 核心信号包
    "v2ray.com/core/common/task" // 导入 v2ray 核心任务包
    "v2ray.com/core/features/policy" // 导入 v2ray 核心策略包
    "v2ray.com/core/features/routing" // 导入 v2ray 核心路由包
    "v2ray.com/core/transport/internet" // 导入 v2ray 核心网络传输包
)

// Server is an HTTP proxy server.
type Server struct {
    config        *ServerConfig // 服务器配置
    policyManager policy.Manager // 策略管理器
}

// NewServer creates a new HTTP inbound handler.
func NewServer(ctx context.Context, config *ServerConfig) (*Server, error) {
    v := core.MustFromContext(ctx) // 从上下文中获取核心实例
    s := &Server{
        config:        config, // 设置服务器配置
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager), // 获取并设置策略管理器
    }

    return s, nil // 返回服务器实例和空错误
}

func (s *Server) policy() policy.Session {
    config := s.config // 获取服务器配置
    p := s.policyManager.ForLevel(config.UserLevel) // 根据用户级别获取策略
    if config.Timeout > 0 && config.UserLevel == 0 { // 如果超时大于0且用户级别为0
        p.Timeouts.ConnectionIdle = time.Duration(config.Timeout) * time.Second // 设置连接空闲超时时间
    }
    return p // 返回策略
}

// Network implements proxy.Inbound.
func (*Server) Network() []net.Network {
    return []net.Network{net.Network_TCP} // 返回 TCP 网络
}

func isTimeout(err error) bool {
    nerr, ok := errors.Cause(err).(net.Error) // 判断错误是否为超时错误
    return ok && nerr.Timeout() // 返回是否为超时错误
}

func parseBasicAuth(auth string) (username, password string, ok bool) {
    const prefix = "Basic " // 定义基本认证的前缀
    if !strings.HasPrefix(auth, prefix) { // 如果认证字符串不以基本认证前缀开头
        return // 返回空
    }
    c, err := base64.StdEncoding.DecodeString(auth[len(prefix):]) // 解码认证字符串
    if err != nil { // 如果解码出错
        return // 返回空
    }
    cs := string(c) // 将解码后的字节转换为字符串
    s := strings.IndexByte(cs, ':') // 查找用户名和密码分隔符的位置
    if s < 0 { // 如果找不到分隔符
        return // 返回空
    }
    return cs[:s], cs[s+1:], true // 返回用户名、密码和 true
}
# 定义一个只读的结构体，包含一个 io.Reader 接口
type readerOnly struct {
    io.Reader
}

# 处理服务器的请求
func (s *Server) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error {
    # 从上下文中获取入站信息
    inbound := session.InboundFromContext(ctx)
    if inbound != nil {
        # 如果入站信息不为空，设置用户级别
        inbound.User = &protocol.MemoryUser{
            Level: s.config.UserLevel,
        }
    }

    # 创建一个带缓冲的读取器
    reader := bufio.NewReaderSize(readerOnly{conn}, buf.Size)

Start:
    # 设置读取截止时间
    if err := conn.SetReadDeadline(time.Now().Add(s.policy().Timeouts.Handshake)); err != nil {
        newError("failed to set read deadline").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }

    # 读取 HTTP 请求
    request, err := http.ReadRequest(reader)
    if err != nil {
        trace := newError("failed to read http request").Base(err)
        if errors.Cause(err) != io.EOF && !isTimeout(errors.Cause(err)) {
            trace.AtWarning() // nolint: errcheck
        }
        return trace
    }

    # 如果配置中有账户信息，解析基本认证信息
    if len(s.config.Accounts) > 0 {
        user, pass, ok := parseBasicAuth(request.Header.Get("Proxy-Authorization"))
        if !ok || !s.config.HasAccount(user, pass) {
            return common.Error2(conn.Write([]byte("HTTP/1.1 407 Proxy Authentication Required\r\nProxy-Authenticate: Basic realm=\"proxy\"\r\nConnection: close\r\n\r\n")))
        }
        if inbound != nil {
            inbound.User.Email = user
        }
    }

    # 记录请求的方法、主机和 URL
    newError("request to Method [", request.Method, "] Host [", request.Host, "] with URL [", request.URL, "]").WriteToLog(session.ExportIDToError(ctx))
    if err := conn.SetReadDeadline(time.Time{}); err != nil {
        newError("failed to clear read deadline").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }

    # 设置默认端口号
    defaultPort := net.Port(80)
    if strings.EqualFold(request.URL.Scheme, "https") {
        defaultPort = net.Port(443)
    }
    host := request.Host
    if host == "" {
        host = request.URL.Host
    }
    # 解析主机和端口
    dest, err := http_proto.ParseHost(host, defaultPort)
    // 如果存在错误，返回一个包含错误信息的新错误对象
    if err != nil {
        return newError("malformed proxy host: ", host).AtWarning().Base(err)
    }
    // 为访问消息创建上下文
    ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
        From:   conn.RemoteAddr(),
        To:     request.URL,
        Status: log.AccessAccepted,
        Reason: "",
    })

    // 如果请求方法是 CONNECT，则调用 handleConnect 方法处理
    if strings.EqualFold(request.Method, "CONNECT") {
        return s.handleConnect(ctx, request, reader, conn, dest, dispatcher)
    }

    // 检查是否需要保持连接
    keepAlive := (strings.TrimSpace(strings.ToLower(request.Header.Get("Proxy-Connection"))) == "keep-alive")

    // 调用 handlePlainHTTP 方法处理普通的 HTTP 请求
    err = s.handlePlainHTTP(ctx, request, conn, dest, dispatcher)
    // 如果返回的错误是 errWaitAnother
    if err == errWaitAnother {
        // 如果需要保持连接，则跳转到 Start 标签处重新开始处理
        if keepAlive {
            goto Start
        }
        // 否则将错误置为空
        err = nil
    }

    // 返回处理结果
    return err
// 处理 CONNECT 方法的请求
func (s *Server) handleConnect(ctx context.Context, request *http.Request, reader *bufio.Reader, conn internet.Connection, dest net.Destination, dispatcher routing.Dispatcher) error {
    // 向客户端发送连接建立成功的响应
    _, err := conn.Write([]byte("HTTP/1.1 200 Connection established\r\n\r\n"))
    if err != nil {
        return newError("failed to write back OK response").Base(err)
    }

    // 获取服务器的策略
    plcy := s.policy()
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 根据策略设置定时器
    timer := signal.CancelAfterInactivity(ctx, cancel, plcy.Timeouts.ConnectionIdle)

    // 根据策略设置上下文的缓冲策略
    ctx = policy.ContextWithBufferPolicy(ctx, plcy.Buffer)
    // 根据目标地址分发连接
    link, err := dispatcher.Dispatch(ctx, dest)
    if err != nil {
        return err
    }

    // 如果读取缓冲区中有数据，则读取数据并发送到目标地址
    if reader.Buffered() > 0 {
        payload, err := buf.ReadFrom(io.LimitReader(reader, int64(reader.Buffered())))
        if err != nil {
            return err
        }
        if err := link.Writer.WriteMultiBuffer(payload); err != nil {
            return err
        }
        reader = nil
    }

    // 处理请求的函数
    requestDone := func() error {
        defer timer.SetTimeout(plcy.Timeouts.DownlinkOnly)

        return buf.Copy(buf.NewReader(conn), link.Writer, buf.UpdateActivity(timer))
    }

    // 处理响应的函数
    responseDone := func() error {
        defer timer.SetTimeout(plcy.Timeouts.UplinkOnly)

        v2writer := buf.NewWriter(conn)
        if err := buf.Copy(link.Reader, v2writer, buf.UpdateActivity(timer)); err != nil {
            return err
        }

        return nil
    }

    // 关闭写入连接的任务
    var closeWriter = task.OnSuccess(requestDone, task.Close(link.Writer))
    if err := task.Run(ctx, closeWriter, responseDone); err != nil {
        common.Interrupt(link.Reader)
        common.Interrupt(link.Writer)
        return newError("connection ends").Base(err)
    }

    return nil
}

// 错误类型：等待另一个请求
var errWaitAnother = newError("keep alive")

// 处理普通的 HTTP 请求
func (s *Server) handlePlainHTTP(ctx context.Context, request *http.Request, writer io.Writer, dest net.Destination, dispatcher routing.Dispatcher) error {
    // 如果不允许透明代理并且请求的 URL 主机为空
    if !s.config.AllowTransparent && request.URL.Host == "" {
        // 创建一个 HTTP 响应对象
        response := &http.Response{
            Status:        "Bad Request",  // 设置状态信息为 "Bad Request"
            StatusCode:    400,  // 设置状态码为 400
            Proto:         "HTTP/1.1",  // 设置协议版本为 HTTP/1.1
            ProtoMajor:    1,  // 设置主版本号为 1
            ProtoMinor:    1,  // 设置次版本号为 1
            Header:        http.Header(make(map[string][]string)),  // 创建空的 HTTP 头部
            Body:          nil,  // 设置消息体为空
            ContentLength: 0,  // 设置内容长度为 0
            Close:         true,  // 设置连接关闭标志为 true
        }
        // 设置响应头部的 "Proxy-Connection" 字段为 "close"
        response.Header.Set("Proxy-Connection", "close")
        // 设置响应头部的 "Connection" 字段为 "close"
        response.Header.Set("Connection", "close")
        // 将响应写入到 writer 中
        return response.Write(writer)
    }

    // 如果请求的 URL 主机不为空
    if len(request.URL.Host) > 0 {
        // 将请求的 Host 字段设置为请求的 URL 主机
        request.Host = request.URL.Host
    }
    // 移除请求头部中的 hop-by-hop 头部字段
    http_proto.RemoveHopByHopHeaders(request.Header)

    // 如果请求头部中的 "User-Agent" 字段为空
    // 防止 UA 被设置为 golang 的默认值
    if request.Header.Get("User-Agent") == "" {
        // 将请求头部中的 "User-Agent" 字段设置为空字符串
        request.Header.Set("User-Agent", "")
    }

    // 创建一个 session.Content 对象
    content := &session.Content{
        Protocol: "http/1.1",  // 设置协议版本为 http/1.1
    }

    // 设置 content 对象的 ":method" 属性为请求方法的大写形式
    content.SetAttribute(":method", strings.ToUpper(request.Method))
    // 设置 content 对象的 ":path" 属性为请求的 URL 路径
    content.SetAttribute(":path", request.URL.Path)
    // 遍历请求头部，将头部字段名转换为小写并设置到 content 对象的属性中
    for key := range request.Header {
        value := request.Header.Get(key)
        content.SetAttribute(strings.ToLower(key), value)
    }

    // 使用 content 对象创建一个新的上下文
    ctx = session.ContextWithContent(ctx, content)

    // 调度请求并获取响应
    link, err := dispatcher.Dispatch(ctx, dest)
    // 如果出现错误，返回错误
    if err != nil {
        return err
    }

    // 延迟关闭请求的写入流
    defer common.Close(link.Writer) // nolint: errcheck
    // 初始化 result 变量为 errWaitAnother
    var result error = errWaitAnother
    # 定义一个匿名函数，用于处理请求完成时的操作
    requestDone := func() error {
        # 设置请求头中的 Connection 字段为 close，表示请求完成后关闭连接
        request.Header.Set("Connection", "close")

        # 创建一个不带缓冲的写入器，用于将请求写入到链接中
        requestWriter := buf.NewBufferedWriter(link.Writer)
        # 设置写入器为非缓冲模式
        common.Must(requestWriter.SetBuffered(false))
        # 将整个请求写入到链接中
        if err := request.Write(requestWriter); err != nil {
            return newError("failed to write whole request").Base(err).AtWarning()
        }
        return nil
    }

    # 定义一个匿名函数，用于处理响应完成时的操作
    responseDone := func() error {
        # 创建一个指定大小的带缓冲的读取器，用于从链接中读取响应
        responseReader := bufio.NewReaderSize(&buf.BufferedReader{Reader: link.Reader}, buf.Size)
        # 从响应读取器中读取响应，并返回响应对象和可能的错误
        response, err := http.ReadResponse(responseReader, request)
        # 如果没有错误
        if err == nil {
            # 移除响应头中的与连接相关的字段
            http_proto.RemoveHopByHopHeaders(response.Header)
            # 如果响应的内容长度大于等于0
            if response.ContentLength >= 0 {
                # 设置响应头中的 Connection 和 Proxy-Connection 字段为 keep-alive，表示保持连接
                response.Header.Set("Proxy-Connection", "keep-alive")
                response.Header.Set("Connection", "keep-alive")
                # 设置响应头中的 Keep-Alive 字段为 timeout=4，表示保持连接的超时时间为4秒
                response.Header.Set("Keep-Alive", "timeout=4")
                # 设置响应的 Close 字段为 false，表示不关闭连接
                response.Close = false
            } else {
                # 如果响应的内容长度小于0，则设置响应的 Close 字段为 true，表示关闭连接，并将结果设置为nil
                response.Close = true
                result = nil
            }
        } else {
            # 如果有错误，则记录错误信息，并创建一个包含错误信息的响应对象
            newError("failed to read response from ", request.Host).Base(err).AtWarning().WriteToLog(session.ExportIDToError(ctx))
            response = &http.Response{
                Status:        "Service Unavailable",
                StatusCode:    503,
                Proto:         "HTTP/1.1",
                ProtoMajor:    1,
                ProtoMinor:    1,
                Header:        http.Header(make(map[string][]string)),
                Body:          nil,
                ContentLength: 0,
                Close:         true,
            }
            # 设置响应头中的 Connection 和 Proxy-Connection 字段为 close，表示关闭连接
            response.Header.Set("Connection", "close")
            response.Header.Set("Proxy-Connection", "close")
        }
        # 将响应写入到写入器中
        if err := response.Write(writer); err != nil {
            return newError("failed to write response").Base(err).AtWarning()
        }
        return nil
    }
    # 如果 task.Run 方法返回错误，则执行以下代码块
    if err := task.Run(ctx, requestDone, responseDone); err != nil:
        # 中断 link.Reader
        common.Interrupt(link.Reader)
        # 中断 link.Writer
        common.Interrupt(link.Writer)
        # 返回一个新的错误，基于原始错误 "connection ends"
        return newError("connection ends").Base(err)
    
    # 返回结果
    return result
# 初始化函数，用于注册服务器配置并创建服务器实例
func init() {
    # 注册服务器配置，并在注册后立即创建服务器实例
    common.Must(common.RegisterConfig((*ServerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewServer(ctx, config.(*ServerConfig))
    }))
}
```