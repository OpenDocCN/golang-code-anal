# go-ipfs 源码解析 43

# `/opt/kubo/fuse/ipns/ipns_unix.go`

这段代码是一个 Go 语言编写的 FUSE 文件系统实现，用于支持类似于 IPv6 名称空间（IPNS）的命名系统。它实现了 FUSE 规范中的基本功能，但没有包含一些可能不需要的代码，如网络文件系统（NetFS）和 Plan 9 文件系统。

首先，该代码使用 `build` 工具（go build）来构建代码，这将生成一个名为 `ipns_fuse.go` 的可执行文件。接下来，它会定义一个名为 `ipns` 的包。

然后，它会导入一些用于 FUSE 和 Go 标准库的库，如 `dag`、`ft`、`iface`、`options` 和 `mfs`。这些库提供了一系列用于实现 FUSE 规范的功能。

接下来，该代码定义了一个名为 `ipns_fuse` 的函数。这个函数使用了来自 `boxo` 库的一些工具链，如 `merkledag` 和 `unixfs`。这些库提供了对 FUSE 系统中树的树状结构表示的支持。

接着，该函数实现了以下功能：

1. 初始化 FUSE 系统。
2. 创建一个空的 FUSE 系统岛。
3. 将 FUSE 系统树推送到树状结构中。
4. 创建一个名为 `ipns_name` 的文件系统。
5. 使用 FUSE 系统树中的文件系统文件创建一个名为 `ipns_fuse` 的文件系统。
6. 通过调用 `fuse.NewFileSystemContext` 函数，返回一个名为 `ctx` 的文件系统上下文，从而实现文件系统的功能。

最后，该代码导出了来自 `ipns` 包的 `ipns_fuse` 函数。


```
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9

// package fuse/ipns implements a fuse filesystem that interfaces
// with ipns, the naming system for ipfs.
package ipns

import (
	"context"
	"errors"
	"fmt"
	"io"
	"os"
	"strings"
	"syscall"

	dag "github.com/ipfs/boxo/ipld/merkledag"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	"github.com/ipfs/boxo/path"

	fuse "bazil.org/fuse"
	fs "bazil.org/fuse/fs"
	iface "github.com/ipfs/boxo/coreiface"
	options "github.com/ipfs/boxo/coreiface/options"
	mfs "github.com/ipfs/boxo/mfs"
	cid "github.com/ipfs/go-cid"
	logging "github.com/ipfs/go-log"
)

```

这段代码定义了一个名为“init”的函数，作为FUSE文件系统初始化函数的一部分。该函数检查操作系统是否具有名为“IPFS_FUSE_DEBUG”的环境变量，如果是，则定义了一个名为“fuse.Debug”的函数，它接受一个表示IPFS FUSE调试信息的消息和一个额外的参数(“msg”指定了要打印的消息)，然后使用fmt.Println函数打印该消息。

此外，还定义了一个名为“log”的变量，将其设置为名为“fuse/ipns”的日志类中的一个logger实例。

最后，定义了一个名为“FileSystem”的类型，该类型表示IPFS文件系统。该类型有一个名为“Ipfs”的成员变量，它是一个iface.CoreAPI类型，表示IPFS的根节点。还有一个名为“RootNode”的成员变量，它是一个指向IPFS根节点的指针类型。


```
func init() {
	if os.Getenv("IPFS_FUSE_DEBUG") != "" {
		fuse.Debug = func(msg interface{}) {
			fmt.Println(msg)
		}
	}
}

var log = logging.Logger("fuse/ipns")

// FileSystem is the readwrite IPNS Fuse Filesystem.
type FileSystem struct {
	Ipfs     iface.CoreAPI
	RootNode *Root
}

```

这段代码定义了一个名为NewFileSystem的函数，它接受两个参数：一个上下文上下文和两个文件路径。函数的作用是通过调用IPFS网络中的ipfs.Key().Self函数来创建一个新的根目录对象，并使用给定的IPFS网络接口的节点实例来创建一个新的文件系统实例。

进一步地，该函数首先尝试从IPFS网络中获取一个名为“local”的密钥，并使用上下文上下文和密钥来创建一个名为“root”的根目录对象。然后，它返回一个新的FileSystem实例，其中IPFS表示IPFS网络接口，RootNode表示根目录对象。最后，如果函数在创建根目录对象时遇到任何错误，它将返回一个error。


```
// NewFileSystem constructs new fs using given core.IpfsNode instance.
func NewFileSystem(ctx context.Context, ipfs iface.CoreAPI, ipfspath, ipnspath string) (*FileSystem, error) {
	key, err := ipfs.Key().Self(ctx)
	if err != nil {
		return nil, err
	}
	root, err := CreateRoot(ctx, ipfs, map[string]iface.Key{"local": key}, ipfspath, ipnspath)
	if err != nil {
		return nil, err
	}

	return &FileSystem{Ipfs: ipfs, RootNode: root}, nil
}

// Root constructs the Root of the filesystem, a Root object.
```

这段代码定义了两个函数，一个是`Root()`，返回值为根节点`fs.Node`和`nil`，另一个是`Destroy()`，用于关闭文件系统并输出错误。

具体来说，`Root()`函数返回根节点，`f.RootNode`指的是`f`指向的`FileSystem`中的根节点。这个函数使用了`f.RootNode.Close()`来关闭根节点。如果关闭失败或者根节点为空，函数返回`nil`。

`Destroy()`函数用于关闭整个文件系统并输出错误。首先，使用`f.RootNode.Close()`关闭整个文件系统。如果关闭失败，函数输出错误并关闭`f.RootNode`。然后，所有关闭的文件系统节点都会被关闭，并输出错误。


```
func (f *FileSystem) Root() (fs.Node, error) {
	log.Debug("filesystem, get root")
	return f.RootNode, nil
}

func (f *FileSystem) Destroy() {
	err := f.RootNode.Close()
	if err != nil {
		log.Errorf("Error Shutting Down Filesystem: %s\n", err)
	}
}

// Root is the root object of the filesystem tree.
type Root struct {
	Ipfs iface.CoreAPI
	Keys map[string]iface.Key

	// Used for symlinking into ipfs
	IpfsRoot  string
	IpnsRoot  string
	LocalDirs map[string]fs.Node
	Roots     map[string]*mfs.Root

	LocalLinks map[string]*Link
}

```

这两段代码定义了两个函数：ipnsPubFunc 和 loadRoot。

ipnsPubFunc 函数接收两个参数：mfs.PubFunc 和 iface.Key。函数内部通过调用 ipfs.Name().Publish 函数并传入 iface.Key 和路径参数 c，将一个名为 "ipns" 的目录节点发布到 ipfs 命名空间中。

loadRoot 函数接收两个参数：mfs.Context 和 iface.Key。函数内部首先使用 ipfs.ResolveNode 函数查找 iface.Key 对应的路径节点，然后使用 ft.EmptyDirNode 函数创建一个新的空目录节点。如果过程中出现错误，函数会记录错误并返回。最后，函数调用 mfs.NewRoot 将创建的根目录节点设置为 root，并将根目录的目录设置为 iface.Key 对应的目录。


