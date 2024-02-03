# `kubo\test\dependencies\ma-pipe-unidir\main.go`

```go
package main

import (
    "flag"  // 导入 flag 包，用于命令行参数解析
    "fmt"   // 导入 fmt 包，用于格式化输出
    "io"    // 导入 io 包，用于 I/O 操作
    "os"    // 导入 os 包，用于操作系统功能
    "strconv"   // 导入 strconv 包，用于字符串和基本数据类型之间的转换

    ma "github.com/multiformats/go-multiaddr"  // 导入 ma 别名的 multiaddr 包
    manet "github.com/multiformats/go-multiaddr/net"  // 导入 manet 别名的 multiaddr/net 包
)

const USAGE = "ma-pipe-unidir [-l|--listen] [--pidFile=path] [-h|--help] <send|recv> <multiaddr>\n"  // 定义 USAGE 常量，用于提示用户使用说明

type Opts struct {
    Listen  bool    // 定义结构体 Opts，包含 Listen 字段，表示是否监听
    PidFile string   // 定义结构体 Opts，包含 PidFile 字段，表示进程 ID 文件路径
}

func app() int {
    opts := Opts{}  // 创建 Opts 结构体实例 opts
    flag.BoolVar(&opts.Listen, "l", false, "")  // 解析命令行参数，设置监听标志
    flag.BoolVar(&opts.Listen, "listen", false, "")  // 解析命令行参数，设置监听标志
    flag.StringVar(&opts.PidFile, "pidFile", "", "")  // 解析命令行参数，设置进程 ID 文件路径
    flag.Usage = func() {  // 设置自定义的 Usage 函数
        fmt.Print(USAGE)  // 输出使用说明
    }
    flag.Parse()  // 解析命令行参数
    args := flag.Args()  // 获取非标志参数

    if len(args) < 2 { // <mode> <addr>，如果参数个数小于 2
        fmt.Print(USAGE)  // 输出使用说明
        return 1  // 返回错误码 1
    }

    mode := args[0]  // 获取模式参数
    addr := args[1]  // 获取地址参数

    if mode != "send" && mode != "recv" {  // 如果模式参数不是 send 或 recv
        fmt.Print(USAGE)  // 输出使用说明
        return 1  // 返回错误码 1
    }

    maddr, err := ma.NewMultiaddr(addr)  // 创建多地址对象
    if err != nil {  // 如果创建失败
        return 1  // 返回错误码 1
    }

    var conn manet.Conn  // 定义多地址网络连接

    if opts.Listen {  // 如果设置了监听标志
        listener, err := manet.Listen(maddr)  // 监听指定多地址
        if err != nil {  // 如果监听失败
            return 1  // 返回错误码 1
        }

        if len(opts.PidFile) > 0 {  // 如果设置了进程 ID 文件路径
            data := []byte(strconv.Itoa(os.Getpid()))  // 获取当前进程 ID，并转换为字节数组
            err := os.WriteFile(opts.PidFile, data, 0o644)  // 将进程 ID 写入文件
            if err != nil {  // 如果写入失败
                return 1  // 返回错误码 1
            }

            defer os.Remove(opts.PidFile)  // 延迟删除进程 ID 文件
        }

        conn, err = listener.Accept()  // 接受连接
        if err != nil {  // 如果接受失败
            return 1  // 返回错误码 1
        }
    } else {  // 如果未设置监听标志
        var err error
        conn, err = manet.Dial(maddr)  // 拨号连接指定多地址
        if err != nil {  // 如果连接失败
            return 1  // 返回错误码 1
        }

        if len(opts.PidFile) > 0 {  // 如果设置了进程 ID 文件路径
            data := []byte(strconv.Itoa(os.Getpid()))  // 获取当前进程 ID，并转换为字节数组
            err := os.WriteFile(opts.PidFile, data, 0o644)  // 将进程 ID 写入文件
            if err != nil {  // 如果写入失败
                return 1  // 返回错误码 1
            }

            defer os.Remove(opts.PidFile)  // 延迟删除进程 ID 文件
        }

    }

    defer conn.Close()  // 延迟关闭连接
    switch mode {  // 根据模式参数进行处理
    case "recv":  // 如果是接收模式
        _, err = io.Copy(os.Stdout, conn)  // 将连接数据拷贝到标准输出
    # 如果命令是 "send"，则将标准输入内容拷贝到连接中
    case "send":
        _, err = io.Copy(conn, os.Stdin)
    # 如果命令不是 "send"，则返回 1
    default:
        return 1
    }
    # 如果发生错误，返回 1
    if err != nil {
        return 1
    }
    # 没有错误发生，返回 0
    return 0
# 主函数入口
func main() {
    # 退出应用程序并返回退出代码
    os.Exit(app())
}
```