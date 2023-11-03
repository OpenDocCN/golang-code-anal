# v2ray-core源码解析 34

# `infra/conf/dns_proxy.go`

这段代码定义了一个名为 `DnsOutboundConfig` 的结构体，它用于配置一个用于域名解析的网络服务。

该结构体包含以下字段：

* `Network`：表示用于域名解析的网络服务。它通过 `json:` 标签与 `v2ray.com/core/common/net` 包中的 `Network` 字段进行关联。
* `Address`：表示域名服务器监听的地址。它通过 `json:` 标签与 `v2ray.com/core/common/net` 包中的 `Address` 字段进行关联。
* `Port`：表示域名服务器的端口。它通过 `json:` 标签与 `v2ray.com/core/common/net` 包中的 `Port` 字段进行关联。

该结构体还包含一个名为 `Build` 的方法，该方法返回一个 `proto.Message` 类型的配置结构体，它是通过将 `DnsOutboundConfig` 结构体转换为 `proto.Message` 类型后得到的。如果转换过程中出现错误，该方法返回一个非 `nil` 的 `error`。

另外，该代码还定义了一个名为 `DnsOutboundConfigValidator` 的函数，该函数接收一个 `DnsOutboundConfig` 结构体作为输入参数，并返回一个是否有效的布尔值。


```go
package conf

import (
	"github.com/golang/protobuf/proto"
	"v2ray.com/core/common/net"
	"v2ray.com/core/proxy/dns"
)

type DnsOutboundConfig struct {
	Network Network  `json:"network"`
	Address *Address `json:"address"`
	Port    uint16   `json:"port"`
}

func (c *DnsOutboundConfig) Build() (proto.Message, error) {
	config := &dns.Config{
		Server: &net.Endpoint{
			Network: c.Network.Build(),
			Port:    uint32(c.Port),
		},
	}
	if c.Address != nil {
		config.Server.Address = c.Address.Build()
	}
	return config, nil
}

```

# `infra/conf/dns_proxy_test.go`

这段代码的作用是测试一个名为 "DnsProxyConfig" 的功能。该功能与 "v2ray.com/core/proxy/dns" 包有关。它通过创建一个名为 "DnsOutboundConfig" 的接口类型来实现，该类型表示一个配置 DNS 代理的参数。

首先，该代码定义了一个名为 "creator" 的函数，该函数返回一个名为 "DnsOutboundConfig" 的接口类型。这个函数的作用并未在代码中明确给出，但可以推测它可能用于创建一个 DNS 代理实例。

接下来，代码使用 "testing" 包中的 "runMultiTestCase" 函数来运行多个测试用例。每个测试用例都包含一个输入和一个预期输出。输入是一个 JSON 配置，表示要配置的 DNS 代理的参数。预期输出的类型与输入的类型相同，但不是用于测试 "DnsProxyConfig" 函数本身，而是用于测试 DNS 代理的连接。

具体地说，在每一个测试用例中，函数首先调用 "DnsOutboundConfig" 的 "new" 函数创建一个新的 "DnsOutboundConfig" 实例。然后，它调用另一个函数 "loadJSON" 并将新的 "DnsOutboundConfig" 实例和它的创建者一起传递给 "parser" 参数，这样 "parser" 可以读取并解析 JSON 配置文件。

接下来，函数创建一个名为 "server" 的新的 "net.Endpoint" 实例。然后，它设置 "server" 的 "network" 字段为 "tcp"，这意味着将要使用 TCP 网络连接。接下来，它设置 "server" 的 "address" 字段为传入的 IP 地址，并将 "port" 字段设置为 53，这是 DNS 代理的默认端口。

最后，函数使用 "net.NewIPOrDomain" 函数创建一个新的 IP 地址或域名，并将其作为 "address" 字段的值。这样，当 DNS 代理需要连接到 DNS 服务器时，它就可以使用传递给它的 IP 地址或域名了。

请注意，尽管 "DnsProxyConfig" 函数的名称和作用前程似相似，但它与 "v2ray.com/core/proxy/dns" 包中的其他函数和方法并没有直接关联。因此，如果你想要了解更多关于这些函数和方法的信息，可以查看该包的源代码。


```go
package conf_test

import (
	"testing"

	"v2ray.com/core/common/net"
	. "v2ray.com/core/infra/conf"
	"v2ray.com/core/proxy/dns"
)

func TestDnsProxyConfig(t *testing.T) {
	creator := func() Buildable {
		return new(DnsOutboundConfig)
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"address": "8.8.8.8",
				"port": 53,
				"network": "tcp"
			}`,
			Parser: loadJSON(creator),
			Output: &dns.Config{
				Server: &net.Endpoint{
					Network: net.Network_TCP,
					Address: net.NewIPOrDomain(net.IPAddress([]byte{8, 8, 8, 8})),
					Port:    53,
				},
			},
		},
	})
}

```

# `infra/conf/dns_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Conf 协议的实现。 Conf 协议是一个二进制文件传输协议，用于在本地计算机和远程服务器之间传输二进制定本。

具体来说，这个测试框架包括以下组件：

1. 从项目中导入一些需要的依赖项，例如 `encoding/json` 用于将 JSON 数据编码成字节序列，`os` 用于操作系统相关的操作，`path/filepath` 用于文件路径相关的操作，`testing` 用于用于测试。

2. 从 `github.com/golang/protobuf/proto` 导入用于定义 ProtocolBuffers。

3. 从 `v2ray.com/core/app/dns` 导入用于实现 DNS 解析。

4. 从 `v2ray.com/core/app/router` 导入用于实现路由器。

5. 从 `v2ray.com/core/common` 导入用于实现一些通用的功能，例如 JSON 序列化和反序列化，网络套接字操作等。

6. 从 `v2ray.com/core/common/net` 导入用于实现网络相关的操作。

7. 从 `v2ray.com/core/common/platform` 导入用于实现平台相关的操作。

8. 从 `v2ray.com/core/common/platform/filesystem` 导入用于实现文件系统相关的操作。

9. 从 `v2ray.com/core/infra/conf` 导入用于实现一些与 Infra 相关的配置。

10. 在package.json中声明了需要测试的文件和依赖项。

综上所述，这段代码的作用是用于测试 Conf 协议的实现，包括对协议的定义、测试、验证和部署等。


```go
package conf_test

import (
	"encoding/json"
	"os"
	"path/filepath"
	"testing"

	"github.com/golang/protobuf/proto"
	"v2ray.com/core/app/dns"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/platform"
	"v2ray.com/core/common/platform/filesystem"
	. "v2ray.com/core/infra/conf"
)

```

这段代码是一个匿名函数，也就是在不需要命名的情况下定义的。它负责将一些文件和配置文件复制到指定的目录中，并根据需要设置一些环境变量。

具体来说，它做了以下几件事情：

1. 获取当前工作目录（也就是用户当前所在的位置）。
2. 尝试运行 `os.Getwd()` 函数，获取出当前工作目录的路径。
3. 检查当前工作目录是否已经存在一个名为 `geoip.dat` 的文件，如果不存在，就复制这个文件到当前工作目录的上级目录（也就是 `..`）的子目录中，并且命名为 `release` 和 `config` 目录下的 `geoip.dat` 文件名。
4. 设置一个名为 `v2ray.location.asset` 的环境变量，并将当前工作目录的路径存储到该变量中。
5. 打开一个名为 `geoise.dat` 的文件，并将列表 `list` 中的所有元素写入该文件中。

列表 `list` 包含一个包含多个 `router.GeoSite` 对象的 `Entry` 字段，每个 `router.GeoSite` 对象包含一个国家的字符串 `CountryCode` 和一个域名列表 `Domain`。这里存储了一些地理信息，例如 `example.com` 是一个例子。


