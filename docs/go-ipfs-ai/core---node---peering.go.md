# `kubo\core\node\peering.go`

```go
package node

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "github.com/ipfs/boxo/peering"  // 导入peering包，用于构建peering服务
    "github.com/libp2p/go-libp2p/core/host"  // 导入libp2p的host包，用于构建主机
    "github.com/libp2p/go-libp2p/core/peer"  // 导入libp2p的peer包，用于构建对等体
    "go.uber.org/fx"  // 导入fx包，用于依赖注入和生命周期管理
)

// Peering constructs the peering service and hooks it into fx's lifetime
// management system.
func Peering(lc fx.Lifecycle, host host.Host) *peering.PeeringService {
    ps := peering.NewPeeringService(host)  // 创建peering服务实例
    lc.Append(fx.Hook{  // 在生命周期管理系统中添加钩子
        OnStart: func(context.Context) error {  // 在启动时执行的函数
            return ps.Start()  // 启动peering服务
        },
        OnStop: func(context.Context) error {  // 在停止时执行的函数
            ps.Stop()  // 停止peering服务
            return nil
        },
    })
    return ps  // 返回peering服务实例
}

// PeerWith configures the peering service to peer with the specified peers.
func PeerWith(peers ...peer.AddrInfo) fx.Option {
    return fx.Invoke(func(ps *peering.PeeringService) {  // 使用依赖注入调用函数
        for _, ai := range peers {  // 遍历指定的对等体信息
            ps.AddPeer(ai)  // 将对等体添加到peering服务中
        }
    })
}
```