# `kubo\core\commands\files.go`

```go
package commands

import (
    "context"  // 上下文包，用于控制命令执行的上下文
    "errors"   // 错误包，用于处理错误
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，用于处理输入输出
    "os"       // 操作系统包，用于操作系统相关功能
    gopath "path"  // 路径包，用于处理文件路径
    "sort"     // 排序包，用于排序
    "strings"  // 字符串包，用于处理字符串

    humanize "github.com/dustin/go-humanize"  // 人性化包，用于格式化输出人性化的数据
    "github.com/ipfs/kubo/core"  // IPFS核心包，用于IPFS核心功能
    "github.com/ipfs/kubo/core/commands/cmdenv"  // IPFS命令包，用于IPFS命令相关功能

    bservice "github.com/ipfs/boxo/blockservice"  // 区块服务包，用于区块服务相关功能
    offline "github.com/ipfs/boxo/exchange/offline"  // 离线交换包，用于离线交换相关功能
    dag "github.com/ipfs/boxo/ipld/merkledag"  // IPLD默克尔有向无环图包，用于IPLD相关功能
    ft "github.com/ipfs/boxo/ipld/unixfs"  // UnixFS包，用于UnixFS相关功能
    mfs "github.com/ipfs/boxo/mfs"  // MFS包，用于MFS相关功能
    "github.com/ipfs/boxo/path"  // 路径包，用于处理路径
    cid "github.com/ipfs/go-cid"  // CID包，用于CID相关功能
    cidenc "github.com/ipfs/go-cidutil/cidenc"  // CID编码包，用于CID编码相关功能
    cmds "github.com/ipfs/go-ipfs-cmds"  // IPFS命令包，用于IPFS命令相关功能
    ipld "github.com/ipfs/go-ipld-format"  // IPLD包，用于IPLD相关功能
    logging "github.com/ipfs/go-log"  // 日志包，用于日志相关功能
    iface "github.com/ipfs/kubo/core/coreiface"  // IPFS核心接口包，用于IPFS核心接口相关功能
    mh "github.com/multiformats/go-multihash"  // 多哈希包，用于多哈希相关功能
)

var flog = logging.Logger("cmds/files")  // 定义日志记录器

// FilesCmd is the 'ipfs files' command
var FilesCmd = &cmds.Command{  // 定义FilesCmd命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Interact with unixfs files.",  // 简介
        ShortDescription: `  // 简短描述
Files is an API for manipulating IPFS objects as if they were a Unix
filesystem.

The files facility interacts with MFS (Mutable File System). MFS acts as a
single, dynamic filesystem mount. MFS has a root CID that is transparently
updated when a change happens (and can be checked with "ipfs files stat /").

All files and folders within MFS are respected and will not be deleted
during garbage collections. However, a DAG may be referenced in MFS without
being fully available locally (MFS content is lazy loaded when accessed).
MFS is independent from the list of pinned items ("ipfs pin ls"). Calls to
"ipfs pin add" and "ipfs pin rm" will add and remove pins independently of
MFS. If MFS content that was additionally pinned is removed by calling
"ipfs files rm", it will still remain pinned.

Content added with "ipfs add" (which by default also becomes pinned), is not
added to MFS. Any content can be lazily referenced from MFS with the command
// IPFS命令行工具，用于将IPFS对象复制到本地文件系统中的指定路径
"ipfs files cp /ipfs/<cid> /some/path/" (see ipfs files cp --help).

// 注意事项：
// 'ipfs files'的大多数子命令都接受'--flush'标志。它默认为true。在将此标志设置为false时要小心。它会提高大量文件操作的性能，但以一致性保证为代价。
// 如果在对相关文件运行'ipfs files flush'之前意外终止了守护进程，则可能会丢失数据。这也适用于同时运行'ipfs repo gc'和'--flush=false'操作。

// IPFS文件系统命令的选项
Options: []cmds.Option{
    // 刷新目标和祖先节点的写入后，默认为true
    cmds.BoolOption(filesFlushOptionName, "f", "Flush target and ancestors after write.").WithDefault(true),
},

// IPFS文件系统命令的子命令
Subcommands: map[string]*cmds.Command{
    "read":  filesReadCmd,
    "write": filesWriteCmd,
    "mv":    filesMvCmd,
    "cp":    filesCpCmd,
    "ls":    filesLsCmd,
    "mkdir": filesMkdirCmd,
    "stat":  filesStatCmd,
    "rm":    filesRmCmd,
    "flush": filesFlushCmd,
    "chcid": filesChcidCmd,
},

// IPFS文件系统命令的常量
const (
    filesCidVersionOptionName = "cid-version"
    filesHashOptionName       = "hash"
)

// IPFS文件系统命令的选项
var (
    cidVersionOption = cmds.IntOption(filesCidVersionOptionName, "cid-ver", "Cid version to use. (experimental)")
    hashOption       = cmds.StringOption(filesHashOptionName, "Hash function to use. Will set Cid version to 1 if used. (experimental)")
)

// IPFS文件系统命令的错误类型
var errFormat = errors.New("format was set by multiple options. Only one format option is allowed")

// IPFS文件系统命令的输出结构
type statOutput struct {
    Hash           string
    Size           uint64
    CumulativeSize uint64
    Blocks         int
    Type           string
    WithLocality   bool   `json:",omitempty"`
    Local          bool   `json:",omitempty"`
    SizeLocal      uint64 `json:",omitempty"`
}

