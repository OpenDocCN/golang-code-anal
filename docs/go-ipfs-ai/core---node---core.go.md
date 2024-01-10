# `kubo\core\node\core.go`

```
package node

import (
    "context" // 导入上下文包，用于处理请求的取消、超时等操作
    "fmt" // 导入格式化包，用于格式化输出

    "github.com/ipfs/boxo/blockservice" // 导入区块服务包，提供获取内容可寻址区块的接口
    blockstore "github.com/ipfs/boxo/blockstore" // 导入区块存储包，用于存储区块数据
    exchange "github.com/ipfs/boxo/exchange" // 导入交换包，用于节点之间的数据交换
    offline "github.com/ipfs/boxo/exchange/offline" // 导入离线交换包，用于离线数据交换
    "github.com/ipfs/boxo/fetcher" // 导入获取器包，用于获取数据
    bsfetcher "github.com/ipfs/boxo/fetcher/impl/blockservice" // 导入区块服务获取器包，用于从区块服务获取数据
    "github.com/ipfs/boxo/filestore" // 导入文件存储包，用于文件存储
    "github.com/ipfs/boxo/ipld/merkledag" // 导入默克尔有向无环图包，用于处理 IPLD 数据
    "github.com/ipfs/boxo/ipld/unixfs" // 导入 UnixFS 包，用于处理 UnixFS 数据
    "github.com/ipfs/boxo/mfs" // 导入多文件系统包，用于操作多文件系统
    pathresolver "github.com/ipfs/boxo/path/resolver" // 导入路径解析器包，用于解析路径
    pin "github.com/ipfs/boxo/pinning/pinner" // 导入固定包，用于固定数据块
    "github.com/ipfs/boxo/pinning/pinner/dspinner" // 导入分布式固定包，用于分布式固定数据块
    "github.com/ipfs/go-cid" // 导入 CID 包，用于处理内容标识符
    "github.com/ipfs/go-datastore" // 导入数据存储包，用于数据存储
    format "github.com/ipfs/go-ipld-format" // 导入 IPLD 格式包，用于处理 IPLD 数据格式
    "github.com/ipfs/go-unixfsnode" // 导入 UnixFS 节点包，用于处理 UnixFS 节点
    dagpb "github.com/ipld/go-codec-dagpb" // 导入 DAGPB 编解码包，用于处理 DAGPB 数据
    "go.uber.org/fx" // 导入 Uber Fx 包，用于依赖注入

    "github.com/ipfs/kubo/core/node/helpers" // 导入节点助手包，用于节点辅助功能
    "github.com/ipfs/kubo/repo" // 导入仓库包，用于处理节点仓库
)

// BlockService creates new blockservice which provides an interface to fetch content-addressable blocks
// 创建新的区块服务，提供获取内容可寻址区块的接口
func BlockService(lc fx.Lifecycle, bs blockstore.Blockstore, rem exchange.Interface) blockservice.BlockService {
    // 创建新的区块服务
    bsvc := blockservice.New(bs, rem)

    // 在生命周期结束时关闭区块服务
    lc.Append(fx.Hook{
        OnStop: func(ctx context.Context) error {
            return bsvc.Close()
        },
    })

    return bsvc // 返回区块服务
}

// Pinning creates new pinner which tells GC which blocks should be kept
// 创建新的固定器，告诉 GC 应该保留哪些区块
func Pinning(bstore blockstore.Blockstore, ds format.DAGService, repo repo.Repo) (pin.Pinner, error) {
    rootDS := repo.Datastore() // 获取仓库数据存储

    syncFn := func(ctx context.Context) error {
        if err := rootDS.Sync(ctx, blockstore.BlockPrefix); err != nil { // 同步区块数据存储
            return err
        }
        return rootDS.Sync(ctx, filestore.FilestorePrefix) // 同步文件数据存储
    }
    syncDs := &syncDagService{ds, syncFn} // 创建同步 DAG 服务

    ctx := context.TODO() // 创建上下文

    pinning, err := dspinner.New(ctx, rootDS, syncDs) // 创建新的分布式固定器
    if err != nil {
        return nil, err
    }

    return pinning, nil // 返回固定器
}

var (
    # 创建一个新的 merkledag.SessionMaker 实例，并赋值给 syncDagService 类型的变量
    _ merkledag.SessionMaker = new(syncDagService)
    # 创建一个新的 format.DAGService 实例，并赋值给 syncDagService 类型的变量
    _ format.DAGService      = new(syncDagService)
// syncDagService用于确保数据被持久化到底层数据存储中
type syncDagService struct {
    format.DAGService
    syncFn func(context.Context) error
}

// Sync方法用于同步数据
func (s *syncDagService) Sync(ctx context.Context) error {
    return s.syncFn(ctx)
}

// Session方法返回一个新的节点获取器
func (s *syncDagService) Session(ctx context.Context) format.NodeGetter {
    return merkledag.NewSession(ctx, s.DAGService)
}

// FetchersOut允许注入获取器
type FetchersOut struct {
    fx.Out
    IPLDFetcher          fetcher.Factory `name:"ipldFetcher"`
    UnixfsFetcher        fetcher.Factory `name:"unixfsFetcher"`
    OfflineIPLDFetcher   fetcher.Factory `name:"offlineIpldFetcher"`
    OfflineUnixfsFetcher fetcher.Factory `name:"offlineUnixfsFetcher"`
}

// FetchersIn允许使用获取器作为其他依赖项
type FetchersIn struct {
    fx.In
    IPLDFetcher          fetcher.Factory `name:"ipldFetcher"`
    UnixfsFetcher        fetcher.Factory `name:"unixfsFetcher"`
    OfflineIPLDFetcher   fetcher.Factory `name:"offlineIpldFetcher"`
    OfflineUnixfsFetcher fetcher.Factory `name:"offlineUnixfsFetcher"`
}

// FetcherConfig返回一个可以构建新获取器实例的获取器配置
func FetcherConfig(bs blockservice.BlockService) FetchersOut {
    ipldFetcher := bsfetcher.NewFetcherConfig(bs)
    ipldFetcher.PrototypeChooser = dagpb.AddSupportToChooser(bsfetcher.DefaultPrototypeChooser)
    unixFSFetcher := ipldFetcher.WithReifier(unixfsnode.Reify)

    // 构建离线版本，可以在不通过交换获取新块的情况下安全使用
    offlineBs := blockservice.New(bs.Blockstore(), offline.Exchange(bs.Blockstore()))
    offlineIpldFetcher := bsfetcher.NewFetcherConfig(offlineBs)
    offlineIpldFetcher.PrototypeChooser = dagpb.AddSupportToChooser(bsfetcher.DefaultPrototypeChooser)
    offlineUnixFSFetcher := offlineIpldFetcher.WithReifier(unixfsnode.Reify)
}
    # 返回一个包含不同类型数据获取器的结构体
    return FetchersOut{
        # 将ipldFetcher赋值给IPLDFetcher字段
        IPLDFetcher:          ipldFetcher,
        # 将unixFSFetcher赋值给UnixfsFetcher字段
        UnixfsFetcher:        unixFSFetcher,
        # 将offlineIpldFetcher赋值给OfflineIPLDFetcher字段
        OfflineIPLDFetcher:   offlineIpldFetcher,
        # 将offlineUnixFSFetcher赋值给OfflineUnixfsFetcher字段
        OfflineUnixfsFetcher: offlineUnixFSFetcher,
    }
// PathResolversOut 允许注入路径解析器
type PathResolversOut struct {
    fx.Out
    IPLDPathResolver          pathresolver.Resolver `name:"ipldPathResolver"`
    UnixFSPathResolver        pathresolver.Resolver `name:"unixFSPathResolver"`
    OfflineIPLDPathResolver   pathresolver.Resolver `name:"offlineIpldPathResolver"`
    OfflineUnixFSPathResolver pathresolver.Resolver `name:"offlineUnixFSPathResolver"`
}

// PathResolverConfig 使用给定的获取器创建路径解析器
func PathResolverConfig(fetchers FetchersIn) PathResolversOut {
    return PathResolversOut{
        IPLDPathResolver:          pathresolver.NewBasicResolver(fetchers.IPLDFetcher),
        UnixFSPathResolver:        pathresolver.NewBasicResolver(fetchers.UnixfsFetcher),
        OfflineIPLDPathResolver:   pathresolver.NewBasicResolver(fetchers.OfflineIPLDFetcher),
        OfflineUnixFSPathResolver: pathresolver.NewBasicResolver(fetchers.OfflineUnixfsFetcher),
    }
}

// Dag 创建新的 DAGService
func Dag(bs blockservice.BlockService) format.DAGService {
    return merkledag.NewDAGService(bs)
}

// Files 加载持久化的 MFS 根目录
func Files(mctx helpers.MetricsCtx, lc fx.Lifecycle, repo repo.Repo, dag format.DAGService) (*mfs.Root, error) {
    dsk := datastore.NewKey("/local/filesroot")
    pf := func(ctx context.Context, c cid.Cid) error {
        rootDS := repo.Datastore()
        if err := rootDS.Sync(ctx, blockstore.BlockPrefix); err != nil {
            return err
        }
        if err := rootDS.Sync(ctx, filestore.FilestorePrefix); err != nil {
            return err
        }

        if err := rootDS.Put(ctx, dsk, c.Bytes()); err != nil {
            return err
        }
        return rootDS.Sync(ctx, dsk)
    }

    var nd *merkledag.ProtoNode
    ctx := helpers.LifecycleCtx(mctx, lc)
    val, err := repo.Datastore().Get(ctx, dsk)

    switch {
    # 如果数据未找到或者值为空
    case err == datastore.ErrNotFound || val == nil:
        # 创建一个空的目录节点
        nd = unixfs.EmptyDirNode()
        # 将节点添加到有向无环图中
        err := dag.Add(ctx, nd)
        # 如果添加失败，返回错误信息
        if err != nil {
            return nil, fmt.Errorf("failure writing to dagstore: %s", err)
        }
    # 如果没有错误
    case err == nil:
        # 将值转换为CID
        c, err := cid.Cast(val)
        # 如果转换失败，返回错误信息
        if err != nil {
            return nil, err
        }
        # 从有向无环图中获取节点
        rnd, err := dag.Get(ctx, c)
        # 如果获取失败，返回错误信息
        if err != nil {
            return nil, fmt.Errorf("error loading filesroot from DAG: %s", err)
        }
        # 将节点转换为ProtoNode
        pbnd, ok := rnd.(*merkledag.ProtoNode)
        # 如果转换失败，返回错误信息
        if !ok {
            return nil, merkledag.ErrNotProtobuf
        }
        # 将节点赋值给nd
        nd = pbnd
    # 如果出现其他情况，返回错误信息
    default:
        return nil, err
    }

    # 创建一个新的Merkle根节点
    root, err := mfs.NewRoot(ctx, dag, nd, pf)

    # 在生命周期中添加一个钩子函数，在停止时关闭根节点
    lc.Append(fx.Hook{
        OnStop: func(ctx context.Context) error {
            return root.Close()
        },
    })

    # 返回根节点和错误信息
    return root, err
# 闭合前面的函数定义
```