# `v2ray-core\proxy\socks\client.go`

```
// +build !confonly

package socks

import (
    "context"
    "time"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/retry"
    "v2ray.com/core/common/session"
    "v2ray.com/core/common/signal"
    "v2ray.com/core/common/task"
    "v2ray.com/core/features/policy"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/internet"
)

// Client is a Socks5 client.
type Client struct {
    serverPicker  protocol.ServerPicker  // 服务器选择器
    policyManager policy.Manager  // 策略管理器
}

// NewClient create a new Socks5 client based on the given config.
func NewClient(ctx context.Context, config *ClientConfig) (*Client, error) {
    serverList := protocol.NewServerList()  // 创建一个新的服务器列表
    for _, rec := range config.Server {
        s, err := protocol.NewServerSpecFromPB(rec)  // 从配置中创建新的服务器规范
        if err != nil {
            return nil, newError("failed to get server spec").Base(err)  // 如果创建失败，则返回错误
        }
        serverList.AddServer(s)  // 将新创建的服务器添加到服务器列表中
    }
    if serverList.Size() == 0 {
        return nil, newError("0 target server")  // 如果服务器列表为空，则返回错误
    }

    v := core.MustFromContext(ctx)  // 从上下文中获取核心实例
    return &Client{
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList),  // 使用服务器列表创建一个新的轮询服务器选择器
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),  // 从核心实例中获取策略管理器
    }, nil
}

// Process implements proxy.Outbound.Process.
func (c *Client) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
    outbound := session.OutboundFromContext(ctx)  // 从上下文中获取出站会话
    if outbound == nil || !outbound.Target.IsValid() {
        return newError("target not specified.")  // 如果出站目标未指定，则返回错误
    }
    destination := outbound.Target  // 获取出站目标

    var server *protocol.ServerSpec  // 声明服务器规范变量
    var conn internet.Connection  // 声明网络连接变量
    # 使用指数退避算法进行重试，最多重试5次，初始间隔100毫秒
    if err := retry.ExponentialBackoff(5, 100).On(func() error {
        # 从服务器选择器中选择一个服务器
        server = c.serverPicker.PickServer()
        # 获取服务器的目的地地址
        dest := server.Destination()
        # 使用拨号器连接目的地地址
        rawConn, err := dialer.Dial(ctx, dest)
        if err != nil:
            return err
        # 将连接赋值给conn
        conn = rawConn

        return nil
    }); err != nil:
        return newError("failed to find an available destination").Base(err)

    # 延迟执行，用于在函数返回时关闭连接
    defer func() {
        if err := conn.Close(); err != nil:
            newError("failed to closed connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
    }()

    # 获取级别为0的策略管理器
    p := c.policyManager.ForLevel(0)

    # 创建一个请求头
    request := &protocol.RequestHeader{
        Version: socks5Version,
        Command: protocol.RequestCommandTCP,
        Address: destination.Address,
        Port:    destination.Port,
    }
    # 如果目的地网络是UDP，则将请求命令设置为UDP
    if destination.Network == net.Network_UDP:
        request.Command = protocol.RequestCommandUDP

    # 从服务器中选择一个用户
    user := server.PickUser()
    if user != nil:
        # 如果用户不为空，则将请求头中的用户设置为该用户
        request.User = user
        # 获取该用户级别对应的策略管理器
        p = c.policyManager.ForLevel(user.Level)

    # 设置连接的截止时间为握手超时时间
    if err := conn.SetDeadline(time.Now().Add(p.Timeouts.Handshake)); err != nil:
        newError("failed to set deadline for handshake").Base(err).WriteToLog(session.ExportIDToError(ctx))

    # 进行客户端握手，获取UDP请求
    udpRequest, err := ClientHandshake(request, conn, conn)
    if err != nil:
        return newError("failed to establish connection to server").AtWarning().Base(err)

    # 清除连接的截止时间
    if err := conn.SetDeadline(time.Time{}); err != nil:
        newError("failed to clear deadline after handshake").Base(err).WriteToLog(session.ExportIDToError(ctx))

    # 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(ctx)
    # 设置一个定时器，在连接空闲时取消连接
    timer := signal.CancelAfterInactivity(ctx, cancel, p.Timeouts.ConnectionIdle)

    # 定义请求函数和响应函数
    var requestFunc func() error
    var responseFunc func() error
    # 如果请求命令是TCP
    if request.Command == protocol.RequestCommandTCP:
        # 设置请求函数，延迟设置超时时间，然后将数据从link.Reader复制到conn的新写入缓冲区，并更新活动时间
        requestFunc = func() error {
            defer timer.SetTimeout(p.Timeouts.DownlinkOnly)
            return buf.Copy(link.Reader, buf.NewWriter(conn), buf.UpdateActivity(timer))
        }
        # 设置响应函数，延迟设置超时时间，然后将数据从conn的新读取缓冲区复制到link.Writer，并更新活动时间
        responseFunc = func() error {
            defer timer.SetTimeout(p.Timeouts.UplinkOnly)
            return buf.Copy(buf.NewReader(conn), link.Writer, buf.UpdateActivity(timer))
        }
    # 如果请求命令是UDP
    else if request.Command == protocol.RequestCommandUDP:
        # 使用dialer.Dial方法创建UDP连接
        udpConn, err := dialer.Dial(ctx, udpRequest.Destination())
        # 如果创建UDP连接失败，则返回错误
        if err != nil:
            return newError("failed to create UDP connection").Base(err)
        # 延迟关闭UDP连接
        defer udpConn.Close() // nolint: errcheck
        # 设置请求函数，延迟设置超时时间，然后将数据从link.Reader复制到新的UDP写入缓冲区，并更新活动时间
        requestFunc = func() error {
            defer timer.SetTimeout(p.Timeouts.DownlinkOnly)
            return buf.Copy(link.Reader, &buf.SequentialWriter{Writer: NewUDPWriter(request, udpConn)}, buf.UpdateActivity(timer))
        }
        # 设置响应函数，延迟设置超时时间，然后创建UDPReader对象，将数据从UDP连接读取到link.Writer，并更新活动时间
        responseFunc = func() error {
            defer timer.SetTimeout(p.Timeouts.UplinkOnly)
            reader := &UDPReader{reader: udpConn}
            return buf.Copy(reader, link.Writer, buf.UpdateActivity(timer))
        }

    # 设置响应完成后的任务，成功时关闭link.Writer
    var responseDonePost = task.OnSuccess(responseFunc, task.Close(link.Writer))
    # 运行请求函数和响应完成后的任务，如果出现错误，则返回连接结束的错误
    if err := task.Run(ctx, requestFunc, responseDonePost); err != nil:
        return newError("connection ends").Base(err)

    # 返回空值
    return nil
# 初始化函数，用于注册客户端配置并创建客户端实例
func init() {
    # 注册客户端配置，并在注册后立即创建客户端实例
    common.Must(common.RegisterConfig((*ClientConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewClient(ctx, config.(*ClientConfig))
    }))
}
```