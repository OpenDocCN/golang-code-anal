# go-ipfs 源码解析 27

# `core/corehttp/metrics_test.go`

这段代码是CoreHub中的一个测试，用于测试libp2p的Swarm作为客户端连接到服务器端swarm时，是否能正常工作。通过引入corehttp、testing包以及内部依赖的github.com/ipfs/kubo、github.com/libp2p/go-libp2p等库，实现了一个高可用轻量级的libp2p swarm作为服务器端网络服务，以满足各种网络应用场景的需求。


```go
package corehttp

import (
	"context"
	"testing"
	"time"

	"github.com/ipfs/kubo/core"

	inet "github.com/libp2p/go-libp2p/core/network"
	bhost "github.com/libp2p/go-libp2p/p2p/host/basic"
	swarmt "github.com/libp2p/go-libp2p/p2p/net/swarm/testing"
)

// This test is based on go-libp2p/p2p/net/swarm.TestConnectednessCorrect
```

This is a Go test that creates a context with a timeout of 100 milliseconds and a number of virtual hosts with the IP addresses `10.8.8.1, 10.8.8.2, 10.8.8.3, and 10.8.8.4`. It then uses the `bhost` package to perform DNS lookups and TCP connection attempts to these hosts.

The `bhost.BasicHost` struct represents a virtual host and has the local IP address and a network address for connecting to it. The `bhost.NewHost` function creates a new virtual host by passing the `swarmt.GenSwarm` function and then initializing it with the local IP address and a timeout of 100 milliseconds.

The `dial` function is defined to perform a TCP connection attempt to a given host and the local endpoint for the host's network. It uses the `a.DialPeer` method to connect to the remote host and the `bhost.DivulgeAddresses` method to perform the DNS lookup to get the local endpoint for the host's network.

The test initializes four virtual hosts with the IP addresses in the range `10.8.8.100` to `10.8.8.101`. It then performs a DNS lookup for each host and performs a TCP connection attempt to each host using the `dial` function.

The `IpfsNodeCollector` class is used to store the data collected by the `dial` function for each host. The `collector.PeersTotalValues` method is used to store the data collected by the `dial` function for each host's network address, including both TCP and UPDATE/QUIC connections.

The total number of peers that were found for each transport is stored in the `peersTransport` map. If the number of peers for a particular transport is less than 2, an error is raised. If the total number of peers for all transports is less than 3, an error is also raised. These tests are run for each of the four virtual hosts.


```go
// It builds 4 nodes and connects them, one being the sole center.
// Then it checks that the center reports the correct number of peers.
func TestPeersTotal(t *testing.T) {
	ctx := context.Background()

	hosts := make([]*bhost.BasicHost, 4)
	for i := 0; i < 4; i++ {
		var err error
		hosts[i], err = bhost.NewHost(swarmt.GenSwarm(t), nil)
		if err != nil {
			t.Fatal(err)
		}
	}

	dial := func(a, b inet.Network) {
		swarmt.DivulgeAddresses(b, a)
		if _, err := a.DialPeer(ctx, b.LocalPeer()); err != nil {
			t.Fatalf("Failed to dial: %s", err)
		}
	}

	dial(hosts[0].Network(), hosts[1].Network())
	dial(hosts[0].Network(), hosts[2].Network())
	dial(hosts[0].Network(), hosts[3].Network())

	// there's something wrong with dial, i think. it's not finishing
	// completely. there must be some async stuff.
	<-time.After(100 * time.Millisecond)

	node := &core.IpfsNode{PeerHost: hosts[0]}
	collector := IpfsNodeCollector{Node: node}
	peersTransport := collector.PeersTotalValues()
	if len(peersTransport) > 2 {
		t.Fatalf("expected at most 2 peers transport (tcp and upd/quic), got %d, transport map %v",
			len(peersTransport), peersTransport)
	}
	totalPeers := peersTransport["/ip4/tcp"] + peersTransport["/ip4/udp/quic-v1"]
	if totalPeers != 3 {
		t.Fatalf("expected 3 peers in either tcp or upd/quic transport, got %f", totalPeers)
	}
}

```

# `core/corehttp/mutex_profile.go`

这段代码定义了一个名为MutexFractionOption的函数，用于通过HTTP请求设置IPFS的互斥分数。

具体来说，这个函数接收一个参数"path"，代表请求的路径。然后，它通过POST请求，传递一个参数"fraction"，用于设置互斥分数。如果请求的参数格式不正确，或者参数不存在，函数会返回一个错误，并使用HTTP 400 Bad Request状态码。

如果参数正确设置，函数内部会执行以下操作：

1. 根据设定的互斥分数，从请求的参数中提取一个浮点数，并将其转换为整数。
2. 将提取的浮点数设置为设置的互斥分数，以便IPFS能够正确设置互斥分数策略。
3. 通过调用`runtime.SetMutexProfileFraction`函数，设置互斥分数策略。这个函数会在当前线程的栈上执行，所以所有后续的请求都会受到这个策略的影响。
4. 最后，函数返回一个`ServeOption`，用于告诉使用者的HTTP客户端如何处理这个请求。如果设置有误或者参数不正确，函数的返回值都是`nil`。


```go
package corehttp

import (
	"net"
	"net/http"
	"runtime"
	"strconv"

	core "github.com/ipfs/kubo/core"
)

// MutexFractionOption allows to set runtime.SetMutexProfileFraction via HTTP
// using POST request with parameter 'fraction'.
func MutexFractionOption(path string) ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
			if r.Method != http.MethodPost {
				http.Error(w, "only POST allowed", http.StatusMethodNotAllowed)
				return
			}
			if err := r.ParseForm(); err != nil {
				http.Error(w, err.Error(), http.StatusBadRequest)
				return
			}

			asfr := r.Form.Get("fraction")
			if len(asfr) == 0 {
				http.Error(w, "parameter 'fraction' must be set", http.StatusBadRequest)
				return
			}

			fr, err := strconv.Atoi(asfr)
			if err != nil {
				http.Error(w, err.Error(), http.StatusBadRequest)
				return
			}
			log.Infof("Setting MutexProfileFraction to %d", fr)
			runtime.SetMutexProfileFraction(fr)
		})

		return mux, nil
	}
}

```

这段代码定义了一个名为BlockProfileRateOption的函数，通过使用HTTP POST请求参数'rate'来设置运行时.SetBlockProfileRate。函数的实现包括以下步骤：

1. 检查请求方法是否为POST，如果不是，则返回错误信息。
2. 解析ISO-8601格式的速率参数。
3. 如果解析失败或者参数不包含速率，则返回错误信息。
4. 如果成功设置速率，则记录日志信息。
5. 调用内部函数runtime.SetBlockProfileRate设置block profile rate。
6. 返回http.ServeMux和nil表示，使得http.ServeMux接受处理函数作为其next，从而将http request路由到函数内部处理。


```go
// BlockProfileRateOption allows to set runtime.SetBlockProfileRate via HTTP
// using POST request with parameter 'rate'.
// The profiler tries to sample 1 event every <rate> nanoseconds.
// If rate == 1, then the profiler samples every blocking event.
// To disable, set rate = 0.
func BlockProfileRateOption(path string) ServeOption {
	return func(_ *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
			if r.Method != http.MethodPost {
				http.Error(w, "only POST allowed", http.StatusMethodNotAllowed)
				return
			}
			if err := r.ParseForm(); err != nil {
				http.Error(w, err.Error(), http.StatusBadRequest)
				return
			}

			rateStr := r.Form.Get("rate")
			if len(rateStr) == 0 {
				http.Error(w, "parameter 'rate' must be set", http.StatusBadRequest)
				return
			}

			rate, err := strconv.Atoi(rateStr)
			if err != nil {
				http.Error(w, err.Error(), http.StatusBadRequest)
				return
			}
			log.Infof("Setting BlockProfileRate to %d", rate)
			runtime.SetBlockProfileRate(rate)
		})
		return mux, nil
	}
}

```

# `core/corehttp/option_test.go`

这段代码定义了一个名为`testcaseCheckVersion`的测试函数，属于`corehttp`包。它通过引入`net/http`和`github.com/ipfs/kubo`包，以及定义一个名为`testcasecheckversion`的结构体，实现了对HTTP协议版本处理的功能。

具体来说，这段代码的作用是：

1. 定义了一个名为`testcasecheckversion`的结构体，该结构体包含以下字段：
	* `userAgent`：设置为用户代理（User-Agent）的操作系统、浏览器和应用程序版本号。
	* `uri`：设置为测试的API URI。
	* `shouldHandle`：设置为测试函数应该处理的情况。
	* `responseBody`：设置为预期HTTP响应的数据。
	* `responseCode`：设置为预期HTTP响应的响应状态码。
2. 通过引入`net/http`包，可以方便地使用`http.Response`类型的变量来获取HTTP响应。
3. 通过引入`github.com/ipfs/kubo`包，可以方便地使用Kubernetes对象（如`corev1.Kubernetes`）来测试HTTP请求。
4. 在函数体中，首先会设置`userAgent`字段，然后设置`uri`字段作为测试API的URL。接着设置`shouldHandle`字段为`true`，表示测试函数应该能够正确处理所有预期的情况。然后设置`responseBody`字段，用于存储预期HTTP响应的数据。最后设置`responseCode`字段，用于存储预期HTTP响应的响应状态码。
5. 调用`http.Response`类型的变量，获取期望的HTTP响应，并输出响应的状态码。如果`shouldHandle`字段为`true`，则会对响应进行处理，否则不会处理。


```go
package corehttp

import (
	"fmt"
	"io"
	"net/http"
	"net/http/httptest"
	"testing"

	version "github.com/ipfs/kubo"
)

type testcasecheckversion struct {
	userAgent    string
	uri          string
	shouldHandle bool
	responseBody string
	responseCode int
}

```

This is a Go test that simulates a HTTP request using a different version of the handler. The handler is expected to handle the request using the specified version of the handler.

The test cases are divided into two groups:

1. The first group includes requests that are expected to be handled by the handler. These requests are sent using a base URL and the handler is expected to return a 200 OK status code.
2. The second group includes requests that are expected to be handled by the handler but are sent using a non-base URL. These requests are expected to return a 404 Not Found status code.

Each test case is followed by a request to the handler using a different tool, such as Firefox and Node.js. The `APIPath` is used to store the base URL for the requests to the handler.

The simulation of the handler using `CheckVersionOption` is done using the `http.Handler` struct. The `CheckVersionOption` is used to check the HTTP version of the handler. If the version does not match the expected version, the handler is expected to return a 404 Not Found status code.


```go
func (tc testcasecheckversion) body() string {
	if !tc.shouldHandle && tc.responseBody == "" {
		return fmt.Sprintf("%s (%s != %s)\n", errAPIVersionMismatch, version.ApiVersion, tc.userAgent)
	}

	return tc.responseBody
}

func TestCheckVersionOption(t *testing.T) {
	tcs := []testcasecheckversion{
		{"/go-ipfs/0.1/", APIPath + "/test/", false, "", http.StatusBadRequest},
		{"/go-ipfs/0.1/", APIPath + "/version", true, "check!", http.StatusOK},
		{version.ApiVersion, APIPath + "/test", true, "check!", http.StatusOK},
		{"Mozilla Firefox/no go-ipfs node", APIPath + "/test", true, "check!", http.StatusOK},
		{"/go-ipfs/0.1/", "/webui", true, "check!", http.StatusOK},
	}

	for _, tc := range tcs {
		t.Logf("%#v", tc)
		r := httptest.NewRequest(http.MethodPost, tc.uri, nil)
		r.Header.Add("User-Agent", tc.userAgent) // old version, should fail

		called := false
		root := http.NewServeMux()
		mux, err := CheckVersionOption()(nil, nil, root)
		if err != nil {
			t.Fatal(err)
		}

		mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
			called = true
			if !tc.shouldHandle {
				t.Error("handler was called even though version didn't match")
			}
			if _, err := io.WriteString(w, "check!"); err != nil {
				t.Error(err)
			}
		})

		w := httptest.NewRecorder()

		root.ServeHTTP(w, r)

		if tc.shouldHandle && !called {
			t.Error("handler wasn't called even though it should have")
		}

		if w.Code != tc.responseCode {
			t.Errorf("expected code %d but got %d", tc.responseCode, w.Code)
		}

		if w.Body.String() != tc.body() {
			t.Errorf("expected error message %q, got %q", tc.body(), w.Body.String())
		}
	}
}

```

