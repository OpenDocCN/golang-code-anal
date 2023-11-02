# go-ipfs 源码解析 56

# `test/cli/rcmgr_test.go`

该代码是一个 Go 语言编写的名为 "cli" 的工具包。它主要用于在 libp2p 网络中进行测试，libp2p 是一个高性能的点对点网络，用于连接和交换数据。

该代码的作用是提供一个简单的 libp2p 工具包，用于在 Go 语言中测试 libp2p 网络。它包括以下功能：

1. 定义了一个名为 "testutils" 的测试实用程序，用于帮助进行测试。
2. 定义了一个名为 "cli" 的包，该包包含一些与 libp2p 网络有关的函数和类型。
3. 导入了自 "github.com/ipfs/kubo/config"、"github.com/ipfs/kubo/core/node/libp2p" 和 "github.com/ipfs/kubo/test/cli/harness" 等外部的库。
4. 通过导入自 "github.com/libp2p/go-libp2p/core/peer"、"github.com/libp2p/go-libp2p/core/protocol" 和 "github.com/libp2p/go-libp2p/p2p/host/resource-manager" 等库，实现了与 libp2p 网络的交互。
5. 通过导入 "github.com/stretchr/testify/assert" 和 "github.com/stretchr/testify/require" 等库，实现了断言测试。
6. 通过导入 "encoding/json" 和 "testing" 这两库，实现了 json 编码和解码，以及测试功能。


```go
package cli

import (
	"encoding/json"
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/protocol"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

```

This is a Go test function that tests the functionality of an IPFS (InterPlanetary File System) daemon when configured to handle high and low water marks for streams inbound. The IPFS daemon is configured to update its settings using a function that is passed to the `NewT` and `UpdateUserSuppliedResourceManagerOverrides` functions of the ` harness.NewT` and ` harness.NewNode` classes.

The `func(t *testing.T)` parameter is a hint for the testing framework, suggesting that the test function should be independent of any specific testing frameworks.

The first test case, `func(t *testing.T) { ...func(t *testing.T) { ...}`, appears to be the main test case for the IPFS daemon. It tests both the `system streams inbound` and `daemon` configurations.

The `func(t *testing.T)` parameter is a hint for the testing framework, suggesting that the test function should be independent of any specific testing frameworks.

The `t.Parallel()` annotation is used to indicate that the test functions should be run independently of each other.

The `assert.Equal()` function is used to compare the exit code of the `RunIPFS()` method with the expected exit code. The assert is wrapped with a anonymous function, `assert.`, which allows the function to have its own type and避免 creating a new instance of the anonymous function in each test.

The `t.Run()` annotation is used to indicate the test function to run.


```go
func TestRcmgr(t *testing.T) {
	t.Parallel()

	t.Run("Resource manager disabled", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Swarm.ResourceMgr.Enabled = config.False
		})

		node.StartDaemon()

		t.Run("swarm resources should fail", func(t *testing.T) {
			res := node.RunIPFS("swarm", "resources")
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "missing ResourceMgr")
		})
	})

	t.Run("Node with resource manager disabled", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Swarm.ResourceMgr.Enabled = config.False
		})
		node.StartDaemon()

		t.Run("swarm resources should fail", func(t *testing.T) {
			res := node.RunIPFS("swarm", "resources")
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "missing ResourceMgr")
		})
	})

	t.Run("Very high connmgr highwater", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(1000)
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		require.Equal(t, 0, res.ExitCode())
		limits := unmarshalLimits(t, res.Stdout.Bytes())

		rl := limits.System.ToResourceLimits()
		s := rl.Build(rcmgr.BaseLimit{})
		assert.GreaterOrEqual(t, s.ConnsInbound, 2000)
		assert.GreaterOrEqual(t, s.StreamsInbound, 2000)
	})

	t.Run("default configuration", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateConfig(func(cfg *config.Config) {
			cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(1000)
		})
		node.StartDaemon()

		t.Run("conns and streams are above 800 for default connmgr settings", func(t *testing.T) {
			t.Parallel()
			res := node.RunIPFS("swarm", "resources", "--enc=json")
			require.Equal(t, 0, res.ExitCode())
			limits := unmarshalLimits(t, res.Stdout.Bytes())

			if limits.System.ConnsInbound > rcmgr.DefaultLimit {
				assert.GreaterOrEqual(t, limits.System.ConnsInbound, 800)
			}
			if limits.System.StreamsInbound > rcmgr.DefaultLimit {
				assert.GreaterOrEqual(t, limits.System.StreamsInbound, 800)
			}
		})

		t.Run("limits should succeed", func(t *testing.T) {
			t.Parallel()
			res := node.RunIPFS("swarm", "resources", "--enc=json")
			assert.Equal(t, 0, res.ExitCode())

			limits := rcmgr.PartialLimitConfig{}
			err := json.Unmarshal(res.Stdout.Bytes(), &limits)
			require.NoError(t, err)

			assert.NotEqual(t, limits.Transient.Memory, rcmgr.BlockAllLimit64)
			assert.NotEqual(t, limits.System.Memory, rcmgr.BlockAllLimit64)
			assert.NotEqual(t, limits.System.FD, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.Conns, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.ConnsInbound, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.ConnsOutbound, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.Streams, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.StreamsInbound, rcmgr.BlockAllLimit)
			assert.NotEqual(t, limits.System.StreamsOutbound, rcmgr.BlockAllLimit)
		})

		t.Run("swarm stats works", func(t *testing.T) {
			t.Parallel()
			res := node.RunIPFS("swarm", "resources", "--enc=json")
			require.Equal(t, 0, res.ExitCode())

			limits := unmarshalLimits(t, res.Stdout.Bytes())

			// every scope has the same fields, so we only inspect system
			assert.Zero(t, limits.System.MemoryUsage)
			assert.Zero(t, limits.System.FDUsage)
			assert.Zero(t, limits.System.ConnsInboundUsage)
			assert.Zero(t, limits.System.ConnsOutboundUsage)
			assert.Zero(t, limits.System.StreamsInboundUsage)
			assert.Zero(t, limits.System.StreamsOutboundUsage)
			assert.Zero(t, limits.Transient.MemoryUsage)
		})
	})

	t.Run("smoke test unlimited System inbounds", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			overrides.System.StreamsInbound = rcmgr.Unlimited
			overrides.System.ConnsInbound = rcmgr.Unlimited
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		limits := unmarshalLimits(t, res.Stdout.Bytes())

		assert.Equal(t, rcmgr.Unlimited, limits.System.ConnsInbound)
		assert.Equal(t, rcmgr.Unlimited, limits.System.StreamsInbound)
	})

	t.Run("smoke test transient scope", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			overrides.Transient.Memory = 88888
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		limits := unmarshalLimits(t, res.Stdout.Bytes())
		assert.Equal(t, rcmgr.LimitVal64(88888), limits.Transient.Memory)
	})

	t.Run("smoke test service scope", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			overrides.Service = map[string]rcmgr.ResourceLimits{"foo": {Memory: 77777}}
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		limits := unmarshalLimits(t, res.Stdout.Bytes())
		assert.Equal(t, rcmgr.LimitVal64(77777), limits.Services["foo"].Memory)
	})

	t.Run("smoke test protocol scope", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init()
		node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			overrides.Protocol = map[protocol.ID]rcmgr.ResourceLimits{"foo": {Memory: 66666}}
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		limits := unmarshalLimits(t, res.Stdout.Bytes())
		assert.Equal(t, rcmgr.LimitVal64(66666), limits.Protocols["foo"].Memory)
	})

	t.Run("smoke test peer scope", func(t *testing.T) {
		t.Parallel()
		validPeerID, err := peer.Decode("QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN")
		assert.NoError(t, err)
		node := harness.NewT(t).NewNode().Init()
		node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			overrides.Peer = map[peer.ID]rcmgr.ResourceLimits{validPeerID: {Memory: 55555}}
		})
		node.StartDaemon()

		res := node.RunIPFS("swarm", "resources", "--enc=json")
		limits := unmarshalLimits(t, res.Stdout.Bytes())
		assert.Equal(t, rcmgr.LimitVal64(55555), limits.Peers[validPeerID].Memory)
	})

	t.Run("blocking and allowlists", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(3).Init()
		node0, node1, node2 := nodes[0], nodes[1], nodes[2]
		peerID1, peerID2 := node1.PeerID().String(), node2.PeerID().String()

		node0.UpdateConfig(func(cfg *config.Config) {
			cfg.Swarm.ResourceMgr.Enabled = config.True
			cfg.Swarm.ResourceMgr.Allowlist = []string{"/ip4/0.0.0.0/ipcidr/0/p2p/" + peerID2}
		})
		node0.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
			*overrides = rcmgr.PartialLimitConfig{
				System: rcmgr.ResourceLimits{
					Conns:         rcmgr.BlockAllLimit,
					ConnsInbound:  rcmgr.BlockAllLimit,
					ConnsOutbound: rcmgr.BlockAllLimit,
				},
			}
		})

		nodes.StartDaemons()

		t.Run("node 0 should fail to connect to and ping node 1", func(t *testing.T) {
			t.Parallel()
			res := node0.Runner.Run(harness.RunRequest{
				Path: node0.IPFSBin,
				Args: []string{"swarm", "connect", node1.SwarmAddrsWithPeerIDs()[0].String()},
			})
			assert.Equal(t, 1, res.ExitCode())
			testutils.AssertStringContainsOneOf(t, res.Stderr.String(),
				"failed to find any peer in table",
				"resource limit exceeded",
			)

			res = node0.RunIPFS("ping", "-n2", peerID1)
			assert.Equal(t, 1, res.ExitCode())
			assert.Contains(t, res.Stderr.String(), "Error: ping failed")
		})

		t.Run("node 0 should connect to and ping node 2 since it is allowlisted", func(t *testing.T) {
			t.Parallel()
			res := node0.Runner.Run(harness.RunRequest{
				Path: node0.IPFSBin,
				Args: []string{"swarm", "connect", node2.SwarmAddrsWithPeerIDs()[0].String()},
			})
			assert.Equal(t, 0, res.ExitCode())

			res = node0.RunIPFS("ping", "-n2", peerID2)
			assert.Equal(t, 0, res.ExitCode())
		})
	})

	t.Run("daemon should refuse to start if connmgr.highwater < resources inbound", func(t *testing.T) {
		t.Run("system conns", func(t *testing.T) {
			t.Parallel()
			node := harness.NewT(t).NewNode().Init()
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(128)
				cfg.Swarm.ConnMgr.LowWater = config.NewOptionalInteger(64)
			})
			node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
				*overrides = rcmgr.PartialLimitConfig{
					System: rcmgr.ResourceLimits{Conns: 128},
				}
			})

			res := node.RunIPFS("daemon")
			assert.Equal(t, 1, res.ExitCode())
		})
		t.Run("system conns inbound", func(t *testing.T) {
			t.Parallel()
			node := harness.NewT(t).NewNode().Init()
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(128)
				cfg.Swarm.ConnMgr.LowWater = config.NewOptionalInteger(64)
			})
			node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
				*overrides = rcmgr.PartialLimitConfig{
					System: rcmgr.ResourceLimits{ConnsInbound: 128},
				}
			})

			res := node.RunIPFS("daemon")
			assert.Equal(t, 1, res.ExitCode())
		})
		t.Run("system streams", func(t *testing.T) {
			t.Parallel()
			node := harness.NewT(t).NewNode().Init()
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(128)
				cfg.Swarm.ConnMgr.LowWater = config.NewOptionalInteger(64)
			})
			node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
				*overrides = rcmgr.PartialLimitConfig{
					System: rcmgr.ResourceLimits{Streams: 128},
				}
			})

			res := node.RunIPFS("daemon")
			assert.Equal(t, 1, res.ExitCode())
		})
		t.Run("system streams inbound", func(t *testing.T) {
			t.Parallel()
			node := harness.NewT(t).NewNode().Init()
			node.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(128)
				cfg.Swarm.ConnMgr.LowWater = config.NewOptionalInteger(64)
			})
			node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
				*overrides = rcmgr.PartialLimitConfig{
					System: rcmgr.ResourceLimits{StreamsInbound: 128},
				}
			})

			res := node.RunIPFS("daemon")
			assert.Equal(t, 1, res.ExitCode())
		})
	})
}

```

