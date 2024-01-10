# `trojan-go\proxy\client\config.go`

```
package client

import "github.com/p4gefau1t/trojan-go/config"

type MuxConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  # 定义 MuxConfig 结构体，包含一个布尔类型的 Enabled 字段
}

type WebsocketConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  # 定义 WebsocketConfig 结构体，包含一个布尔类型的 Enabled 字段
}

type RouterConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  # 定义 RouterConfig 结构体，包含一个布尔类型的 Enabled 字段
}

type ShadowsocksConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  # 定义 ShadowsocksConfig 结构体，包含一个布尔类型的 Enabled 字段
}

type TransportPluginConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  # 定义 TransportPluginConfig 结构体，包含一个布尔类型的 Enabled 字段
}

type Config struct {
    Mux             MuxConfig             `json:"mux" yaml:"mux"`  # 定义 Config 结构体，包含 MuxConfig 类型的 Mux 字段
    Websocket       WebsocketConfig       `json:"websocket" yaml:"websocket"`  # 包含 WebsocketConfig 类型的 Websocket 字段
    Router          RouterConfig          `json:"router" yaml:"router"`  # 包含 RouterConfig 类型的 Router 字段
    Shadowsocks     ShadowsocksConfig     `json:"shadowsocks" yaml:"shadowsocks"`  # 包含 ShadowsocksConfig 类型的 Shadowsocks 字段
    TransportPlugin TransportPluginConfig `json:"transport_plugin" yaml:"transport-plugin"`  # 包含 TransportPluginConfig 类型的 TransportPlugin 字段
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {  # 在初始化时注册配置创建函数
        return new(Config)  # 返回一个新的 Config 实例
    })
}
```