```go
func init() {
	wd, err := os.Getwd()
	common.Must(err)

	if _, err := os.Stat(platform.GetAssetLocation("geoip.dat")); err != nil && os.IsNotExist(err) {
		common.Must(filesystem.CopyFile(platform.GetAssetLocation("geoip.dat"), filepath.Join(wd, "..", "..", "release", "config", "geoip.dat")))
	}

	geositeFilePath := filepath.Join(wd, "geosite.dat")
	os.Setenv("v2ray.location.asset", wd)
	geositeFile, err := os.OpenFile(geositeFilePath, os.O_CREATE|os.O_WRONLY, 0600)
	common.Must(err)
	defer geositeFile.Close()

	list := &router.GeoSiteList{
		Entry: []*router.GeoSite{
			{
				CountryCode: "TEST",
				Domain: []*router.Domain{
					{Type: router.Domain_Full, Value: "example.com"},
				},
			},
		},
	}

	listBytes, err := proto.Marshal(list)
	common.Must(err)
	common.Must2(geositeFile.Write(listBytes))
}
```

This appears to be a valid `dns.Config_HostMapping` object for a v2ray.com domain. It defines a host with the hostname "v2ray.com". The `Size` field is set to 1, indicating that this is a static host. The `StaticHosts` field is a list of `dns.Config_HostMapping` objects, each of which defines a host with a different type of matching domain or keyword. The `dns.DomainMatchingType_Subdomain` matches any subdomain, `dns.DomainMatchingType_Full` matches the entire domain, `dns.DomainMatchingType_Keyword` matches a keyword, and `dns.DomainMatchingType_Regex` matches a regular expression.


```go
func TestDnsConfigParsing(t *testing.T) {
	geositePath := platform.GetAssetLocation("geosite.dat")
	defer func() {
		os.Remove(geositePath)
		os.Unsetenv("v2ray.location.asset")
	}()

	parserCreator := func() func(string) (proto.Message, error) {
		return func(s string) (proto.Message, error) {
			config := new(DnsConfig)
			if err := json.Unmarshal([]byte(s), config); err != nil {
				return nil, err
			}
			return config.Build()
		}
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"servers": [{
					"address": "8.8.8.8",
					"port": 5353,
					"domains": ["domain:v2ray.com"]
				}],
				"hosts": {
					"v2ray.com": "127.0.0.1",
					"domain:example.com": "google.com",
					"geosite:test": "10.0.0.1",
					"keyword:google": "8.8.8.8",
					"regexp:.*\\.com": "8.8.4.4"
				},
				"clientIp": "10.0.0.1"
			}`,
			Parser: parserCreator(),
			Output: &dns.Config{
				NameServer: []*dns.NameServer{
					{
						Address: &net.Endpoint{
							Address: &net.IPOrDomain{
								Address: &net.IPOrDomain_Ip{
									Ip: []byte{8, 8, 8, 8},
								},
							},
							Network: net.Network_UDP,
							Port:    5353,
						},
						PrioritizedDomain: []*dns.NameServer_PriorityDomain{
							{
								Type:   dns.DomainMatchingType_Subdomain,
								Domain: "v2ray.com",
							},
						},
						OriginalRules: []*dns.NameServer_OriginalRule{
							{
								Rule: "domain:v2ray.com",
								Size: 1,
							},
						},
					},
				},
				StaticHosts: []*dns.Config_HostMapping{
					{
						Type:          dns.DomainMatchingType_Subdomain,
						Domain:        "example.com",
						ProxiedDomain: "google.com",
					},
					{
						Type:   dns.DomainMatchingType_Full,
						Domain: "example.com",
						Ip:     [][]byte{{10, 0, 0, 1}},
					},
					{
						Type:   dns.DomainMatchingType_Keyword,
						Domain: "google",
						Ip:     [][]byte{{8, 8, 8, 8}},
					},
					{
						Type:   dns.DomainMatchingType_Regex,
						Domain: ".*\\.com",
						Ip:     [][]byte{{8, 8, 4, 4}},
					},
					{
						Type:   dns.DomainMatchingType_Full,
						Domain: "v2ray.com",
						Ip:     [][]byte{{127, 0, 0, 1}},
					},
				},
				ClientIp: []byte{10, 0, 0, 1},
			},
		},
	})
}

```

# `infra/conf/dokodemo.go`

这段代码定义了一个名为`DokodemoConfig`的结构体，用于配置Dokodemo代理的选项。

具体来说，这个结构体包含以下字段：

- `*Address`类型，类型为`Address`的别针，用于设置代理的地址。
- `uint16`类型的字段，类型为`uint16`的别针，用于设置代理的端口号。
- `*NetworkList`类型，类型为`NetworkList`的别针，用于设置是否使用代理的网络。
- `uint32`类型的字段，类型为`uint32`的别针，用于设置代理的超时时间。
- `bool`类型的字段，类型为`bool`的别针，用于设置是否启用重定向。
- `uint32`类型的字段，类型为`uint32`的别针，用于设置用户级别。

这个结构体的值将被转换为JSON格式，并存储在`conf`包中的`DokodemoConfig`字段中。


```go
package conf

import (
	"github.com/golang/protobuf/proto"
	"v2ray.com/core/proxy/dokodemo"
)

type DokodemoConfig struct {
	Host         *Address     `json:"address"`
	PortValue    uint16       `json:"port"`
	NetworkList  *NetworkList `json:"network"`
	TimeoutValue uint32       `json:"timeout"`
	Redirect     bool         `json:"followRedirect"`
	UserLevel    uint32       `json:"userLevel"`
}

```

此函数接受一个名为v的*DokodemoConfig类型的参数，并返回一个名为DokodemoConfig的proto.Message类型和一个名为error的错误类型。

函数内部首先创建一个名为config的新dokodemo.Config实例。然后，函数检查给定的v是否为nil，如果是，则执行config.Address = v.Host.Build()语句，将v的主机地址作为config的地址。接下来，函数将v的端口值作为config的端口，并将v的网络列表作为config的networks。然后，函数将v的超时时间作为config的timeout，并检查v是否使用了redirect标志，如果是，则将redirect附加到config。最后，函数将v的用户级别作为config的用户级别，并将其返回。

函数返回的DokodemoConfig类型是一个结构体类型，其中包含与配置相关的字段。如果函数没有返回任何错误，则它返回一个成功的调用。如果函数返回一个错误，则它将包含错误类型。


```go
func (v *DokodemoConfig) Build() (proto.Message, error) {
	config := new(dokodemo.Config)
	if v.Host != nil {
		config.Address = v.Host.Build()
	}
	config.Port = uint32(v.PortValue)
	config.Networks = v.NetworkList.Build()
	config.Timeout = v.TimeoutValue
	config.FollowRedirect = v.Redirect
	config.UserLevel = v.UserLevel
	return config, nil
}

```

# `infra/conf/dokodemo_test.go`

这段代码是一个测试用例，它的目的是测试 Dokodemo 配置文件是否正确。Dokodemo 是一个用于测试 Web 服务的库，需要一个配置文件来指定连接信息，如地址、端口、网络类型、超时时间等。

在此代码中，首先定义了一个名为 `creator` 的函数，它返回一个自 `DokodemoConfig` 的结构体。然后，定义了一个名为 `TestDokodemoConfig` 的函数，它使用 `creator` 来创建一个新的 `DokodemoConfig` 实例，并传入一个测试数据作为参数。

接下来，定义了一个测试用例列表 `testCases`，该列表包含多个测试用例。每个测试用例包含输入数据、解析器、输出数据等部分。

在 `TestDokodemoConfig` 函数中，使用 `loadJSON` 函数将创建的 `DokodemoConfig` 实例的 JSON 数据加载到内存中，并将其传递给 `DokodemoConfig` 函数的 `Create` 函数。然后，设置选中 `address`、`port`、`network` 和 `timeout` 属性的值，并设置 `followRedirect` 和 `userLevel` 属性为适当的值。

接着，创建一个新的 `DokodemoConfig` 实例，并将其设置为传递给 `DokodemoConfig` 函数的实例，然后调用 `Create` 函数来创建并返回该实例。

最后，在 `TestDokodemoConfig` 函数的 `main` 函数中，创建一系列测试用例，并使用 `runMultiTestCase` 函数来运行这些测试用例。测试用例的 `输入`、`期望输出` 和 `实际输出` 数据作为参数传递给 `runMultiTestCase` 函数。


```go
package conf_test

