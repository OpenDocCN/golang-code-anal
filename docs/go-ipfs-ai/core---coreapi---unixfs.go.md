# `kubo\core\coreapi\unixfs.go`

```
package coreapi

import (
    "context" // 上下文包，用于控制程序的执行流程
    "fmt" // 格式化包，用于格式化输出

    "github.com/ipfs/kubo/core" // 导入核心功能包
    "github.com/ipfs/kubo/tracing" // 导入跟踪包
    "go.opentelemetry.io/otel/attribute" // 导入属性包
    "go.opentelemetry.io/otel/trace" // 导入跟踪包

    "github.com/ipfs/kubo/core/coreunix" // 导入核心 Unix 功能包

    blockservice "github.com/ipfs/boxo/blockservice" // 导入块服务包
    bstore "github.com/ipfs/boxo/blockstore" // 导入块存储包
    "github.com/ipfs/boxo/files" // 导入文件包
    filestore "github.com/ipfs/boxo/filestore" // 导入文件存储包
    merkledag "github.com/ipfs/boxo/ipld/merkledag" // 导入 MerkleDAG 包
    dagtest "github.com/ipfs/boxo/ipld/merkledag/test" // 导入 MerkleDAG 测试包
    ft "github.com/ipfs/boxo/ipld/unixfs" // 导入 UnixFS 包
    unixfile "github.com/ipfs/boxo/ipld/unixfs/file" // 导入 UnixFS 文件包
    uio "github.com/ipfs/boxo/ipld/unixfs/io" // 导入 UnixFS IO 包
    mfs "github.com/ipfs/boxo/mfs" // 导入多文件系统包
    "github.com/ipfs/boxo/path" // 导入路径包
    cid "github.com/ipfs/go-cid" // 导入 CID 包
    cidutil "github.com/ipfs/go-cidutil" // 导入 CID 工具包
    ipld "github.com/ipfs/go-ipld-format" // 导入 IPLD 包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入核心接口包
    options "github.com/ipfs/kubo/core/coreiface/options" // 导入核心接口选项包
)

type UnixfsAPI CoreAPI // 定义 UnixFS API 类型，继承自 CoreAPI

var (
    nilNode *core.IpfsNode // 定义空节点指针
    once    sync.Once // 定义一次性执行对象
)

func getOrCreateNilNode() (*core.IpfsNode, error) {
    once.Do(func() { // 一次性执行函数
        if nilNode != nil { // 如果空节点不为空，则返回
            return
        }
        node, err := core.NewNode(context.Background(), &core.BuildCfg{ // 创建新节点
            // TODO: need this to be true or all files
            // hashed will be stored in memory!
            NilRepo: true, // 设置 NilRepo 为 true
        })
        if err != nil { // 如果出错，则抛出异常
            panic(err)
        }
        nilNode = node // 将新节点赋值给空节点
    })

    return nilNode, nil // 返回空节点
}

// Add builds a merkledag node from a reader, adds it to the blockstore,
// and returns the key representing that node.
func (api *UnixfsAPI) Add(ctx context.Context, files files.Node, opts ...options.UnixfsAddOption) (path.ImmutablePath, error) {
    ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Add") // 跟踪函数执行
    defer span.End() // 函数结束时结束跟踪

    settings, prefix, err := options.UnixfsAddOptions(opts...) // 获取 UnixFS 添加选项
    if err != nil { // 如果出错，则返回空路径和错误
        return path.ImmutablePath{}, err
    }
    # 设置 span 的属性，包括 chunker、cidversion、inline 等等
    span.SetAttributes(
        attribute.String("chunker", settings.Chunker),
        attribute.Int("cidversion", settings.CidVersion),
        attribute.Bool("inline", settings.Inline),
        attribute.Int("inlinelimit", settings.InlineLimit),
        attribute.Bool("rawleaves", settings.RawLeaves),
        attribute.Bool("rawleavesset", settings.RawLeavesSet),
        attribute.Int("layout", int(settings.Layout)),
        attribute.Bool("pin", settings.Pin),
        attribute.Bool("onlyhash", settings.OnlyHash),
        attribute.Bool("fscache", settings.FsCache),
        attribute.Bool("nocopy", settings.NoCopy),
        attribute.Bool("silent", settings.Silent),
        attribute.Bool("progress", settings.Progress),
    )

    # 获取 repo 的配置信息
    cfg, err := api.repo.Config()
    if err != nil:
        # 如果出错，返回空的 ImmutablePath 和错误信息
        return path.ImmutablePath{}, err
    }

    # 检查如果添加后的 repo 将超出存储限制
    # TODO: 这里没有处理已经存在于块中（去重）的情况
    # TODO: 由于无法将大小传递给守护程序，条件 GC 被禁用
    #if err := corerepo.ConditionalGC(req.Context(), n, uint64(size)); err != nil {
    #    res.SetError(err, cmds.ErrNormal)
    #    return
    #}

    # 如果设置了 NoCopy 但未启用 filestore 或 urlstore，则返回错误信息
    if settings.NoCopy && !(cfg.Experimental.FilestoreEnabled || cfg.Experimental.UrlstoreEnabled) {
        return path.ImmutablePath{}, fmt.Errorf("either the filestore or the urlstore must be enabled to use nocopy, see: https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-filestore")
    }

    # 如果未设置 FsCache 或 NoCopy，则创建新的 GCBlockstore
    addblockstore := api.blockstore
    if !(settings.FsCache || settings.NoCopy) {
        addblockstore = bstore.NewGCBlockstore(api.baseBlocks, api.blockstore)
    }
    # 设置 exch 和 pinning
    exch := api.exchange
    pinning := api.pinning
    // 如果设置了 OnlyHash 标志，则获取或创建一个空节点
    if settings.OnlyHash {
        node, err := getOrCreateNilNode()
        if err != nil {
            return path.ImmutablePath{}, err
        }
        addblockstore = node.Blockstore
        exch = node.Exchange
        pinning = node.Pinning
    }

    // 创建一个新的块服务，使用添加的块存储和交换机
    bserv := blockservice.New(addblockstore, exch) // hash security 001
    // 创建一个新的有向无环图服务，使用块服务
    dserv := merkledag.NewDAGService(bserv)

    // 添加一个同步调用到DagService
    // 这确保写入DagService的数据被持久化到底层数据存储
    // TODO: 通过块存储、块服务和有向无环图服务传播同步函数
    var syncDserv *syncDagService
    if settings.OnlyHash {
        // 如果设置了 OnlyHash 标志，则创建一个带有空同步函数的同步DagService
        syncDserv = &syncDagService{
            DAGService: dserv,
            syncFn:     func() error { return nil },
        }
    } else {
        // 否则，创建一个带有数据存储同步函数的同步DagService
        syncDserv = &syncDagService{
            DAGService: dserv,
            syncFn: func() error {
                ds := api.repo.Datastore()
                if err := ds.Sync(ctx, bstore.BlockPrefix); err != nil {
                    return err
                }
                return ds.Sync(ctx, filestore.FilestorePrefix)
            },
        }
    }

    // 创建一个新的文件添加器，使用上下文、固定、添加的块存储和同步DagService
    fileAdder, err := coreunix.NewAdder(ctx, pinning, addblockstore, syncDserv)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    // 设置文件添加器的分块器
    fileAdder.Chunker = settings.Chunker
    // 如果设置了事件，则将其输出到文件添加器的输出和进度
    if settings.Events != nil {
        fileAdder.Out = settings.Events
        fileAdder.Progress = settings.Progress
    }
    // 设置文件添加器的固定、静默、原始叶子、不复制和CID构建器
    fileAdder.Pin = settings.Pin && !settings.OnlyHash
    fileAdder.Silent = settings.Silent
    fileAdder.RawLeaves = settings.RawLeaves
    fileAdder.NoCopy = settings.NoCopy
    fileAdder.CidBuilder = prefix

    // 根据设置的布局类型进行不同的处理
    switch settings.Layout {
    case options.BalancedLayout:
        // 默认处理
    case options.TrickleLayout:
        // 如果设置了 TrickleLayout，则设置文件添加器的 Trickle 标志为 true
        fileAdder.Trickle = true
    default:
        // 如果设置了未知的布局类型，则返回错误
        return path.ImmutablePath{}, fmt.Errorf("unknown layout: %d", settings.Layout)
    }
    # 如果设置为内联模式
    if settings.Inline {
        # 设置文件添加器的 CID 构建器为内联构建器
        fileAdder.CidBuilder = cidutil.InlineBuilder{
            Builder: fileAdder.CidBuilder,
            Limit:   settings.InlineLimit,
        }
    }

    # 如果设置为仅哈希模式
    if settings.OnlyHash {
        # 创建一个模拟的 DAG
        md := dagtest.Mock()
        # 创建一个空的目录节点
        emptyDirNode := ft.EmptyDirNode()
        # 使用与文件添加器相同的前缀为“空”MFS根节点设置 CID 构建器
        err := emptyDirNode.SetCidBuilder(fileAdder.CidBuilder)
        if err != nil {
            return path.ImmutablePath{}, err
        }
        # 创建一个新的 MFS 根节点
        mr, err := mfs.NewRoot(ctx, md, emptyDirNode, nil)
        if err != nil {
            return path.ImmutablePath{}, err
        }

        # 设置文件添加器的 MFS 根节点
        fileAdder.SetMfsRoot(mr)
    }

    # 添加所有文件并固定
    nd, err := fileAdder.AddAllAndPin(ctx, files)
    if err != nil {
        return path.ImmutablePath{}, err
    }

    # 如果不是仅哈希模式
    if !settings.OnlyHash {
        # 如果提供者无法提供节点的 CID
        if err := api.provider.Provide(nd.Cid()); err != nil {
            return path.ImmutablePath{}, err
        }
    }

    # 返回节点的路径和空错误
    return path.FromCid(nd.Cid()), nil
// Get方法用于获取指定路径的节点，并返回节点和错误信息
func (api *UnixfsAPI) Get(ctx context.Context, p path.Path) (files.Node, error) {
    // 创建一个跟踪span，记录方法名和路径信息
    ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Get", trace.WithAttributes(attribute.String("path", p.String())))
    // 延迟结束span
    defer span.End()

    // 获取API核心的会话
    ses := api.core().getSession(ctx)

    // 解析指定路径的节点
    nd, err := ses.ResolveNode(ctx, p)
    if err != nil {
        return nil, err
    }

    // 返回Unixfs文件
    return unixfile.NewUnixfsFile(ctx, ses.dag, nd)
}

// Ls方法返回指定路径的IPFS或IPNS对象的内容
func (api *UnixfsAPI) Ls(ctx context.Context, p path.Path, opts ...options.UnixfsLsOption) (<-chan coreiface.DirEntry, error) {
    // 创建一个跟踪span，记录方法名和路径信息
    ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Ls", trace.WithAttributes(attribute.String("path", p.String())))
    // 延迟结束span
    defer span.End()

    // 获取UnixfsLsOptions
    settings, err := options.UnixfsLsOptions(opts...)
    if err != nil {
        return nil, err
    }

    // 设置解析子节点的属性
    span.SetAttributes(attribute.Bool("resolvechildren", settings.ResolveChildren))

    // 获取API核心的会话
    ses := api.core().getSession(ctx)
    uses := (*UnixfsAPI)(ses)

    // 解析指定路径的节点
    dagnode, err := ses.ResolveNode(ctx, p)
    if err != nil {
        return nil, err
    }

    // 从节点创建目录
    dir, err := uio.NewDirectoryFromNode(ses.dag, dagnode)
    if err == uio.ErrNotADir {
        return uses.lsFromLinks(ctx, dagnode.Links(), settings)
    }
    if err != nil {
        return nil, err
    }

    // 从链接异步获取目录内容
    return uses.lsFromLinksAsync(ctx, dir, settings)
}

// processLink方法用于处理链接结果
func (api *UnixfsAPI) processLink(ctx context.Context, linkres ft.LinkResult, settings *options.UnixfsLsSettings) coreiface.DirEntry {
    // 创建一个跟踪span，记录方法名
    ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "ProcessLink")
    // 延迟结束span
    defer span.End()
    if linkres.Link != nil {
        // 设置链接名和CID属性
        span.SetAttributes(attribute.String("linkname", linkres.Link.Name), attribute.String("cid", linkres.Link.Cid.String()))
    }

    if linkres.Err != nil {
        return coreiface.DirEntry{Err: linkres.Err}
    }
}
    # 创建一个coreiface.DirEntry结构体，包含文件名和CID
    lnk := coreiface.DirEntry{
        Name: linkres.Link.Name,
        Cid:  linkres.Link.Cid,
    }

    # 根据CID的类型进行不同的处理
    switch lnk.Cid.Type() {
    case cid.Raw:
        # 对于原始叶子节点，无需进行检查
        lnk.Type = coreiface.TFile
        lnk.Size = linkres.Link.Size
    case cid.DagProtobuf:
        # 如果设置了解析子节点
        if settings.ResolveChildren {
            # 获取链接节点的数据
            linkNode, err := linkres.Link.GetNode(ctx, api.dag)
            if err != nil {
                lnk.Err = err
                break
            }

            # 如果节点是ProtoNode类型
            if pn, ok := linkNode.(*merkledag.ProtoNode); ok {
                # 从ProtoNode数据中获取FSNode
                d, err := ft.FSNodeFromBytes(pn.Data())
                if err != nil {
                    lnk.Err = err
                    break
                }
                # 根据FSNode的类型进行处理
                switch d.Type() {
                case ft.TFile, ft.TRaw:
                    lnk.Type = coreiface.TFile
                case ft.THAMTShard, ft.TDirectory, ft.TMetadata:
                    lnk.Type = coreiface.TDirectory
                case ft.TSymlink:
                    lnk.Type = coreiface.TSymlink
                    lnk.Target = string(d.Data())
                }
                # 如果不使用累积大小，则设置文件大小
                if !settings.UseCumulativeSize {
                    lnk.Size = d.FileSize()
                }
            }
        }

        # 如果使用累积大小，则设置文件大小
        if settings.UseCumulativeSize {
            lnk.Size = linkres.Link.Size
        }
    }

    # 返回处理后的lnk结构体
    return lnk
// 从链接异步枚举目录，并返回目录条目通道和错误
func (api *UnixfsAPI) lsFromLinksAsync(ctx context.Context, dir uio.Directory, settings *options.UnixfsLsSettings) (<-chan coreiface.DirEntry, error) {
    out := make(chan coreiface.DirEntry, uio.DefaultShardWidth)

    go func() {
        defer close(out)
        for l := range dir.EnumLinksAsync(ctx) {
            select {
            case out <- api.processLink(ctx, l, settings): // TODO: perf: processing can be done in background and in parallel
            case <-ctx.Done():
                return
            }
        }
    }()

    return out, nil
}

// 从链接同步枚举目录，并返回目录条目通道和错误
func (api *UnixfsAPI) lsFromLinks(ctx context.Context, ndlinks []*ipld.Link, settings *options.UnixfsLsSettings) (<-chan coreiface.DirEntry, error) {
    links := make(chan coreiface.DirEntry, len(ndlinks))
    for _, l := range ndlinks {
        lr := ft.LinkResult{Link: &ipld.Link{Name: l.Name, Size: l.Size, Cid: l.Cid}}

        links <- api.processLink(ctx, lr, settings) // TODO: can be parallel if settings.Async
    }
    close(links)
    return links, nil
}

// 返回核心API
func (api *UnixfsAPI) core() *CoreAPI {
    return (*CoreAPI)(api)
}

// syncDagService 由Adder使用，以确保块被持久化到底层数据存储
type syncDagService struct {
    ipld.DAGService
    syncFn func() error
}

// 同步DAG服务用于同步块到底层数据存储
func (s *syncDagService) Sync() error {
    return s.syncFn()
}
```