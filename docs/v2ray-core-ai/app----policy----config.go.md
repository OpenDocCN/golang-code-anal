# `v2ray-core\app\policy\config.go`

```go
// 导入必要的包
package policy

import (
    "time"
    "v2ray.com/core/features/policy"
)

// Duration 将秒转换为时间段
func (s *Second) Duration() time.Duration {
    // 如果 Second 对象为空，则返回 0
    if s == nil {
        return 0
    }
    // 返回秒数对应的时间段
    return time.Second * time.Duration(s.Value)
}

// defaultPolicy 返回默认策略
func defaultPolicy() *Policy {
    // 获取默认会话策略
    p := policy.SessionDefault()

    // 返回自定义的策略对象
    return &Policy{
        Timeout: &Policy_Timeout{
            Handshake:      &Second{Value: uint32(p.Timeouts.Handshake / time.Second)},
            ConnectionIdle: &Second{Value: uint32(p.Timeouts.ConnectionIdle / time.Second)},
            UplinkOnly:     &Second{Value: uint32(p.Timeouts.UplinkOnly / time.Second)},
            DownlinkOnly:   &Second{Value: uint32(p.Timeouts.DownlinkOnly / time.Second)},
        },
        Buffer: &Policy_Buffer{
            Connection: p.Buffer.PerConnection,
        },
    }
}

// overrideWith 使用另一个 Policy_Timeout 对象覆盖当前对象的值
func (p *Policy_Timeout) overrideWith(another *Policy_Timeout) {
    // 如果另一个对象的 Handshake 不为空，则覆盖当前对象的值
    if another.Handshake != nil {
        p.Handshake = &Second{Value: another.Handshake.Value}
    }
    // 如果另一个对象的 ConnectionIdle 不为空，则覆盖当前对象的值
    if another.ConnectionIdle != nil {
        p.ConnectionIdle = &Second{Value: another.ConnectionIdle.Value}
    }
    // 如果另一个对象的 UplinkOnly 不为空，则覆盖当前对象的值
    if another.UplinkOnly != nil {
        p.UplinkOnly = &Second{Value: another.UplinkOnly.Value}
    }
    // 如果另一个对象的 DownlinkOnly 不为空，则覆盖当前对象的值
    if another.DownlinkOnly != nil {
        p.DownlinkOnly = &Second{Value: another.DownlinkOnly.Value}
    }
}

// overrideWith 使用另一个 Policy 对象覆盖当前对象的值
func (p *Policy) overrideWith(another *Policy) {
    // 如果另一个对象的 Timeout 不为空，则覆盖当前对象的值
    if another.Timeout != nil {
        p.Timeout.overrideWith(another.Timeout)
    }
    // 如果另一个对象的 Stats 不为空且当前对象的 Stats 为空，则覆盖当前对象的值
    if another.Stats != nil && p.Stats == nil {
        p.Stats = &Policy_Stats{}
        p.Stats = another.Stats
    }
    // 如果另一个对象的 Buffer 不为空，则覆盖当前对象的值
    if another.Buffer != nil {
        p.Buffer = &Policy_Buffer{
            Connection: another.Buffer.Connection,
        }
    }
}

// ToCorePolicy 将当前 Policy 对象转换为 policy.Session 对象
func (p *Policy) ToCorePolicy() policy.Session {
    // 获取默认会话策略
    cp := policy.SessionDefault()
    # 如果传入的参数 p.Timeout 不为空，则将其各个时间段的值赋给连接池的超时时间
    if p.Timeout != nil:
        cp.Timeouts.ConnectionIdle = p.Timeout.ConnectionIdle.Duration()
        cp.Timeouts.Handshake = p.Timeout.Handshake.Duration()
        cp.Timeouts.DownlinkOnly = p.Timeout.DownlinkOnly.Duration()
        cp.Timeouts.UplinkOnly = p.Timeout.UplinkOnly.Duration()
    # 如果传入的参数 p.Stats 不为空，则将其用户上行和下行流量统计值赋给连接池的统计信息
    if p.Stats != nil:
        cp.Stats.UserUplink = p.Stats.UserUplink
        cp.Stats.UserDownlink = p.Stats.UserDownlink
    # 如果传入的参数 p.Buffer 不为空，则将其连接缓冲大小赋给连接池的缓冲设置
    if p.Buffer != nil:
        cp.Buffer.PerConnection = p.Buffer.Connection
    # 返回连接池对象
    return cp
// 将 SystemPolicy 转换为 policy.System 类型
func (p *SystemPolicy) ToCorePolicy() policy.System {
    // 返回一个 policy.System 对象，包含 SystemStats 结构体
    return policy.System{
        // 将 InboundUplink 字段赋值为 SystemPolicy 的 InboundUplink 字段的值
        Stats: policy.SystemStats{
            InboundUplink:    p.Stats.InboundUplink,
            // 将 InboundDownlink 字段赋值为 SystemPolicy 的 InboundDownlink 字段的值
            InboundDownlink:  p.Stats.InboundDownlink,
            // 将 OutboundUplink 字段赋值为 SystemPolicy 的 OutboundUplink 字段的值
            OutboundUplink:   p.Stats.OutboundUplink,
            // 将 OutboundDownlink 字段赋值为 SystemPolicy 的 OutboundDownlink 字段的值
            OutboundDownlink: p.Stats.OutboundDownlink,
        },
    }
}
```