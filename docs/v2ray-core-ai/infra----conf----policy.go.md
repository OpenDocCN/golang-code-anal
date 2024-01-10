# `v2ray-core\infra\conf\policy.go`

```
package conf

import (
    "v2ray.com/core/app/policy"
)

type Policy struct {
    Handshake         *uint32 `json:"handshake"`  // 握手超时时间
    ConnectionIdle    *uint32 `json:"connIdle"`   // 连接空闲超时时间
    UplinkOnly        *uint32 `json:"uplinkOnly"` // 仅上行流量超时时间
    DownlinkOnly      *uint32 `json:"downlinkOnly"` // 仅下行流量超时时间
    StatsUserUplink   bool    `json:"statsUserUplink"` // 统计用户上行流量
    StatsUserDownlink bool    `json:"statsUserDownlink"` // 统计用户下行流量
    BufferSize        *int32  `json:"bufferSize"` // 缓冲区大小
}

func (t *Policy) Build() (*policy.Policy, error) {
    config := new(policy.Policy_Timeout)  // 创建超时策略配置对象
    if t.Handshake != nil {
        config.Handshake = &policy.Second{Value: *t.Handshake}  // 设置握手超时时间
    }
    if t.ConnectionIdle != nil {
        config.ConnectionIdle = &policy.Second{Value: *t.ConnectionIdle}  // 设置连接空闲超时时间
    }
    if t.UplinkOnly != nil {
        config.UplinkOnly = &policy.Second{Value: *t.UplinkOnly}  // 设置仅上行流量超时时间
    }
    if t.DownlinkOnly != nil {
        config.DownlinkOnly = &policy.Second{Value: *t.DownlinkOnly}  // 设置仅下行流量超时时间
    }

    p := &policy.Policy{  // 创建策略对象
        Timeout: config,  // 设置超时策略
        Stats: &policy.Policy_Stats{  // 设置统计策略
            UserUplink:   t.StatsUserUplink,  // 设置统计用户上行流量
            UserDownlink: t.StatsUserDownlink,  // 设置统计用户下行流量
        },
    }

    if t.BufferSize != nil {
        bs := int32(-1)  // 初始化缓冲区大小
        if *t.BufferSize >= 0 {
            bs = (*t.BufferSize) * 1024  // 计算缓冲区大小
        }
        p.Buffer = &policy.Policy_Buffer{  // 设置缓冲区策略
            Connection: bs,  // 设置连接缓冲区大小
        }
    }

    return p, nil  // 返回策略对象
}

type SystemPolicy struct {
    StatsInboundUplink    bool `json:"statsInboundUplink"`  // 统计入站上行流量
    StatsInboundDownlink  bool `json:"statsInboundDownlink"`  // 统计入站下行流量
    StatsOutboundUplink   bool `json:"statsOutboundUplink"`  // 统计出站上行流量
    StatsOutboundDownlink bool `json:"statsOutboundDownlink"`  // 统计出站下行流量
}

func (p *SystemPolicy) Build() (*policy.SystemPolicy, error) {
    # 返回系统策略对象，包含统计信息
    return &policy.SystemPolicy{
        # 设置统计信息中的上行入站流量
        Stats: &policy.SystemPolicy_Stats{
            InboundUplink:    p.StatsInboundUplink,
            # 设置统计信息中的下行入站流量
            InboundDownlink:  p.StatsInboundDownlink,
            # 设置统计信息中的上行出站流量
            OutboundUplink:   p.StatsOutboundUplink,
            # 设置统计信息中的下行出站流量
            OutboundDownlink: p.StatsOutboundDownlink,
        },
    }, nil
// 定义 PolicyConfig 结构体，包含 Levels 和 System 两个字段
type PolicyConfig struct {
    Levels map[uint32]*Policy `json:"levels"` // 定义 Levels 字段为 uint32 到 Policy 指针的映射
    System *SystemPolicy      `json:"system"` // 定义 System 字段为 SystemPolicy 指针
}

// Build 方法用于构建 policy.Config 对象
func (c *PolicyConfig) Build() (*policy.Config, error) {
    levels := make(map[uint32]*policy.Policy) // 创建一个空的 uint32 到 Policy 指针的映射
    // 遍历 Levels 字段，构建 policy.Policy 对象并添加到 levels 中
    for l, p := range c.Levels {
        if p != nil {
            pp, err := p.Build() // 调用 Policy 对象的 Build 方法
            if err != nil {
                return nil, err
            }
            levels[l] = pp // 将构建好的 policy.Policy 对象添加到 levels 中
        }
    }
    // 创建 policy.Config 对象，设置其 Level 字段为 levels
    config := &policy.Config{
        Level: levels,
    }

    // 如果 System 字段不为空，构建 SystemPolicy 对象并设置为 config 的 System 字段
    if c.System != nil {
        sc, err := c.System.Build() // 调用 SystemPolicy 对象的 Build 方法
        if err != nil {
            return nil, err
        }
        config.System = sc // 设置 config 的 System 字段
    }

    return config, nil // 返回构建好的 policy.Config 对象和 nil 错误
}
```