# `trojan-go\proxy\custom\config.go`

```
package custom

import "github.com/p4gefau1t/trojan-go/config"

// 定义常量 Name 为 "CUSTOM"
const Name = "CUSTOM"

// 定义 NodeConfig 结构体，包含 Protocol、Tag 和 Config 三个字段
type NodeConfig struct {
    Protocol string      `json:"protocol" yaml:"protocol"`
    Tag      string      `json:"tag" yaml:"tag"`
    Config   interface{} `json:"config" yaml:"config"`
}

// 定义 StackConfig 结构体，包含 Path 和 Node 两个字段
type StackConfig struct {
    Path [][]string   `json:"path" yaml:"path"`
    Node []NodeConfig `json:"node" yaml:"node"`
}

// 定义 Config 结构体，包含 Inbound 和 Outbound 两个字段
type Config struct {
    Inbound  StackConfig `json:"inbound" yaml:"inbound"`
    Outbound StackConfig `json:"outbound" yaml:"outbound"`
}

// 在包初始化时注册 ConfigCreator，用于创建 Config 实例
func init() {
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(Config)
    })
}
```