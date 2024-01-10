# `v2ray-core\common\net\address.go`

```
package net

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "net"    // 导入 net 包，用于网络操作
    "strings"  // 导入 strings 包，用于字符串操作
)

var (
    // LocalHostIP 是 IPv4 下本地主机 IP 的常量值
    LocalHostIP = IPAddress([]byte{127, 0, 0, 1})

    // AnyIP 是 IPv4 下任意 IP 的常量值
    AnyIP = IPAddress([]byte{0, 0, 0, 0})

    // LocalHostDomain 是本地主机域名的常量值
    LocalHostDomain = DomainAddress("localhost")

    // LocalHostIPv6 是 IPv6 下本地主机 IP 的常量值
    LocalHostIPv6 = IPAddress([]byte{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1})

    // AnyIPv6 是 IPv6 下任意 IP 的常量值
    AnyIPv6 = IPAddress([]byte{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0})
)

// AddressFamily 是地址的类型
type AddressFamily byte

const (
    // AddressFamilyIPv4 表示地址为 IPv4
    AddressFamilyIPv4 = AddressFamily(0)

    // AddressFamilyIPv6 表示地址为 IPv6
    AddressFamilyIPv6 = AddressFamily(1)

    // AddressFamilyDomain 表示地址为域名
    AddressFamilyDomain = AddressFamily(2)
)

// 如果当前 AddressFamily 是 IPv4，则返回 true
func (af AddressFamily) IsIPv4() bool {
    return af == AddressFamilyIPv4
}

// 如果当前 AddressFamily 是 IPv6，则返回 true
func (af AddressFamily) IsIPv6() bool {
    return af == AddressFamilyIPv6
}

// 如果当前 AddressFamily 是 IPv6 或 IPv4，则返回 true
func (af AddressFamily) IsIP() bool {
    return af == AddressFamilyIPv4 || af == AddressFamilyIPv6
}

// 如果当前 AddressFamily 是域名，则返回 true
func (af AddressFamily) IsDomain() bool {
    return af == AddressFamilyDomain
}

// Address 表示要进行通信的网络地址。它可以是 IP 地址或域名地址，但不能同时是两者。该接口不会为给定域名解析 IP 地址。
type Address interface {
    IP() net.IP     // 返回该 Address 的 IP
    Domain() string // 返回该 Address 的域名
    Family() AddressFamily  // 返回该 Address 的类型
    // 定义一个字符串类型的字段，用于表示该地址的字符串形式
    String() string // String representation of this Address
// 判断一个字节是否为字母或数字
func isAlphaNum(c byte) bool {
    return (c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z')
}

// ParseAddress函数将字符串解析为Address。当字符串为IPv4或IPv6地址时，返回值将是IPAddress，否则为DomainAddress。
func ParseAddress(addr string) Address {
    // 处理形如"[2001:4860:0:2001::68]"的IPv6地址
    lenAddr := len(addr)
    if lenAddr > 0 && addr[0] == '[' && addr[lenAddr-1] == ']' {
        addr = addr[1 : lenAddr-1]
        lenAddr -= 2
    }

    if lenAddr > 0 && (!isAlphaNum(addr[0]) || !isAlphaNum(addr[len(addr)-1])) {
        addr = strings.TrimSpace(addr)
    }

    // 将字符串解析为IP地址
    ip := net.ParseIP(addr)
    if ip != nil {
        return IPAddress(ip)
    }
    return DomainAddress(addr)
}

// 创建具有给定IP的Address
func IPAddress(ip []byte) Address {
    switch len(ip) {
    case net.IPv4len:
        var addr ipv4Address = [4]byte{ip[0], ip[1], ip[2], ip[3]}
        return addr
    case net.IPv6len:
        if bytes.Equal(ip[:10], bytes0) && ip[10] == 0xff && ip[11] == 0xff {
            return IPAddress(ip[12:16])
        }
        var addr ipv6Address = [16]byte{
            ip[0], ip[1], ip[2], ip[3],
            ip[4], ip[5], ip[6], ip[7],
            ip[8], ip[9], ip[10], ip[11],
            ip[12], ip[13], ip[14], ip[15],
        }
        return addr
    default:
        newError("invalid IP format: ", ip).AtError().WriteToLog()
        return nil
    }
}

// 创建具有给定域名的Address
func DomainAddress(domain string) Address {
    return domainAddress(domain)
}

// 定义IPv4Address类型，包含4个字节
type ipv4Address [4]byte

// 返回IPv4Address的IP
func (a ipv4Address) IP() net.IP {
    return net.IP(a[:])
}

// 在IPv4Address上调用Domain()时抛出异常
func (ipv4Address) Domain() string {
    panic("Calling Domain() on an IPv4Address.")
}

// 返回IPv4Address的地址族
func (ipv4Address) Family() AddressFamily {
    return AddressFamilyIPv4
}

// 返回IPv4Address的字符串表示形式
func (a ipv4Address) String() string {
    # 调用对象a的IP方法，获取其IP地址，并以字符串形式返回
    return a.IP().String()
// 定义一个包含16个字节的ipv6Address类型
type ipv6Address [16]byte

// 返回ipv6Address类型的IP地址
func (a ipv6Address) IP() net.IP {
    return net.IP(a[:])
}

// 返回ipv6Address类型的域名，如果调用该方法会抛出异常
func (ipv6Address) Domain() string {
    panic("Calling Domain() on an IPv6Address.")
}

// 返回ipv6Address类型的地址族
func (ipv6Address) Family() AddressFamily {
    return AddressFamilyIPv6
}

// 返回ipv6Address类型的字符串表示形式
func (a ipv6Address) String() string {
    return "[" + a.IP().String() + "]"
}

// 定义一个包含字符串的domainAddress类型
type domainAddress string

// 如果调用该方法会抛出异常，因为domainAddress类型没有IP地址
func (domainAddress) IP() net.IP {
    panic("Calling IP() on a DomainAddress.")
}

// 返回domainAddress类型的域名
func (a domainAddress) Domain() string {
    return string(a)
}

// 返回domainAddress类型的地址族
func (domainAddress) Family() AddressFamily {
    return AddressFamilyDomain
}

// 返回domainAddress类型的字符串表示形式
func (a domainAddress) String() string {
    return a.Domain()
}

// 将IPOrDomain类型转换为Address类型
func (d *IPOrDomain) AsAddress() Address {
    if d == nil {
        return nil
    }
    switch addr := d.Address.(type) {
    case *IPOrDomain_Ip:
        return IPAddress(addr.Ip)
    case *IPOrDomain_Domain:
        return DomainAddress(addr.Domain)
    }
    panic("Common|Net: Invalid address.")
}

// 将Address类型转换为IPOrDomain类型
func NewIPOrDomain(addr Address) *IPOrDomain {
    switch addr.Family() {
    case AddressFamilyDomain:
        return &IPOrDomain{
            Address: &IPOrDomain_Domain{
                Domain: addr.Domain(),
            },
        }
    case AddressFamilyIPv4, AddressFamilyIPv6:
        return &IPOrDomain{
            Address: &IPOrDomain_Ip{
                Ip: addr.IP(),
            },
        }
    default:
        panic("Unknown Address type.")
    }
}
```