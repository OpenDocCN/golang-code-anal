# `v2ray-core\app\policy\manager.go`

```go
// 定义 policy 包，引入必要的依赖包
package policy

import (
    "context"

    "v2ray.com/core/common"
    "v2ray.com/core/features/policy"
)

// Instance 是 Policy 管理器的实例
type Instance struct {
    levels map[uint32]*Policy // 存储不同级别的策略
    system *SystemPolicy // 存储系统级别的策略
}

// New 创建一个新的 Policy 管理器实例
func New(ctx context.Context, config *Config) (*Instance, error) {
    m := &Instance{
        levels: make(map[uint32]*Policy), // 初始化 levels 字典
        system: config.System, // 设置系统级别的策略
    }
    if len(config.Level) > 0 {
        for lv, p := range config.Level {
            pp := defaultPolicy() // 创建默认策略
            pp.overrideWith(p) // 使用配置的策略覆盖默认策略
            m.levels[lv] = pp // 将配置的策略存储到 levels 字典中
        }
    }

    return m, nil // 返回 Policy 管理器实例
}

// Type 实现 common.HasType 接口
func (*Instance) Type() interface{} {
    return policy.ManagerType() // 返回策略管理器的类型
}

// ForLevel 实现 policy.Manager 接口
func (m *Instance) ForLevel(level uint32) policy.Session {
    if p, ok := m.levels[level]; ok {
        return p.ToCorePolicy() // 返回指定级别的策略
    }
    return policy.SessionDefault() // 返回默认的会话策略
}

// ForSystem 实现 policy.Manager 接口
func (m *Instance) ForSystem() policy.System {
    if m.system == nil {
        return policy.System{} // 如果系统级别策略为空，则返回默认的系统策略
    }
    return m.system.ToCorePolicy() // 返回系统级别的策略
}

// Start 实现 common.Runnable.Start() 接口
func (m *Instance) Start() error {
    return nil // 返回空错误，表示启动成功
}

// Close 实现 common.Closable.Close() 接口
func (m *Instance) Close() error {
    return nil // 返回空错误，表示关闭成功
}

// 在包初始化时注册配置
func init() {
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config)) // 注册配置并返回新的 Policy 管理器实例
    }))
}
```