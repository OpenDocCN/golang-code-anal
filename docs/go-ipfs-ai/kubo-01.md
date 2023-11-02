# go-ipfs 源码解析 1

# `/opt/kubo/client/rpc/api_test.go`

这段代码是一个RPC（远程过程调用）框架，它允许客户端与服务器之间通过网络进行通信和交互。它包含了一个名为"rpc"的包，以及一些用于实现RPC框架的组件。

首先，它导入了外部库：`github.com/ipfs/boxo/coreiface`（用于提供IPFS（ InterPlanetary File System ）基本的文件系统接口 ）、`github.com/ipfs/boxo/coreiface/tests`（用于提供测试用的文件系统接口 ）、`github.com/ipfs/boxo/path`（用于提供路径相关的API ）、`github.com/ipfs/kubo/test/cli/harness`（用于与Kubernetes进行交互 ）、`ma`（用于提供跨链信息 ）、`go.uber.org/multierr`（用于处理错误）。

接下来，定义了一些常量和函数：

1. `const (` 是一个字符串，用于表示常量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `const (` 是一个字符串，用于表示常量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `const (` 是一个字符串，用于表示常量 )、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，用于表示变量 ）、 `var (` 是一个字符串，


```
package rpc

import (
	"context"
	"net/http"
	"net/http/httptest"
	"runtime"
	"strconv"
	"strings"
	"sync"
	"testing"
	"time"

	iface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/coreiface/tests"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/kubo/test/cli/harness"
	ma "github.com/multiformats/go-multiaddr"
	"go.uber.org/multierr"
)

```

This appears to be a Go program that is using the IPFS (InterPlanetary File System) protocol to create and manage isolated storage nodes. It appears to be using the Node仪 (Node.js) library to handle the interaction with the IPFS API and the GoL刑罚系统 (Glsent) to create and manage the nodes.

The program appears to be setting up a single node that is configured to enable experimental file storage through the IPFS protocol. It is also configured to run in offline mode, which means that it will not be able to access the IPFS network or reach other nodes that are running in online mode.

The program uses a loop to repeatedly read the configuration of the Node仪 node and then sets up the experimental file storage by writing the configuration to the Node仪 node. It then starts the Node仪 node and the GoL刑罚 system, which will create and manage the experimental file storage nodes.

The program also appears to be using the GoL刑罚系统 to create and manage the experimental file storage nodes. It uses the GoL刑罚系统中内置的 ipfsonly decorator to create the nodes, which enables the experimental file storage feature.

The program also appears to be using the GoL刑罚系统的 pin feature to pin the nodes to the IPFS network, which will prevent the node from being removed if the network becomes unavailable.


```
type NodeProvider struct{}

func (np NodeProvider) MakeAPISwarm(t *testing.T, ctx context.Context, fullIdentity, online bool, n int) ([]iface.CoreAPI, error) {
	h := harness.NewT(t)

	apis := make([]iface.CoreAPI, n)
	nodes := h.NewNodes(n)

	var wg, zero sync.WaitGroup
	zeroNode := nodes[0]
	wg.Add(len(apis))
	zero.Add(1)

	var errs []error
	var errsLk sync.Mutex

	for i, n := range nodes {
		go func(i int, n *harness.Node) {
			if err := func() error {
				defer wg.Done()
				var err error

				n.Init("--empty-repo")

				c := n.ReadConfig()
				c.Experimental.FilestoreEnabled = true
				n.WriteConfig(c)

				n.StartDaemon("--enable-pubsub-experiment", "--offline="+strconv.FormatBool(!online))

				if online {
					if i > 0 {
						zero.Wait()
						n.Connect(zeroNode)
					} else {
						zero.Done()
					}
				}

				apiMaddr, err := n.TryAPIAddr()
				if err != nil {
					return err
				}

				api, err := NewApi(apiMaddr)
				if err != nil {
					return err
				}
				apis[i] = api

				// empty node is pinned even with --empty-repo, we don't want that
				emptyNode, err := path.NewPath("/ipfs/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn")
				if err != nil {
					return err
				}

				if err := api.Pin().Rm(ctx, emptyNode); err != nil {
					return err
				}
				return nil
			}(); err != nil {
				errsLk.Lock()
				errs = append(errs, err)
				errsLk.Unlock()
			}
		}(i, n)
	}

	wg.Wait()

	return apis, multierr.Combine(errs...)
}

```

该代码是一个 Go 语言中的测试框架函数，用于测试 HTTP API 的功能。

具体来说，该函数的作用是：

1. 测试函数内部的新路


```
func TestHttpApi(t *testing.T) {
	t.Parallel()

	if runtime.GOOS == "windows" {
		t.Skip("skipping due to #9905")
	}

	tests.TestApi(NodeProvider{})(t)
}

func Test_NewURLApiWithClient_With_Headers(t *testing.T) {
	t.Parallel()

	var (
		headerToTest        = "Test-Header"
		expectedHeaderValue = "thisisaheadertest"
	)
	ts := httptest.NewServer(
		http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			val := r.Header.Get(headerToTest)
			if val != expectedHeaderValue {
				w.WriteHeader(400)
				return
			}
			http.ServeContent(w, r, "", time.Now(), strings.NewReader("test"))
		}),
	)
	defer ts.Close()
	api, err := NewURLApiWithClient(ts.URL, &http.Client{
		Transport: &http.Transport{
			Proxy:             http.ProxyFromEnvironment,
			DisableKeepAlives: true,
		},
	})
	if err != nil {
		t.Fatal(err)
	}
	api.Headers.Set(headerToTest, expectedHeaderValue)
	p, err := path.NewPath("/ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv")
	if err != nil {
		t.Fatal(err)
	}
	if err := api.Pin().Rm(context.Background(), p); err != nil {
		t.Fatal(err)
	}
}

```

该代码是一个名为 "Test_NewURLApiWithClient_HTTP_Variant" 的函数，它用于测试使用 NewURLApiWithClient HTTP API 进行客户端连通时，对于不同地址和端口的预期 HTTP 请求是否正确。

具体来说，该函数使用 "t.Parallel" 平行测试模式，测试结论断言中包含多个测试用例，每个测试用例包含一个地址和预期的 HTTP 请求。这些测试用例预先定义了一个字符串类型的数组 testcases，其中每个测试用例包含一个地址和预期的 HTTP 请求。

在函数内部，首先会根据地址创建一个 Multiaddr 对象，然后使用 NewApiWithClient 函数根据 Multiaddr 对象创建一个 HTTP API 对象，并使用该 API 对象进行客户端连通。在客户端连通成功后，会执行一系列测试，检查 API 的 URL 是否与预期的 HTTP 请求一致。

如果 API 的 URL 与预期的 HTTP 请求不一致，函数会通过 t.Errorf 函数输出错误信息，并指出期望的 URL 和实际上获取到的 URL。如果所有测试用例的 API URL 都是一致的，函数不会输出任何错误信息，而是返回一个代表测试通过了的逻辑值。


```
func Test_NewURLApiWithClient_HTTP_Variant(t *testing.T) {
	t.Parallel()

	testcases := []struct {
		address  string
		expected string
	}{
		{address: "/ip4/127.0.0.1/tcp/80", expected: "http://127.0.0.1:80"},
		{address: "/ip4/127.0.0.1/tcp/443/tls", expected: "https://127.0.0.1:443"},
		{address: "/ip4/127.0.0.1/tcp/443/https", expected: "https://127.0.0.1:443"},
		{address: "/ip4/127.0.0.1/tcp/443/tls/http", expected: "https://127.0.0.1:443"},
	}

	for _, tc := range testcases {
		address, err := ma.NewMultiaddr(tc.address)
		if err != nil {
			t.Fatal(err)
		}

		api, err := NewApiWithClient(address, &http.Client{})
		if err != nil {
			t.Fatal(err)
		}

		if api.url != tc.expected {
			t.Errorf("Expected = %s; got %s", tc.expected, api.url)
		}
	}
}

```

# `/opt/kubo/client/rpc/block.go`

这段代码定义了一个名为"rpc"的包，包含了与RPC(远程过程调用)相关的接口、路径和工具函数。

