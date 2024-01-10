# `kubo\test\cli\delegated_routing_v1_http_server_test.go`

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

func TestRoutingV1Server(t *testing.T) {
    t.Parallel()

    // 设置测试环境，初始化5个节点
    setupNodes := func(t *testing.T) harness.Nodes {
        nodes := harness.NewT(t).NewNodes(5).Init()
        nodes.ForEachPar(func(node *harness.Node) {
            // 更新节点配置，暴露路由API并设置路由类型为"dht"
            node.UpdateConfig(func(cfg *config.Config) {
                cfg.Gateway.ExposeRoutingAPI = config.True
                cfg.Routing.Type = config.NewOptionalString("dht")
            })
        })
        // 启动节点守护进程并连接
        nodes.StartDaemons().Connect()
        return nodes
    }

    t.Run("Get Providers Responds With Correct Peers", func(t *testing.T) {
        t.Parallel()
        // 设置测试环境
        nodes := setupNodes(t)

        // 创建测试文本和CID
        text := "hello world " + uuid.New().String()
        cidStr := nodes[2].IPFSAddStr(text)
        _ = nodes[3].IPFSAddStr(text)

        cid, err := cid.Decode(cidStr)
        assert.NoError(t, err)

        // 创建HTTP客户端
        c, err := client.New(nodes[1].GatewayURL())
        assert.NoError(t, err)

        // 查找提供CID的节点
        resultsIter, err := c.FindProviders(context.Background(), cid)
        assert.NoError(t, err)

        // 读取所有结果
        records, err := iter.ReadAllResults(resultsIter)
        assert.NoError(t, err)

        var peers []peer.ID
        for _, record := range records {
            assert.Equal(t, types.SchemaPeer, record.GetSchema())

            peer, ok := record.(*types.PeerRecord)
            assert.True(t, ok)
            peers = append(peers, *peer.ID)
        }

        // 断言包含特定节点的PeerID
        assert.Contains(t, peers, nodes[2].PeerID())
        assert.Contains(t, peers, nodes[3].PeerID())
    })
}
    # 使用测试框架运行"Get Peers Responds With Correct Peers"测试
    t.Run("Get Peers Responds With Correct Peers", func(t *testing.T) {
        # 标记该测试可以并行执行
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 创建客户端连接到指定节点的网关
        c, err := client.New(nodes[1].GatewayURL())
        assert.NoError(t, err)

        # 查找指定节点的对等节点
        resultsIter, err := c.FindPeers(context.Background(), nodes[2].PeerID())
        assert.NoError(t, err)

        # 读取所有查找结果
        records, err := iter.ReadAllResults(resultsIter)
        assert.NoError(t, err)
        # 断言结果记录的数量为1
        assert.Len(t, records, 1)
        # 断言结果记录的类型为PeerRecord
        assert.IsType(t, records[0].GetSchema(), records[0].GetSchema())
        assert.IsType(t, records[0], &types.PeerRecord{})

        # 获取第一个记录作为对等节点
        peer := records[0]
        # 断言对等节点的ID与指定节点的对等节点ID相等
        assert.Equal(t, nodes[2].PeerID().String(), peer.ID.String())
        # 断言对等节点的地址列表不为空
        assert.NotEmpty(t, peer.Addrs)
    })

    # 使用测试框架运行"Get IPNS Record Responds With Correct Record"测试
    t.Run("Get IPNS Record Responds With Correct Record", func(t *testing.T) {
        # 标记该测试可以并行执行
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 创建一个包含随机文本的IPNS记录，并发布到IPFS网络
        text := "hello ipns test " + uuid.New().String()
        cidStr := nodes[0].IPFSAddStr(text)
        nodes[0].IPFS("name", "publish", "--allow-offline", cidStr)

        # 从不同的节点请求IPNS记录
        c, err := client.New(nodes[1].GatewayURL())
        assert.NoError(t, err)

        # 获取指定节点的IPNS记录
        record, err := c.GetIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()))
        assert.NoError(t, err)

        # 获取记录的值
        value, err := record.Value()
        assert.NoError(t, err)
        # 断言记录的值与预期的CID字符串相等
        assert.Equal(t, "/ipfs/"+cidStr, value.String())
    })
    t.Run("Put IPNS Record Succeeds", func(t *testing.T) {
        t.Parallel()
        nodes := setupNodes(t)

        // 在测试中放置一个 IPNS 记录，并确认 /routing/v1/ipns API 是否公开了该 IPNS 记录
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

        // 启动一个孤立的节点，该节点不连接到其他节点。
        node := harness.NewT(t).NewNode().Init()
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Gateway.ExposeRoutingAPI = config.True
            cfg.Routing.Type = config.NewOptionalString("dht")
        })
        node.StartDaemon()

        // 在孤立节点中放置 IPNS 记录。如果是有效记录，应该被接受。
        c, err = client.New(node.GatewayURL())
        assert.NoError(t, err)
        err = c.PutIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()), record)
        assert.NoError(t, err)

        // 从孤立节点中获取记录并进行双重检查。
        record, err = c.GetIPNS(context.Background(), ipns.NameFromPeer(nodes[0].PeerID()))
        assert.NoError(t, err)
        value, err = record.Value()
        assert.NoError(t, err)
        assert.Equal(t, "/ipfs/"+cidStr, value.String())
    })
# 闭合前面的函数定义
```