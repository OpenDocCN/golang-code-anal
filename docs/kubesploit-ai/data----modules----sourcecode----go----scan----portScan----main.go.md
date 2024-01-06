# `kubesploit\data\modules\sourcecode\go\scan\portScan\main.go`

```
package main

import (
	"fmt" // 导入 fmt 包，用于格式化输出
	"net" // 导入 net 包，用于网络操作
	"strconv" // 导入 strconv 包，用于字符串和基本数据类型之间的转换
	"strings" // 导入 strings 包，用于字符串操作
	"time" // 导入 time 包，用于时间操作
)

// 检查指定主机和端口是否开放
func isPortOpen(host string, port int) (bool, error) {
	isOpen := false // 初始化端口是否开放的标志为 false
	timeout := time.Second // 设置超时时间为 1 秒

	portStr := strconv.Itoa(port) // 将端口号转换为字符串
	conn, err := net.DialTimeout("tcp", net.JoinHostPort(host, portStr), timeout) // 使用 TCP 协议连接指定主机和端口，设置超时时间
	if err != nil { // 如果连接出现错误
		//fmt.Println("Connecting error:", Err) // 打印连接错误信息
	}
	if conn != nil { // 如果连接成功
		defer conn.Close()  // 延迟关闭连接，确保在函数返回时关闭连接
		isOpen = true  // 设置isOpen变量为true，表示连接已打开
	}

	return isOpen, err  // 返回isOpen和err变量的值

var KNOWN_PORTS = map[int]string{  // 定义一个整型到字符串的映射，表示已知端口和对应的服务
	27017: "mongodb",  // MongoDB数据库服务
	28017: "mongodb web admin",  // MongoDB Web管理界面
	21:    "ftp",  // FTP文件传输协议
	22:    "SSH",  // SSH安全外壳协议
	23:    "telnet",  // Telnet远程登录协议
	25:    "SMTP",  // SMTP邮件传输协议
	69:    "tftp",  // TFTP简单文件传输协议
	80:    "http",  // HTTP超文本传输协议
	88:    "kerberos",  // Kerberos网络认证协议
	109:   "pop2",  // POP2邮局协议
	110:   "pop3",  // POP3邮局协议
	123:   "ntp",  // NTP网络时间协议
# 端口号和对应的服务名称的映射关系
137:   "netbios",  # NetBIOS服务
139:   "netbios",  # NetBIOS服务
443:   "https",    # HTTPS服务
445:   "Samba",     # Samba文件共享服务
631:   "cups",      # CUPS打印服务
5800:  "VNC remote desktop",  # VNC远程桌面服务
194:   "IRC",       # IRC聊天服务
118:   "SQL service?",  # 未知的SQL服务
150:   "SQL-net?",  # 未知的SQL服务
1433:  "Microsoft SQL server",  # Microsoft SQL Server服务
1434:  "Microsoft SQL monitor",  # Microsoft SQL Monitor服务
3306:  "MySQL",    # MySQL数据库服务
3396:  "Novell NDPS Printer Agent",  # Novell NDPS打印代理服务
3535:  "SMTP (alternate)",  # 备用的SMTP服务
554:   "RTSP",     # RTSP流媒体服务
9160:  "Cassandra [ http://cassandra.apache.org/ ]",  # Cassandra数据库服务
2379:  "ETCD server port, kubernetes database",  # ETCD服务器端口，Kubernetes数据库服务
4194:  "cAdvisor, container metrics",  # cAdvisor容器监控服务
6443:  "Kubernetes API port",  # Kubernetes API端口
6666:  "ETCD server port, kubernetes database",  # ETCD服务器端口，Kubernetes数据库服务
	6782:  "weave, metrics and endpoints",
	6783:  "weave, metrics and endpoints",
	6784:  "weave, metrics and endpoints",
	8443:  "kube-apiserver, Kubernetes API port",
	8080:  "Possible Insecure API port",
	9099:  "calico-felix, Health check server for Calico",
	10250: "kubelet HTTPS API which allows full node access",
	10255: "kubelet unauthenticated read-only HTTP port: pods, runningpods and node state. Should be deprecated",
	10256: "Kube proxy health check server",
}
// 定义一个包含端口号和描述信息的映射

type PortInfo struct {
	IsPortOpen bool
	Port       int
	PortDescription string
	Err        error
}
// 定义一个结构体，用于存储端口扫描的结果信息

func scanPorts(ipAddress string, ports map[int]string, concurrencyLimit int) []PortInfo {
	// make a slice to hold the results we're expecting
	// 创建一个切片，用于存储扫描端口的结果
	// 创建一个空的开放端口信息数组
	var openPorts []PortInfo

	// 创建一个带有并发限制的缓冲通道，用于控制并发请求
	semaphoreChan := make(chan struct{}, concurrencyLimit)

	// 创建一个不会阻塞的通道，用于收集 HTTP 请求的结果
	resultsChan := make(chan *PortInfo)

	// 确保在使用完毕后关闭这些通道
	defer func() {
		close(semaphoreChan)
		close(resultsChan)
	}()

	// 遍历每个端口和端口描述
	for port, portDescription := range ports {

		// 使用闭包在一个 Go 协程中启动请求
		go func(ipAddress string, port int, portDescription string) {
// 将一个空的结构体发送到 semaphoreChan 中，这基本上是在向限制添加一个，但当达到限制时，会阻塞直到有空间
semaphoreChan <- struct{}{}

var err error
isOpen := false

// 检查指定 IP 地址和端口是否打开
isOpen, err = isPortOpen(ipAddress, port)

// 创建 PortInfo 结构体，包含 isOpen 状态、端口号、端口描述和错误信息
result := &PortInfo{isOpen, port, portDescription, err}

// 现在我们可以通过 resultsChan 发送 PortInfo 结构体
resultsChan <- result

// 一旦完成，从 semaphoreChan 中读取，这会将限制减少一个，允许另一个 goroutine 开始
<-semaphoreChan
	}

	// 开始监听结果通道上的任何结果
	// 一旦我们得到一个PortInfo，就将其附加到PortInfo切片中
	var count int
	for {
		// 从结果通道中获取结果
		result := <-resultsChan
		count += 1

		// 如果端口是打开的，则将其附加到openPorts切片中
		if result.IsPortOpen {
			openPorts = append(openPorts, *result)
		}

		// 如果我们达到了预期的URL数量，则停止循环
		if count == len(ports) {
			break
		}
	}

	// 现在我们完成了，返回结果
// 返回 openPorts 变量，这里缺少了函数名，需要补充
	return openPorts
}

// 在 JSON 中
//var arg1ArrayOfAddresses = []string{"127.0.0.1"}
//var arg1ArrayOfAddresses []string

// 主函数 mainfunc，接受一个字符串类型的参数 inputAddresses
func mainfunc(inputAddresses string) {
	// 并发限制设置为 15
	concurrencyLimit := 15
	// 打印提示信息，显示并发限制
	fmt.Printf("[*] Scanning for open ports (%d threads)\n", concurrencyLimit)
	// 程序休眠 1 秒
	time.Sleep(1)
	// 去除输入地址的空白字符
	inputAddresses = strings.TrimSpace(inputAddresses)

	// 必须先声明 inputArrayOfAddresses 变量，不能使用 ":="，因为这会导致 Yaegi 识别错误
	var inputArrayOfAddresses []string
	// 使用分号分割输入地址字符串，存入 inputArrayOfAddresses 切片中
	inputArrayOfAddresses = strings.Split(inputAddresses, ";")

	// 如果输入地址长度小于 1，则将 "127.0.0.1" 添加到 inputArrayOfAddresses 中
	if len(inputAddresses) < 1 {
		inputArrayOfAddresses = append(inputArrayOfAddresses, "127.0.0.1")
	}
```

