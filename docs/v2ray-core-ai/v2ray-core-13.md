# v2ray-core源码解析 13

# `app/router/condition_geoip_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试名为 "router" 的包。它主要是通过 Import 导入一些标准库以及自己的库，然后通过 protobuf 协议来定义一些用于测试的接口，并定义了测试所需的常量和错误代码。下面是具体的解释：

1. `package router_test`：定义了该测试框架的包名。
2. `import (`：导入了一些标准库，包括 os、path、filepath 和 testing。
3. `proto "github.com/golang/protobuf/proto"`：导入了一项名为 protobuf 的库，该库定义了用于测试的接口规范。
4. `router.proto`：定义了一个名为 router.proto 的接口，该接口用于定义路由信息，包括路由ID、目标 URL 等。
5. `v2ray.com/core/app/router`：定义了一个名为 router 的类，该类实现了路由器的功能，通过 `router.route` 方法设置路由，通过 `router.routeMatchers` 方法设置路由匹配器。
6. `router.test`：定义了一个名为 router_test 的测试类，该类继承自 `testing.T` 类的子类，提供了测试前的准备工作和测试后的清理工作。
7. `os.Args`：定义了一个名为 os_args 的函数，该函数用于获取用户输入的命令行参数。
8. `path.filepath`：定义了一个名为 path 的函数，该函数用于从用户输入的文件路径中返回文件路径。
9. `testing.T`：继承自 `testing.T` 类的包，提供了测试前的准备工作和测试后的清理工作。
10. `strings`：定义了一个名为 strings 的函数，该函数提供了一组字符串操作的函数，包括拆分、连接、截取等。
11. `net/http`：定义了一个名为 net_http 的包，该包提供了一个 HTTP 客户端的接口，包括请求、响应等操作。
12. `net/url`：定义了一个名为 url 的包，该包提供了一个 URL 对象的接口，包括解析 URL、设置 URL 等操作。
13. `net/http/httptest`：定义了一个名为 http_test 的包，该包提供了一个 HTTP 客户端的测试框架，包括断言请求、断言响应等。
14. `path/internal/github.com/golang/protobuf/protobuf`：导入了来自 https://github.com/golang/protobuf/proto 的 protobuf 库。
15. `path/internal/github.com/golang/protobuf/proto/router_protobuf`：导入了来自 https://github.com/golang/protobuf/proto/router_protobuf 的路由信息定义文件。

综上所述，这段代码定义了一个用于测试 "router" 包的测试框架，通过导入相关的库和定义接口，使得测试可以更加简单、明确。


```go
package router_test

import (
	"os"
	"path/filepath"
	"testing"

	proto "github.com/golang/protobuf/proto"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/platform"
	"v2ray.com/core/common/platform/filesystem"
)

```

该代码定义了一个名为“init”的函数，它在函数开始时，使用 os.Getwd() 函数获取当前工作目录(wd)，并将其存储到变量 wd 中。然后，它使用 os.Stat 和 filesystem.CopyFile 函数检查 geoip.dat 和 geotype.dat 文件是否存在。如果它们不存在，则使用 filesystem.CopyFile 函数将它们从AssetLocation复制到wd目录中的“release”目录下的“config”目录中。

接下来，该函数使用 os.Stat 和 filesystem.CopyFile 函数检查“geoip.dat”和“geotype.dat”文件是否存在。如果它们不存在，则使用 filesystem.CopyFile 函数将它们从AssetLocation复制到wd目录中的“release”目录中。

最后，该函数使用 NewGeoIPMatcherContainer 函数创建一个名为“container”的“GeoIPMatcherContainer”实例，然后使用 Add 函数添加两个“GeoIP”实例。然后，该函数使用一系列 if 语句检查容器中是否包含相同的matcher。如果容器中包含的 matcher 不同，则使用 assert 函数在测试中检查。


```go
func init() {
	wd, err := os.Getwd()
	common.Must(err)

	if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) {
		common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat")))
	}
	if _, err := os.Stat(platform.GetAssetLocation("geosite.dat")); err != nil && os.IsNotExist(err) {
		common.Must(filesystem.CopyFile(platform.GetAssetLocation("geosite.dat"), filepath.Join(wd, "..", "..", "release", "config", "geosite.dat")))
	}
}

func TestGeoIPMatcherContainer(t *testing.T) {
	container := &router.GeoIPMatcherContainer{}

	m1, err := container.Add(&router.GeoIP{
		CountryCode: "CN",
	})
	common.Must(err)

	m2, err := container.Add(&router.GeoIP{
		CountryCode: "US",
	})
	common.Must(err)

	m3, err := container.Add(&router.GeoIP{
		CountryCode: "CN",
	})
	common.Must(err)

	if m1 != m3 {
		t.Error("expect same matcher for same geoip, but not")
	}

	if m1 == m2 {
		t.Error("expect different matcher for different geoip, but actually same")
	}
}

```

This is a Go test program that tests the functionality of a GeoIP matcher. It uses the `router.GeoIPMatcher` class to perform tests against a list of IP addresses with the CIDR notation, and checks whether the output of the matcher matches the expected value for each input IP address.

The program reads a list of IP addresses from a file, and uses the `router.GeoIPMatcher` class to perform tests against each IP address. It uses the `Must` method to ensure that the matcher is initialized correctly, and the `GeoIPMatcher#Init` method to set up the matcher for each test case.

The program then defines a struct with a list of inputs and the expected output for each input. It then defines a test loop that goes through each test case and performs the same tests against each input IP address. Finally, it prints out the results of the tests and compares them to the expected outputs.


```go
func TestGeoIPMatcher(t *testing.T) {
	cidrList := router.CIDRList{
		{Ip: []byte{0, 0, 0, 0}, Prefix: 8},
		{Ip: []byte{10, 0, 0, 0}, Prefix: 8},
		{Ip: []byte{100, 64, 0, 0}, Prefix: 10},
		{Ip: []byte{127, 0, 0, 0}, Prefix: 8},
		{Ip: []byte{169, 254, 0, 0}, Prefix: 16},
		{Ip: []byte{172, 16, 0, 0}, Prefix: 12},
		{Ip: []byte{192, 0, 0, 0}, Prefix: 24},
		{Ip: []byte{192, 0, 2, 0}, Prefix: 24},
		{Ip: []byte{192, 168, 0, 0}, Prefix: 16},
		{Ip: []byte{192, 18, 0, 0}, Prefix: 15},
		{Ip: []byte{198, 51, 100, 0}, Prefix: 24},
		{Ip: []byte{203, 0, 113, 0}, Prefix: 24},
		{Ip: []byte{8, 8, 8, 8}, Prefix: 32},
		{Ip: []byte{91, 108, 4, 0}, Prefix: 16},
	}

	matcher := &router.GeoIPMatcher{}
	common.Must(matcher.Init(cidrList))

	testCases := []struct {
		Input  string
		Output bool
	}{
		{
			Input:  "192.168.1.1",
			Output: true,
		},
		{
			Input:  "192.0.0.0",
			Output: true,
		},
		{
			Input:  "192.0.1.0",
			Output: false,
		}, {
			Input:  "0.1.0.0",
			Output: true,
		},
		{
			Input:  "1.0.0.1",
			Output: false,
		},
		{
			Input:  "8.8.8.7",
			Output: false,
		},
		{
			Input:  "8.8.8.8",
			Output: true,
		},
		{
			Input:  "2001:cdba::3257:9652",
			Output: false,
		},
		{
			Input:  "91.108.255.254",
			Output: true,
		},
	}

	for _, testCase := range testCases {
		ip := net.ParseAddress(testCase.Input).IP()
		actual := matcher.Match(ip)
		if actual != testCase.Output {
			t.Error("expect input", testCase.Input, "to be", testCase.Output, ", but actually", actual)
		}
	}
}

```

该代码的两个函数分别测试了基于GeoIP的验证码匹配器函数。该函数使用GeoIP地址存储在ips变量中，然后创建一个GeoIP匹配器对象matcher并使用初始化函数初始化它。测试使用两种不同的GeoIP地址进行匹配。

如果匹配测试成功通过，则两个函数都会成功打印出错误消息。否则，第一个函数将在测试中打印出预期失败的消息，而第二个函数将打印出预期成功的消息。


```go
func TestGeoIPMatcher4CN(t *testing.T) {
	ips, err := loadGeoIP("CN")
	common.Must(err)

	matcher := &router.GeoIPMatcher{}
	common.Must(matcher.Init(ips))

	if matcher.Match([]byte{8, 8, 8, 8}) {
		t.Error("expect CN geoip doesn't contain 8.8.8.8, but actually does")
	}
}

func TestGeoIPMatcher6US(t *testing.T) {
	ips, err := loadGeoIP("US")
	common.Must(err)

	matcher := &router.GeoIPMatcher{}
	common.Must(matcher.Init(ips))

	if !matcher.Match(net.ParseAddress("2001:4860:4860::8888").IP()) {
		t.Error("expect US geoip contain 2001:4860:4860::8888, but actually not")
	}
}

```

此代码定义了一个名为 `loadGeoIP` 的函数，接受一个 `country` 参数，它是一个字符串。函数的作用是读取一个名为 `geoip.dat` 的文件，其中包含许多 `GeoIP` 记录。函数返回一个表示 `GeoIP` 记录的 `CIDR` 切片和可能的错误。

函数的实现首先读取文件并将其字节数据存储在变量 `geoipBytes` 中。然后，使用 `proto.Unmarshal` 函数将字节数据转换为 `GeoIP` 类型，并将其存储在变量 `geoipList` 中。

接下来，函数遍历 `geoipList` 中的每个 `GeoIP` 记录。如果记录中的 `CountryCode` 等于传入的 `country`，函数将返回该 `GeoIP` 的 `CIDR`，并输出一条错误消息。如果函数无法找到 `country`，它将输出一条错误消息并返回 `nil`。


```go
func loadGeoIP(country string) ([]*router.CIDR, error) {
	geoipBytes, err := filesystem.ReadAsset("geoip.dat")
	if err != nil {
		return nil, err
	}
	var geoipList router.GeoIPList
	if err := proto.Unmarshal(geoipBytes, &geoipList); err != nil {
		return nil, err
	}

	for _, geoip := range geoipList.Entry {
		if geoip.CountryCode == country {
			return geoip.Cidr, nil
		}
	}

	panic("country not found: " + country)
}

```

这两段代码测试了一个名为"GeoIPMatcher"的函数，该函数对GeoIP进行匹配。

每个测试函数都有以下参数：

- `b *testing.B`：表示该测试函数组中的测试用例数。
- `ips, err`：第一个参数是一个字符串，代表要测试的GeoIP地址。这个参数是 loading() 函数的回调函数的参数。该函数将下载的IP地址存储在 `ips` 变量中，并将任何错误记录在 `err` 变量中。
- `matcher, err`：第二个参数是一个指向 `router.GeoIPMatcher` 类型的变量。这个参数在函数中使用，如果出现错误，将调用该函数的 `Init()` 函数初始化 GeoIP 匹配器。
- `common.Must(err)`：这个函数是 `Must` 函数的别称，用于强制执行函数中的代码，即使 `err` 变量为 `nil`。
- `for i := 0; i < b.N; i++`：表示一个循环，用于运行测试用例。
- `_ = matcher.Match([]byte{8, 8, 8, 8})`：匹配一个字符串。
- `_ = matcher.Match(net.ParseAddress("2001:4860:4860::8888").IP())`：匹配一个IP地址。

该测试函数的作用是验证 "GeoIPMatcher" 函数的实现是否正确，即是否可以检测到给定的IP地址。


```go
func BenchmarkGeoIPMatcher4CN(b *testing.B) {
	ips, err := loadGeoIP("CN")
	common.Must(err)

	matcher := &router.GeoIPMatcher{}
	common.Must(matcher.Init(ips))

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		_ = matcher.Match([]byte{8, 8, 8, 8})
	}
}

func BenchmarkGeoIPMatcher6US(b *testing.B) {
	ips, err := loadGeoIP("US")
	common.Must(err)

	matcher := &router.GeoIPMatcher{}
	common.Must(matcher.Init(ips))

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		_ = matcher.Match(net.ParseAddress("2001:4860:4860::8888").IP())
	}
}

```

# `app/router/condition_test.go`

This package is a testing package for the "router" protocol.

It imports several packages:

* "os"
* "path/filepath"
* "strconv"
* "testing"
* "proto"
* "github.com/golang/protobuf/proto"
* "v2ray.com/core/app/router"
* "v2ray.com/core/common"
* "v2ray.com/core/common/errors"
* "v2ray.com/core/common/net"
* "v2ray.com/core/common/platform"
* "v2ray.com/core/common/platform/filesystem"
* "v2ray.com/core/common/protocol"
* "v2ray.com/core/common/protocol/http"
* "v2ray.com/core/common/session"
* "v2ray.com/core/features/routing"
* "v2ray.com/core/features/routing/session"

This package defines a function called "testingRouter" which takes a routing instance and a file path. It then performs various tests, such as checking if the router can connect to a remote server, sending a request to the router, and checking if the router responds to the request.


```go
package router_test

import (
	"os"
	"path/filepath"
	"strconv"
	"testing"

	proto "github.com/golang/protobuf/proto"

	. "v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/platform"
	"v2ray.com/core/common/platform/filesystem"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/protocol/http"
	"v2ray.com/core/common/session"
	"v2ray.com/core/features/routing"
	routing_session "v2ray.com/core/features/routing/session"
)

```

这段代码定义了一个名为`init`的函数，用于在开始一个名为`router`的Router之前做一些设置工作。

具体来说，代码首先获取当前工作目录(wd)，并将结果保存到变量`err`中。然后，代码调用`os.Getwd`函数获取wd的值并将其保存到变量`wd`中。

接下来，代码使用条件语句检查`geoip.dat`和`geotype.dat`文件是否存在于当前目录下。如果不存在，代码将使用`filesystem.CopyFile`函数将从`geoip.dat`文件复制到当前目录中，并覆盖名为`..`的子目录。如果`geoip.dat`或`geotype.dat`文件已存在，但不符合某个条件，则不会执行任何操作。

最后，代码使用`withBackground`函数返回一个`routing.Context`，这个上下文将在路由处理程序中被使用。


```go
func init() {
	wd, err := os.Getwd()
	common.Must(err)

	if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) {
		common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat")))
	}
	if _, err := os.Stat(platform.GetAssetLocation("geosite.dat")); err != nil && os.IsNotExist(err) {
		common.Must(filesystem.CopyFile(platform.GetAssetLocation("geosite.dat"), filepath.Join(wd, "..", "..", "release", "config", "geosite.dat")))
	}
}

func withBackground() routing.Context {
	return &routing_session.Context{}
}

```

This is a Go program that performs a simple network application load test. The program defines a series of rules, each of which tests a different aspect of the network application. The rules are defined using the `RoutingRule` struct, which specifies the protocol (in this case, HTTP), the path pattern, and any attributes. The `WithInbound` and `WithContent` functions are used to apply the rules to incoming requests and the response content, respectively. The `output` field specifies whether the output from the rule should be ignored or not.

The program uses a series of test cases to verify that each rule behaves correctly. The `rule.BuildCondition` function is used to build the condition for each test case, based on the input provided. The condition is then applied to the input using the `WithContent` or `WithInbound` function, and the output is compared to the expected output using the `output` field of the `RoutingRule` struct.

The program includes several helper functions, such as `withInbound`, `withContent`, `withExternalLink`, and `withManyOrigins`, which are used to apply the rules to specific inputs. The `withManyOrigins` function is used to apply a rule to all origins.

The program also defines a `Content` struct that represents the content being sent in the network application. This struct includes fields for the protocol, the path, and any attributes.

Overall, this program provides a simple and straightforward way to test the functionality of a network application.


