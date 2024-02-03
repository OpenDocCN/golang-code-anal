# `v2ray-core\features\outbound\outbound.go`

```go
// 导入所需的包
package outbound

import (
    "context"  // 上下文包，用于处理请求的上下文信息

    "v2ray.com/core/common"  // V2Ray 核心通用包
    "v2ray.com/core/features"  // V2Ray 核心特性包
    "v2ray.com/core/transport"  // V2Ray 核心传输包
)

// Handler 是处理出站连接的接口
//
// v2ray:api:stable
type Handler interface {
    common.Runnable  // 可运行的接口
    Tag() string  // 返回处理器的标签
    Dispatch(ctx context.Context, link *transport.Link)  // 处理出站连接
}

type HandlerSelector interface {
    Select([]string) []string  // 选择处理器
}

// Manager 是管理出站处理器的特性
//
// v2ray:api:stable
type Manager interface {
    features.Feature  // 特性接口
    // GetHandler 根据标签返回出站处理器
    GetHandler(tag string) Handler
    // GetDefaultHandler 返回默认的出站处理器，通常是配置中指定的第一个出站处理器
    GetDefaultHandler() Handler
    // AddHandler 向出站管理器中添加一个处理器
    AddHandler(ctx context.Context, handler Handler) error
    // RemoveHandler 从出站管理器中移除一个处理器
    RemoveHandler(ctx context.Context, tag string) error
}

// ManagerType 返回 Manager 接口的类型，可用于实现 common.HasType 接口
//
// v2ray:api:stable
func ManagerType() interface{} {
    return (*Manager)(nil)
}
```