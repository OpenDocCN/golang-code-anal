# `v2ray-core\proxy\shadowsocks\server.go`

```
// +build !confonly

package shadowsocks

import (
    "context"  // 导入上下文包，用于处理请求上下文
    "time"  // 导入时间包，用于处理时间相关操作

    "v2ray.com/core"  // 导入v2ray核心包
    "v2ray.com/core/common"  // 导入v2ray通用包
    "v2ray.com/core/common/buf"  // 导入v2ray缓冲区包
    "v2ray.com/core/common/log"  // 导入v2ray日志包
    "v2ray.com/core/common/net"  // 导入v2ray网络包
    "v2ray.com/core/common/protocol"  // 导入v2ray协议包
    udp_proto "v2ray.com/core/common/protocol/udp"  // 导入v2ray UDP协议包
    "v2ray.com/core/common/session"  // 导入v2ray会话包
    "v2ray.com/core/common/signal"  // 导入v2ray信号包
    "v2ray.com/core/common/task"  // 导入v2ray任务包
    "v2ray.com/core/features/policy"  // 导入v2ray策略包
    "v2ray.com/core/features/routing"  // 导入v2ray路由包
    "v2ray.com/core/transport/internet"  // 导入v2ray互联网传输包
    "v2ray.com/core/transport/internet/udp"  // 导入v2ray UDP传输包
)

type Server struct {
    config        *ServerConfig  // 服务器配置
    user          *protocol.MemoryUser  // 用户内存信息
    policyManager policy.Manager  // 策略管理器
}

// NewServer create a new Shadowsocks server.
func NewServer(ctx context.Context, config *ServerConfig) (*Server, error) {
    if config.GetUser() == nil {  // 如果配置中没有用户信息
        return nil, newError("user is not specified")  // 返回错误信息
    }

    mUser, err := config.User.ToMemoryUser()  // 将配置中的用户信息转换为内存用户信息
    if err != nil {
        return nil, newError("failed to parse user account").Base(err)  // 返回转换错误信息
    }

    v := core.MustFromContext(ctx)  // 从上下文中获取v2ray核心信息
    s := &Server{  // 创建服务器对象
        config:        config,  // 设置配置信息
        user:          mUser,  // 设置用户信息
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),  // 设置策略管理器
    }

    return s, nil  // 返回服务器对象和空错误
}

func (s *Server) Network() []net.Network {
    list := s.config.Network  // 获取配置中的网络列表
    if len(list) == 0 {  // 如果网络列表为空
        list = append(list, net.Network_TCP)  // 添加TCP网络到列表中
    }
    if s.config.UdpEnabled {  // 如果UDP启用
        list = append(list, net.Network_UDP)  // 添加UDP网络到列表中
    }
    return list  // 返回网络列表
}

func (s *Server) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error {
    switch network {  // 根据网络类型进行处理
    case net.Network_TCP:  // 如果是TCP网络
        return s.handleConnection(ctx, conn, dispatcher)  // 处理连接
    case net.Network_UDP:  // 如果是UDP网络
        return s.handlerUDPPayload(ctx, conn, dispatcher)  // 处理UDP数据包
    default:  // 其他情况
        return newError("unknown network: ", network)  // 返回未知网络错误
    }
}
func (s *Server) handlerUDPPayload(ctx context.Context, conn internet.Connection, dispatcher routing.Dispatcher) error {
    // 创建一个 UDP 服务器分发器，用于处理 UDP 数据包
    udpServer := udp.NewDispatcher(dispatcher, func(ctx context.Context, packet *udp_proto.Packet) {
        // 从上下文中获取请求头部信息
        request := protocol.RequestHeaderFromContext(ctx)
        if request == nil {
            return
        }

        // 获取 UDP 数据包的载荷
        payload := packet.Payload
        // 编码 UDP 数据包
        data, err := EncodeUDPPacket(request, payload.Bytes())
        // 释放载荷
        payload.Release()
        if err != nil {
            // 如果编码失败，记录错误信息并返回
            newError("failed to encode UDP packet").Base(err).AtWarning().WriteToLog(session.ExportIDToError(ctx))
            return
        }
        // 在函数返回时释放数据
        defer data.Release()

        // 将编码后的数据写入连接
        conn.Write(data.Bytes())
    })

    // 从上下文中获取入站连接信息
    inbound := session.InboundFromContext(ctx)
    if inbound == nil {
        // 如果入站连接信息为空，抛出异常
        panic("no inbound metadata")
    }
    // 设置入站连接的用户信息
    inbound.User = s.user

    // 创建一个连接数据包读取器
    reader := buf.NewPacketReader(conn)
}
    // 无限循环，读取多个缓冲区的数据
    for {
        // 读取多个缓冲区的数据
        mpayload, err := reader.ReadMultiBuffer()
        // 如果读取出错，则跳出循环
        if err != nil {
            break
        }

        // 遍历多个缓冲区的数据
        for _, payload := range mpayload {
            // 解码 UDP 数据包
            request, data, err := DecodeUDPPacket(s.user, payload)
            // 如果解码出错
            if err != nil {
                // 如果存在传入连接并且连接源地址有效，则记录日志并丢弃无效的 UDP 数据包
                if inbound := session.InboundFromContext(ctx); inbound != nil && inbound.Source.IsValid() {
                    newError("dropping invalid UDP packet from: ", inbound.Source).Base(err).WriteToLog(session.ExportIDToError(ctx))
                    log.Record(&log.AccessMessage{
                        From:   inbound.Source,
                        To:     "",
                        Status: log.AccessRejected,
                        Reason: err,
                    })
                }
                // 释放数据包的缓冲区并继续下一次循环
                payload.Release()
                continue
            }

            // 设置当前数据包的上下文
            currentPacketCtx := ctx
            dest := request.Destination()
            // 如果传入连接的源地址有效，则更新当前数据包的上下文
            if inbound.Source.IsValid() {
                currentPacketCtx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
                    From:   inbound.Source,
                    To:     dest,
                    Status: log.AccessAccepted,
                    Reason: "",
                    Email:  request.User.Email,
                })
            }
            // 记录日志，表示正在将请求转发到目标地址
            newError("tunnelling request to ", dest).WriteToLog(session.ExportIDToError(currentPacketCtx))

            // 更新当前数据包的上下文，并分发数据包
            currentPacketCtx = protocol.ContextWithRequestHeader(currentPacketCtx, request)
            udpServer.Dispatch(currentPacketCtx, dest, data)
        }
    }

    // 返回空值
    return nil
// 处理连接的方法，接收上下文、连接、调度器作为参数，返回错误
func (s *Server) handleConnection(ctx context.Context, conn internet.Connection, dispatcher routing.Dispatcher) error {
    // 根据用户级别获取会话策略，并设置连接的读取截止时间
    sessionPolicy := s.policyManager.ForLevel(s.user.Level)
    conn.SetReadDeadline(time.Now().Add(sessionPolicy.Timeouts.Handshake))

    // 创建带缓冲的读取器，并从连接中读取TCP会话
    bufferedReader := buf.BufferedReader{Reader: buf.NewReader(conn)}
    request, bodyReader, err := ReadTCPSession(s.user, &bufferedReader)
    if err != nil {
        // 如果读取失败，记录访问日志并返回错误
        log.Record(&log.AccessMessage{
            From:   conn.RemoteAddr(),
            To:     "",
            Status: log.AccessRejected,
            Reason: err,
        })
        return newError("failed to create request from: ", conn.RemoteAddr()).Base(err)
    }
    // 重置连接的读取截止时间
    conn.SetReadDeadline(time.Time{})

    // 从上下文中获取入站会话信息，如果为空则抛出异常
    inbound := session.InboundFromContext(ctx)
    if inbound == nil {
        panic("no inbound metadata")
    }
    // 设置入站会话的用户信息为当前用户
    inbound.User = s.user

    // 获取请求的目标地址，并在上下文中记录访问日志
    dest := request.Destination()
    ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
        From:   conn.RemoteAddr(),
        To:     dest,
        Status: log.AccessAccepted,
        Reason: "",
        Email:  request.User.Email,
    })
    // 记录日志，表示正在将请求隧道传输到目标地址
    newError("tunnelling request to ", dest).WriteToLog(session.ExportIDToError(ctx))

    // 创建一个新的上下文，并设置定时器，用于在连接空闲时取消连接
    ctx, cancel := context.WithCancel(ctx)
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)

    // 在上下文中设置缓冲策略，并使用调度器分发请求到目标地址
    ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)
    link, err := dispatcher.Dispatch(ctx, dest)
    if err != nil {
        return err
    }
    responseDone := func() error {
        // 设置定时器，确保在函数结束时执行
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        // 创建带缓冲的写入器
        bufferedWriter := buf.NewBufferedWriter(buf.NewWriter(conn))
        // 写入 TCP 响应
        responseWriter, err := WriteTCPResponse(request, bufferedWriter)
        if err != nil {
            return newError("failed to write response").Base(err)
        }

        {
            // 从链接读取多个数据包
            payload, err := link.Reader.ReadMultiBuffer()
            if err != nil {
                return err
            }
            // 将读取的数据包写入响应
            if err := responseWriter.WriteMultiBuffer(payload); err != nil {
                return err
            }
        }

        // 将缓冲写入器的缓冲状态设置为 false
        if err := bufferedWriter.SetBuffered(false); err != nil {
            return err
        }

        // 将链接读取的数据传输到响应写入器，同时更新定时器的活动状态
        if err := buf.Copy(link.Reader, responseWriter, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transport all TCP response").Base(err)
        }

        return nil
    }

    requestDone := func() error {
        // 设置定时器，确保在函数结束时执行
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        // 将请求体的数据传输到链接的写入器，同时更新定时器的活动状态
        if err := buf.Copy(bodyReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transport all TCP request").Base(err)
        }

        return nil
    }

    // 创建一个任务，当请求完成后关闭链接的写入器
    var requestDoneAndCloseWriter = task.OnSuccess(requestDone, task.Close(link.Writer))
    // 运行任务，如果出现错误则中断链接的读取和写入，并返回错误信息
    if err := task.Run(ctx, requestDoneAndCloseWriter, responseDone); err != nil {
        common.Interrupt(link.Reader)
        common.Interrupt(link.Writer)
        return newError("connection ends").Base(err)
    }

    return nil
# 初始化函数，用于注册配置并创建服务器实例
func init() {
    # 注册 ServerConfig 结构体，并指定创建服务器实例的函数
    common.Must(common.RegisterConfig((*ServerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewServer(ctx, config.(*ServerConfig))
    }))
}
```