该函数`unmarshalLimits`接受两个参数：`t`是一个用于测试的`testing.T`对象，`b`是一个字节切片，代表一个JSON数据结构。

函数的作用是解码一个JSON数据结构为`libp2p.LimitsConfigAndUsage`类型的结构，并返回该结构。具体实现包括以下几个步骤：

1. 设置`limits`变量为`libp2p.LimitsConfigAndUsage`类型的实例。
2. 尝试将字节切片`b`中的JSON数据结构解码为`libp2p.LimitsConfigAndUsage`类型的结构。
3. 如果解码成功，则返回`limits`。
4. 输出错误信息。


```go
func unmarshalLimits(t *testing.T, b []byte) *libp2p.LimitsConfigAndUsage {
	limits := &libp2p.LimitsConfigAndUsage{}
	err := json.Unmarshal(b, limits)
	require.NoError(t, err)
	return limits
}

```

# `test/cli/routing_dht_test.go`

This is a Go test that tests the "routing" commands for the IPFS-based network node "harness". The "routing" commands are used to interact with the IPFS network by finding providers, finding peers, and putting data.

The "nodes" struct contains the current node information. The "harness.NewT" function creates a new "harness" instance and returns a testable node object. The "testutils.CIDEmptyDir" function creates a fake directory for the tests.

The "assertions" section tests the output of the "routing" commands. When an offline "findprovs" or "findpeer" command is called, the test checks that the exit code is 1 and that the error message contains the string "this command must be run in online mode". When an "uploading" (i.e., "put") command is called, the test checks that the exit code is 1 and that the error message contains the string "this action must be run in online mode".

The "run" section of the "t.Run" method calls the "Run" method on the "harness" instance to run the tests. This is followed by a series of assertions that test the output of the "routing" commands.

In summary, this test suite ensures that the "routing" commands can only be run when the harness is online and that the "put" command can only be run when the harness is online.


