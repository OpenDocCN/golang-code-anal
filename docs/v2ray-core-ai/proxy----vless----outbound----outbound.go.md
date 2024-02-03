# `v2ray-core\proxy\vless\outbound\outbound.go`

```go
// +build !confonly

package outbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context" // 导入上下文包，用于处理请求上下文
    "time" // 导入时间包，用于处理时间相关操作

    "v2ray.com/core" // 导入 V2Ray 核心包
    "v2ray.com/core/common" // 导入 V2Ray 公共包
    "v2ray.com/core/common/buf" // 导入 V2Ray 缓冲区包
    "v2ray.com/core/common/net" // 导入 V2Ray 网络包
    "v2ray.com/core/common/platform" // 导入 V2Ray 平台包
    "v2ray.com/core/common/protocol" // 导入 V2Ray 协议包
    "v2ray.com/core/common/retry" // 导入 V2Ray 重试包
    "v2ray.com/core/common/session" // 导入 V2Ray 会话包
    "v2ray.com/core/common/signal" // 导入 V2Ray 信号包
    "v2ray.com/core/common/task" // 导入 V2Ray 任务包
    "v2ray.com/core/features/policy" // 导入 V2Ray 策略包
    "v2ray.com/core/proxy/vless" // 导入 VLess 代理包
    "v2ray.com/core/proxy/vless/encoding" // 导入 VLess 编码包
    "v2ray.com/core/transport" // 导入传输包
    "v2ray.com/core/transport/internet" // 导入互联网传输包
    "v2ray.com/core/transport/internet/xtls" // 导入 XTLS 互联网传输包
)

var (
    xtls_show = false // 初始化 XTLS 显示标志为假
)

func init() {
    // 注册配置并返回新的 VLess 处理程序
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))
    }))

    const defaultFlagValue = "NOT_DEFINED_AT_ALL" // 定义默认标志值为 "NOT_DEFINED_AT_ALL"

    // 获取环境变量中的 xtls.show 标志值
    xtlsShow := platform.NewEnvFlag("v2ray.vless.xtls.show").GetValue(func() string { return defaultFlagValue })
    // 如果 xtlsShow 为 "true"，则将 xtls_show 设置为真
    if xtlsShow == "true" {
        xtls_show = true
    }
}

// Handler is an outbound connection handler for VLess protocol.
type Handler struct {
    serverList    *protocol.ServerList // 服务器列表
    serverPicker  protocol.ServerPicker // 服务器选择器
    policyManager policy.Manager // 策略管理器
}

// New creates a new VLess outbound handler.
func New(ctx context.Context, config *Config) (*Handler, error) {
    // 创建新的服务器列表
    serverList := protocol.NewServerList()
    // 遍历配置中的 Vnext
    for _, rec := range config.Vnext {
        // 从配置中创建新的服务器规范
        s, err := protocol.NewServerSpecFromPB(rec)
        // 如果出错，则返回错误
        if err != nil {
            return nil, newError("failed to parse server spec").Base(err).AtError()
        }
        // 将服务器添加到服务器列表中
        serverList.AddServer(s)
    }

    // 从上下文中获取 V2Ray 实例
    v := core.MustFromContext(ctx)
    // 创建新的 VLess 处理程序
    handler := &Handler{
        serverList:    serverList,
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList),
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),
    }
    # 返回 handler 变量和空值
    return handler, nil
// Process 实现了 proxy.Outbound.Process() 接口
func (h *Handler) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    var rec *protocol.ServerSpec  // 定义一个指向 protocol.ServerSpec 结构体的指针变量 rec
    var conn internet.Connection  // 定义一个 internet.Connection 类型的变量 conn

    // 使用指数退避算法进行重试，最多重试 5 次，初始间隔 200 毫秒
    if err := retry.ExponentialBackoff(5, 200).On(func() error {
        rec = h.serverPicker.PickServer()  // 从 serverPicker 中选择一个服务器
        var err error
        conn, err = dialer.Dial(ctx, rec.Destination())  // 使用 dialer 进行拨号连接
        if err != nil {
            return err  // 如果连接出错，返回错误
        }
        return nil  // 连接成功，返回 nil
    }); err != nil {
        return newError("failed to find an available destination").Base(err).AtWarning()  // 如果重试失败，返回错误信息
    }
    defer conn.Close() // nolint: errcheck  // 延迟关闭连接，忽略错误

    iConn := conn  // 将 conn 赋值给 iConn
    if statConn, ok := iConn.(*internet.StatCouterConnection); ok {  // 判断 iConn 是否为 StatCouterConnection 类型
        iConn = statConn.Connection  // 如果是，则将其赋值给 iConn
    }

    outbound := session.OutboundFromContext(ctx)  // 从上下文中获取出站连接
    if outbound == nil || !outbound.Target.IsValid() {  // 如果出站连接为空或目标无效
        return newError("target not specified").AtError()  // 返回错误信息
    }

    target := outbound.Target  // 获取出站连接的目标地址
    newError("tunneling request to ", target, " via ", rec.Destination()).AtInfo().WriteToLog(session.ExportIDToError(ctx))  // 记录信息到日志

    command := protocol.RequestCommandTCP  // 默认命令为 TCP
    if target.Network == net.Network_UDP {  // 如果目标网络为 UDP
        command = protocol.RequestCommandUDP  // 则命令为 UDP
    }
    if target.Address.Family().IsDomain() && target.Address.Domain() == "v1.mux.cool" {  // 如果目标地址为域名且为 "v1.mux.cool"
        command = protocol.RequestCommandMux  // 则命令为 Mux
    }

    request := &protocol.RequestHeader{  // 创建一个 protocol.RequestHeader 结构体
        Version: encoding.Version,  // 版本号
        User:    rec.PickUser(),  // 用户信息
        Command: command,  // 命令
        Address: target.Address,  // 目标地址
        Port:    target.Port,  // 目标端口
    }

    account := request.User.Account.(*vless.MemoryAccount)  // 获取用户账户信息

    requestAddons := &encoding.Addons{  // 创建一个 encoding.Addons 结构体
        Flow: account.Flow,  // 流量信息
    }

    allowUDP443 := false  // 默认不允许 UDP 443
    switch requestAddons.Flow {  // 根据流量信息进行判断
    case vless.XRO + "-udp443", vless.XRD + "-udp443":  // 如果流量信息为 XRO-udp443 或 XRD-udp443
        allowUDP443 = true  // 则允许 UDP 443
        requestAddons.Flow = requestAddons.Flow[:16]  // 截取流量信息的前 16 个字符
        fallthrough  // 继续执行下面的 case
    // 其他 case 省略
    }
}
    # 根据 vless.XRO 和 vless.XRD 的值进行不同的处理
    case vless.XRO, vless.XRD:
        # 根据请求的命令进行不同的处理
        switch request.Command:
        # 如果请求命令是 protocol.RequestCommandMux
        case protocol.RequestCommandMux:
            # 返回一个警告错误，提示不支持 Mux
            return newError(requestAddons.Flow + " doesn't support Mux").AtWarning()
        # 如果请求命令是 protocol.RequestCommandUDP
        case protocol.RequestCommandUDP:
            # 如果不允许 UDP 443 并且请求的端口是 443
            if !allowUDP443 && request.Port == 443:
                # 返回一个信息错误，提示已停止 UDP/443
                return newError(requestAddons.Flow + " stopped UDP/443").AtInfo()
            # 清空请求附加信息的 Flow 字段
            requestAddons.Flow = ""
        # 如果请求命令是 protocol.RequestCommandTCP
        case protocol.RequestCommandTCP:
            # 如果 iConn 是 xtls.Conn 类型
            if xtlsConn, ok := iConn.(*xtls.Conn); ok:
                # 设置 xtlsConn 的 RPRX、SHOW 和 MARK 字段
                xtlsConn.RPRX = true
                xtlsConn.SHOW = xtls_show
                xtlsConn.MARK = "XTLS"
                # 如果请求附加信息的 Flow 字段是 vless.XRD
                if requestAddons.Flow == vless.XRD:
                    # 设置 xtlsConn 的 DirectMode 字段为 true
                    xtlsConn.DirectMode = true
            else:
                # 返回一个警告错误，提示使用 requestAddons.Flow 失败，可能是 "security" 不是 "xtls"
                return newError(`failed to use ` + requestAddons.Flow + `, maybe "security" is not "xtls"`).AtWarning()
        # 如果请求命令不是 vless.XRO 或 vless.XRD
    default:
        # 如果 iConn 是 xtls.Conn 类型
        if _, ok := iConn.(*xtls.Conn); ok:
            # 抛出一个 panic，提示在使用 XTLS 时必须填写 VLESS 的 "flow" 字段
            panic(`To avoid misunderstanding, you must fill in VLESS "flow" when using XTLS.`)
    }

    # 根据请求用户的级别获取会话策略
    sessionPolicy := h.policyManager.ForLevel(request.User.Level)
    # 创建一个可取消的上下文
    ctx, cancel := context.WithCancel(ctx)
    # 根据会话策略的连接空闲超时时间创建一个定时器
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)

    # 将 link.Reader 和 link.Writer 赋值给 clientReader 和 clientWriter
    clientReader := link.Reader // .(*pipe.Reader)
    clientWriter := link.Writer // .(*pipe.Writer)
    // 定义一个名为 postRequest 的匿名函数，用于发送 POST 请求
    postRequest := func() error {
        // 在函数结束时设置定时器超时时间为 sessionPolicy.Timeouts.DownlinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        // 创建一个带缓冲的写入器，用于向连接中写入数据
        bufferWriter := buf.NewBufferedWriter(buf.NewWriter(conn))
        // 将请求头编码并写入 bufferWriter 中
        if err := encoding.EncodeRequestHeader(bufferWriter, request, requestAddons); err != nil {
            return newError("failed to encode request header").Base(err).AtWarning()
        }

        // 默认情况下，serverWriter 等于 bufferWriter，用于向服务器写入请求体
        serverWriter := encoding.EncodeBodyAddons(bufferWriter, request, requestAddons)
        // 从 clientReader 中读取数据并写入 serverWriter，设置超时时间为 100 毫秒
        if err := buf.CopyOnceTimeout(clientReader, serverWriter, time.Millisecond*100); err != nil && err != buf.ErrNotTimeoutReader && err != buf.ErrReadTimeout {
            return err // ...
        }

        // 刷新缓冲区，使 bufferWriter.WriteMultiBufer 立即写入数据
        if err := bufferWriter.SetBuffered(false); err != nil {
            return newError("failed to write A request payload").Base(err).AtWarning()
        }

        // 从 clientReader 中读取数据并写入 serverWriter，更新定时器的活动时间
        if err := buf.Copy(clientReader, serverWriter, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transfer request payload").Base(err).AtInfo()
        }

        // 表示请求体结束
        switch requestAddons.Flow {
        default:

        }

        return nil
    }
    // 定义名为getResponse的匿名函数，用于处理获取响应的逻辑
    getResponse := func() error {
        // 在函数执行完毕后设置超时时间
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        // 解码响应头部，并返回响应附加信息和可能的错误
        responseAddons, err := encoding.DecodeResponseHeader(conn, request)
        if err != nil {
            // 如果解码失败，返回一个包含错误信息的新错误
            return newError("failed to decode response header").Base(err).AtWarning()
        }

        // 根据响应附加信息解码响应体
        serverReader := encoding.DecodeBodyAddons(conn, request, responseAddons)

        // 从serverReader读取数据并写入clientWriter，同时更新定时器的活动时间
        if err := buf.Copy(serverReader, clientWriter, buf.UpdateActivity(timer)); err != nil {
            // 如果传输响应负载失败，返回一个包含错误信息的新错误
            return newError("failed to transfer response payload").Base(err).AtInfo()
        }

        // 返回nil表示获取响应成功
        return nil
    }

    // 运行任务，如果postRequest成功则执行getResponse和关闭clientWriter，否则返回错误信息
    if err := task.Run(ctx, postRequest, task.OnSuccess(getResponse, task.Close(clientWriter))); err != nil {
        // 如果任务运行失败，返回一个包含错误信息的新错误
        return newError("connection ends").Base(err).AtInfo()
    }

    // 返回nil表示整个函数执行成功
    return nil
# 闭合前面的函数定义
```