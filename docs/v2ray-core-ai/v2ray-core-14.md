# v2ray-core源码解析 14

# `app/router/errors.generated.go`

这段代码定义了一个名为 "router" 的包，其中包含以下内容：

1. 导入 "v2ray.com/core/common/errors" 包以使用其提供的错误处理功能。

2. 定义了一个名为 "errPathObjHolder" 的类型，该类型包含一个空的字典，该字典没有任何键值对。

3. 定义了一个名为 "newError" 的函数，该函数接收多个参数(...interface{})并返回一个带有 "errPathObjHolder" 类型的 errors.Error 对象。该函数使用 values... 参数传递给其中的参数，将其注入到 errPathObjHolder 类型的空字典中，并返回一个错误消息。如果任何给定的参数不正确，该函数将抛出 an errors.Error 异常。

4. 在函数内部，没有对 "errPathObjHolder" 类型进行任何操作。


```go
package router

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `app/router/router.go`

这段代码是一个 Go 语言编写的路由器代码。它包含了一些用于构建 Go 应用程序的指令，以及一些用于将该应用程序打包成可执行文件的指令。

首先，该代码使用 Go 标准库中的 `build` 函数和 `confonly` 标志编译该应用程序。这将只生成二进制文件，而不生成源代码。

其次，该代码通过 `go build` 命令生成一个名为 `v2ray.com/core/router/router_generated.go` 的文件。这个文件的目的是让 Go 编译器根据提供的源代码自动生成一个可以被链接到其他库的文件。

然后，该代码导入了自定义的错误处理程序 `errorgen`，它可能是从 v2ray.com/core/common/errors/errorgen 包中导入的。

接着，该代码导入了用于设置路由器中 DNS 查询的函数。这些函数包括使用 v2ray.com/core/features/routing/dns 中的 `DNSQueryString` 函数设置查询名称，并使用 v2ray.com/core/features/outbound 中的 `IConf赏品` 函数设置查询类型为 Outbound。

最后，该代码导入了用于设置路由器中路由的函数。这些函数包括使用 v2ray.com/core/features/routing 中的 `RouterCreatePredicate` 函数设置路由前缀，使用 v2ray.com/core/features/routing/dns 中的 `DNSQueryString` 函数设置路由目标，和使用 v2ray.com/core/features/routing/dns 中的 `RouterUpdatePredicate` 函数更新路由信息。


```go
// +build !confonly

package router

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/features/dns"
	"v2ray.com/core/features/outbound"
	"v2ray.com/core/features/routing"
	routing_dns "v2ray.com/core/features/routing/dns"
)

```

这段代码定义了一个名为 Router 的路由器 struct，它可以用来处理 HTTP 路由请求。

Router  struct 包含以下成员变量：

* domainStrategy: 这是一个用于处理域名分发的策略，可以设置为使用 IP 地址或者域名。
* rules: 这是一个数组，包含了所有规则，每个规则都是一个指向Rule struct的引用。
* balancers: 这是一个 map， map 中包含的是域名和对应的平衡策略。
* dns: 这是一个 dns.Client 类型的成员变量，用于设置 DNS 服务器。

Router struct 还包含一个名为 routes 的字段，它是一个 Map[string]*Rule 的结构体，用于存储所有的路由规则，它的键是字符串，表示一个 HTTP 路由前缀，而值则是 Rule struct 的引用。

此外，Router struct 还包含一个名为 id 的字段，用于标识不同的路由器实例。


```go
// Router is an implementation of routing.Router.
type Router struct {
	domainStrategy Config_DomainStrategy
	rules          []*Rule
	balancers      map[string]*Balancer
	dns            dns.Client
}

// Route is an implementation of routing.Route.
type Route struct {
	routing.Context
	outboundGroupTags []string
	outboundTag       string
}

```

该代码初始化了一个名为 `Router` 的路由器对象。它接受三个参数：`config` 表示路由器配置结构体，`d` 表示 DNS 客户端实例，`ohm` 表示出站连接管理器实例。

在函数体中，首先将路由器配置结构体的 `DomainStrategy` 字段与 `config` 中的 `DomainStrategy` 字段进行绑定，然后将 DNS 客户端实例 `d` 与 `config` 中的 `dns` 字段进行绑定。

接着，使用循环遍历 `config.BalancingRule` 切片，根据每个规则的 `Build` 函数计算出每个平衡器的构建过程，并将结果存储到 `r.balancers` 映射中。

然后，使用循环遍历 `config.Rule` 切片，根据每个规则的 `BuildCondition` 函数计算出每个规则的构建过程，并将结果构建出 `r.rules` 数组。在规则的 `Condition` 字段中，如果包含一个名为 `Balancer` 的关键词，则会从 `r.balancers` 中查找相应的 balancing 实例，否则会抛出错误。

最后，该函数返回一个 `nil` 表示没有错误。


```go
// Init initializes the Router.
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

```

该代码实现了一个名为 PickRoute 的路由器方法，它使用 routing.Router 中的 pickRouteInternal 函数来查找路由表中匹配给定前缀的规则。

具体来说，该方法接收一个 routing.Context 类型的上下文，首先检查是否正在使用基于 IP 地址的自动路由策略（通过调用 routing_dns.ContextWithDNSClient 函数实现），如果是，则将上下文使用 DNS 客户端设置为当前上下文，并继续使用 pickRouteInternal 函数查找匹配规则。

如果正在使用基于 IP 地址的自动路由策略，则会遍历所有规则，查找是否有一个规则匹配给定前缀。如果找到一个规则，则返回该规则并更新上下文，以便继续查找下一个匹配的规则。

如果正在使用基于域名的人工配置路由，则会尝试使用 DNS 客户端将上下文设置为 DNS 服务器，并查找所有规则。如果找到一个规则，则返回该规则并更新上下文，以便继续查找下一个匹配的规则。

最后，如果既没有使用基于 IP 地址的自动路由策略，也没有使用基于域名的人工配置路由，则会返回一个规则并将上下文返回。如果出现任何错误，则会返回一个 error。


```go
// PickRoute implements routing.Router.
func (r *Router) PickRoute(ctx routing.Context) (routing.Route, error) {
	rule, ctx, err := r.pickRouteInternal(ctx)
	if err != nil {
		return nil, err
	}
	tag, err := rule.GetTag()
	if err != nil {
		return nil, err
	}
	return &Route{Context: ctx, outboundTag: tag}, nil
}

func (r *Router) pickRouteInternal(ctx routing.Context) (*Rule, routing.Context, error) {
	if r.domainStrategy == Config_IpOnDemand {
		ctx = routing_dns.ContextWithDNSClient(ctx, r.dns)
	}

	for _, rule := range r.rules {
		if rule.Apply(ctx) {
			return rule, ctx, nil
		}
	}

	if r.domainStrategy != Config_IpIfNonMatch || len(ctx.GetTargetDomain()) == 0 {
		return nil, ctx, common.ErrNoClue
	}

	ctx = routing_dns.ContextWithDNSClient(ctx, r.dns)

	// Try applying rules again if we have IPs.
	for _, rule := range r.rules {
		if rule.Apply(ctx) {
			return rule, ctx, nil
		}
	}

	return nil, ctx, common.ErrNoClue
}

```

这是一段实现 common.Runnable、common.Closable 和 common.HasType 的代码。

这段代码定义了一个名为 Router 的 Runnable 类型的异步路由器。它实现了两个主要方法：

* Start()：返回一个 nil 错误，表示运行成功。
* Close()：返回一个 nil 错误，表示关闭成功。

这两个方法都是纯方法，没有带参数的实现。

另外，还实现了 common.HasType 接口，它允许 Router 类型具有 "Runnable" 和 "Closable" 的类型。

总的来说，这段代码定义了一个抽象类 Router，使我们可以创建一个通道，用于发送和接收 HTTP 请求。在实现 common.Runnable 和 common.Closable 接口的同时，可以确保 Router 对象可以作为 Runnable 和 Closable 类型的实例来使用。


```go
// Start implements common.Runnable.
func (*Router) Start() error {
	return nil
}

// Close implements common.Closable.
func (*Router) Close() error {
	return nil
}

// Type implement common.HasType.
func (*Router) Type() interface{} {
	return routing.RouterType()
}

```

这段代码定义了两个名为 "GetOutboundGroupTags" 和 "GetOutboundTag" 的函数，属于名为 "routing.Route" 的路由器实现。

具体来说，第一个函数 "GetOutboundGroupTags" 返回一个字符串数组，包含了所有属于 "outboundGroupTags" 路由的组，即该路由的所有目的地组。第二个函数 "GetOutboundTag" 返回一个字符串，包含了所有属于 "outboundTag" 路由的标签，即该路由的所有目的地标签。

该代码段没有定义任何外部变量，也没有接收任何外部参数。它实现了两个函数，分别用于获取出站的组和标签，这两个函数的实现都在 "routing.Route" 函数中。此外，该代码段中定义了一个名为 "init" 的函数，该函数接收一个配置参数 "ctx" 和一个配置参数 "config"，然后返回一个指向 "Router" 类型的变量 "r"，并且使用 "registerFeatures" 函数注册路由器功能。


```go
// GetOutboundGroupTags implements routing.Route.
func (r *Route) GetOutboundGroupTags() []string {
	return r.outboundGroupTags
}

