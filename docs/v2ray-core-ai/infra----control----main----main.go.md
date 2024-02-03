# `v2ray-core\infra\control\main\main.go`

```go
package main

import (
    "flag"  // 导入 flag 包，用于解析命令行参数
    "fmt"   // 导入 fmt 包，用于格式化输出
    "os"    // 导入 os 包，提供对操作系统功能的访问

    commlog "v2ray.com/core/common/log"  // 导入 v2ray 日志包
    // _ "v2ray.com/core/infra/conf/command"
    "v2ray.com/core/infra/control"  // 导入 v2ray 控制包
)

func getCommandName() string {
    if len(os.Args) > 1 {
        return os.Args[1]  // 如果命令行参数大于 1，返回第一个参数作为命令名
    }
    return ""  // 否则返回空字符串
}

func main() {
    // 让 v2ctl 在标准错误输出打印日志
    commlog.RegisterHandler(commlog.NewLogger(commlog.CreateStderrLogWriter()))
    name := getCommandName()  // 获取命令名
    cmd := control.GetCommand(name)  // 获取对应命令的控制对象
    if cmd == nil {
        fmt.Fprintln(os.Stderr, "Unknown command:", name)  // 如果命令为空，打印错误信息
        fmt.Fprintln(os.Stderr)

        fmt.Println("v2ctl <command>")  // 打印使用说明
        fmt.Println("Available commands:")
        control.PrintUsage()  // 打印可用命令
        return
    }

    if err := cmd.Execute(os.Args[2:]); err != nil {  // 执行命令
        hasError := false
        if err != flag.ErrHelp {  // 如果错误不是帮助信息
            fmt.Fprintln(os.Stderr, err.Error())  // 打印错误信息
            fmt.Fprintln(os.Stderr)
            hasError = true
        }

        for _, line := range cmd.Description().Usage {  // 遍历命令的使用说明
            fmt.Println(line)  // 打印每行使用说明
        }

        if hasError {  // 如果有错误
            os.Exit(-1)  // 退出程序并返回错误状态
        }
    }
}
```