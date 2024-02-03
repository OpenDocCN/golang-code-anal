# `kubo\core\commands\external.go`

```go
# 导入所需的包
package commands

import (
    "bytes"  # 导入 bytes 包，用于操作字节切片
    "fmt"  # 导入 fmt 包，用于格式化输入输出
    "io"  # 导入 io 包，提供了基本的 I/O 接口
    "os"  # 导入 os 包，提供了操作系统功能
    "os/exec"  # 导入 os/exec 包，用于执行外部命令
    "strings"  # 导入 strings 包，提供了操作字符串的函数

    cmds "github.com/ipfs/go-ipfs-cmds"  # 导入自定义的 cmds 包
)

# 定义一个返回 *cmds.Command 类型的函数 ExternalBinary，参数为 instructions 字符串
func ExternalBinary(instructions string) *cmds.Command {
    // 返回一个命令对象，该对象包含参数、外部执行标志和不允许远程执行标志
    return &cmds.Command{
        // 定义命令的参数，包括参数名、是否必需、是否可变和参数描述
        Arguments: []cmds.Argument{
            cmds.StringArg("args", false, true, "Arguments for subcommand."),
        },
        // 标识该命令为外部执行
        External: true,
        // 标识该命令不允许远程执行
        NoRemote: true,
        // 定义命令的执行函数
        Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
            // 根据请求的路径和参数拼接出子命令的可执行文件名
            binname := strings.Join(append([]string{"ipfs"}, req.Path...), "-")
            // 检查可执行文件是否存在
            _, err := exec.LookPath(binname)
            if err != nil {
                // 对未安装的可执行文件的特殊处理，例如 '--help'
                for _, arg := range req.Arguments {
                    if arg == "--help" || arg == "-h" {
                        // 构建提示信息并返回给调用者
                        buf := new(bytes.Buffer)
                        fmt.Fprintf(buf, "%s is an 'external' command.\n", binname)
                        fmt.Fprintf(buf, "It does not currently appear to be installed.\n")
                        fmt.Fprintf(buf, "%s\n", instructions)
                        return res.Emit(buf)
                    }
                }
                // 返回错误信息，指示可执行文件未安装
                return fmt.Errorf("%s not installed", binname)
            }

            // 创建管道用于命令的输入输出
            r, w := io.Pipe()

            // 创建子命令对象
            cmd := exec.Command(binname, req.Arguments...)

            // 设置子命令的标准输入、输出和错误输出
            cmd.Stdin = io.LimitReader(nil, 0)
            cmd.Stdout = w
            cmd.Stderr = w

            // 设置子命令的环境变量
            osenv := os.Environ()
            cmd.Env = osenv

            // 启动子命令
            err = cmd.Start()
            if err != nil {
                return fmt.Errorf("failed to start subcommand: %s", err)
            }

            // 创建用于接收子命令执行结果的通道
            errC := make(chan error)

            // 启动协程等待子命令执行完成
            go func() {
                var err error
                defer func() { errC <- err }()
                err = cmd.Wait()
                w.Close()
            }()

            // 将子命令的输出发送给调用者
            err = res.Emit(r)
            if err != nil {
                return err
            }

            // 返回子命令执行的结果
            return <-errC
        },
    }
# 闭合前面的函数定义
```