# `trojan-go\tunnel\trojan\config.go`

```go
package trojan

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
    LocalHost        string      `json:"local_addr" yaml:"local-addr"`  // 本地主机地址
    LocalPort        int         `json:"local_port" yaml:"local-port"`  // 本地端口
    RemoteHost       string      `json:"remote_addr" yaml:"remote-addr"`  // 远程主机地址
    RemotePort       int         `json:"remote_port" yaml:"remote-port"`  // 远程端口
    DisableHTTPCheck bool        `json:"disable_http_check" yaml:"disable-http-check"`  // 禁用 HTTP 检查
    MySQL            MySQLConfig `json:"mysql" yaml:"mysql"`  // MySQL 配置
    API              APIConfig   `json:"api" yaml:"api"`  // API 配置
}

type MySQLConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  // MySQL 是否启用
}

type APIConfig struct {
    Enabled bool `json:"enabled" yaml:"enabled"`  // API 是否启用
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {
        return &Config{}  // 注册配置创建器
    })
}
```