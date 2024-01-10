# `v2ray-core\transport\internet\tcp\config.go`

```
// +build !confonly
// 声明该文件不仅仅是配置文件

package tcp
// 声明包名为tcp

import (
    "v2ray.com/core/common"
    "v2ray.com/core/transport/internet"
)
// 导入所需的包

const protocolName = "tcp"
// 声明常量protocolName为"tcp"

func init() {
    // 初始化函数
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        return new(Config)
    }))
    // 注册TCP协议的配置创建器
}
```