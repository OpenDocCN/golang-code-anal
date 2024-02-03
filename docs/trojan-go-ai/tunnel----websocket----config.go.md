# `trojan-go\tunnel\websocket\config.go`

```go
package websocket

import "github.com/p4gefau1t/trojan-go/config"

type WebsocketConfig struct {
    Enabled bool   `json:"enabled" yaml:"enabled"`  // WebSocket 是否启用
    Host    string `json:"host" yaml:"host"`        // WebSocket 服务器主机地址
    Path    string `json:"path" yaml:"path"`        // WebSocket 路径
}

type Config struct {
    RemoteHost string          `json:"remote_addr" yaml:"remote-addr"`  // 远程主机地址
    RemotePort int             `json:"remote_port" yaml:"remote-port"`  // 远程主机端口
    Websocket  WebsocketConfig `json:"websocket" yaml:"websocket"`      // WebSocket 配置
}

func init() {
    // 注册配置创建函数，当需要创建配置时调用该函数
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(Config)
    })
}
```