# `core/corehttp/p2p_proxy.go`

这段代码定义了一个名为“corehttp”的包，并导入了几个相关的包： core、net、net/http、net/http/httputil 和net/url。

然后，它导入了url库和strings库。

接下来，它定义了一个名为“core”的结构体，它包含一个名为“fmt”的函数和一个名为“net”的函数，分别用于输出信息和解析网络请求和响应。

然后，它定义了一个名为“peer”的结构体，它包含一个名为“net”的函数，用于定义与对等网络中的Peer通信。

接着，它定义了一个名为“p2phttp”的函数，用于在Peer之间发送和接收 HTTP 请求和响应。

最后，它定义了一个名为“protocol”的函数，用于定义Go-libp2p中的协议。


```go
package corehttp

import (
	"fmt"
	"net"
	"net/http"
	"net/http/httputil"
	"net/url"
	"strings"

	core "github.com/ipfs/kubo/core"
	peer "github.com/libp2p/go-libp2p/core/peer"

	p2phttp "github.com/libp2p/go-libp2p-http"
	protocol "github.com/libp2p/go-libp2p/core/protocol"
)

```

这段代码定义了一个名为 P2PProxyOption 的 P2PProxyOption 函数，它接受一个名为 ServeOption 的函数参数，用于将一个 HTTP 请求代理到另一个 IPFS（InterPlanetary File System） peer。

函数实现中，首先定义了一个名为 P2PProxyOption 的函数，它接收一个 IpfsNode 类型的参数，表示一个 IPFS 节点，以及一个 net.Listener 类型的参数，表示一个监听套接字，和一个 HTTP.ServeMux 类型的参数，用于处理代理的 HTTP 请求。

函数内部，首先通过 parseRequest 函数解析 HTTP 请求，并获取其目标 URL 和目标 IPFS peer。然后，通过 url.Parse 函数将目标 IPFS peer 解析为一个 URL。

接下来，通过 p2phttp.NewTransport 和httputil.NewSingleHostReverseProxy 函数创建两个套接字，并将它们用于代理 HTTP 请求。最后，将代理的 HTTP 请求发送到目标 URL。

如果解析或创建套接字时出现错误，函数将返回一个错误并输出错误信息。


```go
// P2PProxyOption is an endpoint for proxying a HTTP request to another ipfs peer
func P2PProxyOption() ServeOption {
	return func(ipfsNode *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		mux.HandleFunc("/p2p/", func(w http.ResponseWriter, request *http.Request) {
			// parse request
			parsedRequest, err := parseRequest(request)
			if err != nil {
				handleError(w, "failed to parse request", err, 400)
				return
			}

			request.Host = "" // Let URL's Host take precedence.
			request.URL.Path = parsedRequest.httpPath
			target, err := url.Parse(fmt.Sprintf("libp2p://%s", parsedRequest.target))
			if err != nil {
				handleError(w, "failed to parse url", err, 400)
				return
			}

			rt := p2phttp.NewTransport(ipfsNode.PeerHost, p2phttp.ProtocolOption(parsedRequest.name))
			proxy := httputil.NewSingleHostReverseProxy(target)
			proxy.Transport = rt
			proxy.ServeHTTP(w, request)
		})
		return mux, nil
	}
}

```

这段代码定义了一个名为 `proxyRequest` 的结构体类型，用于表示代理请求。该结构体类型包含三个字段：`target`、`name` 和 `httpPath`。

`target` 字段表示目标代理的服务器地址。

`name` 字段表示发送代理请求时使用的用户名。

`httpPath` 字段表示目标代理的 HTTP 路径，用于将请求发送到代理服务器。

该函数 `parseRequest` 接收一个 `http.Request` 类型的参数，并返回一个 `proxyRequest` 类型的结构体。如果解析失败，函数将返回 `nil`。

函数解析请求路径：

1. 使用 `path` 字段的 `/` 符号将请求路径分割为五个子路径段。

2. 如果请求路径小于 5 个子路径段，将返回 `nil`。

3. 解析第 5 个子路径段，尝试将其转换为 `protocol.ID` 类型。如果是 `"http"`，则返回一个代理请求的结构体，包含目标服务器地址、用户名和 HTTP 路径。

4. 如果第 5 个子路径段不是 `"http"`，或者不是有效的 `protocol.ID`，函数将返回 `nil`。

5. 如果解析成功，函数将返回代理请求结构体。


```go
type proxyRequest struct {
	target   string
	name     protocol.ID
	httpPath string // path to send to the proxy-host
}

// from the url path parse the peer-ID, name and http path
// /p2p/$peer_id/http/$http_path
// or
// /p2p/$peer_id/x/$protocol/http/$http_path
func parseRequest(request *http.Request) (*proxyRequest, error) {
	path := request.URL.Path

	split := strings.SplitN(path, "/", 5)
	if len(split) < 5 {
		return nil, fmt.Errorf("invalid request path '%s'", path)
	}

	if _, err := peer.Decode(split[2]); err != nil {
		return nil, fmt.Errorf("invalid request path '%s'", path)
	}

	if split[3] == "http" {
		return &proxyRequest{split[2], protocol.ID("/http"), split[4]}, nil
	}

	split = strings.SplitN(path, "/", 7)
	if len(split) < 7 || split[3] != "x" || split[5] != "http" {
		return nil, fmt.Errorf("invalid request path '%s'", path)
	}

	return &proxyRequest{split[2], protocol.ID("/x/" + split[4] + "/http"), split[6]}, nil
}

```

这段代码定义了一个名为 "handleError" 的函数，接受四个参数：

1. 一个名为 "w" 的 HTTP 响应写入器（http.ResponseWriter）类型的变量。
2. 一个字符串类型的变量 "msg"，用于表示错误消息。
3. 一个名为 "err" 的错误类型的变量。
4. 一个整数类型的变量 "code"，用于表示 HTTP 状态码。

函数的作用是当 "handleError" 函数接收到一个 HTTP 请求时，根据传入的参数，输出一条错误消息，并设置 HTTP 状态码。


```go
func handleError(w http.ResponseWriter, msg string, err error, code int) {
	http.Error(w, fmt.Sprintf("%s: %s", msg, err), code)
}

```

# `core/corehttp/p2p_proxy_test.go`

该代码的作用是定义一个名为`TestCase`的结构体，它代表了所有可能的测试用例。

`package corehttp`导入了一些与HTTP客户端和Kubernetes相关的包。

`import (`net/http`和`strings`)用于导入HTTP客户端和字符串操作的相关功能。

`import protocol`导入了一个名为`protocol`的接口，它定义了HTTP客户端和服务器之间的通信规则。

然后，定义了一个`TestCase`结构体，其中包含以下字段：

- `urlprefix`字段是一个字符串，用于指定一个前缀，用于将所有的测试请求路由分组。
- `target`字段是一个字符串，表示要测试的API的URL。
- `name`字段是一个字符串，用于为测试用例命名。
- `path`字段是一个字符串，用于指定测试请求的路径。

最后，导入了`assert`包，以便在测试中使用断言。


```go
package corehttp

import (
	"net/http"
	"strings"
	"testing"

	"github.com/ipfs/kubo/thirdparty/assert"

	protocol "github.com/libp2p/go-libp2p/core/protocol"
)

type TestCase struct {
	urlprefix string
	target    string
	name      string
	path      string
}

```

这段代码定义了一个名为 `validtestCases` 的数组，其中包含了四个 `TestCase` 结构体，用于测试 HTTP 代理的使用。每个 `TestCase` 包含三个字段：`uri`、`name` 和 `path`，分别表示 HTTP 代理的 URL 前缀、请求名称和请求路径。

函数 `TestParseRequest` 接收一个 `validtestCases` 数组，并遍历其中的每个 `TestCase`。对于每个 `TestCase`，创建一个 HTTP 请求对象 `req`，并调用 `parseRequest` 函数解析 `req`。如果解析失败，则程序输出错误，否则输出解析结果。

`parseRequest` 函数接收一个 HTTP 请求对象 `req`，其中包括 HTTP 方法、URL 和请求头。函数将解析 `req`，并返回两个值：`parsed` 和 `err`。`parsed` 表示 HTTP 路径，`err` 表示解析是否成功。


```go
var validtestCases = []TestCase{
	{"http://localhost:5001", "QmT8JtU54XSmC38xSb1XHFSMm775VuTeajg7LWWWTAwzxT", "/http", "path/to/index.txt"},
	{"http://localhost:5001", "QmT8JtU54XSmC38xSb1XHFSMm775VuTeajg7LWWWTAwzxT", "/x/custom/http", "path/to/index.txt"},
	{"http://localhost:5001", "QmT8JtU54XSmC38xSb1XHFSMm775VuTeajg7LWWWTAwzxT", "/x/custom/http", "http/path/to/index.txt"},
}

func TestParseRequest(t *testing.T) {
	for _, tc := range validtestCases {
		url := tc.urlprefix + "/p2p/" + tc.target + tc.name + "/" + tc.path
		req, _ := http.NewRequest(http.MethodGet, url, strings.NewReader(""))

		parsed, err := parseRequest(req)
		if err != nil {
			t.Fatal(err)
		}
		assert.True(parsed.httpPath == tc.path, t, "proxy request path")
		assert.True(parsed.name == protocol.ID(tc.name), t, "proxy request name")
		assert.True(parsed.target == tc.target, t, "proxy request peer-id")
	}
}

```

这段代码是一个用于测试 `http.Request.ParseRequest` 函数的 Go 语言程序。其作用是针对给定的测试用例数组（invalidtestCases）中的每个测试用例，构造一个 HTTP GET 请求，并使用自定义代理（url）进行访问，然后测试 parseRequest 函数是否会在请求解析时出现错误。如果所有测试用例均成功，则表示函数正常工作，如果出现错误，则需要进一步排查错误原因。

具体来说，这段代码首先定义了一个字符串数组 `invalidtestCases`，其中包含多个测试用例。接下来，定义了一个名为 `TestParseRequestInvalidPath` 的函数，该函数接收一个字符串数组 `invalidtestCases`，然后使用一个循环遍历每个测试用例。

在每次遍历中，首先创建一个 HTTP GET 请求，设置请求方法为 `http.MethodGet`，并设置请求的目标 URL。接着，使用字符串模板 `strings.NewReader` 创建一个 HTTP GET 请求的请求体，并使用 `http.Client.DefaultClient` 设置代理设置为 `url`。然后，调用 `parseRequest` 函数处理请求。

在 `parseRequest` 函数中，由于 `url` 已经被设置为 `url`，所以 `parseRequest` 函数会直接解析请求，而不会产生新的错误。如果 `parseRequest` 函数在解析请求时产生错误，那么错误的堆栈信息（或者错误类型）将会被输出，这样就可以根据错误信息进一步诊断问题所在。

最后，在 `TestParseRequestInvalidPath` 函数中，如果所有测试用例均成功，那么程序将会正常退出。否则，将输出一个错误，并给出可能的错误信息，以便开发人员进一步排查问题。


```go
var invalidtestCases = []string{
	"http://localhost:5001/p2p/http/foobar",
	"http://localhost:5001/p2p/QmT8JtU54XSmC38xSb1XHFSMm775VuTeajg7LWWWTAwzxT/x/custom/foobar",
}

func TestParseRequestInvalidPath(t *testing.T) {
	for _, tc := range invalidtestCases {
		url := tc
		req, _ := http.NewRequest(http.MethodGet, url, strings.NewReader(""))

		_, err := parseRequest(req)
		if err == nil {
			t.Fail()
		}
	}
}

