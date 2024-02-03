# `v2ray-core\proxy\vmess\encoding\encoding.go`

```go
package encoding

import (
    "v2ray.com/core/common/net"  // 导入网络相关的包
    "v2ray.com/core/common/protocol"  // 导入协议相关的包
)

//go:generate go run v2ray.com/core/common/errors/errorgen  // 使用go:generate命令生成错误处理代码

const (
    Version = byte(1)  // 定义常量Version为字节1
)

var addrParser = protocol.NewAddressParser(  // 创建地址解析器
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv4), net.AddressFamilyIPv4),  // IPv4地址类型映射
    protocol.AddressFamilyByte(byte(protocol.AddressTypeDomain), net.AddressFamilyDomain),  // 域名地址类型映射
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv6), net.AddressFamilyIPv6),  // IPv6地址类型映射
    protocol.PortThenAddress(),  // 端口然后地址的解析方式
)
```