```go
func withOutbound(outbound *session.Outbound) routing.Context {
	return &routing_session.Context{Outbound: outbound}
}

func withInbound(inbound *session.Inbound) routing.Context {
	return &routing_session.Context{Inbound: inbound}
}

func withContent(content *session.Content) routing.Context {
	return &routing_session.Context{Content: content}
}

func TestRoutingRule(t *testing.T) {
	type ruleTest struct {
		input  routing.Context
		output bool
	}

	cases := []struct {
		rule *RoutingRule
		test []ruleTest
	}{
		{
			rule: &RoutingRule{
				Domain: []*Domain{
					{
						Value: "v2ray.com",
						Type:  Domain_Plain,
					},
					{
						Value: "google.com",
						Type:  Domain_Domain,
					},
					{
						Value: "^facebook\\.com$",
						Type:  Domain_Regex,
					},
				},
			},
			test: []ruleTest{
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.com"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("www.v2ray.com.www"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("v2ray.co"), 80)}),
					output: false,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("www.google.com"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("facebook.com"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.DomainAddress("www.facebook.com"), 80)}),
					output: false,
				},
				{
					input:  withBackground(),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				Cidr: []*CIDR{
					{
						Ip:     []byte{8, 8, 8, 8},
						Prefix: 32,
					},
					{
						Ip:     []byte{8, 8, 8, 8},
						Prefix: 32,
					},
					{
						Ip:     net.ParseAddress("2001:0db8:85a3:0000:0000:8a2e:0370:7334").IP(),
						Prefix: 128,
					},
				},
			},
			test: []ruleTest{
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.8.8"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.4.4"), 80)}),
					output: false,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("2001:0db8:85a3:0000:0000:8a2e:0370:7334"), 80)}),
					output: true,
				},
				{
					input:  withBackground(),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				Geoip: []*GeoIP{
					{
						Cidr: []*CIDR{
							{
								Ip:     []byte{8, 8, 8, 8},
								Prefix: 32,
							},
							{
								Ip:     []byte{8, 8, 8, 8},
								Prefix: 32,
							},
							{
								Ip:     net.ParseAddress("2001:0db8:85a3:0000:0000:8a2e:0370:7334").IP(),
								Prefix: 128,
							},
						},
					},
				},
			},
			test: []ruleTest{
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.8.8"), 80)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.4.4"), 80)}),
					output: false,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("2001:0db8:85a3:0000:0000:8a2e:0370:7334"), 80)}),
					output: true,
				},
				{
					input:  withBackground(),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				SourceCidr: []*CIDR{
					{
						Ip:     []byte{192, 168, 0, 0},
						Prefix: 16,
					},
				},
			},
			test: []ruleTest{
				{
					input:  withInbound(&session.Inbound{Source: net.TCPDestination(net.ParseAddress("192.168.0.1"), 80)}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{Source: net.TCPDestination(net.ParseAddress("10.0.0.1"), 80)}),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				UserEmail: []string{
					"admin@v2ray.com",
				},
			},
			test: []ruleTest{
				{
					input:  withInbound(&session.Inbound{User: &protocol.MemoryUser{Email: "admin@v2ray.com"}}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{User: &protocol.MemoryUser{Email: "love@v2ray.com"}}),
					output: false,
				},
				{
					input:  withBackground(),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				Protocol: []string{"http"},
			},
			test: []ruleTest{
				{
					input:  withContent(&session.Content{Protocol: (&http.SniffHeader{}).Protocol()}),
					output: true,
				},
			},
		},
		{
			rule: &RoutingRule{
				InboundTag: []string{"test", "test1"},
			},
			test: []ruleTest{
				{
					input:  withInbound(&session.Inbound{Tag: "test"}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{Tag: "test2"}),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				PortList: &net.PortList{
					Range: []*net.PortRange{
						{From: 443, To: 443},
						{From: 1000, To: 1100},
					},
				},
			},
			test: []ruleTest{
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 443)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 1100)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 1005)}),
					output: true,
				},
				{
					input:  withOutbound(&session.Outbound{Target: net.TCPDestination(net.LocalHostIP, 53)}),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				SourcePortList: &net.PortList{
					Range: []*net.PortRange{
						{From: 123, To: 123},
						{From: 9993, To: 9999},
					},
				},
			},
			test: []ruleTest{
				{
					input:  withInbound(&session.Inbound{Source: net.UDPDestination(net.LocalHostIP, 123)}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{Source: net.UDPDestination(net.LocalHostIP, 9999)}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{Source: net.UDPDestination(net.LocalHostIP, 9994)}),
					output: true,
				},
				{
					input:  withInbound(&session.Inbound{Source: net.UDPDestination(net.LocalHostIP, 53)}),
					output: false,
				},
			},
		},
		{
			rule: &RoutingRule{
				Protocol:   []string{"http"},
				Attributes: "attrs[':path'].startswith('/test')",
			},
			test: []ruleTest{
				{
					input:  withContent(&session.Content{Protocol: "http/1.1", Attributes: map[string]string{":path": "/test/1"}}),
					output: true,
				},
			},
		},
	}

	for _, test := range cases {
		cond, err := test.rule.BuildCondition()
		common.Must(err)

		for _, subtest := range test.test {
			actual := cond.Apply(subtest.input)
			if actual != subtest.output {
				t.Error("test case failed: ", subtest.input, " expected ", subtest.output, " but got ", actual)
			}
		}
	}
}

```

这段代码定义了一个名为`loadGeoSite`的函数，接受一个国家字符串参数（`country`）。函数的作用是读取到一个名为`geosite.dat`的文件，并将其中的数据存储在一个`GeoSiteList`结构体中。如果文件读取成功，函数将返回一个包含`GeoSite`结构体的切片，或者一个错误。

函数的具体实现包括以下几个步骤：

1. 从文件系统中读取`GeoSite.dat`文件的内容，并将其存储在一个名为`geoiseBytes`的变量中。
2. 尝试使用`proto.Unmarshal`函数将`geotypeBytes`字节数组（包含GeoSite结构体数据）解码为`GeoSiteList`结构体。如果解码成功，将`geoiseList`存储在一个`GeoSiteList`变量中。
3. 遍历`geoiseList`中的每个`GeoSite`结构体，检查当前结构体的`CountryCode`字段是否与传入的国家字符串`country`相等。如果是，返回该结构体的`domain`字段（包含该GeoSite的域名），否则返回一个`nil`值（表示未找到该国家）。
4. 如果所有GeoSite结构体均未找到对应的国家字符串，函数返回一个`error`。


```go
func loadGeoSite(country string) ([]*Domain, error) {
	geositeBytes, err := filesystem.ReadAsset("geosite.dat")
	if err != nil {
		return nil, err
	}
	var geositeList GeoSiteList
	if err := proto.Unmarshal(geositeBytes, &geositeList); err != nil {
		return nil, err
	}

	for _, site := range geositeList.Entry {
		if site.CountryCode == country {
			return site.Domain, nil
		}
	}

	return nil, errors.New("country not found: " + country)
}

```

该代码定义了一个名为 `TestChinaSites` 的函数，它接受一个名为 `t` 的测试断言者对象作为参数。

该函数首先加载一个名为 "CN" 的地理站点，然后使用 `loadGeoSite` 函数获取一个或多个地理站点。函数使用 `common.Must` 函数确保加载成功。

然后，该函数创建一个 `DomainMatcher` 对象，该对象使用上面加载的地理站点数据进行测试。

接下来，定义一个名为 `TestCase` 的结构体，它包含一个测试域和测试输出。

最后，使用一个循环来遍历测试用例列表。对于每个测试用例，使用 `matcher.ApplyDomain` 函数应用 `DomainMatcher` 并获取结果。然后，将结果与测试用例的预期输出进行比较，并输出错误信息，如果结果与预期输出不同。

该函数旨在对测试用例列表中的每个测试用例进行测试，以确保所有测试用例的预期输出都符合要求。


```go
func TestChinaSites(t *testing.T) {
	domains, err := loadGeoSite("CN")
	common.Must(err)

	matcher, err := NewDomainMatcher(domains)
	common.Must(err)

	type TestCase struct {
		Domain string
		Output bool
	}
	testCases := []TestCase{
		{
			Domain: "163.com",
			Output: true,
		},
		{
			Domain: "163.com",
			Output: true,
		},
		{
			Domain: "164.com",
			Output: false,
		},
		{
			Domain: "164.com",
			Output: false,
		},
	}

	for i := 0; i < 1024; i++ {
		testCases = append(testCases, TestCase{Domain: strconv.Itoa(i) + ".not-exists.com", Output: false})
	}

	for _, testCase := range testCases {
		r := matcher.ApplyDomain(testCase.Domain)
		if r != testCase.Output {
			t.Error("expected output ", testCase.Output, " for domain ", testCase.Domain, " but got ", r)
		}
	}
}

```

这段代码是一个名为"BenchmarkMultiGeoIPMatcher"的函数，它用于测试一个名为"GeoIPMatcher"的函数的正确性。

函数的作用是加载三个国家(中国、日本和加拿大)的IP地址，并将它们存储在一个名为"geoips"的数组中。然后，它使用一个名为"NewMultiGeoIPMatcher"的函数创建一个"MultiGeoIPMatcher"实例，并将"geoips"数组和禁用"match"函数作为参数传递。

接下来，函数使用"withOutbound"函数创建一个输出入方向为目标IP地址(8.8.8.8)80端口的"session.Outbound"实例。然后，它使用"ResetTimer"函数重置计时器，并在一个循环中调用"matcher.Apply"函数。每次循环都会在计时器中增加一些时间，然后使用"ctx"上下文函数中的"_ = matcher.Apply(ctx)"语句来执行"matcher.Apply"函数。

"matcher.Apply"函数接受一个上下文对象"ctx"，在上下文对象中执行"matcher.Apply"函数可以确保所有操作都发生在同一个线程中。上下文对象是一个"withOutbound"函数创建的"session.Outbound"实例，包含了与传入的目标IP地址和端口匹配的函数。通过使用"withOutbound"函数创建的上下文对象，可以确保在函数中执行的操作都发生在同一个线程中，从而保证代码的可移植性和可读性。


```go
func BenchmarkMultiGeoIPMatcher(b *testing.B) {
	var geoips []*GeoIP

	{
		ips, err := loadGeoIP("CN")
		common.Must(err)
		geoips = append(geoips, &GeoIP{
			CountryCode: "CN",
			Cidr:        ips,
		})
	}

	{
		ips, err := loadGeoIP("JP")
		common.Must(err)
		geoips = append(geoips, &GeoIP{
			CountryCode: "JP",
			Cidr:        ips,
		})
	}

	{
		ips, err := loadGeoIP("CA")
		common.Must(err)
		geoips = append(geoips, &GeoIP{
			CountryCode: "CA",
			Cidr:        ips,
		})
	}

	{
		ips, err := loadGeoIP("US")
		common.Must(err)
		geoips = append(geoips, &GeoIP{
			CountryCode: "US",
			Cidr:        ips,
		})
	}

	matcher, err := NewMultiGeoIPMatcher(geoips, false)
	common.Must(err)

	ctx := withOutbound(&session.Outbound{Target: net.TCPDestination(net.ParseAddress("8.8.8.8"), 80)})

	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		_ = matcher.Apply(ctx)
	}
}

```

# `app/router/config.go`

这段代码定义了一个名为 "router" 的包，其中包含以下函数和类型：

1. `build()` 函数：这个函数在没有参数的情况下返回一个空字符串，表示路由器没有配置任何插件或服务。这个函数通常在插件或服务配置完畢后执行，以清理任何残留的配置字符串。

2. `!confonly`：这是一个布尔类型，表示是否允许在运行时重新加载路由器配置。如果这个标志被设置为 `true`，那么在配置更改后，路由器将不会重新加载任何配置文件，从而避免可能的配置不一致问题。

3. `CIDRList` 类型：这个类型表示一个数组，每个元素都是一个指向 `CIDR` 类型的指针。`CIDR` 是一种用于 IP 地址路由的表示方法，它由四个用圆括号括起来的数字组成，分别表示网络前缀、子网前缀、网络前缀和子网前缀。

4. `len()` 函数：这个函数用于计算 `CIDRList` 类型的元素的缓存长度，用于支持 `len()` 函数和 `切片()` 函数。

5. `sort.Interface` 接口：这个接口表明了一个函数需要按照某个特定的顺序对 `CIDRList` 进行排序，从而支持 `sort()` 函数和 `range()` 循环等操作。

6. `sortCIDR()` 函数：这个函数的实现与 `sort.Interface` 接口的 `sort()` 函数类似，但是它使用 `CIDRList` 而不是切片，根据 `len()` 函数计算缓存长度，然后对 `CIDRList` 进行排序。具体来说，它遍历 `CIDRList` 中的所有元素，每次比较两个元素的大小，如果它们的缓存长度不同，就交换它们的位置，并且继续比较下一对元素。排序后，`CIDRList` 将按照按照其 `Len()` 函数计算的缓存长度有序排列。


```go
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

```

这段代码定义了一个名为 `Less` 的函数，它接受两个整数参数 `i` 和 `j`，表示两个 CIDR 列表。函数的目的是比较两个 CIDR 列表是否按某种方式排序，具体方式如下：

1. 比较两个列表的长度是否相同，如果不相同，说明这两个列表没有按相同的方式排序，返回 `false`。
2. 比较两个列表中的 IPv4 地址是否按相同顺序排列。如果列表中的 IPv4 地址长度不同，那么自动将较短的地址添加到列表的头部，这样就可以保证两个列表在 IPv4 地址的顺序上是相同的。如果列表中的 IPv4 地址已经按相同顺序排列，那么检查两个列表是否按相同的方式对 IPv4 地址进行排序，如果已经按相同的方式排序，那么返回 `true`，否则返回 `false`。
3. 比较两个列表的前缀是否按相同顺序排列。如果列表中的前缀已经按相同顺序排列，那么检查两个列表是否按相同的方式对前缀进行排序，如果已经按相同的方式排序，那么返回 `true`，否则返回 `false`。

该函数的实现使用了 CIDR 列表的一些特点，比如将 IPv4 地址按照前缀排序，即将前缀相同的地址放在一起，使得函数可以处理 IPv4 地址的分段。


```go
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

```

该代码定义了一个名为 `Swap` 的函数接口，以及一个名为 `Rule` 的结构体。

`Swap` 函数接口定义了一个整数变量 `i` 和 `j`，以及一个整数变量 `l`。函数的作用是交换变量 `l` 的第 `i` 个元素和第 `j` 个元素，即将元素 `(*l)[i]` 和元素 `(*l)[j]` 交换位置。

`Rule` 结构体定义了一个包含 `Tag`、`Balancer` 和 `Condition` 的字段。其中，`Tag` 字段表示 tag,`Balancer` 字段是一个指向 `Balancer` 结构体的指针，`Condition` 字段是一个条件判断。

`GetTag` 函数是一个 `Rule` 结构体方法的别名，用于获取 `Rule` 结构体中 `Tag` 字段的值，并返回一个字符串类型的结果，或者是 `nil` 错误。

具体的，如果 `Rule` 结构体中 `Balancer` 字段尚不等于 `nil`，则可以调用 `Balancer` 的 `PickOutbound` 方法获取出 `Balancer` 对象，然后返回该对象的 `Tag` 字段。如果 `Balancer` 字段为 `nil`，则直接返回 `Tag` 字段的值，它是 `Error` 类型，可以将它转换为字符串类型来返回。


```go
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

```

This is a method that creates a network matcher object based on a given rule\n\nThe rule can be based on one or more of the following fields:

- rr.NetworkList: A list of networks in the required format
- rr.Geoip: A list of geoip records
- rr.Cidr: A list of CIDR blocks
- rr.SourceGeoip: A list of source geoip records
- rr.SourceCidr: A list of source CIDR blocks
- rr.Protocol: The protocol used for the network traffic
- rr.Attributes: An optional list of attributes to match on

