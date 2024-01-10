# `kubo\core\commands\object\object.go`

```
// 导入所需的包
package objectcmd

import (
    "encoding/base64" // 导入 base64 编码包
    "errors" // 导入 errors 包
    "fmt" // 导入 fmt 包
    "io" // 导入 io 包
    "text/tabwriter" // 导入 tabwriter 包

    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 go-ipfs-cmds 包
    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入 cmdenv 包
    "github.com/ipfs/kubo/core/commands/cmdutils" // 导入 cmdutils 包

    humanize "github.com/dustin/go-humanize" // 导入 go-humanize 包
    dag "github.com/ipfs/boxo/ipld/merkledag" // 导入 merkledag 包
    "github.com/ipfs/go-cid" // 导入 go-cid 包
    ipld "github.com/ipfs/go-ipld-format" // 导入 go-ipld-format 包
    "github.com/ipfs/kubo/core/coreiface/options" // 导入 options 包
)

// 定义 Node 结构体
type Node struct {
    Links []Link // Links 字段为 Link 类型的切片
    Data  string // Data 字段为字符串类型
}

// 定义 Link 结构体
type Link struct {
    Name, Hash string // Name 和 Hash 字段为字符串类型
    Size       uint64 // Size 字段为无符号整数类型
}

// 定义 Object 结构体
type Object struct {
    Hash  string `json:"Hash,omitempty"` // Hash 字段为字符串类型，使用 json 标签指定字段名
    Links []Link `json:"Links,omitempty"` // Links 字段为 Link 类型的切片，使用 json 标签指定字段名
}

// 定义错误变量 ErrDataEncoding
var ErrDataEncoding = errors.New("unknown data field encoding") // 初始化错误变量

// 定义常量
const (
    headersOptionName      = "headers" // 定义 headersOptionName 常量
    encodingOptionName     = "data-encoding" // 定义 encodingOptionName 常量
    inputencOptionName     = "inputenc" // 定义 inputencOptionName 常量
    datafieldencOptionName = "datafieldenc" // 定义 datafieldencOptionName 常量
    pinOptionName          = "pin" // 定义 pinOptionName 常量
    quietOptionName        = "quiet" // 定义 quietOptionName 常量
    humanOptionName        = "human" // 定义 humanOptionName 常量
)

// 定义 ObjectCmd 命令
var ObjectCmd = &cmds.Command{
    Status: cmds.Deprecated, // 设置命令状态为 Deprecated
    Helptext: cmds.HelpText{ // 设置命令的帮助文本
        Tagline: "Deprecated commands to interact with dag-pb objects. Use 'dag' or 'files' instead.", // 设置命令的一句话描述
        ShortDescription: `
'ipfs object' is a legacy plumbing command used to manipulate dag-pb objects
directly. Deprecated, use more modern 'ipfs dag' and 'ipfs files' instead.`, // 设置命令的简短描述
    },

    Subcommands: map[string]*cmds.Command{ // 设置子命令
        "data":  ObjectDataCmd, // 设置子命令 "data" 为 ObjectDataCmd
        "diff":  ObjectDiffCmd, // 设置子命令 "diff" 为 ObjectDiffCmd
        "get":   ObjectGetCmd, // 设置子命令 "get" 为 ObjectGetCmd
        "links": ObjectLinksCmd, // 设置子命令 "links" 为 ObjectLinksCmd
        "new":   ObjectNewCmd, // 设置子命令 "new" 为 ObjectNewCmd
        "patch": ObjectPatchCmd, // 设置子命令 "patch" 为 ObjectPatchCmd
        "put":   ObjectPutCmd, // 设置子命令 "put" 为 ObjectPutCmd
        "stat":  ObjectStatCmd, // 设置子命令 "stat" 为 ObjectStatCmd
    },
}

// 定义 ObjectDataCmd 命令
var ObjectDataCmd = &cmds.Command{
    Status: cmds.Deprecated, // 设置命令状态为 Deprecated
    # 定义帮助文本对象cmds.HelpText，包括标语和简短描述
    Helptext: cmds.HelpText{
        # 标语，指示使用该命令的方式已经过时，建议使用'dag get'命令代替
        Tagline: "Deprecated way to read the raw bytes of a dag-pb object: use 'dag get' instead.",
        # 简短描述，提供命令的简要说明
        ShortDescription: `
