# `v2ray-core\infra\conf\reverse.go`

```
package conf

import (
    "github.com/golang/protobuf/proto"  // 导入 protobuf 库
    "v2ray.com/core/app/reverse"  // 导入 reverse 应用程序包
)

type BridgeConfig struct {
    Tag    string `json:"tag"`  // 定义桥接配置结构体，包含标签和域名字段
    Domain string `json:"domain"`
}

func (c *BridgeConfig) Build() (*reverse.BridgeConfig, error) {
    return &reverse.BridgeConfig{  // 构建桥接配置对象
        Tag:    c.Tag,  // 使用传入的标签和域名
        Domain: c.Domain,
    }, nil
}

type PortalConfig struct {
    Tag    string `json:"tag"`  // 定义门户配置结构体，包含标签和域名字段
    Domain string `json:"domain"`
}

func (c *PortalConfig) Build() (*reverse.PortalConfig, error) {
    return &reverse.PortalConfig{  // 构建门户配置对象
        Tag:    c.Tag,  // 使用传入的标签和域名
        Domain: c.Domain,
    }, nil
}

type ReverseConfig struct {
    Bridges []BridgeConfig `json:"bridges"`  // 定义反向配置结构体，包含桥接配置和门户配置切片
    Portals []PortalConfig `json:"portals"`
}

func (c *ReverseConfig) Build() (proto.Message, error) {
    config := &reverse.Config{}  // 构建反向配置对象
    for _, bconfig := range c.Bridges {  // 遍历桥接配置切片
        b, err := bconfig.Build()  // 构建桥接配置对象
        if err != nil {
            return nil, err  // 如果构建失败，返回错误
        }
        config.BridgeConfig = append(config.BridgeConfig, b)  // 将构建的桥接配置对象添加到反向配置对象中
    }

    for _, pconfig := range c.Portals {  // 遍历门户配置切片
        p, err := pconfig.Build()  // 构建门户配置对象
        if err != nil {
            return nil, err  // 如果构建失败，返回错误
        }
        config.PortalConfig = append(config.PortalConfig, p)  // 将构建的门户配置对象添加到反向配置对象中
    }

    return config, nil  // 返回构建好的反向配置对象
}
```