The function returns a network matcher object if the rule is valid, or an error if the rule is invalid.

The following is the code for the function:

 
public func NewNetworkMatcher(networks ...Network) *NetworkMatcher {
	return &NetworkMatcher{
		networks: networks...,
	}
}

public func NewMultiGeoIPMatcher(geoip ...GeoIP) *MultiGeoIPMatcher {
	return &MultiGeoIPMatcher{
		geoip: geoip...,
	}
}

public func NewAttributeMatcher(attributes ...Attribute) *AttributeMatcher {
	return &AttributeMatcher{
		attributes: attributes...,
	}
}

public interface NetworkMatcher, MultiGeoIPMatcher, AttributeMatcher {
	Conditions: []Condition
}

type NetworkMatcher struct {
	networks  []Network
	Conditions  []Condition
}

func (n *NetworkMatcher) AddCondition(cond *Condition) bool {
	return n.Conditions = append(n.Conditions, cond)
	return cond.Validate() == nil
}

func (n *NetworkMatcher) AddConditions(conds ...Condition) bool {
	return n.AddCondition(conds...)
}

type MultiGeoIPMatcher struct {
	geoip  []GeoIP
	Conditions []Condition
}

func (m *MultiGeoIPMatcher) AddCondition(cond *Condition) bool {
	return m.Conditions = append(m.Conditions, cond)
	return cond.Validate() == nil
}

func (m *MultiGeoIPMatcher) AddConditions(conds ...Condition) bool {
	return m.AddCondition(conds...)
}

type AttributeMatcher struct {
	attributes ...Attribute
}

func (a *AttributeMatcher) AddAttribute(attr ...Attribute) bool {
	return a.attributes = append(a.attributes, attr...)
	return attr.Validate() == nil
}


Note that the implementation assumes that the `NewNetworkMatcher`, `NewMultiGeoIPMatcher`, and `NewAttributeMatcher` functions have been implemented as follows:
 
publicfunc NewNetworkMatcher(networks ...Network) *NetworkMatcher {
	return &NetworkMatcher{
		networks: networks...,
	}
}

publicfunc NewMultiGeoIPMatcher(geoip ...GeoIP) *MultiGeoIPMatcher {
	return &MultiGeoIPMatcher{
		geoip: geoip...,
	}
}

publicfunc NewAttributeMatcher(attributes ...Attribute) *AttributeMatcher {
	return &AttributeMatcher{
		attributes: attributes...,
	}
}



```go
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

	if rr.SourcePortList != nil {
		conds.Add(NewPortMatcher(rr.SourcePortList, true))
	}

	if len(rr.Networks) > 0 {
		conds.Add(NewNetworkMatcher(rr.Networks))
	} else if rr.NetworkList != nil {
		conds.Add(NewNetworkMatcher(rr.NetworkList.Network))
	}

	if len(rr.Geoip) > 0 {
		cond, err := NewMultiGeoIPMatcher(rr.Geoip, false)
		if err != nil {
			return nil, err
		}
		conds.Add(cond)
	} else if len(rr.Cidr) > 0 {
		cond, err := NewMultiGeoIPMatcher([]*GeoIP{{Cidr: rr.Cidr}}, false)
		if err != nil {
			return nil, err
		}
		conds.Add(cond)
	}

	if len(rr.SourceGeoip) > 0 {
		cond, err := NewMultiGeoIPMatcher(rr.SourceGeoip, true)
		if err != nil {
			return nil, err
		}
		conds.Add(cond)
	} else if len(rr.SourceCidr) > 0 {
		cond, err := NewMultiGeoIPMatcher([]*GeoIP{{Cidr: rr.SourceCidr}}, true)
		if err != nil {
			return nil, err
		}
		conds.Add(cond)
	}

	if len(rr.Protocol) > 0 {
		conds.Add(NewProtocolMatcher(rr.Protocol))
	}

	if len(rr.Attributes) > 0 {
		cond, err := NewAttributeMatcher(rr.Attributes)
		if err != nil {
			return nil, err
		}
		conds.Add(cond)
	}

	if conds.Len() == 0 {
		return nil, newError("this rule has no effective fields").AtWarning()
	}

	return conds, nil
}

```

这段代码定义了一个名为"func"的函数，它接受一个名为"br"的参数类型为"BalancingRule"的指针变量，并返回一个指向"Balancer"类型对象的引用和一个指向错误对象的"error"类型指针。

函数的作用是创建一个新的"Balancer"对象，根据传入的"BalancingRule"指针变量来设置选线策略和随机策略，并将创建该对象的上下文指针存储到"ohm"变量中。如果传入的参数"br"为空，则返回一个空的"Balancer"对象和一个指向错误对象的"error"类型指针。


```go
func (br *BalancingRule) Build(ohm outbound.Manager) (*Balancer, error) {
	return &Balancer{
		selectors: br.OutboundSelector,
		strategy:  &RandomStrategy{},
		ohm:       ohm,
	}, nil
}

```

# `app/router/config.pb.go`

该代码是一个 Go 语言项目中的路由器类实现。通过 protoc-gen-go 工具生成了 Go 语言需要的类型定义。下面是生成的 Go 语言代码的作用：

1. 定义了路由器协议（router.proto）的接口，包括静态字段（版本号、作者等）和静态方法（config、dump等）。
2. 实现了 protobuf 类型中对应的方法，用于与其它支持 protobuf 的语言进行交互。
3. 导入了自定义的 reflow 函数（reflect.sync）。
4. 导入了 net 包（v2ray.com/core/common/net）。
5. 在 config 方法中，将 Protobuf 类型中的 config 字段映射为路由器对应的字段。
6. 在 dump 方法中，将路由器实例的 config 字段中的键值对导出为 JSON 字节切片。
7. 在 sync 包中，添加了几个自定义的同步方法，如 sleep、cancel 等，用于协调多路复用。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: app/router/config.proto

package router

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	net "v2ray.com/core/common/net"
)

```

这段代码是一个 Rust 代码片段，它定义了一个名为 `Domain_Type` 的枚举类型，以及一个名为 `_` 的常量。

首先，它使用 `protoimpl.EnforceVersion` 函数来确保所使用的存档版本与定义的接口的版本相匹配。这个函数接受两个参数：要检查的最低版本和要检查的最高版本。这里的版本号是 `20 - protoimpl.MinVersion` 和 `protoimpl.MaxVersion - 20`。

接下来，它定义了一个名为 `_` 的常量，并在该常量的范围内定义了一个枚举类型 `Domain_Type`。这里 `Domain_Type` 枚举类型包含四个成员变量：`Domain_Plain`、`Domain_Regex`、`Domain_Domain` 和 `Domain_Full`。这些成员变量都有不同的值，分别为 `0`、`1`、`2` 和 `3`。

最后，它使用 `proto.ProtoPackageIsVersion4` 函数来检查当前使用的存档版本是否为 `4`。这个函数返回一个布尔值，表示当前使用的存档版本是否足够新。如果足够新，那么该代码片段就不会输出任何错误。


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

// Type of domain value.
type Domain_Type int32

const (
	// The value is used as is.
	Domain_Plain Domain_Type = 0
	// The value is used as a regular expression.
	Domain_Regex Domain_Type = 1
	// The value is a root domain.
	Domain_Domain Domain_Type = 2
	// The value is a domain.
	Domain_Full Domain_Type = 3
)

```

这段代码定义了一个枚举类型 `Domain_Type`，并创建了两个变量 `Domain_Type_name` 和 `Domain_Type_value`。

`Domain_Type_name` 变量是一个包含四个枚举值的常量，每个枚举值对应一个名为 `Domain_Type_name` 的字符串常量。具体来说，`0` 对应的是 "Plain",`1` 对应的是 "Regex",`2` 对应的是 "Domain",`3` 对应的是 "Full"。

`Domain_Type_value` 变量是一个包含四个枚举值的常量，每个枚举值对应一个整型常量。具体来说，`0` 对应的整型常量是 `0`,`1` 对应的整型常量是 `1`,`2` 对应的整型常量是 `2`,`3` 对应的整型常量是 `3`。

该代码的作用是定义了一个枚举类型 `Domain_Type`，用于表示字符串类型变量 `Domain_Type_name` 和整型类型变量 `Domain_Type_value` 的枚举值。通过这种方式，`Domain_Type_name` 和 `Domain_Type_value` 可以被用来根据枚举值来获取对应的枚举值常量。


```go
// Enum value maps for Domain_Type.
var (
	Domain_Type_name = map[int32]string{
		0: "Plain",
		1: "Regex",
		2: "Domain",
		3: "Full",
	}
	Domain_Type_value = map[string]int32{
		"Plain":  0,
		"Regex":  1,
		"Domain": 2,
		"Full":   3,
	}
)

```

这段代码定义了一个名为“Domain_Type”的枚举类型。它的作用是定义了一个名为“func”的函数，该函数接收一个名为“x”的整数参数，并返回一个“Domain_Type”类型的指针。

具体来说，代码中定义了三个函数：

1. “Enum()”函数，它创建了一个名为“p”的整数类型的指针，并将其赋值为传入的“x”参数。然后，它返回了整数类型的指针“p”。

2. “String()”函数，它接收一个名为“x”的“Domain_Type”类型的参数，并返回了一个字符串类型的结果。它是通过调用外部的“X”函数类型实现的，这个函数类型接收一个“Domain_Type”类型的参数，并返回一个字符串类型的结果。

3. “Descriptor()”函数，它返回了一个“Domain_Type”类型的枚举描述符。它是通过调用外部的“Descriptor”函数类型实现的，这个函数类型返回一个“protoreflect.EnumDescriptor”类型的结果，与“Domain_Type”类型的参数匹配。

4. “Type()”函数，它返回了一个“Domain_Type”类型的枚举类型。它是通过调用外部的“EnumType”函数类型实现的，这个函数类型返回一个“protoreflect.EnumType”类型的结果，与“Domain_Type”类型的参数匹配。


```go
func (x Domain_Type) Enum() *Domain_Type {
	p := new(Domain_Type)
	*p = x
	return p
}

func (x Domain_Type) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Domain_Type) Descriptor() protoreflect.EnumDescriptor {
	return file_app_router_config_proto_enumTypes[0].Descriptor()
}

func (Domain_Type) Type() protoreflect.EnumType {
	return &file_app_router_config_proto_enumTypes[0]
}

```

这段代码定义了两个函数，第一个函数是一个字符函数`func`，接收一个`Domain_Type`类型的参数`x`，返回一个`protoreflect.EnumNumber`类型的值，这个函数的作用是将`Domain_Type`类型的参数`x`转换为`protoreflect.EnumNumber`类型，然后返回`protoreflect.EnumNumber`类型的值。

第二个函数也是一个字符函数`func`，接收一个`Domain_Type`类型的参数`Domain_Type`，返回两个字节类型的值，一个表示`Domain_Type`的描述信息，另一个表示`Domain_Type`类型的约束条件列表。这个函数的作用是将`Domain_Type`类型的参数`Domain_Type`转换为`protoreflect.Enum`类型，然后将`protoreflect.Enum`类型的值转换为字节切片，最后将字节切片转换为`Domain_Type`类型的描述信息返回，这个函数的第二个参数表示`Domain_Type`的约束条件列表，其中约束条件是一个列表参数，每个约束条件都是一个复合类型，包含以下字段：

- `Config_DomainAsIs`：表示是否使用领域，值为`0`或`1`。
- `Config_UseIp`：表示是否使用IP，值为`0`或`1`。
- `Config_IpIfNonMatch`：表示是否使用IP进行匹配，值为`0`或`1`。
- `Config_IpOnDemand`：表示是否在需要时使用IP，值为`0`或`1`。

函数的作用是将`Domain_Type`类型的参数`Domain_Type`转换为`protoreflect.Enum`类型，然后返回`protoreflect.EnumNumber`类型的值，这个值用于表示`Domain_Type`的枚举类型。如果`Domain_Type`为`Config_DomainStrategy`时，函数返回`Config_DomainStrategy`的枚举类型。


```go
func (x Domain_Type) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

// Deprecated: Use Domain_Type.Descriptor instead.
func (Domain_Type) EnumDescriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{0, 0}
}

type Config_DomainStrategy int32

const (
	// Use domain as is.
	Config_AsIs Config_DomainStrategy = 0
	// Always resolve IP for domains.
	Config_UseIp Config_DomainStrategy = 1
	// Resolve to IP if the domain doesn't match any rules.
	Config_IpIfNonMatch Config_DomainStrategy = 2
	// Resolve to IP if any rule requires IP matching.
	Config_IpOnDemand Config_DomainStrategy = 3
)

```

这段代码定义了一个枚举类型 Config_DomainStrategy，并输出了该枚举类型的值映射。该枚举类型有四个成员，分别对应的值为 0、1、2 和 3，分别代表 AsIs、UseIp、IpIfNonMatch 和 IpOnDemand 四个策略。

同时，该代码还定义了一个包含四个键值对的 map 变量 Config_DomainStrategy_name，该 map 类型将枚举类型的成员值映射到了相应的键值对中。

最后，该代码没有输出具体的配置策略名称和对应的枚举类型值，而是定义了这两个变量，以便在程序运行时根据需要进行输出或修改。


```go
// Enum value maps for Config_DomainStrategy.
var (
	Config_DomainStrategy_name = map[int32]string{
		0: "AsIs",
		1: "UseIp",
		2: "IpIfNonMatch",
		3: "IpOnDemand",
	}
	Config_DomainStrategy_value = map[string]int32{
		"AsIs":         0,
		"UseIp":        1,
		"IpIfNonMatch": 2,
		"IpOnDemand":   3,
	}
)

```

这段代码定义了一个名为 `Config_DomainStrategy` 的枚举类型，并实现了两个函数：

1. `func (x Config_DomainStrategy) Enum() *Config_DomainStrategy {
	p := new(Config_DomainStrategy)
	*p = x
	return p
}`

该函数接收一个 `Config_DomainStrategy` 类型的参数 `x`，并将其赋值给一个新创建的 `Config_DomainStrategy` 类型的变量 `p`。然后，返回 `p`，这样就可以在函数内部使用 `x` 了。

1. `func (x Config_DomainStrategy) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}`

该函数返回一个字符串，使用了 `protoimpl.X.EnumStringOf` 函数将 `Config_DomainStrategy` 的 `Descriptor` 函数的返回值转换为该枚举类型的字符串表示。函数的第一个参数是 `x`，第二个参数是 `Descriptor` 函数的返回值，第三个参数是 `protoreflect.EnumNumber`。

1. `func (Config_DomainStrategy) Descriptor() protoreflect.EnumDescriptor {
	return file_app_router_config_proto_enumTypes[1].Descriptor()
}`

该函数返回一个枚举描述符，使用了 `file_app_router_config_proto_enumTypes[1].Descriptor()` 函数将枚举类型文件中的第二个元素的描述符返回。

1. `func (Config_DomainStrategy) Type() protoreflect.EnumType {
	return &file_app_router_config_proto_enumTypes[1]`

该函数返回一个枚举类型，与上面返回的相同，类型名为 `file_app_router_config_proto_enumTypes[1]`。


```go
func (x Config_DomainStrategy) Enum() *Config_DomainStrategy {
	p := new(Config_DomainStrategy)
	*p = x
	return p
}

func (x Config_DomainStrategy) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Config_DomainStrategy) Descriptor() protoreflect.EnumDescriptor {
	return file_app_router_config_proto_enumTypes[1].Descriptor()
}

func (Config_DomainStrategy) Type() protoreflect.EnumType {
	return &file_app_router_config_proto_enumTypes[1]
}

```

这段代码定义了一个名为`Domain`的结构体，用于表示路由决策域名。

函数`func (x Config_DomainStrategy) Number() protoreflect.EnumNumber`接收一个`x`参数，并返回一个`protoreflect.EnumNumber`类型的值，该类型表示一个枚举类型。