// 'ipfs object data'是一个废弃的plumbing命令，用于检索存储在dag-pb节点中的原始字节。它输出到标准输出，<key>是base58编码的multihash。出于遗留原因而提供。请改用'ipfs dag get'。
// 'ipfs object data' is a deprecated plumbing command for retrieving the raw bytes stored in a dag-pb node. It outputs to stdout, and <key> is a base58 encoded multihash. Provided for legacy reasons. Use 'ipfs dag get' instead.

// 注意，"--encoding"选项不影响输出，因为输出是对象的原始数据。
// Note that the "--encoding" option does not affect the output, since the output is the raw data of the object.

// ObjectDataCmd对象数据命令
var ObjectDataCmd = &cmds.Command{
    // 状态为废弃
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 帮助文本
    Helptext: cmds.HelpText{
        // 简短描述
        Tagline: "Deprecated way to retrieve the raw bytes of a dag-pb object: use 'dag get' instead.",
        // 简短描述
        ShortDescription: `
'ipfs object data' is a deprecated plumbing command for retrieving the raw
bytes stored in a dag-pb node. It outputs to stdout, and <key> is a base58
encoded multihash. Provided for legacy reasons. Use 'ipfs dag get' instead.
`,
        // 长描述
        LongDescription: `
'ipfs object data' is a deprecated plumbing command for retrieving the raw
bytes stored in a dag-pb node. It outputs to stdout, and <key> is a base58
encoded multihash. Provided for legacy reasons. Use 'ipfs dag get' instead.

Note that the "--encoding" option does not affect the output, since the output
is the raw data of the object.
`,
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, false, "Key of the object to retrieve, in base58-encoded multihash format.").EnableStdin(),
    },
    // 运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        path, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        data, err := api.Object().Data(req.Context, path)
        if err != nil {
            return err
        }

        return res.Emit(data)
    },
}

// ObjectLinksCmd对象链接命令
var ObjectLinksCmd = &cmds.Command{
    // 状态为废弃
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 帮助文本
    Helptext: cmds.HelpText{
        // 简短描述
        Tagline: "Deprecated way to output links in the specified dag-pb object: use 'dag get' instead.",
        // 简短描述
        ShortDescription: `
'ipfs object links' is a plumbing command for retrieving the links from
a dag-pb node. It outputs to stdout, and <key> is a base58 encoded
multihash. Provided for legacy reasons. Use 'ipfs dag get' instead.
`,
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, false, "Key of the dag-pb object to retrieve, in base58-encoded multihash format.").EnableStdin(),
    },
    # 定义命令行选项，包含一个布尔类型的选项，用于控制是否打印表头
    Options: []cmds.Option{
        cmds.BoolOption(headersOptionName, "v", "Print table headers (Hash, Size, Name)."),
    },
    # 定义命令的执行逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求中获取低级 CID 编码器
        enc, err := cmdenv.GetLowLevelCidEncoder(req)
        if err != nil {
            return err
        }

        # 从请求参数中获取路径或 CID 路径
        path, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 解析路径
        rp, _, err := api.ResolvePath(req.Context, path)
        if err != nil {
            return err
        }

        # 获取对象的链接
        links, err := api.Object().Links(req.Context, rp)
        if err != nil {
            return err
        }

        # 创建输出链接的数组
        outLinks := make([]Link, len(links))
        for i, link := range links {
            outLinks[i] = Link{
                Hash: enc.Encode(link.Cid),
                Name: link.Name,
                Size: link.Size,
            }
        }

        # 创建输出对象
        out := &Object{
            Hash:  enc.Encode(rp.RootCid()),
            Links: outLinks,
        }

        # 发送一次性的输出
        return cmds.EmitOnce(res, out)
    },
    # 定义编码器，用于将输出对象编码为文本格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Object) error {
            # 创建 tabwriter 对象
            tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
            # 获取是否需要打印表头的选项值
            headers, _ := req.Options[headersOptionName].(bool)
            if headers {
                # 如果需要打印表头，则在输出中添加表头信息
                fmt.Fprintln(tw, "Hash\tSize\tName")
            }
            # 遍历输出对象的链接，将链接信息格式化输出到 tabwriter 中
            for _, link := range out.Links {
                fmt.Fprintf(tw, "%s\t%v\t%s\n", link.Hash, link.Size, cmdenv.EscNonPrint(link.Name))
            }
            # 刷新 tabwriter
            tw.Flush()

            return nil
        }),
    },
    # 定义对象类型
    Type: &Object{},
