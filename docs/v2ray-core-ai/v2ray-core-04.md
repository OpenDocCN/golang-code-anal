# v2ray-core源码解析 4

# `app/dns/dnscommon_test.go`

该代码是一个 Go 语言编写的字符串，用于定义 DNS 服务器端的配置选项。它包含以下几行：

1. `// +build !confonly`：这是一个 build 选项，告诉编译器不要在 build 过程中输出这段代码。这个选项通常在开发过程中使用，以便保持代码的简洁。

2. `package dns`：定义了该代码所在的包。

3. `import (`：导入了一些函数或类型。

4. `"math/rand"`, `"testing"`, `"time"`：导入了一些数学函数、测试函数和时间函数。

5. `import dnsmessage"`, `"github.com/google/go-cmp/cmp"`, `"github.com/miekg/dns"`, `"v2ray.com/core/common/net"`：导入了一些来自网络库的函数或类型。

6. `package dns`：定义了该代码所在的包。

7. `func TestMain(t *testing.T) {`：定义了一个函数，名为 `TestMain`，用于在测试过程中运行该代码。

8. `func TestDNS(t *testing.T) {`：定义了一个函数，名为 `TestDNS`，用于测试 DNS 服务器端的配置选项。

9. `func TestDNSMessage(t *testing.T) {`：定义了一个函数，名为 `TestDNSMessage`，用于测试 DNS 服务器端的 `dnsmessage` 类型。

10. `"test/dns-server"`：该函数可能用于编译并运行测试服务器。

11. `"build.sh"`：该函数可能用于编译并运行命令行工具。

12. `"build/main.go"`：该函数可能用于编译并运行主应用程序。

13. `"go.mod"`：该函数可能用于在 Go 应用程序中加载必要的模块。

Go 语言的编译器会根据这个选项来决定是否在 build 过程中输出这段代码。如果设置了 `+build` 选项，则编译器会在构建过程中输出这段代码。如果设置了 `+confonly` 选项，则编译器不会输出这段代码。


```go
// +build !confonly

package dns

import (
	"math/rand"
	"testing"
	"time"

	"github.com/google/go-cmp/cmp"
	"github.com/miekg/dns"
	"golang.org/x/net/dns/dnsmessage"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
)

```

This looks like a testing framework for IPRecord. It defines a struct IPRecord struct {
	name        string
	want       *IPRecord
	wantErr  bool
	,wantMsg string
}

tests := []struct {
	name    string
	want    *IPRecord
	wantErr bool
	,wantMsg string
}{
	{"empty",
		&IPRecord{0, []net.Address(nil), time.Time{}, dnsmessage.RCodeSuccess},
		false,
	},
	{"error",
		nil,
		true,
	},
	{"a record",
		&IPRecord{1, []net.Address{net.ParseAddress("8.8.8.8"), net.ParseAddress("8.8.4.4")},
			time.Time{}, dnsmessage.RCodeSuccess},
		false,
	},
	{"aaaa record",
		&IPRecord{2, []net.Address{net.ParseAddress("2001::123:8888"), net.ParseAddress("2001::123:8844")},
			time.Time{}, dnsmessage.RCodeSuccess},
		false,
	},
}

for i, tt := range tests {
	t.Run(tt.name, func(t *testing.T) {
		got, err := parseResponse(p[i])
		if (err != nil) != tt.wantErr {
			t.Errorf("handleResponse() error = %v, wantErr %v", err, tt.wantErr)
			return
		}

		if got != nil {
			// reset the time
			got.Expire = time.Time{}
			// reset the want
			tt.want = nil
		}
		if cmp.Diff(got, tt.want) != "" {
			t.Errorf(cmp.Diff(got, tt.want))
		}
	})
}


```go
func Test_parseResponse(t *testing.T) {
	var p [][]byte

	ans := new(dns.Msg)
	ans.Id = 0
	p = append(p, common.Must2(ans.Pack()).([]byte))

	p = append(p, []byte{})

	ans = new(dns.Msg)
	ans.Id = 1
	ans.Answer = append(ans.Answer,
		common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN CNAME fake.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN A 8.8.8.8")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN A 8.8.4.4")).(dns.RR),
	)
	p = append(p, common.Must2(ans.Pack()).([]byte))

	ans = new(dns.Msg)
	ans.Id = 2
	ans.Answer = append(ans.Answer,
		common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN CNAME fake.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN CNAME m.test.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN CNAME test.google.com")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN AAAA 2001::123:8888")).(dns.RR),
		common.Must2(dns.NewRR("google.com. IN AAAA 2001::123:8844")).(dns.RR),
	)
	p = append(p, common.Must2(ans.Pack()).([]byte))

	tests := []struct {
		name    string
		want    *IPRecord
		wantErr bool
	}{
		{"empty",
			&IPRecord{0, []net.Address(nil), time.Time{}, dnsmessage.RCodeSuccess},
			false,
		},
		{"error",
			nil,
			true,
		},
		{"a record",
			&IPRecord{1, []net.Address{net.ParseAddress("8.8.8.8"), net.ParseAddress("8.8.4.4")},
				time.Time{}, dnsmessage.RCodeSuccess},
			false,
		},
		{"aaaa record",
			&IPRecord{2, []net.Address{net.ParseAddress("2001::123:8888"), net.ParseAddress("2001::123:8844")}, time.Time{}, dnsmessage.RCodeSuccess},
			false,
		},
	}
	for i, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := parseResponse(p[i])
			if (err != nil) != tt.wantErr {
				t.Errorf("handleResponse() error = %v, wantErr %v", err, tt.wantErr)
				return
			}

			if got != nil {
				// reset the time
				got.Expire = time.Time{}
			}
			if cmp.Diff(got, tt.want) != "" {
				t.Errorf(cmp.Diff(got, tt.want))
				// t.Errorf("handleResponse() = %#v, want %#v", got, tt.want)
			}
		})
	}
}

```

这段代码定义了一个名为 `Test_buildReqMsgs` 的函数风验函数 `func`。该函数接受一个名为 `t` 的 testing.T 类型的参数，会执行一系列测试用例。

在每个测试用例中，函数会调用一个名为 `buildReqMsgs` 的函数，该函数接收一个字符串参数 `domain`、一个 IP 选项参数 `option` 和一个 `dnsmessage.Resource` 类型的参数 `reqOpts`。函数的实现如下：


func buildReqMsgs(domain string, option IPOption, stubID func()) *dnsmessage.Resource {
	request := &dnsmessage.Request{
		Message: &dnsmessage.Message{
			Type:   "REQUEST",
			Fields: map[string] *dnsmessage.Field{
				"domain": {
					"Type":   "IP 地址",
					"Value": *ipv4only.Value,
				},
				"option": {
					"Type":   "IP 选项",
					"Value": option.Value,
				},
				},
			},
		},
		Timestamp: *time.Now(),
		StubID:     stubID,
	}

	res, err := stubID()
	if err != nil {
		t.Fatalf("failed to create stub: %v", err)
	}

	request = &res.Call<dnsmessage.Resource>("test.com", request)
	if request == nil || !res.Status.Code == dnsmessage.ResponseStatusOK {
		t.Fatalf("buildReqMsgs() returned %v, want %v", request, dnsmessage.ResponseStatusOK)
	}

	return request
}


函数首先定义了一个名为 `stubID` 的函数，该函数接受一个 `uint16` 类型的参数，然后使用 `rand` 生成一个随机整数，并将其作为 `stubID` 的返回值。

接下来定义了一个名为 `args` 的结构体，该结构体代表了所有的测试用例需要传递给 `buildReqMsgs` 的参数。其中，`domain` 是字符串参数，`option` 是 IP 选项参数，`reqOpts` 是 `dnsmessage.Resource` 类型的参数。

然后定义了一个名为 `tests` 的切片，该切片代表了所有的测试用例，每个测试用例都有一个测试名称和相应的测试参数。在 `for` 循环中，使用嵌套的 `t.Run` 函数来运行每个测试用例。在每次运行测试用例时，会将 `domain`、`option` 和 `reqOpts` 参数传递给 `buildReqMsgs` 函数，并接收其返回的结果。然后比较函数的返回结果与期望的结果是否相等，如果不相等，则会输出错误信息。


```go
func Test_buildReqMsgs(t *testing.T) {

	stubID := func() uint16 {
		return uint16(rand.Uint32())
	}
	type args struct {
		domain  string
		option  IPOption
		reqOpts *dnsmessage.Resource
	}
	tests := []struct {
		name string
		args args
		want int
	}{
		{"dual stack", args{"test.com", IPOption{true, true}, nil}, 2},
		{"ipv4 only", args{"test.com", IPOption{true, false}, nil}, 1},
		{"ipv6 only", args{"test.com", IPOption{false, true}, nil}, 1},
		{"none/error", args{"test.com", IPOption{false, false}, nil}, 0},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := buildReqMsgs(tt.args.domain, tt.args.option, stubID, tt.args.reqOpts); !(len(got) == tt.want) {
				t.Errorf("buildReqMsgs() = %v, want %v", got, tt.want)
			}
		})
	}
}

```

该代码定义了一个名为 "Test\_genEDNS0Options" 的函数，它接受一个 testing.T 类型的参数。在这个函数中，定义了一个名为 "args" 的结构体类型，它包含一个 "clientIP" 字段和一个 "name" 字段。

接下来，定义了一个名为 "tests" 的结构体类型，它包含一个字符串类型的字段 "name" 和一个 "args" 类型的字段，这个字段的值是一个嵌套的结构体类型，包含了 "clientIP" 和 "name" 字段。

接着，定义了一个名为 "tt" 的循环结构体类型，它包含一个字符串类型的字段 "name" 和一个 "args" 类型的字段，这个字段的值在每次循环中都是不同的。

在 "for" 循环中，将 "args" 和 "tt" 中的值绑定到 "t" 变量上，然后调用 "genEDNS0Options" 函数，并且使用 "args" 字段的 "clientIP" 字段作为参数。