```go
package cli

import (
	"fmt"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func testRoutingDHT(t *testing.T, enablePubsub bool) {
	t.Run(fmt.Sprintf("enablePubSub=%v", enablePubsub), func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(5).Init()
		nodes.ForEachPar(func(node *harness.Node) {
			node.IPFS("config", "Routing.Type", "dht")
		})

		var daemonArgs []string
		if enablePubsub {
			daemonArgs = []string{
				"--enable-pubsub-experiment",
				"--enable-namesys-pubsub",
			}
		}

		nodes.StartDaemons(daemonArgs...).Connect()

		t.Run("ipfs routing findpeer", func(t *testing.T) {
			t.Parallel()
			res := nodes[1].RunIPFS("routing", "findpeer", nodes[0].PeerID().String())
			assert.Equal(t, 0, res.ExitCode())

			swarmAddr := nodes[0].SwarmAddrsWithoutPeerIDs()[0]
			require.Equal(t, swarmAddr.String(), res.Stdout.Trimmed())
		})

		t.Run("ipfs routing get <key>", func(t *testing.T) {
			t.Parallel()
			hash := nodes[2].IPFSAddStr("hello world")
			nodes[2].IPFS("name", "publish", "/ipfs/"+hash)

			res := nodes[1].IPFS("routing", "get", "/ipns/"+nodes[2].PeerID().String())
			assert.Contains(t, res.Stdout.String(), "/ipfs/"+hash)

			t.Run("put round trips (#3124)", func(t *testing.T) {
				t.Parallel()
				nodes[0].WriteBytes("get_result", res.Stdout.Bytes())
				res := nodes[0].IPFS("routing", "put", "/ipns/"+nodes[2].PeerID().String(), "get_result")
				assert.Greater(t, len(res.Stdout.Lines()), 0, "should put to at least one node")
			})

			t.Run("put with bad keys fails (issue #5113, #4611)", func(t *testing.T) {
				t.Parallel()
				keys := []string{"foo", "/pk/foo", "/ipns/foo"}
				for _, key := range keys {
					key := key
					t.Run(key, func(t *testing.T) {
						t.Parallel()
						res := nodes[0].RunIPFS("routing", "put", key)
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
						res := nodes[0].RunIPFS("routing", "get", key)
						assert.Equal(t, 1, res.ExitCode())
						assert.Contains(t, res.Stderr.String(), "invalid")
						assert.Empty(t, res.Stdout.String())
					})
				}
			})
		})

		t.Run("ipfs routing findprovs", func(t *testing.T) {
			t.Parallel()
			hash := nodes[3].IPFSAddStr("some stuff")
			res := nodes[4].IPFS("routing", "findprovs", hash)
			assert.Equal(t, nodes[3].PeerID().String(), res.Stdout.Trimmed())
		})

		t.Run("routing commands fail when offline", func(t *testing.T) {
			t.Parallel()
			node := harness.NewT(t).NewNode().Init()

			// these cannot be run in parallel due to repo locking
			// this seems like a bug, we should be able to run these without locking the repo

			t.Run("routing findprovs", func(t *testing.T) {
				res := node.RunIPFS("routing", "findprovs", testutils.CIDEmptyDir)
				assert.Equal(t, 1, res.ExitCode())
				assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
			})

			t.Run("routing findpeer", func(t *testing.T) {
				res := node.RunIPFS("routing", "findpeer", testutils.CIDEmptyDir)
				assert.Equal(t, 1, res.ExitCode())
				assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
			})

			t.Run("routing put", func(t *testing.T) {
				node.WriteBytes("foo", []byte("foo"))
				res := node.RunIPFS("routing", "put", "/ipns/"+node.PeerID().String(), "foo")
				assert.Equal(t, 1, res.ExitCode())
				assert.Contains(t, res.Stderr.String(), "this action must be run in online mode")
			})
		})
	})
}

```

这段代码是一个名为 `testSelfFindDHT` 的函数，它使用了 `testing.T` 标头，表明这是一个测试函数。

函数的作用是测试一个名为 `"ipfs routing findpeer fails for self"` 的断言，具体内容如下：

1. 初始化一些 ` harness.Node` 类型的 `test.Run` 实例，并使用 `NewNodes` 方法从 `Harness` 类中获取一些节点，同时使用 `ForEachPar` 函数对每个节点执行一次 `IPFS("config", "Routing.Type", "dht")` 命令，将路由类型设置为 `"dht"`。

2. 调用 `nodes.StartDaemons()` 方法启动所有节点作为开发人员服务器。

3. 使用 `nodes[0].RunIPFS` 方法执行 `"dht"` 和 `"findpeer"` 命令，其中 `nodes[0]` 是节点 `Harness` 实例中的第一个节点，`nodes[0].PeerID()` 方法返回节点ID，作为参数传递给 `RunIPFS` 函数。

4. 使用 `assert.Equal` 函数检查 `res` 变量的值是否为1,`res` 的 `ExitCode()` 方法返回 `1`。如果返回值不是1，说明运行 `"dht"` 和 `"findpeer"` 命令的 `Harness` 实例的退出代码不是1，也就是测试失败。


```go
func testSelfFindDHT(t *testing.T) {
	t.Run("ipfs routing findpeer fails for self", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(1).Init()
		nodes.ForEachPar(func(node *harness.Node) {
			node.IPFS("config", "Routing.Type", "dht")
		})

		nodes.StartDaemons()

		res := nodes[0].RunIPFS("dht", "findpeer", nodes[0].PeerID().String())
		assert.Equal(t, 1, res.ExitCode())
	})
}

```

该代码是一个名为 "TestRoutingDHT" 的函数，它使用 testing 包的一个测试函数指针 "t"。

函数的主要作用是测试一个名为 "func TestRoutingDHT(t *testing.T)" 的函数。在这个函数中，首先使用 "testRoutingDHT(t, false)" 函数对路由器进行测试，然后使用 "testRoutingDHT(t, true)" 函数对路由器进行测试，最后使用 "testSelfFindDHT(t)" 函数测试自我查找路由器。

具体来说，函数中执行以下操作：

1. 调用 "testRoutingDHT(t, false)" 函数，这个函数的作用是测试路由器在当前设置下是否能够正常工作，主要是测试路由器的基本功能是否正常运行。

2. 调用 "testRoutingDHT(t, true)" 函数，这个函数的作用是测试路由器在当前设置下一个共主模式是否正常工作。

3. 调用 "testSelfFindDHT(t)" 函数，这个函数的作用是测试路由器是否能够通过单点故障恢复路由。

通过执行这三个测试函数，可以检验路由器的运行状况，并验证其是否能够正常工作。


```go
func TestRoutingDHT(t *testing.T) {
	testRoutingDHT(t, false)
	testRoutingDHT(t, true)
	testSelfFindDHT(t)
}

```

# `test/cli/stats_test.go`

这段代码是一个 Go 语言编写的命令行工具 "kubo" 的测试框架。它旨在测试 IPFS-based Kubernetes ("kube") harbor 的 stats 功能，具体来说，测试 "dht" 以为全球多地服务器提供 DNS 解析的 DNS 服务。

该代码主要包含以下步骤：

1. 导入 "testing" 和 "stretchr/testify/assert" 包。
2. 通过 "github.com/stretchr/testify/assert" 包中的 "assert" 函数对测试的输出进行断言。
3. 通过 "github.com/ipfs/kubo/test/cli/harness" 包中的 "NewT" 和 "NewNodes" 函数创建一个新的测试 harness。
4. 通过 "harness.NewT" 函数创建一个新的测试环境，并通过 "harness.StartDaemons" 函数启动 daemons。
5. 通过 "harness.Connect" 函数连接到远程服务器，并使用 "kubo" 的 "test" 命令发起 "dht" 请求。
6. 通过 "assert.NoError" 和 "assert.Equal" 函数检查 "dht" 请求是否成功，并检查 stderr 和 stdout 是否正确。

该代码的主要目的是验证 "dht" 命令行工具是否能够成功提供全球多地服务器。


```go
package cli

import (
	"testing"

	"github.com/stretchr/testify/assert"

	"github.com/ipfs/kubo/test/cli/harness"
)

func TestStats(t *testing.T) {
	t.Parallel()

	t.Run("stats dht", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()
		node1 := nodes[0]

		res := node1.IPFS("stats", "dht")
		assert.NoError(t, res.Err)
		assert.Equal(t, 0, len(res.Stderr.Lines()))
		assert.NotEqual(t, 0, len(res.Stdout.Lines()))
	})
}

```

