# `v2ray-core\proxy\socks\server.go`

```
// +build !confonly

package socks

import (
    "context"  // 导入上下文包
    "io"  // 导入输入输出包
    "time"  // 导入时间包

    "v2ray.com/core"  // 导入 V2Ray 核心包
    "v2ray.com/core/common"  // 导入 V2Ray 核心通用包
    "v2ray.com/core/common/buf"  // 导入 V2Ray 核心通用缓冲包
    "v2ray.com/core/common/log"  // 导入 V2Ray 核心通用日志包
    "v2ray.com/core/common/net"  // 导入 V2Ray 核心通用网络包
    "v2ray.com/core/common/protocol"  // 导入 V2Ray 核心通用协议包
    udp_proto "v2ray.com/core/common/protocol/udp"  // 导入 V2Ray 核心通用 UDP 协议包
    "v2ray.com/core/common/session"  // 导入 V2Ray 核心通用会话包
    "v2ray.com/core/common/signal"  // 导入 V2Ray 核心通用信号包
    "v2ray.com/core/common/task"  // 导入 V2Ray 核心通用任务包
    "v2ray.com/core/features"  // 导入 V2Ray 核心特性包
    "v2ray.com/core/features/policy"  // 导入 V2Ray 核心特性策略包
    "v2ray.com/core/features/routing"  // 导入 V2Ray 核心特性路由包
    "v2ray.com/core/transport/internet"  // 导入 V2Ray 核心传输互联网包
    "v2ray.com/core/transport/internet/udp"  // 导入 V2Ray 核心传输互联网 UDP 包
)

// Server is a SOCKS 5 proxy server
type Server struct {
    config        *ServerConfig  // 服务器配置
    policyManager policy.Manager  // 策略管理器
}

// NewServer creates a new Server object.
func NewServer(ctx context.Context, config *ServerConfig) (*Server, error) {
    v := core.MustFromContext(ctx)  // 从上下文中获取核心对象
    s := &Server{
        config:        config,  // 设置服务器配置
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),  // 获取并设置策略管理器
    }
    return s, nil  // 返回服务器对象和空错误
}

func (s *Server) policy() policy.Session {
    config := s.config  // 获取服务器配置
    p := s.policyManager.ForLevel(config.UserLevel)  // 根据用户级别获取策略
    if config.Timeout > 0 {  // 如果超时大于0
        features.PrintDeprecatedFeatureWarning("Socks timeout")  // 打印警告信息
    }
    if config.Timeout > 0 && config.UserLevel == 0 {  // 如果超时大于0且用户级别为0
        p.Timeouts.ConnectionIdle = time.Duration(config.Timeout) * time.Second  // 设置连接空闲超时时间
    }
    return p  // 返回策略
}

// Network implements proxy.Inbound.
func (s *Server) Network() []net.Network {
    list := []net.Network{net.Network_TCP}  // 创建网络列表，包含 TCP
    if s.config.UdpEnabled {  // 如果 UDP 启用
        list = append(list, net.Network_UDP)  // 添加 UDP 到网络列表
    }
    return list  // 返回网络列表
}

// Process implements proxy.Inbound.
func (s *Server) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error {
    if inbound := session.InboundFromContext(ctx); inbound != nil {  // 如果存在入站会话
        inbound.User = &protocol.MemoryUser{  // 设置入站用户
            Level: s.config.UserLevel,  // 用户级别为服务器配置的用户级别
        }
    }

    switch network {  // 根据网络类型进行处理
    # 如果网络类型是TCP，则调用processTCP方法处理连接
    case net.Network_TCP:
        return s.processTCP(ctx, conn, dispatcher)
    # 如果网络类型是UDP，则调用handleUDPPayload方法处理连接
    case net.Network_UDP:
        return s.handleUDPPayload(ctx, conn, dispatcher)
    # 如果网络类型未知，则返回一个新的错误信息
    default:
        return newError("unknown network: ", network)
    }
}
// 处理 TCP 连接的函数
func (s *Server) processTCP(ctx context.Context, conn internet.Connection, dispatcher routing.Dispatcher) error {
    // 获取服务器的策略
    plcy := s.policy()
    // 设置连接的读取截止时间
    if err := conn.SetReadDeadline(time.Now().Add(plcy.Timeouts.Handshake)); err != nil {
        newError("failed to set deadline").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }

    // 从上下文中获取入站连接信息
    inbound := session.InboundFromContext(ctx)
    // 如果入站连接为空或者网关无效，则返回错误
    if inbound == nil || !inbound.Gateway.IsValid() {
        return newError("inbound gateway not specified")
    }

    // 创建服务器会话对象
    svrSession := &ServerSession{
        config: s.config,
        port:   inbound.Gateway.Port,
    }

    // 创建读取器对象
    reader := &buf.BufferedReader{Reader: buf.NewReader(conn)}
    // 进行握手，获取请求信息
    request, err := svrSession.Handshake(reader, conn)
    // 如果出现错误，则记录日志并返回错误
    if err != nil {
        if inbound != nil && inbound.Source.IsValid() {
            log.Record(&log.AccessMessage{
                From:   inbound.Source,
                To:     "",
                Status: log.AccessRejected,
                Reason: err,
            })
        }
        return newError("failed to read request").Base(err)
    }
    // 如果请求中包含用户信息，则更新入站用户信息
    if request.User != nil {
        inbound.User.Email = request.User.Email
    }

    // 清除连接的读取截止时间
    if err := conn.SetReadDeadline(time.Time{}); err != nil {
        newError("failed to clear deadline").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }

    // 如果请求命令为 TCP 连接，则记录日志并调用传输函数
    if request.Command == protocol.RequestCommandTCP {
        dest := request.Destination()
        newError("TCP Connect request to ", dest).WriteToLog(session.ExportIDToError(ctx))
        if inbound != nil && inbound.Source.IsValid() {
            ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
                From:   inbound.Source,
                To:     dest,
                Status: log.AccessAccepted,
                Reason: "",
            })
        }
        return s.transport(ctx, reader, conn, dest, dispatcher)
    }

    // 如果请求命令为 UDP 连接，则处理 UDP 连接
    if request.Command == protocol.RequestCommandUDP {
        return s.handleUDP(conn)
    }

    // 其他情况返回空
    return nil
}
# 处理 UDP 数据包的方法，参数为一个实现了 io.Reader 接口的对象
func (*Server) handleUDP(c io.Reader) error {
    # TCP 连接在该方法返回后关闭。需要等待客户端关闭连接。
    return common.Error2(io.Copy(buf.DiscardBytes, c))
}

# 传输数据的方法，参数包括上下文、读取器、写入器、目标地址和路由分发器
func (s *Server) transport(ctx context.Context, reader io.Reader, writer io.Writer, dest net.Destination, dispatcher routing.Dispatcher) error {
    # 创建一个新的上下文，用于取消操作
    ctx, cancel := context.WithCancel(ctx)
    # 设置一个定时器，在连接空闲时取消操作
    timer := signal.CancelAfterInactivity(ctx, cancel, s.policy().Timeouts.ConnectionIdle)

    # 获取服务器的策略
    plcy := s.policy()
    # 在上下文中设置缓冲策略
    ctx = policy.ContextWithBufferPolicy(ctx, plcy.Buffer)
    # 使用分发器将数据发送到目标地址，并返回连接
    link, err := dispatcher.Dispatch(ctx, dest)
    if err != nil {
        return err
    }

    # 处理请求完成的函数
    requestDone := func() error {
        # 在规定时间内设置定时器
        defer timer.SetTimeout(plcy.Timeouts.DownlinkOnly)
        # 从读取器中读取数据，并写入连接的写入器
        if err := buf.Copy(buf.NewReader(reader), link.Writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transport all TCP request").Base(err)
        }

        return nil
    }

    # 处理响应完成的函数
    responseDone := func() error {
        # 在规定时间内设置定时器
        defer timer.SetTimeout(plcy.Timeouts.UplinkOnly)
        # 创建一个新的写入器，并从连接的读取器中读取数据并写入
        v2writer := buf.NewWriter(writer)
        if err := buf.Copy(link.Reader, v2writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transport all TCP response").Base(err)
        }

        return nil
    }

    # 处理请求完成后的操作
    var requestDonePost = task.OnSuccess(requestDone, task.Close(link.Writer))
    # 运行请求完成后的操作和响应完成的函数
    if err := task.Run(ctx, requestDonePost, responseDone); err != nil {
        # 中断连接的读取器和写入器
        common.Interrupt(link.Reader)
        common.Interrupt(link.Writer)
        return newError("connection ends").Base(err)
    }

    return nil
}

# 处理 UDP 数据包的负载
func (s *Server) handleUDPPayload(ctx context.Context, conn internet.Connection, dispatcher routing.Dispatcher) error {
    # 创建一个 UDP 服务器，使用给定的调度器和处理函数
    udpServer := udp.NewDispatcher(dispatcher, func(ctx context.Context, packet *udp_proto.Packet) {
        # 获取 UDP 数据包的载荷
        payload := packet.Payload
        # 记录 UDP 响应的字节长度
        newError("writing back UDP response with ", payload.Len(), " bytes").AtDebug().WriteToLog(session.ExportIDToError(ctx))

        # 从上下文中获取请求头
        request := protocol.RequestHeaderFromContext(ctx)
        # 如果请求头为空，则返回
        if request == nil {
            return
        }
        # 编码 UDP 数据包
        udpMessage, err := EncodeUDPPacket(request, payload.Bytes())
        # 释放载荷
        payload.Release()

        # 延迟释放 UDP 消息
        defer udpMessage.Release()
        # 如果出现错误，则记录警告日志
        if err != nil {
            newError("failed to write UDP response").AtWarning().Base(err).WriteToLog(session.ExportIDToError(ctx))
        }

        # 写入 UDP 消息到连接中，忽略错误检查
        conn.Write(udpMessage.Bytes()) // nolint: errcheck
    })

    # 从上下文中获取入站连接，并记录客户端 UDP 连接信息
    if inbound := session.InboundFromContext(ctx); inbound != nil && inbound.Source.IsValid() {
        newError("client UDP connection from ", inbound.Source).WriteToLog(session.ExportIDToError(ctx))
    }

    # 创建一个连接的数据包读取器
    reader := buf.NewPacketReader(conn)
    // 无限循环，读取多个缓冲区的数据
    for {
        // 从读取器中读取多个缓冲区的数据
        mpayload, err := reader.ReadMultiBuffer()
        // 如果读取出错，则返回错误
        if err != nil {
            return err
        }

        // 遍历多个缓冲区的数据
        for _, payload := range mpayload {
            // 解码 UDP 数据包
            request, err := DecodeUDPPacket(payload)

            // 如果解码出错，则记录错误信息并释放缓冲区，然后继续下一次循环
            if err != nil {
                newError("failed to parse UDP request").Base(err).WriteToLog(session.ExportIDToError(ctx))
                payload.Release()
                continue
            }

            // 如果数据包为空，则释放缓冲区，然后继续下一次循环
            if payload.IsEmpty() {
                payload.Release()
                continue
            }
            // 复制当前上下文
            currentPacketCtx := ctx
            // 记录调试信息，包括目的地和数据长度
            newError("send packet to ", request.Destination(), " with ", payload.Len(), " bytes").AtDebug().WriteToLog(session.ExportIDToError(ctx))
            // 如果会话上下文中存在入站连接，并且来源有效，则创建新的上下文
            if inbound := session.InboundFromContext(ctx); inbound != nil && inbound.Source.IsValid() {
                currentPacketCtx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
                    From:   inbound.Source,
                    To:     request.Destination(),
                    Status: log.AccessAccepted,
                    Reason: "",
                })
            }

            // 将请求头信息添加到当前上下文中
            currentPacketCtx = protocol.ContextWithRequestHeader(currentPacketCtx, request)
            // 调度 UDP 数据包
            udpServer.Dispatch(currentPacketCtx, request.Destination(), payload)
        }
    }
# 初始化函数，用于注册服务器配置并创建服务器实例
func init() {
    # 注册服务器配置，并在注册后立即创建服务器实例
    common.Must(common.RegisterConfig((*ServerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewServer(ctx, config.(*ServerConfig))
    }))
}
```