import (
	"testing"

	"v2ray.com/core/common/net"
	. "v2ray.com/core/infra/conf"
	"v2ray.com/core/proxy/dokodemo"
)

func TestDokodemoConfig(t *testing.T) {
	creator := func() Buildable {
		return new(DokodemoConfig)
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"address": "8.8.8.8",
				"port": 53,
				"network": "tcp",
				"timeout": 10,
				"followRedirect": true,
				"userLevel": 1
			}`,
			Parser: loadJSON(creator),
			Output: &dokodemo.Config{
				Address: &net.IPOrDomain{
					Address: &net.IPOrDomain_Ip{
						Ip: []byte{8, 8, 8, 8},
					},
				},
				Port:           53,
				Networks:       []net.Network{net.Network_TCP},
				Timeout:        10,
				FollowRedirect: true,
				UserLevel:      1,
			},
		},
	})
}

```

# `infra/conf/errors.generated.go`

这段代码定义了一个名为 "conf" 的包，其中包含以下内容：

1. 导入 "v2ray.com/core/common/errors" 包以导入其 errors 类型。

2. 定义一个名为 "errPathObjHolder" 的新结构体，该结构体包含一个空的字符串字段 "errPathObj"，以及一个名为 "err" 的字段，该字段使用 "errors.New" 函数创建一个新错误并添加一个名为 "errPathObj" 的路径偏移对象。

3. 定义一个新的名为 "newError" 的函数，该函数接收多个参数，这些参数可以是任何类型的实际对象。函数返回一个名为 "errors.Error" 的类型的新错误对象，该对象使用传递给它的值初始化，并添加一个名为 "errPathObj" 的路径偏移对象。

4. 该函数使用 "withPathObj" 方法获取路径偏移对象，并将其添加到新错误对象上。

通过调用该函数，可以创建一个新的错误对象，该对象可以使用 "errPathObj" 路径偏移对象中提供的错误信息。


```go
package conf

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `infra/conf/freedom.go`

该代码是一个Go语言编写的JavaNative自由软件包配置类。它定义了一个名为FreedomConfig的结构体类型，该类型包含以下字段：

* `domainStrategy`：表示DNS解析策略，可选的值为`smart`、`low`或`stock`。
* `timeout`：设置DNS解析超时时间的选项，单位为毫秒。
* `redirect`：设置是否在发送请求时进行重定向，可选的值为`true`或`false`。
* `userLevel`：设置用户的等级，可能的值为`0`、`1`或`2`。

这个JavaNative自由软件包配置类允许用户指定DNS解析策略、DNS解析超时时间、是否进行重定向以及用户的等级。这些设置对通过代理的流量进行处理时非常重要。


```go
package conf

import (
	"net"
	"strings"

	"github.com/golang/protobuf/proto"
	v2net "v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/proxy/freedom"
)

type FreedomConfig struct {
	DomainStrategy string  `json:"domainStrategy"`
	Timeout        *uint32 `json:"timeout"`
	Redirect       string  `json:"redirect"`
	UserLevel      uint32  `json:"userLevel"`
}

```

这段代码定义了一个名为`FreedomConfig`的结构体，该结构体实现了`Buildable`接口。`Build`函数是这个结构的`build`方法，用于创建一个`FreedomConfig`实例并返回它。

`config`变量在新建一个名为`freedom.Config`的`FreedomConfig`实例后，将其赋值为`new(freedom.Config)`。接着，将`c.DomainStrategy`的值存储在`config.DomainStrategy`变量中。

接下来，判断字符串`c.DomainStrategy`根据`strings.ToLower(c.DomainStrategy)`的结果，将其存储为`"useip"`，`"use_ip"`，`"useip4"`，`"useipv4"`，`"use_ip_v4"`，`"useip6"`，`"useipv6"`中的一个。然后，根据`c.DomainStrategy`的值，设置`config.DomainStrategy`为相应的值。

接下来，判断是否有一个`c.Timeout`，如果是，则将其赋值给`config.Timeout`。然后，设置`config.UserLevel`为`c.UserLevel`。

最后，处理`c.Redirect`，如果它不为空，则创建一个`net.IP`实例，使用`v2net.ToNetIP`将其转换为IPv4地址，并将其作为`config.DestinationOverride.Server.Address`的值。接着，如果`c.Redirect`中包含一个`host`字段，则使用`v2net.ToNetIP`将其转换为IPv4地址，并将其作为`config.DestinationOverride.Server.Address`的值。

函数返回一个`proto.Message`类型的`config`实例，表示编译配置，以及一个`error`类型的`Build`函数返回的错误。


```go
// Build implements Buildable
func (c *FreedomConfig) Build() (proto.Message, error) {
	config := new(freedom.Config)
	config.DomainStrategy = freedom.Config_AS_IS
	switch strings.ToLower(c.DomainStrategy) {
	case "useip", "use_ip":
		config.DomainStrategy = freedom.Config_USE_IP
	case "useip4", "useipv4", "use_ipv4", "use_ip_v4", "use_ip4":
		config.DomainStrategy = freedom.Config_USE_IP4
	case "useip6", "useipv6", "use_ipv6", "use_ip_v6", "use_ip6":
		config.DomainStrategy = freedom.Config_USE_IP6
	}

	if c.Timeout != nil {
		config.Timeout = *c.Timeout
	}
	config.UserLevel = c.UserLevel
	if len(c.Redirect) > 0 {
		host, portStr, err := net.SplitHostPort(c.Redirect)
		if err != nil {
			return nil, newError("invalid redirect address: ", c.Redirect, ": ", err).Base(err)
		}
		port, err := v2net.PortFromString(portStr)
		if err != nil {
			return nil, newError("invalid redirect port: ", c.Redirect, ": ", err).Base(err)
		}
		config.DestinationOverride = &freedom.DestinationOverride{
			Server: &protocol.ServerEndpoint{
				Port: uint32(port),
			},
		}

		if len(host) > 0 {
			config.DestinationOverride.Server.Address = v2net.NewIPOrDomain(v2net.ParseAddress(host))
		}
	}
	return config, nil
}

```

# `infra/conf/freedom_test.go`

这段代码是一个测试用例，用于测试 FreedomConfig 功能。

首先，定义了一个名为 creater 的函数，它返回一个 FreedomConfig 实例。

接着，定义了一个名为 TestFreedomConfig 的函数，它接收一个测试套件（可能是用来运行测试的框架，如 Go、JUnit、 etc.）和一个或多个测试用例。

在函数内部，使用了一个名为 loadJSON 的函数，该函数从指定的接口加载 JSON 数据并返回一个 *test.傷历器 类型。然后，将该 *test.傷历器 类型传递给一个自定义的 Parser 函数，该函数将 JSON 数据解析为实际的 FreedomConfig 实例。

接着，定义了一个名为 freedom.Config 的结构体，用于表示 FreedomConfig 的各个字段。

然后，定义了一个名为 freedom.DestinationOverride 的结构体，用于表示 DestinationOverride。

接着，定义了一个名为 TestCase 的函数，该函数定义了输入数据、解析器、输出数据等参数，然后使用 loadJSON 和自定义的 Parser 函数获取并解析 JSON 数据，最后，使用 *testing.T 函数发送请求到指定的服务器，以验证是否与预期的结果一致。


```go
package conf_test

import (
	"testing"

	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	. "v2ray.com/core/infra/conf"
	"v2ray.com/core/proxy/freedom"
)

func TestFreedomConfig(t *testing.T) {
	creator := func() Buildable {
		return new(FreedomConfig)
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"domainStrategy": "AsIs",
				"timeout": 10,
				"redirect": "127.0.0.1:3366",
				"userLevel": 1
			}`,
			Parser: loadJSON(creator),
			Output: &freedom.Config{
				DomainStrategy: freedom.Config_AS_IS,
				Timeout:        10,
				DestinationOverride: &freedom.DestinationOverride{
					Server: &protocol.ServerEndpoint{
						Address: &net.IPOrDomain{
							Address: &net.IPOrDomain_Ip{
								Ip: []byte{127, 0, 0, 1},
							},
						},
						Port: 3366,
					},
				},
				UserLevel: 1,
			},
		},
	})
}

```

# `infra/conf/general_test.go`

这段代码定义了一个名为 "conf_test" 的包，其中包含了一些用于测试 "conf" 包的函数。

1. "loadJSON" 函数接受一个 "creator" 函数作为参数，这个函数会被用来创建 "conf" 包中的消息类型。然后，这个函数返回一个 "proto.Message" 类型的实例，以及一个 "error" 类型的值。如果在 "loadJSON" 函数中出现了错误，那么会返回一个 "error" 类型的值。

2. "test" 包中包含了一些测试函数，这些函数使用了 "conf_test" 包中定义的 "loadJSON" 函数。这些测试函数可能会对 "conf" 包中的消息类型进行测试，以验证其正确性。


```go
package conf_test

import (
	"encoding/json"
	"testing"

	"github.com/golang/protobuf/proto"
	"v2ray.com/core/common"
	. "v2ray.com/core/infra/conf"
)

func loadJSON(creator func() Buildable) func(string) (proto.Message, error) {
	return func(s string) (proto.Message, error) {
		instance := creator()
		if err := json.Unmarshal([]byte(s), instance); err != nil {
			return nil, err
		}
		return instance.Build()
	}
}

```

以上代码定义了一个名为 `TestCase` 的结构体，该结构体包含一个输入参数 `Input` 和一个用于解析输入参数的 `Parser` 函数。此外，该结构体还包含一个由 `proto.Message` 类型定义的 `Output` 字段。

`runMultiTestCase` 函数会运行给定的测试用例，并对每个测试用例的 `Parser` 和 `Output` 字段进行比较。如果这两个字段的值不同，函数会在测试失败时输出一条错误消息。否则，函数不会输出任何错误，而是继续执行下一个测试用例。


```go
type TestCase struct {
	Input  string
	Parser func(string) (proto.Message, error)
	Output proto.Message
}

func runMultiTestCase(t *testing.T, testCases []TestCase) {
	for _, testCase := range testCases {
		actual, err := testCase.Parser(testCase.Input)
		common.Must(err)
		if !proto.Equal(actual, testCase.Output) {
			t.Fatalf("Failed in test case:\n%s\nActual:\n%v\nExpected:\n%v", testCase.Input, actual, testCase.Output)
		}
	}
}

```

# `infra/conf/http.go`

这段代码定义了一个名为`HttpAccount`的结构体，用于表示HTTP帐户的配置信息。它包含了两个字段：`Username`和`Password`。这两个字段都是用JSON格式来序列化的，对应的JSON字段名是`user`和`pass`。

具体来说，`HttpAccount`结构体定义了一个`Username`字段，它表示用户名，`Password`字段表示密码。这两个字段都是非空字符串类型的变量。

此外，`HttpAccount`结构体还定义了一个自定义的`encoding/json`类型，称为`HttpAccountEncoder`。这个类型提供了一个将`HttpAccount`结构体转换为JSON字符串的函数，以及一个将JSON字符串转换回`HttpAccount`结构体的函数。这个自定义类型可能是在`v2ray.com/core/account/httpaccount`包中定义的。

最后，代码导入了三个外部依赖：`encoding/json`、`github.com/golang/protobuf/proto`和`v2ray.com/core/common/protocol`。其中，`encoding/json`类型用于将`HttpAccount`结构体转换为JSON字符串，`github.com/golang/protobuf/proto`类型用于定义Protobuf接口，`v2ray.com/core/common/protocol`类型可能是一个与HTTP协议相关的包。


```go
package conf

import (
	"encoding/json"

	"github.com/golang/protobuf/proto"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/http"
)

type HttpAccount struct {
	Username string `json:"user"`
	Password string `json:"pass"`
}

```

这段代码定义了一个名为`HttpServerConfig`的结构体，用于配置HTTP服务器。它包括以下字段：

- `Timeout`：服务器连接超时时间，以秒为单位。
- `Accounts`：配置了HTTP账户的列表。
- `Transparent`：是否允许透明的HTTP连接。
- `UserLevel`：允许的用户级别，以2的整数形式表示。

另外，还定义了一个名为`func (v *HttpAccount) Build() *http.Account`的函数，该函数将`v` struct中的`Username`和`Password`字段与`http.Account`结构体的`Username`和`Password`字段进行比较，返回一个`http.Account`类型的结构体。


```go
func (v *HttpAccount) Build() *http.Account {
	return &http.Account{
		Username: v.Username,
		Password: v.Password,
	}
}

type HttpServerConfig struct {
	Timeout     uint32         `json:"timeout"`
	Accounts    []*HttpAccount `json:"accounts"`
	Transparent bool           `json:"allowTransparent"`
	UserLevel   uint32         `json:"userLevel"`
}

func (c *HttpServerConfig) Build() (proto.Message, error) {
	config := &http.ServerConfig{
		Timeout:          c.Timeout,
		AllowTransparent: c.Transparent,
		UserLevel:        c.UserLevel,
	}

	if len(c.Accounts) > 0 {
		config.Accounts = make(map[string]string)
		for _, account := range c.Accounts {
			config.Accounts[account.Username] = account.Password
		}
	}

	return config, nil
}

```

这段代码定义了两个结构体：HttpRemoteConfig 和 HttpClientConfig。其中，HttpRemoteConfig 是用于配置 HTTP 客户端的远程服务器地址、端口和用户信息；HttpClientConfig 是用于配置 HTTP 客户端的服务器列表。

具体来说，这段代码的实现主要包含以下几个步骤：

1. 定义一个名为 HttpRemoteConfig 的结构体，其中包含 Address、Port 和 Users 三个字段。其中，Address 和 Port 是用于配置 HTTP 客户端与远程服务器之间的通信信息，而 Users 是用于配置 HTTP 客户端的用户信息。

2. 定义一个名为 HttpClientConfig 的结构体，其中包含 Servers 和配置列表两个字段。其中，Servers 是用于配置 HTTP 客户端与远程服务器之间的通信信息的服务器列表，而配置列表是一个包含了多个 HttpRemoteConfig 实例的结构体。

3. 在 HttpClientConfig 的配置列表字段中，遍历所有的服务器配置，为每个服务器配置创建一个 HTTP ServerEndpoint 实例。其中，ServerEndpoint 是 protobuf 中定义的一个结构体，它包含 Address、Port、User 和 Account 四个性质。

4. 将创建好的 HTTP ServerEndpoint 实例添加到配置列表中。

5. 最后，返回配置列表和 nil，表示操作成功。


```go
type HttpRemoteConfig struct {
	Address *Address          `json:"address"`
	Port    uint16            `json:"port"`
	Users   []json.RawMessage `json:"users"`
}
type HttpClientConfig struct {
	Servers []*HttpRemoteConfig `json:"servers"`
}

func (v *HttpClientConfig) Build() (proto.Message, error) {
	config := new(http.ClientConfig)
	config.Server = make([]*protocol.ServerEndpoint, len(v.Servers))
	for idx, serverConfig := range v.Servers {
		server := &protocol.ServerEndpoint{
			Address: serverConfig.Address.Build(),
			Port:    uint32(serverConfig.Port),
		}
		for _, rawUser := range serverConfig.Users {
			user := new(protocol.User)
			if err := json.Unmarshal(rawUser, user); err != nil {
				return nil, newError("failed to parse HTTP user").Base(err).AtError()
			}
			account := new(HttpAccount)
			if err := json.Unmarshal(rawUser, account); err != nil {
				return nil, newError("failed to parse HTTP account").Base(err).AtError()
			}
			user.Account = serial.ToTypedMessage(account.Build())
			server.User = append(server.User, user)
		}
		config.Server[idx] = server
	}
	return config, nil
}

```

# `infra/conf/http_test.go`

这段代码是一个用于测试 http 服务器配置的 Go 语言 package。它包括一个名为 `TestHttpServerConfig` 的函数，该函数使用一个名为 `creator` 的函数创建一个 HTTP 服务器配置实例，然后使用一系列参数来指定服务器的行为。

具体来说，这段代码的作用是测试 HTTP 服务器配置，包括设置超时时间、用户名和密码、允许透明传输数据、用户等级等参数。它通过调用 `loadJSON` 函数来读取 HTTP 服务器配置文件，并将文件内容作为参数传递给 `creator` 函数，以便创建一个 HTTP 服务器配置实例。

接着，代码使用一个名为 `runMultiTestCase` 的函数来运行多个测试用例。每个测试用例都包含一个测试函数，该函数使用不同的参数化来模拟不同的 HTTP 服务器配置。

综合来看，这段代码的主要作用是测试 HTTP 服务器在不同配置下的行为，以及验证 HTTP 服务器是否可以正常工作。


```go
package conf_test

import (
	"testing"

	. "v2ray.com/core/infra/conf"
	"v2ray.com/core/proxy/http"
)

func TestHttpServerConfig(t *testing.T) {
	creator := func() Buildable {
		return new(HttpServerConfig)
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"timeout": 10,
				"accounts": [
					{
						"user": "my-username",
						"pass": "my-password"
					}
				],
				"allowTransparent": true,
				"userLevel": 1
			}`,
			Parser: loadJSON(creator),
			Output: &http.ServerConfig{
				Accounts: map[string]string{
					"my-username": "my-password",
				},
				AllowTransparent: true,
				UserLevel:        1,
				Timeout:          10,
			},
		},
	})
}

