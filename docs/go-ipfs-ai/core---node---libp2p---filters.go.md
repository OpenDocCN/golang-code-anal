# `kubo\core\node\libp2p\filters.go`

```
package libp2p

import (
    "github.com/libp2p/go-libp2p/core/connmgr"  // 导入连接管理器
    "github.com/libp2p/go-libp2p/core/control"  // 导入控制模块
    "github.com/libp2p/go-libp2p/core/network"  // 导入网络模块
    "github.com/libp2p/go-libp2p/core/peer"     // 导入对等节点模块

    ma "github.com/multiformats/go-multiaddr"   // 导入多地址模块
)

// filtersConnectionGater is an adapter that turns multiaddr.Filter into a
// connmgr.ConnectionGater.
type filtersConnectionGater ma.Filters  // 定义一个类型，将多地址过滤器转换为连接管理器

var _ connmgr.ConnectionGater = (*filtersConnectionGater)(nil)  // 确保 filtersConnectionGater 实现了 connmgr.ConnectionGater 接口

func (f *filtersConnectionGater) InterceptAddrDial(_ peer.ID, addr ma.Multiaddr) (allow bool) {
    return !(*ma.Filters)(f).AddrBlocked(addr)  // 拦截地址拨号，检查地址是否被阻止
}

func (f *filtersConnectionGater) InterceptPeerDial(p peer.ID) (allow bool) {
    return true  // 拦截对等节点拨号，始终允许
}

func (f *filtersConnectionGater) InterceptAccept(connAddr network.ConnMultiaddrs) (allow bool) {
    return !(*ma.Filters)(f).AddrBlocked(connAddr.RemoteMultiaddr())  // 拦截接受连接，检查远程地址是否被阻止
}

func (f *filtersConnectionGater) InterceptSecured(_ network.Direction, _ peer.ID, connAddr network.ConnMultiaddrs) (allow bool) {
    return !(*ma.Filters)(f).AddrBlocked(connAddr.RemoteMultiaddr())  // 拦截已安全连接，检查远程地址是否被阻止
}

func (f *filtersConnectionGater) InterceptUpgraded(_ network.Conn) (allow bool, reason control.DisconnectReason) {
    return true, 0  // 拦截升级连接，始终允许
}
```