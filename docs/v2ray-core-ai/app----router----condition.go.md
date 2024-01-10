# `v2ray-core\app\router\condition.go`

```
// +build !confonly
// 声明当前文件不仅仅是用于配置

package router
// 声明包名为router

import (
    "strings"
    // 导入strings包

    "go.starlark.net/starlark"
    "go.starlark.net/syntax"
    // 导入starlark包

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/strmatcher"
    "v2ray.com/core/features/routing"
    // 导入v2ray核心库中的net、strmatcher和routing包
)

type Condition interface {
    Apply(ctx routing.Context) bool
}
// 定义Condition接口，包含Apply方法

type ConditionChan []Condition
// 定义ConditionChan类型为Condition接口的切片

func NewConditionChan() *ConditionChan {
    var condChan ConditionChan = make([]Condition, 0, 8)
    return &condChan
}
// 创建并返回一个新的ConditionChan对象

func (v *ConditionChan) Add(cond Condition) *ConditionChan {
    *v = append(*v, cond)
    return v
}
// 向ConditionChan对象中添加条件

// Apply applies all conditions registered in this chan.
func (v *ConditionChan) Apply(ctx routing.Context) bool {
    for _, cond := range *v {
        if !cond.Apply(ctx) {
            return false
        }
    }
    return true
}
// 应用ConditionChan中注册的所有条件

func (v *ConditionChan) Len() int {
    return len(*v)
}
// 返回ConditionChan中条件的数量

var matcherTypeMap = map[Domain_Type]strmatcher.Type{
    Domain_Plain:  strmatcher.Substr,
    Domain_Regex:  strmatcher.Regex,
    Domain_Domain: strmatcher.Domain,
    Domain_Full:   strmatcher.Full,
}
// 定义matcherTypeMap，将Domain_Type映射到strmatcher.Type

func domainToMatcher(domain *Domain) (strmatcher.Matcher, error) {
    matcherType, f := matcherTypeMap[domain.Type]
    if !f {
        return nil, newError("unsupported domain type", domain.Type)
    }

    matcher, err := matcherType.New(domain.Value)
    if err != nil {
        return nil, newError("failed to create domain matcher").Base(err)
    }

    return matcher, nil
}
// 将Domain转换为Matcher

type DomainMatcher struct {
    matchers strmatcher.IndexMatcher
}
// 定义DomainMatcher结构体，包含matchers字段

func NewDomainMatcher(domains []*Domain) (*DomainMatcher, error) {
    g := new(strmatcher.MatcherGroup)
    for _, d := range domains {
        m, err := domainToMatcher(d)
        if err != nil {
            return nil, err
        }
        g.Add(m)
    }

    return &DomainMatcher{
        matchers: g,
    }, nil
}
// 创建并返回一个新的DomainMatcher对象

func (m *DomainMatcher) ApplyDomain(domain string) bool {
    return len(m.matchers.Match(domain)) > 0
}
// 应用DomainMatcher对象到指定的域名

// Apply implements Condition.
// Apply 方法用于应用域名匹配器，根据上下文中的目标域名进行匹配
func (m *DomainMatcher) Apply(ctx routing.Context) bool {
    // 获取上下文中的目标域名
    domain := ctx.GetTargetDomain()
    // 如果域名长度为0，则返回false
    if len(domain) == 0 {
        return false
    }
    // 调用 ApplyDomain 方法进行域名匹配
    return m.ApplyDomain(domain)
}

// MultiGeoIPMatcher 结构体用于多个地理位置匹配器
type MultiGeoIPMatcher struct {
    matchers []*GeoIPMatcher
    onSource bool
}

// NewMultiGeoIPMatcher 用于创建一个新的多地理位置匹配器
func NewMultiGeoIPMatcher(geoips []*GeoIP, onSource bool) (*MultiGeoIPMatcher, error) {
    var matchers []*GeoIPMatcher
    // 遍历地理位置列表，添加到匹配器中
    for _, geoip := range geoips {
        matcher, err := globalGeoIPContainer.Add(geoip)
        if err != nil {
            return nil, err
        }
        matchers = append(matchers, matcher)
    }
    // 创建并返回多地理位置匹配器
    matcher := &MultiGeoIPMatcher{
        matchers: matchers,
        onSource: onSource,
    }
    return matcher, nil
}

// Apply 方法用于应用多地理位置匹配器，根据上下文中的IP地址进行匹配
func (m *MultiGeoIPMatcher) Apply(ctx routing.Context) bool {
    var ips []net.IP
    // 根据 onSource 决定获取源IP地址还是目标IP地址
    if m.onSource {
        ips = ctx.GetSourceIPs()
    } else {
        ips = ctx.GetTargetIPs()
    }
    // 遍历IP地址列表，使用地理位置匹配器进行匹配
    for _, ip := range ips {
        for _, matcher := range m.matchers {
            if matcher.Match(ip) {
                return true
            }
        }
    }
    return false
}

// PortMatcher 结构体用于端口匹配器
type PortMatcher struct {
    port     net.MemoryPortList
    onSource bool
}

// NewPortMatcher 用于创建一个新的端口匹配器，可以匹配源端口或目标端口
func NewPortMatcher(list *net.PortList, onSource bool) *PortMatcher {
    return &PortMatcher{
        port:     net.PortListFromProto(list),
        onSource: onSource,
    }
}

// Apply 方法用于应用端口匹配器，根据上下文中的源或目标端口进行匹配
func (v *PortMatcher) Apply(ctx routing.Context) bool {
    // 根据 onSource 决定匹配源端口还是目标端口
    if v.onSource {
        return v.port.Contains(ctx.GetSourcePort())
    } else {
        return v.port.Contains(ctx.GetTargetPort())
    }
}

// NetworkMatcher 结构体用于网络匹配器
type NetworkMatcher struct {
    list [8]bool
}

// NewNetworkMatcher 用于创建一个新的网络匹配器
func NewNetworkMatcher(network []net.Network) NetworkMatcher {
    var matcher NetworkMatcher
    // 遍历网络列表，将对应位置置为true
    for _, n := range network {
        matcher.list[int(n)] = true
    }
    return matcher
}
// Apply 方法实现了 Condition 接口，用于 NetworkMatcher 结构体，根据上下文中的网络信息判断是否匹配
func (v NetworkMatcher) Apply(ctx routing.Context) bool {
    return v.list[int(ctx.GetNetwork())]
}

// UserMatcher 结构体，用于存储用户信息
type UserMatcher struct {
    user []string
}

// NewUserMatcher 用于创建 UserMatcher 结构体实例，根据传入的用户信息切片
func NewUserMatcher(users []string) *UserMatcher {
    usersCopy := make([]string, 0, len(users))
    for _, user := range users {
        if len(user) > 0 {
            usersCopy = append(usersCopy, user)
        }
    }
    return &UserMatcher{
        user: usersCopy,
    }
}

// Apply 方法实现了 Condition 接口，用于 UserMatcher 结构体，根据上下文中的用户信息判断是否匹配
func (v *UserMatcher) Apply(ctx routing.Context) bool {
    user := ctx.GetUser()
    if len(user) == 0 {
        return false
    }
    for _, u := range v.user {
        if u == user {
            return true
        }
    }
    return false
}

// InboundTagMatcher 结构体，用于存储入站标签信息
type InboundTagMatcher struct {
    tags []string
}

// NewInboundTagMatcher 用于创建 InboundTagMatcher 结构体实例，根据传入的入站标签信息切片
func NewInboundTagMatcher(tags []string) *InboundTagMatcher {
    tagsCopy := make([]string, 0, len(tags))
    for _, tag := range tags {
        if len(tag) > 0 {
            tagsCopy = append(tagsCopy, tag)
        }
    }
    return &InboundTagMatcher{
        tags: tagsCopy,
    }
}

// Apply 方法实现了 Condition 接口，用于 InboundTagMatcher 结构体，根据上下文中的入站标签信息判断是否匹配
func (v *InboundTagMatcher) Apply(ctx routing.Context) bool {
    tag := ctx.GetInboundTag()
    if len(tag) == 0 {
        return false
    }
    for _, t := range v.tags {
        if t == tag {
            return true
        }
    }
    return false
}

// ProtocolMatcher 结构体，用于存储协议信息
type ProtocolMatcher struct {
    protocols []string
}

// NewProtocolMatcher 用于创建 ProtocolMatcher 结构体实例，根据传入的协议信息切片
func NewProtocolMatcher(protocols []string) *ProtocolMatcher {
    pCopy := make([]string, 0, len(protocols))

    for _, p := range protocols {
        if len(p) > 0 {
            pCopy = append(pCopy, p)
        }
    }

    return &ProtocolMatcher{
        protocols: pCopy,
    }
}

// Apply 方法实现了 Condition 接口，用于 ProtocolMatcher 结构体，根据上下文中的协议信息判断是否匹配
func (m *ProtocolMatcher) Apply(ctx routing.Context) bool {
    protocol := ctx.GetProtocol()
    if len(protocol) == 0 {
        return false
    }
    for _, p := range m.protocols {
        if strings.HasPrefix(protocol, p) {
            return true
        }
    }
    return false
}
# 定义属性匹配器结构体
type AttributeMatcher struct {
    program *starlark.Program
}

# 创建新的属性匹配器实例
func NewAttributeMatcher(code string) (*AttributeMatcher, error) {
    # 解析 Starlark 代码并生成语法树
    starFile, err := syntax.Parse("attr.star", "satisfied=("+code+")", 0)
    if err != nil {
        return nil, newError("attr rule").Base(err)
    }
    # 根据语法树生成 Starlark 程序
    p, err := starlark.FileProgram(starFile, func(name string) bool {
        return name == "attrs"
    })
    if err != nil {
        return nil, err
    }
    # 返回属性匹配器实例
    return &AttributeMatcher{
        program: p,
    }, nil
}

# 实现属性匹配
func (m *AttributeMatcher) Match(attrs map[string]string) bool {
    # 创建 Starlark 字典并填充属性
    attrsDict := new(starlark.Dict)
    for key, value := range attrs {
        attrsDict.SetKey(starlark.String(key), starlark.String(value))
    }

    # 创建预定义的属性字典
    predefined := make(starlark.StringDict)
    predefined["attrs"] = attrsDict

    # 创建线程并初始化程序
    thread := &starlark.Thread{
        Name: "matcher",
    }
    results, err := m.program.Init(thread, predefined)
    if err != nil {
        newError("attr matcher").Base(err).WriteToLog()
    }
    # 获取匹配结果并返回
    satisfied := results["satisfied"]
    return satisfied != nil && bool(satisfied.Truth())
}

# 实现条件应用
func (m *AttributeMatcher) Apply(ctx routing.Context) bool {
    # 获取上下文中的属性
    attributes := ctx.GetAttributes()
    if attributes == nil {
        return false
    }
    # 调用属性匹配方法并返回结果
    return m.Match(attributes)
}
```