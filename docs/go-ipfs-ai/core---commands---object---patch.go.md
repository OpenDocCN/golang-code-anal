# `kubo\core\commands\object\patch.go`

```go
package objectcmd

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作

    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 go-ipfs-cmds 包，用于处理 IPFS 命令
    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入 cmdenv 包，用于处理 IPFS 命令环境
    "github.com/ipfs/kubo/core/commands/cmdutils" // 导入 cmdutils 包，用于处理 IPFS 命令工具

    "github.com/ipfs/kubo/core/coreiface/options" // 导入 options 包，用于处理 IPFS 核心接口选项
)

var ObjectPatchCmd = &cmds.Command{
    Status: cmds.Deprecated, // 设置命令状态为 Deprecated，表示已废弃
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to create a new merkledag object based on an existing one. Use MFS with 'files cp|rm' instead.", // 设置命令的简短描述
        ShortDescription: `
'ipfs object patch <root> <cmd> <args>' is a plumbing command used to
build custom dag-pb objects. It mutates objects, creating new objects as a
result. This is the Merkle-DAG version of modifying an object.

DEPRECATED and provided for legacy reasons.
For modern use cases, use MFS with 'files' commands: 'ipfs files --help'.

  $ ipfs files cp /ipfs/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn /some-dir
  $ ipfs files cp /ipfs/Qmayz4F4UzqcAMitTzU4zCSckDofvxstDuj3y7ajsLLEVs /some-dir/added-file.jpg
  $ ipfs files stat --hash /some-dir

  The above will add 'added-file.jpg' to the directory placed under /some-dir
  and the CID of updated directory is returned by 'files stat'

  'files cp' does not download the data, only the root block, which makes it
  possible to build arbitrary directory trees without fetching them in full to
  the local node.
`, // 设置命令的详细描述
    },
    Arguments: []cmds.Argument{}, // 设置命令的参数为空
    Subcommands: map[string]*cmds.Command{ // 设置命令的子命令
        "append-data": patchAppendDataCmd, // 添加 append-data 子命令
        "add-link":    patchAddLinkCmd, // 添加 add-link 子命令
        "rm-link":     patchRmLinkCmd, // 添加 rm-link 子命令
        "set-data":    patchSetDataCmd, // 添加 set-data 子命令
    },
    Options: []cmds.Option{ // 设置命令的选项
        cmdutils.AllowBigBlockOption, // 允许大块选项
    },
}

var patchAppendDataCmd = &cmds.Command{
    Status: cmds.Deprecated, // 设置命令状态为 Deprecated，表示已废弃
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to append data to the data segment of a DAG node.", // 设置命令的简短描述
        ShortDescription: `
// 在给定对象的数据段中追加数据
// 示例：$ echo "hello" | ipfs object patch $HASH append-data
// 注意：这不是向文件追加数据，而是修改dag-pb对象中的原始数据。块的最大大小为1MiB，超过限制的对象将不被网络所接受。
// 已弃用，仅出于遗留原因提供。请改用'ipfs add'或'ipfs files'。
var patchAppendDataCmd = &cmds.Command{
    Options: []cmds.Option{
        // 添加命令选项
        cmds.StringOption("format", "f", "The format that the data is in.").WithDefault(""),
    },
    Arguments: []cmds.Argument{
        // 添加命令参数
        cmds.StringArg("root", true, false, "The hash of the node to modify."),
        cmds.FileArg("data", true, false, "Data to append.").EnableStdin(),
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取API
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        // 获取根哈希
        root, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }
        // 获取文件
        file, err := cmdenv.GetFileArg(req.Files.Entries())
        if err != nil {
            return err
        }
        // 追加数据
        p, err := api.Object().AppendData(req.Context, root, file)
        if err != nil {
            return err
        }
        // 检查CID大小
        if err := cmdutils.CheckCIDSize(req, p.RootCid(), api.Dag()); err != nil {
            return err
        }
        // 发送结果
        return cmds.EmitOnce(res, &Object{Hash: p.RootCid().String()})
    },
    Type: &Object{},
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, obj *Object) error {
            _, err := fmt.Fprintln(w, obj.Hash)
            return err
        }),
    },
}

