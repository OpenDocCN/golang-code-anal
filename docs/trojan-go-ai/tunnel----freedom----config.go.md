# `trojan-go\tunnel\freedom\config.go`

```
package freedom

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
    LocalHost    string             `json:"local_addr" yaml:"local-addr"`  // 本地主机地址
    LocalPort    int                `json:"local_port" yaml:"local-port"`  // 本地端口号
    TCP          TCPConfig          `json:"tcp" yaml:"tcp"`                // TCP 配置
    ForwardProxy ForwardProxyConfig `json:"forward_proxy" yaml:"forward-proxy"`  // 转发代理配置
}

type TCPConfig struct {
    PreferIPV4 bool `json:"prefer_ipv4" yaml:"prefer-ipv4"`  // 是否优先使用 IPV4
    KeepAlive  bool `json:"keep_alive" yaml:"keep-alive"`    // 是否开启 TCP 连接保持活动状态
    NoDelay    bool `json:"no_delay" yaml:"no-delay"`        // 是否禁用 Nagle 算法
}

type ForwardProxyConfig struct {
    Enabled   bool   `json:"enabled" yaml:"enabled"`         // 是否启用转发代理
    ProxyHost string `json:"proxy_addr" yaml:"proxy-addr"`    // 代理主机地址
    ProxyPort int    `json:"proxy_port" yaml:"proxy-port"`    // 代理端口号
    Username  string `json:"username" yaml:"username"`       // 用户名
    Password  string `json:"password" yaml:"password"`       // 密码
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {
        return &Config{
            TCP: TCPConfig{
                PreferIPV4: false,  // 默认为 false
                NoDelay:    true,   // 默认为 true
                KeepAlive:  true,   // 默认为 true
            },
        }
    })
}
```