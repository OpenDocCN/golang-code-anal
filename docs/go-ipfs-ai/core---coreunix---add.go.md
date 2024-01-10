# `kubo\core\coreunix\add.go`

```
package coreunix

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "errors"   // 错误处理包，用于处理错误信息
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，提供了基本的输入输出功能
    gopath "path"  // 路径包，用于处理文件路径
    "strconv"  // 字符串转换包，用于字符串和基本数据类型之间的转换

    bstore "github.com/ipfs/boxo/blockstore"  // 引入外部包，用于块存储
    chunker "github.com/ipfs/boxo/chunker"   // 引入外部包，用于分块处理
    "github.com/ipfs/boxo/files"             // 引入外部包，用于文件操作
    posinfo "github.com/ipfs/boxo/filestore/posinfo"  // 引入外部包，用于文件位置信息
    dag "github.com/ipfs/boxo/ipld/merkledag"  // 引入外部包，用于MerkleDAG操作
    "github.com/ipfs/boxo/ipld/unixfs"        // 引入外部包，用于UnixFS操作
    "github.com/ipfs/boxo/ipld/unixfs/importer/balanced"  // 引入外部包，用于平衡导入
    ihelper "github.com/ipfs/boxo/ipld/unixfs/importer/helpers"  // 引入外部包，用于UnixFS导入助手
    "github.com/ipfs/boxo/ipld/unixfs/importer/trickle"  // 引入外部包，用于滴水导入
    "github.com/ipfs/boxo/mfs"               // 引入外部包，用于多文件系统操作
    "github.com/ipfs/boxo/path"              // 引入外部包，用于路径操作
    pin "github.com/ipfs/boxo/pinning/pinner"  // 引入外部包，用于固定操作
    "github.com/ipfs/go-cid"                 // 引入外部包，用于CID操作
    ipld "github.com/ipfs/go-ipld-format"     // 引入外部包，用于IPLD格式操作
    logging "github.com/ipfs/go-log"          // 引入外部包，用于日志记录
    coreiface "github.com/ipfs/kubo/core/coreiface"  // 引入外部包，用于核心接口操作

    "github.com/ipfs/kubo/tracing"           // 引入外部包，用于跟踪操作
)

var log = logging.Logger("coreunix")  // 定义日志记录器

// how many bytes of progress to wait before sending a progress update message
const progressReaderIncrement = 1024 * 256  // 定义进度更新消息的等待字节数

var liveCacheSize = uint64(256 << 10)  // 定义缓存大小

type Link struct {  // 定义Link结构体
    Name, Hash string  // 文件名和哈希值
    Size       uint64  // 文件大小
}

type syncer interface {  // 定义同步器接口
    Sync() error  // 同步操作
}

// NewAdder Returns a new Adder used for a file add operation.
func NewAdder(ctx context.Context, p pin.Pinner, bs bstore.GCLocker, ds ipld.DAGService) (*Adder, error) {
    bufferedDS := ipld.NewBufferedDAG(ctx, ds)  // 创建缓冲DAG对象

    return &Adder{  // 返回Adder对象
        ctx:        ctx,  // 上下文
        pinning:    p,    // 固定
        gcLocker:   bs,   // 块存储锁
        dagService: ds,   // DAG服务
        bufferedDS: bufferedDS,  // 缓冲DAG对象
        Progress:   false,  // 进度
        Pin:        true,   // 固定
        Trickle:    false,  // 滴水
        Chunker:    "",     // 分块器
    }, nil  // 返回Adder对象和nil错误
}

// Adder holds the switches passed to the `add` command.
type Adder struct {  // 定义Adder结构体
    ctx        context.Context  // 上下文
    pinning    pin.Pinner       // 固定
    gcLocker   bstore.GCLocker  // 块存储锁
    dagService ipld.DAGService  // DAG服务
    bufferedDS *ipld.BufferedDAG  // 缓冲DAG对象
    Out        chan<- interface{}  // 输出通道
    Progress   bool  // 进度
    Pin        bool  // 固定
    Trickle    bool  // 滴水
    RawLeaves  bool  // 原始叶子
    # 定义一个布尔类型的变量，用于表示是否为静默模式
    Silent     bool
    # 定义一个布尔类型的变量，用于表示是否禁止复制
    NoCopy     bool
    # 定义一个字符串类型的变量，用于表示分块器
    Chunker    string
    # 定义一个指向mfs.Root类型的指针变量，用于表示根节点
    mroot      *mfs.Root
    # 定义一个bstore.Unlocker类型的变量，用于表示解锁器
    unlocker   bstore.Unlocker
    # 定义一个cid.Cid类型的变量，用于表示临时根节点的CID
    tempRoot   cid.Cid
    # 定义一个cid.Builder类型的变量，用于表示CID构建器
    CidBuilder cid.Builder
    # 定义一个无符号整数类型的变量，用于表示活跃节点数
    liveNodes  uint64
// 返回 Adder 的 mfs 根节点
func (adder *Adder) mfsRoot() (*mfs.Root, error) {
    // 如果 mroot 不为空，则返回 mroot 和 nil
    if adder.mroot != nil {
        return adder.mroot, nil
    }
    // 创建一个空的目录节点
    rnode := unixfs.EmptyDirNode()
    // 设置节点的 CidBuilder
    err := rnode.SetCidBuilder(adder.CidBuilder)
    if err != nil {
        return nil, err
    }
    // 使用给定的上下文、dag 服务、节点和选项创建一个新的根节点
    mr, err := mfs.NewRoot(adder.ctx, adder.dagService, rnode, nil)
    if err != nil {
        return nil, err
    }
    // 将 mr 赋值给 mroot
    adder.mroot = mr
    return adder.mroot, nil
}

// 设置 `r` 为 Adder 的根节点
func (adder *Adder) SetMfsRoot(r *mfs.Root) {
    adder.mroot = r
}

// 从读取器的数据构造一个节点，并添加它。不进行固定
func (adder *Adder) add(reader io.Reader) (ipld.Node, error) {
    // 使用给定的读取器和分块器创建一个块
    chnk, err := chunker.FromString(reader, adder.Chunker)
    if err != nil {
        return nil, err
    }

    // 设置 DAG 构建器参数
    params := ihelper.DagBuilderParams{
        Dagserv:    adder.bufferedDS,
        RawLeaves:  adder.RawLeaves,
        Maxlinks:   ihelper.DefaultLinksPerBlock,
        NoCopy:     adder.NoCopy,
        CidBuilder: adder.CidBuilder,
    }

    // 使用给定的参数和块创建一个新的 DAG
    db, err := params.New(chnk)
    if err != nil {
        return nil, err
    }
    var nd ipld.Node
    // 如果 Trickle 为真，则使用 trickle 布局，否则使用 balanced 布局
    if adder.Trickle {
        nd, err = trickle.Layout(db)
    } else {
        nd, err = balanced.Layout(db)
    }
    if err != nil {
        return nil, err
    }

    // 返回节点和缓冲数据存储的提交状态
    return nd, adder.bufferedDS.Commit()
}

// 返回当前的 mfs 根节点
func (adder *Adder) curRootNode() (ipld.Node, error) {
    // 获取 mfs 根节点
    mr, err := adder.mfsRoot()
    if err != nil {
        return nil, err
    }
    // 获取根目录节点
    root, err := mr.GetDirectory().GetNode()
    if err != nil {
        return nil, err
    }

    // 如果只有一个根文件，则使用该哈希作为根
    if len(root.Links()) == 1 {
        // 获取根节点
        nd, err := root.Links()[0].GetNode(adder.ctx, adder.dagService)
        if err != nil {
            return nil, err
        }

        root = nd
    }

    return root, err
}

// 递归固定 Adder 的根节点，并将固定状态写入后备数据存储
func (adder *Adder) PinRoot(ctx context.Context, root ipld.Node) error {
    // 创建一个新的上下文，并创建一个跟踪 span
    ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "PinRoot")
    // 在函数返回时结束 span
    defer span.End()

    // 如果不需要 Pin，直接返回
    if !adder.Pin {
        return nil
    }

    // 获取根节点的 CID
    rnk := root.Cid()

    // 将根节点添加到 DAG 服务中
    err := adder.dagService.Add(ctx, root)
    if err != nil {
        return err
    }

    // 如果临时根节点已定义，则取消固定
    if adder.tempRoot.Defined() {
        err := adder.pinning.Unpin(ctx, adder.tempRoot, true)
        if err != nil {
            return err
        }
        // 更新临时根节点为当前根节点的 CID
        adder.tempRoot = rnk
    }

    // 固定当前根节点
    err = adder.pinning.PinWithMode(ctx, rnk, pin.Recursive, "")
    if err != nil {
        return err
    }

    // 刷新固定状态
    return adder.pinning.Flush(ctx)
}

func (adder *Adder) outputDirs(path string, fsn mfs.FSNode) error {
    // 根据 FSNode 类型进行不同的处理
    switch fsn := fsn.(type) {
    case *mfs.File:
        return nil
    case *mfs.Directory:
        // 获取目录下的所有文件名
        names, err := fsn.ListNames(adder.ctx)
        if err != nil {
            return err
        }

        // 遍历目录下的文件和子目录
        for _, name := range names {
            child, err := fsn.Child(name)
            if err != nil {
                return err
            }

            // 递归处理子目录
            childpath := gopath.Join(path, name)
            err = adder.outputDirs(childpath, child)
            if err != nil {
                return err
            }

            // 清除缓存
            fsn.Uncache(name)
        }
        // 获取目录节点
        nd, err := fsn.GetNode()
        if err != nil {
            return err
        }

        // 输出 Dagnode
        return outputDagnode(adder.Out, path, nd)
    default:
        return fmt.Errorf("unrecognized fsn type: %#v", fsn)
    }
}

func (adder *Adder) addNode(node ipld.Node, path string) error {
    // 如果路径为空，则使用节点的 CID 作为路径
    if path == "" {
        path = node.Cid().String()
    }

    // 如果节点是 FilestoreNode，则获取其 Node
    if pi, ok := node.(*posinfo.FilestoreNode); ok {
        node = pi.Node
    }

    // 获取 MFS 根节点
    mr, err := adder.mfsRoot()
    if err != nil {
        return err
    }
    // 获取路径所在的目录
    dir := gopath.Dir(path)
}
    # 如果目录不是当前目录
    if dir != "." {
        # 设置创建目录的选项
        opts := mfs.MkdirOpts{
            Mkparents:  true,  # 创建所有父目录
            Flush:      false,  # 不立即刷新到磁盘
            CidBuilder: adder.CidBuilder,  # 使用指定的 CidBuilder
        }
        # 创建目录
        if err := mfs.Mkdir(mr, dir, opts); err != nil {
            return err  # 如果出错，返回错误
        }
    }

    # 将节点放入 MFS 中指定的路径
    if err := mfs.PutNode(mr, path, node); err != nil {
        return err  # 如果出错，返回错误
    }

    # 如果不是静默模式，输出 Dagnode
    if !adder.Silent {
        return outputDagnode(adder.Out, path, node)  # 输出 Dagnode
    }
    return nil  # 返回空值
// AddAllAndPin 函数用于添加给定请求的文件并将其固定。
func (adder *Adder) AddAllAndPin(ctx context.Context, file files.Node) (ipld.Node, error) {
    // 创建一个跟踪 span
    ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "AddAllAndPin")
    // 在函数返回前结束 span
    defer span.End()

    // 如果需要固定文件，则获取固定锁
    if adder.Pin {
        adder.unlocker = adder.gcLocker.PinLock(ctx)
    }
    // 在函数返回前解锁
    defer func() {
        if adder.unlocker != nil {
            adder.unlocker.Unlock(ctx)
        }
    }()

    // 如果添加文件节点出错，则返回错误
    if err := adder.addFileNode(ctx, "", file, true); err != nil {
        return nil, err
    }

    // 获取根节点
    mr, err := adder.mfsRoot()
    if err != nil {
        return nil, err
    }
    var root mfs.FSNode
    rootdir := mr.GetDirectory()
    root = rootdir

    // 刷新根节点
    err = root.Flush()
    if err != nil {
        return nil, err
    }

    // 如果添加的是一个文件而不是一个目录，则将根节点替换为该文件
    _, dir := file.(files.Directory)
    var name string
    if !dir {
        children, err := rootdir.ListNames(adder.ctx)
        if err != nil {
            return nil, err
        }

        if len(children) == 0 {
            return nil, fmt.Errorf("expected at least one child dir, got none")
        }

        // 用第一个子节点替换根节点
        name = children[0]
        root, err = rootdir.Child(name)
        if err != nil {
            return nil, err
        }
    }

    // 关闭 MFS
    err = mr.Close()
    if err != nil {
        return nil, err
    }

    // 获取节点
    nd, err := root.GetNode()
    if err != nil {
        return nil, err
    }

    // 输出目录事件
    err = adder.outputDirs(name, root)
    if err != nil {
        return nil, err
    }

    // 如果 dagService 是同步器，则同步
    if asyncDagService, ok := adder.dagService.(syncer); ok {
        err = asyncDagService.Sync()
        if err != nil {
            return nil, err
        }
    }

    // 如果不需要固定文件，则返回节点
    if !adder.Pin {
        return nd, nil
    }
    // 否则固定根节点
    return nd, adder.PinRoot(ctx, nd)
}
func (adder *Adder) addFileNode(ctx context.Context, path string, file files.Node, toplevel bool) error {
    // 创建一个新的 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "AddFileNode")
    defer span.End()

    // 在函数结束时关闭文件
    defer file.Close()

    // 可能暂停 GC
    err := adder.maybePauseForGC(ctx)
    if err != nil {
        return err
    }

    // 如果 liveNodes 达到缓存大小，刷新内存并重置 liveNodes
    if adder.liveNodes >= liveCacheSize {
        // TODO: 使用某种带有驱逐处理程序的 lru 缓存的更智能的缓存
        mr, err := adder.mfsRoot()
        if err != nil {
            return err
        }
        if err := mr.FlushMemFree(adder.ctx); err != nil {
            return err
        }

        adder.liveNodes = 0
    }
    adder.liveNodes++

    // 根据文件类型执行不同的操作
    switch f := file.(type) {
    case files.Directory:
        return adder.addDir(ctx, path, f, toplevel)
    case *files.Symlink:
        return adder.addSymlink(path, f)
    case files.File:
        return adder.addFile(path, f)
    default:
        return errors.New("unknown file type")
    }
}

func (adder *Adder) addSymlink(path string, l *files.Symlink) error {
    // 获取符号链接的数据并创建相应的 DAG 节点
    sdata, err := unixfs.SymlinkData(l.Target)
    if err != nil {
        return err
    }

    dagnode := dag.NodeWithData(sdata)
    err = dagnode.SetCidBuilder(adder.CidBuilder)
    if err != nil {
        return err
    }
    err = adder.dagService.Add(adder.ctx, dagnode)
    if err != nil {
        return err
    }

    return adder.addNode(dagnode, path)
}

func (adder *Adder) addFile(path string, file files.File) error {
    // 如果指定了进度标志，则包装文件以便我们可以向客户端发送进度更新
    var reader io.Reader = file
    if adder.Progress {
        rdr := &progressReader{file: reader, path: path, out: adder.Out}
        if fi, ok := file.(files.FileInfo); ok {
            reader = &progressReader2{rdr, fi}
        } else {
            reader = rdr
        }
    }

    // 添加文件并返回相应的 DAG 节点
    dagnode, err := adder.add(reader)
    if err != nil {
        return err
    }
}
    // 将节点添加到根节点
    return adder.addNode(dagnode, path)
// addDir 方法用于向存储添加目录及其内容
func (adder *Adder) addDir(ctx context.Context, path string, dir files.Directory, toplevel bool) error {
    // 打印日志，记录添加的目录路径
    log.Infof("adding directory: %s", path)

    // 如果不是顶层目录或者路径不为空，则执行以下操作
    if !(toplevel && path == "") {
        // 获取MFS根节点
        mr, err := adder.mfsRoot()
        if err != nil {
            return err
        }
        // 在MFS中创建目录
        err = mfs.Mkdir(mr, path, mfs.MkdirOpts{
            Mkparents:  true,
            Flush:      false,
            CidBuilder: adder.CidBuilder,
        })
        if err != nil {
            return err
        }
    }

    // 遍历目录中的文件和子目录
    it := dir.Entries()
    for it.Next() {
        // 拼接文件路径
        fpath := gopath.Join(path, it.Name())
        // 添加文件节点到存储
        err := adder.addFileNode(ctx, fpath, it.Node(), false)
        if err != nil {
            return err
        }
    }

    // 返回遍历过程中的错误
    return it.Err()
}

// maybePauseForGC 方法用于在可能的情况下暂停进行垃圾回收
func (adder *Adder) maybePauseForGC(ctx context.Context) error {
    // 创建追踪span
    ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "MaybePauseForGC")
    defer span.End()

    // 如果存在解锁器并且存在垃圾回收请求，则执行以下操作
    if adder.unlocker != nil && adder.gcLocker.GCRequested(ctx) {
        // 获取当前根节点
        rn, err := adder.curRootNode()
        if err != nil {
            return err
        }

        // 将根节点固定到存储
        err = adder.PinRoot(ctx, rn)
        if err != nil {
            return err
        }

        // 解锁并获取垃圾回收锁
        adder.unlocker.Unlock(ctx)
        adder.unlocker = adder.gcLocker.PinLock(ctx)
    }
    return nil
}

// outputDagnode 方法用于向输出通道发送dagnode信息
func outputDagnode(out chan<- interface{}, name string, dn ipld.Node) error {
    // 如果输出通道为空，则直接返回
    if out == nil {
        return nil
    }

    // 获取输出信息
    o, err := getOutput(dn)
    if err != nil {
        return err
    }

    // 向输出通道发送coreiface.AddEvent类型的数据
    out <- &coreiface.AddEvent{
        Path: o.Path,
        Name: name,
        Size: o.Size,
    }

    return nil
}

// getOutput 方法用于获取dagnode的输出信息
// 来自core/commands/object.go
func getOutput(dagnode ipld.Node) (*coreiface.AddEvent, error) {
    // 获取dagnode的CID
    c := dagnode.Cid()
    // 获取dagnode的大小
    s, err := dagnode.Size()
    if err != nil {
        return nil, err
    }

    // 创建coreiface.AddEvent类型的输出信息
    output := &coreiface.AddEvent{
        Path: path.FromCid(c),
        Size: strconv.FormatUint(s, 10),
    }

    return output, nil
}
# 定义一个结构体 progressReader，包含文件、路径、输出通道、字节数和上次进度
type progressReader struct {
    file         io.Reader  # 文件读取器
    path         string     # 文件路径
    out          chan<- interface{}  # 输出通道
    bytes        int64      # 字节数
    lastProgress int64      # 上次进度
}

# 实现 progressReader 结构体的 Read 方法
func (i *progressReader) Read(p []byte) (int, error) {
    # 读取文件内容到 p 中，返回读取的字节数和可能的错误
    n, err := i.file.Read(p)

    # 更新已读取的字节数
    i.bytes += int64(n)
    # 如果已读取的字节数超过上次进度增量，或者出现了文件结束的错误
    if i.bytes-i.lastProgress >= progressReaderIncrement || err == io.EOF {
        # 更新上次进度
        i.lastProgress = i.bytes
        # 向输出通道发送进度事件
        i.out <- &coreiface.AddEvent{
            Name:  i.path,  # 文件路径
            Bytes: i.bytes,  # 已读取的字节数
        }
    }

    # 返回读取的字节数和可能的错误
    return n, err
}

# 定义一个结构体 progressReader2，包含 progressReader 结构体和文件信息
type progressReader2 struct {
    *progressReader  # 继承 progressReader 结构体
    files.FileInfo   # 文件信息
}

# 实现 progressReader2 结构体的 Read 方法
func (i *progressReader2) Read(p []byte) (int, error) {
    # 调用 progressReader 的 Read 方法
    return i.progressReader.Read(p)
}
```