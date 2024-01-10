# `v2ray-core\app\proxyman\inbound\dynamic.go`

```
package inbound

import (
    "context"  // 导入上下文包
    "sync"  // 导入同步包
    "time"  // 导入时间包

    "v2ray.com/core"  // 导入v2ray核心包
    "v2ray.com/core/app/proxyman"  // 导入代理管理包
    "v2ray.com/core/common/dice"  // 导入随机数生成包
    "v2ray.com/core/common/mux"  // 导入多路复用包
    "v2ray.com/core/common/net"  // 导入网络通用包
    "v2ray.com/core/common/task"  // 导入任务包
    "v2ray.com/core/proxy"  // 导入代理包
    "v2ray.com/core/transport/internet"  // 导入网络传输包
)

type DynamicInboundHandler struct {
    tag            string  // 定义标签字符串
    v              *core.Instance  // 定义v2ray实例指针
    proxyConfig    interface{}  // 定义代理配置接口
    receiverConfig *proxyman.ReceiverConfig  // 定义代理接收配置指针
    streamSettings *internet.MemoryStreamConfig  // 定义内存流配置指针
    portMutex      sync.Mutex  // 定义端口互斥锁
    portsInUse     map[net.Port]bool  // 定义端口使用情况映射
    workerMutex    sync.RWMutex  // 定义工作线程互斥锁
    worker         []worker  // 定义工作线程数组
    lastRefresh    time.Time  // 定义最后刷新时间
    mux            *mux.Server  // 定义多路复用服务器指针
    task           *task.Periodic  // 定义周期性任务指针

    ctx context.Context  // 定义上下文
}

func NewDynamicInboundHandler(ctx context.Context, tag string, receiverConfig *proxyman.ReceiverConfig, proxyConfig interface{}) (*DynamicInboundHandler, error) {
    v := core.MustFromContext(ctx)  // 从上下文中获取v2ray实例
    h := &DynamicInboundHandler{  // 创建动态入站处理器对象
        tag:            tag,  // 设置标签
        proxyConfig:    proxyConfig,  // 设置代理配置
        receiverConfig: receiverConfig,  // 设置代理接收配置
        portsInUse:     make(map[net.Port]bool),  // 初始化端口使用情况映射
        mux:            mux.NewServer(ctx),  // 创建新的多路复用服务器
        v:              v,  // 设置v2ray实例
        ctx:            ctx,  // 设置上下文
    }

    mss, err := internet.ToMemoryStreamConfig(receiverConfig.StreamSettings)  // 将接收配置的流设置转换为内存流配置
    if err != nil {  // 如果转换出错
        return nil, newError("failed to parse stream settings").Base(err).AtWarning()  // 返回错误信息
    }
    if receiverConfig.ReceiveOriginalDestination {  // 如果接收原始目的地为真
        if mss.SocketSettings == nil {  // 如果套接字设置为空
            mss.SocketSettings = &internet.SocketConfig{}  // 创建新的套接字设置
        }
        if mss.SocketSettings.Tproxy == internet.SocketConfig_Off {  // 如果透明代理关闭
            mss.SocketSettings.Tproxy = internet.SocketConfig_Redirect  // 设置为重定向
        }
        mss.SocketSettings.ReceiveOriginalDestAddress = true  // 设置接收原始目的地地址为真
    }

    h.streamSettings = mss  // 设置内存流配置
    # 将任务设置为周期性任务
    h.task = &task.Periodic{
        # 设置任务执行间隔为配置中指定的刷新数值乘以一分钟的时间间隔
        Interval: time.Minute * time.Duration(h.receiverConfig.AllocationStrategy.GetRefreshValue()),
        # 设置任务执行函数为 h.refresh
        Execute:  h.refresh,
    }

    # 返回任务和空错误
    return h, nil
# 分配一个可用的端口
func (h *DynamicInboundHandler) allocatePort() net.Port {
    # 获取端口范围的起始值
    from := int(h.receiverConfig.PortRange.From)
    # 计算端口范围的长度
    delta := int(h.receiverConfig.PortRange.To) - from + 1

    # 加锁，确保并发安全
    h.portMutex.Lock()
    # 延迟解锁，确保在函数返回前解锁
    defer h.portMutex.Unlock()

    # 循环直到找到一个未被使用的端口
    for {
        # 生成一个随机数，用于选择端口
        r := dice.Roll(delta)
        # 计算选中的端口
        port := net.Port(from + r)
        # 检查端口是否已被使用
        _, used := h.portsInUse[port]
        # 如果端口未被使用，则将其标记为已使用并返回
        if !used {
            h.portsInUse[port] = true
            return port
        }
    }
}

# 关闭一组 worker
func (h *DynamicInboundHandler) closeWorkers(workers []worker) {
    # 创建一个用于存储需要删除的端口的切片
    ports2Del := make([]net.Port, len(workers))
    # 遍历 worker 切片
    for idx, worker := range workers {
        # 将 worker 的端口添加到需要删除的端口切片中
        ports2Del[idx] = worker.Port()
        # 如果关闭 worker 出现错误，则记录日志
        if err := worker.Close(); err != nil {
            newError("failed to close worker").Base(err).WriteToLog()
        }
    }

    # 加锁，确保并发安全
    h.portMutex.Lock()
    # 遍历需要删除的端口切片，从已使用的端口中删除
    for _, port := range ports2Del {
        delete(h.portsInUse, port)
    }
    # 解锁
    h.portMutex.Unlock()
}

# 刷新处理程序状态
func (h *DynamicInboundHandler) refresh() error {
    # 更新最后刷新时间
    h.lastRefresh = time.Now()

    # 计算超时时间和并发值
    timeout := time.Minute * time.Duration(h.receiverConfig.AllocationStrategy.GetRefreshValue()) * 2
    concurrency := h.receiverConfig.AllocationStrategy.GetConcurrencyValue()
    # 创建一个 worker 切片
    workers := make([]worker, 0, concurrency)

    # 获取监听地址
    address := h.receiverConfig.Listen.AsAddress()
    # 如果地址为空，则使用任意 IP 地址
    if address == nil {
        address = net.AnyIP
    }

    # 获取上行和下行流量统计计数器
    uplinkCounter, downlinkCounter := getStatCounter(h.v, h.tag)
}
    for i := uint32(0); i < concurrency; i++ {
        // 循环创建指定数量的代理实例
        port := h.allocatePort()
        // 分配端口号
        rawProxy, err := core.CreateObject(h.v, h.proxyConfig)
        // 创建代理对象
        if err != nil {
            // 如果创建代理对象失败，则记录错误信息并继续下一次循环
            newError("failed to create proxy instance").Base(err).AtWarning().WriteToLog()
            continue
        }
        p := rawProxy.(proxy.Inbound)
        // 将代理对象转换为入站代理对象
        nl := p.Network()
        // 获取代理对象的网络类型
        if net.HasNetwork(nl, net.Network_TCP) {
            // 如果代理对象支持 TCP 网络，则创建 TCP 工作器
            worker := &tcpWorker{
                tag:             h.tag,
                address:         address,
                port:            port,
                proxy:           p,
                stream:          h.streamSettings,
                recvOrigDest:    h.receiverConfig.ReceiveOriginalDestination,
                dispatcher:      h.mux,
                sniffingConfig:  h.receiverConfig.GetEffectiveSniffingSettings(),
                uplinkCounter:   uplinkCounter,
                downlinkCounter: downlinkCounter,
                ctx:             h.ctx,
            }
            if err := worker.Start(); err != nil {
                // 如果创建 TCP 工作器失败，则记录错误信息并继续下一次循环
                newError("failed to create TCP worker").Base(err).AtWarning().WriteToLog()
                continue
            }
            workers = append(workers, worker)
        }

        if net.HasNetwork(nl, net.Network_UDP) {
            // 如果代理对象支持 UDP 网络，则创建 UDP 工作器
            worker := &udpWorker{
                tag:             h.tag,
                proxy:           p,
                address:         address,
                port:            port,
                dispatcher:      h.mux,
                uplinkCounter:   uplinkCounter,
                downlinkCounter: downlinkCounter,
                stream:          h.streamSettings,
            }
            if err := worker.Start(); err != nil {
                // 如果创建 UDP 工作器失败，则记录错误信息并继续下一次循环
                newError("failed to create UDP worker").Base(err).AtWarning().WriteToLog()
                continue
            }
            workers = append(workers, worker)
        }
    }

    h.workerMutex.Lock()
    // 加锁以确保对 workers 的操作是线程安全的
    h.worker = workers
    // 更新 worker 列表
    h.workerMutex.Unlock()
    // 解锁
    # 在指定的超时时间后执行给定的函数
    time.AfterFunc(timeout, func() {
        # 调用 h 对象的 closeWorkers 方法，关闭指定的 workers
        h.closeWorkers(workers)
    })

    # 返回空值
    return nil
# 关闭 DynamicInboundHandler 对象
}

# 启动 DynamicInboundHandler 对象的任务
func (h *DynamicInboundHandler) Start() error {
    return h.task.Start()
}

# 关闭 DynamicInboundHandler 对象的任务
func (h *DynamicInboundHandler) Close() error {
    return h.task.Close()
}

# 获取随机的入站代理
func (h *DynamicInboundHandler) GetRandomInboundProxy() (interface{}, net.Port, int) {
    # 获取读取锁
    h.workerMutex.RLock()
    # 在函数返回时释放读取锁
    defer h.workerMutex.RUnlock()

    # 如果 worker 数组为空，则返回默认值
    if len(h.worker) == 0 {
        return nil, 0, 0
    }
    # 从 worker 数组中随机选择一个元素
    w := h.worker[dice.Roll(len(h.worker))]
    # 计算代理过期时间
    expire := h.receiverConfig.AllocationStrategy.GetRefreshValue() - uint32(time.Since(h.lastRefresh)/time.Minute)
    return w.Proxy(), w.Port(), int(expire)
}

# 获取 DynamicInboundHandler 对象的标签
func (h *DynamicInboundHandler) Tag() string {
    return h.tag
}
```