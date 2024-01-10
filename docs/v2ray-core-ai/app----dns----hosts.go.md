# `v2ray-core\app\dns\hosts.go`

```
// +build !confonly

// 导入必要的包
package dns

import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/strmatcher"
    "v2ray.com/core/features"
)

// StaticHosts 表示 DNS 服务器中的静态域名-IP 映射
type StaticHosts struct {
    ips      [][]net.Address
    matchers *strmatcher.MatcherGroup
}

// typeMap 定义了域名匹配类型到字符串匹配类型的映射
var typeMap = map[DomainMatchingType]strmatcher.Type{
    DomainMatchingType_Full:      strmatcher.Full,
    DomainMatchingType_Subdomain: strmatcher.Domain,
    DomainMatchingType_Keyword:   strmatcher.Substr,
    DomainMatchingType_Regex:     strmatcher.Regex,
}

// toStrMatcher 将域名匹配类型和域名转换为字符串匹配器
func toStrMatcher(t DomainMatchingType, domain string) (strmatcher.Matcher, error) {
    strMType, f := typeMap[t]
    if !f {
        return nil, newError("unknown mapping type", t).AtWarning()
    }
    matcher, err := strMType.New(domain)
    if err != nil {
        return nil, newError("failed to create str matcher").Base(err)
    }
    return matcher, nil
}

// NewStaticHosts 创建一个新的 StaticHosts 实例
func NewStaticHosts(hosts []*Config_HostMapping, legacy map[string]*net.IPOrDomain) (*StaticHosts, error) {
    g := new(strmatcher.MatcherGroup)
    sh := &StaticHosts{
        ips:      make([][]net.Address, len(hosts)+len(legacy)+16),
        matchers: g,
    }

    if legacy != nil {
        // 打印废弃特性警告
        features.PrintDeprecatedFeatureWarning("simple host mapping")

        // 遍历 legacy 中的域名和 IP 映射
        for domain, ip := range legacy {
            // 创建全匹配的字符串匹配器
            matcher, err := strmatcher.Full.New(domain)
            common.Must(err)
            // 将匹配器添加到匹配器组中
            id := g.Add(matcher)

            // 将 IP 转换为地址
            address := ip.AsAddress()
            // 如果地址是域名类型，则返回错误
            if address.Family().IsDomain() {
                return nil, newError("invalid domain address in static hosts: ", address.Domain()).AtWarning()
            }

            // 将地址添加到 ips 中
            sh.ips[id] = []net.Address{address}
        }
    }
}
    // 遍历hosts映射，_为占位符，mapping为当前遍历到的值
    for _, mapping := range hosts {
        // 将mapping.Type和mapping.Domain转换为字符串匹配器
        matcher, err := toStrMatcher(mapping.Type, mapping.Domain)
        // 如果转换出错，返回错误信息
        if err != nil {
            return nil, newError("failed to create domain matcher").Base(err)
        }
        // 将matcher添加到g中，并返回其id
        id := g.Add(matcher)
        // 创建一个空的net.Address切片，容量为mapping.Ip的长度加1
        ips := make([]net.Address, 0, len(mapping.Ip)+1)
        // 如果mapping.Ip的长度大于0
        if len(mapping.Ip) > 0 {
            // 遍历mapping.Ip
            for _, ip := range mapping.Ip {
                // 将ip转换为net.IPAddress类型
                addr := net.IPAddress(ip)
                // 如果转换失败，返回错误信息
                if addr == nil {
                    return nil, newError("invalid IP address in static hosts: ", ip).AtWarning()
                }
                // 将addr添加到ips中
                ips = append(ips, addr)
            }
        } else if len(mapping.ProxiedDomain) > 0 {
            // 将mapping.ProxiedDomain转换为net.DomainAddress类型，并添加到ips中
            ips = append(ips, net.DomainAddress(mapping.ProxiedDomain))
        } else {
            // 如果mapping.Ip和mapping.ProxiedDomain都未指定，返回错误信息
            return nil, newError("neither IP address nor proxied domain specified for domain: ", mapping.Domain).AtWarning()
        }

        // 对localhost IPv6进行特殊处理，因为JSON配置只支持单个IP映射
        if len(ips) == 1 && ips[0] == net.LocalHostIP {
            // 如果ips中只有一个元素且为net.LocalHostIP，则添加net.LocalHostIPv6到ips中
            ips = append(ips, net.LocalHostIPv6)
        }

        // 将ips映射到id
        sh.ips[id] = ips
    }

    // 返回sh和nil
    return sh, nil
// 根据给定的 IP 地址列表和选项进行过滤，返回过滤后的 IP 地址列表
func filterIP(ips []net.Address, option IPOption) []net.Address {
    // 创建一个空的切片，用于存储过滤后的 IP 地址
    filtered := make([]net.Address, 0, len(ips))
    // 遍历给定的 IP 地址列表
    for _, ip := range ips {
        // 如果 IP 地址是 IPv4 并且选项中启用了 IPv4，或者 IP 地址是 IPv6 并且选项中启用了 IPv6，则将其添加到过滤后的列表中
        if (ip.Family().IsIPv4() && option.IPv4Enable) || (ip.Family().IsIPv6() && option.IPv6Enable) {
            filtered = append(filtered, ip)
        }
    }
    // 如果过滤后的列表为空，则返回空
    if len(filtered) == 0 {
        return nil
    }
    // 返回过滤后的 IP 地址列表
    return filtered
}

// 在 StaticHosts 中查找给定域名的 IP 地址，如果存在则返回 IP 地址列表
func (h *StaticHosts) LookupIP(domain string, option IPOption) []net.Address {
    // 使用匹配器在 StaticHosts 中查找域名对应的索引
    indices := h.matchers.Match(domain)
    // 如果没有找到匹配的索引，则返回空
    if len(indices) == 0 {
        return nil
    }
    // 创建一个空的 IP 地址列表
    ips := []net.Address{}
    // 遍历匹配的索引，将对应的 IP 地址添加到列表中
    for _, id := range indices {
        ips = append(ips, h.ips[id]...)
    }
    // 如果 IP 地址列表中只有一个元素且为域名类型，则直接返回该 IP 地址
    if len(ips) == 1 && ips[0].Family().IsDomain() {
        return ips
    }
    // 否则调用 filterIP 函数对 IP 地址列表进行过滤，并返回过滤后的结果
    return filterIP(ips, option)
}
```