// ObjectGetCmd object get command
// 定义 ObjectGetCmd 命令对象，用于获取对象
var ObjectGetCmd = &cmds.Command{
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 设置命令状态为 Deprecated，并提供相关链接
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to get and serialize the dag-pb node. Use 'dag get' instead",
        // 设置命令的简短描述和标语
        ShortDescription: `
'ipfs object get' is a plumbing command for retrieving dag-pb nodes.
It serializes the DAG node to the format specified by the "--encoding"
flag. It outputs to stdout, and <key> is a base58 encoded multihash.

DEPRECATED and provided for legacy reasons. Use 'ipfs dag get' instead.
`,
    },
    // 设置命令的帮助文本，包括简短描述和详细描述

    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, false, "Key of the dag-pb object to retrieve, in base58-encoded multihash format.").EnableStdin(),
    },
    // 设置命令的参数，包括参数名、是否必需、是否可变长度、参数描述，并启用标准输入

    Options: []cmds.Option{
        cmds.StringOption(encodingOptionName, "Encoding type of the data field, either \"text\" or \"base64\".").WithDefault("text"),
    },
    // 设置命令的选项，包括选项名、选项描述，并设置默认值
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 运行函数，处理请求并返回结果
        api, err := cmdenv.GetApi(env, req)
        // 获取环境中的 API 对象
        if err != nil {
            return err
        }

        enc, err := cmdenv.GetLowLevelCidEncoder(req)
        // 获取低级 CID 编码器
        if err != nil {
            return err
        }

        path, err := cmdutils.PathOrCidPath(req.Arguments[0])
        // 获取请求参数中的路径或 CID 路径
        if err != nil {
            return err
        }

        datafieldenc, _ := req.Options[encodingOptionName].(string)
        // 获取请求选项中的数据字段编码方式
        if err != nil {
            return err
        }

        nd, err := api.Object().Get(req.Context, path)
        // 通过 API 获取指定路径的对象
        if err != nil {
            return err
        }

        r, err := api.Object().Data(req.Context, path)
        // 通过 API 获取指定路径的对象数据
        if err != nil {
            return err
        }

        data, err := io.ReadAll(r)
        // 读取对象数据
        if err != nil {
            return err
        }

        out, err := encodeData(data, datafieldenc)
        // 对数据进行编码处理
        if err != nil {
            return err
        }

        node := &Node{
            Links: make([]Link, len(nd.Links())),
            Data:  out,
        }

        for i, link := range nd.Links() {
            node.Links[i] = Link{
                Hash: enc.Encode(link.Cid),
                Name: link.Name,
                Size: link.Size,
            }
        }

        return cmds.EmitOnce(res, node)
        // 发送结果给响应处理器
    },
    Type: Node{},
    Encoders: cmds.EncoderMap{
        cmds.Protobuf: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Node) error {
            // 使用 Protobuf 编码器对数据进行编码
            object, err := deserializeNode(out, "text")
            // 反序列化数据字段为文本，这是标准行为
            if err != nil {
                return nil
            }

            marshaled, err := object.Marshal()
            // 对对象进行编组
            if err != nil {
                return err
            }
            _, err = w.Write(marshaled)
            // 将编组后的数据写入到输出流
            return err
        }),
    },