// IPFS文件系统命令的常量
const (
    defaultStatFormat = `<hash>
Size: <size>
CumulativeSize: <cumulsize>
ChildBlocks: <childs>
Type: <type>`
    filesFormatOptionName    = "format"
    filesSizeOptionName      = "size"
    # 定义一个变量，用于存储文件名的本地选项名称
    filesWithLocalOptionName = "with-local"
# 定义名为 filesStatCmd 的命令对象，用于显示文件状态
var filesStatCmd = &cmds.Command{
    # 命令的帮助文本，简要介绍显示文件状态的功能
    Helptext: cmds.HelpText{
        Tagline: "Display file status.",
    },

    # 命令的参数，包含一个必需的字符串参数，表示要查询状态的节点路径
    Arguments: []cmds.Argument{
        cmds.StringArg("path", true, false, "Path to node to stat."),
    },
    # 命令的选项，包含多个选项，用于控制输出格式和计算方式
    Options: []cmds.Option{
        # 指定输出统计信息的格式，默认为默认格式
        cmds.StringOption(filesFormatOptionName, "Print statistics in given format. Allowed tokens: "+
            "<hash> <size> <cumulsize> <type> <childs>. Conflicts with other format options.").WithDefault(defaultStatFormat),
        # 仅打印哈希值，等同于指定 '--format=<hash>' 选项
        cmds.BoolOption(filesHashOptionName, "Print only hash. Implies '--format=<hash>'. Conflicts with other format options."),
        # 仅打印文件大小，等同于指定 '--format=<cumulsize>' 选项
        cmds.BoolOption(filesSizeOptionName, "Print only size. Implies '--format=<cumulsize>'. Conflicts with other format options."),
        # 计算本地 DAG 的大小和总大小（如果可能的话）
        cmds.BoolOption(filesWithLocalOptionName, "Compute the amount of the dag that is local, and if possible the total size"),
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取格式选项的状态
        _, err := statGetFormatOptions(req)
        if err != nil {
            # 如果出现错误，返回客户端错误
            return cmds.Errorf(cmds.ErrClient, err.Error())
        }

        # 获取节点信息
        node, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 获取 API 接口
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 检查路径
        path, err := checkPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 获取本地选项
        withLocal, _ := req.Options[filesWithLocalOptionName].(bool)

        # 获取 CID 编码器
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        # 定义 DAGService 变量
        var dagserv ipld.DAGService
        if withLocal {
            # 如果使用本地选项，创建离线 DAGService 不会从网络获取数据
            dagserv = dag.NewDAGService(bservice.New(
                node.Blockstore,
                offline.Exchange(node.Blockstore),
            ))
        } else {
            dagserv = node.DAG
        }

        # 从路径获取节点
        nd, err := getNodeFromPath(req.Context, node, api, path)
        if err != nil {
            return err
        }

        # 获取节点状态
        o, err := statNode(nd, enc)
        if err != nil {
            return err
        }

        # 如果不使用本地选项，只返回一次结果
        if !withLocal {
            return cmds.EmitOnce(res, o)
        }

        # 遍历块并获取本地信息
        local, sizeLocal, err := walkBlock(req.Context, dagserv, nd)
        if err != nil {
            return err
        }

        # 设置本地性信息
        o.WithLocality = true
        o.Local = local
        o.SizeLocal = sizeLocal

        # 返回一次结果
        return cmds.EmitOnce(res, o)
    },
    # 创建一个编码器映射，将文本命令映射到一个特定的编码器函数
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器函数
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *statOutput) error {
            # 获取格式选项并替换其中的占位符
            s, _ := statGetFormatOptions(req)
            s = strings.Replace(s, "<hash>", out.Hash, -1)
            s = strings.Replace(s, "<size>", fmt.Sprintf("%d", out.Size), -1)
            s = strings.Replace(s, "<cumulsize>", fmt.Sprintf("%d", out.CumulativeSize), -1)
            s = strings.Replace(s, "<childs>", fmt.Sprintf("%d", out.Blocks), -1)
            s = strings.Replace(s, "<type>", out.Type, -1)

            # 将格式化后的字符串写入到输出流中
            fmt.Fprintln(w, s)

            # 如果包含局部性信息，则将局部性信息写入到输出流中
            if out.WithLocality {
                fmt.Fprintf(w, "Local: %s of %s (%.2f%%)\n",
                    humanize.Bytes(out.SizeLocal),
                    humanize.Bytes(out.CumulativeSize),
                    100.0*float64(out.SizeLocal)/float64(out.CumulativeSize),
                )
            }

            # 返回空值表示编码器函数执行成功
            return nil
        }),
    },
    # 初始化一个名为 Type 的 statOutput 结构体
    Type: statOutput{},
// 判断是否有多个选项为 true
func moreThanOne(a, b, c bool) bool {
    return a && b || b && c || a && c
}

// 获取格式选项
func statGetFormatOptions(req *cmds.Request) (string, error) {
    // 获取哈希选项
    hash, _ := req.Options[filesHashOptionName].(bool)
    // 获取大小选项
    size, _ := req.Options[filesSizeOptionName].(bool)
    // 获取格式选项
    format, _ := req.Options[filesFormatOptionName].(string)

    // 如果有多个选项为 true 或者格式选项不等于默认格式，则返回错误
    if moreThanOne(hash, size, format != defaultStatFormat) {
        return "", errFormat
    }

    // 如果哈希选项为 true，则返回 "<hash>"
    if hash {
        return "<hash>", nil
    // 如果大小选项为 true，则返回 "<cumulsize>"
    } else if size {
        return "<cumulsize>", nil
    // 否则返回格式选项
    } else {
        return format, nil
    }
}