```
func ipnsPubFunc(ipfs iface.CoreAPI, key iface.Key) mfs.PubFunc {
	return func(ctx context.Context, c cid.Cid) error {
		_, err := ipfs.Name().Publish(ctx, path.FromCid(c), options.Name.Key(key.Name()))
		return err
	}
}

func loadRoot(ctx context.Context, ipfs iface.CoreAPI, key iface.Key) (*mfs.Root, fs.Node, error) {
	node, err := ipfs.ResolveNode(ctx, key.Path())
	switch err {
	case nil:
	case iface.ErrResolveFailed:
		node = ft.EmptyDirNode()
	default:
		log.Errorf("looking up %s: %s", key.Path(), err)
		return nil, nil, err
	}

	pbnode, ok := node.(*dag.ProtoNode)
	if !ok {
		return nil, nil, dag.ErrNotProtobuf
	}

	root, err := mfs.NewRoot(ctx, ipfs.Dag(), pbnode, ipnsPubFunc(ipfs, key))
	if err != nil {
		return nil, nil, err
	}

	return root, &Directory{dir: root.GetDirectory()}, nil
}

```

该函数创建了一个根目录结构，其中根目录的名称由传递给它的参数中的键来确定。它还设置了一个IPFS实例和一个键值对 map，用于存储目录中的文件系统节点。

函数首先加载每个键值对中的根目录，如果遇到错误，就返回。然后，它遍历存储键值对中的每个节点，设置一个节点对应的文件系统节点，并设置一个键值对，用于存储符号链接。最后，函数返回一个根目录结构对象，其中包含IPFS实例，根目录的文件系统路径和键值对中存储的键值对。


```
func CreateRoot(ctx context.Context, ipfs iface.CoreAPI, keys map[string]iface.Key, ipfspath, ipnspath string) (*Root, error) {
	ldirs := make(map[string]fs.Node)
	roots := make(map[string]*mfs.Root)
	links := make(map[string]*Link)
	for alias, k := range keys {
		root, fsn, err := loadRoot(ctx, ipfs, k)
		if err != nil {
			return nil, err
		}

		name := k.ID().String()

		roots[name] = root
		ldirs[name] = fsn

		// set up alias symlink
		links[alias] = &Link{
			Target: name,
		}
	}

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

```

该代码定义了一个名为Root的FUSE文件系统根目录的FUSE Attribute实现。Attr函数用于设置文件或目录的权限，并返回错误。Lookup函数用于查找指定目录下的文件或目录，并返回文件或目录的FS Node。

具体来说，代码可以拆分为以下几个部分：

1. logging：在函数实现中，通过log.Debug函数输出一些日志信息。这些日志信息包括当前目录、正在执行的FUSE Attribute操作以及从IPFS（InterPlanetary File System，星际文件系统）上尝试获取的文件等信息。

2. FUSE Attribute操作：定义了Attr函数和Lookup函数。Attr函数接受一个FUSE Attribute对象和一个名为a的局部变量，尝试设置文件或目录的权限，并返回错误。Lookup函数接受一个上下文和一个文件或目录的名称，尝试查找目录中是否存在该文件或目录，并返回FS Node或错误。

3. IPFS查找：定义了一个名为ipns的内部变量，用于存储IPFS上资源名称解析后的名称。在某些FUSE Attribute操作中，通过调用Ipfs.Name().Resolve函数尝试从IPFS上获取资源名称，如果解析失败，则返回一个已知错误。

4. 格式化输出：在函数实现中，定义了一个名为"ipfs: namesys resolve error："的错误日志格式。如果Ipfs上获取资源名称时出现错误，则使用该格式输出错误信息。


```
// Attr returns file attributes.
func (r *Root) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("Root Attr")
	a.Mode = os.ModeDir | 0o111 // -rw+x
	return nil
}

// Lookup performs a lookup under this node.
func (r *Root) Lookup(ctx context.Context, name string) (fs.Node, error) {
	switch name {
	case "mach_kernel", ".hidden", "._.":
		// Just quiet some log noise on OS X.
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

	// other links go through ipns resolution and are symlinked into the ipfs mountpoint
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

```

这是一个使用Go的函数，代表一个文件系统的根目录。函数接收一个名为r的指针，代表根目录。函数内部执行以下操作：

1. 通过循环遍历根目录的子目录：func (r *Root) Close() error {
	for _, mr := range r.Roots {
		err := mr.Close()
		if err != nil {
			return err
		}
	}
	return nil
}

2. 关闭根目录本身：func (r *Root) Forget() {
	err := r.Close()
	if err != nil {
		log.Error(err)
	}
}

函数的作用是：

1. 对根目录的子目录执行关闭操作，并在关闭操作成功后返回 nil。
2. 当文件系统被卸载时，调用根目录的 close() 函数，并在关闭操作成功后返回 nil。
3. 调用根目录的 forget() 函数，使得根目录及其子目录的 close() 函数不会返回错误。


```
func (r *Root) Close() error {
	for _, mr := range r.Roots {
		err := mr.Close()
		if err != nil {
			return err
		}
	}
	return nil
}

// Forget is called when the filesystem is unmounted. probably.
// see comments here: http://godoc.org/bazil.org/fuse/fs#FSDestroyer
func (r *Root) Forget() {
	err := r.Close()
	if err != nil {
		log.Error(err)
	}
}

```

这段代码实现了一个名为"ReadDirAll"的函数，该函数用于读取一个特定的目录，并返回一个大小为len(r.Keys)*2的 slice，其中 slice 包含当前目录及其子目录中的所有文件和子目录的 FUSE 类型信息。此外，该函数还返回一个指向 peerID 键的符号链接类型（即，符号链接）的 FUSE 类型信息。函数的实现主要分为以下几个步骤：

1. 首先定义了一个名为 "log.Debug" 的函数，用于在日志中输出一条 "Root ReadDirAll" 的消息。

2. 在函数的 body 中，首先创建了一个名为 "listing" 的 slice，其大小为 len(r.Keys) * 2。然后遍历了当前目录及其子目录中的所有文件和子目录，为每个文件创建了一个 FUSE 类型信息，并将其添加到 "listing" slice 中。

3. 循环遍历 "r.Keys" 切片，为每个文件创建了一个符号链接类型信息，并将其添加到 "listing" slice 中。

4. 返回 "listing" slice 和 nil 作为结果。

5. 在函数头部，没有做其他操作，只是定义了一个名为 "ReadDirAll" 的函数。


```
// ReadDirAll reads a particular directory. Will show locally available keys
// as well as a symlink to the peerID key.
func (r *Root) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
	log.Debug("Root ReadDirAll")

	listing := make([]fuse.Dirent, 0, len(r.Keys)*2)
	for alias, k := range r.Keys {
		ent := fuse.Dirent{
			Name: k.ID().String(),
			Type: fuse.DT_Dir,
		}
		link := fuse.Dirent{
			Name: alias,
			Type: fuse.DT_Link,
		}
		listing = append(listing, ent, link)
	}
	return listing, nil
}

```

这段代码定义了一个名为Directory的结构体，它通过一个mfs Directory来满足fuse fs接口。

定义了一个名为FileNode的结构体，它通过一个mfs File来满足fuse fs接口。

最后定义了一个名为File的结构体，它通过一个mfs FileDescriptor来满足fuse fs接口。

该代码看起来是为了定义一个目录节点（Directory）和一些与目录节点相关的类型，以及一个文件节点（FileNode）和与文件节点相关的类型。这些类型和结构体看起来用于在名为mfs的文件系统上提供一些fs接口的功能。


