# `kubo\core\corehttp\routing.go`

```
package corehttp

import (
    "context"
    "net"
    "net/http"
    "time"

    "github.com/ipfs/boxo/ipns"
    "github.com/ipfs/boxo/routing/http/server"
    "github.com/ipfs/boxo/routing/http/types"
    "github.com/ipfs/boxo/routing/http/types/iter"
    cid "github.com/ipfs/go-cid"
    core "github.com/ipfs/kubo/core"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/libp2p/go-libp2p/core/routing"
)

func RoutingOption() ServeOption {
    return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        // 创建路由处理器
        handler := server.Handler(&contentRouter{n})
        // 将路由处理器注册到路由路径
        mux.Handle("/routing/v1/", handler)
        return mux, nil
    }
}

type contentRouter struct {
    n *core.IpfsNode
}

func (r *contentRouter) FindProviders(ctx context.Context, key cid.Cid, limit int) (iter.ResultIter[types.Record], error) {
    // 创建新的上下文和取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 异步查找提供者
    ch := r.n.Routing.FindProvidersAsync(ctx, key, limit)
    // 将结果转换为迭代器
    return iter.ToResultIter[types.Record](&peerChanIter{
        ch:     ch,
        cancel: cancel,
    }), nil
}

// nolint deprecated
func (r *contentRouter) ProvideBitswap(ctx context.Context, req *server.BitswapWriteProvideRequest) (time.Duration, error) {
    // 返回不支持的错误
    return 0, routing.ErrNotSupported
}

func (r *contentRouter) FindPeers(ctx context.Context, pid peer.ID, limit int) (iter.ResultIter[*types.PeerRecord], error) {
    // 创建新的上下文和取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 延迟调用取消函数
    defer cancel()

    // 查找对等节点
    addr, err := r.n.Routing.FindPeer(ctx, pid)
    if err != nil {
        return nil, err
    }

    // 创建对等节点记录
    rec := &types.PeerRecord{
        Schema: types.SchemaPeer,
        ID:     &addr.ID,
    }

    // 将地址添加到对等节点记录
    for _, addr := range addr.Addrs {
        rec.Addrs = append(rec.Addrs, types.Multiaddr{Multiaddr: addr})
    }

    // 将结果转换为迭代器
    return iter.ToResultIter[*types.PeerRecord](iter.FromSlice[*types.PeerRecord]([]*types.PeerRecord{rec})), nil
}

func (r *contentRouter) GetIPNS(ctx context.Context, name ipns.Name) (*ipns.Record, error) {
    // TODO: 添加获取 IPNS 记录的逻辑
}
    # 使用传入的上下文创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 在函数返回时调用取消函数，确保上下文被正确释放
    defer cancel()

    # 通过路由键获取值，并将结果存储在raw中，同时检查是否有错误发生
    raw, err := r.n.Routing.GetValue(ctx, string(name.RoutingKey()))
    # 如果发生错误，则返回nil和错误信息
    if err != nil {
        return nil, err
    }

    # 将raw数据解析为IPNS记录，并返回解析后的结果
    return ipns.UnmarshalRecord(raw)
// PutIPNS 将 IPNS 记录存储到路由中
func (r *contentRouter) PutIPNS(ctx context.Context, name ipns.Name, record *ipns.Record) error {
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 延迟调用取消函数
    defer cancel()

    // 将 IPNS 记录序列化为原始数据
    raw, err := ipns.MarshalRecord(record)
    if err != nil {
        return err
    }

    // 调用者保证 name 与记录匹配。这在 PutValue 的内部进行了双重检查
    return r.n.Routing.PutValue(ctx, string(name.RoutingKey()), raw)
}

// peerChanIter 是一个迭代器，用于遍历 peer.AddrInfo 通道
type peerChanIter struct {
    ch     <-chan peer.AddrInfo
    cancel context.CancelFunc
    next   *peer.AddrInfo
}

// Next 用于获取下一个元素
func (it *peerChanIter) Next() bool {
    addr, ok := <-it.ch
    if ok {
        it.next = &addr
        return true
    }
    it.next = nil
    return false
}

// Val 返回当前元素的值
func (it *peerChanIter) Val() types.Record {
    if it.next == nil {
        return nil
    }

    rec := &types.PeerRecord{
        Schema: types.SchemaPeer,
        ID:     &it.next.ID,
    }

    for _, addr := range it.next.Addrs {
        rec.Addrs = append(rec.Addrs, types.Multiaddr{Multiaddr: addr})
    }

    return rec
}

// Close 用于关闭迭代器
func (it *peerChanIter) Close() error {
    it.cancel()
    return nil
}
```