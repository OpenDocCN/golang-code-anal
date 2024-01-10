# `v2ray-core\proxy\dokodemo\dokodemo.go`

```
// +build !confonly

package dokodemo

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"  // 导入 context 包
    "sync/atomic"  // 导入 sync/atomic 包
    "time"  // 导入 time 包

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/buf"  // 导入 v2ray 核心通用缓冲包
    "v2ray.com/core/common/log"  // 导入 v2ray 核心通用日志包
    "v2ray.com/core/common/net"  // 导入 v2ray 核心通用网络包
    "v2ray.com/core/common/protocol"  // 导入 v2ray 核心通用协议包
    "v2ray.com/core/common/session"  // 导入 v2ray 核心通用会话包
    "v2ray.com/core/common/signal"  // 导入 v2ray 核心通用信号包
    "v2ray.com/core/common/task"  // 导入 v2ray 核心通用任务包
    "v2ray.com/core/features/policy"  // 导入 v2ray 核心特性策略包
    "v2ray.com/core/features/routing"  // 导入 v2ray 核心特性路由包
    "v2ray.com/core/transport/internet"  // 导入 v2ray 核心传输互联网包
)

func init() {
    // 注册配置
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        d := new(DokodemoDoor)  // 创建 DokodemoDoor 实例
        err := core.RequireFeatures(ctx, func(pm policy.Manager) error {
            return d.Init(config.(*Config), pm, session.SockoptFromContext(ctx))  // 初始化 DokodemoDoor 实例
        })
        return d, err
    }))
}

type DokodemoDoor struct {
    policyManager policy.Manager  // 策略管理器
    config        *Config  // 配置
    address       net.Address  // 网络地址
    port          net.Port  // 端口
    sockopt       *session.Sockopt  // 会话选项
}

// Init initializes the DokodemoDoor instance with necessary parameters.
func (d *DokodemoDoor) Init(config *Config, pm policy.Manager, sockopt *session.Sockopt) error {
    if (config.NetworkList == nil || len(config.NetworkList.Network) == 0) && len(config.Networks) == 0 {
        return newError("no network specified")  // 如果未指定网络，则返回错误
    }
    d.config = config  // 设置配置
    d.address = config.GetPredefinedAddress()  // 获取预定义地址
    d.port = net.Port(config.Port)  // 设置端口
    d.policyManager = pm  // 设置策略管理器
    d.sockopt = sockopt  // 设置会话选项

    return nil  // 返回空
}

// Network implements proxy.Inbound.
func (d *DokodemoDoor) Network() []net.Network {
    if len(d.config.Networks) > 0 {
        return d.config.Networks  // 如果配置中有网络，则返回配置中的网络
    }

    return d.config.NetworkList.Network  // 返回配置中的网络列表
}

func (d *DokodemoDoor) policy() policy.Session {
    config := d.config  // 获取配置
    p := d.policyManager.ForLevel(config.UserLevel)  // 获取用户级别的策略
    # 如果配置的超时时间大于0并且用户级别为0
    if config.Timeout > 0 && config.UserLevel == 0:
        # 将连接空闲超时时间设置为配置的超时时间
        p.Timeouts.ConnectionIdle = time.Duration(config.Timeout) * time.Second
    # 返回程序对象
    return p
// 定义一个接口类型，该接口包含 HandshakeAddress 方法
type hasHandshakeAddress interface {
    HandshakeAddress() net.Address
}

// 实现 proxy.Inbound 接口的 Process 方法
func (d *DokodemoDoor) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error {
    // 记录连接来源的地址信息
    newError("processing connection from: ", conn.RemoteAddr()).AtDebug().WriteToLog(session.ExportIDToError(ctx))
    // 设置目标地址
    dest := net.Destination{
        Network: network,
        Address: d.address,
        Port:    d.port,
    }

    // 标记目标地址是否被重定向
    destinationOverridden := false
    // 如果配置允许重定向
    if d.config.FollowRedirect {
        // 如果会话中存在出站连接，并且目标地址有效，则使用出站连接的目标地址
        if outbound := session.OutboundFromContext(ctx); outbound != nil && outbound.Target.IsValid() {
            dest = outbound.Target
            destinationOverridden = true
        } else if handshake, ok := conn.(hasHandshakeAddress); ok {
            // 如果连接实现了 hasHandshakeAddress 接口，则使用握手地址作为目标地址
            addr := handshake.HandshakeAddress()
            if addr != nil {
                dest.Address = addr
                destinationOverridden = true
            }
        }
    }
    // 如果目标地址无效或者地址为空，则返回错误
    if !dest.IsValid() || dest.Address == nil {
        return newError("unable to get destination")
    }

    // 如果会话中存在入站连接，则设置用户级别
    if inbound := session.InboundFromContext(ctx); inbound != nil {
        inbound.User = &protocol.MemoryUser{
            Level: d.config.UserLevel,
        }
    }

    // 记录访问日志
    ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
        From:   conn.RemoteAddr(),
        To:     dest,
        Status: log.AccessAccepted,
        Reason: "",
    })
    newError("received request for ", conn.RemoteAddr()).WriteToLog(session.ExportIDToError(ctx))

    // 获取策略并设置超时
    plcy := d.policy()
    ctx, cancel := context.WithCancel(ctx)
    timer := signal.CancelAfterInactivity(ctx, cancel, plcy.Timeouts.ConnectionIdle)

    // 设置缓冲策略
    ctx = policy.ContextWithBufferPolicy(ctx, plcy.Buffer)
    // 调度请求并获取链接
    link, err := dispatcher.Dispatch(ctx, dest)
    if err != nil {
        return newError("failed to dispatch request").Base(err)
    }

    // 记录请求计数
    requestCount := int32(1)
}
    # 定义一个匿名函数，用于处理请求完成后的操作
    requestDone := func() error {
        # 延迟执行的函数，用于在请求完成后减少请求计数，并在计数为0时设置超时
        defer func() {
            if atomic.AddInt32(&requestCount, -1) == 0 {
                timer.SetTimeout(plcy.Timeouts.DownlinkOnly)
            }
        }()

        # 定义一个变量用于读取数据
        var reader buf.Reader
        # 如果目标网络是UDP，则使用PacketReader来读取数据
        if dest.Network == net.Network_UDP {
            reader = buf.NewPacketReader(conn)
        } else {
            # 否则使用普通的Reader来读取数据
            reader = buf.NewReader(conn)
        }
        # 将读取到的数据写入到link.Writer中，并更新活动时间
        if err := buf.Copy(reader, link.Writer, buf.UpdateActivity(timer)); err != nil {
            # 如果出现错误，则返回包含错误信息的新Error
            return newError("failed to transport request").Base(err)
        }

        # 请求处理完成，返回nil
        return nil
    }

    # 定义一个匿名函数，用于处理tproxy请求
    tproxyRequest := func() error {
        # tproxy请求处理，暂时返回nil
        return nil
    }

    # 定义一个变量用于写入数据
    var writer buf.Writer
    # 如果网络类型是TCP，则使用NewWriter来创建写入数据的对象
    if network == net.Network_TCP {
        writer = buf.NewWriter(conn)
    } else {
        // 如果处于 TPROXY 模式，则使用 Linux 的 UDP 伪造功能
        if !destinationOverridden {
            // 如果目标地址没有被覆盖，则使用顺序写入器
            writer = &buf.SequentialWriter{Writer: conn}
        } else {
            // 否则，创建一个 Socket 配置对象
            sockopt := &internet.SocketConfig{
                Tproxy: internet.SocketConfig_TProxy,
            }
            // 如果目标地址是 IP 地址，则设置绑定地址和端口
            if dest.Address.Family().IsIP() {
                sockopt.BindAddress = dest.Address.IP()
                sockopt.BindPort = uint32(dest.Port)
            }
            // 如果存在自定义的 sockopt，则设置 Mark
            if d.sockopt != nil {
                sockopt.Mark = d.sockopt.Mark
            }
            // 使用系统调用方式建立连接
            tConn, err := internet.DialSystem(ctx, net.DestinationFromAddr(conn.RemoteAddr()), sockopt)
            if err != nil {
                return err
            }
            defer tConn.Close()

            // 使用顺序写入器
            writer = &buf.SequentialWriter{Writer: tConn}
            // 创建一个 PacketReader
            tReader := buf.NewPacketReader(tConn)
            // 增加请求计数
            requestCount++
            // 定义 tproxyRequest 函数
            tproxyRequest = func() error {
                defer func() {
                    // 如果请求计数减为 0，则设置定时器超时时间
                    if atomic.AddInt32(&requestCount, -1) == 0 {
                        timer.SetTimeout(plcy.Timeouts.DownlinkOnly)
                    }
                }()
                // 通过 tReader 从 link.Writer 读取数据，并更新活动时间
                if err := buf.Copy(tReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
                    return newError("failed to transport request (TPROXY conn)").Base(err)
                }
                return nil
            }
        }
    }

    // 定义 responseDone 函数
    responseDone := func() error {
        defer timer.SetTimeout(plcy.Timeouts.UplinkOnly)

        // 通过 link.Reader 从 writer 写入数据，并更新活动时间
        if err := buf.Copy(link.Reader, writer, buf.UpdateActivity(timer)); err != nil {
            return newError("failed to transport response").Base(err)
        }
        return nil
    }

    // 运行任务，成功后执行 requestDone 和 tproxyRequest 函数
    if err := task.Run(ctx, task.OnSuccess(func() error {
        return task.Run(ctx, requestDone, tproxyRequest)
    // 如果在发送任务关闭信号时发生错误，则中断读取和写入操作，并返回连接结束的错误
    }, task.Close(link.Writer)), responseDone); err != nil {
        common.Interrupt(link.Reader)
        common.Interrupt(link.Writer)
        return newError("connection ends").Base(err)
    }
    // 如果没有发生错误，则返回空值
    return nil
# 闭合前面的函数定义
```