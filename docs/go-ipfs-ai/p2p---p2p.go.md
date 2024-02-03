# `kubo\p2p\p2p.go`

```go
package p2p

import (
    logging "github.com/ipfs/go-log"  // 导入日志库
    p2phost "github.com/libp2p/go-libp2p/core/host"  // 导入 libp2p 核心主机库
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心对等体库
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 libp2p 核心对等体存储库
    "github.com/libp2p/go-libp2p/core/protocol"  // 导入 libp2p 核心协议库
)

var log = logging.Logger("p2p-mount")  // 定义日志记录器

// P2P structure holds information on currently running streams/Listeners.
type P2P struct {
    ListenersLocal *Listeners  // 本地监听器
    ListenersP2P   *Listeners  // P2P 监听器
    Streams        *StreamRegistry  // 流注册表

    identity  peer.ID  // 对等体 ID
    peerHost  p2phost.Host  // libp2p 主机
    peerstore pstore.Peerstore  // 对等体存储
}

// New creates new P2P struct.
func New(identity peer.ID, peerHost p2phost.Host, peerstore pstore.Peerstore) *P2P {
    return &P2P{
        identity:  identity,  // 设置对等体 ID
        peerHost:  peerHost,  // 设置 libp2p 主机
        peerstore: peerstore,  // 设置对等体存储

        ListenersLocal: newListenersLocal(),  // 创建新的本地监听器
        ListenersP2P:   newListenersP2P(peerHost),  // 创建新的 P2P 监听器
        Streams: &StreamRegistry{  // 创建新的流注册表
            Streams:     map[uint64]*Stream{},  // 初始化流映射
            ConnManager: peerHost.ConnManager(),  // 设置连接管理器
            conns:       map[peer.ID]int{},  // 初始化连接映射
        },
    }
}

// CheckProtoExists checks whether a proto handler is registered to
// mux handler.
func (p2p *P2P) CheckProtoExists(proto protocol.ID) bool {
    protos := p2p.peerHost.Mux().Protocols()  // 获取当前注册的协议

    for _, p := range protos {  // 遍历注册的协议
        if p != proto {  // 如果当前协议不等于目标协议
            continue  // 继续下一次循环
        }
        return true  // 返回 true，表示目标协议已注册
    }
    return false  // 返回 false，表示目标协议未注册
}
```