// GetOutboundTag implements routing.Route.
func (r *Route) GetOutboundTag() string {
	return r.outboundTag
}

func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		r := new(Router)
		if err := core.RequireFeatures(ctx, func(d dns.Client, ohm outbound.Manager) error {
			return r.Init(config.(*Config), d, ohm)
		}); err != nil {
			return nil, err
		}
		return r, nil
	}))
}

```

# `app/router/router_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试名为 "router" 的包。这个包可能用于测试 routing 相关的功能。

首先，它导入了 testing 和 mock 包，这是 Go 语言标准库中的测试框架和mock包。

接着，它导入了 "router_test.go" 和 "router.go" 两个文件，这些文件可能是用于测试 "router" 包的。

接下来，它定义了一个名为 "testRouter" 的函数，这个函数可能是用于测试 "router" 包的。在函数内部，它使用了 "context" 和 "testing" 包，这些包可能用于处理应用程序的上下文和进行测试。

最后，它导入了 "mocks" 包，这个包可能是用于模拟 "router" 包中的代码和行为。

整个函数的作用是测试 "router" 包的功能，但具体是测试哪些功能并没有具体说明。


```go
package router_test

import (
	"context"
	"testing"

	"github.com/golang/mock/gomock"
	. "v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/session"
	"v2ray.com/core/features/outbound"
	routing_session "v2ray.com/core/features/routing/session"
	"v2ray.com/core/testing/mocks"
)

```

该代码的作用是测试一个简单的路由器（Router）的隔离测试，该路由器使用 mockOutboundManager 来模拟出站接口的经理和 handler selector，以实现对目标标记 "test" 的路由转发。

具体来说，代码会创建一个 mockOutboundManager 模拟对象，该对象包含一个 outbound 经理（Manager）和一个 outbound 处理器选择器（HandlerSelector），分别用于处理出站流量和选择出站路由器中的下一个路由。同时，代码还会创建一个 mocks.NewDNSClient 和 mocks.NewOutboundManager 模拟对象，用于设置 DNS 查询和出站接口的经理。

在 main 函数中，会创建一个新的路由器实例，并使用 mockOutboundManager 中的 Manager 设置该实例的 manager。然后，使用该实例的 HandlerSelector 设置下一个路由器中处理 "test" 目标标记的处理器选择器。最后，使用该实例的 Manager 设置该实例的 DNS 查询。

在 TestSimpleRouter 函数中，使用 gomock.NewController 创建一个控制器，用于模拟整个测试过程。在控制器死后，使用控制器模拟 DNS 查询、经理设置和处理器选择器。

该代码的主要目的是提供一个简单的路由器测试场景，以便在测试中验证相关功能点的正确性。


```go
type mockOutboundManager struct {
	outbound.Manager
	outbound.HandlerSelector
}

func TestSimpleRouter(t *testing.T) {
	config := &Config{
		Rule: []*RoutingRule{
			{
				TargetTag: &RoutingRule_Tag{
					Tag: "test",
				},
				Networks: []net.Network{net.Network_TCP},
			},
		},
	}

	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockDns := mocks.NewDNSClient(mockCtl)
	mockOhm := mocks.NewOutboundManager(mockCtl)
	mockHs := mocks.NewOutboundHandlerSelector(mockCtl)

	r := new(Router)
	common.Must(r.Init(config, mockDns, &mockOutboundManager{
		Manager:         mockOhm,
		HandlerSelector: mockHs,
	}))

	ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
	route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
	common.Must(err)
	if tag := route.GetOutboundTag(); tag != "test" {
		t.Error("expect tag 'test', bug actually ", tag)
	}
}

```

这段代码是一个 Go 语言中的测试函数，名为 TestSimpleBalancer。函数的作用是测试一个名为 SimpleBalancer 的路由器是否符合预设的规则。

具体来说，这段代码创建了一个名为 Config 的结构体，其中包含了一个路由器的配置信息，包括目标标签、出站网络等。接着，定义了一系列的 RoutingRule 和 BalancingRule，用于定义路由器的路由规则和流量转发规则。最后，通过 mockCtl 模拟 DNS 服务器和 Outbound 管理器的功能，实现了对路由器出站流量进行拦截和模拟测试等功能。

在 TestSimpleBalancer 函数中，首先创建了一个 Config 结构体，其中包含了一些路由器的配置信息。然后，通过 gomock 模拟 DNS 服务器和 Outbound 管理器的功能，实现了对路由器出站流量进行拦截和模拟测试等功能。最后，创建了一个名为 Router 的结构体，实现了对路由器的初始化、选择路由和拦截出站流量等功能。在整个函数中，通过模拟测试，测试不同标签号下路由器是否能够正常工作，并对异常情况进行错误处理。


```go
func TestSimpleBalancer(t *testing.T) {
	config := &Config{
		Rule: []*RoutingRule{
			{
				TargetTag: &RoutingRule_BalancingTag{
					BalancingTag: "balance",
				},
				Networks: []net.Network{net.Network_TCP},
			},
		},
		BalancingRule: []*BalancingRule{
			{
				Tag:              "balance",
				OutboundSelector: []string{"test-"},
			},
		},
	}

	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockDns := mocks.NewDNSClient(mockCtl)
	mockOhm := mocks.NewOutboundManager(mockCtl)
	mockHs := mocks.NewOutboundHandlerSelector(mockCtl)

	mockHs.EXPECT().Select(gomock.Eq([]string{"test-"})).Return([]string{"test"})

	r := new(Router)
	common.Must(r.Init(config, mockDns, &mockOutboundManager{
		Manager:         mockOhm,
		HandlerSelector: mockHs,
	}))

	ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
	route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
	common.Must(err)
	if tag := route.GetOutboundTag(); tag != "test" {
		t.Error("expect tag 'test', bug actually ", tag)
	}
}

```

这段代码是一个名为 "TestIPOnDemand" 的函数，它用于测试是否能够通过 IP 出站时demand 机制。

函数内部定义了一个名为 "config" 的变量，它是一个实现了 Config_IpOnDemand 接口的 Config 对象。该 Config 对象包含了以下配置项：

* DomainStrategy: Config_IpOnDemand
* Rule: 配置了一个 HTTP 路由规则，用于测试。

具体来说，该路由规则指定了目标标记为 "test"，IP 地址前缀为 16，匹配的 CIDR 地址为 {192,168,0,0}。该路由规则使用了 OnDemand 机制，根据可用性、安全性和其他因素选择一个可用的 IP 地址，用于从服务器出站。

函数内部还定义了一个名为 "mockCtl" 的 gomock 偶读，用于模拟 DNS 客户端操作。该偶读包含一个名为 "mockDns" 的 mocks 偶，用于向 "mockCtl" 发送 DNS 查询请求，以获取 "v2ray.com" 的 IP 地址。

偶中还定义了一个名为 "r" 的 new Router 对象，该对象实现了 Router 接口，并在初始化时使用了上面定义的 Config 对象和模拟 DNS 客户端。

最后，函数内部定义了一个名为 "ctx" 的 context，该上下文包含一个用于从服务器出站的 outbound 通道和一个用于设置该通道目标地址的任意网络接口。

函数内部使用 r.PickRoute 函数从出站通道中选择一个路由，并使用上面定义的 r.config 函数设置该路由的配置对象。然后，使用 context.Background() 函数创建一个新的上下文，该上下文将用于向模拟 DNS 客户端发送查询请求，并使用 r.PickRoute 函数选择一个从上下文中获取的路由，该路由将使用从上下文中获取的配置对象出站。

最后，使用偶中定义的 "mockDns.Expect" 函数模拟 DNS 客户端向 "v2ray.com" 发送查询请求，并使用偶中定义的 "anyTimes" 函数确保查询请求至少成功一次。如果查询请求失败，则使用 "t.Error" 函数输出一个错误消息，其中包含获取到的标签 "test"。


```go
func TestIPOnDemand(t *testing.T) {
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

	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockDns := mocks.NewDNSClient(mockCtl)
	mockDns.EXPECT().LookupIP(gomock.Eq("v2ray.com")).Return([]net.IP{{192, 168, 0, 1}}, nil).AnyTimes()

	r := new(Router)
	common.Must(r.Init(config, mockDns, nil))

	ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
	route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
	common.Must(err)
	if tag := route.GetOutboundTag(); tag != "test" {
		t.Error("expect tag 'test', bug actually ", tag)
	}
}

```

这段代码是一个名为 `TestIPIfNonMatchDomain` 的测试函数，属于网络 `net/http` 包的 `Router` 类型的测试用例。它的作用是测试在非匹配域名的情况下，是否能够正确地进行负载均衡。

具体来说，这段代码的作用如下：

