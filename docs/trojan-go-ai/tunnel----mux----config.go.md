# `trojan-go\tunnel\mux\config.go`

```
package mux

import "github.com/p4gefau1t/trojan-go/config"

type MuxConfig struct {
    Enabled     bool `json:"enabled" yaml:"enabled"`  // 是否启用多路复用
    IdleTimeout int  `json:"idle_timeout" yaml:"idle-timeout"`  // 空闲超时时间
    Concurrency int  `json:"concurrency" yaml:"concurrency"`  // 并发连接数
}

type Config struct {
    Mux MuxConfig `json:"mux" yaml:"mux"`  // 多路复用配置
}

func init() {
    // 注册配置创建函数
    config.RegisterConfigCreator(Name, func() interface{} {
        // 返回配置对象
        return &Config{
            Mux: MuxConfig{
                Enabled:     false,  // 默认禁用多路复用
                IdleTimeout: 30,  // 默认空闲超时时间为30秒
                Concurrency: 8,  // 默认并发连接数为8
            },
        }
    })
}
```