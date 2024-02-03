# `v2ray-core\app\dispatcher\default.go`

```go
// +build !confonly
// 定义了构建标签，表示不仅仅是配置文件

package dispatcher
// 包名为 dispatcher，用于分发任务

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用go:generate命令自动生成错误处理代码

import (
    "context"
    "strings"
    "sync"
    "time"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/log"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/session"
    "v2ray.com/core/features/outbound"
    "v2ray.com/core/features/policy"
    "v2ray.com/core/features/routing"
    routing_session "v2ray.com/core/features/routing/session"
    "v2ray.com/core/features/stats"
    "v2ray.com/core/transport"
    "v2ray.com/core/transport/pipe"
)
// 导入所需的包

var (
    errSniffingTimeout = newError("timeout on sniffing")
)
// 定义全局变量 errSniffingTimeout

type cachedReader struct {
    sync.Mutex
    reader *pipe.Reader
    cache  buf.MultiBuffer
}
// 定义结构体 cachedReader，包含互斥锁、管道读取器和缓存多缓冲区

func (r *cachedReader) Cache(b *buf.Buffer) {
    mb, _ := r.reader.ReadMultiBufferTimeout(time.Millisecond * 100)
    r.Lock()
    if !mb.IsEmpty() {
        r.cache, _ = buf.MergeMulti(r.cache, mb)
    }
    b.Clear()
    rawBytes := b.Extend(buf.Size)
    n := r.cache.Copy(rawBytes)
    b.Resize(0, int32(n))
    r.Unlock()
}
// 定义方法 Cache，用于缓存数据

func (r *cachedReader) readInternal() buf.MultiBuffer {
    r.Lock()
    defer r.Unlock()

    if r.cache != nil && !r.cache.IsEmpty() {
        mb := r.cache
        r.cache = nil
        return mb
    }

    return nil
}
// 定义方法 readInternal，用于内部读取数据

func (r *cachedReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    mb := r.readInternal()
    if mb != nil {
        return mb, nil
    }

    return r.reader.ReadMultiBuffer()
}
// 定义方法 ReadMultiBuffer，用于读取多缓冲区数据

func (r *cachedReader) ReadMultiBufferTimeout(timeout time.Duration) (buf.MultiBuffer, error) {
    mb := r.readInternal()
    if mb != nil {
        return mb, nil
    }

    return r.reader.ReadMultiBufferTimeout(timeout)
}
// 定义方法 ReadMultiBufferTimeout，用于读取多缓冲区数据并设置超时

func (r *cachedReader) Interrupt() {
    r.Lock()
    if r.cache != nil {
        r.cache = buf.ReleaseMulti(r.cache)
    }
    r.Unlock()
    r.reader.Interrupt()
}
// 定义方法 Interrupt，用于中断操作
// DefaultDispatcher 是 Dispatcher 的默认实现
type DefaultDispatcher struct {
    ohm    outbound.Manager
    router routing.Router
    policy policy.Manager
    stats  stats.Manager
}

func init() {
    // 注册配置并初始化 DefaultDispatcher
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        d := new(DefaultDispatcher)
        // 要求必要的特性，并初始化 DefaultDispatcher
        if err := core.RequireFeatures(ctx, func(om outbound.Manager, router routing.Router, pm policy.Manager, sm stats.Manager) error {
            return d.Init(config.(*Config), om, router, pm, sm)
        }); err != nil {
            return nil, err
        }
        return d, nil
    }))
}

// Init 初始化 DefaultDispatcher
func (d *DefaultDispatcher) Init(config *Config, om outbound.Manager, router routing.Router, pm policy.Manager, sm stats.Manager) error {
    d.ohm = om
    d.router = router
    d.policy = pm
    d.stats = sm
    return nil
}

// Type 实现 common.HasType
func (*DefaultDispatcher) Type() interface{} {
    return routing.DispatcherType()
}

// Start 实现 common.Runnable
func (*DefaultDispatcher) Start() error {
    return nil
}

// Close 实现 common.Closable
func (*DefaultDispatcher) Close() error { return nil }

func (d *DefaultDispatcher) getLink(ctx context.Context) (*transport.Link, *transport.Link) {
    opt := pipe.OptionsFromContext(ctx)
    uplinkReader, uplinkWriter := pipe.New(opt...)
    downlinkReader, downlinkWriter := pipe.New(opt...)

    inboundLink := &transport.Link{
        Reader: downlinkReader,
        Writer: uplinkWriter,
    }

    outboundLink := &transport.Link{
        Reader: uplinkReader,
        Writer: downlinkWriter,
    }

    sessionInbound := session.InboundFromContext(ctx)
    var user *protocol.MemoryUser
    if sessionInbound != nil {
        user = sessionInbound.User
    }
}
    # 检查用户是否存在且邮箱长度大于0
    if user != nil && len(user.Email) > 0:
        # 根据用户级别获取对应的策略
        p := d.policy.ForLevel(user.Level)
        # 如果策略允许统计用户上行流量
        if p.Stats.UserUplink:
            # 构建上行流量统计名称
            name := "user>>>" + user.Email + ">>>traffic>>>uplink"
            # 获取或注册计数器，并将其赋值给c
            if c, _ := stats.GetOrRegisterCounter(d.stats, name); c != nil:
                # 将上行流量统计器作为参数传递给SizeStatWriter，同时保持原始的inboundLink.Writer
                inboundLink.Writer = &SizeStatWriter{
                    Counter: c,
                    Writer:  inboundLink.Writer,
                }
        # 如果策略允许统计用户下行流量
        if p.Stats.UserDownlink:
            # 构建下行流量统计名称
            name := "user>>>" + user.Email + ">>>traffic>>>downlink"
            # 获取或注册计数器，并将其赋值给c
            if c, _ := stats.GetOrRegisterCounter(d.stats, name); c != nil:
                # 将下行流量统计器作为参数传递给SizeStatWriter，同时保持原始的outboundLink.Writer
                outboundLink.Writer = &SizeStatWriter{
                    Counter: c,
                    Writer:  outboundLink.Writer,
                }
    # 返回统计后的inboundLink和outboundLink
    return inboundLink, outboundLink
// 判断是否需要覆盖目标地址，根据给定的域名列表进行判断
func shouldOverride(result SniffResult, domainOverride []string) bool {
    // 遍历域名覆盖列表，如果协议以列表中的某个元素开头，则返回 true
    for _, p := range domainOverride {
        if strings.HasPrefix(result.Protocol(), p) {
            return true
        }
    }
    // 如果都不匹配，则返回 false
    return false
}

// Dispatch 实现了 routing.Dispatcher 接口
func (d *DefaultDispatcher) Dispatch(ctx context.Context, destination net.Destination) (*transport.Link, error) {
    // 如果目标地址无效，则触发 panic
    if !destination.IsValid() {
        panic("Dispatcher: Invalid destination.")
    }
    // 创建出站会话对象
    ob := &session.Outbound{
        Target: destination,
    }
    // 将出站会话对象添加到上下文中
    ctx = session.ContextWithOutbound(ctx, ob)

    // 获取入站和出站链接
    inbound, outbound := d.getLink(ctx)
    // 从上下文中获取内容
    content := session.ContentFromContext(ctx)
    // 如果内容为空，则创建一个新的内容对象，并添加到上下文中
    if content == nil {
        content = new(session.Content)
        ctx = session.ContextWithContent(ctx, content)
    }
    // 从内容中获取嗅探请求
    sniffingRequest := content.SniffingRequest
    // 如果目标网络不是 TCP，或者嗅探请求未启用，则进行路由分发
    if destination.Network != net.Network_TCP || !sniffingRequest.Enabled {
        // 在新的 goroutine 中进行路由分发
        go d.routedDispatch(ctx, outbound, destination)
    } else {
        // 如果是 TCP 网络且嗅探请求已启用，则进行嗅探处理
        go func() {
            // 创建缓存读取器
            cReader := &cachedReader{
                reader: outbound.Reader.(*pipe.Reader),
            }
            // 将出站读取器替换为缓存读取器
            outbound.Reader = cReader
            // 进行嗅探操作，获取嗅探结果和可能的错误
            result, err := sniffer(ctx, cReader)
            // 如果没有错误，则将协议信息添加到内容中
            if err == nil {
                content.Protocol = result.Protocol()
            }
            // 如果没有错误且需要覆盖目标地址，则进行目标地址的覆盖操作
            if err == nil && shouldOverride(result, sniffingRequest.OverrideDestinationForProtocol) {
                domain := result.Domain()
                newError("sniffed domain: ", domain).WriteToLog(session.ExportIDToError(ctx))
                destination.Address = net.ParseAddress(domain)
                ob.Target = destination
            }
            // 进行路由分发
            d.routedDispatch(ctx, outbound, destination)
        }()
    }
    // 返回入站链接和 nil 错误
    return inbound, nil
}

// 进行嗅探操作，返回嗅探结果和可能的错误
func sniffer(ctx context.Context, cReader *cachedReader) (SniffResult, error) {
    // 创建新的缓冲区
    payload := buf.New()
    // 在函数返回时释放缓冲区
    defer payload.Release()

    // 创建嗅探器对象
    sniffer := NewSniffer()
    // 初始化尝试次数
    totalAttempt := 0
    # 无限循环，等待从通道中接收数据
    for {
        # 选择不同的 case 进行处理
        select {
            # 如果上下文被取消，则返回错误
            case <-ctx.Done():
                return nil, ctx.Err()
            # 默认情况下
            default:
                # 增加尝试次数
                totalAttempt++
                # 如果尝试次数超过2次，则返回超时错误
                if totalAttempt > 2:
                    return nil, errSniffingTimeout

                # 将数据缓存起来
                cReader.Cache(payload)
                # 如果数据不为空
                if !payload.IsEmpty():
                    # 对数据进行嗅探
                    result, err := sniffer.Sniff(payload.Bytes())
                    # 如果出现错误
                    if err != common.ErrNoClue:
                        # 返回嗅探结果和错误
                        return result, err
                # 如果数据已满
                if payload.IsFull():
                    # 返回未知内容错误
                    return nil, errUnknownContent
        }
    }
// routedDispatch 方法用于根据目标地址进行路由分发
func (d *DefaultDispatcher) routedDispatch(ctx context.Context, link *transport.Link, destination net.Destination) {
    var handler outbound.Handler

    // 检查是否需要跳过路由选择
    skipRoutePick := false
    if content := session.ContentFromContext(ctx); content != nil {
        skipRoutePick = content.SkipRoutePick
    }

    // 如果存在路由器并且不需要跳过路由选择
    if d.router != nil && !skipRoutePick {
        // 从路由器中选择路由
        if route, err := d.router.PickRoute(routing_session.AsRoutingContext(ctx)); err == nil {
            // 获取选择的路由的出站标签
            tag := route.GetOutboundTag()
            // 根据标签获取对应的出站处理程序
            if h := d.ohm.GetHandler(tag); h != nil {
                // 如果存在对应的出站处理程序，则使用该处理程序进行分发
                newError("taking detour [", tag, "] for [", destination, "]").WriteToLog(session.ExportIDToError(ctx))
                handler = h
            } else {
                // 如果不存在对应的出站处理程序，则记录警告日志
                newError("non existing tag: ", tag).AtWarning().WriteToLog(session.ExportIDToError(ctx))
            }
        } else {
            // 如果无法选择路由，则记录默认路由日志
            newError("default route for ", destination).WriteToLog(session.ExportIDToError(ctx))
        }
    }

    // 如果没有找到对应的出站处理程序，则使用默认的出站处理程序
    if handler == nil {
        handler = d.ohm.GetDefaultHandler()
    }

    // 如果仍然没有找到出站处理程序，则记录错误日志并关闭连接
    if handler == nil {
        newError("default outbound handler not exist").WriteToLog(session.ExportIDToError(ctx))
        common.Close(link.Writer)
        common.Interrupt(link.Reader)
        return
    }

    // 如果存在访问消息，则记录访问消息中的出站标签，并记录访问日志
    if accessMessage := log.AccessMessageFromContext(ctx); accessMessage != nil {
        if tag := handler.Tag(); tag != "" {
            accessMessage.Detour = tag
        }
        log.Record(accessMessage)
    }

    // 使用选择的出站处理程序进行分发
    handler.Dispatch(ctx, link)
}
```