1. 配置一个名为 `config` 的 `Config` 对象，其中包含以下信息：

 - `DomainStrategy`: 使用IP地址如果非匹配域名。
 - `Rule`: 路由策略，其中包含一个包含目标标记（`TargetTag`）和CIDR规则的`RoutingRule`。

2. 创建一个 mock DNS 客户端，并将其缓存到 `mockDns` 变量中。

3. 创建一个名为 `router` 的 `Router` 对象，并使用 `config` 和 `mockDns` 来初始化它。

4. 使用 `router` 的 `PickRoute` 函数来选择一条出站路由。

5. 使用 `router` 的 `RoutingContext` 函数创建一个出站路由的上下文，并使用上下文中的 `routing_session` 来设置目标地址为 `net.DomainAddress("v2ray.com")`（这里作为测试的目标网站）。

6. 循环调用 `router.PickRoute` 函数，并将返回的结果打印到控制台上。

7. 测试 `config.DomainStrategy` 是否为 `Config_IpIfNonMatch`，如果是，就进行测试。如果不是，则需要进一步测试。


```go
func TestIPIfNonMatchDomain(t *testing.T) {
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

	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockDns := mocks.NewDNSClient(mockCtl)
	mockDns.EXPECT().LookupIP(gomock.Eq("v2ray.com")).Return([]net.IP{{192, 168, 0, 1}}, nil).AnyTimes()

	r := new(Router)
	common.Must(r.Init(config, mockDns, nil))

	ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)})
	route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
	common.Must(err)
	if tag := route.GetOutboundTag(); tag != "test" {
		t.Error("expect tag 'test', bug actually ", tag)
	}
}

```

该代码是一个名为 "TestIPIfNonMatchIP" 的测试函数，属于 "ip" 包。其作用是测试一个名为 "test.example.com" 的域名是否可以通过非匹配 IP 地址进行解析，即测试是否能够通过具有 "test" 标签的 IP 地址（该 IP 地址的 CIDR 表示为 127.0.0.0/8）。

具体来说，该函数内置于一个名为 "func TestIPIfNonMatchIP" 的函数内部，使用 Go 标准库中的 "testing" 和 "io" 包，以及一个名为 "mocks" 的外部库。函数的参数为 "t" 和一个名为 "Config" 的结构体，其中 "Config" 包含一个 DNS 策略和一个路由规则列表。

函数首先创建一个名为 "config" 的结构体，其中包含一个 DNS 策略和一个路由规则列表。然后，函数创建一个名为 "mockCtl" 的 gomock 模拟器，用于模拟 DNS 客户端操作。接着，函数创建一个名为 "mockDns" 的 mocks 模拟 DNS 客户端，用于向 DNS 服务器发送查询请求。

接下来，函数创建一个名为 "r" 的路由器实例，并使用模拟器初始化该路由器，将 DNS 策略设置为 "ipIfNonMatch"，将路由规则列表设置为模拟器生成的规则列表。

最后，函数创建一个名为 "ctx" 的上下文，其中包含一个代表入站流量的 "outbound" 上下文，该上下文的 "target" 字段设置为 127.0.0.1 和 80。然后，函数使用上下文开始模拟入站流量，并从入站流量中选择一个路由，检查选定的路由是否与模拟 DNS 查询请求中的标签 "test" 匹配。如果匹配，则函数将在函数输出中记录一个错误，并指出该标签。否则，函数不会输出任何错误，表明模拟器可以成功解析 "test.example.com" 域名。


```go
func TestIPIfNonMatchIP(t *testing.T) {
	config := &Config{
		DomainStrategy: Config_IpIfNonMatch,
		Rule: []*RoutingRule{
			{
				TargetTag: &RoutingRule_Tag{
					Tag: "test",
				},
				Cidr: []*CIDR{
					{
						Ip:     []byte{127, 0, 0, 0},
						Prefix: 8,
					},
				},
			},
		},
	}

	mockCtl := gomock.NewController(t)
	defer mockCtl.Finish()

	mockDns := mocks.NewDNSClient(mockCtl)

	r := new(Router)
	common.Must(r.Init(config, mockDns, nil))

	ctx := session.ContextWithOutbound(context.Background(), &session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 80)})
	route, err := r.PickRoute(routing_session.AsRoutingContext(ctx))
	common.Must(err)
	if tag := route.GetOutboundTag(); tag != "test" {
		t.Error("expect tag 'test', bug actually ", tag)
	}
}

```

# `app/router/command/command.go`

这段代码是一个 Go 语言编写的命令行工具，用于处理 v2ray.com 服务器的配置文件。它的主要作用是生成一个名为 "v2ray.com/core/common/errors/errorgen" 的统计报告，以提供关于 v2ray.com 服务器运行时出现的错误的情况。

这里使用了几个 Go 语言的标准库：

1. `build` 包：这是一个用于构建 Go 应用程序的工具链，这里可能用于构建 v2ray.com 服务器。
2. `confonly`：这是一个用于只读模式运行 Go 应用程序的工具，这里可能用于避免在写入配置文件时意外的更改。
3. `google.golang.org/grpc`：这是一个用于与 Google Cloud 平台服务器进行交互的 Go 语言库，这里可能用于在 v2ray.com 服务器上运行远程代理。
4. `v2ray.com/core`：这是一个 v2ray.com 服务器的前端库，这里可能用于提供一些 v2ray.com 服务器的核心功能。
5. `v2ray.com/core/common`：这是一个 v2ray.com 服务器的前端库，这里可能用于提供一些 v2ray.com 服务器的前端功能。
6. `v2ray.com/core/features/routing`：这是一个 v2ray.com 服务器的前端库，这里可能用于提供一些 v2ray.com 服务器的前端路由功能。
7. `v2ray.com/core/features/stats`：这是一个 v2ray.com 服务器的前端库，这里可能用于提供一些 v2ray.com 服务器的前端统计功能。

另外，该命令行工具可能还使用了一些第三方库或工具，但以上信息已经足够了解它的作用了。


```go
// +build !confonly

package command

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"
	"time"

	"google.golang.org/grpc"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/features/stats"
)

```

这段代码定义了一个名为 `routingServer` 的类型，它实现了 `RoutingService` 接口，该接口定义了一个 `Router` 和一个 `RoutingStats` 统计信息。

`NewRoutingServer` 函数接受两个参数，一个是 `router` 实现 `RoutingService` 接口的 `Router` 类型，另一个是 `routingStats` 实现 `RoutingStats` 接口的 `Stats` 类型，它们都被赋值给一个名为 `routingServer` 的引用，最终返回一个实现了 `RoutingService` 接口的 `routingServer` 实例。

`TestRoute` 函数接收一个 `TestRouteRequest` 类型的参数，它包含一个 `RoutingContext` 类型的 `ctx` 和一个 `RequestMessage` 类型的 `request`，这个函数的作用是在一个 `ctx` 中执行 `Router` 实例的 `PickRoute` 方法，并将结果返回，同时也可以发布结果，将结果发送到指定的 `routingStats` 实例。

具体来说，`NewRoutingServer` 和 `TestRoute` 函数都在创建和配置一个 `RoutingServer` 实例，`Router` 和 `RoutingStats` 统计信息会被设置为默认值，可以在需要时动态地获取统计信息，例如通过调用 `Router` 实例的 `PickRoute` 方法来获取路由信息，然后将路由信息发送到 `routingStats` 统计信息中，以便进行统计和分析。


```go
// routingServer is an implementation of RoutingService.
type routingServer struct {
	router       routing.Router
	routingStats stats.Channel
}

// NewRoutingServer creates a statistics service with statistics manager.
func NewRoutingServer(router routing.Router, routingStats stats.Channel) RoutingServiceServer {
	return &routingServer{
		router:       router,
		routingStats: routingStats,
	}
}

func (s *routingServer) TestRoute(ctx context.Context, request *TestRouteRequest) (*RoutingContext, error) {
	if request.RoutingContext == nil {
		return nil, newError("Invalid routing request.")
	}
	route, err := s.router.PickRoute(AsRoutingContext(request.RoutingContext))
	if err != nil {
		return nil, err
	}
	if request.PublishResult && s.routingStats != nil {
		ctx, _ := context.WithTimeout(context.Background(), 4*time.Second) // nolint: govet
		s.routingStats.Publish(ctx, route)
	}
	return AsProtobufMessage(request.FieldSelectors)(route), nil
}

```

此代码定义了一个名为 `func` 的函数，接收两个参数：一个指向 `RoutingService_SubscribeRoutingStatsServer` 的 `s` 引用，以及一个指向 `SubscribeRoutingStatsRequest` 的 `request` 参数。

函数的作用是：如果 `RoutingServer` 实例中的 `routingStats` 变量为空，那么函数会返回一个错误，指出 routing statistics 未启用。否则，函数会将 `stats.SubscribeRunnableChannel` 函数作为 `s.routingStats` 引用，并传入 `request.FieldSelectors` 作为参数。如果此函数成功，函数会返回一个指向 `SubscribeRoutingStatsServer` 的 `subscriber` 变量，用于接收 statistics 数据。

