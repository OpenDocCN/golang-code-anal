# `v2ray-core\app\reverse\bridge.go`

```
// +build !confonly

package reverse

import (
    "context"
    "time"

    "github.com/golang/protobuf/proto"  // 导入 proto 包
    "v2ray.com/core/common/mux"  // 导入 mux 包
    "v2ray.com/core/common/net"  // 导入 net 包
    "v2ray.com/core/common/session"  // 导入 session 包
    "v2ray.com/core/common/task"  // 导入 task 包
    "v2ray.com/core/features/routing"  // 导入 routing 包
    "v2ray.com/core/transport"  // 导入 transport 包
    "v2ray.com/core/transport/pipe"  // 导入 pipe 包
)

// Bridge is a component in reverse proxy, that relays connections from Portal to local address.
type Bridge struct {
    dispatcher  routing.Dispatcher  // 路由分发器
    tag         string  // 标签
    domain      string  // 域名
    workers     []*BridgeWorker  // BridgeWorker 切片
    monitorTask *task.Periodic  // 定时任务
}

// NewBridge creates a new Bridge instance.
func NewBridge(config *BridgeConfig, dispatcher routing.Dispatcher) (*Bridge, error) {
    if config.Tag == "" {  // 如果标签为空
        return nil, newError("bridge tag is empty")  // 返回错误信息
    }
    if config.Domain == "" {  // 如果域名为空
        return nil, newError("bridge domain is empty")  // 返回错误信息
    }

    b := &Bridge{  // 创建 Bridge 实例
        dispatcher: dispatcher,  // 设置分发器
        tag:        config.Tag,  // 设置标签
        domain:     config.Domain,  // 设置域名
    }
    b.monitorTask = &task.Periodic{  // 创建定时任务
        Execute:  b.monitor,  // 执行监控方法
        Interval: time.Second * 2,  // 设置执行间隔
    }
    return b, nil  // 返回 Bridge 实例和空错误
}

func (b *Bridge) cleanup() {  // 清理方法
    var activeWorkers []*BridgeWorker  // 活跃的 BridgeWorker 切片

    for _, w := range b.workers {  // 遍历 workers
        if w.IsActive() {  // 如果活跃
            activeWorkers = append(activeWorkers, w)  // 添加到活跃的 workers 中
        }
    }

    if len(activeWorkers) != len(b.workers) {  // 如果活跃的 workers 数量不等于原始 workers 数量
        b.workers = activeWorkers  // 更新 workers
    }
}

func (b *Bridge) monitor() error {  // 监控方法
    b.cleanup()  // 调用清理方法

    var numConnections uint32  // 连接数量
    var numWorker uint32  // worker 数量

    for _, w := range b.workers {  // 遍历 workers
        if w.IsActive() {  // 如果活跃
            numConnections += w.Connections()  // 增加连接数量
            numWorker++  // 增加 worker 数量
        }
    }
    # 如果工作人数为0或者每个工作人员的连接数大于16，则执行以下操作
    if numWorker == 0 || numConnections/numWorker > 16 {
        # 创建一个新的桥接工作人员，并传入域名、标签和调度器
        worker, err := NewBridgeWorker(b.domain, b.tag, b.dispatcher)
        # 如果创建工作人员时出现错误，则记录错误信息并返回空
        if err != nil {
            newError("failed to create bridge worker").Base(err).AtWarning().WriteToLog()
            return nil
        }
        # 将新创建的工作人员添加到工作人员列表中
        b.workers = append(b.workers, worker)
    }

    # 返回空
    return nil
// 关闭 Bridge 对象
func (b *Bridge) Close() error {
    // 调用 monitorTask 的 Close 方法关闭任务
    return b.monitorTask.Close()
}

// BridgeWorker 结构体
type BridgeWorker struct {
    tag        string
    worker     *mux.ServerWorker
    dispatcher routing.Dispatcher
    state      Control_State
}

// 创建新的 BridgeWorker 对象
func NewBridgeWorker(domain string, tag string, d routing.Dispatcher) (*BridgeWorker, error) {
    // 创建一个新的上下文
    ctx := context.Background()
    // 将 tag 添加到上下文中的入站信息中
    ctx = session.ContextWithInbound(ctx, &session.Inbound{
        Tag: tag,
    })
    // 使用调度器分发网络目标
    link, err := d.Dispatch(ctx, net.Destination{
        Network: net.Network_TCP,
        Address: net.DomainAddress(domain),
        Port:    0,
    })
    if err != nil {
        return nil, err
    }

    // 创建新的 BridgeWorker 对象
    w := &BridgeWorker{
        dispatcher: d,
        tag:        tag,
    }

    // 使用 link 创建新的服务器工作对象
    worker, err := mux.NewServerWorker(context.Background(), w, link)
    if err != nil {
        return nil, err
    }
    w.worker = worker

    return w, nil
}

// 返回 BridgeWorker 对象的类型
func (w *BridgeWorker) Type() interface{} {
    return routing.DispatcherType()
}

// 启动 BridgeWorker 对象
func (w *BridgeWorker) Start() error {
    return nil
}

// 关闭 BridgeWorker 对象
func (w *BridgeWorker) Close() error {
    return nil
}

// 检查 BridgeWorker 对象是否处于活动状态
func (w *BridgeWorker) IsActive() bool {
    return w.state == Control_ACTIVE && !w.worker.Closed()
}

// 返回 BridgeWorker 对象的连接数
func (w *BridgeWorker) Connections() uint32 {
    return w.worker.ActiveConnections()
}

// 处理内部连接
func (w *BridgeWorker) handleInternalConn(link transport.Link) {
    go func() {
        reader := link.Reader
        for {
            // 读取多缓冲区数据
            mb, err := reader.ReadMultiBuffer()
            if err != nil {
                break
            }
            for _, b := range mb {
                var ctl Control
                // 解析协议消息
                if err := proto.Unmarshal(b.Bytes(), &ctl); err != nil {
                    newError("failed to parse proto message").Base(err).WriteToLog()
                    break
                }
                // 更新状态
                if ctl.State != w.state {
                    w.state = ctl.State
                }
            }
        }
    }()
}
func (w *BridgeWorker) Dispatch(ctx context.Context, dest net.Destination) (*transport.Link, error) {
    // 如果目标不是内部域名，则将传入上下文与入站连接的标签关联
    if !isInternalDomain(dest) {
        ctx = session.ContextWithInbound(ctx, &session.Inbound{
            Tag: w.tag,
        })
        // 调用调度器的 Dispatch 方法
        return w.dispatcher.Dispatch(ctx, dest)
    }

    // 设置管道选项，限制大小为16 * 1024
    opt := []pipe.Option{pipe.WithSizeLimit(16 * 1024)}
    // 创建上行和下行管道
    uplinkReader, uplinkWriter := pipe.New(opt...)
    downlinkReader, downlinkWriter := pipe.New(opt...)

    // 处理内部连接，传入上行和下行管道
    w.handleInternalConn(transport.Link{
        Reader: downlinkReader,
        Writer: uplinkWriter,
    })

    // 返回上行和下行管道的链接
    return &transport.Link{
        Reader: uplinkReader,
        Writer: downlinkWriter,
    }, nil
}
```