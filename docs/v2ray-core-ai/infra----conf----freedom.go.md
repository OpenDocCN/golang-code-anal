# `v2ray-core\infra\conf\freedom.go`

```go
package conf

import (
    "net"  // 导入 net 包，提供了基本的网络 I/O 接口
    "strings"  // 导入 strings 包，提供了操作字符串的函数

    "github.com/golang/protobuf/proto"  // 导入 proto 包，用于处理 Protocol Buffers 数据
    v2net "v2ray.com/core/common/net"  // 导入 v2net 包，提供了网络相关的功能
    "v2ray.com/core/common/protocol"  // 导入 protocol 包，提供了协议相关的功能
    "v2ray.com/core/proxy/freedom"  // 导入 freedom 包，提供了自由代理的功能
)

type FreedomConfig struct {
    DomainStrategy string  `json:"domainStrategy"`  // 自由代理的域名策略
    Timeout        *uint32 `json:"timeout"`  // 超时时间
    Redirect       string  `json:"redirect"`  // 重定向地址
    UserLevel      uint32  `json:"userLevel"`  // 用户级别
}

// Build implements Buildable
func (c *FreedomConfig) Build() (proto.Message, error) {
    config := new(freedom.Config)  // 创建自由代理配置对象
    config.DomainStrategy = freedom.Config_AS_IS  // 默认域名策略为 AS_IS
    switch strings.ToLower(c.DomainStrategy) {  // 根据配置的域名策略进行处理
    case "useip", "use_ip":
        config.DomainStrategy = freedom.Config_USE_IP  // 使用 IP 地址
    case "useip4", "useipv4", "use_ipv4", "use_ip_v4", "use_ip4":
        config.DomainStrategy = freedom.Config_USE_IP4  // 使用 IPv4 地址
    case "useip6", "useipv6", "use_ipv6", "use_ip_v6", "use_ip6":
        config.DomainStrategy = freedom.Config_USE_IP6  // 使用 IPv6 地址
    }

    if c.Timeout != nil {  // 如果超时时间不为空
        config.Timeout = *c.Timeout  // 设置超时时间
    }
    config.UserLevel = c.UserLevel  // 设置用户级别
    if len(c.Redirect) > 0 {  // 如果重定向地址不为空
        host, portStr, err := net.SplitHostPort(c.Redirect)  // 分割重定向地址的主机和端口
        if err != nil {
            return nil, newError("invalid redirect address: ", c.Redirect, ": ", err).Base(err)  // 返回错误信息
        }
        port, err := v2net.PortFromString(portStr)  // 解析端口字符串
        if err != nil {
            return nil, newError("invalid redirect port: ", c.Redirect, ": ", err).Base(err)  // 返回错误信息
        }
        config.DestinationOverride = &freedom.DestinationOverride{  // 设置目标重定向
            Server: &protocol.ServerEndpoint{  // 设置服务器端点
                Port: uint32(port),  // 设置端口
            },
        }

        if len(host) > 0 {  // 如果主机地址不为空
            config.DestinationOverride.Server.Address = v2net.NewIPOrDomain(v2net.ParseAddress(host))  // 设置服务器地址
        }
    }
    return config, nil  // 返回配置对象和空错误
}
```