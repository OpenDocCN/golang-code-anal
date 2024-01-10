# `v2ray-core\features\routing\context.go`

```
package routing

import (
    "v2ray.com/core/common/net"
)

// Context是用于存储路由连接信息的特性。

// v2ray:api:stable

type Context interface {
    // GetInboundTag返回连接所属的入站标签。
    GetInboundTag() string

    // GetSourcesIPs返回连接绑定的源IP。
    GetSourceIPs() []net.IP

    // GetSourcePort返回连接的源端口。
    GetSourcePort() net.Port

    // GetTargetIPs返回连接的目标IP或目标域名的解析IP。
    GetTargetIPs() []net.IP

    // GetTargetPort返回连接的目标端口。
    GetTargetPort() net.Port

    // GetTargetDomain返回连接的目标域名（如果存在）。
    GetTargetDomain() string

    // GetNetwork返回连接的网络类型。
    GetNetwork() net.Network

    // GetProtocol返回连接内容中嗅探出的协议（如果有）。
    GetProtocol() string

    // GetUser返回连接内容中的用户电子邮件（如果存在）。
    GetUser() string

    // GetAttributes返回连接内容中的额外属性。
    GetAttributes() map[string]string
}
```