```

# `infra/conf/loader.go`

这段代码定义了一个名为“conf”的包，其中包含了一些用于配置到生产环境的配置类。

具体来说，这段代码定义了一个名为“ConfigCreator”的函数类型，它代表一个可以创建和注册配置的函数。然后，定义了一个名为“ConfigCreatorCache”的地图类型，用于存储注册的配置创建者。

接下来，定义了一个名为“RegisterCreator”的函数，该函数接收一个ID和一个配置创建者函数作为参数。如果ID已经在配置创建者缓存中存在，函数将返回一个新的错误，否则尝试在ID上注册新的配置创建者。如果注册成功，函数将返回一个非错误值。

最后，在包的说明中，定义了一个名为“main”的主函数，该函数将从命令行读取一个或多个配置ID，并使用“RegisterCreator”函数将它们注册到配置创建者缓存中。


```go
package conf

import (
	"encoding/json"
	"strings"
)

type ConfigCreator func() interface{}

type ConfigCreatorCache map[string]ConfigCreator

func (v ConfigCreatorCache) RegisterCreator(id string, creator ConfigCreator) error {
	if _, found := v[id]; found {
		return newError(id, " already registered.").AtError()
	}

	v[id] = creator
	return nil
}

```

该代码定义了一个名为 `func` 的函数，其接收一个名为 `v` 的变量和一个名为 `id` 的字符串参数。函数的作用是创建一个与传入 `id` 参数相关的配置，并返回该配置的 `Creator` 和错误信息。

具体来说，函数内部首先检查缓存中是否存在具有指定 `id` 的配置。如果不存在，函数将返回一个 `Error` 和一个 `nil` 值。否则，函数返回创建的 `Creator` 和 `nil`。

该函数可以被理解为一个简单的配置加载器，它从缓存中读取一个配置，并返回其 `Creator` 和错误信息。当缓存中不存在具有指定 `id` 的配置时，函数会抛出一个错误。


```go
func (v ConfigCreatorCache) CreateConfig(id string) (interface{}, error) {
	creator, found := v[id]
	if !found {
		return nil, newError("unknown config id: ", id)
	}
	return creator(), nil
}