```
// Directory is wrapper over an mfs directory to satisfy the fuse fs interface.
type Directory struct {
	dir *mfs.Directory
}

type FileNode struct {
	fi *mfs.File
}

// File is wrapper over an mfs file to satisfy the fuse fs interface.
type File struct {
	fi mfs.FileDescriptor
}

// Attr returns the attributes of a given node.
```

这两段代码是两个名为"Attr"的函数，它们都是形参为*fuse.Attr的函数，接受一个名为*Directory的指针和一个名为*FileNode的指针参数。这两段代码在尝试获取目录或文件的相关属性时执行。

第一段代码的作用是获取目录 Attr。在函数中，首先输出一条日志消息，然后设置所选目录对象的 Attr 对象的 Mode 和 Uid，最后返回 nil，表示操作成功。

第二段代码的作用是获取文件 Attr。在函数中，首先输出一条日志消息，然后设置所选文件 Node 对象的 Attr 对象的 Mode、Size 和 Uid，最后返回 nil，表示操作成功。


```
func (d *Directory) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("Directory Attr")
	a.Mode = os.ModeDir | 0o555
	a.Uid = uint32(os.Getuid())
	a.Gid = uint32(os.Getgid())
	return nil
}

// Attr returns the attributes of a given node.
func (fi *FileNode) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("File Attr")
	size, err := fi.fi.Size()
	if err != nil {
		// In this case, the dag node in question may not be unixfs
		return fmt.Errorf("fuse/ipns: failed to get file.Size(): %s", err)
	}
	a.Mode = os.FileMode(0o666)
	a.Size = uint64(size)
	a.Uid = uint32(os.Getuid())
	a.Gid = uint32(os.Getgid())
	return nil
}

```

该代码定义了一个名为 "Directory" 的名为 "lookup" 的函数，其作用是查找给定目录中给定名称的子目录或文件的路径。

函数接收一个名为 "ctx" 的上下文参数，一个名为 "name" 的字符串参数和一个名为 "d" 的指向 "Directory" 类型对象的指针参数 "d"。函数内部使用 "dir.Child" 方法在目录中查找给定目录名称的子目录，如果查找成功，则返回子目录的路径，否则返回错误代码 "ENOENT"。

如果给定的 "name" 参数在 "dir" 对象中是无效类型，函数将输出错误代码 "ENOENT"。

函数的作用是帮助用户查找给定目录中给定名称的子目录或文件的路径，并在需要时返回指定的路径。


```
// Lookup performs a lookup under this node.
func (d *Directory) Lookup(ctx context.Context, name string) (fs.Node, error) {
	child, err := d.dir.Child(name)
	if err != nil {
		// todo: make this error more versatile.
		return nil, syscall.Errno(syscall.ENOENT)
	}

	switch child := child.(type) {
	case *mfs.Directory:
		return &Directory{dir: child}, nil
	case *mfs.File:
		return &FileNode{fi: child}, nil
	default:
		// NB: if this happens, we do not want to continue, unpredictable behaviour
		// may occur.
		panic("invalid type found under directory. programmer error.")
	}
}

```

这段代码定义了一个名为 `ReadDirAll` 的函数，它属于 `fuse.Directory` 类型。

该函数的作用是读取目录中所有链接结构（directory entries）并返回它们。函数首先使用 `d.dir.List` 函数获取目录中的所有链接结构。如果这个函数返回时出现错误，那么整个函数也会返回一个非空错误。如果目录中没有找到任何链接结构，函数就不会返回任何结果。

函数内部，首先定义了一个名为 `listing` 的变量，它用来存储目录中的链接结构。然后定义了一个名为 `entries` 的变量，它用来存储所有读取到的链接结构。

接下来，使用一个循环来遍历 `listing` 中的每个链接结构。在循环中，首先定义了一个 `fuse.Dirent` 类型的变量 `dirent`，它包含目录名称、类型以及名称等信息。然后使用 `switch` 语句来根据目录类型选择正确的链接类型。如果是 `mfs.TDir`，则说明这个链接结构是一个目录，需要返回它的类型为 `fuse.DT_Dir`。如果是 `mfs.TFile`，则说明这个链接结构是一个文件，需要返回它的类型为 `fuse.DT_File`。

在循环结束后，如果 `listing` 中所有链接结构都被成功读取，则返回 `entries` 切片，否则返回一个非空错误。


```
// ReadDirAll reads the link structure as directory entries.
func (d *Directory) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
	listing, err := d.dir.List(ctx)
	if err != nil {
		return nil, err
	}
	entries := make([]fuse.Dirent, len(listing))
	for i, entry := range listing {
		dirent := fuse.Dirent{Name: entry.Name}

		switch mfs.NodeType(entry.Type) {
		case mfs.TDir:
			dirent.Type = fuse.DT_Dir
		case mfs.TFile:
			dirent.Type = fuse.DT_File
		}

		entries[i] = dirent
	}

	if len(entries) > 0 {
		return entries, nil
	}
	return nil, syscall.Errno(syscall.ENOENT)
}

```

这段代码是一个名为`func`的函数，接受一个名为`File`的输入参数，返回一个名为`Read`的函数。

函数的作用是读取文件中的数据并返回，具体的实现过程如下：

1. 使用`Seek`函数请求文件从指定位置开始读取，如果遇到错误则返回。
2. 使用`Size`函数获取文件的大小，如果遇到错误则返回。
3. 创建一个名为`min`的函数，用于计算请求的数据最小长度，这个长度是输入数据大小和文件大小中的最小值。
4. 创建一个名为`CtxReadFull`的函数，用于在上下文上下文里读取文件全部的数据并返回，如果文件大小小于输入的数据大小，则只读取输入的数据。
5. 创建一个名为`Read`的函数，接受一个名为`ReadRequest`的输入参数和一个名为`ReadResponse`的输出参数，这个函数的作用就是调用`CtxReadFull`函数并返回读取的数据。
6. 在`Read`函数中，如果文件读取操作遇到错误，则返回错误信息，否则，根据`min`函数计算出的最小数据长度，读取文件全部的数据并返回。


```
func (fi *File) Read(ctx context.Context, req *fuse.ReadRequest, resp *fuse.ReadResponse) error {
	_, err := fi.fi.Seek(req.Offset, io.SeekStart)
	if err != nil {
		return err
	}

	fisize, err := fi.fi.Size()
	if err != nil {
		return err
	}

	select {
	case <-ctx.Done():
		return ctx.Err()
	default:
	}

	readsize := min(req.Size, int(fisize-req.Offset))
	n, err := fi.fi.CtxReadFull(ctx, resp.Data[:readsize])
	resp.Data = resp.Data[:n]
	return err
}

```

这两函数一起对File对象进行写入操作。

func (fi *File) Write(ctx context.Context, req *fuse.WriteRequest, resp *fuse.WriteResponse) error {
	// 确保在WriteAt方法中，data和offset都是有效的局部变量
	wrote, err := fi.fi.WriteAt(req.Data, req.Offset)
	if err != nil {
		return err
	}
	resp.Size = wrote
	return nil
}

func (fi *File) Flush(ctx context.Context, req *fuse.FlushRequest) error {
	errs := make(chan error, 1)
	go func() {
		errs <- fi.fi.Flush()
	}()
	select {
	case err := <-errs:
		return err
	case <-ctx.Done():
		return ctx.Err()
	}
}

在这两函数中，我们首先将请求的Data和Offset作为参数传给fi.fi.WriteAt函数。如果出现错误，我们将error作为返回值。如果一切正常，我们将返回0并设置WriteResponse的Size为写入的数据长度。

