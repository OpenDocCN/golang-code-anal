# `v2ray-core\proxy\vmess\inbound\config.go`

```
// +build !confonly
// 设置构建标签，指定该代码块不仅仅是配置文件

package inbound
// 定义 GetDefaultValue 方法，返回 DefaultConfig 的默认设置
func (c *Config) GetDefaultValue() *DefaultConfig {
    // 如果获取到的默认配置为空，则返回默认的 DefaultConfig
    if c.GetDefault() == nil {
        return &DefaultConfig{
            AlterId: 32, // 设置默认的 AlterId 为 32
            Level:   0,  // 设置默认的 Level 为 0
        }
    }
    return c.Default  // 返回获取到的默认配置
}
```