接着，在函数体中判断 "genEDNS0Options" 函数的返回值是否为 nil，如果是，就使用 "t.Errorf" 函数打印错误信息，否则就打印正确信息。


```go
func Test_genEDNS0Options(t *testing.T) {
	type args struct {
		clientIP net.IP
	}
	tests := []struct {
		name string
		args args
		want *dnsmessage.Resource
	}{
		// TODO: Add test cases.
		{"ipv4", args{net.ParseIP("4.3.2.1")}, nil},
		{"ipv6", args{net.ParseIP("2001::4321")}, nil},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := genEDNS0Options(tt.args.clientIP); got == nil {
				t.Errorf("genEDNS0Options() = %v, want %v", got, tt.want)
			}
		})
	}
}

```

该代码定义了一个名为 `TestFqdn` 的函数，它接受一个名为 `t` 的测试参量。内部定义了一个名为 `args` 的结构体，它包含一个域（domain）和一个将 `domain` 字面值转换为字符串的函数 `ToFqdn`。

接着定义了一个字符串数组 `tests`，其中的每个元素都是一个包含两个名字的 `struct`，分别对应 `"with fqdn"` 和 `"without fqdn"` 这两个测试用例。

最后，通过一个嵌套的循环 `for` 遍历 `tests`，对每个测试用例调用 `Fqdn` 函数，比较函数返回值与期望值是否一致，如果不一致，则输出错误信息。


```go
func TestFqdn(t *testing.T) {
	type args struct {
		domain string
	}
	tests := []struct {
		name string
		args args
		want string
	}{
		{"with fqdn", args{"www.v2ray.com."}, "www.v2ray.com."},
		{"without fqdn", args{"www.v2ray.com"}, "www.v2ray.com."},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := Fqdn(tt.args.domain); got != tt.want {
				t.Errorf("Fqdn() = %v, want %v", got, tt.want)
			}
		})
	}
}

```

# `app/dns/dohdns.go`

这段代码是一个 Go 语言编写的字符串，它通过构建工具编译，并输出一个名为 "dns" 的包。这个包通过引入一些常用的库，比如 "net/http" 和 "v2ray.com/core/features/dns"，来实现 DNS 解析的功能。

这个包的核心实现是通过一个 DNS 客户端来发起 DNS 查询，获取目标域名对应的 IP 地址。代码中定义了一个名为 "build" 的函数，它的作用是编译这个包。而 "!confonly" 参数则是告诉编译器只输出一个不包含额外输出的包。

具体来说，这个包的实现包括以下步骤：

1. 定义了一个名为 "dnsquery" 的函数，它是 DNS 客户端的实现。这个函数会接收一个目标域名和解析区域，使用 "net/http" 库发起 HTTP GET 请求，获取目标域名对应的 IP 地址，然后返回这个 IP 地址。

2. 定义了一个名为 "dnslookup" 的函数，它是 "dnsquery" 的扩展，用于在本地 DNS 库中查找目标域名对应的 IP 地址。这个函数会尝试使用不同的 DNS 库，包括 OpenDNS 和 Google DNS。

3. 在 "main" 函数中，定义了一个名为 "build" 的函数，它的作用是编译 "dns" 包。然后就直接启动了这个函数，相当于直接运行 "dnsbuild" 命令，这个命令会把所有以 "dns" 为后缀的文件编译成 package.json 文件。

4. 在 "build" 函数中，引入了一些需要使用的库，包括 "net/http"、"v2ray.com/core/features/dns" 和 "v2ray.com/core/features/routing"。然后就是定义了一个名为 "dnsclient" 的变量，它是 "dnsquery" 和 "dnslookup" 函数的实现，用于发起 DNS 查询和解析。

5. 在 "dnsclient" 函数中，定义了一个名为 "buildconfonly" 的函数，它的作用是告诉编译器只输出 "dns" 包，而不输出它的子模块。

6. 在 "buildconfonly" 函数之后，定义了一个 "main" 函数，它的作用是编译 "dns" 包。


```go
// +build !confonly

package dns

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"net/url"
	"sync"
	"sync/atomic"
	"time"

	"golang.org/x/net/dns/dnsmessage"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/dns"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal/pubsub"
	"v2ray.com/core/common/task"
	dns_feature "v2ray.com/core/features/dns"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport/internet"
)

```

这段代码定义了一个名为`DoHNameServer`的`struct`类型，用于实现通过HTTPS实现DNS over HTTP。与传统的DOH实现基于UDP不同，它基于TCP套接字，兼容于基于UDP的DOH(RFC8484)和基于TCP套接字的DOH(RFC1035)。

具体来说，该代码实现了以下功能：

1. 定义了一个`DoHNameServer`结构体，它包含以下字段：
	- `ips`：一个`map`类型，用于存储IP地址映射到DNS服务器，每个键都是一个IP地址，对应一个值为`record`类型，包含DNS解析结果。
	- `pub`：一个`pubsub.Service`类型，用于发布DNS解析结果。
	- `cleanup`：一个`task.Periodic`类型，用于定期检查和清理过期的DNS解析结果。
	- `reqID`：一个`uint32`类型，用于标识请求消息的唯一ID。
	- `clientIP`：一个`net.IP`类型，用于存储客户端的IP地址。
	- `httpClient`：一个`http.Client`类型，用于发起HTTP请求。
	- `dohURL`：一个`string`类型，用于存储DOH服务器的URL。
	- `name`：一个`string`类型，用于存储服务器主机的名称。

2. 实现了` cleanupPeriodically`方法，用于定期检查和清理过期的DNS解析结果。该方法每20秒检查一次，如果超过20秒未检查到有效的解析结果，就将对应的键值对从`ips`中移除。

3. 实现了`handleRequest`方法，用于处理接收到的请求消息并返回解析结果。该方法首先检查请求消息中是否包含有效的时间戳，如果是，则解析DNS记录并返回。

4. 实现了`run`函数，用于初始化该`DoHNameServer`实例并启动清理周期。

由于该代码实现在了DOH over HTTP上，因此它可以用来实现DNS over HTTP服务器，而不需要支持DOH协议的客户端软件。


```go
// DoHNameServer implemented DNS over HTTPS (RFC8484) Wire Format,
// which is compatible with traditional dns over udp(RFC1035),
// thus most of the DOH implementation is copied from udpns.go
type DoHNameServer struct {
	sync.RWMutex
	ips        map[string]record
	pub        *pubsub.Service
	cleanup    *task.Periodic
	reqID      uint32
	clientIP   net.IP
	httpClient *http.Client
	dohURL     string
	name       string
}

```

This function creates a new remote DOH client for the given DNS URL. It uses the "baseDOHNameServer" function to create a connection to the DNS server and sets up a connection pool for the HTTP/HTTPS requests. The connection pool is configured with a maximum number of idle connections and an idle timeout.

The function also sets up an HTTP/HTTPS transport for the connection and sets the forceAttemptHTTP2 flag to true, which ensures that HTTPS attempts are made whenever a connection is lost. The `dialContext` function is used to create a new connection to the DNS server with the given network and address.

Finally, the `httpClient` property of the `s` object is set to the `dispatchedClient` object, which is an instance of the `http.Client` struct with the added configuration settings.

This function can be useful for setting up a remote DOH client for the purpose of making DNS requests to a DNS server, such as when using the "DOHL" or "DOH" protocol in the "v2ray" application.


```go
// NewDoHNameServer creates DOH client object for remote resolving
func NewDoHNameServer(url *url.URL, dispatcher routing.Dispatcher, clientIP net.IP) (*DoHNameServer, error) {

	newError("DNS: created Remote DOH client for ", url.String()).AtInfo().WriteToLog()
	s := baseDOHNameServer(url, "DOH", clientIP)

	// Dispatched connection will be closed (interrupted) after each request
	// This makes DOH inefficient without a keep-alived connection
	// See: core/app/proxyman/outbound/handler.go:113
	// Using mux (https request wrapped in a stream layer) improves the situation.
	// Recommend to use NewDoHLocalNameServer (DOHL:) if v2ray instance is running on
	//  a normal network eg. the server side of v2ray
	tr := &http.Transport{
		MaxIdleConns:        30,
		IdleConnTimeout:     90 * time.Second,
		TLSHandshakeTimeout: 30 * time.Second,
		ForceAttemptHTTP2:   true,
		DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
			dest, err := net.ParseDestination(network + ":" + addr)
			if err != nil {
				return nil, err
			}

			link, err := dispatcher.Dispatch(ctx, dest)
			if err != nil {
				return nil, err
			}
			return net.NewConnection(
				net.ConnectionInputMulti(link.Writer),
				net.ConnectionOutputMulti(link.Reader),
			), nil
		},
	}

	dispatchedClient := &http.Client{
		Transport: tr,
		Timeout:   60 * time.Second,
	}

	s.httpClient = dispatchedClient
	return s, nil
}

```

这段代码定义了一个名为`NewDoHLocalNameServer`的函数，它接收一个`url.URL`类型的参数作为远程服务器的URL，以及一个`net.IP`类型的参数作为客户端的IP地址。它返回一个`DoHNameServer`类型的指针，代表一个已经创建好的本地DOH服务器实例。

函数首先将远程服务器的URL设置为`https`，然后使用`baseDOHNameServer`函数创建一个DOH客户端对象。使用`互联网.DialSystem`函数尝试连接到服务器，如果连接成功，则返回客户端的TCP连接。如果连接失败，函数返回一个`nil`表示错误。

接下来，函数创建一个HTTP传输，设置一些超时时间。使用`http.Client`类型的`transport`参数设置HTTP传输的超时时间，`timeout`参数设置HTTP传输的超时时间(默认为3秒),`forceAttemptHTTP2`参数设置为真，使HTTP/2协议支持尝试三次握手。

最后，函数创建一个`DoHNameServer`类型的实例，并将之前创建的TCP连接和HTTP传输作为该实例的依赖项。

函数的作用是创建一个本地DOH服务器实例，以便在客户端连接到服务器时进行DOH客户端的配置和连接。


