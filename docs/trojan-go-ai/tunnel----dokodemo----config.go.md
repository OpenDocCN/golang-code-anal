# `trojan-go\tunnel\dokodemo\config.go`

```go
package dokodemo

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
    LocalHost  string `json:"local_addr" yaml:"local-addr"`  // 本地主机地址
    LocalPort  int    `json:"local_port" yaml:"local-port"`  // 本地端口号
    TargetHost string `json:"target_addr" yaml:"target-addr"`  // 目标主机地址
    TargetPort int    `json:"target_port" yaml:"target-port"`  // 目标端口号
    UDPTimeout int    `json:"udp_timeout" yaml:"udp-timeout"`  // UDP 超时时间
}

func init() {
    // 注册配置创建器，指定配置类型和创建函数
    config.RegisterConfigCreator(Name, func() interface{} {
        // 返回一个新的 Config 对象，设置 UDP 超时时间为 60 秒
        return &Config{
            UDPTimeout: 60,
        }
    })
}
```