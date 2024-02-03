# `v2ray-core\app\router\command\config.go`

```go
package command

import (
    "strings"

    "v2ray.com/core/common/net"
    "v2ray.com/core/features/routing"
)

// routingContext is an wrapper of protobuf RoutingContext as implementation of routing.Context and routing.Route.
type routingContext struct {
    *RoutingContext
}

func (c routingContext) GetSourceIPs() []net.IP {
    return mapBytesToIPs(c.RoutingContext.GetSourceIPs())
}

func (c routingContext) GetSourcePort() net.Port {
    return net.Port(c.RoutingContext.GetSourcePort())
}

func (c routingContext) GetTargetIPs() []net.IP {
    return mapBytesToIPs(c.RoutingContext.GetTargetIPs())
}

func (c routingContext) GetTargetPort() net.Port {
    return net.Port(c.RoutingContext.GetTargetPort())
}

// AsRoutingContext converts a protobuf RoutingContext into an implementation of routing.Context.
func AsRoutingContext(r *RoutingContext) routing.Context {
    return routingContext{r}
}

// AsRoutingRoute converts a protobuf RoutingContext into an implementation of routing.Route.
func AsRoutingRoute(r *RoutingContext) routing.Route {
    return routingContext{r}
}

var fieldMap = map[string]func(*RoutingContext, routing.Route){
    "inbound":        func(s *RoutingContext, r routing.Route) { s.InboundTag = r.GetInboundTag() },
    "network":        func(s *RoutingContext, r routing.Route) { s.Network = r.GetNetwork() },
    "ip_source":      func(s *RoutingContext, r routing.Route) { s.SourceIPs = mapIPsToBytes(r.GetSourceIPs()) },
    "ip_target":      func(s *RoutingContext, r routing.Route) { s.TargetIPs = mapIPsToBytes(r.GetTargetIPs()) },
    "port_source":    func(s *RoutingContext, r routing.Route) { s.SourcePort = uint32(r.GetSourcePort()) },
    "port_target":    func(s *RoutingContext, r routing.Route) { s.TargetPort = uint32(r.GetTargetPort()) },
    "domain":         func(s *RoutingContext, r routing.Route) { s.TargetDomain = r.GetTargetDomain() },
    "protocol":       func(s *RoutingContext, r routing.Route) { s.Protocol = r.GetProtocol() },
    # 将路由上下文中的用户信息设置为路由对象中的用户信息
    "user":           func(s *RoutingContext, r routing.Route) { s.User = r.GetUser() },
    # 将路由上下文中的属性设置为路由对象中的属性
    "attributes":     func(s *RoutingContext, r routing.Route) { s.Attributes = r.GetAttributes() },
    # 将路由上下文中的出站组标签设置为路由对象中的出站组标签
    "outbound_group": func(s *RoutingContext, r routing.Route) { s.OutboundGroupTags = r.GetOutboundGroupTags() },
    # 将路由上下文中的出站标签设置为路由对象中的出站标签
    "outbound":       func(s *RoutingContext, r routing.Route) { s.OutboundTag = r.GetOutboundTag() },
// 定义一个函数，根据字段选择器返回一个将 routing.Route 转换为 protobuf RoutingContext 的函数
func AsProtobufMessage(fieldSelectors []string) func(routing.Route) *RoutingContext {
    // 初始化一个空的函数列表
    initializers := []func(*RoutingContext, routing.Route){}
    // 遍历字段映射表
    for field, init := range fieldMap {
        // 如果字段选择器为空，则获取所有字段
        if len(fieldSelectors) == 0 {
            initializers = append(initializers, init)
            continue
        }
        // 遍历字段选择器
        for _, selector := range fieldSelectors {
            // 如果字段以选择器开头，则添加到初始化函数列表中
            if strings.HasPrefix(field, selector) {
                initializers = append(initializers, init)
                break
            }
        }
    }
    // 返回一个函数，将 routing.Route 转换为 protobuf RoutingContext
    return func(ctx routing.Route) *RoutingContext {
        // 初始化一个 RoutingContext 对象
        message := new(RoutingContext)
        // 遍历初始化函数列表，对 RoutingContext 对象进行初始化
        for _, init := range initializers {
            init(message, ctx)
        }
        // 返回初始化后的 RoutingContext 对象
        return message
    }
}

// 将字节切片映射为 IP 地址切片
func mapBytesToIPs(bytes [][]byte) []net.IP {
    // 初始化一个空的 IP 地址切片
    var ips []net.IP
    // 遍历字节切片，将每个字节转换为 IP 地址并添加到 IP 地址切片中
    for _, rawIP := range bytes {
        ips = append(ips, net.IP(rawIP))
    }
    // 返回 IP 地址切片
    return ips
}

// 将 IP 地址切片映射为字节切片
func mapIPsToBytes(ips []net.IP) [][]byte {
    // 初始化一个空的字节切片
    var bytes [][]byte
    // 遍历 IP 地址切片，将每个 IP 地址转换为字节并添加到字节切片中
    for _, ip := range ips {
        bytes = append(bytes, []byte(ip))
    }
    // 返回字节切片
    return bytes
}
```