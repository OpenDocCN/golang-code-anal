# `kubo\coverage\main\main.go`

```
//go:build testrunmain
// +build testrunmain

package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于实现 I/O 操作
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "os/exec"  // 导入 os/exec 包，用于执行外部命令
    "os/signal"  // 导入 os/signal 包，用于处理信号
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "syscall"  // 导入 syscall 包，提供了操作系统底层的系统调用接口
)

func main() {
    coverDir := os.Getenv("IPFS_COVER_DIR")  // 获取环境变量 IPFS_COVER_DIR 的值
    if len(coverDir) == 0 {  // 如果 IPFS_COVER_DIR 为空
        fmt.Println("IPFS_COVER_DIR not defined")  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }
    coverFile, err := os.CreateTemp(coverDir, "coverage-")  // 在指定目录创建临时文件
    if err != nil {  // 如果创建临时文件出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }

    retFile, err := os.CreateTemp("", "cover-ret-file")  // 在系统默认目录创建临时文件
    if err != nil {  // 如果创建临时文件出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }

    args := []string{"-test.run", "^TestRunMain$", "-test.coverprofile=" + coverFile.Name(), "--"}  // 定义命令行参数
    args = append(args, os.Args[1:]...)  // 将程序运行时的参数追加到命令行参数中

    p := exec.Command("ipfs-test-cover", args...)  // 创建一个执行外部命令的对象
    p.Stdin = os.Stdin  // 设置标准输入
    p.Stdout = os.Stdout  // 设置标准输出
    p.Stderr = os.Stderr  // 设置标准错误输出
    p.Env = append(os.Environ(), "IPFS_COVER_RET_FILE="+retFile.Name())  // 设置环境变量

    p.SysProcAttr = &syscall.SysProcAttr{  // 设置进程属性
        Pdeathsig: syscall.SIGTERM,  // 设置进程退出信号
    }

    sig := make(chan os.Signal, 10)  // 创建一个信号通道
    start := make(chan struct{})  // 创建一个启动通道
    go func() {  // 启动一个 goroutine
        <-start  // 等待启动信号
        for {  // 循环处理信号
            p.Process.Signal(<-sig)  // 向进程发送信号
        }
    }()

    signal.Notify(sig, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM)  // 监听指定的信号

    err = p.Start()  // 启动外部命令
    if err != nil {  // 如果启动出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }

    close(start)  // 关闭启动通道

    err = p.Wait()  // 等待外部命令执行结束
    if err != nil {  // 如果等待出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }

    b, err := io.ReadAll(retFile)  // 读取临时文件的内容
    if err != nil {  // 如果读取出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }
    b = b[:len(b)-1]  // 去掉末尾的换行符
    d, err := strconv.Atoi(string(b))  // 将字符串转换为整数
    if err != nil {  // 如果转换出错
        fmt.Println(err.Error())  // 输出错误信息
        os.Exit(1)  // 退出程序并返回状态码 1
    }
    os.Exit(d)  // 退出程序并返回转换后的整数
}
```