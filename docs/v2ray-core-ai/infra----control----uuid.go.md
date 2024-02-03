# `v2ray-core\infra\control\uuid.go`

```go
package control

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "v2ray.com/core/common"  // 导入 v2ray 核心通用包
    "v2ray.com/core/common/uuid"  // 导入 v2ray 核心通用 UUID 包
)

type UUIDCommand struct{}  // 定义 UUIDCommand 结构体

func (c *UUIDCommand) Name() string {  // 定义 Name 方法，返回字符串
    return "uuid"  // 返回字符串 "uuid"
}

func (c *UUIDCommand) Description() Description {  // 定义 Description 方法，返回 Description 结构体
    return Description{  // 返回 Description 结构体
        Short: "Generate new UUIDs",  // 设置 Short 字段为 "Generate new UUIDs"
        Usage: []string{"v2ctl uuid"},  // 设置 Usage 字段为包含字符串 "v2ctl uuid" 的字符串数组
    }
}

func (c *UUIDCommand) Execute([]string) error {  // 定义 Execute 方法，接收字符串数组参数，返回错误
    u := uuid.New()  // 生成新的 UUID
    fmt.Println(u.String())  // 格式化输出 UUID 字符串
    return nil  // 返回空错误
}

func init() {  // 初始化函数
    common.Must(RegisterCommand(&UUIDCommand{}))  // 注册 UUIDCommand 结构体
}
```