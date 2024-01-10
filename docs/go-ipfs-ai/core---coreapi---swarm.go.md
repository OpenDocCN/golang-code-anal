# `kubo\core\coreapi\swarm.go`

```
package coreapi

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息
    "sort"  // 导入排序包，用于对数据进行排序
    "time"  // 导入时间包，用于处理时间相关的操作

    coreiface "github.com/ipfs/kubo/core/coreiface"  // 导入自定义包coreiface
    "github.com/ipfs/kubo/tracing"  // 导入tracing包，用于跟踪
    inet "github.com/libp2p/go-libp2p/core/network"  // 导入网络包，用于处理网络相关操作
    "github.com/libp2p/go-libp2p/core/peer"  // 导入peer包，用于处理peer相关操作
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入peerstore包，用于处理peerstore相关操作
    "github.com/libp2p/go-libp2p/core/protocol"  // 导入protocol包，用于处理协议相关操作
    "github.com/libp2p/go-libp2p/p2p/net/swarm"  // 导入swarm包，用于处理网络相关操作
    ma "github.com/multiformats/go-multiaddr"  // 导入multiaddr包，用于处理多地址相关操作
    "go.opentelemetry.io/otel/attribute"  // 导入attribute包，用于处理属性相关操作
    "go.opentelemetry.io/otel/trace"  // 导入trace包，用于处理跟踪相关操作
)

type SwarmAPI CoreAPI  // 定义SwarmAPI类型为CoreAPI类型

type connInfo struct {
    peerstore pstore.Peerstore  // 定义peerstore字段为Peerstore类型
    conn      inet.Conn  // 定义conn字段为Conn类型
    dir       inet.Direction  // 定义dir字段为Direction类型

    addr ma.Multiaddr  // 定义addr字段为Multiaddr类型
    peer peer.ID  // 定义peer字段为ID类型
}

// tag used in the connection manager when explicitly connecting to a peer.
const (
    connectionManagerTag    = "user-connect"  // 定义连接管理器标签为"user-connect"
    connectionManagerWeight = 100  // 定义连接管理器权重为100
)

func (api *SwarmAPI) Connect(ctx context.Context, pi peer.AddrInfo) error {
    ctx, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Connect", trace.WithAttributes(attribute.String("peerid", pi.ID.String())))  // 创建跟踪span
    defer span.End()  // 延迟结束span

    if api.peerHost == nil {  // 如果peerHost为空
        return coreiface.ErrOffline  // 返回离线错误
    }

    if swrm, ok := api.peerHost.Network().(*swarm.Swarm); ok {  // 如果peerHost的网络类型是Swarm
        swrm.Backoff().Clear(pi.ID)  // 清除Backoff
    }

    if err := api.peerHost.Connect(ctx, pi); err != nil {  // 连接到peerHost
        return err  // 返回错误
    }

    api.peerHost.ConnManager().TagPeer(pi.ID, connectionManagerTag, connectionManagerWeight)  // 给peer打标签
    return nil  // 返回空
}

func (api *SwarmAPI) Disconnect(ctx context.Context, addr ma.Multiaddr) error {
    _, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Disconnect", trace.WithAttributes(attribute.String("addr", addr.String())))  // 创建跟踪span
    defer span.End()  // 延迟结束span

    if api.peerHost == nil {  // 如果peerHost为空
        return coreiface.ErrOffline  // 返回离线错误
    }

    taddr, id := peer.SplitAddr(addr)  // 分割地址
    if id == "" {  // 如果id为空
        return peer.ErrInvalidAddr  // 返回无效地址错误
    }

    span.SetAttributes(attribute.String("peerid", id.String()))  // 设置属性

    net := api.peerHost.Network()  // 获取peerHost的网络
    # 如果目标地址为空
    if taddr == nil:
        # 如果与指定 ID 的节点未连接
        if net.Connectedness(id) != inet.Connected:
            # 返回未连接错误
            return coreiface.ErrNotConnected
        # 尝试关闭指定 ID 的节点连接
        if err := net.ClosePeer(id); err != nil:
            # 返回关闭连接时的错误
            return err
        # 返回空值
        return nil
    # 遍历与指定 ID 相关的连接
    for _, conn := range net.ConnsToPeer(id):
        # 如果连接的远程地址不等于目标地址
        if !conn.RemoteMultiaddr().Equal(taddr):
            # 继续下一次循环
            continue
        # 关闭当前连接
        return conn.Close()
    # 返回连接未找到的错误
    return coreiface.ErrConnNotFound
# 获取已知节点的地址信息，返回节点 ID 到地址列表的映射
func (api *SwarmAPI) KnownAddrs(ctx context.Context) (map[peer.ID][]ma.Multiaddr, error) {
    # 创建一个跟踪 span
    _, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "KnownAddrs")
    # 延迟结束跟踪 span
    defer span.End()

    # 如果节点主机为空，则返回错误信息
    if api.peerHost == nil {
        return nil, coreiface.ErrOffline
    }

    # 创建一个节点 ID 到地址列表的映射
    addrs := make(map[peer.ID][]ma.Multiaddr)
    # 获取节点主机的 peerstore
    ps := api.peerHost.Network().Peerstore()
    # 遍历 peerstore 中的所有节点
    for _, p := range ps.Peers() {
        # 将节点的地址添加到地址列表中
        addrs[p] = append(addrs[p], ps.Addrs(p)...)
        # 对地址列表进行排序
        sort.Slice(addrs[p], func(i, j int) bool {
            return addrs[p][i].String() < addrs[p][j].String()
        })
    }

    # 返回节点 ID 到地址列表的映射
    return addrs, nil
}

# 获取本地节点的地址信息
func (api *SwarmAPI) LocalAddrs(ctx context.Context) ([]ma.Multiaddr, error) {
    # 创建一个跟踪 span
    _, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "LocalAddrs")
    # 延迟结束跟踪 span
    defer span.End()

    # 如果节点主机为空，则返回错误信息
    if api.peerHost == nil {
        return nil, coreiface.ErrOffline
    }

    # 返回节点主机的地址列表
    return api.peerHost.Addrs(), nil
}

# 获取节点监听的地址信息
func (api *SwarmAPI) ListenAddrs(ctx context.Context) ([]ma.Multiaddr, error) {
    # 创建一个跟踪 span
    _, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "ListenAddrs")
    # 延迟结束跟踪 span
    defer span.End()

    # 如果节点主机为空，则返回错误信息
    if api.peerHost == nil {
        return nil, coreiface.ErrOffline
    }

    # 返回节点主机网络接口的监听地址列表
    return api.peerHost.Network().InterfaceListenAddresses()
}

# 获取节点的连接信息
func (api *SwarmAPI) Peers(ctx context.Context) ([]coreiface.ConnectionInfo, error) {
    # 创建一个跟踪 span
    _, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Peers")
    # 延迟结束跟踪 span
    defer span.End()

    # 如果节点主机为空，则返回错误信息
    if api.peerHost == nil {
        return nil, coreiface.ErrOffline
    }

    # 获取节点主机网络的连接列表
    conns := api.peerHost.Network().Conns()

    # 创建一个连接信息列表
    out := make([]coreiface.ConnectionInfo, 0, len(conns))
    # 遍历连接数组，使用下划线 _ 忽略索引，c 为当前连接对象
    for _, c := range conns {
        # 创建 connInfo 结构体对象 ci，并初始化其属性
        ci := &connInfo{
            peerstore: api.peerstore,  # 设置 peerstore 属性为 api.peerstore
            conn:      c,               # 设置 conn 属性为 c
            dir:       c.Stat().Direction,  # 设置 dir 属性为 c.Stat().Direction

            addr: c.RemoteMultiaddr(),  # 设置 addr 属性为 c.RemoteMultiaddr()
            peer: c.RemotePeer(),       # 设置 peer 属性为 c.RemotePeer()
        }

        '''
            # FIXME(steb): 
            # 检查当前连接对象 c 是否为 swarm.Conn 类型，如果是则执行以下操作
            swcon, ok := c.(*swarm.Conn)
            if ok {
                # 设置 ci 的 muxer 属性为 swcon.StreamConn().Conn() 的类型字符串
                ci.muxer = fmt.Sprintf("%T", swcon.StreamConn().Conn())
            }
        '''

        # 将 ci 添加到输出数组 out 中
        out = append(out, ci)
    }

    # 返回输出数组 out 和空错误
    return out, nil
# 获取连接信息的对等节点ID
func (ci *connInfo) ID() peer.ID {
    return ci.peer
}

# 获取连接信息的对等节点地址
func (ci *connInfo) Address() ma.Multiaddr {
    return ci.addr
}

# 获取连接信息的方向
func (ci *connInfo) Direction() inet.Direction {
    return ci.dir
}

# 获取连接信息的延迟时间
func (ci *connInfo) Latency() (time.Duration, error) {
    return ci.peerstore.LatencyEWMA(peer.ID(ci.ID())), nil
}

# 获取连接信息的流协议
func (ci *connInfo) Streams() ([]protocol.ID, error) {
    # 获取连接的所有流
    streams := ci.conn.GetStreams()

    # 创建一个存储流协议的数组
    out := make([]protocol.ID, len(streams))
    # 遍历所有流，获取其协议并存储到数组中
    for i, s := range streams {
        out[i] = s.Protocol()
    }

    return out, nil
}
```