type JSONConfigLoader struct {
	cache     ConfigCreatorCache
	idKey     string
	configKey string
}

func NewJSONConfigLoader(cache ConfigCreatorCache, idKey string, configKey string) *JSONConfigLoader {
	return &JSONConfigLoader{
		idKey:     idKey,
		configKey: configKey,
		cache:     cache,
	}
}

```

这段代码定义了两个函数，一个是`func (v *JSONConfigLoader) LoadWithID(raw []byte, id string) (interface{}, error)`，另一个是`func (v *JSONConfigLoader) Load(raw []byte) (interface{}, string, error)`。这两个函数的作用都是从给定的JSON数据中加载相应的配置文件并返回。

具体来说，这两个函数的实现都包含以下步骤：

1. 将传入的`raw`字节数组转换为小写字母。
2. 调用`v.cache.CreateConfig(id)`，使用给定的ID创建一个新的配置文件。如果失败，返回一个`nil`表示的错误。
3. 解码从`raw`字节数组中获取的JSON配置文件并将其存储在`config`字段中。
4. 如果配置文件解析失败，返回一个`nil`表示的错误。
5. 如果`v.idKey`与`raw`中的ID匹配，那么将`config`中的`id`字段设置为`id`。
6. 调用`v.LoadWithID(raw []byte, id string)`函数，使用给定的`raw`字节数组和ID将配置文件加载到`config`字段中。如果加载成功，返回`config`和对应的ID，否则返回一个错误。
7. 如果`v.idKey`与`raw`中的ID不匹配，那么将`config`初始化为一个空的JSON对象。
8. 调用`v.Load(raw []byte)`函数，使用给定的`raw`字节数组加载配置文件。如果加载成功，返回`config`和对应的ID，否则返回一个错误。

在这两个函数中，`v`是一个`JSONConfigLoader`实例，它负责处理所有的配置文件加载和解析操作。


```go
func (v *JSONConfigLoader) LoadWithID(raw []byte, id string) (interface{}, error) {
	id = strings.ToLower(id)
	config, err := v.cache.CreateConfig(id)
	if err != nil {
		return nil, err
	}
	if err := json.Unmarshal(raw, config); err != nil {
		return nil, err
	}
	return config, nil
}

func (v *JSONConfigLoader) Load(raw []byte) (interface{}, string, error) {
	var obj map[string]json.RawMessage
	if err := json.Unmarshal(raw, &obj); err != nil {
		return nil, "", err
	}
	rawID, found := obj[v.idKey]
	if !found {
		return nil, "", newError(v.idKey, " not found in JSON context").AtError()
	}
	var id string
	if err := json.Unmarshal(rawID, &id); err != nil {
		return nil, "", err
	}
	rawConfig := json.RawMessage(raw)
	if len(v.configKey) > 0 {
		configValue, found := obj[v.configKey]
		if found {
			rawConfig = configValue
		} else {
			// Default to empty json object.
			rawConfig = json.RawMessage([]byte("{}"))
		}
	}
	config, err := v.LoadWithID([]byte(rawConfig), id)
	if err != nil {
		return nil, id, err
	}
	return config, id, nil
}

```

# `infra/conf/log.go`

这段代码定义了一个名为 "conf" 的包，其中包含了一个名为 "DefaultLogConfig" 的函数。

函数内部定义了一个名为 "DefaultLogConfig" 的函数，这个函数返回一个指向 "log.Config" 类型的变量，该变量用于设置日志配置。

函数内部通过导入 "strings" 和 "v2ray.com/core/app/log" 和 "v2ray.com/core/common/log" 包，从而可以访问一些用于设置日志配置的函数或类型。

函数内部创建了一个 "log.Config" 类型的变量，该变量包含了一些用于设置日志配置的选项，如 AccessLogType、ErrorLogType 和 ErrorLogLevel 等。这些选项可以设置日志的访问级别、错误级别的日志类型以及错误级别的日志级别。

最后，函数返回指向 "log.Config" 类型的变量，该变量可以用于设置默认日志配置。


```go
package conf

import (
	"strings"

	"v2ray.com/core/app/log"
	clog "v2ray.com/core/common/log"
)

func DefaultLogConfig() *log.Config {
	return &log.Config{
		AccessLogType: log.LogType_None,
		ErrorLogType:  log.LogType_Console,
		ErrorLogLevel: clog.Severity_Warning,
	}
}