# `test/cli/swarm_test.go`

This appears to be a unit test for the IPFS (Inter-Peer File System) `swarm` and `identify` commands. The `swarm` command is used to form a swarm of nodes and form a chain reaction, while the `identify` command is used to identify and connect to these nodes.

The `t.Run` function defines a test for the `swarm` and `identify` commands, and provides a failure scenario in which the test will fail. The failure scenario includes several assertions, such as checking that the output of the `swarm` command is a valid `identify` request, and checking that the output of the `identify` command contains the expected data.

The `assertions` section of the test is where the actual test code would go, including checks for the expected data in the output of the `swarm` and `identify` commands. The specific checks may vary depending on the requirements of the unit test.


```go
package cli

import (
	"encoding/json"
	"fmt"
	"testing"

	"github.com/ipfs/kubo/test/cli/harness"

	"github.com/stretchr/testify/assert"
)

// TODO: Migrate the rest of the sharness swarm test.
func TestSwarm(t *testing.T) {
	type identifyType struct {
		ID           string
		PublicKey    string
		Addresses    []string
		AgentVersion string
		Protocols    []string
	}
	type peer struct {
		Identify identifyType
	}
	type expectedOutputType struct {
		Peers []peer
	}

	t.Parallel()

	t.Run("ipfs swarm peers returns empty peers when a node is not connected to any peers", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")
		var output expectedOutputType
		err := json.Unmarshal(res.Stdout.Bytes(), &output)
		assert.NoError(t, err)
		assert.Equal(t, 0, len(output.Peers))
	})
	t.Run("ipfs swarm peers with flag identify outputs expected identify information about connected peers", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		otherNode := harness.NewT(t).NewNode().Init().StartDaemon()
		node.Connect(otherNode)

		res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")
		var output expectedOutputType
		err := json.Unmarshal(res.Stdout.Bytes(), &output)
		assert.NoError(t, err)
		actualID := output.Peers[0].Identify.ID
		actualPublicKey := output.Peers[0].Identify.PublicKey
		actualAgentVersion := output.Peers[0].Identify.AgentVersion
		actualAdresses := output.Peers[0].Identify.Addresses
		actualProtocols := output.Peers[0].Identify.Protocols

		expectedID := otherNode.PeerID().String()
		expectedAddresses := []string{fmt.Sprintf("%s/p2p/%s", otherNode.SwarmAddrs()[0], actualID)}

		assert.Equal(t, actualID, expectedID)
		assert.NotNil(t, actualPublicKey)
		assert.NotNil(t, actualAgentVersion)
		assert.Len(t, actualAdresses, 1)
		assert.Equal(t, expectedAddresses[0], actualAdresses[0])
		assert.Greater(t, len(actualProtocols), 0)
	})

	t.Run("ipfs swarm peers with flag identify outputs Identify field with data that matches calling ipfs id on a peer", func(t *testing.T) {
		t.Parallel()
		node := harness.NewT(t).NewNode().Init().StartDaemon()
		otherNode := harness.NewT(t).NewNode().Init().StartDaemon()
		node.Connect(otherNode)

		otherNodeIDResponse := otherNode.RunIPFS("id", "--enc=json")
		var otherNodeIDOutput identifyType
		err := json.Unmarshal(otherNodeIDResponse.Stdout.Bytes(), &otherNodeIDOutput)
		assert.NoError(t, err)
		res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")

		var output expectedOutputType
		err = json.Unmarshal(res.Stdout.Bytes(), &output)
		assert.NoError(t, err)
		outputIdentify := output.Peers[0].Identify

		assert.Equal(t, outputIdentify.ID, otherNodeIDOutput.ID)
		assert.Equal(t, outputIdentify.PublicKey, otherNodeIDOutput.PublicKey)
		assert.Equal(t, outputIdentify.AgentVersion, otherNodeIDOutput.AgentVersion)
		assert.ElementsMatch(t, outputIdentify.Addresses, otherNodeIDOutput.Addresses)
		assert.ElementsMatch(t, outputIdentify.Protocols, otherNodeIDOutput.Protocols)
	})
}

```

# `test/cli/tracing_test.go`

这段代码是一个名为“cli”的包的导入语句，其中定义了一个名为“testutils”的包，它似乎包含了一些测试相关的函数和变量。

这里似乎是通过导入“github.com/ipfs/kubo/test/cli/harness”包来测试的。这个包似乎是一个名为“Kubernetes”的库的子命令行工具“kubo”的测试框架。

此外，导入了一些标准库中的函数和变量，比如“fmt”输出格式化、“os”操作系统相关函数、“path/filepath”文件路径相关的函数、“time”时间相关的函数以及一些与字符串比较相关的函数。

最后，通过导入“testutils”包中的函数来对测试进行断言，似乎是测试“kubo”命令行工具的功能是否符合预期，具体测试代码中包括了一些测试用例，比如对测试工具输出的日志进行断言，通过运行“kubo”命令行工具来测试这些断言是否能够正确输出。


```go
package cli

import (
	"fmt"
	"os"
	"os/exec"
	"path/filepath"
	"strings"
	"testing"
	"time"

	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

```

这段代码定义了一个 `OtelCollectorConfigYAML` 对象，用于配置 `OtelTraces` 服务的管道。具体来说，它包括以下几个部分：

1. `receivers`：定义了一个 `otlp` 接收者，表示使用 `otlp` 协议接收 traces。
2. `processors`：定义了一个 `batch` 处理器，用于对 traces 进行批处理。
3. `exporters`：定义了一个 `file` 出口者，用于将 traces 输出到 `/traces/traces.json` 文件中。
4. `service`：定义了一个 `OtelTraces` 服务，包含上述 receiver、processor 和 exporter。
5. `pipelines`：定义了服务中所有 pipeline，其中 `otlp` 接收者作为必经之路。


```go
var otelCollectorConfigYAML = `
receivers:
  otlp:
    protocols:
      grpc:

processors:
  batch:

exporters:
  file:
    path: /traces/traces.json

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [file]
```

This is a Go program that runs an Iterator Daemon that collects tracing data from the Iterator in an IPFS (InterPlanetary File System) cluster, and exports it to an OTLP (On-Demand Tracing Log Exporter) service using the OpenTelemetry Collector. The program uses the "docker" binary to run the Iterator Daemon as a separate process, and uses a configuration file (docker-config.yaml) to specify the configuration options for the Iterator.

The program starts a daemon that runs the Iterator Daemon and listens for new requests to the OTLP service. When a request is received, the program reads the tracing data from the Iterator, transforms it using the OpenTelemetry Collector, and writes it to the OTLP service. The program also uses the OpenTelemetry Collector's experimental Collector to run the Iterator as a separate process that is detached from the main program, and that runs using the OpenTelemetry Collector's experimental Collector configuration.

The program uses the "docker-config.yaml" file to specify the configuration options for the Iterator. The configuration file specifies the path to the IPFS cluster node that the Iterator should access, and the options for the Iterator, such as the file that the Iterator should watch for changes.

The program uses the "docker" command to start the Iterator Daemon as a separate process, and uses the "stop" command to stop the Iterator when it is no longer needed.


```go
`

func TestTracing(t *testing.T) {
	testutils.RequiresDocker(t)
	t.Parallel()
	node := harness.NewT(t).NewNode().Init()

	node.WriteBytes("collector-config.yaml", []byte(otelCollectorConfigYAML))

	// touch traces.json and give it 777 perms in case Docker runs as a different user
	node.WriteBytes("traces.json", nil)
	err := os.Chmod(filepath.Join(node.Dir, "traces.json"), 0o777)
	require.NoError(t, err)

	dockerBin, err := exec.LookPath("docker")
	require.NoError(t, err)
	node.Runner.MustRun(harness.RunRequest{
		Path: dockerBin,
		Args: []string{
			"run",
			"--rm",
			"--detach",
			"--volume", fmt.Sprintf("%s:/config.yaml", filepath.Join(node.Dir, "collector-config.yaml")),
			"--volume", fmt.Sprintf("%s:/traces", node.Dir),
			"--net", "host",
			"--name", "ipfs-test-otel-collector",
			"otel/opentelemetry-collector-contrib:0.52.0",
			"--config", "/config.yaml",
		},
	})

	t.Cleanup(func() {
		node.Runner.MustRun(harness.RunRequest{
			Path: dockerBin,
			Args: []string{"stop", "ipfs-test-otel-collector"},
		})
	})

	node.Runner.Env["OTEL_TRACES_EXPORTER"] = "otlp"
	node.Runner.Env["OTEL_EXPORTER_OTLP_PROTOCOL"] = "grpc"
	node.Runner.Env["OTEL_EXPORTER_OTLP_ENDPOINT"] = "http://localhost:4317"
	node.StartDaemon()

	assert.Eventually(t,
		func() bool {
			b, err := os.ReadFile(filepath.Join(node.Dir, "traces.json"))
			require.NoError(t, err)
			return strings.Contains(string(b), "go-ipfs")
		},
		5*time.Minute,
		10*time.Millisecond,
	)
}

```

# `test/cli/transports_test.go`

这段代码是一个 Go 语言编写的命令行工具 package，名为 "cli"，旨在为测试 "ipfs-kubo" 项目的不同功能提供简单的命令行工具。具体来说，这段代码包括以下几个主要部分：

1. 导入所需的 "fmt"、"os" 和 "path/filepath" 包，这些包分别提供格式化输出、操作系统功能和文件路径处理等功能。

2. 导入来自 "ipfs-kubo/config" 和 "ipfs-kubo/test/cli/harness" 包的 "config" 和 "harness" 函数，这些函数分别设置和获取 Ipfs Kubernetes 服务的配置和上下文。

3. 导入来自 "ipfs-kubo/test/cli/testutils" 包的 "is度量衡" 和 "formatLinuxValue" 函数，这些函数分别用于度量衡支持和将数据格式化为常用的字符串格式。

4. 导出 "test-cli.go" 和 "test-cli.go.template" 两个文件，分别为测试 "ipfs-kubo" 项目不同功能的命令行工具和测试工具的接口定义。

5. 在 "test-cli.go" 中定义了一个名为 "SetupFixture" 的函数，该函数用于设置测试设施和执行测试 "ipfs-kubo" 项目的不同功能。

6. 在 "test-cli.go.template" 中定义了一个名为 "TestCli" 的函数，该函数为测试 "ipfs-kubo" 项目不同功能提供命令行接口，并使用 "SetupFixture" 设置测试设施。

7. 在 "test-cli.go" 中进行了一些测试，包括：

 - 测试 "ipfs-kubo/test/cli/is-service-up-with-options" 功能，验证服务是否能够在不同的选项下正常运行。

 - 测试 "ipfs-kubo/test/cli/is-integer-output" 功能，验证输出为整数是否正确。

 - 测试 "ipfs-kubo/test/cli/is-float-output" 功能，验证输出为浮点数是否正确。

 - 测试 "ipfs-kubo/test/cli/is-text-output" 功能，验证输出为文本是否正确。

 - 测试 "ipfs-kubo/test/cli/is-image-output" 功能，验证输出为图像是否正确。

8. 最后，通过运行 "go test" 命令来运行所有的测试，并根据测试结果输出相应的错误信息。


```go
package cli

import (
	"fmt"
	"os"
	"path/filepath"
	"testing"

	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/test/cli/harness"
	"github.com/ipfs/kubo/test/cli/testutils"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

```

This appears to be a Go program that performs a unit test to verify that the Kubernetes nodes it creates can properly configure a Quic network.

The program first configures the nodes to have a specific IP address (127.0.0.1) and a port that the nodes can use for announcements (0). It then configures the Quic node to use a specific transport (QUIC) and an optional WebSocket transport.

The program then disables any built-in routing, then starts the nodes and connects them to the test network. Finally, it runs the tests to verify that the nodes can properly configure the Quic network and the WebSocket transport.


```go
func TestTransports(t *testing.T) {
	disableRouting := func(nodes harness.Nodes) {
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Routing.Type = config.NewOptionalString("none")
				cfg.Bootstrap = nil
			})
		})
	}
	checkSingleFile := func(nodes harness.Nodes) {
		s := testutils.RandomStr(100)
		hash := nodes[0].IPFSAddStr(s)
		nodes.ForEachPar(func(n *harness.Node) {
			val := n.IPFS("cat", hash).Stdout.String()
			assert.Equal(t, s, val)
		})
	}
	checkRandomDir := func(nodes harness.Nodes) {
		randDir := filepath.Join(nodes[0].Dir, "foobar")
		require.NoError(t, os.Mkdir(randDir, 0o777))
		rf := testutils.NewRandFiles()
		rf.FanoutDirs = 3
		rf.FanoutFiles = 6
		require.NoError(t, rf.WriteRandomFiles(randDir, 4))

		hash := nodes[1].IPFS("add", "-r", "-Q", randDir).Stdout.Trimmed()
		nodes.ForEachPar(func(n *harness.Node) {
			res := n.RunIPFS("refs", "-r", hash)
			assert.Equal(t, 0, res.ExitCode())
		})
	}

	runTests := func(nodes harness.Nodes) {
		checkSingleFile(nodes)
		checkRandomDir(nodes)
	}

	tcpNodes := func(t *testing.T) harness.Nodes {
		nodes := harness.NewT(t).NewNodes(2).Init()
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/tcp/0"}
				cfg.Swarm.Transports.Network.QUIC = config.False
				cfg.Swarm.Transports.Network.Relay = config.False
				cfg.Swarm.Transports.Network.WebTransport = config.False
				cfg.Swarm.Transports.Network.Websocket = config.False
			})
		})
		disableRouting(nodes)
		return nodes
	}

	t.Run("tcp", func(t *testing.T) {
		t.Parallel()
		nodes := tcpNodes(t).StartDaemons().Connect()
		runTests(nodes)
	})

	t.Run("tcp with mplex", func(t *testing.T) {
		// FIXME(#10069): we don't want this to exists anymore
		t.Parallel()
		nodes := tcpNodes(t)
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.Transports.Multiplexers.Yamux = config.Disabled
				cfg.Swarm.Transports.Multiplexers.Mplex = 200
			})
		})
		nodes.StartDaemons().Connect()
		runTests(nodes)
	})

	t.Run("tcp with NOISE", func(t *testing.T) {
		t.Parallel()
		nodes := tcpNodes(t)
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Swarm.Transports.Security.TLS = config.Disabled
			})
		})
		nodes.StartDaemons().Connect()
		runTests(nodes)
	})

	t.Run("QUIC", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(5).Init()
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/udp/0/quic-v1"}
				cfg.Swarm.Transports.Network.QUIC = config.True
				cfg.Swarm.Transports.Network.TCP = config.False
			})
		})
		disableRouting(nodes)
		nodes.StartDaemons().Connect()
		runTests(nodes)
	})

	t.Run("QUIC", func(t *testing.T) {
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(5).Init()
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/udp/0/quic-v1/webtransport"}
				cfg.Swarm.Transports.Network.QUIC = config.True
				cfg.Swarm.Transports.Network.WebTransport = config.True
			})
		})
		disableRouting(nodes)
		nodes.StartDaemons().Connect()
		runTests(nodes)
	})

	t.Run("QUIC connects with non-dialable transports", func(t *testing.T) {
		// This test targets specific Kubo internals which may change later. This checks
		// if we can announce an address we do not listen on, and then are able to connect
		// via a different address that is available.
		t.Parallel()
		nodes := harness.NewT(t).NewNodes(5).Init()
		nodes.ForEachPar(func(n *harness.Node) {
			n.UpdateConfig(func(cfg *config.Config) {
				// We need a specific port to announce so we first generate a random port.
				// We can't use 0 here to automatically assign an available port because
				// that would only work with Swarm, but not for the announcing.
				port := harness.NewRandPort()
				quicAddr := fmt.Sprintf("/ip4/127.0.0.1/udp/%d/quic-v1", port)
				cfg.Addresses.Swarm = []string{quicAddr}
				cfg.Addresses.Announce = []string{quicAddr, quicAddr + "/webtransport"}
			})
		})
		disableRouting(nodes)
		nodes.StartDaemons().Connect()
		runTests(nodes)
	})
}

```

