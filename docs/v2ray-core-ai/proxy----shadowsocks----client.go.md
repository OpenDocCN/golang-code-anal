# `v2ray-core\proxy\shadowsocks\client.go`

```
// +build !confonly

package shadowsocks

import (
    "context"  // 导入 context 包，用于处理上下文
    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/buf"  // 导入 v2ray 核心通用缓冲包
    "v2ray.com/core/common/net"  // 导入 v2ray 核心通用网络包
    "v2ray.com/core/common/protocol"  // 导入 v2ray 核心通用协议包
    "v2ray.com/core/common/retry"  // 导入 v2ray 核心通用重试包
    "v2ray.com/core/common/session"  // 导入 v2ray 核心通用会话包
    "v2ray.com/core/common/signal"  // 导入 v2ray 核心通用信号包
    "v2ray.com/core/common/task"  // 导入 v2ray 核心通用任务包
    "v2ray.com/core/features/policy"  // 导入 v2ray 核心特性策略包
    "v2ray.com/core/transport"  // 导入 v2ray 核心传输包
    "v2ray.com/core/transport/internet"  // 导入 v2ray 核心传输互联网包
)

// Client is a inbound handler for Shadowsocks protocol
type Client struct {
    serverPicker  protocol.ServerPicker  // 定义 serverPicker 属性，用于选择服务器
    policyManager policy.Manager  // 定义 policyManager 属性，用于管理策略
}

// NewClient create a new Shadowsocks client.
func NewClient(ctx context.Context, config *ClientConfig) (*Client, error) {
    serverList := protocol.NewServerList()  // 创建新的服务器列表
    for _, rec := range config.Server {  // 遍历配置中的服务器
        s, err := protocol.NewServerSpecFromPB(rec)  // 从配置中创建新的服务器规范
        if err != nil {
            return nil, newError("failed to parse server spec").Base(err)  // 如果出错，返回错误信息
        }
        serverList.AddServer(s)  // 将服务器添加到服务器列表中
    }
    if serverList.Size() == 0 {  // 如果服务器列表为空
        return nil, newError("0 server")  // 返回错误信息
    }

    v := core.MustFromContext(ctx)  // 从上下文中获取核心
    client := &Client{  // 创建新的客户端
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList),  // 使用轮询方式选择服务器
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),  // 获取并设置策略管理器
    }
    return client, nil  // 返回客户端和空错误
}

// Process implements OutboundHandler.Process().
func (c *Client) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    outbound := session.OutboundFromContext(ctx)  // 从上下文中获取出站会话
    if outbound == nil || !outbound.Target.IsValid() {  // 如果出站会话为空或目标无效
        return newError("target not specified")  // 返回错误信息
    }
    destination := outbound.Target  // 获取目标地址
    network := destination.Network  // 获取目标地址的网络类型

    var server *protocol.ServerSpec  // 定义服务器规范变量
    var conn internet.Connection  // 定义互联网连接变量
    // 使用指数退避算法进行重试，最多重试5次，初始间隔100毫秒
    err := retry.ExponentialBackoff(5, 100).On(func() error {
        // 从服务器选择器中选择一个服务器
        server = c.serverPicker.PickServer()
        // 获取服务器的目的地信息
        dest := server.Destination()
        // 设置目的地的网络类型
        dest.Network = network
        // 使用拨号器拨号连接目的地
        rawConn, err := dialer.Dial(ctx, dest)
        // 如果出现错误，返回错误信息
        if err != nil {
            return err
        }
        // 将连接赋值给conn
        conn = rawConn

        return nil
    })
    // 如果重试过程中出现错误，返回找不到可用目的地的错误信息
    if err != nil {
        return newError("failed to find an available destination").AtWarning().Base(err)
    }
    // 记录隧道请求的日志信息
    newError("tunneling request to ", destination, " via ", server.Destination()).WriteToLog(session.ExportIDToError(ctx))

    // 延迟关闭连接
    defer conn.Close()

    // 创建一个协议请求头
    request := &protocol.RequestHeader{
        Version: Version,
        Address: destination.Address,
        Port:    destination.Port,
    }
    // 根据目的地的网络类型设置请求命令
    if destination.Network == net.Network_TCP {
        request.Command = protocol.RequestCommandTCP
    } else {
        request.Command = protocol.RequestCommandUDP
    }

    // 从服务器中选择一个用户
    user := server.PickUser()
    // 检查用户账户是否为内存账户
    _, ok := user.Account.(*MemoryAccount)
    if !ok {
        return newError("user account is not valid")
    }
    // 将用户信息赋值给请求头
    request.User = user

    // 根据用户级别获取会话策略
    sessionPolicy := c.policyManager.ForLevel(user.Level)
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(ctx)
    // 设置在连接空闲时自动取消的定时器
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
    # 如果请求命令是TCP
    if request.Command == protocol.RequestCommandTCP {
        # 创建一个带缓冲的写入器
        bufferedWriter := buf.NewBufferedWriter(buf.NewWriter(conn))
        # 将TCP请求写入到缓冲区
        bodyWriter, err := WriteTCPRequest(request, bufferedWriter)
        # 如果写入失败，返回错误
        if err != nil {
            return newError("failed to write request").Base(err)
        }

        # 关闭缓冲区
        if err := bufferedWriter.SetBuffered(false); err != nil {
            return err
        }

        # 定义请求完成的函数
        requestDone := func() error {
            # 设置定时器超时时间为下行连接超时时间
            defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)
            # 将数据从link.Reader复制到bodyWriter，并更新定时器活动时间
            return buf.Copy(link.Reader, bodyWriter, buf.UpdateActivity(timer))
        }

        # 定义响应完成的函数
        responseDone := func() error {
            # 设置定时器超时时间为上行连接超时时间
            defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)
            # 读取TCP响应，并将数据从responseReader复制到link.Writer，并更新定时器活动时间
            responseReader, err := ReadTCPResponse(user, conn)
            if err != nil {
                return err
            }
            return buf.Copy(responseReader, link.Writer, buf.UpdateActivity(timer))
        }

        # 定义响应完成后关闭写入器的任务
        var responseDoneAndCloseWriter = task.OnSuccess(responseDone, task.Close(link.Writer))
        # 运行请求完成、响应完成和关闭写入器的任务，并返回错误信息
        if err := task.Run(ctx, requestDone, responseDoneAndCloseWriter); err != nil {
            return newError("connection ends").Base(err)
        }

        # 返回空
        return nil
    }
    # 如果请求命令是UDP
    if request.Command == protocol.RequestCommandUDP {

        # 创建一个UDPWriter对象，用于向UDP连接写入数据
        writer := &buf.SequentialWriter{Writer: &UDPWriter{
            Writer:  conn,
            Request: request,
        }}

        # 定义请求完成的函数
        requestDone := func() error {
            # 设置定时器超时时间为sessionPolicy.Timeouts.DownlinkOnly
            defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

            # 将link.Reader的数据复制到writer中，并更新定时器活动时间
            if err := buf.Copy(link.Reader, writer, buf.UpdateActivity(timer)); err != nil {
                return newError("failed to transport all UDP request").Base(err)
            }
            return nil
        }

        # 定义响应完成的函数
        responseDone := func() error {
            # 设置定时器超时时间为sessionPolicy.Timeouts.UplinkOnly
            defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

            # 创建一个UDPReader对象，用于从UDP连接读取数据
            reader := &UDPReader{
                Reader: conn,
                User:   user,
            }

            # 将reader中的数据复制到link.Writer中，并更新定时器活动时间
            if err := buf.Copy(reader, link.Writer, buf.UpdateActivity(timer)); err != nil {
                return newError("failed to transport all UDP response").Base(err)
            }
            return nil
        }

        # 定义响应完成并关闭writer的任务
        var responseDoneAndCloseWriter = task.OnSuccess(responseDone, task.Close(link.Writer))
        
        # 运行请求完成、响应完成并关闭writer的任务，并检查是否有错误发生
        if err := task.Run(ctx, requestDone, responseDoneAndCloseWriter); err != nil {
            return newError("connection ends").Base(err)
        }

        # 返回空值
        return nil
    }

    # 返回空值
    return nil
# 初始化函数，用于注册配置和创建客户端
func init() {
    # 注册配置并创建客户端，如果出现错误则返回错误信息
    common.Must(common.RegisterConfig((*ClientConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewClient(ctx, config.(*ClientConfig))
    }))
}
```