// 将输入的地址数组赋值给args变量
args := inputArrayOfAddresses
// 遍历args数组中的每个IP地址
for _, ip := range args {
    // 打印正在扫描的IP地址
    fmt.Printf("[*] Scanning IP: %s\n", ip)
    // 检查IP地址中是否包含字母，如果包含则进行域名解析
    if strings.ContainsAny(strings.ToLower(ip), "a | b | c | d | e | f | g | h | i | j | k | l | m | n | o | p | q | r | s | t | u | v | w | x | y | z"){
        // 进行域名解析
        addr,err := net.LookupIP(ip)
        // 如果解析出错，则打印未知主机并继续下一个IP的扫描
        if err != nil {
            fmt.Println("[*] Unknown host")
            continue
        } else {
            // 如果解析成功且返回的IP地址数量大于0，则将IP地址更新为解析得到的第一个IP地址
            if len(addr) > 0 {
                ip = addr[0].String()
            }else {
                // 如果解析成功但返回的IP地址数量为0，则打印未知IP并继续下一个IP的扫描
                fmt.Println("[*] Unknown IP")
                continue
            }
        }
    }
    // 调用scanPorts函数扫描IP地址的已知端口，并限制并发数为concurrencyLimit
    portsStatus := scanPorts(ip, KNOWN_PORTS, concurrencyLimit)
}
# 遍历端口状态列表，打印开放的端口信息
for _, portInfo := range portsStatus{
    if portInfo.IsPortOpen {
        fmt.Printf("    %d: %s\n", portInfo.Port, portInfo.PortDescription)
    }
}

# 将端口状态转换为 JSON 格式的字节数据，如果出现错误则打印错误信息
jsonByteData, err := json.Marshal(portsStatus)
if err != nil {
    log.Fatal("Cannot encode to JSON ", err)
}

# 打印 JSON 格式的字节数据
fmt.Println(string(jsonByteData))
*/

/*
# 主函数入口
func main(){
    mainfunc("example.com;example.com")
}
*/
这部分代码是一个注释块的开始。
```