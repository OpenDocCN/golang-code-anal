# `v2ray-core\app\proxyman\config.go`

```go
package proxyman

// 获取并发值，如果 AllocationStrategy 为空或者并发值为空，则返回默认值 3
func (s *AllocationStrategy) GetConcurrencyValue() uint32 {
    if s == nil || s.Concurrency == nil {
        return 3
    }
    return s.Concurrency.Value
}

// 获取刷新值，如果 AllocationStrategy 为空或者刷新值为空，则返回默认值 5
func (s *AllocationStrategy) GetRefreshValue() uint32 {
    if s == nil || s.Refresh == nil {
        return 5
    }
    return s.Refresh.Value
}

// 获取有效的嗅探设置，如果 ReceiverConfig 的嗅探设置不为空，则返回该设置
// 如果 DomainOverride 不为空，则根据其中的值创建嗅探设置
func (c *ReceiverConfig) GetEffectiveSniffingSettings() *SniffingConfig {
    if c.SniffingSettings != nil {
        return c.SniffingSettings
    }

    if len(c.DomainOverride) > 0 {
        var p []string
        for _, kd := range c.DomainOverride {
            switch kd {
            case KnownProtocols_HTTP:
                p = append(p, "http")
            case KnownProtocols_TLS:
                p = append(p, "tls")
            }
        }
        return &SniffingConfig{
            Enabled:             true,
            DestinationOverride: p,
        }
    }

    return nil
}
```