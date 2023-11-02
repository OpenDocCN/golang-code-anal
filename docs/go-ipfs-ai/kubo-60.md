# go-ipfs 源码解析 60

# `test/integration/pubsub_msg_seen_cache_test.go`

该代码是一个 Go 语言编写的测试框架，用于测试 Kubernetes 集群中的数据存储库（如 ByteSHA256、Gomake、以及一些其他工具）的 integration。

该框架引入了来自不同库的依赖，包括：

- "github.com/ipfs/boxo/bootstrap"：用于 Boxo 数据存储库的 bootstrap 工具
 - "github.com/ipfs/kubo/config"：用于 Kubernetes 集群的配置管理器
 - "github.com/ipfs/kubo/core":Kubernetes 集群的核心组件
 - "github.com/ipfs/kubo/core/coreapi":Kubernetes 集群的 API
 - "github.com/ipfs/kubo/node/libp2p"：用于在 Node 机上管理 libp2p 的库
 - "github.com/ipfs/kubo/repo"：用于 Kubernetes 集群的存储库访问
 - "github.com/ipfs/go-datastore"：用于 Go 语言中的 datastore 库
 - "github.com/ipfs/go-libp2p-pubsub":Go 语言中用于 libp2p 通信的库
 - "github.com/ipfs/go-libp2p-pubsub/pb":Go 语言中用于 libp2p 通信的协议定义
 - "github.com/ipfs/go-libp2p/core/peer"：用于在节点之间通信的 libp2p 库
 - "github.com/libp2p/go-libp2p/core":Go 语言中用于管理本地点的库
 - "github.com/libp2p/go-libp2p/event":Go 语言中用于事件通知的库
 - "github.com/libp2p/go-libp2p/rpc":Go 语言中用于 RPC 通信的库

该框架中的一些函数和变量被初始化为常量，用于提供测试数据的来源。


```go
package integrationtest

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"testing"
	"time"

	"go.uber.org/fx"

	"github.com/ipfs/boxo/bootstrap"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	libp2p2 "github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/repo"

	"github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"

	pubsub "github.com/libp2p/go-libp2p-pubsub"
	pubsub_pb "github.com/libp2p/go-libp2p-pubsub/pb"
	"github.com/libp2p/go-libp2p-pubsub/timecache"
	"github.com/libp2p/go-libp2p/core/peer"

	mock "github.com/ipfs/kubo/core/mock"
	mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"
)

```

This is a Go test that checks if the cache TTL test works as expected.

The `RunMessageSeenCacheTTLTest` function tests the cache TTL by sending a message and waiting for it to expire. If it does not expire within the specified timeout, the test will consider it to be a failure.

The `mockNode` function is responsible for setting up the mock node that will be used for the tests. It creates a `core.IpfsNode` object and returns it.

The `mockNet` function is used to create a mock network that will be used to send the message. It sets up a mock network with a specific IP address and port for the test.

The `config.Init` function initializes the cache with the given configuration options such as the IP address and port of the mock network.

The `ttl` variable is used to store the TTL of the cache.

The `RunMessageSeenCacheTTLTest` function sends a message and waits for it to expire.

The `mockNode` function is used to create a `core.IpfsNode` object and


```go
func TestMessageSeenCacheTTL(t *testing.T) {
	t.Skip("skipping PubSub seen cache TTL test due to flakiness")
	if err := RunMessageSeenCacheTTLTest(t, "10s"); err != nil {
		t.Fatal(err)
	}
}

func mockNode(ctx context.Context, mn mocknet.Mocknet, pubsubEnabled bool, seenMessagesCacheTTL string) (*core.IpfsNode, error) {
	ds := syncds.MutexWrap(datastore.NewMapDatastore())
	cfg, err := config.Init(io.Discard, 2048)
	if err != nil {
		return nil, err
	}
	count := len(mn.Peers())
	cfg.Addresses.Swarm = []string{
		fmt.Sprintf("/ip4/18.0.%d.%d/tcp/4001", count>>16, count&0xFF),
	}
	cfg.Datastore = config.Datastore{}
	if pubsubEnabled {
		cfg.Pubsub.Enabled = config.True
		var ttl *config.OptionalDuration
		if len(seenMessagesCacheTTL) > 0 {
			ttl = &config.OptionalDuration{}
			if err = ttl.UnmarshalJSON([]byte(seenMessagesCacheTTL)); err != nil {
				return nil, err
			}
		}
		cfg.Pubsub.SeenMessagesTTL = ttl
	}
	return core.NewNode(ctx, &core.BuildCfg{
		Online:  true,
		Routing: libp2p2.DHTServerOption,
		Repo: &repo.Mock{
			C: *cfg,
			D: ds,
		},
		Host: mock.MockHostOption(mn),
		ExtraOpts: map[string]bool{
			"pubsub": pubsubEnabled,
		},
	})
}

```

This is a function that handles sending a message with a unique message ID to a SeenMessagesTTL window. The window is implemented using a sliding window algorithm, and messages are sent with increasing IDs, so that messages sent one minute after the last message have not been seen yet. The function takes a message text (`msgTxt`) as input, and sends the message with a unique message ID (`MsgID`) in response. It then checks if the message has already been sent to the window, and if not, sends it. Once the message has been sent, the expiration for the message should also be pushed out for a whole SeenMessagesTTL window.


