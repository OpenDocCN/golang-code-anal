# `kubo\test\cli\peering_test.go`

```
package cli

import (
    "testing"
    "time"

    "github.com/ipfs/kubo/test/cli/harness"
    . "github.com/ipfs/kubo/test/cli/testutils"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/stretchr/testify/assert"
)

func TestPeering(t *testing.T) {
    t.Parallel()

    containsPeerID := func(p peer.ID, peers []peer.ID) bool {
        // 检查给定的 peer.ID 是否在 peers 数组中
        for _, peerID := range peers {
            if p == peerID {
                return true
            }
        }
        return false
    }

    assertPeered := func(h *harness.Harness, from *harness.Node, to *harness.Node) {
        // 断言 from 节点和 to 节点之间建立了对等连接
        assert.Eventuallyf(t, func() bool {
            fromPeers := from.Peers()
            if len(fromPeers) == 0 {
                return false
            }
            var fromPeerIDs []peer.ID
            for _, p := range fromPeers {
                fromPeerIDs = append(fromPeerIDs, h.ExtractPeerID(p))
            }
            return containsPeerID(to.PeerID(), fromPeerIDs)
        }, time.Minute, 10*time.Millisecond, "%d -> %d not peered", from.ID, to.ID)
    }

    assertNotPeered := func(h *harness.Harness, from *harness.Node, to *harness.Node) {
        // 断言 from 节点和 to 节点之间没有建立对等连接
        assert.Eventuallyf(t, func() bool {
            fromPeers := from.Peers()
            if len(fromPeers) == 0 {
                return false
            }
            var fromPeerIDs []peer.ID
            for _, p := range fromPeers {
                fromPeerIDs = append(fromPeerIDs, h.ExtractPeerID(p))
            }
            return !containsPeerID(to.PeerID(), fromPeerIDs)
        }, 20*time.Second, 10*time.Millisecond, "%d -> %d peered", from.ID, to.ID)
    }

    assertPeerings := func(h *harness.Harness, nodes []*harness.Node, peerings []harness.Peering) {
        // 对于给定的节点和对等关系，断言节点之间建立了对等连接
        ForEachPar(peerings, func(peering harness.Peering) {
            assertPeered(h, nodes[peering.From], nodes[peering.To])
        })
    }
}
    t.Run("bidirectional peering should work (simultaneous connect)", func(t *testing.T) {
        // 并行测试，创建一个包含三个节点的对等网络，并进行双向连接测试
        t.Parallel()
        // 定义节点之间的对等关系
        peerings := []harness.Peering{{From: 0, To: 1}, {From: 1, To: 0}, {From: 1, To: 2}}
        // 创建节点和对等关系
        h, nodes := harness.CreatePeerNodes(t, 3, peerings)

        // 启动节点守护进程
        nodes.StartDaemons()
        // 断言节点之间的对等关系是否符合预期
        assertPeerings(h, nodes, peerings)

        // 断开节点0和节点1的连接，并断言对等关系是否符合预期
        nodes[0].Disconnect(nodes[1])
        assertPeerings(h, nodes, peerings)
    })

    t.Run("1 should reconnect to 2 when 2 disconnects from 1", func(t *testing.T) {
        // 并行测试，当节点2从节点1断开连接时，节点1应重新连接到节点2
        t.Parallel()
        // 定义节点之间的对等关系
        peerings := []harness.Peering{{From: 0, To: 1}, {From: 1, To: 0}, {From: 1, To: 2}}
        // 创建节点和对等关系
        h, nodes := harness.CreatePeerNodes(t, 3, peerings)

        // 启动节点守护进程
        nodes.StartDaemons()
        // 断言节点之间的对等关系是否符合预期
        assertPeerings(h, nodes, peerings)

        // 断开节点2和节点1的连接，并断言对等关系是否符合预期
        nodes[2].Disconnect(nodes[1])
        assertPeerings(h, nodes, peerings)
    })

    t.Run("1 will peer with 2 when it comes online", func(t *testing.T) {
        // 并行测试，当节点1上线时，它将与节点2建立对等关系
        t.Parallel()
        // 定义节点之间的对等关系
        peerings := []harness.Peering{{From: 0, To: 1}, {From: 1, To: 0}, {From: 1, To: 2}}
        // 创建节点和对等关系
        h, nodes := harness.CreatePeerNodes(t, 3, peerings)

        // 启动节点1和节点2的守护进程
        nodes[0].StartDaemon()
        nodes[1].StartDaemon()
        // 断言节点之间的对等关系是否符合预期
        assertPeerings(h, nodes, []harness.Peering{{From: 0, To: 1}, {From: 1, To: 0}})

        // 启动节点2的守护进程，并断言对等关系是否符合预期
        nodes[2].StartDaemon()
        assertPeerings(h, nodes, peerings)
    })

    t.Run("1 will re-peer with 2 when it disconnects and then comes back online", func(t *testing.T) {
        // 并行测试，当节点2断开连接然后重新上线时，节点1将重新与节点2建立对等关系
        t.Parallel()
        // 定义节点之间的对等关系
        peerings := []harness.Peering{{From: 0, To: 1}, {From: 1, To: 0}, {From: 1, To: 2}}
        // 创建节点和对等关系
        h, nodes := harness.CreatePeerNodes(t, 3, peerings)

        // 启动节点守护进程
        nodes.StartDaemons()
        // 断言节点之间的对等关系是否符合预期
        assertPeerings(h, nodes, peerings)

        // 停止节点2的守护进程，并断言节点1和节点2之间不再对等
        nodes[2].StopDaemon()
        assertNotPeered(h, nodes[1], nodes[2])

        // 启动节点2的守护进程，并断言对等关系是否符合预期
        nodes[2].StartDaemon()
        assertPeerings(h, nodes, peerings)
    })
# 闭合前面的函数定义
```