# `v2ray-core\proxy\vmess\outbound\outbound.go`

```
// +build !confonly

package outbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "time"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/platform"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/retry"
    "v2ray.com/core/common/session"
    "v2ray.com/core/common/signal"
    "v2ray.com/core/common/task"
    "v2ray.com/core/features/policy"
    "v2ray.com/core/proxy/vmess"
    "v2ray.com/core/proxy/vmess/encoding"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/internet"
)

// Handler is an outbound connection handler for VMess protocol.
type Handler struct {
    serverList    *protocol.ServerList  // 服务器列表
    serverPicker  protocol.ServerPicker  // 服务器选择器
    policyManager policy.Manager  // 策略管理器
}

// New creates a new VMess outbound handler.
func New(ctx context.Context, config *Config) (*Handler, error) {
    serverList := protocol.NewServerList()  // 创建新的服务器列表
    for _, rec := range config.Receiver {
        s, err := protocol.NewServerSpecFromPB(rec)  // 从配置中创建新的服务器规范
        if err != nil {
            return nil, newError("failed to parse server spec").Base(err)  // 如果出错，返回错误
        }
        serverList.AddServer(s)  // 将服务器规范添加到服务器列表中
    }

    v := core.MustFromContext(ctx)
    handler := &Handler{
        serverList:    serverList,  // 设置服务器列表
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList),  // 创建新的轮询服务器选择器
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),  // 获取策略管理器
    }

    return handler, nil  // 返回处理器和空错误
}

// Process implements proxy.Outbound.Process().
func (h *Handler) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    var rec *protocol.ServerSpec  // 服务器规范
    var conn internet.Connection  // 网络连接

    err := retry.ExponentialBackoff(5, 200).On(func() error {  // 使用指数退避重试
        rec = h.serverPicker.PickServer()  // 选择服务器
        rawConn, err := dialer.Dial(ctx, rec.Destination())  // 使用拨号器拨号连接到服务器目标地址
        if err != nil {
            return err  // 如果出错，返回错误
        }
        conn = rawConn  // 设置连接为原始连接

        return nil  // 返回空错误
    }) // 如果发生错误，则返回一个包含警告信息的错误
    if err != nil { // 如果错误不为空
        return newError("failed to find an available destination").Base(err).AtWarning() // 返回一个包含警告信息的错误
    }
    defer conn.Close() // 延迟关闭连接，忽略错误检查

    outbound := session.OutboundFromContext(ctx) // 从上下文中获取出站连接
    if outbound == nil || !outbound.Target.IsValid() { // 如果出站连接为空或目标无效
        return newError("target not specified").AtError() // 返回一个包含错误信息的错误
    }

    target := outbound.Target // 获取出站连接的目标
    newError("tunneling request to ", target, " via ", rec.Destination()).WriteToLog(session.ExportIDToError(ctx)) // 记录隧道请求的目标和目的地

    command := protocol.RequestCommandTCP // 默认请求命令为 TCP
    if target.Network == net.Network_UDP { // 如果目标网络是 UDP
        command = protocol.RequestCommandUDP // 请求命令为 UDP
    }
    if target.Address.Family().IsDomain() && target.Address.Domain() == "v1.mux.cool" { // 如果目标地址是域名并且是特定域名
        command = protocol.RequestCommandMux // 请求命令为 Mux
    }

    user := rec.PickUser() // 选择用户
    request := &protocol.RequestHeader{ // 创建请求头
        Version: encoding.Version, // 版本号
        User:    user, // 用户
        Command: command, // 请求命令
        Address: target.Address, // 目标地址
        Port:    target.Port, // 目标端口
        Option:  protocol.RequestOptionChunkStream, // 请求选项为分块流
    }

    account := request.User.Account.(*vmess.MemoryAccount) // 获取用户账户
    request.Security = account.Security // 设置请求安全类型为账户安全类型

    if request.Security == protocol.SecurityType_AES128_GCM || request.Security == protocol.SecurityType_NONE || request.Security == protocol.SecurityType_CHACHA20_POLY1305 { // 如果安全类型是 AES128_GCM、NONE 或 CHACHA20_POLY1305
        request.Option.Set(protocol.RequestOptionChunkMasking) // 设置请求选项为分块掩码
    }

    if shouldEnablePadding(request.Security) && request.Option.Has(protocol.RequestOptionChunkMasking) { // 如果应该启用填充并且请求选项包含分块掩码
        request.Option.Set(protocol.RequestOptionGlobalPadding) // 设置请求选项为全局填充
    }

    input := link.Reader // 输入为连接的读取器
    output := link.Writer // 输出为连接的写入器

    isAEAD := false // 是否使用 AEAD 加密，默认为 false
    if !aead_disabled && len(account.AlterIDs) == 0 { // 如果 AEAD 加密未禁用并且备用 ID 数为 0
        isAEAD = true // 设置为使用 AEAD 加密
    }

    session := encoding.NewClientSession(isAEAD, protocol.DefaultIDHash, ctx) // 创建新的客户端会话
    sessionPolicy := h.policyManager.ForLevel(request.User.Level) // 获取用户级别的策略

    ctx, cancel := context.WithCancel(ctx) // 创建带有取消功能的上下文
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle) // 在空闲时取消信号
    // 定义一个函数，用于处理请求完成时的操作
    requestDone := func() error {
        // 在函数结束时设置定时器超时时间为 sessionPolicy.Timeouts.DownlinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        // 创建一个带缓冲的写入器
        writer := buf.NewBufferedWriter(buf.NewWriter(conn))
        // 编码请求头部，并将结果写入到写入器中
        if err := session.EncodeRequestHeader(request, writer); err != nil {
            return newError("failed to encode request").Base(err).AtWarning()
        }

        // 编码请求体，并将结果写入到写入器中
        bodyWriter := session.EncodeRequestBody(request, writer)
        // 将输入数据拷贝到请求体写入器中，设置超时时间为 100 毫秒
        if err := buf.CopyOnceTimeout(input, bodyWriter, time.Millisecond*100); err != nil && err != buf.ErrNotTimeoutReader && err != buf.ErrReadTimeout {
            return newError("failed to write first payload").Base(err)
        }

        // 关闭写入器的缓冲
        if err := writer.SetBuffered(false); err != nil {
            return err
        }

        // 将输入数据拷贝到请求体写入器中，更新定时器活动时间
        if err := buf.Copy(input, bodyWriter, buf.UpdateActivity(timer)); err != nil {
            return err
        }

        // 如果请求选项包含 protocol.RequestOptionChunkStream，则写入一个空的多缓冲区
        if request.Option.Has(protocol.RequestOptionChunkStream) {
            if err := bodyWriter.WriteMultiBuffer(buf.MultiBuffer{}); err != nil {
                return err
            }
        }

        return nil
    }

    // 定义一个函数，用于处理响应完成时的操作
    responseDone := func() error {
        // 在函数结束时设置定时器超时时间为 sessionPolicy.Timeouts.UplinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        // 创建一个带缓冲的读取器
        reader := &buf.BufferedReader{Reader: buf.NewReader(conn)}
        // 解码响应头部，并返回头部和可能的错误
        header, err := session.DecodeResponseHeader(reader)
        if err != nil {
            return newError("failed to read header").Base(err)
        }
        // 处理命令
        h.handleCommand(rec.Destination(), header.Command)

        // 解码响应体，并返回读取器
        bodyReader := session.DecodeResponseBody(request, reader)

        // 将响应体读取器的内容拷贝到输出数据中，更新定时器活动时间
        return buf.Copy(bodyReader, output, buf.UpdateActivity(timer))
    }

    // 定义一个任务，当响应完成后关闭输出数据
    var responseDonePost = task.OnSuccess(responseDone, task.Close(output))
    // 运行请求完成和响应完成的任务，并返回可能的错误
    if err := task.Run(ctx, requestDone, responseDonePost); err != nil {
        return newError("connection ends").Base(err)
    }

    return nil
# 定义全局变量，用于控制是否启用填充和是否禁用 AEAD
var (
    enablePadding = false
    aead_disabled = false
)

# 根据安全类型判断是否需要启用填充
func shouldEnablePadding(s protocol.SecurityType) bool {
    return enablePadding || s == protocol.SecurityType_AES128_GCM || s == protocol.SecurityType_CHACHA20_POLY1305 || s == protocol.SecurityType_AUTO
}

# 初始化函数，在程序启动时执行
func init() {
    # 注册配置并返回配置对象
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))
    }))

    # 定义默认标志值
    const defaultFlagValue = "NOT_DEFINED_AT_ALL"

    # 从环境变量中获取填充值
    paddingValue := platform.NewEnvFlag("v2ray.vmess.padding").GetValue(func() string { return defaultFlagValue })
    # 如果填充值不等于默认值，则启用填充
    if paddingValue != defaultFlagValue {
        enablePadding = true
    }

    # 从环境变量中获取 AEAD 是否禁用的值
    aeadDisabled := platform.NewEnvFlag("v2ray.vmess.aead.disabled").GetValue(func() string { return defaultFlagValue })
    # 如果 AEAD 被禁用，则设置全局变量为 true
    if aeadDisabled == "true" {
        aead_disabled = true
    }
}
```