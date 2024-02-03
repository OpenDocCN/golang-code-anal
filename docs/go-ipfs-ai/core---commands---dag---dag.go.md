# `kubo\core\commands\dag\dag.go`

```go
package dagcmd

import (
    "encoding/csv"  // 导入 encoding/csv 包，用于读写 CSV 格式的数据
    "encoding/json"  // 导入 encoding/json 包，用于读写 JSON 格式的数据
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，用于进行 I/O 操作
    "path"  // 导入 path 包，用于处理文件路径

    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入 cmdenv 包，用于命令环境相关操作
    "github.com/ipfs/kubo/core/commands/cmdutils"  // 导入 cmdutils 包，用于命令工具相关操作

    cid "github.com/ipfs/go-cid"  // 导入 go-cid 包，用于处理 CID 相关操作
    cidenc "github.com/ipfs/go-cidutil/cidenc"  // 导入 go-cidutil/cidenc 包，用于 CID 编码相关操作
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 go-ipfs-cmds 包，用于命令相关操作
)

const (
    pinRootsOptionName = "pin-roots"  // 定义 pinRootsOptionName 常量，表示 pin-roots 选项名称
    progressOptionName = "progress"  // 定义 progressOptionName 常量，表示 progress 选项名称
    silentOptionName   = "silent"  // 定义 silentOptionName 常量，表示 silent 选项名称
    statsOptionName    = "stats"  // 定义 statsOptionName 常量，表示 stats 选项名称
)

// DagCmd provides a subset of commands for interacting with ipld dag objects
var DagCmd = &cmds.Command{  // 定义 DagCmd 变量，表示与 ipld dag 对象交互的一组命令
    Helptext: cmds.HelpText{  // 设置 Helptext 属性
        Tagline: "Interact with IPLD DAG objects.",  // 设置 Tagline 属性
        ShortDescription: `  // 设置 ShortDescription 属性
'ipfs dag' is used for creating and manipulating DAG objects/hierarchies.

This subcommand is intended to deprecate and replace
the existing 'ipfs object' command moving forward.
`,
    },
    Subcommands: map[string]*cmds.Command{  // 设置 Subcommands 属性
        "put":     DagPutCmd,  // 设置 put 子命令
        "get":     DagGetCmd,  // 设置 get 子命令
        "resolve": DagResolveCmd,  // 设置 resolve 子命令
        "import":  DagImportCmd,  // 设置 import 子命令
        "export":  DagExportCmd,  // 设置 export 子命令
        "stat":    DagStatCmd,  // 设置 stat 子命令
    },
}

// OutputObject is the output type of 'dag put' command
type OutputObject struct {  // 定义 OutputObject 结构体
    Cid cid.Cid  // 设置 Cid 属性
}

// ResolveOutput is the output type of 'dag resolve' command
type ResolveOutput struct {  // 定义 ResolveOutput 结构体
    Cid     cid.Cid  // 设置 Cid 属性
    RemPath string  // 设置 RemPath 属性
}

type CarImportStats struct {  // 定义 CarImportStats 结构体
    BlockCount      uint64  // 设置 BlockCount 属性
    BlockBytesCount uint64  // 设置 BlockBytesCount 属性
}

// CarImportOutput is the output type of the 'dag import' commands
type CarImportOutput struct {  // 定义 CarImportOutput 结构体
    Root  *RootMeta       `json:",omitempty"`  // 设置 Root 属性
    Stats *CarImportStats `json:",omitempty"`  // 设置 Stats 属性
}

// RootMeta is the metadata for a root pinning response
type RootMeta struct {  // 定义 RootMeta 结构体
    Cid         cid.Cid  // 设置 Cid 属性
    PinErrorMsg string  // 设置 PinErrorMsg 属性
}

// DagPutCmd is a command for adding a dag node
var DagPutCmd = &cmds.Command{  // 定义 DagPutCmd 变量，表示添加 dag 节点的命令
    Helptext: cmds.HelpText{  // 设置 Helptext 属性
        Tagline: "Add a DAG node to IPFS.",  // 设置 Tagline 属性
        ShortDescription: `  // 设置 ShortDescription 属性
