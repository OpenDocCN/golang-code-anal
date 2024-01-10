# `v2ray-core\infra\conf\dns_proxy.go`

```
package conf

import (
    "github.com/golang/protobuf/proto"  // 导入 protobuf 库
    "v2ray.com/core/common/net"         // 导入网络库
    "v2ray.com/core/proxy/dns"          // 导入 DNS 代理库
)

type DnsOutboundConfig struct {
    Network Network  `json:"network"`   // 定义网络类型字段
    Address *Address `json:"address"`   // 定义地址字段
    Port    uint16   `json:"port"`      // 定义端口字段
}

func (c *DnsOutboundConfig) Build() (proto.Message, error) {
    config := &dns.Config{               // 创建 DNS 配置对象
        Server: &net.Endpoint{           // 设置服务器端点
            Network: c.Network.Build(),  // 设置网络类型
            Port:    uint32(c.Port),     // 设置端口
        },
    }
    if c.Address != nil {                // 如果地址不为空
        config.Server.Address = c.Address.Build()  // 设置服务器地址
    }
    return config, nil                   // 返回配置对象和空错误
}
```