# `kubo\test\cli\dht_legacy_test.go`

```go
package cli

import (
    "sort"  // 导入 sort 包，用于对数据进行排序
    "sync"  // 导入 sync 包，用于实现并发安全的数据结构
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/test/cli/harness"  // 导入自定义包 harness，用于测试 CLI
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入自定义包 testutils，用于测试 CLI
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 包中的 peer 模块，用于处理节点的 peer ID
    "github.com/stretchr/testify/assert"  // 导入 testify 包中的 assert 模块，用于编写测试断言
    "github.com/stretchr/testify/require"  // 导入 testify 包中的 require 模块，用于编写测试断言
)

func TestLegacyDHT(t *testing.T) {
    t.Parallel()  // 标记测试函数可以并行执行
    nodes := harness.NewT(t).NewNodes(5).Init()  // 创建测试节点并初始化
    nodes.ForEachPar(func(node *harness.Node) {  // 并行遍历测试节点
        node.IPFS("config", "Routing.Type", "dht")  // 设置节点的路由类型为 dht
    })
    nodes.StartDaemons().Connect()  // 启动节点守护进程并连接节点之间的网络

    t.Run("ipfs dht findpeer", func(t *testing.T) {  // 运行子测试函数 "ipfs dht findpeer"
        t.Parallel()  // 标记子测试函数可以并行执行
        res := nodes[1].RunIPFS("dht", "findpeer", nodes[0].PeerID().String())  // 在节点 1 上运行 IPFS 命令查找节点 0 的 peer ID
        assert.Equal(t, 0, res.ExitCode())  // 使用断言检查命令执行结果的退出码是否为 0

        swarmAddr := nodes[0].SwarmAddrsWithoutPeerIDs()[0]  // 获取节点 0 的 Swarm 地址
        require.Equal(t, swarmAddr.String(), res.Stdout.Trimmed())  // 使用断言检查命令执行结果的标准输出是否与 Swarm 地址匹配
    })
}
    # 测试 IPFS DHT 获取指定键的内容
    t.Run("ipfs dht get <key>", func(t *testing.T) {
        t.Parallel()
        # 在节点2上添加字符串 "hello world" 并返回哈希值
        hash := nodes[2].IPFSAddStr("hello world")
        # 在节点2上发布哈希值对应的 IPNS
        nodes[2].IPFS("name", "publish", "/ipfs/"+hash)

        # 从节点1的 IPFS DHT 中获取节点2的 IPNS 对应的内容
        res := nodes[1].IPFS("dht", "get", "/ipns/"+nodes[2].PeerID().String())
        # 断言获取的内容中包含指定的哈希值
        assert.Contains(t, res.Stdout.String(), "/ipfs/"+hash)

        # 测试 IPFS DHT 放置往返 (#3124)
        t.Run("put round trips (#3124)", func(t *testing.T) {
            t.Parallel()
            # 将获取的结果写入节点0
            nodes[0].WriteBytes("get_result", res.Stdout.Bytes())
            # 将获取的结果放置到节点0的 IPNS 中
            res := nodes[0].IPFS("dht", "put", "/ipns/"+nodes[2].PeerID().String(), "get_result")
            # 断言获取的结果中包含至少一个节点的信息
            assert.Greater(t, len(res.Stdout.Lines()), 0, "should put to at least one node")
        })

        # 测试 IPFS DHT 放置无效键失败 (问题 #5113, #4611)
        t.Run("put with bad keys fails (issue #5113, #4611)", func(t *testing.T) {
            t.Parallel()
            # 定义一组无效的键
            keys := []string{"foo", "/pk/foo", "/ipns/foo"}
            for _, key := range keys:
                key := key
                t.Run(key, func(t *testing.T) {
                    t.Parallel()
                    # 尝试将无效的键放置到节点0的 IPNS 中
                    res := nodes[0].RunIPFS("dht", "put", key)
                    # 断言结果的退出码为1
                    assert.Equal(t, 1, res.ExitCode())
                    # 断言结果的标准错误输出包含 "invalid"
                    assert.Contains(t, res.Stderr.String(), "invalid")
                    # 断言结果的标准输出为空
                    assert.Empty(t, res.Stdout.String())
                })
            }
        })

        # 测试 IPFS DHT 获取无效键 (问题 #4611)
        t.Run("get with bad keys (issue #4611)", func(t *testing.T) {
            for _, key := range []string{"foo", "/pk/foo"}:
                key := key
                t.Run(key, func(t *testing.T) {
                    t.Parallel()
                    # 尝试从节点0的 IPNS 中获取无效的键
                    res := nodes[0].RunIPFS("dht", "get", key)
                    # 断言结果的退出码为1
                    assert.Equal(t, 1, res.ExitCode())
                    # 断言结果的标准错误输出包含 "invalid"
                    assert.Contains(t, res.Stderr.String(), "invalid")
                    # 断言结果的标准输出为空
                    assert.Empty(t, res.Stdout.String())
                })
            }
        })
    })
    # 运行测试 "ipfs dht findprovs"，并传入测试对象 t
    t.Run("ipfs dht findprovs", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 在节点 3 上添加字符串 "some stuff" 并返回哈希值
        hash := nodes[3].IPFSAddStr("some stuff")
        # 在节点 4 上执行 IPFS 命令，查找提供者
        res := nodes[4].IPFS("dht", "findprovs", hash)
        # 断言节点 3 的对等节点 ID 是否与结果的标准输出相等
        assert.Equal(t, nodes[3].PeerID().String(), res.Stdout.Trimmed())
    })

    # 运行测试 "ipfs dht query <peerID>"，并传入测试对象 t
    t.Run("ipfs dht query <peerID>", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 在正常的 DHT 配置下运行测试
        t.Run("normal DHT configuration", func(t *testing.T) {
            # 并行执行测试
            t.Parallel()
            # 在节点 0 上添加字符串 "some other stuff" 并返回哈希值
            hash := nodes[0].IPFSAddStr("some other stuff")
            # 创建一个映射，用于存储对等节点的计数
            peerCounts := map[string]int{}
            # 创建一个互斥锁，用于保护对 peerCounts 的并发访问
            peerCountsMut := sync.Mutex{}
            # 对节点数组中的每个节点并行执行操作
            harness.Nodes(nodes).ForEachPar(func(node *harness.Node) {
                # 在每个节点上执行 IPFS 命令，查询指定哈希值的最近对等节点
                res := node.IPFS("dht", "query", hash)
                # 获取结果的标准输出的第一行，即最近的对等节点
                closestPeer := res.Stdout.Lines()[0]
                # 检查最近的对等节点是否是有效的对等节点 ID
                _, err := peer.Decode(closestPeer)
                require.NoError(t, err)

                # 加锁，以保护对 peerCounts 的并发访问
                peerCountsMut.Lock()
                # 增加对最近对等节点的计数
                peerCounts[closestPeer]++
                # 解锁
                peerCountsMut.Unlock()
            })
            # 4 个节点应该看到相同的对等节点 ID
            # 1 个节点（最近的节点）应该看到不同的对等节点 ID
            var counts []int
            for _, count := range peerCounts {
                counts = append(counts, count)
            }
            # 对计数进行排序
            sort.IntSlice(counts).Sort()
            # 断言计数是否符合预期
            assert.Equal(t, []int{1, 4}, counts)
        })
    })
    # 测试当离线时 DHT 命令是否失败
    t.Run("dht commands fail when offline", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建新节点并初始化
        node := harness.NewT(t).NewNode().Init()

        # 由于存储库锁定，这些命令不能并行运行（似乎是一个 bug）

        # 测试 dht findprovs 命令
        t.Run("dht findprovs", func(t *testing.T) {
            # 运行 IPFS 命令，查找提供者
            res := node.RunIPFS("dht", "findprovs", testutils.CIDEmptyDir)
            # 断言退出码为 1
            assert.Equal(t, 1, res.ExitCode())
            # 断言标准错误输出包含指定信息
            assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
        })

        # 测试 dht findpeer 命令
        t.Run("dht findpeer", func(t *testing.T) {
            # 运行 IPFS 命令，查找对等点
            res := node.RunIPFS("dht", "findpeer", testutils.CIDEmptyDir)
            # 断言退出码为 1
            assert.Equal(t, 1, res.ExitCode())
            # 断言标准错误输出包含指定信息
            assert.Contains(t, res.Stderr.String(), "this command must be run in online mode")
        })

        # 测试 dht put 命令
        t.Run("dht put", func(t *testing.T) {
            # 写入字节到节点
            node.WriteBytes("foo", []byte("foo"))
            # 运行 IPFS 命令，将数据放入 DHT
            res := node.RunIPFS("dht", "put", "/ipns/"+node.PeerID().String(), "foo")
            # 断言退出码为 1
            assert.Equal(t, 1, res.ExitCode())
            # 断言标准错误输出包含指定信息
            assert.Contains(t, res.Stderr.String(), "can't put while offline: pass `--allow-offline` to override")
        })
    })
# 闭合前面的函数定义
```