```go
func RunMessageSeenCacheTTLTest(t *testing.T, seenMessagesCacheTTL string) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	var bootstrapNode, consumerNode, producerNode *core.IpfsNode
	var bootstrapPeerID, consumerPeerID, producerPeerID peer.ID

	mn := mocknet.New()
	bootstrapNode, err := mockNode(ctx, mn, false, "") // no need for PubSub configuration
	if err != nil {
		t.Fatal(err)
	}
	bootstrapPeerID = bootstrapNode.PeerHost.ID()
	defer bootstrapNode.Close()

	consumerNode, err = mockNode(ctx, mn, true, seenMessagesCacheTTL) // use passed seen cache TTL
	if err != nil {
		t.Fatal(err)
	}
	consumerPeerID = consumerNode.PeerHost.ID()
	defer consumerNode.Close()

	ttl, err := time.ParseDuration(seenMessagesCacheTTL)
	if err != nil {
		t.Fatal(err)
	}

	// Used for logging the timeline
	startTime := time.Time{}

	// Used for overriding the message ID
	sendMsgID := ""

	// Set up the pubsub message ID generation override for the producer
	core.RegisterFXOptionFunc(func(info core.FXNodeInfo) ([]fx.Option, error) {
		var pubsubOptions []pubsub.Option
		pubsubOptions = append(
			pubsubOptions,
			pubsub.WithSeenMessagesTTL(ttl),
			pubsub.WithMessageIdFn(func(pmsg *pubsub_pb.Message) string {
				now := time.Now()
				if startTime.Second() == 0 {
					startTime = now
				}
				timeElapsed := now.Sub(startTime).Seconds()
				msg := string(pmsg.Data)
				from, _ := peer.IDFromBytes(pmsg.From)
				var msgID string
				if from == producerPeerID {
					msgID = sendMsgID
					t.Logf("sending [%s] with message ID [%s] at T%fs", msg, msgID, timeElapsed)
				} else {
					msgID = pubsub.DefaultMsgIdFn(pmsg)
				}
				return msgID
			}),
			pubsub.WithSeenMessagesStrategy(timecache.Strategy_LastSeen),
		)
		return append(
			info.FXOptions,
			fx.Provide(libp2p2.TopicDiscovery()),
			fx.Decorate(libp2p2.GossipSub(pubsubOptions...)),
		), nil
	})

	producerNode, err = mockNode(ctx, mn, false, "") // PubSub configuration comes from overrides above
	if err != nil {
		t.Fatal(err)
	}
	producerPeerID = producerNode.PeerHost.ID()
	defer producerNode.Close()

	t.Logf("bootstrap peer=%s, consumer peer=%s, producer peer=%s", bootstrapPeerID, consumerPeerID, producerPeerID)

	producerAPI, err := coreapi.NewCoreAPI(producerNode)
	if err != nil {
		t.Fatal(err)
	}
	consumerAPI, err := coreapi.NewCoreAPI(consumerNode)
	if err != nil {
		t.Fatal(err)
	}

	err = mn.LinkAll()
	if err != nil {
		t.Fatal(err)
	}

	bis := bootstrapNode.Peerstore.PeerInfo(bootstrapNode.PeerHost.ID())
	bcfg := bootstrap.BootstrapConfigWithPeers([]peer.AddrInfo{bis})
	if err = producerNode.Bootstrap(bcfg); err != nil {
		t.Fatal(err)
	}
	if err = consumerNode.Bootstrap(bcfg); err != nil {
		t.Fatal(err)
	}

	// Set up the consumer subscription
	const TopicName = "topic"
	consumerSubscription, err := consumerAPI.PubSub().Subscribe(ctx, TopicName)
	if err != nil {
		t.Fatal(err)
	}
	// Utility functions defined inline to include context in closure
	now := func() float64 {
		return time.Since(startTime).Seconds()
	}
	ctr := 0
	msgGen := func() string {
		ctr++
		return fmt.Sprintf("msg_%d", ctr)
	}
	produceMessage := func() string {
		msgTxt := msgGen()
		err = producerAPI.PubSub().Publish(ctx, TopicName, []byte(msgTxt))
		if err != nil {
			t.Fatal(err)
		}
		return msgTxt
	}
	consumeMessage := func(msgTxt string, shouldFind bool) {
		// Set up a separate timed context for receiving messages
		rxCtx, rxCancel := context.WithTimeout(context.Background(), time.Second)
		defer rxCancel()
		msg, err := consumerSubscription.Next(rxCtx)
		if shouldFind {
			if err != nil {
				t.Logf("expected but did not receive [%s] at T%fs", msgTxt, now())
				t.Fatal(err)
			}
			t.Logf("received [%s] at T%fs", string(msg.Data()), now())
			if !bytes.Equal(msg.Data(), []byte(msgTxt)) {
				t.Fatalf("consumed data [%s] does not match published data [%s]", string(msg.Data()), msgTxt)
			}
		} else {
			if err == nil {
				t.Logf("not expected but received [%s] at T%fs", string(msg.Data()), now())
				t.Fail()
			}
			t.Logf("did not receive [%s] at T%fs", msgTxt, now())
		}
	}

	const MsgID1 = "MsgID1"
	const MsgID2 = "MsgID2"
	const MsgID3 = "MsgID3"

	// Send message 1 with the message ID we're going to duplicate
	sentMsg1 := time.Now()
	sendMsgID = MsgID1
	msgTxt := produceMessage()
	// Should find the message because it's new
	consumeMessage(msgTxt, true)

	// Send message 2 with a duplicate message ID
	sendMsgID = MsgID1
	msgTxt = produceMessage()
	// Should NOT find message because it got deduplicated (sent 2 times within the SeenMessagesTTL window).
	consumeMessage(msgTxt, false)

	// Send message 3 with a new message ID
	sendMsgID = MsgID2
	msgTxt = produceMessage()
	// Should find the message because it's new
	consumeMessage(msgTxt, true)

	// Wait till just before the SeenMessagesTTL window has passed since message 1 was sent
	time.Sleep(time.Until(sentMsg1.Add(ttl - 100*time.Millisecond)))

	// Send message 4 with a duplicate message ID
	sendMsgID = MsgID1
	msgTxt = produceMessage()
	// Should NOT find the message because it got deduplicated (sent 3 times within the SeenMessagesTTL window). This
	// time, however, the expiration for the message should also get pushed out for a whole SeenMessagesTTL window since
	// the default time cache now implements a sliding window algorithm.
	consumeMessage(msgTxt, false)

	// Send message 5 with a duplicate message ID. This will be a second after the last attempt above since NOT finding
	// a message takes a second to determine. That would put this attempt at ~1 second after the SeenMessagesTTL window
	// starting at message 1 has expired.
	sentMsg5 := time.Now()
	sendMsgID = MsgID1
	msgTxt = produceMessage()
	// Should NOT find the message, because it got deduplicated (sent 2 times since the updated SeenMessagesTTL window
	// started). This time again, the expiration should get pushed out for another SeenMessagesTTL window.
	consumeMessage(msgTxt, false)

	// Send message 6 with a message ID that hasn't been seen within a SeenMessagesTTL window
	sendMsgID = MsgID2
	msgTxt = produceMessage()
	// Should find the message since last read > SeenMessagesTTL, so it looks like a new message.
	consumeMessage(msgTxt, true)

	// Sleep for a full SeenMessagesTTL window to let cache entries time out
	time.Sleep(time.Until(sentMsg5.Add(ttl + 100*time.Millisecond)))

	// Send message 7 with a duplicate message ID
	sendMsgID = MsgID1
	msgTxt = produceMessage()
	// Should find the message this time since last read > SeenMessagesTTL, so it looks like a new message.
	consumeMessage(msgTxt, true)

	// Send message 8 with a brand new message ID
	//
	// This step is not strictly necessary, but has been added for good measure.
	sendMsgID = MsgID3
	msgTxt = produceMessage()
	// Should find the message because it's new
	consumeMessage(msgTxt, true)
	return nil
}

```

# `test/integration/three_legged_cat_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Libp2p 库中的Integrationtest 包。

首先，该代码导入了 Integrationtest 包中需要的所有依赖。然后，通过 "files" 包中的 "testutil" 函数，导入了其中需要的依赖。

接下来，该代码定义了一个名为 "test_bootstrap2" 的函数，该函数使用 IPFS(即 InterPlanetary File System)的 "bootstrap2" 包创建一个根目录 "test_bootstrap2"。这个目录将用于存放测试用例和其他测试相关的内容。

然后，该代码定义了一个名为 "test_createKubernetesCluster" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包创建一个 Kubernetes 集群。该函数中包含一个示例用法，用于演示如何创建一个测试用的 Kubernetes 集群。

接下来，该代码定义了一个名为 "test_asyncWithContext" 的函数，该函数使用 Go 的 "context" 和 "time" 包，实现了异步操作的 "WithContext" 选项。该函数可以在主函数中使用 "context.WithAnonymousContext" 的方式获取当前的上下文，然后使用 "time.sleep" 函数来模拟延迟操作。

接着，该代码定义了一个名为 "test_getBox" 的函数，该函数使用 "bootstrap2" 包中的 "files.Box" 函数获取一个测试文件夹中的 "box.json" 文件，并返回其中的内容。

然后，该代码定义了一个名为 "test_writeFile" 的函数，该函数使用 "files" 包中的 "testutil" 函数，向测试文件夹中的 "box.json" 文件中写入一些内容。

接下来，该代码定义了一个名为 "test_appendToFile" 的函数，该函数使用 "files" 包中的 "testutil" 函数，将内容追加到 "box.json" 文件的末尾。

接着，该代码定义了一个名为 "test_normalizeFilename" 的函数，该函数使用 "strings" 包将文件名中的 "integrationtest" 字符串截断，只返回剩余的字符串。

然后，该代码定义了一个名为 "test_getKubernetesEndpoints" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包，获取一个 Kubernetes 集群中的所有端点。

接下来，该代码定义了一个名为 "test_getPeerID" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包，获取一个 Kubernetes 集群中的对端 ID。

接着，该代码定义了一个名为 "test_setPeerID" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包，设置一个 Kubernetes 集群的对端 ID。

然后，该代码定义了一个名为 "test_createBootstrapClient" 的函数，该函数使用 "bootstrap2" 包中的 "files.Client" 函数，创建一个测试用的 "bootstrap2" 客户端。

接下来，该代码定义了一个名为 "test_getPeerClient" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包，获取一个 Kubernetes 集群中的客户端。

接着，该代码定义了一个名为 "test_sendRequest" 的函数，该函数使用 "test_createBootstrapClient" 函数创建的 "bootstrap2" 客户端，发送一个 HTTP GET 请求到对端，然后获取响应的内容。

