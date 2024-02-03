# `kubo\core\commands\pin\pin.go`

```go
package pin

import (
    "context" // 上下文包，用于控制函数调用的超时、取消和截止
    "encoding/json" // JSON 编解码包
    "fmt" // 格式化包，用于打印输出
    "io" // 输入输出包，提供了基本的接口和函数
    "os" // 操作系统功能包，提供了对操作系统功能的访问
    "time" // 时间包，提供了时间的显示和测量用的函数

    bserv "github.com/ipfs/boxo/blockservice" // 引入 boxo 包中的 blockservice 模块
    offline "github.com/ipfs/boxo/exchange/offline" // 引入 boxo 包中的 exchange/offline 模块
    dag "github.com/ipfs/boxo/ipld/merkledag" // 引入 boxo 包中的 ipld/merkledag 模块
    verifcid "github.com/ipfs/boxo/verifcid" // 引入 boxo 包中的 verifcid 模块
    cid "github.com/ipfs/go-cid" // 引入 go-cid 包
    cidenc "github.com/ipfs/go-cidutil/cidenc" // 引入 go-cidutil 包中的 cidenc 模块
    cmds "github.com/ipfs/go-ipfs-cmds" // 引入 go-ipfs-cmds 包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 引入 kubo 包中的 coreiface 模块
    options "github.com/ipfs/kubo/core/coreiface/options" // 引入 kubo 包中的 coreiface/options 模块

    core "github.com/ipfs/kubo/core" // 引入 kubo 包中的 core 模块
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv" // 引入 kubo 包中的 commands/cmdenv 模块
    "github.com/ipfs/kubo/core/commands/cmdutils" // 引入 kubo 包中的 commands/cmdutils 模块
    e "github.com/ipfs/kubo/core/commands/e" // 引入 kubo 包中的 commands/e 模块
)

var PinCmd = &cmds.Command{ // 定义 PinCmd 变量为 cmds.Command 类型
    Helptext: cmds.HelpText{ // 设置帮助文本
        Tagline: "Pin (and unpin) objects to local storage.", // 设置简短的描述
    },

    Subcommands: map[string]*cmds.Command{ // 设置子命令
        "add":    addPinCmd, // 添加 add 子命令
        "rm":     rmPinCmd, // 添加 rm 子命令
        "ls":     listPinCmd, // 添加 ls 子命令
        "verify": verifyPinCmd, // 添加 verify 子命令
        "update": updatePinCmd, // 添加 update 子命令
        "remote": remotePinCmd, // 添加 remote 子命令
    },
}

type PinOutput struct { // 定义 PinOutput 结构体
    Pins []string // 字符串切片类型的 Pins 字段
}

type AddPinOutput struct { // 定义 AddPinOutput 结构体
    Pins     []string `json:",omitempty"` // 字符串切片类型的 Pins 字段，使用 JSON 编码时忽略空值
    Progress int      `json:",omitempty"` // 整数类型的 Progress 字段，使用 JSON 编码时忽略空值
}

const ( // 定义常量
    pinRecursiveOptionName = "recursive" // 递归选项名称为 "recursive"
    pinProgressOptionName  = "progress" // 进度选项名称为 "progress"
)

var addPinCmd = &cmds.Command{ // 定义 addPinCmd 变量为 cmds.Command 类型
    Helptext: cmds.HelpText{ // 设置帮助文本
        Tagline:          "Pin objects to local storage.", // 设置简短的描述
        ShortDescription: "Stores an IPFS object(s) from a given path locally to disk.", // 设置短描述
        LongDescription: ` // 设置长描述
Create a pin for the given object, protecting resolved CID from being garbage
collected.

An optional name can be provided, and read back via 'ipfs pin ls --names'.

Be mindful of defaults:

