# `trojan-go\tunnel\tproxy\config.go`

```go
// 标记该文件只在 Linux 系统下编译
// 构建标签，指定该文件只在 Linux 系统下编译

package tproxy

// 导入 trojan-go/config 包
import "github.com/p4gefau1t/trojan-go/config"

// 定义 Config 结构体
type Config struct {
    LocalHost  string `json:"local_addr" yaml:"local-addr"` // 本地主机地址，json 和 yaml 格式的标签
    LocalPort  int    `json:"local_port" yaml:"local-port"` // 本地端口号，json 和 yaml 格式的标签
    UDPTimeout int    `json:"udp_timeout" yaml:"udp-timeout"` // UDP 超时时间，json 和 yaml 格式的标签
}

// 初始化函数
func init() {
    // 注册配置创建器，指定配置名称和创建函数
    config.RegisterConfigCreator(Name, func() interface{} {
        // 返回 Config 结构体的实例，设置 UDPTimeout 默认值为 60
        return &Config{
            UDPTimeout: 60,
        }
    })
}
```