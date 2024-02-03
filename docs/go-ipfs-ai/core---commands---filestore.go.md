# `kubo\core\commands\filestore.go`

```go
package commands

import (
    "context" // 上下文包，用于处理请求的上下文信息
    "fmt" // 格式化包，用于格式化输出
    "io" // 输入输出包，用于处理输入输出流
    "os" // 操作系统包，用于操作文件和目录

    filestore "github.com/ipfs/boxo/filestore" // 引入文件存储包
    cmds "github.com/ipfs/go-ipfs-cmds" // 引入命令行包
    core "github.com/ipfs/kubo/core" // 引入核心包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv" // 引入命令行环境包
    e "github.com/ipfs/kubo/core/commands/e" // 引入错误处理包

    "github.com/ipfs/go-cid" // 引入CID包
)

var FileStoreCmd = &cmds.Command{ // 定义文件存储命令
    Helptext: cmds.HelpText{ // 帮助文本
        Tagline: "Interact with filestore objects.", // 简要说明
    },
    Subcommands: map[string]*cmds.Command{ // 子命令
        "ls":     lsFileStore, // 列出文件存储中的对象
        "verify": verifyFileStore, // 验证文件存储中的对象
        "dups":   dupsFileStore, // 查找文件存储中的重复对象
    },
}

const (
    fileOrderOptionName = "file-order" // 文件排序选项名称
)

var lsFileStore = &cmds.Command{ // 定义列出文件存储中的对象命令
    Helptext: cmds.HelpText{ // 帮助文本
        Tagline: "List objects in filestore.", // 简要说明
        LongDescription: ` // 长描述
List objects in the filestore.

If one or more <obj> is specified only list those specific objects,
otherwise list all objects.

The output is:

<hash> <size> <path> <offset>
`, // 输出格式说明
    },
    Arguments: []cmds.Argument{ // 参数
        cmds.StringArg("obj", false, true, "Cid of objects to list."), // 对象的CID列表
    },
    Options: []cmds.Option{ // 选项
        cmds.BoolOption(fileOrderOptionName, "sort the results based on the path of the backing file"), // 根据后备文件的路径对结果进行排序
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 运行函数
        _, fs, err := getFilestore(env) // 获取文件存储
        if err != nil {
            return err
        }
        args := req.Arguments // 获取参数
        if len(args) > 0 { // 如果参数长度大于0
            return listByArgs(req.Context, res, fs, args) // 列出指定对象
        }

        fileOrder, _ := req.Options[fileOrderOptionName].(bool) // 获取文件排序选项
        next, err := filestore.ListAll(req.Context, fs, fileOrder) // 列出所有对象
        if err != nil {
            return err
        }

        for { // 循环
            r := next(req.Context) // 获取下一个对象
            if r == nil { // 如果对象为空
                break // 跳出循环
            }
            if err := res.Emit(r); err != nil { // 发送对象
                return err
            }
        }

        return nil
    },
    # 定义 PostRunMap 结构体，包含 CLI 和对应的处理函数
    PostRun: cmds.PostRunMap{
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 获取 CID 编码器
            enc, err := cmdenv.GetCidEncoder(res.Request())
            if err != nil {
                return err
            }
            # 返回流式结果
            return streamResult(func(v interface{}, out io.Writer) nonFatalError {
                r := v.(*filestore.ListRes)
                # 如果有错误信息，则返回非致命错误
                if r.ErrorMsg != "" {
                    return nonFatalError(r.ErrorMsg)
                }
                # 格式化输出文件信息
                fmt.Fprintf(out, "%s\n", r.FormatLong(enc.Encode))
                return ""
            })(res, re)
        },
    },
    # 定义 Type 结构体为 filestore.ListRes
    Type: filestore.ListRes{},
# 定义一个名为 verifyFileStore 的命令
var verifyFileStore = &cmds.Command{
    # 命令的帮助文本，包括简短描述和详细描述
    Helptext: cmds.HelpText{
        Tagline: "Verify objects in filestore.",
        LongDescription: `
Verify objects in the filestore.

If one or more <obj> is specified only verify those specific objects,
otherwise verify all objects.

The output is:

<status> <hash> <size> <path> <offset>

Where <status> is one of:
ok:       the block can be reconstructed
changed:  the contents of the backing file have changed
no-file:  the backing file could not be found
error:    there was some other problem reading the file
missing:  <obj> could not be found in the filestore
ERROR:    internal error, most likely due to a corrupt database