// 设置dag-pb对象的数据字段的已弃用方式
var patchSetDataCmd = &cmds.Command{
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to set the data field of dag-pb object.",
        ShortDescription: `
Set the data of an IPFS object from stdin or with the contents of a file.

Example:
    # 使用 echo 命令生成 "my data" 字符串，并通过管道传递给 ipfs object patch 命令，将数据设置到指定的哈希值对应的 IPFS 对象中
    $ echo "my data" | ipfs object patch $MYHASH set-data
// 声明一个名为 patchRmLinkCmd 的命令，用于从 dag-pb 对象中移除一个 Merkle-link
var patchRmLinkCmd = &cmds.Command{
    // 声明该命令为已废弃状态，提供给遗留系统使用。新系统应使用 'files rm' 命令
    Status: cmds.Deprecated, // https://github.com/ipfs/kubo/issues/7936
    // 帮助文本，包括一句简短的描述和一个简短的说明
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to remove a link from dag-pb object.", // 从 dag-pb 对象中移除链接的已废弃方法
        ShortDescription: `
Remove a Merkle-link from the given object and return the hash of the result.

DEPRECATED and provided for legacy reasons. Use 'files rm' instead.
`, // 从给定对象中移除 Merkle-link 并返回结果的哈希。已废弃，提供给遗留系统使用。新系统应使用 'files rm' 命令
    },
    // 声明命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("root", true, false, "The hash of the node to modify."), // 要修改的节点的哈希值
        cmds.StringArg("name", true, false, "Name of the link to remove."), // 要移除的链接的名称
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 获取指定路径的根节点或 CID 路径
        root, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 获取请求中的第二个参数作为名称
        name := req.Arguments[1]
        # 调用 API 对象的 RmLink 方法删除指定节点的链接
        p, err := api.Object().RmLink(req.Context, root, name)
        if err != nil {
            return err
        }

        # 检查 CID 大小是否符合要求
        if err := cmdutils.CheckCIDSize(req, p.RootCid(), api.Dag()); err != nil {
            return err
        }

        # 发送一次性的结果
        return cmds.EmitOnce(res, &Object{Hash: p.RootCid().String()})
    },
    # 指定类型为 Object
    Type: Object{},
    # 设置编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Object) error {
            # 将结果哈希值写入输出流
            fmt.Fprintln(w, out.Hash)
            return nil
        }),
    },
}
// 定义常量 createOptionName，值为 "create"
const (
    createOptionName = "create"
)

// 定义 patchAddLinkCmd 命令
var patchAddLinkCmd = &cmds.Command{
    // 状态为 Deprecated，参考链接 https://github.com/ipfs/kubo/issues/7936
    Status: cmds.Deprecated,
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Deprecated way to add a link to a given dag-pb.",
        ShortDescription: `
Add a Merkle-link to the given object and return the hash of the result.

DEPRECATED and provided for legacy reasons.

Use MFS and 'files' commands instead:

  $ ipfs files cp /ipfs/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn /some-dir
  $ ipfs files cp /ipfs/Qmayz4F4UzqcAMitTzU4zCSckDofvxstDuj3y7ajsLLEVs /some-dir/added-file.jpg
  $ ipfs files stat --hash /some-dir

  The above will add 'added-file.jpg' to the directory placed under /some-dir
  and the CID of updated directory is returned by 'files stat'

  'files cp' does not download the data, only the root block, which makes it
  possible to build arbitrary directory trees without fetching them in full to
  the local node.
`,
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.StringArg("root", true, false, "The hash of the node to modify."),
        cmds.StringArg("name", true, false, "Name of link to create."),
        cmds.StringArg("ref", true, false, "IPFS object to add link to."),
    },
    // 选项
    Options: []cmds.Option{
        cmds.BoolOption(createOptionName, "p", "Create intermediary nodes."),
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 获取或解析路径或 CID 路径
        root, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 获取请求中的第二个参数作为名称
        name := req.Arguments[1]

        # 获取或解析路径或 CID 路径
        child, err := cmdutils.PathOrCidPath(req.Arguments[2])
        if err != nil {
            return err
        }

        # 获取创建选项，如果出现错误则返回错误
        create, _ := req.Options[createOptionName].(bool)
        if err != nil {
            return err
        }

        # 调用 API 对象的 AddLink 方法，将子节点链接到根节点
        p, err := api.Object().AddLink(req.Context, root, name, child,
            options.Object.Create(create))
        if err != nil {
            return err
        }

        # 检查 CID 大小是否符合要求
        if err := cmdutils.CheckCIDSize(req, p.RootCid(), api.Dag()); err != nil {
            return err
        }

        # 发送一次性的结果，包含根 CID 的哈希值
        return cmds.EmitOnce(res, &Object{Hash: p.RootCid().String()})
    },
    # 定义类型为 Object
    Type: Object{},
    # 设置编码器映射
    Encoders: cmds.EncoderMap{
        # 文本编码器，将哈希值输出到写入器中
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *Object) error {
            fmt.Fprintln(w, out.Hash)
            return nil
        }),
    },
# 闭合前面的函数定义
```