```go
// NewDoHLocalNameServer creates DOH client object for local resolving
func NewDoHLocalNameServer(url *url.URL, clientIP net.IP) *DoHNameServer {
	url.Scheme = "https"
	s := baseDOHNameServer(url, "DOHL", clientIP)
	tr := &http.Transport{
		IdleConnTimeout:   90 * time.Second,
		ForceAttemptHTTP2: true,
		DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
			dest, err := net.ParseDestination(network + ":" + addr)
			if err != nil {
				return nil, err
			}
			conn, err := internet.DialSystem(ctx, dest, nil)
			if err != nil {
				return nil, err
			}
			return conn, nil
		},
	}
	s.httpClient = &http.Client{
		Timeout:   time.Second * 180,
		Transport: tr,
	}
	newError("DNS: created Local DOH client for ", url.String()).AtInfo().WriteToLog()
	return s
}

```

该函数的目的是创建并返回一个名为 "baseDOHNameServer" 的函数，该函数接受一个 URL 类型参数 "url"，一个字符串参数 "prefix"，和一个 IP 类型参数 "clientIP"。

函数内部首先创建一个名为 "s" 的 DoHNameServer 类型的变量，然后设置其 IP 类型参数为传入的 "clientIP"，并设置其名称设置为 "baseDOHNameServer" 和 URL 的主机名。

接着，设置其订阅服务为 pubsub.NewService，并设置其名字设置为 "prefix" 和 URL 的主机名。

最后，设置其 dohURL 参数为传入的 "url"，并创建一个 Periodic 类型的清理任务，其周期为 1 分钟，执行操作为 "s.cleanup"。

函数返回 s 实例，表示已经创建好的 DoHNameServer 类型的变量。


```go
func baseDOHNameServer(url *url.URL, prefix string, clientIP net.IP) *DoHNameServer {

	s := &DoHNameServer{
		ips:      make(map[string]record),
		clientIP: clientIP,
		pub:      pubsub.NewService(),
		name:     prefix + "//" + url.Host,
		dohURL:   url.String(),
	}
	s.cleanup = &task.Periodic{
		Interval: time.Minute,
		Execute:  s.Cleanup,
	}

	return s
}

```

该代码定义了一个名为DoHNameServer的 struct，该 struct包含一个名为Name的函数，该函数返回客户端名称。

此外，还定义了一个名为Cleanup的函数，该函数从缓存中清除过期的项目，并释放锁。

具体来说，该函数首先获取当前时间并作为锁的一个参与者，以确保在函数中对缓存进行更改时只有一个线程执行。

然后，遍历缓存中的所有项目，检查其 A 字段是否有效，并检查其是否存在。如果是，则检查其是否过期的，如果是，则将其失效并将其从缓存中删除。如果项目既不是 A 字段也不是AAAA字段，则当清理缓存时创建一个新的IPA记录并将其添加到缓存中。

最后，如果当前缓存中仍有项目，则返回 nil，否则，创建一个空字符串作为客户端名称并返回该客户端名称。


```go
// Name returns client name
func (s *DoHNameServer) Name() string {
	return s.name
}

// Cleanup clears expired items from cache
func (s *DoHNameServer) Cleanup() error {
	now := time.Now()
	s.Lock()
	defer s.Unlock()

	if len(s.ips) == 0 {
		return newError("nothing to do. stopping...")
	}

	for domain, record := range s.ips {
		if record.A != nil && record.A.Expire.Before(now) {
			record.A = nil
		}
		if record.AAAA != nil && record.AAAA.Expire.Before(now) {
			record.AAAA = nil
		}

		if record.A == nil && record.AAAA == nil {
			newError(s.name, " cleanup ", domain).AtDebug().WriteToLog()
			delete(s.ips, domain)
		} else {
			s.ips[domain] = record
		}
	}

	if len(s.ips) == 0 {
		s.ips = make(map[string]record)
	}

	return nil
}

```

这段代码是一个名为 `func` 的函数，接收两个参数：一个指向 `DoHNameServer` 结构的指针 `s` 和一个包含 `IPRecord` 结构体的参数 `ipRec`。

函数的主要作用是更新一个名为 `ipRec` 的 `IPRecord` 结构体中的字段，其中可能包含一个 IPv6 地址。当 `ipRec` 中的字段需要被更新时，函数首先会获取一个名为 `ip` 的切片，然后检查它是否是 IPv6 地址。如果是 IPv6 地址，并且 `ipRec` 中的 `IP` 字段包含的地址比 `ip` 中的任何地址都新，那么就更新 `ipRec` 中的字段。

接下来，函数会根据 `ipRec` 中包含的请求类型，输出相应的 DNS 查询结果。如果 `ipRec` 中的字段需要被更新，函数会先尝试使用 `isNewer` 函数检查是否需要更新。如果需要更新，函数会使用 `net/ipv6` 包中的 `make` 函数创建一个新的 IPv6 地址切片，并更新 `ipRec` 中的 `IP` 字段。最后，函数会根据 `ipRec` 中的请求类型，输出相应的 DNS 查询结果。

函数使用了多个来自 Go 标准库的函数和常量，包括 `time` 包中的 `Since` 函数，`net/ipv6` 包中的 `make` 函数和 `net/ipv6` 包中的 `AtInfo` 函数。


```go
func (s *DoHNameServer) updateIP(req *dnsRequest, ipRec *IPRecord) {
	elapsed := time.Since(req.start)

	s.Lock()
	rec := s.ips[req.domain]
	updated := false

	switch req.reqType {
	case dnsmessage.TypeA:
		if isNewer(rec.A, ipRec) {
			rec.A = ipRec
			updated = true
		}
	case dnsmessage.TypeAAAA:
		addr := make([]net.Address, 0)
		for _, ip := range ipRec.IP {
			if len(ip.IP()) == net.IPv6len {
				addr = append(addr, ip)
			}
		}
		ipRec.IP = addr
		if isNewer(rec.AAAA, ipRec) {
			rec.AAAA = ipRec
			updated = true
		}
	}
	newError(s.name, " got answer: ", req.domain, " ", req.reqType, " -> ", ipRec.IP, " ", elapsed).AtInfo().WriteToLog()

	if updated {
		s.ips[req.domain] = rec
	}
	switch req.reqType {
	case dnsmessage.TypeA:
		s.pub.Publish(req.domain+"4", nil)
	case dnsmessage.TypeAAAA:
		s.pub.Publish(req.domain+"6", nil)
	}
	s.Unlock()
	common.Must(s.cleanup.Start())
}

```

This is a function that performs a DNS query using the DoH (Diesel On Host) protocol. The function takes in a slice of DNS requests and handles each request by using the `dns` query to retrieve the response. If a response cannot be found for a particular request, the function will log the error and return.

The function uses the `context` and `deadline` keywords to help manage the lifecycle of the request and response. The `context.Background()` method is used to create a new context for the current request, while `session.ContextWithInbound()` and `session.ContextWithContent()` are used to reserve a new context for the response and to set up the response for each request.

The function also uses the `mux` package to handle the response from the DOH protocol, as the current implementation does not use the DOH protocol directly. The `dns.PackMessage()` function is used to pack the DNS query message into a byte array, while the `s.dohHTTPSContext()` function is used to send the request to the DOH server.

The `parseResponse()` function is used to parse the response from the DOH server, while the `s.updateIP()` function is used to update the IP address of the response. If an error occurs during the query or response, the function will log the error and return.


```go
func (s *DoHNameServer) newReqID() uint16 {
	return uint16(atomic.AddUint32(&s.reqID, 1))
}

func (s *DoHNameServer) sendQuery(ctx context.Context, domain string, option IPOption) {
	newError(s.name, " querying: ", domain).AtInfo().WriteToLog(session.ExportIDToError(ctx))

	reqs := buildReqMsgs(domain, option, s.newReqID, genEDNS0Options(s.clientIP))

	var deadline time.Time
	if d, ok := ctx.Deadline(); ok {
		deadline = d
	} else {
		deadline = time.Now().Add(time.Second * 5)
	}

	for _, req := range reqs {
		go func(r *dnsRequest) {
			// generate new context for each req, using same context
			// may cause reqs all aborted if any one encounter an error
			dnsCtx := context.Background()

			// reserve internal dns server requested Inbound
			if inbound := session.InboundFromContext(ctx); inbound != nil {
				dnsCtx = session.ContextWithInbound(dnsCtx, inbound)
			}

			dnsCtx = session.ContextWithContent(dnsCtx, &session.Content{
				Protocol:      "https",
				SkipRoutePick: true,
			})

			// forced to use mux for DOH
			dnsCtx = session.ContextWithMuxPrefered(dnsCtx, true)

			var cancel context.CancelFunc
			dnsCtx, cancel = context.WithDeadline(dnsCtx, deadline)
			defer cancel()

			b, err := dns.PackMessage(r.msg)
			if err != nil {
				newError("failed to pack dns query").Base(err).AtError().WriteToLog()
				return
			}
			resp, err := s.dohHTTPSContext(dnsCtx, b.Bytes())
			if err != nil {
				newError("failed to retrieve response").Base(err).AtError().WriteToLog()
				return
			}
			rec, err := parseResponse(resp)
			if err != nil {
				newError("failed to handle DOH response").Base(err).AtError().WriteToLog()
				return
			}
			s.updateIP(r, rec)
		}(req)
	}
}

```

这段代码定义了一个名为`dohHTTPSContext`的函数，接收一个名为`s`的`DoHNameServer`类型的参数，以及一个字节数组`b`作为参数。函数的作用是在给定的`ctx`上下文中执行HTTP请求，并将请求发送到`s.dohURL`，然后获取服务器返回的响应。

函数首先创建一个名为`body`的字节缓冲区，并将传递给`http.NewRequest`的`b`字节数组设置为要发送的请求主体。然后，函数将请求设置为POST方法，并添加到请求头中的`Accept`和`Content-Type`字段。这些字段指定期望的服务器响应内容类型。