'ipfs dag put' accepts input from a file or stdin and parses it
// DagPutCmd 是一个用于将对象放入 IPFS 的命令
var DagPutCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "将对象放入 IPFS 中。",
        ShortDescription: `
'ipfs dag put' 将对象放入 IPFS，并以指定格式打印出来。
`,
    },
    Arguments: []cmds.Argument{
        cmds.FileArg("object data", true, true, "要放入的对象").EnableStdin(),
    },
    Options: []cmds.Option{
        cmds.StringOption("store-codec", "存储对象时使用的编解码器").WithDefault("dag-cbor"),
        cmds.StringOption("input-codec", "输入对象所使用的编解码器").WithDefault("dag-json"),
        cmds.BoolOption("pin", "在添加时固定此对象。"),
        cmds.StringOption("hash", "要使用的哈希函数").WithDefault("sha2-256"),
        cmdutils.AllowBigBlockOption,
    },
    Run:  dagPut,
    Type: OutputObject{},
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *OutputObject) error {
            enc, err := cmdenv.GetLowLevelCidEncoder(req)
            if err != nil {
                return err
            }
            fmt.Fprintln(w, enc.Encode(out.Cid))
            return nil
        }),
    },
}

// DagGetCmd 是一个用于从 IPFS 获取 DAG 节点的命令
var DagGetCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "从 IPFS 获取 DAG 节点。",
        ShortDescription: `
'ipfs dag get' 从 IPFS 获取 DAG 节点，并以指定格式打印出来。
`,
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("ref", true, false, "要获取的对象").EnableStdin(),
    },
    Options: []cmds.Option{
        cmds.StringOption("output-codec", "对象将被编码为的格式。").WithDefault("dag-json"),
    },
    Run: dagGet,
}

// DagResolveCmd 返回路径中最高块的地址和路径剩余部分
var DagResolveCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "解析 IPLD 块。",
        ShortDescription: `
'ipfs dag resolve' 从 IPFS 获取 DAG 节点，打印其地址和剩余路径。
`,
}
    # 定义命令行参数，包含一个必填的字符串参数，用于指定要解析的路径
    Arguments: []cmds.Argument{
        cmds.StringArg("ref", true, false, "The path to resolve").EnableStdin(),
    },
    # 指定命令的运行函数为dagResolve
    Run: dagResolve,
    # 指定编码器，将输出编码为文本格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *ResolveOutput) error {
            var (
                enc cidenc.Encoder
                err error
            )
            # 根据条件选择合适的CID编码器
            switch {
            case !cmdenv.CidBaseDefined(req):
                # 如果未指定CID编码方式，则检查路径并选择合适的编码器
                enc, err = cmdenv.CidEncoderFromPath(req.Arguments[0])
                if err == nil {
                    break
                }
                # 如果未找到合适的编码器，则使用默认的编码器
                fallthrough
            default:
                enc, err = cmdenv.GetLowLevelCidEncoder(req)
                if err != nil {
                    return err
                }
            }
            # 使用选定的编码器对CID进行编码
            p := enc.Encode(out.Cid)
            # 如果存在剩余路径，则拼接到编码后的CID路径上
            if out.RemPath != "" {
                p = path.Join(p, out.RemPath)
            }
            # 将结果输出到指定的io.Writer
            fmt.Fprint(w, p)
            return nil
        }),
    },
    # 指定输出类型为ResolveOutput结构体
    Type: ResolveOutput{},