```

# `core/corehttp/redirect.go`

这段代码定义了一个名为"RedirectOption"的函数，用于设置HTTP重定向选项。函数接受两个参数，一个是路径参数，另一个是重定向字符串。函数返回一个ServeOption函数，用于设置HTTP请求上下文。

具体来说，代码首先导入了两个必要的库：net和net/http。然后导入了一个名为core的库，可能是为了能够在函数中使用该库中的定义。接着定义了一个名为RedirectOption的函数，该函数使用NetCore中的几个函数来设置HTTP重定向选项。函数内部包含一个名为handler的参数，该参数是一个redirectHandler类型的变量，该变量实现了net/http.Handler，用于处理HTTP请求。

函数的具体实现包括以下几个步骤：

1. 设置API头，告诉NetCore将所有请求的HTTP头中的一致字段设置为redirect头中的值。

2. 如果请求路径参数为空，则默认重定向到根路径，否则设置重定向路径。

3. 创建一个redirectHandler实例，并将其设置为当前处理请求的上下文中的handler。

4. 最后，将handler设置为请求的上下文中的handler，并返回该上下文，同时不执行任何其他操作。


```go
package corehttp

import (
	"net"
	"net/http"

	core "github.com/ipfs/kubo/core"
)

func RedirectOption(path string, redirect string) ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		cfg, err := n.Repo.Config()
		if err != nil {
			return nil, err
		}

		handler := &redirectHandler{redirect, cfg.API.HTTPHeaders}

		if len(path) > 0 {
			mux.Handle("/"+path+"/", handler)
		} else {
			mux.Handle("/", handler)
		}
		return mux, nil
	}
}

```

这段代码定义了一个名为`redirectHandler`的结构体，它包含一个路径`path`和一个包含键值对`headers`的`headers`Map`字段。

` ServeHTTP`函数是此结构的`redirectHandler`类型的别名，它接受一个`ResponseWriter`类型的参数`w`和一个`Request`类型的参数`r`。该函数的作用是在处理HTTP请求时，根据传递的`path`和`headers`字段生成一个HTTP响应。

具体而言，函数内部首先遍历`headers`字段，将所有键值对存储在`headers`Map中的值遍历并添加到`w.Header()`上。然后使用`http.Redirect`函数将生成的HTTP重定向响应发送回给客户端，同时将`path`和`status`字段设置为客户端请求的路径和HTTP状态码。


```go
type redirectHandler struct {
	path    string
	headers map[string][]string
}

func (i *redirectHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	for k, v := range i.headers {
		w.Header()[http.CanonicalHeaderKey(k)] = v
	}

	http.Redirect(w, r, i.path, http.StatusFound)
}

```

# `core/corehttp/routing.go`

这段代码定义了一个名为 "corehttp" 的包。它导入了以下依赖项：

- "net" 和 "net/http"，用于创建 HTTP 客户端和处理 HTTP 请求。
- "github.com/ipfs/boxo/ipns"，用于与 IPFS 网络进行交互。
- "github.com/ipfs/boxo/routing/http/server"，用于设置 HTTP 服务器。
- "github.com/ipfs/boxo/routing/http/types" 和 "github.com/ipfs/boxo/routing/http/types/iter"，用于设置 HTTP 请求和响应的类型和遍历。
- "github.com/ipfs/boxo/routing/http"，用于设置 HTTP 路由。
- "github.com/ipfs/go-cid"，用于与 CID 网络进行交互。
- "github.com/ipfs/kubo/core"，用于与 IPFS 库 "kubo" 进行交互。
- "github.com/libp2p/go-libp2p/core/peer"，用于与 LIBP2P 网络中的对等连接进行交互。
- "github.com/libp2p/go-libp2p/core/routing"，用于设置路由。

综上所述，此代码定义了一个 HTTP 服务器，可以处理 IPFS 网络中的请求，并将其路由到相应的处理程序。


```go
package corehttp

import (
	"context"
	"net"
	"net/http"
	"time"

	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/routing/http/server"
	"github.com/ipfs/boxo/routing/http/types"
	"github.com/ipfs/boxo/routing/http/types/iter"
	cid "github.com/ipfs/go-cid"
	core "github.com/ipfs/kubo/core"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
)

```

该代码定义了一个名为 "RoutingOption" 的函数，用于路由到指定 URL 并返回一个 HTTP Server。该函数接收三个参数：一个 IPFS 节点对象、一个 Listener 对象和一个 HTTP Server Mux 对象。该函数使用一个名为 server 的函数实现了一个 HTTP 服务器，这个函数在加载了一个路由配置文件后，使用它来处理请求。

函数返回一个 Mux 对象，代表 HTTP 服务器，然后使用 `mux.Handle` 方法将路由配置文件中的路由注册到 HTTP 服务器上。最后，函数返回 Mux 对象和 nil，表示成功完成了路由配置。


```go
func RoutingOption() ServeOption {
	return func(n *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
		handler := server.Handler(&contentRouter{n})
		mux.Handle("/routing/v1/", handler)
		return mux, nil
	}
}

type contentRouter struct {
	n *core.IpfsNode
}

func (r *contentRouter) FindProviders(ctx context.Context, key cid.Cid, limit int) (iter.ResultIter[types.Record], error) {
	ctx, cancel := context.WithCancel(ctx)
	ch := r.n.Routing.FindProvidersAsync(ctx, key, limit)
	return iter.ToResultIter[types.Record](&peerChanIter{
		ch:     ch,
		cancel: cancel,
	}), nil
}

```

这段代码定义了两个函数，分别是 `ProvideBitswap` 和 `FindPeers`。这两个函数的作用如下：

1. `ProvideBitswap` 函数接收一个 `server.BitswapWriteProvideRequest` 类型的参数，并返回一个 `time.Duration` 类型的值，或者是错误。

2. `FindPeers` 函数接收一个 `peer.ID` 类型的参数、一个 `int` 类型的参数，并返回一个迭代器 `iter.ResultIter` 类型的结果，或者是错误。

具体来说，这两个函数的实现比较简单，主要涉及到对路由器网络的查询和一些网络请求的发送。

`ProvideBitswap` 函数的作用是提供一些 Bitswap 写入服务，但是它的实现比较复杂，需要通过 `nolint` 插件进行依赖管理，因此在生产环境中可能会被禁止使用。

`FindPeers` 函数的作用是查询与指定 `peer.ID` 相关的路由器，并返回一个包含多个 `types.PeerRecord` 类型的结果。它使用了 `r.n.Routing` 切片，这个切片可能包含了一些已查询过的路由器，因此可以通过它来查询最新的路由器。


```go
// nolint deprecated
func (r *contentRouter) ProvideBitswap(ctx context.Context, req *server.BitswapWriteProvideRequest) (time.Duration, error) {
	return 0, routing.ErrNotSupported
}

func (r *contentRouter) FindPeers(ctx context.Context, pid peer.ID, limit int) (iter.ResultIter[*types.PeerRecord], error) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	addr, err := r.n.Routing.FindPeer(ctx, pid)
	if err != nil {
		return nil, err
	}

	rec := &types.PeerRecord{
		Schema: types.SchemaPeer,
		ID:     &addr.ID,
	}

	for _, addr := range addr.Addrs {
		rec.Addrs = append(rec.Addrs, types.Multiaddr{Multiaddr: addr})
	}

	return iter.ToResultIter[*types.PeerRecord](iter.FromSlice[*types.PeerRecord]([]*types.PeerRecord{rec})), nil
}

```

这两函数的作用是用于获取和设置IPNS（命名空间）的值。

第一个函数：

1. 获取IPNS命名空间：根据传入的名称参数，使用r.n.Routing.GetValue(ctx, string(name.RoutingKey()))方法获取。
2. 返回IPNS命名空间对应的Record值：将获取到的命名空间名称映射为相应的Record类型。

第二个函数：

1. 创建一个名为"ipns.Name"的命名空间：通过设置r.n.Routing.SetValue(ctx, string(name.RoutingKey()), ipns.Name)来创建。
2. 设置IPNS命名空间对应的Record：将获取到的Record类型设置给传入的Record参数。
3. 返回设置是否成功：如果设置成功，则返回 nil，否则返回错误。


```go
func (r *contentRouter) GetIPNS(ctx context.Context, name ipns.Name) (*ipns.Record, error) {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	raw, err := r.n.Routing.GetValue(ctx, string(name.RoutingKey()))
	if err != nil {
		return nil, err
	}

	return ipns.UnmarshalRecord(raw)
}

func (r *contentRouter) PutIPNS(ctx context.Context, name ipns.Name, record *ipns.Record) error {
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	raw, err := ipns.MarshalRecord(record)
	if err != nil {
		return err
	}

	// The caller guarantees that name matches the record. This is double checked
	// by the internals of PutValue.
	return r.n.Routing.PutValue(ctx, string(name.RoutingKey()), raw)
}

```

该代码定义了一个名为`peerChanIter`的结构体，它包含一个通道`ch`，一个取消上下文`cancel`，和一个指向`peer.AddrInfo`类型的变量`next`。

`peerChanIter`类型代表了一个`peer.AddrInfo`类型中的通道，允许通过通道发送`peer.AddrInfo`类型的数据。

`ch`通道是一个用于接收`peer.AddrInfo`类型的数据的通道。

`cancel`是一个取消上下文，允许函数在接收到取消信号时停止执行。

`next`是一个指向`peer.AddrInfo`类型的指针，用于跟踪下一个`peer.AddrInfo`类型数据的时间。

该代码的作用是定义了一个通道`peerChanIter`，允许通过该通道发送`peer.AddrInfo`类型的数据，并可阻止对该通道的发送操作。


```go
type peerChanIter struct {
	ch     <-chan peer.AddrInfo
	cancel context.CancelFunc
	next   *peer.AddrInfo
}

func (it *peerChanIter) Next() bool {
	addr, ok := <-it.ch
	if ok {
		it.next = &addr
		return true
	}
	it.next = nil
	return false
}

```

该函数接收一个名为 `peerChanIter` 的迭代器作为参数，并返回一个名为 `types.Record` 的类型。函数的作用是将传入的迭代器中的每个元素添加到输出 `types.PeerRecord` 类型中。

具体来说，函数首先检查传入的迭代器是否为 `nil`，如果是，则返回 `nil`。否则，函数创建一个名为 `types.PeerRecord` 的 `types.Record` 类型，其中 `ID` 字段设置为传入的迭代器元素的 `ID`,`Schema` 字段设置为传入的迭代器元素的 `Schema`。

接着，函数遍历传入的迭代器中的每个元素，并将它们的 `Multiaddr` 字段添加到 `PeerRecord` 类型中的 `Addrs` 字段中。这样，函数返回的 `types.PeerRecord` 类型中包含了输入迭代器中的所有元素。


```go
func (it *peerChanIter) Val() types.Record {
	if it.next == nil {
		return nil
	}

	rec := &types.PeerRecord{
		Schema: types.SchemaPeer,
		ID:     &it.next.ID,
	}

	for _, addr := range it.next.Addrs {
		rec.Addrs = append(rec.Addrs, types.Multiaddr{Multiaddr: addr})
	}

	return rec
}

```

该函数名为`func`，其作用是接受一个`peerChanIter`类型的参数。

在该函数中，首先使用`it.cancel()`方法取消当前`peerChanIter`迭代器的订阅。

接着，该函数返回一个`nil`类型的值。

综合来看，该函数的作用是取消当前`peerChanIter`迭代器的订阅，并返回一个`nil`类型的值。


```go
func (it *peerChanIter) Close() error {
	it.cancel()
	return nil
}

```

# `core/corehttp/webui.go`

IPFS(InterPlanetary File System) is a decentralized, distributed file system that allows users to store and share files in a trustless manner over the internet. It is designed to be a peer-to-peer network, so that users can store and access their own files without relying on a centralized server or intermediary.

IPFS uses a cryptographic哈希函数(如RIPEMD-2041)来验证文件的完整性，但它并不会对文件进行加密。IPFS支持快速、安全地传输大型文件，并且可以有效地处理大量数据的并发访问。此外，IPFS还支持离线下载和上传，以及增量下载。

