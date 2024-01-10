# `kubo\test\dependencies\go-timeout\main.go`

```
package main

import (
    "context"  // 导入上下文包，用于控制命令执行的超时
    "fmt"  // 导入格式化包，用于格式化输出
    "os"  // 导入操作系统包，用于访问命令行参数和退出程序
    "os/exec"  // 导入执行外部命令包，用于执行外部命令
    "strconv"  // 导入字符串转换包，用于将字符串转换为数字
    "syscall"  // 导入系统调用包，用于获取命令执行的退出状态
    "time"  // 导入时间包，用于设置超时时间
)

func main() {
    if len(os.Args) < 3 {  // 检查命令行参数是否足够
        fmt.Fprintf(os.Stderr,
            "Usage: %s <timeout-in-sec> <command ...>\n", os.Args[0])  // 格式化输出错误信息
        os.Exit(1)  // 退出程序
    }
    timeout, err := strconv.ParseUint(os.Args[1], 10, 32)  // 将超时时间参数转换为无符号整数
    if err != nil {  // 检查转换是否出错
        fmt.Fprintf(os.Stderr, "Error: %v\n", err)  // 格式化输出错误信息
        os.Exit(1)  // 退出程序
    }
    ctx, cancel := context.WithTimeout(context.Background(), time.Duration(timeout)*time.Second)  // 创建带超时的上下文
    defer cancel()  // 延迟取消上下文

    cmd := exec.CommandContext(ctx, os.Args[2], os.Args[3:]...)  // 使用带超时的上下文创建命令
    cmd.Stdin = os.Stdin  // 设置命令的标准输入
    cmd.Stdout = os.Stdout  // 设置命令的标准输出
    cmd.Stderr = os.Stderr  // 设置命令的标准错误输出
    err = cmd.Start()  // 启动命令
    if err != nil {  // 检查启动命令是否出错
        fmt.Fprintf(os.Stderr, "Error: %v\n", err)  // 格式化输出错误信息
    }
    err = cmd.Wait()  // 等待命令执行结束

    if err != nil {  // 检查命令执行是否出错
        if ctx.Err() != nil {  // 检查是否超时
            os.Exit(124)  // 超时退出码
        } else {
            exitErr, ok := err.(*exec.ExitError)  // 检查是否为退出错误
            if !ok {  // 如果不是退出错误
                fmt.Fprintf(os.Stderr, "Error: %v\n", err)  // 格式化输出错误信息
                os.Exit(255)  // 一般错误退出码
            }
            waits, ok := exitErr.Sys().(syscall.WaitStatus)  // 获取退出状态
            if !ok {  // 如果获取退出状态出错
                fmt.Fprintf(os.Stderr, "Error: %v\n", err)  // 格式化输出错误信息
                os.Exit(255)  // 一般错误退出码
            }
            os.Exit(waits.ExitStatus())  // 使用退出状态作为退出码
        }
    } else {
        os.Exit(0)  // 正常退出码
    }
}
```