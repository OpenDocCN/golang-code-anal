# `kubo\config\peering.go`

```
// 导入 "github.com/libp2p/go-libp2p/core/peer" 包，用于获取 peer.AddrInfo 结构体
import "github.com/libp2p/go-libp2p/core/peer"

// Peering 结构体用于配置对等网络服务
type Peering struct {
    // Peers 字段用于列出要尝试保持连接的节点
    Peers []peer.AddrInfo
}
```