For ERROR entries the error will also be printed to stderr.
`,
    },
    # 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("obj", false, true, "Cid of objects to verify."),
    },
    # 定义命令的选项
    Options: []cmds.Option{
        cmds.BoolOption(fileOrderOptionName, "verify the objects based on the order of the backing file"),
    },
    # 定义命令的运行逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取文件存储环境
        _, fs, err := getFilestore(env)
        if err != nil {
            return err
        }
        # 获取命令的参数
        args := req.Arguments
        # 如果参数数量大于 0，则按参数列表进行操作
        if len(args) > 0 {
            return listByArgs(req.Context, res, fs, args)
        }
        # 获取文件顺序选项
        fileOrder, _ := req.Options[fileOrderOptionName].(bool)
        # 验证所有对象
        next, err := filestore.VerifyAll(req.Context, fs, fileOrder)
        if err != nil {
            return err
        }
        # 遍历验证结果并输出
        for {
            r := next(req.Context)
            if r == nil {
                break
            }
            if err := res.Emit(r); err != nil {
                return err
            }
        }
        # 返回空
        return nil
    },
    # 定义 PostRunMap 结构体，用于存储不同命令的后续运行操作
    PostRun: cmds.PostRunMap{
        # 定义 CLI 命令的后续运行操作
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 获取 CID 编码器
            enc, err := cmdenv.GetCidEncoder(res.Request())
            if err != nil {
                return err
            }

            # 循环处理响应结果
            for {
                v, err := res.Next()
                if err != nil {
                    # 处理错误情况
                    if err == io.EOF {
                        return nil
                    }
                    return err
                }

                # 将结果转换为文件列表响应类型
                list, ok := v.(*filestore.ListRes)
                if !ok {
                    return e.TypeErr(list, v)
                }

                # 处理文件列表状态为其他错误的情况
                if list.Status == filestore.StatusOtherError {
                    fmt.Fprintf(os.Stderr, "%s\n", list.ErrorMsg)
                }
                # 输出文件列表状态和编码后的文件名
                fmt.Fprintf(os.Stdout, "%s %s\n", list.Status.Format(), list.FormatLong(enc.Encode))
            }
        },
    },
    # 定义文件列表响应类型
    Type: filestore.ListRes{},
// 定义一个名为 dupsFileStore 的命令对象
var dupsFileStore = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List blocks that are both in the filestore and standard block storage.", // 设置命令的简短描述
    },
    // 定义命令的运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取文件存储对象
        _, fs, err := getFilestore(env)
        if err != nil {
            return err
        }

        // 获取 CID 编码器
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        // 获取所有键的通道
        ch, err := fs.FileManager().AllKeysChan(req.Context)
        if err != nil {
            return err
        }

        // 遍历通道中的 CID
        for cid := range ch {
            // 检查主块存储中是否存在该 CID
            have, err := fs.MainBlockstore().Has(req.Context, cid)
            if err != nil {
                return res.Emit(&RefWrapper{Err: err.Error()})
            }
            // 如果存在，则将其编码后发送到响应中
            if have {
                if err := res.Emit(&RefWrapper{Ref: enc.Encode(cid)}); err != nil {
                    return err
                }
            }
        }

        return nil
    },
    Encoders: refsEncoderMap, // 设置命令的编码器
    Type:     RefWrapper{}, // 设置命令的类型
}

// 获取文件存储对象的函数
func getFilestore(env cmds.Environment) (*core.IpfsNode, *filestore.Filestore, error) {
    n, err := cmdenv.GetNode(env)
    if err != nil {
        return nil, nil, err
    }
    fs := n.Filestore
    if fs == nil {
        return n, nil, filestore.ErrFilestoreNotEnabled
    }
    return n, fs, err
}

// 根据参数列出文件存储对象中的内容
func listByArgs(ctx context.Context, res cmds.ResponseEmitter, fs *filestore.Filestore, args []string) error {
    for _, arg := range args {
        // 解码参数为 CID
        c, err := cid.Decode(arg)
        if err != nil {
            // 如果解码失败，则发送错误消息到响应中
            ret := &filestore.ListRes{
                Status:   filestore.StatusOtherError,
                ErrorMsg: fmt.Sprintf("%s: %v", arg, err),
            }
            if err := res.Emit(ret); err != nil {
                return err
            }
            continue
        }
        // 验证 CID 并将结果发送到响应中
        r := filestore.Verify(ctx, fs, c)
        if err := res.Emit(r); err != nil {
            return err
        }
    }

    return nil
}
```