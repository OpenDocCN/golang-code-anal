# `v2ray-core\app\reverse\portal.go`

```
// +build !confonly

package reverse

import (
    "context"  // 引入上下文包，用于处理请求的上下文信息
    "sync"  // 引入同步包，用于实现并发安全的数据访问
    "time"  // 引入时间包，用于处理时间相关的操作

    "github.com/golang/protobuf/proto"  // 引入protobuf包，用于处理protobuf相关的操作
    "v2ray.com/core/common"  // 引入v2ray的common包
    "v2ray.com/core/common/buf"  // 引入v2ray的buf包，用于处理数据缓冲区
    "v2ray.com/core/common/mux"  // 引入v2ray的mux包，用于处理多路复用
    "v2ray.com/core/common/net"  // 引入v2ray的net包，用于处理网络相关的操作
    "v2ray.com/core/common/session"  // 引入v2ray的session包，用于处理会话相关的操作
    "v2ray.com/core/common/task"  // 引入v2ray的task包，用于处理任务相关的操作
    "v2ray.com/core/features/outbound"  // 引入v2ray的outbound包，用于处理出站相关的操作
    "v2ray.com/core/transport"  // 引入v2ray的transport包，用于处理传输相关的操作
    "v2ray.com/core/transport/pipe"  // 引入v2ray的pipe包，用于处理管道相关的操作
)

type Portal struct {
    ohm    outbound.Manager  // 出站管理器
    tag    string  // 标签
    domain string  // 域名
    picker *StaticMuxPicker  // 静态多路复用选择器
    client *mux.ClientManager  // 客户端管理器
}

func NewPortal(config *PortalConfig, ohm outbound.Manager) (*Portal, error) {
    if config.Tag == "" {  // 如果配置的标签为空
        return nil, newError("portal tag is empty")  // 返回错误信息
    }

    if config.Domain == "" {  // 如果配置的域名为空
        return nil, newError("portal domain is empty")  // 返回错误信息
    }

    picker, err := NewStaticMuxPicker()  // 创建静态多路复用选择器
    if err != nil {  // 如果创建失败
        return nil, err  // 返回错误信息
    }

    return &Portal{  // 返回Portal对象
        ohm:    ohm,  // 设置出站管理器
        tag:    config.Tag,  // 设置标签
        domain: config.Domain,  // 设置域名
        picker: picker,  // 设置选择器
        client: &mux.ClientManager{  // 设置客户端管理器
            Picker: picker,  // 设置选择器
        },
    }, nil  // 返回nil错误
}

func (p *Portal) Start() error {
    return p.ohm.AddHandler(context.Background(), &Outbound{  // 添加处理器
        portal: p,  // 设置portal
        tag:    p.tag,  // 设置标签
    })
}

func (p *Portal) Close() error {
    return p.ohm.RemoveHandler(context.Background(), p.tag)  // 移除处理器
}

func (p *Portal) HandleConnection(ctx context.Context, link *transport.Link) error {
    outboundMeta := session.OutboundFromContext(ctx)  // 从上下文中获取出站元数据
    if outboundMeta == nil {  // 如果出站元数据为空
        return newError("outbound metadata not found").AtError()  // 返回错误信息
    }
    # 如果目标地址是域名，则执行以下操作
    if isDomain(outboundMeta.Target, p.domain) {
        # 创建一个新的 Mux 客户端工作器
        muxClient, err := mux.NewClientWorker(*link, mux.ClientStrategy{})
        # 如果创建失败，则返回错误
        if err != nil {
            return newError("failed to create mux client worker").Base(err).AtWarning()
        }

        # 创建一个新的 Portal 工作器
        worker, err := NewPortalWorker(muxClient)
        # 如果创建失败，则返回错误
        if err != nil {
            return newError("failed to create portal worker").Base(err)
        }

        # 将工作器添加到选择器中
        p.picker.AddWorker(worker)
        # 返回空值
        return nil
    }

    # 如果目标地址不是域名，则将请求分发给客户端
    return p.client.Dispatch(ctx, link)
// 定义 Outbound 结构体，包含指向 Portal 的指针和标签字符串
type Outbound struct {
    portal *Portal
    tag    string
}

// 返回 Outbound 结构体的标签字符串
func (o *Outbound) Tag() string {
    return o.tag
}

// 处理传出连接，调用 portal 的 HandleConnection 方法处理连接，如果出错则中断连接的写入
func (o *Outbound) Dispatch(ctx context.Context, link *transport.Link) {
    if err := o.portal.HandleConnection(ctx, link); err != nil {
        newError("failed to process reverse connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
        common.Interrupt(link.Writer)
    }
}

// 开始 Outbound 结构体的操作，始终返回 nil
func (o *Outbound) Start() error {
    return nil
}

// 关闭 Outbound 结构体的操作，始终返回 nil
func (o *Outbound) Close() error {
    return nil
}

// 定义 StaticMuxPicker 结构体，包含互斥锁、PortalWorker 切片和定时任务指针
type StaticMuxPicker struct {
    access  sync.Mutex
    workers []*PortalWorker
    cTask   *task.Periodic
}

// 创建并返回 StaticMuxPicker 结构体的实例，同时启动定时任务
func NewStaticMuxPicker() (*StaticMuxPicker, error) {
    p := &StaticMuxPicker{}
    p.cTask = &task.Periodic{
        Execute:  p.cleanup,
        Interval: time.Second * 30,
    }
    p.cTask.Start()
    return p, nil
}

// 定时清理方法，清理已关闭的 PortalWorker
func (p *StaticMuxPicker) cleanup() error {
    p.access.Lock()
    defer p.access.Unlock()

    var activeWorkers []*PortalWorker
    for _, w := range p.workers {
        if !w.Closed() {
            activeWorkers = append(activeWorkers, w)
        }
    }

    if len(activeWorkers) != len(p.workers) {
        p.workers = activeWorkers
    }

    return nil
}

// 选择可用的 ClientWorker，根据条件选择最佳的 PortalWorker
func (p *StaticMuxPicker) PickAvailable() (*mux.ClientWorker, error) {
    p.access.Lock()
    defer p.access.Unlock()

    if len(p.workers) == 0 {
        return nil, newError("empty worker list")
    }

    var minIdx int = -1
    var minConn uint32 = 9999
    for i, w := range p.workers {
        if w.draining {
            continue
        }
        if w.client.ActiveConnections() < minConn {
            minConn = w.client.ActiveConnections()
            minIdx = i
        }
    }

    if minIdx == -1 {
        for i, w := range p.workers {
            if w.IsFull() {
                continue
            }
            if w.client.ActiveConnections() < minConn {
                minConn = w.client.ActiveConnections()
                minIdx = i
            }
        }
    }
    }
    # 如果找到最小负载的 worker 索引不为 -1
    if minIdx != -1 {
        # 返回该 worker 对应的客户端和空错误
        return p.workers[minIdx].client, nil
    }
    # 如果没有找到最小负载的 worker
    # 返回空值和新的错误信息
    return nil, newError("no mux client worker available")
}



func (p *StaticMuxPicker) AddWorker(worker *PortalWorker) {
    // 加锁，保证并发安全
    p.access.Lock()
    // 延迟解锁
    defer p.access.Unlock()

    // 将新的 worker 添加到 workers 列表中
    p.workers = append(p.workers, worker)
}

type PortalWorker struct {
    client   *mux.ClientWorker
    control  *task.Periodic
    writer   buf.Writer
    reader   buf.Reader
    draining bool
}

func NewPortalWorker(client *mux.ClientWorker) (*PortalWorker, error) {
    // 设置管道选项
    opt := []pipe.Option{pipe.WithSizeLimit(16 * 1024)}
    // 创建上行和下行管道
    uplinkReader, uplinkWriter := pipe.New(opt...)
    downlinkReader, downlinkWriter := pipe.New(opt...)

    // 创建上下文
    ctx := context.Background()
    // 设置上下文的出站连接信息
    ctx = session.ContextWithOutbound(ctx, &session.Outbound{
        Target: net.UDPDestination(net.DomainAddress(internalDomain), 0),
    })
    // 分发控制连接
    f := client.Dispatch(ctx, &transport.Link{
        Reader: uplinkReader,
        Writer: downlinkWriter,
    })
    // 如果分发失败，则返回错误
    if !f {
        return nil, newError("unable to dispatch control connection")
    }
    // 创建 PortalWorker 对象
    w := &PortalWorker{
        client: client,
        reader: downlinkReader,
        writer: uplinkWriter,
    }
    // 创建周期性任务
    w.control = &task.Periodic{
        Execute:  w.heartbeat,
        Interval: time.Second * 2,
    }
    // 启动周期性任务
    w.control.Start()
    return w, nil
}

func (w *PortalWorker) heartbeat() error {
    // 如果客户端连接已关闭，则返回错误
    if w.client.Closed() {
        return newError("client worker stopped")
    }

    // 如果正在排空或写入器为空，则返回错误
    if w.draining || w.writer == nil {
        return newError("already disposed")
    }

    // 创建控制消息
    msg := &Control{}
    msg.FillInRandom()

    // 如果客户端总连接数大于 256，则设置排空标志
    if w.client.TotalConnections() > 256 {
        w.draining = true
        msg.State = Control_DRAIN

        // 延迟执行，关闭写入器和中断读取器
        defer func() {
            common.Close(w.writer)
            common.Interrupt(w.reader)
            w.writer = nil
        }()
    }

    // 将消息序列化为字节流
    b, err := proto.Marshal(msg)
    common.Must(err)
    mb := buf.MergeBytes(nil, b)
    // 写入消息到写入器
    return w.writer.WriteMultiBuffer(mb)
}

func (w *PortalWorker) IsFull() bool {
    // 判断客户端是否已满
    return w.client.IsFull()
}

func (w *PortalWorker) Closed() bool {
    // 判断客户端是否已关闭
    return w.client.Closed()
}
```