```

这段代码定义了一个名为 `LogConfig` 的结构体，其中包含三个字段：`AccessLog`、`ErrorLog` 和 `LogLevel`。这些字段都是 JSON 类型的，并且使用了 `json:"..."` 字段定义。

`v` 指向一个 `LogConfig` 类型的指针，该指针在函数 `Build` 被调用时会被传递给 `log.Config` 类型的参数 `config`。

函数 `Build` 的目的是将 `LogConfig` 结构体中的字段转换为 `log.Config` 类型的结构体，并返回该结构体。

具体实现中，首先检查 `v` 是否为空，如果是，则返回一个空结构体。否则，根据 `v` 的值来设置 `config` 参数中的字段。

对于 `AccessLog` 字段，根据 `v.AccessLog` 的值来设置相应的字段类型和路径。如果 `v.AccessLog` 的值为 "none"，则将 `config.AccessLogType` 设置为 `log.LogType_None`，将 `config.AccessLogPath` 设置为 `v.AccessLog`，将 `config.AccessLogType` 设置为 `log.LogType_File`。

对于 `ErrorLog` 字段，根据 `v.ErrorLog` 的值来设置相应的字段类型和路径。如果 `v.ErrorLog` 的值为 "none"，则将 `config.ErrorLogType` 设置为 `log.LogType_None`，将 `config.ErrorLogPath` 设置为 `v.ErrorLog`，将 `config.ErrorLogType` 设置为 `log.LogType_File`。

对于 `LogLevel` 字段，根据 `v.LogLevel` 的值来设置相应的字段类型和路径。如果 `v.LogLevel` 的值为 "debug"，则将 `config.ErrorLogLevel` 设置为 `clog.Severity_Debug`，将 `config.AccessLogType` 和 `config.AccessLogPath` 的设置留空。如果 `v.LogLevel` 的值为 "info" 或 "error"，则将 `config.ErrorLogLevel` 设置为 `clog.Severity_Info` 或 `clog.Severity_Error`，将 `config.AccessLogType` 和 `config.AccessLogPath` 的设置留空。如果 `v.LogLevel` 的值为 "none"，则将 `config.ErrorLogType` 和 `config.AccessLogType` 的设置留空，将 `config.ErrorLogPath` 设置为 `v.ErrorLog`，将 `config.ErrorLogType` 设置为 `log.LogType_File`。


```go
type LogConfig struct {
	AccessLog string `json:"access"`
	ErrorLog  string `json:"error"`
	LogLevel  string `json:"loglevel"`
}

func (v *LogConfig) Build() *log.Config {
	if v == nil {
		return nil
	}
	config := &log.Config{
		ErrorLogType:  log.LogType_Console,
		AccessLogType: log.LogType_Console,
	}

	if v.AccessLog == "none" {
		config.AccessLogType = log.LogType_None
	} else if len(v.AccessLog) > 0 {
		config.AccessLogPath = v.AccessLog
		config.AccessLogType = log.LogType_File
	}
	if v.ErrorLog == "none" {
		config.ErrorLogType = log.LogType_None
	} else if len(v.ErrorLog) > 0 {
		config.ErrorLogPath = v.ErrorLog
		config.ErrorLogType = log.LogType_File
	}

	level := strings.ToLower(v.LogLevel)
	switch level {
	case "debug":
		config.ErrorLogLevel = clog.Severity_Debug
	case "info":
		config.ErrorLogLevel = clog.Severity_Info
	case "error":
		config.ErrorLogLevel = clog.Severity_Error
	case "none":
		config.ErrorLogType = log.LogType_None
		config.AccessLogType = log.LogType_None
	default:
		config.ErrorLogLevel = clog.Severity_Warning
	}
	return config
}

```

# `infra/conf/mtproto.go`

这段代码定义了一个名为MTProtoAccount的结构体，用于表示MTP协议中的账户信息。

具体来说，该结构体包含一个名为"secret"的字段，其类型为字符串类型，用于存储该账户的秘钥。

此外，该结构体还包含一个名为"IsMaster"的布尔字段，表示该账户是否为管理员账户。

该结构体的定义是为了在v2ray代理中传递MTP协议中的账户信息，v2ray代理需要知道该账户的秘钥才能建立与该代理的连接。




```go
package conf

import (
	"encoding/hex"
	"encoding/json"

	"github.com/golang/protobuf/proto"

	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/mtproto"
)

type MTProtoAccount struct {
	Secret string `json:"secret"`
}

```

这段代码定义了一个名为MTProtoServerConfig的结构体，表示一个MTProto服务器配置。它包含一个名为"users"的数组，数组长度为32。

同时，它还实现了一个名为MTProtoAccount的接口，该接口包含一个名为"Build"的函数。该函数首先检查给定的"Secret"字段的长度是否为32，如果不满足，则会返回一个空的字符串和一个错误消息。如果"Secret"解析成功，则返回一个包含32字节数据的MTproto.Account类型的实例，表示已知的秘密。

总的来说，这段代码定义了一个MTproto服务器配置结构体，其中包含一个名为MTProtoAccount的接口和一个名为MTProtoServerConfig的 struct，该 struct定义了MTproto服务器需要连接的用户列表。


```go
// Build implements Buildable
func (a *MTProtoAccount) Build() (*mtproto.Account, error) {
	if len(a.Secret) != 32 {
		return nil, newError("MTProto secret must have 32 chars")
	}
	secret, err := hex.DecodeString(a.Secret)
	if err != nil {
		return nil, newError("failed to decode secret: ", a.Secret).Base(err)
	}
	return &mtproto.Account{
		Secret: secret,
	}, nil
}

type MTProtoServerConfig struct {
	Users []json.RawMessage `json:"users"`
}

```

这段代码定义了一个名为`func`的函数，它接收一个名为`c`的复用指针变量，并返回一个名为`proto.Message`的接口类型和一个名为`error`的错误类型。

函数的作用是构建一个MTProto服务器配置对象，并检查传入的用户数是否为0。如果为0，函数将返回一个非空字符串`nil`和一个错误对象`newError`，其`err`字段包含错误信息。

如果传入的用户数不为0，函数首先检查`c`是否包含任何用户，如果是，函数创建一个长度为`len(c.Users)`的`protocol.User`切片，并将每个用户对象存储在`config.User`切片中的对应索引位置。然后，函数使用`json.Unmarshal`函数将每个用户对象存储为`MTProtoAccount`类型，并使用`protocol.User`类型构建一个`MTprotoAccount`对象的`Protocol`字段，然后将其存储在`config.User[idx]`切片中的对应索引位置。

最后，函数使用`json.Unmarshal`函数将MTproto服务器配置对象存储为`protocol.Message`类型，如果没有错误，函数返回`protocol.Message`类型和`nil`，否则返回非空字符串`nil`和错误对象`newError`。


```go
func (c *MTProtoServerConfig) Build() (proto.Message, error) {
	config := &mtproto.ServerConfig{}

	if len(c.Users) == 0 {
		return nil, newError("zero MTProto users configured.")
	}
	config.User = make([]*protocol.User, len(c.Users))
	for idx, rawData := range c.Users {
		user := new(protocol.User)
		if err := json.Unmarshal(rawData, user); err != nil {
			return nil, newError("invalid MTProto user").Base(err)
		}
		account := new(MTProtoAccount)
		if err := json.Unmarshal(rawData, account); err != nil {
			return nil, newError("invalid MTProto user").Base(err)
		}
		accountProto, err := account.Build()
		if err != nil {
			return nil, newError("failed to parse MTProto user").Base(err)
		}
		user.Account = serial.ToTypedMessage(accountProto)
		config.User[idx] = user
	}

	return config, nil
}

