# `trojan-go\tunnel\router\client.go`

```go
package router

import (
    "context"  // 上下文包，用于跟踪请求的上下文信息
    "net"  // 网络包，提供了基本的网络功能
    "regexp"  // 正则表达式包，用于处理正则表达式
    "runtime"  // 运行时包，提供了与 Go 程序运行时的交互
    "strconv"  // 字符串转换包，用于字符串和基本数据类型之间的转换
    "strings"  // 字符串包，提供了处理字符串的函数

    v2router "github.com/v2fly/v2ray-core/v4/app/router"  // V2Ray 路由包，用于处理 V2Ray 的路由功能

    "github.com/p4gefau1t/trojan-go/common"  // Trojan-Go 公共包
    "github.com/p4gefau1t/trojan-go/common/geodata"  // Trojan-Go 公共地理数据包
    "github.com/p4gefau1t/trojan-go/config"  // Trojan-Go 配置包
    "github.com/p4gefau1t/trojan-go/log"  // Trojan-Go 日志包
    "github.com/p4gefau1t/trojan-go/tunnel"  // Trojan-Go 隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // Trojan-Go 自由隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // Trojan-Go 传输隧道包
)

const (
    Block  = 0  // 常量定义：拦截
    Bypass = 1  // 常量定义：绕过
    Proxy  = 2  // 常量定义：代理
)

const (
    AsIs         = 0  // 常量定义：原样
    IPIfNonMatch = 1  // 常量定义：如果不匹配则使用 IP
    IPOnDemand   = 2  // 常量定义：按需使用 IP
)

const MaxPacketSize = 1024 * 8  // 常量定义：最大数据包大小为 1024 * 8

func matchDomain(list []*v2router.Domain, target string) bool {
    # 遍历列表中的每个元素，使用下划线 _ 忽略索引值
    for _, d := range list {
        # 根据类型进行不同的处理
        switch d.GetType() {
        # 如果类型是 Domain_Full
        case v2router.Domain_Full:
            # 获取值
            domain := d.GetValue()
            # 如果值等于目标值，记录日志并返回 true
            if domain == target {
                log.Tracef("domain %s hit domain(full) rule: %s", target, domain)
                return true
            }
        # 如果类型是 Domain_Domain
        case v2router.Domain_Domain:
            # 获取值
            domain := d.GetValue()
            # 如果目标值以该值结尾，记录日志并返回 true
            if strings.HasSuffix(target, domain) {
                idx := strings.Index(target, domain)
                if idx == 0 || target[idx-1] == '.' {
                    log.Tracef("domain %s hit domain rule: %s", target, domain)
                    return true
                }
            }
        # 如果类型是 Domain_Plain
        case v2router.Domain_Plain:
            # 包含关键字
            if strings.Contains(target, d.GetValue()) {
                log.Tracef("domain %s hit keyword rule: %s", target, d.GetValue())
                return true
            }
        # 如果类型是 Domain_Regex
        case v2router.Domain_Regex:
            # 使用正则表达式匹配目标值
            matched, err := regexp.Match(d.GetValue(), []byte(target))
            if err != nil {
                log.Error("invalid regex", d.GetValue())
                return false
            }
            if matched {
                log.Tracef("domain %s hit regex rule: %s", target, d.GetValue())
                return true
            }
        # 默认情况
        default:
            log.Debug("unknown rule type:", d.GetType().String())
        }
    }
    # 如果没有匹配的规则，返回 false
    return false
// 匹配 IP 地址是否在 CIDR 列表中
func matchIP(list []*v2router.CIDR, target net.IP) bool {
    // 判断目标 IP 地址是否为 IPv6
    isIPv6 := true
    // 默认 IP 地址长度为 IPv6 长度
    len := net.IPv6len
    // 如果目标 IP 地址为 IPv4，则长度为 IPv4 长度，且不是 IPv6
    if target.To4() != nil {
        len = net.IPv4len
        isIPv6 = false
    }
    // 遍历 CIDR 列表
    for _, c := range list {
        // 获取 CIDR 的前缀长度
        n := int(c.GetPrefix())
        // 根据前缀长度和 IP 地址长度生成掩码
        mask := net.CIDRMask(n, 8*len)
        // 获取 CIDR 的 IP 地址
        cidrIP := net.IP(c.GetIp())
        // 判断 CIDR 类型，IPv4 或 IPv6
        if cidrIP.To4() != nil { // IPv4 CIDR
            // 如果目标 IP 地址为 IPv6，则跳过
            if isIPv6 {
                continue
            }
        } else { // IPv6 CIDR
            // 如果目标 IP 地址为 IPv4，则跳过
            if !isIPv6 {
                continue
            }
        }
        // 根据 IP 地址和掩码生成子网
        subnet := &net.IPNet{IP: cidrIP.Mask(mask), Mask: mask}
        // 判断目标 IP 地址是否在子网中
        if subnet.Contains(target) {
            return true
        }
    }
    return false
}

// 创建新的 IP 地址对象
func newIPAddress(address *tunnel.Address) (*tunnel.Address, error) {
    // 解析 IP 地址
    ip, err := address.ResolveIP()
    // 如果解析失败，则返回错误
    if err != nil {
        return nil, common.NewError("router failed to resolve ip").Base(err)
    }
    // 创建新的 IP 地址对象
    newAddress := &tunnel.Address{
        IP:   ip,
        Port: address.Port,
    }
    // 判断 IP 地址类型，IPv4 或 IPv6
    if ip.To4() != nil {
        newAddress.AddressType = tunnel.IPv4
    } else {
        newAddress.AddressType = tunnel.IPv6
    }
    return newAddress, nil
}

// 客户端结构体
type Client struct {
    domains        [3][]*v2router.Domain
    cidrs          [3][]*v2router.CIDR
    defaultPolicy  int
    domainStrategy int
    underlay       tunnel.Client
    direct         *freedom.Client
    ctx            context.Context
    cancel         context.CancelFunc
}

// 路由方法，用于匹配地址并返回策略
func (c *Client) Route(address *tunnel.Address) int {
    # 如果地址类型为域名
    if address.AddressType == tunnel.DomainName:
        # 如果域名策略为按需获取IP
        if c.domainStrategy == IPOnDemand:
            # 解析域名对应的IP地址
            resolvedIP, err := newIPAddress(address)
            # 如果解析成功
            if err == nil:
                # 遍历不同类型的地址范围
                for i := Block; i <= Proxy; i++:
                    # 如果解析的IP地址匹配某个地址范围
                    if matchIP(c.cidrs[i], resolvedIP.IP):
                        # 返回该地址范围的策略
                        return i
        # 遍历不同类型的地址范围
        for i := Block; i <= Proxy; i++:
            # 如果域名匹配某个域名范围
            if matchDomain(c.domains[i], address.DomainName):
                # 返回该域名范围的策略
                return i
        # 如果域名策略为非匹配时获取IP
        if c.domainStrategy == IPIfNonMatch:
            # 解析域名对应的IP地址
            resolvedIP, err := newIPAddress(address)
            # 如果解析成功
            if err == nil:
                # 遍历不同类型的地址范围
                for i := Block; i <= Proxy; i++:
                    # 如果解析的IP地址匹配某个地址范围
                    if matchIP(c.cidrs[i], resolvedIP.IP):
                        # 返回该地址范围的策略
                        return i
    # 如果地址类型为IP
    else:
        # 遍历不同类型的地址范围
        for i := Block; i <= Proxy; i++:
            # 如果IP地址匹配某个地址范围
            if matchIP(c.cidrs[i], address.IP):
                # 返回该地址范围的策略
                return i
    # 返回默认策略
    return c.defaultPolicy
// 关闭连接的方法
func (c *Client) Close() error {
    // 取消上下文
    c.cancel()
    // 调用底层连接的关闭方法
    return c.underlay.Close()
}

// 代码信息结构体
type codeInfo struct {
    code     string
    strategy int
}

// 加载代码信息
func loadCode(cfg *Config, prefix string) []codeInfo {
    // 初始化代码信息数组
    codes := []codeInfo{}
    // 遍历代理路由配置
    for _, s := range cfg.Router.Proxy {
        // 判断是否以指定前缀开头
        if strings.HasPrefix(s, prefix) {
            // 截取前缀后的字符串
            if left := s[len(prefix):]; len(left) > 0 {
                // 添加代码信息到数组
                codes = append(codes, codeInfo{
                    code:     left,
                    strategy: Proxy,
                })
            } else {
                // 记录无效的空规则
                log.Warn("invalid empty rule:", s)
            }
        }
    }
}
    # 遍历配置中的路由器绕过规则列表
    for _, s := range cfg.Router.Bypass {
        # 如果规则以指定前缀开头
        if strings.HasPrefix(s, prefix) {
            # 获取规则中指定前缀后的部分
            if left := s[len(prefix):]; len(left) > 0 {
                # 将规则信息添加到代码信息列表中
                codes = append(codes, codeInfo{
                    code:     left,
                    strategy: Bypass,
                })
            } else {
                # 如果规则为空，则记录警告日志
                log.Warn("invalid empty rule:", s)
            }
        }
    }
    # 遍历配置中的路由器阻止规则列表
    for _, s := range cfg.Router.Block {
        # 如果规则以指定前缀开头
        if strings.HasPrefix(s, prefix) {
            # 获取规则中指定前缀后的部分
            if left := s[len(prefix):]; len(left) > 0 {
                # 将规则信息添加到代码信息列表中
                codes = append(codes, codeInfo{
                    code:     left,
                    strategy: Block,
                })
            } else {
                # 如果规则为空，则记录警告日志
                log.Warn("invalid empty rule:", s)
            }
        }
    }
    # 返回代码信息列表
    return codes
}
// NewClient 创建一个新的客户端
func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    // 定义四个内存统计变量
    m1 := runtime.MemStats{}
    m2 := runtime.MemStats{}
    m3 := runtime.MemStats{}
    m4 := runtime.MemStats{}

    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    var cancel context.CancelFunc
    // 创建一个新的带有取消功能的上下文
    ctx, cancel = context.WithCancel(ctx)

    // 创建一个直连的客户端
    direct, err := freedom.NewClient(ctx, nil)
    if err != nil {
        cancel()
        return nil, common.NewError("router failed to initialize raw client").Base(err)
    }

    // 初始化客户端对象
    client := &Client{
        domains:  [3][]*v2router.Domain{},
        cidrs:    [3][]*v2router.CIDR{},
        underlay: underlay,
        direct:   direct,
        ctx:      ctx,
        cancel:   cancel,
    }
    // 根据配置设置域名策略
    switch strings.ToLower(cfg.Router.DomainStrategy) {
    case "as_is", "as-is", "asis":
        client.domainStrategy = AsIs
    case "ip_if_non_match", "ip-if-non-match", "ipifnonmatch":
        client.domainStrategy = IPIfNonMatch
    case "ip_on_demand", "ip-on-demand", "ipondemand":
        client.domainStrategy = IPOnDemand
    default:
        return nil, common.NewError("unknown strategy: " + cfg.Router.DomainStrategy)
    }

    // 根据配置设置默认策略
    switch strings.ToLower(cfg.Router.DefaultPolicy) {
    case "proxy":
        client.defaultPolicy = Proxy
    case "bypass":
        client.defaultPolicy = Bypass
    case "block":
        client.defaultPolicy = Block
    default:
        return nil, common.NewError("unknown strategy: " + cfg.Router.DomainStrategy)
    }

    // 读取内存统计信息
    runtime.ReadMemStats(&m1)

    // 创建一个地理数据加载器
    geodataLoader := geodata.NewGeodataLoader()

    // 加载 IP 代码
    ipCode := loadCode(cfg, "geoip:")
    for _, c := range ipCode {
        code := c.code
        // 加载 IP 地址范围
        cidrs, err := geodataLoader.LoadIP(cfg.Router.GeoIPFilename, code)
        if err != nil {
            log.Error(err)
        } else {
            log.Infof("geoip:%s loaded", code)
            client.cidrs[c.strategy] = append(client.cidrs[c.strategy], cidrs...)
        }
    }

    // 读取内存统计信息
    runtime.ReadMemStats(&m2)
    // 从配置文件中加载以"geosite:"开头的代码
    siteCode := loadCode(cfg, "geosite:")
    // 遍历加载的代码
    for _, c := range siteCode {
        // 获取当前代码
        code := c.code
        // 初始化属性字符串
        attrWanted := ""
        // 检查用户是否想要具有属性的域名
        if attrIdx := strings.Index(code, "@"); attrIdx > 0 {
            // 如果代码不以"@"结尾
            if !strings.HasSuffix(code, "@") {
                // 截取属性前的代码
                code = c.code[:attrIdx]
                // 获取属性
                attrWanted = c.code[attrIdx+1:]
            } else { // "geosite:google@" 是无效的
                // 记录日志并继续下一次循环
                log.Warnf("geosite:%s invalid", code)
                continue
            }
        } else if attrIdx == 0 { // "geosite:@cn" 是无效的
            // 记录日志并继续下一次循环
            log.Warnf("geosite:%s invalid", code)
            continue
        }

        // 加载域名列表
        domainList, err := geodataLoader.LoadSite(cfg.Router.GeoSiteFilename, code)
        if err != nil {
            // 记录错误日志
            log.Error(err)
        } else {
            // 初始化找到标志
            found := false
            // 如果属性字符串不为空
            if attrWanted != "" {
                // 遍历域名列表
                for _, domain := range domainList {
                    // 遍历域名的属性
                    for _, attr := range domain.GetAttribute() {
                        // 如果属性匹配
                        if strings.EqualFold(attrWanted, attr.GetKey()) {
                            // 将域名添加到客户端的域名列表中
                            client.domains[c.strategy] = append(client.domains[c.strategy], domain)
                            // 设置找到标志为true
                            found = true
                        }
                    }
                }
            } else {
                // 将整个域名列表添加到客户端的域名列表中
                client.domains[c.strategy] = append(client.domains[c.strategy], domainList...)
                // 设置找到标志为true
                found = true
            }
            // 如果找到了匹配的域名
            if found {
                // 记录信息日志
                log.Infof("geosite:%s loaded", c.code)
            } else {
                // 记录错误日志
                log.Errorf("geosite:%s not found", c.code)
            }
        }
    }

    // 读取内存统计信息
    runtime.ReadMemStats(&m3)

    // 加载以"domain:"开头的代码
    domainInfo := loadCode(cfg, "domain:")
    // 遍历加载的代码
    for _, info := range domainInfo {
        // 将域名信息添加到客户端的域名列表中
        client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
            Type:      v2router.Domain_Domain,
            Value:     strings.ToLower(info.code),
            Attribute: nil,
        })
    }
    // 从配置文件中加载以 "keyword:" 开头的代码信息
    keywordInfo := loadCode(cfg, "keyword:")
    // 遍历关键词信息列表
    for _, info := range keywordInfo {
        // 将关键词信息添加到对应策略的域名列表中
        client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
            Type:      v2router.Domain_Plain,
            Value:     strings.ToLower(info.code),
            Attribute: nil,
        })
    }

    // 从配置文件中加载以 "regex:" 开头的代码信息
    regexInfo := loadCode(cfg, "regex:")
    // 遍历正则表达式信息列表
    for _, info := range regexInfo {
        // 编译正则表达式，如果出错则返回错误
        if _, err := regexp.Compile(info.code); err != nil {
            return nil, common.NewError("invalid regular expression: " + info.code).Base(err)
        }
        // 将正则表达式信息添加到对应策略的域名列表中
        client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
            Type:      v2router.Domain_Regex,
            Value:     info.code,
            Attribute: nil,
        })
    }

    // 从配置文件中加载以 "regexp:" 开头的代码信息，用于兼容 V2Ray 规则类型 `regexp`
    regexpInfo := loadCode(cfg, "regexp:")
    // 遍历正则表达式信息列表
    for _, info := range regexpInfo {
        // 编译正则表达式，如果出错则返回错误
        if _, err := regexp.Compile(info.code); err != nil {
            return nil, common.NewError("invalid regular expression: " + info.code).Base(err)
        }
        // 将正则表达式信息添加到对应策略的域名列表中
        client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
            Type:      v2router.Domain_Regex,
            Value:     info.code,
            Attribute: nil,
        })
    }

    // 从配置文件中加载以 "full:" 开头的代码信息
    fullInfo := loadCode(cfg, "full:")
    // 遍历完整域名信息列表
    for _, info := range fullInfo {
        // 将完整域名信息添加到对应策略的域名列表中
        client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
            Type:      v2router.Domain_Full,
            Value:     strings.ToLower(info.code),
            Attribute: nil,
        })
    }

    // 从配置文件中加载以 "cidr:" 开头的代码信息
    cidrInfo := loadCode(cfg, "cidr:")
    // 遍历 cidrInfo 数组
    for _, info := range cidrInfo {
        // 使用 "/" 分割 info.code 字符串
        tmp := strings.Split(info.code, "/")
        // 如果分割结果不是两个部分，返回错误
        if len(tmp) != 2 {
            return nil, common.NewError("invalid cidr: " + info.code)
        }
        // 解析 IP 地址
        ip := net.ParseIP(tmp[0])
        // 如果 IP 地址无效，返回错误
        if ip == nil {
            return nil, common.NewError("invalid cidr ip: " + info.code)
        }
        // 解析前缀
        prefix, err := strconv.ParseInt(tmp[1], 10, 32)
        // 如果解析失败，返回错误
        if err != nil {
            return nil, common.NewError("invalid prefix").Base(err)
        }
        // 将 IP 地址和前缀添加到 client.cidrs[info.strategy] 数组中
        client.cidrs[info.strategy] = append(client.cidrs[info.strategy], &v2router.CIDR{
            Ip:     ip,
            Prefix: uint32(prefix),
        })
    }

    // 输出日志信息
    log.Info("router client created")

    // 读取内存统计信息
    runtime.ReadMemStats(&m4)

    // 输出调试日志信息
    log.Debugf("GeoIP rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m2.Alloc-m1.Alloc), common.HumanFriendlyTraffic(m2.TotalAlloc-m1.TotalAlloc))
    log.Debugf("GeoSite rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m3.Alloc-m2.Alloc), common.HumanFriendlyTraffic(m3.TotalAlloc-m2.TotalAlloc))
    log.Debugf("Plaintext rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m4.Alloc-m3.Alloc), common.HumanFriendlyTraffic(m4.TotalAlloc-m3.TotalAlloc))
    log.Debugf("Total(router) -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m4.Alloc-m1.Alloc), common.HumanFriendlyTraffic(m4.TotalAlloc-m1.TotalAlloc))

    // 返回 client 对象和空错误
    return client, nil
# 闭合前面的函数定义
```