接下来，函数使用`s.httpClient`执行HTTP请求，并将请求的`ctx`和`body`作为参数传递给`http.Do`方法。函数返回一个`http.Response`对象，该对象包含服务器返回的响应。如果函数在执行过程中遇到错误，它将返回一个`error`类型的错误信息，并将其打印到控制台。如果服务器返回的响应状态码不是`http.StatusOK`，函数将尝试从响应主体中读取所有内容，并返回错误信息。

最后，函数使用`ioutil.ReadAll`从响应主体中读取所有内容，并将其返回。


```go
func (s *DoHNameServer) dohHTTPSContext(ctx context.Context, b []byte) ([]byte, error) {
	body := bytes.NewBuffer(b)
	req, err := http.NewRequest("POST", s.dohURL, body)
	if err != nil {
		return nil, err
	}

	req.Header.Add("Accept", "application/dns-message")
	req.Header.Add("Content-Type", "application/dns-message")

	resp, err := s.httpClient.Do(req.WithContext(ctx))
	if err != nil {
		return nil, err
	}

	defer resp.Body.Close()
	if resp.StatusCode != http.StatusOK {
		io.Copy(ioutil.Discard, resp.Body) // flush resp.Body so that the conn is reusable
		return nil, fmt.Errorf("DOH server returned code %d", resp.StatusCode)
	}

	return ioutil.ReadAll(resp.Body)
}

```

该函数的作用是获取一个域名服务器（DoHNameServer）中所有与给定域名对应的IP地址。它使用了两个选项（IPOption）来进行筛选，一个选项用于IPv6，另一个选项用于IPv4。函数内部首先检查给定的域名服务器是否包含与给定域名相对应的记录，如果存在记录，则检查给定选项中指定的IP类型（IPv6或IPv4）是否启用。如果选项中指定的IP类型启用，函数将尝试获取与给定域名相对应的IP地址，并将它们添加到ips数组中。如果选项中指定的IP类型禁用，或者给定的域名服务器中不存在与给定域名相对应的记录，函数将返回一个空数组并返回一个非空错误。函数的最终目的是返回一个包含所有与给定域名相对应的IP地址的数组，或者一个非空错误。


```go
func (s *DoHNameServer) findIPsForDomain(domain string, option IPOption) ([]net.IP, error) {
	s.RLock()
	record, found := s.ips[domain]
	s.RUnlock()

	if !found {
		return nil, errRecordNotFound
	}

	var ips []net.Address
	var lastErr error
	if option.IPv6Enable && record.AAAA != nil && record.AAAA.RCode == dnsmessage.RCodeSuccess {
		aaaa, err := record.AAAA.getIPs()
		if err != nil {
			lastErr = err
		}
		ips = append(ips, aaaa...)
	}

	if option.IPv4Enable && record.A != nil && record.A.RCode == dnsmessage.RCodeSuccess {
		a, err := record.A.getIPs()
		if err != nil {
			lastErr = err
		}
		ips = append(ips, a...)
	}

	if len(ips) > 0 {
		return toNetIP(ips), nil
	}

	if lastErr != nil {
		return nil, lastErr
	}

	if (option.IPv4Enable && record.A != nil) || (option.IPv6Enable && record.AAAA != nil) {
		return nil, dns_feature.ErrEmptyResponse
	}

	return nil, errRecordNotFound
}

```

这段代码的作用是实现了一个名为 QueryIP 的函数，用于从名为 DoHNameServer 的服务中查询域名对应的 IP 地址。函数接收一个 context 上下文，一个域名参数和一个名为 Option 的 IP 选项参数。

函数首先将输入的域名转换为 FQDN，然后使用 FindIPsForDomain 函数查找与域名对应的 IPv4 和 IPv6 地址。如果目录缓存中存在相应域名的 IPv4 或 IPv6 地址，则直接返回。否则，函数会执行以下操作：

1. 如果 Option 中 IPv4 选项启用，那么创建一个名为 "4" 的订阅者并将其设置为完成。
2. 如果 Option 中 IPv6 选项启用，那么创建一个名为 "6" 的订阅者并将其设置为完成。
3. 创建一个名为 "done" 的通道，用于通知函数已经完成其工作。
4. 循环查询目录缓存中与域名对应的 IPv4 和 IPv6 地址，如果目录缓存中不存在相应域名的 IPv4 或 IPv6 地址，则执行以下操作：
	1. 从订阅者中选择一条 IP 地址并将其发送到上下文。
	2. 如果发生了错误，设置 ctx.Err() 并返回。
	3. 如果查询操作超时，设置 <br />错误并返回 <br />。
	4. 循环结束后，如果订阅者仍然存在，将 "done" 中的值发送到上下文。这样，函数就可以继续使用之前选择的所有 IP 地址，而无需重新查询。


```go
// QueryIP is called from dns.Server->queryIPTimeout
func (s *DoHNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {
	fqdn := Fqdn(domain)

	ips, err := s.findIPsForDomain(fqdn, option)
	if err != errRecordNotFound {
		newError(s.name, " cache HIT ", domain, " -> ", ips).Base(err).AtDebug().WriteToLog()
		return ips, err
	}

	// ipv4 and ipv6 belong to different subscription groups
	var sub4, sub6 *pubsub.Subscriber
	if option.IPv4Enable {
		sub4 = s.pub.Subscribe(fqdn + "4")
		defer sub4.Close()
	}
	if option.IPv6Enable {
		sub6 = s.pub.Subscribe(fqdn + "6")
		defer sub6.Close()
	}
	done := make(chan interface{})
	go func() {
		if sub4 != nil {
			select {
			case <-sub4.Wait():
			case <-ctx.Done():
			}
		}
		if sub6 != nil {
			select {
			case <-sub6.Wait():
			case <-ctx.Done():
			}
		}
		close(done)
	}()
	s.sendQuery(ctx, fqdn, option)

	for {
		ips, err := s.findIPsForDomain(fqdn, option)
		if err != errRecordNotFound {
			return ips, err
		}

		select {
		case <-ctx.Done():
			return nil, ctx.Err()
		case <-done:
		}
	}
}

```

# `app/dns/errors.generated.go`

这段代码定义了一个名为 `errPathObjHolder` 的结构体，它包含一个空的字符串字段 `errPath` 和一个空的字符串字段 `errMsg`。

接着，定义了一个名为 `newError` 的函数，该函数接收一个或多个 `interface{}` 类型的参数。函数内部创建一个名为 `errPathObjHolder` 的新的字符串对象，并将其作为构造函数的参数传递给 `errors.New` 函数。如果调用 `newError` 函数时传递了空字符串参数，则会自动将其转换为一个新的字符串对象，并将其作为构造函数的参数传递给 `errors.New` 函数。

通过调用 `errPathObjHolder`，创建一个新的 `errPathObjHolder` 对象，并将其传递给 `errors.New` 函数的构造函数。如果调用 `newError` 函数时传递的参数不包含字符串，则 `errPathObjHolder` 对象将无法创建，并抛出一个异常。


```go
package dns

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `app/dns/hosts.go`

这段代码定义了一个名为`StaticHosts`的结构体，表示在DNS服务器上的静态主机到IP的映射。它包含一个IP地址数组`ips`，用于存储静态主机的目标IP地址，以及一个`strmatcher.MatcherGroup`类型的`matchers`字段，用于检查DNS查询中的输入数据是否符合预定义的正则表达式。最后，它使用了`+build`和`!confonly`选项，表示在编译时生成二进制文件，并仅在配置文件中运行。


```go
// +build !confonly

package dns

import (
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/strmatcher"
	"v2ray.com/core/features"
)

// StaticHosts represents static domain-ip mapping in DNS server.
type StaticHosts struct {
	ips      [][]net.Address
	matchers *strmatcher.MatcherGroup
}

```

这段代码定义了一个名为 `typeMap` 的 map，它包含了一个名为 `DomainMatchingType` 的键，其值为 `strmatcher.Type` 类型。这个键下面的值为 `strmatcher.Full`, `strmatcher.Domain`, `strmatcher.Keyword`, 和 `strmatcher.Regex` 四个不同类型的 `strmatcher.Type` 值。

该函数 `toStrMatcher` 接受两个参数，一个是 `DomainMatchingType` 类型，另一个是字符串 `domain`。它首先尝试从 `typeMap` 中查找与 `DomainMatchingType` 匹配的键，如果不匹配，则返回一个错误并输出警告。如果找到了匹配的键，它使用该键对应的 `strmatcher.Type` 值创建一个 `strmatcher.Matcher` 实例，并返回该实例。

该函数的作用是帮助用户创建一个与传入的 `DomainMatchingType` 类型和字符串 `domain` 匹配的 `strmatcher.Matcher` 实例。


```go
var typeMap = map[DomainMatchingType]strmatcher.Type{
	DomainMatchingType_Full:      strmatcher.Full,
	DomainMatchingType_Subdomain: strmatcher.Domain,
	DomainMatchingType_Keyword:   strmatcher.Substr,
	DomainMatchingType_Regex:     strmatcher.Regex,
}

func toStrMatcher(t DomainMatchingType, domain string) (strmatcher.Matcher, error) {
	strMType, f := typeMap[t]
	if !f {
		return nil, newError("unknown mapping type", t).AtWarning()
	}
	matcher, err := strMType.New(domain)
	if err != nil {
		return nil, newError("failed to create str matcher").Base(err)
	}
	return matcher, nil
}

