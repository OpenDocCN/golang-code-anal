# `kubo\core\commands\commands.go`

```
// Package commands implements the ipfs command interface
//
// Using github.com/ipfs/kubo/commands to define the command line and HTTP
// APIs.  This is the interface available to folks using IPFS from outside of
// the Go language.
package commands

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "fmt"    // 导入 fmt 包，用于格式化输入输出
    "io"     // 导入 io 包，提供了基本的 I/O 接口
    "os"     // 导入 os 包，提供了操作系统功能
    "sort"   // 导入 sort 包，用于排序
    "strings"  // 导入 strings 包，用于操作字符串

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入 github.com/ipfs/go-ipfs-cmds 包，用于定义命令行和 HTTP API
)

type commandEncoder struct {
    w io.Writer  // 定义一个 io.Writer 接口类型的字段 w
}

func (e *commandEncoder) Encode(v interface{}) error {
    var (
        cmd *Command  // 定义一个 Command 类型的变量 cmd
        ok  bool      // 定义一个布尔类型的变量 ok
    )

    if cmd, ok = v.(*Command); !ok {  // 判断 v 是否为 *Command 类型
        return fmt.Errorf(`core/commands: unexpected type %T, expected *"core/commands".Command`, v)  // 如果不是 *Command 类型，则返回错误
    }

    for _, s := range cmdPathStrings(cmd, cmd.showOpts) {  // 遍历 cmdPathStrings 函数返回的字符串切片
        _, err := e.w.Write([]byte(s + "\n"))  // 将字符串写入到 e.w 中
        if err != nil {  // 如果出现错误
            return err  // 返回错误
        }
    }

    return nil  // 返回空值
}

type Command struct {
    Name        string        // 定义一个字符串类型的字段 Name
    Subcommands []Command     // 定义一个 Command 类型的切片字段 Subcommands
    Options     []Option      // 定义一个 Option 类型的切片字段 Options

    showOpts bool  // 定义一个布尔类型的字段 showOpts
}

type Option struct {
    Names []string  // 定义一个字符串类型的切片字段 Names
}

const (
    flagsOptionName = "flags"  // 定义一个常量 flagsOptionName，值为 "flags"
)

// CommandsCmd takes in a root command,
// and returns a command that lists the subcommands in that root
func CommandsCmd(root *cmds.Command) *cmds.Command {
    # 返回一个命令对象
    return &cmds.Command{
        # 帮助文本，包括标语和简短描述
        Helptext: cmds.HelpText{
            Tagline:          "List all available commands.",
            ShortDescription: `Lists all available commands (and subcommands) and exits.`,
        },
        # 子命令映射，包括子命令的名称和对应的命令对象
        Subcommands: map[string]*cmds.Command{
            "completion": CompletionCmd(root),
        },
        # 选项列表，包括命令的选项
        Options: []cmds.Option{
            cmds.BoolOption(flagsOptionName, "f", "Show command flags"),
        },
        # 额外信息，包括设置不使用仓库
        Extra: CreateCmdExtras(SetDoesNotUseRepo(true)),
        # 运行函数，处理请求并返回结果
        Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
            rootCmd := cmd2outputCmd("ipfs", root)
            rootCmd.showOpts, _ = req.Options[flagsOptionName].(bool)
            return cmds.EmitOnce(res, &rootCmd)
        },
        # 编码器映射，包括文本编码器的函数
        Encoders: cmds.EncoderMap{
            cmds.Text: func(req *cmds.Request) func(io.Writer) cmds.Encoder {
                return func(w io.Writer) cmds.Encoder { return &commandEncoder{w} }
            },
        },
        # 类型为命令对象
        Type: Command{},
    }
// 为给定命令生成输出命令
func cmd2outputCmd(name string, cmd *cmds.Command) Command {
    // 创建一个与命令选项数量相同的选项切片
    opts := make([]Option, len(cmd.Options))
    // 遍历命令选项，将其名称添加到选项切片中
    for i, opt := range cmd.Options {
        opts[i] = Option{opt.Names()}
    }

    // 创建输出命令对象
    output := Command{
        Name:        name,
        Subcommands: make([]Command, 0, len(cmd.Subcommands)),
        Options:     opts,
    }

    // 遍历命令的子命令，将其转换为输出命令并添加到输出命令的子命令切片中
    for name, sub := range cmd.Subcommands {
        output.Subcommands = append(output.Subcommands, cmd2outputCmd(name, sub))
    }

    // 返回输出命令对象
    return output
}

// 为给定命令生成路径字符串切片
func cmdPathStrings(cmd *Command, showOptions bool) []string {
    var cmds []string

    // 递归函数，用于生成路径字符串
    var recurse func(prefix string, cmd *Command)
    recurse = func(prefix string, cmd *Command) {
        newPrefix := prefix + cmd.Name
        cmds = append(cmds, newPrefix)
        // 如果需要显示选项，则将命令的选项添加到路径字符串切片中
        if prefix != "" && showOptions {
            for _, options := range cmd.Options {
                var cmdOpts []string
                for _, flag := range options.Names {
                    if len(flag) == 1 {
                        flag = "-" + flag
                    } else {
                        flag = "--" + flag
                    }
                    cmdOpts = append(cmdOpts, newPrefix+" "+flag)
                }
                cmds = append(cmds, strings.Join(cmdOpts, " / "))
            }
        }
        // 递归处理子命令
        for _, sub := range cmd.Subcommands {
            recurse(newPrefix+" ", &sub)
        }
    }

    // 从根命令开始递归生成路径字符串
    recurse("", cmd)
    // 对路径字符串切片进行排序
    sort.Strings(cmds)
    // 返回路径字符串切片
    return cmds
}

// 生成用于完成命令的命令对象
func CompletionCmd(root *cmds.Command) *cmds.Command {
    return &cmds.Command{
        Helptext: cmds.HelpText{
            Tagline: "Generate shell completions.",
        },
        NoRemote: true,
        Subcommands: map[string]*cmds.Command{
            "bash": {
                Helptext: cmds.HelpText{
                    Tagline:          "Generate bash shell completions.",
                    ShortDescription: "Generates command completions for the bash shell.",
                    LongDescription: `