IPFS 的最主要特点是去中心化、分布式的。它是通过一个网络中的节点来存储和传输文件的，所以可以认为它是非中心化的。此外，由于它没有集中式的服务器，所以也避免了单点故障。


```go
package corehttp

// TODO: move to IPNS
const WebUIPath = "/ipfs/bafybeiamycmd52xvg6k3nzr6z3n33de6a2teyhquhj4kspdtnvetnkrfim" // v4.1.1

// WebUIPaths is a list of all past webUI paths.
var WebUIPaths = []string{
	WebUIPath,
	"/ipfs/bafybeieqdeoqkf7xf4aozd524qncgiloh33qgr25lyzrkusbcre4c3fxay",
	"/ipfs/bafybeicyp7ssbnj3hdzehcibmapmpuc3atrsc4ch3q6acldfh4ojjdbcxe",
	"/ipfs/bafybeigs6d53gpgu34553mbi5bbkb26e4ikruoaaar75jpfdywpup2r3my",
	"/ipfs/bafybeic4gops3d3lyrisqku37uio33nvt6fqxvkxihrwlqsuvf76yln4fm",
	"/ipfs/bafybeifeqt7mvxaniphyu2i3qhovjaf3sayooxbh5enfdqtiehxjv2ldte",
	"/ipfs/bafybeiequgo72mrvuml56j4gk7crewig5bavumrrzhkqbim6b3s2yqi7ty",
	"/ipfs/bafybeibjbq3tmmy7wuihhhwvbladjsd3gx3kfjepxzkq6wylik6wc3whzy",
	"/ipfs/bafybeiavrvt53fks6u32n5p2morgblcmck4bh4ymf4rrwu7ah5zsykmqqa",
	"/ipfs/bafybeiageaoxg6d7npaof6eyzqbwvbubyler7bq44hayik2hvqcggg7d2y",
	"/ipfs/bafybeidb5eryh72zajiokdggzo7yct2d6hhcflncji5im2y5w26uuygdsm",
	"/ipfs/bafybeibozpulxtpv5nhfa2ue3dcjx23ndh3gwr5vwllk7ptoyfwnfjjr4q",
	"/ipfs/bafybeiednzu62vskme5wpoj4bjjikeg3xovfpp4t7vxk5ty2jxdi4mv4bu",
	"/ipfs/bafybeihcyruaeza7uyjd6ugicbcrqumejf6uf353e5etdkhotqffwtguva",
	"/ipfs/bafybeiflkjt66aetfgcrgvv75izymd5kc47g6luepqmfq6zsf5w6ueth6y",
	"/ipfs/bafybeid26vjplsejg7t3nrh7mxmiaaxriebbm4xxrxxdunlk7o337m5sqq",
	"/ipfs/bafybeif4zkmu7qdhkpf3pnhwxipylqleof7rl6ojbe7mq3fzogz6m4xk3i",
	"/ipfs/bafybeianwe4vy7sprht5sm3hshvxjeqhwcmvbzq73u55sdhqngmohkjgs4",
	"/ipfs/bafybeicitin4p7ggmyjaubqpi3xwnagrwarsy6hiihraafk5rcrxqxju6m",
	"/ipfs/bafybeihpetclqvwb4qnmumvcn7nh4pxrtugrlpw4jgjpqicdxsv7opdm6e",
	"/ipfs/bafybeibnnxd4etu4tq5fuhu3z5p4rfu3buabfkeyr3o3s4h6wtesvvw6mu",
	"/ipfs/bafybeid6luolenf4fcsuaw5rgdwpqbyerce4x3mi3hxfdtp5pwco7h7qyq",
	"/ipfs/bafybeigkbbjnltbd4ewfj7elajsbnjwinyk6tiilczkqsibf3o7dcr6nn4",
	"/ipfs/bafybeicp23nbcxtt2k2twyfivcbrc6kr3l5lnaiv3ozvwbemtrb7v52r6i",
	"/ipfs/bafybeidatpz2hli6fgu3zul5woi27ujesdf5o5a7bu622qj6ugharciwjq",
	"/ipfs/QmfQkD8pBSBCBxWEwFSu4XaDVSWK6bjnNuaWZjMyQbyDub",
	"/ipfs/QmXc9raDM1M5G5fpBnVyQ71vR4gbnskwnB9iMEzBuLgvoZ",
	"/ipfs/QmenEBWcAk3tN94fSKpKFtUMwty1qNwSYw3DMDFV6cPBXA",
	"/ipfs/QmUnXcWZC5Ve21gUseouJsH5mLAyz5JPp8aHsg8qVUUK8e",
	"/ipfs/QmSDgpiHco5yXdyVTfhKxr3aiJ82ynz8V14QcGKicM3rVh",
	"/ipfs/QmRuvWJz1Fc8B9cTsAYANHTXqGmKR9DVfY5nvMD1uA2WQ8",
	"/ipfs/QmQLXHs7K98JNQdWrBB2cQLJahPhmupbDjRuH1b9ibmwVa",
	"/ipfs/QmXX7YRpU7nNBKfw75VG7Y1c3GwpSAGHRev67XVPgZFv9R",
	"/ipfs/QmXdu7HWdV6CUaUabd9q2ZeA4iHZLVyDRj3Gi4dsJsWjbr",
	"/ipfs/QmaaqrHyAQm7gALkRW8DcfGX3u8q9rWKnxEMmf7m9z515w",
	"/ipfs/QmSHDxWsMPuJQKWmVA1rB5a3NX2Eme5fPqNb63qwaqiqSp",
	"/ipfs/QmctngrQAt9fjpQUZr7Bx3BsXUcif52eZGTizWhvcShsjz",
	"/ipfs/QmS2HL9v5YeKgQkkWMvs1EMnFtUowTEdFfSSeMT4pos1e6",
	"/ipfs/QmR9MzChjp1MdFWik7NjEjqKQMzVmBkdK3dz14A6B5Cupm",
	"/ipfs/QmRyWyKWmphamkMRnJVjUTzSFSAAZowYP4rnbgnfMXC9Mr",
	"/ipfs/QmU3o9bvfenhTKhxUakbYrLDnZU7HezAVxPM6Ehjw9Xjqy",
	"/ipfs/QmPhnvn747LqwPYMJmQVorMaGbMSgA7mRRoyyZYz3DoZRQ",
	"/ipfs/QmQNHd1suZTktPRhP7DD4nKWG46ZRSxkwHocycHVrK3dYW",
	"/ipfs/QmNyMYhwJUS1cVvaWoVBhrW8KPj1qmie7rZcWo8f1Bvkhz",
	"/ipfs/QmVTiRTQ72qiH4usAGT4c6qVxCMv4hFMUH9fvU6mktaXdP",
	"/ipfs/QmYcP4sp1nraBiCYi6i9kqdaKobrK32yyMpTrM5JDA8a2C",
	"/ipfs/QmUtMmxgHnDvQq4bpH6Y9MaLN1hpfjJz5LZcq941BEqEXs",
	"/ipfs/QmPURAjo3oneGH53ovt68UZEBvsc8nNmEhQZEpsVEQUMZE",
	"/ipfs/QmeSXt32frzhvewLKwA1dePTSjkTfGVwTh55ZcsJxrCSnk",
	"/ipfs/QmcjeTciMNgEBe4xXvEaA4TQtwTRkXucx7DmKWViXSmX7m",
	"/ipfs/QmfNbSskgvTXYhuqP8tb9AKbCkyRcCy3WeiXwD9y5LeoqK",
	"/ipfs/QmPkojhjJkJ5LEGBDrAvdftrjAYmi9GU5Cq27mWvZTDieW",
	"/ipfs/Qmexhq2sBHnXQbvyP2GfUdbnY7HCagH2Mw5vUNSBn2nxip",
}

```

这段代码定义了一个名为 "WebUIOption" 的 RedirectOption 类，它可能是用于 WebUI(Web 用户界面) 的选项之一。

具体来说，它接收一个参数 "webui"和一个参数 "WebUIPath"，然后根据这些参数返回一个指向 RedirectOption 的对象。RedirectOption 可能是用于在 WebUI 中进行选项重定向的类。

因此，这段代码的作用是创建一个 RedirectOption 对象，用于在 WebUI 中进行选项重定向。选项重定向的具体实现可能会因 WebUI 的具体实现而有所不同。


```go
var WebUIOption = RedirectOption("webui", WebUIPath)

```

# `core/corerepo/gc.go`

这段代码定义了一个名为“corerepo”的包，它提供了在IPFS（InterPlanetary File System）中执行操作的函数。IPFS是一个去中心化的点对点分布式文件系统，可以在全球范围内存储和共享文件。

下面是每个函数的摘要：


1. 初始化函数：通过调用IPFS的“repo.Init”函数，创建一个新的IPFS根目录（repo），并返回根目录的路径。


2. 创建文件函数：通过调用IPFS的“repo.File”函数，创建一个新的文件并返回其对象的指针。


3. 读取文件函数：通过调用IPFS的“repo.File”函数，读取文件并提供给用户进行后续操作。


4. 修改文件函数：通过调用IPFS的“repo.File”函数，修改文件并提供给用户进行后续操作。


5. 删除文件函数：通过调用IPFS的“repo.File”函数，删除指定的文件并提供给用户进行后续操作。


6. 目录操作函数：通过调用IPFS的“repo.File”函数，递归地操作目录并提供给用户进行后续操作。


7. 错误处理函数：通过定义一个名为“Error”的错误类型，在执行错误操作时捕获并提供给用户进行确认。


8. 打印日志函数：通过定义一个名为“Logger”的日志类型，在执行操作时记录操作日志。


9. 格式化字符串函数：通过定义一个名为“ humanize”的函数，对字符串进行格式化并提供给用户进行确认。


10. 从命令行读取参数函数：通过定义一个名为“ ParseArgs”的函数，从命令行读取指定参数。


11. 将IPFS路径解析为字符串函数：通过定义一个名为“ ParseIPFSPath”的函数，将IPFS路径解析为字符串并提供给用户进行确认。


12. 将IPFS路径解析为核心根目录函数：通过定义一个名为“ GetIPFSRoot”的函数，将IPFS路径解析为核心根目录并提供给用户进行确认。


13. 创建文件系统函数：通过调用“mfs.NewFileSystem”函数，创建一个新的文件系统。


14. 关闭文件系统函数：通过调用“mfs.CloseFileSystem”函数，关闭指定的文件系统。


15. 初始化日志函数：通过定义一个名为“ InitLog”的函数，初始化日志记录器。


16. 记录请求日志函数：通过定义一个名为“ RecordRequestLog”的函数，记录请求的日志信息。


17. 记录响应日志函数：通过定义一个名为“ RecordResponseLog”的函数，记录响应的日志信息。


18. 初始化IPFS根目录函数：通过调用“repo.Init”函数，初始化IPFS根目录。




```go
package corerepo

import (
	"bytes"
	"context"
	"errors"
	"time"

	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/gc"
	"github.com/ipfs/kubo/repo"

	"github.com/dustin/go-humanize"
	"github.com/ipfs/boxo/mfs"
	"github.com/ipfs/go-cid"
	logging "github.com/ipfs/go-log"
)

```

This is a Go function that creates a new instance of a Gathering Hub, which is a piece of the IPFS (InterPlanetary File System) ecosystem. This Gathering Hub has a storage limit, a storage growth ceiling, and a slack space for the storage limit.

The function takes in an IPFSNode object from the core.IpfsNode struct, which represents an IPFS node. It also takes a reference to a repo, which is an instance of the repo.repo struct. This repo is used to interact with the local IPFS storage.

The function sets the initial values for the Gathering Hub's configuration parameters. First, it sets the maximum storage limit to a value of 10GB, which is the default value for the StorageMax configuration key. Next, it sets the watermark for the storage generation to 90, which is the default value for the StorageGCWatermark configuration key. Finally, it sets the slack space to 1GB, which is calculated as 10GB minus the storage generation watermark.