函数`func (x Config_DomainStrategy) EnumDescriptor() ([]byte, []int)`接收一个`x`参数，并返回一个元组，包含一个`file_app_router_config_proto_rawDescGZIP`类型的值和一个整数类型的值，该整数表示枚举类型的长度。函数的实现在函数体中，直接将`file_app_router_config_proto_rawDescGZIP`类型的值作为第一个参数，将其转换为整数类型并返回。

函数`type Domain struct { ... }`定义了一个名为`Domain`的结构体，该结构体包含以下字段：

- `state       `  `Domain_State`类型字段，用于表示路由决策的状态。
- `sizeCache`  `Domain_SizeCache`类型字段，用于表示缓存路由器实例的大小。
- `unknownFields` `Domain_UnknownFields`类型字段，用于表示无法从文档中读取的未知字段。
- `Type          ` Domain_Type`类型字段，用于表示域名类型。
- `Value        ` Domain_Value`类型字段，用于表示域名的值。
- `Attribute    ` Domain_Attribute`类型字段，用于表示用于路由过滤的属性。

函数`func (x Config_DomainStrategy) EnumDescriptor() ([]byte, []int)`接收一个`x`参数，并返回一个元组，包含一个`file_app_router_config_proto_rawDescGZIP`类型的值和一个整数类型的值，该整数表示枚举类型的长度。函数的实现在函数体中，直接将`file_app_router_config_proto_rawDescGZIP`类型的值作为第一个参数，将其转换为整数类型并返回。


```go
func (x Config_DomainStrategy) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

// Deprecated: Use Config_DomainStrategy.Descriptor instead.
func (Config_DomainStrategy) EnumDescriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{8, 0}
}

// Domain for routing decision.
type Domain struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// Domain matching type.
	Type Domain_Type `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.router.Domain_Type" json:"type,omitempty"`
	// Domain value.
	Value string `protobuf:"bytes,2,opt,name=value,proto3" json:"value,omitempty"`
	// Attributes of this domain. May be used for filtering.
	Attribute []*Domain_Attribute `protobuf:"bytes,3,rep,name=attribute,proto3" json:"attribute,omitempty"`
}

```

这段代码定义了两个函数：

1. `func (x *Domain) Reset()` 函数接收一个`Domain`类型的参数 `x`，并将其赋值为 `Domain{}`。然后，如果 `protoimpl.UnsafeEnabled` 是 `true`，则代码会执行以下操作：

- 获取 `file_app_router_config_proto_msgTypes` 类型的大纲类型。
- 创建一个指向 `Domain` 类型实例的指针变量 `mi`。
- 创建一个包含 `x` 类型实例的内存中的 `MessageInfo` 类型的指针变量 `ms`。
- 将大纲类型中的第一个类型 (`file_app_router_config_proto_msgTypes`) 的指针变量设置为 `ms`，并将类型名称设置为 `file_app_router_config_proto_msgTypes`。
- 调用 `ms.StoreMessageInfo` 函数将 `mi` 存储为 `ms` 类型中包含的信息的 `MessageInfo` 类型。

2. `func (x *Domain) String()` 函数返回一个 `Domain` 类型实例的 `String` 类型的值。它实现了 `protoimpl.X.MessageStringOf` 函数，该函数将 `Domain` 类型实例转换为字符串，并返回一个字符串。

3. `func (x *Domain) ProtoMessage()` 函数返回一个 `Domain` 类型实例的 `Message` 类型的接口的 `Methods` 字段。这个函数返回的 `Methods` 字段应该返回一个包含所有 `Domain` 类型实例的方法的列表，以便将 `Domain` 类型实例传递给 `Service` 函数。


```go
func (x *Domain) Reset() {
	*x = Domain{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Domain) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Domain) ProtoMessage() {}

```

这段代码定义了两个函数，函数一是将一个指向Domain类型对象的变量x作为参数，返回一个名为"file_app_router_config_proto_msgTypes"的类型对象的指针；函数二是返回一个名为"file_app_router_config_proto_msgTypes"的类型对象的值。

函数一首先检查x是否为空，如果不是，则执行以下操作：

1. 从"file_app_router_config_proto_msgTypes"的类型对象中查找与x指向的类型对象相对应的类型；
2. 如果找到了相应的类型对象，那么执行以下操作：
  1. 从类型对象的类型字段中获取消息类型信息；
  2. 如果类型字段中包含的消息类型信息为nil，那么执行以下操作：
     1. 创建一个新的类型对象，设置其类型字段为刚刚获取到的消息类型信息；
     2. 设置类型对象的LoadMessageInfo()函数的值为新创建的类型对象的负载消息信息；
     3. 返回新创建的类型对象的负载消息信息。

函数二返回Domain类型对象的一个描述，其中包括了域的接口类型、字段类型和序列化/反序列化方法。函数二首先返回一个名为"file_app_router_config_proto_rawDescGZIP"的类型对象的值，表示该域对象的支持原始序列化和反序列化的二进制数据类型。其次，函数二返回一个包含两个元素的切片，分别为"file_app_router_config_proto_msgTypes"和"file_app_router_config_proto_rawDescGZIP"。


```go
func (x *Domain) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Domain.ProtoReflect.Descriptor instead.
func (*Domain) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{0}
}

```



该代码定义了三个函数，分别接收一个名为 `x` 的 `Domain` 类型的对象，并返回该对象的相关信息。

1. `GetType()` 函数，返回 `x` 对象所属的 `Domain_Plain` 类型，如果 `x` 对象为 `nil`，则返回 `Domain_Plain`。

2. `GetValue()` 函数，返回 `x` 对象所对应的属性的值，如果 `x` 对象为 `nil`，则返回一个空字符串。

3. `GetAttribute()` 函数，返回 `x` 对象所对应的属性的列表，如果 `x` 对象为 `nil`，则返回一个空列表。

函数的作用是帮助用户获取 `x` 对象属性的相关信息，其中 `x` 对象必须存在。`GetType()`、`GetValue()` 和 `GetAttribute()` 函数返回的信息不依赖于 `x` 对象的具体类型和值，而只是根据 `x` 对象是否存在来返回相应的类型。


```go
func (x *Domain) GetType() Domain_Type {
	if x != nil {
		return x.Type
	}
	return Domain_Plain
}

func (x *Domain) GetValue() string {
	if x != nil {
		return x.Value
	}
	return ""
}

func (x *Domain) GetAttribute() []*Domain_Attribute {
	if x != nil {
		return x.Attribute
	}
	return nil
}

```

该代码定义了一个名为 `CIDR` 的结构体，用于表示IP路由决策中的IP地址。该结构体包含以下字段：

- `Ip`：IP地址，为4或16字节。
- `Prefix`：网络前缀，用于指定网络掩码。

此外，该结构体还包含一个名为 `unknownFields` 的字段，该字段保存了IP地址中未知的字段，但不会将其输出。

该代码实现了一个 `CIDR` 类型的变量 `x`，其中包含了一个 `Reset` 方法和一个 `Set` 方法。 `Reset` 方法重置了 `x` 的状态，并设置了一些元数据，使其与一个新的 `CIDR` 结构体初始化时一致。`Set` 方法允许用户更改 `x` 的IP地址和前缀。

最后，该代码还包含一个名为 `file_app_router_config_proto_msgTypes` 的变量，它包含了一个IP路由决策原型（Google Cloud Protocol Buffers）的 ID。


```go
// IP for routing decision, in CIDR form.
type CIDR struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// IP address, should be either 4 or 16 bytes.
	Ip []byte `protobuf:"bytes,1,opt,name=ip,proto3" json:"ip,omitempty"`
	// Number of leading ones in the network mask.
	Prefix uint32 `protobuf:"varint,2,opt,name=prefix,proto3" json:"prefix,omitempty"`
}

func (x *CIDR) Reset() {
	*x = CIDR{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了两个函数，以及一个指针变量x。

第一个函数名为func (x *CIDR) String() string，它接收一个指向CIDR类型的参数x，并将其转换为字符串类型。具体实现是通过调用protoimpl.X.MessageStringOf(x)来实现的。

第二个函数名为func (*CIDR) ProtoMessage()，它接收一个指向CIDR类型的参数x，并将其转换为protoimpl.Message类型。具体实现是通过直接返回一个空的空的结构体类型来实现的，这个结构体类型没有任何内部的字段，也没有任何方法。

第三个函数名为func (x *CIDR) ProtoReflect() protoreflect.Message，它接收一个指向CIDR类型的参数x，并将其转换为protoreflect.Message类型。具体实现是通过接收x的指针类型并将其转换为mi，然后通过mi的MessageOf(x)方法将x的值转换为字节数组类型，最后将这个字节数组类型转换为protoreflect.Message类型。


```go
func (x *CIDR) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*CIDR) ProtoMessage() {}

func (x *CIDR) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[1]
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

此代码是一个 Go 语言中的函数，它定义了一个名为 "CIDR" 的类型，并提供了两个方法：

1. Descriptor()，该方法返回一个字节切片和两个整数，表示 CIDR 类型的描述。
2. GetIp() 和 GetPrefix()，这两个方法分别返回一个字节切片和一个整数，表示 CIDR 类型的实例。

对于第一组方法，"Descriptor()"，它将 "file_app_router_config_proto_rawDescGZIP()" 和一个空字节切片作为参数，并返回它们。其中，"file_app_router_config_proto_rawDescGZIP()" 是一个来自 "file_app_router_config_proto.proto" 的函数，该函数将返回一个字符串，表示一个 CIDR 类型描述。空字节切片 "[]byte{}`"，在方法中未被使用，但作为参数传递给了 "file_app_router_config_proto_rawDescGZIP()"。

对于第二组方法，"GetIp()" 和 "GetPrefix()"，它们分别返回一个 CIDR 类型的实例。

"GetIp()" 方法首先检查 x 是否为 nil，如果是，则 x.Ip 是一个字节切片，表示 CIDR 类型的实例。如果 x 不是 nil，则 x.Ip 是 CIDR 类型的实例，该实例的 Ip 地址字段包含一个字符串，表示 CIDR 类型的实例。

"GetPrefix()" 方法首先检查 x 是否为 nil，如果是，则 x.Prefix 是一个整数，表示 CIDR 类型的实例。如果 x 不是 nil，则 x.Prefix 是 CIDR 类型的实例，该实例的 Priority 字段包含一个整数，表示 CIDR 类型的实例。


```go
// Deprecated: Use CIDR.ProtoReflect.Descriptor instead.
func (*CIDR) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{1}
}

func (x *CIDR) GetIp() []byte {
	if x != nil {
		return x.Ip
	}
	return nil
}

func (x *CIDR) GetPrefix() uint32 {
	if x != nil {
		return x.Prefix
	}
	return 0
}

```

这段代码定义了一个名为 "GeoIP" 的 struct 类型，它包含三个字段：

1. "state"，这是一个名为 "GeoIP" 的 struct 类型字段，用于保存 GeoIP 的状态，这里使用了 protobuf 中的 message state 字段类型。
2. "sizeCache"，这是一个名为 "GeoIP" 的 struct 类型字段，用于保存 GeoIP 的大小缓存，这里使用了 protobuf 中的 message size 字段类型。
3. "unknownFields"，这是一个名为 "GeoIP" 的 struct 类型字段，用于保存 GeoIP 的未知字段，这里使用了 protobuf 中的 message unknown fields 字段类型。

此外，还有一段名为 "Reset" 的函数，它接受一个 GeoIP 类型的参数 x，并执行了以下操作：

1. 将 x 赋值为一个空的 GeoIP 类型，这样就实现了 reset。
2. 如果 `protoimpl.UnsafeEnabled` 是 true，那么执行以下操作：
  1. 通过 `mi` 获取到 "GeoIP" 类型在 "file_app_router_config_proto" 中的实例，然后获取到 "messageState" 字段。
  2. 通过 `x.messageState.StoreMessageInfo` 方法，设置 "GeoIP" 实例的 "messageState" 字段的值。


```go
type GeoIP struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	CountryCode string  `protobuf:"bytes,1,opt,name=country_code,json=countryCode,proto3" json:"country_code,omitempty"`
	Cidr        []*CIDR `protobuf:"bytes,2,rep,name=cidr,proto3" json:"cidr,omitempty"`
}

func (x *GeoIP) Reset() {
	*x = GeoIP{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为 func 的函数，接收一个名为 x 的整型指针变量，并返回一个字符串类型的函数调用，该函数调用使用了 Protocol Buffers 的类型。

具体来说，这段代码：

1. 定义了一个名为 func 的函数，它的参数是一个整型指针变量 x，类型为名为 GeoIP 的结构体类型，使用了名为 protoimpl 的内部接口类型。

2. 在函数体内，调用了名为 protoimpl.X 的类型实现的 MessageStringOf 函数，该函数接收一个名为 x 的整型指针变量，并返回一个字符串类型的结果。

3. 在函数体内，调用了名为 protoimpl.UnsafeEnabled 类型的变量，如果该变量为 true，则执行以下操作：

	1. 创建一个名为 mi 的整型指针变量，它对应于名为 file_app_router_config_proto_msgTypes 的结构体类型中的第二个成员。

	2. 如果 x 不是空指针，则执行以下操作：

		1. 创建一个名为 ms 的整型指针变量，它对应于名为 file_app_router_config_proto_msgTypes 的结构体类型中的第二个成员。

		2. 如果 ms.LoadMessageInfo() 的返回结果为 nil，则执行以下操作：

			1. 设置 mi.MessageInfo 字段的值为 Reserved field: 0。

			2. 设置 mi.MessageOf 函数的返回结果为 x。

		3. 返回 mi。

4. 在函数体外，定义了一个名为 *GeoIP 的类型接口，该接口没有实现任何方法。

5. 在函数体外，定义了一个名为 *GeoIP 的类型别名，该别名继承了名为 protoreflect.Message 的类型接口，该接口定义了 Message 的结构体类型。

6. 在函数体内，定义了一个名为 ProtoMessage 的函数，它的参数是一个空指针，返回一个名为空的类型接口。

7. 在函数体内，定义了一个名为 ProtoReflect 的函数，它的参数是一个空指针，返回一个名为空的类型接口。

8. 在函数体内，通过 mi.MessageOf 和 x.MessageStringOf 函数，实现了将 *GeoIP 转换为字符串的功能。


```go
func (x *GeoIP) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GeoIP) ProtoMessage() {}

func (x *GeoIP) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[2]
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

该代码是一个Go语言的函数，定义了两个方法：

1. Descriptor()函数，返回一个由两个字节组成的地理国家码和两个整数的切片。这个函数是 deprecated的，建议使用GeoIP.protoReflect.Descriptor函数代替。
2. GetCountryCode()函数，返回一个字符串表示国家代码。如果x不等于 nil，则返回x.CountryCode，否则返回一个空字符串。
3. GetCidr()函数，返回一个包含国际顶级域名（如.com，.org等）的切片，如果x不等于 nil，则返回x.Cidr，否则返回 nil。


```go
// Deprecated: Use GeoIP.ProtoReflect.Descriptor instead.
func (*GeoIP) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{2}
}

func (x *GeoIP) GetCountryCode() string {
	if x != nil {
		return x.CountryCode
	}
	return ""
}

func (x *GeoIP) GetCidr() []*CIDR {
	if x != nil {
		return x.Cidr
	}
	return nil
}

```

这段代码定义了一个名为"GeoIPList"的结构体类型，它包含以下字段：

1. "state"：一个属于"protoimpl.MessageState"的内部字段，用于保存该结构体所代表的协议实例的状态信息。
2. "sizeCache"：一个属于"protoimpl.SizeCache"的内部字段，用于保存该结构体所代表的协议实例的大小信息，该字段将在编译时进行计算，并在运行时进行更新。
3. "unknownFields"：一个属于"protoimpl.UnknownFields"的内部字段，用于保存该结构体所代表的协议实例中未定义的字段的信息，该字段将在编译时进行计算，并在运行时进行更新。
4. "Entry"：一个包含"GeoIP"类型的字段，用于保存该结构体所代表的协议实例的Entry信息。

另外，该结构体还包含两个方法：

1. "Reset"：该方法重置了该结构体所代表的协议实例，清除了所有未定义的字段，并且清空了该结构体所代表的目标"entry"字段。
2. "Load"：该方法将一个"GeoIP"类型的实例加载到该结构体所代表的协议实例中，并且设置该实例的Entry信息为指定的"GeoIP"实例。


```go
type GeoIPList struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Entry []*GeoIP `protobuf:"bytes,1,rep,name=entry,proto3" json:"entry,omitempty"`
}

func (x *GeoIPList) Reset() {
	*x = GeoIPList{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了两个函数，一个接收一个指向名为GeoIPList的值的指针，返回一个字符串表示这个值，另一个返回一个指向名为GeoIPList的值的指针，返回一个空接口类型。

第一个函数，当x不等于零时，创建一个名为ms的MessageStateOf(x)的实例，然后设置LoadMessageInfo()函数的值为mi，最后返回ms。如果x为零，则返回mi。

第二个函数，返回一个指向名为GeoIPList的值的指针，该值可以被作为最后陈述的一部分传递给构造函数，同时也可以被赋值给函数参数。

第三个函数，返回一个指向名为GeoIPList的值的指针，该值可以被作为最后陈述的一部分传递给构造函数，同时也可以被赋值给函数参数。这个函数的作用类似于第二个函数，但它使用了一个名为file_app_router_config_proto_msgTypes的常量，这个常量包含了一个可以用来获取MessageOf和MessageStringOf函数指针的类型。


```go
func (x *GeoIPList) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GeoIPList) ProtoMessage() {}

