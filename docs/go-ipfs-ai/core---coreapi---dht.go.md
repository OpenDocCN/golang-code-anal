# `kubo\core\coreapi\dht.go`

```
package coreapi

import (
    "context"
    "fmt"

    blockservice "github.com/ipfs/boxo/blockservice"
    blockstore "github.com/ipfs/boxo/blockstore"
    offline "github.com/ipfs/boxo/exchange/offline"
    dag "github.com/ipfs/boxo/ipld/merkledag"
    "github.com/ipfs/boxo/path"
    cid "github.com/ipfs/go-cid"
    cidutil "github.com/ipfs/go-cidutil"
    coreiface "github.com/ipfs/kubo/core/coreiface"
    caopts "github.com/ipfs/kubo/core/coreiface/options"
    "github.com/ipfs/kubo/tracing"
    peer "github.com/libp2p/go-libp2p/core/peer"
    routing "github.com/libp2p/go-libp2p/core/routing"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"
)

type DhtAPI CoreAPI

// FindPeer 根据 peer.ID 查找对应的 peer.AddrInfo
func (api *DhtAPI) FindPeer(ctx context.Context, p peer.ID) (peer.AddrInfo, error) {
    // 创建一个新的 span，记录操作名称和 peer.ID
    ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "FindPeer", trace.WithAttributes(attribute.String("peer", p.String())))
    defer span.End() // 在函数返回前结束 span
    // 检查节点是否在线
    err := api.checkOnline(false)
    if err != nil {
        return peer.AddrInfo{}, err
    }

    // 调用底层 routing 模块的 FindPeer 方法查找 peer.AddrInfo
    pi, err := api.routing.FindPeer(ctx, peer.ID(p))
    if err != nil {
        return peer.AddrInfo{}, err
    }

    return pi, nil
}

// FindProviders 根据路径查找数据提供者的 peer.AddrInfo
func (api *DhtAPI) FindProviders(ctx context.Context, p path.Path, opts ...caopts.DhtFindProvidersOption) (<-chan peer.AddrInfo, error) {
    // 创建一个新的 span，记录操作名称和路径
    ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "FindProviders", trace.WithAttributes(attribute.String("path", p.String())))
    defer span.End() // 在函数返回前结束 span

    // 解析 DhtFindProvidersOption
    settings, err := caopts.DhtFindProvidersOptions(opts...)
    if err != nil {
        return nil, err
    }
    span.SetAttributes(attribute.Int("numproviders", settings.NumProviders)) // 设置 span 属性

    // 检查节点是否在线
    err = api.checkOnline(false)
    if err != nil {
        return nil, err
    }

    // 解析路径并查找数据提供者
    rp, _, err := api.core().ResolvePath(ctx, p)
    if err != nil {
        return nil, err
    }

    numProviders := settings.NumProviders
    if numProviders < 1 {
        return nil, fmt.Errorf("number of providers must be greater than 0")
    }
}
    # 使用 API 对象的路由功能异步查找提供程序
    pchan := api.routing.FindProvidersAsync(ctx, rp.RootCid(), numProviders)
    # 返回提供程序通道和空错误
    return pchan, nil
// DhtAPI 结构体的 Provide 方法，用于提供指定路径的内容到 DHT 网络
func (api *DhtAPI) Provide(ctx context.Context, path path.Path, opts ...caopts.DhtProvideOption) error {
    // 创建一个新的 span 用于追踪 Provide 方法的执行
    ctx, span := tracing.Span(ctx, "CoreAPI.DhtAPI", "Provide", trace.WithAttributes(attribute.String("path", path.String())))
    // 在方法执行结束时结束 span
    defer span.End()

    // 解析提供选项
    settings, err := caopts.DhtProvideOptions(opts...)
    if err != nil {
        return err
    }
    // 设置 span 的属性，记录是否递归提供
    span.SetAttributes(attribute.Bool("recursive", settings.Recursive))

    // 检查节点是否在线
    err = api.checkOnline(false)
    if err != nil {
        return err
    }

    // 解析路径并获取根 CID
    rp, _, err := api.core().ResolvePath(ctx, path)
    if err != nil {
        return err
    }

    c := rp.RootCid()

    // 检查本地是否存在指定的块
    has, err := api.blockstore.Has(ctx, c)
    if err != nil {
        return err
    }

    if !has {
        return fmt.Errorf("block %s not found locally, cannot provide", c)
    }

    // 如果设置为递归提供，则调用 provideKeysRec 方法，否则调用 provideKeys 方法
    if settings.Recursive {
        err = provideKeysRec(ctx, api.routing, api.blockstore, []cid.Cid{c})
    } else {
        err = provideKeys(ctx, api.routing, []cid.Cid{c})
    }
    if err != nil {
        return err
    }

    return nil
}

// provideKeys 方法，用于向 DHT 网络提供指定 CID 的内容
func provideKeys(ctx context.Context, r routing.Routing, cids []cid.Cid) error {
    for _, c := range cids {
        err := r.Provide(ctx, c, true)
        if err != nil {
            return err
        }
    }
    return nil
}

// provideKeysRec 方法，用于递归地向 DHT 网络提供指定 CID 的内容
func provideKeysRec(ctx context.Context, r routing.Routing, bs blockstore.Blockstore, cids []cid.Cid) error {
    // 创建一个新的 StreamingSet 用于记录已经提供的 CID
    provided := cidutil.NewStreamingSet()

    // 创建一个错误通道
    errCh := make(chan error)
    // 启动一个 goroutine，用于递归地提供指定 CID 的内容
    go func() {
        // 创建一个 DAGService，并使用提供的 Blockstore 和 Exchange
        dserv := dag.NewDAGService(blockservice.New(bs, offline.Exchange(bs)))
        for _, c := range cids {
            // 递归地遍历指定 CID 的链接，并提供到 DHT 网络
            err := dag.Walk(ctx, dag.GetLinksDirect(dserv), c, provided.Visitor(ctx))
            if err != nil {
                errCh <- err
            }
        }
    }()
    # 无限循环，等待多个通道的消息
    for {
        # 从 provided.New 通道接收消息，并赋值给 k
        select {
        # 如果从 provided.New 通道接收到消息
        case k := <-provided.New:
            # 调用 r.Provide 方法，传入上下文和 k，第二个参数为 true
            err := r.Provide(ctx, k, true)
            # 如果出现错误
            if err != nil:
                # 返回错误
                return err
        # 如果从 errCh 通道接收到消息
        case err := <-errCh:
            # 返回错误
            return err
        # 如果上下文的 Done 通道关闭
        case <-ctx.Done():
            # 返回上下文的错误
            return ctx.Err()
        }
    }
# 定义一个方法，返回DhtAPI结构体的coreiface.CoreAPI接口
func (api *DhtAPI) core() coreiface.CoreAPI {
    # 将api转换为CoreAPI类型，并返回
    return (*CoreAPI)(api)
}
```