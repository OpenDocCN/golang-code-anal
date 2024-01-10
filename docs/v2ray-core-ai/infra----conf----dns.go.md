# `v2ray-core\infra\conf\dns.go`

```
package conf

import (
    "encoding/json"  // 导入 JSON 编解码包
    "sort"  // 导入排序包
    "strings"  // 导入字符串处理包

    "v2ray.com/core/app/dns"  // 导入 DNS 应用包
    "v2ray.com/core/app/router"  // 导入路由应用包
    "v2ray.com/core/common/net"  // 导入网络通用包
)

type NameServerConfig struct {
    Address   *Address  // 地址结构体指针
    Port      uint16  // 端口号
    Domains   []string  // 域名列表
    ExpectIPs StringList  // 期望的 IP 地址列表
}

func (c *NameServerConfig) UnmarshalJSON(data []byte) error {
    var address Address  // 地址结构体
    if err := json.Unmarshal(data, &address); err == nil {  // 解析 JSON 数据到地址结构体
        c.Address = &address  // 将解析后的地址赋值给配置的地址
        return nil
    }

    var advanced struct {  // 高级配置结构体
        Address   *Address   `json:"address"`  // 地址结构体指针
        Port      uint16     `json:"port"`  // 端口号
        Domains   []string   `json:"domains"`  // 域名列表
        ExpectIPs StringList `json:"expectIps"`  // 期望的 IP 地址列表
    }
    if err := json.Unmarshal(data, &advanced); err == nil {  // 解析 JSON 数据到高级配置结构体
        c.Address = advanced.Address  // 将解析后的地址赋值给配置的地址
        c.Port = advanced.Port  // 将解析后的端口号赋值给配置的端口号
        c.Domains = advanced.Domains  // 将解析后的域名列表赋值给配置的域名列表
        c.ExpectIPs = advanced.ExpectIPs  // 将解析后的 IP 地址列表赋值给配置的 IP 地址列表
        return nil
    }

    return newError("failed to parse name server: ", string(data))  // 返回解析失败的错误信息
}

func toDomainMatchingType(t router.Domain_Type) dns.DomainMatchingType {
    switch t {  // 根据路由域名类型转换为 DNS 域名匹配类型
    case router.Domain_Domain:
        return dns.DomainMatchingType_Subdomain  // 子域名匹配类型
    case router.Domain_Full:
        return dns.DomainMatchingType_Full  // 完整域名匹配类型
    case router.Domain_Plain:
        return dns.DomainMatchingType_Keyword  // 关键词匹配类型
    case router.Domain_Regex:
        return dns.DomainMatchingType_Regex  // 正则表达式匹配类型
    default:
        panic("unknown domain type")  // 抛出未知域名类型的异常
    }
}

func (c *NameServerConfig) Build() (*dns.NameServer, error) {
    if c.Address == nil {  // 如果地址为空
        return nil, newError("NameServer address is not specified.")  // 返回地址未指定的错误信息
    }

    var domains []*dns.NameServer_PriorityDomain  // 域名列表
    var originalRules []*dns.NameServer_OriginalRule  // 原始规则列表
    # 遍历域名规则列表
    for _, rule := range c.Domains:
        # 解析域名规则
        parsedDomain, err := parseDomainRule(rule)
        # 如果解析出错，返回错误信息
        if err != nil:
            return nil, newError("invalid domain rule: ", rule).Base(err)

        # 遍历解析后的域名规则列表
        for _, pd := range parsedDomain:
            # 将解析后的域名规则添加到域名列表中
            domains = append(domains, &dns.NameServer_PriorityDomain{
                Type:   toDomainMatchingType(pd.Type),
                Domain: pd.Value,
            })
        # 将原始域名规则添加到原始规则列表中
        originalRules = append(originalRules, &dns.NameServer_OriginalRule{
            Rule: rule,
            Size: uint32(len(parsedDomain)),
        })

    # 将期望的 IP 地址列表转换为 CIDR 列表
    geoipList, err := toCidrList(c.ExpectIPs)
    # 如果转换出错，返回错误信息
    if err != nil:
        return nil, newError("invalid ip rule: ", c.ExpectIPs).Base(err)

    # 返回 DNS 服务器对象
    return &dns.NameServer{
        Address: &net.Endpoint{
            Network: net.Network_UDP,
            Address: c.Address.Build(),
            Port:    uint32(c.Port),
        },
        PrioritizedDomain: domains,
        Geoip:             geoipList,
        OriginalRules:     originalRules,
    }, nil
// 定义一个映射，将router.Domain_Type映射到dns.DomainMatchingType
var typeMap = map[router.Domain_Type]dns.DomainMatchingType{
    router.Domain_Full:   dns.DomainMatchingType_Full,
    router.Domain_Domain: dns.DomainMatchingType_Subdomain,
    router.Domain_Plain:  dns.DomainMatchingType_Keyword,
    router.Domain_Regex:  dns.DomainMatchingType_Regex,
}

// DnsConfig是一个可JSON序列化的对象，用于dns.Config
type DnsConfig struct {
    Servers  []*NameServerConfig `json:"servers"`  // 服务器列表
    Hosts    map[string]*Address `json:"hosts"`    // 主机地址映射
    ClientIP *Address            `json:"clientIp"` // 客户端IP地址
    Tag      string              `json:"tag"`      // 标签
}

// 根据地址构建主机映射
func getHostMapping(addr *Address) *dns.Config_HostMapping {
    if addr.Family().IsIP() {
        return &dns.Config_HostMapping{
            Ip: [][]byte{[]byte(addr.IP())},  // 如果是IP地址，返回IP地址映射
        }
    } else {
        return &dns.Config_HostMapping{
            ProxiedDomain: addr.Domain(),  // 如果是域名，返回代理域名
        }
    }
}

// 实现Buildable接口
func (c *DnsConfig) Build() (*dns.Config, error) {
    config := &dns.Config{
        Tag: c.Tag,  // 设置配置标签
    }

    if c.ClientIP != nil {
        if !c.ClientIP.Family().IsIP() {
            return nil, newError("not an IP address:", c.ClientIP.String())  // 如果不是IP地址，返回错误
        }
        config.ClientIp = []byte(c.ClientIP.IP())  // 设置客户端IP地址
    }

    for _, server := range c.Servers {
        ns, err := server.Build()  // 构建服务器
        if err != nil {
            return nil, newError("failed to build name server").Base(err)  // 如果构建失败，返回错误
        }
        config.NameServer = append(config.NameServer, ns)  // 添加到名称服务器列表
    }

    return config, nil  // 返回配置和空错误
}
```