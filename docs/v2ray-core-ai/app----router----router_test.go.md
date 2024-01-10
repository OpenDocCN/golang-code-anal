# `v2ray-core\app\router\router_test.go`

```
package router_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "github.com/golang/mock/gomock"  // 导入 mock 包
    . "v2ray.com/core/app/router"  // 导入路由包
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/session"  // 导入会话包
    "v2ray.com/core/features/outbound"  // 导入出站包
    routing_session "v2ray.com/core/features/routing/session"  // 导入路由会话包
    "v2ray.com/core/testing/mocks"  // 导入模拟包
)

type mockOutboundManager struct {
    outbound.Manager  // 定义模拟出站管理器结构
    outbound.HandlerSelector  // 定义模拟出站处理选择器结构
}

func TestSimpleRouter(t *testing.T) {
    config := &Config{  // 定义配置对象
        Rule: []*RoutingRule{  // 定义路由规则列表
            {
                TargetTag: &RoutingRule_Tag{  // 目标标签为标签类型
                    Tag: "test",  // 标签值为 "test"
                },
                Networks: []net.Network{net.Network_TCP},  // 网络类型为 TCP
            },
        },
    }

    mockCtl := gomock.NewController(t)  // 创建模拟控制器
    defer mockCtl.Finish()  // 延迟完成模拟控制器

    mockDns := mocks.NewDNSClient(mockCtl)  // 创建 DNS 客户端模拟
    mockOhm := mocks.NewOutboundManager(mockCtl)  // 创建出站管理器模拟
    mockHs := mocks.NewOutboundHandlerSelector(mockCtl)  // 创建出站处理选择器模拟

    r := new(Router)  // 创建路由对象
    common.Must(r.Init(config, mockDns, &mockOutboundManager{  // 初始化路由对象
        Manager:         mockOhm,  // 使用模拟出站管理器
        HandlerSelector: mockHs,  // 使用模拟出站处理选择器
    }))

    ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})  // 创建带有出站信息的上下文
    route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))  // 选择路由
    common.Must(err)  // 确保没有错误发生
    if tag := route.GetOutboundTag(); tag != "test" {  // 获取出站标签并进行断言
        t.Error("expect tag 'test', bug actually ", tag)  // 输出错误信息
    }
}

func TestSimpleBalancer(t *testing.T) {
    config := &Config{  // 定义配置对象
        Rule: []*RoutingRule{  // 定义路由规则列表
            {
                TargetTag: &RoutingRule_BalancingTag{  // 目标标签为负载均衡标签类型
                    BalancingTag: "balance",  // 负载均衡标签值为 "balance"
                },
                Networks: []net.Network{net.Network_TCP},  // 网络类型为 TCP
            },
        },
        BalancingRule: []*BalancingRule{  // 定义负载均衡规则列表
            {
                Tag:              "balance",  // 标签为 "balance"
                OutboundSelector: []string{"test-"},  // 出站选择器为 "test-"
            },
        },
    }
    # 创建一个gomock的控制器
    mockCtl := gomock.NewController(t)
    # 在函数返回时调用mockCtl的Finish方法，确保mock对象被正确释放
    defer mockCtl.Finish()

    # 创建一个DNSClient的mock对象
    mockDns := mocks.NewDNSClient(mockCtl)
    # 创建一个OutboundManager的mock对象
    mockOhm := mocks.NewOutboundManager(mockCtl)
    # 创建一个OutboundHandlerSelector的mock对象
    mockHs := mocks.NewOutboundHandlerSelector(mockCtl)

    # 设置mockHs的行为期望，当传入参数为["test-"]时，返回["test"]
    mockHs.EXPECT().Select(gomock.Eq([]string{"test-"})).Return([]string{"test"})

    # 创建一个Router对象
    r := new(Router)
    # 初始化Router对象，传入配置、mockDns和mockOutboundManager对象
    common.Must(r.Init(config, mockDns, &mockOutboundManager{
        Manager:         mockOhm,
        HandlerSelector: mockHs,
    }))

    # 创建一个带有Outbound信息的上下文
    ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
    # 选择路由并返回路由信息和错误
    route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
    # 如果有错误发生，则抛出异常
    common.Must(err)
    # 检查返回的路由信息中的出站标签，如果不是"test"，则抛出错误
    if tag := route.GetOutboundTag(); tag != "test" {
        t.Error("expect tag 'test', bug actually ", tag)
    }
}
// 测试IP按需策略
func TestIPOnDemand(t *testing.T) {
    // 创建配置对象
    config := &Config{
        DomainStrategy: Config_IpOnDemand,
        Rule: []*RoutingRule{
            {
                TargetTag: &RoutingRule_Tag{
                    Tag: "test",
                },
                Cidr: []*CIDR{
                    {
                        Ip:     []byte{192, 168, 0, 0},
                        Prefix: 16,
                    },
                },
            },
        },
    }

    // 创建模拟控制器
    mockCtl := gomock.NewController(t)
    defer mockCtl.Finish()

    // 创建模拟DNS客户端
    mockDns := mocks.NewDNSClient(mockCtl)
    // 设置模拟DNS客户端的期望行为
    mockDns.EXPECT().LookupIP(gomock.Eq("v2ray.com")).Return([]net.IP{{192, 168, 0, 1}}, nil).AnyTimes()

    // 创建路由器对象
    r := new(Router)
    // 初始化路由器对象
    common.Must(r.Init(config, mockDns, nil))

    // 创建带有出站信息的上下文
    ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
    // 选择路由
    route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
    // 必须处理错误
    common.Must(err)
    // 检查选择的出站标签是否符合预期
    if tag := route.GetOutboundTag(); tag != "test" {
        t.Error("expect tag 'test', bug actually ", tag)
    }
}

// 测试IP如果不匹配域名策略
func TestIPIfNonMatchDomain(t *testing.T) {
    // 创建配置对象
    config := &Config{
        DomainStrategy: Config_IpIfNonMatch,
        Rule: []*RoutingRule{
            {
                TargetTag: &RoutingRule_Tag{
                    Tag: "test",
                },
                Cidr: []*CIDR{
                    {
                        Ip:     []byte{192, 168, 0, 0},
                        Prefix: 16,
                    },
                },
            },
        },
    }

    // 创建模拟控制器
    mockCtl := gomock.NewController(t)
    defer mockCtl.Finish()

    // 创建模拟DNS客户端
    mockDns := mocks.NewDNSClient(mockCtl)
    // 设置模拟DNS客户端的期望行为
    mockDns.EXPECT().LookupIP(gomock.Eq("v2ray.com")).Return([]net.IP{{192, 168, 0, 1}}, nil).AnyTimes()

    // 创建路由器对象
    r := new(Router)
    // 初始化路由器对象
    common.Must(r.Init(config, mockDns, nil))
    // 使用 session.ContextWithOutbound 方法创建一个带有出站连接信息的上下文
    ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
    // 使用 routing_session.AsRoutingContext 方法从上下文中提取路由信息
    route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
    // 检查错误并处理
    common.Must(err)
    // 如果路由中的出站标签不是 "test"，则输出错误信息
    if tag := route.GetOutboundTag(); tag != "test" {
        t.Error("expect tag 'test', bug actually ", tag)
    }
func TestIPIfNonMatchIP(t *testing.T) {
    // 创建配置对象
    config := &Config{
        DomainStrategy: Config_IpIfNonMatch,
        Rule: []*RoutingRule{
            {
                // 设置目标标签为"test"
                TargetTag: &RoutingRule_Tag{
                    Tag: "test",
                },
                // 设置CIDR规则
                Cidr: []*CIDR{
                    {
                        // 设置IP地址为127.0.0.0，前缀为8
                        Ip:     []byte{127, 0, 0, 0},
                        Prefix: 8,
                    },
                },
            },
        },
    }

    // 创建mock控制器
    mockCtl := gomock.NewController(t)
    // 延迟执行mock控制器的Finish方法
    defer mockCtl.Finish()

    // 创建DNS客户端的mock对象
    mockDns := mocks.NewDNSClient(mockCtl)

    // 创建路由器对象
    r := new(Router)
    // 初始化路由器对象
    common.Must(r.Init(config, mockDns, nil))

    // 创建带有本地目标IP和端口80的出站上下文
    ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 80)})
    // 选择路由并返回路由信息和错误
    route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
    // 必须处理错误
    common.Must(err)
    // 检查路由的出站标签是否为"test"
    if tag := route.GetOutboundTag(); tag != "test" {
        // 如果不是"test"，则输出错误信息
        t.Error("expect tag 'test', bug actually ", tag)
    }
}
```