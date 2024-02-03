# `v2ray-core\common\protocol\http\headers_test.go`

```go
package http_test

import (
    "bufio"  // 导入 bufio 包，用于读取数据
    "net/http"  // 导入 net/http 包，用于处理 HTTP 请求
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试用例

    "github.com/google/go-cmp/cmp"  // 导入 google/go-cmp/cmp 包，用于比较数据

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/net"  // 导入 v2ray.com/core/common/net 包
    . "v2ray.com/core/common/protocol/http"  // 导入 v2ray.com/core/common/protocol/http 包，并将其中的所有公共函数和变量导入当前命名空间
)

func TestParseXForwardedFor(t *testing.T) {
    header := http.Header{}  // 创建一个空的 HTTP 头部
    header.Add("X-Forwarded-For", "129.78.138.66, 129.78.64.103")  // 向 HTTP 头部添加 X-Forwarded-For 字段
    addrs := ParseXForwardedFor(header)  // 调用 ParseXForwardedFor 函数解析 X-Forwarded-For 字段
    if r := cmp.Diff(addrs, []net.Address{net.ParseAddress("129.78.138.66"), net.ParseAddress("129.78.64.103")}); r != "" {  // 使用 cmp.Diff 比较解析结果和预期结果
        t.Error(r)  // 如果结果不符合预期，则输出错误信息
    }
}

func TestHopByHopHeadersRemoving(t *testing.T) {
    rawRequest := `GET /pkg/net/http/ HTTP/1.1
Host: golang.org
Connection: keep-alive,Foo, Bar
Foo: foo
Bar: bar
Proxy-Connection: keep-alive
Proxy-Authenticate: abc
Accept-Encoding: gzip
Accept-Charset: ISO-8859-1,UTF-8;q=0.7,*;q=0.7
Cache-Control: no-cache
Accept-Language: de,en;q=0.7,en-us;q=0.3

`  // 创建一个原始的 HTTP 请求
    b := bufio.NewReader(strings.NewReader(rawRequest))  // 使用 bufio 包创建一个读取器
    req, err := http.ReadRequest(b)  // 读取 HTTP 请求
    common.Must(err)  // 检查是否有错误发生
    headers := []struct {
        Key   string
        Value string
    }{
        {
            Key:   "Foo",
            Value: "foo",
        },
        {
            Key:   "Bar",
            Value: "bar",
        },
        {
            Key:   "Connection",
            Value: "keep-alive,Foo, Bar",
        },
        {
            Key:   "Proxy-Connection",
            Value: "keep-alive",
        },
        {
            Key:   "Proxy-Authenticate",
            Value: "abc",
        },
    }  // 创建一个包含预期 HTTP 头部的结构体数组
    for _, header := range headers {  // 遍历预期的 HTTP 头部
        if v := req.Header.Get(header.Key); v != header.Value {  // 检查实际的 HTTP 头部是否符合预期
            t.Error("header ", header.Key, " = ", v, " want ", header.Value)  // 如果不符合预期，则输出错误信息
        }
    }

    RemoveHopByHopHeaders(req.Header)  // 调用 RemoveHopByHopHeaders 函数移除 HTTP 头部中的特定字段

    for _, header := range []string{"Connection", "Foo", "Bar", "Proxy-Connection", "Proxy-Authenticate"} {  // 遍历需要移除的 HTTP 头部字段
        if v := req.Header.Get(header); v != "" {  // 检查移除后的 HTTP 头部是否符合预期
            t.Error("header ", header, " = ", v)  // 如果不符合预期，则输出错误信息
        }
    }
}
func TestParseHost(t *testing.T) {
    // 定义测试用例
    testCases := []struct {
        RawHost     string
        DefaultPort net.Port
        Destination net.Destination
        Error       bool
    }{
        // 测试用例1
        {
            RawHost:     "v2ray.com:80",
            DefaultPort: 443,
            Destination: net.TCPDestination(net.DomainAddress("v2ray.com"), 80),
        },
        // 测试用例2
        {
            RawHost:     "tls.v2ray.com",
            DefaultPort: 443,
            Destination: net.TCPDestination(net.DomainAddress("tls.v2ray.com"), 443),
        },
        // 测试用例3
        {
            RawHost:     "[2401:1bc0:51f0:ec08::1]:80",
            DefaultPort: 443,
            Destination: net.TCPDestination(net.ParseAddress("[2401:1bc0:51f0:ec08::1]"), 80),
        },
    }

    // 遍历测试用例
    for _, testCase := range testCases {
        // 调用 ParseHost 函数进行解析
        dest, err := ParseHost(testCase.RawHost, testCase.DefaultPort)
        // 检查是否期望出现错误
        if testCase.Error {
            // 如果期望出现错误但实际没有出现错误，则输出错误信息
            if err == nil {
                t.Error("for test case: ", testCase.RawHost, " expected error, but actually nil")
            }
        } else {
            // 如果不期望出现错误，则检查解析结果是否符合预期
            if dest != testCase.Destination {
                t.Error("for test case: ", testCase.RawHost, " expected host: ", testCase.Destination.String(), " but got ", dest.String())
            }
        }
    }
}
```