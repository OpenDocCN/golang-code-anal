# go-ipfs 源码解析 54

# `/opt/kubo/test/cli/content_routing_http_test.go`

该代码是一个 Go 语言编写的 CLI 工具，用于在测试中使用 libp2p 和 boxo 库。具体来说，该代码包括以下主要部分：

1. 导入需要使用的库：
python
import (
	"context"
	"net/http"
	"net/http/httptest"
	"os/exec"
	"sync"
	"testing"
	"time"

	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/routing/http/server"
	"github.com/ipfs/boxo/routing/http/types"
	"github.com/ipfs/boxo/routing/http/types/iter"
	"github.com/ipfs/boxo/routing/http/types/params"
	"github.com/ipfs/boxo/routing/http/transport/目录"
	"github.com/ipfs/boxo/services/目录"
	"github.com/ipfs/boxo/services/diagnostics"
	"github.com/ipfs/boxo/services/Eventer"
	"github.com/ipfs/boxo/services/log"
	"github.com/ipfs/boxo/services/配置"
	"github.com/ipfs/boxo/services/注册"
	"github.com/ipfs/boxo/services/亭台"
	"github.com/ipfs/boxo/services/store/RocksDB"
	"github.com/ipfs/boxo/services/store/Codec"
	"github.com/ipfs/boxo/services/store/霍锦物联网"
	"github.com/ipfs/boxo/services/store/连接"
	"github.com/ipfs/boxo/services/store/路段"
	"github.com/ipfs/boxo/services/store/随从"
	"github.com/ipfs/boxo/services/store/生态"
	"github.com/ipfs/boxo/services/store/提供者"
	"github.com/ipfs/boxo/services/store/ UniqueSuffix"
	"github.com/ipfs/boxo/services/store/争端"
	"github.com/ipfs/boxo/services/store/恢复"
	"github.com/ipfs/boxo/services/store/验证"
	"github.com/ipfs/boxo/services/store/归档"
	"github.com/ipfs/boxo/services/store/提取/盒子提取器"
	"github.com/ipfs/boxo/services/store/脚本/盒子运行时"
	"github.com/ipfs/boxo/services/store/跳表/跳表模拟"
	"github.com/ipfs/boxo/services/store/基数/RocksDB"
	"github.com/ipfs/boxo/services/store/树/二叉搜索树"
	"github.com/ipfs/boxo/services/store/ Merkle/构建Merkle树"
	"github.com/ipfs/boxo/services/store/ 从树/提取Index"
	"github.com/ipfs/boxo/services/store/ 从树/提取验证"
	"github.com/ipfs/boxo/services/store/ 从树/提取最近/expiration"
	"github.com/ipfs/boxo/services/store/ FromChainlink/linktoc"
	"github.com/ipfs/boxo/services/store/ FromChainlink/pingtoc"
	"github.com/ipfs/boxo/services/store/广播/ IPFSDiscovery"
	"github.com/ipfs/boxo/services/store/广播/封禁抑制"
	"github.com/ipfs/boxo/services/store/广播/时间轮"
	"github.com/ipfs/boxo/services/store/广播/改善"
	"github.com/ipfs/boxo/services/store/广播/保留"
	"github.com/ipfs/boxo/services/store/广播/已验证"
	"github.com/ipfs/boxo/services/store/广播/寻找/Kubernetes"
	"github.com/ipfs/boxo/services/store/版本控制/Git"
	"github.com/ipfs/boxo/services/store/文件操作/静态节点"
	"github.com/ipfs/boxo/services/store/文件操作/操作员"
	"github.com/ipfs/boxo/services/store/文件操作/注册/石头注册"
	"github.com/ipfs/boxo/services/store/解压缩/BoxoDecompress"
	"github.com/ipfs/boxo/services/store/解压缩/解压缩"
	"github.com/ipfs/boxo/services/store/架构"
	"github.com/ipfs/boxo/services/store/由"
	"github.com/ipfs/boxo/services/store/ 由/files"
	"github.com/ipfs/boxo/services/store/ 由/walker/files"
	"github.com/ipfs/boxo/services/store/ 由/workflow"
	"github.com/ipfs/boxo/services/store/由/哈希"
	"github.com/ipfs/boxo/services/store/由/编号"
	"github.com/ipfs/boxo/services/store/由/删除"
	"github.com/ipfs/boxo/services/store/由国家/人民/保持"
	"github.com/ipfs/boxo/services/store/由国家/人民/宣传"
	"github.com/ipfs/boxo/services/store/由国家/人民/身份验证"
	"github.com/ipfs/boxo/services/store/由国家/人民/注册"
	"github.com/ipfs/boxo/services/store/由国家/人民/验证"
	"github.com/ipfs/boxo/services/store/由国家/人民/验证/网络"
	"github.com/ipfs/boxo/services/store/由国家/人民/验证/端到端"
	"github.com/ipfs/boxo/services/store/由国家/人民/验证/服务器"
	"github.com/ipfs/boxo/services/store/由国家/人民/验证/客户端"
)

