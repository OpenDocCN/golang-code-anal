# `v2ray-core\infra\conf\command\command.go`

```
package command

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "os"

    "github.com/golang/protobuf/proto"
    "v2ray.com/core/common"
    "v2ray.com/core/infra/conf/serial"
    "v2ray.com/core/infra/control"
)

// ConfigCommand 结构体
type ConfigCommand struct{}

// 返回命令名称
func (c *ConfigCommand) Name() string {
    return "config"
}

// 返回命令描述
func (c *ConfigCommand) Description() control.Description {
    return control.Description{
        Short: "Convert config among different formats.",
        Usage: []string{
            "v2ctl config",
        },
    }
}

// 执行命令
func (c *ConfigCommand) Execute(args []string) error {
    // 从标准输入加载 JSON 格式的配置
    pbConfig, err := serial.LoadJSONConfig(os.Stdin)
    if err != nil {
        return newError("failed to parse json config").Base(err)
    }

    // 将配置转换为字节流
    bytesConfig, err := proto.Marshal(pbConfig)
    if err != nil {
        return newError("failed to marshal proto config").Base(err)
    }

    // 将字节流写入标准输出
    if _, err := os.Stdout.Write(bytesConfig); err != nil {
        return newError("failed to write proto config").Base(err)
    }
    return nil
}

// 注册 ConfigCommand 结构体为命令
func init() {
    common.Must(control.RegisterCommand(&ConfigCommand{}))
}
```