func (x *GeoIPList) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[3]
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

该代码定义了一个名为 `GeoIPList` 的接口，该接口有一个名为 `Descriptor` 的方法，用于返回地理信息（GeoIP）列表的描述符。

在 `Descriptor` 方法中，使用了来自 `file_app_router_config_proto_rawDescGZIP` 文件中定义的 `Descriptor` 结构体，该结构体在注释中被标记为“Deprecated”，因此建议使用 `GeoIPList.proto.Descriptor` 来代替。

该接口还有一个名为 `GetEntry` 的方法，用于返回包含与传入 `*GeoIPList` 相关的 `GeoIP` 类型的切片。

该代码中的 `*GeoIPList` 类型是一个自定义类型，它包含了一个 `GeoIP` 类型的结构体。该 `GeoIP` 类型包含一个 `state` 字段，它表示该 `GeoIP` 的状态（如 `Connected` 或 `Disconnected`）。还包括一个 `sizeCache` 字段，它保存了该 `GeoIP` 类型的 `count` 字段的引用计数。最后，该 `GeoIP` 类型包含一个或多个字段，这些字段包含了地理信息（如 `countryCode` 和 `domain`）。

该代码中的 `GeoSite` 类型是一个 `protoimpl.MessageState`，该类型表示一个地理信息（GeoIP）的 `Message` 子类。该类型包含一个字段 `countryCode` 和一个字段 `domain`，这些字段都是 `string` 类型的。

该代码中的 `file_app_router_config_proto_rawDescGZIP` 函数返回了一个 `file_app_router_config_proto_rawDescGZIP` 函数的返回值，该函数实际是从 `file_app_router_config_proto.proto` 文件中定义的。


```go
// Deprecated: Use GeoIPList.ProtoReflect.Descriptor instead.
func (*GeoIPList) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{3}
}

func (x *GeoIPList) GetEntry() []*GeoIP {
	if x != nil {
		return x.Entry
	}
	return nil
}

type GeoSite struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	CountryCode string    `protobuf:"bytes,1,opt,name=country_code,json=countryCode,proto3" json:"country_code,omitempty"`
	Domain      []*Domain `protobuf:"bytes,2,rep,name=domain,proto3" json:"domain,omitempty"`
}

```

这段代码定义了两个函数，一个是`func (x *GeoSite) Reset()`，另一个是`func (x *GeoSite) String()`。这两个函数都在创建一个名为`GeoSite`的`GeoSite`类型的指针变量`x`时执行。

1. `func (x *GeoSite) Reset()`函数会将`x`变量设置为一个新的`GeoSite`类型为空的`GeoSite`对象，即`x = GeoSite{}`。然后，如果定义在函数外部的是`protoimpl.UnsafeEnabled`，则会启用`mi`变量，这个变量是一个`file_app_router_config_proto_msgTypes`类型指针，代表一个四元组，它包含一个指向`GeoSite`类型实例的指针。最后，如果`mi`被启用，则会将`x`的`MessageInfo`字段设置为`mi`的`StoreMessageInfo`方法返回的`box`字段的值。

2. `func (x *GeoSite) String()`函数会返回`x`的`MessageStringOf`方法的返回值，这个函数将`x`的`MessageStringOf`方法的返回值作为字符串返回。

3. `func (x *GeoSite) ProtoMessage()`函数返回一个空的`GeoSite`类型，这个类型表示了原始 `GeoSite` 类型的`message`字段的定义，但没有实现任何方法。这个函数通常在定义类型时被调用，而不是在创建指针变量时。


```go
func (x *GeoSite) Reset() {
	*x = GeoSite{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[4]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GeoSite) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GeoSite) ProtoMessage() {}

```

这段代码定义了两个函数，一个是`func (x *GeoSite) ProtoReflect() protoreflect.Message`，另一个是`func (x *GeoSite) Descriptor() ([]byte, []int)`。

`func (x *GeoSite) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[4]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}`

这个函数接收一个`*GeoSite`类型的参数`x`，然后返回一个`protoreflect.Message`类型的结果。函数内部首先检查`protoimpl.UnsafeEnabled`是否为`true`，如果是，则代表使用的是`file_app_router_config_proto_msgTypes`类型，这个类型定义了`protoreflect.Message`的类型定义。接着，函数创建一个`ms`对象，使用`protoimpl.X.MessageStateOf`方法将`x`转换为一个`file_app_router_config_proto_msgTypes`类型的指针，然后使用`ms.LoadMessageInfo`方法判断`x`是否携带了消息信息，如果不携带，则创建一个新的`ms`对象并将消息信息存储为`mi`。最后，函数使用`ms.MessageOf`方法将`x`转换为一个`file_app_router_config_proto_msgTypes`类型的对象。

`func (x *GeoSite) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{4}`

这个函数同样接收一个`*GeoSite`类型的参数`x`，然后返回一个包含`file_app_router_config_proto_rawDescGZIP`类型数据和两个整数的`descriptor`。函数创建一个空的字节数组`byte`并将其赋值为`file_app_router_config_proto_rawDescGZIP`类型，然后使用`descriptor`函数的第一个参数返回字节数组，使用第二个参数返回消息类型数量。


```go
func (x *GeoSite) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[4]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GeoSite.ProtoReflect.Descriptor instead.
func (*GeoSite) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{4}
}

```

此代码定义了两个函数，以及一个名为 GeoSiteList 的结构体类型。

第一个函数 `func (x *GeoSite) GetCountryCode() string` 接收一个名为 `x` 的整数指针作为参数，并尝试获取该 `x` 指向的对象中的 `CountryCode` 字段。如果 `x` 指向的对象不为空，则返回该对象的 `CountryCode` 字段，否则返回一个空字符串。

第二个函数 `func (x *GeoSite) GetDomain() []*Domain` 接收一个名为 `x` 的整数指针作为参数，并尝试获取该 `x` 指向的对象中的 `Domain` 字段。如果 `x` 指向的对象不为空，则返回该对象的 `Domain` 字段，否则返回一个空数组。

结构体类型 `GeoSiteList` 定义了一个包含 `GeoSite` 类型的实例变量 `entry`，该变量具有 `rep` 参数，表示有一个 `GeoSite` 类型的字段，该字段名称为 `entry`，类型为 `protoimpl.Message`。该结构体还定义了一个名为 `unknownFields` 的字段，用于存储 `GeoSite` 对象中未填充的字段。

此外，该结构体还定义了一个名为 `entry` 的字段，该字段具有 `bytes` 参数类型，表示该字段的数据以字节数组形式传递。


```go
func (x *GeoSite) GetCountryCode() string {
	if x != nil {
		return x.CountryCode
	}
	return ""
}

func (x *GeoSite) GetDomain() []*Domain {
	if x != nil {
		return x.Domain
	}
	return nil
}

type GeoSiteList struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Entry []*GeoSite `protobuf:"bytes,1,rep,name=entry,proto3" json:"entry,omitempty"`
}

```

这段代码定义了两个函数，以及一个名为`GeoSiteList`的类型。

第一个函数`func (x *GeoSiteList) Reset()`的作用是重置`x`的值，将`x`指向一个空的`GeoSiteList`。

第二个函数`func (x *GeoSiteList) String()`的作用是返回`x`的`GeoSiteList`类型的字符串表示。这个函数使用了`protoimpl.X`的`String`函数，根据给定的`GeoSiteList`类型，返回了`x`的`GeoSiteList`类型。

第三个函数`func (x *GeoSiteList) ProtoMessage()`的作用是定义`GeoSiteList`类型，以便在定义变量或函数时可以将其作为`string`返回。这个函数使用了`protoimpl.X`的`Message`函数，根据给定的`GeoSiteList`类型，返回了一个`Message`类型，其中包含了`GeoSiteList`的接口定义。


```go
func (x *GeoSiteList) Reset() {
	*x = GeoSiteList{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[5]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *GeoSiteList) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*GeoSiteList) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个名为x的GeoSiteList类型的参数，并返回其对应的消息类型和GeoSiteList的描述信息。

第一个函数，名为func (x *GeoSiteList) ProtoReflect() protoreflect.Message，接收一个GeoSiteList类型的参数x，然后尝试使用一个*file_app_router_config_proto_msgTypes[5]类型的变量mi来存储该消息类型。如果使用的是deprecated的接口，则允许在函数内部设置mi。然后，尝试使用x的指向来存储消息信息，如果消息信息为nil，则将其设置为mi。最后，如果使用的是unsafe=true，则允许函数继续执行，否则会抛出异常并返回0。

第二个函数，名为(*GeoSiteList) Descriptor，接收一个GeoSiteList类型的参数，然后返回其描述信息。函数返回两个值，第一个值是一个二进制字节数组，存储了该GeoSiteList的描述信息，第二个值是一个整数数组，包含两个元素，第一个元素是该GeoSiteList的描述信息的长度，第二个元素是该描述信息的类型。如果GeoSiteList的类型使用的是deprecated的接口，则该函数将直接返回一个包含两个整数的切片。


```go
func (x *GeoSiteList) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[5]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use GeoSiteList.ProtoReflect.Descriptor instead.
func (*GeoSiteList) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{5}
}

```

This is a Protobuf message that defines the structure of a `Deprecated` message.

The message has several fields:

- `Deprecated` field
- `Protobuf` field for defining the message's schema
- `varint` field for defining the `NetworkList` field
- `bytes` field for defining the `Networks` field
- `varint` field for defining the `SourceCidr` field
- `bytes` field for defining the `SourceGeoip` field
- `bytes` field for defining the `SourcePortList` field
- `string` field for defining the `UserEmail` field
- `bytes` field for defining the `InboundTag` field
- `string` field for defining the `Protocol` field
- `bytes` field for defining the `Attributes` field

The `Deprecated` field is used to indicate that certain fields are deprecated and should not be used.


```go
func (x *GeoSiteList) GetEntry() []*GeoSite {
	if x != nil {
		return x.Entry
	}
	return nil
}

type RoutingRule struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// Types that are assignable to TargetTag:
	//	*RoutingRule_Tag
	//	*RoutingRule_BalancingTag
	TargetTag isRoutingRule_TargetTag `protobuf_oneof:"target_tag"`
	// List of domains for target domain matching.
	Domain []*Domain `protobuf:"bytes,2,rep,name=domain,proto3" json:"domain,omitempty"`
	// List of CIDRs for target IP address matching.
	// Deprecated. Use geoip below.
	//
	// Deprecated: Do not use.
	Cidr []*CIDR `protobuf:"bytes,3,rep,name=cidr,proto3" json:"cidr,omitempty"`
	// List of GeoIPs for target IP address matching. If this entry exists, the
	// cidr above will have no effect. GeoIP fields with the same country code are
	// supposed to contain exactly same content. They will be merged during
	// runtime. For customized GeoIPs, please leave country code empty.
	Geoip []*GeoIP `protobuf:"bytes,10,rep,name=geoip,proto3" json:"geoip,omitempty"`
	// A range of port [from, to]. If the destination port is in this range, this
	// rule takes effect. Deprecated. Use port_list.
	//
	// Deprecated: Do not use.
	PortRange *net.PortRange `protobuf:"bytes,4,opt,name=port_range,json=portRange,proto3" json:"port_range,omitempty"`
	// List of ports.
	PortList *net.PortList `protobuf:"bytes,14,opt,name=port_list,json=portList,proto3" json:"port_list,omitempty"`
	// List of networks. Deprecated. Use networks.
	//
	// Deprecated: Do not use.
	NetworkList *net.NetworkList `protobuf:"bytes,5,opt,name=network_list,json=networkList,proto3" json:"network_list,omitempty"`
	// List of networks for matching.
	Networks []net.Network `protobuf:"varint,13,rep,packed,name=networks,proto3,enum=v2ray.core.common.net.Network" json:"networks,omitempty"`
	// List of CIDRs for source IP address matching.
	//
	// Deprecated: Do not use.
	SourceCidr []*CIDR `protobuf:"bytes,6,rep,name=source_cidr,json=sourceCidr,proto3" json:"source_cidr,omitempty"`
	// List of GeoIPs for source IP address matching. If this entry exists, the
	// source_cidr above will have no effect.
	SourceGeoip []*GeoIP `protobuf:"bytes,11,rep,name=source_geoip,json=sourceGeoip,proto3" json:"source_geoip,omitempty"`
	// List of ports for source port matching.
	SourcePortList *net.PortList `protobuf:"bytes,16,opt,name=source_port_list,json=sourcePortList,proto3" json:"source_port_list,omitempty"`
	UserEmail      []string      `protobuf:"bytes,7,rep,name=user_email,json=userEmail,proto3" json:"user_email,omitempty"`
	InboundTag     []string      `protobuf:"bytes,8,rep,name=inbound_tag,json=inboundTag,proto3" json:"inbound_tag,omitempty"`
	Protocol       []string      `protobuf:"bytes,9,rep,name=protocol,proto3" json:"protocol,omitempty"`
	Attributes     string        `protobuf:"bytes,15,opt,name=attributes,proto3" json:"attributes,omitempty"`
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`RoutingRule`的整型指针参数，并实现了两个函数：`Reset`和`String`。

`Reset`函数的作用是重置接收到的`RoutingRule`对象，将里面的值都设置为`RoutingRule{}`，然后检查`protoimpl.UnsafeEnabled`是否为`true`，如果是，则执行以下操作：

1. 在`file_app_router_config_proto_msgTypes`数组中查找与`RoutingRule`同名的结构体类型，创建一个同名的`RoutingRule`实例并将其存储为`x`的值。
2. 如果`protoimpl.UnsafeEnabled`为`true`，则执行以下操作：
a. 在`file_app_router_config_proto_msgTypes`数组中查找与`RoutingRule`同名的结构体类型，创建一个同名的`RoutingRule`实例并将其存储为`x`的值。
b. 调用`RoutingRule`的`String`函数，返回其创建的`RoutingRule`对象的`String`字段。

`String`函数的作用是打印出接收到的`RoutingRule`对象的`String`字段，以便在调试信息中显示。