Default pin type is 'recursive' (entire DAG).
Pass -r=false to create a direct pin for a single block.
Use 'pin ls -t recursive' to only list roots of recursively pinned DAGs
(significantly faster when many big DAGs are pinned recursively)
    // 默认的 pin 名称为空。使用 'pin add' 命令并传入 '--name' 来设置一个名称，并使用 'pin ls --names' 来查看它。
    // Pin add 是幂等的：对已经 pinning 的 CID 进行再次 pinning 不会改变名称，传入 '--name' 的数值会与原始 pinning 的数值保持一致。
    // 要重命名 pin，使用 'pin rm' 和 'pin add --name'。

    // 如果守护程序正在运行，任何丢失的块将从网络中检索。这可能需要一些时间。传入 '--progress' 来跟踪进度。
    `,

    Arguments: []cmds.Argument{
        cmds.StringArg("ipfs-path", true, true, "Path to object(s) to be pinned.").EnableStdin(),
    },
    Options: []cmds.Option{
        cmds.BoolOption(pinRecursiveOptionName, "r", "Recursively pin the object linked to by the specified object(s).").WithDefault(true),
        cmds.StringOption(pinNameOptionName, "n", "An optional name for created pin(s)."),
        cmds.BoolOption(pinProgressOptionName, "Show progress"),
    },
    Type: AddPinOutput{},
    },
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *AddPinOutput) error {
            rec, found := req.Options["recursive"].(bool)
            var pintype string
            if rec || !found {
                pintype = "recursively"
            } else {
                pintype = "directly"
            }

            for _, k := range out.Pins {
                fmt.Fprintf(w, "pinned %s %s\n", k, pintype)
            }

            return nil
        }),
    },
    # 定义 PostRunMap 结构体，并初始化为 cmds.PostRunMap 类型
    PostRun: cmds.PostRunMap{
        # 定义 CLI 方法，接收 res 和 re 两个参数，返回 error 类型
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 进入无限循环
            for {
                # 从 res 中获取下一个值和错误信息
                v, err := res.Next()
                # 如果有错误
                if err != nil {
                    # 如果错误是 io.EOF，表示读取结束，返回 nil
                    if err == io.EOF {
                        return nil
                    }
                    # 否则返回错误
                    return err
                }

                # 将 v 转换为 AddPinOutput 类型，如果转换失败，返回类型错误
                out, ok := v.(*AddPinOutput)
                if !ok {
                    return e.TypeErr(out, v)
                }
                # 如果 out.Pins 为 nil，表示进度选项已设置
                if out.Pins == nil {
                    # 打印进度信息到标准错误输出
                    fmt.Fprintf(os.Stderr, "Fetched/Processed %d nodes\r", out.Progress)
                } else {
                    # 否则，通过 re 发送 out，并检查是否有错误
                    err = re.Emit(out)
                    if err != nil {
                        return err
                    }
                }
            }
        },
    },
func pinAddMany(ctx context.Context, api coreiface.CoreAPI, enc cidenc.Encoder, paths []string, recursive bool, name string) ([]string, error) {
    // 创建一个空的字符串数组，用于存储添加的路径
    added := make([]string, len(paths))
    // 遍历传入的路径数组
    for i, b := range paths {
        // 将路径或者 CID 转换为路径对象
        p, err := cmdutils.PathOrCidPath(b)
        if err != nil {
            return nil, err
        }
        // 解析路径，获取最终的路径对象和错误信息
        rp, _, err := api.ResolvePath(ctx, p)
        if err != nil {
            return nil, err
        }
        // 将路径对象添加到 pinset 中
        if err := api.Pin().Add(ctx, rp, options.Pin.Recursive(recursive), options.Pin.Name(name)); err != nil {
            return nil, err
        }
        // 将添加的路径对象的根 CID 编码并存储到数组中
        added[i] = enc.Encode(rp.RootCid())
    }
    // 返回添加的路径数组和空的错误信息
    return added, nil
}

var rmPinCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Remove object from pin-list.",
        ShortDescription: `
Removes the pin from the given object allowing it to be garbage
collected if needed. (By default, recursively. Use -r=false for direct pins.)
`,
        LongDescription: `
