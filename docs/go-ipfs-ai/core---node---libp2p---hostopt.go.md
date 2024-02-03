# `kubo\core\node\libp2p\hostopt.go`

```go
package libp2p

import (
    "fmt"

    "github.com/libp2p/go-libp2p"  // 导入 libp2p 包
    "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 核心主机包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等体包
    "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 libp2p 核心对等体存储包
)

type HostOption func(id peer.ID, ps peerstore.Peerstore, options ...libp2p.Option) (host.Host, error)  // 定义 HostOption 类型的函数

var DefaultHostOption HostOption = constructPeerHost  // 定义 DefaultHostOption 变量并赋值为 constructPeerHost 函数

// isolates the complex initialization steps
func constructPeerHost(id peer.ID, ps peerstore.Peerstore, options ...libp2p.Option) (host.Host, error) {
    pkey := ps.PrivKey(id)  // 获取指定 ID 的私钥
    if pkey == nil {
        return nil, fmt.Errorf("missing private key for node ID: %s", id)  // 如果私钥为空，则返回错误信息
    }
    options = append([]libp2p.Option{libp2p.Identity(pkey), libp2p.Peerstore(ps)}, options...)  // 将身份和对等体存储选项添加到 options 中
    return libp2p.New(options...)  // 使用 options 创建新的 libp2p 主机
}
```