`ProtoMessage`函数的作用是定义一个接收`RoutingRule`对象的`Message`函数，用于将`RoutingRule`对象转换为字节切片，使得`Message`函数可以被`fmt.Println`等函数打印出来。


```go
func (x *RoutingRule) Reset() {
	*x = RoutingRule{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[6]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *RoutingRule) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*RoutingRule) ProtoMessage() {}

```

这段代码定义了一个名为 "func" 的函数，它接收一个名为 "x" 的整数类型的参数，并返回一个名为 "protoreflect.Message" 的接口类型。

具体来说，这段代码实现了一个 "protoreflect.Message" 类型的函数 "descriptor"，它会根据传入的 "x"，在 "file_app_router_config_proto_rawDescGZIP" 包中查找与 "RoutingRule.Descriptor" 相对应的描述，然后将该描述作为 "ms" 变量返回，其中 "ms" 是一个指向 "RoutingRule.Descriptor" 类型的指针。如果 "x" 不为空，则先尝试使用 "RoutingRule.Descriptor.LoadMessageInfo" 函数获取与 "x" 相关的 "ms" 对象，如果该对象为空，则将 "mi"（一个保存了 "file_app_router_config_proto_rawDescGZIP" 包中与 "RoutingRule.Descriptor" 相对应的描述的指针）存储为 "ms"，并返回 "ms"。如果 "x" 为空，则直接返回 "mi"。


```go
func (x *RoutingRule) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[6]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use RoutingRule.ProtoReflect.Descriptor instead.
func (*RoutingRule) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{6}
}

```

这段代码定义了两个函数，分别用于获取目标标签、标签和均衡标签。

第一个函数 `GetTargetTag` 接收一个指向 `RoutingRule` 类型对象的指针 `m`，并返回该 `RoutingRule` 类型对象 `m` 的 `TargetTag` 属性值，如果 `m` 为 `nil`，则返回 `nil`。

第二个函数 `GetTag` 接收一个指向 `RoutingRule` 类型对象的指针 `x`，并返回该 `RoutingRule` 类型对象 `x` 的 `Tag` 属性值，如果 `x` 为 `nil`，则返回 `""`（即空字符串）。

第三个函数 `GetBalancingTag` 与第二个函数类似，但它接收一个指向 `RoutingRule` 类型对象的指针 `x`，并返回该 `RoutingRule` 类型对象 `x` 的 `BalancingTag` 属性值，如果 `x` 为 `nil`，则返回 `""`。


```go
func (m *RoutingRule) GetTargetTag() isRoutingRule_TargetTag {
	if m != nil {
		return m.TargetTag
	}
	return nil
}

func (x *RoutingRule) GetTag() string {
	if x, ok := x.GetTargetTag().(*RoutingRule_Tag); ok {
		return x.Tag
	}
	return ""
}

func (x *RoutingRule) GetBalancingTag() string {
	if x, ok := x.GetTargetTag().(*RoutingRule_BalancingTag); ok {
		return x.BalancingTag
	}
	return ""
}

```

这两函数是定义在函数类型别名下（函数声明前有“func”修饰），实现在一个名为“x”的“*RoutingRule”类型的变量上。它们的作用是获取输入参数（即“x *RoutingRule”）中的域名和CIDR。

具体来说，这两函数如果输入参数“x”不等于 nil，就执行x.Domain和x.Cidr的Get操作，并返回。否则，返回 nil（即空）。

这两函数与另外两个函数一起作用于传入的“*RoutingRule”类型的变量，根据其定义，可以推断出这些函数可能与网络路由有关。


```go
func (x *RoutingRule) GetDomain() []*Domain {
	if x != nil {
		return x.Domain
	}
	return nil
}

// Deprecated: Do not use.
func (x *RoutingRule) GetCidr() []*CIDR {
	if x != nil {
		return x.Cidr
	}
	return nil
}

```

这两段代码定义了两个函数，分别是 `func (x *RoutingRule) GetGeoip() []*GeoIP` 和 `func (x *RoutingRule) GetPortRange() *net.PortRange`。

第一个函数 `func (x *RoutingRule) GetGeoip` 的作用是获取一个 `RoutingRule` 类型的参数 `x` 的 `Geoip` 字段值并返回。如果参数 `x` 有效，则返回 `x` 的 `Geoip` 字段值；如果 `x` 为空或者 `x` 的 `Geoip` 字段值为 `nil`，则返回 `nil`。

第二个函数 `func (x *RoutingRule) GetPortRange` 的作用是获取一个 `RoutingRule` 类型的参数 `x` 的 `PortRange` 字段值并返回。如果参数 `x` 有效，则返回 `x` 的 `PortRange` 字段值；如果 `x` 为空或者 `x` 的 `PortRange` 字段值为 `nil`，则返回 `nil`。


```go
func (x *RoutingRule) GetGeoip() []*GeoIP {
	if x != nil {
		return x.Geoip
	}
	return nil
}

// Deprecated: Do not use.
func (x *RoutingRule) GetPortRange() *net.PortRange {
	if x != nil {
		return x.PortRange
	}
	return nil
}

```

这两位作者都使用了“func”函数，但一个使用了“portList”类型，另一个使用了“networkList”类型。从语义上看，我认为第一个函数接收一个“RoutingRule”类型的参数，并返回一个“net.PortList”类型的指针；而第二个函数接收一个“RoutingRule”类型的参数，并返回一个“net.NetworkList”类型的指针。

第一个函数的作用是获取“RoutingRule”类型的实例中的“portList”字段值，并将其返回。在函数体中，首先检查“x”是否为空。如果是，则直接返回“x”实例中的“portList”字段值。否则，返回一个空的指针（nil）。

第二个函数的作用是获取“RoutingRule”类型的实例中的“networkList”字段值，并将其返回。在函数体中，首先检查“x”是否为空。如果是，则直接返回“x”实例中的“networkList”字段值。否则，返回一个空的指针（nil）。

需要注意的是，第一个函数的参数使用了“*RoutingRule”类型，并且返回了一个“net.PortList”类型的指针。而第二个函数的参数使用了“RoutingRule”类型，并且返回了一个“net.NetworkList”类型的指针。这可能是故意的设计，因为这两个字段的名字不同，而且它们的类型也不同。


```go
func (x *RoutingRule) GetPortList() *net.PortList {
	if x != nil {
		return x.PortList
	}
	return nil
}

// Deprecated: Do not use.
func (x *RoutingRule) GetNetworkList() *net.NetworkList {
	if x != nil {
		return x.NetworkList
	}
	return nil
}

```

这两函数的作用如下：

1. `func (x *RoutingRule) GetNetworks() []net.Network`

此函数接收一个 `RoutingRule` 类型的参数 `x`，并在检查 `x` 是否为 `nil` 之后返回 `x.Networks`。`RoutingRule` 是一个 `net.RoutingRule` 类型的数据结构，它包含了网络的规则，如网关、子网掩码等。通过调用此函数，可以获取到 `x` 所对应的网络，如果 `x` 为 `nil`，则返回一个空列表。

2. `func (x *RoutingRule) GetSourceCidr() []*CIDR`

此函数接收一个 `RoutingRule` 类型的参数 `x`，并在检查 `x` 是否为 `nil` 之后返回 `x.SourceCidr`。`CIDR` 是 IP 地址段的一种表示方法，通过将 IP 地址段划分为多个 CIDR 片段，可以更方便地进行 IP 地址的管理。通过调用此函数，可以获取到 `x` 所对应的源 CIDR，如果 `x` 为 `nil`，则返回一个空列表。注意，这个函数是过时的，因为早先的 `RoutingRule` 类型可能不支持 `CIDR` 这样的参数，使用这种参数可能会导致编译错误或者运行时错误。


```go
func (x *RoutingRule) GetNetworks() []net.Network {
	if x != nil {
		return x.Networks
	}
	return nil
}

// Deprecated: Do not use.
func (x *RoutingRule) GetSourceCidr() []*CIDR {
	if x != nil {
		return x.SourceCidr
	}
	return nil
}

```

这两段代码都是函数，接收一个名为 x 的 *RoutingRule 类型的参数，并返回相应的函数值。

第一段代码：`func (x *RoutingRule) GetSourceGeoip() []*GeoIP`

这个函数的作用是获取 *RoutingRule 中的SourceGeoip成员。如果 x 指向的值不为 nil，那么它返回x的SourceGeoip成员；否则返回 nil。

第二段代码：`func (x *RoutingRule) GetSourcePortList() *net.PortList`

这个函数的作用是获取 *RoutingRule 中的SourcePortList成员。如果 x 指向的值不为 nil，那么它返回x的SourcePortList成员；否则返回 nil。


```go
func (x *RoutingRule) GetSourceGeoip() []*GeoIP {
	if x != nil {
		return x.SourceGeoip
	}
	return nil
}

func (x *RoutingRule) GetSourcePortList() *net.PortList {
	if x != nil {
		return x.SourcePortList
	}
	return nil
}

func (x *RoutingRule) GetUserEmail() []string {
	if x != nil {
		return x.UserEmail
	}
	return nil
}

```

这是一段用于获取路由规则中入站标签、协议和属性的函数。

函数接收一个名为 `x` 的 `RoutingRule` 类型的参数，并返回其中 `InboundTag`、`Protocol` 和 `Attributes` 字段的值。

如果 `x` 本身不为 ` nil`，则函数首先尝试返回 `x` 的 `InboundTag` 字段，如果为 ` nil` 则返回 `nil`。如果 `x` 为 ` nil`，则函数为空 ` []string` 返回 `nil`。

如果 `x` 为 ` RoutingRule` 类型的 ` nil` 值，则函数为空 ` []string` 返回 `nil`。如果 `x` 为 ` RoutingRule` 类型的非 ` nil` 值，则函数返回 `x` 的 `InboundTag`、`Protocol` 和 `Attributes` 字段的值。


```go
func (x *RoutingRule) GetInboundTag() []string {
	if x != nil {
		return x.InboundTag
	}
	return nil
}

func (x *RoutingRule) GetProtocol() []string {
	if x != nil {
		return x.Protocol
	}
	return nil
}

func (x *RoutingRule) GetAttributes() string {
	if x != nil {
		return x.Attributes
	}
	return ""
}

```

这段代码定义了一个名为 isRoutingRule_TargetTag 的接口类型，它由一个名为 RoutingRule_Tag 的结构体成员组成。

具体来说，该接口类型有一个名为 isRoutingRule_TargetTag 的方法，该方法返回一个布尔值。

然后，该接口类型的结构体成员中包含一个名为 Tag 的字段，它是一个字符串类型，用于表示出站路由规则的目标标记。

接着，该接口类型的结构体成员中包含一个名为 BalancingTag 的字段，它也是一个字符串类型，用于表示路由平衡器的标签。

最后，该接口类型的 isRoutingRule_TargetTag 方法包括一个 isRoutingRule_TargetTag 函数，该函数返回一个布尔值，表示出站路由规则的目标标记是否为空。


```go
type isRoutingRule_TargetTag interface {
	isRoutingRule_TargetTag()
}

type RoutingRule_Tag struct {
	// Tag of outbound that this rule is pointing to.
	Tag string `protobuf:"bytes,1,opt,name=tag,proto3,oneof"`
}

type RoutingRule_BalancingTag struct {
	// Tag of routing balancer.
	BalancingTag string `protobuf:"bytes,12,opt,name=balancing_tag,json=balancingTag,proto3,oneof"`
}

func (*RoutingRule_Tag) isRoutingRule_TargetTag() {}

```

该代码定义了一个名为 BalancingRule 的 struct 类型，用于表示路由策略中的一个规则。该 struct 类型包含一个名为 Tag 的字符串类型和一个名为 OutboundSelector 的字符串类型，用于表示该规则的出方向选择。

此外，该 struct 类型还包含一个名为 state 的保护类型变量，该类型变量被声明为 protobuf.MessageState。该类型的实现依赖于您的 protobuf 定义，可能是一个独立的 protobuf 文件或者是您的项目中的默认定义。如果该文件未定义，该类型的默认实现将默认为 nil。

最后，该 struct 类型包含一个名为 unknownFields 的保护类型变量，该类型变量被声明为 protobuf.UnknownFields。该类型的实现依赖于您的 protobuf 定义，可能是一个独立的 protobuf 文件或者是您的项目中的默认定义。如果该文件未定义，该类型的默认实现将默认为 nil。


```go
func (*RoutingRule_BalancingTag) isRoutingRule_TargetTag() {}

type BalancingRule struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Tag              string   `protobuf:"bytes,1,opt,name=tag,proto3" json:"tag,omitempty"`
	OutboundSelector []string `protobuf:"bytes,2,rep,name=outbound_selector,json=outboundSelector,proto3" json:"outbound_selector,omitempty"`
}

func (x *BalancingRule) Reset() {
	*x = BalancingRule{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[7]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为"BalancingRule"的接口类型，并实现了两个方法：

1. "String()": 该方法将包装一个"BalancingRule"类型的实例，并返回一个字符串表示该实例。根据protoimpl.X.MessageStringOf函数，这个字符串是由X类型生成的，其中X是接口类型"BalancingRule"的实现者。

2. "ProtoMessage()": 该方法返回一个"BalancingRule"类型的接口类型的实例，并且使用protoimpl.UnsafeEnabled选项启用类型检查。这个方法没有实现，只是一个空方法，因为"BalancingRule"接口没有定义任何需要反射的成员变量。

3. "ProtoReflect()": 该方法返回一个protoreflect.Message类型的接口类型的实例，该实例表示"BalancingRule"类型的实例。这个方法首先检查是否启用了类型检查，如果是，那么就会创建一个指向"BalancingRule"类型实例的指针，然后使用该指针获取该实例的类型信息。如果类型信息没有被加载，那么就会创建一个空的"BalancingRule"类型实例，并将其存储为mi。最后，返回mi作为消息类型。


```go
func (x *BalancingRule) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*BalancingRule) ProtoMessage() {}

func (x *BalancingRule) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[7]
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

此代码是一个 Go 语言中的函数指针，它概述了一个名为 `BalancingRule` 的类型，并提供了该类型的两个方法：

1. `Descriptor()` 函数使用 `file_app_router_config_proto_rawDescGZIP()` 函数返回一个元组的表示，然后返回一个字符串类型的切片，片区的长度为该元组的长度，即两个整数的并集，它们的值分别为该元组中的两个元素的索引号。
2. `GetTag()` 函数使用 `x != nil` 条件语句检查是否关联到一个名为 `x` 的变量，如果是，则返回该变量的 `Tag` 字段，否则返回一个空字符串。
3. `GetOutboundSelector()` 函数使用 `x != nil` 条件语句检查是否关联到一个名为 `x` 的变量，如果是，则返回该变量的 `OutboundSelector` 字段，否则返回一个空字符串。


```go
// Deprecated: Use BalancingRule.ProtoReflect.Descriptor instead.
func (*BalancingRule) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{7}
}

func (x *BalancingRule) GetTag() string {
	if x != nil {
		return x.Tag
	}
	return ""
}

func (x *BalancingRule) GetOutboundSelector() []string {
	if x != nil {
		return x.OutboundSelector
	}
	return nil
}

```

这段代码定义了一个名为 Config 的结构体类型，包含了 state、sizeCache 和 unknownFields。

Config 结构体代表了文件中名为 Config 的配置项。其中，state 表示一个 Protocol Impl 实现的 MessageState，sizeCache 表示一个 SizeCache 实现的 MessageSize，unknownFields 表示一个 UnknownFields 实现的 MessageUnknownFields。

另外，还定义了一个名为 DomainStrategy 的 field，它是一个自定义的名称，对应于一个名为 v2ray.core.app.router.Config_DomainStrategy 的内置类型。

接着，定义了一个名为 Rule 的 field，它是一个由 RoutingRule 实现的结构体，包含了 routing 的规则。

