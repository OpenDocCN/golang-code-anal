# `v2ray-core\context.go`

```go
// +build !confonly
// 标记此文件不仅仅是配置文件

package core
// 导入 context 包

import (
    "context"
)

// V2rayKey 是上下文中 Instance 的键类型，为测试而导出
type V2rayKey int

const v2rayKey V2rayKey = 1
// 定义 v2rayKey 常量并赋值为 1

// 从上下文中返回 Instance，如果上下文中不包含则返回 nil
func FromContext(ctx context.Context) *Instance {
    if s, ok := ctx.Value(v2rayKey).(*Instance); ok {
        return s
    }
    return nil
}

// 从上下文中返回 Instance，如果不存在则引发 panic
func MustFromContext(ctx context.Context) *Instance {
    v := FromContext(ctx)
    if v == nil {
        panic("V is not in context.")
    }
    return v
}
```