2. 定义了一些接口，用于与测试中使用的库进行交互：
python
import (
	"context"
	"net/http"
	"net/http/httptest"
	"os/exec"
	"sync"
	"testing"
	"time"

	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/routing/http/server"
	"github.com/ipfs/boxo/routing/http/types"
	"github.com/ipfs/boxo/routing/http/types/iter"
	"github.com/ipfs/boxo/routing/http/transport/目录"
	"github.com/ipfs/boxo/services/目录"
	"github.com


```
package cli

import (
	"context"
	"net/http"
	"net/http/httptest"
	"os/exec"
	"sync"
	"testing"
	"time"

	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/routing/http/server"
	"github.com/ipfs/boxo/routing/http/types"
	"github.com/ipfs/boxo/routing/http/types/iter"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
	"github.com/stretchr/testify/assert"
)

```

这是一个 Go 语言中的结构体 `fakeHTTPContentRouter`。它的主要作用是模拟 HTTP 内容路由器的行为，用于在本地模拟 HTTP 请求和响应的过程。

该结构体包含以下字段：

* `m`：一个 `sync.Mutex`，用于在 `FindProviders` 和 `FindPeers` 函数中对资源进行互斥锁。
* `provideBitswapCalls`：一个 `int`，用于记录已经调用了 `provideBitswap` 函数的次数。
* `findProvidersCalls`：一个 `int`，用于记录已经调用了 `FindProviders` 函数的次数。
* `findPeersCalls`：一个 `int`，用于记录已经调用了 `FindPeers` 函数的次数。

`FindProviders` 函数用于模拟 HTTP 内容路由器中的 `/providers` 路径。它接收一个 `ctx` 参数和一个 `key` 参数，并返回一个迭代器 `iter.ResultIter` 和一个错误。

在该函数中，首先会获取一个 `sync.Mutex`，然后获取路由器中已经调用 `provideBitswap` 和 `FindProviders` 函数的次数，最后返回一个已经解析过的 `iter.ResultIter`。

`FindPeers` 函数用于模拟 HTTP 内容路由器中的 `/peers` 路径。它接收一个 `ctx` 参数和一个 `key` 参数，并返回一个迭代器 `iter.ResultIter` 和一个错误。

在该函数中，首先会获取一个 `sync.Mutex`，然后获取路由器中已经调用 `findPeers` 函数的次数，最后返回一个已经解析过的 `iter.ResultIter`。


```
type fakeHTTPContentRouter struct {
	m                   sync.Mutex
	provideBitswapCalls int
	findProvidersCalls  int
	findPeersCalls      int
}

func (r *fakeHTTPContentRouter) FindProviders(ctx context.Context, key cid.Cid, limit int) (iter.ResultIter[types.Record], error) {
	r.m.Lock()
	defer r.m.Unlock()
	r.findProvidersCalls++
	return iter.FromSlice([]iter.Result[types.Record]{}), nil
}

// nolint deprecated
```

这是一个与HTTP内容路由器相关的中介组件，实现了一些核心功能。接下来我会逐步解释代码的作用。

1. `func (r *fakeHTTPContentRouter) ProvideBitswap(ctx context.Context, req *server.BitswapWriteProvideRequest) (time.Duration, error)`：

这个函数接收一个`BitswapWriteProvideRequest`类型的参数，并返回一个`time.Duration`类型的响应和一个`error`类型的返回。

在这个函数中，首先会获取一个`r`指针（从名为`fakeHTTPContentRouter`的上下文中获取），然后创建一个锁`r.m.Lock()`，以便在函数内部对`r`进行操作。

接着，会执行一个`defer`短路语句，确保在函数外部关闭锁。然后，定义一个名为`r.provideBitswapCalls`的变量，用于跟踪调用次数。这个变量 increments 每次`r.m.Lock()`被调用时。

最后，函数返回一个`time.Duration`类型，表示执行操作可能需要的时间，以及一个`error`类型，表示在函数执行过程中发生的错误。

1. `func (r *fakeHTTPContentRouter) FindPeers(ctx context.Context, pid peer.ID, limit int) (iter.ResultIter<*types.PeerRecord>, error)`：

这个函数接收一个`PeerID`类型的参数和一个`limit`类型的参数，并返回一个`iter.ResultIter`类型的响应和一个`error`类型的返回。

在这个函数中，首先会获取一个`r`指针（从名为`fakeHTTPContentRouter`的上下文中获取），然后创建一个锁`r.m.Lock()`，以便在函数内部对`r`进行操作。

接着，会执行一个`r.findPeersCalls++`，用于跟踪调用次数。然后，定义一个名为`iter.FromSlice`的函数，接收一个整数切片并返回一个`iter.ResultIter`类型的初始化结果。

最后，函数返回一个`iter.ResultIter`类型的响应，表示用于表示查询过程中返回的记录。同时，如果执行过程中发生错误，返回一个`error`类型的错误。


```
func (r *fakeHTTPContentRouter) ProvideBitswap(ctx context.Context, req *server.BitswapWriteProvideRequest) (time.Duration, error) {
	r.m.Lock()
	defer r.m.Unlock()
	r.provideBitswapCalls++
	return 0, nil
}

func (r *fakeHTTPContentRouter) FindPeers(ctx context.Context, pid peer.ID, limit int) (iter.ResultIter[*types.PeerRecord], error) {
	r.m.Lock()
	defer r.m.Unlock()
	r.findPeersCalls++
	return iter.FromSlice([]iter.Result[*types.PeerRecord]{}), nil
}

func (r *fakeHTTPContentRouter) GetIPNS(ctx context.Context, name ipns.Name) (*ipns.Record, error) {
	return nil, routing.ErrNotSupported
}

```

这段代码定义了两个函数，以及一个结构体类型的变量。

函数1：

func (r *fakeHTTPContentRouter) PutIPNS(ctx context.Context, name ipns.Name, rec *ipns.Record) error {
	return routing.ErrNotSupported
}

这个函数接收一个`ipns.Name`类型的参数`name`，以及一个`ipns.Record`类型的参数`rec`。它尝试调用`routing.ErrNotSupported`并返回一个错误。具体来说，这个函数是在`fakeHTTPContentRouter`结构体中定义的，它实现了`http.Handler`接口，用于路由HTTP请求。当调用此函数时，如果路由器不支持IPNS，则会返回一个错误。

函数2：

func (r *fakeHTTPContentRouter) numFindProvidersCalls() int {
	r.m.Lock()
	defer r.m.Unlock()
	return r.findProvidersCalls
}

这个函数在一个`fakeHTTPContentRouter`结构体中定义，它使用了`routing.mu`类型来确保对函数计数的锁的安全访问。具体来说，这个函数会在任何调用者开始锁住`r.m`之前记录调用者计数的次数。当调用者解锁`r.m`时，它将递归地遍历以确保所有记录都被正确地递归。函数返回调用者计数的次数。

最后，我们定义了一个`userAgentRecorder`类型，它是一个`http.Handler`类型的参数的`userAgent`字段的代理。


```
func (r *fakeHTTPContentRouter) PutIPNS(ctx context.Context, name ipns.Name, rec *ipns.Record) error {
	return routing.ErrNotSupported
}

func (r *fakeHTTPContentRouter) numFindProvidersCalls() int {
	r.m.Lock()
	defer r.m.Unlock()
	return r.findProvidersCalls
}

// userAgentRecorder records the user agent of every HTTP request
type userAgentRecorder struct {
	delegate   http.Handler
	userAgents []string
}

```

This is a Go test that checks the content router of the IPFS-HTTP-Router, which is a component of the IPFS (InterPlanetary File System) network. The IPFS network is designed to store and share large files across the network, and the content router is responsible for managing the routing of requests for file blocks.

The test starts by setting up a test node and a content router using the `harness.NewT` and `newNode` functions from the `harness` package. The content router is then configured to use the `userAgentRecorder` which is a recording the user agent recorder of the content router, which will be used to simulate the responses from a client.

The test then performs a series of tests to verify that the content router was called and responded correctly. These tests include:

* Verify that the content router was called and responded with the expected HTTP status code of 200
* Verify that the content router returned the correct user agent for each request
* Verify that the content router called the `contentRouter.numFindProvidersCalls()` function when a new file block was requested
* Verify that the content router returned an HTTP lookup for the uncached block that was requested

Finally, the test clears out the content router and cleans up the node.


```
func (r *userAgentRecorder) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	r.userAgents = append(r.userAgents, req.UserAgent())
	r.delegate.ServeHTTP(w, req)
}

func TestContentRoutingHTTP(t *testing.T) {
	cr := &fakeHTTPContentRouter{}

	// run the content routing HTTP server
	userAgentRecorder := &userAgentRecorder{delegate: server.Handler(cr)}
	server := httptest.NewServer(userAgentRecorder)
	t.Cleanup(func() { server.Close() })

	// setup the node
	node := harness.NewT(t).NewNode().Init()
	node.Runner.Env["IPFS_HTTP_ROUTERS"] = server.URL
	node.StartDaemon()

	// compute a random CID
	randStr := string(testutils.RandomBytes(100))
	res := node.PipeStrToIPFS(randStr, "add", "-qn")
	wantCIDStr := res.Stdout.Trimmed()

	t.Run("fetching an uncached block results in an HTTP lookup", func(t *testing.T) {
		statRes := node.Runner.Run(harness.RunRequest{
			Path:    node.IPFSBin,
			Args:    []string{"block", "stat", wantCIDStr},
			RunFunc: (*exec.Cmd).Start,
		})
		defer func() {
			if err := statRes.Cmd.Process.Kill(); err != nil {
				t.Logf("error killing 'block stat' cmd: %s", err)
			}
		}()

		// verify the content router was called
		assert.Eventually(t, func() bool {
			return cr.numFindProvidersCalls() > 0
		}, time.Minute, 10*time.Millisecond)

		assert.NotEmpty(t, userAgentRecorder.userAgents)
		version := node.IPFS("id", "-f", "<aver>").Stdout.Trimmed()
		for _, userAgent := range userAgentRecorder.userAgents {
			assert.Equal(t, version, userAgent)
		}
	})
}

```

# `/opt/kubo/test/cli/dag_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试一个名为 "cli" 的包。通过分析 code，我们可以看到它实现了以下主要功能：

1. 定义了一个名为 "fixtureFile" 的变量，用于存储测试用例的文件。
2. 定义了一个名为 "textOutputPath" 的变量，用于存储测试结果应该输出到的地方。
3. 定义了一个名为 "node1Cid" 和 "node2Cid" 的变量，它们分别表示两个测试节点的 ID。
4. 定义了一个名为 "fixtureCid" 的变量，用于标识包含测试用例的包。
5. 在一个名为 "testHarness" 的函数中，初始化了一个名为 "testCmd" 的上下文，该上下文用于在命令行中运行 "kubo run" 命令。
6. 在一个名为 "testify" 的函数中，读取一个名为 "node1Cid" 的节点的外部输出，并将其存储到一个名为 "output" 的字符串中。
7. 在一个名为 "asserts" 的函数中，使用 "assert" 包中的 "assert" 函数来检查 "output" 是否等于预期的输出。

这个测试框架的作用是测试 "cli" 包中的一些函数是否按照预期进行工作。


```
package cli

import (
	"encoding/json"
	"io"
	"os"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

const (
	fixtureFile    = "./fixtures/TestDagStat.car"
	textOutputPath = "./fixtures/TestDagStatExpectedOutput.txt"
	node1Cid       = "bafyreibmdfd7c5db4kls4ty57zljfhqv36gi43l6txl44pi423wwmeskwy"
	node2Cid       = "bafyreie3njilzdi4ixumru4nzgecsnjtu7fzfcwhg7e6s4s5i7cnbslvn4"
	fixtureCid     = "bafyreifrm6uf5o4dsaacuszf35zhibyojlqclabzrms7iak67pf62jygaq"
)

```

这段代码定义了一个名为 `DagStat` 的数据结构，用于表示一个 DAG 统计信息。每个 `DagStat` 对象包含三个属性：Cid、Size 和 NumBlocks，分别表示图中的节点 ID、节点大小和块的数量。

接着，定义了一个名为 `Data` 的数据结构，用于表示 DAG 统计信息。每个 `Data` 对象包含四个属性：UniqueBlocks、TotalSize、SharedSize 和 Ratio，分别表示图中的块的数量、总大小、每块的大小和占比。此外，还包含一个名为 `DagStats` 的数组，用于存储每个节点 `DagStat` 的信息。

最后，在 `Data` 结构体的 `DagStats` 字段中，包含一个名为 `DagStat` 的数组，用于存储每个节点的统计信息。每个 `DagStat` 对象包含三个属性：Cid、Size 和 NumBlocks，分别表示图中的节点 ID、节点大小和块的数量。


```
type DagStat struct {
	Cid       string `json:"Cid"`
	Size      int    `json:"Size"`
	NumBlocks int    `json:"NumBlocks"`
}

type Data struct {
	UniqueBlocks int       `json:"UniqueBlocks"`
	TotalSize    int       `json:"TotalSize"`
	SharedSize   int       `json:"SharedSize"`
	Ratio        float64   `json:"Ratio"`
	DagStats     []DagStat `json:"DagStats"`
}

// The Fixture file represents a dag where 2 nodes of size = 46B each, have a common child of 7B
```

This is a Go test that tests the IPFS DAG import functionality. The test reads two files, one containing DAG data and the other a fixture file, and uses the IPFS DAG import function to run a DAG import on the selected node.

The DAG data file contains information about the nodes and the connections between them. The fixture file contains some sample DAG data and a file to be used as input for the IPFS DAG import.

The expected output of the IPFS DAG import test is that the output of the "dag" and "stat" commands should contain the contents of the DAG data in the selected node. The "dag" command should also report the number of blocks for each node in the DAG.

The tests also check that the IPFS DAG import function uses the expected number of blocks for each node in the DAG and that the DAG data is loaded correctly.


```
// when traversing the DAG from the root's children (node1 and node2) we count (46 + 7)x2 bytes (counting redundant bytes) = 106
// since both nodes share a common child of 7 bytes we actually had to read (46)x2 + 7 =  99 bytes
// we should get a dedup ratio of 106/99 that results in approximatelly 1.0707071

func TestDag(t *testing.T) {
	t.Parallel()

	t.Run("ipfs dag stat --enc=json", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		// Import fixture
		r, err := os.Open(fixtureFile)
		assert.Nil(t, err)
		defer r.Close()
		err = node.IPFSDagImport(r, fixtureCid)
		assert.NoError(t, err)
		stat := node.RunIPFS("dag", "stat", "--progress=false", "--enc=json", node1Cid, node2Cid)
		var data Data
		err = json.Unmarshal(stat.Stdout.Bytes(), &data)
		assert.NoError(t, err)

		expectedUniqueBlocks := 3
		expectedSharedSize := 7
		expectedTotalSize := 99
		expectedRatio := float64(expectedSharedSize+expectedTotalSize) / float64(expectedTotalSize)
		expectedDagStatsLength := 2
		// Validate UniqueBlocks
		assert.Equal(t, expectedUniqueBlocks, data.UniqueBlocks)
		assert.Equal(t, expectedSharedSize, data.SharedSize)
		assert.Equal(t, expectedTotalSize, data.TotalSize)
		assert.Equal(t, testutils.FloatTruncate(expectedRatio, 4), testutils.FloatTruncate(data.Ratio, 4))

		// Validate DagStats
		assert.Equal(t, expectedDagStatsLength, len(data.DagStats))
		node1Output := data.DagStats[0]
		node2Output := data.DagStats[1]

		assert.Equal(t, node1Output.Cid, node1Cid)
		assert.Equal(t, node2Output.Cid, node2Cid)

		expectedNode1Size := (expectedTotalSize + expectedSharedSize) / 2
		expectedNode2Size := (expectedTotalSize + expectedSharedSize) / 2
		assert.Equal(t, expectedNode1Size, node1Output.Size)
		assert.Equal(t, expectedNode2Size, node2Output.Size)

		expectedNode1Blocks := 2
		expectedNode2Blocks := 2
		assert.Equal(t, expectedNode1Blocks, node1Output.NumBlocks)
		assert.Equal(t, expectedNode2Blocks, node2Output.NumBlocks)
	})

	t.Run("ipfs dag stat", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		r, err := os.Open(fixtureFile)
		assert.NoError(t, err)
		defer r.Close()
		f, err := os.Open(textOutputPath)
		assert.NoError(t, err)
		defer f.Close()
		content, err := io.ReadAll(f)
		assert.NoError(t, err)
		err = node.IPFSDagImport(r, fixtureCid)
		assert.NoError(t, err)
		stat := node.RunIPFS("dag", "stat", "--progress=false", node1Cid, node2Cid)
		assert.Equal(t, content, stat.Stdout.Bytes())
	})
}

```

# `/opt/kubo/test/cli/delegated_routing_v1_http_client_test.go`

This is a Go test that performs the following actions:

1. Creates a Router and a Peer on a test network using the `ToJSONStr` function from the `golang.org/x/text/source/认识了json/json间隙和用电量计算器`库.
2. Creates an HTTP router that listens for incoming connections on ports 4001 and 4002 using the `bitswap` schema defined by the `golang.org/x/text/source/认识了json/json间隙和用电量计算器`库.
3. Emits OpenCeurometer metrics for the HTTP router using the `node.StartDaemon` function from the `golang.org/x/text/source/认识了json/json间隙和用电量计算器`库 and the `res.Stdout.Trimmed()` function from the `golang.org/x/text/source/认识了json/json间隙和用电量计算器`库.
4. Tests that the HTTP client is able to retrieve the router's metrics using the `node.APIClient().Get` function from the `golang.org/x/text/source/认识了json/json间隙和用电量计算器`库.

These actions are intended to verify that the HTTP router is able to correctly route traffic to the specified IP addresses and that the HTTP client is able to retrieve the router's metrics.


```
package cli

import (
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

func TestHTTPDelegatedRouting(t *testing.T) {
	t.Parallel()
	node := harness.NewT(t).NewNode().Init().StartDaemon()

	fakeServer := func(contentType string, resp ...string) *httptest.Server {
		return httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			w.Header().Set("Content-Type", contentType)
			for _, r := range resp {
				_, err := w.Write([]byte(r))
				if err != nil {
					panic(err)
				}
			}
		}))
	}

	findProvsCID := "baeabep4vu3ceru7nerjjbk37sxb7wmftteve4hcosmyolsbsiubw2vr6pqzj6mw7kv6tbn6nqkkldnklbjgm5tzbi4hkpkled4xlcr7xz4bq"
	provs := []string{"12D3KooWAobjw92XDcnQ1rRmRJDA3zAQpdPYUpZKrJxH6yccSpje", "12D3KooWARYacCc6eoCqvsS9RW9MA2vo51CV75deoiqssx3YgyYJ"}

	t.Run("default routing config has no routers defined", func(t *testing.T) {
		assert.Nil(t, node.ReadConfig().Routing.Routers)
	})

	t.Run("no routers means findprovs returns no results", func(t *testing.T) {
		res := node.IPFS("routing", "findprovs", findProvsCID).Stdout.String()
		assert.Empty(t, res)
	})

	t.Run("no routers means findprovs returns no results", func(t *testing.T) {
		res := node.IPFS("routing", "findprovs", findProvsCID).Stdout.String()
		assert.Empty(t, res)
	})

	node.StopDaemon()

	t.Run("missing method params make the daemon fail", func(t *testing.T) {
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Routing.Type = config.NewOptionalString("custom")
			cfg.Routing.Methods = config.Methods{
				"find-peers":     {RouterName: "TestDelegatedRouter"},
				"find-providers": {RouterName: "TestDelegatedRouter"},
				"get-ipns":       {RouterName: "TestDelegatedRouter"},
				"provide":        {RouterName: "TestDelegatedRouter"},
			}
		})
		res := node.RunIPFS("daemon")
		assert.Equal(t, 1, res.ExitErr.ProcessState.ExitCode())
		assert.Contains(
			t,
			res.Stderr.String(),
			`method name "put-ipns" is missing from Routing.Methods config param`,
		)
	})

	t.Run("having wrong methods makes daemon fail", func(t *testing.T) {
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Routing.Type = config.NewOptionalString("custom")
			cfg.Routing.Methods = config.Methods{
				"find-peers":     {RouterName: "TestDelegatedRouter"},
				"find-providers": {RouterName: "TestDelegatedRouter"},
				"get-ipns":       {RouterName: "TestDelegatedRouter"},
				"provide":        {RouterName: "TestDelegatedRouter"},
				"put-ipns":       {RouterName: "TestDelegatedRouter"},
				"NOT_SUPPORTED":  {RouterName: "TestDelegatedRouter"},
			}
		})
		res := node.RunIPFS("daemon")
		assert.Equal(t, 1, res.ExitErr.ProcessState.ExitCode())
		assert.Contains(
			t,
			res.Stderr.String(),
			`method name "NOT_SUPPORTED" is not a supported method on Routing.Methods config param`,
		)
	})

	t.Run("adding HTTP delegated routing endpoint to Routing.Routers config works", func(t *testing.T) {
		server := fakeServer("application/json", ToJSONStr(JSONObj{
			"Providers": []JSONObj{
				{
					"Schema":   "bitswap", // Legacy bitswap schema.
					"Protocol": "transport-bitswap",
					"ID":       provs[1],
					"Addrs":    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
				},
				{
					"Schema":    "peer",
					"Protocols": []string{"transport-bitswap"},
					"ID":        provs[0],
					"Addrs":     []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
				},
			},
		}))
		t.Cleanup(server.Close)

		node.IPFS("config", "Routing.Type", "custom")
		node.IPFS("config", "Routing.Routers.TestDelegatedRouter", "--json", ToJSONStr(JSONObj{
			"Type": "http",
			"Parameters": JSONObj{
				"Endpoint": server.URL,
			},
		}))
		node.IPFS("config", "Routing.Methods", "--json", ToJSONStr(JSONObj{
			"find-peers":     JSONObj{"RouterName": "TestDelegatedRouter"},
			"find-providers": JSONObj{"RouterName": "TestDelegatedRouter"},
			"get-ipns":       JSONObj{"RouterName": "TestDelegatedRouter"},
			"provide":        JSONObj{"RouterName": "TestDelegatedRouter"},
			"put-ipns":       JSONObj{"RouterName": "TestDelegatedRouter"},
		}))

		res := node.IPFS("config", "Routing.Routers.TestDelegatedRouter.Parameters.Endpoint")
		assert.Equal(t, res.Stdout.Trimmed(), server.URL)

		node.StartDaemon()
		res = node.IPFS("routing", "findprovs", findProvsCID)
		assert.Equal(t, provs[1]+"\n"+provs[0], res.Stdout.Trimmed())
	})

	node.StopDaemon()

	t.Run("adding HTTP delegated routing endpoint to Routing.Routers config works (streaming)", func(t *testing.T) {
		server := fakeServer("application/x-ndjson", ToJSONStr(JSONObj{
			"Schema":    "peer",
			"Protocols": []string{"transport-bitswap"},
			"ID":        provs[0],
			"Addrs":     []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
		}), ToJSONStr(JSONObj{
			"Schema":   "bitswap", // Legacy bitswap schema.
			"Protocol": "transport-bitswap",
			"ID":       provs[1],
			"Addrs":    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/tcp/4002"},
		}))
		t.Cleanup(server.Close)

		node.IPFS("config", "Routing.Routers.TestDelegatedRouter", "--json", ToJSONStr(JSONObj{
			"Type": "http",
			"Parameters": JSONObj{
				"Endpoint": server.URL,
			},
		}))

		res := node.IPFS("config", "Routing.Routers.TestDelegatedRouter.Parameters.Endpoint")
		assert.Equal(t, res.Stdout.Trimmed(), server.URL)

		node.StartDaemon()
		res = node.IPFS("routing", "findprovs", findProvsCID)
		assert.Equal(t, provs[0]+"\n"+provs[1], res.Stdout.Trimmed())
	})

	t.Run("HTTP client should emit OpenCensus metrics", func(t *testing.T) {
		resp := node.APIClient().Get("/debug/metrics/prometheus")
		assert.Contains(t, resp.Body, "routing_http_client_length_count")
	})
}

```

# `/opt/kubo/test/cli/delegated_routing_v1_http_proxy_test.go`

This is a Go test that checks if the resolved IPNS name is equal to the expected name in case of a published record. The IPFS NameFromPeer function is used to get the IPNS name from the given peer. The test sets up a cluster with 2 nodes, both of which have IPFS and publish a random string as an IPNS name. The test then retrieves the published name using the Routing V1 endpoint and another IPFS endpoint, and checks if the retrieved name is equal to the expected name.

The `setupNodes` function is a helper function that sets up the nodes for the test. In this case, it returns a slice of 2 nodes, which will be used for the test. The function is responsible for creating and setting up the nodes, which includes configuring them with the appropriate IPFS endpoints and other settings.

The `Node1Publish` function is a helper function that publishes a random string as an IPNS name on Node 1. It is responsible for setting up the necessary publish settings on Node 1 and returning the IPFS name of the published string.

The `Node0Resolve` function is a helper function that retrieves the IPNS name for a published record from Node 0. It uses the Routing V1 endpoint to retrieve the name, which is then compared to the expected name to see if they are equal.

The `testutils` package provides some utility functions for testing IPFS. The `RandomStr` function generates a random string of 1000 characters.


```
package cli

import (
	"testing"

	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestRoutingV1Proxy(t *testing.T) {
	t.Parallel()

	setupNodes := func(t *testing.T) harness.Nodes {
		nodes := harness.NewT(t).NewNodes(2).Init()

		// Node 0 uses DHT and exposes the Routing API.
		nodes[0].UpdateConfig(func(cfg *config.Config) {
			cfg.Gateway.ExposeRoutingAPI = config.True
			cfg.Discovery.MDNS.Enabled = false
			cfg.Routing.Type = config.NewOptionalString("dht")
		})
		nodes[0].StartDaemon()

		// Node 1 uses Node 0 as Routing V1 source, no DHT.
		nodes[1].UpdateConfig(func(cfg *config.Config) {
			cfg.Discovery.MDNS.Enabled = false
			cfg.Routing.Type = config.NewOptionalString("custom")
			cfg.Routing.Methods = config.Methods{
				config.MethodNameFindPeers:     {RouterName: "KuboA"},
				config.MethodNameFindProviders: {RouterName: "KuboA"},
				config.MethodNameGetIPNS:       {RouterName: "KuboA"},
				config.MethodNamePutIPNS:       {RouterName: "KuboA"},
				config.MethodNameProvide:       {RouterName: "KuboA"},
			}
			cfg.Routing.Routers = config.Routers{
				"KuboA": config.RouterParser{
					Router: config.Router{
						Type: config.RouterTypeHTTP,
						Parameters: &config.HTTPRouterParams{
							Endpoint: nodes[0].GatewayURL(),
						},
					},
				},
			}
		})
		nodes[1].StartDaemon()

		// Connect them.
		nodes.Connect()

		return nodes
	}

	t.Run("Kubo can find provider for CID via Routing V1", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		cidStr := nodes[0].IPFSAddStr(testutils.RandomStr(1000))

		res := nodes[1].IPFS("routing", "findprovs", cidStr)
		assert.Equal(t, nodes[0].PeerID().String(), res.Stdout.Trimmed())
	})

	t.Run("Kubo can find peer via Routing V1", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		// Start lonely node that is not connected to other nodes.
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Discovery.MDNS.Enabled = false
			cfg.Routing.Type = config.NewOptionalString("dht")
		})
		node.StartDaemon()

		// Connect Node 0 to Lonely Node.
		nodes[0].Connect(node)

		// Node 1 must find Lonely Node through Node 0 Routing V1.
		res := nodes[1].IPFS("routing", "findpeer", node.PeerID().String())
		assert.Equal(t, node.SwarmAddrs()[0].String(), res.Stdout.Trimmed())
	})

	t.Run("Kubo can retrieve IPNS record via Routing V1", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		nodeName := "/ipns/" + ipns.NameFromPeer(nodes[0].PeerID()).String()

		// Can't resolve the name as isn't published yet.
		res := nodes[1].RunIPFS("routing", "get", nodeName)
		require.Error(t, res.ExitErr)

		// Publish record on Node 0.
		path := "/ipfs/" + nodes[0].IPFSAddStr(testutils.RandomStr(1000))
		nodes[0].IPFS("name", "publish", "--allow-offline", path)

		// Get record on Node 1 (no DHT).
		res = nodes[1].IPFS("routing", "get", nodeName)
		record, err := ipns.UnmarshalRecord(res.Stdout.Bytes())
		require.NoError(t, err)
		value, err := record.Value()
		require.NoError(t, err)
		require.Equal(t, path, value.String())
	})

	t.Run("Kubo can resolve IPNS name via Routing V1", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		nodeName := "/ipns/" + ipns.NameFromPeer(nodes[0].PeerID()).String()

		// Can't resolve the name as isn't published yet.
		res := nodes[1].RunIPFS("routing", "get", nodeName)
		require.Error(t, res.ExitErr)

		// Publish name.
		path := "/ipfs/" + nodes[0].IPFSAddStr(testutils.RandomStr(1000))
		nodes[0].IPFS("name", "publish", "--allow-offline", path)

		// Resolve IPNS name
		res = nodes[1].IPFS("name", "resolve", nodeName)
		require.Equal(t, path, res.Stdout.Trimmed())
	})

	t.Run("Kubo can provide IPNS record via Routing V1", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		// Publish something on Node 1 (no DHT).
		nodeName := "/ipns/" + ipns.NameFromPeer(nodes[1].PeerID()).String()
		path := "/ipfs/" + nodes[1].IPFSAddStr(testutils.RandomStr(1000))
		nodes[1].IPFS("name", "publish", "--allow-offline", path)

		// Retrieve through Node 0.
		res := nodes[0].IPFS("routing", "get", nodeName)
		record, err := ipns.UnmarshalRecord(res.Stdout.Bytes())
		require.NoError(t, err)
		value, err := record.Value()
		require.NoError(t, err)
		require.Equal(t, path, value.String())
	})
}

```

# `/opt/kubo/test/cli/delegated_routing_v1_http_server_test.go`

该代码是一个名为 "cli" 的包，其中定义了一些用于测试薄片存储桶(也就是IPFS)的客户端(也就是Boxo)的函数。具体来说，该代码包括以下内容：

1. 导入了一些外部的库和常量：

  - "github.com/google/uuid"
  - "github.com/ipfs/boxo/ipns"
  - "github.com/ipfs/boxo/routing/http/client"
  - "github.com/ipfs/boxo/routing/http/types"
  - "github.com/ipfs/boxo/routing/http/types/iter"
  - "github.com/ipfs/boxo/structure/box"
  - "github.com/ipfs/boxo/test/cli/desc"

2. 定义了一个名为 "Context" 的结构体，其中包含了一些与上下文相关的字段，例如上下文 UUID、上下文类型(测试或生产环境)、上下文是否包含测试用的标识符(该标识符用于在测试和生产环境中区分自身环境)，以及当前上下文中的上下文对象(ContextObject)等。

3. 定义了一个名为 "test suite" 的函数，该函数是一个测试套件，用于在上下文中执行一些测试操作。具体来说，该函数会使用 "desc" 包中的 "describe" 函数来描述测试套件中要测试的用例，并根据这些用例生成测试代码。

4. 定义了一个名为 "fixture" 的函数，该函数是一个 fixture，用于在测试中提供一些上下文属性和方法。具体来说，该函数包含一个 "fixture" 类型的字段，该字段包含用于设置或获取上下文的属性和方法，例如上下文的 UUID、上下文的类型、上下文是否包含测试标识符等。

5. 定义了一个名为 "Client" 的函数，该函数是一个 Client，用于通过IPFS协议连通IPFS的节点。具体来说，该函数使用 "client" 包中的 "Client" 函数来创建一个IPFS的 "Client" 对象，并使用该对象来连通IPFS节点。

6. 定义了一个名为 "Node" 的函数，该函数是一个 Node，用于执行IPFS节点的 "describe" 函数。具体来说，该函数使用 "client" 包中的 "describe" 函数来获取IPFS节点的一些描述性信息，例如该节点的UUID、所处的位置、以及该节点上的度量单位等。

7. 定义了一个名为 "MerkleNode" 的函数，该函数是一个 MerkleNode，用于执行Merkle散列验证。具体来说，该函数使用 "box" 包中的 "MerkleNode" 函数来创建一个Merkle散列验证的 "MerkleNode" 对象，并使用该对象来执行Merkle散列验证。

8. 定义了一个名为 "Cid" 的函数，该函数是一个 CID，用于在IPFS中执行CID操作。具体来说，该函数使用 "box" 包中的 "Cid" 函数来创建一个IPFS的 "Cid" 对象，并使用该对象来执行CID操作。

9. 定义了一个名为 "ClientCredentials" 的函数，该函数是一个 ClientCredentials，用于在IPFS中执行用户验证。具体来说，该函数使用 "box" 包中的 "ClientCredentials" 函数来创建一个IPFS的 "ClientCredentials" 对象，并使用该对象来执行用户验证。

10. 定义了一个名为 "BoxoStore" 的函数，该函数是一个 BoxoStore，用于管理IPFS的 Boxo 对象。具体来说，该函数使用 "box" 包中的 "BoxoStore" 函数来创建一个IPFS的 "BoxoStore" 对象，并使用该对象来管理Boxo对象。

11. 定义了一个名为 "Ledger" 的函数，该函数是一个 EdgeCredential，用于在IPFS中执行验证。具体来说，该函数使用 "box" 包中的 "Ledger" 函数来创建一个IPFS的 "Ledger" 对象，并使用该对象来执行验证。

12. 定义了一个名为 "Peer" 的函数，该函数是一个 Peer，用于在IPFS中执行与其它Peer通信的实验。具体来说，该函数使用 "peer" 包中的 "Peer" 函数来创建一个IPFS的 "Peer" 对象，并使用该对象来执行与其它Peer通信的实验。

13. 定义了一个名为 "Router" 的函数，该函数是一个 Router，用于在IPFS中执行一些路由相关的操作。具体来说，该函数使用 "boxo-router" 库中的 "Router" 函数来创建一个IPFS的 "Router" 对象，并使用该对象来执行路由相关的操作。

14. 定义了一个名为 "庭血缘" 的函数，该函数是一个 place，用于在 IPFS 中执行全球查询操作。具体来说，该函数使用 "desc" 包中的 "place" 函数来创建一个 IPFS的 "place" 对象，并使用该对象来执行全球查询操作。

15. 定义了一个名为 "客户端" 的函数，该函数是一个 Client，用于通过IPFS协议连通IPFS的节点。具体来说，该函数使用 "box/client" 客户端库中的 "Client" 函数来创建一个IPFS的 "Client" 对象，并使用该对象来连通IPFS节点。


```
package cli

import (
	"context"
	"testing"

	"github.com/google/uuid"
	"github.com/ipfs/boxo/ipns"
	"github.com/ipfs/boxo/routing/http/client"
	"github.com/ipfs/boxo/routing/http/types"
	"github.com/ipfs/boxo/routing/http/types/iter"
	"github.com/ipfs/go-cid"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/stretchr/testify/assert"
)

```

This is a test case for IPNS (Inter-Peer System), which is a decentralized, peer-to-peer data store that enables efficient and secure sharing of data across a network of autonomous computers.

The test case starts with a lone IPNS node that is not connected to other nodes. It then puts an IPNS record (hello world text) in that node and then gets the record back from the node. Finally, it checks that the record is accepted as it is a valid record and also checks that the lonely node is able to put the record back.


```
func TestRoutingV1Server(t *testing.T) {
	t.Parallel()

	setupNodes := func(t *testing.T) harness.Nodes {
		nodes := harness.NewT(t).NewNodes(5).Init()
		nodes.ForEachPar(func(node *harness.Node) {
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Gateway.ExposeRoutingAPI = config.True
				cfg.Routing.Type = config.NewOptionalString("dht")
			})
		})
		nodes.StartDaemons().Connect()
		return nodes
	}

	t.Run("Get Providers Responds With Correct Peers", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		text := "hello world " + uuid.New().String()
		cidStr := nodes[2].IPFSAddStr(text)
		_ = nodes[3].IPFSAddStr(text)

		cid, err := cid.Decode(cidStr)
		assert.NoError(t, err)

		c, err := client.New(nodes[1].GatewayURL())
		assert.NoError(t, err)

		resultsIter, err := c.FindProviders(context.Background(), cid)
		assert.NoError(t, err)

		records, err := iter.ReadAllResults(resultsIter)
		assert.NoError(t, err)

		var peers []peer.ID
		for _, record := range records {
			assert.Equal(t, types.SchemaPeer, record.GetSchema())

			peer, ok := record.(*types.PeerRecord)
			assert.True(t, ok)
			peers = append(peers, *peer.ID)
		}

		assert.Contains(t, peers, nodes[2].PeerID())
		assert.Contains(t, peers, nodes[3].PeerID())
	})

	t.Run("Get Peers Responds With Correct Peers", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		c, err := client.New(nodes[1].GatewayURL())
		assert.NoError(t, err)

		resultsIter, err := c.FindPeers(context.Background(), nodes[2].PeerID())
		assert.NoError(t, err)

		records, err := iter.ReadAllResults(resultsIter)
		assert.NoError(t, err)
		assert.Len(t, records, 1)
		assert.IsType(t, records[0].GetSchema(), records[0].GetSchema())
		assert.IsType(t, records[0], &types.PeerRecord{})

		peer := records[0]
		assert.Equal(t, nodes[2].PeerID().String(), peer.ID.String())
		assert.NotEmpty(t, peer.Addrs)
	})

	t.Run("Get IPNS Record Responds With Correct Record", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		text := "hello ipns test " + uuid.New().String()
		cidStr := nodes[0].IPFSAddStr(text)
		nodes[0].IPFS("name", "publish", "--allow-offline", cidStr)

		// Ask for record from a different peer.
		c, err := client.New(nodes[1].GatewayURL())
		assert.NoError(t, err)

		record, err := c.GetIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()))
		assert.NoError(t, err)

		value, err := record.Value()
		assert.NoError(t, err)
		assert.Equal(t, "/ipfs/"+cidStr, value.String())
	})

	t.Run("Put IPNS Record Succeeds", func(t *testing.T) {
		t.Parallel()
		nodes := setupNodes(t)

		// Publish a record and confirm the /routing/v1/ipns API exposes the IPNS record
		text := "hello ipns test " + uuid.New().String()
		cidStr := nodes[0].IPFSAddStr(text)
		nodes[0].IPFS("name", "publish", "--allow-offline", cidStr)
		c, err := client.New(nodes[0].GatewayURL())
		assert.NoError(t, err)
		record, err := c.GetIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()))
		assert.NoError(t, err)
		value, err := record.Value()
		assert.NoError(t, err)
		assert.Equal(t, "/ipfs/"+cidStr, value.String())

		// Start lonely node that is not connected to other nodes.
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Gateway.ExposeRoutingAPI = config.True
			cfg.Routing.Type = config.NewOptionalString("dht")
		})
		node.StartDaemon()

		// Put IPNS record in lonely node. It should be accepted as it is a valid record.
		c, err = client.New(node.GatewayURL())
		assert.NoError(t, err)
		err = c.PutIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()), record)
		assert.NoError(t, err)

		// Get the record from lonely node and double check.
		record, err = c.GetIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()))
		assert.NoError(t, err)
		value, err = record.Value()
		assert.NoError(t, err)
		assert.Equal(t, "/ipfs/"+cidStr, value.String())
	})
}

```

# `/opt/kubo/test/cli/dht_autoclient_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Kubernetes (k8s) 的自动客户端 (DHT Autoclient) 功能。Harness 是 Kubernetes 自客户机工具测试框架，提供了一个用于测试 Kubernetes 自客户机工具 Harness.。

这段代码的主要部分如下：

1. 导入需要使用的标准：

package cli

import (
	"bytes"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

1. 定义一个名为 TestDHTAutoclient 的测试函数：

func TestDHTAutoclient(t *testing.T) {
	t.Parallel()
	nodes := harness.NewT(t).NewNodes(10).Init()
	harness.Nodes(nodes[8:]).ForEachPar(func(node *harness.Node) {
		node.IPFS("config", "Routing.Type", "autoclient")
	})
	nodes.StartDaemons().Connect()

	这段代码创建了一个名为 harness 的实例，并使用 `NewT` 方法创建了一个包含 10 个节点的 ` harness.Node` 实例。通过调用 `harness.Nodes(nodes[8:])`，将自定义节点设置为异步客户端模式。然后调用 `nodes.StartDaemons().Connect()` 来启动自定义 DNS 服务器。

	1. 编写两个测试函数：

	t.Run("file added on node in client mode is retrievable from node in client mode", func(t *testing.T) {
		t.Parallel()
		randomBytes := testutils.RandomBytes(1000)
		randomBytes = append(randomBytes, '\r')
		hash := nodes[8].IPFSAdd(bytes.NewReader(randomBytes))

		res := nodes[9].IPFS("cat", hash)
		assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
	})
	t.Run("file added on node in server mode is retrievable from all nodes", func(t *testing.T) {
		t.Parallel()
		randomBytes := testutils.RandomBytes(1000)
		hash := nodes[0].IPFSAdd(bytes.NewReader(randomBytes))

		for i := 0; i < 10; i++ {
			res := nodes[i].IPFS("cat", hash)
			assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
		}
	})

	1. 编写一个名为 harness 的自定义 DNS 服务器：

	// Add an entry for a file on a node in client mode
	func TestDHTAutoclient(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(10).Init()
		harness.Nodes(nodes[8:]).ForEachPar(func(node *harness.Node) {
			node.IPFS("config", "Routing.Type", "autoclient")
		})
		nodes.StartDaemons().Connect()

		t.Run("file added on node in client mode is retrieved



```
package cli

import (
	"bytes"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

func TestDHTAutoclient(t *testing.T) {
	t.Parallel()
	nodes := harness.NewT(t).NewNodes(10).Init()
	harness.Nodes(nodes[8:]).ForEachPar(func(node *harness.Node) {
		node.IPFS("config", "Routing.Type", "autoclient")
	})
	nodes.StartDaemons().Connect()

	t.Run("file added on node in client mode is retrievable from node in client mode", func(t *testing.T) {
		t.Parallel()
		randomBytes := testutils.RandomBytes(1000)
		randomBytes = append(randomBytes, '\r')
		hash := nodes[8].IPFSAdd(bytes.NewReader(randomBytes))

		res := nodes[9].IPFS("cat", hash)
		assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
	})

	t.Run("file added on node in server mode is retrievable from all nodes", func(t *testing.T) {
		t.Parallel()
		randomBytes := testutils.RandomBytes(1000)
		hash := nodes[0].IPFSAdd(bytes.NewReader(randomBytes))

		for i := 0; i < 10; i++ {
			res := nodes[i].IPFS("cat", hash)
			assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
		}
	})
}

```

# `/opt/kubo/test/cli/dht_legacy_test.go`

This is a Go test that tests the distributed hash table (DHT) of the IPFS (Interoperable Push-Pull Service) peer-to-peer network. It checks if the various DHT commands work as expected when running on a node that is connected to a peer network and has the peers' hashes stored in a hash table.

The test covers the following DHT commands:

1. `dht findprovs`: This command is used to query the node that is considered the closest (based on the count of the node) to find the peers of a given node. It should return the node ID.
2. `dht findpeer`: This command is used to query the node that is considered the closest (based on the count of the node) to find the peers of a given node. It should return the node ID.
3. `dht put`: This command is used to add a file to the node's local DHT. It should update the file's checksum and the file's metadata (such as the file's content type and the file's size) in the node's local DHT.

The tests use the `testutils.CIDEmptyDir` directory to create a fake data directory that is not present in the node's local storage. The ` harness.NewT` function is used to create a testHarness instance, and the ` harness.NewNode` function is used to create a new node instance. The `init` method is used to initialize the node's state, and the `runIPFS` method is used to run the `dht` commands on the node.

The tests first check that the `dht findprovs` command returns the expected node ID. Then, they check that the `dht findpeer` command returns the expected node ID. Finally, they check that the `dht put` command updates the file's checksum and metadata in the node's local DHT.

If any of the `dht` commands fail with an error message, the tests will not add the file to the node's local DHT and will not attempt to update the file's metadata.


```
package cli

import (
	"sort"
	"sync"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestLegacyDHT(t *testing.T) {
	t.Parallel()
	nodes := harness.NewT(t).NewNodes(5).Init()
	nodes.ForEachPar(func(node *harness.Node) {
		node.IPFS("config", "Routing.Type", "dht")
	})
	nodes.StartDaemons().Connect()

	t.Run("ipfs dht findpeer", func(t *testing.T) {
		t.Parallel()
		res := nodes[1].RunIPFS("dht", "findpeer", nodes[0].PeerID().String())
		assert.Equal(t, 0, res.ExitCode())

		swarmAddr := nodes[0].SwarmAddrsWithoutPeerIDs()[0]
		require.Equal(t, swarmAddr.String(), res.Stdout.Trimmed())
	})

	t.Run("ipfs dht get <key>", func(t *testing.T) {
		t.Parallel()
		hash := nodes[2].IPFSAddStr("hello world")
		nodes[2].IPFS("name", "publish", "/ipfs/"+hash)

		res := nodes[1].IPFS("dht", "get", "/ipns/"+nodes[2].PeerID().String())
		assert.Contains(t, res.Stdout.String(), "/ipfs/"+hash)

		t.Run("put round trips (#3124)", func(t *testing.T) {
			t.Parallel()
			nodes[0].WriteBytes("get_result", res.Stdout.Bytes())
			res := nodes[0].IPFS("dht", "put", "/ipns/"+nodes[2].PeerID().String(), "get_result")
			assert.Greater(t, len(res.Stdout.Lines()), 0, "should put to at least one node")
		})

		t.Run("put with bad keys fails (issue #5113, #4611)", func(t *testing.T) {
			t.Parallel()
			keys := []string{"foo", "/pk/foo", "/ipns/foo"}
			for _, key := range keys {
				key := key
				t.Run(key, func(t *testing.T) {
					t.Parallel()
					res := nodes[0].RunIPFS("dht", "put", key)
					assert.Equal(t, 1, res.ExitCode())
					assert.Contains(t, res.Stderr.String(), "invalid")
					assert.Empty(t, res.Stdout.String())
				})
			}
		})

		t.Run("get with bad keys (issue #4611)", func(t *testing.T) {
			for _, key := range []string{"foo", "/pk/foo"} {
				key := key
				t.Run(key, func(t *testing.T) {
					t.Parallel()
					res := nodes[0].RunIPFS("dht", "get", key)
					assert.Equal(t, 1, res.ExitCode())
					assert.Contains(t, res.Stderr.String(), "invalid")
					assert.Empty(t, res.Stdout.String())
				})
			}
		})
	})

	t.Run("ipfs dht findprovs", func(t *testing.T) {
		t.Parallel()
		hash := nodes[3].IPFSAddStr("some stuff")
		res := nodes[4].IPFS("dht", "findprovs", hash)
		assert.Equal(t, nodes[3].PeerID().String(), res.Stdout.Trimmed())
	})

	t.Run("ipfs dht query <peerID>", func(t *testing.T) {
		t.Parallel()
		t.Run("normal DHT configuration", func(t *testing.T) {
			t.Parallel()
			hash := nodes[0].IPFSAddStr("some other stuff")
			peerCounts := map[string]int{}
			peerCountsMut := sync.Mutex{}
			harness.Nodes(nodes).ForEachPar(func(node *harness.Node) {
				res := node.IPFS("dht", "query", hash)
				closestPeer := res.Stdout.Lines()[0]
				// check that it's a valid peer ID
				_, err := peer.Decode(closestPeer)
				require.NoError(t, err)

				peerCountsMut.Lock()
				peerCounts[closestPeer]++
				peerCountsMut.Unlock()
			})
			// 4 nodes should see the same peer ID
			// 1 node (the closest) should see a different one
			var counts []int
			for _, count := range peerCounts {
				counts = append(counts, count)
			}
			sort.IntSlice(counts).Sort()
			assert.Equal(t, []int{1, 4}, counts)
		})
	})

	t.Run("dht commands fail when offline", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()

		// these cannot be run in parallel due to repo locking (seems like a bug)

		t.Run("dht findprovs", func(t *testing.T) {
			res := node.RunIPFS("dht", "findprovs", testutils.CIDEmptyDir)
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
		})

		t.Run("dht findpeer", func(t *testing.T) {
			res := node.RunIPFS("dht", "findpeer", testutils.CIDEmptyDir)
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
		})

		t.Run("dht put", func(t *testing.T) {
			node.WriteBytes("foo", []byte("foo"))
			res := node.RunIPFS("dht", "put", "/ipns/"+node.PeerID().String(), "foo")
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "this action must be run in online mode")
		})
	})
}

```

# `/opt/kubo/test/cli/dht_opt_prov_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 DHT 优化提供功能。它定义了一个名为 `TestDHTOptimisticProvide` 的函数，该函数对两个测试函数 `optimistic provide smoke test` 和 `optimistic provide with self-data` 起作用。

具体来说，这段代码：

1. 定义了一个名为 `TestDHTOptimisticProvide` 的函数；
2. 在该函数内，定义了一个名为 `t` 的 testing 包别名 `testing`；
3. 在 `t` 包别名内部，定义了一个名为 `TestDHTOptimisticProvide` 的函数；
4. 在 `TestDHTOptimisticProvide` 函数内，创建了一些测试函数使用的节点；
5. 对于每个测试函数，执行以下操作：
	1. 更新控制台的 `Experimental.OptimisticProvide` 设置为 `true`；
	2. 启动两个节点；
	3. 通过 `nodes[0].IPFS` 方法向第一个节点发送 "dht" 根节点请求，并指定一个随机字符串作为根哈希；
	4. 通过 `nodes[0].IPFS` 方法向第二个节点发送命令，要求第二个节点返回以第一个节点为根的哈希表中，包含 `--num-providers=1` 参数的哈希列表的第一个哈希值；
	5. 检查第二个节点的输出是否与预期结果相同。

这段代码的作用是测试 DHT 优化提供功能。通过在两个测试函数中运行相同的操作，并检查它们的输出结果是否一致，可以确保哈希表在启动多个节点后能够正确地工作，并提供一个最优的性能。


```
package cli

import (
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
)

func TestDHTOptimisticProvide(t *testing.T) {
	t.Parallel()

	t.Run("optimistic provide smoke test", func(t *testing.T) {
		nodes := harness.NewT(t).NewNodes(2).Init()

		nodes[0].UpdateConfig(func(cfg *config.Config) {
			cfg.Experimental.OptimisticProvide = true
		})

		nodes.StartDaemons().Connect()

		hash := nodes[0].IPFSAddStr(testutils.RandomStr(100))
		nodes[0].IPFS("dht", "provide", hash)

		res := nodes[1].IPFS("routing", "findprovs", "--num-providers=1", hash)
		assert.Equal(t, nodes[0].PeerID().String(), res.Stdout.Trimmed())
	})
}

```

# `/opt/kubo/test/cli/gateway_range_test.go`

This is a Go test that uses theHarness library to interact with the Kubernetes node that is running the Kubernetes CLI harness. TheHarness library is a simple wrapper around the `net/http` and `os` packages that makes it easier to perform common Kubernetes tasks such as fetching a list of HAMT blocks.

The `TestGatewayHAMTDirectory` function tests the gateway by opening the HAMT directory that has been sharded and熙熙攘攘 it with 10k items. It then imports the fixtures from the HAMT directory and uses the gateway to fetch the minimal set of blocks that are necessary for directory listing. It asserts that the response from the gateway is an HTTP status code of 200 (OK) and that the fetch is successful.


```
package cli

import (
	"fmt"
	"net/http"
	"os"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/stretchr/testify/assert"
)

func TestGatewayHAMTDirectory(t *testing.T) {
	t.Parallel()

	const (
		// The CID of the HAMT-sharded directory that has 10k items
		hamtCid = "bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm"

		// fixtureCid is the CID of root of the DAG that is a subset of hamtCid DAG
		// representing the minimal set of blocks necessary for directory listing.
		// It also includes a "files_refs" file with the list of the references
		// we do NOT needs to fetch (files inside the directory)
		fixtureCid = "bafybeig3yoibxe56aolixqa4zk55gp5sug3qgaztkakpndzk2b2ynobd4i"
	)

	// Start node
	h := harness.NewT(t)
	node := h.NewNode().Init("--empty-repo", "--profile=test").StartDaemon("--offline")
	client := node.GatewayClient()

	// Import fixtures
	r, err := os.Open("./fixtures/TestGatewayHAMTDirectory.car")
	assert.NoError(t, err)
	defer r.Close()
	err = node.IPFSDagImport(r, fixtureCid)
	assert.NoError(t, err)

	// Fetch HAMT directory succeeds with minimal refs
	resp := client.Get(fmt.Sprintf("/ipfs/%s/", hamtCid))
	assert.Equal(t, http.StatusOK, resp.StatusCode)
}

```

This is a Go test that tests the fetching of different ranges of a file using the IPFS (InterPlanetary File System) client.

The test starts by sending a GET request to the IPFS endpoint for a file with a specified file name (fileCid), which includes a range of two numbers: 1276 to 1279 (inclusive) and 29839070 to 29839080 (inclusive).

The test then asserts that the server returns an HTTP status code of 200 (OK) and the response body contains the expected content (the file's content in this case).

The test then goes on to send a second GET request to the IPFS endpoint for the same file with a range of two different numbers (1276-1279 and 29839070-29839080), and asserts that the server returns the same HTTP status code (200).

Finally, the test checks that the server returns the correct Content-Range header and the correct content type (the "iana" format in this case).

This test is a very basic example to check if the IPFS client can fetch the content of a file and if it can return the correct information.


```
func TestGatewayHAMTRanges(t *testing.T) {
	t.Parallel()

	const (
		// fileCid is the CID of the large HAMT-sharded file.
		fileCid = "bafybeiae5abzv6j3ucqbzlpnx3pcqbr2otbnpot7d2k5pckmpymin4guau"

		// fixtureCid is the CID of root of the DAG that is a subset of fileCid DAG
		// representing the minimal set of blocks necessary for a simple byte range request.
		fixtureCid = "bafybeicgsg3lwyn3yl75lw7sn4zhyj5dxtb7wfxwscpq6yzippetmr2w3y"
	)

	// Start node
	h := harness.NewT(t)
	node := h.NewNode().Init("--empty-repo", "--profile=test").StartDaemon("--offline")
	client := node.GatewayClient()

	// Import fixtures
	r, err := os.Open("./fixtures/TestGatewayMultiRange.car")
	assert.NoError(t, err)
	defer r.Close()
	err = node.IPFSDagImport(r, fixtureCid)
	assert.NoError(t, err)

	t.Run("Succeeds Fetching Range", func(t *testing.T) {
		t.Parallel()

		resp := client.Get(fmt.Sprintf("/ipfs/%s", fileCid), func(r *http.Request) {
			r.Header.Set("Range", "bytes=1276-1279")
		})
		assert.Equal(t, http.StatusPartialContent, resp.StatusCode)
		assert.Equal(t, "bytes 1276-1279/109266405", resp.Headers.Get("Content-Range"))
		assert.Equal(t, "iana", resp.Body)
	})

	t.Run("Succeeds Fetching Second Range", func(t *testing.T) {
		t.Parallel()

		resp := client.Get(fmt.Sprintf("/ipfs/%s", fileCid), func(r *http.Request) {
			r.Header.Set("Range", "bytes=29839070-29839080")
		})
		assert.Equal(t, http.StatusPartialContent, resp.StatusCode)
		assert.Equal(t, "bytes 29839070-29839080/109266405", resp.Headers.Get("Content-Range"))
		assert.Equal(t, "EXAMPLE.COM", resp.Body)
	})

	t.Run("Succeeds Fetching First Range of Multi-range Request", func(t *testing.T) {
		t.Parallel()

		resp := client.Get(fmt.Sprintf("/ipfs/%s", fileCid), func(r *http.Request) {
			r.Header.Set("Range", "bytes=1276-1279, 29839070-29839080")
		})
		assert.Equal(t, http.StatusPartialContent, resp.StatusCode)
		assert.Equal(t, "bytes 1276-1279/109266405", resp.Headers.Get("Content-Range"))
		assert.Equal(t, "iana", resp.Body)
	})
}

```

# `/opt/kubo/test/cli/gateway_test.go`

该代码是一个 Go 语言编写的命令行工具 cli 的包。它定义了一个名为 "cli" 的包，其中定义了一些函数和类型。

具体来说，该包通过以下方式定义了一些内置函数：

- "fmt.Printf"：用于格式化输出。
- "net/http"：用于创建 HTTP 客户端。
- "os"：用于操作系统相关操作。
- "path/filepath"：用于文件和目录操作。
- "regexp"：用于正则表达式匹配。
- "strconv"：用于字符串转义。
- "strings"：用于字符串操作。
- "testing"：用于测试相关操作。
- "github.com/ipfs/kubo/config"：导入 IPFS 和 Kubernetes 的配置。
- "github.com/ipfs/kubo/test/cli/harness"：导入用于测试的 harness 函数。
- "github.com/ipfs/kubo/test/cli/testutils"：导入用于测试的工具函数。
- "github.com/libp2p/go-libp2p/core/peer"：导入用于 IPFS 网络的底层协议的客户端。
- "github.com/multiformats/go-multiaddr"：导入用于 IPFS 地址的客户端。
- "github.com/multiformats/go-multibase"：导入用于 IPFS 数据结构支持的客户端。
- "github.com/stretchr/testify/assert"：用于断言的客户端。
- "github.com/stretchr/testify/require"：用于断言的客户端。


```
package cli

import (
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"os"
	"path/filepath"
	"regexp"
	"strconv"
	"strings"
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/multiformats/go-multiaddr"
	manet "github.com/multiformats/go-multiaddr/net"
	"github.com/multiformats/go-multibase"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

```

This appears to be a testing framework for a JavaScript harness that implements HTTP/IPFS. The tests are organized into categories based on whether they test for a specific configuration setting or for an HTTP error.

The tests include a variety of HTTP request and response checks, such as verifying that the HTTP response includes the "Content-Type" header with the value "text/html". Additionally, some tests check for specific HTTP status codes, such as "404", "500", and "502".

It appears that the harness code being tested is actually the code implementing the HTTP/IPFS interface, rather than the harness code that provides the interface.


```
func TestGateway(t *testing.T) {
	t.Parallel()
	h := harness.NewT(t)
	node := h.NewNode().Init().StartDaemon("--offline")
	cid := node.IPFSAddStr("Hello Worlds!")

	peerID, err := peer.ToCid(node.PeerID()).StringOfBase(multibase.Base36)
	assert.NoError(t, err)

	client := node.GatewayClient()
	client.TemplateData = map[string]string{
		"CID":    cid,
		"PeerID": peerID,
	}

	t.Run("GET IPFS path succeeds", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/{{.CID}}")
		assert.Equal(t, 200, resp.StatusCode)
	})

	t.Run("GET IPFS path with explicit ?filename succeeds with proper header", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/{{.CID}}?filename=testтест.pdf")
		assert.Equal(t, 200, resp.StatusCode)
		assert.Equal(t,
			`inline; filename="test____.pdf"; filename*=UTF-8''test%D1%82%D0%B5%D1%81%D1%82.pdf`,
			resp.Headers.Get("Content-Disposition"),
		)
	})

	t.Run("GET IPFS path with explicit ?filename and &download=true succeeds with proper header", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/{{.CID}}?filename=testтест.mp4&download=true")
		assert.Equal(t, 200, resp.StatusCode)
		assert.Equal(t,
			`attachment; filename="test____.mp4"; filename*=UTF-8''test%D1%82%D0%B5%D1%81%D1%82.mp4`,
			resp.Headers.Get("Content-Disposition"),
		)
	})

	// https://github.com/ipfs/go-ipfs/issues/4025#issuecomment-342250616
	t.Run("GET for Server Worker registration outside of an IPFS content root errors", func(t *testing.T) {
		t.Parallel()
		resp := client.Get("/ipfs/{{.CID}}?filename=sw.js", client.WithHeader("Service-Worker", "script"))
		assert.Equal(t, 400, resp.StatusCode)
		assert.Contains(t, resp.Body, "navigator.serviceWorker: registration is not allowed for this scope")
	})

	t.Run("GET IPFS directory path succeeds", func(t *testing.T) {
		t.Parallel()
		client := node.GatewayClient().DisableRedirects()

		pageContents := "hello i am a webpage"
		fileContents := "12345"
		h.WriteFile("dir/test", fileContents)
		h.WriteFile("dir/dirwithindex/index.html", pageContents)
		cids := node.IPFS("add", "-r", "-q", filepath.Join(h.Dir, "dir")).Stdout.Lines()

		rootCID := cids[len(cids)-1]
		client.TemplateData = map[string]string{
			"IndexFileCID": cids[0],
			"TestFileCID":  cids[1],
			"RootCID":      rootCID,
		}

		t.Run("GET IPFS the index file CID", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/{{.IndexFileCID}}")
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, pageContents, resp.Body)
		})

		t.Run("GET IPFS the test file CID", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/{{.TestFileCID}}")
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, fileContents, resp.Body)
		})

		t.Run("GET IPFS directory with index.html returns redirect to add trailing slash", func(t *testing.T) {
			t.Parallel()
			resp := client.Head("/ipfs/{{.RootCID}}/dirwithindex?query=to-remember")
			assert.Equal(t, 301, resp.StatusCode)
			assert.Equal(t,
				fmt.Sprintf("/ipfs/%s/dirwithindex/?query=to-remember", rootCID),
				resp.Headers.Get("Location"),
			)
		})

		// This enables go get to parse go-import meta tags from index.html files stored in IPFS
		// https://github.com/ipfs/kubo/pull/3963
		t.Run("GET IPFS directory with index.html and no trailing slash returns expected output when go-get is passed", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/{{.RootCID}}/dirwithindex?go-get=1")
			assert.Equal(t, pageContents, resp.Body)
		})

		t.Run("GET IPFS directory with index.html and trailing slash returns expected output", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/{{.RootCID}}/dirwithindex/?query=to-remember")
			assert.Equal(t, pageContents, resp.Body)
		})

		t.Run("GET IPFS nonexistent file returns 404 (Not Found)", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/{{.RootCID}}/pleaseDontAddMe")
			assert.Equal(t, 404, resp.StatusCode)
		})

		t.Run("GET IPFS invalid CID returns 400 (Bad Request)", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/QmInvalid/pleaseDontAddMe")
			assert.Equal(t, 400, resp.StatusCode)
		})

		t.Run("GET IPFS inlined zero-length data object returns ok code (200)", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/bafkqaaa")
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, "0", resp.Resp.Header.Get("Content-Length"))
			assert.Equal(t, "", resp.Body)
		})

		t.Run("GET IPFS inlined zero-length data object with byte range returns ok code (200)", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/bafkqaaa", client.WithHeader("Range", "bytes=0-1048575"))
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, "0", resp.Resp.Header.Get("Content-Length"))
			assert.Equal(t, "text/plain", resp.Resp.Header.Get("Content-Type"))
		})

		t.Run("GET /ipfs/ipfs/{cid} returns redirect to the valid path", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/ipfs/bafkqaaa?query=to-remember")
			assert.Contains(t,
				resp.Body,
				`<meta http-equiv="refresh" content="10;url=/ipfs/bafkqaaa?query=to-remember" />`,
			)
			assert.Contains(t,
				resp.Body,
				`<link rel="canonical" href="/ipfs/bafkqaaa?query=to-remember" />`,
			)
		})
	})

	t.Run("IPNS", func(t *testing.T) {
		t.Parallel()
		node.IPFS("name", "publish", "--allow-offline", "--ttl", "42h", cid)

		t.Run("GET invalid IPNS root returns 500 (Internal Server Error)", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipns/QmInvalid/pleaseDontAddMe")
			assert.Equal(t, 500, resp.StatusCode)
		})

		t.Run("GET IPNS path succeeds", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipns/{{.PeerID}}")
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, "Hello Worlds!", resp.Body)
		})

		t.Run("GET IPNS path has correct Cache-Control", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipns/{{.PeerID}}")
			assert.Equal(t, 200, resp.StatusCode)
			cacheControl := resp.Headers.Get("Cache-Control")
			assert.True(t, strings.HasPrefix(cacheControl, "public, max-age="))
			maxAge, err := strconv.Atoi(strings.TrimPrefix(cacheControl, "public, max-age="))
			assert.NoError(t, err)
			assert.True(t, maxAge-151200 < 60) // MaxAge within 42h and 42h-1m
		})

		t.Run("GET /ipfs/ipns/{peerid} returns redirect to the valid path", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/ipfs/ipns/{{.PeerID}}?query=to-remember")

			assert.Contains(t,
				resp.Body,
				fmt.Sprintf(`<meta http-equiv="refresh" content="10;url=/ipns/%s?query=to-remember" />`, peerID),
			)
			assert.Contains(t,
				resp.Body,
				fmt.Sprintf(`<link rel="canonical" href="/ipns/%s?query=to-remember" />`, peerID),
			)
		})
	})

	t.Run("GET invalid IPFS path errors", func(t *testing.T) {
		t.Parallel()
		assert.Equal(t, 400, client.Get("/ipfs/12345").StatusCode)
	})

	t.Run("GET invalid path errors", func(t *testing.T) {
		t.Parallel()
		assert.Equal(t, 404, client.Get("/12345").StatusCode)
	})

	// TODO: these tests that use the API URL shouldn't be part of gateway tests...
	t.Run("GET /webui returns 301 or 302", func(t *testing.T) {
		t.Parallel()
		resp := node.APIClient().DisableRedirects().Get("/webui")
		assert.Contains(t, []int{302, 301}, resp.StatusCode)
	})

	t.Run("GET /webui/ returns 301 or 302", func(t *testing.T) {
		t.Parallel()
		resp := node.APIClient().DisableRedirects().Get("/webui/")
		assert.Contains(t, []int{302, 301}, resp.StatusCode)
	})

	t.Run("GET /webui/ returns user-specified headers", func(t *testing.T) {
		t.Parallel()

		header := "Access-Control-Allow-Origin"
		values := []string{"http://localhost:3000", "https://webui.ipfs.io"}

		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.API.HTTPHeaders = map[string][]string{header: values}
		})
		node.StartDaemon()

		resp := node.APIClient().DisableRedirects().Get("/webui/")
		assert.Equal(t, resp.Headers.Values(header), values)
		assert.Contains(t, []int{302, 301}, resp.StatusCode)
	})

	t.Run("GET /logs returns logs", func(t *testing.T) {
		t.Parallel()
		apiClient := node.APIClient()
		reqURL := apiClient.BuildURL("/logs")

		ctx, cancel := context.WithCancel(context.Background())
		defer cancel()

		req, err := http.NewRequestWithContext(ctx, http.MethodGet, reqURL, nil)
		require.NoError(t, err)

		resp, err := apiClient.Client.Do(req)
		require.NoError(t, err)
		defer resp.Body.Close()

		// read the first line of the output and parse its JSON
		dec := json.NewDecoder(resp.Body)
		event := struct{ Event string }{}
		err = dec.Decode(&event)
		require.NoError(t, err)

		assert.Equal(t, "log API client connected", event.Event)
	})

	t.Run("POST /api/v0/version succeeds", func(t *testing.T) {
		t.Parallel()
		resp := node.APIClient().Post("/api/v0/version", nil)
		assert.Equal(t, 200, resp.StatusCode)

		assert.Len(t, resp.Resp.TransferEncoding, 1)
		assert.Equal(t, "chunked", resp.Resp.TransferEncoding[0])

		vers := struct{ Version string }{}
		err := json.Unmarshal([]byte(resp.Body), &vers)
		require.NoError(t, err)
		assert.NotEmpty(t, vers.Version)
	})

	t.Run("pprof", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		apiClient := node.APIClient()
		t.Run("mutex", func(t *testing.T) {
			t.Parallel()
			t.Run("setting the mutex fraction works (negative so it doesn't enable)", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Post("/debug/pprof-mutex/?fraction=-1", nil)
				assert.Equal(t, 200, resp.StatusCode)
			})
			t.Run("mutex endpoint doesn't accept a string as an argument", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Post("/debug/pprof-mutex/?fraction=that_is_a_string", nil)
				assert.Equal(t, 400, resp.StatusCode)
			})
			t.Run("mutex endpoint returns 405 on GET", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Get("/debug/pprof-mutex/?fraction=-1")
				assert.Equal(t, 405, resp.StatusCode)
			})
		})
		t.Run("block", func(t *testing.T) {
			t.Parallel()
			t.Run("setting the block profiler rate works (0 so it doesn't enable)", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Post("/debug/pprof-block/?rate=0", nil)
				assert.Equal(t, 200, resp.StatusCode)
			})
			t.Run("block profiler endpoint doesn't accept a string as an argument", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Post("/debug/pprof-block/?rate=that_is_a_string", nil)
				assert.Equal(t, 400, resp.StatusCode)
			})
			t.Run("block profiler endpoint returns 405 on GET", func(t *testing.T) {
				t.Parallel()
				resp := apiClient.Get("/debug/pprof-block/?rate=0")
				assert.Equal(t, 405, resp.StatusCode)
			})
		})
	})

	t.Run("index content types", func(t *testing.T) {
		t.Parallel()
		h := harness.NewT(t)
		node := h.NewNode().Init().StartDaemon()

		h.WriteFile("index/index.html", "<p></p>")
		cid := node.IPFS("add", "-Q", "-r", filepath.Join(h.Dir, "index")).Stderr.Trimmed()

		apiClient := node.APIClient()
		apiClient.TemplateData = map[string]string{"CID": cid}

		t.Run("GET index.html has correct content type", func(t *testing.T) {
			t.Parallel()
			res := apiClient.Get("/ipfs/{{.CID}}/")
			assert.Equal(t, "text/html; charset=utf-8", res.Resp.Header.Get("Content-Type"))
		})

		t.Run("HEAD index.html has no content", func(t *testing.T) {
			t.Parallel()
			res := apiClient.Head("/ipfs/{{.CID}}/")
			assert.Equal(t, "", res.Body)
			assert.Equal(t, "", res.Resp.Header.Get("Content-Length"))
		})
	})

	t.Run("readonly API", func(t *testing.T) {
		t.Parallel()

		client := node.GatewayClient()

		fileContents := "12345"
		h.WriteFile("readonly/dir/test", fileContents)
		cids := node.IPFS("add", "-r", "-q", filepath.Join(h.Dir, "readonly/dir")).Stdout.Lines()

		rootCID := cids[len(cids)-1]
		client.TemplateData = map[string]string{"RootCID": rootCID}

		t.Run("Get IPFS directory file through readonly API succeeds", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/api/v0/cat?arg={{.RootCID}}/test")
			assert.Equal(t, 200, resp.StatusCode)
			assert.Equal(t, fileContents, resp.Body)
		})

		t.Run("refs IPFS directory file through readonly API succeeds", func(t *testing.T) {
			t.Parallel()
			resp := client.Get("/api/v0/refs?arg={{.RootCID}}/test")
			assert.Equal(t, 200, resp.StatusCode)
		})

		t.Run("test gateway API is sanitized", func(t *testing.T) {
			t.Parallel()
			for _, cmd := range []string{
				"add",
				"block/put",
				"bootstrap",
				"config",
				"dag/put",
				"dag/import",
				"dht",
				"diag",
				"id",
				"mount",
				"name/publish",
				"object/put",
				"object/new",
				"object/patch",
				"pin",
				"ping",
				"repo",
				"stats",
				"swarm",
				"file",
				"update",
				"bitswap",
			} {
				t.Run(cmd, func(t *testing.T) {
					cmd := cmd
					t.Parallel()
					assert.Equal(t, 404, client.Get("/api/v0/"+cmd).StatusCode)
				})
			}
		})
	})

	t.Run("refs/local", func(t *testing.T) {
		t.Parallel()
		gatewayAddr := URLStrToMultiaddr(node.GatewayURL())
		res := node.RunIPFS("--api", gatewayAddr.String(), "refs", "local")
		assert.Contains(t,
			res.Stderr.Trimmed(),
			`Error: invalid path "local":`,
		)
	})

	t.Run("raw leaves node", func(t *testing.T) {
		t.Parallel()
		contents := "This is RAW!"
		cid := node.IPFSAddStr(contents, "--raw-leaves")
		assert.Equal(t, contents, client.Get("/ipfs/"+cid).Body)
	})

	t.Run("compact blocks", func(t *testing.T) {
		t.Parallel()
		block1 := "\x0a\x09\x08\x02\x12\x03\x66\x6f\x6f\x18\x03"
		block2 := "\x0a\x04\x08\x02\x18\x06\x12\x24\x0a\x22\x12\x20\xcf\x92\xfd\xef\xcd\xc3\x4c\xac\x00\x9c" +
			"\x8b\x05\xeb\x66\x2b\xe0\x61\x8d\xb9\xde\x55\xec\xd4\x27\x85\xe9\xec\x67\x12\xf8\xdf\x65" +
			"\x12\x24\x0a\x22\x12\x20\xcf\x92\xfd\xef\xcd\xc3\x4c\xac\x00\x9c\x8b\x05\xeb\x66\x2b\xe0" +
			"\x61\x8d\xb9\xde\x55\xec\xd4\x27\x85\xe9\xec\x67\x12\xf8\xdf\x65"

		node.PipeStrToIPFS(block1, "block", "put")
		block2CID := node.PipeStrToIPFS(block2, "block", "put", "--cid-codec=dag-pb").Stdout.Trimmed()

		resp := client.Get("/ipfs/" + block2CID)
		assert.Equal(t, 200, resp.StatusCode)
		assert.Equal(t, "foofoo", resp.Body)
	})

	t.Run("verify gateway file", func(t *testing.T) {
		t.Parallel()
		r := regexp.MustCompile(`Gateway server listening on (?P<addr>.+)\s`)
		matches := r.FindStringSubmatch(node.Daemon.Stdout.String())
		ma, err := multiaddr.NewMultiaddr(matches[1])
		require.NoError(t, err)
		netAddr, err := manet.ToNetAddr(ma)
		require.NoError(t, err)
		expURL := "http://" + netAddr.String()

		b, err := os.ReadFile(filepath.Join(node.Dir, "gateway"))
		require.NoError(t, err)

		assert.Equal(t, expURL, string(b))
	})

	t.Run("verify gateway file diallable while on unspecified", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Addresses.Gateway = config.Strings{"/ip4/127.0.0.1/tcp/32563"}
		})
		node.StartDaemon()

		b, err := os.ReadFile(filepath.Join(node.Dir, "gateway"))
		require.NoError(t, err)

		assert.Equal(t, "http://127.0.0.1:32563", string(b))
	})

	t.Run("NoFetch", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(2).Init()
		node1 := nodes[0]
		node2 := nodes[1]

		node1.UpdateConfig(func(cfg *config.Config) {
			cfg.Gateway.NoFetch = true
		})

		node2PeerID, err := peer.ToCid(node2.PeerID()).StringOfBase(multibase.Base36)
		assert.NoError(t, err)

		nodes.StartDaemons().Connect()

		t.Run("not present", func(t *testing.T) {
			cidFoo := node2.IPFSAddStr("foo")

			t.Run("not present key from node 1", func(t *testing.T) {
				t.Parallel()
				assert.Equal(t, 500, node1.GatewayClient().Get("/ipfs/"+cidFoo).StatusCode)
			})

			t.Run("not present IPNS key from node 1", func(t *testing.T) {
				t.Parallel()
				assert.Equal(t, 500, node1.GatewayClient().Get("/ipns/"+node2PeerID).StatusCode)
			})
		})

		t.Run("present", func(t *testing.T) {
			cidBar := node1.IPFSAddStr("bar")

			t.Run("present key from node 1", func(t *testing.T) {
				t.Parallel()
				assert.Equal(t, 200, node1.GatewayClient().Get("/ipfs/"+cidBar).StatusCode)
			})

			t.Run("present IPNS key from node 1", func(t *testing.T) {
				t.Parallel()
				node2.IPFS("name", "publish", "/ipfs/"+cidBar)
				assert.Equal(t, 200, node1.GatewayClient().Get("/ipns/"+node2PeerID).StatusCode)
			})
		})
	})

	t.Run("DeserializedResponses", func(t *testing.T) {
		type testCase struct {
			globalValue                   config.Flag
			gatewayValue                  config.Flag
			deserializedGlobalStatusCode  int
			deserializedGatewayStaticCode int
			message                       string
		}

		setHost := func(r *http.Request) {
			r.Host = "example.com"
		}

		withAccept := func(accept string) func(r *http.Request) {
			return func(r *http.Request) {
				r.Header.Set("Accept", accept)
			}
		}

		withHostAndAccept := func(accept string) func(r *http.Request) {
			return func(r *http.Request) {
				setHost(r)
				withAccept(accept)(r)
			}
		}

		makeTest := func(test *testCase) func(t *testing.T) {
			return func(t *testing.T) {
				t.Parallel()

				node := harness.NewT(t).NewNode().Init()
				node.UpdateConfig(func(cfg *config.Config) {
					cfg.Gateway.DeserializedResponses = test.globalValue
					cfg.Gateway.PublicGateways = map[string]*config.GatewaySpec{
						"example.com": {
							Paths:                 []string{"/ipfs", "/ipns"},
							DeserializedResponses: test.gatewayValue,
						},
					}
				})
				node.StartDaemon()

				cidFoo := node.IPFSAddStr("foo")
				client := node.GatewayClient()

				deserializedPath := "/ipfs/" + cidFoo

				blockPath := deserializedPath + "?format=raw"
				carPath := deserializedPath + "?format=car"

				// Global Check (Gateway.DeserializedResponses)
				assert.Equal(t, http.StatusOK, client.Get(blockPath).StatusCode)
				assert.Equal(t, http.StatusOK, client.Get(deserializedPath, withAccept("application/vnd.ipld.raw")).StatusCode)

				assert.Equal(t, http.StatusOK, client.Get(carPath).StatusCode)
				assert.Equal(t, http.StatusOK, client.Get(deserializedPath, withAccept("application/vnd.ipld.car")).StatusCode)

				assert.Equal(t, test.deserializedGlobalStatusCode, client.Get(deserializedPath).StatusCode)
				assert.Equal(t, test.deserializedGlobalStatusCode, client.Get(deserializedPath, withAccept("application/json")).StatusCode)

				// Public Gateway (example.com) Check (Gateway.PublicGateways[example.com].DeserializedResponses)
				assert.Equal(t, http.StatusOK, client.Get(blockPath, setHost).StatusCode)
				assert.Equal(t, http.StatusOK, client.Get(deserializedPath, withHostAndAccept("application/vnd.ipld.raw")).StatusCode)

				assert.Equal(t, http.StatusOK, client.Get(carPath, setHost).StatusCode)
				assert.Equal(t, http.StatusOK, client.Get(deserializedPath, withHostAndAccept("application/vnd.ipld.car")).StatusCode)

				assert.Equal(t, test.deserializedGatewayStaticCode, client.Get(deserializedPath, setHost).StatusCode)
				assert.Equal(t, test.deserializedGatewayStaticCode, client.Get(deserializedPath, withHostAndAccept("application/json")).StatusCode)
			}
		}

		for _, test := range []*testCase{
			{config.True, config.Default, http.StatusOK, http.StatusOK, "when Gateway.DeserializedResponses is globally enabled, leaving implicit default for Gateway.PublicGateways[example.com] should inherit the global setting (enabled)"},
			{config.False, config.Default, http.StatusNotAcceptable, http.StatusNotAcceptable, "when Gateway.DeserializedResponses is globally disabled, leaving implicit default on Gateway.PublicGateways[example.com] should inherit the global setting (disabled)"},
			{config.False, config.True, http.StatusNotAcceptable, http.StatusOK, "when Gateway.DeserializedResponses is globally disabled, explicitly enabling on Gateway.PublicGateways[example.com] should override global (enabled)"},
			{config.True, config.False, http.StatusOK, http.StatusNotAcceptable, "when Gateway.DeserializedResponses is globally enabled, explicitly disabling on Gateway.PublicGateways[example.com] should override global (disabled)"},
		} {
			t.Run(test.message, makeTest(test))
		}
	})

	t.Run("DisableHTMLErrors", func(t *testing.T) {
		t.Parallel()

		t.Run("Returns HTML error without DisableHTMLErrors, Accept contains text/html", func(t *testing.T) {
			t.Parallel()

			node := harness.NewT(t).NewNode().Init()
			node.StartDaemon()
			client := node.GatewayClient()

			res := client.Get("/ipfs/invalid-thing", func(r *http.Request) {
				r.Header.Set("Accept", "text/html")
			})
			assert.NotEqual(t, http.StatusOK, res.StatusCode)
			assert.Contains(t, res.Resp.Header.Get("Content-Type"), "text/html")
		})

		t.Run("Does not return HTML error with DisableHTMLErrors enabled, and Accept contains text/html", func(t *testing.T) {
			t.Parallel()

			node := harness.NewT(t).NewNode().Init()
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Gateway.DisableHTMLErrors = config.True
			})
			node.StartDaemon()
			client := node.GatewayClient()

			res := client.Get("/ipfs/invalid-thing", func(r *http.Request) {
				r.Header.Set("Accept", "text/html")
			})
			assert.NotEqual(t, http.StatusOK, res.StatusCode)
			assert.NotContains(t, res.Resp.Header.Get("Content-Type"), "text/html")
		})
	})
}

```