// DagImportCmd 是一个用于将车辆导入到 IPFS 的命令
var DagImportCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "导入 .car 文件的内容",
        ShortDescription: `
'ipfs dag import' 导入所提供的 .car（内容地址存档）文件中的所有块，递归固定 CAR 文件头中指定的任何根，除非设置了 --pin-roots 为 false。

注意：
  此命令将导入 CAR 文件中的所有块，而不仅仅是从指定根可达的块。但是，这些其他块将不会被固定，可能会在以后被垃圾回收。

  固定根在处理所有 car 文件之后进行，允许导入跨多个文件的 DAG。

  固定只在离线模式下进行，一次一个根。如果从导入的 CAR 文件中的块和块存储中当前存在的内容不代表完整的 DAG，那么固定该单个根将失败。

最大支持的 CAR 版本：2
CAR 格式的规范：https://ipld.io/specs/transport/car/
`,
    },
    Arguments: []cmds.Argument{
        cmds.FileArg("path", true, true, "要导入的 .car 文件的路径。").EnableStdin(),
    },
    Options: []cmds.Option{
        cmds.BoolOption(pinRootsOptionName, "导入后固定在 .car 头中列出的可选根。").WithDefault(true),
        cmds.BoolOption(silentOptionName, "无输出。"),
        cmds.BoolOption(statsOptionName, "输出统计信息。"),
        cmdutils.AllowBigBlockOption,
    },
    Type: CarImportOutput{},
    Run:  dagImport,
    # 创建一个编码器映射，将文本命令映射到特定的编码器函数
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, event *CarImportOutput) error {
            # 从请求参数中获取静默选项的布尔值
            silent, _ := req.Options[silentOptionName].(bool)
            # 如果静默选项为真，则返回空
            if silent {
                return nil
            }

            # 事件应该只有`Root`或`Stats`中的一个被设置，而不是两者都被设置
            if event.Root == nil {
                if event.Stats == nil {
                    return fmt.Errorf("unexpected message from DAG import")
                }
                # 从请求参数中获取统计选项的布尔值
                stats, _ := req.Options[statsOptionName].(bool)
                # 如果统计选项为真，则在写入器中格式化输出导入的块数量和字节数
                if stats {
                    fmt.Fprintf(w, "Imported %d blocks (%d bytes)\n", event.Stats.BlockCount, event.Stats.BlockBytesCount)
                }
                return nil
            }

            # 如果事件的统计信息不为空，则返回意外的DAG导入消息
            if event.Stats != nil {
                return fmt.Errorf("unexpected message from DAG import")
            }

            # 获取低级CID编码器
            enc, err := cmdenv.GetLowLevelCidEncoder(req)
            if err != nil {
                return err
            }

            # 如果事件的根节点固定错误消息不为空，则返回固定根节点失败的消息
            if event.Root.PinErrorMsg != "" {
                return fmt.Errorf("pinning root %q FAILED: %s", enc.Encode(event.Root.Cid), event.Root.PinErrorMsg)
            }

            # 设置事件的根节点固定错误消息为成功
            event.Root.PinErrorMsg = "success"

            # 在写入器中格式化输出固定的根节点CID和固定错误消息
            _, err = fmt.Fprintf(
                w,
                "Pinned root\t%s\t%s\n",
                enc.Encode(event.Root.Cid),
                event.Root.PinErrorMsg,
            )
            return err
        }),
    },
// DagExportCmd 是一个用于将 ipfs dag 导出为 car 文件的命令
var DagExportCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Streams the selected DAG as a .car stream on stdout.", // 在标准输出流上将所选的 DAG 作为 .car 流输出
        ShortDescription: `
'ipfs dag export' fetches a DAG and streams it out as a well-formed .car file.
Note that at present only single root selections / .car files are supported.
The output of blocks happens in strict DAG-traversal, first-seen, order.
CAR file follows the CARv1 format: https://ipld.io/specs/transport/car/carv1/
`, // 'ipfs dag export' 获取 DAG 并将其作为格式良好的 .car 文件流输出。请注意，目前仅支持单个根选择 / .car 文件。块的输出按照严格的 DAG 遍历，先看到的顺序进行。CAR 文件遵循 CARv1 格式：https://ipld.io/specs/transport/car/carv1/
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("root", true, false, "CID of a root to recursively export").EnableStdin(), // 递归导出的根的 CID
    },
    Options: []cmds.Option{
        cmds.BoolOption(progressOptionName, "p", "Display progress on CLI. Defaults to true when STDERR is a TTY."), // 在 CLI 上显示进度。当 STDERR 是 TTY 时，默认为 true。
    },
    Run: dagExport, // 运行 dagExport 函数
    PostRun: cmds.PostRunMap{
        cmds.CLI: finishCLIExport, // 在 CLI 上完成导出
    },
}

