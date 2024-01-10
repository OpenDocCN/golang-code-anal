# `v2ray-core\transport\internet\memory_settings.go`

```
// MemoryStreamConfig是StreamConfig的解析形式。这用于减少Protobuf解析的次数。
type MemoryStreamConfig struct {
    ProtocolName     string
    ProtocolSettings interface{}
    SecurityType     string
    SecuritySettings interface{}
    SocketSettings   *SocketConfig
}

// ToMemoryStreamConfig将StreamConfig转换为MemoryStreamConfig。对于nil输入，它返回一个默认的非nil的MemoryStreamConfig。
func ToMemoryStreamConfig(s *StreamConfig) (*MemoryStreamConfig, error) {
    // 获取有效的传输设置
    ets, err := s.GetEffectiveTransportSettings()
    if err != nil {
        return nil, err
    }

    // 创建MemoryStreamConfig对象
    mss := &MemoryStreamConfig{
        ProtocolName:     s.GetEffectiveProtocol(),
        ProtocolSettings: ets,
    }

    // 如果s不为nil，则将SocketSettings赋值给mss
    if s != nil {
        mss.SocketSettings = s.SocketSettings
    }

    // 如果s不为nil且具有安全设置，则获取有效的安全设置
    if s != nil && s.HasSecuritySettings() {
        ess, err := s.GetEffectiveSecuritySettings()
        if err != nil {
            return nil, err
        }
        mss.SecurityType = s.SecurityType
        mss.SecuritySettings = ess
    }

    return mss, nil
}
```