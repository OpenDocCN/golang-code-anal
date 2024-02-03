# `v2ray-core\transport\internet\xtls\config_windows.go`

```go
// 根据构建标签，该代码块仅在 Windows 系统下编译
// 根据构建标签，该代码块不仅仅用于配置

package xtls

import "crypto/x509"

// 获取证书池的方法
func (c *Config) getCertPool() (*x509.CertPool, error) {
    // 如果禁用系统根证书，则加载自定义证书池
    if c.DisableSystemRoot {
        return c.loadSelfCertPool()
    }

    // 否则返回空值
    return nil, nil
}
```