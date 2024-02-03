# `trojan-go\statistic\memory\config.go`

```go
# 导入内存包
package memory

# 导入配置包
import (
    "github.com/p4gefau1t/trojan-go/config"
)

# 定义配置结构体
type Config struct {
    Passwords []string `json:"password" yaml:"password"`  # 定义密码字段，用于 JSON 和 YAML 格式的序列化和反序列化
}

# 初始化函数
func init() {
    # 注册配置创建函数，当需要创建配置时调用该函数
    config.RegisterConfigCreator(Name, func() interface{} {
        return &Config{}  # 返回一个新的配置结构体实例
    })
}
```