接下来，函数会进入一个无限循环，每轮循环会接收来自 `subscriber` 的 `routing.Route` 类型数据，以及来自 `RoutingService_SubscribeRoutingStatsServer` 的统计数据。如果获取统计数据过程中出现错误，函数会返回一个错误。如果所有情况都正常，函数会继续执行循环，直到遇到循环结束的条件，即 `stream.Context().Done()`。

函数的实现细节如下：

java
func (s *routingServer) SubscribeRoutingStats(request *SubscribeRoutingStatsRequest, stream RoutingService_SubscribeRoutingStatsServer) error {
	if s.routingStats == nil {
		return newError("Routing statistics not enabled.")
	}
	genMessage := AsProtobufMessage(request.FieldSelectors)
	subscriber, err := stats.SubscribeRunnableChannel(s.routingStats)
	if err != nil {
		return err
	}
	defer stats.UnsubscribeClosableChannel(s.routingStats, subscriber) // nolint: errcheck
	for {
		select {
		case value, ok := <-subscriber:
			if !ok {
				return newError("Upstream closed the subscriber channel.")
			}
			route, ok := value.(routing.Route)
			if !ok {
				return newError("Upstream sent malformed statistics.")
			}
			err := stream.Send(genMessage(route))
			if err != nil {
				return err
			}
		case <-stream.Context().Done():
			return stream.Context().Err()
		}
	}
	return nil
}



```go
func (s *routingServer) SubscribeRoutingStats(request *SubscribeRoutingStatsRequest, stream RoutingService_SubscribeRoutingStatsServer) error {
	if s.routingStats == nil {
		return newError("Routing statistics not enabled.")
	}
	genMessage := AsProtobufMessage(request.FieldSelectors)
	subscriber, err := stats.SubscribeRunnableChannel(s.routingStats)
	if err != nil {
		return err
	}
	defer stats.UnsubscribeClosableChannel(s.routingStats, subscriber) // nolint: errcheck
	for {
		select {
		case value, ok := <-subscriber:
			if !ok {
				return newError("Upstream closed the subscriber channel.")
			}
			route, ok := value.(routing.Route)
			if !ok {
				return newError("Upstream sent malformed statistics.")
			}
			err := stream.Send(genMessage(route))
			if err != nil {
				return err
			}
		case <-stream.Context().Done():
			return stream.Context().Err()
		}
	}
}

```

这段代码定义了一个名为“routingServer”的接口类型和一个名为“service”的结构体。

该代码的实际功能是创建一个名为“routingServer”的接口类型，该接口类型包含一个“register”方法。

具体来说，这段代码创建了一个名为“s”的结构体，该结构体包含一个名为“v”的指针，该指针类型为“core.Instance”。

此外，该代码还定义了一个名为“register”的函数，该函数接收一个名为“server”的“grpc.Server”类型的参数，并调用了包含一个名为“router”的“routing.Router”类型和一个名为“stats”的“stats.Manager”类型的参数。

该函数通过调用名为“RegisterRoutingServiceServer”的函数，并传递一个名为“server”的“routing.Server”类型的参数和一个名为“nil”的参数，该参数表示不需要嵌入实现unimplemented的routingServiceServer。


```go
func (s *routingServer) mustEmbedUnimplementedRoutingServiceServer() {}

type service struct {
	v *core.Instance
}

func (s *service) Register(server *grpc.Server) {
	common.Must(s.v.RequireFeatures(func(router routing.Router, stats stats.Manager) {
		RegisterRoutingServiceServer(server, NewRoutingServer(router, nil))
	}))
}

func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
		s := core.MustFromContext(ctx)
		return &service{v: s}, nil
	}))
}

```

# `app/router/command/command.pb.go`

该代码定义了一个名为 "command" 的包，其中包含一个名为 "Command" 的接口，该接口用于定义命令的数据结构。

在此代码中，首先定义了 "protoc-gen-go" 和 "protoc" 两个依赖项，它们是 Go 语言生成的protobuf文件的生成器。然后，定义了 "version" 变量，用于存储生成的文件版本信息。

接着，定义了 "package.proto" 文件，该文件是 Google官方发布的 protobuf 定义文件，描述了命令数据结构的构件。

然后，定义了 "Command" 接口，该接口定义了命令的数据结构，包括字段名称、数据类型、编码等信息。

接着，定义了 "reflect" 和 "sync" 包，它们是 Go 语言标准库中的反射和同步包，可以用于操作命令数据结构中的字段。

最后，定义了 "net" 包，该包提供了与网络相关的接口，可以用于定义命令的网络传输协议。

总结起来，该代码定义了一个名为 "command" 的包，其中包含一个名为 "Command" 的接口，该接口用于定义命令的数据结构，并定义了一些依赖项和文件，用于生成和定义该包的定义。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: app/router/command/command.proto

package command

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	net "v2ray.com/core/common/net"
)

```

This is a protobuf message definition for an `InboundTag` object. It defines the properties of an `InboundTag` message and its binary representation.

This message has the following fields:
- `InboundTag`: A tag indicating whether this is an incoming request or a outgoing request.
- `Network`: The network protocol to use for the incoming and outgoing requests.
- `SourceIPs`: The source IP addresses to which this tag should be associated.
- `TargetIPs`: The destination IP addresses to which this tag should be associated.
- `SourcePort`: The source port to which this tag should be associated.
- `TargetPort`: The destination port to which this tag should be associated.
- `TargetDomain`: The target domain to which this tag should be associated.
- `Protocol`: The protocol to use for the incoming and outgoing requests.
- `User`: The user ID to which this tag should be associated.
- `Attributes`: Additional attributes associated with this tag.
- `OutboundGroupTags`: The tags for outbound group requests.
- `OutboundTag`: The tag for outbound requests.

This message is part of the protobuf specification version 2.0 and is supported by all versions of the protobuf compiler.


```go
const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// This is a compile-time assertion that a sufficiently up-to-date version
// of the legacy proto package is being used.
const _ = proto.ProtoPackageIsVersion4

// RoutingContext is the context with information relative to routing process.
// It conforms to the structure of v2ray.core.features.routing.Context and
// v2ray.core.features.routing.Route.
type RoutingContext struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	InboundTag        string            `protobuf:"bytes,1,opt,name=InboundTag,proto3" json:"InboundTag,omitempty"`
	Network           net.Network       `protobuf:"varint,2,opt,name=Network,proto3,enum=v2ray.core.common.net.Network" json:"Network,omitempty"`
	SourceIPs         [][]byte          `protobuf:"bytes,3,rep,name=SourceIPs,proto3" json:"SourceIPs,omitempty"`
	TargetIPs         [][]byte          `protobuf:"bytes,4,rep,name=TargetIPs,proto3" json:"TargetIPs,omitempty"`
	SourcePort        uint32            `protobuf:"varint,5,opt,name=SourcePort,proto3" json:"SourcePort,omitempty"`
	TargetPort        uint32            `protobuf:"varint,6,opt,name=TargetPort,proto3" json:"TargetPort,omitempty"`
	TargetDomain      string            `protobuf:"bytes,7,opt,name=TargetDomain,proto3" json:"TargetDomain,omitempty"`
	Protocol          string            `protobuf:"bytes,8,opt,name=Protocol,proto3" json:"Protocol,omitempty"`
	User              string            `protobuf:"bytes,9,opt,name=User,proto3" json:"User,omitempty"`
	Attributes        map[string]string `protobuf:"bytes,10,rep,name=Attributes,proto3" json:"Attributes,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
	OutboundGroupTags []string          `protobuf:"bytes,11,rep,name=OutboundGroupTags,proto3" json:"OutboundGroupTags,omitempty"`
	OutboundTag       string            `protobuf:"bytes,12,opt,name=OutboundTag,proto3" json:"OutboundTag,omitempty"`
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `x` 的参数，并返回一个指向 `RoutingContext` 类型的 `x` 的指针。

这个函数的作用是重置 `x` 指向的对象，将其设置为 `RoutingContext{}` 类型的空对象。如果 `protoimpl.UnsafeEnabled` 是 `true`，那么会尝试使用 `file_app_router_command_command_proto_msgTypes[0]` 和 `protoimpl.X.MessageStateOf(protoimpl.Pointer(x))` 创建一个新的 `RoutingContext` 实例，并将其存储在 `mi` 和 `ms` 变量中。

函数还定义了一个名为 `String` 的函数，接收一个 `*RoutingContext` 类型的 `x` 作为参数，并返回一个字符串表示 `x`。

函数还定义了一个名为 `ProtoMessage` 的函数，该函数的实现为空，似乎没有做任何事情。


```go
func (x *RoutingContext) Reset() {
	*x = RoutingContext{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_command_command_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *RoutingContext) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*RoutingContext) ProtoMessage() {}

