# `v2ray-core\app\proxyman\outbound\outbound.go`

```go
package outbound

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "strings"
    "sync"

    "v2ray.com/core"
    "v2ray.com/core/app/proxyman"
    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/features/outbound"
)

// Manager is to manage all outbound handlers.
type Manager struct {
    access           sync.RWMutex      // 用于控制并发访问的读写锁
    defaultHandler   outbound.Handler  // 默认的出站处理器
    taggedHandler    map[string]outbound.Handler  // 标记的出站处理器
    untaggedHandlers []outbound.Handler  // 未标记的出站处理器
    running          bool  // 标记管理器是否正在运行
}

// New creates a new Manager.
func New(ctx context.Context, config *proxyman.OutboundConfig) (*Manager, error) {
    m := &Manager{
        taggedHandler: make(map[string]outbound.Handler),  // 初始化标记的出站处理器
    }
    return m, nil
}

// Type implements common.HasType.
func (m *Manager) Type() interface{} {
    return outbound.ManagerType()  // 返回出站管理器的类型
}

// Start implements core.Feature
func (m *Manager) Start() error {
    m.access.Lock()  // 获取写锁
    defer m.access.Unlock()  // 在函数返回前释放写锁

    m.running = true  // 标记管理器正在运行

    for _, h := range m.taggedHandler {  // 遍历标记的出站处理器
        if err := h.Start(); err != nil {  // 启动出站处理器，如果出错则返回错误
            return err
        }
    }

    for _, h := range m.untaggedHandlers {  // 遍历未标记的出站处理器
        if err := h.Start(); err != nil {  // 启动出站处理器，如果出错则返回错误
            return err
        }
    }

    return nil
}

// Close implements core.Feature
func (m *Manager) Close() error {
    m.access.Lock()  // 获取写锁
    defer m.access.Unlock()  // 在函数返回前释放写锁

    m.running = false  // 标记管理器停止运行

    var errs []error
    for _, h := range m.taggedHandler {  // 遍历标记的出站处理器
        errs = append(errs, h.Close())  // 关闭出站处理器，并将错误添加到errs切片中
    }

    for _, h := range m.untaggedHandlers {  // 遍历未标记的出站处理器
        errs = append(errs, h.Close())  // 关闭出站处理器，并将错误添加到errs切片中
    }

    return errors.Combine(errs...)  // 返回所有错误的组合
}

// GetDefaultHandler implements outbound.Manager.
func (m *Manager) GetDefaultHandler() outbound.Handler {
    m.access.RLock()  // 获取读锁
    defer m.access.RUnlock()  // 在函数返回前释放读锁

    if m.defaultHandler == nil {  // 如果默认出站处理器为空
        return nil
    }
    return m.defaultHandler  // 返回默认出站处理器
}

// GetHandler implements outbound.Manager.
func (m *Manager) GetHandler(tag string) outbound.Handler {
    # 获取读取锁，确保并发安全
    m.access.RLock()
    # 在函数返回时释放读取锁
    defer m.access.RUnlock()
    # 如果存在以tag为键的处理程序，则返回该处理程序
    if handler, found := m.taggedHandler[tag]; found {
        return handler
    }
    # 如果不存在以tag为键的处理程序，则返回空
    return nil
// AddHandler 实现了 outbound.Manager 接口，用于向管理器中添加处理程序
func (m *Manager) AddHandler(ctx context.Context, handler outbound.Handler) error {
    // 加锁以确保线程安全
    m.access.Lock()
    defer m.access.Unlock()

    // 如果默认处理程序为空，则将传入的处理程序设置为默认处理程序
    if m.defaultHandler == nil {
        m.defaultHandler = handler
    }

    // 获取处理程序的标签
    tag := handler.Tag()
    // 如果标签长度大于0，则将处理程序添加到标记处理程序映射中
    if len(tag) > 0 {
        m.taggedHandler[tag] = handler
    } else {
        // 否则将处理程序添加到未标记的处理程序列表中
        m.untaggedHandlers = append(m.untaggedHandlers, handler)
    }

    // 如果管理器正在运行，则启动处理程序
    if m.running {
        return handler.Start()
    }

    return nil
}

// RemoveHandler 实现了 outbound.Manager 接口，用于从管理器中移除处理程序
func (m *Manager) RemoveHandler(ctx context.Context, tag string) error {
    // 如果标签为空，则返回错误
    if tag == "" {
        return common.ErrNoClue
    }
    m.access.Lock()
    defer m.access.Unlock()

    // 从标记处理程序映射中删除指定标签的处理程序
    delete(m.taggedHandler, tag)
    // 如果默认处理程序不为空且其标签与指定标签相同，则将默认处理程序设置为空
    if m.defaultHandler != nil && m.defaultHandler.Tag() == tag {
        m.defaultHandler = nil
    }

    return nil
}

// Select 实现了 outbound.HandlerSelector 接口，用于选择处理程序
func (m *Manager) Select(selectors []string) []string {
    m.access.RLock()
    defer m.access.RUnlock()

    // 创建一个空的标签列表
    tags := make([]string, 0, len(selectors))

    // 遍历标记处理程序映射
    for tag := range m.taggedHandler {
        match := false
        // 遍历选择器列表，检查标签是否以选择器开头
        for _, selector := range selectors {
            if strings.HasPrefix(tag, selector) {
                match = true
                break
            }
        }
        // 如果匹配，则将标签添加到标签列表中
        if match {
            tags = append(tags, tag)
        }
    }

    return tags
}

// 初始化函数，用于注册配置和创建新的管理器和处理程序
func init() {
    common.Must(common.RegisterConfig((*proxyman.OutboundConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*proxyman.OutboundConfig))
    }))
    common.Must(common.RegisterConfig((*core.OutboundHandlerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return NewHandler(ctx, config.(*core.OutboundHandlerConfig))
    }))
}
```