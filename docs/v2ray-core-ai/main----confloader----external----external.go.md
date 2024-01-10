# `v2ray-core\main\confloader\external\external.go`

```
package external

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "io" // 导入 io 包，提供 I/O 原语的基本接口
    "io/ioutil" // 导入 ioutil 包，提供了一些实用的 I/O 函数
    "net/http" // 导入 http 包，提供 HTTP 客户端和服务端的实现
    "net/url" // 导入 url 包，用于 URL 解析
    "os" // 导入 os 包，提供了与操作系统交互的功能
    "strings" // 导入 strings 包，提供了操作字符串的函数
    "time" // 导入 time 包，提供了时间的显示和测量用的函数

    "v2ray.com/core/common/buf" // 导入 buf 包，提供了缓冲区的实现
    "v2ray.com/core/common/platform/ctlcmd" // 导入 ctlcmd 包，提供了控制命令的实现
    "v2ray.com/core/main/confloader" // 导入 confloader 包，提供了配置加载的实现
)

func ConfigLoader(arg string) (out io.Reader, err error) {
    // 定义一个变量 data，用于存储读取的数据
    var data []byte
    // 如果参数以 "http://" 或 "https://" 开头
    if strings.HasPrefix(arg, "http://") || strings.HasPrefix(arg, "https://") {
        // 从 HTTP 获取内容
        data, err = FetchHTTPContent(arg)
    } else if arg == "stdin:" {
        // 从标准输入读取内容
        data, err = ioutil.ReadAll(os.Stdin)
    } else {
        // 从文件中读取内容
        data, err = ioutil.ReadFile(arg)
    }

    // 如果读取过程中出现错误，则直接返回
    if err != nil {
        return
    }
    // 将读取的数据存储到一个新的字节流中
    out = bytes.NewBuffer(data)
    return
}

func FetchHTTPContent(target string) ([]byte, error) {
    // 解析目标 URL
    parsedTarget, err := url.Parse(target)
    if err != nil {
        return nil, newError("invalid URL: ", target).Base(err)
    }

    // 检查 URL 的 scheme 是否为 "http" 或 "https"
    if s := strings.ToLower(parsedTarget.Scheme); s != "http" && s != "https" {
        return nil, newError("invalid scheme: ", parsedTarget.Scheme)
    }

    // 创建一个 HTTP 客户端
    client := &http.Client{
        Timeout: 30 * time.Second, // 设置超时时间为 30 秒
    }
    // 发起 HTTP GET 请求
    resp, err := client.Do(&http.Request{
        Method: "GET",
        URL:    parsedTarget,
        Close:  true,
    })
    if err != nil {
        return nil, newError("failed to dial to ", target).Base(err)
    }
    defer resp.Body.Close() // 在函数返回前关闭响应体

    // 检查 HTTP 响应状态码是否为 200
    if resp.StatusCode != 200 {
        return nil, newError("unexpected HTTP status code: ", resp.StatusCode)
    }

    // 读取 HTTP 响应内容到字节流中
    content, err := buf.ReadAllToBytes(resp.Body)
    if err != nil {
        return nil, newError("failed to read HTTP response").Base(err)
    }

    return content, nil
}

func ExtConfigLoader(files []string) (io.Reader, error) {
    // 运行控制命令，获取配置信息
    buf, err := ctlcmd.Run(append([]string{"config"}, files...), os.Stdin)
    if err != nil {
        return nil, err
    }

    // 将获取的配置信息存储到一个新的字符串读取器中
    return strings.NewReader(buf.String()), nil
}

func init() {
    // 将 ConfigLoader 函数赋值给 EffectiveConfigFileLoader
    confloader.EffectiveConfigFileLoader = ConfigLoader
}
    # 将EffectiveExtConfigLoader属性设置为ExtConfigLoader类，用于加载有效的外部配置
    confloader.EffectiveExtConfigLoader = ExtConfigLoader
# 闭合前面的函数定义
```