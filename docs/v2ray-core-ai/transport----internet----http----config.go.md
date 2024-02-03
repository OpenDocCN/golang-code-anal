# `v2ray-core\transport\internet\http\config.go`

```go
// +build !confonly  // 如果不是仅用于配置，则包含该代码

package http  // 定义包名为http

import (  // 导入需要的包
    "v2ray.com/core/common"  // 导入common包
    "v2ray.com/core/common/dice"  // 导入dice包
    "v2ray.com/core/transport/internet"  // 导入internet包
)

const protocolName = "http"  // 定义常量protocolName为"http"

func (c *Config) getHosts() []string {  // 定义方法getHosts，返回字符串数组
    if len(c.Host) == 0 {  // 如果Host字段长度为0
        return []string{"www.example.com"}  // 返回包含字符串"www.example.com"的数组
    }
    return c.Host  // 返回Host字段的值
}

func (c *Config) isValidHost(host string) bool {  // 定义方法isValidHost，接收字符串参数，返回布尔值
    hosts := c.getHosts()  // 调用getHosts方法，获取主机列表
    for _, h := range hosts {  // 遍历主机列表
        if h == host {  // 如果当前主机等于参数中的主机
            return true  // 返回true
        }
    }
    return false  // 如果没有匹配的主机，返回false
}

func (c *Config) getRandomHost() string {  // 定义方法getRandomHost，返回字符串
    hosts := c.getHosts()  // 调用getHosts方法，获取主机列表
    return hosts[dice.Roll(len(hosts))]  // 返回随机选择的主机
}

func (c *Config) getNormalizedPath() string {  // 定义方法getNormalizedPath，返回字符串
    if c.Path == "" {  // 如果Path字段为空
        return "/"  // 返回"/"
    }
    if c.Path[0] != '/' {  // 如果Path字段第一个字符不是'/'
        return "/" + c.Path  // 返回"/"加上Path字段的值
    }
    return c.Path  // 返回Path字段的值
}

func init() {  // 初始化函数
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {  // 调用common包的Must方法和internet包的RegisterProtocolConfigCreator方法
        return new(Config)  // 返回一个新的Config对象
    }))
}
```