// 获取节点状态
func statNode(nd ipld.Node, enc cidenc.Encoder) (*statOutput, error) {
    // 获取节点的 CID
    c := nd.Cid()

    // 获取节点的累积大小
    cumulsize, err := nd.Size()
    if err != nil {
        return nil, err
    }

    // 根据节点类型进行处理
    switch n := nd.(type) {
    case *dag.ProtoNode:
        // 从节点数据中获取文件系统节点
        d, err := ft.FSNodeFromBytes(n.Data())
        if err != nil {
            return nil, err
        }

        var ndtype string
        // 根据文件系统节点类型设置节点类型
        switch d.Type() {
        case ft.TDirectory, ft.THAMTShard:
            ndtype = "directory"
        case ft.TFile, ft.TMetadata, ft.TRaw:
            ndtype = "file"
        default:
            return nil, fmt.Errorf("unrecognized node type: %s", d.Type())
        }

        // 返回节点状态输出
        return &statOutput{
            Hash:           enc.Encode(c),
            Blocks:         len(nd.Links()),
            Size:           d.FileSize(),
            CumulativeSize: cumulsize,
            Type:           ndtype,
        }, nil
    case *dag.RawNode:
        // 返回原始节点状态输出
        return &statOutput{
            Hash:           enc.Encode(c),
            Blocks:         0,
            Size:           cumulsize,
            CumulativeSize: cumulsize,
            Type:           "file",
        }, nil
    default:
        // 返回错误，表示不是 UnixFS 节点（proto 或 raw）
        return nil, fmt.Errorf("not unixfs node (proto or raw)")
    }
}

// 遍历块
func walkBlock(ctx context.Context, dagserv ipld.DAGService, nd ipld.Node) (bool, uint64, error) {
    // 从节点原始数据中获取大小
    sizeLocal := uint64(len(nd.RawData()))

    local := true
    # 遍历节点的链接列表
    for _, link := range nd.Links() {
        # 获取链接对应的子节点
        child, err := dagserv.Get(ctx, link.Cid)

        # 如果子节点未找到，则将 local 置为 false，并继续下一个链接
        if ipld.IsNotFound(err) {
            local = false
            continue
        }

        # 如果出现其他错误，则返回错误
        if err != nil {
            return local, sizeLocal, err
        }

        # 递归地遍历子节点，并获取子节点的本地状态和大小
        childLocal, childLocalSize, err := walkBlock(ctx, dagserv, child)
        if err != nil {
            return local, sizeLocal, err
        }

        # 递归地将子节点的本地状态合并到父节点的本地状态中
        local = local && childLocal
        # 累加子节点的大小到父节点的大小中
        sizeLocal += childLocalSize
    }

    # 返回父节点的本地状态和大小，没有错误
    return local, sizeLocal, nil
# 定义名为 filesCpCmd 的命令对象
var filesCpCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Add references to IPFS files and directories in MFS (or copy within MFS).",
        ShortDescription: `
"ipfs files cp" can be used to add references to any IPFS file or directory
(usually in the form /ipfs/<CID>, but also any resolvable path) into MFS.
This performs a lazy copy: the full DAG will not be fetched, only the root
node being copied.

It can also be used to copy files within MFS, but in the case when an
IPFS-path matches an existing MFS path, the IPFS path wins.

In order to add content to MFS from disk, you can use "ipfs add" to obtain the
IPFS Content Identifier and then "ipfs files cp" to copy it into MFS:

$ ipfs add --quieter --pin=false <your file>
# ...
# ... outputs the root CID at the end
$ ipfs files cp /ipfs/<CID> /your/desired/mfs/path

If you wish to fully copy content from a different IPFS peer into MFS, do not
forget to force IPFS to fetch to full DAG after doing the "cp" operation. i.e:

$ ipfs files cp /ipfs/<CID> /your/desired/mfs/path
$ ipfs pin add <CID>

The lazy-copy feature can also be used to protect partial DAG contents from
garbage collection. i.e. adding the Wikipedia root to MFS would not download
all the Wikipedia, but will prevent any downloaded Wikipedia-DAG content from
being GC'ed.
`,
    },
    # 参数列表，包括源路径和目标路径
    Arguments: []cmds.Argument{
        cmds.StringArg("source", true, false, "Source IPFS or MFS path to copy."),
        cmds.StringArg("dest", true, false, "Destination within MFS."),
    },
    # 选项列表，包括创建父目录的选项
    Options: []cmds.Option{
        cmds.BoolOption(filesParentsOptionName, "p", "Make parent directories as needed."),
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求选项中获取是否创建父目录的标志
        mkParents, _ := req.Options[filesParentsOptionName].(bool)
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 获取新的路径前缀
        prefix, err := getPrefixNew(req)
        if err != nil {
            return err
        }

        # 从环境中获取 API
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求选项中获取是否刷新的标志
        flush, _ := req.Options[filesFlushOptionName].(bool)

        # 检查并处理源路径
        src, err := checkPath(req.Arguments[0])
        if err != nil {
            return err
        }
        src = strings.TrimRight(src, "/")

        # 检查并处理目标路径
        dst, err := checkPath(req.Arguments[1])
        if err != nil {
            return err
        }

        # 如果目标路径以斜杠结尾，则添加源路径的基本名称
        if dst[len(dst)-1] == '/' {
            dst += gopath.Base(src)
        }

        # 从路径获取节点信息
        node, err := getNodeFromPath(req.Context, nd, api, src)
        if err != nil {
            return fmt.Errorf("cp: cannot get node from path %s: %s", src, err)
        }

        # 如果需要创建父目录，则确保包含目录存在
        if mkParents {
            err := ensureContainingDirectoryExists(nd.FilesRoot, dst, prefix)
            if err != nil {
                return err
            }
        }

        # 将节点放入目标路径
        err = mfs.PutNode(nd.FilesRoot, dst, node)
        if err != nil {
            return fmt.Errorf("cp: cannot put node in path %s: %s", dst, err)
        }

        # 如果需要刷新，则刷新创建的文件
        if flush {
            _, err := mfs.FlushPath(req.Context, nd.FilesRoot, dst)
            if err != nil {
                return fmt.Errorf("cp: cannot flush the created file %s: %s", dst, err)
            }
        }

        # 返回空错误表示操作成功
        return nil
    },