```

This is a Go function that creates a Static IP (SSH) proxy. It takes a domain name as input and creates a mapping of IP addresses to host aliases for that domain.

Here is the function implementation:
go
func New(domain, ip string) (*StaticIp, error) {
	staticIp, err := ipv4.NewStaticIP(domain, ip)
	if err != nil {
		return nil, err
	}

	err = ipv6.Addr(staticIp, &staticIp.IP{IPv6Address: ipv6.Parse(ipv6.Any)})
	if err != nil {
		return nil, err
	}

	return staticIp, nil
}

This function takes a `domain` name and an optional `ip` string as input. It creates a new `StaticIp` object and adds the IP address to it. It also adds the domain to the mapping of IP addresses to host aliases.

Here is the `StaticIp` struct:
go
type StaticIp struct {
	IP	*ipv4.IP
	IPv6	*ipv6.IP
}

This struct represents an IP address and its IPv6 address.

The `New(domain, ip)` function creates a new `StaticIp` object with the given `domain` name and `ip` string. It first checks if the input `ip` is a valid IPv4 or IPv6 address. If the input `ip` is not a valid IP address, the function panics with an error.

The `StaticIp` struct is then initialized with the input `domain` and `ip`.

The `ipv4.NewStaticIP` function is used to create the IPv4 address. This function returns a new `StaticIp` object.

The `ipv6.Addr` function is used to add the IPv6 address to the `StaticIp` object. This function takes a `*ipv6.IP` object and adds it to the `StaticIp` object.

The `return` statement returns the `StaticIp` object. If an error occurs, it returns `nil`.


```go
// NewStaticHosts creates a new StaticHosts instance.
func NewStaticHosts(hosts []*Config_HostMapping, legacy map[string]*net.IPOrDomain) (*StaticHosts, error) {
	g := new(strmatcher.MatcherGroup)
	sh := &StaticHosts{
		ips:      make([][]net.Address, len(hosts)+len(legacy)+16),
		matchers: g,
	}

	if legacy != nil {
		features.PrintDeprecatedFeatureWarning("simple host mapping")

		for domain, ip := range legacy {
			matcher, err := strmatcher.Full.New(domain)
			common.Must(err)
			id := g.Add(matcher)

			address := ip.AsAddress()
			if address.Family().IsDomain() {
				return nil, newError("invalid domain address in static hosts: ", address.Domain()).AtWarning()
			}

			sh.ips[id] = []net.Address{address}
		}
	}

	for _, mapping := range hosts {
		matcher, err := toStrMatcher(mapping.Type, mapping.Domain)
		if err != nil {
			return nil, newError("failed to create domain matcher").Base(err)
		}
		id := g.Add(matcher)
		ips := make([]net.Address, 0, len(mapping.Ip)+1)
		if len(mapping.Ip) > 0 {
			for _, ip := range mapping.Ip {
				addr := net.IPAddress(ip)
				if addr == nil {
					return nil, newError("invalid IP address in static hosts: ", ip).AtWarning()
				}
				ips = append(ips, addr)
			}
		} else if len(mapping.ProxiedDomain) > 0 {
			ips = append(ips, net.DomainAddress(mapping.ProxiedDomain))
		} else {
			return nil, newError("neither IP address nor proxied domain specified for domain: ", mapping.Domain).AtWarning()
		}

		// Special handling for localhost IPv6. This is a dirty workaround as JSON config supports only single IP mapping.
		if len(ips) == 1 && ips[0] == net.LocalHostIP {
			ips = append(ips, net.LocalHostIPv6)
		}

		sh.ips[id] = ips
	}

	return sh, nil
}

```

这两段代码都是实现了一个 IP 地址选择器，主要通过在路由器（Router）模式下匹配静态主机（Static Hosts）的 IP 地址，然后返回匹配到的第一个 IP 地址。实现过程如下：

1. `filterIP`函数的作用是筛选出与给定的选项（IP 选项）匹配的 IP 地址。它接收一个 IP 地址数组`ips`和一个选项`option`，首先检查给定的选项中是否启用了 IPv4 或 IPv6。然后，逐个检查输入的 IP 地址。如果地址属于 IPv4 并且选项中启用了 IPv4，或者地址属于 IPv6 并且选项中启用了 IPv6，那么该地址就会被添加到 `filtered` 数组中。最后，返回 `filtered` 数组，如果没有匹配到的 IP 地址，返回 `nil`。

2. `LookupIP`函数的作用是在给定的静态主机域名中查找 IP 地址。它接收一个域名`domain`和一个选项`option`，首先查找静态主机数组中的 IP 地址。如果域名中包含多个 IPv4 或 IPv6 子网，那么函数将返回子网中的第一个 IP 地址。否则，函数调用 `filterIP` 函数来查找匹配的 IP 地址。最后，返回匹配到的第一个 IP 地址。


```go
func filterIP(ips []net.Address, option IPOption) []net.Address {
	filtered := make([]net.Address, 0, len(ips))
	for _, ip := range ips {
		if (ip.Family().IsIPv4() && option.IPv4Enable) || (ip.Family().IsIPv6() && option.IPv6Enable) {
			filtered = append(filtered, ip)
		}
	}
	if len(filtered) == 0 {
		return nil
	}
	return filtered
}

// LookupIP returns IP address for the given domain, if exists in this StaticHosts.
func (h *StaticHosts) LookupIP(domain string, option IPOption) []net.Address {
	indices := h.matchers.Match(domain)
	if len(indices) == 0 {
		return nil
	}
	ips := []net.Address{}
	for _, id := range indices {
		ips = append(ips, h.ips[id]...)
	}
	if len(ips) == 1 && ips[0].Family().IsDomain() {
		return ips
	}
	return filterIP(ips, option)
}

```

# `app/dns/hosts_test.go`

The code appears to be testing a function that uses the `NewStaticHosts` function from the `v2ray.go` package to get a list of IP addresses for a given hostname.

The function takes two arguments:

* `pb`: A struct that implements the `v2ray.v1000` protocol. This structure appears to be a marker for the protocol, and does not appear to have any direct implementation.
* `nil`: A pointer to a value of type `byte`.

The function returns nothing, but it performs the following actions:

* It creates a new instance of the `StaticHosts` class using the `New` method.
* It calls the `hosts.LookupIP` method on this instance, passing in the hostname and an empty byte slice.
* It returns the result of the call.

The `LookupIP` method appears to return a list of IP addresses for the given hostname.

The tests for the function use the `hosts.LookupIP` method to look up the IP addresses of the `www.v2ray.cn` and `baidu.com` domains.

The tests check that the function returns the correct number of IP addresses for each domain, and also check that the IP addresses returned by the function are valid.

If the tests pass, then the function should work as expected. If any of the tests fail, then there may be a bug in the code.


```go
package dns_test

import (
	"testing"

	"github.com/google/go-cmp/cmp"

	. "v2ray.com/core/app/dns"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
)

func TestStaticHosts(t *testing.T) {
	pb := []*Config_HostMapping{
		{
			Type:   DomainMatchingType_Full,
			Domain: "v2ray.com",
			Ip: [][]byte{
				{1, 1, 1, 1},
			},
		},
		{
			Type:   DomainMatchingType_Subdomain,
			Domain: "v2ray.cn",
			Ip: [][]byte{
				{2, 2, 2, 2},
			},
		},
		{
			Type:   DomainMatchingType_Subdomain,
			Domain: "baidu.com",
			Ip: [][]byte{
				{127, 0, 0, 1},
			},
		},
	}

	hosts, err := NewStaticHosts(pb, nil)
	common.Must(err)

	{
		ips := hosts.LookupIP("v2ray.com", IPOption{
			IPv4Enable: true,
			IPv6Enable: true,
		})
		if len(ips) != 1 {
			t.Error("expect 1 IP, but got ", len(ips))
		}
		if diff := cmp.Diff([]byte(ips[0].IP()), []byte{1, 1, 1, 1}); diff != "" {
			t.Error(diff)
		}
	}

	{
		ips := hosts.LookupIP("www.v2ray.cn", IPOption{
			IPv4Enable: true,
			IPv6Enable: true,
		})
		if len(ips) != 1 {
			t.Error("expect 1 IP, but got ", len(ips))
		}
		if diff := cmp.Diff([]byte(ips[0].IP()), []byte{2, 2, 2, 2}); diff != "" {
			t.Error(diff)
		}
	}

	{
		ips := hosts.LookupIP("baidu.com", IPOption{
			IPv4Enable: false,
			IPv6Enable: true,
		})
		if len(ips) != 1 {
			t.Error("expect 1 IP, but got ", len(ips))
		}
		if diff := cmp.Diff([]byte(ips[0].IP()), []byte(net.LocalHostIPv6.IP())); diff != "" {
			t.Error(diff)
		}
	}
}

```

# `app/dns/nameserver.go`

这段代码是一个 Go 语言编写的 build 工具脚本，用于构建 Go 语言项目。它使用了几个用于构建工具的常用工具链，其中包括：

1. `build`：这是一个用于构建项目的工具链，通常与 `go build` 命令一起使用。它会在项目目录中创建一个名为 `.github.org/处分.赫尔墨违反限制用时build包含源代码的仓库` 的目录，并在该目录下执行 `go build` 命令。
2. `confonly`：这是一个用于仅在特定构建设置中输出文件的工具链，通常与 `go build --force-confonly` 一起使用。它会在项目目录中创建一个名为 `.github.org/分布式确保本地hostsbook包含源代码的仓库` 的目录，并在该目录下执行 `go build --force-confonly` 命令。

这两部分工具链的目的是让开发人员在构建 Go 语言项目时，能够更轻松地构建和仅在所需的构建设置中输出文件。


```go
// +build !confonly

package dns

import (
	"context"

	"v2ray.com/core/common/net"
	"v2ray.com/core/features/dns/localdns"
)

// IPOption is an object for IP query options.
type IPOption struct {
	IPv4Enable bool
	IPv6Enable bool
}

```

此代码定义了一个名为Client的接口，该接口表示DNS客户端。该接口有一个名为Name的属性，表示客户端的名称。

该接口还有一个名为QueryIP的函数，用于向其配置服务器发送IP查询。该函数需要一个上下文句柄和一个域名参数，以及一个IP选项。函数返回一个IPv4或IPv6 slice（表示查询结果）和错误。

该代码还定义了一个名为localNameServer的类型，该类型表示本地名称服务器。该类型包含一个名为client的属性，该属性表示一个指向localDNS.Client的引用。

该代码还实现了一个名为QueryIP的函数，该函数使用上面定义的QueryIP函数从localNameServer的client中查询给定的域名，并返回查询结果。如果该服务器支持IPv4或IPv6查询，则该函数将返回查询结果。如果服务器不支持任何查询，则会返回一个error。


```go
// Client is the interface for DNS client.
type Client interface {
	// Name of the Client.
	Name() string

	// QueryIP sends IP queries to its configured server.
	QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error)
}

