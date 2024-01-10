# `kubo\core\coreiface\coreapi.go`

```
// Package iface 定义了 IPFS 核心 API，用于与 IPFS 节点进行交互。
package iface

import (
    "context" // 导入 context 包，用于处理请求的取消、超时和截止日期

    "github.com/ipfs/boxo/path" // 导入路径包，用于处理 IPFS 路径
    "github.com/ipfs/kubo/core/coreiface/options" // 导入选项包，用于处理 IPFS 核心接口的选项

    ipld "github.com/ipfs/go-ipld-format" // 导入 ipld 包，用于处理 IPFS 数据结构
)

// CoreAPI 定义了一个统一的接口，用于 Go 程序与 IPFS 进行交互
type CoreAPI interface {
    // Unixfs 返回 Unixfs API 的实现
    Unixfs() UnixfsAPI

    // Block 返回 Block API 的实现
    Block() BlockAPI

    // Dag 返回 Dag API 的实现
    Dag() APIDagService

    // Name 返回 Name API 的实现
    Name() NameAPI

    // Key 返回 Key API 的实现
    Key() KeyAPI

    // Pin 返回 Pin API 的实现
    Pin() PinAPI

    // Object 返回 Object API 的实现
    Object() ObjectAPI

    // Dht 返回 Dht API 的实现
    Dht() DhtAPI

    // Swarm 返回 Swarm API 的实现
    Swarm() SwarmAPI

    // PubSub 返回 PubSub API 的实现
    PubSub() PubSubAPI

    // Routing 返回 Routing API 的实现
    Routing() RoutingAPI

    // ResolvePath 使用 UnixFS 解析器解析路径，并返回解析后的不可变路径，以及无法在 UnixFS 中解析的路径段的剩余部分
    ResolvePath(context.Context, path.Path) (path.ImmutablePath, []string, error)

    // ResolveNode 使用 Unixfs 解析器解析路径（如果尚未解析），获取并返回解析后的节点
    ResolveNode(context.Context, path.Path) (ipld.Node, error)

    // WithOptions 根据应用了一组选项的当前实例创建新的 CoreAPI 实例
    WithOptions(...options.ApiOption) (CoreAPI, error)
}
```