这两函数还会一起使用一个channel来传递errs。如果一个错误发生了，我们将 err作为返回值。如果所有的操作都成功完成，我们将err为空。


```
func (fi *File) Write(ctx context.Context, req *fuse.WriteRequest, resp *fuse.WriteResponse) error {
	// TODO: at some point, ensure that WriteAt here respects the context
	wrote, err := fi.fi.WriteAt(req.Data, req.Offset)
	if err != nil {
		return err
	}
	resp.Size = wrote
	return nil
}

func (fi *File) Flush(ctx context.Context, req *fuse.FlushRequest) error {
	errs := make(chan error, 1)
	go func() {
		errs <- fi.fi.Flush()
	}()
	select {
	case err := <-errs:
		return err
	case <-ctx.Done():
		return ctx.Err()
	}
}

```

这段代码是一个函数，名为 `func (fi *File) Setattr(ctx context.Context, req *fuse.SetattrRequest, resp *fuse.SetattrResponse) error`。它接收两个指针参数：`fi` 和 `req`，分别代表一个文件设备和请求设置属性。函数的作用是在文件系统上设置一个名为 `req.Name` 的属性，并将给定的大小设置给 `req.Size`。如果 `req.Valid` 的长度小于给定的大小，函数不会执行。如果已设置的大小不正确，函数会尝试截断文件系统以设置正确的大小，并返回相应的错误。如果函数在执行过程中遇到任何错误，它将返回一个非空错误对象。


```
func (fi *File) Setattr(ctx context.Context, req *fuse.SetattrRequest, resp *fuse.SetattrResponse) error {
	if req.Valid.Size() {
		cursize, err := fi.fi.Size()
		if err != nil {
			return err
		}
		if cursize != int64(req.Size) {
			err := fi.fi.Truncate(int64(req.Size))
			if err != nil {
				return err
			}
		}
	}
	return nil
}

```

这段代码定义了一个名为 `FileNode` 的类型，它实现了 `fuse.FileNode` 接口。这个 `FileNode` 类型有一个 `Fsync` 方法，它的作用是同步地将文件内容从内存flush到磁盘。

该代码的作用是执行一个完整的文件flush操作，将内存中的所有内容写入磁盘。这个方法需要在文件系统的根目录更新之后才能真正完成写入操作。在执行flush操作之前，需要确保所有读写操作都已完成，这是因为，在MFS中，一个写入操作只有在根目录更新之后才能被持久化。

由于flush操作需要写入磁盘，因此必须确保所有读写操作都已完成，才能保证flush操作成功。如果在这个过程中出现错误，则返回错误。如果flush操作完成后，由于任何原因操作未完成，则返回上一个错误。


```
// Fsync flushes the content in the file to disk.
func (fi *FileNode) Fsync(ctx context.Context, req *fuse.FsyncRequest) error {
	// This needs to perform a *full* flush because, in MFS, a write isn't
	// persisted until the root is updated.
	errs := make(chan error, 1)
	go func() {
		errs <- fi.fi.Flush()
	}()
	select {
	case err := <-errs:
		return err
	case <-ctx.Done():
		return ctx.Err()
	}
}

```

这两段代码定义了两个函数，主要作用是文件 I/O 相关的操作。

1. `func (fi *File) Forget()` 函数接收一个 `File` 类型的参数 `fi`，并且这个函数本身没有做任何实际的逻辑，只是将 `fi` 重置为其初始状态（可能是已经打开或者已关闭的状态）。

2. `func (d *Directory) Mkdir(ctx context.Context, req *fuse.MkdirRequest) (fs.Node, error)` 函数接收一个 `Directory` 类型的参数 `d` 和一个 `MkdirRequest` 类型的参数 `req`，并且这个函数本身也并没有做任何实际的逻辑，只是将 `d` 的目录小孩创建一个新的目录，并且返回创建后 `fs.Node` 类型的值和可能的错误。

这两个函数在实际的应用场景中，可能会被其他更核心的函数或业务逻辑所依赖，所以具体的使用方式还需要根据具体需求来调整。


```
func (fi *File) Forget() {
	// TODO(steb): this seems like a place where we should be *uncaching*, not flushing.
	err := fi.fi.Flush()
	if err != nil {
		log.Debug("forget file error: ", err)
	}
}

func (d *Directory) Mkdir(ctx context.Context, req *fuse.MkdirRequest) (fs.Node, error) {
	child, err := d.dir.Mkdir(req.Name)
	if err != nil {
		return nil, err
	}

	return &Directory{dir: child}, nil
}

```

这段代码是一个名为`func`的函数，接受两个参数：一个`FileNode`类型的整数切片和一个`fuse.OpenRequest`类型的整数切片。

该函数的作用是打开一个文件或者创建一个新文件，并返回文件的文件描述符（fs.Handle）或者错误。

具体来说，函数的实现分为以下几个步骤：

1. 读取文件描述符并检查是否为空，如果是，则直接返回`nil`和`nil`。
2. 如果文件描述符不为`nil`，则继续执行下一步。
3. 尝试打开文件，并检查请求是否包含`fuse.OpenTruncate`标志。如果是，那么有一些限制：只能读或者写，不能同时读写；必须同步。
4. 如果请求包含`fuse.OpenTruncate`标志，那么尝试 truncate（写入）文件。如果`fuse.OpenTruncate`标志是错误的，则记录错误并返回错误代码。
5. 如果请求包含`fuse.OpenAppend`标志，那么尝试追加数据到文件。如果`fuse.OpenAppend`标志是错误的，则记录错误并返回错误代码。
6. 最后，返回文件的文件描述符。

函数的实现基于两个前提条件：

1. `FileNode`是一个包含文件元数据的结构体，提供了对文件的访问能力。
2. `fuse.OpenRequest`是一个包含文件描述符、权限和其他选项的请求结构体，用于描述如何打开或关闭文件。

另外，如果函数在执行过程中遇到错误，会将其记录在函数内部，并返回错误代码。


```
func (fi *FileNode) Open(ctx context.Context, req *fuse.OpenRequest, resp *fuse.OpenResponse) (fs.Handle, error) {
	fd, err := fi.fi.Open(mfs.Flags{
		Read:  req.Flags.IsReadOnly() || req.Flags.IsReadWrite(),
		Write: req.Flags.IsWriteOnly() || req.Flags.IsReadWrite(),
		Sync:  true,
	})
	if err != nil {
		return nil, err
	}

	if req.Flags&fuse.OpenTruncate != 0 {
		if req.Flags.IsReadOnly() {
			log.Error("tried to open a readonly file with truncate")
			return nil, syscall.Errno(syscall.ENOTSUP)
		}
		log.Info("Need to truncate file!")
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

		_, err := fd.Seek(0, io.SeekEnd)
		if err != nil {
			log.Error("seek reset failed: ", err)
			return nil, err
		}
	}

	return &File{fi: fd}, nil
}

```

这两段代码定义了两个函数，分别作用于文件设备和目录。

1. `func (fi *File) Release(ctx context.Context, req *fuse.ReleaseRequest) error {
	return fi.fi.Close()
}`

这个函数接收一个文件设备对象 `fi` 和一个 `fuse.ReleaseRequest` 结构体，负责关闭文件设备并返回。它通过调用 `fi.fi.Close()` 来关闭文件设备，并返回一个错误。