type localNameServer struct {
	client *localdns.Client
}

func (s *localNameServer) QueryIP(ctx context.Context, domain string, option IPOption) ([]net.IP, error) {
	if option.IPv4Enable && option.IPv6Enable {
		return s.client.LookupIP(domain)
	}

	if option.IPv4Enable {
		return s.client.LookupIPv4(domain)
	}

	if option.IPv6Enable {
		return s.client.LookupIPv6(domain)
	}

	return nil, newError("neither IPv4 nor IPv6 is enabled")
}

```

这段代码定义了两个函数，以及一个名为“localNameServer”的结构体类型。

第一个函数名为“func”，它接收一个名为“s”的局部名称服务器指针，并返回该指针所指向的服务器的名称。函数内部没有定义参数，因为它们已经在函数签名中声明了。

第二个函数名为“NewLocalNameServer”，它返回一个名为“localNameServer”的结构体类型的实例。该结构体包含一个名为“client”的成员变量，该成员变量是一个指向名为“localdns.New”的函数的指针。

此外，还有一段注释。


```go
func (s *localNameServer) Name() string {
	return "localhost"
}

func NewLocalNameServer() *localNameServer {
	newError("DNS: created localhost client").AtInfo().WriteToLog()
	return &localNameServer{
		client: localdns.New(),
	}
}

```

# `app/dns/nameserver_test.go`

这段代码是一个名为"dns_test"的包，它定义了一个名为"TestLocalNameServer"的测试函数，用于测试本地名称服务器(LocalNameServer)的功能。

具体来说，代码中使用了以下几点：

1. 导入了一些外部依赖：

- "v2ray.com/core/app/dns"：定义了DNS客户端(DNS client)的接口，通过它进行DNS查询。
- "testing"：用于元测试(Testing)。

2. 实现了DNS客户端的接口：

- "NewLocalNameServer"：实现了DNS客户端的"NewLocalNameServer"函数，用于创建一个本地名称服务器(LocalNameServer)，并返回其客户端实例。
- "QueryIP"：实现了DNS客户端的"QueryIP"函数，用于查询域名对应的IP地址。

3. 实现了"TestLocalNameServer"函数：

- "s"：定义了一个名为"s"的变量，保存了一个DNS客户端实例，该实例使用默认的DNS服务器(通常是本地系统)。
- "ctx, cancel"：使用了"context"和"timeout"函数，用于创建一个上下文(Context)、设置超时(Timeout)并取消超时。
- "QueryIP"：使用了"QueryIP"函数，并传入了一些参数，用于查询google.com域名的IP地址。

4. 通过调用"s.QueryIP"函数，测试了域名"google.com"的IP地址是否正确：

- 如果测试成功，说明"google.com"域名的IP地址可以被正确查询到，否则会输出错误信息。


```go
package dns_test

import (
	"context"
	"testing"
	"time"

	. "v2ray.com/core/app/dns"
	"v2ray.com/core/common"
)

func TestLocalNameServer(t *testing.T) {
	s := NewLocalNameServer()
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*2)
	ips, err := s.QueryIP(ctx, "google.com", IPOption{
		IPv4Enable: true,
		IPv6Enable: true,
	})
	cancel()
	common.Must(err)
	if len(ips) == 0 {
		t.Error("expect some ips, but got 0")
	}
}

```

# `app/dns/server.go`

这段代码是一个 Go 语言编写的文本，它定义了一个名为 "dns" 的包。这个包通过 "build" 命令构建，并在构建过程中排除 "confonly" 选项，说明这个包不会在构建过程中被缓存。

具体来说，这段代码以下划线标记了两个用于构建模式的方法：

1. 第一个方法以 "dns" 包为前缀导入了 "v2ray.com/core/features/dns" 和 "v2ray.com/core/features/routing" 包。这两个包一起用于创建一个用于 DNS 查询和路由的路由器实例。

2. 第二个方法定义了一个名为 "dnsconfig" 的常量，它使用 "golang.org/x/net/ipv4/config" 包配置 DNS 服务器的主机名和端口号。

3. 第三个方法 "generateDnsServers" 尝试从 "v2ray.com/core/features/exec" 包中执行 "generate" 命令，以生成 DNS 服务器列表。但是，如果失败，它会记录一个名为 "errorgen" 的错误。

4. 最后一个方法 "buildAndCheck" 尝试构建 "dns" 包，并在构建过程中排除 "confonly" 选项。如果构建成功，它会尝试启动 DNS 服务器以确认是否可以使用 "dnsconfig" 中指定的主机名和端口号。如果 DNS 服务器启动成功，则表示构建成功。


```go
// +build !confonly

package dns

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"
	"fmt"
	"log"
	"net/url"
	"strings"
	"sync"
	"time"

	"v2ray.com/core"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/strmatcher"
	"v2ray.com/core/common/uuid"
	"v2ray.com/core/features"
	"v2ray.com/core/features/dns"
	"v2ray.com/core/features/routing"
)

```

这段代码定义了一个名为Server的DNS服务器实例结构体。它实现了多个并发通信相关的方法，用于管理客户端与服务器之间的交互。主要作用是确保客户端与服务器之间的数据安全以及高效地处理客户端请求。具体来说，这段代码实现了以下功能：

1. 定义了一个名为Server的客户端结构体，其中包含了与服务器进行交互所需的全部数据结构和方法。

2. 实现了一个名为ipIndexMap的MultiGeoIPMatcher类型数组，用于存储客户端与服务器之间的IP地址映射。

3. 实现了一个名为domainRules的二维切片类型数组，其中包含了一个域名规则数组，用于存储DNS域名的规则信息。

4. 实现了一个名为domainMatcher的DomainMatcherInfo类型数组，其中包含了一个域名匹配器接口，用于在客户端与服务器之间传递域名和匹配器ID。

5. 实现了一个名为tag的固定长字符串类型变量，用于在客户端与服务器之间传递一些元数据信息。

6. 通过在整个代码中使用synchronized关键字，确保了客户端与服务器之间的多个并行交互操作的安全性和可靠性。

7. 通过使用静态类型的StaticHosts类型，实现了在内存中缓存客户端的IP地址和域名信息，提高客户端与服务器之间的通信效率。


```go
// Server is a DNS rely server.
type Server struct {
	sync.Mutex
	hosts         *StaticHosts
	clientIP      net.IP
	clients       []Client             // clientIdx -> Client
	ipIndexMap    []*MultiGeoIPMatcher // clientIdx -> *MultiGeoIPMatcher
	domainRules   [][]string           // clientIdx -> domainRuleIdx -> DomainRule
	domainMatcher strmatcher.IndexMatcher
	matcherInfos  []DomainMatcherInfo // matcherIdx -> DomainMatcherInfo
	tag           string
}

// DomainMatcherInfo contains information attached to index returned by Server.domainMatcher
type DomainMatcherInfo struct {
	clientIdx     uint16
	domainRuleIdx uint16
}

```

这段代码定义了一个名为MultiGeoIPMatcher的结构体，它包含一个字符串类型的slice变量，称为matchers，用于存储所有用于检查IP匹配的matcher。

该结构体还定义了一个名为errExpectedIPNonMatch的错误类型，用于在IP匹配不匹配时抛出。

该MultiGeoIPMatcher结构体有一个名为Match的函数，该函数接收一个网络IP地址作为参数，并返回一个布尔值以表示是否匹配。在Match函数内部，遍历所有的matchers，如果有一个matcher能够匹配IP地址，则返回true，否则返回false。

最后，该代码没有定义任何其他函数或变量，可能是因为需要根据具体使用场景进行自定义使用。


```go
// MultiGeoIPMatcher for match
type MultiGeoIPMatcher struct {
	matchers []*router.GeoIPMatcher
}

var errExpectedIPNonMatch = errors.New("expectIPs not match")

// Match check ip match
func (c *MultiGeoIPMatcher) Match(ip net.IP) bool {
	for _, matcher := range c.matchers {
		if matcher.Match(ip) {
			return true
		}
	}
	return false
}

```

This is a Go-like language (expressed as用Go编写的？) that implements a DNS server to handle and forward local domain name system (LD) traffic. It does so by using the libnss3 library to handle the DNS response and geoip-api library to handle the geolocation of the clients.

The DNS server takes in a list of rules, which are LD rules, and a list of domain name systems (DNS servers), and returns the DNS server to handle the given domains. It also provides a mechanism for generating new rules based on a current domain name system (DNS) matcher and supports the configuration of a GeoIP system.

The DNS server has a basic structure where it takes in a list of rules and a list of DNS servers, and returns the DNS server to handle the given domains. It also has functions to add domain name systems, generate rules based on DNS matches, and configure a GeoIP system.

It also has a function to configure the number of clients that will be used to communicate with the DNS server and initializes the DNS server with the default configuration.


```go
// HasMatcher check has matcher
func (c *MultiGeoIPMatcher) HasMatcher() bool {
	return len(c.matchers) > 0
}

func generateRandomTag() string {
	id := uuid.New()
	return "v2ray.system." + id.String()
}