# Dataset Description / Sources

TestGatewayHAMTDirectory.car generated with:

```gobash
ipfs version
# ipfs version 0.19.0

export HAMT_DIR=bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm
export IPFS_PATH=$(mktemp -d)

# Init and start daemon, ensure we have an empty repository.
ipfs init --empty-repo
ipfs daemon &> /dev/null &
export IPFS_PID=$!

# Retrieve the directory listing, forcing the daemon to download all required DAGs. Kill daemon.
curl -o dir.html http://127.0.0.1:8080/ipfs/$HAMT_DIR/
kill $IPFS_PID

# Get the list with all the downloaded refs and sanity check.
ipfs refs local > required_refs
cat required_refs | wc -l
# 962

# Get the list of all the files CIDs inside the directory and sanity check.
cat dir.html| pup '#content tbody .ipfs-hash attr{href}' | sed 's/\/ipfs\///g;s/\?filename=.*//g' > files_refs
cat files_refs | wc -l
# 10100

# Make and export our fixture.
ipfs files mkdir --cid-version 1 /fixtures
cat required_refs | xargs -I {} ipfs files cp /ipfs/{} /fixtures/{}
cat files_refs | ipfs files write --create /fixtures/files_refs
export FIXTURE_CID=$(ipfs files stat --hash /fixtures/)
echo $FIXTURE_CID
# bafybeig3yoibxe56aolixqa4zk55gp5sug3qgaztkakpndzk2b2ynobd4i
ipfs dag export $FIXTURE_CID > TestGatewayHAMTDirectory.car
```

TestGatewayMultiRange.car generated with:


```gosh
ipfs version
# ipfs version 0.19.0

export FILE_CID=bafybeiae5abzv6j3ucqbzlpnx3pcqbr2otbnpot7d2k5pckmpymin4guau
export IPFS_PATH=$(mktemp -d)

# Init and start daemon, ensure we have an empty repository.
ipfs init --empty-repo
ipfs daemon &> /dev/null &
export IPFS_PID=$!

# Get a specific byte range from the file. 
curl http://127.0.0.1:8080/ipfs/$FILE_CID -i -H "Range: bytes=1276-1279, 29839070-29839080"
kill $IPFS_PID

# Get the list with all the downloaded refs and sanity check.
ipfs refs local > required_refs
cat required_refs | wc -l
# 19

# Make and export our fixture.
ipfs files mkdir --cid-version 1 /fixtures
cat required_refs | xargs -I {} ipfs files cp /ipfs/{} /fixtures/{}
export FIXTURE_CID=$(ipfs files stat --hash /fixtures/)
echo $FIXTURE_CID
# bafybeicgsg3lwyn3yl75lw7sn4zhyj5dxtb7wfxwscpq6yzippetmr2w3y
ipfs dag export $FIXTURE_CID > TestGatewayMultiRange.car
```


# `test/cli/harness/buffer.go`

这段代码定义了一个名为“harness”的包，它使用了“Buffer”类型来提供线程安全的字节缓冲区。

具体来说，这段代码创建了一个名为“Buffer”的类，它包含一个名为“b”的字符串缓冲区，以及一个名为“m”的读写锁。

这个“Buffer”类被用来作为命令行工具“kubo”的一部分，用于将kubo命令的输出结果打印到控制台。具体来说，“kubo”命令会将一个字符串“ harness find <POD_NAMES> -o jsonpath=10000 -w json_string | jq '.+ 0'”的结果打印到控制台，其中“ harness”在这里作为整个命令的参数。

而“Buffer”类的“b”字段则是这个字符串的缓冲区，用于暂存尚未打印的字符串，而“m”则是一个读写锁，用于保证对“b”的读写操作是同步的。这样，即使同时有多个协程在访问“b”中的字符串，也不会发生冲突，从而保证代码的正确性。


```go
package harness

import (
	"strings"
	"sync"

	"github.com/ipfs/kubo/test/cli/testutils"
)

// Buffer is a thread-safe byte buffer.
type Buffer struct {
	b strings.Builder
	m sync.Mutex
}

```

以上代码定义了三个函数，分别是：

1. `func (b *Buffer) Write(p []byte) (n int, err error)` 函数接收一个 `Buffer` 类型的参数 `b` 和一个字节数组 `p`，并返回 `n` 个字节，或者错误信息 `err`。函数使用了 `b.m.Lock()` 和 `b.m.Unlock()` 来进行互斥锁和释放锁操作，确保在同一时刻只有一个 `Write` 操作在进行。函数实现了将字节数组 `p` 中的字节写入到 `Buffer` 类型的 `b` 中，并返回更新后的 `b` 指针。

2. `func (b *Buffer) String() string` 函数接收一个 `Buffer` 类型的参数 `b`，并返回其 `String` 方法返回的字符串。函数使用了 `b.m.Lock()` 和 `b.m.Unlock()` 来进行互斥锁和释放锁操作，确保在同一时刻只有一个 `String` 操作在进行。函数实现了将 `Buffer` 类型的 `b` 中的字节串拼接成一个字符串，并返回该字符串。

3. `func (b *Buffer) Trimmed() string` 函数接收一个 `Buffer` 类型的参数 `b`，并返回其 `Trimmed` 方法返回的字符串。函数使用了 `b.m.Lock()` 和 `b.m.Unlock()` 来进行互斥锁和释放锁操作，确保在同一时刻只有一个 `Trimmed` 操作在进行。函数实现了将 `Buffer` 类型的 `b` 中的字节串截去字符串末尾的换行符并返回，如果字符串长度为0，则返回原字符串。


```go
func (b *Buffer) Write(p []byte) (n int, err error) {
	b.m.Lock()
	defer b.m.Unlock()
	return b.b.Write(p)
}

func (b *Buffer) String() string {
	b.m.Lock()
	defer b.m.Unlock()
	return b.b.String()
}

// Trimmed returns the bytes as a string, but with the trailing newline removed.
// This only removes a single trailing newline, not all whitespace.
func (b *Buffer) Trimmed() string {
	b.m.Lock()
	defer b.m.Unlock()
	s := b.b.String()
	if len(s) == 0 {
		return s
	}
	if s[len(s)-1] == '\n' {
		return s[:len(s)-1]
	}
	return s
}

```

