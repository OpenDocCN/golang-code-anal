# `kubo\test\integration\bitswap_wo_routing_test.go`

```go
package integrationtest

import (
    "bytes" // 导入 bytes 包
    "context" // 导入 context 包
    "testing" // 导入 testing 包

    blocks "github.com/ipfs/go-block-format" // 导入 blocks 包
    "github.com/ipfs/go-cid" // 导入 cid 包
    "github.com/ipfs/kubo/core" // 导入 core 包
    coremock "github.com/ipfs/kubo/core/mock" // 导入 coremock 包
    "github.com/ipfs/kubo/core/node/libp2p" // 导入 libp2p 包
    mocknet "github.com/libp2p/go-libp2p/p2p/net/mock" // 导入 mocknet 包
)

func TestBitswapWithoutRouting(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background()) // 创建一个带有取消函数的上下文
    defer cancel() // 延迟调用取消函数
    const numPeers = 4 // 定义常量 numPeers 为 4

    // create network
    mn := mocknet.New() // 创建一个模拟网络

    var nodes []*core.IpfsNode // 创建一个 IpfsNode 类型的指针数组
    for i := 0; i < numPeers; i++ { // 循环创建 numPeers 个节点
        n, err := core.NewNode(ctx, &core.BuildCfg{ // 创建一个新的节点
            Online:  true, // 设置节点在线
            Host:    coremock.MockHostOption(mn), // 设置节点的主机选项为模拟主机
            Routing: libp2p.NilRouterOption, // 设置节点的路由选项为 NilRouterOption，即无路由
        })
        if err != nil { // 如果有错误发生
            t.Fatal(err) // 终止测试并输出错误信息
        }
        defer n.Close() // 延迟关闭节点
        nodes = append(nodes, n) // 将节点添加到节点数组中
    }

    err := mn.LinkAll() // 将所有节点连接起来
    if err != nil { // 如果有错误发生
        t.Fatal(err) // 终止测试并输出错误信息
    }

    // connect them
    for _, n1 := range nodes { // 遍历节点数组
        for _, n2 := range nodes { // 再次遍历节点数组
            if n1 == n2 { // 如果两个节点相同
                continue // 继续下一次循环
            }

            log.Debug("connecting to other hosts") // 输出调试信息
            p2 := n2.PeerHost.Peerstore().PeerInfo(n2.PeerHost.ID()) // 获取节点 n2 的对等节点信息
            if err := n1.PeerHost.Connect(ctx, p2); err != nil { // 如果连接失败
                t.Fatal(err) // 终止测试并输出错误信息
            }
        }
    }

    // add blocks to each before
    log.Debug("adding block.") // 输出调试信息
    block0 := blocks.NewBlock([]byte("block0")) // 创建一个新的块
    block1 := blocks.NewBlock([]byte("block1")) // 创建一个新的块

    // put 1 before
    if err := nodes[0].Blockstore.Put(ctx, block0); err != nil { // 将块 block0 存储到节点 0 的块存储中
        t.Fatal(err) // 终止测试并输出错误信息
    }

    //  get it out.
}
    // 遍历节点列表，获取节点索引和节点对象
    for i, n := range nodes {
        // 如果是第一个节点，则跳过，因为块不在其交换中，会导致 hang
        if i == 0 {
            continue
        }

        // 打印调试信息，表示获取块的操作
        log.Debugf("%d %s get block.", i, n.Identity)
        // 从节点获取块数据，并检查错误
        b, err := n.Blocks.GetBlock(ctx, cid.NewCidV0(block0.Multihash()))
        if err != nil {
            t.Error(err)
        } else if !bytes.Equal(b.RawData(), block0.RawData()) {
            t.Error("byte comparison fail")
        } else {
            // 打印调试信息，表示成功获取块
            log.Debug("got block: %s", b.Cid())
        }
    }

    // 将第一个块放入节点1的块存储中
    if err := nodes[1].Blockstore.Put(ctx, block1); err != nil {
        t.Fatal(err)
    }

    // 从所有节点中获取第一个块
    for _, n := range nodes {
        // 从节点获取块数据，并检查错误
        b, err := n.Blocks.GetBlock(ctx, cid.NewCidV0(block1.Multihash()))
        if err != nil {
            t.Error(err)
        } else if !bytes.Equal(b.RawData(), block1.RawData()) {
            t.Error("byte comparison fail")
        } else {
            // 打印调试信息，表示成功获取块
            log.Debug("got block: %s", b.Cid())
        }
    }
# 闭合前面的函数定义
```