最后，定义了一个名为 BalancingRule 的 field，它也是一个由 RoutingRule 实现的结构体，包含了平衡的规则。

总的来说，这段代码定义了一个 Config 结构体类型，包含了用于描述文件中配置项的数据，以及一些用于实现自动路由的规则。


```go
type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	DomainStrategy Config_DomainStrategy `protobuf:"varint,1,opt,name=domain_strategy,json=domainStrategy,proto3,enum=v2ray.core.app.router.Config_DomainStrategy" json:"domain_strategy,omitempty"`
	Rule           []*RoutingRule        `protobuf:"bytes,2,rep,name=rule,proto3" json:"rule,omitempty"`
	BalancingRule  []*BalancingRule      `protobuf:"bytes,3,rep,name=balancing_rule,json=balancingRule,proto3" json:"balancing_rule,omitempty"`
}

func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[8]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`Config`的整数类型的参数`x`，并返回一个字符串类型的结果。

具体来说，这段代码实现了一个名为`string`的类型转换函数，该函数接收一个`Config`类型的参数`x`，并将其转换为一个字符串类型。这个函数的实现主要依赖于`protoimpl`和`protoreflect`两个库，它们定义了与该函数类型相关的接口和类型。

首先，在函数体中，定义了一个名为`func`的函数，接收一个名为`Config`的整数类型的参数`x`，并将其发送到一个名为`protoimpl.X.MessageStringOf`的函数中，这个函数接收一个`Config`类型的指针类型的参数`x`，并返回一个字符串类型的结果。这个函数的作用类似于`str.ToLowercase`函数，将输入参数中的`Config`类型转换为字符串类型并返回。

接下来，定义了一个名为`func`的函数，接收一个名为`Config`的整数类型的参数`x`，并将其返回一个名为`protoimpl.X.MessageStateOf`的函数中，这个函数接收一个`Config`类型的指针类型的参数`x`，并返回一个指向`message_api.Config`类型的`MessageStateOf`函数的指针类型。这个函数的作用类似于`李证明论`函数，将输入参数中的`Config`类型转换为一个`message_api.Config`类型，并返回该类型的`MessageStateOf`函数的指针类型。

最后，在`func`函数内部，定义了一个名为`x`的变量，其类型为`message_api.Config`类型。这个变量在`func`函数内部被用作参数传递给`protoimpl.X.MessageStringOf`函数，然后又被用于获取`MessageStringOf`函数返回的结果。


```go
func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[8]
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

这段代码是一个 Go 语言中的函数，定义在名为 "file_app_router_config_proto_rawDescGZIP" 的文件中。它实现了 Config 类型的依赖关系，用于从一个名为 "file_app_router_config_proto_rawDescGZIP" 的文件中读取相应的配置描述，并返回两个值：一个字节切片和一个整数切片。

具体来说，这段代码实现了一个 Config 类型的函数 "Descriptor"，该函数返回了从原始文件中读取的配置描述。通过调用这个函数，可以访问到原始文件中定义的配置项，如文件、路由策略等。

此外，还实现了一个名为 "GetDomainStrategy" 的函数，用于从 Config 类型中获取 "domain_strategy" 字段对应的值。如果当前的 Config 对象已经被关联了一个非 nil 的 "domain_strategy"，则返回该策略；否则，返回一个名为 Config_AsIs 的默认策略。

还实现了一个名为 "GetRule" 的函数，用于从 Config 类型中获取 "rule" 字段对应的值。如果当前的 Config 对象已经被关联了一个非 nil 的 "rule"，则返回该规则；否则，返回一个名为 nil 的空字符串。


```go
// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{8}
}

func (x *Config) GetDomainStrategy() Config_DomainStrategy {
	if x != nil {
		return x.DomainStrategy
	}
	return Config_AsIs
}

func (x *Config) GetRule() []*RoutingRule {
	if x != nil {
		return x.Rule
	}
	return nil
}

```

该函数接收一个`Config`类型的参数`x`，并返回一个`[]*BalancingRule`类型的值。函数首先检查`x`是否为`nil`，如果是，则返回一个空切片。否则，函数返回`x.BalancingRule`。

该函数定义了一个`Domain_Attribute`类型，该类型包含一个键、一个`sizeCache`和一个`unknownFields`字段。函数的`protobuf`定义了一个`message`类型，其中包括了`Domain_Attribute`类型，该类型包含一个名为`key`的整数值。

该函数还定义了一个`isDomain_Attribute_TypedValue`类型，该类型是一个`Oneof`类型，允许将`Domain_Attribute_TypedValue`和`Domain_Attribute_UnknownValue`中的任何一个作为`isDomain_Attribute_TypedValue`的`typed_value`字段的值。

该函数的作用是返回一个`[]*BalancingRule`类型的值，该值是由`x.BalancingRule`返回的，如果`x`为`nil`则返回一个空切片。


```go
func (x *Config) GetBalancingRule() []*BalancingRule {
	if x != nil {
		return x.BalancingRule
	}
	return nil
}

type Domain_Attribute struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Key string `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`
	// Types that are assignable to TypedValue:
	//	*Domain_Attribute_BoolValue
	//	*Domain_Attribute_IntValue
	TypedValue isDomain_Attribute_TypedValue `protobuf_oneof:"typed_value"`
}

```

这是一个 Go 语言中的接口类型 `Domain_Attribute` 的函数指针。它包括了两个函数：`Reset()` 和 `String()`。

1. `Reset()` 函数的作用是重置 `x` 对象，将其设置为 `Domain_Attribute{}`。这个函数没有做任何实际的计算或者修改，只是将 `x` 对象的状态设置为默认状态。

2. `String()` 函数的作用是将 `x` 对象转换为字符串形式并返回。这个函数使用了 `protoimpl.X.MessageStringOf()` 函数，它通过 `x` 对象所指向的 `Domain_Attribute` 类型中的 `MessageInfo` 字段，将 `x` 对象转换为字符串并返回。

3. `ProtoMessage()` 函数的作用是定义一个接口 `Domain_Attribute` 的函数指针。这个函数指针返回的对象将会在定义的接口中继承自 `Message` 接口，因此它可以从任何实现了 `Message` 接口的对象上继承方法。这个函数指针的实现与具体接口的接口定义无关，因此它可以在任何实现了 `Message` 接口的对象上使用。


```go
func (x *Domain_Attribute) Reset() {
	*x = Domain_Attribute{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_router_config_proto_msgTypes[9]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Domain_Attribute) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Domain_Attribute) ProtoMessage() {}

```

此代码定义了两个函数，分别是`func (x *Domain_Attribute) ProtoReflect() protoreflect.Message` 和 `func (*Domain_Attribute) Descriptor() ([]byte, []int)`。

函数1：`func (x *Domain_Attribute) ProtoReflect() protoreflect.Message`
此函数接收一个`Domain_Attribute`类型的参数`x`，并返回一个`protoreflect.Message`类型的变量。函数的作用是将`x`包装成一个`protoreflect.Message`类型的实例，其中包括了`Domain_Attribute`类型的一些元数据。

函数2：`func (*Domain_Attribute) Descriptor() ([]byte, []int)`
此函数接收一个`Domain_Attribute`类型的参数`x`，并返回一个字节数组和一个整数数组。函数的作用是返回`x`的元数据，其中包括了`Domain_Attribute`类型的一些元数据。需要注意的是，此函数是过时的，建议使用`Descriptor`函数代替。


```go
func (x *Domain_Attribute) ProtoReflect() protoreflect.Message {
	mi := &file_app_router_config_proto_msgTypes[9]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Domain_Attribute.ProtoReflect.Descriptor instead.
func (*Domain_Attribute) Descriptor() ([]byte, []int) {
	return file_app_router_config_proto_rawDescGZIP(), []int{0, 0}
}

```

这段代码定义了两个函数，分别是 `func (x *Domain_Attribute) GetKey() string` 和 `func (m *Domain_Attribute) GetTypedValue() isDomain_Attribute_TypedValue`。

第一个函数 `func (x *Domain_Attribute) GetKey() string` 的参数是一个指向 `Domain_Attribute` 类型的指针变量 `x`，函数的作用是获取 `x` 的键（也就是 `x` 的类型所代表的意义）。如果 `x` ≠ `nil`，则返回 `x` 的键；否则返回一个空字符串。这个函数可以看作是一个简单的数据类型检查，判断 `x` 是否为 `nil`。

第二个函数 `func (m *Domain_Attribute) GetTypedValue() isDomain_Attribute_TypedValue` 的参数是一个指向 `Domain_Attribute` 类型的指针变量 `m`，函数的作用是获取 `m` 的类型（也就是 `m` 所代表的意义）。如果 `m` ≠ `nil`，则返回 `m` 的类型；否则返回 `nil`。这个函数可以看作是一个简单的数据类型检查，判断 `m` 是否为 `nil`。

第三个函数 `func (x *Domain_Attribute) GetBoolValue() bool` 的参数是一个指向 `Domain_Attribute_BoolValue` 类型的指针变量 `x`，函数的作用是获取 `x` 的布尔类型（也就是 `x` 所代表的意义）。如果 `x`，`m` 均不是 `nil`，则返回 `x` 的布尔类型；否则返回 `false`。


```go
func (x *Domain_Attribute) GetKey() string {
	if x != nil {
		return x.Key
	}
	return ""
}

func (m *Domain_Attribute) GetTypedValue() isDomain_Attribute_TypedValue {
	if m != nil {
		return m.TypedValue
	}
	return nil
}

func (x *Domain_Attribute) GetBoolValue() bool {
	if x, ok := x.GetTypedValue().(*Domain_Attribute_BoolValue); ok {
		return x.BoolValue
	}
	return false
}

```

该函数接受一个`Domain_Attribute`类型的参数`x`，并返回一个`int64`类型的值。函数的作用是获取`x`的`IntValue`属性。

首先，函数检查`x`是否为`Domain_Attribute_IntValue`类型。如果是，函数将返回`x.IntValue`的值。如果不是，函数将返回`0`。

接下来，函数定义了一个`isDomain_Attribute_TypedValue`接口，该接口要求返回`isDomain_Attribute_TypedValue()`的`isDomain_Attribute_TypedValue()`方法返回`true`。

最后，函数定义了一个`Domain_Attribute_BoolValue`结构体，该结构体包含一个`bool`类型的字段`BoolValue`。


```go
func (x *Domain_Attribute) GetIntValue() int64 {
	if x, ok := x.GetTypedValue().(*Domain_Attribute_IntValue); ok {
		return x.IntValue
	}
	return 0
}

type isDomain_Attribute_TypedValue interface {
	isDomain_Attribute_TypedValue()
}

type Domain_Attribute_BoolValue struct {
	BoolValue bool `protobuf:"varint,2,opt,name=bool_value,json=boolValue,proto3,oneof"`
}

```

I'm sorry, I am not sure what you are asking for. Could you please provide more context or clarify your question?



