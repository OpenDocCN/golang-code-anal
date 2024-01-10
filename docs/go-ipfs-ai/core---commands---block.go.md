# `kubo\core\commands\block.go`

```
package commands

import (
    "errors"  // 导入 errors 包
    "fmt"  // 导入 fmt 包
    "io"  // 导入 io 包
    "os"  // 导入 os 包

    "github.com/ipfs/boxo/files"  // 导入 boxo/files 包

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包
    "github.com/ipfs/kubo/core/commands/cmdutils"  // 导入 cmdutils 包

    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入 coreiface/options 包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 go-ipfs-cmds 包
    mh "github.com/multiformats/go-multihash"  // 导入 go-multihash 包
)

type BlockStat struct {
    Key  string  // 定义 BlockStat 结构体的 Key 字段
    Size int  // 定义 BlockStat 结构体的 Size 字段
}

func (bs BlockStat) String() string {
    return fmt.Sprintf("Key: %s\nSize: %d\n", bs.Key, bs.Size)  // 实现 BlockStat 结构体的 String 方法
}

var BlockCmd = &cmds.Command{
    Helptext: cmds.HelpText{  // 定义 BlockCmd 命令的帮助文本
        Tagline: "Interact with raw IPFS blocks.",  // 命令的简短描述
        ShortDescription: `  // 命令的详细描述
'ipfs block' is a plumbing command used to manipulate raw IPFS blocks.
Reads from stdin or writes to stdout. A block is identified by a Multihash
passed with a valid CID.
`,
    },

    Subcommands: map[string]*cmds.Command{  // 定义 BlockCmd 命令的子命令
        "stat": blockStatCmd,  // 子命令 "stat" 对应 blockStatCmd
        "get":  blockGetCmd,  // 子命令 "get" 对应 blockGetCmd
        "put":  blockPutCmd,  // 子命令 "put" 对应 blockPutCmd
        "rm":   blockRmCmd,  // 子命令 "rm" 对应 blockRmCmd
    },
}

var blockStatCmd = &cmds.Command{
    Helptext: cmds.HelpText{  // 定义 blockStatCmd 命令的帮助文本
        Tagline: "Print information of a raw IPFS block.",  // 命令的简短描述
        ShortDescription: `  // 命令的详细描述
'ipfs block stat' is a plumbing command for retrieving information
on raw IPFS blocks. It outputs the following to stdout:

    Key  - the CID of the block
    Size - the size of the block in bytes

`,
    },

    Arguments: []cmds.Argument{  // 定义 blockStatCmd 命令的参数
        cmds.StringArg("cid", true, false, "The CID of an existing block to stat.").EnableStdin(),  // 参数 "cid"，必须提供，不支持多个值，描述为 "The CID of an existing block to stat."
    },
    # 运行函数，处理请求并发送响应
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 获取请求参数中的路径或 CID 路径
        p, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 获取块的统计信息
        b, err := api.Block().Stat(req.Context, p)
        if err != nil {
            return err
        }

        # 发送一次性的响应，包含块的统计信息
        return cmds.EmitOnce(res, &BlockStat{
            Key:  b.Path().RootCid().String(),
            Size: b.Size(),
        })
    },
    # 声明 BlockStat 类型
    Type: BlockStat{},
    # 声明编码器映射
    Encoders: cmds.EncoderMap{
        # 使用文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, bs *BlockStat) error {
            # 将块的统计信息格式化为字符串并写入到输出流
            _, err := fmt.Fprintf(w, "%s", bs)
            return err
        }),
    },
// 定义一个名为 blockGetCmd 的命令对象，用于获取原始 IPFS 块
var blockGetCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Get a raw IPFS block.", // 简短描述
        ShortDescription: `
'ipfs block get' is a plumbing command for retrieving raw IPFS blocks.
It takes a <cid>, and outputs the block to stdout.
`, // 详细描述
    },

    Arguments: []cmds.Argument{
        cmds.StringArg("cid", true, false, "The CID of an existing block to get.").EnableStdin(), // 接受一个 CID 参数
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        api, err := cmdenv.GetApi(env, req) // 获取 API 对象
        if err != nil {
            return err
        }

        p, err := cmdutils.PathOrCidPath(req.Arguments[0]) // 获取路径或 CID 路径
        if err != nil {
            return err
        }

        r, err := api.Block().Get(req.Context, p) // 获取指定块的数据
        if err != nil {
            return err
        }

        return res.Emit(r) // 返回获取的块数据
    },
}

// 定义一些常量
const (
    blockFormatOptionName   = "format" // 块格式选项名称
    blockCidCodecOptionName = "cid-codec" // CID 编解码器选项名称
    mhtypeOptionName        = "mhtype" // 摘要类型选项名称
    mhlenOptionName         = "mhlen" // 摘要长度选项名称
)

// 定义一个名为 blockPutCmd 的命令对象，用于将输入存储为 IPFS 块
var blockPutCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Store input as an IPFS block.", // 简短描述
        ShortDescription: `
