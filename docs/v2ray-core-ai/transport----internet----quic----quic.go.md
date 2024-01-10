# `v2ray-core\transport\internet\quic\quic.go`

```
// +build !confonly
// 标记此文件不仅仅是配置文件

package quic
// 定义包名为 quic

import (
    "v2ray.com/core/common"
    "v2ray.com/core/transport/internet"
)
// 导入所需的包

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用go:generate命令生成错误代码

// Here is some modification needs to be done before update quic vendor.
// * use bytespool in buffer_pool.go
// * set MaxReceivePacketSize to 1452 - 32 (16 bytes auth, 16 bytes head)
// 在更新quic供应商之前需要进行一些修改。

const protocolName = "quic"
// 定义常量protocolName为"quic"
const internalDomain = "quic.internal.v2ray.com"
// 定义常量internalDomain为"quic.internal.v2ray.com"

func init() {
    // 初始化函数
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        return new(Config)
    }))
    // 注册quic协议配置创建器
}
```