# `kubo\fuse\readonly\readonly_unix.go`

```
// 根据条件构建只在 linux、darwin、freebsd 平台上使用 FUSE 的包
// +build linux darwin freebsd
// +build !nofuse

package readonly

import (
    "context" // 导入 context 包
    "fmt" // 导入 fmt 包
    "io" // 导入 io 包
    "os" // 导入 os 包
    "syscall" // 导入 syscall 包

    fuse "bazil.org/fuse" // 导入 bazil.org/fuse 包，用于 FUSE 操作
    fs "bazil.org/fuse/fs" // 导入 bazil.org/fuse/fs 包，用于 FUSE 文件系统操作
    mdag "github.com/ipfs/boxo/ipld/merkledag" // 导入 merkledag 包
    ft "github.com/ipfs/boxo/ipld/unixfs" // 导入 unixfs 包
    uio "github.com/ipfs/boxo/ipld/unixfs/io" // 导入 unixfs/io 包
    "github.com/ipfs/boxo/path" // 导入 path 包
    "github.com/ipfs/go-cid" // 导入 go-cid 包
    ipld "github.com/ipfs/go-ipld-format" // 导入 go-ipld-format 包
    logging "github.com/ipfs/go-log" // 导入 go-log 包
    core "github.com/ipfs/kubo/core" // 导入 kubo/core 包
    ipldprime "github.com/ipld/go-ipld-prime" // 导入 go-ipld-prime 包
    cidlink "github.com/ipld/go-ipld-prime/linking/cid" // 导入 go-ipld-prime/linking/cid 包
)

var log = logging.Logger("fuse/ipfs") // 定义日志记录器

// FileSystem 是只读 IPFS Fuse 文件系统
type FileSystem struct {
    Ipfs *core.IpfsNode // IPFS 节点
}

// NewFileSystem 使用给定的 core.IpfsNode 实例构建新的文件系统
func NewFileSystem(ipfs *core.IpfsNode) *FileSystem {
    return &FileSystem{Ipfs: ipfs} // 返回新的文件系统实例
}

// Root 构建文件系统的根，即根对象
func (f FileSystem) Root() (fs.Node, error) {
    return &Root{Ipfs: f.Ipfs}, nil // 返回根对象
}

// Root 是文件系统树的根对象
type Root struct {
    Ipfs *core.IpfsNode // IPFS 节点
}

// Attr 返回文件属性
func (*Root) Attr(ctx context.Context, a *fuse.Attr) error {
    a.Mode = os.ModeDir | 0o111 // 设置文件夹权限为 -rw+x
    return nil
}

// Lookup 在此节点下执行查找
func (s *Root) Lookup(ctx context.Context, name string) (fs.Node, error) {
    log.Debugf("Root Lookup: '%s'", name) // 记录查找日志
    switch name {
    case "mach_kernel", ".hidden", "._.":
        // 在 OS X 上静默处理一些日志噪音
        return nil, syscall.Errno(syscall.ENOENT)
    }

    p, err := path.NewPath(name) // 解析路径
    if err != nil {
        log.Debugf("fuse failed to parse path: %q: %s", name, err) // 记录解析路径失败的日志
        return nil, syscall.Errno(syscall.ENOENT)
    }

    imPath, err := path.NewImmutablePath(p) // 解析不可变路径
    // 如果发生错误，则记录错误信息并返回文件不存在的系统调用错误
    if err != nil {
        log.Debugf("fuse failed to convert path: %q: %s", name, err)
        return nil, syscall.Errno(syscall.ENOENT)
    }

    // 使用 UnixFSPathResolver 解析路径
    nd, ndLnk, err := s.Ipfs.UnixFSPathResolver.ResolvePath(ctx, imPath)
    if err != nil {
        // todo: make this error more versatile.
        return nil, syscall.Errno(syscall.ENOENT)
    }

    // 检查解析后的链接是否为 cidlink.Link 类型
    cidLnk, ok := ndLnk.(cidlink.Link)
    if !ok {
        log.Debugf("non-cidlink returned from ResolvePath: %v", ndLnk)
        return nil, syscall.Errno(syscall.ENOENT)
    }

    // 根据链接的 CID 获取块数据
    blk, err := s.Ipfs.Blockstore.Get(ctx, cidLnk.Cid)
    if err != nil {
        log.Debugf("fuse failed to retrieve block: %v: %s", cidLnk, err)
        return nil, syscall.Errno(syscall.ENOENT)
    }

    // 根据 CID 的编解码格式进行不同的处理
    var fnd ipld.Node
    switch cidLnk.Cid.Prefix().Codec {
    case cid.DagProtobuf:
        adl, ok := nd.(ipldprime.ADL)
        if ok {
            substrate := adl.Substrate()
            fnd, err = mdag.ProtoNodeConverter(blk, substrate)
        } else {
            fnd, err = mdag.ProtoNodeConverter(blk, nd)
        }
    case cid.Raw:
        fnd, err = mdag.RawNodeConverter(blk, nd)
    default:
        log.Error("fuse node was not a supported type")
        return nil, syscall.Errno(syscall.ENOTSUP)
    }
    if err != nil {
        log.Errorf("could not convert protobuf or raw node: %s", err)
        return nil, syscall.Errno(syscall.ENOENT)
    }

    // 返回包含 IPFS 和节点的结构体指针
    return &Node{Ipfs: s.Ipfs, Nd: fnd}, nil
// ReadDirAll reads a particular directory. Disallowed for root.
// 读取特定目录的所有内容。对于根目录是不允许的。
func (*Root) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
    // 记录调试信息
    log.Debug("read Root")
    // 返回空值和系统调用错误
    return nil, syscall.Errno(syscall.EPERM)
}

// Node is the core object representing a filesystem tree node.
// Node 是表示文件系统树节点的核心对象。
type Node struct {
    Ipfs   *core.IpfsNode
    Nd     ipld.Node
    cached *ft.FSNode
}

func (s *Node) loadData() error {
    if pbnd, ok := s.Nd.(*mdag.ProtoNode); ok {
        fsn, err := ft.FSNodeFromBytes(pbnd.Data())
        if err != nil {
            return err
        }
        s.cached = fsn
    }
    return nil
}

// Attr returns the attributes of a given node.
// Attr 返回给定节点的属性。
func (s *Node) Attr(ctx context.Context, a *fuse.Attr) error {
    // 记录调试信息
    log.Debug("Node attr")
    if rawnd, ok := s.Nd.(*mdag.RawNode); ok {
        a.Mode = 0o444
        a.Size = uint64(len(rawnd.RawData()))
        a.Blocks = 1
        return nil
    }

    if s.cached == nil {
        if err := s.loadData(); err != nil {
            return fmt.Errorf("readonly: loadData() failed: %s", err)
        }
    }
    switch s.cached.Type() {
    case ft.TDirectory, ft.THAMTShard:
        a.Mode = os.ModeDir | 0o555
    case ft.TFile:
        size := s.cached.FileSize()
        a.Mode = 0o444
        a.Size = uint64(size)
        a.Blocks = uint64(len(s.Nd.Links()))
    case ft.TRaw:
        a.Mode = 0o444
        a.Size = uint64(len(s.cached.Data()))
        a.Blocks = uint64(len(s.Nd.Links()))
    case ft.TSymlink:
        a.Mode = 0o777 | os.ModeSymlink
        a.Size = uint64(len(s.cached.Data()))
    default:
        return fmt.Errorf("invalid data type - %s", s.cached.Type())
    }
    return nil
}

// Lookup performs a lookup under this node.
// Lookup 在此节点下执行查找。
func (s *Node) Lookup(ctx context.Context, name string) (fs.Node, error) {
    // 记录调试信息
    log.Debugf("Lookup '%s'", name)
    link, _, err := uio.ResolveUnixfsOnce(ctx, s.Ipfs.DAG, s.Nd, []string{name})
    switch err {
    // 如果错误是文件不存在或链接未找到，则返回 ENOENT 错误
    case os.ErrNotExist, mdag.ErrLinkNotFound:
        // todo: make this error more versatile.
        return nil, syscall.Errno(syscall.ENOENT)
    // 如果错误为空，不执行任何操作
    case nil:
        // noop
    // 如果错误不为空，记录错误信息并返回 EIO 错误
    default:
        log.Errorf("fuse lookup %q: %s", name, err)
        return nil, syscall.Errno(syscall.EIO)
    }

    // 根据链接的 CID 从 IPFS 的 DAG 中获取节点
    nd, err := s.Ipfs.DAG.Get(ctx, link.Cid)
    // 如果出现错误并且不是未找到错误，则记录错误信息并返回错误
    if err != nil && !ipld.IsNotFound(err) {
        log.Errorf("fuse lookup %q: %s", name, err)
        return nil, err
    }

    // 返回包含 IPFS 和节点的 Node 结构体
    return &Node{Ipfs: s.Ipfs, Nd: nd}, nil
// ReadDirAll函数读取链接结构作为目录条目
func (s *Node) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
    // 记录调试信息
    log.Debug("Node ReadDir")
    // 从Node的Ipfs.DAG和Nd创建一个新的目录
    dir, err := uio.NewDirectoryFromNode(s.Ipfs.DAG, s.Nd)
    if err != nil {
        return nil, err
    }

    var entries []fuse.Dirent
    // 遍历目录中的每个链接
    err = dir.ForEachLink(ctx, func(lnk *ipld.Link) error {
        n := lnk.Name
        // 如果链接名为空，则使用链接的Cid字符串作为名称
        if len(n) == 0 {
            n = lnk.Cid.String()
        }
        // 获取链接的子节点
        nd, err := s.Ipfs.DAG.Get(ctx, lnk.Cid)
        if err != nil {
            log.Warn("error fetching directory child node: ", err)
        }

        t := fuse.DT_Unknown
        // 根据子节点的类型设置目录条目的类型
        switch nd := nd.(type) {
        case *mdag.RawNode:
            t = fuse.DT_File
        case *mdag.ProtoNode:
            if fsn, err := ft.FSNodeFromBytes(nd.Data()); err != nil {
                log.Warn("failed to unmarshal protonode data field:", err)
            } else {
                switch fsn.Type() {
                case ft.TDirectory, ft.THAMTShard:
                    t = fuse.DT_Dir
                case ft.TFile, ft.TRaw:
                    t = fuse.DT_File
                case ft.TSymlink:
                    t = fuse.DT_Link
                case ft.TMetadata:
                    log.Error("metadata object in fuse should contain its wrapped type")
                default:
                    log.Error("unrecognized protonode data type: ", fsn.Type())
                }
            }
        }
        // 将目录条目添加到entries中
        entries = append(entries, fuse.Dirent{Name: n, Type: t})
        return nil
    })
    if err != nil {
        return nil, err
    }

    // 如果entries不为空，则返回entries，否则返回syscall.ENOENT错误
    if len(entries) > 0 {
        return entries, nil
    }
    return nil, syscall.Errno(syscall.ENOENT)
}

// Getxattr函数获取扩展属性
func (s *Node) Getxattr(ctx context.Context, req *fuse.GetxattrRequest, resp *fuse.GetxattrResponse) error {
    // TODO: is nil the right response for 'bug off, we ain't got none' ?
    // 设置resp.Xattr为nil
    resp.Xattr = nil
    return nil
}
// 读取符号链接的目标路径
func (s *Node) Readlink(ctx context.Context, req *fuse.ReadlinkRequest) (string, error) {
    // 如果缓存为空或者缓存类型不是符号链接，则返回无效参数错误
    if s.cached == nil || s.cached.Type() != ft.TSymlink {
        return "", fuse.Errno(syscall.EINVAL)
    }
    // 返回缓存数据转换为字符串的结果
    return string(s.cached.Data()), nil
}

// 读取文件内容
func (s *Node) Read(ctx context.Context, req *fuse.ReadRequest, resp *fuse.ReadResponse) error {
    // 创建用于读取数据的 DAG 读取器
    r, err := uio.NewDagReader(ctx, s.Nd, s.Ipfs.DAG)
    if err != nil {
        return err
    }
    // 移动读取器到指定偏移量处
    _, err = r.Seek(req.Offset, io.SeekStart)
    if err != nil {
        return err
    }
    // 创建缓冲区，大小为请求的数据大小
    buf := resp.Data[:int(req.Size)]
    // 从读取器中读取数据到缓冲区中
    n, err := io.ReadFull(r, buf)
    // 将响应的数据设置为实际读取的数据
    resp.Data = buf[:n]
    // 根据不同的错误类型进行处理
    switch err {
    case nil, io.EOF, io.ErrUnexpectedEOF:
    default:
        return err
    }
    // 更新响应的数据为实际读取的数据
    resp.Data = resp.Data[:n]
    return nil // 可能是非空/未成功的
}

// 检查 Node 是否实现了我们想要的所有接口
type roRoot interface {
    fs.Node
    fs.HandleReadDirAller
    fs.NodeStringLookuper
}

// 确保 Root 实现了 roRoot 接口
var _ roRoot = (*Root)(nil)

// 检查 Node 是否实现了我们想要的所有接口
type roNode interface {
    fs.HandleReadDirAller
    fs.HandleReader
    fs.Node
    fs.NodeStringLookuper
    fs.NodeReadlinker
    fs.NodeGetxattrer
}

// 确保 Node 实现了 roNode 接口
var _ roNode = (*Node)(nil)
```