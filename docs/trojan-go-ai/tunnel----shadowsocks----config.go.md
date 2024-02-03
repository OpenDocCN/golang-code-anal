# `trojan-go\tunnel\shadowsocks\config.go`

```go
package shadowsocks

import "github.com/p4gefau1t/trojan-go/config"

type ShadowsocksConfig struct {
    Enabled  bool   `json:"enabled" yaml:"enabled"`  # 定义 Shadowsocks 配置结构体，包含 enabled、method、password 三个字段
    Method   string `json:"method" yaml:"method"`    # 使用 json 和 yaml 标签定义字段的序列化方式
    Password string `json:"password" yaml:"password"`
}

type Config struct {
    RemoteHost  string            `json:"remote_addr" yaml:"remote-addr"`  # 定义 Config 结构体，包含 remote_addr、remote_port、shadowsocks 三个字段
    RemotePort  int               `json:"remote_port" yaml:"remote-port"`  # 使用 json 和 yaml 标签定义字段的序列化方式
    Shadowsocks ShadowsocksConfig `json:"shadowsocks" yaml:"shadowsocks"`
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {  # 在初始化时注册配置创建函数
        return &Config{  # 返回一个 Config 结构体指针
            Shadowsocks: ShadowsocksConfig{  # 初始化 ShadowsocksConfig 结构体
                Method: "AES-128-GCM",  # 设置 method 字段的默认值
            },
        }
    })
}
```