// New creates a new DNS server with given configuration.
func New(ctx context.Context, config *Config) (*Server, error) {
	server := &Server{
		clients: make([]Client, 0, len(config.NameServers)+len(config.NameServer)),
		tag:     config.Tag,
	}
	if server.tag == "" {
		server.tag = generateRandomTag()
	}
	if len(config.ClientIp) > 0 {
		if len(config.ClientIp) != net.IPv4len && len(config.ClientIp) != net.IPv6len {
			return nil, newError("unexpected IP length", len(config.ClientIp))
		}
		server.clientIP = net.IP(config.ClientIp)
	}

	hosts, err := NewStaticHosts(config.StaticHosts, config.Hosts)
	if err != nil {
		return nil, newError("failed to create hosts").Base(err)
	}
	server.hosts = hosts

	addNameServer := func(ns *NameServer) int {
		endpoint := ns.Address
		address := endpoint.Address.AsAddress()
		if address.Family().IsDomain() && address.Domain() == "localhost" {
			server.clients = append(server.clients, NewLocalNameServer())
			// Priotize local domains with specific TLDs or without any dot to local DNS
			// References:
			// https://www.iana.org/assignments/special-use-domain-names/special-use-domain-names.xhtml
			// https://unix.stackexchange.com/questions/92441/whats-the-difference-between-local-home-and-lan
			localTLDsAndDotlessDomains := []*NameServer_PriorityDomain{
				{Type: DomainMatchingType_Regex, Domain: "^[^.]+$"}, // This will only match domains without any dot
				{Type: DomainMatchingType_Subdomain, Domain: "local"},
				{Type: DomainMatchingType_Subdomain, Domain: "localdomain"},
				{Type: DomainMatchingType_Subdomain, Domain: "localhost"},
				{Type: DomainMatchingType_Subdomain, Domain: "lan"},
				{Type: DomainMatchingType_Subdomain, Domain: "home.arpa"},
				{Type: DomainMatchingType_Subdomain, Domain: "example"},
				{Type: DomainMatchingType_Subdomain, Domain: "invalid"},
				{Type: DomainMatchingType_Subdomain, Domain: "test"},
			}
			ns.PrioritizedDomain = append(ns.PrioritizedDomain, localTLDsAndDotlessDomains...)
		} else if address.Family().IsDomain() && strings.HasPrefix(address.Domain(), "https+local://") {
			// URI schemed string treated as domain
			// DOH Local mode
			u, err := url.Parse(address.Domain())
			if err != nil {
				log.Fatalln(newError("DNS config error").Base(err))
			}
			server.clients = append(server.clients, NewDoHLocalNameServer(u, server.clientIP))
		} else if address.Family().IsDomain() && strings.HasPrefix(address.Domain(), "https://") {
			// DOH Remote mode
			u, err := url.Parse(address.Domain())
			if err != nil {
				log.Fatalln(newError("DNS config error").Base(err))
			}
			idx := len(server.clients)
			server.clients = append(server.clients, nil)

			// need the core dispatcher, register DOHClient at callback
			common.Must(core.RequireFeatures(ctx, func(d routing.Dispatcher) {
				c, err := NewDoHNameServer(u, d, server.clientIP)
				if err != nil {
					log.Fatalln(newError("DNS config error").Base(err))
				}
				server.clients[idx] = c
			}))
		} else {
			// UDP classic DNS mode
			dest := endpoint.AsDestination()
			if dest.Network == net.Network_Unknown {
				dest.Network = net.Network_UDP
			}
			if dest.Network == net.Network_UDP {
				idx := len(server.clients)
				server.clients = append(server.clients, nil)

				common.Must(core.RequireFeatures(ctx, func(d routing.Dispatcher) {
					server.clients[idx] = NewClassicNameServer(dest, d, server.clientIP)
				}))
			}
		}
		server.ipIndexMap = append(server.ipIndexMap, nil)
		return len(server.clients) - 1
	}

	if len(config.NameServers) > 0 {
		features.PrintDeprecatedFeatureWarning("simple DNS server")
		for _, destPB := range config.NameServers {
			addNameServer(&NameServer{Address: destPB})
		}
	}

	if len(config.NameServer) > 0 {
		clientIndices := []int{}
		domainRuleCount := 0
		for _, ns := range config.NameServer {
			idx := addNameServer(ns)
			clientIndices = append(clientIndices, idx)
			domainRuleCount += len(ns.PrioritizedDomain)
		}

		domainRules := make([][]string, len(server.clients))
		domainMatcher := &strmatcher.MatcherGroup{}
		matcherInfos := make([]DomainMatcherInfo, domainRuleCount+1) // matcher index starts from 1
		var geoIPMatcherContainer router.GeoIPMatcherContainer
		for nidx, ns := range config.NameServer {
			idx := clientIndices[nidx]

			// Establish domain rule matcher
			rules := []string{}
			ruleCurr := 0
			ruleIter := 0
			for _, domain := range ns.PrioritizedDomain {
				matcher, err := toStrMatcher(domain.Type, domain.Domain)
				if err != nil {
					return nil, newError("failed to create prioritized domain").Base(err).AtWarning()
				}
				midx := domainMatcher.Add(matcher)
				if midx >= uint32(len(matcherInfos)) { // This rarely happens according to current matcher's implementation
					newError("expanding domain matcher info array to size ", midx, " when adding ", matcher).AtDebug().WriteToLog()
					matcherInfos = append(matcherInfos, make([]DomainMatcherInfo, midx-uint32(len(matcherInfos))+1)...)
				}
				info := &matcherInfos[midx]
				info.clientIdx = uint16(idx)
				if ruleCurr < len(ns.OriginalRules) {
					info.domainRuleIdx = uint16(ruleCurr)
					rule := ns.OriginalRules[ruleCurr]
					if ruleCurr >= len(rules) {
						rules = append(rules, rule.Rule)
					}
					ruleIter++
					if ruleIter >= int(rule.Size) {
						ruleIter = 0
						ruleCurr++
					}
				} else { // No original rule, generate one according to current domain matcher (majorly for compatibility with tests)
					info.domainRuleIdx = uint16(len(rules))
					rules = append(rules, matcher.String())
				}
			}
			domainRules[idx] = rules

			// only add to ipIndexMap if GeoIP is configured
			if len(ns.Geoip) > 0 {
				var matchers []*router.GeoIPMatcher
				for _, geoip := range ns.Geoip {
					matcher, err := geoIPMatcherContainer.Add(geoip)
					if err != nil {
						return nil, newError("failed to create ip matcher").Base(err).AtWarning()
					}
					matchers = append(matchers, matcher)
				}
				matcher := &MultiGeoIPMatcher{matchers: matchers}
				server.ipIndexMap[idx] = matcher
			}
		}
		server.domainRules = domainRules
		server.domainMatcher = domainMatcher
		server.matcherInfos = matcherInfos
	}

	if len(server.clients) == 0 {
		server.clients = append(server.clients, NewLocalNameServer())
		server.ipIndexMap = append(server.ipIndexMap, nil)
	}

	return server, nil
}

```

这段代码定义了一个名为`Server`的服务器类型，它实现了两个接口：`common.HasType`和`common.Runnable`。

具体来说，`Server`类型具有一些通用的功能，如可以创建一个`Start`方法用于启动服务器，并在`Close`方法中返回一个`nil`表示成功关闭服务器。

同时，`Server`类型实现了`common.HasType`接口，它提供了一个名为`Type`的类型，通过这个类型，可以调用`dns.ClientType()`函数来获取服务器的主机类型。

另外，`Server`类型还实现了`common.Runnable`接口，它提供了一个名为`Start`的`Start`方法，通过调用这个方法，可以启动服务器并返回一个`error`表示结果。


```go
// Type implements common.HasType.
func (*Server) Type() interface{} {
	return dns.ClientType()
}

// Start implements common.Runnable.
func (s *Server) Start() error {
	return nil
}

// Close implements common.Closable.
func (s *Server) Close() error {
	return nil
}

```

此代码定义了两个函数，分别为`func (s *Server) IsOwnLink`和`func (s *Server) Match`。

1. `func (s *Server) IsOwnLink`函数接收一个`Server`类型的参数，并返回一个布尔值。函数的作用是检查传入的`Server`对象是否是自己的链接。函数的实现中，首先从会话中获取当前客户端的`Inbound`标记，然后检查`Server`对象的`tag`是否与当前客户端的`tag`相同。如果两个`tag`相同，那么函数返回`true`，否则返回`false`。

2. `func (s *Server) Match`函数接收一个整数参数`idx`、一个`Client`类型的参数`client`和一个字符串参数`domain`，并返回一个IP数组或错误。函数的作用是检查客户端与服务器IP地址是否匹配。函数的实现中，首先尝试从`Server`对象中某个索引映射的`MultiGeoIPMatcher`对象中获取一个`Matcher`。如果`Matcher`对象存在，则使用`Matcher`的`HasMatcher()`方法判断当前客户端的`domain`是否属于服务器分配的IP地址范围内。如果不属于，函数会尝试匹配客户端的IP地址。如果匹配成功，函数返回新的IP数组。如果匹配失败，函数会记录错误并返回。


```go
func (s *Server) IsOwnLink(ctx context.Context) bool {
	inbound := session.InboundFromContext(ctx)
	return inbound != nil && inbound.Tag == s.tag
}

// Match check dns ip match geoip
func (s *Server) Match(idx int, client Client, domain string, ips []net.IP) ([]net.IP, error) {
	var matcher *MultiGeoIPMatcher
	if idx < len(s.ipIndexMap) {
		matcher = s.ipIndexMap[idx]
	}
	if matcher == nil {
		return ips, nil
	}

	if !matcher.HasMatcher() {
		newError("domain ", domain, " server has no valid matcher: ", client.Name(), " idx:", idx).AtDebug().WriteToLog()
		return ips, nil
	}

	newIps := []net.IP{}
	for _, ip := range ips {
		if matcher.Match(ip) {
			newIps = append(newIps, ip)
		}
	}
	if len(newIps) == 0 {
		return nil, errExpectedIPNonMatch
	}
	newError("domain ", domain, " expectIPs ", newIps, " matched at server ", client.Name(), " idx:", idx).AtDebug().WriteToLog()
	return newIps, nil
}

