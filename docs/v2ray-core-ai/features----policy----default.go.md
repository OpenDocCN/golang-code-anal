# `v2ray-core\features\policy\default.go`

```go
// 定义 DefaultManager 结构体，实现了 Manager 接口
type DefaultManager struct{}

// 实现 common.HasType 接口的 Type 方法
func (DefaultManager) Type() interface{} {
    return ManagerType()
}

// 实现 Manager 接口的 ForLevel 方法
func (DefaultManager) ForLevel(level uint32) Session {
    // 创建默认会话对象
    p := SessionDefault()
    // 如果级别为 1，则设置连接空闲超时时间为 600 秒
    if level == 1 {
        p.Timeouts.ConnectionIdle = time.Second * 600
    }
    return p
}

// 实现 Manager 接口的 ForSystem 方法
func (DefaultManager) ForSystem() System {
    // 返回空的系统对象
    return System{}
}

// 实现 common.Runnable 接口的 Start 方法
func (DefaultManager) Start() error {
    // 返回空错误
    return nil
}

// 实现 common.Closable 接口的 Close 方法
func (DefaultManager) Close() error {
    // 返回空错误
    return nil
}
```