具体来说，这个包通过引入了以下依赖：

- "github.com/ipfs/boxo/coreiface" 和 "github.com/ipfs/boxo/coreiface/options" 提供了 IRC 文件系统的支持和相关选项。
- "github.com/ipfs/boxo/path" 提供了文件路径相关的操作。
- "github.com/ipfs/go-cid" 提供了 CID 文件的操作。
- "github.com/multiformats/go-multicodec" 和 "github.com/multiformats/go-multihash" 提供了多链时数据编码和哈希相关的支持。

通过这些依赖，这个包使得我们可以轻松地实现 RPC 相关功能，例如通过 Boxo 文件系统支持远程调用，通过文件路径操作支持本地文件操作，通过 CID 文件操作支持数据持久化等等。


```
package rpc

import (
	"bytes"
	"context"
	"fmt"
	"io"

	iface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/path"
	"github.com/ipfs/go-cid"
	mc "github.com/multiformats/go-multicodec"
	mh "github.com/multiformats/go-multihash"
)

```

这段代码定义了一个名为BlockAPI的类型，其中包含一个名为blockStat的结构体类型。

blockStat结构体包含一个名为Key的公共字符串字段，一个名为BSize的整数字段，以及一个名为cid的私有字符串字段。

函数Size()返回blockStat结构体的BSize字段，即该BlockAPI类型中包含的块的大小。

函数Path()返回一个路径，将块的ID作为路径的元素。

此代码定义了一个BlockAPI类型，该类型包含一个块的统计信息，如大小和路径。这个类型可能用于记录块的信息，以使系统可以跟踪和统计有关块的数据。


```
type BlockAPI HttpApi

type blockStat struct {
	Key   string
	BSize int `json:"Size"`

	cid cid.Cid
}

func (s *blockStat) Size() int {
	return s.BSize
}

func (s *blockStat) Path() path.ImmutablePath {
	return path.FromCid(s.cid)
}

```

该函数名为 `Put`，作用是创建一个名为 `BlockAPI` 的接口的块（Block）操作（Put）函数。

具体来说，该函数接收一个名为 `ctx` 的 `context.Context`、一个名为 `r` 的 `io.Reader` 和一个包含多个 `caopts.BlockPutOption` 的选项切片 `opts ...caopts.BlockPutOption`。函数首先根据传递的选项切片初始化一个名为 `px` 的变量，该变量表示块的设备类型（MhType）。如果初始化失败，函数返回 `nil` 和相应的错误信息。

接下来，函数检查传入的 `mh` 实例是否已存在，如果不存在，函数返回 `nil` 和错误信息。如果已存在，函数解析传入的 `cid` 选项键（CidPutOption）中的 `mhType` 字段，并检查传入的 `mhlen` 和 `pin` 选项是否正确。然后，函数创建一个名为 `req` 的 `api.core.Request` 实例，并设置请求的 URL 为 "block/put"，同时设置请求参数中的 `mhtype` 字段为传入的 `mh` 实例的 `mhType` 字段的值，`mhlen` 字段的值为传入的 `mh` 实例的 `mhLength` 字段的值，`pin` 选项中的 `pin` 字段为 `options.Pin`。最后，函数创建一个名为 `out` 的 `block.BlockStat` 实例，解码传入的 `cid` 选项数据，然后解析块设备的编码类型。如果解码成功，函数返回 `out` 和 `nil`，否则返回 `nil` 和错误信息。


```
func (api *BlockAPI) Put(ctx context.Context, r io.Reader, opts ...caopts.BlockPutOption) (iface.BlockStat, error) {
	options, err := caopts.BlockPutOptions(opts...)
	px := options.CidPrefix
	if err != nil {
		return nil, err
	}

	mht, ok := mh.Codes[px.MhType]
	if !ok {
		return nil, fmt.Errorf("unknowm mhType %d", px.MhType)
	}

	var cidOptKey, cidOptVal string
	switch {
	case px.Version == 0 && px.Codec == cid.DagProtobuf:
		// ensure legacy --format=v0 passes as BlockPutOption still works
		cidOptKey = "format"
		cidOptVal = "v0"
	default:
		// pass codec as string
		cidOptKey = "cid-codec"
		cidOptVal = mc.Code(px.Codec).String()
	}

	req := api.core().Request("block/put").
		Option("mhtype", mht).
		Option("mhlen", px.MhLength).
		Option(cidOptKey, cidOptVal).
		Option("pin", options.Pin).
		FileBody(r)

	var out blockStat
	if err := req.Exec(ctx, &out); err != nil {
		return nil, err
	}
	out.cid, err = cid.Parse(out.Key)
	if err != nil {
		return nil, err
	}

	return &out, nil
}

```

这段代码定义了一个名为func的函数，接受一个名为api的BlockAPI类型和一个名为path的路径参数。函数的作用是调用BlockAPI的core()方法中的request()方法，并获取请求的响应。

函数的核心部分可以简要描述为：首先，调用api的core()方法中的request("block/get", p.String()).Send(ctx)方法，获取请求的响应。然后，如果响应存在错误，则返回一个非空字符串并调用parseErrNotFoundWithFallbackToError函数。最后，如果响应正确，则返回一个非空字符串，该字符串包含请求的输出。

如果请求返回的是一些数据，则函数会将其复制到一个名为b的缓冲区中。然后，函数返回b，如果调用方在传递参数时遇到任何错误，则返回nil。


```
func (api *BlockAPI) Get(ctx context.Context, p path.Path) (io.Reader, error) {
	resp, err := api.core().Request("block/get", p.String()).Send(ctx)
	if err != nil {
		return nil, err
	}
	if resp.Error != nil {
		return nil, parseErrNotFoundWithFallbackToError(resp.Error)
	}

	// TODO: make get return ReadCloser to avoid copying
	defer resp.Close()
	b := new(bytes.Buffer)
	if _, err := io.Copy(b, resp.Output); err != nil {
		return nil, err
	}

	return b, nil
}

```

此函数的作用是实现 blockapi 内部的一个名为 Rm 的方法。它接收一个名为 api 的 blockapi 实例，一个路径参数 p，以及一个可选的参数集 opts。

函数首先检查给定的选项集是否为空，如果是，则返回一个错误。然后，函数创建一个名为 removedBlock 的结构体，该结构体包含从给定路径的块中删除的块的哈希信息和错误信息。

接下来，函数创建一个名为 req 的请求，该请求设置了一些选项，包括 "force"，它是可选的，使用了给定的选项集，以及路径参数 p。req 然后发送这个请求，并将 removesBlock 结构体中的信息作为请求的 arguments 传递给 caopts.Request。

最后，如果请求成功，函数使用 parseErrNotFoundWithFallbackToMSG 函数，如果请求失败，函数将调用 parseErrNotFoundWithFallbackToMSG 函数，并使用 removesBlock.Error 作为错误消息。


```
func (api *BlockAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.BlockRmOption) error {
	options, err := caopts.BlockRmOptions(opts...)
	if err != nil {
		return err
	}

	removedBlock := struct {
		Hash  string `json:",omitempty"`
		Error string `json:",omitempty"`
	}{}

	req := api.core().Request("block/rm").
		Option("force", options.Force).
		Arguments(p.String())

	if err := req.Exec(ctx, &removedBlock); err != nil {
		return err
	}

	return parseErrNotFoundWithFallbackToMSG(removedBlock.Error)
}

```

该函数`func (api *BlockAPI) Stat(ctx context.Context, p path.Path) (iface.BlockStat, error)`接收一个`path.Path`参数，然后执行以下操作：

1. 创建一个名为`out`的变量，用于存储BlockAPI的Stat结果。
2. 如果调用`api.core().Request("block/stat", p.String()).Exec(ctx, &out)`时出现错误，则将错误传递给`parseErrNotFoundWithFallbackToError`函数，并返回`nil`表示没有结果。
3. 如果`out.Key`解析失败，则将错误传递给`err`变量，并返回`nil`表示没有结果。
4. 创建一个名为`out`的变量，用于存储BlockAPI的Stat结果，并将结果的`cid`字段设置为从`out.Key`解析得到的`cid.Value`字段。
5. 返回`out`和 nil，表示成功执行Stat操作。