Removes the pin from the given object allowing it to be garbage
collected if needed. (By default, recursively. Use -r=false for direct pins.)

A pin may not be removed because the specified object is not pinned or pinned
indirectly. To determine if the object is pinned indirectly, use the command:
ipfs pin ls -t indirect <cid>
`,
    },
    // 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("ipfs-path", true, true, "Path to object(s) to be unpinned.").EnableStdin(),
    },
    // 定义命令的选项
    Options: []cmds.Option{
        cmds.BoolOption(pinRecursiveOptionName, "r", "Recursively unpin the object linked to by the specified object(s).").WithDefault(true),
    },
    // 定义命令的输出类型
    Type: PinOutput{},
}
    // Run 函数处理请求并返回结果，接受请求、响应和环境作为参数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 从环境中获取 API 对象，如果出现错误则返回错误
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        // 设置递归标志
        recursive, _ := req.Options[pinRecursiveOptionName].(bool)

        // 解析请求体参数，如果出现错误则返回错误
        if err := req.ParseBodyArgs(); err != nil {
            return err
        }

        // 获取 CID 编码器，如果出现错误则返回错误
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        // 创建空的字符串切片用于存储 pin
        pins := make([]string, 0, len(req.Arguments))
        // 遍历请求参数
        for _, b := range req.Arguments {
            // 将参数转换为路径或 CID 路径，如果出现错误则返回错误
            p, err := cmdutils.PathOrCidPath(b)
            if err != nil {
                return err
            }

            // 解析路径并获取根 CID，如果出现错误则返回错误
            rp, _, err := api.ResolvePath(req.Context, p)
            if err != nil {
                return err
            }

            // 对根 CID 进行编码并添加到 pins 中
            id := enc.Encode(rp.RootCid())
            pins = append(pins, id)
            // 根据递归标志进行 pin 移除操作，如果出现错误则返回错误
            if err := api.Pin().Rm(req.Context, rp, options.Pin.RmRecursive(recursive)); err != nil {
                return err
            }
        }

        // 将 pins 发送到响应中
        return cmds.EmitOnce(res, &PinOutput{pins})
    },
    // Encoders 包含了输出格式到编码器的映射
    Encoders: cmds.EncoderMap{
        // 文本格式的编码器，用于将 PinOutput 编码为文本格式
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *PinOutput) error {
            // 遍历 pins 并将其格式化为文本输出
            for _, k := range out.Pins {
                fmt.Fprintf(w, "unpinned %s\n", k)
            }

            return nil
        }),
    },
// 定义常量，用于指定不同的选项名称
const (
    pinTypeOptionName   = "type"
    pinQuietOptionName  = "quiet"
    pinStreamOptionName = "stream"
    pinNamesOptionName  = "names"
)

// 定义命令对象 listPinCmd
var listPinCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        // 简短描述
        Tagline: "List objects pinned to local storage.",
        ShortDescription: `
Returns a list of objects that are pinned locally.
By default, all pinned objects are returned, but the '--type' flag or
arguments can restrict that to a specific pin type or to some specific objects
respectively.
`,
        // 长描述
        LongDescription: `
Returns a list of objects that are pinned locally.

By default, all pinned objects are returned, but the '--type' flag or
arguments can restrict that to a specific pin type or to some specific objects
respectively.

Use --type=<type> to specify the type of pinned keys to list.
Valid values are:
    * "direct": pin that specific object.
    * "recursive": pin that specific object, and indirectly pin all its
      descendants
    * "indirect": pinned indirectly by an ancestor (like a refcount)
    * "all"

By default, pin names are not included (returned as empty).
Pass '--names' flag to return pin names (set with '--name' from 'pin add').

With arguments, the command fails if any of the arguments is not a pinned
object. And if --type=<type> is additionally used, the command will also fail
if any of the arguments is not of the specified type.