然后，该代码定义了一个名为 "test_sendPeerRequest" 的函数，该函数使用 Kubernetes 的 "coreapi" 和 "mock" 包，发送一个 HTTP GET 请求到对端，然后获取响应的内容。

接下来，该代码定义了一个名为 "test_getBlock" 的函数，该函数使用 "bootstrap2" 包中的 "files.Box" 函数，使用给定的 "box.json" 文件中的内容，返回一个 block 对象。

接着，该代码定义了一个名为 "test_asyncWrite" 的函数，该函数使用 Go 的 "context" 和 "time" 包，实现了异步操作的 "WithContext" 选项。该函数可以在主函数中使用 "context.WithAnonymousContext" 的方式获取当前的上下文，然后使用 "time.sleep" 函数来模拟延迟操作。该函数的实现在 "test_writeFile"


```go
package integrationtest

import (
	"bytes"
	"context"
	"errors"
	"io"
	"math"
	"testing"
	"time"

	bootstrap2 "github.com/ipfs/boxo/bootstrap"
	"github.com/ipfs/kubo/core/coreapi"
	mock "github.com/ipfs/kubo/core/mock"
	"github.com/ipfs/kubo/thirdparty/unit"

	"github.com/ipfs/boxo/files"
	testutil "github.com/libp2p/go-libp2p-testing/net"
	"github.com/libp2p/go-libp2p/core/peer"
	mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"
)

```

这两段代码是在测试“ThreeLeggedCatTransfer”和“ThreeLeggedCatDegenerateSlowBlockstore”两个函数。它们都使用了“RunThreeLeggedCat”函数作为测试的输入，这个函数的输入参数是一个随机长度的字节切片，以及一个“LatencyConfig”对象，这个对象包含了网络延迟、路由延迟和块存储延迟，用于设置延迟水平。

第一个测试函数“TestThreeLeggedCatTransfer”的作用是测试“RunThreeLeggedCat”函数在延迟配置为0的情况下是否能够正常工作。如果函数运行时出现了错误，那么这个错误会被记录下来，并输出错误信息。

第二个测试函数“TestThreeLeggedCatDegenerateSlowBlockstore”的作用是测试“RunThreeLeggedCat”函数在延迟配置为50微秒的情况下是否能够正常工作。如果函数运行时出现了错误，那么这个错误会被记录下来，并输出错误信息。由于这个测试函数中使用了“SkipUnlessEpic”函数，因此只有当第一个测试失败时才会输出错误信息，否则不会输出错误信息。


```go
func TestThreeLeggedCatTransfer(t *testing.T) {
	conf := testutil.LatencyConfig{
		NetworkLatency:    0,
		RoutingLatency:    0,
		BlockstoreLatency: 0,
	}
	if err := RunThreeLeggedCat(RandomBytes(100*unit.MB), conf); err != nil {
		t.Fatal(err)
	}
}

func TestThreeLeggedCatDegenerateSlowBlockstore(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{BlockstoreLatency: 50 * time.Millisecond}
	if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
		t.Fatal(err)
	}
}

```

这两个函数的作用是测试“ThreeLeggedCatDegenerateSlowNetwork”和“ThreeLeggedCatDegenerateSlowRouting”两个函数的 behavior。

具体来说，这两个函数分别创建了一个测试上下文，并使用“RunThreeLeggedCat”函数来向该上下文发送随机数据。如果发送的数据包有任何错误，则函数的断言将会失败并输出相应的错误信息。

func TestThreeLeggedCatDegenerateSlowNetwork(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{NetworkLatency: 400 * time.Millisecond}
	if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
		t.Fatal(err)
	}
}

func TestThreeLeggedCatDegenerateSlowRouting(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{RoutingLatency: 400 * time.Millisecond}
	if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
		t.Fatal(err)
	}
}

注意，上述代码非常简短，无法获取到具体的函数实现。


```go
func TestThreeLeggedCatDegenerateSlowNetwork(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{NetworkLatency: 400 * time.Millisecond}
	if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
		t.Fatal(err)
	}
}

func TestThreeLeggedCatDegenerateSlowRouting(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{RoutingLatency: 400 * time.Millisecond}
	if err := RunThreeLeggedCat(RandomBytes(1*unit.KB), conf); err != nil {
		t.Fatal(err)
	}
}

```

This is a Go program that creates a mock node that implements the Core API for the Unix file system.

The node has a `math.MaxInt32` configuration, which limits the value it can accept as an input.

The node has three methods: `Bootstrap`, `Unixfs`, and `Get`.

The `Bootstrap` method is used to initialize the node and register itself as a bootstrap node.

The `Unixfs` method is used to perform file operations on the file system, such as writing and reading.

The `Get` method is used to retrieve the contents of the file specified by the `附加的` field.

In the `UnixfsGet` method, the file is first obtained using the `附加的` field, and then the contents are retrieved using the `Get` method.

In the `Bootstrap` method, the `math.MaxInt32` configuration is set. This limits the value the node can accept as an input.

The `附加的` field is a `math.MaxInt32` configuration, which limits the value it can accept as an input.

The `math.MaxInt32` configuration is applied to the `附加的` field, which is set in the `math.MaxInt32` configuration.

The `CreateCoreAPI` method is used to create a `coreapi.CoreAPI` object for the `腺苷酸` node, which is the node that implements the Core API for the Unix file system.

The `CoreAPI` object is used to implement the `Unixfs` and `Get` methods for the `腺苷酸` node.

The `Bootstrap` method is used to initialize the node and register itself as a bootstrap node.

The `Unixfs` method is used to perform file operations on the file system, such as writing and reading.

The `Get` method is used to retrieve the contents of the file specified by the `附加的` field.

The `附加的` field is a `math.MaxInt32` configuration, which limits the value it can accept as an input.

The `math.MaxInt32` configuration is applied to the `附加的` field, which is set in the `math.MaxInt32` configuration.

The `CreateCoreAPI` method is used to create a `coreapi.CoreAPI` object for the `CoreAPI` node, which is the node that implements the Unix file system API.

The `CoreAPI` object is used to implement the `Unixfs` and `Get` methods for the `CoreAPI` node.

The `Bootstrap` method is used to initialize the node and register itself as a bootstrap node.

The `Unixfs` method is used to perform file operations on the file system, such as writing and reading.

The `Get` method is used to retrieve the contents of the file specified by the `附加的` field.

The `附加的` field is a `math.MaxInt32` configuration, which limits the value it can accept as an input.

The `math.MaxInt32` configuration is applied to the `附加的` field, which is set in the `math.MaxInt32` configuration.


```go
func TestThreeLeggedCat100MBMacbookCoastToCoast(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{}.NetworkNYtoSF().BlockstoreSlowSSD2014().RoutingSlow()
	if err := RunThreeLeggedCat(RandomBytes(100*unit.MB), conf); err != nil {
		t.Fatal(err)
	}
}

func RunThreeLeggedCat(data []byte, conf testutil.LatencyConfig) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// create network
	mn := mocknet.New()
	mn.SetLinkDefaults(mocknet.LinkOptions{
		Latency: conf.NetworkLatency,
		// TODO add to conf. This is tricky because we want 0 values to be functional.
		Bandwidth: math.MaxInt32,
	})

	bootstrap, err := mock.MockPublicNode(ctx, mn)
	if err != nil {
		return err
	}
	defer bootstrap.Close()

	adder, err := mock.MockPublicNode(ctx, mn)
	if err != nil {
		return err
	}
	defer adder.Close()

	catter, err := mock.MockPublicNode(ctx, mn)
	if err != nil {
		return err
	}
	defer catter.Close()

	adderAPI, err := coreapi.NewCoreAPI(adder)
	if err != nil {
		return err
	}

	catterAPI, err := coreapi.NewCoreAPI(catter)
	if err != nil {
		return err
	}

	err = mn.LinkAll()
	if err != nil {
		return err
	}

	bis := bootstrap.Peerstore.PeerInfo(bootstrap.PeerHost.ID())
	bcfg := bootstrap2.BootstrapConfigWithPeers([]peer.AddrInfo{bis})
	if err := adder.Bootstrap(bcfg); err != nil {
		return err
	}
	if err := catter.Bootstrap(bcfg); err != nil {
		return err
	}

	added, err := adderAPI.Unixfs().Add(ctx, files.NewBytesFile(data))
	if err != nil {
		return err
	}

	readerCatted, err := catterAPI.Unixfs().Get(ctx, added)
	if err != nil {
		return err
	}

	// verify
	var bufout bytes.Buffer
	_, err = io.Copy(&bufout, readerCatted.(io.Reader))
	if err != nil {
		return err
	}
	if !bytes.Equal(bufout.Bytes(), data) {
		return errors.New("catted data does not match added data")
	}
	return nil
}

```

