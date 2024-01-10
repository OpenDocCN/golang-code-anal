# `v2ray-core\transport\config.go`

```
// 导入 transport 包和 internet 包
package transport

import (
    "v2ray.com/core/transport/internet"
)

// Apply 方法用于应用该配置
func (c *Config) Apply() error {
    // 如果配置为空，则返回空
    if c == nil {
        return nil
    }
    // 应用全局传输设置到传输配置中
    return internet.ApplyGlobalTransportSettings(c.TransportSettings)
}
```