If any of the configuration keys are not present in the repo, the function will panics with an error. If any of the configuration values are invalid, the function will also panics with an error.

The function returns a pointer to a new Gathering Hub instance, as well as an error if any problems occurred during initialization.


```go
var log = logging.Logger("corerepo")

var ErrMaxStorageExceeded = errors.New("maximum storage limit exceeded. Try to unpin some files")

type GC struct {
	Node       *core.IpfsNode
	Repo       repo.Repo
	StorageMax uint64
	StorageGC  uint64
	SlackGB    uint64
	Storage    uint64
}

func NewGC(n *core.IpfsNode) (*GC, error) {
	r := n.Repo
	cfg, err := r.Config()
	if err != nil {
		return nil, err
	}

	// check if cfg has these fields initialized
	// TODO: there should be a general check for all of the cfg fields
	// maybe distinguish between user config file and default struct?
	if cfg.Datastore.StorageMax == "" {
		if err := r.SetConfigKey("Datastore.StorageMax", "10GB"); err != nil {
			return nil, err
		}
		cfg.Datastore.StorageMax = "10GB"
	}
	if cfg.Datastore.StorageGCWatermark == 0 {
		if err := r.SetConfigKey("Datastore.StorageGCWatermark", 90); err != nil {
			return nil, err
		}
		cfg.Datastore.StorageGCWatermark = 90
	}

	storageMax, err := humanize.ParseBytes(cfg.Datastore.StorageMax)
	if err != nil {
		return nil, err
	}
	storageGC := storageMax * uint64(cfg.Datastore.StorageGCWatermark) / 100

	// calculate the slack space between StorageMax and StorageGCWatermark
	// used to limit GC duration
	slackGB := (storageMax - storageGC) / 10e9
	if slackGB < 1 {
		slackGB = 1
	}

	return &GC{
		Node:       n,
		Repo:       r,
		StorageMax: storageMax,
		StorageGC:  storageGC,
		SlackGB:    slackGB,
	}, nil
}

```

这两段代码定义了两个函数：`BestEffortRoots` 和 `GarbageCollect`。它们的作用如下：

1. `BestEffortRoots` 函数的作用是查找所有具有特定 `cid.Cid` 值的文件根。它返回了一组文件根的 `cid.Cid` 值，如果返回过程中出现错误，则返回一个空数组和错误。

2. `GarbageCollect` 函数的作用是收集指定 `core.IpfsNode` 实例的垃圾回收。它首先调用 `BestEffortRoots` 函数查找所有文件根，然后使用 `gc.GC` 函数将垃圾回收。`gc.GC` 函数负责回收内存中的数据，并将其释放到垃圾回收器。最后，它将收集结果返回。


```go
func BestEffortRoots(filesRoot *mfs.Root) ([]cid.Cid, error) {
	rootDag, err := filesRoot.GetDirectory().GetNode()
	if err != nil {
		return nil, err
	}

	return []cid.Cid{rootDag.Cid()}, nil
}

func GarbageCollect(n *core.IpfsNode, ctx context.Context) error {
	roots, err := BestEffortRoots(n.FilesRoot)
	if err != nil {
		return err
	}
	rmed := gc.GC(ctx, n.Blockstore, n.Repo.Datastore(), n.Pinning, roots)

	return CollectResult(ctx, rmed, nil)
}

```

这段代码定义了一个名为 `CollectResult` 的函数，用于从垃圾回收器运行结束时收集输出，并调用给定的回调函数处理被删除的对象。该函数还会收集所有错误并返回一个 `MultiError` 类型的对象。

函数的作用是等待垃圾回收器运行结束，并收集其输出。在循环中，它等待垃圾回收器有新输出，如果有新输出，它就会检查输出是否包含错误。如果是这种情况，它就会处理新的错误并将其添加到 `errors` 数组中。如果不是这种情况，它就会等待函数上下文中的 `Done` 信号，并将其添加到 `errors` 数组中。

最后，函数会根据 `errors` 数组长度返回一个 `MultiError` 类型的对象，其中包含所有收集到的错误。


```go
// CollectResult collects the output of a garbage collection run and calls the
// given callback for each object removed.  It also collects all errors into a
// MultiError which is returned after the gc is completed.
func CollectResult(ctx context.Context, gcOut <-chan gc.Result, cb func(cid.Cid)) error {
	var errors []error
loop:
	for {
		select {
		case res, ok := <-gcOut:
			if !ok {
				break loop
			}
			if res.Error != nil {
				errors = append(errors, res.Error)
			} else if res.KeyRemoved.Defined() && cb != nil {
				cb(res.KeyRemoved)
			}
		case <-ctx.Done():
			errors = append(errors, ctx.Err())
			break loop
		}
	}

	switch len(errors) {
	case 0:
		return nil
	case 1:
		return errors[0]
	default:
		return NewMultiError(errors...)
	}
}

```

这段代码定义了一个名为 `NewMultiError` 的函数，它接收一个或多个错误对象（errs...），并返回一个新的 `MultiError` 对象。

函数的实现是将接收到的错误对象复制到一个名为 `errs` 的切片上，然后将切片的长度减去一个错误对象的长度，并将结果存储到一个名为 `errs[:len(errs)-1]` 的切片上。另一个错误对象存储到名为 `errs[len(errs)-1]` 的切片上。

函数还定义了一个名为 `MultiError` 的接口，该接口包含一个名为 `Errors` 的切片和一个名为 `Summary` 的字段，它们都存储了多个错误对象的计算结果。

最后，函数的 `Error` 方法从 `e` 引用了 `MultiError` 对象，并遍历它的 `Errors` 和 `Summary` 字段，将它们的结果组合成一个字符串并返回。


```go
// NewMultiError creates a new MultiError object from a given slice of errors.
func NewMultiError(errs ...error) *MultiError {
	return &MultiError{errs[:len(errs)-1], errs[len(errs)-1]}
}

// MultiError contains the results of multiple errors.
type MultiError struct {
	Errors  []error
	Summary error
}

func (e *MultiError) Error() string {
	var buf bytes.Buffer
	for _, err := range e.Errors {
		buf.WriteString(err.Error())
		buf.WriteString("; ")
	}
	buf.WriteString(e.Summary.Error())
	return buf.String()
}

```

该函数 `GarbageCollectAsync` 是 `ipfs-server/ipfs` 项目中用于垃圾回收的异步函数。该函数的输入参数为 `n *core.IpfsNode` 和 `ctx context.Context`。

函数的作用是执行以下操作：

1. 调用 `BestEffortRoots` 函数，该函数的输入参数为 `n.FilesRoot`，用于查找根节点。
2. 如果 `BestEffortRoots` 函数执行成功，则返回一个 `gc.Result` 类型的通道，其中包含垃圾回收的结果。
3. 如果 `BestEffortRoots` 函数执行失败，则返回一个 `gc.Result` 类型的通道，其中包含错误信息。
4. 如果需要定期执行垃圾回收，则调用 `PeriodicGC` 函数，该函数的输入参数为 `ctx context.Context` 和 `node *core.IpfsNode`。

`PeriodicGC` 函数的作用是每 `cfg.Datastore.GCPeriod` 小时执行一次垃圾回收。如果 `cfg.Datastore.GCPeriod` 配置为空，则认为垃圾回收enabled。函数的实现可能还涉及设置一些选项，例如 `datastore` 选项的 `GCPeriod` 选项。


```go
func GarbageCollectAsync(n *core.IpfsNode, ctx context.Context) <-chan gc.Result {
	roots, err := BestEffortRoots(n.FilesRoot)
	if err != nil {
		out := make(chan gc.Result)
		out <- gc.Result{Error: err}
		close(out)
		return out
	}

	return gc.GC(ctx, n.Blockstore, n.Repo.Datastore(), n.Pinning, roots)
}

func PeriodicGC(ctx context.Context, node *core.IpfsNode) error {
	cfg, err := node.Repo.Config()
	if err != nil {
		return err
	}

	if cfg.Datastore.GCPeriod == "" {
		cfg.Datastore.GCPeriod = "1h"
	}

	period, err := time.ParseDuration(cfg.Datastore.GCPeriod)
	if err != nil {
		return err
	}
	if int64(period) == 0 {
		// if duration is 0, it means GC is disabled.
		return nil
	}

	gc, err := NewGC(node)
	if err != nil {
		return err
	}

	for {
		select {
		case <-ctx.Done():
			return nil
		case <-time.After(period):
			// the private func maybeGC doesn't compute storageMax, storageGC, slackGC so that they are not re-computed for every cycle
			if err := gc.maybeGC(ctx, 0); err != nil {
				log.Error(err)
			}
		}
	}
}

```

该函数`ConditionalGC`的作用是使用`NewGC`函数创建一个`GC`对象，然后使用`GC.MaybeGC`函数尝试执行垃圾回收。如果执行过程中出现错误，函数将返回该错误。

具体来说，该函数的作用如下：

1. 创建一个名为`gc`的`GC`对象；
2. 如果`gc`对象已存在，则执行`GC.MaybeGC`函数，传递给定上下文`ctx`和偏移量`offset`；
3. 如果`GC`对象不存在，则执行以下操作：
	1. 从存储库中获取存储空间使用情况；
	2. 如果`GC`对象之前没有分配过存储空间，则创建一个新的`GC`对象并执行垃圾回收；
	3. 如果`GC`对象已存在，则检查当前使用的存储空间是否超过了`GC.StorageGC`和`GC.StorageMax`的较小值。如果是，则执行以下操作：
		1. 输出一条警告信息；
		2. 如果`GC.StorageMax`被超过，则执行以下操作：
			1. 从存储库中获取更多的存储空间；
			2. 输出一条警告信息；
		3. 执行垃圾回收操作。


```go
func ConditionalGC(ctx context.Context, node *core.IpfsNode, offset uint64) error {
	gc, err := NewGC(node)
	if err != nil {
		return err
	}
	return gc.maybeGC(ctx, offset)
}

func (gc *GC) maybeGC(ctx context.Context, offset uint64) error {
	storage, err := gc.Repo.GetStorageUsage(ctx)
	if err != nil {
		return err
	}

	if storage+offset > gc.StorageGC {
		if storage+offset > gc.StorageMax {
			log.Warnf("pre-GC: %s", ErrMaxStorageExceeded)
		}

		// Do GC here
		log.Info("Watermark exceeded. Starting repo GC...")

		if err := GarbageCollect(gc.Node, ctx); err != nil {
			return err
		}
		log.Infof("Repo GC done. See `ipfs repo stat` to see how much space got freed.\n")
	}
	return nil
}

```

# `core/corerepo/stat.go`

这段代码定义了一个名为 `SizeStat` 的结构体，用于表示 Kubernetes Repo 中的文件和目录的大小和容量限制。它包含以下字段：

* `repoSize`：表示整个 Repo 的总大小，单位是字节。
* `fileSize`：表示当前文件的大小，单位是字节。
* `dirSize`：表示当前目录的大小，单位是字节。
* `freeSize`：表示当前可用的文件和目录的大小，单位是字节。
* `limit`：表示 Repo 的限制，即在当前容量下可以存储的最大文件和目录的总大小。

此外，还包含一个名为 `fmt.Printf` 的函数，用于将 `SizeStat` 中的字段格式化输出。


```go
package corerepo

import (
	"fmt"
	"math"

	context "context"

	"github.com/ipfs/kubo/core"
	fsrepo "github.com/ipfs/kubo/repo/fsrepo"

	humanize "github.com/dustin/go-humanize"
)

// SizeStat wraps information about the repository size and its limit.
```

这段代码定义了一个名为SizeStat的结构体，其中包含两个整型字段，一个是RepoSize，表示存储库中对象的数量，另一个是StorageMax，表示存储库的最大容量。

接着定义了一个名为Stat的结构体，其中包含一个SizeStat类型的字段，一个整型字段，表示存储库中对象的数目，一个字符串字段，表示存储库文件的路径，一个字符串字段，表示存储库的版本。

然后定义了一个常量，NoLimit，表示无限存储库。

该代码接下来没有做其他事情，只是一个简单的定义，定义了两个结构体和一个常量。


