# `kubo\fuse\ipns\ipns_unix.go`

```go
// 根据构建标记，确定是否构建此文件系统
// 构建标记表示不是 nofuse、openbsd、netbsd、plan9
// package fuse/ipns 实现了一个与 ipfs 的命名系统 ipns 接口的 fuse 文件系统

package ipns

import (
    "context"  // 上下文包，用于控制goroutine的执行
    "errors"   // 错误处理包
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包
    "os"       // 操作系统功能包
    "strings"  // 字符串处理包
    "syscall"  // 系统调用包

    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入 ipld/merkledag 包并重命名为 dag
    ft "github.com/ipfs/boxo/ipld/unixfs"      // 导入 ipld/unixfs 包并重命名为 ft
    "github.com/ipfs/boxo/path"                // 导入 boxo/path 包

    fuse "bazil.org/fuse"                       // 导入 bazil.org/fuse 包并重命名为 fuse
    fs "bazil.org/fuse/fs"                      // 导入 bazil.org/fuse/fs 包并重命名为 fs
    mfs "github.com/ipfs/boxo/mfs"              // 导入 boxo/mfs 包并重命名为 mfs
    cid "github.com/ipfs/go-cid"                // 导入 go-cid 包并重命名为 cid
    logging "github.com/ipfs/go-log"            // 导入 go-log 包并重命名为 logging
    iface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包并重命名为 iface
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 coreiface/options 包并重命名为 options
)

func init() {
    // 如果环境变量 IPFS_FUSE_DEBUG 存在，则设置 fuse.Debug 函数为 fmt.Println
    if os.Getenv("IPFS_FUSE_DEBUG") != "" {
        fuse.Debug = func(msg interface{}) {
            fmt.Println(msg)
        }
    }
}

var log = logging.Logger("fuse/ipns")  // 创建一个名为 log 的日志记录器，用于记录 fuse/ipns 相关的日志

// FileSystem 是可读写的 IPNS Fuse 文件系统
type FileSystem struct {
    Ipfs     iface.CoreAPI  // IPFS 接口
    RootNode *Root          // 根节点
}

// NewFileSystem 使用给定的 core.IpfsNode 实例构建新的文件系统
func NewFileSystem(ctx context.Context, ipfs iface.CoreAPI, ipfspath, ipnspath string) (*FileSystem, error) {
    key, err := ipfs.Key().Self(ctx)  // 获取 IPFS 节点的密钥
    if err != nil {
        return nil, err
    }
    root, err := CreateRoot(ctx, ipfs, map[string]iface.Key{"local": key}, ipfspath, ipnspath)  // 创建根节点
    if err != nil {
        return nil, err
    }

    return &FileSystem{Ipfs: ipfs, RootNode: root}, nil  // 返回文件系统实例
}

// Root 构造文件系统的根节点，一个根对象
func (f *FileSystem) Root() (fs.Node, error) {
    log.Debug("filesystem, get root")  // 记录调试信息
    return f.RootNode, nil  // 返回根节点
}

func (f *FileSystem) Destroy() {
    err := f.RootNode.Close()  // 关闭根节点
    if err != nil {
        log.Errorf("Error Shutting Down Filesystem: %s\n", err)  // 记录错误信息
    }
}

// Root 是文件系统树的根对象
type Root struct {
    Ipfs iface.CoreAPI  // IPFS 接口
    Keys map[string]iface.Key  // 密钥映射

    // 用于在 ipfs 中创建符号链接
    # IPFS 根目录的路径
    IpfsRoot  string
    # IPNS 根目录的路径
    IpnsRoot  string
    # 本地目录的映射，将目录名映射到文件系统节点
    LocalDirs map[string]fs.Node
    # 根据路径名映射到 MFS 根节点的映射
    Roots     map[string]*mfs.Root
    # 根据路径名映射到链接对象的映射
    LocalLinks map[string]*Link
// 返回一个发布 IPNS 记录的函数
func ipnsPubFunc(ipfs iface.CoreAPI, key iface.Key) mfs.PubFunc {
    return func(ctx context.Context, c cid.Cid) error {
        // 使用 IPFS 核心 API 发布指定 CID 的内容到 IPNS
        _, err := ipfs.Name().Publish(ctx, path.FromCid(c), options.Name.Key(key.Name()))
        return err
    }
}

// 加载根节点
func loadRoot(ctx context.Context, ipfs iface.CoreAPI, key iface.Key) (*mfs.Root, fs.Node, error) {
    // 解析节点
    node, err := ipfs.ResolveNode(ctx, key.Path())
    switch err {
    case nil:
        // 如果没有错误，继续执行
    case iface.ErrResolveFailed:
        // 如果解析失败，创建一个空目录节点
        node = ft.EmptyDirNode()
    default:
        // 如果出现其他错误，记录错误信息并返回
        log.Errorf("looking up %s: %s", key.Path(), err)
        return nil, nil, err
    }

    // 将节点转换为 ProtoNode 类型
    pbnode, ok := node.(*dag.ProtoNode)
    if !ok {
        return nil, nil, dag.ErrNotProtobuf
    }

    // 使用 IPFS DAG API 创建新的根节点
    root, err := mfs.NewRoot(ctx, ipfs.Dag(), pbnode, ipnsPubFunc(ipfs, key))
    if err != nil {
        return nil, nil, err
    }

    // 返回根节点和目录节点
    return root, &Directory{dir: root.GetDirectory()}, nil
}

// 创建根节点
func CreateRoot(ctx context.Context, ipfs iface.CoreAPI, keys map[string]iface.Key, ipfspath, ipnspath string) (*Root, error) {
    ldirs := make(map[string]fs.Node)
    roots := make(map[string]*mfs.Root)
    links := make(map[string]*Link)
    for alias, k := range keys {
        // 加载根节点
        root, fsn, err := loadRoot(ctx, ipfs, k)
        if err != nil {
            return nil, err
        }

        name := k.ID().String()

        roots[name] = root
        ldirs[name] = fsn

        // 设置别名符号链接
        links[alias] = &Link{
            Target: name,
        }
    }

    // 返回根节点
    return &Root{
        Ipfs:       ipfs,
        IpfsRoot:   ipfspath,
        IpnsRoot:   ipnspath,
        Keys:       keys,
        LocalDirs:  ldirs,
        LocalLinks: links,
        Roots:      roots,
    }, nil
}

// 返回文件属性
func (r *Root) Attr(ctx context.Context, a *fuse.Attr) error {
    log.Debug("Root Attr")
    a.Mode = os.ModeDir | 0o111 // -rw+x
    return nil
}

// 执行节点查找
func (r *Root) Lookup(ctx context.Context, name string) (fs.Node, error) {
    switch name {
    case "mach_kernel", ".hidden", "._.":
        // 在 OS X 上静默处理一些日志噪音
        return nil, syscall.Errno(syscall.ENOENT)
    }

    if lnk, ok := r.LocalLinks[name]; ok {
        return lnk, nil
    }

    nd, ok := r.LocalDirs[name]
    if ok {
        switch nd := nd.(type) {
        case *Directory:
            return nd, nil
        case *FileNode:
            return nd, nil
        default:
            return nil, syscall.Errno(syscall.EIO)
        }
    }

    // 其他链接通过 IPNS 解析，并链接到 IPFS 挂载点
    ipnsName := "/ipns/" + name
    resolved, err := r.Ipfs.Name().Resolve(ctx, ipnsName)
    if err != nil {
        log.Warnf("ipns: namesys resolve error: %s", err)
        return nil, syscall.Errno(syscall.ENOENT)
    }

    if resolved.Namespace() != path.IPFSNamespace {
        return nil, errors.New("invalid path from ipns record")
    }

    return &Link{r.IpfsRoot + "/" + strings.TrimPrefix(resolved.String(), "/ipfs/")}, nil
}

func (r *Root) Close() error {
    for _, mr := range r.Roots {
        err := mr.Close()
        if err != nil {
            return err
        }
    }
    return nil
}

// Forget 在文件系统卸载时调用，可能会被调用。
// 参见这里的注释：http://godoc.org/bazil.org/fuse/fs#FSDestroyer
func (r *Root) Forget() {
    err := r.Close()
    if err != nil {
        log.Error(err)
    }
}

// ReadDirAll 读取特定目录。将显示本地可用的密钥以及对 peerID 密钥的符号链接。
func (r *Root) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
    log.Debug("Root ReadDirAll")

    listing := make([]fuse.Dirent, 0, len(r.Keys)*2)
}
    # 遍历 r.Keys 中的每个元素，使用 alias 作为键，k 作为值
    for alias, k := range r.Keys {
        # 创建一个 fuse.Dirent 对象，设置其名称为 k 的 ID 字符串表示，类型为目录
        ent := fuse.Dirent{
            Name: k.ID().String(),
            Type: fuse.DT_Dir,
        }
        # 创建一个 fuse.Dirent 对象，设置其名称为 alias，类型为链接
        link := fuse.Dirent{
            Name: alias,
            Type: fuse.DT_Link,
        }
        # 将 ent 和 link 添加到 listing 列表中
        listing = append(listing, ent, link)
    }
    # 返回 listing 列表和空值
    return listing, nil
// Directory 结构体是对 mfs 目录的封装，以满足 fuse 文件系统接口的要求
type Directory struct {
    dir *mfs.Directory
}

// FileNode 结构体是对 mfs 文件的封装，以满足 fuse 文件系统接口的要求
type FileNode struct {
    fi *mfs.File
}

// File 结构体是对 mfs 文件的封装，以满足 fuse 文件系统接口的要求
type File struct {
    fi mfs.FileDescriptor
}

// Attr 返回给定节点的属性
func (d *Directory) Attr(ctx context.Context, a *fuse.Attr) error {
    log.Debug("Directory Attr")
    a.Mode = os.ModeDir | 0o555
    a.Uid = uint32(os.Getuid())
    a.Gid = uint32(os.Getgid())
    return nil
}

// Attr 返回给定节点的属性
func (fi *FileNode) Attr(ctx context.Context, a *fuse.Attr) error {
    log.Debug("File Attr")
    size, err := fi.fi.Size()
    if err != nil {
        // 在这种情况下，可能无法获取文件大小
        return fmt.Errorf("fuse/ipns: failed to get file.Size(): %s", err)
    }
    a.Mode = os.FileMode(0o666)
    a.Size = uint64(size)
    a.Uid = uint32(os.Getuid())
    a.Gid = uint32(os.Getgid())
    return nil
}

// Lookup 在该节点下执行查找
func (d *Directory) Lookup(ctx context.Context, name string) (fs.Node, error) {
    child, err := d.dir.Child(name)
    if err != nil {
        // todo: 使此错误更通用
        return nil, syscall.Errno(syscall.ENOENT)
    }

    switch child := child.(type) {
    case *mfs.Directory:
        return &Directory{dir: child}, nil
    case *mfs.File:
        return &FileNode{fi: child}, nil
    default:
        // 注意：如果发生这种情况，我们不希望继续，可能会出现不可预测的行为
        panic("在目录下发现无效类型。程序员错误。")
    }
}

// ReadDirAll 读取链接结构作为目录条目
func (d *Directory) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
    listing, err := d.dir.List(ctx)
    if err != nil {
        return nil, err
    }
    entries := make([]fuse.Dirent, len(listing))
    # 遍历列表中的每个条目，同时获取索引和条目内容
    for i, entry := range listing {
        # 创建一个文件系统目录实体对象，并设置其名称为当前条目的名称
        dirent := fuse.Dirent{Name: entry.Name}

        # 根据条目类型进行判断
        switch mfs.NodeType(entry.Type):
            # 如果是目录类型，则设置实体对象的类型为目录
            case mfs.TDir:
                dirent.Type = fuse.DT_Dir
            # 如果是文件类型，则设置实体对象的类型为文件
            case mfs.TFile:
                dirent.Type = fuse.DT_File

        # 将创建好的实体对象放入对应的索引位置
        entries[i] = dirent
    }

    # 如果实体对象列表的长度大于0，则返回实体对象列表和空值
    if len(entries) > 0:
        return entries, nil
    # 如果实体对象列表的长度为0，则返回空值和系统调用错误信息
    return nil, syscall.Errno(syscall.ENOENT)
// 从文件中读取数据到内存中
func (fi *File) Read(ctx context.Context, req *fuse.ReadRequest, resp *fuse.ReadResponse) error {
    // 将文件指针移动到指定的偏移量
    _, err := fi.fi.Seek(req.Offset, io.SeekStart)
    if err != nil {
        return err
    }

    // 获取文件的大小
    fisize, err := fi.fi.Size()
    if err != nil {
        return err
    }

    // 检查上下文是否已经取消
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }

    // 计算实际读取的数据大小
    readsize := min(req.Size, int(fisize-req.Offset))
    // 从文件中读取数据到响应中
    n, err := fi.fi.CtxReadFull(ctx, resp.Data[:readsize])
    resp.Data = resp.Data[:n]
    return err
}

// 将数据写入文件
func (fi *File) Write(ctx context.Context, req *fuse.WriteRequest, resp *fuse.WriteResponse) error {
    // TODO: at some point, ensure that WriteAt here respects the context
    // 在指定的偏移量处写入数据
    wrote, err := fi.fi.WriteAt(req.Data, req.Offset)
    if err != nil {
        return err
    }
    resp.Size = wrote
    return nil
}

// 刷新文件
func (fi *File) Flush(ctx context.Context, req *fuse.FlushRequest) error {
    // 创建一个用于接收错误的通道
    errs := make(chan error, 1)
    // 异步执行文件的刷新操作
    go func() {
        errs <- fi.fi.Flush()
    }()
    // 使用 select 语句处理多个通道操作
    select {
    case err := <-errs:
        return err
    case <-ctx.Done():
        return ctx.Err()
    }
}

// 设置文件属性
func (fi *File) Setattr(ctx context.Context, req *fuse.SetattrRequest, resp *fuse.SetattrResponse) error {
    // 如果请求中包含文件大小的有效值
    if req.Valid.Size() {
        // 获取当前文件的大小
        cursize, err := fi.fi.Size()
        if err != nil {
            return err
        }
        // 如果当前文件大小与请求中的大小不一致，则调整文件大小
        if cursize != int64(req.Size) {
            err := fi.fi.Truncate(int64(req.Size))
            if err != nil {
                return err
            }
        }
    }
    return nil
}

// 将文件内容刷新到磁盘
func (fi *FileNode) Fsync(ctx context.Context, req *fuse.FsyncRequest) error {
    // 需要执行完整的刷新操作，因为在 MFS 中，只有在根目录更新后，写入才会持久化
    // 创建一个用于接收错误的通道
    errs := make(chan error, 1)
    // 异步执行文件的刷新操作
    go func() {
        errs <- fi.fi.Flush()
    }()
    // 使用 select 语句处理多个通道操作
    select {
    case err := <-errs:
        return err
    case <-ctx.Done():
        return ctx.Err()
    }
}
func (fi *File) Forget() {
    // TODO(steb): this seems like a place where we should be *uncaching*, not flushing.
    // 调用文件的 Flush 方法，将文件内容刷新到磁盘
    err := fi.fi.Flush()
    if err != nil {
        // 如果出现错误，记录错误信息
        log.Debug("forget file error: ", err)
    }
}

func (d *Directory) Mkdir(ctx context.Context, req *fuse.MkdirRequest) (fs.Node, error) {
    // 在当前目录下创建一个子目录
    child, err := d.dir.Mkdir(req.Name)
    if err != nil {
        return nil, err
    }

    // 返回新创建的子目录
    return &Directory{dir: child}, nil
}

func (fi *FileNode) Open(ctx context.Context, req *fuse.OpenRequest, resp *fuse.OpenResponse) (fs.Handle, error) {
    // 打开文件并返回文件句柄
    fd, err := fi.fi.Open(mfs.Flags{
        Read:  req.Flags.IsReadOnly() || req.Flags.IsReadWrite(),
        Write: req.Flags.IsWriteOnly() || req.Flags.IsReadWrite(),
        Sync:  true,
    })
    if err != nil {
        return nil, err
    }

    // 根据请求的标志位进行相应的操作
    if req.Flags&fuse.OpenTruncate != 0 {
        if req.Flags.IsReadOnly() {
            log.Error("tried to open a readonly file with truncate")
            return nil, syscall.Errno(syscall.ENOTSUP)
        }
        log.Info("Need to truncate file!")
        // 截断文件
        err := fd.Truncate(0)
        if err != nil {
            return nil, err
        }
    } else if req.Flags&fuse.OpenAppend != 0 {
        log.Info("Need to append to file!")
        if req.Flags.IsReadOnly() {
            log.Error("tried to open a readonly file with append")
            return nil, syscall.Errno(syscall.ENOTSUP)
        }

        // 将文件指针移动到文件末尾
        _, err := fd.Seek(0, io.SeekEnd)
        if err != nil {
            log.Error("seek reset failed: ", err)
            return nil, err
        }
    }

    // 返回文件句柄
    return &File{fi: fd}, nil
}

func (fi *File) Release(ctx context.Context, req *fuse.ReleaseRequest) error {
    // 关闭文件
    return fi.fi.Close()
}

func (d *Directory) Create(ctx context.Context, req *fuse.CreateRequest, resp *fuse.CreateResponse) (fs.Node, fs.Handle, error) {
    // 创建一个新的空文件节点
    nd := dag.NodeWithData(ft.FilePBData(nil, 0))
    // 将新文件节点添加到当前目录下
    err := d.dir.AddChild(req.Name, nd)
    # 如果发生错误，返回空值和错误信息
    if err != nil:
        return nil, nil, err

    # 根据请求的名称获取目录下的子节点
    child, err := d.dir.Child(req.Name)
    if err != nil:
        return nil, nil, err

    # 将子节点转换为文件对象
    fi, ok := child.(*mfs.File)
    if !ok:
        return nil, nil, errors.New("child creation failed")

    # 创建文件节点对象
    nodechild := &FileNode{fi: fi}

    # 打开文件并返回文件描述符
    fd, err := fi.Open(mfs.Flags{
        Read:  req.Flags.IsReadOnly() || req.Flags.IsReadWrite(),
        Write: req.Flags.IsWriteOnly() || req.Flags.IsReadWrite(),
        Sync:  true,
    })
    if err != nil:
        return nil, nil, err

    # 返回文件节点和文件描述符
    return nodechild, &File{fi: fd}, nil
}

// Remove removes the specified file from the directory
func (d *Directory) Remove(ctx context.Context, req *fuse.RemoveRequest) error {
    // Unlink the file from the directory
    err := d.dir.Unlink(req.Name)
    if err != nil {
        // If there is an error, return ENOENT error
        return syscall.Errno(syscall.ENOENT)
    }
    // If successful, return nil
    return nil
}

// Rename renames a file or directory within the directory
func (d *Directory) Rename(ctx context.Context, req *fuse.RenameRequest, newDir fs.Node) error {
    // Get the current file or directory to be renamed
    cur, err := d.dir.Child(req.OldName)
    if err != nil {
        return err
    }

    // Unlink the old file or directory
    err = d.dir.Unlink(req.OldName)
    if err != nil {
        return err
    }

    // Check the type of the new directory
    switch newDir := newDir.(type) {
    case *Directory:
        // If the new directory is a Directory, move the node to the new directory
        nd, err := cur.GetNode()
        if err != nil {
            return err
        }

        err = newDir.dir.AddChild(req.NewName, nd)
        if err != nil {
            return err
        }
    case *FileNode:
        // If the new directory is a FileNode, return EPERM error
        log.Error("Cannot move node into a file!")
        return syscall.Errno(syscall.EPERM)
    default:
        // If the new directory is of unknown type, return an error
        log.Error("Unknown node type for rename target dir!")
        return errors.New("unknown fs node type")
    }
    // If successful, return nil
    return nil
}

// min returns the minimum of two integers
func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}

// ipnsRoot is an interface to check if Node implements all the required interfaces
type ipnsRoot interface {
    fs.Node
    fs.HandleReadDirAller
    fs.NodeStringLookuper
}

// Check that Root implements ipnsRoot interface
var _ ipnsRoot = (*Root)(nil)

// ipnsDirectory is an interface to check if Directory implements all the required interfaces
type ipnsDirectory interface {
    fs.HandleReadDirAller
    fs.Node
    fs.NodeCreater
    fs.NodeMkdirer
    fs.NodeRemover
    fs.NodeRenamer
    fs.NodeStringLookuper
}

// Check that Directory implements ipnsDirectory interface
var _ ipnsDirectory = (*Directory)(nil)

// ipnsFile is an interface to check if File implements all the required interfaces
type ipnsFile interface {
    fs.HandleFlusher
    fs.HandleReader
    fs.HandleWriter
    fs.HandleReleaser
}

// ipnsFileNode is an interface to check if FileNode implements all the required interfaces
type ipnsFileNode interface {
    fs.Node
    fs.NodeFsyncer
    fs.NodeOpener
}

// Check that FileNode and File implement the required interfaces
var (
    _ ipnsFileNode = (*FileNode)(nil)
    _ ipnsFile     = (*File)(nil)
)
```