// 从给定路径获取节点，返回节点和错误信息
func getNodeFromPath(ctx context.Context, node *core.IpfsNode, api iface.CoreAPI, p string) (ipld.Node, error) {
    // 根据路径前缀判断是 IPFS 还是 MFS 路径
    switch {
    case strings.HasPrefix(p, "/ipfs/"):
        // 如果是 IPFS 路径，创建路径对象
        pth, err := path.NewPath(p)
        if err != nil {
            return nil, err
        }
        // 解析节点并返回
        return api.ResolveNode(ctx, pth)
    default:
        // 如果是 MFS 路径，查找节点并返回
        fsn, err := mfs.Lookup(node.FilesRoot, p)
        if err != nil {
            return nil, err
        }
        return fsn.GetNode()
    }
}

// 定义文件列表输出结构
type filesLsOutput struct {
    Entries []mfs.NodeListing
}

// 定义长选项和不排序选项的名称
const (
    longOptionName     = "long"
    dontSortOptionName = "U"
)

// 定义文件列表命令
var filesLsCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List directories in the local mutable namespace.",
        ShortDescription: `
List directories in the local mutable namespace (works on both IPFS and MFS paths).

Examples:

    $ ipfs files ls /welcome/docs/
    about
    contact
    help
    quick-start
    readme
    security-notes

    $ ipfs files ls /myfiles/a/b/c/d
    foo
    bar
`,
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("path", false, false, "Path to show listing for. Defaults to '/'."),
    },
    Options: []cmds.Option{
        cmds.BoolOption(longOptionName, "l", "Use long listing format."),
        cmds.BoolOption(dontSortOptionName, "Do not sort; list entries in directory order."),
    },
    },
    # 创建一个命令编码器映射，将文本命令映射到特定的编码器函数
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *filesLsOutput) error {
            # 从请求选项中获取是否不排序的标志
            noSort, _ := req.Options[dontSortOptionName].(bool)
            # 如果不需要排序，则对输出进行排序
            if !noSort {
                sort.Slice(out.Entries, func(i, j int) bool {
                    return strings.Compare(out.Entries[i].Name, out.Entries[j].Name) < 0
                })
            }

            # 从请求选项中获取是否长格式的标志
            long, _ := req.Options[longOptionName].(bool)
            # 遍历输出条目并根据选项输出相应格式的信息
            for _, o := range out.Entries {
                if long:
                    # 如果是长格式，则输出文件名、哈希和大小
                    if o.Type == int(mfs.TDir) {
                        o.Name += "/"
                    }
                    fmt.Fprintf(w, "%s\t%s\t%d\n", o.Name, o.Hash, o.Size)
                } else {
                    # 否则只输出文件名
                    fmt.Fprintf(w, "%s\n", o.Name)
                }
            }

            # 返回空错误表示编码器函数执行成功
            return nil
        }),
    },
    # 创建一个空的文件列表输出对象
    Type: filesLsOutput{},
// 定义常量，表示文件偏移和文件数量的选项名称
const (
    filesOffsetOptionName = "offset"
    filesCountOptionName  = "count"
)