```

这段代码定义了一个名为MTProtoClientConfig的结构体，表示客户端的配置信息。

在函数(c)中，创建了一个名为MTProtoClientConfig的 pointer变量c，该变量类型为MTProtoClientConfig。

接着，定义了一个名为Build的函数，该函数接收一个空的MTProtoClientConfig结构体参数，并返回一个客户端配置信息和可能的错误。

如果调用Build函数，将会创建一个空的MTProtoClientConfig结构体，将其赋值为空的MTProtoClientConfig结构体，则返回一个空的MTProtoClientConfig结构体。如果设置的结构体为非空结构体，则返回该结构体，否则返回 nil。


```go
type MTProtoClientConfig struct {
}

func (c *MTProtoClientConfig) Build() (proto.Message, error) {
	config := new(mtproto.ClientConfig)
	return config, nil
}

```

# `infra/conf/mtproto_test.go`

该代码是一个测试套件，名为`conf_test`，旨在测试`MTProtoServerConfig`接口的实现。

具体来说，该测试套件包含一个名为`creator`的函数，该函数用于创建一个`MTProtoServerConfig`实例。

此外，该测试套件还包括一个名为`runMultiTestCase`的函数，该函数使用一个包含多个测试用例的切片，每个测试用例都包含一个输入文件和一个用于解析输入文件的解析器。

在测试套件中，`create MTProtoServerConfig`函数的实现 那么就必须慎重考虑


```go
package conf_test

import (
	"testing"

	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	. "v2ray.com/core/infra/conf"
	"v2ray.com/core/proxy/mtproto"
)

func TestMTProtoServerConfig(t *testing.T) {
	creator := func() Buildable {
		return new(MTProtoServerConfig)
	}

	runMultiTestCase(t, []TestCase{
		{
			Input: `{
				"users": [{
					"email": "love@v2ray.com",
					"level": 1,
					"secret": "b0cbcef5a486d9636472ac27f8e11a9d"
				}]
			}`,
			Parser: loadJSON(creator),
			Output: &mtproto.ServerConfig{
				User: []*protocol.User{
					{
						Email: "love@v2ray.com",
						Level: 1,
						Account: serial.ToTypedMessage(&mtproto.Account{
							Secret: []byte{176, 203, 206, 245, 164, 134, 217, 99, 100, 114, 172, 39, 248, 225, 26, 157},
						}),
					},
				},
			},
		},
	})
}

```

# `infra/conf/policy.go`

这段代码定义了一个名为`Policy`的结构体类型，用于配置V2Ray客户端的安全策略。具体来说，该类型包含以下字段：

- `Handshake`字段表示开始 handshake 的时间戳。
- `ConnectionIdle`字段表示空闲时间，也就是在两次 handshake 之间的时间。
- `UplinkOnly`字段表示是否启用上行数据传输。
- `DownlinkOnly`字段表示是否启用下行数据传输。
- `StatsUserUplink`字段表示是否开启统计用户上行的数据传输。
- `StatsUserDownlink`字段表示是否开启统计用户下行的数据传输。
- `BufferSize`字段表示缓冲区大小，用于控制每次发送的数据量。

这个类型被用于定义一个名为`Policy`的结构体变量，它用于配置V2Ray客户端的安全策略。通过使用`json`字段，这个结构体变量可以被 JSON 序列化和反序列化。


```go
package conf

import (
	"v2ray.com/core/app/policy"
)

type Policy struct {
	Handshake         *uint32 `json:"handshake"`
	ConnectionIdle    *uint32 `json:"connIdle"`
	UplinkOnly        *uint32 `json:"uplinkOnly"`
	DownlinkOnly      *uint32 `json:"downlinkOnly"`
	StatsUserUplink   bool    `json:"statsUserUplink"`
	StatsUserDownlink bool    `json:"statsUserDownlink"`
	BufferSize        *int32  `json:"bufferSize"`
}

```

该函数的作用是创建一个名为 `t` 的 `Policy` 对象的 `Policy` 实例，并返回该实例以及错误。

具体来说，函数首先创建一个名为 `config` 的 `policy.Policy_Timeout` 实例，将 `t.Handshake`、`t.ConnectionIdle` 和 `t.UplinkOnly` 和 `t.DownlinkOnly` 的值作为参数传入。然后，函数创建一个名为 `p` 的 `policy.Policy` 实例，将 `config` 作为其 `Timeout` 属性的值，将 `p.Buffer` 作为其 `Stats` 属性的值，将 `t.StatsUserUplink` 和 `t.StatsUserDownlink` 作为 `UserUplink` 和 `UserDownlink` 属性的值作为 `UserUplink` 和 `UserDownlink` 属性的统计数据，最后将 `p` 返回。

如果 `t.Handshake`、`t.ConnectionIdle`、`t.UplinkOnly` 或 `t.DownlinkOnly` 中有一个或多个为 `nil` 的话，函数将返回一个名为 `nil` 的错误。


```go
func (t *Policy) Build() (*policy.Policy, error) {
	config := new(policy.Policy_Timeout)
	if t.Handshake != nil {
		config.Handshake = &policy.Second{Value: *t.Handshake}
	}
	if t.ConnectionIdle != nil {
		config.ConnectionIdle = &policy.Second{Value: *t.ConnectionIdle}
	}
	if t.UplinkOnly != nil {
		config.UplinkOnly = &policy.Second{Value: *t.UplinkOnly}
	}
	if t.DownlinkOnly != nil {
		config.DownlinkOnly = &policy.Second{Value: *t.DownlinkOnly}
	}

	p := &policy.Policy{
		Timeout: config,
		Stats: &policy.Policy_Stats{
			UserUplink:   t.StatsUserUplink,
			UserDownlink: t.StatsUserDownlink,
		},
	}

	if t.BufferSize != nil {
		bs := int32(-1)
		if *t.BufferSize >= 0 {
			bs = (*t.BufferSize) * 1024
		}
		p.Buffer = &policy.Policy_Buffer{
			Connection: bs,
		}
	}

	return p, nil
}

```

此代码定义了一个名为 SystemPolicy 的结构体，它包含进入和出去的数据。这些数据将在 JSON 数据中传输。

这个结构体有一个名为 StatsInboundUplink 的布尔值，表示进入的上下连接的统计数据是否启用。它还包含一个名为 StatsInboundDownlink 的布尔值，表示进入的上下连接的统计数据是否启用。

这个结构体还有一个名为 StatsOutboundUplink 的布尔值，表示出去的上下连接的统计数据是否启用。它还包含一个名为 StatsOutboundDownlink 的布尔值，表示出去的上下连接的统计数据是否启用。

最后，这个结构体有一个名为 Build 的函数，该函数返回一个名为 SystemPolicy 的 policy.SystemPolicy 类型，以及一个表示错误错误对象的错误。如果调用此函数并传入有效的结构体，则返回一个政策，否则返回一个空结构体。


```go
type SystemPolicy struct {
	StatsInboundUplink    bool `json:"statsInboundUplink"`
	StatsInboundDownlink  bool `json:"statsInboundDownlink"`
	StatsOutboundUplink   bool `json:"statsOutboundUplink"`
	StatsOutboundDownlink bool `json:"statsOutboundDownlink"`
}

func (p *SystemPolicy) Build() (*policy.SystemPolicy, error) {
	return &policy.SystemPolicy{
		Stats: &policy.SystemPolicy_Stats{
			InboundUplink:    p.StatsInboundUplink,
			InboundDownlink:  p.StatsInboundDownlink,
			OutboundUplink:   p.StatsOutboundUplink,
			OutboundDownlink: p.StatsOutboundDownlink,
		},
	}, nil
}

```

这段代码定义了一个名为PolicyConfig的结构体类型，该类型包含了一个名为Levels的map类型，以及一个名为System的指针类型和一个名为Policy的map类型。

函数Build()函数接收一个PolicyConfig类型的变量c，并返回一个指向policy.Config类型的变量config和一个指向SystemPolicy类型的变量sc，或者错误。

函数内部首先定义了一个名为levels的map类型，该map类型的键是uint32类型，值为一个指向Policy类型的指针类型。然后，使用for循环遍历c.Levels类型的键值对，对于每个键值对，如果该键的值为nil，则执行循环内部的policy.Build()函数，并将返回值存储在对应的levels的键中。

接着，定义了一个名为config的policy.Config类型变量，该类型的键是levels，值为levels中所有的Policy类型构建的policy.Config类型。然后，如果c.System是nil，函数将执行SystemPolicy.Build()函数并将其存储在config中。

最后，函数返回config和sc，或者错误。


```go
type PolicyConfig struct {
	Levels map[uint32]*Policy `json:"levels"`
	System *SystemPolicy      `json:"system"`
}

