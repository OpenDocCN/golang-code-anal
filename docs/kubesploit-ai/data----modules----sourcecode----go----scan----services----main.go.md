# `kubesploit\data\modules\sourcecode\go\scan\services\main.go`

```
package main

import (
	"fmt"  // 导入 fmt 包，用于格式化输出
	"log"  // 导入 log 包，用于日志记录
	"net"  // 导入 net 包，用于网络操作
	"strconv"  // 导入 strconv 包，用于字符串和数字之间的转换
)

func getIPAddresses(cidr string) ([]string, error) {
	ip, ipnet, err := net.ParseCIDR(cidr)  // 解析 CIDR 地址
	if err != nil {
		return nil, err  // 如果解析出错，返回错误
	}

	var ips []string  // 创建存放 IP 地址的切片
	for ip := ip.Mask(ipnet.Mask); ipnet.Contains(ip); inc(ip) {  // 遍历 IP 地址范围
		ips = append(ips, ip.String())  // 将 IP 地址添加到切片中
	}
	// 移除网络地址和广播地址
	// 获取 IP 地址切片的长度
	lenIPs := len(ips)
	// 声明一个字符串切片用于存储 IP 地址
	var ipAddresses []string

	// 根据 IP 地址切片的长度进行不同的处理
	switch {
	case lenIPs < 2:
		// 如果 IP 地址切片长度小于 2，则直接赋值给 ipAddresses
		ipAddresses = ips

	default:
		// 如果 IP 地址切片长度大于等于 2，则取出除第一个和最后一个元素之外的所有元素
		// 注意：这里有一条注释说明不应该在这里触发 panic，因为在之前已经检查了 lenIPs 的值
		ipAddresses = ips[1 : len(ips)-1]
	}

	// 返回处理后的 IP 地址切片和 nil 错误
	return ipAddresses, nil
}

//  http://play.golang.org/p/m8TNTtygK0
// 对 IP 地址进行递增操作
func inc(ip net.IP) {
	// 从 IP 地址的最后一位开始逐个递减
	for j := len(ip) - 1; j >= 0; j-- {
		// 对 IP 地址的每一位进行递增操作
		ip[j]++
		// 如果递增后不为 0，则说明没有溢出，直接返回
		if ip[j] > 0 {
		// 如果条件成立，跳出循环
		break
	}
}

func mainfunc(cidr string, threadsStr string, intervalsPrint string) {
	// 声明变量
	var numberOfThreads int
	var err error
	// 判断线程数是否为空，如果是则使用默认设置：100个线程
	if threadsStr == "" {
		fmt.Println("[*] Using default settings: 100 threads")
		numberOfThreads = 100
	} else {
		// 将线程数转换为整数
		numberOfThreads, err = strconv.Atoi(threadsStr)
		// 如果转换出错，则使用默认设置：100个线程
		if err != nil {
			fmt.Println("[*] Using default settings: 100 threads")
			numberOfThreads = 100
		}
	}

	// 如果 CIDR 地址为空
	if cidr == "" {
		# 设置CIDR变量为"10.96.0.0/12"
		cidr = "10.96.0.0/12"
		# 打印默认设置信息
		fmt.Println("[*] Using default settings: CIDR: 10.96.0.0/12")
	}

	# 打印扫描kubernetes服务的信息，包括CIDR和线程数
	fmt.Printf("[*] Scanning for kubernetes services\n[*] CIDR: %s\n[*] Threads: %s\n", cidr, strconv.Itoa(numberOfThreads)

	# 获取CIDR范围内的IP地址
	ipAddresses, err := getIPAddresses(cidr)
	if err != nil {
		# 如果出现错误，记录错误信息并退出程序
		log.Fatal(err)
	}

	# 打印扫描的IP地址数量和线程数
	fmt.Printf("[*] Scanning %d IP addresses\n", len(ipAddresses))
	fmt.Printf("[*] Threads: %d\n", numberOfThreads)

	# 创建一个信号量，用于控制并发线程数量
	sem := make(chan bool, numberOfThreads)

	# 初始化计数器
	count := 0
// 声明一个变量 interval 用于存储转换后的打印间隔
var interval int
// 尝试将 intervalsPrint 转换为整数并赋值给 interval，如果出错则设置 interval 为默认值 10000
interval, err = strconv.Atoi(intervalsPrint)
if err != nil {
    interval = 10000
    fmt.Printf("[*] Printing every %s addresses\n", interval)
} else {
    fmt.Printf("[*] Printing every %s addresses\n", interval)
}

// 遍历 ipAddresses 切片中的 IP 地址
for _, ip := range(ipAddresses) {
    // 向信号量通道发送信号
    sem <- true

    // 启动一个匿名 goroutine 处理每个 IP 地址
    go func(ip string) {
        // 打印正在检查的 IP 地址
        //fmt.Printf("Checking IP: %s\n", ip)
        // 增加计数器
        count += 1
        // 如果计数器是间隔的倍数，则打印扫描的地址数量
        if count % interval == 0 {
            fmt.Printf("[*] Scan: %d addresses till now...\n", count)
        }
// 使用IP地址查询主机名
host, err := net.LookupAddr(ip)

// 如果发生错误，则打印错误信息
if err != nil {
    //log.Fatal(err)
    //fmt.Println("Error")
} else {
    // 打印IP地址和主机名
    fmt.Printf("[*] %s -> %s\n", ip, host)
}

// 释放信号量
<- sem
		sem <- true  // 将 true 写入信号量通道，表示任务完成
	}
	fmt.Println("[*] Done")  // 打印任务完成的提示信息
}

/*
func main(){
	mainfunc("10.96.0.0/16", "100", "100")  // 调用 mainfunc 函数，传入参数 "10.96.0.0/16", "100", "100"
}
*/
```