// 定义一个命令对象，用于读取 MFS 中的文件
var filesReadCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Read a file from MFS.", // 命令的简短描述
        ShortDescription: `
Read a specified number of bytes from a file at a given offset. By default,
it will read the entire file similar to the Unix cat.

Examples:

    $ ipfs files read /test/hello
    hello
    `, // 命令的详细描述
    },

    // 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("path", true, false, "Path to file to be read."), // 文件路径参数
    },
    // 定义命令的选项
    Options: []cmds.Option{
        cmds.Int64Option(filesOffsetOptionName, "o", "Byte offset to begin reading from."), // 文件偏移选项
        cmds.Int64Option(filesCountOptionName, "n", "Maximum number of bytes to read."), // 文件数量选项
    },
    // Run 函数接收请求、响应和环境参数，处理文件系统操作
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 从环境参数中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        // 检查请求参数中的路径是否合法
        path, err := checkPath(req.Arguments[0])
        if err != nil {
            return err
        }

        // 在节点文件系统中查找指定路径的节点
        fsn, err := mfs.Lookup(nd.FilesRoot, path)
        if err != nil {
            return err
        }

        // 将节点转换为文件节点
        fi, ok := fsn.(*mfs.File)
        if !ok {
            return fmt.Errorf("%s was not a file", path)
        }

        // 打开文件节点，准备读取数据
        rfd, err := fi.Open(mfs.Flags{Read: true})
        if err != nil {
            return err
        }

        // 延迟关闭文件节点
        defer rfd.Close()

        // 从请求参数中获取偏移量
        offset, _ := req.Options[offsetOptionName].(int64)
        if offset < 0 {
            return fmt.Errorf("cannot specify negative offset")
        }

        // 获取文件节点的大小
        filen, err := rfd.Size()
        if err != nil {
            return err
        }

        // 检查偏移量是否超出文件大小
        if int64(offset) > filen {
            return fmt.Errorf("offset was past end of file (%d > %d)", offset, filen)
        }

        // 设置文件读取偏移量
        _, err = rfd.Seek(int64(offset), io.SeekStart)
        if err != nil {
            return err
        }

        // 创建文件读取器，并根据请求参数中的 'count' 限制读取长度
        var r io.Reader = &contextReaderWrapper{R: rfd, ctx: req.Context}
        count, found := req.Options[filesCountOptionName].(int64)
        if found {
            if count < 0 {
                return fmt.Errorf("cannot specify negative 'count'")
            }
            r = io.LimitReader(r, int64(count))
        }
        // 发送读取到的数据作为响应
        return res.Emit(r)
    },
// 定义一个接口类型 contextReader，包含 CtxReadFull 方法
type contextReader interface {
    CtxReadFull(context.Context, []byte) (int, error)
}

// 定义一个结构体类型 contextReaderWrapper，包含 R 和 ctx 两个字段
type contextReaderWrapper struct {
    R   contextReader
    ctx context.Context
}

// 为 contextReaderWrapper 结构体定义 Read 方法
func (crw *contextReaderWrapper) Read(b []byte) (int, error) {
    return crw.R.CtxReadFull(crw.ctx, b)
}

// 定义一个命令对象 filesMvCmd
var filesMvCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Move files.",
        ShortDescription: `
Move files around. Just like the traditional Unix mv.

Example:

    $ ipfs files mv /myfs/a/b/c /myfs/foo/newc

`,
    },
    // 参数列表
    Arguments: []cmds.Argument{
        cmds.StringArg("source", true, false, "Source file to move."),
        cmds.StringArg("dest", true, false, "Destination path for file to be moved to."),
    },
    // 运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        flush, _ := req.Options[filesFlushOptionName].(bool)

        src, err := checkPath(req.Arguments[0])
        if err != nil {
            return err
        }
        dst, err := checkPath(req.Arguments[1])
        if err != nil {
            return err
        }

        err = mfs.Mv(nd.FilesRoot, src, dst)
        if err == nil && flush {
            _, err = mfs.FlushPath(req.Context, nd.FilesRoot, "/")
        }
        return err
    },
}

// 定义一些常量
const (
    filesCreateOptionName    = "create"
    filesParentsOptionName   = "parents"
    filesTruncateOptionName  = "truncate"
    filesRawLeavesOptionName = "raw-leaves"
    filesFlushOptionName     = "flush"
)

// 定义一个命令对象 filesWriteCmd
var filesWriteCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Append to (modify) a file in MFS.",
        ShortDescription: `
A low-level MFS command that allows you to append data to a file. If you want
to add a file without modifying an existing one, use 'ipfs add --to-files'
instead.
`,
        LongDescription: `
// 长描述
A low-level MFS command that allows you to append data to a file. If you want
to add a file without modifying an existing one, use 'ipfs add --to-files'
instead.
`,
# 低级 MFS 命令，允许在文件末尾追加数据，或指定文件内的起始偏移量进行写入。输入的整个长度将被写入。

# 如果指定了 '--create' 选项，如果文件不存在，则将创建文件。除非指定了 '--parents' 选项，否则不会创建不存在的中间目录。

# 新创建的文件将具有与父目录相同的 CID 版本和哈希函数，除非使用了 '--cid-version' 和 '--hash' 选项。

# 如果 CID 版本为 0，则新创建的叶子将以传统格式（Protobuf）存在，如果 CID 版本为非零，则以原始格式存在。使用 '--raw-leaves' 选项将覆盖此行为。

# 如果 '--flush' 选项设置为 false，则更改将不会传播到 merkledag 根。当对较深的目录结构进行大量写入时，这可以使操作速度更快。

# 示例：
# 通过管道将 "hello world" 写入 /myfs/a/b/file 文件，如果文件不存在则创建，并创建不存在的中间目录
# echo "hello world" | ipfs files write --create --parents /myfs/a/b/file
# 通过管道将 "hello world" 写入 /myfs/a/b/file 文件，如果文件存在则截断并写入
# echo "hello world" | ipfs files write --truncate /myfs/a/b/file

# 警告：
# 使用 '--flush=false' 选项不能保证数据持久性，直到树被刷新。可以通过在文件或其任何祖先上运行 'ipfs files stat' 来实现这一点。

