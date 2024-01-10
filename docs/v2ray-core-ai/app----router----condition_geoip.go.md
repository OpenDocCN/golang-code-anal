# `v2ray-core\app\router\condition_geoip.go`

```
// +build !confonly
// 定义了不是 confonly 的构建标签

package router
// 声明了包名为 router

import (
    "encoding/binary"
    "sort"

    "v2ray.com/core/common/net"
)
// 导入了必要的包

type ipv6 struct {
    a uint64
    b uint64
}
// 定义了 ipv6 结构体，包含两个 uint64 类型的字段

type GeoIPMatcher struct {
    countryCode string
    ip4         []uint32
    prefix4     []uint8
    ip6         []ipv6
    prefix6     []uint8
}
// 定义了 GeoIPMatcher 结构体，包含了国家代码、IPv4 地址、IPv4 前缀、IPv6 地址和 IPv6 前缀

func normalize4(ip uint32, prefix uint8) uint32 {
    return (ip >> (32 - prefix)) << (32 - prefix)
}
// 定义了 normalize4 函数，用于对 IPv4 地址进行规范化处理

func normalize6(ip ipv6, prefix uint8) ipv6 {
    if prefix <= 64 {
        ip.a = (ip.a >> (64 - prefix)) << (64 - prefix)
        ip.b = 0
    } else {
        ip.b = (ip.b >> (128 - prefix)) << (128 - prefix)
    }
    return ip
}
// 定义了 normalize6 函数，用于对 IPv6 地址进行规范化处理

func (m *GeoIPMatcher) Init(cidrs []*CIDR) error {
    ip4Count := 0
    ip6Count := 0

    for _, cidr := range cidrs {
        ip := cidr.Ip
        switch len(ip) {
        case 4:
            ip4Count++
        case 16:
            ip6Count++
        default:
            return newError("unexpect ip length: ", len(ip))
        }
    }
    // 初始化 IPv4 和 IPv6 地址的数量

    cidrList := CIDRList(cidrs)
    sort.Sort(&cidrList)
    // 对 CIDR 列表进行排序

    m.ip4 = make([]uint32, 0, ip4Count)
    m.prefix4 = make([]uint8, 0, ip4Count)
    m.ip6 = make([]ipv6, 0, ip6Count)
    m.prefix6 = make([]uint8, 0, ip6Count)
    // 初始化 IPv4 和 IPv6 地址的切片

    for _, cidr := range cidrs {
        ip := cidr.Ip
        prefix := uint8(cidr.Prefix)
        switch len(ip) {
        case 4:
            m.ip4 = append(m.ip4, normalize4(binary.BigEndian.Uint32(ip), prefix))
            m.prefix4 = append(m.prefix4, prefix)
        case 16:
            ip6 := ipv6{
                a: binary.BigEndian.Uint64(ip[0:8]),
                b: binary.BigEndian.Uint64(ip[8:16]),
            }
            ip6 = normalize6(ip6, prefix)

            m.ip6 = append(m.ip6, ip6)
            m.prefix6 = append(m.prefix6, prefix)
        }
    }
    // 遍历 CIDR 列表，将 IPv4 和 IPv6 地址及其前缀添加到对应的切片中

    return nil
}

func (m *GeoIPMatcher) match4(ip uint32) bool {
    if len(m.ip4) == 0 {
        return false
    }

    if ip < m.ip4[0] {
        return false
    }

    size := uint32(len(m.ip4))
    // 定义了 match4 方法，用于匹配 IPv4 地址
    # 初始化左边界为0
    l := uint32(0)
    # 初始化右边界为size
    r := size
    # 当左边界小于右边界时进行循环
    for l < r {
        # 取中间值
        x := ((l + r) >> 1)
        # 如果要查找的IP小于中间值对应的IP，则将右边界移动到中间值
        if ip < m.ip4[x] {
            r = x
            continue
        }
        # 对要查找的IP进行规范化处理
        nip := normalize4(ip, m.prefix4[x])
        # 如果规范化后的IP等于中间值对应的IP，则返回true
        if nip == m.ip4[x] {
            return true
        }
        # 否则将左边界移动到中间值的下一个位置
        l = x + 1
    }
    # 返回左边界大于0且规范化后的IP等于左边界前一个位置对应的IP
    return l > 0 && normalize4(ip, m.prefix4[l-1]) == m.ip4[l-1]
// 定义一个比较函数，用于比较两个 ipv6 结构体的大小
func less6(a ipv6, b ipv6) bool {
    return a.a < b.a || (a.a == b.a && a.b < b.b)
}

// 匹配给定的 ipv6 地址是否在 GeoIP 中
func (m *GeoIPMatcher) match6(ip ipv6) bool {
    // 如果 ip6 切片为空，则返回 false
    if len(m.ip6) == 0 {
        return false
    }

    // 如果给定的 ipv6 小于 ip6 切片中的第一个元素，则返回 false
    if less6(ip, m.ip6[0]) {
        return false
    }

    // 初始化变量 size 为 ip6 切片的长度，l 为 0，r 为 size
    size := uint32(len(m.ip6))
    l := uint32(0)
    r := size
    // 循环直到 l 大于等于 r
    for l < r {
        // 计算中间值 x
        x := (l + r) / 2
        // 如果给定的 ipv6 小于 ip6 切片中的第 x 个元素，则将 r 更新为 x
        if less6(ip, m.ip6[x]) {
            r = x
            continue
        }

        // 如果给定的 ipv6 经过规范化后等于 prefix6 切片中的第 x 个元素，则返回 true
        if normalize6(ip, m.prefix6[x]) == m.ip6[x] {
            return true
        }

        // 更新 l 为 x+1
        l = x + 1
    }

    // 返回 l 大于 0 并且给定的 ipv6 经过规范化后等于 prefix6 切片中的第 l-1 个元素
    return l > 0 && normalize6(ip, m.prefix6[l-1]) == m.ip6[l-1]
}

// Match 函数根据给定的 IP 地址进行匹配，返回是否在 GeoIP 中
func (m *GeoIPMatcher) Match(ip net.IP) bool {
    switch len(ip) {
    case 4:
        // 如果 IP 地址长度为 4，则调用 match4 函数
        return m.match4(binary.BigEndian.Uint32(ip))
    case 16:
        // 如果 IP 地址长度为 16，则调用 match6 函数
        return m.match6(ipv6{
            a: binary.BigEndian.Uint64(ip[0:8]),
            b: binary.BigEndian.Uint64(ip[8:16]),
        })
    default:
        // 其他情况返回 false
        return false
    }
}

// GeoIPMatcherContainer 是 GeoIPMatcher 的容器，根据国家代码保存唯一的 GeoIPMatcher
type GeoIPMatcherContainer struct {
    matchers []*GeoIPMatcher
}

// Add 函数将新的 GeoIP 添加到容器中
// 如果 GeoIP 的国家代码不为空，GeoIPMatcherContainer 会尝试查找现有的，而不是添加新的
func (c *GeoIPMatcherContainer) Add(geoip *GeoIP) (*GeoIPMatcher, error) {
    if len(geoip.CountryCode) > 0 {
        for _, m := range c.matchers {
            if m.countryCode == geoip.CountryCode {
                return m, nil
            }
        }
    }

    m := &GeoIPMatcher{
        countryCode: geoip.CountryCode,
    }
    if err := m.Init(geoip.Cidr); err != nil {
        return nil, err
    }
    if len(geoip.CountryCode) > 0 {
        c.matchers = append(c.matchers, m)
    }
    return m, nil
}

// 定义全局变量 globalGeoIPContainer 为 GeoIPMatcherContainer 类型
var (
    globalGeoIPContainer GeoIPMatcherContainer
)
```