2. `func (d *Directory) Create(ctx context.Context, req *fuse.CreateRequest, resp *fuse.CreateResponse) (fs.Node, fs.Handle, error)`

这个函数接收一个目录对象 `d` 和一个 `fuse.CreateRequest` 结构体，负责创建一个新的目录并返回其数据。它首先创建一个新文件，然后返回新文件的路径、文件类型和错误。

`d.Create` 函数接收一个目录名称 `req.Name` 和一个 `fuse.CreateRequest` 结构体，负责创建一个新的目录。它首先检查 `d` 是否为空，如果是，则创建一个新目录并返回其名称。如果 `d` 不是空，则创建目录中的新文件并返回其名称、文件类型和错误。


```
func (fi *File) Release(ctx context.Context, req *fuse.ReleaseRequest) error {
	return fi.fi.Close()
}

func (d *Directory) Create(ctx context.Context, req *fuse.CreateRequest, resp *fuse.CreateResponse) (fs.Node, fs.Handle, error) {
	// New 'empty' file
	nd := dag.NodeWithData(ft.FilePBData(nil, 0))
	err := d.dir.AddChild(req.Name, nd)
	if err != nil {
		return nil, nil, err
	}

	child, err := d.dir.Child(req.Name)
	if err != nil {
		return nil, nil, err
	}

	fi, ok := child.(*mfs.File)
	if !ok {
		return nil, nil, errors.New("child creation failed")
	}

	nodechild := &FileNode{fi: fi}

	fd, err := fi.Open(mfs.Flags{
		Read:  req.Flags.IsReadOnly() || req.Flags.IsReadWrite(),
		Write: req.Flags.IsWriteOnly() || req.Flags.IsReadWrite(),
		Sync:  true,
	})
	if err != nil {
		return nil, nil, err
	}

	return nodechild, &File{fi: fd}, nil
}

```

这两函数是用来操作FUSE文件系统的。

`func (d *Directory) Remove(ctx context.Context, req *fuse.RemoveRequest) error {
	err := d.dir.Unlink(req.Name)
	if err != nil {
		return syscall.Errno(syscall.ENOENT)
	}
	return nil
}`

这个函数接收一个指针变量 `d` 和一个 `fuse.RemoveRequest` 结构体 `req` 作为参数。它通过调用 `d.dir.Unlink` 来删除文件或者目录，如果出现错误，将返回一个 `syscall.Errno`。

`func (d *Directory) Rename(ctx context.Context, req *fuse.RenameRequest, newDir fs.Node) error {
	cur, err := d.dir.Child(req.OldName)
	if err != nil {
		return err
	}

	err = d.dir.Unlink(req.OldName)
	if err != nil {
		return err
	}

	switch newDir.(type) {
	case *Directory:
		nd, err := cur.GetNode()
		if err != nil {
			return err
		}

		err = newDir.dir.AddChild(req.NewName, nd)
		if err != nil {
			return err
		}
	case *FileNode:
		log.Error("Cannot move node into a file!")
		return syscall.Errno(syscall.EPERM)
	default:
		log.Error("Unknown node type for rename target dir!")
		return errors.New("unknown fs node type")
	}
	return nil
}`

这两个函数都接受一个 `fuse.RemoveRequest` 和一个 `fs.Node` 类型的参数，并返回一个 `error`。第一个函数是 `Directory` 类的 `Remove` 函数，它通过调用 `Unlink` 和 `Link` 方法来删除文件或者目录。第二个函数是 `Directory` 类的 `Rename` 函数，它实现了 `NodeRenamer` 接口，实现了 FUSE 文件系统的 `rename` 操作。


```
func (d *Directory) Remove(ctx context.Context, req *fuse.RemoveRequest) error {
	err := d.dir.Unlink(req.Name)
	if err != nil {
		return syscall.Errno(syscall.ENOENT)
	}
	return nil
}

// Rename implements NodeRenamer.
func (d *Directory) Rename(ctx context.Context, req *fuse.RenameRequest, newDir fs.Node) error {
	cur, err := d.dir.Child(req.OldName)
	if err != nil {
		return err
	}

	err = d.dir.Unlink(req.OldName)
	if err != nil {
		return err
	}

	switch newDir := newDir.(type) {
	case *Directory:
		nd, err := cur.GetNode()
		if err != nil {
			return err
		}

		err = newDir.dir.AddChild(req.NewName, nd)
		if err != nil {
			return err
		}
	case *FileNode:
		log.Error("Cannot move node into a file!")
		return syscall.Errno(syscall.EPERM)
	default:
		log.Error("Unknown node type for rename target dir!")
		return errors.New("unknown fs node type")
	}
	return nil
}

```

这段代码定义了一个名为min的函数，它接收两个整数参数a和b，并返回较小参数a的值。

在函数内部，首先检查a是否小于b，如果是，则返回a，否则返回b。

该函数在代码中使用了if语句，当a小于b时，返回a；否则返回b。

接下来定义了一个名为ipnsRoot的类型，它表示一个根节点接口的实现。ipnsRoot包含三个方法：fs.Node、fs.HandleReadDirAller和fs.NodeStringLookuper。

最后，该函数创建了一个名为_ipnsRoot的指针变量，该指针被初始化为nil，即没有实际值。


```
func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

// to check that out Node implements all the interfaces we want.
type ipnsRoot interface {
	fs.Node
	fs.HandleReadDirAller
	fs.NodeStringLookuper
}

var _ ipnsRoot = (*Root)(nil)

```

这段代码定义了一个名为ipnsDirectory的接口类型和一个名为ipnsFile的接口类型。ipnsDirectory定义了一个FS目录接口，包含了fs.HandleReadDirAller、fs.Node、fs.NodeCreater、fs.NodeMkdirEder、fs.NodeRemover和fs.NodeRenamer这几个fs.Handle函数类型。而ipnsFile定义了一个FS文件接口，包含了fs.HandleFlanner、fs.HandleReader、fs.HandleWriter和fs.HandleReleaser这几个fs.Handle函数类型。

ipnsDirectory是一个指针类型变量，它存储了一个FS目录对象。然后，代码通过将nil赋值给ipnsDirectory，将其存储了一个空的FS目录对象。

ipnsFile是一个指针类型变量，它存储了一个FS文件对象。在代码的后续部分，ipnsFile被用来创建和操作FS文件。


```
type ipnsDirectory interface {
	fs.HandleReadDirAller
	fs.Node
	fs.NodeCreater
	fs.NodeMkdirer
	fs.NodeRemover
	fs.NodeRenamer
	fs.NodeStringLookuper
}

var _ ipnsDirectory = (*Directory)(nil)

type ipnsFile interface {
	fs.HandleFlusher
	fs.HandleReader
	fs.HandleWriter
	fs.HandleReleaser
}

```

这段代码定义了一个名为ipnsFileNode的类型，该类型实现了三个名为fs.Node、fs.NodeFsyncer和fs.NodeOpener的接口。

然后，分别创建了一个名为_ipnsFileNode的指针和一个名为_ipnsFile的指针，且两者都为nil，即都代表一个空的文件节点对象。

通过这两行代码，ipnsFileNode和_ipnsFileNode都被初始化了，但ipnsFileNode是类型，而_ipnsFileNode是变量。类型通常用于定义变量所需的数据类型，而变量则用于存储该类型的实例。