# 警告：
# 'files write' 生成的 CID 与 'ipfs add' 不同，因为 'ipfs file write' 创建了一个针对追加操作进行了优化的 trickle-dag。有关更多信息，请参见 'ipfs add --help' 中的 '--trickle'。

# 如果要添加文件而不修改现有文件，请使用带有 '--to-files' 的 'ipfs add'：
# 创建目录 /myfs/dir
# ipfs files mkdir -p /myfs/dir
# 将 example.jpg 添加到 /myfs/dir/ 中
# ipfs add example.jpg --to-files /myfs/dir/
# 列出 /myfs/dir/ 中的文件
# ipfs files ls /myfs/dir/
# example.jpg

# 有关更多信息，请参见 'ipfs add --help' 中的 '--to-files'。
    # 创建一个选项列表，包含各种文件操作的选项
    Options: []cmds.Option{
        # 整数选项：指定写入的起始字节偏移量
        cmds.Int64Option(filesOffsetOptionName, "o", "Byte offset to begin writing at."),
        # 布尔选项：如果文件不存在则创建
        cmds.BoolOption(filesCreateOptionName, "e", "Create the file if it does not exist."),
        # 布尔选项：如果需要，创建父目录
        cmds.BoolOption(filesParentsOptionName, "p", "Make parent directories as needed."),
        # 布尔选项：在写入之前将文件截断为零大小
        cmds.BoolOption(filesTruncateOptionName, "t", "Truncate the file to size zero before writing."),
        # 整数选项：最大读取字节数
        cmds.Int64Option(filesCountOptionName, "n", "Maximum number of bytes to read."),
        # 布尔选项：对新创建的叶节点使用原始块（实验性功能）
        cmds.BoolOption(filesRawLeavesOptionName, "Use raw blocks for newly created leaf nodes. (experimental)"),
        # CID 版本选项
        cidVersionOption,
        # 哈希选项
        hashOption,
    },
    },
# 定义一个名为 filesMkdirCmd 的命令对象，用于创建目录
var filesMkdirCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Make directories.",  # 命令的简短描述
        ShortDescription: `
Create the directory if it does not already exist.

The directory will have the same CID version and hash function of the
parent directory unless the --cid-version and --hash options are used.

NOTE: All paths must be absolute.

Examples:

    $ ipfs files mkdir /test/newdir
    $ ipfs files mkdir -p /test/does/not/exist/yet
`,  # 命令的详细描述
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("path", true, false, "Path to dir to make."),  # 接受一个字符串参数作为目录路径
    },
    Options: []cmds.Option{
        cmds.BoolOption(filesParentsOptionName, "p", "No error if existing, make parent directories as needed."),  # 接受一个布尔选项，如果存在则不报错，需要时创建父目录
        cidVersionOption,  # CID 版本选项
        hashOption,  # 哈希选项
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        n, err := cmdenv.GetNode(env)  # 获取节点信息
        if err != nil {
            return err
        }

        dashp, _ := req.Options[filesParentsOptionName].(bool)  # 获取父目录选项的值
        dirtomake, err := checkPath(req.Arguments[0])  # 检查要创建的目录路径
        if err != nil {
            return err
        }

        flush, _ := req.Options[filesFlushOptionName].(bool)  # 获取刷新选项的值

        prefix, err := getPrefix(req)  # 获取前缀信息
        if err != nil {
            return err
        }
        root := n.FilesRoot  # 获取文件根目录

        err = mfs.Mkdir(root, dirtomake, mfs.MkdirOpts{  # 调用 Mkdir 方法创建目录
            Mkparents:  dashp,  # 是否创建父目录
            Flush:      flush,  # 是否刷新
            CidBuilder: prefix,  # CID 构建器
        })

        return err  # 返回错误信息
    },
}

# 定义一个名为 flushRes 的结构体，用于存储 CID 信息
type flushRes struct {
    Cid string  # CID 字符串
}

# 定义一个名为 filesFlushCmd 的命令对象，用于刷新给定路径的数据到磁盘
var filesFlushCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Flush a given path's data to disk.",  # 命令的简短描述
        ShortDescription: `
Flush a given path to the disk. This is only useful when other commands
are run with the '--flush=false'.
`,  # 命令的详细描述
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("path", false, false, "Path to flush. Default: '/'."),  # 接受一个字符串参数作为要刷新的路径，默认为 '/'
    },
    # 运行函数，处理请求并发送响应
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 从请求中获取 CID 编码器
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        # 设置默认路径为根目录
        path := "/"
        # 如果请求参数中有路径信息，则使用请求参数中的路径
        if len(req.Arguments) > 0 {
            path = req.Arguments[0]
        }

        # 刷新指定路径下的所有更改
        n, err := mfs.FlushPath(req.Context, nd.FilesRoot, path)
        if err != nil {
            return err
        }

        # 发送一次性响应，包含刷新后的节点的 CID 编码
        return cmds.EmitOnce(res, &flushRes{enc.Encode(n.Cid())})
    },
    # 指定类型为 flushRes 结构体
    Type: flushRes{},