'ipfs block put' is a plumbing command for storing raw IPFS blocks.
It reads data from stdin, and outputs the block's CID to stdout.

Unless cid-codec is specified, this command returns raw (0x55) CIDv1 CIDs.

Passing alternative --cid-codec does not modify imported data, nor run any
validation. It is provided solely for convenience for users who create blocks
in userland.

NOTE:
Do not use --format for any new code. It got superseded by --cid-codec and left
only for backward compatibility when a legacy CIDv0 is required (--format=v0).
`, // 详细描述
    },

    Arguments: []cmds.Argument{
        cmds.FileArg("data", true, true, "The data to be stored as an IPFS block.").EnableStdin(), // 接受一个数据文件参数
    },
    # 定义一个 Options 列表，包含多个 cmds.Option 对象
    Options: []cmds.Option{
        # 定义一个 StringOption 对象，用于指定返回的 CID 的 Multicodec，默认为 "raw"
        cmds.StringOption(blockCidCodecOptionName, "Multicodec to use in returned CID").WithDefault("raw"),
        # 定义一个 StringOption 对象，用于指定 Multihash 的哈希函数，默认为 "sha2-256"
        cmds.StringOption(mhtypeOptionName, "Multihash hash function").WithDefault("sha2-256"),
        # 定义一个 IntOption 对象，用于指定 Multihash 的哈希长度，默认为 -1
        cmds.IntOption(mhlenOptionName, "Multihash hash length").WithDefault(-1),
        # 定义一个 BoolOption 对象，用于指定是否递归固定添加的块，默认为 false
        cmds.BoolOption(pinOptionName, "Pin added blocks recursively").WithDefault(false),
        # 允许大块选项
        cmdutils.AllowBigBlockOption,
        # 定义一个 StringOption 对象，用于指定返回的 CID 的格式，默认为 "f"，已废弃
        cmds.StringOption(blockFormatOptionName, "f", "Use legacy format for returned CID (DEPRECATED)"),
    },
    // 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        // 获取请求中的多哈希类型
        mhtype, _ := req.Options[mhtypeOptionName].(string)
        // 根据多哈希类型获取哈希值
        mhtval, ok := mh.Names[mhtype]
        if !ok {
            return fmt.Errorf("unrecognized multihash function: %s", mhtype)
        }

        // 获取请求中的哈希长度
        mhlen, ok := req.Options[mhlenOptionName].(int)
        if !ok {
            return errors.New("missing option \"mhlen\"")
        }

        // 获取请求中的 CID 编解码器和格式（已弃用）
        cidCodec, _ := req.Options[blockCidCodecOptionName].(string)
        format, _ := req.Options[blockFormatOptionName].(string) // deprecated

        // 使用已弃用的 'format' 需要禁止 'cid-codec'
        if format != "" {
            if cidCodec != "" && cidCodec != "raw" {
                return fmt.Errorf("unable to use %q (deprecated) and a custom %q at the same time", blockFormatOptionName, blockCidCodecOptionName)
            }
            cidCodec = "" // makes it no-op
        }

        // 获取请求中的是否固定的选项
        pin, _ := req.Options[pinOptionName].(bool)

        // 获取请求中的文件条目并遍历
        it := req.Files.Entries()
        for it.Next() {
            // 从文件条目中获取文件对象
            file := files.FileFromEntry(it)
            if file == nil {
                return errors.New("expected a file")
            }

            // 将文件放入块存储，并设置哈希、CID 编解码器、格式和固定选项
            p, err := api.Block().Put(req.Context, file,
                options.Block.Hash(mhtval, mhlen),
                options.Block.CidCodec(cidCodec),
                options.Block.Format(format),
                options.Block.Pin(pin))
            if err != nil {
                return err
            }

            // 检查块大小是否符合要求
            if err := cmdutils.CheckBlockSize(req, uint64(p.Size())); err != nil {
                return err
            }

            // 发送块的统计信息
            err = res.Emit(&BlockStat{
                Key:  p.Path().RootCid().String(),
                Size: p.Size(),
            })
            if err != nil {
                return err
            }
        }

        // 返回遍历文件条目时的错误
        return it.Err()
    },
    # 创建一个名为Encoders的映射，将cmds.Text映射到一个函数
    Encoders: cmds.EncoderMap{
        # 将cmds.Text映射到一个函数，该函数接受请求、写入器和块状态作为参数，并返回错误
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, bs *BlockStat) error {
            # 使用格式化字符串将块状态的键写入到写入器中
            _, err := fmt.Fprintf(w, "%s\n", bs.Key)
            # 返回可能出现的错误
            return err
        }),
    },
    # 创建一个名为Type的块状态对象
    Type: BlockStat{},
// 定义常量 forceOptionName 为 "force"，blockQuietOptionName 为 "quiet"
const (
    forceOptionName      = "force"
    blockQuietOptionName = "quiet"
)

// 定义结构体 removedBlock，包含 Hash 和 Error 两个字段，使用 json 标签omitempty表示在序列化时如果字段为空则忽略
type removedBlock struct {
    Hash  string `json:",omitempty"`
    Error string `json:",omitempty"`
}

// 定义命令 blockRmCmd
var blockRmCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Remove IPFS block(s) from the local datastore.", // 简短描述
        ShortDescription: `
