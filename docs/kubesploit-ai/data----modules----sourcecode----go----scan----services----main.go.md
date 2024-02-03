# `kubesploit\data\modules\sourcecode\go\scan\services\main.go`

```go
package main

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "log"  // 导入 log 包，用于记录错误日志
    "net"  // 导入 net 包，用于处理网络相关操作
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
)

func getIPAddresses(cidr string) ([]string, error) {
    // 解析 CIDR 地址
    ip, ipnet, err := net.ParseCIDR(cidr)
    if err != nil {
        return nil, err
    }

    var ips []string
    // 遍历 CIDR 地址范围内的 IP 地址
    for ip := ip.Mask(ipnet.Mask); ipnet.Contains(ip); inc(ip) {
        ips = append(ips, ip.String())
    }
    // 移除网络地址和广播地址
    lenIPs := len(ips)
    var ipAddresses []string

    switch {
    case lenIPs < 2:
        ipAddresses = ips

    default:
        // 如果 IP 地址数量大于等于 2，则移除第一个和最后一个地址
        ipAddresses = ips[1 : len(ips)-1]
    }

    return ipAddresses, nil
}

//  http://play.golang.org/p/m8TNTtygK0
func inc(ip net.IP) {
    // 递增 IP 地址
    for j := len(ip) - 1; j >= 0; j-- {
        ip[j]++
        if ip[j] > 0 {
            break
        }
    }
}

func mainfunc(cidr string, threadsStr string, intervalsPrint string) {
    var numberOfThreads int
    var err error
    if threadsStr == "" {
        fmt.Println("[*] Using default settings: 100 threads")  // 输出默认设置信息
        numberOfThreads = 100
    } else {
        numberOfThreads, err = strconv.Atoi(threadsStr)  // 将字符串转换为整数
        if err != nil {
            fmt.Println("[*] Using default settings: 100 threads")  // 输出默认设置信息
            numberOfThreads = 100
        }
    }

    if cidr == "" {
        cidr = "10.96.0.0/12"
        fmt.Println("[*] Using default settings: CIDR: 10.96.0.0/12")  // 输出默认设置信息
    }

    fmt.Printf("[*] Scanning for kubernetes services\n[*] CIDR: %s\n[*] Threads: %s\n", cidr, strconv.Itoa(numberOfThreads))  // 格式化输出扫描信息

    ipAddresses, err := getIPAddresses(cidr)  // 获取 CIDR 地址范围内的 IP 地址
    if err != nil {
        log.Fatal(err)  // 如果出现错误，记录错误日志并退出程序
    }

    fmt.Printf("[*] Scanning %d IP addresses\n", len(ipAddresses))  // 输出扫描到的 IP 地址数量
    fmt.Printf("[*] Threads: %d\n", numberOfThreads)  // 输出线程数量

    //var hostsMap map[string][]string
    //l := sync.Mutex{}

    sem := make(chan bool, numberOfThreads)  // 创建一个带缓冲区的信号量通道，用于控制并发数量

    count := 0
    var interval int
    interval, err = strconv.Atoi(intervalsPrint)  // 将字符串转换为整数
    // 如果发生错误，则设置间隔为10000，并打印每个地址
    if err != nil {
        interval = 10000
        fmt.Printf("[*] Printing every %s addresses\n", interval)
    } else {
        // 否则，打印每个地址
        fmt.Printf("[*] Printing every %s addresses\n", interval)
    }

    // 遍历IP地址列表
    for _, ip := range(ipAddresses) {
        // 向信号量通道发送true
        sem <- true

        // 启动并发匿名函数
        go func(ip string) {
            // 计数加一
            count += 1
            // 如果计数是间隔的倍数，则打印扫描的地址数量
            if count % interval == 0 {
                fmt.Printf("[*] Scan: %d addresses till now...\n", count)
            }

            // 查询IP地址对应的主机名
            host, err := net.LookupAddr(ip)

            // 如果发生错误，则打印错误信息
            if err != nil {
                //log.Fatal(err)
                //fmt.Println("Error")
            } else {
                // 否则，打印IP地址和主机名
                fmt.Printf("[*] %s -> %s\n", ip, host)
            }
            /*if err == nil {
                l.Lock()
                hostsMap[ip] = hosts
                fmt.Printf("added %s\n", hosts)
                l.Unlock()
            }*/

            // 从信号量通道接收数据
            <- sem
        }(ip)
    }

    // 通过循环向信号量通道发送true，以等待所有并发任务完成
    for i := 0; i < cap(sem); i++ {
        sem <- true
    }
    // 打印任务完成信息
    fmt.Println("[*] Done")
# 主函数入口
func main(){
    # 调用主功能函数，传入参数"10.96.0.0/16", "100", "100"
    mainfunc("10.96.0.0/16", "100", "100")
}
```