# `v2ray-core\proxy\dokodemo\config.go`

```
// 导入 dokodemo 包和 net 包
package dokodemo

import (
    "v2ray.com/core/common/net"
)

// GetPredefinedAddress 从 proto 配置中返回预定义的地址。如果地址无效，则返回 Null。
func (v *Config) GetPredefinedAddress() net.Address {
    // 将地址转换为 net.Address 类型
    addr := v.Address.AsAddress()
    // 如果地址为空，则返回空值
    if addr == nil {
        return nil
    }
    // 返回地址
    return addr
}
```