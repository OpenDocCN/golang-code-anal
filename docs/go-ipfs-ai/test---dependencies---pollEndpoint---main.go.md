# `kubo\test\dependencies\pollEndpoint\main.go`

```
// pollEndpoint是一个辅助工具，用于等待http端点可达并返回http.StatusOK
package main

import (
    "context"
    "flag"
    "io"
    "net"
    "net/http"
    "os"
    "time"

    logging "github.com/ipfs/go-log"
    ma "github.com/multiformats/go-multiaddr"
    manet "github.com/multiformats/go-multiaddr/net"
)

var (
    host    = flag.String("host", "/ip4/127.0.0.1/tcp/5001", "the multiaddr host to dial on") // 定义host参数，指定要拨号的多地址主机
    tries   = flag.Int("tries", 10, "how many tries to make before failing") // 定义tries参数，指定失败前要尝试的次数
    timeout = flag.Duration("tout", time.Second, "how long to wait between attempts") // 定义timeout参数，指定尝试之间等待的时间
    httpURL = flag.String("http-url", "", "HTTP URL to fetch") // 定义httpURL参数，指定要获取的HTTP URL
    httpOut = flag.Bool("http-out", false, "Print the HTTP response body to stdout") // 定义httpOut参数，指定是否将HTTP响应主体打印到标准输出
    verbose = flag.Bool("v", false, "verbose logging") // 定义verbose参数，指定是否启用详细日志记录
)

var log = logging.Logger("pollEndpoint") // 创建日志记录器

func main() {
    flag.Parse() // 解析命令行参数

    // 从host标志提取地址
    addr, err := ma.NewMultiaddr(*host)
    if err != nil {
        log.Fatal("NewMultiaddr() failed: ", err) // 如果提取地址失败，则记录错误并退出程序
    }

    if *verbose { // 降低日志级别
        logging.SetDebugLogging()
    }

    // 显示我们得到的内容
    start := time.Now()
    log.Debugf("starting at %s, tries: %d, timeout: %s, addr: %s", start, *tries, *timeout, addr) // 记录开始时间、尝试次数、超时时间和地址

    connTries := *tries
    for connTries > 0 { // 循环尝试连接
        c, err := manet.Dial(addr) // 拨号连接
        if err == nil { // 如果连接成功
            log.Debugf("ok -  endpoint reachable with %d tries remaining, took %s", *tries, time.Since(start)) // 记录连接成功的信息
            c.Close() // 关闭连接
            break // 退出循环
        }
        log.Debug("connect failed: ", err) // 记录连接失败的信息
        time.Sleep(*timeout) // 等待一段时间后继续尝试连接
        connTries-- // 减少剩余尝试次数
    }

    if err != nil { // 如果仍然存在错误
        goto Fail // 跳转到Fail标签处
    }
}
    # 如果传入的 httpURL 不为空
    if *httpURL != "" {
        # 创建一个自定义的连接拨号器，设置地址为 addr
        dialer := &connDialer{addr: addr}
        # 创建一个 HTTP 客户端，设置传输层为自定义的拨号器
        httpClient := http.Client{Transport: &http.Transport{
            DialContext: dialer.DialContext,
        }}
        # 设置请求尝试次数为传入参数 tries 的值
        reqTries := *tries
        # 循环，直到请求尝试次数为 0
        for reqTries > 0 {
            # 计算当前尝试次数
            try := (*tries - reqTries) + 1
            # 打印调试信息，显示当前尝试的 HTTP 请求次数和 URL
            log.Debugf("trying HTTP req %d: '%s'", try, *httpURL)
            # 如果尝试发送 HTTP GET 请求成功
            if tryHTTPGet(&httpClient, *httpURL) {
                # 打印调试信息，显示当前尝试的 HTTP 请求次数和 URL，并标记成功
                log.Debugf("HTTP req %d to '%s' succeeded", try, *httpURL)
                # 跳转到 Success 标签处
                goto Success
            }
            # 打印调试信息，显示当前尝试的 HTTP 请求次数和 URL，并标记失败
            log.Debugf("HTTP req %d to '%s' failed", try, *httpURL)
            # 休眠一段时间，等待下一次尝试
            time.Sleep(*timeout)
            # 减少请求尝试次数
            reqTries--
        }
        # 跳转到 Fail 标签处
        goto Fail
    }
# 成功时退出程序，返回状态码 0
Success:
    os.Exit(0)

# 失败时记录错误信息并退出程序，返回状态码 1
Fail:
    log.Error("failed")
    os.Exit(1)
}

# 尝试使用 HTTP 客户端发送 GET 请求
func tryHTTPGet(client *http.Client, url string) bool {
    # 发送 GET 请求并获取响应
    resp, err := client.Get(*httpURL)
    # 如果响应不为空且响应体不为空，则在函数返回前关闭响应体
    if resp != nil && resp.Body != nil {
        defer resp.Body.Close()
    }
    # 如果发生错误，则返回 false
    if err != nil {
        return false
    }
    # 如果响应状态码不是 200 OK，则返回 false
    if resp.StatusCode != http.StatusOK {
        return false
    }
    # 如果需要将响应内容输出到标准输出，则将响应内容拷贝到标准输出
    if *httpOut {
        _, err := io.Copy(os.Stdout, resp.Body)
        # 如果拷贝过程中发生错误，则抛出异常
        if err != nil {
            panic(err)
        }
    }
    # 返回 true 表示请求成功
    return true
}

# 定义一个连接拨号器类型
type connDialer struct {
    addr ma.Multiaddr
}

# 实现 DialContext 方法，用于在指定上下文中拨号并返回连接
func (d connDialer) DialContext(ctx context.Context, network, addr string) (net.Conn, error) {
    return (&manet.Dialer{}).DialContext(ctx, d.addr)
}
```