```
type ipnsFileNode interface {
	fs.Node
	fs.NodeFsyncer
	fs.NodeOpener
}

var (
	_ ipnsFileNode = (*FileNode)(nil)
	_ ipnsFile     = (*File)(nil)
)

```

# `/opt/kubo/fuse/ipns/link_unix.go`

这段代码是一个 Go 语言编写的 FUSE 文件系统模块，它定义了一个名为 "ipns" 的包。这个模块的作用是创建一个名为 "/ipns" 的目录，并在其中创建一个名为 "ipns-manifest.yaml" 的文件。

具体来说，这段代码实现了以下操作：

1. 使用 `build` 命令指定只在此处编译，而不是在运行时编译。
2. 使用 `nofuse` 和 `openbsd` 环境变量，避免了使用它们所指定的系统。
3. 使用 `netbsd` 和 `plan9` 环境变量，避免了使用它们所指定的系统。
4. 使用 `nofuse` 和 `openbsd` 环境变量，并设置 `Link` 类型的结构体 `Link` 的 `Target` 字段为 "/ipns"。
5. 创建名为 "ipns-manifest.yaml" 的文件，其中包含一个名为 "ipns" 的 FUSE 文件系统模块定义。


```
//go:build !nofuse && !openbsd && !netbsd && !plan9
// +build !nofuse,!openbsd,!netbsd,!plan9

package ipns

import (
	"context"
	"os"

	"bazil.org/fuse"
	"bazil.org/fuse/fs"
)

type Link struct {
	Target string
}

```

该代码定义了两个函数，分别为`func`和`func (l *Link)`。这两个函数的作用如下：

1. `func`函数接受一个`Link`类型的参数`l`和一个`fuse.Attr`类型的参数`a`，返回一个`error`类型的变量。函数的作用是将`a`设置为`l`的`Target`属性，并输出一条日志信息。具体实现如下：
java
func (l *Link) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("Link attr.")
	a.Mode = os.ModeSymlink | 0o555
	return nil
}

2. `func (l *Link)`函数接受一个`Link`类型的参数`l`，和一个`fuse.ReadlinkRequest`类型的参数`req`，返回一个`string`类型的变量和一个`error`类型的变量。函数的作用是返回`l`的`Target`属性，并输出一条日志信息。具体实现如下：
kotlin
func (l *Link) Readlink(ctx context.Context, req *fuse.ReadlinkRequest) (string, error) {
	log.Debugf("ReadLink: %s", l.Target)
	return l.Target, nil
}

另外，该代码还定义了一个`var _ fs.NodeReadlinker = (*Link)(nil)`类型的变量，用于表示`Link`类型的指针`l`。


```
func (l *Link) Attr(ctx context.Context, a *fuse.Attr) error {
	log.Debug("Link attr.")
	a.Mode = os.ModeSymlink | 0o555
	return nil
}

func (l *Link) Readlink(ctx context.Context, req *fuse.ReadlinkRequest) (string, error) {
	log.Debugf("ReadLink: %s", l.Target)
	return l.Target, nil
}

var _ fs.NodeReadlinker = (*Link)(nil)

```

# `/opt/kubo/fuse/ipns/mount_unix.go`

这段代码是一个用于在IPFS中挂载不同发行版Linux、Darwin、FreeBSD和NetBSD的IPNS子系统的工具。它通过在IPFS中安装所需的软件包，并检查IPFS是否支持挂载其他发行版，从而允许用户在不同发行版之间进行无缝的IPFS共享。

具体来说，代码首先定义了一个名为"ipns"的包，其中包含了一些用于与IPFS和Kubernetes进行交互的函数和结构体。然后，代码定义了一个名为"Mount"的函数，用于将IPNS挂载到指定的位置，并返回一个名为"mount.Mount"的接口。

接下来，代码通过使用"github.com/ipfs/kubo/core"库从IPFS中导入"core"和"coreapi"功能，并使用"github.com/ipfs/kubo/fuse/mount"库中的"mount"函数来创建一个挂载点。最后，代码通过检查IPFS配置中的"Mounts.FuseAllowOther"选项来决定是否允许其他发行版的IPNS。


```
//go:build (linux || darwin || freebsd || netbsd || openbsd) && !nofuse
// +build linux darwin freebsd netbsd openbsd
// +build !nofuse

package ipns

import (
	core "github.com/ipfs/kubo/core"
	coreapi "github.com/ipfs/kubo/core/coreapi"
	mount "github.com/ipfs/kubo/fuse/mount"
)

// Mount mounts ipns at a given location, and returns a mount.Mount instance.
func Mount(ipfs *core.IpfsNode, ipnsmp, ipfsmp string) (mount.Mount, error) {
	coreAPI, err := coreapi.NewCoreAPI(ipfs)
	if err != nil {
		return nil, err
	}

	cfg, err := ipfs.Repo.Config()
	if err != nil {
		return nil, err
	}

	allowOther := cfg.Mounts.FuseAllowOther

	fsys, err := NewFileSystem(ipfs.Context(), coreAPI, ipfsmp, ipnsmp)
	if err != nil {
		return nil, err
	}

	return mount.NewMount(ipfs.Process, fsys, ipnsmp, allowOther)
}

```

# `/opt/kubo/fuse/mount/fuse.go`

这段代码是一个 Go 语言编写的 FUSE 文件系统相关的构建脚本，用于编译依赖库并安装必要的工具和设置环境。

首先，它使用 `go build` 命令构建依赖库，这将生成一个名为 `mount.go. built` 的二进制文件。

接下来，它使用 `!nofuse` 和 `!windows` 条件语句来禁止或允许在当前环境中使用 FUSE。

然后，它使用 `!openbsd` 和 `!netbsd` 条件语句来禁止或允许使用 OpenBSD 和 NetBSD。

此外，它使用 `!plan9` 条件语句来禁止或允许使用 Plan 9。

最后，它使用 `+build !nofuse,!windows,!openbsd,!netbsd,!plan9` 命令编译依赖库，并在编译时安装必要的工具和设置环境。

该代码主要用于在 FUSE 文件系统环境中编译必要的工具和设置环境，以便从 FUSE 仓库中构建自定义的 Go 语言应用程序。


```
//go:build !nofuse && !windows && !openbsd && !netbsd && !plan9
// +build !nofuse,!windows,!openbsd,!netbsd,!plan9

package mount

import (
	"errors"
	"fmt"
	"sync"
	"time"

	"bazil.org/fuse"
	"bazil.org/fuse/fs"
	"github.com/jbenet/goprocess"
)

```

这段代码定义了一个名为“ErrNotMounted”的错误类型，该类型由一个字符串和一个新的错误消息构成，错误消息描述了“fuse 未挂载”的情况。

接着，定义了一个名为“mount”的结构体，该结构体包含一个用于挂载 FUSE FS 的 mount 点、一个文件系统客户端（如 fs.FS）和一个 FUSE 连接客户端（如 fuse.Conn）。

然后，该结构体包含一个名为“active”的布尔值，表示是否正在挂载 FUSE FS，以及一个名为“activeLock”的互斥锁，用于同步挂载过程中对多个文件的读写操作。

最后，该结构体包含一个名为“proc”的 Go 进程函数，用于执行挂载操作的上下文处理和错误处理。