```go
type SizeStat struct {
	RepoSize   uint64 // size in bytes
	StorageMax uint64 // size in bytes
}

// Stat wraps information about the objects stored on disk.
type Stat struct {
	SizeStat
	NumObjects uint64
	RepoPath   string
	Version    string
}

// NoLimit represents the value for unlimited storage
const NoLimit uint64 = math.MaxUint64

```

此代码定义了一个名为RepoStat的函数，接收两个参数，一个是IPFS节点对象的引用，另一个是表示Ipfs块存储器中所有键的通道的引用。函数返回一个Stat对象，其中包含SizeStat和NumObjects字段，以及RepoPath和Version字段。

具体来说，RepoStat函数首先调用RepoSize函数，该函数返回一个SizeStat对象，包含了当前Ipfs节点对象的RepoSize字段以及存储器最大值。如果RepoSize对象的错误，函数返回Stat{}，错误。

然后，RepoStat函数读取Ipfs块存储器中的所有键，并循环计数。计数器变量count逐渐增加，最终存储在uint64类型的变量中。

最后，如果必要，RepoStat函数使用fsrepo.BestKnownPath函数找到Ipfs节点对象的最佳已知路径，并返回该路径。函数的返回值是Stat对象，其中包含SizeStat、NumObjects、RepoPath和Version字段。


```go
// RepoStat returns a *Stat object with all the fields set.
func RepoStat(ctx context.Context, n *core.IpfsNode) (Stat, error) {
	sizeStat, err := RepoSize(ctx, n)
	if err != nil {
		return Stat{}, err
	}

	allKeys, err := n.Blockstore.AllKeysChan(ctx)
	if err != nil {
		return Stat{}, err
	}

	count := uint64(0)
	for range allKeys {
		count++
	}

	path, err := fsrepo.BestKnownPath()
	if err != nil {
		return Stat{}, err
	}

	return Stat{
		SizeStat: SizeStat{
			RepoSize:   sizeStat.RepoSize,
			StorageMax: sizeStat.StorageMax,
		},
		NumObjects: count,
		RepoPath:   path,
		Version:    fmt.Sprintf("fs-repo@%d", fsrepo.RepoVersion),
	}, nil
}

```

此代码实现了一个名为RepoSize的函数，接收一个名为n的IpfsNode对象，返回一个SizeStat对象，并设置其RepoSize和StorageMax字段。函数的实现主要步骤如下：

1. 根据IpfsNode对象的Repo属性计算RepoSize。
2. 根据IpfsNode对象的Config属性计算StorageMax，若Config属性中包含StorageMax，则直接使用该值，否则将cfg.Datastore.StorageMax设置为NoLimit（不限制存储空间）。
3. 返回计算得到的SizeStat对象，若计算过程中出现错误，返回NoSizeStat表示没有可输出的大小信息。


```go
// RepoSize returns a *Stat object with the RepoSize and StorageMax fields set.
func RepoSize(ctx context.Context, n *core.IpfsNode) (SizeStat, error) {
	r := n.Repo

	cfg, err := r.Config()
	if err != nil {
		return SizeStat{}, err
	}

	usage, err := r.GetStorageUsage(ctx)
	if err != nil {
		return SizeStat{}, err
	}

	storageMax := NoLimit
	if cfg.Datastore.StorageMax != "" {
		storageMax, err = humanize.ParseBytes(cfg.Datastore.StorageMax)
		if err != nil {
			return SizeStat{}, err
		}
	}

	return SizeStat{
		RepoSize:   usage,
		StorageMax: storageMax,
	}, nil
}

```

# `core/coreunix/add.go`

该代码是一个 Go 语言项目，名为 "coreunix"，旨在提供对 IPFS（InterPlanetary File System）的本地化支持。IPFS 是一个分布式文件系统，可以在全球范围内提供高效的文件存储和访问。

该项目的核心组件包括：

1. Box-O（Box-O）块存储引擎：提供对 IPFS 原始数据存储的支持，并作为文件系统的底层存储。
2. 块处理引擎（Chunker）：负责将 IPFS 数据划分为固定大小的块，并将这些块存储到 Box-O 块存储中。
3. Core-IFACE（核心 IPFS 接口）：提供与 IPFS 原始存储的底层接口，实现与外部系统（如文件系统、数据库等）的对接。
4. File-Store（文件存储目录）：负责管理 IPFS 数据在文件系统中的存储。
5. 文件系统导入（Importer）：负责将文件系统（如 Git、AppleFS 等）中的文件系统导入 IPFS 树。
6. Merkle-DAG（ Merkle-DAG）：用于构建 File-Store 树的数据结构。
7. 文件系统导入（Importer）：实现从文件系统中导入文件系统数据到 IPFS 树的过程。
8. Block-Store Importer（块存储导入器）：实现从块存储（如 balanced 或 trickle）中导入数据到 IPFS 树的过程。
9. i-Helper（助手函数）：一些辅助函数，如对文件系统或块存储的路径进行解析。
10. Trickle（滴答）：用于 IPFS 网络中的并行文件系统操作。
11. CLI（命令行界面）：提供用于操作 IPFS 文件的命令行工具。
12.日志记录：记录 IPFS 文件的访问日志。

通过这些组件，该项目可以提供一个完整的 IPFS 文件系统支持，使得开发人员可以在自己的本地环境或全球分布式环境中更轻松地使用 IPFS。


```go
package coreunix

import (
	"context"
	"errors"
	"fmt"
	"io"
	gopath "path"
	"strconv"

	bstore "github.com/ipfs/boxo/blockstore"
	chunker "github.com/ipfs/boxo/chunker"
	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/files"
	posinfo "github.com/ipfs/boxo/filestore/posinfo"
	dag "github.com/ipfs/boxo/ipld/merkledag"
	"github.com/ipfs/boxo/ipld/unixfs"
	"github.com/ipfs/boxo/ipld/unixfs/importer/balanced"
	ihelper "github.com/ipfs/boxo/ipld/unixfs/importer/helpers"
	"github.com/ipfs/boxo/ipld/unixfs/importer/trickle"
	"github.com/ipfs/boxo/mfs"
	"github.com/ipfs/boxo/path"
	pin "github.com/ipfs/boxo/pinning/pinner"
	"github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	logging "github.com/ipfs/go-log"

	"github.com/ipfs/kubo/tracing"
)

```

这段代码定义了一个名为“coreunix”的日志输出器，并在其中记录了 progress 的进度。它是如何工作的以及如何使用 progress 变量，请参阅以下解释：

1. “var log = logging.Logger("coreunix")”：这行代码创建了一个名为“coreunix”的输出器实例，并将其记录到名为“log”的变量中。

2. “const progressReaderIncrement = 1024 * 256”：这行代码定义了一个名为“progressReaderIncrement”的常量，它等于 1024 个字节（1MB）的进度更新量。

3. “var liveCacheSize = uint64(256 << 10)”：这行代码定义了一个名为“liveCacheSize”的变量，它表示一个 64 字节的整数，将 256 个字节（16 字节的整数）的内存分配给缓存。

4. “type Link struct { Name, Hash string, Size   uint64 }”：这行代码定义了一个名为“Link”的接口类型，它包含了一个“Name”和“Hash”字段，以及一个“Size”字段，用于存储数据。

5. “type syncer interface { Sync() error }”：这行代码定义了一个名为“syncer”的接口类型，它包含了一个名为“Sync”的函数，用于同步操作。

6. “var progress = 0, liveSync bool”：这行代码定义了两个变量，“progress”和“liveSync”。其中，“progress”是一个整数，用于记录当前的进度。它初始化为 0，因为还没有收到任何 progress update。而“liveSync”是一个布尔变量，用于同步数据，初始化为假（false）。

7. “log.Printf(" progress: %d bytes", progressReaderIncrement)”：这行代码使用“Printf”函数的格式字符串，打印当前的 progress 值，即 progressReaderIncrement 字节数。

8. “var w = &linkSync, r = &linkSync”：这行代码定义了两个变量，分别为“w”和“r”。它们都引用了“&linkSync”这个指针，用于同步“Link”类型的数据。

9. “var reader = log.Logger("coreunix")”。至此，这段代码的完整实现已经完整地解释出来。


```go
var log = logging.Logger("coreunix")

// how many bytes of progress to wait before sending a progress update message
const progressReaderIncrement = 1024 * 256

var liveCacheSize = uint64(256 << 10)

type Link struct {
	Name, Hash string
	Size       uint64
}

type syncer interface {
	Sync() error
}

```

这段代码定义了一个名为 `NewAdder` 的函数，它返回一个用于文件添加操作的新 Adder 对象。函数接受三个参数：一个上下文上下文 `ctx`，一个验证 `pin`，以及一个 `bs` 类型的锁套筒 `bs`。此外，它还接受一个 `ds` 类型的 IPLDaagService 对象 `ds`。函数返回一个指向 `Adder` 类型的引用，如果函数创建或返回的任何值是 `nil`，则表示操作失败。


```go
// NewAdder Returns a new Adder used for a file add operation.
func NewAdder(ctx context.Context, p pin.Pinner, bs bstore.GCLocker, ds ipld.DAGService) (*Adder, error) {
	bufferedDS := ipld.NewBufferedDAG(ctx, ds)

	return &Adder{
		ctx:        ctx,
		pinning:    p,
		gcLocker:   bs,
		dagService: ds,
		bufferedDS: bufferedDS,
		Progress:   false,
		Pin:        true,
		Trickle:    false,
		Chunker:    "",
	}, nil
}

```

该代码定义了一个名为 `Adder` 的结构体，用于在点击按钮时添加指定的switch。

具体来说，该结构体包含以下字段：

- `ctx`: 上下文句柄，用于跨 gc 上下文。
- `pinning`: 用于锁定 `pin.Pinner` 的字段。
- `gcLocker`: 用于锁定 `bstore.GCLocker` 的字段。
- `dagService`: 用于访问智能代理服务器的字段。
- `bufferedDS`: 用于访问缓冲区数据结构的字段。
- `Out`: 用于从 `ipld.BufferedDAG` 接收输出数据的字段。
- `Progress`: 用于设置进度条的指示器的字段。
- `Pin`: 用于设置是否对所选数据进行复制的字段。
- `Trickle`: 用于设置是否尝试同步所有同步操作的字段。
- `RawLeaves`: 用于设置是否尝试从本地文件系统中读取数据的字段。
- `Silent`: 用于设置是否输出沉默的指示器的字段。
- `NoCopy`: 用于设置是否避免复制所选数据的字段。
- `Chunker`: 用于设置是否对分片进行合并的指示器的字段。
- `mroot`: 用于访问根目录的引用字段。
- `unlocker`: 用于访问锁定的数据结构的字段。
- `tempRoot`: 用于在内存中存储临时根目录的引用字段。
- `CidBuilder`: 用于构建 `cid.Cid` 对象的 builder。
- `liveNodes`: 用于计数实际同步操作的节点数量。

该结构体的实现可能需要根据具体需求和环境进行调整。


```go
// Adder holds the switches passed to the `add` command.
type Adder struct {
	ctx        context.Context
	pinning    pin.Pinner
	gcLocker   bstore.GCLocker
	dagService ipld.DAGService
	bufferedDS *ipld.BufferedDAG
	Out        chan<- interface{}
	Progress   bool
	Pin        bool
	Trickle    bool
	RawLeaves  bool
	Silent     bool
	NoCopy     bool
	Chunker    string
	mroot      *mfs.Root
	unlocker   bstore.Unlocker
	tempRoot   cid.Cid
	CidBuilder cid.Builder
	liveNodes  uint64
}

```

这段代码定义了一个名为"func"的函数，它接受一个名为"Adder"的类类型的参数"mfsRoot()"，并返回一个指向名为"mfs.Root"的类型和一个名为"error"的类型的变量。