这两段代码都是函数，接收一个名为 "b" 的缓冲区（即一个字节切片），并返回一个字节切片（也就是一个字符串）或一个包含多行文本的切片。

第一段代码实现了一个将缓冲区中的所有元素存储为字符串并返回的功能。具体来说，它先尝试获取缓冲区中的内存句柄（即一个指向内存区域的指针），然后使用一个临时变量 "deferred" 来避免在函数内部修改原始缓冲区。接着，它调用一个名为 "testutils.SplitLines" 的函数，将缓冲区中的字符串分割成多行文本并存储为一个新的字节切片。最后，它将分割得到的字节切片返回。

第二段代码实现了一个将缓冲区中的字符串分割成多行文本并返回的功能。它与第一段代码类似，只是使用了不同的函数名称。具体来说，它与第一段代码的实现过程类似，只是将函数名称替换为了 "testutils.SplitLines"。


```go
func (b *Buffer) Bytes() []byte {
	b.m.Lock()
	defer b.m.Unlock()
	return []byte(b.b.String())
}

func (b *Buffer) Lines() []string {
	b.m.Lock()
	defer b.m.Unlock()
	return testutils.SplitLines(b.b.String())
}

```

# `test/cli/harness/harness.go`

该代码包名为“ harness”，是一个 Go 项目。它导入了多个外部依赖项：
- "errors"：用于处理错误。
- "fmt"：用于格式化字符串。
- "os"：用于操作系统相关操作。
- "path/filepath"：用于文件路径操作。
- "strings"：用于字符串操作。
- "testing"：用于测试的依赖项。
- "time"：用于时间操作。
- "github.com/ipfs/go-log/v2"：从 IPFS-Log 库导入。
- "github.com/ipfs/kubo/test/cli/testutils"：从 IPFS-Kubernetes 的命令行工具导入。
- "github.com/multiformats/go-multiaddr"：从 Multi-Addr 库导入。

接下来，定义了一个名为 "Harness" 的函数，它的作用是帮助测试。函数内部定义了一系列函数，用于将进测试的值传递给 "github.com/ipfs/go-log/v2" 和 "github.com/ipfs/kubo/test/cli/testutils" 这两个库。


```go
package harness

import (
	"errors"
	"fmt"
	"os"
	"path/filepath"
	"strings"
	"testing"
	"time"

	logging "github.com/ipfs/go-log/v2"
	. "github.com/ipfs/kubo/test/cli/testutils"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/multiformats/go-multiaddr"
)

```

这段代码定义了一个名为Harness的结构体类型，用于表示测试的状态，例如临时目录和IFPS节点。该类型包含一个测试目录(Dir)，一个IPFS二进制文件(IPFSBin)，一个用于记录当前运行的Runner的指针(Runner)，以及一个包含所有测试节点的字符串(NodesRoot)和节点(Nodes)。

另外，该代码还定义了一个名为EnableDebugLogging的函数，该函数使用zaptest.NewLogger(t)来设置 logging.SetLogLevel("testharness", "DEBUG")。如果设置过程中出现错误，则会输出当前堆栈信息。

最后，该代码没有输出任何内容，而是自己定义了一个类型的实例并进行了初始化。


```go
// Harness tracks state for a test, such as temp dirs and IFPS nodes, and cleans them up after the test.
type Harness struct {
	Dir       string
	IPFSBin   string
	Runner    *Runner
	NodesRoot string
	Nodes     Nodes
}

// TODO: use zaptest.NewLogger(t) instead
func EnableDebugLogging() {
	err := logging.SetLogLevel("testharness", "DEBUG")
	if err != nil {
		panic(err)
	}
}

```

该代码定义了两个函数，一个名为`NewT`，另一个名为`New`。这两个函数都是创建一个`Harness`类型的 harness，该harness在测试完成时会清理残留资源。

`NewT`函数接受一个测试套件`t`以及一个或多个选项`options`。函数内部创建一个新的`Harness`实例，设置其运行时`Runner`为`osEnviron()`，然后将其它的选项应用到实例上，最后返回该实例。

`New`函数接受一个或多个选项`options`，然后创建一个新的`Harness`实例。设置`Runner`为`&Runner{Env: osEnviron()}`，然后将其它的选项应用到实例上，最后返回该实例。`Runner`函数设置存储在`Env`字段中的环境变量，这些变量将在运行时被应用程序链接到运行时所需的库。

这两个函数都首先创建一个根目录，然后设置`Harness`实例的`Runner`和`Dir`，接着设置一个`nodes`文件夹，该文件夹将包含一个名为`ipfs.txt`的文件，其中包含一些节点配置信息。在创建`Harness`实例的选项之后，可以调用其中的任何一个函数来实例化一个`Harness`实例，并将一个`Harness`实例返回给调用者。


```go
// NewT constructs a harness that cleans up after the given test is done.
func NewT(t *testing.T, options ...func(h *Harness)) *Harness {
	h := New(options...)
	t.Cleanup(h.Cleanup)
	return h
}

func New(options ...func(h *Harness)) *Harness {
	h := &Harness{Runner: &Runner{Env: osEnviron()}}

	// walk up to find the root dir, from which we can locate the binary
	wd, err := os.Getwd()
	if err != nil {
		panic(err)
	}
	goMod := FindUp("go.mod", wd)
	if goMod == "" {
		panic("unable to find root dir")
	}
	rootDir := filepath.Dir(goMod)
	h.IPFSBin = filepath.Join(rootDir, "cmd", "ipfs", "ipfs")

	// setup working dir
	tmpDir, err := os.MkdirTemp("", "")
	if err != nil {
		log.Panicf("error creating temp dir: %s", err)
	}
	h.Dir = tmpDir
	h.Runner.Dir = h.Dir

	h.NodesRoot = filepath.Join(h.Dir, ".nodes")

	// apply any customizations
	// this should happen after all initialization
	for _, o := range options {
		o(h)
	}

	return h
}

```

这是一个使用 Go 语言编写的函数，它的作用是获取操作系统环境中的所有环境变量，并将它们存储在一个Map对象中。

具体来说，代码首先定义了一个名为 "m" 的Map类型变量，用于存储环境变量键值对。接着，代码使用一个for循环遍历了操作系统环境中的所有环境变量，并将每个环境变量使用strings.Split函数分割成两个部分，分别存储到Map对象的键和值中。这样，就完成了对环境变量的存储。

最后，代码返回了一个名为 "m" 的Map对象，其中包含了所有操作系统环境中的环境变量。


```go
func osEnviron() map[string]string {
	m := map[string]string{}
	for _, entry := range os.Environ() {
		split := strings.Split(entry, "=")
		m[split[0]] = split[1]
	}
	return m
}

func (h *Harness) NewNode() *Node {
	nodeID := len(h.Nodes)
	node := BuildNode(h.IPFSBin, h.NodesRoot, nodeID)
	h.Nodes = append(h.Nodes, node)
	return node
}

```

这两函数的作用如下：