```

这段代码定义了两个函数，一个是`func (x *RoutingContext) ProtoReflect() protoreflect.Message`，另一个是`func (*RoutingContext) Descriptor() ([]byte, []int)`。

`func (x *RoutingContext) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_command_command_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}`

这个函数接收一个`RoutingContext`类型的参数`x`，并返回一个`protoreflect.Message`类型的变量`mi`。`x`参数可以是任何实现了`file_app_router_command_command_proto`接口的`RoutingContext`实例。如果`x`不为空，则执行以下操作：检查`protoimpl.UnsafeEnabled`是否为`true`，如果是，则执行以下操作：尝试从`x``文件应用的`RouterCommand`实例中获取`MessageStateOf`方法，并尝试获取`MessageInfo`。如果`MessageInfo`为`nil`，则执行以下操作：将`mi`存储为`MessageInfo`的`StoreMessageInfo`方法的返回值，并将`x`作为参数传递给`MessageOf`方法。如果`protoimpl.UnsafeEnabled`为`false`，则执行以下操作：直接返回`x`的`MessageOf`方法。无论哪种情况，最后返回的值都是`mi`的`MessageOf`方法的返回值。

`func (*RoutingContext) Descriptor() ([]byte, []int) {
	return file_app_router_command_command_proto_rawDescGZIP(), []int{0}`

这个函数返回`RoutingContext`实例的`Descriptor`方法的返回值。`Descriptor`方法的返回值包括`file_app_router_command_command_proto_rawDescGZIP`类型和一个包含两个元素的切片类型，第一个元素是`descriptor`方法的返回值，第二个元素是编译器生成的类型描述符。


```go
func (x *RoutingContext) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_command_command_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use RoutingContext.ProtoReflect.Descriptor instead.
func (*RoutingContext) Descriptor() ([]byte, []int) {
	return file_app_router_command_command_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了三个函数，分别作用于一个名为 `RoutingContext` 的结构体。

`func (x *RoutingContext) GetInboundTag() string {
	if x != nil {
		return x.InboundTag
	}
	return ""
}` 函数接收一个 `RoutingContext` 类型的参数 `x`，并返回 `x` 的入站标签。如果 `x` 为 `nil`，则返回空字符串。

`func (x *RoutingContext) GetNetwork() net.Network {
	if x != nil {
		return x.Network
	}
	return net.Network_Unknown
}` 函数接收一个 `RoutingContext` 类型的参数 `x`，并返回 `x` 的网络类型。如果 `x` 为 `nil`，则返回 `net.Network_Unknown`。

`func (x *RoutingContext) GetSourceIPs() [][]byte {
	if x != nil {
		return x.SourceIPs
	}
	return nil
}` 函数接收一个 `RoutingContext` 类型的参数 `x`，并返回 `x` 的源 IP 地址数组。如果 `x` 为 `nil`，则返回 `nil`。


```go
func (x *RoutingContext) GetInboundTag() string {
	if x != nil {
		return x.InboundTag
	}
	return ""
}

func (x *RoutingContext) GetNetwork() net.Network {
	if x != nil {
		return x.Network
	}
	return net.Network_Unknown
}

func (x *RoutingContext) GetSourceIPs() [][]byte {
	if x != nil {
		return x.SourceIPs
	}
	return nil
}

```

这两函数主要作用于 `RoutingContext` 类型的数据结构中。`RoutingContext` 是一个用于获取路由信息的数据结构，它包含了目标 IP 地址和端口、源 IP 地址和端口等信息。这两函数主要实现了从 `RoutingContext` 中获取目标 IP 地址和端口、源 IP 地址和端口，并返回给调用者。

具体来说，第一个函数 `GetTargetIPs` 接收一个 `RoutingContext` 类型的参数 `x`，然后判断它是否为 `nil`。如果是 `nil`，那么返回 `nil`。否则，它调用 `x.TargetIPs` 来获取目标 IP 地址，并返回给调用者。如果 `x` 为 `nil`，那么第一个返回结果与上面相同，都是 `nil`。

第二个函数 `GetSourcePort` 与第一个函数类似，判断 `x` 是否为 `nil`。如果是 `nil`，那么返回 `0`。否则，它调用 `x.SourcePort` 来获取源端口，并返回给调用者。如果 `x` 为 `nil`，那么第二个返回结果与上面相同，都是 `0`。

第三个函数 `GetTargetPort` 与第二个函数类似，判断 `x` 是否为 `nil`。如果是 `nil`，那么返回 `0`。否则，它调用 `x.TargetPort` 来获取目标端口，并返回给调用者。如果 `x` 为 `nil`，那么第三个返回结果与上面相同，都是 `0`。


```go
func (x *RoutingContext) GetTargetIPs() [][]byte {
	if x != nil {
		return x.TargetIPs
	}
	return nil
}

func (x *RoutingContext) GetSourcePort() uint32 {
	if x != nil {
		return x.SourcePort
	}
	return 0
}

func (x *RoutingContext) GetTargetPort() uint32 {
	if x != nil {
		return x.TargetPort
	}
	return 0
}

```

这段代码定义了三个函数，分别接收一个名为RoutingContext的接口类型的参数x，并返回以下三个字符串类型的值：

func (x *RoutingContext) GetTargetDomain() string {
	if x != nil {
		return x.TargetDomain
	}
	return ""
}

func (x *RoutingContext) GetProtocol() string {
	if x != nil {
		return x.Protocol
	}
	return ""
}

func (x *RoutingContext) GetUser() string {
	if x != nil {
		return x.User
	}
	return ""
}

函数的作用是获取RoutingContext对象中对应的TargetDomain、Protocol和User的值。如果参数x为 nil，则返回默认值。函数在函数体内部使用if语句判断x是否为 nil，如果是，则执行相应的代码返回相应的值。


```go
func (x *RoutingContext) GetTargetDomain() string {
	if x != nil {
		return x.TargetDomain
	}
	return ""
}

func (x *RoutingContext) GetProtocol() string {
	if x != nil {
		return x.Protocol
	}
	return ""
}

func (x *RoutingContext) GetUser() string {
	if x != nil {
		return x.User
	}
	return ""
}

```

这段代码定义了三个函数，分别接收一个名为x的RoutingContext对象作为参数。这些函数的作用如下：

1. `GetAttributes()`函数：如果`x`不等于`nil`，则返回`x.Attributes`，否则返回`nil`。`x.Attributes`可能是一个元组的键值对，其中键是RoutingContext的属性名称，值可以是任何类型（包括字符串、整数、布尔值等）。
2. `GetOutboundGroupTags()`函数：如果`x`不等于`nil`，则返回`x.OutboundGroupTags`，否则返回`nil`。`x.OutboundGroupTags`可能是一个字符串数组，其中每个元素都是一个包含标签信息的元组。
3. `GetOutboundTag()`函数：如果`x`不等于`nil`，则返回`x.OutboundTag`，否则返回` ""`（即没有标签信息）。`x.OutboundTag`可能是一个字符串，用于表示路由的出站标签。


```go
func (x *RoutingContext) GetAttributes() map[string]string {
	if x != nil {
		return x.Attributes
	}
	return nil
}

func (x *RoutingContext) GetOutboundGroupTags() []string {
	if x != nil {
		return x.OutboundGroupTags
	}
	return nil
}

func (x *RoutingContext) GetOutboundTag() string {
	if x != nil {
		return x.OutboundTag
	}
	return ""
}

```

这段代码定义了一个名为 ` subscribesToRoutingStatisticsChannel` 的函数，它接收一个 `RoutingStatisticsChannel` 对象，并在其打开时订阅该频道。

该函数使用了 `FieldSelectors` 类来选择一些路由统计信息字段。这些选择器允许用户选择感兴趣的字段，以便从给定的路由统计信息中检索数据。

函数中包含的字段选择器如下：

- `inbound`: 选择连接的 `inbound` 标签。
- `network`: 选择连接的网络。
- `ip`: 等同于 `ip_source` 和 `ip_target`，选择两个IP，或选择 `source` 和 `target` IP。
- `port`: 等同于 `port_source` 和 `port_target`，选择两个端口，或选择 `source` 和 `target` 端口。
- `domain`: 选择目标域名。
- `protocol`: 选择连接的协议。
- `user`: 选择连接的 Inbound 用户电子邮件。
- `attributes`: 选择连接的附加属性。
- `outbound`: 等同于 `outbound` 和 `outbound_group`，选择两个选项，或选择 `source` 和 `target` 组。


```go
// SubscribeRoutingStatsRequest subscribes to routing statistics channel if
// opened by v2ray-core.
// * FieldSelectors selects a subset of fields in routing statistics to return.
// Valid selectors:
//  - inbound: Selects connection's inbound tag.
//  - network: Selects connection's network.
//  - ip: Equivalent as "ip_source" and "ip_target", selects both source and
//  target IP.
//  - port: Equivalent as "port_source" and "port_target", selects both source
//  and target port.
//  - domain: Selects target domain.
//  - protocol: Select connection's protocol.
//  - user: Select connection's inbound user email.
//  - attributes: Select connection's additional attributes.
//  - outbound: Equivalent as "outbound" and "outbound_group", select both
```

此代码定义了一个名为 `SubscribeRoutingStatsRequest` 的 struct 类型，用于在系统地广播接收到的路由统计信息时进行注册。该 struct 类型包含以下字段：

1. `state`：一个 `protoimpl.MessageState` 类型的字段，用于保存最近的 `protoimpl.MessageInfo` 的状态。
2. `sizeCache`：一个 `protoimpl.SizeCache` 类型的字段，用于缓存已经解析过的 `protoimpl.MessageInfo` 的大小信息，以避免重复计算。
3. `unknownFields`：一个 `protoimpl.UnknownFields` 类型的字段，用于保存可能还没有被明确定义的未知字段。
4. `FieldSelectors`：一个字符数组，用于标识要返回的数据字段。如果该字段为空，则所有字段都将返回。

函数 `Reset` 重置了 `SubscribeRoutingStatsRequest` 的状态，并可能触发了 `protoimpl.UnsafeEnabled` 设置为 true 的警告，该设置表示您应该手工处理未完全定义的字段。


```go
//  outbound tag and outbound group tags.
// * If FieldSelectors is left empty, all fields will be returned.
type SubscribeRoutingStatsRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	FieldSelectors []string `protobuf:"bytes,1,rep,name=FieldSelectors,proto3" json:"FieldSelectors,omitempty"`
}