Example:
    $ echo "hello" | ipfs add -q
    QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
    $ ipfs pin ls
    QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN recursive
    # now remove the pin, and repin it directly
    $ ipfs pin rm QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
    unpinned QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
    $ ipfs pin add -r=false QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
    pinned QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN directly
    $ ipfs pin ls --type=direct
    QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN direct
`,
    },
}
    # 使用IPFS命令行工具查询特定哈希值的文件是否被固定在IPFS网络上
    $ ipfs pin ls QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
    # 返回结果为QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN direct，表示该文件直接固定在IPFS网络上
# 定义一个结构体，用于描述 IPFS 对象列表命令
type PinLsOptions struct {
    // IPFS 对象路径参数，需要列出的对象路径
    Arguments: []cmds.Argument{
        cmds.StringArg("ipfs-path", false, true, "Path to object(s) to be listed."),
    },
    // 列出固定的键类型选项，可以是 "direct", "indirect", "recursive", 或 "all"，默认值为 "all"
    Options: []cmds.Option{
        cmds.StringOption(pinTypeOptionName, "t", "The type of pinned keys to list. Can be \"direct\", \"indirect\", \"recursive\", or \"all\".").WithDefault("all"),
        // 仅输出对象的哈希值选项
        cmds.BoolOption(pinQuietOptionName, "q", "Write just hashes of objects."),
        // 启用发现时流式传输固定的选项
        cmds.BoolOption(pinStreamOptionName, "s", "Enable streaming of pins as they are discovered."),
        // 启用显示固定名称的选项（速度较慢）
        cmds.BoolOption(pinNamesOptionName, "n", "Enable displaying pin names (slower)."),
    },
}
    # 定义一个函数 Run，接收请求、响应和环境参数，并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求参数中获取 pinTypeOptionName 对应的值，并转换为字符串类型
        typeStr, _ := req.Options[pinTypeOptionName].(string)
        # 从请求参数中获取 pinStreamOptionName 对应的值，并转换为布尔类型
        stream, _ := req.Options[pinStreamOptionName].(bool)
        # 从请求参数中获取 pinNamesOptionName 对应的值，并转换为布尔类型
        displayNames, _ := req.Options[pinNamesOptionName].(bool)

        # 根据 typeStr 的值进行不同的处理
        switch typeStr {
        case "all", "direct", "indirect", "recursive":
        default:
            # 如果 typeStr 的值不在指定范围内，则返回错误
            err = fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", typeStr)
            return err
        }

        # 定义一个 emit 函数，根据 stream 的值选择不同的处理方式
        var emit func(PinLsOutputWrapper) error
        lgcList := map[string]PinLsType{}
        if !stream {
            emit = func(v PinLsOutputWrapper) error {
                lgcList[v.PinLsObject.Cid] = PinLsType{Type: v.PinLsObject.Type, Name: v.PinLsObject.Name}
                return nil
            }
        } else {
            emit = func(v PinLsOutputWrapper) error {
                return res.Emit(v)
            }
        }

        # 根据请求参数的不同情况调用不同的函数处理
        if len(req.Arguments) > 0 {
            err = pinLsKeys(req, typeStr, api, emit)
        } else {
            err = pinLsAll(req, typeStr, displayNames, api, emit)
        }
        if err != nil {
            return err
        }

        # 如果不是流式输出，则将结果发送一次
        if !stream {
            return cmds.EmitOnce(res, PinLsOutputWrapper{
                PinLsList: PinLsList{Keys: lgcList},
            })
        }

        # 如果是流式输出，则直接返回
        return nil
    },
    # 定义类型为 PinLsOutputWrapper 的 Type 属性
    Type: PinLsOutputWrapper{},
    # 创建一个编码器映射，包含 JSON 和 Text 两种编码器
    Encoders: cmds.EncoderMap{
        # JSON 编码器
        cmds.JSON: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out PinLsOutputWrapper) error {
            # 从请求参数中获取是否为流式输出的标志
            stream, _ := req.Options[pinStreamOptionName].(bool)

            # 创建一个 JSON 编码器
            enc := json.NewEncoder(w)

            # 如果是流式输出
            if stream {
                # 将 PinLsObject 编码并写入输出流
                return enc.Encode(out.PinLsObject)
            }

            # 否则将 PinLsList 编码并写入输出流
            return enc.Encode(out.PinLsList)
        }),
        # Text 编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out PinLsOutputWrapper) error {
            # 从请求参数中获取是否为安静模式的标志
            quiet, _ := req.Options[pinQuietOptionName].(bool)
            # 从请求参数中获取是否为流式输出的标志
            stream, _ := req.Options[pinStreamOptionName].(bool)

            # 如果是流式输出
            if stream {
                # 如果是安静模式，只输出对象的 CID
                if quiet {
                    fmt.Fprintf(w, "%s\n", out.PinLsObject.Cid)
                } 
                # 如果对象没有名称，输出 CID 和类型
                else if out.PinLsObject.Name == "" {
                    fmt.Fprintf(w, "%s %s\n", out.PinLsObject.Cid, out.PinLsObject.Type)
                } 
                # 否则输出 CID、类型和名称
                else {
                    fmt.Fprintf(w, "%s %s %s\n", out.PinLsObject.Cid, out.PinLsObject.Type, out.PinLsObject.Name)
                }
                return nil
            }

            # 遍历 PinLsList 中的键值对
            for k, v := range out.PinLsList.Keys {
                # 如果是安静模式，只输出键
                if quiet {
                    fmt.Fprintf(w, "%s\n", k)
                } 
                # 如果值没有名称，输出键和类型
                else if v.Name == "" {
                    fmt.Fprintf(w, "%s %s\n", k, v.Type)
                } 
                # 否则输出键、类型和名称
                else {
                    fmt.Fprintf(w, "%s %s %s\n", k, v.Type, v.Name)
                }
            }

            return nil
        }),
    },
// PinLsOutputWrapper是pin ls命令的输出类型。
// Pin ls需要根据是否流式传输来输出两种不同类型的数据。
// 我们使用这个结构来绕过cmds库拒绝使用interface{}的限制
type PinLsOutputWrapper struct {
    PinLsList
    PinLsObject
}

// PinLsList是一组带有它们类型的pin
type PinLsList struct {
    Keys map[string]PinLsType `json:",omitempty"`
}

// PinLsType包含pin的类型
type PinLsType struct {
    Type string
    Name string
}

// PinLsObject包含pin的描述
type PinLsObject struct {
    Cid  string `json:",omitempty"`
    Name string `json:",omitempty"`
    Type string `json:",omitempty"`
}

// pinLsKeys函数用于处理pin ls命令的请求，根据类型字符串和coreAPI来输出结果
func pinLsKeys(req *cmds.Request, typeStr string, api coreiface.CoreAPI, emit func(value PinLsOutputWrapper) error) error {
    // 获取CID编码器
    enc, err := cmdenv.GetCidEncoder(req)
    if err != nil {
        return err
    }

    // 根据类型字符串进行不同的处理
    switch typeStr {
    case "all", "direct", "indirect", "recursive":
    default:
        return fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", typeStr)
    }

    // 根据pin类型字符串创建选项
    opt, err := options.Pin.IsPinned.Type(typeStr)
    if err != nil {
        panic("unhandled pin type")
    }
}
    # 遍历请求参数中的路径或 CID 路径
    for _, p := range req.Arguments {
        # 将路径或 CID 路径解析成标准格式的路径
        p, err := cmdutils.PathOrCidPath(p)
        if err != nil {
            return err
        }

        # 解析路径，获取标准格式的路径和可能的错误
        rp, _, err := api.ResolvePath(req.Context, p)
        if err != nil {
            return err
        }

        # 检查路径是否被固定，获取固定类型、固定状态和可能的错误
        pinType, pinned, err := api.Pin().IsPinned(req.Context, rp, opt)
        if err != nil {
            return err
        }

        # 如果路径未被固定，则返回错误信息
        if !pinned {
            return fmt.Errorf("path '%s' is not pinned", p)
        }

        # 根据固定类型进行不同的处理
        switch pinType {
        case "direct", "indirect", "recursive", "internal":
        default:
            pinType = "indirect through " + pinType
        }

        # 发送固定信息到输出流
        err = emit(PinLsOutputWrapper{
            PinLsObject: PinLsObject{
                Type: pinType,
                Cid:  enc.Encode(rp.RootCid()),
            },
        })
        if err != nil {
            return err
        }
    }

    # 返回空值表示处理成功
    return nil
// pinLsAll 函数用于列出所有的 pin 信息
func pinLsAll(req *cmds.Request, typeStr string, detailed bool, api coreiface.CoreAPI, emit func(value PinLsOutputWrapper) error) error {
    // 获取 CID 编码器
    enc, err := cmdenv.GetCidEncoder(req)
    if err != nil {
        return err
    }

    // 根据 typeStr 的值进行不同的操作
    switch typeStr {
    case "all", "direct", "indirect", "recursive":
    default:
        // 如果 typeStr 不在指定范围内，返回错误
        err = fmt.Errorf("invalid type '%s', must be one of {direct, indirect, recursive, all}", typeStr)
        return err
    }

    // 获取 pin 类型的选项
    opt, err := options.Pin.Ls.Type(typeStr)
    if err != nil {
        // 如果获取失败，触发 panic
        panic("unhandled pin type")
    }

    // 获取 pin 列表
    pins, err := api.Pin().Ls(req.Context, opt, options.Pin.Ls.Detailed(detailed))
    if err != nil {
        return err
    }

    // 遍历 pin 列表
    for p := range pins {
        if err := p.Err(); err != nil {
            return err
        }
        // 发送 pin 信息
        err = emit(PinLsOutputWrapper{
            PinLsObject: PinLsObject{
                Type: p.Type(),
                Name: p.Name(),
                Cid:  enc.Encode(p.Path().RootCid()),
            },
        })
        if err != nil {
            return err
        }
    }

    return nil
}

// 定义 pinUnpinOptionName 常量
const (
    pinUnpinOptionName = "unpin"
)

// 定义 updatePinCmd 命令
var updatePinCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Update a recursive pin.",
        ShortDescription: `
