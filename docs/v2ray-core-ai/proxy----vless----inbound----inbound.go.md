# `v2ray-core\proxy\vless\inbound\inbound.go`

```go
// +build !confonly

package inbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context" // 导入上下文包，用于处理请求上下文
    "io" // 导入io包，用于实现输入输出操作
    "strconv" // 导入strconv包，用于字符串和基本数据类型之间的转换
    "time" // 导入time包，用于处理时间相关操作

    "v2ray.com/core" // 导入v2ray核心包
    "v2ray.com/core/common" // 导入v2ray通用包
    "v2ray.com/core/common/buf" // 导入v2ray缓冲区包
    "v2ray.com/core/common/errors" // 导入v2ray错误处理包
    "v2ray.com/core/common/log" // 导入v2ray日志包
    "v2ray.com/core/common/net" // 导入v2ray网络包
    "v2ray.com/core/common/platform" // 导入v2ray平台包
    "v2ray.com/core/common/protocol" // 导入v2ray协议包
    "v2ray.com/core/common/retry" // 导入v2ray重试包
    "v2ray.com/core/common/session" // 导入v2ray会话包
    "v2ray.com/core/common/signal" // 导入v2ray信号包
    "v2ray.com/core/common/task" // 导入v2ray任务包
    "v2ray.com/core/features/dns" // 导入v2ray DNS包
    feature_inbound "v2ray.com/core/features/inbound" // 导入v2ray入站特性包
    "v2ray.com/core/features/policy" // 导入v2ray策略包
    "v2ray.com/core/features/routing" // 导入v2ray路由包
    "v2ray.com/core/proxy/vless" // 导入v2ray vless代理包
    "v2ray.com/core/proxy/vless/encoding" // 导入v2ray vless编码包
    "v2ray.com/core/transport/internet" // 导入v2ray网络传输包
    "v2ray.com/core/transport/internet/tls" // 导入v2ray TLS包
    "v2ray.com/core/transport/internet/xtls" // 导入v2ray XTLS包
)

var (
    xtls_show = false // 定义并初始化XTLS显示标志
)

func init() {
    // 注册配置并返回新的配置对象
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        var dc dns.Client // 定义DNS客户端
        if err := core.RequireFeatures(ctx, func(d dns.Client) error {
            dc = d
            return nil
        }); err != nil {
            return nil, err
        }
        return New(ctx, config.(*Config), dc) // 返回新的处理程序
    }))

    const defaultFlagValue = "NOT_DEFINED_AT_ALL" // 定义默认标志值

    xtlsShow := platform.NewEnvFlag("v2ray.vless.xtls.show").GetValue(func() string { return defaultFlagValue }) // 获取XTLS显示标志的值
    if xtlsShow == "true" {
        xtls_show = true // 如果XTLS显示标志为true，则设置XTLS显示标志为true
    }
}

// Handler is an inbound connection handler that handles messages in VLess protocol.
type Handler struct {
    inboundHandlerManager feature_inbound.Manager // 入站处理程序管理器
    policyManager         policy.Manager // 策略管理器
    validator             *vless.Validator // VLess验证器
    dns                   dns.Client // DNS客户端
    fallbacks             map[string]map[string]*Fallback // 或nil的回退映射
    //regexps               map[string]*regexp.Regexp       // 或nil的正则表达式映射
}
// New creates a new VLess inbound handler.
func New(ctx context.Context, config *Config, dc dns.Client) (*Handler, error) {

    v := core.MustFromContext(ctx)  // 从上下文中获取核心对象
    handler := &Handler{  // 创建一个新的处理程序对象
        inboundHandlerManager: v.GetFeature(feature_inbound.ManagerType()).(feature_inbound.Manager),  // 从核心对象中获取入站处理程序管理器
        policyManager:         v.GetFeature(policy.ManagerType()).(policy.Manager),  // 从核心对象中获取策略管理器
        validator:             new(vless.Validator),  // 创建一个新的 VLESS 验证器
        dns:                   dc,  // 设置 DNS 客户端
    }

    for _, user := range config.Clients {  // 遍历配置中的客户端
        u, err := user.ToMemoryUser()  // 将用户转换为内存用户
        if err != nil {
            return nil, newError("failed to get VLESS user").Base(err).AtError()  // 如果转换失败，则返回错误
        }
        if err := handler.AddUser(ctx, u); err != nil {  // 将用户添加到处理程序中
            return nil, newError("failed to initiate user").Base(err).AtError()  // 如果添加失败，则返回错误
        }
    }

    if config.Fallbacks != nil {  // 如果存在回退配置
        handler.fallbacks = make(map[string]map[string]*Fallback)  // 初始化回退映射
        for _, fb := range config.Fallbacks {  // 遍历回退配置
            if handler.fallbacks[fb.Alpn] == nil {  // 如果回退映射中不存在当前协议
                handler.fallbacks[fb.Alpn] = make(map[string]*Fallback)  // 初始化当前协议的回退映射
            }
            handler.fallbacks[fb.Alpn][fb.Path] = fb  // 将回退配置添加到回退映射中
            /*
                if fb.Path != "" {
                    if r, err := regexp.Compile(fb.Path); err != nil {
                        return nil, newError("invalid path regexp").Base(err).AtError()
                    } else {
                        handler.regexps[fb.Path] = r
                    }
                }
            */
        }
        if handler.fallbacks[""] != nil {  // 如果存在默认回退
            for alpn, pfb := range handler.fallbacks {  // 遍历回退映射
                if alpn != "" { // && alpn != "h2" {
                    for path, fb := range handler.fallbacks[""] {  // 遍历默认回退
                        if pfb[path] == nil {  // 如果当前协议的回退映射中不存在当前路径的回退
                            pfb[path] = fb  // 将默认回退添加到当前协议的回退映射中
                        }
                    }
                }
            }
        }
    }

    return handler, nil  // 返回处理程序对象和空错误
}
// Close 方法实现了 common.Closable.Close() 接口
func (h *Handler) Close() error {
    // 调用 common.Close 方法关闭验证器
    return errors.Combine(common.Close(h.validator))
}

// AddUser 方法实现了 proxy.UserManager.AddUser() 接口
func (h *Handler) AddUser(ctx context.Context, u *protocol.MemoryUser) error {
    // 调用验证器的 Add 方法添加用户
    return h.validator.Add(u)
}

// RemoveUser 方法实现了 proxy.UserManager.RemoveUser() 接口
func (h *Handler) RemoveUser(ctx context.Context, e string) error {
    // 调用验证器的 Del 方法移除用户
    return h.validator.Del(e)
}

// Network 方法实现了 proxy.Inbound.Network() 接口
func (*Handler) Network() []net.Network {
    // 返回一个包含 TCP 网络的切片
    return []net.Network{net.Network_TCP}
}

// Process 方法实现了 proxy.Inbound.Process() 接口
func (h *Handler) Process(ctx context.Context, network net.Network, connection internet.Connection, dispatcher routing.Dispatcher) error {
    // 导出会话 ID 到错误日志
    sid := session.ExportIDToError(ctx)

    // 获取连接的统计信息
    iConn := connection
    if statConn, ok := iConn.(*internet.StatCouterConnection); ok {
        iConn = statConn.Connection
    }

    // 获取会话策略
    sessionPolicy := h.policyManager.ForLevel(0)
    // 设置读取截止时间
    if err := connection.SetReadDeadline(time.Now().Add(sessionPolicy.Timeouts.Handshake)); err != nil {
        return newError("unable to set read deadline").Base(err).AtWarning()
    }

    // 创建一个缓冲区用于读取数据
    first := buf.New()
    defer first.Release()

    // 从连接中读取数据到缓冲区
    firstLen, _ := first.ReadFrom(connection)
    // 将读取的数据长度写入日志
    newError("firstLen = ", firstLen).AtInfo().WriteToLog(sid)

    // 创建一个带缓冲区的读取器
    reader := &buf.BufferedReader{
        Reader: buf.NewReader(connection),
        Buffer: buf.MultiBuffer{first},
    }

    var request *protocol.RequestHeader
    var requestAddons *encoding.Addons
    var err error

    // 判断是否存在回退策略
    apfb := h.fallbacks
    isfb := apfb != nil

    // 如果存在回退策略且读取的数据长度小于 18，则直接回退
    if isfb && firstLen < 18 {
        err = newError("fallback directly")
    } else {
        // 解码请求头
        request, requestAddons, err, isfb = encoding.DecodeRequestHeader(isfb, first, reader, h.validator)
    }

    // 恢复读取截止时间
    if err := connection.SetReadDeadline(time.Time{}); err != nil {
        newError("unable to set back read deadline").Base(err).AtWarning().WriteToLog(sid)
    }
}
    # 创建一个新的错误消息，包含请求的目的地信息，并将其记录到日志中
    newError("received request for ", request.Destination()).AtInfo().WriteToLog(sid)

    # 从上下文中获取入站连接
    inbound := session.InboundFromContext(ctx)
    # 如果入站连接为空，则触发 panic
    if inbound == nil:
        panic("no inbound metadata")

    # 将请求中的用户信息赋值给入站连接的用户信息
    inbound.User = request.User

    # 将请求中的用户账户信息转换为 vless.MemoryAccount 类型
    account := request.User.Account.(*vless.MemoryAccount)

    # 创建一个编码附加信息的结构体
    responseAddons := &encoding.Addons{
        //Flow: requestAddons.Flow,
    }

    # 根据请求附加信息中的流类型进行不同的处理
    switch requestAddons.Flow {
    case vless.XRO, vless.XRD:
        # 如果账户的流类型与请求的流类型相同，则根据请求的命令类型进行不同的处理
        if account.Flow == requestAddons.Flow {
            switch request.Command {
            case protocol.RequestCommandMux:
                # 如果请求的命令类型为 Mux，则返回一个新的警告错误
                return newError(requestAddons.Flow + " doesn't support Mux").AtWarning()
            case protocol.RequestCommandUDP:
                # 如果请求的命令类型为 UDP，则返回一个新的警告错误
                return newError(requestAddons.Flow + " doesn't support UDP").AtWarning()
            case protocol.RequestCommandTCP:
                # 如果请求的命令类型为 TCP，则根据连接类型进行不同的处理
                if xtlsConn, ok := iConn.(*xtls.Conn); ok {
                    xtlsConn.RPRX = true
                    xtlsConn.SHOW = xtls_show
                    xtlsConn.MARK = "XTLS"
                    # 如果请求的流类型为 XRD，则设置连接为直连模式
                    if requestAddons.Flow == vless.XRD {
                        xtlsConn.DirectMode = true
                    }
                } else {
                    # 如果无法使用请求的流类型，则返回一个新的警告错误
                    return newError(`failed to use ` + requestAddons.Flow + `, maybe "security" is not "xtls"`).AtWarning()
                }
            }
        } else {
            # 如果账户的流类型与请求的流类型不同，则返回一个新的警告错误
            return newError(account.ID.String() + " is not able to use " + requestAddons.Flow).AtWarning()
        }
    case "":
    default:
        # 如果请求的流类型为空或未知，则返回一个新的警告错误
        return newError("unknown request flow " + requestAddons.Flow).AtWarning()
    }

    # 如果请求的命令类型不为 Mux，则将访问消息上下文中的信息进行更新
    if request.Command != protocol.RequestCommandMux:
        ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
            From:   connection.RemoteAddr(),
            To:     request.Destination(),
            Status: log.AccessAccepted,
            Reason: "",
            Email:  request.User.Email,
        })

    # 根据用户级别获取会话策略
    sessionPolicy = h.policyManager.ForLevel(request.User.Level)
    # 使用传入的上下文创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 根据上下文创建一个定时器，用于在连接空闲时取消请求
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
    # 使用策略创建一个带有缓冲区的上下文
    ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)

    # 使用调度器将请求分发到目标地址，并返回连接
    link, err := dispatcher.Dispatch(ctx, request.Destination())
    if err != nil:
        # 如果分发请求失败，则返回一个包含错误信息的新错误
        return newError("failed to dispatch request to ", request.Destination()).Base(err).AtWarning()

    # 从连接中获取服务器端的读取器和写入器
    serverReader := link.Reader // .(*pipe.Reader)
    serverWriter := link.Writer // .(*pipe.Writer)

    # 定义一个函数用于发送请求
    postRequest := func() error:
        # 在函数结束时设置定时器的超时时间为下行数据的超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        # 默认情况下，客户端读取器使用传入的读取器
        clientReader := encoding.DecodeBodyAddons(reader, request, requestAddons)

        # 将客户端读取器的数据传输到服务器端写入器
        if err := buf.Copy(clientReader, serverWriter, buf.UpdateActivity(timer)); err != nil:
            # 如果传输失败，则返回一个包含错误信息的新错误
            return newError("failed to transfer request payload").Base(err).AtInfo()

        return nil
    // 定义名为getResponse的匿名函数，用于处理获取响应的逻辑
    getResponse := func() error {
        // 在函数执行完毕后设置超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        // 创建一个带缓冲的写入器
        bufferWriter := buf.NewBufferedWriter(buf.NewWriter(connection))
        // 编码响应头部，并将结果写入bufferWriter
        if err := encoding.EncodeResponseHeader(bufferWriter, request, responseAddons); err != nil {
            return newError("failed to encode response header").Base(err).AtWarning()
        }

        // 默认情况下，clientWriter等于bufferWriter
        clientWriter := encoding.EncodeBodyAddons(bufferWriter, request, responseAddons)
        {
            // 从serverReader中读取多个缓冲区
            multiBuffer, err := serverReader.ReadMultiBuffer()
            if err != nil {
                return err // ...
            }
            // 将读取的多个缓冲区写入clientWriter
            if err := clientWriter.WriteMultiBuffer(multiBuffer); err != nil {
                return err // ...
            }
        }

        // 刷新缓冲区；bufferWriter.WriteMultiBufer现在是bufferWriter.writer.WriteMultiBuffer
        if err := bufferWriter.SetBuffered(false); err != nil {
            return newError("failed to write A response payload").Base(err).AtWarning()
        }

        // 从serverReader读取多个缓冲区，然后写入clientWriter
        if err := buf.Copy(serverReader, clientWriter, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transfer response payload").Base(err).AtInfo()
        }

        // 表示响应负载结束
        switch responseAddons.Flow {
        default:

        }

        return nil
    }

    // 运行任务，如果出现错误则中断serverReader和serverWriter，并返回错误信息
    if err := task.Run(ctx, task.OnSuccess(postRequest, task.Close(serverWriter)), getResponse); err != nil {
        common.Interrupt(serverReader)
        common.Interrupt(serverWriter)
        return newError("connection ends").Base(err).AtInfo()
    }

    return nil
# 闭合前面的函数定义
```