# `kubo\core\coreiface\dht.go`

```
package iface

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时

    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理 IPFS 路径

    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包，用于处理核心接口的选项

    "github.com/libp2p/go-libp2p/core/peer"  // 导入对等节点包，用于处理对等节点的操作
)

// DhtAPI specifies the interface to the DHT
// Note: This API will likely get deprecated in near future, see
// https://github.com/ipfs/interface-ipfs-core/issues/249 for more context.
type DhtAPI interface {
    // FindPeer queries the DHT for all of the multiaddresses associated with a
    // Peer ID
    FindPeer(context.Context, peer.ID) (peer.AddrInfo, error)  // 查询 DHT 中与对等节点 ID 相关的所有多地址

    // FindProviders finds peers in the DHT who can provide a specific value
    // given a key.
    FindProviders(context.Context, path.Path, ...options.DhtFindProvidersOption) (<-chan peer.AddrInfo, error)  // 在 DHT 中查找可以提供特定值的对等节点

    // Provide announces to the network that you are providing given values
    Provide(context.Context, path.Path, ...options.DhtProvideOption) error  // 向网络宣布您正在提供给定的值
}
```