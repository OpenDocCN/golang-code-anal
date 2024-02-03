# `kubo\core\corehttp\metrics_test.go`

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
// It builds 4 nodes and connects them, one being the sole center.
// Then it checks that the center reports the correct number of peers.
func TestPeersTotal(t *testing.T) {
    ctx := context.Background()

    hosts := make([]*bhost.BasicHost, 4)  // 创建一个包含4个bhost.BasicHost指针的切片
    for i := 0; i < 4; i++ {  // 循环4次
        var err error
        hosts[i], err = bhost.NewHost(swarmt.GenSwarm(t), nil)  // 使用swarmt.GenSwarm(t)创建新的主机，并将其赋值给hosts[i]
        if err != nil {  // 如果有错误发生
            t.Fatal(err)  // 输出错误信息并终止测试
        }
    }

    dial := func(a, b inet.Network) {  // 定义一个名为dial的函数，接受两个inet.Network参数
        swarmt.DivulgeAddresses(b, a)  // 将b主机的地址透露给a主机
        if _, err := a.DialPeer(ctx, b.LocalPeer()); err != nil {  // 如果在a主机上拨号到b主机的本地对等点时发生错误
            t.Fatalf("Failed to dial: %s", err)  // 输出错误信息并终止测试
        }
    }

    dial(hosts[0].Network(), hosts[1].Network())  // 使用dial函数连接hosts[0]和hosts[1]
    dial(hosts[0].Network(), hosts[2].Network())  // 使用dial函数连接hosts[0]和hosts[2]
    dial(hosts[0].Network(), hosts[3].Network())  // 使用dial函数连接hosts[0]和hosts[3]

    // there's something wrong with dial, i think. it's not finishing
    // completely. there must be some async stuff.
    <-time.After(100 * time.Millisecond)  // 等待100毫秒

    node := &core.IpfsNode{PeerHost: hosts[0]}  // 创建一个core.IpfsNode结构体，将hosts[0]赋值给PeerHost字段
    collector := IpfsNodeCollector{Node: node}  // 创建一个IpfsNodeCollector结构体，将node赋值给Node字段
    peersTransport := collector.PeersTotalValues()  // 调用PeersTotalValues方法，获取peersTransport值
    if len(peersTransport) > 2 {  // 如果peersTransport的长度大于2
        t.Fatalf("expected at most 2 peers transport (tcp and upd/quic), got %d, transport map %v",
            len(peersTransport), peersTransport)  // 输出错误信息并终止测试
    }
    totalPeers := peersTransport["/ip4/tcp"] + peersTransport["/ip4/udp/quic-v1"]  // 计算总对等点数
    if totalPeers != 3 {  // 如果总对等点数不等于3
        t.Fatalf("expected 3 peers in either tcp or upd/quic transport, got %f", totalPeers)  // 输出错误信息并终止测试
    }
}
```