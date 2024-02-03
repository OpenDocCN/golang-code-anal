# `kubo\core\coreapi\coreapi.go`

```go
/*
**NOTE: this package is experimental.**

Package coreapi provides direct access to the core commands in IPFS. If you are
embedding IPFS directly in your Go program, this package is the public
interface you should use to read and write files or otherwise control IPFS.

If you are running IPFS as a separate process, you should use `client/rpc` to
work with it via HTTP.
*/
package coreapi

import (
    "context" // 导入上下文包，用于控制程序的执行流程
    "errors" // 导入错误处理包，用于处理错误
    "fmt" // 导入格式化包，用于格式化输出

    bserv "github.com/ipfs/boxo/blockservice" // 导入块服务包
    blockstore "github.com/ipfs/boxo/blockstore" // 导入块存储包
    exchange "github.com/ipfs/boxo/exchange" // 导入交换包
    offlinexch "github.com/ipfs/boxo/exchange/offline" // 导入离线交换包
    "github.com/ipfs/boxo/fetcher" // 导入获取器包
    dag "github.com/ipfs/boxo/ipld/merkledag" // 导入MerkleDAG包
    pathresolver "github.com/ipfs/boxo/path/resolver" // 导入路径解析器包
    pin "github.com/ipfs/boxo/pinning/pinner" // 导入固定包
    provider "github.com/ipfs/boxo/provider" // 导入提供者包
    offlineroute "github.com/ipfs/boxo/routing/offline" // 导入离线路由包
    ipld "github.com/ipfs/go-ipld-format" // 导入IPLD格式包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入核心接口包
    "github.com/ipfs/kubo/core/coreiface/options" // 导入核心接口选项包
    pubsub "github.com/libp2p/go-libp2p-pubsub" // 导入发布订阅包
    record "github.com/libp2p/go-libp2p-record" // 导入记录包
    ci "github.com/libp2p/go-libp2p/core/crypto" // 导入加密包
    p2phost "github.com/libp2p/go-libp2p/core/host" // 导入主机包
    peer "github.com/libp2p/go-libp2p/core/peer" // 导入对等体包
    pstore "github.com/libp2p/go-libp2p/core/peerstore" // 导入对等体存储包
    routing "github.com/libp2p/go-libp2p/core/routing" // 导入路由包
    madns "github.com/multiformats/go-multiaddr-dns" // 导入多地址DNS包

    "github.com/ipfs/boxo/namesys" // 导入命名系统包
    "github.com/ipfs/kubo/core" // 导入核心包
    "github.com/ipfs/kubo/core/node" // 导入节点包
    "github.com/ipfs/kubo/repo" // 导入存储库包
)

type CoreAPI struct {
    nctx context.Context // 上下文对象

    identity   peer.ID // 对等体ID
    privateKey ci.PrivKey // 私钥

    repo       repo.Repo // 存储库
    blockstore blockstore.GCBlockstore // 垃圾回收块存储
    baseBlocks blockstore.Blockstore // 基本块存储
    pinning    pin.Pinner // 固定

    blocks               bserv.BlockService // 块服务
    dag                  ipld.DAGService // DAG服务
    ipldFetcherFactory   fetcher.Factory // 获取器工厂
    // UnixFSFetcherFactory 是一个用于创建 UnixFSFetcher 的工厂
    unixFSFetcherFactory fetcher.Factory
    // Peerstore 是用于存储节点信息的数据结构
    peerstore pstore.Peerstore
    // PeerHost 是 IPFS 节点的主机实例
    peerHost p2phost.Host
    // RecordValidator 是用于验证记录的接口
    recordValidator record.Validator
    // Exchange 是 IPFS 节点之间进行数据交换的接口
    exchange exchange.Interface

    // Namesys 是用于解析 IPNS 名称的系统
    namesys namesys.NameSystem
    // Routing 是 IPFS 节点之间进行路由选择的接口
    routing routing.Routing
    // DNSResolver 是用于解析 DNS 的解析器
    dnsResolver *madns.Resolver
    // IPLDPathResolver 是用于解析 IPLD 路径的解析器
    ipldPathResolver pathresolver.Resolver
    // UnixFSPathResolver 是用于解析 UnixFS 路径的解析器
    unixFSPathResolver pathresolver.Resolver

    // Provider 是 IPFS 节点的提供者系统
    provider provider.System

    // PubSub 是 IPFS 节点的发布-订阅系统
    pubSub *pubsub.PubSub

    // checkPublishAllowed 是一个用于检查是否允许发布内容的函数
    checkPublishAllowed func() error
    // checkOnline 是一个用于检查节点是否在线的函数
    checkOnline func(allowOffline bool) error

    // 仅用于在 WithOptions 中重新应用选项，不要在其他地方使用
    nd *core.IpfsNode
    // parentOpts 是 API 设置的父级选项
    parentOpts options.ApiSettings
// NewCoreAPI创建一个由go-ipfs节点支持的IPFS CoreAPI的新实例。
func NewCoreAPI(n *core.IpfsNode, opts ...options.ApiOption) (coreiface.CoreAPI, error) {
    // 获取父级选项
    parentOpts, err := options.ApiOptions()
    if err != nil {
        return nil, err
    }
    // 返回带有选项的CoreAPI实例
    return (&CoreAPI{nd: n, parentOpts: *parentOpts}).WithOptions(opts...)
}

// Unixfs返回由go-ipfs节点支持的UnixfsAPI接口实现
func (api *CoreAPI) Unixfs() coreiface.UnixfsAPI {
    return (*UnixfsAPI)(api)
}

// Block返回由go-ipfs节点支持的BlockAPI接口实现
func (api *CoreAPI) Block() coreiface.BlockAPI {
    return (*BlockAPI)(api)
}

// Dag返回由go-ipfs节点支持的DagAPI接口实现
func (api *CoreAPI) Dag() coreiface.APIDagService {
    return &dagAPI{
        api.dag,
        api,
    }
}

// Name返回由go-ipfs节点支持的NameAPI接口实现
func (api *CoreAPI) Name() coreiface.NameAPI {
    return (*NameAPI)(api)
}

// Key返回由go-ipfs节点支持的KeyAPI接口实现
func (api *CoreAPI) Key() coreiface.KeyAPI {
    return (*KeyAPI)(api)
}

// Object返回由go-ipfs节点支持的ObjectAPI接口实现
func (api *CoreAPI) Object() coreiface.ObjectAPI {
    return (*ObjectAPI)(api)
}

// Pin返回由go-ipfs节点支持的PinAPI接口实现
func (api *CoreAPI) Pin() coreiface.PinAPI {
    return (*PinAPI)(api)
}

// Dht返回由go-ipfs节点支持的DhtAPI接口实现
func (api *CoreAPI) Dht() coreiface.DhtAPI {
    return (*DhtAPI)(api)
}

// Swarm返回由go-ipfs节点支持的SwarmAPI接口实现
func (api *CoreAPI) Swarm() coreiface.SwarmAPI {
    return (*SwarmAPI)(api)
}

// PubSub返回由go-ipfs节点支持的PubSubAPI接口实现
func (api *CoreAPI) PubSub() coreiface.PubSubAPI {
    return (*PubSubAPI)(api)
}
// Routing 返回由 kubo 节点支持的 RoutingAPI 接口实现
func (api *CoreAPI) Routing() coreiface.RoutingAPI {
    return (*RoutingAPI)(api)
}

// WithOptions 返回应用全局选项后的 api
func (api *CoreAPI) WithOptions(opts ...options.ApiOption) (coreiface.CoreAPI, error) {
    settings := api.parentOpts // 确保复制
    _, err := options.ApiOptionsTo(&settings, opts...)
    if err != nil {
        return nil, err
    }

    if api.nd == nil {
        return nil, errors.New("无法将选项应用到没有节点的 api")
    }

    n := api.nd

    subAPI := &CoreAPI{
        nctx: n.Context(),

        identity:   n.Identity,
        privateKey: n.PrivateKey,

        repo:       n.Repo,
        blockstore: n.Blockstore,
        baseBlocks: n.BaseBlocks,
        pinning:    n.Pinning,

        blocks:               n.Blocks,
        dag:                  n.DAG,
        ipldFetcherFactory:   n.IPLDFetcherFactory,
        unixFSFetcherFactory: n.UnixFSFetcherFactory,

        peerstore:          n.Peerstore,
        peerHost:           n.PeerHost,
        namesys:            n.Namesys,
        recordValidator:    n.RecordValidator,
        exchange:           n.Exchange,
        routing:            n.Routing,
        dnsResolver:        n.DNSResolver,
        ipldPathResolver:   n.IPLDPathResolver,
        unixFSPathResolver: n.UnixFSPathResolver,

        provider: n.Provider,

        pubSub: n.PubSub,

        nd:         n,
        parentOpts: settings,
    }

    subAPI.checkOnline = func(allowOffline bool) error {
        if !n.IsOnline && !allowOffline {
            return coreiface.ErrOffline
        }
        return nil
    }

    subAPI.checkPublishAllowed = func() error {
        if n.Mounts.Ipns != nil && n.Mounts.Ipns.IsActive() {
            return errors.New("在挂载 IPNS 时无法手动发布")
        }
        return nil
    }
}
    // 如果设置为离线模式
    if settings.Offline {
        // 从存储库中获取配置信息
        cfg, err := n.Repo.Config()
        if err != nil {
            return nil, err
        }

        // 获取IPNS解析缓存大小
        cs := cfg.Ipns.ResolveCacheSize
        // 如果未设置缓存大小，则使用默认值
        if cs == 0 {
            cs = node.DefaultIpnsCacheSize
        }
        // 如果缓存大小为负数，则返回错误
        if cs < 0 {
            return nil, fmt.Errorf("cannot specify negative resolve cache size")
        }

        // 创建离线路由器
        subAPI.routing = offlineroute.NewOfflineRouter(subAPI.repo.Datastore(), subAPI.recordValidator)

        // 创建命名系统
        subAPI.namesys, err = namesys.NewNameSystem(subAPI.routing,
            namesys.WithDatastore(subAPI.repo.Datastore()),
            namesys.WithDNSResolver(subAPI.dnsResolver),
            namesys.WithCache(cs))
        if err != nil {
            return nil, fmt.Errorf("error constructing namesys: %w", err)
        }

        // 创建空的提供程序
        subAPI.provider = provider.NewNoopProvider()

        // 清空对等节点存储、对等主机和记录验证器
        subAPI.peerstore = nil
        subAPI.peerHost = nil
        subAPI.recordValidator = nil
    }

    // 如果设置为离线模式或者不需要获取区块
    if settings.Offline || !settings.FetchBlocks {
        // 创建离线交换
        subAPI.exchange = offlinexch.Exchange(subAPI.blockstore)
        // 创建新的区块服务
        subAPI.blocks = bserv.New(subAPI.blockstore, subAPI.exchange)
        // 创建新的DAG服务
        subAPI.dag = dag.NewDAGService(subAPI.blocks)
    }

    // 返回subAPI和nil
    return subAPI, nil
// getSession函数返回一个新的API，该API由相同节点支持，并具有只读会话DAG
func (api *CoreAPI) getSession(ctx context.Context) *CoreAPI {
    // 复制api对象，以便在其上进行只读会话操作
    sesAPI := *api

    // TODO: 我们也可以将此应用于api.blocks，并组合成可写的api，但这需要在blockservice/merkledag中进行一些更改
    // 使用只读会话创建一个新的只读DAG服务，并将其赋值给sesAPI的dag字段
    sesAPI.dag = dag.NewReadOnlyDagService(dag.NewSession(ctx, api.dag))

    // 返回新的只读会话API
    return &sesAPI
}
```