# `v2ray-core\app\stats\stats.go`

```
// +build !confonly

package stats

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "sync"

    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/features/stats"
)

// Manager is an implementation of stats.Manager.
type Manager struct {
    access   sync.RWMutex  // 读写锁，用于保护 counters 和 channels 的并发访问
    counters map[string]*Counter  // 计数器的映射，以计数器名为键，Counter 对象为值
    channels map[string]*Channel  // 通道的映射，以通道名为键，Channel 对象为值
    running  bool  // 标志位，表示 Manager 是否正在运行
}

// NewManager creates an instance of Statistics Manager.
func NewManager(ctx context.Context, config *Config) (*Manager, error) {
    m := &Manager{
        counters: make(map[string]*Counter),  // 初始化计数器映射
        channels: make(map[string]*Channel),  // 初始化通道映射
    }

    return m, nil  // 返回 Manager 对象
}

// Type implements common.HasType.
func (*Manager) Type() interface{} {
    return stats.ManagerType()  // 返回 stats.ManagerType() 的结果
}

// RegisterCounter implements stats.Manager.
func (m *Manager) RegisterCounter(name string) (stats.Counter, error) {
    m.access.Lock()  // 获取写锁
    defer m.access.Unlock()  // 在函数返回时释放写锁

    if _, found := m.counters[name]; found {  // 如果计数器映射中已存在该计数器名
        return nil, newError("Counter ", name, " already registered.")  // 返回错误信息
    }
    newError("create new counter ", name).AtDebug().WriteToLog()  // 记录调试日志
    c := new(Counter)  // 创建新的计数器对象
    m.counters[name] = c  // 将新计数器对象加入计数器映射
    return c, nil  // 返回新计数器对象
}

// UnregisterCounter implements stats.Manager.
func (m *Manager) UnregisterCounter(name string) error {
    m.access.Lock()  // 获取写锁
    defer m.access.Unlock()  // 在函数返回时释放写锁

    if _, found := m.counters[name]; found {  // 如果计数器映射中存在该计数器名
        newError("remove counter ", name).AtDebug().WriteToLog()  // 记录调试日志
        delete(m.counters, name)  // 从计数器映射中删除该计数器
    }
    return nil  // 返回空错误
}

// GetCounter implements stats.Manager.
func (m *Manager) GetCounter(name string) stats.Counter {
    m.access.RLock()  // 获取读锁
    defer m.access.RUnlock()  // 在函数返回时释放读锁

    if c, found := m.counters[name]; found {  // 如果计数器映射中存在该计数器名
        return c  // 返回该计数器对象
    }
    return nil  // 返回空
}

// VisitCounters calls visitor function on all managed counters.
func (m *Manager) VisitCounters(visitor func(string, stats.Counter) bool) {
    m.access.RLock()  // 获取读锁
    defer m.access.RUnlock()  // 在函数返回时释放读锁
    # 遍历 m.counters 中的每个元素，name 为键，c 为值
    for name, c := range m.counters:
        # 如果 visitor 函数返回 False，则跳出循环
        if !visitor(name, c):
            break
// RegisterChannel 实现了 stats.Manager 接口，用于注册新的通道
func (m *Manager) RegisterChannel(name string) (stats.Channel, error) {
    // 加锁，确保并发安全
    m.access.Lock()
    // 在函数返回时解锁
    defer m.access.Unlock()

    // 检查通道是否已经注册过
    if _, found := m.channels[name]; found {
        // 如果已经注册过，则返回错误
        return nil, newError("Channel ", name, " already registered.")
    }
    // 创建新的通道并添加到管理器中
    newError("create new channel ", name).AtDebug().WriteToLog()
    c := NewChannel(&ChannelConfig{BufferSize: 64, Blocking: false})
    m.channels[name] = c
    // 如果管理器正在运行，则启动新创建的通道
    if m.running {
        return c, c.Start()
    }
    return c, nil
}

// UnregisterChannel 实现了 stats.Manager 接口，用于注销已注册的通道
func (m *Manager) UnregisterChannel(name string) error {
    m.access.Lock()
    defer m.access.Unlock()

    // 检查通道是否存在，如果存在则移除
    if c, found := m.channels[name]; found {
        newError("remove channel ", name).AtDebug().WriteToLog()
        delete(m.channels, name)
        return c.Close()
    }
    return nil
}

// GetChannel 实现了 stats.Manager 接口，用于获取已注册的通道
func (m *Manager) GetChannel(name string) stats.Channel {
    m.access.RLock()
    defer m.access.RUnlock()

    // 根据通道名称获取已注册的通道
    if c, found := m.channels[name]; found {
        return c
    }
    return nil
}

// Start 实现了 common.Runnable 接口，用于启动管理器
func (m *Manager) Start() error {
    m.access.Lock()
    defer m.access.Unlock()
    m.running = true
    errs := []error{}
    // 遍历所有通道并启动，记录启动过程中的错误
    for _, channel := range m.channels {
        if err := channel.Start(); err != nil {
            errs = append(errs, err)
        }
    }
    // 如果有错误发生，则返回合并后的错误
    if len(errs) != 0 {
        return errors.Combine(errs...)
    }
    return nil
}

// Close 实现了 common.Closable 接口，用于关闭管理器
func (m *Manager) Close() error {
    m.access.Lock()
    defer m.access.Unlock()
    m.running = false
    errs := []error{}
    // 遍历所有通道并关闭，记录关闭过程中的错误
    for name, channel := range m.channels {
        newError("remove channel ", name).AtDebug().WriteToLog()
        delete(m.channels, name)
        if err := channel.Close(); err != nil {
            errs = append(errs, err)
        }
    }
    // 如果有错误发生，则返回合并后的错误
    if len(errs) != 0 {
        return errors.Combine(errs...)
    }
    return nil
}
# 在程序初始化时执行的函数
func init() {
    # 注册配置信息，当配置信息改变时，执行回调函数返回新的配置信息
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        # 创建新的管理器对象，并返回
        return NewManager(ctx, config.(*Config))
    }))
}
```