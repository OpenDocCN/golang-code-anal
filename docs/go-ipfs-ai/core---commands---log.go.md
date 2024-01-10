# `kubo\core\commands\log.go`

```
package commands

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作

    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 go-ipfs-cmds 包，用于处理 IPFS 命令
    logging "github.com/ipfs/go-log" // 导入 go-log 包，用于日志记录
    lwriter "github.com/ipfs/go-log/writer" // 导入 go-log/writer 包，用于日志写入
)

// Golang os.Args overrides * and replaces the character argument with
// an array which includes every file in the user's CWD. As a
// workaround, we use 'all' instead. The util library still uses * so
// we convert it at this step.
var logAllKeyword = "all" // 定义变量 logAllKeyword 为字符串 "all"

var LogCmd = &cmds.Command{ // 定义命令 LogCmd
    Helptext: cmds.HelpText{ // 命令的帮助文本
        Tagline: "Interact with the daemon log output.", // 简短描述
        ShortDescription: `
'ipfs log' contains utility commands to affect or read the logging
output of a running daemon.

There are also two environmental variables that direct the logging 
system (not just for the daemon logs, but all commands):
    IPFS_LOGGING - sets the level of verbosity of the logging.
        One of: debug, info, warn, error, dpanic, panic, fatal
    IPFS_LOGGING_FMT - sets formatting of the log output.
        One of: color, nocolor
`, // 详细描述
    },

    Subcommands: map[string]*cmds.Command{ // 子命令
        "level": logLevelCmd, // 设置子命令 "level" 对应的处理函数为 logLevelCmd
        "ls":    logLsCmd, // 设置子命令 "ls" 对应的处理函数为 logLsCmd
        "tail":  logTailCmd, // 设置子命令 "tail" 对应的处理函数为 logTailCmd
    },
}

var logLevelCmd = &cmds.Command{ // 定义命令 logLevelCmd
    Helptext: cmds.HelpText{ // 命令的帮助文本
        Tagline: "Change the logging level.", // 简短描述
        ShortDescription: `
Change the verbosity of one or all subsystems log output. This does not affect
the event log.
`, // 详细描述
    },

    Arguments: []cmds.Argument{ // 命令的参数
        // TODO use a different keyword for 'all' because all can theoretically
        // clash with a subsystem name
        cmds.StringArg("subsystem", true, false, fmt.Sprintf("The subsystem logging identifier. Use '%s' for all subsystems.", logAllKeyword)), // 定义字符串类型的参数 "subsystem"，并提供说明
        cmds.StringArg("level", true, false, `The log level, with 'debug' the most verbose and 'fatal' the least verbose.
            One of: debug, info, warn, error, dpanic, panic, fatal.
        `), // 定义字符串类型的参数 "level"，并提供说明
    },
    NoLocal: true, // 设置 NoLocal 属性为 true
    # Run 方法，处理请求并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取请求参数
        args := req.Arguments
        subsystem, level := args[0], args[1]

        # 如果子系统为logAllKeyword，则将其替换为"*"
        if subsystem == logAllKeyword {
            subsystem = "*"
        }

        # 设置子系统的日志级别
        if err := logging.SetLogLevel(subsystem, level); err != nil {
            return err
        }

        # 格式化日志级别变更的消息
        s := fmt.Sprintf("Changed log level of '%s' to '%s'\n", subsystem, level)
        # 记录日志
        log.Info(s)

        # 发送一次性消息
        return cmds.EmitOnce(res, &MessageOutput{s})
    },
    # 编码器映射
    Encoders: cmds.EncoderMap{
        # 文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *MessageOutput) error {
            # 将消息输出到写入器
            fmt.Fprint(w, out.Message)
            return nil
        }),
    },
    # 消息输出类型
    Type: MessageOutput{},
# 定义一个名为logLsCmd的命令对象
var logLsCmd = &cmds.Command{
    # 帮助文本，包括简短描述和标语
    Helptext: cmds.HelpText{
        Tagline: "List the logging subsystems.",
        ShortDescription: `
'ipfs log ls' is a utility command used to list the logging
subsystems of a running daemon.
`,
    },
    # 运行命令的函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 通过cmds.EmitOnce将日志子系统列表发送到响应中
        return cmds.EmitOnce(res, &stringList{logging.GetSubsystems()})
    },
    # 编码器映射，将文本编码器与特定类型的编码器关联
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, list *stringList) error {
            # 遍历字符串列表并将其写入到io.Writer中
            for _, s := range list.Strings {
                fmt.Fprintln(w, s)
            }
            return nil
        }),
    },
    # 类型为stringList的命令对象
    Type: stringList{},
}

# 定义一个名为logTailCmd的命令对象
var logTailCmd = &cmds.Command{
    # 实验性状态
    Status: cmds.Experimental,
    # 帮助文本，包括简短描述和标语
    Helptext: cmds.HelpText{
        Tagline: "Read the event log.",
        ShortDescription: `
Outputs event log messages (not other log messages) as they are generated.

Currently broken. Follow https://github.com/ipfs/kubo/issues/9245 for updates.
`,
    },
    # 运行命令的函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取请求的上下文
        ctx := req.Context
        # 创建一个io.Pipe，用于在goroutine中读取事件日志消息
        r, w := io.Pipe()
        go func() {
            # 在goroutine中监听上下文的完成信号，关闭写入端
            defer w.Close()
            <-ctx.Done()
        }()
        # 将写入端添加到日志写入组
        lwriter.WriterGroup.AddWriter(w)
        # 将读取端的内容发送到响应中
        return res.Emit(r)
    },
}
```