// DagStat 是一个 dag stat 命令的响应
type DagStat struct {
    Cid       cid.Cid `json:",omitempty"` // CID
    Size      uint64  `json:",omitempty"` // 大小
    NumBlocks int64   `json:",omitempty"` // 块数量
}

func (s *DagStat) String() string {
    return fmt.Sprintf("%s  %d  %d", s.Cid.String()[:20], s.Size, s.NumBlocks) // 返回格式化的字符串
}

func (s *DagStat) MarshalJSON() ([]byte, error) {
    type Alias DagStat
    /*
        We can't rely on cid.Cid.MarshalJSON since it uses the {"/": "..."}
        format. To make the output consistent and follow the Kubo API patterns
        we use the Cid.String method
    */ // 无法依赖 cid.Cid.MarshalJSON，因为它使用 {"/": "..."} 格式。为了使输出一致并遵循 Kubo API 模式，我们使用 Cid.String 方法
    return json.Marshal(struct {
        Cid string `json:"Cid"` // CID
        *Alias
    }{
        Cid:   s.Cid.String(), // CID 字符串
        Alias: (*Alias)(s), // 别名
    })
}

func (s *DagStat) UnmarshalJSON(data []byte) error {
    /*
        We can't rely on cid.Cid.UnmarshalJSON since it uses the {"/": "..."}
        format. To make the output consistent and follow the Kubo API patterns
        we use the Cid.Parse method
    */ // 无法依赖 cid.Cid.UnmarshalJSON，因为它使用 {"/": "..."} 格式。为了使输出一致并遵循 Kubo API 模式，我们使用 Cid.Parse 方法
    type Alias DagStat
    # 创建一个匿名结构体，包含字段Cid和一个指向Alias结构体的指针
    aux := struct {
        Cid string `json:"Cid"`
        *Alias
    }{
        Alias: (*Alias)(s),  # 将传入的s结构体转换为Alias类型，并赋值给匿名结构体的指针字段
    }
    # 将JSON数据解析到匿名结构体中
    if err := json.Unmarshal(data, &aux); err != nil {
        return err
    }
    # 解析Cid字段的值为CID对象
    Cid, err := cid.Parse(aux.Cid)
    if err != nil {
        return err
    }
    # 将解析后的CID对象赋值给s结构体的Cid字段
    s.Cid = Cid
    # 返回空错误，表示操作成功
    return nil
// 定义结构体 DagStatSummary，包含了 DAG 统计信息的字段
type DagStatSummary struct {
    redundantSize uint64     `json:"-"`
    UniqueBlocks  int        `json:",omitempty"`
    TotalSize     uint64     `json:",omitempty"`
    SharedSize    uint64     `json:",omitempty"`
    Ratio         float32    `json:",omitempty"`
    DagStatsArray []*DagStat `json:"DagStats,omitempty"`
}

// 实现结构体 DagStatSummary 的 String 方法，返回格式化的统计信息字符串
func (s *DagStatSummary) String() string {
    return fmt.Sprintf("Total Size: %d\nUnique Blocks: %d\nShared Size: %d\nRatio: %f", s.TotalSize, s.UniqueBlocks, s.SharedSize, s.Ratio)
}

// 实现结构体 DagStatSummary 的 incrementTotalSize 方法，增加总大小统计
func (s *DagStatSummary) incrementTotalSize(size uint64) {
    s.TotalSize += size
}

// 实现结构体 DagStatSummary 的 incrementRedundantSize 方法，增加冗余大小统计
func (s *DagStatSummary) incrementRedundantSize(size uint64) {
    s.redundantSize += size
}

// 实现结构体 DagStatSummary 的 appendStats 方法，添加 DAG 统计信息
func (s *DagStatSummary) appendStats(stats *DagStat) {
    s.DagStatsArray = append(s.DagStatsArray, stats)
}

// 实现结构体 DagStatSummary 的 calculateSummary 方法，计算统计信息中的比率和共享大小
func (s *DagStatSummary) calculateSummary() {
    s.Ratio = float32(s.redundantSize) / float32(s.TotalSize)
    s.SharedSize = s.redundantSize - s.TotalSize
}

// 定义 DagStatCmd 命令，用于获取存储在 ipfs 中的 DAG 的大小信息
var DagStatCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Gets stats for a DAG.",
        ShortDescription: `
'ipfs dag stat' fetches a DAG and returns various statistics about it.
Statistics include size and number of blocks.

Note: This command skips duplicate blocks in reporting both size and the number of blocks
`,
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("root", true, true, "CID of a DAG root to get statistics for").EnableStdin(),
    },
    Options: []cmds.Option{
        cmds.BoolOption(progressOptionName, "p", "Return progressive data while reading through the DAG").WithDefault(true),
    },
    Run:  dagStat, // 运行 dagStat 函数
    Type: DagStatSummary{}, // 使用 DagStatSummary 结构体作为返回类型
    PostRun: cmds.PostRunMap{
        cmds.CLI: finishCLIStat, // 在 CLI 模式下运行 finishCLIStat 函数
    },
}
    # 创建一个编码器映射，将命令类型与对应的编码器函数关联起来
    Encoders: cmds.EncoderMap{
        # 将文本类型的请求与特定的编码器函数关联起来
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, event *DagStatSummary) error {
            # 在写入器中打印换行符
            fmt.Fprintln(w)
            # 创建一个 CSV 写入器
            csvWriter := csv.NewWriter(w)
            # 设置 CSV 写入器的分隔符为制表符
            csvWriter.Comma = '\t'
            # 计算 CID 字段的空格数，用于格式化输出
            cidSpacing := len(event.DagStatsArray[0].Cid.String())
            # 创建 CSV 文件的标题行
            header := []string{fmt.Sprintf("%-*s", cidSpacing, "CID"), fmt.Sprintf("%-15s", "Blocks"), "Size"}
            # 写入 CSV 文件的标题行
            if err := csvWriter.Write(header); err != nil {
                return err
            }
            # 遍历事件的 DAG 统计数组
            for _, dagStat := range event.DagStatsArray {
                # 将块数转换为字符串
                numBlocksStr := fmt.Sprint(dagStat.NumBlocks)
                # 写入 CSV 文件的数据行
                err := csvWriter.Write([]string{
                    dagStat.Cid.String(),
                    fmt.Sprintf("%-15s", numBlocksStr),
                    fmt.Sprint(dagStat.Size),
                })
                # 检查写入过程中是否出现错误
                if err != nil {
                    return err
                }
            }
            # 刷新 CSV 写入器
            csvWriter.Flush()
            # 在写入器中打印换行符和 "Summary" 字符串
            fmt.Fprint(w, "\nSummary\n")
            # 格式化输出事件的摘要信息
            _, err := fmt.Fprintf(
                w,
                "%v\n",
                event,
            )
            # 在写入器中打印两个换行符
            fmt.Fprint(w, "\n\n")
            # 返回可能出现的错误
            return err
        }),
        # 将 JSON 类型的请求与特定的编码器函数关联起来
        cmds.JSON: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, event *DagStatSummary) error {
            # 使用 JSON 编码器将事件编码并写入到写入器中
            return json.NewEncoder(w).Encode(event)
        },
        ),
    },
# 闭合前面的函数定义
```