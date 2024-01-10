# `kubo\core\node\libp2p\peerstore.go`

```
package libp2p

import (
    "context"  // 导入 context 包，用于处理上下文
    "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 peerstore 包，用于管理节点信息
    "github.com/libp2p/go-libp2p/p2p/host/peerstore/pstoremem"  // 导入 pstoremem 包，用于创建内存中的 peerstore
    "go.uber.org/fx"  // 导入 fx 包，用于依赖注入
)

func Peerstore(lc fx.Lifecycle) (peerstore.Peerstore, error) {
    // 创建内存中的 peerstore
    pstore, err := pstoremem.NewPeerstore()
    if err != nil {
        return nil, err  // 如果创建失败，返回错误
    }
    // 在生命周期结束时关闭 peerstore
    lc.Append(fx.Hook{
        OnStop: func(ctx context.Context) error {
            return pstore.Close()  // 在停止时关闭 peerstore
        },
    })

    return pstore, nil  // 返回创建的 peerstore
}
```