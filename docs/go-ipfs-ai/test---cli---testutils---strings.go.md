# `kubo\test\cli\testutils\strings.go`

```go
package testutils

import (
    "bufio"  // 导入 bufio 包，用于读取文本
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net"  // 导入 net 包，提供了用于网络通信的基本功能
    "net/netip"  // 导入 netip 子包，用于处理 IP 地址
    "net/url"  // 导入 url 包，用于解析 URL
    "strings"  // 导入 strings 包，提供了处理字符串的函数
    "sync"  // 导入 sync 包，提供了并发安全的锁
    "github.com/multiformats/go-multiaddr"  // 导入 multiaddr 包，用于处理多地址
    manet "github.com/multiformats/go-multiaddr/net"  // 导入 multiaddr/net 子包，用于处理网络地址
)

// StrCat takes a bunch of strings or string slices
// and concats them all together into one string slice.
// If an arg is not one of those types, this panics.
// If an arg is an empty string, it is dropped.
func StrCat(args ...interface{}) []string {
    res := make([]string, 0)  // 创建一个空的字符串切片
    for _, a := range args {  // 遍历参数列表
        if s, ok := a.(string); ok {  // 判断参数是否为字符串类型
            if s != "" {  // 如果字符串不为空
                res = append(res, s)  // 将字符串添加到结果切片中
            }
            continue
        }
        if ss, ok := a.([]string); ok {  // 判断参数是否为字符串切片类型
            for _, s := range ss {  // 遍历字符串切片
                if s != "" {  // 如果字符串不为空
                    res = append(res, s)  // 将字符串添加到结果切片中
                }
            }
            continue
        }
        panic(fmt.Sprintf("arg '%v' must be a string or string slice, but is '%T'", a, a))  // 抛出异常，参数类型不符合要求
    }
    return res  // 返回结果切片
}

// PreviewStr returns a preview of s, which is a prefix for logging that avoids dumping a huge string to logs.
func PreviewStr(s string) string {
    suffix := "..."  // 设置预览字符串的后缀
    previewLength := 10  // 设置预览字符串的长度
    if len(s) < previewLength {  // 如果字符串长度小于预览长度
        previewLength = len(s)  // 更新预览长度
        suffix = ""  // 清空后缀
    }
    return s[0:previewLength] + suffix  // 返回预览字符串
}

func SplitLines(s string) []string {
    var lines []string  // 创建一个空的字符串切片
    scanner := bufio.NewScanner(strings.NewReader(s))  // 创建一个字符串扫描器
    for scanner.Scan() {  // 循环扫描字符串
        lines = append(lines, scanner.Text())  // 将扫描到的字符串添加到切片中
    }
    return lines  // 返回字符串切片
}

// URLStrToMultiaddr converts a URL string like http://localhost:80 to a multiaddr.
func URLStrToMultiaddr(u string) multiaddr.Multiaddr {
    parsedURL, err := url.Parse(u)  // 解析 URL 字符串
    if err != nil {  // 如果解析出错
        panic(err)  // 抛出异常
    }
    addrPort, err := netip.ParseAddrPort(parsedURL.Host)  // 解析地址和端口
    if err != nil {  // 如果解析出错
        panic(err)  // 抛出异常
    }
    tcpAddr := net.TCPAddrFromAddrPort(addrPort)  // 创建 TCP 地址
    ma, err := manet.FromNetAddr(tcpAddr)  // 从网络地址创建多地址
    if err != nil {  // 如果创建出错
        panic(err)  // 抛出异常
    }
    return ma  // 返回多地址
}
// ForEachPar 函数并发地对切片 s 中的每个元素调用函数 f，并等待所有操作完成
func ForEachPar[T any](s []T, f func(T)) {
    // 创建一个 WaitGroup 对象，用于等待所有 goroutine 完成
    wg := sync.WaitGroup{}
    // 将 WaitGroup 的计数器设置为切片 s 的长度
    wg.Add(len(s))
    // 遍历切片 s 中的每个元素
    for _, x := range s {
        // 在新的 goroutine 中调用函数 f，并传入当前元素 x
        go func(x T) {
            // 在 goroutine 完成时，减少 WaitGroup 的计数器
            defer wg.Done()
            // 调用函数 f 处理当前元素 x
            f(x)
        }(x)
    }
    // 等待所有 goroutine 完成
    wg.Wait()
}
```