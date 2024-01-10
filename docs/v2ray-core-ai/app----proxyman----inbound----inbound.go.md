# `v2ray-core\app\proxyman\inbound\inbound.go`

```
package inbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "sync"

    "v2ray.com/core"
    "v2ray.com/core/app/proxyman"
    "v2ray.com/core/common"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/common/session"
    "v2ray.com/core/features/inbound"
)

// Manager is to manage all inbound handlers.
type Manager struct {
    access          sync.RWMutex        // 读写锁，用于保护下面的字段
    untaggedHandler []inbound.Handler   // 未标记的处理程序列表
    taggedHandlers  map[string]inbound.Handler  // 标记的处理程序映射
    running         bool                // 标记管理器是否正在运行
}

// New returns a new Manager for inbound handlers.
func New(ctx context.Context, config *proxyman.InboundConfig) (*Manager, error) {
    m := &Manager{
        taggedHandlers: make(map[string]inbound.Handler),  // 初始化标记的处理程序映射
    }
    return m, nil
}

// Type implements common.HasType.
func (*Manager) Type() interface{} {
    return inbound.ManagerType()  // 返回管理器的类型
}

// AddHandler implements inbound.Manager.
func (m *Manager) AddHandler(ctx context.Context, handler inbound.Handler) error {
    m.access.Lock()  // 加锁
    defer m.access.Unlock()  // 在函数返回前解锁

    tag := handler.Tag()  // 获取处理程序的标记
    if len(tag) > 0 {
        m.taggedHandlers[tag] = handler  // 将标记的处理程序加入映射
    } else {
        m.untaggedHandler = append(m.untaggedHandler, handler)  // 将未标记的处理程序加入列表
    }

    if m.running {
        return handler.Start()  // 如果管理器正在运行，则启动处理程序
    }

    return nil
}

// GetHandler implements inbound.Manager.
func (m *Manager) GetHandler(ctx context.Context, tag string) (inbound.Handler, error) {
    m.access.RLock()  // 加读锁
    defer m.access.RUnlock()  // 在函数返回前解读锁

    handler, found := m.taggedHandlers[tag]  // 从映射中获取标记对应的处理程序
    if !found {
        return nil, newError("handler not found: ", tag)  // 如果未找到处理程序，则返回错误
    }
    return handler, nil
}

// RemoveHandler implements inbound.Manager.
func (m *Manager) RemoveHandler(ctx context.Context, tag string) error {
    if tag == "" {
        return common.ErrNoClue  // 如果标记为空，则返回错误
    }

    m.access.Lock()  // 加锁
    defer m.access.Unlock()  // 在函数返回前解锁
    # 如果存在以指定标签为键的处理程序，则执行以下操作
    if handler, found := m.taggedHandlers[tag]; found:
        # 如果关闭处理程序时发生错误，则记录错误信息并返回警告
        if err := handler.Close(); err != nil:
            newError("failed to close handler ", tag).Base(err).AtWarning().WriteToLog(session.ExportIDToError(ctx))
        # 从处理程序映射中删除指定标签的处理程序
        delete(m.taggedHandlers, tag)
        # 返回空值
        return nil

    # 如果不存在以指定标签为键的处理程序，则返回常见的错误信息
    return common.ErrNoClue
// Start 方法实现了 common.Runnable 接口
func (m *Manager) Start() error {
    // 获取锁
    m.access.Lock()
    // 在函数返回时释放锁
    defer m.access.Unlock()

    // 设置运行状态为 true
    m.running = true

    // 遍历标记处理程序并启动
    for _, handler := range m.taggedHandlers {
        if err := handler.Start(); err != nil {
            return err
        }
    }

    // 遍历未标记处理程序并启动
    for _, handler := range m.untaggedHandler {
        if err := handler.Start(); err != nil {
            return err
        }
    }
    return nil
}

// Close 方法实现了 common.Closable 接口
func (m *Manager) Close() error {
    // 获取锁
    m.access.Lock()
    // 在函数返回时释放锁
    defer m.access.Unlock()

    // 设置运行状态为 false
    m.running = false

    // 存储关闭处理程序时的错误
    var errors []interface{}
    for _, handler := range m.taggedHandlers {
        if err := handler.Close(); err != nil {
            errors = append(errors, err)
        }
    }
    for _, handler := range m.untaggedHandler {
        if err := handler.Close(); err != nil {
            errors = append(errors, err)
        }
    }

    // 如果有错误发生，则返回包含错误信息的错误
    if len(errors) > 0 {
        return newError("failed to close all handlers").Base(newError(serial.Concat(errors...)))
    }

    return nil
}

// NewHandler 根据给定的配置创建一个新的入站处理程序
func NewHandler(ctx context.Context, config *core.InboundHandlerConfig) (inbound.Handler, error) {
    // 获取接收器设置实例
    rawReceiverSettings, err := config.ReceiverSettings.GetInstance()
    if err != nil {
        return nil, err
    }
    // 获取代理设置实例
    proxySettings, err := config.ProxySettings.GetInstance()
    if err != nil {
        return nil, err
    }
    // 获取标签
    tag := config.Tag

    // 将接收器设置转换为代理管理器接收器配置
    receiverSettings, ok := rawReceiverSettings.(*proxyman.ReceiverConfig)
    if !ok {
        return nil, newError("not a ReceiverConfig").AtError()
    }

    // 获取流设置并根据需要设置套接字选项
    streamSettings := receiverSettings.StreamSettings
    if streamSettings != nil && streamSettings.SocketSettings != nil {
        ctx = session.ContextWithSockopt(ctx, &session.Sockopt{
            Mark: streamSettings.SocketSettings.Mark,
        })
    }

    // 获取分配策略
    allocStrategy := receiverSettings.AllocationStrategy
}
    # 如果分配策略为空或者类型为Always，则返回一个新的AlwaysOnInboundHandler对象
    if allocStrategy == nil || allocStrategy.Type == proxyman.AllocationStrategy_Always {
        return NewAlwaysOnInboundHandler(ctx, tag, receiverSettings, proxySettings)
    }

    # 如果分配策略类型为Random，则返回一个新的DynamicInboundHandler对象
    if allocStrategy.Type == proxyman.AllocationStrategy_Random {
        return NewDynamicInboundHandler(ctx, tag, receiverSettings, proxySettings)
    }
    # 如果分配策略类型未知，则返回空和一个新的错误对象
    return nil, newError("unknown allocation strategy: ", receiverSettings.AllocationStrategy.Type).AtError()
# 初始化函数，用于注册配置和处理函数
func init() {
    # 注册传入连接的配置和处理函数
    common.Must(common.RegisterConfig((*proxyman.InboundConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*proxyman.InboundConfig))
    }))
    # 注册传入连接的处理器配置和处理函数
    common.Must(common.RegisterConfig((*core.InboundHandlerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewHandler(ctx, config.(*core.InboundHandlerConfig))
    }))
}
```