```

该函数`func (s *Server) queryIPTimeout`接收三个参数：

1. `s`：类型为`Server`的引用，可能是用于管理服务器工作的`Server`实例。
2. `client`：类型为`Client`的引用，可能是用于与服务器通信的客户端实例。
3. `domain`：类型为`string`的字符串，表示要查询的域名。
4. `option`：类型为`IPOption`的引用，可能是一个用于指定IP查询选项的接口。

函数的作用是查询指定域名下的IP地址，并返回它们的地址，如果查询失败则返回一个错误。

具体实现步骤如下：

1. 确保函数在带有超时时间的上下文中执行。上下文由`context.WithTimeout`函数创建，超时时间为4秒钟。
2. 如果`s.tag`字段中包含任何内容，则将超时上下文更改为包含`s.tag`的上下文，这样函数将使用预定义的入站选中。
3. 调用客户端的`QueryIP`函数查询指定域名下的IP地址。
4. 如果查询成功，将返回查询到的IP地址。
5. 如果查询失败，函数将返回一个错误。

函数的返回值是一个包含所有查询结果的`[]net.IP`切片，或者是错误。


```go
func (s *Server) queryIPTimeout(idx int, client Client, domain string, option IPOption) ([]net.IP, error) {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*4)
	if len(s.tag) > 0 {
		ctx = session.ContextWithInbound(ctx, &session.Inbound{
			Tag: s.tag,
		})
	}
	ips, err := client.QueryIP(ctx, domain, option)
	cancel()

	if err != nil {
		return ips, err
	}

	ips, err = s.Match(idx, client, domain, ips)
	return ips, err
}

```

这段代码定义了两个名为 `LookupIP` 的函数，属于 `dns.Client` 类型。这两个函数用于查找给定域名对应的 IP 地址。

具体来说，第一个函数 `LookupIP` 接收一个字符串 `domain`，使用 `s.lookupIPInternal` 函数查找该域名对应的 IP 地址。该函数包含两个参数，第一个参数是一个 `IPOption` 类型，指定了在 IPv4 网络上启用 IPv4 查询，第二个参数是一个 DNS 查询选项，指定了是否启用 IPv6 查询。

第二个函数 `LookupIPv4` 也接收一个字符串 `domain`，使用 `s.lookupIPInternal` 函数查找该域名对应的 IPv4 地址。该函数包含一个名为 `IPOption` 的参数，指定了在 IPv4 网络上启用 IPv4 查询，以及一个名为 `IPv6Enable` 的参数，指定了是否启用 IPv6 查询。不过，该函数的第二个参数 `IPv6Enable` 被设置为 `false`，表示不启用 IPv6 查询。

这两个函数的实现依赖于 `s.lookupIPInternal` 函数，该函数用于在本地 DNS 服务器中查询给定域名对应的 IP 地址。如果查询成功，函数将返回一个包含指定查询域名的 IPv4 地址列表和可能的错误。如果查询失败，函数将返回一个空 IP 地址列表和可能的错误。


```go
// LookupIP implements dns.Client.
func (s *Server) LookupIP(domain string) ([]net.IP, error) {
	return s.lookupIPInternal(domain, IPOption{
		IPv4Enable: true,
		IPv6Enable: true,
	})
}

// LookupIPv4 implements dns.IPv4Lookup.
func (s *Server) LookupIPv4(domain string) ([]net.IP, error) {
	return s.lookupIPInternal(domain, IPOption{
		IPv4Enable: true,
		IPv6Enable: false,
	})
}

```

该代码实现了一个名为“LookupIPv6”的函数，该函数使用名为“dns.IPv6Lookup”的dns包从网络中查询IPv6地址。

函数有两个参数：

1. “domain”参数：字符串，表示要查询的域名。
2. “option”参数：IPOption结构体，包含以下字段：
	* “IPv4Enable”字段：bool，表示是否使用IPv4查询。
	* “IPv6Enable”字段：bool，表示是否使用IPv6查询。
	* “Recursive”字段：bool，表示是否进行递归查询。
	* “Depth”字段：int32，递归查询的深度。

函数内部首先调用“lookupIPInternal”函数，该函数使用IPv4Enable和IPv6Enable字段设置查询IPv4或IPv6，然后返回查询结果。

然后，使用“hosts”字段中的“LookupIP”函数查询主机的IPv6地址。如果查询结果为nil，则返回 nil。否则，如果查询结果是一个IPv6地址，并且其家庭属于一个有效的IPv6域名，且查询的深度小于设置的深度，那么函数将返回新的IPv6地址。如果查询深度大于设置的深度，则函数将不再返回新的IPv6地址，而是返回原来的IPv6地址。


```go
// LookupIPv6 implements dns.IPv6Lookup.
func (s *Server) LookupIPv6(domain string) ([]net.IP, error) {
	return s.lookupIPInternal(domain, IPOption{
		IPv4Enable: false,
		IPv6Enable: true,
	})
}

func (s *Server) lookupStatic(domain string, option IPOption, depth int32) []net.Address {
	ips := s.hosts.LookupIP(domain, option)
	if ips == nil {
		return nil
	}
	if ips[0].Family().IsDomain() && depth < 5 {
		if newIPs := s.lookupStatic(ips[0].Domain(), option, depth+1); newIPs != nil {
			return newIPs
		}
	}
	return ips
}

```

This is a Go function that appears to handle the case where a DNS server returns only "": HTTP error codes 401, 403, 404, and 405 are commonly returned by a DNS server when an application is unable to resolve a DNS query. The function takes in a DNS query string and a DNS server address, and returns either the DNS server IP addresses or an error.

The function starts by logging any DNS queries that match the input domain using the "debug" log. If the input domain uses a DNS server that returns only "":


// If the input domain uses a DNS server that returns only "":
	return nil, newError("domain ", domain, " uses following DNS first: ", matchingDNS).AtDebug().WriteToLog()


The function then loops through each client that is registered with the DNS server and attempts to resolve the input domain using the DNS server's IP addresses. If the DNS server returns multiple IP addresses, the function returns those IP addresses. If the DNS server does not return any IP addresses, the function returns an error.

The function also checks if the DNS query has already been processed for the same domain at a previous client. If the query has already been processed for a previous client, the function returns an error with the same message as the previous error. If the query has not been processed for a previous client, the function attempts to process the query normally.

The function uses the `indices` slice to keep track of the index of the last client that attempted to resolve the query. If that index is greater than the number of registered clients, the function returns an error with the message "所有的客户端都已不可用于域名 ".

The function使用了三个错误消息：


// 401. 无法解析 DNS 查询
newError("domain ", domain, " uses following DNS first: ", matchingDNS).AtDebug().WriteToLog()

// 403. DNS 服务器未响应
newError("failed to connect to DNS 服务器", err).Base(err).WriteToLog()

// 404. DNS 查询返回结果不正确
newError("domain ", domain, " uses following DNS first: ", matchingDNS).AtDebug().WriteToLog()





```go
func toNetIP(ips []net.Address) []net.IP {
	if len(ips) == 0 {
		return nil
	}
	netips := make([]net.IP, 0, len(ips))
	for _, ip := range ips {
		netips = append(netips, ip.IP())
	}
	return netips
}

func (s *Server) lookupIPInternal(domain string, option IPOption) ([]net.IP, error) {
	if domain == "" {
		return nil, newError("empty domain name")
	}

	// normalize the FQDN form query
	if domain[len(domain)-1] == '.' {
		domain = domain[:len(domain)-1]
	}

	ips := s.lookupStatic(domain, option, 0)
	if ips != nil && ips[0].Family().IsIP() {
		newError("returning ", len(ips), " IPs for domain ", domain).WriteToLog()
		return toNetIP(ips), nil
	}

	if ips != nil && ips[0].Family().IsDomain() {
		newdomain := ips[0].Domain()
		newError("domain replaced: ", domain, " -> ", newdomain).WriteToLog()
		domain = newdomain
	}

	var lastErr error
	var matchedClient Client
	if s.domainMatcher != nil {
		indices := s.domainMatcher.Match(domain)
		domainRules := []string{}
		matchingDNS := []string{}
		for _, idx := range indices {
			info := s.matcherInfos[idx]
			rule := s.domainRules[info.clientIdx][info.domainRuleIdx]
			domainRules = append(domainRules, fmt.Sprintf("%s(DNS idx:%d)", rule, info.clientIdx))
			matchingDNS = append(matchingDNS, s.clients[info.clientIdx].Name())
		}
		if len(domainRules) > 0 {
			newError("domain ", domain, " matches following rules: ", domainRules).AtDebug().WriteToLog()
		}
		if len(matchingDNS) > 0 {
			newError("domain ", domain, " uses following DNS first: ", matchingDNS).AtDebug().WriteToLog()
		}
		for _, idx := range indices {
			clientIdx := int(s.matcherInfos[idx].clientIdx)
			matchedClient = s.clients[clientIdx]
			ips, err := s.queryIPTimeout(clientIdx, matchedClient, domain, option)
			if len(ips) > 0 {
				return ips, nil
			}
			if err == dns.ErrEmptyResponse {
				return nil, err
			}
			if err != nil {
				newError("failed to lookup ip for domain ", domain, " at server ", matchedClient.Name()).Base(err).WriteToLog()
				lastErr = err
			}
		}
	}

	for idx, client := range s.clients {
		if client == matchedClient {
			newError("domain ", domain, " at server ", client.Name(), " idx:", idx, " already lookup failed, just ignore").AtDebug().WriteToLog()
			continue
		}

		ips, err := s.queryIPTimeout(idx, client, domain, option)
		if len(ips) > 0 {
			return ips, nil
		}

		if err != nil {
			newError("failed to lookup ip for domain ", domain, " at server ", client.Name()).Base(err).WriteToLog()
			lastErr = err
		}
		if err != context.Canceled && err != context.DeadlineExceeded && err != errExpectedIPNonMatch {
			return nil, err
		}
	}

	return nil, newError("returning nil for domain ", domain).Base(lastErr)
}

```

这是一段使用Go语言编写的函数代码。函数名为“init”，定义在名为“common”的包中。

函数的作用是初始化一个名为“init”的函数。函数内部调用了另一个名为“registerConfig”的函数，并传入一个名为“nil”的参数。

“registerConfig”函数接收一个名为“ctx”的上下文对象和一个名为“config”的接口类型参数。函数内部创建了一个名为“New”的函数，并接收一个名为“ctx”的上下文对象和一个名为“config”的接口类型参数。函数内部返回一个名为“interface{}”的类型和一个名为“error”的类型的变量。

通过调用“registerConfig”函数并传入其接收到的上下文对象和接口类型参数，函数内部创建了一个名为“New”的函数，该函数返回一个名为“interface{}”的类型和一个名为“error”的类型的变量。


```go
func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		return New(ctx, config.(*Config))
	}))
}

```