'ipfs block rm' is a plumbing command for removing raw ipfs blocks.
It takes a list of CIDs to remove from the local datastore..
`, // 详细描述
    },
    // 参数列表，包含一个名为 cid 的字符串参数，必须提供，可以是多个
    Arguments: []cmds.Argument{
        cmds.StringArg("cid", true, true, "CIDs of block(s) to remove."),
    },
    // 选项列表，包含两个选项，forceOptionName 和 blockQuietOptionName
    Options: []cmds.Option{
        cmds.BoolOption(forceOptionName, "f", "Ignore nonexistent blocks."), // force 选项，使用 -f 缩写，忽略不存在的块
        cmds.BoolOption(blockQuietOptionName, "q", "Write minimal output."), // quiet 选项，使用 -q 缩写，输出最小化信息
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求选项中获取 force 和 quiet 参数
        force, _ := req.Options[forceOptionName].(bool)
        quiet, _ := req.Options[blockQuietOptionName].(bool)

        # 循环处理请求参数中的每个块
        # TODO: 在完成后使用批处理 coreapi
        for _, b := range req.Arguments {
            # 将参数转换为路径或 CID 路径
            p, err := cmdutils.PathOrCidPath(b)
            if err != nil {
                return err
            }

            # 解析路径
            rp, _, err := api.ResolvePath(req.Context, p)
            if err != nil {
                return err
            }

            # 删除块
            err = api.Block().Rm(req.Context, rp, options.Block.Force(force))
            if err != nil {
                # 如果发生错误，则返回已删除的块的哈希和错误信息
                if err := res.Emit(&removedBlock{
                    Hash:  rp.RootCid().String(),
                    Error: err.Error(),
                }); err != nil {
                    return err
                }
                continue
            }

            # 如果不是静默模式，则返回已删除的块的哈希
            if !quiet {
                err := res.Emit(&removedBlock{
                    Hash: rp.RootCid().String(),
                })
                if err != nil {
                    return err
                }
            }
        }

        # 处理完毕，返回空错误
        return nil
    },
    # 定义 PostRunMap 结构体，包含 CLI 和对应的处理函数
    PostRun: cmds.PostRunMap{
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 初始化标志位，用于标记是否有操作失败
            someFailed := false
            # 循环处理每个响应
            for {
                # 获取下一个响应
                res, err := res.Next()
                # 如果到达响应流的末尾，则退出循环
                if err == io.EOF {
                    break
                } else if err != nil {
                    # 如果出现错误，则返回错误
                    return err
                }
                # 将响应转换为 removedBlock 结构体
                r := res.(*removedBlock)
                # 如果哈希为空且有错误信息，则返回错误
                if r.Hash == "" && r.Error != "" {
                    return fmt.Errorf("aborted: %s", r.Error)
                } else if r.Error != "" {
                    # 如果有错误信息，则标记操作失败，并输出错误信息到标准错误流
                    someFailed = true
                    fmt.Fprintf(os.Stderr, "cannot remove %s: %s\n", r.Hash, r.Error)
                } else {
                    # 如果没有错误信息，则输出成功信息到标准输出流
                    fmt.Fprintf(os.Stdout, "removed %s\n", r.Hash)
                }
            }
            # 如果有操作失败，则返回错误信息
            if someFailed {
                return fmt.Errorf("some blocks not removed")
            }
            # 没有操作失败，则返回空
            return nil
        },
    },
    # 定义 Type 为 removedBlock 结构体
    Type: removedBlock{},
# 闭合前面的函数定义
```