func (c *PolicyConfig) Build() (*policy.Config, error) {
	levels := make(map[uint32]*policy.Policy)
	for l, p := range c.Levels {
		if p != nil {
			pp, err := p.Build()
			if err != nil {
				return nil, err
			}
			levels[l] = pp
		}
	}
	config := &policy.Config{
		Level: levels,
	}

	if c.System != nil {
		sc, err := c.System.Build()
		if err != nil {
			return nil, err
		}
		config.System = sc
	}

	return config, nil
}

```

# `infra/conf/policy_test.go`

这段代码是一个名为 "conf_test" 的包，其中包含一个名为 "TestBufferSize" 的测试函数。

这个测试函数使用了 v2ray.com/core/infra/conf 包中提供的一个名为 Policy 的结构体，这个结构体有一个名为 "BufferSize" 的字段，它代表一个缓冲区大小。

在测试函数中，首先定义了一系列测试用例，每个测试用例包含一个输入参数和一个预期输出参数。然后，对于每个测试用例，创建一个名为 pConf 的 Policy 实例，并且将 pConf 的 "BufferSize" 字段设置为测试用例中给出的输入参数。最后，使用 pConf 的 "Build" 方法创建一个名为 p 的 Policy 实例，并且检查 p.Buffer.Connection 是否等于测试用例中给出的预期输出参数。

如果 p.Buffer.Connection 不等于预期输出参数，那么函数会在测试台上输出错误信息，其中包括测试用例的名称、输入参数和预期输出参数。


```go
package conf_test

import (
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/infra/conf"
)

func TestBufferSize(t *testing.T) {
	cases := []struct {
		Input  int32
		Output int32
	}{
		{
			Input:  0,
			Output: 0,
		},
		{
			Input:  -1,
			Output: -1,
		},
		{
			Input:  1,
			Output: 1024,
		},
	}

	for _, c := range cases {
		bs := int32(c.Input)
		pConf := Policy{
			BufferSize: &bs,
		}
		p, err := pConf.Build()
		common.Must(err)
		if p.Buffer.Connection != c.Output {
			t.Error("expected buffer size ", c.Output, " but got ", p.Buffer.Connection)
		}
	}
}

```

# `infra/conf/reverse.go`

这段代码定义了一个名为 `BridgeConfig` 的结构体，该结构体包含一个字段 `Tag` 和一个字段 `Domain`，同时提供了 `json` 方法来将该结构体序列化为 JSON 字节切片。

接着，该代码定义了一个名为 `BridgeConfig` 的函数，该函数接收一个名为 `BridgeConfig` 的结构体作为参数，然后使用该结构体中提供的字段来创建一个 `reverse.BridgeConfig` 类型的值，并返回该值。如果过程中出现错误，该函数返回一个错误对象。

该函数的实现比较简单，主要作用是帮助开发者快速创建一个反向代理服务，该服务接收一个带有标签和域名的请求，并返回一个包含该请求信息的 `reverse.BridgeConfig` 类型的值。


```go
package conf

import (
	"github.com/golang/protobuf/proto"
	"v2ray.com/core/app/reverse"
)

type BridgeConfig struct {
	Tag    string `json:"tag"`
	Domain string `json:"domain"`
}

func (c *BridgeConfig) Build() (*reverse.BridgeConfig, error) {
	return &reverse.BridgeConfig{
		Tag:    c.Tag,
		Domain: c.Domain,
	}, nil
}

```

该代码定义了一个名为"PortalConfig"的结构体类型，它包含一个名为"tag"的整类型字段和一个名为"domain"的整类型字段。

接下来，定义了一个名为"reverse.PortalConfig"的结构体类型，它包含一个名为"tag"的整类型字段和一个名为"domain"的整类型字段。

接着，定义了一个名为"PortalConfig"的函数，它接受一个名为"c"的整类型指针变量，并返回一个指向"reverse.PortalConfig"类型的结构体变量和一个名为"nil"的整类型变量。函数的实现包括对传入的"c"结构体变量中的"tag"和"domain"字段进行拷贝，并创建一个名为"reverse"的整类型变量和一个名为"err"的整类型变量。接着，调用一个名为"str.conv"的函数，将"reverse"的整类型变量转换为字符串类型，并将结果存储到"err"中，最后返回"reverse"的整类型变量。

接着，定义了一个名为"BridgeConfig"的结构体类型，但是没有定义它的字段和函数。

接着，定义了一个名为"PortalConfig"的函数，它接受一个名为"c"的整类型指针变量，并返回一个指向"PortalConfig"类型的结构体变量。函数的实现包括对传入的"c"结构体变量中的"tag"和"domain"字段进行拷贝，并创建一个名为"portal"的整类型变量和一个名为"err"的整类型变量。接着，调用一个名为"str.conv"的函数，将"portal"的整类型变量转换为字符串类型，并将结果存储到"err"中，最后返回"portal"的整类型变量。

接着，定义了一个名为"ReverseConfig"的结构体类型，它包含一个名为"bridges"的整类型字段和一个名为"portals"的整类型字段。

接着，定义了一个名为"BridgeConfig"的函数，它接受一个名为"c"的整类型指针变量，并返回一个指向"BridgeConfig"类型的结构体变量。函数的实现包括对传入的"c"结构体变量中的"bridges"字段进行拷贝，并创建一个名为"bridge"的整类型变量，最后返回"bridge"的整类型变量。

接着，定义了一个名为"PortalConfig"的函数，它接受一个名为"c"的整类型指针变量，并返回一个指向"PortalConfig"类型的结构体变量。函数的实现包括对传入的"c"结构体变量中的"domain"字段进行拷贝，并创建一个名为"portal"的整类型变量，最后返回"portal"的整类型变量。

接着，定义了一个名为"ReverseConfig"的函数，它接受一个名为"c"的整类型指针变量，并返回一个指向"ReverseConfig"类型的结构体变量。函数的实现包括对传入的"c"结构体变量中的"bridges"字段进行拷贝，并创建一个名为"reverse_bridges"的整类型变量，将接收到的"portal"的整类型变量作为参数，然后将两个整类型变量连接，最后返回"reverse_bridges"的整类型变量。


```go
type PortalConfig struct {
	Tag    string `json:"tag"`
	Domain string `json:"domain"`
}

func (c *PortalConfig) Build() (*reverse.PortalConfig, error) {
	return &reverse.PortalConfig{
		Tag:    c.Tag,
		Domain: c.Domain,
	}, nil
}

type ReverseConfig struct {
	Bridges []BridgeConfig `json:"bridges"`
	Portals []PortalConfig `json:"portals"`
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `c` 的整数类型的参数，并返回一个名为 `ReverseConfig` 的类型，以及一个名为 `error` 的整数类型的变量。

函数的作用是：将传入的 `c` 值中的所有桥接配置项通过桥接配置的 `Build` 函数调用，并将得到的配置项添加到 `config.BridgeConfig` 和 `config.PortalConfig` 字段中，最后返回 `config` 和 `nil` 其中 `config` 是指向 `reverse.Config` 类型对象的指针，`nil` 是表示没有错误发生的 `error` 类型变量。


```go
func (c *ReverseConfig) Build() (proto.Message, error) {
	config := &reverse.Config{}
	for _, bconfig := range c.Bridges {
		b, err := bconfig.Build()
		if err != nil {
			return nil, err
		}
		config.BridgeConfig = append(config.BridgeConfig, b)
	}

	for _, pconfig := range c.Portals {
		p, err := pconfig.Build()
		if err != nil {
			return nil, err
		}
		config.PortalConfig = append(config.PortalConfig, p)
	}

	return config, nil
}

```