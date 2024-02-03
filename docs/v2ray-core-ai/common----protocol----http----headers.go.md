# `v2ray-core\common\protocol\http\headers.go`

```go
package http

import (
    "net/http"
    "strconv"
    "strings"

    "v2ray.com/core/common/net"
)

// ParseXForwardedFor parses X-Forwarded-For header in http headers, and return the IP list in it.
func ParseXForwardedFor(header http.Header) []net.Address {
    // 获取 X-Forwarded-For 头部的值
    xff := header.Get("X-Forwarded-For")
    // 如果值为空，则返回空列表
    if xff == "" {
        return nil
    }
    // 以逗号分隔 X-Forwarded-For 头部的值，得到 IP 地址列表
    list := strings.Split(xff, ",")
    // 创建一个空的地址列表
    addrs := make([]net.Address, 0, len(list))
    // 遍历 IP 地址列表，将每个地址解析成 net.Address 类型，并添加到地址列表中
    for _, proxy := range list {
        addrs = append(addrs, net.ParseAddress(proxy))
    }
    // 返回地址列表
    return addrs
}

// RemoveHopByHopHeaders remove hop by hop headers in http header list.
func RemoveHopByHopHeaders(header http.Header) {
    // 根据 RFC 规范，删除 hop-by-hop 头部字段
    // 参考链接：http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.5.1
    // 参考链接：https://www.mnot.net/blog/2011/07/11/what_proxies_must_do

    // 删除指定的 hop-by-hop 头部字段
    header.Del("Proxy-Connection")
    header.Del("Proxy-Authenticate")
    header.Del("Proxy-Authorization")
    header.Del("TE")
    header.Del("Trailers")
    header.Del("Transfer-Encoding")
    header.Del("Upgrade")

    // 获取 Connection 头部的值
    connections := header.Get("Connection")
    // 删除 Connection 头部字段
    header.Del("Connection")
    // 如果 Connection 头部的值为空，则返回
    if connections == "" {
        return
    }
    // 以逗号分隔 Connection 头部的值，遍历并删除对应的头部字段
    for _, h := range strings.Split(connections, ",") {
        header.Del(strings.TrimSpace(h))
    }
}

// ParseHost splits host and port from a raw string. Default port is used when raw string doesn't contain port.
func ParseHost(rawHost string, defaultPort net.Port) (net.Destination, error) {
    // 默认端口为传入的默认端口
    port := defaultPort
    // 从原始字符串中分离主机和端口
    host, rawPort, err := net.SplitHostPort(rawHost)
    // 如果出现错误
    if err != nil {
        // 如果错误是缺少端口，则将主机设置为原始字符串
        if addrError, ok := err.(*net.AddrError); ok && strings.Contains(addrError.Err, "missing port") {
            host = rawHost
        } else {
            return net.Destination{}, err
        }
    } else if len(rawPort) > 0 {
        // 如果原始端口不为空，则将其转换为整数，并设置为端口
        intPort, err := strconv.Atoi(rawPort)
        if err != nil {
            return net.Destination{}, err
        }
        port = net.Port(intPort)
    }
    // 返回解析后的主机和端口
}
    # 返回一个TCPDestination对象，该对象表示要连接的主机和端口
    return net.TCPDestination(net.ParseAddress(host), port), nil
# 闭合前面的函数定义
```