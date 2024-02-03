# `v2ray-core\features\stats\stats.go`

```go
// 生成错误代码
//go:generate go run v2ray.com/core/common/errors/errorgen

// 导入所需的包
import (
    "context"
    "v2ray.com/core/common"
    "v2ray.com/core/features"
)

// Counter 是统计计数器的接口
//
// v2ray:api:stable
type Counter interface {
    // Value 返回计数器的当前值
    Value() int64
    // Set 设置计数器的新值，并返回先前的值
    Set(int64) int64
    // Add 将一个值添加到当前计数器的值，并返回先前的值
    Add(int64) int64
}

// Channel 是统计通道的接口
//
// v2ray:api:stable
type Channel interface {
    // Channel 是一个可运行的单元
    common.Runnable
    // Publish 通过控制上下文向通道广播消息
    Publish(context.Context, interface{})
    // SubscriberCount 返回订阅者的数量
    Subscribers() []chan interface{}
    // Subscribe 注册监听通道流，并返回一个新的监听器通道
    Subscribe() (chan interface{}, error)
    // Unsubscribe 从当前 Channel 对象中取消注册一个监听器通道
    Unsubscribe(chan interface{}) error
}

// SubscribeRunnableChannel 订阅通道并在第一个订阅者到来时启动它
func SubscribeRunnableChannel(c Channel) (chan interface{}, error) {
    if len(c.Subscribers()) == 0 {
        if err := c.Start(); err != nil {
            return nil, err
        }
    }
    return c.Subscribe()
}

// UnsubscribeClosableChannel 取消订阅通道，并在没有更多订阅者时关闭它
func UnsubscribeClosableChannel(c Channel, sub chan interface{}) error {
    if err := c.Unsubscribe(sub); err != nil {
        return err
    }
    if len(c.Subscribers()) == 0 {
        return c.Close()
    }
    return nil
}

// Manager 是统计管理器的接口
//
// v2ray:api:stable
type Manager interface {
    features.Feature
    // 注册一个新的计数器到管理器中。标识符字符串不能为空，并且在其他计数器中必须是唯一的。
    RegisterCounter(string) (Counter, error)
    // 通过标识符从管理器中注销一个计数器。
    UnregisterCounter(string) error
    // 通过标识符返回一个计数器。
    GetCounter(string) Counter
    
    // 注册一个新的通道到管理器中。标识符字符串不能为空，并且在其他通道中必须是唯一的。
    RegisterChannel(string) (Channel, error)
    // 通过标识符从管理器中注销一个通道。
    UnregisterChannel(string) error
    // 通过标识符返回一个通道。
    GetChannel(string) Channel
// GetOrRegisterCounter尝试首先获取StatCounter。如果不存在，则尝试创建一个新的计数器。
func GetOrRegisterCounter(m Manager, name string) (Counter, error) {
    // 获取指定名称的计数器
    counter := m.GetCounter(name)
    // 如果计数器存在，则返回计数器和nil
    if counter != nil {
        return counter, nil
    }
    // 否则注册一个新的计数器
    return m.RegisterCounter(name)
}

// GetOrRegisterChannel尝试首先获取StatChannel。如果不存在，则尝试创建一个新的通道。
func GetOrRegisterChannel(m Manager, name string) (Channel, error) {
    // 获取指定名称的通道
    channel := m.GetChannel(name)
    // 如果通道存在，则返回通道和nil
    if channel != nil {
        return channel, nil
    }
    // 否则注册一个新的通道
    return m.RegisterChannel(name)
}

// ManagerType返回Manager接口的类型。可用于实现common.HasType。
//
// v2ray:api:stable
func ManagerType() interface{} {
    return (*Manager)(nil)
}

// NoopManager是Manager的实现，它没有实际功能。
type NoopManager struct{}

// Type实现common.HasType。
func (NoopManager) Type() interface{} {
    return ManagerType()
}

// RegisterCounter实现Manager。
func (NoopManager) RegisterCounter(string) (Counter, error) {
    return nil, newError("not implemented")
}

// UnregisterCounter实现Manager。
func (NoopManager) UnregisterCounter(string) error {
    return nil
}

// GetCounter实现Manager。
func (NoopManager) GetCounter(string) Counter {
    return nil
}

// RegisterChannel实现Manager。
func (NoopManager) RegisterChannel(string) (Channel, error) {
    return nil, newError("not implemented")
}

// UnregisterChannel实现Manager。
func (NoopManager) UnregisterChannel(string) error {
    return nil
}

// GetChannel实现Manager。
func (NoopManager) GetChannel(string) Channel {
    return nil
}

// Start实现common.Runnable。
func (NoopManager) Start() error { return nil }

// Close实现common.Closable。
func (NoopManager) Close() error { return nil }
```