func (x *SubscribeRoutingStatsRequest) Reset() {
	*x = SubscribeRoutingStatsRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_command_command_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为SubscribeRoutingStatsRequest的接口类型，并实现了两个方法：

1. String()，该方法返回一个字符串表示该接口类型的实例。
2. ProtoMessage()，该方法返回一个Protobuf消息类型，以便在向量化为字节切片时进行转换。
3. ProtoReflect()，该方法返回一个自定义的Reflect消息类型，允许对实例进行反射操作。

具体来说，这段代码主要实现了两个目的：

1. 定义了一个SubscribeRoutingStatsRequest接口类型，该接口类型包含一个*SubscribeRoutingStatsRequest类型的实例。
2. 实现了一个名为SubscribeRoutingStatsRequest的String()方法，用于将该接口类型的实例转换为字符串。
3. 实现了一个名为SubscribeRoutingStatsRequest的ProtoMessage()方法，用于将该接口类型的实例转换为Protobuf消息类型。
4. 实现了一个名为SubscribeRoutingStatsRequest的ProtoReflect()方法，用于获取该接口类型的实例 reflect 操作。


```go
func (x *SubscribeRoutingStatsRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*SubscribeRoutingStatsRequest) ProtoMessage() {}

func (x *SubscribeRoutingStatsRequest) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_command_command_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

此代码定义了一个名为 `SubscribeRoutingStatsRequest` 的接口，用于在流量监测中订阅路由统计信息并返回统计结果。

具体来说，这段代码实现了一个 `Descriptor` 函数，用于返回 `SubscribeRoutingStatsRequest` 的描述信息，以及一个用于遍历 `FieldSelectors` 的函数。

`Descriptor` 函数的实现主要通过反射来获取 `SubscribeRoutingStatsRequest` 的一些属性的信息，并将其按照一定格式进行编码，以便在传输过程中进行传输或者存储。

`GetFieldSelectors` 函数用于获取 `SubscribeRoutingStatsRequest` 中的 `FieldSelectors`，并返回其中的 `string` 类型。这个函数在 `descriptor` 函数中并没有使用，但是在代码的其它部分可能被使用到了，可能是用于在代码中进行一些验证或者记录。

最后，定义了一个名为 `TestRouteRequest` 的函数，这个函数模拟了一个测试路由的场景，接收一个 `RoutingContext` 参数，并根据该参数进行路由测试，以验证代码的正确性。


```go
// Deprecated: Use SubscribeRoutingStatsRequest.ProtoReflect.Descriptor instead.
func (*SubscribeRoutingStatsRequest) Descriptor() ([]byte, []int) {
	return file_app_router_command_command_proto_rawDescGZIP(), []int{1}
}

func (x *SubscribeRoutingStatsRequest) GetFieldSelectors() []string {
	if x != nil {
		return x.FieldSelectors
	}
	return nil
}

// TestRouteRequest manually tests a routing result according to the routing
// context message.
// * RoutingContext is the routing message without outbound information.
```

这段代码定义了一个名为 `TestRouteRequest` 的结构体，用于表示请求路由的结果。

该结构体包含以下字段：

* `state`：表示请求的路由状态，用于在路由处理程序中使用。
* `sizeCache`：表示请求的路由元数据存储空间，用于在路由处理程序中使用。
* `unknownFields`：表示该结构体中使用未知字段，这些字段可能是来自 `FileAppRouterCommand` 类型。

此外，还包含一个 `RoutingContext` 字段，表示请求所处的路由上下文，以及一个名为 `FieldSelectors` 的字段，表示路由器要返回的字段列表。

最后，包含一个名为 `PublishResult` 的字段，表示是否要发布路由结果到路由统计信息频道。如果设置为 true，则将发布路由结果；否则，不发布。


```go
// * FieldSelectors selects the fields to return in the routing result. All
// fields are returned if left empty.
// * PublishResult broadcasts the routing result to routing statistics channel
// if set true.
type TestRouteRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	RoutingContext *RoutingContext `protobuf:"bytes,1,opt,name=RoutingContext,proto3" json:"RoutingContext,omitempty"`
	FieldSelectors []string        `protobuf:"bytes,2,rep,name=FieldSelectors,proto3" json:"FieldSelectors,omitempty"`
	PublishResult  bool            `protobuf:"varint,3,opt,name=PublishResult,proto3" json:"PublishResult,omitempty"`
}

func (x *TestRouteRequest) Reset() {
	*x = TestRouteRequest{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_command_command_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了三个函数，分别接收一个`TestRouteRequest`类型的参数`x`，并返回不同的字符串表示。

第一个函数是一个普通函数，接收一个`TestRouteRequest`类型的参数`x`，并返回一个字符串表示。这个函数的作用是将`x`转换为`TestRouteRequest`类型，然后将其字符串化，最后将得到的字符串返回。

第二个函数接收一个`TestRouteRequest`类型的参数`x`，并返回一个`FileAppRouterCommandCommand`类型的`TestRouteRequest`指针。这个函数的作用是将`x`转换为`TestRouteRequest`类型，然后返回一个指向`FileAppRouterCommand`类型的指针。

第三个函数接收一个`TestRouteRequest`类型的参数`x`，并返回一个`FileAppRouterCommand`类型，这个类型的`TestRouteRequest`指针`x`的`Message`类型的`Message`结构字段。这个函数的作用是将`x`转换为`TestRouteRequest`类型，然后返回一个指向`FileAppRouterCommand`类型的指针，这个指针的`Message`字段即为`x`的`Message`类型的`Message`结构体。


```go
func (x *TestRouteRequest) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*TestRouteRequest) ProtoMessage() {}

func (x *TestRouteRequest) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_command_command_proto_msgTypes[2]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

这段代码定义了一个名为`TestRouteRequest`的接口，用于在测试过程中描述路由的相关信息。

首先，该接口有一个名为`Descriptor`的函数，该函数返回一个表示路由描述的字节数组和两个整数，第一个表示路由的唯一性标识符，第二个表示路由的优先级。这个函数使用了`file_app_router_command_command_proto_rawDescGZIP()`函数和`[]int`类型，其中`file_app_router_command_command_proto_rawDescGZIP()`函数将一个字节数组转换为GZIP压缩的字节数组，增强了代码的可读性。

其次，该接口有一个名为`GetRoutingContext`的函数，用于获取当前路由的上下文信息，如果当前对象不为空，则返回该对象的`RoutingContext`，否则返回`nil`。

最后，该接口有一个名为`GetFieldSelectors`的函数，用于获取路由中当前路由的输入字段选择器，如果当前路由不为空，则返回该路由的`FieldSelectors`字段，否则返回`nil`。


```go
// Deprecated: Use TestRouteRequest.ProtoReflect.Descriptor instead.
func (*TestRouteRequest) Descriptor() ([]byte, []int) {
	return file_app_router_command_command_proto_rawDescGZIP(), []int{2}
}

func (x *TestRouteRequest) GetRoutingContext() *RoutingContext {
	if x != nil {
		return x.RoutingContext
	}
	return nil
}

func (x *TestRouteRequest) GetFieldSelectors() []string {
	if x != nil {
		return x.FieldSelectors
	}
	return nil
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`x`的指针参数，并返回一个布尔值。函数的具体实现如下：


func (x *TestRouteRequest) GetPublishResult() bool {
	if x != nil {
		return x.PublishResult
	}
	return false
}


首先，函数检查传入的`x`是否为空。如果是，则输出一个布尔值`false`。否则，函数调用`x.PublishResult`获取发布结果，并将其输出。

接下来，定义一个名为`Config`的结构体，其中包含`state`、`sizeCache`和`unknownFields`字段。


type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields
}


接着，定义一个名为`Reset`的函数，用于清空`Config`结构体的所有字段。


func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_command_command_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}


最后，上述函数被定义为`func TestRouteRequest_Reset_为例程的函数。


```go
func (x *TestRouteRequest) GetPublishResult() bool {
	if x != nil {
		return x.PublishResult
	}
	return false
}

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields
}

