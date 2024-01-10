# `v2ray-core\transport\internet\udp\config.go`

```
# 导入 UDP 包
package udp

# 导入必要的包
import (
    "v2ray.com/core/common"
    "v2ray.com/core/transport/internet"
)

# 初始化函数
func init() {
    # 注册协议配置创建器，当需要创建该协议的配置时调用
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        # 返回一个新的 Config 对象
        return new(Config)
    }))
}
```