Efficiently pins a new object based on differences from an existing one and,
by default, removes the old pin.

This command is useful when the new pin contains many similarities or is a
derivative of an existing one, particularly for large objects. This allows a more
efficient DAG-traversal which fully skips already-pinned branches from the old
object. As a requirement, the old object needs to be an existing recursive
pin.
`,
    },

    // 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("from-path", true, false, "Path to old object."),
        cmds.StringArg("to-path", true, false, "Path to a new object to be pinned."),
    },
    # 定义命令行选项，包括一个布尔类型的选项，用于指示是否移除旧的固定引用
    Options: []cmds.Option{
        cmds.BoolOption(pinUnpinOptionName, "Remove the old pin.").WithDefault(true),
    },
    # 定义命令的输出类型为 PinOutput 结构体
    Type: PinOutput{},
    # 定义命令的执行逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求中获取 CID 编码器
        enc, err := cmdenv.GetCidEncoder(req)
        if err != nil {
            return err
        }

        # 从选项中获取 pinUnpinOptionName 对应的布尔值
        unpin, _ := req.Options[pinUnpinOptionName].(bool)

        # 解析第一个参数为路径或 CID 路径
        fromPath, err := cmdutils.PathOrCidPath(req.Arguments[0])
        if err != nil {
            return err
        }

        # 解析第二个参数为路径或 CID 路径
        toPath, err := cmdutils.PathOrCidPath(req.Arguments[1])
        if err != nil {
            return err
        }

        # 提前解析路径，以便返回实际的 CID
        from, _, err := api.ResolvePath(req.Context, fromPath)
        if err != nil {
            return err
        }
        to, _, err := api.ResolvePath(req.Context, toPath)
        if err != nil {
            return err
        }

        # 调用 API 对象的 Pin 方法更新固定引用
        err = api.Pin().Update(req.Context, from, to, options.Pin.Unpin(unpin))
        if err != nil {
            return err
        }

        # 发送一次性的响应，包含更新后的固定引用的 CID
        return cmds.EmitOnce(res, &PinOutput{Pins: []string{enc.Encode(from.RootCid()), enc.Encode(to.RootCid())}})
    },
    # 定义编码器，将输出编码为文本格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *PinOutput) error {
            fmt.Fprintf(w, "updated %s to %s\n", out.Pins[0], out.Pins[1])
            return nil
        }),
    },
// 定义常量 pinVerboseOptionName，值为 "verbose"
const (
    pinVerboseOptionName = "verbose"
)

// 定义 verifyPinCmd 命令
var verifyPinCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Verify that recursive pins are complete.", // 命令的简短描述
    },
    Options: []cmds.Option{ // 定义命令的选项
        cmds.BoolOption(pinVerboseOptionName, "Also write the hashes of non-broken pins."), // 布尔类型选项，用于输出非损坏 pin 的哈希值
        cmds.BoolOption(pinQuietOptionName, "q", "Write just hashes of broken pins."), // 布尔类型选项，用于只输出损坏 pin 的哈希值
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 定义命令的执行函数
        n, err := cmdenv.GetNode(env) // 获取节点信息
        if err != nil {
            return err
        }

        verbose, _ := req.Options[pinVerboseOptionName].(bool) // 获取 verbose 选项的值
        quiet, _ := req.Options[pinQuietOptionName].(bool) // 获取 quiet 选项的值

        if verbose && quiet { // 如果同时使用了 --verbose 和 --quiet 选项，则返回错误
            return fmt.Errorf("the --verbose and --quiet options can not be used at the same time")
        }

        enc, err := cmdenv.GetCidEncoder(req) // 获取 CID 编码器
        if err != nil {
            return err
        }

        opts := pinVerifyOpts{ // 定义 pinVerifyOpts 结构体
            explain:   !quiet, // 根据 quiet 的值设置 explain 字段
            includeOk: verbose, // 根据 verbose 的值设置 includeOk 字段
        }
        out, err := pinVerify(req.Context, n, opts, enc) // 执行 pinVerify 函数
        if err != nil {
            return err
        }
        return res.Emit(out) // 输出结果
    },
    Type: PinVerifyRes{}, // 定义命令的返回类型为 PinVerifyRes 结构体
    Encoders: cmds.EncoderMap{ // 定义命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *PinVerifyRes) error { // 文本编码器
            quiet, _ := req.Options[pinQuietOptionName].(bool) // 获取 quiet 选项的值

            if quiet && !out.Ok { // 如果使用了 quiet 选项且结果不是 OK，则输出结果的 CID
                fmt.Fprintf(w, "%s\n", out.Cid)
            } else if !quiet { // 如果没有使用 quiet 选项，则输出结果的格式化信息
                out.Format(w)
            }

            return nil
        }),
    },
}

// PinVerifyRes 是 "pin verify" 命令中每个 pin 检查的返回结果
type PinVerifyRes struct {
    Cid string `json:",omitempty"` // CID 字段，可以为空
    Err string `json:",omitempty"` // 错误信息字段，可以为空
    PinStatus // PinStatus 结构体
}

// PinStatus 是 PinVerifyRes 的一部分，不要直接使用
type PinStatus struct {
    Ok       bool      `json:",omitempty"` // OK 字段，可以为空
    BadNodes []BadNode `json:",omitempty"` // BadNodes 字段，可以为空
}
// BadNode 结构体用于在 PinVerifyRes 中使用
type BadNode struct {
    Cid string
    Err string
}

// pinVerifyOpts 结构体定义了一些选项参数
type pinVerifyOpts struct {
    explain   bool
    includeOk bool
}

// FIXME: 这个实现与 core/coreapi.PinAPI.Verify 中的实现重复，移除这个实现并且完全依赖于 CoreAPI
func pinVerify(ctx context.Context, n *core.IpfsNode, opts pinVerifyOpts, enc cidenc.Encoder) (<-chan any, error) {
    // visited 用于记录已经访问过的 CID 的状态
    visited := make(map[cid.Cid]PinStatus)

    // 获取块存储
    bs := n.Blocks.Blockstore()
    // 创建 DAG 服务
    DAG := dag.NewDAGService(bserv.New(bs, offline.Exchange(bs)))
    // 获取包含 DAG 服务的链接
    getLinks := dag.GetLinksWithDAG(DAG)

    // 定义检查 Pin 的函数
    var checkPin func(root cid.Cid) PinStatus
    checkPin = func(root cid.Cid) PinStatus {
        key := root
        // 如果已经访问过该 CID，则直接返回其状态
        if status, ok := visited[key]; ok {
            return status
        }

        // 验证 CID 是否有效
        if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, root); err != nil {
            status := PinStatus{Ok: false}
            if opts.explain {
                status.BadNodes = []BadNode{{Cid: enc.Encode(key), Err: err.Error()}}
            }
            visited[key] = status
            return status
        }

        // 获取链接
        links, err := getLinks(ctx, root)
        if err != nil {
            status := PinStatus{Ok: false}
            if opts.explain {
                status.BadNodes = []BadNode{{Cid: enc.Encode(key), Err: err.Error()}}
            }
            visited[key] = status
            return status
        }

        // 检查链接中的每个 CID 的状态
        status := PinStatus{Ok: true}
        for _, lnk := range links {
            res := checkPin(lnk.Cid)
            if !res.Ok {
                status.Ok = false
                status.BadNodes = append(status.BadNodes, res.BadNodes...)
            }
        }

        visited[key] = status
        return status
    }

    // 创建一个通道用于输出结果
    out := make(chan any)
}
    # 创建一个匿名的 goroutine 函数
    go func() {
        # 延迟关闭 out 通道，确保在函数退出时关闭通道
        defer close(out)
        # 遍历递归键的通道
        for p := range n.Pinning.RecursiveKeys(ctx, false) {
            # 如果存在错误，将错误信息发送到 out 通道并返回
            if p.Err != nil {
                out <- PinVerifyRes{Err: p.Err.Error()}
                return
            }
            # 检查 pin 状态
            pinStatus := checkPin(p.Pin.Key)
            # 如果 pin 状态不正常或者 opts.includeOk 为真
            if !pinStatus.Ok || opts.includeOk {
                # 通过 select 语句发送 PinVerifyRes 结构体到 out 通道
                select {
                case out <- PinVerifyRes{Cid: enc.Encode(p.Pin.Key), PinStatus: pinStatus}:
                # 如果上下文被取消，立即返回
                case <-ctx.Done():
                    return
                }
            }
        }
    }()
    # 返回 out 通道和空值
    return out, nil
// Format 方法用于格式化 PinVerifyRes 结构体的内容，并输出到指定的 io.Writer
func (r PinVerifyRes) Format(out io.Writer) {
    // 如果存在错误信息，则输出错误信息并返回
    if r.Err != "" {
        fmt.Fprintf(out, "error: %s\n", r.Err)
        return
    }

    // 如果验证通过，则输出 CID 和 ok，并返回
    if r.Ok {
        fmt.Fprintf(out, "%s ok\n", r.Cid)
        return
    }

    // 如果验证不通过，则输出 CID 和 broken
    fmt.Fprintf(out, "%s broken\n", r.Cid)
    // 遍历 BadNodes 切片，输出每个节点的 CID 和错误信息
    for _, e := range r.BadNodes {
        fmt.Fprintf(out, "  %s: %s\n", e.Cid, e.Err)
    }
}
```