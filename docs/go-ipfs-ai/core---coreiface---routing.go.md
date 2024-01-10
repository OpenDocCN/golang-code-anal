# `kubo\core\coreiface\routing.go`

```
// 定义一个名为iface的包
package iface

// 导入所需的包
import (
    "context" // 上下文包
    "github.com/ipfs/kubo/core/coreiface/options" // 导入options包
)

// 定义RoutingAPI接口，用于路由层的接口规范
type RoutingAPI interface {
    // Get方法，用于获取给定键的最佳值
    Get(context.Context, string) ([]byte, error)

    // Put方法，用于为给定键设置一个值
    Put(ctx context.Context, key string, value []byte, opts ...options.RoutingPutOption) error
}
```