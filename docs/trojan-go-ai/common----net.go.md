# `trojan-go\common\net.go`

```go
package common

import (
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "io/ioutil"  // 导入输入输出工具包
    "net"  // 导入网络包
    "net/http"  // 导入 HTTP 包
    "net/url"  // 导入 URL 包
    "os"  // 导入操作系统包
    "strconv"  // 导入字符串转换包
    "strings"  // 导入字符串处理包
    "time"  // 导入时间包
)

const (
    KiB = 1024  // 定义常量 KiB 为 1024
    MiB = KiB * 1024  // 定义常量 MiB 为 KiB 的 1024 倍
    GiB = MiB * 1024  // 定义常量 GiB 为 MiB 的 1024 倍
)

func HumanFriendlyTraffic(bytes uint64) string {
    if bytes <= KiB {  // 如果字节数小于等于 KiB
        return fmt.Sprintf("%d B", bytes)  // 返回格式化后的字节数和单位 B
    }
    if bytes <= MiB {  // 如果字节数小于等于 MiB
        return fmt.Sprintf("%.2f KiB", float32(bytes)/KiB)  // 返回格式化后的字节数和单位 KiB
    }
    if bytes <= GiB {  // 如果字节数小于等于 GiB
        return fmt.Sprintf("%.2f MiB", float32(bytes)/MiB)  // 返回格式化后的字节数和单位 MiB
    }
    return fmt.Sprintf("%.2f GiB", float32(bytes)/GiB)  // 返回格式化后的字节数和单位 GiB
}

func PickPort(network string, host string) int {
    switch network {  // 根据网络类型进行选择
    case "tcp":  // 如果是 TCP
        for retry := 0; retry < 16; retry++ {  // 循环 16 次
            l, err := net.Listen("tcp", host+":0")  // 监听 TCP 端口
            if err != nil {  // 如果出现错误
                continue  // 继续下一次循环
            }
            defer l.Close()  // 延迟关闭监听
            _, port, err := net.SplitHostPort(l.Addr().String())  // 分割主机和端口
            Must(err)  // 检查错误
            p, err := strconv.ParseInt(port, 10, 32)  // 解析端口
            Must(err)  // 检查错误
            return int(p)  // 返回端口
        }
    case "udp":  // 如果是 UDP
        for retry := 0; retry < 16; retry++ {  // 循环 16 次
            conn, err := net.ListenPacket("udp", host+":0")  // 监听 UDP 端口
            if err != nil {  // 如果出现错误
                continue  // 继续下一次循环
            }
            defer conn.Close()  // 延迟关闭连接
            _, port, err := net.SplitHostPort(conn.LocalAddr().String())  // 分割主机和端口
            Must(err)  // 检查错误
            p, err := strconv.ParseInt(port, 10, 32)  // 解析端口
            Must(err)  // 检查错误
            return int(p)  // 返回端口
        }
    default:  // 默认情况
        return 0  // 返回 0
    }
    return 0  // 返回 0
}

func WriteAllBytes(writer io.Writer, payload []byte) error {
    for len(payload) > 0 {  // 当载荷长度大于 0 时
        n, err := writer.Write(payload)  // 写入载荷
        if err != nil {  // 如果出现错误
            return err  // 返回错误
        }
        payload = payload[n:]  // 更新载荷
    }
    return nil  // 返回空
}

func WriteFile(path string, payload []byte) error {
    writer, err := os.Create(path)  // 创建文件
    if err != nil {  // 如果出现错误
        return err  // 返回错误
    }
    defer writer.Close()  // 延迟关闭文件

    return WriteAllBytes(writer, payload)  // 调用写入所有字节的函数
}
func FetchHTTPContent(target string) ([]byte, error) {
    // 解析目标 URL
    parsedTarget, err := url.Parse(target)
    if err != nil {
        return nil, fmt.Errorf("invalid URL: %s", target)
    }

    // 检查 URL 的 scheme 是否为 http 或 https
    if s := strings.ToLower(parsedTarget.Scheme); s != "http" && s != "https" {
        return nil, fmt.Errorf("invalid scheme: %s", parsedTarget.Scheme)
    }

    // 创建 HTTP 客户端
    client := &http.Client{
        Timeout: 30 * time.Second,
    }
    // 发起 HTTP 请求
    resp, err := client.Do(&http.Request{
        Method: "GET",
        URL:    parsedTarget,
        Close:  true,
    })
    if err != nil {
        return nil, fmt.Errorf("failed to dial to %s", target)
    }
    // 延迟关闭响应体
    defer resp.Body.Close()

    // 检查 HTTP 响应状态码
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("unexpected HTTP status code: %d", resp.StatusCode)
    }

    // 读取 HTTP 响应内容
    content, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read HTTP response")
    }

    // 返回 HTTP 响应内容
    return content, nil
}
```