# `test/integration/wan_lan_dht_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Kubernetes Peerset 中基于 libp2p 的节点选举过程。具体来说，这个测试框架实现了以下功能：

1. 导入必要的依赖项，包括 `integrationtest`、`github.com/ipfs/go-cid`、`github.com/ipfs/kubo/core`、`github.com/ipfs/kubo/core/mock`、`github.com/ipfs/kubo/core/node/libp2p`、`github.com/ipfs/go-libp2p-testing/net`、`github.com/libp2p/go-libp2p/core/network`、`github.com/libp2p/go-libp2p/core/peerstore` 和 `github.com/multiformats/go-multiaddr`。

2. 定义了一个 `IntegrationTest` 类型，它继承自 `testing.T` 类型，用于提供测试应用程序所需的基本功能。

3. 实现了一个名为 `Run` 的函数，该函数使用 `testing.T` 提供的 `Run` 函数来运行测试。

4. 内部导入了 `context`、`encoding/binary`、`fmt`、`math` 和 `math/rand` 包，以及从 `github.com/ipfs/go-cid`、`github.com/ipfs/kubo/core`、`github.com/ipfs/kubo/core/mock` 和 `github.com/ipfs/go-libp2p-testing/net` 导入的虚拟节点。

5. 通过 `fmt` 打印一些测试应用程序相关信息，如 `IntegrationTest` 的类名、该测试套件的描述以及 `Run` 函数的标识符。

6. 通过 `time.sleep` 函数让程序在某些时候暂停一下，从而在运行测试时延长疲劳。

7. 通过 `libp2p2` 导入的 `CoreAPI` 和 `CoreAPI_Mock` 类型实现了一个 `test_corenet_node_election` 函数，该函数实现了测试 `CoreAPI_Mock` 类型中 `CoreAPI_NodeElection_Mock` 函数的接口。

8. 通过 `MAKE(ipfs.AccessCloserPath=<path>)` 命令行参数让 `libp2p-个小档案` 代理应用更加喜好这个测试套件。

9. 最后，通过 `Coverage` 标签来覆盖 `integrationtest` 包的其他部分，从而将测试应用程序的运行结果保存在代码中，以便稍后覆盖代码以进行调试。


```go
package integrationtest

import (
	"context"
	"encoding/binary"
	"fmt"
	"math"
	"math/rand"
	"net"
	"testing"
	"time"

	"github.com/ipfs/go-cid"
	"github.com/ipfs/kubo/core"
	mock "github.com/ipfs/kubo/core/mock"
	libp2p2 "github.com/ipfs/kubo/core/node/libp2p"

	testutil "github.com/libp2p/go-libp2p-testing/net"
	corenet "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peerstore"
	mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"

	ma "github.com/multiformats/go-multiaddr"
)

```

这两个函数旨在测试DHT的连通性。这两个函数分别测试高速网络连接和低速网络连接的连通性。

具体来说，第一个函数（TestDHTConnectivityFast）测试高速网络连接。在函数内部，我们创建了一个名为`conf`的测试配置对象。这个配置对象包含一个网络延迟设置，我们将其设置为400 * time.Millisecond。然后我们使用`RunDHTConnectivity`函数设置这个配置对象，并传入一个5秒的超时。函数内部还有一个错误处理，如果发生错误，函数会打印错误并中止执行。

第二个函数（TestDHTConnectivitySlowNetwork）测试低速网络连接。我们创建了一个名为`conf`的测试配置对象，并将`NetworkLatency`设置为40 * time.Millisecond。然后我们使用`RunDHTConnectivity`函数设置这个配置对象，并传入一个5秒的超时。函数内部还有一个错误处理，如果发生错误，函数会打印错误并中止执行。

这两个函数的目的是测试DHT在不同的网络连接设置下的连通性。如果任何函数发生错误，那么说明DHT的连通性存在问题。


```go
func TestDHTConnectivityFast(t *testing.T) {
	conf := testutil.LatencyConfig{
		NetworkLatency:    0,
		RoutingLatency:    0,
		BlockstoreLatency: 0,
	}
	if err := RunDHTConnectivity(conf, 5); err != nil {
		t.Fatal(err)
	}
}

func TestDHTConnectivitySlowNetwork(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{NetworkLatency: 400 * time.Millisecond}
	if err := RunDHTConnectivity(conf, 5); err != nil {
		t.Fatal(err)
	}
}

```

这段代码是一个用于测试 DHT 连接性及慢路由路由性能的 Go 语言函数。函数名为 `TestDHTConnectivitySlowRouting`，属于 `testing.T` 标准测试框架。

具体来说，这段代码以下面两行开头。这两行代码定义了一个名为 `func TestDHTConnectivitySlowRouting` 的函数。函数体以下面的行。

这两行代码定义了一个名为 `conf` 的变量，该变量是一个包含路由延迟配置的 `testutil.LatencyConfig` 类型。接下来的 `if err := RunDHTConnectivity(conf, 5); err != nil` 代码块可能引发异常，用于验证实际运行时发生的错误。

接下来定义了两个字符串类型的变量 `wanPrefix` 和 `lanPrefix`，分别指定了 `wan` 和 `lan` 前缀。然后定义了一个名为 `makeAddr` 的函数，该函数接收两个参数，第一个参数是一个 `uint32` 类型的数字，表示目标 IPv6 地址中的前缀长度，第二个参数 `bool` 表示是否使用 ASN（自动系统编号）作为前缀。函数实现将IPv6地址中的前缀设置为输入参数的前缀，然后使用 `ma.Multiaddr` 类型将IPv6地址转换为 `ma.Multiaddr` 类型。

最后，由于以上代码片段，无法提供有关函数实际作用的信息，因此无法提供代码的更多解释。


```go
func TestDHTConnectivitySlowRouting(t *testing.T) {
	SkipUnlessEpic(t)
	conf := testutil.LatencyConfig{RoutingLatency: 400 * time.Millisecond}
	if err := RunDHTConnectivity(conf, 5); err != nil {
		t.Fatal(err)
	}
}

// wan prefix must have a real corresponding ASN for the peer diversity filter to work.
var (
	wanPrefix = net.ParseIP("2001:218:3004::")
	lanPrefix = net.ParseIP("fe80::")
)

func makeAddr(n uint32, wan bool) ma.Multiaddr {
	var ip net.IP
	if wan {
		ip = append(net.IP{}, wanPrefix...)
	} else {
		ip = append(net.IP{}, lanPrefix...)
	}

	binary.LittleEndian.PutUint32(ip[12:], n)
	addr, _ := ma.NewMultiaddr(fmt.Sprintf("/ip6/%s/tcp/4242", ip))
	return addr
}

```

It looks like you are testing the behavior of a Peer using the "wanPeer" and "lanPeer" connectors. Here are the steps you are taking:

1. Creating a "testPeer" with a unique identifier.
2. Configuring the "Peerstore" with the address of the "wanPeer" and the IP address of the "lanPeer".
3. Creating a "lanPeer" by calling the "core.NewNode" method with the "online" flag set and the "mn" network context.
4. Adding the "wanPeer" and the "lanPeer" to the list of "lanPeers".
5. Adding the IP address of the "testPeer" to the list of "testAddr".
6. Linking the "testPeer" to the first "lanPeer".
7. Opening the "PeerHost" of the first "lanPeer".
8. Opening the "PeerStore" of the first "lanPeer".
9. Testing if the "testPeer" is connected to the "lanPeer".
10. Cancelling the startup context.

