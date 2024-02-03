# `v2ray-core\app\proxyman\inbound\always.go`

```go
package inbound

import (
    "context"

    "v2ray.com/core"
    "v2ray.com/core/app/proxyman"
    "v2ray.com/core/common"
    "v2ray.com/core/common/dice"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/mux"
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/policy"
    "v2ray.com/core/features/stats"
    "v2ray.com/core/proxy"
    "v2ray.com/core/transport/internet"
)

func getStatCounter(v *core.Instance, tag string) (stats.Counter, stats.Counter) {
    var uplinkCounter stats.Counter  // 定义上行流量统计器
    var downlinkCounter stats.Counter  // 定义下行流量统计器

    policy := v.GetFeature(policy.ManagerType()).(policy.Manager)  // 获取策略管理器
    if len(tag) > 0 && policy.ForSystem().Stats.InboundUplink {  // 如果标签不为空且策略允许统计入站上行流量
        statsManager := v.GetFeature(stats.ManagerType()).(stats.Manager)  // 获取统计管理器
        name := "inbound>>>" + tag + ">>>traffic>>>uplink"  // 组装上行流量统计器名称
        c, _ := stats.GetOrRegisterCounter(statsManager, name)  // 获取或注册上行流量统计器
        if c != nil {
            uplinkCounter = c  // 如果统计器不为空，则赋值给上行流量统计器
        }
    }
    if len(tag) > 0 && policy.ForSystem().Stats.InboundDownlink {  // 如果标签不为空且策略允许统计入站下行流量
        statsManager := v.GetFeature(stats.ManagerType()).(stats.Manager)  // 获取统计管理器
        name := "inbound>>>" + tag + ">>>traffic>>>downlink"  // 组装下行流量统计器名称
        c, _ := stats.GetOrRegisterCounter(statsManager, name)  // 获取或注册下行流量统计器
        if c != nil {
            downlinkCounter = c  // 如果统计器不为空，则赋值给下行流量统计器
        }
    }

    return uplinkCounter, downlinkCounter  // 返回上行和下行流量统计器
}

type AlwaysOnInboundHandler struct {
    proxy   proxy.Inbound  // 定义入站代理
    workers []worker  // 定义工作线程
    mux     *mux.Server  // 定义多路复用服务器
    tag     string  // 定义标签
}

func NewAlwaysOnInboundHandler(ctx context.Context, tag string, receiverConfig *proxyman.ReceiverConfig, proxyConfig interface{}) (*AlwaysOnInboundHandler, error) {
    rawProxy, err := common.CreateObject(ctx, proxyConfig)  // 创建代理对象
    if err != nil {
        return nil, err  // 如果出错，返回错误
    }
    p, ok := rawProxy.(proxy.Inbound)  // 断言代理对象为入站代理
    if !ok {
        return nil, newError("not an inbound proxy.")  // 如果不是入站代理，返回错误
    }

    h := &AlwaysOnInboundHandler{
        proxy: p,  // 初始化入站代理
        mux:   mux.NewServer(ctx),  // 初始化多路复用服务器
        tag:   tag,  // 初始化标签
    }
    # 从上下文中获取统计计数器，分别获取上行和下行的计数器
    uplinkCounter, downlinkCounter := getStatCounter(core.MustFromContext(ctx), tag)

    # 获取网络实例
    nl := p.Network()
    # 获取接收器配置的端口范围
    pr := receiverConfig.PortRange
    # 获取接收器配置的监听地址，如果为空则设置为任意IP
    address := receiverConfig.Listen.AsAddress()
    if address == nil {
        address = net.AnyIP
    }

    # 将接收器配置的流设置转换为内存流配置
    mss, err := internet.ToMemoryStreamConfig(receiverConfig.StreamSettings)
    if err != nil {
        # 如果转换失败，则返回错误
        return nil, newError("failed to parse stream config").Base(err).AtWarning()
    }

    # 如果接收器配置要接收原始目标地址
    if receiverConfig.ReceiveOriginalDestination {
        # 如果流配置的套接字设置为空，则创建一个新的套接字配置
        if mss.SocketSettings == nil {
            mss.SocketSettings = &internet.SocketConfig{}
        }
        # 如果套接字设置的透明代理为关闭状态，则设置为重定向
        if mss.SocketSettings.Tproxy == internet.SocketConfig_Off {
            mss.SocketSettings.Tproxy = internet.SocketConfig_Redirect
        }
        # 设置套接字设置为接收原始目标地址
        mss.SocketSettings.ReceiveOriginalDestAddress = true
    }
    # 遍历指定范围内的端口号
    for port := pr.From; port <= pr.To; port++ {
        # 检查网络列表中是否包含 TCP 网络
        if net.HasNetwork(nl, net.Network_TCP) {
            # 创建 TCP 工作器，并输出调试信息
            newError("creating stream worker on ", address, ":", port).AtDebug().WriteToLog()

            # 初始化 TCP 工作器对象
            worker := &tcpWorker{
                address:         address,
                port:            net.Port(port),
                proxy:           p,
                stream:          mss,
                recvOrigDest:    receiverConfig.ReceiveOriginalDestination,
                tag:             tag,
                dispatcher:      h.mux,
                sniffingConfig:  receiverConfig.GetEffectiveSniffingSettings(),
                uplinkCounter:   uplinkCounter,
                downlinkCounter: downlinkCounter,
                ctx:             ctx,
            }
            # 将 TCP 工作器对象添加到工作器列表中
            h.workers = append(h.workers, worker)
        }

        # 检查网络列表中是否包含 UDP 网络
        if net.HasNetwork(nl, net.Network_UDP) {
            # 初始化 UDP 工作器对象
            worker := &udpWorker{
                tag:             tag,
                proxy:           p,
                address:         address,
                port:            net.Port(port),
                dispatcher:      h.mux,
                uplinkCounter:   uplinkCounter,
                downlinkCounter: downlinkCounter,
                stream:          mss,
            }
            # 将 UDP 工作器对象添加到工作器列表中
            h.workers = append(h.workers, worker)
        }
    }

    # 返回工作器列表和空错误
    return h, nil
// Start 方法实现了 common.Runnable 接口
func (h *AlwaysOnInboundHandler) Start() error {
    // 遍历 workers 切片，逐个启动 worker
    for _, worker := range h.workers {
        if err := worker.Start(); err != nil {
            return err
        }
    }
    // 返回 nil 表示启动成功
    return nil
}

// Close 方法实现了 common.Closable 接口
func (h *AlwaysOnInboundHandler) Close() error {
    var errs []error
    // 遍历 workers 切片，逐个关闭 worker
    for _, worker := range h.workers {
        errs = append(errs, worker.Close())
    }
    // 关闭 mux
    errs = append(errs, h.mux.Close())
    // 合并所有错误，如果有错误则返回包含错误信息的新错误
    if err := errors.Combine(errs...); err != nil {
        return newError("failed to close all resources").Base(err)
    }
    // 返回 nil 表示关闭成功
    return nil
}

// GetRandomInboundProxy 方法返回随机的入站代理
func (h *AlwaysOnInboundHandler) GetRandomInboundProxy() (interface{}, net.Port, int) {
    // 如果 workers 切片为空，则返回默认值
    if len(h.workers) == 0 {
        return nil, 0, 0
    }
    // 随机选择一个 worker，并返回其代理、端口和固定值 9999
    w := h.workers[dice.Roll(len(h.workers))]
    return w.Proxy(), w.Port(), 9999
}

// Tag 方法返回标签
func (h *AlwaysOnInboundHandler) Tag() string {
    return h.tag
}

// GetInbound 方法返回入站代理
func (h *AlwaysOnInboundHandler) GetInbound() proxy.Inbound {
    return h.proxy
}
```