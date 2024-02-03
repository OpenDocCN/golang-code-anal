# `v2ray-core\app\dns\server.go`

```go
// +build !confonly
// 该代码块表示不仅仅是配置文件

package dns
// 导入所需的包
import (
    "context"
    "fmt"
    "log"
    "net/url"
    "strings"
    "sync"
    "time"

    "v2ray.com/core"
    "v2ray.com/core/app/router"
    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/common/strmatcher"
    "v2ray.com/core/common/uuid"
    "v2ray.com/core/features"
    "v2ray.com/core/features/dns"
    "v2ray.com/core/features/routing"
)

// Server is a DNS rely server.
// Server 是一个 DNS 依赖服务器
type Server struct {
    sync.Mutex
    hosts         *StaticHosts
    clientIP      net.IP
    clients       []Client             // clientIdx -> Client
    ipIndexMap    []*MultiGeoIPMatcher // clientIdx -> *MultiGeoIPMatcher
    domainRules   [][]string           // clientIdx -> domainRuleIdx -> DomainRule
    domainMatcher strmatcher.IndexMatcher
    matcherInfos  []DomainMatcherInfo // matcherIdx -> DomainMatcherInfo
    tag           string
}

// DomainMatcherInfo contains information attached to index returned by Server.domainMatcher
// DomainMatcherInfo 包含附加到 Server.domainMatcher 返回的索引的信息
type DomainMatcherInfo struct {
    clientIdx     uint16
    domainRuleIdx uint16
}

// MultiGeoIPMatcher for match
// MultiGeoIPMatcher 用于匹配
type MultiGeoIPMatcher struct {
    matchers []*router.GeoIPMatcher
}

var errExpectedIPNonMatch = errors.New("expectIPs not match")

// Match check ip match
// Match 检查 IP 是否匹配
func (c *MultiGeoIPMatcher) Match(ip net.IP) bool {
    for _, matcher := range c.matchers {
        if matcher.Match(ip) {
            return true
        }
    }
    return false
}

// HasMatcher check has matcher
// HasMatcher 检查是否有匹配器
func (c *MultiGeoIPMatcher) HasMatcher() bool {
    return len(c.matchers) > 0
}

// 生成随机标签
func generateRandomTag() string {
    id := uuid.New()
    return "v2ray.system." + id.String()
}

// New creates a new DNS server with given configuration.
// New 使用给定的配置创建一个新的 DNS 服务器
func New(ctx context.Context, config *Config) (*Server, error) {
    // 创建一个服务器对象，初始化客户端列表为空，设置标签为配置文件中的标签
    server := &Server{
        clients: make([]Client, 0, len(config.NameServers)+len(config.NameServer)),
        tag:     config.Tag,
    }
    // 如果服务器标签为空，则生成一个随机标签
    if server.tag == "" {
        server.tag = generateRandomTag()
    }
    // 如果配置文件中有客户端 IP 地址，则设置服务器的客户端 IP 地址
    if len(config.ClientIp) > 0 {
        // 如果客户端 IP 地址长度不符合 IPv4 或 IPv6 的长度，则返回错误
        if len(config.ClientIp) != net.IPv4len && len(config.ClientIp) != net.IPv6len {
            return nil, newError("unexpected IP length", len(config.ClientIp))
        }
        server.clientIP = net.IP(config.ClientIp)
    }

    // 根据配置文件中的静态主机和主机列表创建静态主机对象
    hosts, err := NewStaticHosts(config.StaticHosts, config.Hosts)
    if err != nil {
        return nil, newError("failed to create hosts").Base(err)
    }
    server.hosts = hosts

    // 如果配置文件中有名称服务器，则打印警告信息，并为每个名称服务器添加一个名称服务器对象
    if len(config.NameServers) > 0 {
        features.PrintDeprecatedFeatureWarning("simple DNS server")
        for _, destPB := range config.NameServers {
            addNameServer(&NameServer{Address: destPB})
        }
    }

    // 如果服务器的客户端列表为空，则添加一个本地名称服务器，并更新 IP 索引映射
    if len(server.clients) == 0 {
        server.clients = append(server.clients, NewLocalNameServer())
        server.ipIndexMap = append(server.ipIndexMap, nil)
    }

    // 返回服务器对象和空错误
    return server, nil
// Type 方法实现了 common.HasType 接口，返回 DNS 客户端类型
func (*Server) Type() interface{} {
    return dns.ClientType()
}

// Start 方法实现了 common.Runnable 接口，启动服务器
func (s *Server) Start() error {
    return nil
}

// Close 方法实现了 common.Closable 接口，关闭服务器
func (s *Server) Close() error {
    return nil
}

// IsOwnLink 方法检查是否是自己的链接
func (s *Server) IsOwnLink(ctx context.Context) bool {
    inbound := session.InboundFromContext(ctx)
    return inbound != nil && inbound.Tag == s.tag
}

// Match 方法检查 DNS IP 是否匹配地理位置
func (s *Server) Match(idx int, client Client, domain string, ips []net.IP) ([]net.IP, error) {
    // ...
}

// queryIPTimeout 方法查询 IP 超时
func (s *Server) queryIPTimeout(idx int, client Client, domain string, option IPOption) ([]net.IP, error) {
    // ...
}

// LookupIP 方法实现了 dns.Client 接口，查询 IP 地址
func (s *Server) LookupIP(domain string) ([]net.IP, error) {
    return s.lookupIPInternal(domain, IPOption{
        IPv4Enable: true,
        IPv6Enable: true,
    })
}
// 实现 dns.IPv4Lookup 接口，根据域名查询 IPv4 地址
func (s *Server) LookupIPv4(domain string) ([]net.IP, error) {
    // 调用内部方法进行 IP 查询，启用 IPv4，禁用 IPv6
    return s.lookupIPInternal(domain, IPOption{
        IPv4Enable: true,
        IPv6Enable: false,
    })
}

// 实现 dns.IPv6Lookup 接口，根据域名查询 IPv6 地址
func (s *Server) LookupIPv6(domain string) ([]net.IP, error) {
    // 调用内部方法进行 IP 查询，禁用 IPv4，启用 IPv6
    return s.lookupIPInternal(domain, IPOption{
        IPv4Enable: false,
        IPv6Enable: true,
    })
}

// 查询静态 IP 地址
func (s *Server) lookupStatic(domain string, option IPOption, depth int32) []net.Address {
    // 调用 hosts.LookupIP 方法查询 IP 地址
    ips := s.hosts.LookupIP(domain, option)
    // 如果查询结果为空，则返回空
    if ips == nil {
        return nil
    }
    // 如果查询结果为域名类型且深度小于 5，则递归查询
    if ips[0].Family().IsDomain() && depth < 5 {
        if newIPs := s.lookupStatic(ips[0].Domain(), option, depth+1); newIPs != nil {
            return newIPs
        }
    }
    return ips
}

// 将 net.Address 转换为 net.IP
func toNetIP(ips []net.Address) []net.IP {
    if len(ips) == 0 {
        return nil
    }
    netips := make([]net.IP, 0, len(ips))
    for _, ip := range ips {
        netips = append(netips, ip.IP())
    }
    return netips
}

// 内部方法，根据域名查询 IP 地址
func (s *Server) lookupIPInternal(domain string, option IPOption) ([]net.IP, error) {
    // 如果域名为空，则返回错误
    if domain == "" {
        return nil, newError("empty domain name")
    }

    // 规范化 FQDN 形式的查询
    if domain[len(domain)-1] == '.' {
        domain = domain[:len(domain)-1]
    }

    // 查询静态 IP 地址
    ips := s.lookupStatic(domain, option, 0)
    // 如果查询结果不为空且为 IP 类型，则返回查询结果
    if ips != nil && ips[0].Family().IsIP() {
        newError("returning ", len(ips), " IPs for domain ", domain).WriteToLog()
        return toNetIP(ips), nil
    }

    // 如果查询结果不为空且为域名类型，则替换域名并返回查询结果
    if ips != nil && ips[0].Family().IsDomain() {
        newdomain := ips[0].Domain()
        newError("domain replaced: ", domain, " -> ", newdomain).WriteToLog()
        domain = newdomain
    }

    var lastErr error
    var matchedClient Client
}
    # 如果域名匹配器不为空
    if s.domainMatcher != nil:
        # 获取匹配的索引
        indices := s.domainMatcher.Match(domain)
        # 初始化域名规则和匹配的 DNS 列表
        domainRules := []string{}
        matchingDNS := []string{}
        # 遍历匹配的索引
        for _, idx := range indices:
            # 获取匹配信息
            info := s.matcherInfos[idx]
            # 获取域名规则
            rule := s.domainRules[info.clientIdx][info.domainRuleIdx]
            # 将域名规则和客户端索引格式化后添加到域名规则列表
            domainRules = append(domainRules, fmt.Sprintf("%s(DNS idx:%d)", rule, info.clientIdx))
            # 将匹配的 DNS 名称添加到匹配的 DNS 列表
            matchingDNS = append(matchingDNS, s.clients[info.clientIdx].Name())
        # 如果存在匹配的域名规则
        if len(domainRules) > 0:
            # 输出匹配的域名规则到日志
            newError("domain ", domain, " matches following rules: ", domainRules).AtDebug().WriteToLog()
        # 如果存在匹配的 DNS
        if len(matchingDNS) > 0:
            # 输出使用的 DNS 到日志
            newError("domain ", domain, " uses following DNS first: ", matchingDNS).AtDebug().WriteToLog()
        # 遍历匹配的索引
        for _, idx := range indices:
            # 获取匹配的客户端索引
            clientIdx := int(s.matcherInfos[idx].clientIdx)
            # 获取匹配的客户端
            matchedClient = s.clients[clientIdx]
            # 查询域名对应的 IP 地址
            ips, err := s.queryIPTimeout(clientIdx, matchedClient, domain, option)
            # 如果存在 IP 地址，则返回
            if len(ips) > 0:
                return ips, nil
            # 如果返回的错误是空响应，则返回空和错误
            if err == dns.ErrEmptyResponse:
                return nil, err
            # 如果存在其他错误
            if err != nil:
                # 输出错误信息到日志
                newError("failed to lookup ip for domain ", domain, " at server ", matchedClient.Name()).Base(err).WriteToLog()
                # 记录最后一个错误
                lastErr = err
    }
    # 遍历服务器列表中的客户端，获取索引和客户端对象
    for idx, client := range s.clients {
        # 如果客户端对象等于匹配的客户端对象，则记录日志并继续下一次循环
        if client == matchedClient {
            newError("domain ", domain, " at server ", client.Name(), " idx:", idx, " already lookup failed, just ignore").AtDebug().WriteToLog()
            continue
        }

        # 查询指定客户端的域名对应的 IP 地址，获取结果和可能的错误
        ips, err := s.queryIPTimeout(idx, client, domain, option)
        # 如果获取到 IP 地址，则返回结果和空错误
        if len(ips) > 0 {
            return ips, nil
        }

        # 如果出现错误，则记录日志并保存错误信息
        if err != nil {
            newError("failed to lookup ip for domain ", domain, " at server ", client.Name()).Base(err).WriteToLog()
            lastErr = err
        }
        # 如果错误不是被取消的、超时的，以及预期的 IP 不匹配错误，则返回空结果和当前错误
        if err != context.Canceled && err != context.DeadlineExceeded && err != errExpectedIPNonMatch {
            return nil, err
        }
    }

    # 如果所有客户端都无法获取到 IP 地址，则返回空结果和最后一个错误
    return nil, newError("returning nil for domain ", domain).Base(lastErr)
# 初始化函数，用于注册配置和创建新的实例
func init() {
    # 注册配置并创建新的实例
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))
    }))
}
```