func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_command_command_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为 func 的函数接收一个名为 x 的 *Config 类型的参数，并返回一个字符串类型的结果。

首先，函数内部定义了一个名为 ProtoImpl 的内部接口，以及一个名为 ProtoMessage 的内部接口。这些接口在函数外部声明，以便外部可以知道如何使用它们。

接下来，函数内部定义了一个名为 ProtoReflect 的函数，这个函数接收一个 *Config 类型的参数，并返回一个 protoreflect.Message 类型的结果。

在 ProtoReflect函数内部，使用一个名为 mi 的变量来引用一个名为 file_app_router_command_command_proto 的协议类型，并使用 *Config 和 x 指向的指针来获取该协议类型的实例。然后，使用一个 if 语句判断是否启用了 ProtoImpl 中启用了安全漏洞的标记，如果是，则使用 x 指向的值调用 MessageStringOf 函数，获取消息类型对象，并将其存储到 mi 中。最后，如果 x 指向的值未被使用，则直接使用 x 指向的值调用 MessageOf 函数，获取消息类型对象，并将其存储到 mi 中。

最后，函数内部返回 mi 的 MessageOf 函数，这个函数将 x 指向的值转换为字节切片，并将其转换为字符串，以便将消息类型转换为字符串。


```go
func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_command_command_proto_msgTypes[3]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

```

0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d,
	0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1d, 0x56, 0x32, 0x52, 0x61, 0x79,
	0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1d, 0x56, 0x32, 0x52, 0x61, 0x79,
	0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1d, 0x56, 0x32, 0x52, 0x61, 0x79,
	0x2e, 0x41, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1d, 0x56, 0x32, 0x52, 0x61, 0x79,
	0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72,




```go
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
	return file_app_router_command_command_proto_rawDescGZIP(), []int{3}
}

var File_app_router_command_command_proto protoreflect.FileDescriptor

var file_app_router_command_command_proto_rawDesc = []byte{
	0x0a, 0x20, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2f, 0x63, 0x6f, 0x6d,
	0x6d, 0x61, 0x6e, 0x64, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x70, 0x72, 0x6f,
	0x74, 0x6f, 0x12, 0x1d, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61,
	0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e,
	0x64, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x6e, 0x65,
	0x74, 0x77, 0x6f, 0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0xa8, 0x04, 0x0a, 0x0e,
	0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x12, 0x1e,
	0x0a, 0x0a, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x18, 0x01, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x0a, 0x49, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x12, 0x38,
	0x0a, 0x07, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0e, 0x32,
	0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d,
	0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x52,
	0x07, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x12, 0x1c, 0x0a, 0x09, 0x53, 0x6f, 0x75, 0x72,
	0x63, 0x65, 0x49, 0x50, 0x73, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0c, 0x52, 0x09, 0x53, 0x6f, 0x75,
	0x72, 0x63, 0x65, 0x49, 0x50, 0x73, 0x12, 0x1c, 0x0a, 0x09, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74,
	0x49, 0x50, 0x73, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0c, 0x52, 0x09, 0x54, 0x61, 0x72, 0x67, 0x65,
	0x74, 0x49, 0x50, 0x73, 0x12, 0x1e, 0x0a, 0x0a, 0x53, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x50, 0x6f,
	0x72, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x0a, 0x53, 0x6f, 0x75, 0x72, 0x63, 0x65,
	0x50, 0x6f, 0x72, 0x74, 0x12, 0x1e, 0x0a, 0x0a, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74, 0x50, 0x6f,
	0x72, 0x74, 0x18, 0x06, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x0a, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74,
	0x50, 0x6f, 0x72, 0x74, 0x12, 0x22, 0x0a, 0x0c, 0x54, 0x61, 0x72, 0x67, 0x65, 0x74, 0x44, 0x6f,
	0x6d, 0x61, 0x69, 0x6e, 0x18, 0x07, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0c, 0x54, 0x61, 0x72, 0x67,
	0x65, 0x74, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x1a, 0x0a, 0x08, 0x50, 0x72, 0x6f, 0x74,
	0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x08, 0x20, 0x01, 0x28, 0x09, 0x52, 0x08, 0x50, 0x72, 0x6f, 0x74,
	0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x12, 0x0a, 0x04, 0x55, 0x73, 0x65, 0x72, 0x18, 0x09, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x04, 0x55, 0x73, 0x65, 0x72, 0x12, 0x5d, 0x0a, 0x0a, 0x41, 0x74, 0x74, 0x72,
	0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x18, 0x0a, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x3d, 0x2e, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
	0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75,
	0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x2e, 0x41, 0x74, 0x74, 0x72,
	0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x52, 0x0a, 0x41, 0x74, 0x74,
	0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x12, 0x2c, 0x0a, 0x11, 0x4f, 0x75, 0x74, 0x62, 0x6f,
	0x75, 0x6e, 0x64, 0x47, 0x72, 0x6f, 0x75, 0x70, 0x54, 0x61, 0x67, 0x73, 0x18, 0x0b, 0x20, 0x03,
	0x28, 0x09, 0x52, 0x11, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x47, 0x72, 0x6f, 0x75,
	0x70, 0x54, 0x61, 0x67, 0x73, 0x12, 0x20, 0x0a, 0x0b, 0x4f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e,
	0x64, 0x54, 0x61, 0x67, 0x18, 0x0c, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0b, 0x4f, 0x75, 0x74, 0x62,
	0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x1a, 0x3d, 0x0a, 0x0f, 0x41, 0x74, 0x74, 0x72, 0x69,
	0x62, 0x75, 0x74, 0x65, 0x73, 0x45, 0x6e, 0x74, 0x72, 0x79, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65,
	0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x14, 0x0a, 0x05,
	0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x05, 0x76, 0x61, 0x6c,
	0x75, 0x65, 0x3a, 0x02, 0x38, 0x01, 0x22, 0x46, 0x0a, 0x1c, 0x53, 0x75, 0x62, 0x73, 0x63, 0x72,
	0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x74, 0x61, 0x74, 0x73, 0x52,
	0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x12, 0x26, 0x0a, 0x0e, 0x46, 0x69, 0x65, 0x6c, 0x64, 0x53,
	0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x09, 0x52, 0x0e,
	0x46, 0x69, 0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x22, 0xb7,
	0x01, 0x0a, 0x10, 0x54, 0x65, 0x73, 0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x52, 0x65, 0x71, 0x75,
	0x65, 0x73, 0x74, 0x12, 0x55, 0x0a, 0x0e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f,
	0x6e, 0x74, 0x65, 0x78, 0x74, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75,
	0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75, 0x74,
	0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x52, 0x0e, 0x52, 0x6f, 0x75, 0x74,
	0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x12, 0x26, 0x0a, 0x0e, 0x46, 0x69,
	0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x73, 0x18, 0x02, 0x20, 0x03,
	0x28, 0x09, 0x52, 0x0e, 0x46, 0x69, 0x65, 0x6c, 0x64, 0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f,
	0x72, 0x73, 0x12, 0x24, 0x0a, 0x0d, 0x50, 0x75, 0x62, 0x6c, 0x69, 0x73, 0x68, 0x52, 0x65, 0x73,
	0x75, 0x6c, 0x74, 0x18, 0x03, 0x20, 0x01, 0x28, 0x08, 0x52, 0x0d, 0x50, 0x75, 0x62, 0x6c, 0x69,
	0x73, 0x68, 0x52, 0x65, 0x73, 0x75, 0x6c, 0x74, 0x22, 0x08, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66,
	0x69, 0x67, 0x32, 0x89, 0x02, 0x0a, 0x0e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x65,
	0x72, 0x76, 0x69, 0x63, 0x65, 0x12, 0x87, 0x01, 0x0a, 0x15, 0x53, 0x75, 0x62, 0x73, 0x63, 0x72,
	0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x53, 0x74, 0x61, 0x74, 0x73, 0x12,
	0x3b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
	0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e,
	0x53, 0x75, 0x62, 0x73, 0x63, 0x72, 0x69, 0x62, 0x65, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67,
	0x53, 0x74, 0x61, 0x74, 0x73, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2d, 0x2e, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
	0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f, 0x75,
	0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x22, 0x00, 0x30, 0x01, 0x12,
	0x6d, 0x0a, 0x09, 0x54, 0x65, 0x73, 0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x12, 0x2f, 0x2e, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
	0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x54, 0x65, 0x73,
	0x74, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x52, 0x65, 0x71, 0x75, 0x65, 0x73, 0x74, 0x1a, 0x2d, 0x2e,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
	0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x2e, 0x52, 0x6f,
	0x75, 0x74, 0x69, 0x6e, 0x67, 0x43, 0x6f, 0x6e, 0x74, 0x65, 0x78, 0x74, 0x22, 0x00, 0x42, 0x68,
	0x0a, 0x21, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65,
	0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
	0x61, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d,
	0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0xaa, 0x02, 0x1d, 0x56, 0x32, 0x52, 0x61, 0x79,
	0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x61, 0x6e, 0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

这段代码定义了一个名为"file_app_router_command_command_proto_rawDesc"的变量，其作用是返回一个名为"file_app_router_command_command_proto_rawDescGZIP"的函数的输出结果，该函数接收一个名为"file_app_router_command_command_proto_rawDescData"的输入参数，对其进行处理后返回一个字节数组。

具体来说，代码中定义了一个名为"file_app_router_command_command_proto_rawDescOnce"的变量，其作用是确保"file_app_router_command_command_proto_rawDescData"变量只被访问一次，即在函数内部对"file_app_router_command_command_proto_rawDescData"进行初始化操作，并确保每次调用该函数时都有新的值返回。

接着，代码定义了一个名为"file_app_router_command_command_proto_rawDescGZIP"的函数，该函数接收一个名为"file_app_router_command_command_proto_rawDescData"的输入参数，使用一个名为"protoimpl.X.CompressGZIP"的函数对输入参数进行压缩GZIP编码，然后返回压缩后的字节数组。

最后，代码定义了一个名为"file_app_router_command_command_proto_msgTypes"的变量，其作用是初始化一个名为"file_app_router_command_command_proto_goTypes"的数组，该数组包含5个元素，每个元素对应于"file_app_router_command_command_proto_rawDesc"函数输入参数中的每个元素，其类型定义了该元素所代表的接口类型。同时，代码还定义了一个名为"file_app_router_command_command_proto_goTypes"的变量，其作用是与"file_app_router_command_command_proto_msgTypes"数组相同的变量，但只存储了第一个元素的类型信息，即"v2ray.core.app.router.command.RoutingContext"。


```go
var (
	file_app_router_command_command_proto_rawDescOnce sync.Once
	file_app_router_command_command_proto_rawDescData = file_app_router_command_command_proto_rawDesc
)

