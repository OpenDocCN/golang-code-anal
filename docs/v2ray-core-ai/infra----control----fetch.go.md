# `v2ray-core\infra\control\fetch.go`

```
package control

import (
    "net/http"  // 导入处理 HTTP 请求的包
    "net/url"   // 导入处理 URL 的包
    "os"        // 导入操作系统功能的包
    "strings"   // 导入处理字符串的包
    "time"      // 导入处理时间的包

    "v2ray.com/core/common"  // 导入自定义包
    "v2ray.com/core/common/buf"  // 导入自定义包
)

type FetchCommand struct{}  // 定义 FetchCommand 结构体

func (c *FetchCommand) Name() string {  // 定义 FetchCommand 结构体的方法 Name
    return "fetch"  // 返回字符串 "fetch"
}

func (c *FetchCommand) Description() Description {  // 定义 FetchCommand 结构体的方法 Description
    return Description{  // 返回 Description 结构体
        Short: "Fetch resources",  // 设置 Short 字段为 "Fetch resources"
        Usage: []string{"v2ctl fetch <url>"},  // 设置 Usage 字段为字符串切片
    }
}

func (c *FetchCommand) Execute(args []string) error {  // 定义 FetchCommand 结构体的方法 Execute
    if len(args) < 1 {  // 如果参数长度小于 1
        return newError("empty url")  // 返回错误信息 "empty url"
    }
    content, err := FetchHTTPContent(args[0])  // 调用 FetchHTTPContent 方法获取内容和错误
    if err != nil {  // 如果错误不为空
        return newError("failed to read HTTP response").Base(err)  // 返回错误信息 "failed to read HTTP response" 并包含原始错误
    }

    os.Stdout.Write(content)  // 将内容写入标准输出
    return nil  // 返回空值
}

// FetchHTTPContent dials https for remote content
func FetchHTTPContent(target string) ([]byte, error) {  // 定义 FetchHTTPContent 方法，参数为字符串，返回值为字节切片和错误
    parsedTarget, err := url.Parse(target)  // 解析目标 URL
    if err != nil {  // 如果有错误
        return nil, newError("invalid URL: ", target).Base(err)  // 返回错误信息 "invalid URL" 并包含原始错误
    }

    if s := strings.ToLower(parsedTarget.Scheme); s != "http" && s != "https" {  // 如果 URL 协议不是 http 或 https
        return nil, newError("invalid scheme: ", parsedTarget.Scheme)  // 返回错误信息 "invalid scheme" 并包含协议信息
    }

    client := &http.Client{  // 创建 HTTP 客户端
        Timeout: 30 * time.Second,  // 设置超时时间为 30 秒
    }
    resp, err := client.Do(&http.Request{  // 发送 HTTP 请求
        Method: "GET",  // 使用 GET 方法
        URL:    parsedTarget,  // 设置请求的 URL
        Close:  true,  // 关闭连接
    })
    if err != nil {  // 如果有错误
        return nil, newError("failed to dial to ", target).Base(err)  // 返回错误信息 "failed to dial to" 并包含原始错误
    }
    defer resp.Body.Close()  // 延迟关闭响应体

    if resp.StatusCode != 200 {  // 如果响应状态码不是 200
        return nil, newError("unexpected HTTP status code: ", resp.StatusCode)  // 返回错误信息 "unexpected HTTP status code" 并包含状态码
    }

    content, err := buf.ReadAllToBytes(resp.Body)  // 读取响应体内容
    if err != nil {  // 如果有错误
        return nil, newError("failed to read HTTP response").Base(err)  // 返回错误信息 "failed to read HTTP response" 并包含原始错误
    }

    return content, nil  // 返回内容和空值
}

func init() {  // 初始化函数
    common.Must(RegisterCommand(&FetchCommand{}))  // 注册 FetchCommand 结构体
}
```