// ObjectStatCmd object stat command
// 定义对象状态命令

var ObjectStatCmd = &cmds.Command{
    // 设置命令状态为已弃用，提供了一个链接以供参考
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 帮助文本
    Helptext: cmds.HelpText{
        // 简短描述
        Tagline: "Deprecated way to read stats for the dag-pb node. Use 'files stat' instead.",
        // 简短描述
        ShortDescription: `
'ipfs object stat' is a plumbing command to print dag-pb node statistics.
<key> is a base58 encoded multihash.

DEPRECATED: modern replacements are 'files stat' and 'dag stat'
`,
        // 长描述
        LongDescription: `
'ipfs object stat' is a plumbing command to print dag-pb node statistics.
<key> is a base58 encoded multihash. It outputs to stdout:

    NumLinks        int number of links in link table
    BlockSize       int size of the raw, encoded data
    LinksSize       int size of the links segment
    DataSize        int size of the data segment
    CumulativeSize  int cumulative size of object and its references

DEPRECATED: Provided for legacy reasons. Modern replacements:

  For unixfs, 'ipfs files stat' can be used:

    $ ipfs files stat --with-local /ipfs/QmWfVY9y3xjsixTgbd9AorQxH7VtMpzfx2HaWtsoUYecaX
    QmWfVY9y3xjsixTgbd9AorQxH7VtMpzfx2HaWtsoUYecaX
    Size: 5
    CumulativeSize: 13
    ChildBlocks: 0
    Type: file
    Local: 13 B of 13 B (100.00%)

  Reported sizes are based on metadata present in root block, and should not be
  trusted.  A slower, but more secure alternative is 'ipfs dag stat', which
  will work for every DAG type.  It comes with a benefit of calculating the
  size by walking the DAG:

    $ ipfs dag stat /ipfs/QmWfVY9y3xjsixTgbd9AorQxH7VtMpzfx2HaWtsoUYecaX
    Size: 13, NumBlocks: 1
`,
    },
    // 参数
    Arguments: []cmds.Argument{
        // 字符串参数，必需，不支持多个值，描述为以base58编码的多哈希格式的对象键
        cmds.StringArg("key", true, false, "Key of the object to retrieve, in base58-encoded multihash format.").EnableStdin(),
    },
    // 选项
    Options: []cmds.Option{
        // 布尔选项，打印人类可读的格式的大小（例如，1K 234M 2G）
        cmds.BoolOption(humanOptionName, "Print sizes in human readable format (e.g., 1K 234M 2G)"),
    },
    # 定义一个名为 Run 的函数，接受请求、响应和环境参数，并返回一个错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象和错误信息
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求中获取低级 CID 编码器对象和错误信息
        enc, err := cmdenv.GetLowLevelCidEncoder(req)
        if err != nil {
            return err
        }

        # 从请求参数中获取路径或 CID 路径，并返回路径对象和错误信息
        p, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 通过 API 对象获取对象的统计信息，并返回统计信息对象和错误信息
        ns, err := api.Object().Stat(req.Context, p)
        if err != nil {
            return err
        }

        # 创建一个旧的节点统计信息对象，包含哈希、链接数、块大小、链接大小、数据大小和累积大小
        oldStat := &ipld.NodeStat{
            Hash:           enc.Encode(ns.Cid),
            NumLinks:       ns.NumLinks,
            BlockSize:      ns.BlockSize,
            LinksSize:      ns.LinksSize,
            DataSize:       ns.DataSize,
            CumulativeSize: ns.CumulativeSize,
        }

        # 将旧的节点统计信息对象发送给响应
        return cmds.EmitOnce(res, oldStat)
    },
    # 指定类型为 ipld.NodeStat
    Type: ipld.NodeStat{},
    # 指定编码器为文本类型，并定义编码器函数
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *ipld.NodeStat) error {
            # 创建一个制表符写入器
            wtr := tabwriter.NewWriter(w, 0, 0, 1, ' ', 0)
            # 在函数结束时刷新写入器
            defer wtr.Flush()
            # 定义一个辅助函数，用于格式化输出
            fw := func(s string, n int) {
                fmt.Fprintf(wtr, "%s:\t%d\n", s, n)
            }
            # 从请求选项中获取人类可读标志
            human, _ := req.Options[humanOptionName].(bool)
            # 输出节点统计信息的各个属性
            fw("NumLinks", out.NumLinks)
            fw("BlockSize", out.BlockSize)
            fw("LinksSize", out.LinksSize)
            fw("DataSize", out.DataSize)
            # 如果是人类可读格式，则以人类可读的方式输出累积大小，否则以普通方式输出
            if human {
                fmt.Fprintf(wtr, "%s:\t%s\n", "CumulativeSize", humanize.Bytes(uint64(out.CumulativeSize)))
            } else {
                fw("CumulativeSize", out.CumulativeSize)
            }

            return nil
        }),
    },