func file_app_router_command_command_proto_rawDescGZIP() []byte {
	file_app_router_command_command_proto_rawDescOnce.Do(func() {
		file_app_router_command_command_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_router_command_command_proto_rawDescData)
	})
	return file_app_router_command_command_proto_rawDescData
}

var file_app_router_command_command_proto_msgTypes = make([]protoimpl.MessageInfo, 5)
var file_app_router_command_command_proto_goTypes = []interface{}{
	(*RoutingContext)(nil),               // 0: v2ray.core.app.router.command.RoutingContext
	(*SubscribeRoutingStatsRequest)(nil), // 1: v2ray.core.app.router.command.SubscribeRoutingStatsRequest
	(*TestRouteRequest)(nil),             // 2: v2ray.core.app.router.command.TestRouteRequest
	(*Config)(nil),                       // 3: v2ray.core.app.router.command.Config
	nil,                                  // 4: v2ray.core.app.router.command.RoutingContext.AttributesEntry
	(net.Network)(0),                     // 5: v2ray.core.common.net.Network
}
```

这段代码定义了一个名为 "file_app_router_command_command_proto_depIdxs" 的变量，其类型为 "int32"（32 位整数）。

这个变量是一个数组，它包含了从0到5的各种枚举类型。每个枚举类型代表了一个 "RoutingContext" 对象中的不同属性或方法。这些属性和方法用于在 "v2ray.core.app.router.command" 包中定义的一些路由相关的操作。

具体来说，这个数组包含了以下内容：

- 0: "v2ray.core.app.router.command.RoutingContext.Network:type_name" -> "v2ray.core.common.net.Network"
- 1: "v2ray.core.app.router.command.RoutingContext.Attributes:type_name" -> "v2ray.core.app.router.command.RoutingContext.AttributesEntry"
- 2: "v2ray.core.app.router.command.TestRouteRequest.RoutingContext:type_name" -> "v2ray.core.app.router.command.RoutingContext"
- 3: "v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:input_type" -> "v2ray.core.app.router.command.SubscribeRoutingStatsRequest"
- 4: "v2ray.core.app.router.command.RoutingService.TestRoute:input_type" -> "v2ray.core.app.router.command.TestRouteRequest"
- 5: "v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:output_type" -> "v2ray.core.app.router.command.RoutingContext"
- 6: "v2ray.core.app.router.command.RoutingService.TestRoute:output_type" -> "v2ray.core.app.router.command.RoutingContext"

数组中的第5到第7个元素是一个下标，它们指定了方法 "output_type"、"input_type" 和 "extendee" 的下标。


```go
var file_app_router_command_command_proto_depIdxs = []int32{
	5, // 0: v2ray.core.app.router.command.RoutingContext.Network:type_name -> v2ray.core.common.net.Network
	4, // 1: v2ray.core.app.router.command.RoutingContext.Attributes:type_name -> v2ray.core.app.router.command.RoutingContext.AttributesEntry
	0, // 2: v2ray.core.app.router.command.TestRouteRequest.RoutingContext:type_name -> v2ray.core.app.router.command.RoutingContext
	1, // 3: v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:input_type -> v2ray.core.app.router.command.SubscribeRoutingStatsRequest
	2, // 4: v2ray.core.app.router.command.RoutingService.TestRoute:input_type -> v2ray.core.app.router.command.TestRouteRequest
	0, // 5: v2ray.core.app.router.command.RoutingService.SubscribeRoutingStats:output_type -> v2ray.core.app.router.command.RoutingContext
	0, // 6: v2ray.core.app.router.command.RoutingService.TestRoute:output_type -> v2ray.core.app.router.command.RoutingContext
	5, // [5:7] is the sub-list for method output_type
	3, // [3:5] is the sub-list for method input_type
	3, // [3:3] is the sub-list for extension type_name
	3, // [3:3] is the sub-list for extension extendee
	0, // [0:3] is the sub-list for field type_name
}

```

This code exports the `file_app_router_command_command_proto` message to thego package.

The exported message is identified by the name `file_app_router_command_command_proto`, and it has a default message type of `file_app_router_command_command_proto.You can also specify the message type by argument, as described in the `msg_export.go` file in the protobuf plugin.

The message has 5 methods, including the binary `Exporter` field.

The `Exporter` field is an optional field that specifies the field that should be exposed as an external，秘API item. It is a `h借助` field, which means that it should be interpreted as a required field according to the information in the message definition, but it is not strictly required.

The `file_app_router_command_command_proto` message has the following fields:

* `state`: message `state` field type `file_app_router_command_command_proto.Config` struct field.
* `sizeCache`: message `sizeCache` field type `file_app_router_command_command_proto.Config` struct field.
* `unknownFields`: message `unknownFields` field type `file_app_router_command_command_proto.Config` struct field.
* `file_app_router_command_command_proto`: message `file_app_router_command_command_proto` field type `file_app_router_command_command_proto.Config` struct field.
* `file_app_router_command_command_proto_rawDesc`: message `file_app_router_command_command_proto_rawDesc` field type `file_app_router_command_command_proto.Config` struct field.
* `file_app_router_command_command_proto_goTypes`: message `file_app_router_command_command_proto_goTypes` field type `file_app_router_command_command_proto.Config` struct field.
* `file_app_router_command_command_proto_depIdxs`: message `file_app_router_command_command_proto_depIdxs` field type `file_app_router_command_command_proto.Config` struct field.

The `file_app_router_command_command_proto_file_app_router_command_command_proto.Config` field specifies the fields of the `Config` message, which should be exported as an external，秘API item.

The `file_app_router_command_command_proto_file_app_router_command_command_proto.Exporter` field is an optional field that specifies the field that should be exposed as an external，秘API item.

The `file_app_router_command_command_proto_file_app_router_command_command_proto.file_app_router_command_command_proto_rawDesc` field is a raw descriptor of the `file_app_router_command_command_proto.Config` message, which should be exposed as an external，秘API item.

The `file_app_router_command_command_proto_file_app_router_command_command_proto.file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_proto.file_app_router_command_command_goTypes` field type.

The `file_app_router_command_command_proto_file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_goTypes` field type.

The `file_app_router_command_command_proto_file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_goTypes` field type.

The `file_app_router_command_command_proto_file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_goTypes` field type.

The `file_app_router_command_command_proto_file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_goTypes` field type.

The `file_app_router_command_command_proto_file_app_router_command_command_goTypes` field is a generated type that represents the `file_app_router_command_command_proto.Config` message, which is a field type of the `file_app_router_command_command_goTypes` field type.


```go
func init() { file_app_router_command_command_proto_init() }
func file_app_router_command_command_proto_init() {
	if File_app_router_command_command_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_app_router_command_command_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*RoutingContext); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_app_router_command_command_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*SubscribeRoutingStatsRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_app_router_command_command_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*TestRouteRequest); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_app_router_command_command_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Config); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_app_router_command_command_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   5,
			NumExtensions: 0,
			NumServices:   1,
		},
		GoTypes:           file_app_router_command_command_proto_goTypes,
		DependencyIndexes: file_app_router_command_command_proto_depIdxs,
		MessageInfos:      file_app_router_command_command_proto_msgTypes,
	}.Build()
	File_app_router_command_command_proto = out.File
	file_app_router_command_command_proto_rawDesc = nil
	file_app_router_command_command_proto_goTypes = nil
	file_app_router_command_command_proto_depIdxs = nil
}

```