该函数的作用是获取BlockAPI的Stat结果，并可能将其存储在`out`变量中。如果执行过程中出现错误，则返回错误，否则返回成功返回的结果。


```
func (api *BlockAPI) Stat(ctx context.Context, p path.Path) (iface.BlockStat, error) {
	var out blockStat
	err := api.core().Request("block/stat", p.String()).Exec(ctx, &out)
	if err != nil {
		return nil, parseErrNotFoundWithFallbackToError(err)
	}
	out.cid, err = cid.Parse(out.Key)
	if err != nil {
		return nil, err
	}

	return &out, nil
}

func (api *BlockAPI) core() *HttpApi {
	return (*HttpApi)(api)
}

```

# `/opt/kubo/client/rpc/dag.go`

这段代码定义了一个名为 `rpc` 的包，包含了一些可以用于实现分布式系统中的客户端与服务端交互的函数和类型定义。

具体来说，该包通过导入多个外部库，包括 `bytes`、`context`、`fmt`、`io`、`github.com/ipfs/boxo/coreiface/options`、`github.com/ipfs/boxo/path`、`github.com/ipfs/go-block-format`、`github.com/ipfs/go-cid`、`github.com/ipfs/go-ipld-format` 和 `github.com/multiformats/go-multicodec`。

该包还定义了一些类型，如 `path.String` 和 `path.Uri`，以及一些函数，如 `fmt.Printf` 和 `fmt.Print`。

最后，该包通过导入 `blocks`、`io`、`github.com/ipfs/boxo/coreiface/options`、`github.com/ipfs/boxo/path`、`github.com/ipfs/go-block-format`、`github.com/ipfs/go-cid` 和 `github.com/ipfs/go-ipld-format` 来支持对 IPFS 对象进行客户端和客户端之间的交互操作。


```
package rpc

import (
	"bytes"
	"context"
	"fmt"
	"io"

	"github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/path"
	blocks "github.com/ipfs/go-block-format"
	"github.com/ipfs/go-cid"
	format "github.com/ipfs/go-ipld-format"
	multicodec "github.com/multiformats/go-multicodec"
)

```

该代码定义了一个名为`HttpDagServ`的`HttpApi`类型，以及一个名为`pinningHttpNodeAdder`的`httpNodeAdder`类型。

该代码还定义了一个名为`Get`的函数，属于`HttpDagServ`类型的`HttpApi`对象。该函数接收一个`ctx`上下文、一个`c``cid`以及一个`Blocks`对象作为参数。函数首先通过调用`api.core().Block().Get`函数，获取一个`path.FromCid`类型的`Blocks`对象，其中`c`参数是`cid`的值。如果该函数有错误，则返回一个`format.Node`类型表示失败，或者返回一个`error`类型。

如果`Get`函数成功，函数将返回一个`format.Node`类型和一个`error`类型。其中`format.Node`类型表示成功，而`error`类型表示在调用`api.ipldDecoder.DecodeNode`函数时失败。如果`DecodeNode`函数有错误，将返回`error`。


```
type (
	httpNodeAdder        HttpApi
	HttpDagServ          httpNodeAdder
	pinningHttpNodeAdder httpNodeAdder
)

func (api *HttpDagServ) Get(ctx context.Context, c cid.Cid) (format.Node, error) {
	r, err := api.core().Block().Get(ctx, path.FromCid(c))
	if err != nil {
		return nil, err
	}

	data, err := io.ReadAll(r)
	if err != nil {
		return nil, err
	}

	blk, err := blocks.NewBlockWithCid(data, c)
	if err != nil {
		return nil, err
	}

	return api.ipldDecoder.DecodeNode(ctx, blk)
}

```

该函数的作用是获取多个异步 HTTP 请求的进度，并返回它们的格式化输出。

具体来说，该函数接收一个 HTTP Dag 服务和多个身份验证信息 (CID)。它创建一个输出通道 out，用于存储格式化输出。然后，它遍历传递给它的 CID 列表。

对于每个 CID，函数会执行一个内部函数，该函数使用 Get 方法从 HTTP Dag 服务获取请求的进度，并将结果存储在 out 通道中。如果请求失败，函数会从 out 通道中消费失败的结果，并在内部失败处理程序中。

最后，函数返回 out 通道，其中包含格式化后的进度信息。


```
func (api *HttpDagServ) GetMany(ctx context.Context, cids []cid.Cid) <-chan *format.NodeOption {
	out := make(chan *format.NodeOption)

	for _, c := range cids {
		// TODO: Consider limiting concurrency of this somehow
		go func(c cid.Cid) {
			n, err := api.Get(ctx, c)

			select {
			case out <- &format.NodeOption{Node: n, Err: err}:
			case <-ctx.Done():
			}
		}(c)
	}
	return out
}

```

这段代码是一个名为`func`的函数，接受一个名为`api`的`httpNodeAdder`类型的参数，以及一个名为`nd`的`Node`类型的参数和一个名为`pin`的布尔类型的参数。

这段代码的作用是用于在HTTP请求中添加一个新的节点。它首先将`nd`的`Cid`字段和`Prefix`字段提取出来，然后根据`Prefix`字段的值尝试从客户端中查找与该节点相关的`cid-codec`编码。如果找到了，它将`cid-codec`编码的字符串作为参数传递给`api.core().Block().Put`函数，其中`options.Block.CidCodec`用于指定`cid-codec`编码。

如果`options.Block.Pin`为`true`，它将在请求中发送一个Pin节点。最后，函数返回一个非`nil`的错误。


```
func (api *httpNodeAdder) add(ctx context.Context, nd format.Node, pin bool) error {
	c := nd.Cid()
	prefix := c.Prefix()

	// preserve 'cid-codec' when sent over HTTP
	cidCodec := multicodec.Code(prefix.Codec).String()

	// 'format' got replaced by 'cid-codec' in https://github.com/ipfs/interface-go-ipfs-core/pull/80
	// but we still support it here for backward-compatibility with use of CIDv0
	format := ""
	if prefix.Version == 0 {
		cidCodec = ""
		format = "v0"
	}

	stat, err := api.core().Block().Put(ctx, bytes.NewReader(nd.RawData()),
		options.Block.Hash(prefix.MhType, prefix.MhLength),
		options.Block.CidCodec(cidCodec),
		options.Block.Format(format),
		options.Block.Pin(pin))
	if err != nil {
		return err
	}
	if !stat.Path().RootCid().Equals(c) {
		return fmt.Errorf("cids didn't match - local %s, remote %s", c.String(), stat.Path().RootCid().String())
	}
	return nil
}

```

此代码定义了一个名为"func"的函数，它接收一个名为"api"的整数类型指针和一个名为"nds"的格式化Node数组作为参数。函数的作用是添加多个给定格式的节点。

函数首先遍历给定的节点列表，然后执行每个节点的前置条件"if err := api.add(ctx, nd, pin); err != nil"。如果添加节点时出现错误，则返回该错误。如果没有错误，则返回 nil。

在函数内部，我们创建了一个名为"httpNodeAdder"的类型指针变量，并将其赋值为整数类型指针变量"api"。我们还定义了一个名为"AddMany"的函数，它与"func"相反，其接收的参数为空格式化节点列表和布尔类型值"false"。我们将其与之前定义的"Add"函数一起作为函数名，以混淆其含义。

我们可以推断出函数的具体实现会在 "httpNodeAdder" 类型的实现中进行优化，例如使用高效的算法来完成添加操作。


```
func (api *httpNodeAdder) addMany(ctx context.Context, nds []format.Node, pin bool) error {
	for _, nd := range nds {
		// TODO: optimize
		if err := api.add(ctx, nd, pin); err != nil {
			return err
		}
	}
	return nil
}

func (api *HttpDagServ) AddMany(ctx context.Context, nds []format.Node) error {
	return (*httpNodeAdder)(api).addMany(ctx, nds, false)
}

func (api *HttpDagServ) Add(ctx context.Context, nd format.Node) error {
	return (*httpNodeAdder)(api).add(ctx, nd, false)
}

```