函数内部首先检查"adder.mroot"是否为空，如果是，则直接返回"adder.mroot"，否则创建一个名为"rnode"的新节点，并设置其CID为"adder.CidBuilder"。然后使用"mfs.NewRoot"函数在"adder.ctx"上下文中创建一个根节点，并设置其数据持久化 agents 为"adder.dagService"，然后将"rnode"设置为根节点的嵌套层，最后将"adder.mroot"设置为创建的根节点的引用。

函数返回值为根节点，如果没有错误则返回"nil"。


```go
func (adder *Adder) mfsRoot() (*mfs.Root, error) {
	if adder.mroot != nil {
		return adder.mroot, nil
	}
	rnode := unixfs.EmptyDirNode()
	err := rnode.SetCidBuilder(adder.CidBuilder)
	if err != nil {
		return nil, err
	}
	mr, err := mfs.NewRoot(adder.ctx, adder.dagService, rnode, nil)
	if err != nil {
		return nil, err
	}
	adder.mroot = mr
	return adder.mroot, nil
}

```

这段代码定义了一个名为 Adder 的链式数据结构，它可以用于实现 Generator 和相聚树。在这个数据结构中，每个节点包含了数据、指向子节点的指针以及该节点到根节点的路径。

具体来说，这段代码实现了以下功能：

1. `SetMfsRoot` 函数用于设置 Addr 的根目录，即 `r`。
2. `add` 函数用于将给定的数据读取并添加到链表中。该函数使用了 `ipld`（ Interperceptual P伟大复兴论 ）库中的 chunker，根据不同的链式数据结构对数据进行处理，并实现了 add、remove、Commit、Chunk 等方法。其中，`add` 函数可能还实现了其他接口，具体取决于具体的链式数据结构。

除了 `ipld` 库，这段代码还可能使用了其他库或者自己实现了某些功能。


```go
// SetMfsRoot sets `r` as the root for Adder.
func (adder *Adder) SetMfsRoot(r *mfs.Root) {
	adder.mroot = r
}

// Constructs a node from reader's data, and adds it. Doesn't pin.
func (adder *Adder) add(reader io.Reader) (ipld.Node, error) {
	chnk, err := chunker.FromString(reader, adder.Chunker)
	if err != nil {
		return nil, err
	}

	params := ihelper.DagBuilderParams{
		Dagserv:    adder.bufferedDS,
		RawLeaves:  adder.RawLeaves,
		Maxlinks:   ihelper.DefaultLinksPerBlock,
		NoCopy:     adder.NoCopy,
		CidBuilder: adder.CidBuilder,
	}

	db, err := params.New(chnk)
	if err != nil {
		return nil, err
	}
	var nd ipld.Node
	if adder.Trickle {
		nd, err = trickle.Layout(db)
	} else {
		nd, err = balanced.Layout(db)
	}
	if err != nil {
		return nil, err
	}

	return nd, adder.bufferedDS.Commit()
}

```

这段代码定义了一个名为RootNode的函数，用于返回mfs root节点。函数实现包括以下步骤：

1. 如果已经获取到mfs root节点，则输出它。
2. 如果获取到根目录，则从该目录中获取根节点。
3. 如果只有一个根目录，则将该目录的根节点作为根节点。
4. 如果获取到的根节点是复杂的节点，例如子节点也属于root节点，则递归地获取子节点，直到获取到普通的节点为止。
5. 返回根节点，如果获取到错误则返回err。


```go
// RootNode returns the mfs root node
func (adder *Adder) curRootNode() (ipld.Node, error) {
	mr, err := adder.mfsRoot()
	if err != nil {
		return nil, err
	}
	root, err := mr.GetDirectory().GetNode()
	if err != nil {
		return nil, err
	}

	// if one root file, use that hash as root.
	if len(root.Links()) == 1 {
		nd, err := root.Links()[0].GetNode(adder.ctx, adder.dagService)
		if err != nil {
			return nil, err
		}

		root = nd
	}

	return root, err
}

```

这段代码的作用是解决Addrino中存在不确定的延迟问题。在传统的Addrino中，当一个节点存在时，它的子节点也在不断地被添加。这可能会导致在添加过程中，部分节点的状态没有被正确地设置，从而导致整个应用程序的性能下降。

为了解决这个问题，作者实现了两个主要的功能：

1. 定义了一个名为`PinRoot`的函数，该函数接收一个根节点`root`，并将其作为参数。函数首先检查`adder`对象中是否已经将根节点进行锁定，如果是，则返回。否则，函数将尝试将根节点添加到`dagService`中，并将其状态设置为已 pin。

2. 实现了一个名为`pinning.Unpin`的函数，该函数接收一个根节点`root`和一个模式`pin.Recursive`作为参数。函数尝试使用现有的 pin 模式将根节点加 pin，并在尝试失败时返回。如果成功将根节点加 pin，函数会将根节点的名称存储为`tempRoot`，并返回。

3. 实现了一个名为`pinning.PinWithMode`的函数，该函数接收一个根节点`root`、一个模式`pin.Recursive`和一个选项`pinWithMode`作为参数。函数尝试使用根节点的 pin 模式将根节点加 pin，并在尝试失败时返回。如果成功将根节点加 pin，函数会返回，并调用`Flush`函数清除已经添加的 pin。

通过这些函数，作者解决了一个可能导致延迟和不确定的问题，使得Addrino的应用程序更加可靠和高效。


```go
// Recursively pins the root node of Adder and
// writes the pin state to the backing datastore.
func (adder *Adder) PinRoot(ctx context.Context, root ipld.Node) error {
	ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "PinRoot")
	defer span.End()

	if !adder.Pin {
		return nil
	}

	rnk := root.Cid()

	err := adder.dagService.Add(ctx, root)
	if err != nil {
		return err
	}

	if adder.tempRoot.Defined() {
		err := adder.pinning.Unpin(ctx, adder.tempRoot, true)
		if err != nil {
			return err
		}
		adder.tempRoot = rnk
	}

	err = adder.pinning.PinWithMode(ctx, rnk, pin.Recursive)
	if err != nil {
		return err
	}

	return adder.pinning.Flush(ctx)
}

```

这段代码定义了一个名为`outputDirs`的函数，接受两个参数：`path` 和 `fsn`。函数的作用是输出目录下所有子目录的名称，以及目录本身下的子目录的名称。

函数的实现主要分为以下几步：

1. 判断输入的 `fsn` 是否为文件或目录。
2. 如果 `fsn` 是文件，那么直接返回，因为文件不会创建子目录。
3. 如果 `fsn` 是目录，那么执行以下操作：

a. 获取目录下的子目录名称，使用 `fson.ListNames` 函数获取。
b. 对于每个子目录名称，递归调用 `outputDirs` 函数，将子目录名称传递给下一层递归。
c. 在递归过程中，如果遇到错误，返回错误信息。
4. 返回导出的目录名称。

函数的实现遵循了递归的安全模式，即在递归过程中保证安全。因为函数在递归过程中使用了 `fsn.ListNames` 和 `outputDirs` 函数，这两个函数都使用了安全模式，所以在递归过程中对输入进行了检查和过滤，避免了潜在的安全风险。


```go
func (adder *Adder) outputDirs(path string, fsn mfs.FSNode) error {
	switch fsn := fsn.(type) {
	case *mfs.File:
		return nil
	case *mfs.Directory:
		names, err := fsn.ListNames(adder.ctx)
		if err != nil {
			return err
		}

		for _, name := range names {
			child, err := fsn.Child(name)
			if err != nil {
				return err
			}

			childpath := gopath.Join(path, name)
			err = adder.outputDirs(childpath, child)
			if err != nil {
				return err
			}

			fsn.Uncache(name)
		}
		nd, err := fsn.GetNode()
		if err != nil {
			return err
		}

		return outputDagnode(adder.Out, path, nd)
	default:
		return fmt.Errorf("unrecognized fsn type: %#v", fsn)
	}
}

```

该函数的作用是添加一个节点到给定的路径中。如果路径为空，则将节点ID设置为该节点的CID，然后将其设置为根节点。如果节点实现为posinfo.FilestoreNode，则会将其调用mfsRoot()方法，并将其节点设置为当前节点。如果添加节点失败，则返回错误。


```go
func (adder *Adder) addNode(node ipld.Node, path string) error {
	// patch it into the root
	if path == "" {
		path = node.Cid().String()
	}

	if pi, ok := node.(*posinfo.FilestoreNode); ok {
		node = pi.Node
	}

	mr, err := adder.mfsRoot()
	if err != nil {
		return err
	}
	dir := gopath.Dir(path)
	if dir != "." {
		opts := mfs.MkdirOpts{
			Mkparents:  true,
			Flush:      false,
			CidBuilder: adder.CidBuilder,
		}
		if err := mfs.Mkdir(mr, dir, opts); err != nil {
			return err
		}
	}

	if err := mfs.PutNode(mr, path, node); err != nil {
		return err
	}

	if !adder.Silent {
		return outputDagnode(adder.Out, path, node)
	}
	return nil
}

```

This is a Go program that creates a wrapped Node.js HTTPFS Pin. The Node.js HTTPFS Pin is a library that enables the use of Node.js's HTTPFS interface to create HTTP/HTTPS file servers, and the HTTPFS root node is the directory that contains the files that serve as the root of the file server.

The program takes a single parameter, which is a file name. This file is then added to the Node.js HTTPFS Pin, and the wrapper directory is created in the same directory as the file.

The program also creates a directory in the same directory as the file if the file is not a directory, and then lists all the names of the directory and the file, and then opens the directory for writing.

Finally, the program outputs the names of the directories in the same directory as the file and the file itself, and optionally syncs the directory with the Node.js HTTPFS Pin's output directory.

The program uses the `filesystem` package to perform the file operations, and the `node` package to handle the Node.js HTTPFS Pin's synchronization with the HTTPFS root node.


```go
// AddAllAndPin adds the given request's files and pin them.
func (adder *Adder) AddAllAndPin(ctx context.Context, file files.Node) (ipld.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "AddAllAndPin")
	defer span.End()

	if adder.Pin {
		adder.unlocker = adder.gcLocker.PinLock(ctx)
	}
	defer func() {
		if adder.unlocker != nil {
			adder.unlocker.Unlock(ctx)
		}
	}()

	if err := adder.addFileNode(ctx, "", file, true); err != nil {
		return nil, err
	}

	// get root
	mr, err := adder.mfsRoot()
	if err != nil {
		return nil, err
	}
	var root mfs.FSNode
	rootdir := mr.GetDirectory()
	root = rootdir

	err = root.Flush()
	if err != nil {
		return nil, err
	}

	// if adding a file without wrapping, swap the root to it (when adding a
	// directory, mfs root is the directory)
	_, dir := file.(files.Directory)
	var name string
	if !dir {
		children, err := rootdir.ListNames(adder.ctx)
		if err != nil {
			return nil, err
		}

		if len(children) == 0 {
			return nil, fmt.Errorf("expected at least one child dir, got none")
		}

		// Replace root with the first child
		name = children[0]
		root, err = rootdir.Child(name)
		if err != nil {
			return nil, err
		}
	}

	err = mr.Close()
	if err != nil {
		return nil, err
	}

	nd, err := root.GetNode()
	if err != nil {
		return nil, err
	}

	// output directory events
	err = adder.outputDirs(name, root)
	if err != nil {
		return nil, err
	}

	if asyncDagService, ok := adder.dagService.(syncer); ok {
		err = asyncDagService.Sync()
		if err != nil {
			return nil, err
		}
	}

	if !adder.Pin {
		return nd, nil
	}
	return nd, adder.PinRoot(ctx, nd)
}

```

该函数的作用是添加一个文件到名为 "CoreUnix.Adder" 的会话中。它将打开一个文件并将其附加到名为 "CoreUnix.Adder" 的会话中。

具体来说，函数首先打开一个名为 "CoreUnix.Adder" 的会话。然后，它调用一个名为 "addFileNode" 的函数，该函数接受三个参数：上下文上下文、文件路径和文件对象。

如果文件对象是目录，函数将递归地调用 "addDir" 函数。如果文件对象是一个符号链接，函数将调用 "addSymlink" 函数。如果文件对象是一个文件，函数将调用 "addFile" 函数。

