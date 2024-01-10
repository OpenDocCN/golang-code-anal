# `v2ray-core\proxy\trojan\client.go`

```
// +build !confonly

package trojan

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

// Client is a inbound handler for trojan protocol
type Client struct {
    serverPicker  protocol.ServerPicker
    policyManager policy.Manager
}

// NewClient create a new trojan client.
func NewClient(ctx context.Context, config *ClientConfig) (*Client, error) {
    serverList := protocol.NewServerList()
    for _, rec := range config.Server {
        s, err := protocol.NewServerSpecFromPB(rec)
        if err != nil {
            return nil, newError("failed to parse server spec").Base(err)
        }
        serverList.AddServer(s)
    }
    if serverList.Size() == 0 {
        return nil, newError("0 server")
    }

    v := core.MustFromContext(ctx)
    client := &Client{
        serverPicker:  protocol.NewRoundRobinServerPicker(serverList),
        policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),
    }
    return client, nil
}

// Process implements OutboundHandler.Process().
func (c *Client) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error { // nolint: funlen
    outbound := session.OutboundFromContext(ctx)
    if outbound == nil || !outbound.Target.IsValid() {
        return newError("target not specified")
    }
    destination := outbound.Target
    network := destination.Network

    var server *protocol.ServerSpec
    var conn internet.Connection
    // 使用指数退避算法进行重试，最多重试5次，初始等待时间为100毫秒
    err := retry.ExponentialBackoff(5, 100).On(func() error { // nolint: gomnd
        // 从服务器选择器中选择一个服务器
        server = c.serverPicker.PickServer()
        // 使用拨号器连接到选定服务器的目的地
        rawConn, err := dialer.Dial(ctx, server.Destination())
        if err != nil {
            return err
        }

        // 将连接赋值给conn
        conn = rawConn
        return nil
    })
    // 如果发生错误，则返回一个新的错误，指示未找到可用目的地
    if err != nil {
        return newError("failed to find an available destination").AtWarning().Base(err)
    }
    // 将隧道请求记录到日志中
    newError("tunneling request to ", destination, " via ", server.Destination()).WriteToLog(session.ExportIDToError(ctx))

    // 延迟关闭连接
    defer conn.Close()

    // 从服务器中选择一个用户
    user := server.PickUser()
    // 尝试将用户账户断言为MemoryAccount类型，如果不成功则返回一个新的错误
    account, ok := user.Account.(*MemoryAccount)
    if !ok {
        return newError("user account is not valid")
    }

    // 根据用户级别获取会话策略
    sessionPolicy := c.policyManager.ForLevel(user.Level)
    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(ctx)
    // 设置一个定时器，在连接空闲时取消上下文
    timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
    // 定义一个 postRequest 函数，用于发送请求
    postRequest := func() error {
        // 在函数结束时设置定时器超时时间为 sessionPolicy.Timeouts.DownlinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

        // 定义一个 bodyWriter 变量，用于写入请求数据
        var bodyWriter buf.Writer
        // 创建一个 bufferWriter 对象，用于缓冲写入数据
        bufferWriter := buf.NewBufferedWriter(buf.NewWriter(conn))
        // 创建一个 ConnWriter 对象，用于写入数据到连接
        connWriter := &ConnWriter{Writer: bufferWriter, Target: destination, Account: account}

        // 根据目标网络类型选择不同的写入方式
        if destination.Network == net.Network_UDP {
            bodyWriter = &PacketWriter{Writer: connWriter, Target: destination}
        } else {
            bodyWriter = connWriter
        }

        // 将一些请求有效载荷写入缓冲区
        if err = buf.CopyOnceTimeout(link.Reader, bodyWriter, time.Millisecond*100); err != nil && err != buf.ErrNotTimeoutReader && err != buf.ErrReadTimeout { // nolint: lll,gomnd
            return newError("failed to write A reqeust payload").Base(err).AtWarning()
        }

        // 刷新缓冲区；bufferWriter.WriteMultiBufer 现在是 bufferWriter.writer.WriteMultiBuffer
        if err = bufferWriter.SetBuffered(false); err != nil {
            return newError("failed to flush payload").Base(err).AtWarning()
        }

        // 将请求有效载荷传输到连接
        if err = buf.Copy(link.Reader, bodyWriter, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transfer request payload").Base(err).AtInfo()
        }

        return nil
    }

    // 定义一个 getResponse 函数，用于接收响应
    getResponse := func() error {
        // 在函数结束时设置定时器超时时间为 sessionPolicy.Timeouts.UplinkOnly
        defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

        // 定义一个 reader 变量，用于读取响应数据
        var reader buf.Reader
        // 根据网络类型选择不同的读取方式
        if network == net.Network_UDP {
            reader = &PacketReader{
                Reader: conn,
            }
        } else {
            reader = buf.NewReader(conn)
        }
        // 从连接读取数据并写入到 link.Writer 中
        return buf.Copy(reader, link.Writer, buf.UpdateActivity(timer))
    }

    // 定义一个 responseDoneAndCloseWriter 变量，用于在 getResponse 成功后关闭 link.Writer
    var responseDoneAndCloseWriter = task.OnSuccess(getResponse, task.Close(link.Writer))
    // 运行 postRequest 函数和 responseDoneAndCloseWriter 任务
    if err := task.Run(ctx, postRequest, responseDoneAndCloseWriter); err != nil {
        return newError("connection ends").Base(err)
    }

    return nil
# 初始化函数，用于注册配置和创建客户端对象
func init() {
    # 强制注册配置，当配置发生变化时，创建新的客户端对象
    common.Must(common.RegisterConfig((*ClientConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) { // nolint: lll
        # 创建新的客户端对象并返回
        return NewClient(ctx, config.(*ClientConfig))
    }))
}
```