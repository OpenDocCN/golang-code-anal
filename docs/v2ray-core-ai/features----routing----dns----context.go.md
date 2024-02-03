# `v2ray-core\features\routing\dns\context.go`

```go
// 生成错误代码
//go:generate go run v2ray.com/core/common/errors/errorgen

// 导入所需的包
import (
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/dns"
    "v2ray.com/core/features/routing"
)

// ResolvableContext 是 routing.Context 的实现，具有域名解析功能
type ResolvableContext struct {
    routing.Context
    dnsClient   dns.Client
    resolvedIPs []net.IP
}

// GetTargetIPs 覆盖原始 routing.Context 的实现
func (ctx *ResolvableContext) GetTargetIPs() []net.IP {
    // 如果已经有目标 IP，则直接返回
    if ips := ctx.Context.GetTargetIPs(); len(ips) != 0 {
        return ips
    }

    // 如果已经解析过 IP，则直接返回
    if len(ctx.resolvedIPs) > 0 {
        return ctx.resolvedIPs
    }

    // 如果有目标域名，则尝试解析 IP
    if domain := ctx.GetTargetDomain(); len(domain) != 0 {
        ips, err := ctx.dnsClient.LookupIP(domain)
        if err == nil {
            ctx.resolvedIPs = ips
            return ips
        }
        // 如果解析失败，则记录错误日志
        newError("resolve ip for ", domain).Base(err).WriteToLog()
    }

    return nil
}

// ContextWithDNSClient 创建一个具有域名解析功能的新路由上下文
// 可以通过 GetTargetIPs() 获取已解析的域名 IP
func ContextWithDNSClient(ctx routing.Context, client dns.Client) routing.Context {
    return &ResolvableContext{Context: ctx, dnsClient: client}
}
```