函数在开始之前调用了一个名为 "atory pause for GC" 的函数，该函数可能会在函数内块中暂停执行，以确保在函数结束时谷物收集器已准备好。

如果调用 "addFileNode" 函数时出现任何错误，函数将返回该错误。否则，函数返回 void 表示成功。


```go
func (adder *Adder) addFileNode(ctx context.Context, path string, file files.Node, toplevel bool) error {
	ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "AddFileNode")
	defer span.End()

	defer file.Close()

	err := adder.maybePauseForGC(ctx)
	if err != nil {
		return err
	}

	if adder.liveNodes >= liveCacheSize {
		// TODO: A smarter cache that uses some sort of lru cache with an eviction handler
		mr, err := adder.mfsRoot()
		if err != nil {
			return err
		}
		if err := mr.FlushMemFree(adder.ctx); err != nil {
			return err
		}

		adder.liveNodes = 0
	}
	adder.liveNodes++

	switch f := file.(type) {
	case files.Directory:
		return adder.addDir(ctx, path, f, toplevel)
	case *files.Symlink:
		return adder.addSymlink(path, f)
	case files.File:
		return adder.addFile(path, f)
	default:
		return errors.New("unknown file type")
	}
}

```

这段代码定义了一个名为`func`的函数，接收一个名为`Adder`的整型指针和一个名为`files.Symlink`的整型指针作为参数。

函数的作用是添加一个符号链接到指定的路径。函数首先通过调用`unixfs.SymlinkData`函数获取符号链接的目标文件路径，然后获取符号链接本身的数据，接着创建一个`dag.NodeWithData`对象，将数据设置为`sdata`，并设置`cidBuilder`为`adder.cidBuilder`，最后调用`adder.dagService.Add`函数将符号链接添加到指定的路径上，并返回结果。

函数的返回值类型为`error`。


```go
func (adder *Adder) addSymlink(path string, l *files.Symlink) error {
	sdata, err := unixfs.SymlinkData(l.Target)
	if err != nil {
		return err
	}

	dagnode := dag.NodeWithData(sdata)
	err = dagnode.SetCidBuilder(adder.CidBuilder)
	if err != nil {
		return err
	}
	err = adder.dagService.Add(adder.ctx, dagnode)
	if err != nil {
		return err
	}

	return adder.addNode(dagnode, path)
}

```

该函数的作用是接收一个自定义的 adder 类型和一个文件对象 files.File，对文件进行追加操作并返回错误。

函数首先检查是否指定了 progress 标志，如果是，则使用 progressReader 对文件进行读取，并设置 progress 变量为 reader,path 变量为指定路径，out 变量为 adder 的输出通道。

然后，函数通过判断文件是否为 files.FileInfo 类型来确定如何读取文件。如果是，则使用 progressReader2 类型来读取文件，并设置 rdr 为 progressReader,fi 为文件信息。否则，则将 reader 指向文件。

接下来，函数调用 adder.add 函数对读者进行追加，并将结果存储在 dagnode 中。如果在这个过程中出现错误，函数返回该错误。最后，函数通过调用 adder.addNode 函数将追加的结果添加到根节点中。

函数的实现主要围绕着两个主要点：一个是如何读取文件，另一个是如何将追加操作的结果添加到树中。


```go
func (adder *Adder) addFile(path string, file files.File) error {
	// if the progress flag was specified, wrap the file so that we can send
	// progress updates to the client (over the output channel)
	var reader io.Reader = file
	if adder.Progress {
		rdr := &progressReader{file: reader, path: path, out: adder.Out}
		if fi, ok := file.(files.FileInfo); ok {
			reader = &progressReader2{rdr, fi}
		} else {
			reader = rdr
		}
	}

	dagnode, err := adder.add(reader)
	if err != nil {
		return err
	}

	// patch it into the root
	return adder.addNode(dagnode, path)
}

```

这段代码是一个函数，名为func (adder *Adder) addDir(ctx context.Context, path string, dir files.Directory, toplevel bool)。

它接受两个参数，一个是自定义的adder类型Adder，另一个是一个路径参数dir，dir是一个files.Directory类型的变量。

函数的主要作用是添加一个目录到给定的系统中，并设置一些额外的选项，如是否将目录添加到现有目录的顶级目录中，以及使用哪个cid构建器来自动创建目录。

具体来说，函数首先使用log.Infof函数输出一条日志信息，然后检查是否有一个不为空的路径参数。如果是，函数会继续执行以下操作：

1. 如果目录是顶级目录，函数会尝试调用mfs.Mkdir函数来创建目录。如果这个函数失败，函数会返回一个错误。
2. 如果目录不是顶级目录，函数会遍历目录中的所有文件并调用adder.addFileNode函数来添加每个文件节点。如果添加失败，函数会返回一个错误。
3. 函数还设置了一些选项，如是否自动创建目录的顶级目录，以及使用哪个cid构建器。这些选项会被传递给adder.addFileNode函数。

函数的返回值是目录工作的错误。


```go
func (adder *Adder) addDir(ctx context.Context, path string, dir files.Directory, toplevel bool) error {
	log.Infof("adding directory: %s", path)

	if !(toplevel && path == "") {
		mr, err := adder.mfsRoot()
		if err != nil {
			return err
		}
		err = mfs.Mkdir(mr, path, mfs.MkdirOpts{
			Mkparents:  true,
			Flush:      false,
			CidBuilder: adder.CidBuilder,
		})
		if err != nil {
			return err
		}
	}

	it := dir.Entries()
	for it.Next() {
		fpath := gopath.Join(path, it.Name())
		err := adder.addFileNode(ctx, fpath, it.Node(), false)
		if err != nil {
			return err
		}
	}

	return it.Err()
}

```

这段代码是一个名为 `CoreUnix.Adder` 的函数，它是用来暂停添加器的操作，以进行垃圾回收（GC）。函数接收一个名为 `ctx` 的 `context.Context` 参数，用于在发生错误时传递错误信息。

函数内部使用了 `tracing.Span` 来跟踪发生的GC，以便在发生错误时进行调试。函数的一个潜在副作用是，如果 `adder.unlocker` 所关联的 `unlocker` 且 `adder.gcLocker.GCRequested(ctx)` 是 `true`，则函数将暂停添加器的操作，并尝试解锁所有引用计数器（curRootNode）并尝试更新它们。

函数的主要逻辑如下：

1. 如果 `adder.unlocker` 且 `adder.gcLocker.GCRequested(ctx)` 是 `true`，那么创建一个名为 `rn` 的根节点参考计数器。
2. 尝试更新 `adder.curRootNode()` 函数，如果发生错误，返回错误并暂停操作。
3. 如果 `adder.curRootNode()` 更新成功，那么解锁 `adder.unlocker`，并尝试再次锁定 `adder.gcLocker`。
4. 如果 `adder.gcLocker.PinLock(ctx)` 成功，那么解锁 `adder.unlocker`，并尝试停止暂停操作。

这段代码的主要目的是让添加器在 GC 期间暂停操作，以便在 GC 完成后继续执行。如果添加器在 GC 期间发生错误，可以暂停操作并输出调试信息，以便进行调试和故障排查。


```go
func (adder *Adder) maybePauseForGC(ctx context.Context) error {
	ctx, span := tracing.Span(ctx, "CoreUnix.Adder", "MaybePauseForGC")
	defer span.End()

	if adder.unlocker != nil && adder.gcLocker.GCRequested(ctx) {
		rn, err := adder.curRootNode()
		if err != nil {
			return err
		}

		err = adder.PinRoot(ctx, rn)
		if err != nil {
			return err
		}

		adder.unlocker.Unlock(ctx)
		adder.unlocker = adder.gcLocker.PinLock(ctx)
	}
	return nil
}

```

该代码定义了一个名为 `outputDagnode` 的函数，它接受一个名为 `out` 的通道，以及一个名为 `name` 的字符串参数和一个名为 `dn` 的整型参数 `ipld.Node`。

函数的作用是向名为 `outputDagnode` 的输出通道发送 `dn` 节点的信息，包括输出路径、名称和大小。代码首先检查 `out` 是否为空，如果是，则返回 `nil`。然后，代码调用名为 `getOutput` 的函数，并将 `dn` 作为参数传递给该函数。如果 `getOutput` 返回时出现错误，函数将返回该错误。如果 `getOutput` 返回时没有错误，代码将在调用 `out` 通道时发送 `dn` 节点的信息，并返回 `nil`以表示成功。


```go
// outputDagnode sends dagnode info over the output channel
func outputDagnode(out chan<- interface{}, name string, dn ipld.Node) error {
	if out == nil {
		return nil
	}

	o, err := getOutput(dn)
	if err != nil {
		return err
	}

	out <- &coreiface.AddEvent{
		Path: o.Path,
		Name: name,
		Size: o.Size,
	}

	return nil
}

```

这段代码定义了一个名为 `getOutput` 的函数，它接受一个名为 `dagnode` 的 IPLD 节点参数。

该函数首先通过调用 `dagnode.Cid()` 获取当前 IPLD 节点的 ID，然后通过调用 `dagnode.Size()` 获取该节点的大小。如果这两个函数中的任何一个返回值是错误的，那么函数将返回一个 ` nil` 表示错误。

接下来，函数创建一个名为 `output` 的新的 `coreiface.AddEvent` 结构体，该结构体包含一个路径，以及一个表示大小（以字节为单位）的字符串。

最后，函数返回刚刚创建的 `output` 结构体，如果调用过程中没有错误。


```go
// from core/commands/object.go
func getOutput(dagnode ipld.Node) (*coreiface.AddEvent, error) {
	c := dagnode.Cid()
	s, err := dagnode.Size()
	if err != nil {
		return nil, err
	}

	output := &coreiface.AddEvent{
		Path: path.FromCid(c),
		Size: strconv.FormatUint(s, 10),
	}

	return output, nil
}

```

这段代码定义了一个名为"progressReader"的结构体，它包含一个文件指针、一个路径和一个用于输出事件数据的通道。

当读取文件时，结构体将文件指针、路径和当前输出事件数据打包到一个"coreiface.AddEvent"类型的通道中，其中包含了当前进度和文件中读取的数据量。

每次"Read"方法被调用时，它读取文件中的数据并更新当前指针和进度。如果当前指针达到文件结尾或者出现错误，那么将进度设置为当前指针位置，并将事件数据发送到通道中。

"progressReader"的"Read"方法的具体实现可以被理解为读取文件并定期向通道中发送数据的过程。这种模式的应用通常用于并发或异步处理中，例如用于处理大量的I/O操作或网络请求。


```go
type progressReader struct {
	file         io.Reader
	path         string
	out          chan<- interface{}
	bytes        int64
	lastProgress int64
}

func (i *progressReader) Read(p []byte) (int, error) {
	n, err := i.file.Read(p)

	i.bytes += int64(n)
	if i.bytes-i.lastProgress >= progressReaderIncrement || err == io.EOF {
		i.lastProgress = i.bytes
		i.out <- &coreiface.AddEvent{
			Name:  i.path,
			Bytes: i.bytes,
		}
	}

	return n, err
}

```

这是一段使用Go编程语言编写的代码，定义了一个名为"progressReader2"的结构体类型，该类型包含一个指向"progressReader"类型的指针和一个名为"files.FileInfo"的结构体。

具体来说，这段代码实现了一个"progressReader"的实例，该实例可以接受一个字符串类型的参数，并在调用"Read"函数时将输入的字符串读入到一个缓冲区中。同时，该实例还包含一个名为"files.FileInfo"的结构体，其中包含一个文件对象的相关信息，如文件名、文件描述符等。

在"progressReader2"的"Read"函数中，首先通过指针变量"i"访问一个"progressReader"实例，然后调用该实例的"Read"函数，将读取到的数据存储到缓冲区中。最后，通过返回值类型整数和错误类型错误来表示读取过程的进展和结果。


```go
type progressReader2 struct {
	*progressReader
	files.FileInfo
}

func (i *progressReader2) Read(p []byte) (int, error) {
	return i.progressReader.Read(p)
}

```