# `v2ray-core\app\router\router.go`

```
// +build !confonly
// 定义包名为router，不包含confonly标签

package router
// 生成错误代码

// 导入所需的包
import (
    "context"
    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/features/dns"
    "v2ray.com/core/features/outbound"
    "v2ray.com/core/features/routing"
    routing_dns "v2ray.com/core/features/routing/dns"
)

// Router 是 routing.Router 的实现
type Router struct {
    domainStrategy Config_DomainStrategy
    rules          []*Rule
    balancers      map[string]*Balancer
    dns            dns.Client
}

// Route 是 routing.Route 的实现
type Route struct {
    routing.Context
    outboundGroupTags []string
    outboundTag       string
}

// Init 初始化 Router
func (r *Router) Init(config *Config, d dns.Client, ohm outbound.Manager) error {
    r.domainStrategy = config.DomainStrategy
    r.dns = d

    r.balancers = make(map[string]*Balancer, len(config.BalancingRule))
    for _, rule := range config.BalancingRule {
        balancer, err := rule.Build(ohm)
        if err != nil {
            return err
        }
        r.balancers[rule.Tag] = balancer
    }

    r.rules = make([]*Rule, 0, len(config.Rule))
    for _, rule := range config.Rule {
        cond, err := rule.BuildCondition()
        if err != nil {
            return err
        }
        rr := &Rule{
            Condition: cond,
            Tag:       rule.GetTag(),
        }
        btag := rule.GetBalancingTag()
        if len(btag) > 0 {
            brule, found := r.balancers[btag]
            if !found {
                return newError("balancer ", btag, " not found")
            }
            rr.Balancer = brule
        }
        r.rules = append(r.rules, rr)
    }

    return nil
}

// PickRoute 实现 routing.Router
func (r *Router) PickRoute(ctx routing.Context) (routing.Route, error) {
    rule, ctx, err := r.pickRouteInternal(ctx)
    if err != nil {
        return nil, err
    }
    tag, err := rule.GetTag()
}
    # 如果错误不为空，返回空和错误
    if err != nil:
        return nil, err
    # 返回一个包含上下文和出站标签的路由对象和空的错误
    return &Route{Context: ctx, outboundTag: tag}, nil
}

// pickRouteInternal 从路由器中选择路由规则
func (r *Router) pickRouteInternal(ctx routing.Context) (*Rule, routing.Context, error) {
    // 如果域名策略为 Config_IpOnDemand，则使用 DNS 客户端更新上下文
    if r.domainStrategy == Config_IpOnDemand {
        ctx = routing_dns.ContextWithDNSClient(ctx, r.dns)
    }

    // 遍历路由规则
    for _, rule := range r.rules {
        // 如果规则适用于上下文，则返回规则、上下文和空错误
        if rule.Apply(ctx) {
            return rule, ctx, nil
        }
    }

    // 如果域名策略不是 Config_IpIfNonMatch 或者目标域名长度为0，则返回空、上下文和 ErrNoClue 错误
    if r.domainStrategy != Config_IpIfNonMatch || len(ctx.GetTargetDomain()) == 0 {
        return nil, ctx, common.ErrNoClue
    }

    // 使用 DNS 客户端更新上下文
    ctx = routing_dns.ContextWithDNSClient(ctx, r.dns)

    // 如果有 IP，则再次尝试应用规则
    for _, rule := range r.rules {
        if rule.Apply(ctx) {
            return rule, ctx, nil
        }
    }

    // 返回空、上下文和 ErrNoClue 错误
    return nil, ctx, common.ErrNoClue
}

// Start 实现 common.Runnable 接口
func (*Router) Start() error {
    return nil
}

// Close 实现 common.Closable 接口
func (*Router) Close() error {
    return nil
}

// Type 实现 common.HasType 接口
func (*Router) Type() interface{} {
    return routing.RouterType()
}

// GetOutboundGroupTags 实现 routing.Route 接口
func (r *Route) GetOutboundGroupTags() []string {
    return r.outboundGroupTags
}

// GetOutboundTag 实现 routing.Route 接口
func (r *Route) GetOutboundTag() string {
    return r.outboundTag
}

// 初始化函数
func init() {
    // 注册配置
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        r := new(Router)
        // 要求特性
        if err := core.RequireFeatures(ctx, func(d dns.Client, ohm outbound.Manager) error {
            return r.Init(config.(*Config), d, ohm)
        }); err != nil {
            return nil, err
        }
        return r, nil
    }))
}
```