这段代码定义了一个名为 pinningHttpNodeAdder 的接口，该接口代表了一个 HTTP 节点添加器。同时，还定义了一个名为 pinningHttpNodeAdder 的内部类型，以及一个名为 HttpDagServicing 的接口，该接口代表了一个 HTTP 代理服务器。

具体来说，这段代码实现了一个 HTTP 代理服务器，可以对传入的请求进行处理，包括添加节点和删除节点等操作。在实现中，主要使用了场景为 HTTP/1.1 节点的添加和删除，因此 HTTP/1.1 节点添加器（pinningHttpNodeAdder）的实现主要针对这一场景进行优化。另外，由于 HTTP/1.1 节点在添加和删除时需要保证请求的顺序，因此使用了双重循环的方式来处理请求队列，保证请求的流畅性。


```
func (api *pinningHttpNodeAdder) Add(ctx context.Context, nd format.Node) error {
	return (*httpNodeAdder)(api).add(ctx, nd, true)
}

func (api *pinningHttpNodeAdder) AddMany(ctx context.Context, nds []format.Node) error {
	return (*httpNodeAdder)(api).addMany(ctx, nds, true)
}

func (api *HttpDagServ) Pinning() format.NodeAdder {
	return (*pinningHttpNodeAdder)(api)
}

func (api *HttpDagServ) Remove(ctx context.Context, c cid.Cid) error {
	return api.core().Block().Rm(ctx, path.FromCid(c)) // TODO: should we force rm?
}

```

这段代码定义了一个名为 func 的函数，接收两个参数：一个指向 HttpDagServ 类型的 api 变量和一个字符串数组 cids。函数的作用是移除给定 cids 中的每个 cid，并返回一个 nil 表示成功。

函数内部遍历 cids 并执行 api.Remove 函数。如果执行成功，则返回 nil。如果执行失败，则返回错误。优化部分未实现。


```
func (api *HttpDagServ) RemoveMany(ctx context.Context, cids []cid.Cid) error {
	for _, c := range cids {
		// TODO: optimize
		if err := api.Remove(ctx, c); err != nil {
			return err
		}
	}
	return nil
}

func (api *httpNodeAdder) core() *HttpApi {
	return (*HttpApi)(api)
}

func (api *HttpDagServ) core() *HttpApi {
	return (*HttpApi)(api)
}

```

# `/opt/kubo/client/rpc/dht.go`

该代码是一个名为"rpc"的包，其中定义了一个名为"DhtAPI"的Http API。

具体来说，这个API包括以下功能：

1. 名为"FindPeer"的函数，它接收一个名为"peer.ID"的参数，并返回一个名为"peer.AddrInfo"的类型和一个名为"error"的错误类型的数据。

2. 函数的实现包含以下步骤：

a. 构造一个名为"out"的变量，该变量用于存储JSON响应。

b. 构造一个名为"resp"的变量，该变量用于存储JSON请求的响应。

c. 发送一个名为"dht/findpeer"的请求，该请求携带名为"peer.ID"的参数，并发送到"/rpc/api/v1"。

d. 如果请求成功，从响应中读取JSON数据，并使用名为"json.Decoder"的函数将其解码为所需的结构类型。

e. 对于每一项JSON响应，如果它的类型等价于"routing.QueryEventType"，它将包含一个包含"Type"和"Responses"字段的结构体。如果有一个有效的"Type"，它将包含一个包含"Responses"字段的结构体。

f. 如果所有的JSON响应都是有效的，它将返回包含"Type"字段的"peer.AddrInfo"类型和一个非空"error"类型的变量。

g. 如果任何请求失败，它将返回一个非空"error"类型的变量。


```
package rpc

import (
	"context"
	"encoding/json"

	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/path"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
)

type DhtAPI HttpApi

func (api *DhtAPI) FindPeer(ctx context.Context, p peer.ID) (peer.AddrInfo, error) {
	var out struct {
		Type      routing.QueryEventType
		Responses []peer.AddrInfo
	}
	resp, err := api.core().Request("dht/findpeer", p.String()).Send(ctx)
	if err != nil {
		return peer.AddrInfo{}, err
	}
	if resp.Error != nil {
		return peer.AddrInfo{}, resp.Error
	}
	defer resp.Close()
	dec := json.NewDecoder(resp.Output)
	for {
		if err := dec.Decode(&out); err != nil {
			return peer.AddrInfo{}, err
		}
		if out.Type == routing.FinalPeer {
			return out.Responses[0], nil
		}
	}
}

```

