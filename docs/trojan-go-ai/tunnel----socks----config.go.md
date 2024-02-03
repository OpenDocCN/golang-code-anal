# `trojan-go\tunnel\socks\config.go`

```go
# 声明 socks 包
package socks

# 导入 config 包
import "github.com/p4gefau1t/trojan-go/config"

# 定义 Config 结构体
type Config struct {
    LocalHost  string `json:"local_addr" yaml:"local-addr"`  # 本地主机地址
    LocalPort  int    `json:"local_port" yaml:"local-port"`  # 本地端口号
    UDPTimeout int    `json:"udp_timeout" yaml:"udp-timeout"`  # UDP 超时时间
}

# 初始化函数
func init() {
    # 注册配置创建器，当调用时返回 Config 结构体的实例
    config.RegisterConfigCreator(Name, func() interface{} {
        # 返回一个 Config 结构体的实例，其中 UDPTimeout 默认值为 60
        return &Config{
            UDPTimeout: 60,  # 设置 UDP 超时时间为 60
        }
    })
}
```