// 定义一个名为 filesChcidCmd 的命令，用于更改给定路径的根节点的 CID 版本或哈希函数
var filesChcidCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Change the CID version or hash function of the root node of a given path.", // 帮助文本，简短描述更改 CID 版本或哈希函数的作用
        ShortDescription: `
Change the CID version or hash function of the root node of a given path.`, // 简短描述更改给定路径根节点的 CID 版本或哈希函数的作用
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("path", false, false, "Path to change. Default: '/'."), // 定义命令的参数，指定要更改的路径，默认为'/'
    },
    Options: []cmds.Option{
        cidVersionOption, // CID 版本选项
        hashOption, // 哈希函数选项
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 运行命令的函数
        nd, err := cmdenv.GetNode(env) // 获取节点
        if err != nil {
            return err
        }

        path := "/" // 默认路径为'/'
        if len(req.Arguments) > 0 {
            path = req.Arguments[0] // 如果参数中指定了路径，则使用指定的路径
        }

        flush, _ := req.Options[filesFlushOptionName].(bool) // 获取是否刷新的选项

        prefix, err := getPrefix(req) // 获取前缀
        if err != nil {
            return err
        }

        err = updatePath(nd.FilesRoot, path, prefix) // 更新路径
        if err == nil && flush {
            _, err = mfs.FlushPath(req.Context, nd.FilesRoot, path) // 如果需要刷新，则刷新路径
        }
        return err
    },
}

// 更新路径的函数
func updatePath(rt *mfs.Root, pth string, builder cid.Builder) error {
    if builder == nil {
        return nil
    }

    nd, err := mfs.Lookup(rt, pth) // 查找路径
    if err != nil {
        return err
    }

    switch n := nd.(type) {
    case *mfs.Directory:
        n.SetCidBuilder(builder) // 设置 CID 构建器
    default:
        return fmt.Errorf("can only update directories") // 如果不是目录，则返回错误
    }

    return nil
}

// 定义一个名为 filesRmCmd 的命令，用于从 MFS 中删除文件
var filesRmCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Remove a file from MFS.", // 帮助文本，简短描述从 MFS 中删除文件的作用
        ShortDescription: `
Remove files or directories.`, // 简短描述删除文件或目录的作用
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("path", true, true, "File to remove."), // 定义命令的参数，指定要删除的文件
    },
}
    # 定义命令行选项，包括递归删除和强制删除
    Options: []cmds.Option{
        cmds.BoolOption(recursiveOptionName, "r", "Recursively remove directories."),  # 递归删除选项
        cmds.BoolOption(forceOptionName, "Forcibly remove target at path; implies -r for directories"),  # 强制删除选项
    },
    # 定义命令行执行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        # 如果指定了'--force'，则删除任何其他内容，包括文件、目录、损坏的节点等
        force, _ := req.Options[forceOptionName].(bool)
        dashr, _ := req.Options[recursiveOptionName].(bool)
        var errs []error
        # 遍历请求参数中的路径
        for _, arg := range req.Arguments {
            # 检查路径是否有效
            path, err := checkPath(arg)
            if err != nil {
                errs = append(errs, fmt.Errorf("%s is not a valid path: %w", arg, err))
                continue
            }
            # 删除路径
            if err := removePath(nd.FilesRoot, path, force, dashr); err != nil {
                errs = append(errs, fmt.Errorf("%s: %w", path, err))
            }
        }
        # 如果存在错误，则逐个输出错误信息
        if len(errs) > 0 {
            for _, err = range errs {
                e := res.Emit(err.Error())
                if e != nil {
                    return e
                }
            }
            return fmt.Errorf("can't remove some files")
        }
        # 没有错误则返回nil
        return nil
    },
func removePath(filesRoot *mfs.Root, path string, force bool, dashr bool) error {
    // 如果要删除的路径是根目录，则返回错误
    if path == "/" {
        return fmt.Errorf("cannot delete root")
    }

    // 如果路径最后有斜杠，则去掉斜杠
    if path[len(path)-1] == '/' {
        path = path[:len(path)-1]
    }

    // 分割路径，获取目录和文件名
    dir, name := gopath.Split(path)

    // 获取父目录
    pdir, err := getParentDir(filesRoot, dir)
    if err != nil {
        // 如果强制删除并且父目录不存在，则返回空
        if force && err == os.ErrNotExist {
            return nil
        }
        return err
    }

    // 如果强制删除
    if force {
        // 删除文件
        err := pdir.Unlink(name)
        if err != nil {
            // 如果文件不存在，则返回空
            if err == os.ErrNotExist {
                return nil
            }
            return err
        }
        // 刷新父目录
        return pdir.Flush()
    }

    // 通过文件名获取子节点，当节点损坏或不存在时，会返回特定错误
    child, err := pdir.Child(name)
    if err != nil {
        return err
    }

    // 判断子节点类型
    switch child.(type) {
    case *mfs.Directory:
        // 如果不使用 -r 选项删除目录，则返回错误
        if !dashr {
            return fmt.Errorf("path is a directory, use -r to remove directories")
        }
    }

    // 删除文件
    err = pdir.Unlink(name)
    if err != nil {
        return err
    }

    // 刷新父目录
    return pdir.Flush()
}

