# `kubo\fuse\ipns\common.go`

```
package ipns

import (
    "context"  // 导入 context 包，用于处理上下文
    ft "github.com/ipfs/boxo/ipld/unixfs"  // 导入 ft 别名，用于操作 unixfs
    "github.com/ipfs/boxo/namesys"  // 导入 namesys 包，用于处理命名系统
    "github.com/ipfs/boxo/path"  // 导入 path 包，用于处理路径
    "github.com/ipfs/kubo/core"  // 导入 core 包，用于处理核心功能
    ci "github.com/libp2p/go-libp2p/core/crypto"  // 导入 ci 别名，用于处理加密
)

// InitializeKeyspace sets the ipns record for the given key to
// point to an empty directory.
func InitializeKeyspace(n *core.IpfsNode, key ci.PrivKey) error {
    ctx, cancel := context.WithCancel(n.Context())  // 创建一个带有取消函数的上下文
    defer cancel()  // 延迟调用取消函数

    emptyDir := ft.EmptyDirNode()  // 创建一个空目录节点

    err := n.Pinning.Pin(ctx, emptyDir, false, "")  // 将空目录节点固定到 IPFS 节点
    if err != nil {
        return err  // 如果出现错误，返回错误信息
    }

    err = n.Pinning.Flush(ctx)  // 刷新固定的节点
    if err != nil {
        return err  // 如果出现错误，返回错误信息
    }

    pub := namesys.NewIPNSPublisher(n.Routing, n.Repo.Datastore())  // 创建一个新的 IPNS 发布者

    return pub.Publish(ctx, key, path.FromCid(emptyDir.Cid()))  // 发布 IPNS 记录
}
```