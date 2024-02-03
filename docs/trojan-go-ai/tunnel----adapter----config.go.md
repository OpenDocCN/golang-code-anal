# `trojan-go\tunnel\adapter\config.go`

```go
# 导入 adapter 包和 config 包
import "github.com/p4gefau1t/trojan-go/config"

# 定义 Config 结构体，包含 LocalHost 和 LocalPort 两个字段
type Config struct {
    LocalHost string `json:"local_addr" yaml:"local-addr"`  # 使用 json 和 yaml 标签定义字段的序列化和反序列化规则
    LocalPort int    `json:"local_port" yaml:"local-port"`  # 使用 json 和 yaml 标签定义字段的序列化和反序列化规则
}

# 初始化函数
func init() {
    # 注册配置创建器，当需要创建 Config 类型的配置时，调用匿名函数返回一个 Config 实例
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(Config)
    })
}
```