It's difficult to say what you are trying to test without more information about the implementation and the expected behavior. If you have any specific questions or concerns, please let me know.


```go
func RunDHTConnectivity(conf testutil.LatencyConfig, numPeers int) error {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// create network
	mn := mocknet.New()
	mn.SetLinkDefaults(mocknet.LinkOptions{
		Latency:   conf.NetworkLatency,
		Bandwidth: math.MaxInt32,
	})

	testPeer, err := core.NewNode(ctx, &core.BuildCfg{
		Online: true,
		Host:   mock.MockHostOption(mn),
	})
	if err != nil {
		return err
	}
	defer testPeer.Close()

	wanPeers := []*core.IpfsNode{}
	lanPeers := []*core.IpfsNode{}

	connectionContext, connCtxCancel := context.WithTimeout(ctx, 15*time.Second)
	defer connCtxCancel()
	for i := 0; i < numPeers; i++ {
		wanPeer, err := core.NewNode(ctx, &core.BuildCfg{
			Online:  true,
			Routing: libp2p2.DHTServerOption,
			Host:    mock.MockHostOption(mn),
		})
		if err != nil {
			return err
		}
		defer wanPeer.Close()
		wanAddr := makeAddr(uint32(i), true)
		wanPeer.Peerstore.AddAddr(wanPeer.Identity, wanAddr, peerstore.PermanentAddrTTL)
		for _, p := range wanPeers {
			_, _ = mn.LinkPeers(p.Identity, wanPeer.Identity)
			_ = wanPeer.PeerHost.Connect(connectionContext, p.Peerstore.PeerInfo(p.Identity))
		}
		wanPeers = append(wanPeers, wanPeer)

		lanPeer, err := core.NewNode(ctx, &core.BuildCfg{
			Online: true,
			Host:   mock.MockHostOption(mn),
		})
		if err != nil {
			return err
		}
		defer lanPeer.Close()
		lanAddr := makeAddr(uint32(i), false)
		lanPeer.Peerstore.AddAddr(lanPeer.Identity, lanAddr, peerstore.PermanentAddrTTL)
		for _, p := range lanPeers {
			_, _ = mn.LinkPeers(p.Identity, lanPeer.Identity)
			_ = lanPeer.PeerHost.Connect(connectionContext, p.Peerstore.PeerInfo(p.Identity))
		}
		lanPeers = append(lanPeers, lanPeer)
	}
	connCtxCancel()

	// Add interfaces / addresses to test peer.
	wanAddr := makeAddr(0, true)
	testPeer.Peerstore.AddAddr(testPeer.Identity, wanAddr, peerstore.PermanentAddrTTL)
	lanAddr := makeAddr(0, false)
	testPeer.Peerstore.AddAddr(testPeer.Identity, lanAddr, peerstore.PermanentAddrTTL)

	// The test peer is connected to one lan peer.
	for _, p := range lanPeers {
		if _, err := mn.LinkPeers(testPeer.Identity, p.Identity); err != nil {
			return err
		}
	}
	err = testPeer.PeerHost.Connect(ctx, lanPeers[0].Peerstore.PeerInfo(lanPeers[0].Identity))
	if err != nil {
		return err
	}

	startupCtx, startupCancel := context.WithTimeout(ctx, time.Second*60)
```

It looks like you are implementing a network test to validate that a remote device is able to establish a connection with a test device and provide a new CID (CID is a metric that is used to measure the legibility of a host in a network). The test is starting by connecting to a wan peer and then moving to a lan peer. The lan peer is providing a new CID, and the test device is expected to validate that it can find the CID. If the test device is unable to find the CID, it will consider that the remote device is not providing a valid metric and the connection will be terminated. If the lan peer provides a CID, the test device will validate that it can find the CID by connecting to a wan peer and then using the FindProvidersAsync method to check for available providers. The LanPeer class is used to implement the network connections and the CID is used to metric the connectability of the remote device.


```go
StartupWait:
	for {
		select {
		case err := <-testPeer.DHT.LAN.RefreshRoutingTable():
			if err != nil {
				fmt.Printf("Error refreshing routing table: %v\n", err)
			}
			if testPeer.DHT.LAN.RoutingTable() == nil ||
				testPeer.DHT.LAN.RoutingTable().Size() == 0 ||
				err != nil {
				time.Sleep(100 * time.Millisecond)
				continue
			}
			break StartupWait
		case <-startupCtx.Done():
			startupCancel()
			return fmt.Errorf("expected faster dht bootstrap")
		}
	}
	startupCancel()

	// choose a lan peer and validate lan DHT is functioning.
	i := rand.Intn(len(lanPeers))
	if testPeer.PeerHost.Network().Connectedness(lanPeers[i].Identity) == corenet.Connected {
		i = (i + 1) % len(lanPeers)
		if testPeer.PeerHost.Network().Connectedness(lanPeers[i].Identity) == corenet.Connected {
			_ = testPeer.PeerHost.Network().ClosePeer(lanPeers[i].Identity)
			testPeer.PeerHost.Peerstore().ClearAddrs(lanPeers[i].Identity)
		}
	}
	// That peer will provide a new CID, and we'll validate the test node can find it.
	provideCid := cid.NewCidV1(cid.Raw, []byte("Lan Provide Record"))
	provideCtx, cancel := context.WithTimeout(ctx, time.Second)
	defer cancel()
	if err := lanPeers[i].DHT.Provide(provideCtx, provideCid, true); err != nil {
		return err
	}
	provChan := testPeer.DHT.FindProvidersAsync(provideCtx, provideCid, 0)
	prov, ok := <-provChan
	if !ok || prov.ID == "" {
		return fmt.Errorf("Expected provider. stream closed early")
	}
	if prov.ID != lanPeers[i].Identity {
		return fmt.Errorf("Unexpected lan peer provided record")
	}

	// Now, connect with a wan peer.
	for _, p := range wanPeers {
		if _, err := mn.LinkPeers(testPeer.Identity, p.Identity); err != nil {
			return err
		}
	}

	err = testPeer.PeerHost.Connect(ctx, wanPeers[0].Peerstore.PeerInfo(wanPeers[0].Identity))
	if err != nil {
		return err
	}

	startupCtx, startupCancel = context.WithTimeout(ctx, time.Second*60)
```

This is a function that performs a DHT providers Discovery operation on a WAN peer. It does this by following these steps:

1. Retrieves the wan peer from the list of wan peers and uses the `DHT.Provide` method provided by the DHT layer to establish a connection with the peer.
2. Retrieves the cid of the peer from the retrieved wan peer.
3. Retrieves a random wan peer to use as the provider.
4. Retrieves the provider's ID from the wan peer's DHT.
5. Retrieves the provider's ID from the wan peer's Identity.
6. Checks if the provider retrieved from the wan peer is expected or not.
7. If the provider is expected, it creates a new provider channel by calling `testPeer.DHT.FindProvidersAsync`.
8. If the provider is unexpected, it returns an error.
9. Finally, it closes the provider channel and waits for a new provider to be selected.

This function should be called within a context that has a timeout specified, such as a `context.WithTimeout`.


