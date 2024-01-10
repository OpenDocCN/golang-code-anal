# `v2ray-core\infra\conf\router.go`

```
package conf

import (
    "encoding/json" // 导入 JSON 编解码包
    "strconv" // 导入字符串转换包
    "strings" // 导入字符串处理包

    "v2ray.com/core/app/router" // 导入路由器包
    "v2ray.com/core/common/net" // 导入网络通用包
    "v2ray.com/core/common/platform/filesystem" // 导入文件系统包

    "github.com/golang/protobuf/proto" // 导入协议缓冲区包
)

type RouterRulesConfig struct {
    RuleList       []json.RawMessage `json:"rules"` // 规则列表，使用原始 JSON 消息
    DomainStrategy string            `json:"domainStrategy"` // 域名策略
}

type BalancingRule struct {
    Tag       string     `json:"tag"` // 标签
    Selectors StringList `json:"selector"` // 选择器列表
}

func (r *BalancingRule) Build() (*router.BalancingRule, error) {
    if r.Tag == "" { // 如果标签为空
        return nil, newError("empty balancer tag") // 返回空，以及错误信息
    }
    if len(r.Selectors) == 0 { // 如果选择器列表为空
        return nil, newError("empty selector list") // 返回空，以及错误信息
    }

    return &router.BalancingRule{ // 返回负载均衡规则
        Tag:              r.Tag, // 标签
        OutboundSelector: []string(r.Selectors), // 出站选择器
    }, nil
}

type RouterConfig struct {
    Settings       *RouterRulesConfig `json:"settings"` // 路由器规则配置，已弃用
    RuleList       []json.RawMessage  `json:"rules"` // 规则列表，使用原始 JSON 消息
    DomainStrategy *string            `json:"domainStrategy"` // 域名策略
    Balancers      []*BalancingRule   `json:"balancers"` // 负载均衡器列表
}

func (c *RouterConfig) getDomainStrategy() router.Config_DomainStrategy {
    ds := "" // 声明空字符串
    if c.DomainStrategy != nil { // 如果域名策略不为空
        ds = *c.DomainStrategy // 获取域名策略的值
    } else if c.Settings != nil { // 否则，如果设置不为空
        ds = c.Settings.DomainStrategy // 获取设置中的域名策略
    }

    switch strings.ToLower(ds) { // 转换为小写后的域名策略
    case "alwaysip": // 如果是 alwaysip
        return router.Config_UseIp // 返回使用 IP
    case "ipifnonmatch": // 如果是 ipifnonmatch
        return router.Config_IpIfNonMatch // 返回如果非匹配使用 IP
    case "ipondemand": // 如果是 ipondemand
        return router.Config_IpOnDemand // 返回按需使用 IP
    default: // 默认
        return router.Config_AsIs // 返回原样
    }
}

func (c *RouterConfig) Build() (*router.Config, error) {
    config := new(router.Config) // 创建路由器配置对象
    config.DomainStrategy = c.getDomainStrategy() // 设置域名策略

    rawRuleList := c.RuleList // 获取原始规则列表
    if c.Settings != nil { // 如果设置不为空
        rawRuleList = append(c.RuleList, c.Settings.RuleList...) // 将规则列表和设置中的规则列表合并
    }
    # 遍历原始规则列表，每次循环将当前规则赋值给rawRule
    for _, rawRule := range rawRuleList {
        # 调用ParseRule函数，将rawRule解析为规则对象rule，并检查是否有错误
        rule, err := ParseRule(rawRule)
        # 如果有错误，返回nil和错误
        if err != nil {
            return nil, err
        }
        # 将解析后的规则对象rule添加到config.Rule中
        config.Rule = append(config.Rule, rule)
    }
    # 遍历负载均衡器列表，每次循环将当前负载均衡器赋值给rawBalancer
    for _, rawBalancer := range c.Balancers {
        # 调用Build函数，将rawBalancer构建为负载均衡器对象balancer，并检查是否有错误
        balancer, err := rawBalancer.Build()
        # 如果有错误，返回nil和错误
        if err != nil {
            return nil, err
        }
        # 将构建后的负载均衡器对象balancer添加到config.BalancingRule中
        config.BalancingRule = append(config.BalancingRule, balancer)
    }
    # 返回config和nil，表示构建配置成功
    return config, nil
}

这是一个结构体定义的结束标记。


type RouterRule struct {
    Type        string `json:"type"`  // 定义路由规则的类型
    OutboundTag string `json:"outboundTag"`  // 定义出站标签
    BalancerTag string `json:"balancerTag"`  // 定义负载均衡器标签
}


这是一个名为`RouterRule`的结构体，包含了`Type`、`OutboundTag`和`BalancerTag`三个字段，用于定义路由规则的类型、出站标签和负载均衡器标签。


func ParseIP(s string) (*router.CIDR, error) {
    var addr, mask string  // 定义地址和掩码
    i := strings.Index(s, "/")  // 查找字符串中的"/"位置
    if i < 0 {
        addr = s  // 如果找不到"/"，则整个字符串为地址
    } else {
        addr = s[:i]  // 如果找到"/"，则"/"之前为地址
        mask = s[i+1:]  // "/"之后为掩码
    }
    ip := net.ParseAddress(addr)  // 解析地址
    switch ip.Family() {  // 根据地址类型进行判断
    case net.AddressFamilyIPv4:  // IPv4地址
        bits := uint32(32)  // 默认掩码位数为32
        if len(mask) > 0 {  // 如果存在掩码
            bits64, err := strconv.ParseUint(mask, 10, 32)  // 将掩码转换为整数
            if err != nil {
                return nil, newError("invalid network mask for router: ", mask).Base(err)  // 返回错误信息
            }
            bits = uint32(bits64)  // 更新掩码位数
        }
        if bits > 32 {  // 如果掩码位数大于32
            return nil, newError("invalid network mask for router: ", bits)  // 返回错误信息
        }
        return &router.CIDR{  // 返回CIDR对象
            Ip:     []byte(ip.IP()),  // IP地址
            Prefix: bits,  // 掩码位数
        }, nil
    case net.AddressFamilyIPv6:  // IPv6地址
        bits := uint32(128)  // 默认掩码位数为128
        if len(mask) > 0 {  // 如果存在掩码
            bits64, err := strconv.ParseUint(mask, 10, 32)  // 将掩码转换为整数
            if err != nil {
                return nil, newError("invalid network mask for router: ", mask).Base(err)  // 返回错误信息
            }
            bits = uint32(bits64)  // 更新掩码位数
        }
        if bits > 128 {  // 如果掩码位数大于128
            return nil, newError("invalid network mask for router: ", bits)  // 返回错误信息
        }
        return &router.CIDR{  // 返回CIDR对象
            Ip:     []byte(ip.IP()),  // IP地址
            Prefix: bits,  // 掩码位数
        }, nil
    default:  // 不支持的地址类型
        return nil, newError("unsupported address for router: ", s)  // 返回错误信息
    }
}


这是一个名为`ParseIP`的函数，用于解析IP地址和掩码，返回CIDR对象和错误信息。


func loadGeoIP(country string) ([]*router.CIDR, error) {
    return loadIP("geoip.dat", country)  // 调用loadIP函数加载地理位置信息
}


这是一个名为`loadGeoIP`的函数，用于加载地理位置信息。


func loadIP(filename, country string) ([]*router.CIDR, error) {
    geoipBytes, err := filesystem.ReadAsset(filename)  // 读取文件内容
    if err != nil {
        return nil, newError("failed to open file: ", filename).Base(err)  // 返回错误信息
    }
    var geoipList router.GeoIPList  // 定义地理位置列表


这是一个名为`loadIP`的函数，用于加载IP地址信息。
    # 使用 proto.Unmarshal 方法将 geoipBytes 反序列化为 geoipList 对象，如果出现错误则返回错误
    if err := proto.Unmarshal(geoipBytes, &geoipList); err != nil {
        return nil, err
    }

    # 遍历 geoipList 中的每个 geoip 对象
    for _, geoip := range geoipList.Entry {
        # 如果 geoip 的 CountryCode 等于指定的 country，则返回 geoip 的 Cidr 和 nil
        if geoip.CountryCode == country {
            return geoip.Cidr, nil
        }
    }

    # 如果未找到指定的 country，则返回一个新的错误信息
    return nil, newError("country not found in ", filename, ": ", country)
}



func loadSite(filename, country string) ([]*router.Domain, error) {
    // 读取指定文件的内容
    geositeBytes, err := filesystem.ReadAsset(filename)
    // 如果读取失败，返回错误信息
    if err != nil {
        return nil, newError("failed to open file: ", filename).Base(err)
    }
    // 解析读取的内容为 GeoSiteList 结构
    var geositeList router.GeoSiteList
    if err := proto.Unmarshal(geositeBytes, &geositeList); err != nil {
        return nil, err
    }

    // 遍历 GeoSiteList 中的 Entry，找到匹配指定国家的 Domain
    for _, site := range geositeList.Entry {
        if site.CountryCode == country {
            return site.Domain, nil
        }
    }

    // 如果未找到匹配的 Domain，返回错误信息
    return nil, newError("list not found in ", filename, ": ", country)
}

type AttributeMatcher interface {
    Match(*router.Domain) bool
}

type BooleanMatcher string

func (m BooleanMatcher) Match(domain *router.Domain) bool {
    // 遍历 Domain 的 Attribute，匹配是否存在指定的属性
    for _, attr := range domain.Attribute {
        if attr.Key == string(m) {
            return true
        }
    }
    return false
}

type AttributeList struct {
    matcher []AttributeMatcher
}

func (al *AttributeList) Match(domain *router.Domain) bool {
    // 遍历 matcher 中的 AttributeMatcher，匹配 Domain 是否符合所有条件
    for _, matcher := range al.matcher {
        if !matcher.Match(domain) {
            return false
        }
    }
    return true
}

func (al *AttributeList) IsEmpty() bool {
    // 判断 AttributeList 是否为空
    return len(al.matcher) == 0
}

func parseAttrs(attrs []string) *AttributeList {
    // 解析属性列表为 AttributeList 结构
    al := new(AttributeList)
    for _, attr := range attrs {
        lc := strings.ToLower(attr)
        al.matcher = append(al.matcher, BooleanMatcher(lc))
    }
    return al
}

func loadGeositeWithAttr(file string, siteWithAttr string) ([]*router.Domain, error) {
    // 解析 siteWithAttr 字符串，获取国家和属性列表
    parts := strings.Split(siteWithAttr, "@")
    if len(parts) == 0 {
        return nil, newError("empty site")
    }
    country := strings.ToUpper(parts[0])
    attrs := parseAttrs(parts[1:])
    // 加载指定国家的 Domain 列表
    domains, err := loadSite(file, country)
    if err != nil {
        return nil, err
    }

    // 如果属性列表为空，直接返回加载的 Domain 列表
    if attrs.IsEmpty() {
        return domains, nil
    }

    // 根据属性列表过滤 Domain 列表
    filteredDomains := make([]*router.Domain, 0, len(domains))
    # 遍历 domains 列表中的每个元素，使用 _ 作为占位符表示忽略索引
    for _, domain := range domains:
        # 如果 domain 符合 attrs 的匹配条件
        if attrs.Match(domain):
            # 将符合条件的 domain 添加到 filteredDomains 列表中
            filteredDomains = append(filteredDomains, domain)
    # 返回筛选后的 domain 列表和空的错误值
    return filteredDomains, nil
    # 解析域名规则，返回域名规则列表和可能的错误
    func parseDomainRule(domain string) ([]*router.Domain, error) {
        # 如果域名以"geosite:"开头
        if strings.HasPrefix(domain, "geosite:") {
            # 获取国家信息
            country := strings.ToUpper(domain[8:])
            # 载入带有属性的geosite.dat文件
            domains, err := loadGeositeWithAttr("geosite.dat", country)
            # 如果载入失败，返回错误信息
            if err != nil {
                return nil, newError("failed to load geosite: ", country).Base(err)
            }
            # 返回载入的域名规则列表
            return domains, nil
        }
        # 初始化外部数据文件标志
        var isExtDatFile = 0
        {
            # 定义前缀
            const prefix = "ext:"
            # 如果域名以"ext:"开头，更新外部数据文件标志
            if strings.HasPrefix(domain, prefix) {
                isExtDatFile = len(prefix)
            }
            # 定义带有前缀的前缀
            const prefixQualified = "ext-domain:"
            # 如果域名以"ext-domain:"开头，更新外部数据文件标志
            if strings.HasPrefix(domain, prefixQualified) {
                isExtDatFile = len(prefixQualified)
            }
        }
        # 如果是外部数据文件
        if isExtDatFile != 0 {
            # 分割域名和国家信息
            kv := strings.Split(domain[isExtDatFile:], ":")
            # 如果分割结果不是两部分，返回错误信息
            if len(kv) != 2 {
                return nil, newError("invalid external resource: ", domain)
            }
            # 获取文件名和国家信息
            filename := kv[0]
            country := kv[1]
            # 载入带有属性的外部数据文件
            domains, err := loadGeositeWithAttr(filename, country)
            # 如果载入失败，返回错误信息
            if err != nil {
                return nil, newError("failed to load external sites: ", country, " from ", filename).Base(err)
            }
            # 返回载入的域名规则列表
            return domains, nil
        }
    
        # 创建域名规则对象
        domainRule := new(router.Domain)
        # 根据不同类型的域名规则，设置域名规则对象的类型和值
        switch {
        case strings.HasPrefix(domain, "regexp:"):
            domainRule.Type = router.Domain_Regex
            domainRule.Value = domain[7:]
        case strings.HasPrefix(domain, "domain:"):
            domainRule.Type = router.Domain_Domain
            domainRule.Value = domain[7:]
        case strings.HasPrefix(domain, "full:"):
            domainRule.Type = router.Domain_Full
            domainRule.Value = domain[5:]
        case strings.HasPrefix(domain, "keyword:"):
            domainRule.Type = router.Domain_Plain
            domainRule.Value = domain[8:]
    # 检查域名是否以"dotless:"开头
    case strings.HasPrefix(domain, "dotless:"):
        # 如果是以"dotless:"开头，则设置域名规则类型为正则表达式
        domainRule.Type = router.Domain_Regex
        # 根据不同情况设置域名规则的值
        switch substr := domain[8:]; {
        # 如果截取的子串为空，则规则值为匹配不包含点的任意字符
        case substr == "":
            domainRule.Value = "^[^.]*$"
        # 如果截取的子串不包含点，则规则值为匹配不包含点的任意字符 + 截取的子串 + 不包含点的任意字符
        case !strings.Contains(substr, "."):
            domainRule.Value = "^[^.]*" + substr + "[^.]*$"
        # 如果截取的子串包含点，则返回错误
        default:
            return nil, newError("substr in dotless rule should not contain a dot: ", substr)
        }
    # 如果域名不以"dotless:"开头
    default:
        # 设置域名规则类型为普通域名
        domainRule.Type = router.Domain_Plain
        # 设置域名规则的值为域名本身
        domainRule.Value = domain
    }
    # 返回包含域名规则的数组和空错误
    return []*router.Domain{domainRule}, nil
    // 将字符串列表转换为CIDR列表，同时返回GeoIP对象列表和错误信息
    func toCidrList(ips StringList) ([]*router.GeoIP, error) {
        // 初始化GeoIP对象列表和自定义CIDR列表
        var geoipList []*router.GeoIP
        var customCidrs []*router.CIDR

        // 遍历IP列表
        for _, ip := range ips {
            // 如果IP以"geoip:"开头，则加载对应的GeoIP数据
            if strings.HasPrefix(ip, "geoip:") {
                // 获取国家代码
                country := ip[6:]
                // 加载对应国家的GeoIP数据
                geoip, err := loadGeoIP(strings.ToUpper(country))
                if err != nil {
                    return nil, newError("failed to load GeoIP: ", country).Base(err)
                }
                // 将加载的GeoIP数据添加到GeoIP对象列表中
                geoipList = append(geoipList, &router.GeoIP{
                    CountryCode: strings.ToUpper(country),
                    Cidr:        geoip,
                })
                continue
            }
            // 判断是否为外部数据文件
            var isExtDatFile = 0
            {
                const prefix = "ext:"
                // 如果IP以"ext:"开头，则设置isExtDatFile为对应前缀的长度
                if strings.HasPrefix(ip, prefix) {
                    isExtDatFile = len(prefix)
                }
                const prefixQualified = "ext-ip:"
                // 如果IP以"ext-ip:"开头，则设置isExtDatFile为对应前缀的长度
                if strings.HasPrefix(ip, prefixQualified) {
                    isExtDatFile = len(prefixQualified)
                }
            }
            // 如果是外部数据文件
            if isExtDatFile != 0 {
                // 根据":"分割文件名和国家代码
                kv := strings.Split(ip[isExtDatFile:], ":")
                // 如果分割后的长度不为2，则返回错误
                if len(kv) != 2 {
                    return nil, newError("invalid external resource: ", ip)
                }
                // 获取文件名和国家代码
                filename := kv[0]
                country := kv[1]
                // 加载对应文件名和国家代码的IP数据
                geoip, err := loadIP(filename, strings.ToUpper(country))
                if err != nil {
                    return nil, newError("failed to load IPs: ", country, " from ", filename).Base(err)
                }
                // 将加载的IP数据添加到GeoIP对象列表中
                geoipList = append(geoipList, &router.GeoIP{
                    CountryCode: strings.ToUpper(filename + "_" + country),
                    Cidr:        geoip,
                })
                continue
            }
            // 解析IP规则
            ipRule, err := ParseIP(ip)
            if err != nil {
                return nil, newError("invalid IP: ", ip).Base(err)
            }
            // 将解析的IP规则添加到自定义CIDR列表中
            customCidrs = append(customCidrs, ipRule)
        }
        // 如果自定义CIDR列表不为空，则将其添加到GeoIP对象列表中
        if len(customCidrs) > 0 {
            geoipList = append(geoipList, &router.GeoIP{
                Cidr: customCidrs,
            })
        }
    # 返回 geoipList 列表和空值
    return geoipList, nil
// 解析字段规则，将 JSON 原始消息解析为路由规则对象
func parseFieldRule(msg json.RawMessage) (*router.RoutingRule, error) {
    // 定义原始字段规则结构体
    type RawFieldRule struct {
        RouterRule
        Domain     *StringList  `json:"domain"`
        IP         *StringList  `json:"ip"`
        Port       *PortList    `json:"port"`
        Network    *NetworkList `json:"network"`
        SourceIP   *StringList  `json:"source"`
        SourcePort *PortList    `json:"sourcePort"`
        User       *StringList  `json:"user"`
        InboundTag *StringList  `json:"inboundTag"`
        Protocols  *StringList  `json:"protocol"`
        Attributes string       `json:"attrs"`
    }
    // 创建原始字段规则对象
    rawFieldRule := new(RawFieldRule)
    // 解析 JSON 消息到原始字段规则对象
    err := json.Unmarshal(msg, rawFieldRule)
    if err != nil {
        return nil, err
    }

    // 创建路由规则对象
    rule := new(router.RoutingRule)
    // 如果出站标签不为空，则设置目标标签为出站标签
    if len(rawFieldRule.OutboundTag) > 0 {
        rule.TargetTag = &router.RoutingRule_Tag{
            Tag: rawFieldRule.OutboundTag,
        }
    } else if len(rawFieldRule.BalancerTag) > 0 {
        // 如果负载均衡标签不为空，则设置目标标签为负载均衡标签
        rule.TargetTag = &router.RoutingRule_BalancingTag{
            BalancingTag: rawFieldRule.BalancerTag,
        }
    } else {
        // 如果既没有出站标签也没有负载均衡标签，则返回错误
        return nil, newError("neither outboundTag nor balancerTag is specified in routing rule")
    }

    // 如果域名列表不为空，则解析域名规则并添加到路由规则对象中
    if rawFieldRule.Domain != nil {
        for _, domain := range *rawFieldRule.Domain {
            rules, err := parseDomainRule(domain)
            if err != nil {
                return nil, newError("failed to parse domain rule: ", domain).Base(err)
            }
            rule.Domain = append(rule.Domain, rules...)
        }
    }

    // 如果 IP 列表不为空，则转换为 CIDR 列表并设置到路由规则对象中
    if rawFieldRule.IP != nil {
        geoipList, err := toCidrList(*rawFieldRule.IP)
        if err != nil {
            return nil, err
        }
        rule.Geoip = geoipList
    }

    // 如果端口列表不为空，则构建端口列表并设置到路由规则对象中
    if rawFieldRule.Port != nil {
        rule.PortList = rawFieldRule.Port.Build()
    }

    // 如果网络列表不为空，则构建网络列表并设置到路由规则对象中
    if rawFieldRule.Network != nil {
        rule.Networks = rawFieldRule.Network.Build()
    }
    # 如果原始字段规则中的源IP不为空
    if rawFieldRule.SourceIP != nil:
        # 将源IP转换为CIDR列表
        geoipList, err := toCidrList(*rawFieldRule.SourceIP)
        # 如果转换出错，则返回错误
        if err != nil:
            return nil, err
        # 将转换后的CIDR列表赋给规则的源地理位置属性
        rule.SourceGeoip = geoipList

    # 如果原始字段规则中的源端口不为空
    if rawFieldRule.SourcePort != nil:
        # 构建源端口列表
        rule.SourcePortList = rawFieldRule.SourcePort.Build()

    # 如果原始字段规则中的用户不为空
    if rawFieldRule.User != nil:
        # 遍历用户列表，将每个用户添加到规则的用户邮箱列表中
        for _, s := range *rawFieldRule.User:
            rule.UserEmail = append(rule.UserEmail, s)

    # 如果原始字段规则中的入站标签不为空
    if rawFieldRule.InboundTag != nil:
        # 遍历入站标签列表，将每个标签添加到规则的入站标签列表中
        for _, s := range *rawFieldRule.InboundTag:
            rule.InboundTag = append(rule.InboundTag, s)

    # 如果原始字段规则中的协议不为空
    if rawFieldRule.Protocols != nil:
        # 遍历协议列表，将每个协议添加到规则的协议列表中
        for _, s := range *rawFieldRule.Protocols:
            rule.Protocol = append(rule.Protocol, s)

    # 如果原始字段规则中的属性列表长度大于0
    if len(rawFieldRule.Attributes) > 0:
        # 将属性列表赋给规则的属性
        rule.Attributes = rawFieldRule.Attributes

    # 返回规则和空错误
    return rule, nil
// 解析规则消息，返回路由规则和错误信息
func ParseRule(msg json.RawMessage) (*router.RoutingRule, error) {
    // 创建一个新的路由规则对象
    rawRule := new(RouterRule)
    // 解析 JSON 消息到路由规则对象
    err := json.Unmarshal(msg, rawRule)
    // 如果解析出错，返回错误信息
    if err != nil {
        return nil, newError("invalid router rule").Base(err)
    }
    // 如果规则类型为 "field"，解析字段规则并返回
    if rawRule.Type == "field" {
        fieldrule, err := parseFieldRule(msg)
        if err != nil {
            return nil, newError("invalid field rule").Base(err)
        }
        return fieldrule, nil
    }
    // 如果规则类型为 "chinaip"，解析中国 IP 规则并返回
    if rawRule.Type == "chinaip" {
        chinaiprule, err := parseChinaIPRule(msg)
        if err != nil {
            return nil, newError("invalid chinaip rule").Base(err)
        }
        return chinaiprule, nil
    }
    // 如果规则类型为 "chinasites"，解析中国站点规则并返回
    if rawRule.Type == "chinasites" {
        chinasitesrule, err := parseChinaSitesRule(msg)
        if err != nil {
            return nil, newError("invalid chinasites rule").Base(err)
        }
        return chinasitesrule, nil
    }
    // 如果规则类型未知，返回错误信息
    return nil, newError("unknown router rule type: ", rawRule.Type)
}

// 解析中国 IP 规则，返回路由规则和错误信息
func parseChinaIPRule(data []byte) (*router.RoutingRule, error) {
    // 创建一个新的路由规则对象
    rawRule := new(RouterRule)
    // 解析 JSON 数据到路由规则对象
    err := json.Unmarshal(data, rawRule)
    // 如果解析出错，返回错误信息
    if err != nil {
        return nil, newError("invalid router rule").Base(err)
    }
    // 加载中国 IP 地址段
    chinaIPs, err := loadGeoIP("CN")
    // 如果加载出错，返回错误信息
    if err != nil {
        return nil, newError("failed to load geoip:cn").Base(err)
    }
    // 返回中国 IP 地址段的路由规则
    return &router.RoutingRule{
        TargetTag: &router.RoutingRule_Tag{
            Tag: rawRule.OutboundTag,
        },
        Cidr: chinaIPs,
    }, nil
}

// 解析中国站点规则，返回路由规则和错误信息
func parseChinaSitesRule(data []byte) (*router.RoutingRule, error) {
    // 创建一个新的路由规则对象
    rawRule := new(RouterRule)
    // 解析 JSON 数据到路由规则对象
    err := json.Unmarshal(data, rawRule)
    // 如果解析出错，返回错误信息
    if err != nil {
        return nil, newError("invalid router rule").Base(err).AtError()
    }
    // 加载包含中国站点信息的文件
    domains, err := loadGeositeWithAttr("geosite.dat", "CN")
    // 如果加载出错，返回错误信息
    if err != nil {
        return nil, newError("failed to load geosite:cn.").Base(err)
    }
    # 返回一个路由规则对象，包含目标标签和域名列表
    return &router.RoutingRule{
        # 设置目标标签为原始规则的出站标签
        TargetTag: &router.RoutingRule_Tag{
            Tag: rawRule.OutboundTag,
        },
        # 设置域名列表为给定的 domains
        Domain: domains,
    }, nil
# 闭合前面的函数定义
```