This is a function that uses the DHT (Distributed Hash Table) API to find the DHT providers (peer


```
func (api *DhtAPI) FindProviders(ctx context.Context, p path.Path, opts ...caopts.DhtFindProvidersOption) (<-chan peer.AddrInfo, error) {
	options, err := caopts.DhtFindProvidersOptions(opts...)
	if err != nil {
		return nil, err
	}

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return nil, err
	}

	resp, err := api.core().Request("dht/findprovs", rp.RootCid().String()).
		Option("num-providers", options.NumProviders).
		Send(ctx)
	if err != nil {
		return nil, err
	}
	if resp.Error != nil {
		return nil, resp.Error
	}
	res := make(chan peer.AddrInfo)

	go func() {
		defer resp.Close()
		defer close(res)
		dec := json.NewDecoder(resp.Output)

		for {
			var out struct {
				Extra     string
				Type      routing.QueryEventType
				Responses []peer.AddrInfo
			}

			if err := dec.Decode(&out); err != nil {
				return // todo: handle this somehow
			}
			if out.Type == routing.QueryError {
				return // usually a 'not found' error
				// todo: handle other errors
			}
			if out.Type == routing.Provider {
				for _, pi := range out.Responses {
					select {
					case res <- pi:
					case <-ctx.Done():
						return
					}
				}
			}
		}
	}()

	return res, nil
}

```

此函数的作用是使用DhtAPI提供的功能，接收一个上下文上下文、一个路径参数和一个选项列表作为参数，然后返回DhtAPI提供的结果。

函数首先使用caopts.DhtProvideOptions函数将传入的选项列表参数解析为DhtProvideOption类型的数组，然后检查是否发生错误。如果错误，函数返回该错误。

然后，函数使用api.core().ResolvePath函数查找传递的路径，并检查是否发生错误。如果错误，函数返回该错误。

最后，函数使用api.core().Request函数向DhtAPI提供 "dht/provide" 请求，传递路径和选项，然后设置Option("recursive", options.Recursive)选项以使函数递归。最后，函数使用ctx设置上下文并返回结果。


```
func (api *DhtAPI) Provide(ctx context.Context, p path.Path, opts ...caopts.DhtProvideOption) error {
	options, err := caopts.DhtProvideOptions(opts...)
	if err != nil {
		return err
	}

	rp, _, err := api.core().ResolvePath(ctx, p)
	if err != nil {
		return err
	}

	return api.core().Request("dht/provide", rp.RootCid().String()).
		Option("recursive", options.Recursive).
		Exec(ctx, nil)
}

```

这是一段将`DhtAPI`类型的`api`对象转换为`HttpApi`类型并将结果返回的函数。

具体来说，这段代码定义了一个名为`func`的函数，该函数接收一个`DhtAPI`类型的参数`api`，并返回一个指向`HttpApi`类型的指针。函数实现的过程大致如下：

1. 将`api`类型的参数`api`复制一个`DhtAPI`类型的副本；
2. 使用`(*`指针类型对`api`进行解引用，得到一个指向`DhtAPI`类型的指针；
3. 将得到的指针类型转换为`HttpApi`类型；
4. 返回生成的`HttpApi`类型的指针。

这段代码的作用是将一个`DhtAPI`类型的对象转换为`HttpApi`类型，生成的`HttpApi`类型指针可以用来调用`DhtAPI`类型中提供的`core`方法。


```
func (api *DhtAPI) core() *HttpApi {
	return (*HttpApi)(api)
}

```

# `/opt/kubo/client/rpc/errors.go`

这段代码是一个RPC（远程过程调用）库，定义了一个名为“prePostWrappedNotFoundError”的结构体类型来表示错误。这个错误类型在函数签名中使用，表示在调用者传来的错误信息中，类名和函数名与实际不存在的函数或类不匹配，从而导致产生了错误。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为“prePostWrappedNotFoundError”的结构体类型，该类型包含两个字段：一个是“pre”字段，表示函数签名中存在错误的函数名；另一个是“post”字段，表示函数签名中存在错误的类名。

2. 该结构体类型定义了一个名为“wrapped”的字段，该字段是一个IPLD（JSON低层编码）错误对象。这个错误对象可以用来将普通错误信息转换为包含类名和函数名的错误信息，从而使得函数调用者更容易地理解和处理错误信息。

3. 在函数“parseFromError”中，创建了一个新的“prePostWrappedNotFoundError”类型的实例，设置其“wrapped”字段的值，然后使用该实例的“pre”和“post”字段来查找类名和函数名，如果找到，则执行“notFound”方法，抛出一个“prePostWrappedNotFoundError”类型的错误。如果没有找到，则执行普通错误处理流程，抛出一个“未定义的错误”错误。

4. 在函数“parseFromErrorCode”中，创建了一个新的“prePostWrappedNotFoundError”类型的实例，设置其“wrapped”字段的值，然后使用该实例的“pre”和“post”字段来查找类名和函数名，如果找到，则执行“notFound”方法，抛出一个“prePostWrappedNotFoundError”类型的错误。如果没有找到，则执行普通错误处理流程，抛出一个“未定义的错误”错误。

5. 在函数“parseFromMultiErrorCode”中，创建了一个新的“prePostWrappedNotFoundError”类型的实例，设置其“wrapped”字段的值，然后使用该实例的“pre”和“post”字段来查找类名和函数名，如果找到，则执行“notFound”方法，抛出一个“prePostWrappedNotFoundError”类型的错误。如果没有找到，则执行普通错误处理流程，抛出一个“未定义的错误”错误。


```
package rpc

import (
	"errors"
	"strings"
	"unicode/utf8"

	"github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	mbase "github.com/multiformats/go-multibase"
)

// This file handle parsing and returning the correct ABI based errors from error messages

type prePostWrappedNotFoundError struct {
	pre  string
	post string

	wrapped ipld.ErrNotFound
}

```

这段代码定义了一个名为`parseErrNotFoundWithFallbackToMSG`的函数，它接收一个`msg`参数，表示错误消息。函数的作用是处理`msg`这个错误消息，如果已经定义了一个名为`handleError`的函数，则将其包装成一个`prePostWrappedNotFoundError`类型的对象，并返回该对象。如果尚未定义`handleError`函数，则返回一个自定义类型的错误对象，包含原始错误对象的`Error`方法和`pre`、`wrapped`和`post`字段，再将`Error`类型转换为字符串类型并返回。

具体来说，这段代码可以拆解为以下几个部分：

1. `func (e prePostWrappedNotFoundError) String() string` 定义了一个名为`String`的函数，接收一个`e`参数，它是一个`prePostWrappedNotFoundError`类型的对象，函数返回`e`的`Error`字段的`Error`类型。
2. `func (e e.Error) Error() string` 定义了一个名为`Error`的函数，接收一个`e`参数，它是一个`Error`类型的对象，函数返回`e`的`Error`字段的`Error`类型。
3. `func (e e.Error) Unwrap() error` 定义了一个名为`Unwrap`的函数，它接收一个`e`参数，与`Error`函数类似，返回`e`的`Error`字段的`Error`类型。
4. `func parseErrNotFoundWithFallbackToMSG(msg string) error` 定义了一个名为`parseErrNotFoundWithFallbackToMSG`的函数，它接收一个`msg`参数，表示错误消息。函数的作用是处理`msg`这个错误消息，如果已经定义了一个名为`handleError`的函数，则将其包装成一个`prePostWrappedNotFoundError`类型的对象，并返回该对象。如果尚未定义`handleError`函数，则返回一个自定义类型的错误对象，包含原始错误对象的`Error`方法的`Error`类型转换为字符串类型并返回。


```
func (e prePostWrappedNotFoundError) String() string {
	return e.Error()
}

func (e prePostWrappedNotFoundError) Error() string {
	return e.pre + e.wrapped.Error() + e.post
}

func (e prePostWrappedNotFoundError) Unwrap() error {
	return e.wrapped
}

func parseErrNotFoundWithFallbackToMSG(msg string) error {
	err, handled := parseErrNotFound(msg)
	if handled {
		return err
	}

	return errors.New(msg)
}

```

此代码定义了一个名为 `parseErrNotFoundWithFallbackToError` 的函数，用于处理在给定 `msg` 错误对象中找不到解析器的情况下，通过调用其他函数或传递给函数的参数来返回一个 `msg` 错误对象或者返回一些其他信息。

函数有两个参数，一个是 `msg` 错误对象，另一个是 `bool` 类型的变量 `handled`。函数首先尝试使用 `parseIPLDErrNotFound` 函数来解析 `msg` 中的错误对象。如果 `handled` 变量为 `true`(即解析器成功解析了 `msg` 中的错误对象)，则函数返回 `err` 和 `handled` 中的 `true` 值。否则，函数将返回 `msg` 错误对象。

接下来，函数将调用 `parseBlockstoreNotFound` 函数来继续尝试解析 `msg` 中的错误对象。如果 `handled` 变量为 `true`(即解析器成功解析了 `msg` 中的错误对象)，则函数返回 `err` 和 `handled` 中的 `true` 值。否则，函数将返回 `msg` 错误对象。

最后，如果以上两种情况中的任意一种都没有成功解析 `msg` 中的错误对象，函数将返回 `nil` 作为第一个参数，并将 `handled` 变量设置为 `false`。


```
func parseErrNotFoundWithFallbackToError(msg error) error {
	err, handled := parseErrNotFound(msg.Error())
	if handled {
		return err
	}

	return msg
}

func parseErrNotFound(msg string) (error, bool) {
	if msg == "" {
		return nil, true // Fast path
	}

	if err, handled := parseIPLDErrNotFound(msg); handled {
		return err, true
	}

	if err, handled := parseBlockstoreNotFound(msg); handled {
		return err, true
	}

	return nil, false
}

```

This is a function that checks whether a given `msgPostKey` string is a valid CID (C伊斯） value. It takes into account whether the string starts with the prefix "node" and, if it does, it has a fallback case where it is assumed to be a default value for `cid.Undef`. Otherwise, it looks for any instances of the character `cidBreakSet` in the `msgPostKey` string and returns the index of such a occurrence as the postindex.

If the postindex is found, the function decodes the given CID string using the `cid.Decode` function. It then checks whether the CID is a CIDv0 value or a base32 multibase. If the CID is not a multibase, the function returns `nil` and a boolean value indicating that the CID is not a valid CID value. If the CID is a multibase, the function returns the decoded value, a wrapped error object, and a boolean value indicating that the CID was found.

If the `ipld.ErrNotFound.Error` function is called with the `Cid` field set, the function returns `nil` and a boolean value indicating that the CID was not found. If either the `postIndex` or the `postEntity` field is given, the function returns the error with the appropriate message.


```
// Assume CIDs break on:
// - Whitespaces: " \t\n\r\v\f"
// - Semicolon: ";" this is to parse ipld.ErrNotFound wrapped in multierr
// - Double Quotes: "\"" this is for parsing %q and %#v formating.
const cidBreakSet = " \t\n\r\v\f;\""

func parseIPLDErrNotFound(msg string) (error, bool) {
	// The patern we search for is:
	const ipldErrNotFoundKey = "ipld: could not find " /*CID*/
	// We try to parse the CID, if it's invalid we give up and return a simple text error.
	// We also accept "node" in place of the CID because that means it's an Undefined CID.

	keyIndex := strings.Index(msg, ipldErrNotFoundKey)

	if keyIndex < 0 { // Unknown error
		return nil, false
	}

	cidStart := keyIndex + len(ipldErrNotFoundKey)

	msgPostKey := msg[cidStart:]
	var c cid.Cid
	var postIndex int
	if strings.HasPrefix(msgPostKey, "node") {
		// Fallback case
		c = cid.Undef
		postIndex = len("node")
	} else {
		postIndex = strings.IndexFunc(msgPostKey, func(r rune) bool {
			return strings.ContainsAny(string(r), cidBreakSet)
		})
		if postIndex < 0 {
			// no breakage meaning the string look like this something + "ipld: could not find bafy"
			postIndex = len(msgPostKey)
		}

		cidStr := msgPostKey[:postIndex]

		var err error
		c, err = cid.Decode(cidStr)
		if err != nil {
			// failed to decode CID give up
			return nil, false
		}

		// check that the CID is either a CIDv0 or a base32 multibase
		// because that what ipld.ErrNotFound.Error() -> cid.Cid.String() do currently
		if c.Version() != 0 {
			baseRune, _ := utf8.DecodeRuneInString(cidStr)
			if baseRune == utf8.RuneError || baseRune != mbase.Base32 {
				// not a multibase we expect, give up
				return nil, false
			}
		}
	}

	err := ipld.ErrNotFound{Cid: c}
	pre := msg[:keyIndex]
	post := msgPostKey[postIndex:]

	if len(pre) > 0 || len(post) > 0 {
		return prePostWrappedNotFoundError{
			pre:     pre,
			post:    post,
			wrapped: err,
		}, true
	}

	return err, true
}

```

这段代码定义了一个名为`blockstoreNotFoundMatchingIPLDErrNotFound`的结构体类型，它包含一个名为`msg`的静态字段和一个名为`Error`的静态字段。

这两个字段都有特殊的含义。`msg`字段将作为`blockstoreNotFoundMatchingIPLDErrNotFound`类型的实例的`String()`方法的返回值，它将`e`的`Error()`方法的返回值作为字符串返回。同样，当`Is(err)`方法返回`err`为`err.NotFound`时，将返回`blockstoreNotFoundMatchingIPLDErrNotFound`类型的实例，此时`Error()`方法的返回值将被作为字符串返回。

另外，该结构体还包含一个名为`String()`的静态方法，用于将`e`的`Error()`方法的返回值作为字符串返回。


```
// This is a simple error type that just return msg as Error().
// But that also match ipld.ErrNotFound when called with Is(err).
// That is needed to keep compatiblity with code that use string.Contains(err.Error(), "blockstore: block not found")
// and code using ipld.ErrNotFound.
type blockstoreNotFoundMatchingIPLDErrNotFound struct {
	msg string
}

func (e blockstoreNotFoundMatchingIPLDErrNotFound) String() string {
	return e.Error()
}

func (e blockstoreNotFoundMatchingIPLDErrNotFound) Error() string {
	return e.msg
}

```

此代码定义了两个函数，分别是 `func` 和 `func`。它们的作用是：

1. `func(e blockstoreNotFoundMatchingIPLDErrNotFound) Is(err error) bool` 的作用是判断是否为 `ipld.ErrNotFound` 类型的错误，如果是，则返回 `true`，否则返回 `false`。`e` 代表一个错误参数，可以是任何类型的错误，但在这里被用来传递 `blockstoreNotFoundMatchingIPLDErrNotFound`。
2. `func parseBlockstoreNotFound(msg string) (error, bool)` 的作用是判断 `msg` 参数是否包含 `"blockstore: block not found"`，如果是，返回 `blockstoreNotFoundMatchingIPLDErrNotFound` 和 `true`，否则返回 `nil` 和 `false`。首先，使用 `strings.Contains` 函数检查 `msg` 参数是否包含 `"blockstore: block not found"`，如果是，执行 `blockstoreNotFoundMatchingIPLDErrNotFound` 和 `true` 的代码块。


```
func (e blockstoreNotFoundMatchingIPLDErrNotFound) Is(err error) bool {
	_, ok := err.(ipld.ErrNotFound)
	return ok
}

func parseBlockstoreNotFound(msg string) (error, bool) {
	if !strings.Contains(msg, "blockstore: block not found") {
		return nil, false
	}

	return blockstoreNotFoundMatchingIPLDErrNotFound{msg: msg}, true
}

```

# `/opt/kubo/client/rpc/errors_test.go`

该代码定义了一个名为“rpc”的包，并导入了多个外部库，包括“errors”、“fmt”和“testing”。

接下来，定义了一个名为“randomSha256MH”的变量，该变量是一个“Multihash”类型的变量，并设置了一些值。

具体来说，“randomSha256MH”变量中包含了一个“Multihash”类型的值为：

0x12：二进制表示法为“00001212”。

0x20：二进制表示法为“00002020”。

0x88：二进制表示法为“00008800”。

0x82：二进制表示法为“00008200”。

0x73：二进制表示法为“00007300”。

0x37：二进制表示法为“00003700”。

0x7c：二进制表示法为“00007c00”。

0xc1：二进制表示法为“0000c100”。

0xc9：二进制表示法为“0000c900”。

0x96：二进制表示法为“00009600”。

0xad：二进制表示法为“0000ad00”。

0xee：二进制表示法为“0000ee00”。

0xd：二进制表示法为“0000d000”。

0x26：二进制表示法为“00002600”。

0x84：二进制表示法为“00008400”。

0x2：二进制表示法为“00002”。

0xc9：二进制表示法为“0000c900”。

0xc9：二进制表示法为“0000c900”。

0x5c：二进制表示法为“00005c00”。

0xf9：二进制表示法为“0000f900”。

0x5c：二进制表示法为“00005c00”。

0xf9：二进制表示法为“0000f900”。

0x4d：二进制表示法为“00004d00”。

0x9b：二进制表示法为“00009b00”。

0xc3：二进制表示法为“00003c00”。

0x3f：二进制表示法为“00003f00”。

0xfb：二进制表示法为“0000fb00”。

0x4a：二进制表示法为“00004a00”。

0xd8：二进制表示法为“0000d800”。

0xaf：二进制表示法为“0000af00”。

0x28：二进制表示法为“00002800”。

0x6b：二进制表示法为“00006b00”。

0xca：二进制表示法为“0000ca00”。

0x1a：二进制表示法为“00001a00”。

0xf2：二进制表示法为“0000f200”。

最后，还有一句“螯合负载RPC消息”的注释。


```
package rpc

import (
	"errors"
	"fmt"
	"testing"

	"github.com/ipfs/go-cid"
	ipld "github.com/ipfs/go-ipld-format"
	mbase "github.com/multiformats/go-multibase"
	mh "github.com/multiformats/go-multihash"
)

var randomSha256MH = mh.Multihash{0x12, 0x20, 0x88, 0x82, 0x73, 0x37, 0x7c, 0xc1, 0xc9, 0x96, 0xad, 0xee, 0xd, 0x26, 0x84, 0x2, 0xc9, 0xc9, 0x5c, 0xf9, 0x5c, 0x4d, 0x9b, 0xc3, 0x3f, 0xfb, 0x4a, 0xd8, 0xaf, 0x28, 0x6b, 0xca, 0x1a, 0xf2}

```

此代码定义了一个名为 "doParseIpldNotFoundTest" 的函数，它是 testing.T 类型的函数，它接收两个参数：一个 testing.T 类型的实例和一个原始错误。

函数的作用是测试 ipld.IsNotFound 函数在遇到 "Ipld.NotFound" 错误时，是否可以正确地传递原始错误并生成一个正确的错误消息。

具体来说，函数首先创建一个名为 "originalMsg" 的原始错误实例，然后使用 "parseErrNotFoundWithFallbackToMSG" 函数将原始错误转换为一个表示 "Ipld.NotFound" 错误消息的新的错误对象。接着，函数使用 "rebuilt" 变量保存这个新的错误对象，并使用 "rebuiltMsg" 变量获取这个新的错误对象的错误消息。

接下来，函数创建一个名为 "originalNotFound" 的原始错误实例，并使用 ipld.IsNotFound 函数检查原始错误是否为 "Ipld.NotFound"。然后，函数创建一个名为 "rebuiltNotFound" 的错误实例，并使用 ipld.IsNotFound 函数检查重建的错误是否为 "Ipld.NotFound"。

最后，函数使用 if 语句检查原始错误和重建的错误是否相等。如果不相等，函数将会创建一个错误消息并使用 "t.Errorf" 函数打印错误消息，这将导致函数的测试失败。


```
func doParseIpldNotFoundTest(t *testing.T, original error) {
	originalMsg := original.Error()

	rebuilt := parseErrNotFoundWithFallbackToMSG(originalMsg)

	rebuiltMsg := rebuilt.Error()

	if originalMsg != rebuiltMsg {
		t.Errorf("expected message to be %q; got %q", originalMsg, rebuiltMsg)
	}

	originalNotFound := ipld.IsNotFound(original)
	rebuiltNotFound := ipld.IsNotFound(rebuilt)
	if originalNotFound != rebuiltNotFound {
		t.Errorf("for %q expected Ipld.IsNotFound to be %t; got %t", originalMsg, originalNotFound, rebuiltNotFound)
	}
}

```

This appears to be a function written in Go that takes in a CID set and a string containing a CID wrap. It checks for errors that may occur with the CID wraps, such as invalid CIDs or network connection timeouts. If any errors are found, it logs them and returns a "FoundWithFallbackToMSG" error.

The function starts by creating an array of wraps for each CID in the CID set. It then loops through each wrap and attempts to parse the corresponding IPLD test for that CID. If an error is found, it logs it and returns a "FoundWithFallbackToMSG" error. If no errors are found, it successfully parses all the IPLD tests and returns a "FoundWithNoFallbackToMSG" error.

It is worth noting that the function also includes a block of commented out code indicating that it was added to try to fix issues with the original code that this function was added to.


```
func TestParseIPLDNotFound(t *testing.T) {
	t.Parallel()

	if err := parseErrNotFoundWithFallbackToMSG(""); err != nil {
		t.Errorf("expected empty string to give no error; got %T %q", err, err.Error())
	}

	cidBreaks := make([]string, len(cidBreakSet))
	for i, v := range cidBreakSet {
		cidBreaks[i] = "%w" + string(v)
	}

	base58BTCEncoder, err := mbase.NewEncoder(mbase.Base58BTC)
	if err != nil {
		t.Fatalf("expected to find Base58BTC encoder; got error %q", err.Error())
	}

	for _, wrap := range append(cidBreaks,
		"",
		"merkledag: %w",
		"testing: %w the test",
		"%w is wrong",
	) {
		for _, err := range [...]error{
			errors.New("ipld: could not find "),
			errors.New("ipld: could not find Bad_CID"),
			errors.New("ipld: could not find " + cid.NewCidV1(cid.Raw, randomSha256MH).Encode(base58BTCEncoder)), // Test that we only accept CIDv0 and base32 CIDs
			errors.New("network connection timeout"),
			ipld.ErrNotFound{Cid: cid.Undef},
			ipld.ErrNotFound{Cid: cid.NewCidV0(randomSha256MH)},
			ipld.ErrNotFound{Cid: cid.NewCidV1(cid.Raw, randomSha256MH)},
		} {
			if wrap != "" {
				err = fmt.Errorf(wrap, err)
			}

			doParseIpldNotFoundTest(t, err)
		}
	}
}

```

这段代码是一个名为 `TestBlockstoreNotFoundMatchingIPLDErrNotFound` 的函数测试，它旨在测试 ipld 包中 `blockstoreNotFoundMatchingIPLDErrNotFound` 函数的行为。

首先，函数会检查 `ipld.IsNotFound(blockstoreNotFoundMatchingIPLDErrNotFound{})` 是否为真，如果是，则执行以下操作：

1. 打印错误信息并使用 `t.Fatalf` 函数断言。
2. 运行测试时使用 `ipld` 包中的 `isNotFound` 函数，并将其返回值与上面打印的错误信息进行匹配，如果匹配，则认为 `ipld.IsNotFound(blockstoreNotFoundMatchingIPLDErrNotFound{})` 函数能够正确地返回 `true`。

然后，函数遍历一系列不同的 `wrap` 参数，其中 `wrap` 是一个字符串数组，用于在每次运行测试时打印不同的错误信息。对于每个 `wrap` 参数，函数都会遍历其中的所有错误信息，并对其执行以下操作：

1. 如果 `wrap` 不为空，则将错误信息中的 `%w` 字段设置为 `err` 对象，并将 `%w` 作为格式化字符串的模板。
2. 调用 `doParseIpldNotFoundTest` 函数，该函数将错误信息作为 `testing.T` 类型的参数传递，并将 `err` 对象作为 `%w` 字段的值返回。
3. 重复执行步骤 1-2，直到所有错误信息都被打印出来并进行了 `%w` 字段的格式化。


```
func TestBlockstoreNotFoundMatchingIPLDErrNotFound(t *testing.T) {
	t.Parallel()

	if !ipld.IsNotFound(blockstoreNotFoundMatchingIPLDErrNotFound{}) {
		t.Fatalf("expected blockstoreNotFoundMatchingIPLDErrNotFound to match ipld.IsNotFound; got false")
	}

	for _, wrap := range [...]string{
		"",
		"merkledag: %w",
		"testing: %w the test",
		"%w is wrong",
	} {
		for _, err := range [...]error{
			errors.New("network connection timeout"),
			blockstoreNotFoundMatchingIPLDErrNotFound{"blockstore: block not found"},
		} {
			if wrap != "" {
				err = fmt.Errorf(wrap, err)
			}

			doParseIpldNotFoundTest(t, err)
		}
	}
}

```

# `/opt/kubo/client/rpc/key.go`

这段代码定义了一个名为 "KeyAPI" 的 HTTP API，通过使用 libp2p 库实现。它使用了 libp2p 的Peer API 来与对等网络中的其他节点进行通信。下面是这段代码的一些主要部分：

1. 引入了必要的库：`rpc`、`github.com/ipfs/boxo/coreiface`、`github.com/ipfs/boxo/coreiface/options`、`github.com/ipfs/boxo/ipns`、`github.com/libp2p/go-libp2p/core/peer`。
2. 定义了一个名为 "KeyAPI" 的类型，该类型包含一个代表 HTTP API 的函数 "HTTPGet". 函数的参数是一个字符串 "http://"。
3. 定义了 HTTP API 的函数：`KeyAPI` 类型中的 "HTTPGet" 函数，该函数使用 `ctx.鼎炬措ulent` 从请求中获取参数。它还设置了一些选项，如 "path"、"http" 和 "user:password"。
4. 导入了一些来自 `github.com/ipfs/boxo/coreiface` 和 `github.com/ipfs/boxo/coreiface/options` 的变量。
5. 导入了一些来自 `github.com/libp2p/go-libp2p/core` 和 `github.com/libp2p/go-libp2p/core/peer` 的变量。
6. 定义了一个名为 "KeyAPI" 的函数指针类型。
7. 没有其他明显的部分。


```
package rpc

import (
	"context"
	"errors"

	iface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/path"
	"github.com/libp2p/go-libp2p/core/peer"
)

type KeyAPI HttpApi

```

该代码定义了一个名为“key”的结构体类型，该类型包含一个字符串类型的“name”字段和一个“pid”字段，以及一个“path”字段，该字段是一个路径类型。

接着，该代码实现了一个名为“newKey”的函数，该函数接收两个参数，一个是“name”字符串类型，另一个是“pid”字符串类型。函数先将“pid”参数解码为“pid”ID类型，然后创建一个路径类型的“path”，该路径是“/ipns/”目录下的子路径，该路径将包含一个由“ipns”名称从“pid”ID获取的名称。最后，函数创建一个名为“key”的结构体实例，该实例包含一个名为“name”的字符串类型变量和一个名为“pid”的“pid”ID类型变量，以及一个名为“path”的路径类型变量。如果函数在创建“path”时出现错误，则会返回一个空结构体“nil”或者一个非空错误“err”。如果函数在创建“key”实例时出现错误，则会返回一个空结构体“nil”或者一个非空错误“err”。


```
type key struct {
	name string
	pid  peer.ID
	path path.Path
}

func newKey(name, pidStr string) (*key, error) {
	pid, err := peer.Decode(pidStr)
	if err != nil {
		return nil, err
	}

	path, err := path.NewPath("/ipns/" + ipns.NameFromPeer(pid).String())
	if err != nil {
		return nil, err
	}

	return &key{name: name, pid: pid, path: path}, nil
}

```

这是一个 Go 语言中的接口类型，定义了三个函数，分别返回一个名为 `k` 的 `key` 对象的名称、路径和 `ID`。同时，定义了一个名为 `keyOutput` 的类型，该类型包含两个字段，分别为 `Name` 和 `Id`。

具体来说，这段代码定义了一个名为 `func` 的函数，它接收一个名为 `k` 的 `key` 参数，并返回该 `key` 的名称。接着，定义了两个名为 `func` 的函数，它们分别接收一个名为 `k` 的 `key` 参数，并返回该 `key` 的路径和 `ID`。同时，定义了一个名为 `keyOutput` 的类型，该类型包含两个字段，分别为 `Name` 和 `Id`。

总的来说，这段代码定义了一个 `key` 接口类型，以及多个函数，用于获取和设置 `key` 对象的属性。


```
func (k *key) Name() string {
	return k.name
}

func (k *key) Path() path.Path {
	return k.path
}

func (k *key) ID() peer.ID {
	return k.pid
}

type keyOutput struct {
	Name string
	Id   string
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `api` 的整数类型的指针变量和一个字符串类型的参数 `name`，以及一个可选的参数 `opts`，该参数是一个由 `caopts.KeyGenerateOption` 构成的数组。

函数的作用是使用 `api.core().Request` 方法生成密钥，并返回生成的密钥的名称和 ID。

具体来说，函数首先通过调用 `caopts.KeyGenerateOptions` 函数，将传递给它的 `opts` 参数充实到 `caopts.KeyGenerateOption` 类型的数组中。然后，函数使用这个数组作为参数，通过调用 `api.core().Request` 方法并传入参数 `"key/gen"`、`name` 和 `options`，请求 API 服务器生成一个密钥。

如果服务器在生成密钥时出现错误，函数将返回一个非空字符串 `nil` 和一个错误对象 `err`，此时函数返回的 `out` 变量将不再有意义。

函数返回的 `newKey` 函数会根据生成的密钥的名称和 ID 创建一个新的 `Key` 对象，可供后续使用。


```
func (api *KeyAPI) Generate(ctx context.Context, name string, opts ...caopts.KeyGenerateOption) (iface.Key, error) {
	options, err := caopts.KeyGenerateOptions(opts...)
	if err != nil {
		return nil, err
	}

	var out keyOutput
	err = api.core().Request("key/gen", name).
		Option("type", options.Algorithm).
		Option("size", options.Size).
		Exec(ctx, &out)
	if err != nil {
		return nil, err
	}

	return newKey(out.Name, out.Id)
}

```

这段代码定义了一个名为 `func` 的函数，它接受一个名为 `api` 的整数类型的参数，并返回一个名为 `Key` 的接口类型的变量。函数的作用是执行对一个键的重新命名操作。

函数接受四个参数，其中第一个参数是一个 `ctx` 类型的上下文对象，用于组织和传递请求的信息。第二个参数是一个字符串类型的参数，用于指定要更新的键的旧名称。第三个参数是一个字符串类型的参数，用于指定要更新的键的新名称。最后一个参数是一个可传递给 `caopts.KeyRenameOption` 函数的选项参数数组，用于指定键重新命名的选项。

函数的实现包括以下步骤：

1. 调用 `caopts.KeyRenameOptions` 函数，将传入的选项参数传递给该函数，并从函数返回值中获取状态信息。
2. 创建一个名为 `out` 的 `struct` 类型的变量，用于存储重新命名的结果信息。
3. 调用 `api.core().Request` 函数，并传递以下参数：
	- `"key/rename"`: 请求的 API 路径。
	- `oldName`: 要更新的键的旧名称。
	- `newName`: 要更新的键的新名称。
	- `opts...`: 可传递给 `caopts.KeyRenameOption` 函数的选项参数数组。
	- `force`: 这是一个可选的参数，用于指示是否强制执行操作。
	- `ctx`: 这是一个可选的参数，用于指定请求上下文。
	- `&out`: 获取输出参数的引用。
	- `.Exec`: 执行请求并获取结果。
4. 如果函数在请求过程中出现错误，函数将返回 `nil`。如果成功，函数将返回一个 `Key` 类型表示的值，其中包含重新命名的键的信息。
5. 函数返回结果，以便后续使用。


```
func (api *KeyAPI) Rename(ctx context.Context, oldName string, newName string, opts ...caopts.KeyRenameOption) (iface.Key, bool, error) {
	options, err := caopts.KeyRenameOptions(opts...)
	if err != nil {
		return nil, false, err
	}

	var out struct {
		Was       string
		Now       string
		Id        string
		Overwrite bool
	}
	err = api.core().Request("key/rename", oldName, newName).
		Option("force", options.Force).
		Exec(ctx, &out)
	if err != nil {
		return nil, false, err
	}

	key, err := newKey(out.Now, out.Id)
	if err != nil {
		return nil, false, err
	}

	return key, out.Overwrite, err
}

```

该函数的作用是获取API的`key/list`endpoint的输出结构体，并返回它。函数接收一个`KeyAPI`类型的参数`api`，并使用该参数的`core().Request`方法发送请求。如果请求成功，函数使用`ctx.Context().Standard(http.Request)`设置的上下文执行回调函数体，并返回一个包含所请求键的输出结构体`res`。如果请求失败，函数返回一个包含错误信息的空字符串`nil`。


```
func (api *KeyAPI) List(ctx context.Context) ([]iface.Key, error) {
	var out struct {
		Keys []keyOutput
	}
	if err := api.core().Request("key/list").Exec(ctx, &out); err != nil {
		return nil, err
	}

	res := make([]iface.Key, len(out.Keys))
	for i, k := range out.Keys {
		key, err := newKey(k.Name, k.Id)
		if err != nil {
			return nil, err
		}
		res[i] = key
	}

	return res, nil
}

```

这两函数分别实现了 KeyAPI 的两个核心功能：自我检查和删除键。

函数1：实现了 Self 功能。它的作用是获取自身的公钥，并返回该公钥。具体实现过程如下：

1. 解析 `api.core().Request("id").Exec(ctx, &id)` 并请求获取一个 ID 公钥。
2. 如果没有成功获取公钥，就返回 `nil` 和相应的错误。
3. 返回一个新的 `iface.Key` 类型的值，该值包含一个自定义键的名称和该自定义键的 ID。

函数2：实现了 Remove 功能。它的作用是删除指定的键，并返回该键。具体实现过程如下：

1. 解析 `api.core().Request("key/rm", name).Exec(ctx, &out)` 并请求删除指定的键。
2. 如果请求成功，解析返回值并获取到该键的 ID。
3. 返回一个新的 `iface.Key` 类型的值，该值包含该键的名称和 ID。
4. 如果请求失败，返回 `nil` 和相应的错误。


```
func (api *KeyAPI) Self(ctx context.Context) (iface.Key, error) {
	var id struct{ ID string }
	if err := api.core().Request("id").Exec(ctx, &id); err != nil {
		return nil, err
	}

	return newKey("self", id.ID)
}

func (api *KeyAPI) Remove(ctx context.Context, name string) (iface.Key, error) {
	var out struct {
		Keys []keyOutput
	}
	if err := api.core().Request("key/rm", name).Exec(ctx, &out); err != nil {
		return nil, err
	}
	if len(out.Keys) != 1 {
		return nil, errors.New("got unexpected number of keys back")
	}

	return newKey(out.Keys[0].Name, out.Keys[0].Id)
}

```

该函数是一个名为"func"的内部函数，它接收一个名为"api"的整数类型的指针变量，并返回一个名为"HttpApi"的指向整数类型指针类型的指针变量。

函数体中，首先通过"*"运算符获取了"api"的值，并将其存储到整数变量"return"中。接着，通过"(*"运算符获取了"api"指向的整数类型的值，并将其存储到整数变量"return"中。最后，通过"return"将返回的整数类型指针变量返回。

该函数的作用是实现了一个通用的函数，接收一个整数类型的指针变量，返回一个整数类型的指针变量，用于将整数类型的指针变量转换为对应的HttpApi类型。


```
func (api *KeyAPI) core() *HttpApi {
	return (*HttpApi)(api)
}

```