```go
WanStartupWait:
	for {
		select {
		case err := <-testPeer.DHT.WAN.RefreshRoutingTable():
			// if err != nil {
			//	fmt.Printf("Error refreshing routing table: %v\n", err)
			// }
			if testPeer.DHT.WAN.RoutingTable() == nil ||
				testPeer.DHT.WAN.RoutingTable().Size() == 0 ||
				err != nil {
				time.Sleep(100 * time.Millisecond)
				continue
			}
			break WanStartupWait
		case <-startupCtx.Done():
			startupCancel()
			return fmt.Errorf("expected faster wan dht bootstrap")
		}
	}
	startupCancel()

	// choose a wan peer and validate wan DHT is functioning.
	i = rand.Intn(len(wanPeers))
	if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected {
		i = (i + 1) % len(wanPeers)
		if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected {
			_ = testPeer.PeerHost.Network().ClosePeer(wanPeers[i].Identity)
			testPeer.PeerHost.Peerstore().ClearAddrs(wanPeers[i].Identity)
		}
	}

	// That peer will provide a new CID, and we'll validate the test node can find it.
	wanCid := cid.NewCidV1(cid.Raw, []byte("Wan Provide Record"))
	wanProvideCtx, cancel := context.WithTimeout(ctx, time.Second)
	defer cancel()
	if err := wanPeers[i].DHT.Provide(wanProvideCtx, wanCid, true); err != nil {
		return err
	}
	provChan = testPeer.DHT.FindProvidersAsync(wanProvideCtx, wanCid, 0)
	prov, ok = <-provChan
	if !ok || prov.ID == "" {
		return fmt.Errorf("Expected one provider, closed early")
	}
	if prov.ID != wanPeers[i].Identity {
		return fmt.Errorf("Unexpected lan peer provided record")
	}

	// Finally, re-share the lan provided cid from a wan peer and expect a merged result.
	i = rand.Intn(len(wanPeers))
	if testPeer.PeerHost.Network().Connectedness(wanPeers[i].Identity) == corenet.Connected {
		_ = testPeer.PeerHost.Network().ClosePeer(wanPeers[i].Identity)
		testPeer.PeerHost.Peerstore().ClearAddrs(wanPeers[i].Identity)
	}

	provideCtx, cancel = context.WithTimeout(ctx, time.Second)
	defer cancel()
	if err := wanPeers[i].DHT.Provide(provideCtx, provideCid, true); err != nil {
		return err
	}
	provChan = testPeer.DHT.FindProvidersAsync(provideCtx, provideCid, 0)
	prov, ok = <-provChan
	if !ok {
		return fmt.Errorf("Expected two providers, got 0")
	}
	prov, ok = <-provChan
	if !ok {
		return fmt.Errorf("Expected two providers, got 1")
	}

	return nil
}

```

# ipfs whole tests using the [sharness framework](https://github.com/pl-strflt/sharness/tree/feat/junit)

## Running all the tests

Just use `make` in this directory to run all the tests.
Run with `TEST_VERBOSE=1` to get helpful verbose output.

```go
TEST_VERBOSE=1 make
```

The usual ipfs env flags also apply:

```gosh
# the output will make your eyes bleed
IPFS_LOGGING=debug TEST_VERBOSE=1 make
```

To make the tests abort as soon as an error occurs, use the TEST_IMMEDIATE env variable:

```gosh
# this will abort as soon the first error occurs
TEST_IMMEDIATE=1 make
```

## Running just one test

You can run only one test script by launching it like a regular shell
script:

```go
$ ./t0010-basic-commands.sh
```

## Debugging one test

You can use the `-v` option to make it verbose and the `-i` option to
make it stop as soon as one test fails.
For example:

```go
$ ./t0010-basic-commands.sh -v -i
```

## Sharness

When running sharness tests from main Makefile or when `test_sharness_deps`
target is run dependencies for sharness
will be downloaded from its GitHub repo and installed in a "lib/sharness"
directory.

Please do not change anything in the "lib/sharness" directory.