// ObjectPutCmd object put command
// 定义对象存储命令

var ObjectPutCmd = &cmds.Command{
    // 命令状态为已弃用，参考链接 https://github.com/ipfs/kubo/issues/7936
    Status: cmds.Deprecated,
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to store input as a DAG object. Use 'dag put' instead.",
        ShortDescription: `
'ipfs object put' is a plumbing command for storing dag-pb nodes.
It reads from stdin, and the output is a base58 encoded multihash.

DEPRECATED and provided for legacy reasons. Use 'ipfs dag put' instead.
`,
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.FileArg("data", true, false, "Data to be stored as a dag-pb object.").EnableStdin(),
    },
    // 选项
    Options: []cmds.Option{
        cmds.StringOption(inputencOptionName, "Encoding type of input data. One of: {\"protobuf\", \"json\"}.").WithDefault("json"),
        cmds.StringOption(datafieldencOptionName, "Encoding type of the data field, either \"text\" or \"base64\".").WithDefault("text"),
        cmds.BoolOption(pinOptionName, "Pin this object when adding."),
        cmds.BoolOption(quietOptionName, "q", "Write minimal output."),
    },
}
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求中获取低级 CID 编码器
        enc, err := cmdenv.GetLowLevelCidEncoder(req)
        if err != nil {
            return err
        }

        # 从请求文件中获取文件参数
        file, err := cmdenv.GetFileArg(req.Files.Entries())
        if err != nil {
            return err
        }

        # 从请求选项中获取输入编码
        inputenc, _ := req.Options[inputencOptionName].(string)
        if err != nil {
            return err
        }

        # 从请求选项中获取数据字段编码
        datafieldenc, _ := req.Options[datafieldencOptionName].(string)
        if err != nil {
            return err
        }

        # 从请求选项中获取是否进行 pin 操作
        dopin, _ := req.Options[pinOptionName].(bool)
        if err != nil {
            return err
        }

        # 使用 API 对象将文件放入 IPFS，并返回结果
        p, err := api.Object().Put(req.Context, file,
            options.Object.DataType(datafieldenc),
            options.Object.InputEnc(inputenc),
            options.Object.Pin(dopin))
        if err != nil {
            return err
        }

        # 将结果以对象形式发送给响应
        return cmds.EmitOnce(res, &Object{Hash: enc.Encode(p.RootCid())})
    },
    # 编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Object) error {
            # 从请求选项中获取是否安静模式
            quiet, _ := req.Options[quietOptionName].(bool)

            # 获取输出的哈希值，并根据是否安静模式进行处理
            o := out.Hash
            if !quiet {
                o = "added " + o
            }

            # 将结果输出到写入器中
            fmt.Fprintln(w, o)

            return nil
        }),
    },
    # 对象类型
    Type: Object{},
