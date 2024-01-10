# `kubo\core\coreiface\swarm.go`

```
package iface

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "errors"   // 导入错误包，用于定义和处理错误
    "time"     // 导入时间包，用于处理时间相关的操作

    "github.com/libp2p/go-libp2p/core/network"  // 导入libp2p网络核心包，用于处理网络相关操作
    "github.com/libp2p/go-libp2p/core/peer"     // 导入libp2p对等体核心包，用于处理对等体相关操作
    "github.com/libp2p/go-libp2p/core/protocol" // 导入libp2p协议核心包，用于处理协议相关操作

    ma "github.com/multiformats/go-multiaddr"   // 导入多地址包，并重命名为ma
)

var (
    ErrNotConnected = errors.New("not connected")  // 定义一个错误变量，表示未连接
    ErrConnNotFound = errors.New("conn not found")  // 定义一个错误变量，表示连接未找到
)

// ConnectionInfo contains information about a peer
type ConnectionInfo interface {
    // ID returns PeerID
    ID() peer.ID  // 返回对等体ID

    // Address returns the multiaddress via which we are connected with the peer
    Address() ma.Multiaddr  // 返回与对等体连接的多地址

    // Direction returns which way the connection was established
    Direction() network.Direction  // 返回连接建立的方向

    // Latency returns last known round trip time to the peer
    Latency() (time.Duration, error)  // 返回与对等体的最后一次往返时间

    // Streams returns list of streams established with the peer
    Streams() ([]protocol.ID, error)  // 返回与对等体建立的流列表
}

// SwarmAPI specifies the interface to libp2p swarm
type SwarmAPI interface {
    // Connect to a given peer
    Connect(context.Context, peer.AddrInfo) error  // 连接到指定的对等体

    // Disconnect from a given address
    Disconnect(context.Context, ma.Multiaddr) error  // 从指定地址断开连接

    // Peers returns the list of peers we are connected to
    Peers(context.Context) ([]ConnectionInfo, error)  // 返回我们连接的对等体列表

    // KnownAddrs returns the list of all addresses this node is aware of
    KnownAddrs(context.Context) (map[peer.ID][]ma.Multiaddr, error)  // 返回节点知道的所有地址列表

    // LocalAddrs returns the list of announced listening addresses
    LocalAddrs(context.Context) ([]ma.Multiaddr, error)  // 返回已宣布的监听地址列表

    // ListenAddrs returns the list of all listening addresses
    ListenAddrs(context.Context) ([]ma.Multiaddr, error)  // 返回所有监听地址列表
}
```