1. func (h *Harness) NewNodes(count int) Nodes {
这是一个名为`NewNodes`的函数，它接收一个整数参数`count`，并返回一个整数类型的切片`Nodes`，用于存储新创建的节点。

函数内部使用一个for循环，从0到`count`遍历，每次创建一个新节点并将其添加到结果切片`Nodes`中。

2. func (h *Harness) WriteToTemp(contents string) string {
这是一个名为`WriteToTemp`的函数，它接收一个字符串参数`contents`，并返回一个字符串，表示写入的临时文件的路径。

函数内部创建一个名为`f`的文件，并使用`WriteString`方法将`contents`字符串写入文件中。如果创建或读取文件时出现错误，函数会打印错误信息并返回`nil`。

函数返回文件读取路径，这样当调用此函数时，您需要根据实际文件路径关闭文件。


```go
func (h *Harness) NewNodes(count int) Nodes {
	var newNodes []*Node
	for i := 0; i < count; i++ {
		newNodes = append(newNodes, h.NewNode())
	}
	return newNodes
}

// WriteToTemp writes the given contents to a guaranteed-unique temp file, returning its path.
func (h *Harness) WriteToTemp(contents string) string {
	f := h.TempFile()
	_, err := f.WriteString(contents)
	if err != nil {
		log.Panicf("writing to temp file: %s", err.Error())
	}
	err = f.Close()
	if err != nil {
		log.Panicf("closing temp file: %s", err.Error())
	}
	return f.Name()
}

```

这两段代码是针对一个 Harness 类型的 struct，其中的 TempFile 和 WriteFile 函数分别实现了创建临时文件和向文件中写入内容的功能。

具体来说，这段代码首先会创建一个新的临时文件，并返回该文件的指针。在 WriteFile 函数中，首先检查传入的文件名是否为绝对路径，如果不是，则会输出错误信息。接着，会将文件内容写入到指定的文件路径中。如果文件路径解析失败或者写入文件时出现错误，则会输出错误信息并引起系统崩溃。


```go
// TempFile creates a new unique temp file.
func (h *Harness) TempFile() *os.File {
	f, err := os.CreateTemp(h.Dir, "")
	if err != nil {
		log.Panicf("creating temp file: %s", err.Error())
	}
	return f
}

// WriteFile writes a file given a filename and its contents.
// The filename must be a relative path, or this panics.
func (h *Harness) WriteFile(filename, contents string) {
	if filepath.IsAbs(filename) {
		log.Panicf("%s must be a relative path", filename)
	}
	absPath := filepath.Join(h.Runner.Dir, filename)
	err := os.MkdirAll(filepath.Dir(absPath), 0o777)
	if err != nil {
		log.Panicf("creating intermediate dirs for %q: %s", filename, err.Error())
	}
	err = os.WriteFile(absPath, []byte(contents), 0o644)
	if err != nil {
		log.Panicf("writing %q (%q): %s", filename, absPath, err.Error())
	}
}

```

该函数 `WaitForFile` 接收两个参数：路径和超时时间 `timeout`。它的作用是等待文件出现在指定路径并检查文件是否存在或出错。

函数首先记录当前时间和一个计时器 `timer` 和一个定时器 `ticker`, `timer` 用于在超时时间内执行函数体， `ticker` 用于每隔一毫秒执行一次。这两个定时器在函数外部都停止了。

函数内使用了一个无限循环，在每个循环中，它首先检查是否超时，如果超时了，函数就会输出一个错误并退出循环。否则，它尝试获取文件的状态并检查是否可以使用 `os.Stat` 函数成功获取文件信息。

如果文件不存在，函数将输出一个错误并退出循环。如果文件存在但状态不正确，函数将输出一个错误并退出循环。如果文件存在，函数不会做任何处理，因为它只是尝试获取文件的状态。

函数使用了计时器来确保在超时时间内有足够的时间来等待文件出现。如果超时后仍然没有文件出现，函数就会退出循环，并返回一个错误。


```go
func WaitForFile(path string, timeout time.Duration) error {
	start := time.Now()
	timer := time.NewTimer(timeout)
	ticker := time.NewTicker(1 * time.Millisecond)
	defer timer.Stop()
	defer ticker.Stop()
	for {
		select {
		case <-timer.C:
			return fmt.Errorf("timeout waiting for %s after %v", path, time.Since(start))
		case <-ticker.C:
			_, err := os.Stat(path)
			if err == nil {
				return nil
			}
			if errors.Is(err, os.ErrNotExist) {
				continue
			}
			return fmt.Errorf("error waiting for %s: %w", path, err)
		}
	}
}

```

这两段代码定义了两个函数：`func` 和 `Sh`。

函数 `func` 接收一个字符串参数 `paths`，然后使用一个循环遍历 `paths`。 对于每个 `path`，函数先检查它是否为绝对路径，如果是，函数会输出一条错误消息。否则，函数会将 `path` 和绝对路径拼接成 `absPath`，并使用 `os.MkdirAll` 函数创建目录。如果函数在创建目录时出现错误，函数会输出错误消息。

函数 `Sh` 接收一个字符串参数 `expr`，然后使用 `-c` 选项运行 `expr`。函数会将 `expr` 和 `"-c"` 拼接成 `cmd`，并使用 `h.Runner.Run` 函数运行 `cmd`。由于 `Sh` 函数没有其他参数，因此它返回的结果是 `*RunResult` 类型的空指针。


```go
func (h *Harness) Mkdirs(paths ...string) {
	for _, path := range paths {
		if filepath.IsAbs(path) {
			log.Panicf("%s must be a relative path when making dirs", path)
		}
		absPath := filepath.Join(h.Runner.Dir, path)
		err := os.MkdirAll(absPath, 0o777)
		if err != nil {
			log.Panicf("recursively making dirs under %s: %s", absPath, err)
		}
	}
}

func (h *Harness) Sh(expr string) *RunResult {
	return h.Runner.Run(RunRequest{
		Path: "bash",
		Args: []string{"-c", expr},
	})
}

```

此代码定义了一个名为 "Cleanup" 的函数，接受一个名为 "h" 的 *Harness 类型的参数。

此函数的主要作用是清理由该 harness 对象创建的集群资源。具体来说，以下是此函数的主要步骤：

1. 调用 log.Debugf 函数打印一条消息，内容为 "cleaning up cluster"。
2. 调用 h.Nodes.StopDaemons 函数停止该 harness 的所有节点。
3. 调用 log.Debugf 函数打印一条消息，内容为 "removing harness dir"。
4. 使用 os.RemoveAll 函数移除 harness 对象所创建的目录及其所有文件。

函数 "ExtractPeerID" 的作用是提取给定 multiaddr 中包含的 peer ID。具体来说，以下是此函数的主要步骤：

1. 定义一个名为 "h" 的 *Harness 类型的变量。
2. 遍历给定的 multiaddr 中的每个组件，并检查其是否为 P2P 组件。如果是，则提取出组件的值并将其存储在 "peerIDStr" 变量中。
3. 如果 "peerIDStr" 为空，则引发一个名为 "err" 的异常。
4. 使用 peer.Decode 函数将提取出的 peer ID 解析为 peer.ID 类型。
5. 返回提取出的 peer ID。


```go
func (h *Harness) Cleanup() {
	log.Debugf("cleaning up cluster")
	h.Nodes.StopDaemons()
	// TODO: don't do this if test fails, not sure how?
	log.Debugf("removing harness dir")
	err := os.RemoveAll(h.Dir)
	if err != nil {
		log.Panicf("removing temp dir %s: %s", h.Dir, err)
	}
}

// ExtractPeerID extracts a peer ID from the given multiaddr, and fatals if it does not contain a peer ID.
func (h *Harness) ExtractPeerID(m multiaddr.Multiaddr) peer.ID {
	var peerIDStr string
	multiaddr.ForEach(m, func(c multiaddr.Component) bool {
		if c.Protocol().Code == multiaddr.P_P2P {
			peerIDStr = c.Value()
		}
		return true
	})
	if peerIDStr == "" {
		panic(multiaddr.ErrProtocolNotFound)
	}
	peerID, err := peer.Decode(peerIDStr)
	if err != nil {
		panic(err)
	}
	return peerID
}

```