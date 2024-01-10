# `v2ray-core\features\dns\client.go`

```
package dns

import (
    "v2ray.com/core/common/errors"  // 导入错误处理包
    "v2ray.com/core/common/net"  // 导入网络通用包
    "v2ray.com/core/common/serial"  // 导入序列化通用包
    "v2ray.com/core/features"  // 导入特性包
)

// Client is a V2Ray feature for querying DNS information.
//
// v2ray:api:stable
type Client interface {
    features.Feature  // 定义接口继承自 Feature 接口

    // LookupIP returns IP address for the given domain. IPs may contain IPv4 and/or IPv6 addresses.
    LookupIP(domain string) ([]net.IP, error)  // 查询给定域名的 IP 地址，可能包含 IPv4 和/或 IPv6 地址
}

// IPv4Lookup is an optional feature for querying IPv4 addresses only.
//
// v2ray:api:beta
type IPv4Lookup interface {
    LookupIPv4(domain string) ([]net.IP, error)  // 查询给定域名的 IPv4 地址
}

// IPv6Lookup is an optional feature for querying IPv6 addresses only.
//
// v2ray:api:beta
type IPv6Lookup interface {
    LookupIPv6(domain string) ([]net.IP, error)  // 查询给定域名的 IPv6 地址
}

// ClientType returns the type of Client interface. Can be used for implementing common.HasType.
//
// v2ray:api:beta
func ClientType() interface{} {
    return (*Client)(nil)  // 返回 Client 接口的类型
}

// ErrEmptyResponse indicates that DNS query succeeded but no answer was returned.
var ErrEmptyResponse = errors.New("empty response")  // 定义表示 DNS 查询成功但未返回答案的错误

type RCodeError uint16  // 定义 RCodeError 类型为 uint16

func (e RCodeError) Error() string {
    return serial.Concat("rcode: ", uint16(e))  // 返回 RCodeError 的错误信息
}

func RCodeFromError(err error) uint16 {
    if err == nil {
        return 0  // 如果错误为空，返回 0
    }
    cause := errors.Cause(err)  // 获取错误的原因
    if r, ok := cause.(RCodeError); ok {
        return uint16(r)  // 如果原因是 RCodeError 类型，返回其值
    }
    return 0  // 否则返回 0
}
```