# 为 bash shell 生成命令补全
Generates command completions for the bash shell.

# 最简单的方法是将补全写入文件，然后将其源化
# 将补全写入文件，然后将其源化
  > ipfs commands completion bash > ipfs-completion.bash
  > source ./ipfs-completion.bash

# 要永久安装补全，可以将其移动到 /etc/bash_completion.d 或从您的 ~/.bashrc 文件中源化
To install the completions permanently, they can be moved to
/etc/bash_completion.d or sourced from your ~/.bashrc file.
`,
                },
                NoRemote: true,
                Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
                    var buf bytes.Buffer
                    if err := writeBashCompletions(root, &buf); err != nil {
                        return err
                    }
                    res.SetLength(uint64(buf.Len()))
                    return res.Emit(&buf)
                },
            },
            "zsh": {
                Helptext: cmds.HelpText{
                    Tagline:          "Generate zsh shell completions.",
                    ShortDescription: "Generates command completions for the zsh shell.",
                    LongDescription: `
Generates command completions for the zsh shell.

# 最简单的方法是将补全写入文件，然后将其源化
# 将补全写入文件，然后将其源化
  > ipfs commands completion zsh > ipfs-completion.zsh
  > source ./ipfs-completion.zsh

# 要永久安装补全，可以将其移动到 /etc/zsh/completions 或从您的 ~/.zshrc 文件中源化
To install the completions permanently, they can be moved to
/etc/zsh/completions or sourced from your ~/.zshrc file.
// 定义一个名为 nonFatalError 的自定义类型，用于表示非致命错误
type nonFatalError string

// streamResult 是一个辅助函数，用于流式传输可能包含非致命错误的结果。
// 辅助函数允许在内部错误时引发 panic。
func streamResult(procVal func(interface{}, io.Writer) nonFatalError) func(cmds.Response, cmds.ResponseEmitter) error {
    // ...
}
    # 定义一个匿名函数，接收两个参数，返回一个错误
    return func(res cmds.Response, re cmds.ResponseEmitter) (err error) {
        # 延迟执行的函数，用于处理 panic，关闭 ResponseEmitter
        defer func() {
            # 捕获 panic，将错误信息封装成 error 对象
            if r := recover(); r != nil {
                err = fmt.Errorf("internal error: %v", r)
            }
            # 关闭 ResponseEmitter
            re.Close()
        }()

        # 初始化一个变量，用于标记是否有错误发生
        var errors bool
        # 循环处理 Response 中的数据
        for {
            # 从 Response 中获取下一个值
            v, err := res.Next()
            # 处理获取值时可能出现的错误
            if err != nil {
                # 如果是文件结束错误，则跳出循环
                if err == io.EOF {
                    break
                }
                # 如果是其他错误，则直接返回
                return err
            }

            # 处理获取到的值，将处理结果输出到标准输出，并返回可能的错误信息
            errorMsg := procVal(v, os.Stdout)

            # 如果有错误信息，则标记有错误发生，并将错误信息输出到标准错误
            if errorMsg != "" {
                errors = true
                fmt.Fprintf(os.Stderr, "%s\n", errorMsg)
            }
        }

        # 如果有错误发生，则返回一个包含错误信息的 error 对象
        if errors {
            return fmt.Errorf("errors while displaying some entries")
        }
        # 如果没有错误发生，则返回 nil
        return nil
    }
# 闭合前面的函数定义
```