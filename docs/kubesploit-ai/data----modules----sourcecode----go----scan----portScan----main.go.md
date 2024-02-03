# `kubesploit\data\modules\sourcecode\go\scan\portScan\main.go`

```go
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net"  // 导入 net 包，用于网络通信
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "strings"  // 导入 strings 包，用于处理字符串
    "time"  // 导入 time 包，用于时间相关操作
)

// 检查指定主机和端口是否开放
func isPortOpen(host string, port int) (bool, error) {
    isOpen := false  // 初始化端口是否开放的标志为 false
    timeout := time.Second  // 设置超时时间为 1 秒

    portStr := strconv.Itoa(port)  // 将端口号转换为字符串
    conn, err := net.DialTimeout("tcp", net.JoinHostPort(host, portStr), timeout)  // 使用 TCP 协议连接指定主机和端口，设置超时时间
    if err != nil {  // 如果连接出现错误
        //fmt.Println("Connecting error:", Err)  // 打印连接错误信息
    }
    if conn != nil {  // 如果连接成功
        defer conn.Close()  // 延迟关闭连接
        isOpen = true  // 将端口是否开放的标志设置为 true
    }

    return isOpen, err  // 返回端口是否开放的标志和可能出现的错误
}

// 已知端口号和对应服务的映射关系
var KNOWN_PORTS = map[int]string{
    27017: "mongodb",
    // ... 其他端口和服务的映射
}

// 端口信息结构体
type PortInfo struct {
    IsPortOpen bool  // 端口是否开放的标志
    Port       int  // 端口号
    PortDescription string  // 端口对应的服务描述
    Err        error  // 可能出现的错误
}
func scanPorts(ipAddress string, ports map[int]string, concurrencyLimit int) []PortInfo {
    // 创建一个切片来保存我们期望的结果
    var openPorts []PortInfo

    // 这个带缓冲的通道将在并发限制处阻塞
    semaphoreChan := make(chan struct{}, concurrencyLimit)

    // 这个通道不会阻塞，并收集 HTTP 请求的结果
    resultsChan := make(chan *PortInfo)

    // 确保在使用完这些通道后关闭它们
    defer func() {
        close(semaphoreChan)
        close(resultsChan)
    }()

    // 保持索引并循环遍历我们将发送请求的每个 URL
    for port, portDescription := range ports {

        // 使用索引和 URL 启动一个 Go 协程
        go func(ipAddress string, port int, portDescription string) {

            // 这将向 semaphoreChan 发送一个空结构体，基本上是在说将限制加一，但当达到限制时阻塞，直到有空间
            semaphoreChan <- struct{}{}

            var err error
            isOpen := false

            isOpen, err = isPortOpen(ipAddress, port)

            result := &PortInfo{isOpen, port, portDescription, err}

            // 现在我们可以通过 resultsChan 发送 PortInfo 结构体
            resultsChan <- result
            // 一旦完成，我们从 semaphoreChan 中读取，这会将限制减一，并允许另一个 Go 协程启动
            <-semaphoreChan

        }(ipAddress, port, portDescription)
    }

    // 开始监听 resultsChan 上的任何结果
    // 一旦获得 PortInfo，将其附加到 PortInfo 切片中
    var count int
    // 无限循环，从结果通道中接收结果
    for {
        result := <-resultsChan
        // 计数加一
        count += 1

        // 如果端口是开放的，则将结果添加到开放端口列表中
        if result.IsPortOpen {
            openPorts = append(openPorts, *result)
        }

        // 如果已经达到了期望的 URL 数量，则停止循环
        if count == len(ports) {
            break
        }
    }

    // 现在我们完成了，返回结果列表
    return openPorts
// 在 JSON 中
//var arg1ArrayOfAddresses = []string{"127.0.0.1"}
//var arg1ArrayOfAddresses []string

// 主函数，输入参数为地址字符串
func mainfunc(inputAddresses string) {
    // 并发限制为 15
    concurrencyLimit := 15
    fmt.Printf("[*] Scanning for open ports (%d threads)\n", concurrencyLimit)
    // 程序休眠1秒
    time.Sleep(1)
    // 去除输入地址字符串两端的空格
    inputAddresses = strings.TrimSpace(inputAddresses)

    // 必须声明变量，不能使用 ":="，因为这会导致 Yaegi 无法识别
    var inputArrayOfAddresses []string
    // 使用分号分割输入地址字符串，得到地址数组
    inputArrayOfAddresses = strings.Split(inputAddresses, ";")

    // 如果输入地址字符串长度小于1，则默认为 "127.0.0.1"
    if len(inputAddresses) < 1 {
        inputArrayOfAddresses = append(inputArrayOfAddresses, "127.0.0.1")
    }

    // 将地址数组赋值给 args
    args := inputArrayOfAddresses
    // 遍历地址数组
    for _, ip := range args {
        // 打印扫描的 IP 地址
        fmt.Printf("[*] Scanning IP: %s\n", ip)
        // 如果 IP 地址包含字母，则进行 DNS 查询
        if strings.ContainsAny(strings.ToLower(ip), "a | b | c | d | e | f | g | h | i | j | k | l | m | n | o | p | q | r | s | t | u | v | w | x | y | z"){
            // 进行 DNS 查询
            addr,err := net.LookupIP(ip)
            if err != nil {
                fmt.Println("[*] Unknown host")
                continue
            } else {
                if len(addr) > 0 {
                    ip = addr[0].String()
                }else {
                    fmt.Println("[*] Unknown IP")
                    continue
                }
            }
        }
        // 扫描 IP 地址的端口状态
        portsStatus := scanPorts(ip, KNOWN_PORTS, concurrencyLimit)

        // 遍历端口状态数组
        for _, portInfo := range portsStatus{
            // 如果端口是开放状态，则打印端口号和描述
            if portInfo.IsPortOpen {
                fmt.Printf("    %d: %s\n", portInfo.Port, portInfo.PortDescription)
            }
        }
    }

    /*jsonByteData, err := json.Marshal(portsStatus)
    if err != nil {
        log.Fatal("Cannot encode to JSON ", err)
    }

    fmt.Println(string(jsonByteData))*/
}

/*
func main(){
    mainfunc("example.com;example.com")
}
*/
```