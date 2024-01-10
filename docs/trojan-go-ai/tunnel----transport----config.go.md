# `trojan-go\tunnel\transport\config.go`

```
package transport

import (
    "github.com/p4gefau1t/trojan-go/config"
)

type Config struct {
    LocalHost       string                `json:"local_addr" yaml:"local-addr"`  // 本地主机地址
    LocalPort       int                   `json:"local_port" yaml:"local-port"`  // 本地端口号
    RemoteHost      string                `json:"remote_addr" yaml:"remote-addr"`  // 远程主机地址
    RemotePort      int                   `json:"remote_port" yaml:"remote-port"`  // 远程端口号
    TransportPlugin TransportPluginConfig `json:"transport_plugin" yaml:"transport-plugin"`  // 传输插件配置
}

type TransportPluginConfig struct {
    Enabled bool     `json:"enabled" yaml:"enabled"`  // 插件是否启用
    Type    string   `json:"type" yaml:"type"`  // 插件类型
    Command string   `json:"command" yaml:"command"`  // 插件命令
    Option  string   `json:"option" yaml:"option"`  // 插件选项
    Arg     []string `json:"arg" yaml:"arg"`  // 插件参数
    Env     []string `json:"env" yaml:"env"`  // 插件环境变量
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(Config)
    })
}
```