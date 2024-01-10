# `kubo\core\commands\active.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于输入输出操作
    "sort"  // 导入 sort 包，用于排序
    "text/tabwriter"  // 导入 text/tabwriter 包，用于格式化输出
    "time"  // 导入 time 包，用于时间操作

    oldcmds "github.com/ipfs/kubo/commands"  // 导入旧的命令包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 IPFS 命令包
)

const (
    verboseOptionName = "verbose"  // 定义 verbose 选项的名称
)

var ActiveReqsCmd = &cmds.Command{  // 定义 ActiveReqsCmd 命令
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline: "List commands run on this IPFS node.",  // 简短的描述
        ShortDescription: `  // 详细的描述
Lists running and recently run commands.
`,
    },
    NoLocal: true,  // 不允许本地执行
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 定义命令的执行函数
        ctx := env.(*oldcmds.Context)  // 获取环境中的上下文
        return cmds.EmitOnce(res, ctx.ReqLog.Report())  // 发送一次响应，包含上下文中请求日志的报告
    },
    Options: []cmds.Option{  // 定义命令的选项
        cmds.BoolOption(verboseOptionName, "v", "Print extra information."),  // 定义 verbose 选项，用于打印额外信息
    },
    Subcommands: map[string]*cmds.Command{  // 定义命令的子命令
        "clear":    clearInactiveCmd,  // 清除不活跃的命令
        "set-time": setRequestClearCmd,  // 设置请求清除命令
    },
    # 定义 Encoders 字段，包含不同类型命令的编码器
    Encoders: cmds.EncoderMap{
        # 文本类型命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *[]*cmds.ReqLogEntry) error {
            # 从请求的选项中获取 verboseOptionName 对应的值，转换为布尔类型
            verbose, _ := req.Options[verboseOptionName].(bool)

            # 创建一个 tabwriter，用于格式化输出
            tw := tabwriter.NewWriter(w, 4, 4, 2, ' ', 0)
            # 如果 verbose 为真，则输出 ID
            if verbose {
                fmt.Fprint(tw, "ID\t")
            }
            # 输出 Command
            fmt.Fprint(tw, "Command\t")
            # 如果 verbose 为真，则输出 Arguments 和 Options
            if verbose {
                fmt.Fprint(tw, "Arguments\tOptions\t")
            }
            # 输出 Active、StartTime 和 RunTime
            fmt.Fprintln(tw, "Active\tStartTime\tRunTime")

            # 遍历输出的请求日志条目
            for _, req := range *out {
                # 如果 verbose 为真，则输出 ID
                if verbose {
                    fmt.Fprintf(tw, "%d\t", req.ID)
                }
                # 输出 Command
                fmt.Fprintf(tw, "%s\t", req.Command)
                # 如果 verbose 为真，则输出 Arguments 和 Options
                if verbose {
                    fmt.Fprintf(tw, "%v\t[", req.Args)
                    # 获取 Options 的键，并按字母顺序排序
                    var keys []string
                    for k := range req.Options {
                        keys = append(keys, k)
                    }
                    sort.Strings(keys)

                    # 输出 Options 的键值对
                    for _, k := range keys {
                        fmt.Fprintf(tw, "%s=%v,", k, req.Options[k])
                    }
                    fmt.Fprintf(tw, "]\t")
                }

                # 计算请求的活跃时间
                var live time.Duration
                if req.Active {
                    live = time.Since(req.StartTime)
                } else {
                    live = req.EndTime.Sub(req.StartTime)
                }
                # 格式化输出 Active、StartTime 和 RunTime
                t := req.StartTime.Format(time.Stamp)
                fmt.Fprintf(tw, "%t\t%s\t%s\n", req.Active, t, live)
            }
            # 刷新 tabwriter
            tw.Flush()

            return nil
        }),
    },
    # 定义 Type 字段，存储请求日志条目的类型
    Type: []*cmds.ReqLogEntry{},
// 定义清除日志中不活跃请求的命令
var clearInactiveCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Clear inactive requests from the log.",
    },
    // 执行清除不活跃请求的操作
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取环境上下文
        ctx := env.(*oldcmds.Context)
        // 清除不活跃请求
        ctx.ReqLog.ClearInactive()
        return nil
    },
}

// 定义设置日志中保留不活跃请求时间的命令
var setRequestClearCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Set how long to keep inactive requests in the log.",
    },
    // 设置命令参数
    Arguments: []cmds.Argument{
        cmds.StringArg("time", true, false, "Time to keep inactive requests in log."),
    },
    // 执行设置保留时间的操作
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 解析时间参数
        tval, err := time.ParseDuration(req.Arguments[0])
        if err != nil {
            return err
        }
        // 获取环境上下文
        ctx := env.(*oldcmds.Context)
        // 设置保留不活跃请求的时间
        ctx.ReqLog.SetKeepTime(tval)

        return nil
    },
}
```