```go
type Domain_Attribute_IntValue struct {
	IntValue int64 `protobuf:"varint,3,opt,name=int_value,json=intValue,proto3,oneof"`
}

func (*Domain_Attribute_BoolValue) isDomain_Attribute_TypedValue() {}

func (*Domain_Attribute_IntValue) isDomain_Attribute_TypedValue() {}

var File_app_router_config_proto protoreflect.FileDescriptor

var file_app_router_config_proto_rawDesc = []byte{
	0x0a, 0x17, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2f, 0x63, 0x6f, 0x6e,
	0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61, 0x79,
	0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x1a, 0x15, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x70, 0x6f, 0x72,
	0x74, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f,
	0x6e, 0x65, 0x74, 0x2f, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x2e, 0x70, 0x72, 0x6f, 0x74,
	0x6f, 0x22, 0xbf, 0x02, 0x0a, 0x06, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x36, 0x0a, 0x04,
	0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x22, 0x2e, 0x76, 0x32, 0x72,
	0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74,
	0x65, 0x72, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2e, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04,
	0x74, 0x79, 0x70, 0x65, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02, 0x20,
	0x01, 0x28, 0x09, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x12, 0x45, 0x0a, 0x09, 0x61, 0x74,
	0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x27, 0x2e,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
	0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2e, 0x41, 0x74, 0x74,
	0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x52, 0x09, 0x61, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74,
	0x65, 0x1a, 0x6c, 0x0a, 0x09, 0x41, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x12, 0x10,
	0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79,
	0x12, 0x1f, 0x0a, 0x0a, 0x62, 0x6f, 0x6f, 0x6c, 0x5f, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x02,
	0x20, 0x01, 0x28, 0x08, 0x48, 0x00, 0x52, 0x09, 0x62, 0x6f, 0x6f, 0x6c, 0x56, 0x61, 0x6c, 0x75,
	0x65, 0x12, 0x1d, 0x0a, 0x09, 0x69, 0x6e, 0x74, 0x5f, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x03,
	0x20, 0x01, 0x28, 0x03, 0x48, 0x00, 0x52, 0x08, 0x69, 0x6e, 0x74, 0x56, 0x61, 0x6c, 0x75, 0x65,
	0x42, 0x0d, 0x0a, 0x0b, 0x74, 0x79, 0x70, 0x65, 0x64, 0x5f, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x22,
	0x32, 0x0a, 0x04, 0x54, 0x79, 0x70, 0x65, 0x12, 0x09, 0x0a, 0x05, 0x50, 0x6c, 0x61, 0x69, 0x6e,
	0x10, 0x00, 0x12, 0x09, 0x0a, 0x05, 0x52, 0x65, 0x67, 0x65, 0x78, 0x10, 0x01, 0x12, 0x0a, 0x0a,
	0x06, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x10, 0x02, 0x12, 0x08, 0x0a, 0x04, 0x46, 0x75, 0x6c,
	0x6c, 0x10, 0x03, 0x22, 0x2e, 0x0a, 0x04, 0x43, 0x49, 0x44, 0x52, 0x12, 0x0e, 0x0a, 0x02, 0x69,
	0x70, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0c, 0x52, 0x02, 0x69, 0x70, 0x12, 0x16, 0x0a, 0x06, 0x70,
	0x72, 0x65, 0x66, 0x69, 0x78, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x06, 0x70, 0x72, 0x65,
	0x66, 0x69, 0x78, 0x22, 0x5b, 0x0a, 0x05, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x12, 0x21, 0x0a, 0x0c,
	0x63, 0x6f, 0x75, 0x6e, 0x74, 0x72, 0x79, 0x5f, 0x63, 0x6f, 0x64, 0x65, 0x18, 0x01, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x0b, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x72, 0x79, 0x43, 0x6f, 0x64, 0x65, 0x12,
	0x2f, 0x0a, 0x04, 0x63, 0x69, 0x64, 0x72, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1b, 0x2e,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
	0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x43, 0x49, 0x44, 0x52, 0x52, 0x04, 0x63, 0x69, 0x64, 0x72,
	0x22, 0x3f, 0x0a, 0x09, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x32, 0x0a,
	0x05, 0x65, 0x6e, 0x74, 0x72, 0x79, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
	0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x52, 0x05, 0x65, 0x6e, 0x74, 0x72,
	0x79, 0x22, 0x63, 0x0a, 0x07, 0x47, 0x65, 0x6f, 0x53, 0x69, 0x74, 0x65, 0x12, 0x21, 0x0a, 0x0c,
	0x63, 0x6f, 0x75, 0x6e, 0x74, 0x72, 0x79, 0x5f, 0x63, 0x6f, 0x64, 0x65, 0x18, 0x01, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x0b, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x72, 0x79, 0x43, 0x6f, 0x64, 0x65, 0x12,
	0x35, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32,
	0x1d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
	0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x06,
	0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x22, 0x43, 0x0a, 0x0b, 0x47, 0x65, 0x6f, 0x53, 0x69, 0x74,
	0x65, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x34, 0x0a, 0x05, 0x65, 0x6e, 0x74, 0x72, 0x79, 0x18, 0x01,
	0x20, 0x03, 0x28, 0x0b, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
	0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f,
	0x53, 0x69, 0x74, 0x65, 0x52, 0x05, 0x65, 0x6e, 0x74, 0x72, 0x79, 0x22, 0xca, 0x06, 0x0a, 0x0b,
	0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x12, 0x12, 0x0a, 0x03, 0x74,
	0x61, 0x67, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x48, 0x00, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12,
	0x25, 0x0a, 0x0d, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x5f, 0x74, 0x61, 0x67,
	0x18, 0x0c, 0x20, 0x01, 0x28, 0x09, 0x48, 0x00, 0x52, 0x0c, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63,
	0x69, 0x6e, 0x67, 0x54, 0x61, 0x67, 0x12, 0x35, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
	0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x44,
	0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x33, 0x0a,
	0x04, 0x63, 0x69, 0x64, 0x72, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1b, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75,
	0x74, 0x65, 0x72, 0x2e, 0x43, 0x49, 0x44, 0x52, 0x42, 0x02, 0x18, 0x01, 0x52, 0x04, 0x63, 0x69,
	0x64, 0x72, 0x12, 0x32, 0x0a, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x18, 0x0a, 0x20, 0x03, 0x28,
	0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61,
	0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x52,
	0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x12, 0x43, 0x0a, 0x0a, 0x70, 0x6f, 0x72, 0x74, 0x5f, 0x72,
	0x61, 0x6e, 0x67, 0x65, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72,
	0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e,
	0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x42, 0x02, 0x18, 0x01,
	0x52, 0x09, 0x70, 0x6f, 0x72, 0x74, 0x52, 0x61, 0x6e, 0x67, 0x65, 0x12, 0x3c, 0x0a, 0x09, 0x70,
	0x6f, 0x72, 0x74, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x0e, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1f,
	0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
	0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x52,
	0x08, 0x70, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x49, 0x0a, 0x0c, 0x6e, 0x65, 0x74,
	0x77, 0x6f, 0x72, 0x6b, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x05, 0x20, 0x01, 0x28, 0x0b, 0x32,
	0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d,
	0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x4c,
	0x69, 0x73, 0x74, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0b, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b,
	0x4c, 0x69, 0x73, 0x74, 0x12, 0x3a, 0x0a, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73,
	0x18, 0x0d, 0x20, 0x03, 0x28, 0x0e, 0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x4e,
	0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x52, 0x08, 0x6e, 0x65, 0x74, 0x77, 0x6f, 0x72, 0x6b, 0x73,
	0x12, 0x40, 0x0a, 0x0b, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x63, 0x69, 0x64, 0x72, 0x18,
	0x06, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
	0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x43, 0x49,
	0x44, 0x52, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0a, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x43, 0x69,
	0x64, 0x72, 0x12, 0x3f, 0x0a, 0x0c, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x67, 0x65, 0x6f,
	0x69, 0x70, 0x18, 0x0b, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
	0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50, 0x52, 0x0b, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x47, 0x65,
	0x6f, 0x69, 0x70, 0x12, 0x49, 0x0a, 0x10, 0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x5f, 0x70, 0x6f,
	0x72, 0x74, 0x5f, 0x6c, 0x69, 0x73, 0x74, 0x18, 0x10, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1f, 0x2e,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
	0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x52, 0x0e,
	0x73, 0x6f, 0x75, 0x72, 0x63, 0x65, 0x50, 0x6f, 0x72, 0x74, 0x4c, 0x69, 0x73, 0x74, 0x12, 0x1d,
	0x0a, 0x0a, 0x75, 0x73, 0x65, 0x72, 0x5f, 0x65, 0x6d, 0x61, 0x69, 0x6c, 0x18, 0x07, 0x20, 0x03,
	0x28, 0x09, 0x52, 0x09, 0x75, 0x73, 0x65, 0x72, 0x45, 0x6d, 0x61, 0x69, 0x6c, 0x12, 0x1f, 0x0a,
	0x0b, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x5f, 0x74, 0x61, 0x67, 0x18, 0x08, 0x20, 0x03,
	0x28, 0x09, 0x52, 0x0a, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x54, 0x61, 0x67, 0x12, 0x1a,
	0x0a, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x18, 0x09, 0x20, 0x03, 0x28, 0x09,
	0x52, 0x08, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x12, 0x1e, 0x0a, 0x0a, 0x61, 0x74,
	0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x18, 0x0f, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0a,
	0x61, 0x74, 0x74, 0x72, 0x69, 0x62, 0x75, 0x74, 0x65, 0x73, 0x42, 0x0c, 0x0a, 0x0a, 0x74, 0x61,
	0x72, 0x67, 0x65, 0x74, 0x5f, 0x74, 0x61, 0x67, 0x22, 0x4e, 0x0a, 0x0d, 0x42, 0x61, 0x6c, 0x61,
	0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61, 0x67,
	0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x12, 0x2b, 0x0a, 0x11, 0x6f,
	0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x5f, 0x73, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72,
	0x18, 0x02, 0x20, 0x03, 0x28, 0x09, 0x52, 0x10, 0x6f, 0x75, 0x74, 0x62, 0x6f, 0x75, 0x6e, 0x64,
	0x53, 0x65, 0x6c, 0x65, 0x63, 0x74, 0x6f, 0x72, 0x22, 0xad, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e,
	0x66, 0x69, 0x67, 0x12, 0x55, 0x0a, 0x0f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x5f, 0x73, 0x74,
	0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x2c, 0x2e, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f,
	0x75, 0x74, 0x65, 0x72, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x44, 0x6f, 0x6d, 0x61,
	0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x0e, 0x64, 0x6f, 0x6d, 0x61,
	0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x36, 0x0a, 0x04, 0x72, 0x75,
	0x6c, 0x65, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x22, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
	0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72,
	0x2e, 0x52, 0x6f, 0x75, 0x74, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x52, 0x04, 0x72, 0x75,
	0x6c, 0x65, 0x12, 0x4b, 0x0a, 0x0e, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x5f,
	0x72, 0x75, 0x6c, 0x65, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x24, 0x2e, 0x76, 0x32, 0x72,
	0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74,
	0x65, 0x72, 0x2e, 0x42, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65,
	0x52, 0x0d, 0x62, 0x61, 0x6c, 0x61, 0x6e, 0x63, 0x69, 0x6e, 0x67, 0x52, 0x75, 0x6c, 0x65, 0x22,
	0x47, 0x0a, 0x0e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67,
	0x79, 0x12, 0x08, 0x0a, 0x04, 0x41, 0x73, 0x49, 0x73, 0x10, 0x00, 0x12, 0x09, 0x0a, 0x05, 0x55,
	0x73, 0x65, 0x49, 0x70, 0x10, 0x01, 0x12, 0x10, 0x0a, 0x0c, 0x49, 0x70, 0x49, 0x66, 0x4e, 0x6f,
	0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x10, 0x02, 0x12, 0x0e, 0x0a, 0x0a, 0x49, 0x70, 0x4f, 0x6e,
	0x44, 0x65, 0x6d, 0x61, 0x6e, 0x64, 0x10, 0x03, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x72,
	0x6f, 0x75, 0x74, 0x65, 0x72, 0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74,
	0x65, 0x72, 0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
	0x41, 0x70, 0x70, 0x2e, 0x52, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74,
	0x6f, 0x33,
}

```

The table shows the APIs that have the same protection level as the ones you listed before, and the ones that have a different protection level.


```go
var (
	file_app_router_config_proto_rawDescOnce sync.Once
	file_app_router_config_proto_rawDescData = file_app_router_config_proto_rawDesc
)

func file_app_router_config_proto_rawDescGZIP() []byte {
	file_app_router_config_proto_rawDescOnce.Do(func() {
		file_app_router_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_router_config_proto_rawDescData)
	})
	return file_app_router_config_proto_rawDescData
}

var file_app_router_config_proto_enumTypes = make([]protoimpl.EnumInfo, 2)
var file_app_router_config_proto_msgTypes = make([]protoimpl.MessageInfo, 10)
var file_app_router_config_proto_goTypes = []interface{}{
	(Domain_Type)(0),           // 0: v2ray.core.app.router.Domain.Type
	(Config_DomainStrategy)(0), // 1: v2ray.core.app.router.Config.DomainStrategy
	(*Domain)(nil),             // 2: v2ray.core.app.router.Domain
	(*CIDR)(nil),               // 3: v2ray.core.app.router.CIDR
	(*GeoIP)(nil),              // 4: v2ray.core.app.router.GeoIP
	(*GeoIPList)(nil),          // 5: v2ray.core.app.router.GeoIPList
	(*GeoSite)(nil),            // 6: v2ray.core.app.router.GeoSite
	(*GeoSiteList)(nil),        // 7: v2ray.core.app.router.GeoSiteList
	(*RoutingRule)(nil),        // 8: v2ray.core.app.router.RoutingRule
	(*BalancingRule)(nil),      // 9: v2ray.core.app.router.BalancingRule
	(*Config)(nil),             // 10: v2ray.core.app.router.Config
	(*Domain_Attribute)(nil),   // 11: v2ray.core.app.router.Domain.Attribute
	(*net.PortRange)(nil),      // 12: v2ray.core.common.net.PortRange
	(*net.PortList)(nil),       // 13: v2ray.core.common.net.PortList
	(*net.NetworkList)(nil),    // 14: v2ray.core.common.net.NetworkList
	(net.Network)(0),           // 15: v2ray.core.common.net.Network
}
```

It looks like the `v2ray.core.app.router.RoutingRule` object has several properties that are used to define a routing rule. These properties include `type_name`, `source_cidr`, `source_geoip`, `source_port_list`, `domain_strategy`, `rule`, and `balancing_rule`.

The `type_name` property is a string that represents the type of routing rule this rule belongs to. For example, "v2ray.core.app.router.V2RayRouterRule" would represent a routing rule that uses the V2Ray router.

The `source_cidr` property is a string that represents the source IP address CIDR format. This property is used to specify the IP address of the device or network that should be considered when matching this rule.

The `source_geoip` property is a string that represents the source IP address in GeoIP format. This property is used to specify the IP address of the device or network that should be considered when matching this rule.

The `source_port_list` property is a list of strings that represents the port numbers on which this rule should allow connections from. This property is used to specify which ports on the device or network should be accessible from this rule.

The `domain_strategy` property is a string that represents the strategy for resolving domain names when matching this rule. This property is used to specify how domain names should be resolved when matching this rule.

The `rule` property is a dictionary that represents the routing rule. This property is used to define the behavior of the routing rule.

The `balancing_rule` property is a dictionary that represents the balancing rule. This property is used to define the balancing behavior of the rule.


```go
var file_app_router_config_proto_depIdxs = []int32{
	0,  // 0: v2ray.core.app.router.Domain.type:type_name -> v2ray.core.app.router.Domain.Type
	11, // 1: v2ray.core.app.router.Domain.attribute:type_name -> v2ray.core.app.router.Domain.Attribute
	3,  // 2: v2ray.core.app.router.GeoIP.cidr:type_name -> v2ray.core.app.router.CIDR
	4,  // 3: v2ray.core.app.router.GeoIPList.entry:type_name -> v2ray.core.app.router.GeoIP
	2,  // 4: v2ray.core.app.router.GeoSite.domain:type_name -> v2ray.core.app.router.Domain
	6,  // 5: v2ray.core.app.router.GeoSiteList.entry:type_name -> v2ray.core.app.router.GeoSite
	2,  // 6: v2ray.core.app.router.RoutingRule.domain:type_name -> v2ray.core.app.router.Domain
	3,  // 7: v2ray.core.app.router.RoutingRule.cidr:type_name -> v2ray.core.app.router.CIDR
	4,  // 8: v2ray.core.app.router.RoutingRule.geoip:type_name -> v2ray.core.app.router.GeoIP
	12, // 9: v2ray.core.app.router.RoutingRule.port_range:type_name -> v2ray.core.common.net.PortRange
	13, // 10: v2ray.core.app.router.RoutingRule.port_list:type_name -> v2ray.core.common.net.PortList
	14, // 11: v2ray.core.app.router.RoutingRule.network_list:type_name -> v2ray.core.common.net.NetworkList
	15, // 12: v2ray.core.app.router.RoutingRule.networks:type_name -> v2ray.core.common.net.Network
	3,  // 13: v2ray.core.app.router.RoutingRule.source_cidr:type_name -> v2ray.core.app.router.CIDR
	4,  // 14: v2ray.core.app.router.RoutingRule.source_geoip:type_name -> v2ray.core.app.router.GeoIP
	13, // 15: v2ray.core.app.router.RoutingRule.source_port_list:type_name -> v2ray.core.common.net.PortList
	1,  // 16: v2ray.core.app.router.Config.domain_strategy:type_name -> v2ray.core.app.router.Config.DomainStrategy
	8,  // 17: v2ray.core.app.router.Config.rule:type_name -> v2ray.core.app.router.RoutingRule
	9,  // 18: v2ray.core.app.router.Config.balancing_rule:type_name -> v2ray.core.app.router.BalancingRule
	19, // [19:19] is the sub-list for method output_type
	19, // [19:19] is the sub-list for method input_type
	19, // [19:19] is the sub-list for extension type_name
	19, // [19:19] is the sub-list for extension extendee
	0,  // [0:19] is the sub-list for field type_name
}

```

It appears that this code is defining a struct类型的变量 `x`，但并没有为其提供任何特定的含义。同时，该 struct 似乎包含两个字段 `state` 和 `sizeCache`，以及一个名为 `unknownFields` 的字段，但具体情况并不清楚。

由于缺乏上下文，我们无法确定 `file_app_router_config_proto_msgTypes` 数组中包含的具体信息。但是，从 `OneofWrappers` 可以看出，这个结构体可能是从 `Domain_Attribute_BoolValue` 和 `Domain_Attribute_IntValue` 这样的消息中获取信息。

根据 `file_app_router_config_proto_goTypes` 和 `file_app_router_config_proto_rawDesc` 两个变量，我们可以推测这个结构体的定义可能来自某个名为 `file_app_router_config_proto` 的接口，但具体的信息仍然不清楚。


```go
func init() { file_app_router_config_proto_init() }
func file_app_router_config_proto_init() {
	if File_app_router_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_app_router_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Domain); i {
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
		file_app_router_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*CIDR); i {
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
		file_app_router_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GeoIP); i {
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
		file_app_router_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GeoIPList); i {
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
		file_app_router_config_proto_msgTypes[4].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GeoSite); i {
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
		file_app_router_config_proto_msgTypes[5].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*GeoSiteList); i {
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
		file_app_router_config_proto_msgTypes[6].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*RoutingRule); i {
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
		file_app_router_config_proto_msgTypes[7].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*BalancingRule); i {
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
		file_app_router_config_proto_msgTypes[8].Exporter = func(v interface{}, i int) interface{} {
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
		file_app_router_config_proto_msgTypes[9].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Domain_Attribute); i {
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
	file_app_router_config_proto_msgTypes[6].OneofWrappers = []interface{}{
		(*RoutingRule_Tag)(nil),
		(*RoutingRule_BalancingTag)(nil),
	}
	file_app_router_config_proto_msgTypes[9].OneofWrappers = []interface{}{
		(*Domain_Attribute_BoolValue)(nil),
		(*Domain_Attribute_IntValue)(nil),
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_app_router_config_proto_rawDesc,
			NumEnums:      2,
			NumMessages:   10,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_app_router_config_proto_goTypes,
		DependencyIndexes: file_app_router_config_proto_depIdxs,
		EnumInfos:         file_app_router_config_proto_enumTypes,
		MessageInfos:      file_app_router_config_proto_msgTypes,
	}.Build()
	File_app_router_config_proto = out.File
	file_app_router_config_proto_rawDesc = nil
	file_app_router_config_proto_goTypes = nil
	file_app_router_config_proto_depIdxs = nil
}

```