If you really need some changes in sharness, please fork it from
[its canonical repo](https://github.com/mlafeldt/sharness/) and
send pull requests there.

## Writing Tests

Please have a look at existing tests and try to follow their example.

When possible and not too inefficient, that means most of the time,
an ipfs command should not be on the left side of a pipe, because if
the ipfs command fails (exit non zero), the pipe will mask this failure.
For example after `false | true`, `echo $?` prints 0 (despite `false`
failing).

It should be possible to put most of the code inside `test_expect_success`,
or sometimes `test_expect_failure`, blocks, and to chain all the commands
inside those blocks with `&&`, or `||` for diagnostic commands.

### Diagnostics

Make your test case output helpful for when running sharness verbosely.
This means cating certain files, or running diagnostic commands.
For example:

```go
test_expect_success ".ipfs/ has been created" '
  test -d ".ipfs" &&
  test -f ".ipfs/config" &&
  test -d ".ipfs/datastore" &&
  test -d ".ipfs/blocks" ||
  test_fsh ls -al .ipfs
'
```

The `|| ...` is a diagnostic run when the preceding command fails.
test_fsh is a shell function that echoes the args, runs the cmd,
and then also fails, making sure the test case fails. (wouldn't want
the diagnostic accidentally returning true and making it _seem_ like
the test case succeeded!).


### Testing commands on daemon or mounted

Use the provided functions in `lib/test-lib.sh` to run the daemon or mount:

To init, run daemon, and mount in one go:

```gosh
test_launch_ipfs_daemon_and_mount

test_expect_success "'ipfs add --help' succeeds" '
  ipfs add --help >actual
'

# other tests here...

# don't forget to kill the daemon!!
test_kill_ipfs_daemon
```

To init, run daemon, and then mount separately:

```gosh
test_init_ipfs

# tests inited but not running here

test_launch_ipfs_daemon

# tests running but not mounted here

test_mount_ipfs

# tests mounted here

# don't forget to kill the daemon!!
test_kill_ipfs_daemon
```


# Dataset description/sources

- lotus_testnet_export_256_multiroot.car
  - Export of the first 256 block of the testnet chain, with 3 tipset roots. Exported from Lotus by @traviperson on 2019-03-18


- lotus_devnet_genesis.car
  - Source: https://github.com/filecoin-project/lotus/blob/v0.2.10/build/genesis/devnet.car

- lotus_testnet_export_128.car
  - Export of the first 128 block of the testnet chain, exported from Lotus by @traviperson on 2019-03-24


- lotus_devnet_genesis_shuffled_noroots.car
- lotus_testnet_export_128_shuffled_noroots.car
  - versions of the above with an **empty** root array, and having all blocks shuffled

- lotus_devnet_genesis_shuffled_nulroot.car
- lotus_testnet_export_128_shuffled_nulroot.car
  - versions identical to the above, but with a single "empty-block" root each ( in order to work around go-car not following the current "roots can be empty" spec )

- combined_naked_roots_genesis_and_128.car
  - only the roots of `lotus_devnet_genesis.car` and `lotus_testnet_export_128.car`,to be used in combination with the root-less parts to validate "transactional" pinning

- lotus_testnet_export_128_v2.car
- lotus_devnet_genesis_v2.car
  - generated with `car index lotus_testnet_export_128.car > lotus_testnet_export_128_v2.car`
  - install `go-car` CLI from https://github.com/ipld/go-car

- partial-dag-scope-entity.car
  - unixfs directory entity exported from gateway via `?format=car&dag-scope=entity` ([IPIP-402](https://github.com/ipfs/specs/pull/402))
  - CAR roots includes directory CID, but only the root block is included in the CAR, making the DAG incomplete


# Dataset description/sources

- fixtures.car
  - raw CARv1

- QmUKd....ipns-record
  - ipns record, encoded with protocol buffer

- 12D3K....ipns-record
  - ipns record, encoded with protocol buffer

Generated with:

```gosh
# using ipfs version 0.21.0-dev (03a98280e3e642774776cd3d0435ab53e5dfa867)

# CIDv0to1 is necessary because raw-leaves are enabled by default during
# "ipfs add" with CIDv1 and disabled with CIDv0
CID_VAL="hello"
CIDv1=$(echo $CID_VAL | ipfs add --cid-version 1 -Q)
CIDv0=$(echo $CID_VAL | ipfs add --cid-version 0 -Q)
CIDv0to1=$(echo "$CIDv0" | ipfs cid base32)
# sha512 will be over 63char limit, even when represented in Base36
CIDv1_TOO_LONG=$(echo $CID_VAL | ipfs add --cid-version 1 --hash sha2-512 -Q)

echo CID_VAL=${CID_VAL}
echo CIDv1=${CIDv1}
echo CIDv0=${CIDv0}
echo CIDv0to1=${CIDv0to1}
echo CIDv1_TOO_LONG=${CIDv1_TOO_LONG}

# Directory tree crafted to test for edge cases like "/ipfs/ipfs/ipns/bar"
mkdir -p testdirlisting/ipfs/ipns &&
echo "hello" > testdirlisting/hello &&
echo "text-file-content" > testdirlisting/ipfs/ipns/bar &&
mkdir -p testdirlisting/api &&
mkdir -p testdirlisting/ipfs &&
echo "I am a txt file" > testdirlisting/api/file.txt &&
echo "I am a txt file" > testdirlisting/ipfs/file.txt &&
DIR_CID=$(ipfs add -Qr --cid-version 1 testdirlisting)

echo DIR_CID=${DIR_CID} # ./testdirlisting

ipfs files mkdir /t0114/
ipfs files cp /ipfs/${CIDv1} /t0114/
ipfs files cp /ipfs/${CIDv0} /t0114/
ipfs files cp /ipfs/${CIDv0to1} /t0114/
ipfs files cp /ipfs/${DIR_CID} /t0114/
ipfs files cp /ipfs/${CIDv1_TOO_LONG} /t0114/

ROOT=`ipfs files stat /t0114/ --hash`

ipfs dag export ${ROOT} > ./fixtures.car

# Then the keys

KEY_NAME=test_key_rsa_$RANDOM
RSA_KEY=$(ipfs key gen --ipns-base=b58mh --type=rsa --size=2048 ${KEY_NAME} | head -n1 | tr -d "\n")
RSA_IPNS_IDv0=$(echo "$RSA_KEY" | ipfs cid format -v 0)
RSA_IPNS_IDv1=$(echo "$RSA_KEY" | ipfs cid format -v 1 --mc libp2p-key -b base36)
RSA_IPNS_IDv1_DAGPB=$(echo "$RSA_IPNS_IDv0" | ipfs cid format -v 1 -b base36)

# publish a record valid for a 100 years
ipfs name publish --key ${KEY_NAME} --allow-offline -Q --ttl=876600h --lifetime=876600h "/ipfs/$CIDv1"
ipfs routing get /ipns/${RSA_KEY} > ${RSA_KEY}.ipns-record

echo RSA_KEY=${RSA_KEY}
echo RSA_IPNS_IDv0=${RSA_IPNS_IDv0}
echo RSA_IPNS_IDv1=${RSA_IPNS_IDv1}
echo RSA_IPNS_IDv1_DAGPB=${RSA_IPNS_IDv1_DAGPB}

KEY_NAME=test_key_ed25519_$RANDOM
ED25519_KEY=$(ipfs key gen --ipns-base=b58mh --type=ed25519 ${KEY_NAME} | head -n1 | tr -d "\n")
ED25519_IPNS_IDv0=$ED25519_KEY
ED25519_IPNS_IDv1=$(ipfs key list -l --ipns-base=base36 | grep ${KEY_NAME} | cut -d " " -f1 | tr -d "\n")
ED25519_IPNS_IDv1_DAGPB=$(echo "$ED25519_IPNS_IDv1" | ipfs cid format -v 1 -b base36 --mc dag-pb)

# ed25519 fits under 63 char limit when represented in base36
IPNS_ED25519_B58MH=$(ipfs key list -l --ipns-base b58mh | grep $KEY_NAME | cut -d" " -f1 | tr -d "\n")
IPNS_ED25519_B36CID=$(ipfs key list -l --ipns-base base36 | grep $KEY_NAME | cut -d" " -f1 | tr -d "\n")

# publish a record valid for a 100 years
ipfs name publish --key ${KEY_NAME} --allow-offline -Q --ttl=876600h --lifetime=876600h "/ipfs/$CIDv1"
ipfs routing get /ipns/${ED25519_KEY} > ${ED25519_KEY}.ipns-record

echo ED25519_KEY=${ED25519_KEY}
echo ED25519_IPNS_IDv0=${ED25519_IPNS_IDv0}
echo ED25519_IPNS_IDv1=${ED25519_IPNS_IDv1}
echo ED25519_IPNS_IDv1_DAGPB=${ED25519_IPNS_IDv1_DAGPB}
echo IPNS_ED25519_B58MH=${IPNS_ED25519_B58MH}
echo IPNS_ED25519_B36CID=${IPNS_ED25519_B36CID}

# CID_VAL=hello
# CIDv1=bafkreicysg23kiwv34eg2d7qweipxwosdo2py4ldv42nbauguluen5v6am
# CIDv0=QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
# CIDv0to1=bafybeiffndsajwhk3lwjewwdxqntmjm4b5wxaaanokonsggenkbw6slwk4
# CIDv1_TOO_LONG=bafkrgqhhyivzstcz3hhswshfjgy6ertgmnqeleynhwt4dlfsthi4hn7zgh4uvlsb5xncykzapi3ocd4lzogukir6ksdy6wzrnz6ohnv4aglcs
# DIR_CID=bafybeiht6dtwk3les7vqm6ibpvz6qpohidvlshsfyr7l5mpysdw2vmbbhe # ./testdirlisting

# RSA_KEY=QmVujd5Vb7moysJj8itnGufN7MEtPRCNHkKpNuA4onsRa3
# RSA_IPNS_IDv0=QmVujd5Vb7moysJj8itnGufN7MEtPRCNHkKpNuA4onsRa3
# RSA_IPNS_IDv1=k2k4r8m7xvggw5pxxk3abrkwyer625hg01hfyggrai7lk1m63fuihi7w
# RSA_IPNS_IDv1_DAGPB=k2jmtxu61bnhrtj301lw7zizknztocdbeqhxgv76l2q9t36fn9jbzipo

# ED25519_KEY=12D3KooWLQzUv2FHWGVPXTXSZpdHs7oHbXub2G5WC8Tx4NQhyd2d
# ED25519_IPNS_IDv0=12D3KooWLQzUv2FHWGVPXTXSZpdHs7oHbXub2G5WC8Tx4NQhyd2d
# ED25519_IPNS_IDv1=k51qzi5uqu5dk3v4rmjber23h16xnr23bsggmqqil9z2gduiis5se8dht36dam
# ED25519_IPNS_IDv1_DAGPB=k50rm9yjlt0jey4fqg6wafvqprktgbkpgkqdg27tpqje6iimzxewnhvtin9hhq
# IPNS_ED25519_B58MH=12D3KooWLQzUv2FHWGVPXTXSZpdHs7oHbXub2G5WC8Tx4NQhyd2d
# IPNS_ED25519_B36CID=k51qzi5uqu5dk3v4rmjber23h16xnr23bsggmqqil9z2gduiis5se8dht36dam
```


# Dataset description/sources

- fixtures.car
  - raw CARv1

generated with:

```gosh
# using ipfs version 0.18.1
mkdir -p rootDir/ipfs &&
mkdir -p rootDir/ipns &&
mkdir -p rootDir/api &&
mkdir -p rootDir/ą/ę &&
echo "I am a txt file on path with utf8" > rootDir/ą/ę/file-źł.txt &&
echo "I am a txt file in confusing /api dir" > rootDir/api/file.txt &&
echo "I am a txt file in confusing /ipfs dir" > rootDir/ipfs/file.txt &&
echo "I am a txt file in confusing /ipns dir" > rootDir/ipns/file.txt &&
DIR_CID=$(ipfs add -Qr --cid-version 1 rootDir) &&
FILE_CID=$(ipfs files stat --enc=json /ipfs/$DIR_CID/ą/ę/file-źł.txt | jq -r .Hash) &&
FILE_SIZE=$(ipfs files stat --enc=json /ipfs/$DIR_CID/ą/ę/file-źł.txt | jq -r .Size)
echo "$FILE_CID / $FILE_SIZE"

echo DIR_CID=${DIR_CID}
echo FILE_CID=${FILE_CID}
echo FILE_SIZE=${FILE_SIZE}

ipfs dag export ${DIR_CID} > ./fixtures.car

# DIR_CID=bafybeig6ka5mlwkl4subqhaiatalkcleo4jgnr3hqwvpmsqfca27cijp3i # ./rootDir/
# FILE_CID=bafkreialihlqnf5uwo4byh4n3cmwlntwqzxxs2fg5vanqdi3d7tb2l5xkm # ./rootDir/ą/ę/file-źł.txt
# FILE_SIZE=34
```


# Dataset description/sources

- fixtures.car
  - raw CARv1

generated with:

```gosh
# using ipfs version 0.21.0-dev (03a98280e3e642774776cd3d0435ab53e5dfa867)

mkdir -p root2/root3/root4 &&
echo "hello" > root2/root3/root4/index.html &&
ROOT1_CID=$(ipfs add -Qrw --cid-version 1 root2)
ROOT2_CID=$(ipfs resolve -r /ipfs/$ROOT1_CID/root2 | cut -d "/" -f3)
ROOT3_CID=$(ipfs resolve -r /ipfs/$ROOT1_CID/root2/root3 | cut -d "/" -f3)
ROOT4_CID=$(ipfs resolve -r /ipfs/$ROOT1_CID/root2/root3/root4 | cut -d "/" -f3)
FILE_CID=$(ipfs resolve -r /ipfs/$ROOT1_CID/root2/root3/root4/index.html | cut -d "/" -f3)

TEST_IPNS_ID=$(ipfs key gen --ipns-base=base36 --type=ed25519 cache_test_key | head -n1 | tr -d "\n")
# publish a record valid for a 100 years
ipfs name publish --key cache_test_key --allow-offline -Q --ttl=876600h --lifetime=876600h "/ipfs/$ROOT1_CID"
ipfs routing get /ipns/${TEST_IPNS_ID} > ${TEST_IPNS_ID}.ipns-record

echo ROOT1_CID=${ROOT1_CID} # ./
echo ROOT2_CID=${ROOT2_CID} # ./root2
echo ROOT3_CID=${ROOT3_CID} # ./root2/root3
echo ROOT4_CID=${ROOT4_CID} # ./root2/root3/root4
echo FILE_CID=${FILE_CID} # ./root2/root3/root4/index.html
echo TEST_IPNS_ID=${TEST_IPNS_ID}

ipfs dag export ${ROOT1_CID} > ./fixtures.car

# ROOT1_CID=bafybeib3ffl2teiqdncv3mkz4r23b5ctrwkzrrhctdbne6iboayxuxk5ui # ./
# ROOT2_CID=bafybeih2w7hjocxjg6g2ku25hvmd53zj7og4txpby3vsusfefw5rrg5sii # ./root2
# ROOT3_CID=bafybeiawdvhmjcz65x5egzx4iukxc72hg4woks6v6fvgyupiyt3oczk5ja # ./root2/root3
# ROOT4_CID=bafybeifq2rzpqnqrsdupncmkmhs3ckxxjhuvdcbvydkgvch3ms24k5lo7q # ./root2/root3/root4
# FILE_CID=bafkreicysg23kiwv34eg2d7qweipxwosdo2py4ldv42nbauguluen5v6am # ./root2/root3/root4/index.html
# TEST_IPNS_ID=k51qzi5uqu5dlxdsdu5fpuu7h69wu4ohp32iwm9pdt9nq3y5rpn3ln9j12zfhe
```


# OpenSSL generated keys for import/export tests

Created with commands:

```gobash
openssl genpkey -algorithm ED25519 > openssl_ed25519.pem
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 > openssl_rsa.pem
```

secp key used in the 'restrict import key' test.
From: https://www.openssl.org/docs/man1.1.1/man1/openssl-genpkey.html
```gobash
openssl genpkey -genparam -algorithm EC -out ecp.pem \
        -pkeyopt ec_paramgen_curve:secp384r1 \
        -pkeyopt ec_param_enc:named_curve
openssl genpkey -paramfile ecp.pem -out openssl_secp384r1.pem
rm ecp.pem
```
Note: The Bitcoin `secp256k1` curve which is what `go-libp2p-core/crypto`
actually generates and would be of interest to test against is not
recognized by the Go library:
```go
Error: parsing PKCS8 format: x509: failed to parse EC private key embedded
 in PKCS#8: x509: unknown elliptic curve
```
We keep the `secp384r1` type instead from the original openssl example.


# `test/sharness/t0280-plugin-data/example.go`

这段代码是一个Java程序，其中包含以下几个主要部分：

1. 导入一些外部依赖：
python
import "fmt"
import "os"

2. 定义了一个名为`Plugins`的切片，用于存储`main.plugin.Plugin`类型的实例：
go
var Plugins = []plugin.Plugin{
	&testPlugin{},
}

3. 创建了一个名为`testPlugin`的`plugin.Plugin`实例，并将其添加到`Plugins`切片中：
go
var _ = Plugins // used

4. 输出当前工作目录（即`os.GetCurrentWorkDir()`的值）。
go
fmt.Println("当前工作目录为：", os.GetCurrentWorkDir())

5. 遍历`Plugins`切片，并输出每个`plugin.Plugin`实例的名称：
go
for _, plugin := range Plugins {
	fmt.Println("Plugin名称：", plugin.Name)
}

6. 输出一个包含`Plugins`切片所有`plugin.Plugin`实例的`&plugin.Plugin`类型实例：
go
fmt.Println("Plugins:")
for _, plugin := range Plugins {
	fmt.Println("-", plugin)
}

7. 输出一个包含`testPlugin`的`plugin.Plugin`类型实例：
go
fmt.Println("testPlugin:")

8. 输出一个循环计数器，用于统计`Plugins`切片中的`plugin.Plugin`实例：
go
count := 0
for _, plugin := range Plugins {
	count++
	fmt.Println("-", plugin)
}

9. 最后，程序导出了一个名为`main`的包：
go
package main

总体来说，这段代码定义了一个用于测试Kubernetes集群的`plugin.Plugin`集合，并导出了其中的一个名为`testPlugin`的实例。通过循环遍历`Plugins`切片，程序将输出每个`plugin.Plugin`实例的名称，以及一个包含所有实例的`&plugin.Plugin`类型实例。


```go
package main

import (
	"fmt"
	"os"

	"github.com/ipfs/kubo/plugin"
)

var Plugins = []plugin.Plugin{
	&testPlugin{},
}

var _ = Plugins // used

```

这段代码定义了一个名为 `testPlugin` 的类型，它是一个 `test-plugin`，具有 `Name` 和 `Version` 字段，以及 `Init` 方法。

具体来说，这段代码的作用如下：

1. 定义了一个名为 `testPlugin` 的类型，该类型有一个名为 `Name` 的字段和一个名为 `Version` 的字段。

2. 定义了一个名为 `*testPlugin` 的指针类型，该指针类型有一个名为 `Name` 的字段和一个名为 `Version` 的字段。

3. 定义了一个名为 `Init` 的方法，该方法接收一个 `*plugin.Environment` 类型的参数，并输出 `test-plugin` 的名称和版本信息。

4. 在 `Init` 方法中，使用了 `fmt.Fprintf` 函数将 `test-plugin` 的名称和版本信息打印到控制台。

5. 在 `Name` 和 `Version` 字段中，使用了 `*` 符号来表示类型字段是可变的，即它们的值可以随时更改。

6. 在 `Init` 方法中，返回值为 `nil`，表示初始化操作成功。


```go
type testPlugin struct{}

func (*testPlugin) Name() string {
	return "test-plugin"
}

func (*testPlugin) Version() string {
	return "0.1.0"
}

func (*testPlugin) Init(env *plugin.Environment) error {
	fmt.Fprintf(os.Stderr, "testplugin %s\n", env.Repo)
	fmt.Fprintf(os.Stderr, "testplugin %v\n", env.Config)
	return nil
}

```

# Dataset description/sources

- fixtures.car
  - raw CARv1

generated with:

```gosh
# using ipfs version 0.18.1
HASH=$(echo "testing" | ipfs add -q)
ipfs dag export $HASH > fixtures.car

echo HASH=${HASH} # a file containing the string "testing"

# HASH=QmNYERzV2LfD2kkfahtfv44ocHzEFK1sLBaE7zdcYT2GAZ # a file containing the string "testing"
```


thirdparty consists of Golang packages that contain no go-ipfs dependencies and
may be vendored ipfs/go-ipfs at a later date.

packages under this directory _must not_ import packages under
`ipfs/go-ipfs` that are not also under `thirdparty`.