// ObjectNewCmd object new command
// 定义一个名为ObjectNewCmd的命令对象，用于创建新的dag-pb对象
var ObjectNewCmd = &cmds.Command{
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 设置命令状态为已弃用，并提供了相关链接
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to create a new dag-pb object from a template.",
        // 命令简介
        ShortDescription: `
'ipfs object new' is a plumbing command for creating new dag-pb nodes.
DEPRECATED and provided for legacy reasons. Use 'dag put' and 'files' instead.
`,
        // 命令详细描述
        LongDescription: `
'ipfs object new' is a plumbing command for creating new dag-pb nodes.
By default it creates and returns a new empty merkledag node, but
you may pass an optional template argument to create a preformatted
node.

Available templates:
    * unixfs-dir

DEPRECATED and provided for legacy reasons. Use 'dag put' and 'files' instead.
`,
    },
    // 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("template", false, false, "Template to use. Optional."),
    },
    // 定义命令的执行逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取API对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        // 获取低级CID编码器
        enc, err := cmdenv.GetLowLevelCidEncoder(req)
        if err != nil {
            return err
        }
        // 设置默认模板为"empty"
        template := "empty"
        if len(req.Arguments) == 1 {
            template = req.Arguments[0]
        }
        // 创建新的dag-pb对象
        nd, err := api.Object().New(req.Context, options.Object.Type(template))
        if err != nil && err != io.EOF {
            return err
        }
        // 发送命令执行结果
        return cmds.EmitOnce(res, &Object{Hash: enc.Encode(nd.Cid())})
    },
    // 定义命令的编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Object) error {
            fmt.Fprintln(w, out.Hash)
            return nil
        }),
    },
    Type: Object{},
}

// converts the Node object into a real dag.ProtoNode
// 将Node对象转换为真实的dag.ProtoNode
func deserializeNode(nd *Node, dataFieldEncoding string) (*dag.ProtoNode, error) {
    // 创建一个新的dag.ProtoNode对象
    dagnode := new(dag.ProtoNode)
    # 根据数据字段编码类型进行不同的处理
    switch dataFieldEncoding {
    # 如果是文本类型，则直接设置节点的数据
    case "text":
        dagnode.SetData([]byte(nd.Data))
    # 如果是 base64 编码类型，则解码数据并设置节点的数据
    case "base64":
        data, err := base64.StdEncoding.DecodeString(nd.Data)
        if err != nil:
            return nil, err
        dagnode.SetData(data)
    # 如果是其他编码类型，则返回错误
    default:
        return nil, ErrDataEncoding
    }

    # 创建一个与节点链接数量相同的链接数组
    links := make([]*ipld.Link, len(nd.Links))
    # 遍历节点的链接数组，解析链接哈希并创建链接对象
    for i, link := range nd.Links:
        c, err := cid.Decode(link.Hash)
        if err != nil:
            return nil, err
        links[i] = &ipld.Link{
            Name: link.Name,
            Size: link.Size,
            Cid:  c,
        }
    }
    # 设置节点的链接数组
    if err := dagnode.SetLinks(links); err != nil:
        return nil, err

    # 返回处理后的节点和空错误
    return dagnode, nil
# 定义一个函数，用于将数据编码为指定格式的字符串
func encodeData(data []byte, encoding string) (string, error) {
    # 使用 switch 语句根据编码格式进行不同的处理
    switch encoding:
        # 如果编码格式为 "text"，则直接将数据转换为字符串并返回
        case "text":
            return string(data), nil
        # 如果编码格式为 "base64"，则使用 base64 标准编码将数据转换为字符串并返回
        case "base64":
            return base64.StdEncoding.EncodeToString(data), nil
    # 如果编码格式不是 "text" 或 "base64"，则返回空字符串和错误信息 ErrDataEncoding
    return "", ErrDataEncoding
}
```