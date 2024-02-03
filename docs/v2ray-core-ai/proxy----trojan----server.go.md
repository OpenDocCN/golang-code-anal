# `v2ray-core\proxy\trojan\server.go`

```go
// +build !confonly

package trojan

import (
    "context" // 引入上下文包
    "crypto/tls" // 引入加密包
    "io" // 引入输入输出包
    "strconv" // 引入字符串转换包
    "time" // 引入时间包

    "v2ray.com/core" // 引入 V2Ray 核心包
    "v2ray.com/core/common" // 引入 V2Ray 公共包
    "v2ray.com/core/common/buf" // 引入 V2Ray 缓冲区包
    "v2ray.com/core/common/errors" // 引入 V2Ray 错误处理包
    "v2ray.com/core/common/log" // 引入 V2Ray 日志包
    "v2ray.com/core/common/net" // 引入 V2Ray 网络包
    "v2ray.com/core/common/protocol" // 引入 V2Ray 协议包
    udp_proto "v2ray.com/core/common/protocol/udp" // 引入 V2Ray UDP 协议包
    "v2ray.com/core/common/retry" // 引入 V2Ray 重试包
    "v2ray.com/core/common/session" // 引入 V2Ray 会话包
    "v2ray.com/core/common/signal" // 引入 V2Ray 信号包
    "v2ray.com/core/common/task" // 引入 V2Ray 任务包
    "v2ray.com/core/features/policy" // 引入 V2Ray 策略包
    "v2ray.com/core/features/routing" // 引入 V2Ray 路由包
    "v2ray.com/core/transport/internet" // 引入 V2Ray 互联网传输包
    "v2ray.com/core/transport/internet/udp" // 引入 V2Ray UDP 传输包
    "v2ray.com/core/transport/internet/xtls" // 引入 V2Ray XTLS 传输包
)

func init() {
    // 注册服务器配置
    common.Must(common.RegisterConfig((*ServerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) { // nolint: lll
        return NewServer(ctx, config.(*ServerConfig))
    }))
}

// Server is an inbound connection handler that handles messages in trojan protocol.
type Server struct {
    policyManager policy.Manager // 策略管理器
    validator     *Validator // 验证器
    fallbacks     map[string]map[string]*Fallback // or nil
}

// NewServer creates a new trojan inbound handler.
func NewServer(ctx context.Context, config *ServerConfig) (*Server, error) {
    validator := new(Validator) // 创建验证器
    for _, user := range config.Users {
        u, err := user.ToMemoryUser() // 将用户配置转换为内存用户
        if err != nil {
            return nil, newError("failed to get trojan user").Base(err).AtError() // 返回获取用户配置失败的错误
        }

        if err := validator.Add(u); err != nil {
            return nil, newError("failed to add user").Base(err).AtError() // 返回添加用户失败的错误
        }
    }

    v := core.MustFromContext(ctx) // 从上下文中获取核心对象
    server := &Server{
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager), // 获取策略管理器
        validator:     validator, // 设置验证器
    }
    # 如果配置中的 Fallbacks 不为空
    if config.Fallbacks != nil:
        # 初始化 server.fallbacks 为一个空的 map
        server.fallbacks = make(map[string]map[string]*Fallback)
        # 遍历配置中的 Fallbacks
        for _, fb := range config.Fallbacks:
            # 如果 server.fallbacks 中对应的 ALPN 协议不存在，则初始化为一个空的 map
            if server.fallbacks[fb.Alpn] == nil:
                server.fallbacks[fb.Alpn] = make(map[string]*Fallback)
            # 将 Fallback 对象添加到 server.fallbacks 中
            server.fallbacks[fb.Alpn][fb.Path] = fb
        # 如果 server.fallbacks 中包含空字符串的 ALPN 协议
        if server.fallbacks[""] != nil:
            # 遍历 server.fallbacks 中的 ALPN 协议
            for alpn, pfb := range server.fallbacks:
                # 如果 ALPN 协议不为空
                if alpn != "":
                    # 遍历 server.fallbacks 中的路径
                    for path, fb := range server.fallbacks[""]:
                        # 如果当前 ALPN 协议下的路径不存在对应的 Fallback 对象，则添加
                        if pfb[path] == nil:
                            pfb[path] = fb
        # 返回 server 对象和空指针
        return server, nil
// Network 方法实现了 proxy.Inbound.Network() 接口，返回支持的网络类型
func (s *Server) Network() []net.Network {
    return []net.Network{net.Network_TCP}
}

// Process 方法实现了 proxy.Inbound.Process() 接口，处理传入的连接请求
func (s *Server) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error { // nolint: funlen,lll

    // 从上下文中获取会话 ID
    sid := session.ExportIDToError(ctx)

    // 将连接转换为统计连接
    iConn := conn
    if statConn, ok := iConn.(*internet.StatCouterConnection); ok {
        iConn = statConn.Connection
    }

    // 获取会话策略
    sessionPolicy := s.policyManager.ForLevel(0)
    // 设置连接的读取截止时间
    if err := conn.SetReadDeadline(time.Now().Add(sessionPolicy.Timeouts.Handshake)); err != nil {
        return newError("unable to set read deadline").Base(err).AtWarning()
    }

    // 创建一个新的缓冲区
    first := buf.New()
    defer first.Release()

    // 从连接中读取数据到缓冲区
    firstLen, err := first.ReadFrom(conn)
    if err != nil {
        return newError("failed to read first request").Base(err)
    }
    newError("firstLen = ", firstLen).AtInfo().WriteToLog(sid)

    // 创建一个带缓冲的读取器
    bufferedReader := &buf.BufferedReader{
        Reader: buf.NewReader(conn),
        Buffer: buf.MultiBuffer{first},
    }

    var user *protocol.MemoryUser

    // 检查是否存在回退策略
    apfb := s.fallbacks
    isfb := apfb != nil

    // 判断是否需要回退
    shouldFallback := false
    if firstLen < 58 || first.Byte(56) != '\r' { // nolint: gomnd
        // 无效的协议
        err = newError("not trojan protocol")
        log.Record(&log.AccessMessage{
            From:   conn.RemoteAddr(),
            To:     "",
            Status: log.AccessRejected,
            Reason: err,
        })

        shouldFallback = true
    } else {
        // 从验证器获取用户信息，使用前56个字符的十六进制字符串作为参数
        user = s.validator.Get(hexString(first.BytesTo(56))) // nolint: gomnd
        if user == nil {
            // 无效用户，回退处理
            err = newError("not a valid user")
            // 记录访问日志，记录访问被拒绝的情况
            log.Record(&log.AccessMessage{
                From:   conn.RemoteAddr(),
                To:     "",
                Status: log.AccessRejected,
                Reason: err,
            })

            shouldFallback = true
        }
    }

    // 如果需要回退且是fallback模式，则调用fallback函数处理
    if isfb && shouldFallback {
        return s.fallback(ctx, sid, err, sessionPolicy, conn, iConn, apfb, first, firstLen, bufferedReader)
    } else if shouldFallback {
        // 如果需要回退但不是fallback模式，则返回错误信息
        return newError("invalid protocol or invalid user")
    }

    // 创建客户端读取器对象
    clientReader := &ConnReader{Reader: bufferedReader}
    // 解析客户端请求头部
    if err := clientReader.ParseHeader(); err != nil {
        // 记录访问日志，记录访问被拒绝的情况
        log.Record(&log.AccessMessage{
            From:   conn.RemoteAddr(),
            To:     "",
            Status: log.AccessRejected,
            Reason: err,
        })
        // 返回错误信息
        return newError("failed to create request from: ", conn.RemoteAddr()).Base(err)
    }

    // 获取目标地址
    destination := clientReader.Target
    // 取消设置读取截止时间
    if err := conn.SetReadDeadline(time.Time{}); err != nil {
        // 返回错误信息
        return newError("unable to set read deadline").Base(err).AtWarning()
    }

    // 从上下文中获取入站信息
    inbound := session.InboundFromContext(ctx)
    if inbound == nil {
        // 抛出异常
        panic("no inbound metadata")
    }
    // 将用户信息赋值给入站信息
    inbound.User = user
    // 根据用户等级获取会话策略
    sessionPolicy = s.policyManager.ForLevel(user.Level)

    // 如果目标地址的网络类型为UDP，则处理UDP请求
    if destination.Network == net.Network_UDP { // handle udp request
        return s.handleUDPPayload(ctx, &PacketReader{Reader: clientReader}, &PacketWriter{Writer: conn}, dispatcher)
    }

    // 处理TCP请求

    // 在上下文中添加访问消息
    ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
        From:   conn.RemoteAddr(),
        To:     destination,
        Status: log.AccessAccepted,
        Reason: "",
        Email:  user.Email,
    })

    // 记录日志，记录接收到的请求
    newError("received request for ", destination).WriteToLog(sid)
    # 调用 handleConnection 方法处理连接，传入上下文 ctx、会话策略 sessionPolicy、目标地址 destination、客户端读取器 clientReader、连接缓冲区的新写入器和调度器
    return s.handleConnection(ctx, sessionPolicy, destination, clientReader, buf.NewWriter(conn), dispatcher)
}
// 处理 UDP 数据包的函数
func (s *Server) handleUDPPayload(ctx context.Context, clientReader *PacketReader, clientWriter *PacketWriter, dispatcher routing.Dispatcher) error { // nolint: lll
    // 创建一个 UDP 分发器
    udpServer := udp.NewDispatcher(dispatcher, func(ctx context.Context, packet *udp_proto.Packet) {
        // 将数据包的载荷写入到客户端的多缓冲区中
        common.Must(clientWriter.WriteMultiBufferWithMetadata(buf.MultiBuffer{packet.Payload}, packet.Source))
    })

    // 从上下文中获取入站信息
    inbound := session.InboundFromContext(ctx)
    // 获取用户信息
    user := inbound.User

    for {
        select {
        case <-ctx.Done():
            return nil
        default:
            // 从客户端读取多缓冲区数据
            p, err := clientReader.ReadMultiBufferWithMetadata()
            if err != nil {
                if errors.Cause(err) != io.EOF {
                    return newError("unexpected EOF").Base(err)
                }
                return nil
            }

            // 在上下文中添加访问消息
            ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
                From:   inbound.Source,
                To:     p.Target,
                Status: log.AccessAccepted,
                Reason: "",
                Email:  user.Email,
            })
            // 记录日志
            newError("tunnelling request to ", p.Target).WriteToLog(session.ExportIDToError(ctx))

            // 遍历多缓冲区数据，并分发到 UDP 服务器
            for _, b := range p.Buffer {
                udpServer.Dispatch(ctx, p.Target, b)
            }
        }
    }
}

// 处理连接的函数
func (s *Server) handleConnection(ctx context.Context, sessionPolicy policy.Session,
    destination net.Destination,
    clientReader buf.Reader,
    clientWriter buf.Writer, dispatcher routing.Dispatcher) error {
    // 创建一个新的上下文，并返回取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 在空闲时设置定时器取消上下文
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
    // 在上下文中添加缓冲策略
    ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)

    // 分发请求到目标地址，并返回连接
    link, err := dispatcher.Dispatch(ctx, destination)
    if err != nil {
        return newError("failed to dispatch request to ", destination).Base(err)
    }
}
    # 定义一个函数，用于处理请求完成时的操作
    requestDone := func() error {
        # 在函数执行完毕后设置定时器超时时间为 sessionPolicy.Timeouts.DownlinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        # 将客户端读取的数据传输到 link.Writer，更新定时器活动时间
        if err := buf.Copy(clientReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transfer request").Base(err)
        }
        return nil
    }

    # 定义一个函数，用于处理响应完成时的操作
    responseDone := func() error {
        # 在函数执行完毕后设置定时器超时时间为 sessionPolicy.Timeouts.UplinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        # 将 link.Reader 中的数据传输到客户端写入器 clientWriter，更新定时器活动时间
        if err := buf.Copy(link.Reader, clientWriter, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to write response").Base(err)
        }
        return nil
    }

    # 定义一个变量，用于在 requestDone 函数执行成功后执行 task.Close(link.Writer) 操作
    var requestDonePost = task.OnSuccess(requestDone, task.Close(link.Writer))
    # 运行 requestDonePost 和 responseDone 函数，如果出现错误则中断连接并返回错误信息
    if err := task.Run(ctx, requestDonePost, responseDone); err != nil {
        common.Must(common.Interrupt(link.Reader))
        common.Must(common.Interrupt(link.Writer))
        return newError("connection ends").Base(err)
    }

    # 返回空值
    return nil
}
// fallback 方法定义，处理服务器的回退逻辑
func (s *Server) fallback(ctx context.Context, sid errors.ExportOption, err error, sessionPolicy policy.Session, connection internet.Connection, iConn internet.Connection, apfb map[string]map[string]*Fallback, first *buf.Buffer, firstLen int64, reader buf.Reader) error { // nolint: lll
    // 设置连接的读取截止时间为零，表示不设定截止时间
    if err := connection.SetReadDeadline(time.Time{}); err != nil {
        // 记录无法设置读取截止时间的警告日志
        newError("unable to set back read deadline").Base(err).AtWarning().WriteToLog(sid)
    }
    // 记录回退开始的信息日志
    newError("fallback starts").Base(err).AtInfo().WriteToLog(sid)

    // 初始化协议名称变量
    alpn := ""
    // 如果回退配置数量大于1或者默认配置为空
    if len(apfb) > 1 || apfb[""] == nil {
        // 如果当前连接是 TLS 连接
        if tlsConn, ok := iConn.(*tls.Conn); ok {
            // 获取 TLS 连接的协商协议
            alpn = tlsConn.ConnectionState().NegotiatedProtocol
            // 记录实际协商的协议信息日志
            newError("realAlpn = " + alpn).AtInfo().WriteToLog(sid)
        } else if xtlsConn, ok := iConn.(*xtls.Conn); ok {
            // 如果当前连接是 XTLS 连接
            alpn = xtlsConn.ConnectionState().NegotiatedProtocol
            // 记录实际协商的协议信息日志
            newError("realAlpn = " + alpn).AtInfo().WriteToLog(sid)
        }
        // 如果回退配置中不包含当前协商的协议
        if apfb[alpn] == nil {
            // 将协议名称重置为空
            alpn = ""
        }
    }
    // 获取当前协议对应的回退配置
    pfb := apfb[alpn]
    // 如果回退配置为空
    if pfb == nil {
        // 返回找不到默认 "alpn" 配置的错误
        return newError(`failed to find the default "alpn" config`).AtWarning()
    }

    // 初始化路径变量
    path := ""
    // 如果 pfb 的长度大于 1 或者 pfb 中没有空字符串键，则执行以下操作
    if len(pfb) > 1 || pfb[""] == nil {
        // 如果 firstLen 大于等于 18 并且第一个字节不是 '*'，则执行以下操作
        if firstLen >= 18 && first.Byte(4) != '*' { // not h2c
            // 将 first 转换为字节切片
            firstBytes := first.Bytes()
            // 循环遍历第 5 到第 9 个字节
            for i := 4; i <= 8; i++ { // 5 -> 9
                // 如果当前字节是 '/' 并且前一个字节是空格，则执行以下操作
                if firstBytes[i] == '/' && firstBytes[i-1] == ' ' {
                    // 设置搜索范围为 firstBytes 的长度，最大为 64
                    search := len(firstBytes)
                    if search > 64 {
                        search = 64 // up to about 60
                    }
                    // 从当前位置向后搜索，直到遇到 '\r' 或 '\n' 为止
                    for j := i + 1; j < search; j++ {
                        k := firstBytes[j]
                        // 如果当前字节是 '\r' 或 '\n'，则跳出循环
                        if k == '\r' || k == '\n' { // avoid logging \r or \n
                            break
                        }
                        // 如果当前字节是空格，则执行以下操作
                        if k == ' ' {
                            // 将路径设置为从 i 到 j 的子串
                            path = string(firstBytes[i:j])
                            // 记录日志信息
                            newError("realPath = " + path).AtInfo().WriteToLog(sid)
                            // 如果 pfb 中不存在该路径，则将路径设置为空字符串
                            if pfb[path] == nil {
                                path = ""
                            }
                            // 跳出循环
                            break
                        }
                    }
                    // 跳出外层循环
                    break
                }
            }
        }
    }
    // 获取路径对应的配置信息
    fb := pfb[path]
    // 如果配置信息为空，则返回错误信息
    if fb == nil {
        return newError(`failed to find the default "path" config`).AtWarning()
    }

    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(ctx)
    // 设置一个定时器，在连接空闲时取消连接
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
    // 使用策略设置上下文的缓冲策略
    ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)

    var conn net.Conn
    // 使用指数退避算法进行重试，最多重试 5 次，初始间隔为 100 毫秒
    if err := retry.ExponentialBackoff(5, 100).On(func() error {
        var dialer net.Dialer
        // 使用上下文拨号连接到目标地址
        conn, err = dialer.DialContext(ctx, fb.Type, fb.Dest)
        if err != nil {
            return err
        }
        return nil
    }); err != nil {
        // 如果连接失败，则返回错误信息
        return newError("failed to dial to " + fb.Dest).Base(err).AtWarning()
    }
    // 延迟关闭连接
    defer conn.Close()

    // 创建一个从服务器读取数据的缓冲区读取器
    serverReader := buf.NewReader(conn)
    // 创建一个向服务器写入数据的缓冲区写入器
    serverWriter := buf.NewWriter(conn)

    }

    // 创建一个向连接写入数据的缓冲区写入器
    writer := buf.NewWriter(connection)
    # 定义一个名为getResponse的匿名函数，用于处理获取响应的逻辑
    getResponse := func() error {
        # 在函数执行完毕后设置定时器超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)
        # 如果从serverReader读取数据并写入到writer时出现错误
        if err := buf.Copy(serverReader, writer, buf.UpdateActivity(timer)); err != nil {
            # 返回一个包含错误信息的新Error对象
            return newError("failed to deliver response payload").Base(err).AtInfo()
        }
        # 返回nil表示没有错误发生
        return nil
    }

    # 使用task.Run来执行一系列任务，包括postRequest和getResponse，并在成功时关闭serverWriter和writer
    if err := task.Run(ctx, task.OnSuccess(postRequest, task.Close(serverWriter)), task.OnSuccess(getResponse, task.Close(writer))); err != nil {
        # 强制中断serverReader和serverWriter
        common.Must(common.Interrupt(serverReader))
        common.Must(common.Interrupt(serverWriter))
        # 返回一个包含错误信息的新Error对象
        return newError("fallback ends").Base(err).AtInfo()
    }

    # 返回nil表示没有错误发生
    return nil
# 闭合前面的函数定义
```