```
var ErrNotMounted = errors.New("not mounted")

// mount implements go-ipfs/fuse/mount.
type mount struct {
	mpoint   string
	filesys  fs.FS
	fuseConn *fuse.Conn

	active     bool
	activeLock *sync.RWMutex

	proc goprocess.Process
}

// Mount mounts a fuse fs.FS at a given location, and returns a Mount instance.
```

这段代码定义了一个名为NewMount的函数，它接受一个ContextGroup、一个FS实例和一个挂载点。它的作用是将挂载点与ContextGroup绑定，并在绑定的过程中执行mount操作。

具体来说，代码首先创建一个FS连接，然后设置挂载点。接下来，代码根据传入的参数（allowOther参数）来决定是否允许其他进程访问挂载点。最后，代码创建一个名为mount的Mount对象，将fuseconn设置为创建的FS连接，并设置其他一些Mount选项。函数返回一个Mount对象和一个 nil的错误。

由于在代码中没有对传入参数进行校验，因此在实际应用中需要添加相应的错误处理和参数检查。


```
// parent is a ContextGroup to bind the mount's ContextGroup to.
func NewMount(p goprocess.Process, fsys fs.FS, mountpoint string, allowOther bool) (Mount, error) {
	var conn *fuse.Conn
	var err error

	mountOpts := []fuse.MountOption{
		fuse.MaxReadahead(64 * 1024 * 1024),
		fuse.AsyncRead(),
	}

	if allowOther {
		mountOpts = append(mountOpts, fuse.AllowOther())
	}
	conn, err = fuse.Mount(mountpoint, mountOpts...)

	if err != nil {
		return nil, err
	}

	m := &mount{
		mpoint:     mountpoint,
		fuseConn:   conn,
		filesys:    fsys,
		active:     false,
		activeLock: &sync.RWMutex{},
		proc:       goprocess.WithParent(p), // link it to parent.
	}
	m.proc.SetTeardown(m.unmount)

	// launch the mounting process.
	if err := m.mount(); err != nil {
		_ = m.Unmount() // just in case.
		return nil, err
	}

	return m, nil
}

```

这段代码定义了一个名为 mount 的函数，接受一个名为 mount 的参数，该函数用于将文件系统挂载到 FUSE 缓冲区设备 mount 上面。

函数内部首先输出正在挂载的文件系统的名称，然后使用 fs.Serve 函数挂载文件系统。使用一个 goroutine 来负责等待 fs.Serve 函数完成挂载操作，并在完成时通知主程序。在主程序中，会循环等待 fs.Serve 函数返回、错误 channel 中的错误，或者挂载完成。

如果挂载过程中出现错误，函数会返回该错误，否则将文件系统设置为可用，并输出已挂载的文件系统的名称。


```
func (m *mount) mount() error {
	log.Infof("Mounting %s", m.MountPoint())

	errs := make(chan error, 1)
	go func() {
		// fs.Serve blocks until the filesystem is unmounted.
		err := fs.Serve(m.fuseConn, m.filesys)
		log.Debugf("%s is unmounted", m.MountPoint())
		if err != nil {
			log.Debugf("fs.Serve returned (%s)", err)
			errs <- err
		}
		m.setActive(false)
	}()

	// wait for the mount process to be done, or timed out.
	select {
	case <-time.After(MountTimeout):
		return fmt.Errorf("mounting %s timed out", m.MountPoint())
	case err := <-errs:
		return err
	case <-m.fuseConn.Ready:
	}

	// check if the mount process has an error to report
	if err := m.fuseConn.MountError; err != nil {
		return err
	}

	m.setActive(true)

	log.Infof("Mounted %s", m.MountPoint())
	return nil
}

```

这段代码是一个高阶函数，接收一个`mount`对象作为参数。该函数的作用是尝试卸载本地服务器上某个磁盘 mount 目录，并输出相应的错误信息。

具体来说，该函数执行以下操作：

1. 使用 fuse 库尝试卸载磁盘 mount 目录。如果操作成功，则设置 `m` 为非活动的状态，并返回 nil。
2. 如果第一步失败，则关闭与 fuse 连接的客户端，并设置 `m` 为活动的状态。
3. 如果以上两种情况中任何一种失败，则调用 `ForceUnmountManyTimes` 函数强制卸载磁盘 mount 目录。
4. 如果所有的尝试都失败，则输出相应的错误信息，并设置 `m` 为非活动的状态。
5. 如果成功卸载，则输出相应的信息并设置 `m` 为非活动的状态。

该函数的作用是确保在尝试失败时，能够正确地关闭与 fuse 连接的客户端，以免对系统造成不必要的伤害。


```
// umount is called exactly once to unmount this service.
// note that closing the connection will not always unmount
// properly. If that happens, we bring out the big guns
// (mount.ForceUnmountManyTimes, exec unmount).
func (m *mount) unmount() error {
	log.Infof("Unmounting %s", m.MountPoint())

	// try unmounting with fuse lib
	err := fuse.Unmount(m.MountPoint())
	if err == nil {
		m.setActive(false)
		return nil
	}
	log.Warnf("fuse unmount err: %s", err)

	// try closing the fuseConn
	err = m.fuseConn.Close()
	if err == nil {
		m.setActive(false)
		return nil
	}
	log.Warnf("fuse conn error: %s", err)

	// try mount.ForceUnmountManyTimes
	if err := ForceUnmountManyTimes(m, 10); err != nil {
		return err
	}

	log.Infof("Seemingly unmounted %s", m.MountPoint())
	m.setActive(false)
	return nil
}

```

这段代码定义了一个名为mount的结构的体，该结构体包含一个名为proc的成员和一个名为mpoint的成员。

函数1 (Process)：
该函数接收一个名为mount的整数指针变量，并返回该结构体中名为proc的成员的函数指针。函数的作用是处理与mount相关的事务。

函数2 (MountPoint)：
该函数接收一个名为mount的整数指针变量，并返回该结构体中名为mpoint的成员的值。函数的作用是获取mount的根目录。

函数3 (Unmount)：
该函数接收一个名为mount的整数指针变量，并在该结构体中名为isActive的成员为假时执行。函数的作用是关闭与mount相关的事务，并返回一个非-错误的结果。


```
func (m *mount) Process() goprocess.Process {
	return m.proc
}

func (m *mount) MountPoint() string {
	return m.mpoint
}

func (m *mount) Unmount() error {
	if !m.IsActive() {
		return ErrNotMounted
	}

	// call Process Close(), which calls unmount() exactly once.
	return m.proc.Close()
}

```

这两段代码都是定义在名为`mount`的结构的指针变量`m`上。

这两段代码都使用了Clinical Reusability改建（ORM）库，通过这种方式，将数据结构的锁操作与数据结构的访问操作分离，以便在不影响其他客户端代码的情况下，允许不同的开发人员在多并发下对数据结构进行操作。

这两段代码具体实现如下：

1. `func (m *mount) IsActive() bool`函数的作用是判断`mount`数据结构是否处于激活状态。具体流程如下：

  a. 调用`m.activeLock.RLock()`，获取到当前`mount`数据结构所在的内存位置的锁，并获取锁的`RLock()`方法返回的`true`或`false`值。
  b. 调用`m.activeLock.RUnlock()`，释放当前锁。
  c. 由于锁已经被释放，所以直接返回`m.active`的值，即当前`mount`数据结构是否处于激活状态。

