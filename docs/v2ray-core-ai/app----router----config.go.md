# `v2ray-core\app\router\config.go`

```
// +build !confonly

package router

import (
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/outbound"
    "v2ray.com/core/features/routing"
)

// CIDRList is an alias of []*CIDR to provide sort.Interface.
type CIDRList []*CIDR

// Len implements sort.Interface.
func (l *CIDRList) Len() int {
    return len(*l)
}

// Less implements sort.Interface.
func (l *CIDRList) Less(i int, j int) bool {
    ci := (*l)[i]
    cj := (*l)[j]

    if len(ci.Ip) < len(cj.Ip) {
        return true
    }

    if len(ci.Ip) > len(cj.Ip) {
        return false
    }

    for k := 0; k < len(ci.Ip); k++ {
        if ci.Ip[k] < cj.Ip[k] {
            return true
        }

        if ci.Ip[k] > cj.Ip[k] {
            return false
        }
    }

    return ci.Prefix < cj.Prefix
}

// Swap implements sort.Interface.
func (l *CIDRList) Swap(i int, j int) {
    (*l)[i], (*l)[j] = (*l)[j], (*l)[i]
}

type Rule struct {
    Tag       string
    Balancer  *Balancer
    Condition Condition
}

func (r *Rule) GetTag() (string, error) {
    if r.Balancer != nil {
        return r.Balancer.PickOutbound()
    }
    return r.Tag, nil
}

// Apply checks rule matching of current routing context.
func (r *Rule) Apply(ctx routing.Context) bool {
    return r.Condition.Apply(ctx)
}

func (rr *RoutingRule) BuildCondition() (Condition, error) {
    conds := NewConditionChan()

    if len(rr.Domain) > 0 {
        matcher, err := NewDomainMatcher(rr.Domain)
        if err != nil {
            return nil, newError("failed to build domain condition").Base(err)
        }
        conds.Add(matcher)
    }

    if len(rr.UserEmail) > 0 {
        conds.Add(NewUserMatcher(rr.UserEmail))
    }

    if len(rr.InboundTag) > 0 {
        conds.Add(NewInboundTagMatcher(rr.InboundTag))
    }

    if rr.PortList != nil {
        conds.Add(NewPortMatcher(rr.PortList, false))
    } else if rr.PortRange != nil {
        conds.Add(NewPortMatcher(&net.PortList{Range: []*net.PortRange{rr.PortRange}}, false))
    }
}
    # 如果源端口列表不为空，则创建一个新的端口匹配器并添加到条件中
    if rr.SourcePortList != nil:
        conds.Add(NewPortMatcher(rr.SourcePortList, true))

    # 如果网络列表不为空，则创建一个新的网络匹配器并添加到条件中
    if len(rr.Networks) > 0:
        conds.Add(NewNetworkMatcher(rr.Networks))
    # 如果网络列表为空但网络列表不为空，则创建一个新的网络匹配器并添加到条件中
    else if rr.NetworkList != nil:
        conds.Add(NewNetworkMatcher(rr.NetworkList.Network))

    # 如果 Geoip 列表不为空，则创建一个新的多重 GeoIP 匹配器并添加到条件中
    if len(rr.Geoip) > 0:
        cond, err := NewMultiGeoIPMatcher(rr.Geoip, false)
        if err != nil:
            return nil, err
        conds.Add(cond)
    # 如果 Geoip 列表为空但 Cidr 列表不为空，则创建一个新的多重 GeoIP 匹配器并添加到条件中
    else if len(rr.Cidr) > 0:
        cond, err := NewMultiGeoIPMatcher([]*GeoIP{{Cidr: rr.Cidr}}, false)
        if err != nil:
            return nil, err
        conds.Add(cond)

    # 如果源 Geoip 列表不为空，则创建一个新的多重 GeoIP 匹配器并添加到条件中
    if len(rr.SourceGeoip) > 0:
        cond, err := NewMultiGeoIPMatcher(rr.SourceGeoip, true)
        if err != nil:
            return nil, err
        conds.Add(cond)
    # 如果源 Geoip 列表为空但源 Cidr 列表不为空，则创建一个新的多重 GeoIP 匹配器并添加到条件中
    else if len(rr.SourceCidr) > 0:
        cond, err := NewMultiGeoIPMatcher([]*GeoIP{{Cidr: rr.SourceCidr}}, true)
        if err != nil:
            return nil, err
        conds.Add(cond)

    # 如果协议列表不为空，则创建一个新的协议匹配器并添加到条件中
    if len(rr.Protocol) > 0:
        conds.Add(NewProtocolMatcher(rr.Protocol))

    # 如果属性列表不为空，则创建一个新的属性匹配器并添加到条件中
    if len(rr.Attributes) > 0:
        cond, err := NewAttributeMatcher(rr.Attributes)
        if err != nil:
            return nil, err
        conds.Add(cond)

    # 如果条件列表长度为0，则返回一个警告错误
    if conds.Len() == 0:
        return nil, newError("this rule has no effective fields").AtWarning()

    # 返回条件列表和空错误
    return conds, nil
# 定义一个方法，用于构建负载均衡器
func (br *BalancingRule) Build(ohm outbound.Manager) (*Balancer, error) {
    # 返回一个负载均衡器对象，包括选择器、策略和出站管理器
    return &Balancer{
        selectors: br.OutboundSelector,  # 设置负载均衡器的选择器
        strategy:  &RandomStrategy{},     # 设置负载均衡器的策略为随机策略
        ohm:       ohm,                   # 设置负载均衡器的出站管理器
    }, nil  # 返回负载均衡器对象和空错误
}
```