func getPrefixNew(req *cmds.Request) (cid.Builder, error) {
    // 获取 CID 版本和哈希函数
    cidVer, cidVerSet := req.Options[filesCidVersionOptionName].(int)
    hashFunStr, hashFunSet := req.Options[filesHashOptionName].(string)

    // 如果 CID 版本和哈希函数都未设置，则返回空
    if !cidVerSet && !hashFunSet {
        return nil, nil
    }

    // 如果只设置了哈希函数，则默认使用 CID 版本 1
    if hashFunSet && cidVer == 0 {
        cidVer = 1
    }

    // 获取指定 CID 版本的前缀
    prefix, err := dag.PrefixForCidVersion(cidVer)
    if err != nil {
        return nil, err
    }

    // 如果设置了哈希函数，则更新前缀的哈希函数类型和长度
    if hashFunSet {
        hashFunCode, ok := mh.Names[strings.ToLower(hashFunStr)]
        if !ok {
            return nil, fmt.Errorf("unrecognized hash function: %s", strings.ToLower(hashFunStr))
        }
        prefix.MhType = hashFunCode
        prefix.MhLength = -1
    }

    return &prefix, nil
}

func getPrefix(req *cmds.Request) (cid.Builder, error) {
    # 从请求的选项中获取文件 CID 版本和哈希函数名称，并赋值给对应的变量
    cidVer, cidVerSet := req.Options[filesCidVersionOptionName].(int)
    hashFunStr, hashFunSet := req.Options[filesHashOptionName].(string)

    # 如果 CID 版本和哈希函数都没有设置，则返回空值
    if !cidVerSet && !hashFunSet:
        return nil, nil

    # 如果哈希函数已设置且 CID 版本为 0，则将 CID 版本设置为 1
    if hashFunSet && cidVer == 0:
        cidVer = 1

    # 根据 CID 版本获取相应的前缀信息
    prefix, err := dag.PrefixForCidVersion(cidVer)
    if err != nil:
        return nil, err

    # 如果哈希函数已设置，则获取哈希函数的代码，并设置前缀的哈希类型和长度
    if hashFunSet:
        hashFunCode, ok := mh.Names[strings.ToLower(hashFunStr)]
        if !ok:
            return nil, fmt.Errorf("unrecognized hash function: %s", strings.ToLower(hashFunStr))
        prefix.MhType = hashFunCode
        prefix.MhLength = -1

    # 返回前缀信息
    return &prefix, nil
# 确保包含指定路径的目录存在，如果不存在则创建
func ensureContainingDirectoryExists(r *mfs.Root, path string, builder cid.Builder) error:
    # 获取需要创建的目录路径
    dirtomake := gopath.Dir(path)

    # 如果需要创建的目录是根目录，则直接返回
    if dirtomake == "/":
        return nil

    # 创建目录，并指定创建父目录和使用的 CID 构建器
    return mfs.Mkdir(r, dirtomake, mfs.MkdirOpts{
        Mkparents:  true,
        CidBuilder: builder,
    })

# 获取指定路径的文件句柄，如果文件不存在是否创建，使用的 CID 构建器
func getFileHandle(r *mfs.Root, path string, create bool, builder cid.Builder) (*mfs.File, error):
    # 查找指定路径的节点
    target, err := mfs.Lookup(r, path)
    switch err:
        # 如果没有错误，则判断节点类型并返回文件句柄
        case nil:
            fi, ok := target.(*mfs.File)
            if !ok:
                return nil, fmt.Errorf("%s was not a file", path)
            return fi, nil

        # 如果文件不存在且需要创建，则创建文件
        case os.ErrNotExist:
            if !create:
                return nil, err

            # 获取父目录路径和文件名
            dirname, fname := gopath.Split(path)
            pdir, err := getParentDir(r, dirname)
            if err != nil:
                return nil, err

            # 如果没有指定 CID 构建器，则使用父目录的 CID 构建器
            if builder == nil:
                builder = pdir.GetCidBuilder()

            # 创建空节点，并设置 CID 构建器
            nd := dag.NodeWithData(ft.FilePBData(nil, 0))
            err = nd.SetCidBuilder(builder)
            if err != nil:
                return nil, err

            # 将新节点添加到父目录中
            err = pdir.AddChild(fname, nd)
            if err != nil:
                return nil, err

            # 获取新添加的节点，并返回文件句柄
            fsn, err := pdir.Child(fname)
            if err != nil:
                return nil, err
            fi, ok := fsn.(*mfs.File)
            if !ok:
                return nil, errors.New("expected *mfs.File, didn't get it. This is likely a race condition")
            return fi, nil

        # 其他错误情况，直接返回错误
        default:
            return nil, err

# 检查路径是否合法，如果合法则返回清理后的路径，否则返回错误
func checkPath(p string) (string, error):
    # 检查路径是否为空
    if len(p) == 0:
        return "", fmt.Errorf("paths must not be empty")

    # 检查路径是否以斜杠开头
    if p[0] != '/':
        return "", fmt.Errorf("paths must start with a leading slash")

    # 清理路径并添加末尾斜杠（如果有）
    cleaned := gopath.Clean(p)
    if p[len(p)-1] == '/' && p != "/":
        cleaned += "/"
    return cleaned, nil
# 获取指定目录的父目录对象
func getParentDir(root *mfs.Root, dir string) (*mfs.Directory, error) {
    # 在指定的根目录下查找指定的目录对象
    parent, err := mfs.Lookup(root, dir)
    # 如果查找出错，则返回错误
    if err != nil {
        return nil, err
    }

    # 将查找到的父目录对象转换为*mfs.Directory类型
    pdir, ok := parent.(*mfs.Directory)
    # 如果转换失败，则返回错误
    if !ok {
        return nil, errors.New("expected *mfs.Directory, didn't get it. This is likely a race condition")
    }
    # 返回转换后的父目录对象
    return pdir, nil
}
```