1. `func (m *mount) setActive(a bool)`函数的作用是设置`mount`数据结构的激活状态。具体流程如下：

  a. 调用`m.activeLock.Lock()`，获取到当前`mount`数据结构所在的内存位置的锁，并获取锁的`Lock()`方法返回的`true`或`false`值。
  b. 如果当前的`mount`数据结构尚未被激活，则执行以下操作：
     - 调用`m.activeLock.setActive(a)`，设置`mount`数据结构的激活状态为`a`。
     - 调用`m.activeLock.Unlock()`，释放当前锁。

这两段代码，通过对`mount`数据结构的锁操作与访问操作的分离，使得多个开发人员可以在多个并发请求下，对数据结构进行操作，而不会导致数据结构的竞争条件。


```
func (m *mount) IsActive() bool {
	m.activeLock.RLock()
	defer m.activeLock.RUnlock()

	return m.active
}

func (m *mount) setActive(a bool) {
	m.activeLock.Lock()
	m.active = a
	m.activeLock.Unlock()
}

```

# `/opt/kubo/fuse/mount/mount.go`

这段代码定义了一个名为"mount"的包，提供了一个对挂载点的简单抽象。这个包通过使用"fmt"函数打印日志信息，以及使用"io"、"os/exec"和"time"包来处理文件系统操作，来实现在挂载点上执行命令。最后，使用"github.com/ipfs/go-log"和"github.com/jbenet/goprocess"包来在执行操作时记录和处理I/O操作和进程。


```
// package mount provides a simple abstraction around a mount point
package mount

import (
	"fmt"
	"io"
	"os/exec"
	"runtime"
	"time"

	logging "github.com/ipfs/go-log"
	goprocess "github.com/jbenet/goprocess"
)

var log = logging.Logger("mount")

```

这段代码定义了一个名为Mount的接口，用于表示一个文件系统的挂载。Mount接口定义了MountPoint，Unmount和IsActive方法以及一个名为Process的 checksum。

MountTimeout定义了一个名为MountTimeout的常量，表示挂载超时的时间。

该代码没有输出任何函数或变量，因此它无法直接挂载文件系统。它定义了一个Mount接口，可以用来定义文件系统的挂载点，是否处于活跃状态以及返回一个Process类型的挂载的goprocess.Process。


```
var MountTimeout = time.Second * 5

// Mount represents a filesystem mount.
type Mount interface {
	// MountPoint is the path at which this mount is mounted
	MountPoint() string

	// Unmounts the mount
	Unmount() error

	// Checks if the mount is still active.
	IsActive() bool

	// Process returns the mount's Process to be able to link it
	// to other processes. Unmount upon closing.
	Process() goprocess.Process
}

```

这段代码是一个名为 `ForceUnmount` 的函数，它用于尝试强制卸载指定挂载点（Mount）的文件系统。它通过直接调用 `diskutil` 或 `fusermount` 来做到这一点。

具体来说，这段代码的功能如下：

1. 检查挂载点是否存在，如果不存在，则输出一条警告信息。
2. 如果挂载点存在，则尝试使用 `unmount` 命令卸载。如果失败，创建一个名为 `errc` 的通道，用于存储错误信息。
3. 创建一个名为 `err` 的变量来存储错误信息。
4. 使用一个 `select` 语句来等待 7 秒钟，如果超时，则输出一个错误信息，并返回。
5. 如果错误信息是通过 `errc` 通道收到的，那么错误信息将包含在 `err` 变量中，并返回。

通过 `ForceUnmount` 函数，可以强制卸载挂载点文件系统的挂载点。


```
// ForceUnmount attempts to forcibly unmount a given mount.
// It does so by calling diskutil or fusermount directly.
func ForceUnmount(m Mount) error {
	point := m.MountPoint()
	log.Warnf("Force-Unmounting %s...", point)

	cmd, err := UnmountCmd(point)
	if err != nil {
		return err
	}

	errc := make(chan error, 1)
	go func() {
		defer close(errc)

		// try vanilla unmount first.
		if err := exec.Command("umount", point).Run(); err == nil {
			return
		}

		// retry to unmount with the fallback cmd
		errc <- cmd.Run()
	}()

	select {
	case <-time.After(7 * time.Second):
		return fmt.Errorf("umount timeout")
	case err := <-errc:
		return err
	}
}

```

这段代码定义了一个名为"UnmountCmd"的函数，它返回一个执行器（exec.Cmd）和一个错误（error）。函数的作用是创建一个GOOS特定的"unmount"命令，用于挂载FUSE驱动的文件系统。

具体来说，函数根据运行时编译器设置的目标操作系统（GOOS）来执行不同的命令。如果目标操作系统是"darwin"，则函数使用"diskutil"命令来挂载文件系统并强制卸载，如果目标操作系统是"linux"，则函数使用"fusermount"命令来挂载文件系统并强制卸载。如果目标操作系统是"default"（默认），则函数产生一个错误。

函数的实现依赖于运行时编译器设置的目标操作系统。如果目标操作系统没有被设置，函数的行为将会是未定义的。


```
// UnmountCmd creates an exec.Cmd that is GOOS-specific
// for unmount a FUSE mount.
func UnmountCmd(point string) (*exec.Cmd, error) {
	switch runtime.GOOS {
	case "darwin":
		return exec.Command("diskutil", "umount", "force", point), nil
	case "linux":
		return exec.Command("fusermount", "-u", point), nil
	default:
		return nil, fmt.Errorf("unmount: unimplemented")
	}
}

// ForceUnmountManyTimes attempts to forcibly unmount a given mount,
// many times. It does so by calling diskutil or fusermount directly.
```

这段代码定义了一个名为 "ForceUnmountManyTimes" 的函数，尝试执行多次给定的次数，挂载一个目录（Mount）。

具体来说，该函数接收两个参数：一个表示尝试次数的整数（attempts）和一个用于存储挂载点（Mount）的变量。函数内部，首先创建一个名为 "err" 的错误变量，然后执行一系列挂载点的挂载操作。

接下来，代码块使用一个 for 循环来重复执行挂载操作。在循环内部，函数首先检查给定的错误（err）是否为空。如果是，函数直接返回该错误。否则，代码块使用一个准许时间（允许最多 500 毫秒的延迟）来等待一段时间，然后再次尝试挂载点。

最后，代码块通过调用 "fmt.Errorf" 函数并传入一个字符串和一个目录的挂载点，来输出错误消息。


```
// Attempts a given number of times.
func ForceUnmountManyTimes(m Mount, attempts int) error {
	var err error
	for i := 0; i < attempts; i++ {
		err = ForceUnmount(m)
		if err == nil {
			return err
		}

		<-time.After(time.Millisecond * 500)
	}
	return fmt.Errorf("unmount %s failed after 10 seconds of trying", m.MountPoint())
}

type closer struct {
	M Mount
}

```

这两段代码定义了一个名为Close的函数，它接收一个名为c的指针，代表一个Closer类型。函数的作用是关闭与c指向的设备的联系，并返回一个Closer类型。

函数内部首先输出一条警告信息，指出设备的挂载点，然后尝试关闭与设备的联系。如果关闭失败，函数返回一个error类型的值。

Close函数的实现主要依赖于Closer类型的代表c，它包含一个Unmount函数，用于关闭与设备的联系。由于没有定义Unmount函数，因此无法确定Close函数的行为。


```
func (c *closer) Close() error {
	log.Warn(" (c *closer) Close(),", c.M.MountPoint())
	return c.M.Unmount()
}

func Closer(m Mount) io.Closer {
	return &closer{m}
}

```