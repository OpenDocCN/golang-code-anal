# `kubo\test\cli\delegated_routing_v1_http_proxy_test.go`

```
package cli

import (
    "testing"

    "github.com/ipfs/boxo/ipns"  // 导入ipns包
    "github.com/ipfs/kubo/config"  // 导入config包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入harness包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入testutils包
    "github.com/stretchr/testify/assert"  // 导入assert包
    "github.com/stretchr/testify/require"  // 导入require包
)

func TestRoutingV1Proxy(t *testing.T) {
    t.Parallel()  // 并行执行测试用例

    setupNodes := func(t *testing.T) harness.Nodes {  // 定义名为setupNodes的函数
        nodes := harness.NewT(t).NewNodes(2).Init()  // 初始化nodes变量，创建2个节点

        // Node 0 uses DHT and exposes the Routing API.
        nodes[0].UpdateConfig(func(cfg *config.Config) {  // 更新节点0的配置
            cfg.Gateway.ExposeRoutingAPI = config.True  // 设置ExposeRoutingAPI为True
            cfg.Discovery.MDNS.Enabled = false  // 禁用MDNS
            cfg.Routing.Type = config.NewOptionalString("dht")  // 设置Routing类型为"dht"
        })
        nodes[0].StartDaemon()  // 启动节点0的守护进程

        // Node 1 uses Node 0 as Routing V1 source, no DHT.
        nodes[1].UpdateConfig(func(cfg *config.Config) {  // 更新节点1的配置
            cfg.Discovery.MDNS.Enabled = false  // 禁用MDNS
            cfg.Routing.Type = config.NewOptionalString("custom")  // 设置Routing类型为"custom"
            cfg.Routing.Methods = config.Methods{  // 设置Routing方法
                config.MethodNameFindPeers:     {RouterName: "KuboA"},
                config.MethodNameFindProviders: {RouterName: "KuboA"},
                config.MethodNameGetIPNS:       {RouterName: "KuboA"},
                config.MethodNamePutIPNS:       {RouterName: "KuboA"},
                config.MethodNameProvide:       {RouterName: "KuboA"},
            }
            cfg.Routing.Routers = config.Routers{  // 设置Routing路由器
                "KuboA": config.RouterParser{  // 定义名为KuboA的RouterParser
                    Router: config.Router{  // 设置Router
                        Type: config.RouterTypeHTTP,  // 设置Router类型为HTTP
                        Parameters: &config.HTTPRouterParams{  // 设置HTTPRouterParams参数
                            Endpoint: nodes[0].GatewayURL(),  // 设置Endpoint为节点0的GatewayURL
                        },
                    },
                },
            }
        })
        nodes[1].StartDaemon()  // 启动节点1的守护进程

        // Connect them.
        nodes.Connect()  // 连接节点

        return nodes  // 返回节点
    }
    # 测试 Kubo 是否能够通过 Routing V1 找到 CID 的提供者
    t.Run("Kubo can find provider for CID via Routing V1", func(t *testing.T) {
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 生成随机的 1000 字符串作为 CID
        cidStr := nodes[0].IPFSAddStr(testutils.RandomStr(1000))

        # 通过节点 1 的 IPFS 命令查找 CID 的提供者
        res := nodes[1].IPFS("routing", "findprovs", cidStr)
        # 断言节点 0 的 PeerID 是否与结果一致
        assert.Equal(t, nodes[0].PeerID().String(), res.Stdout.Trimmed())
    })

    # 测试 Kubo 是否能够通过 Routing V1 找到对等节点
    t.Run("Kubo can find peer via Routing V1", func(t *testing.T) {
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 启动一个孤立的节点，不连接其他节点
        node := harness.NewT(t).NewNode().Init()
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Discovery.MDNS.Enabled = false
            cfg.Routing.Type = config.NewOptionalString("dht")
        })
        node.StartDaemon()

        # 将节点 0 连接到孤立节点
        nodes[0].Connect(node)

        # 节点 1 通过 Routing V1 找到孤立节点
        res := nodes[1].IPFS("routing", "findpeer", node.PeerID().String())
        # 断言孤立节点的 Swarm 地址是否与结果一致
        assert.Equal(t, node.SwarmAddrs()[0].String(), res.Stdout.Trimmed())
    })

    # 测试 Kubo 是否能够通过 Routing V1 检索 IPNS 记录
    t.Run("Kubo can retrieve IPNS record via Routing V1", func(t *testing.T) {
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 构建节点名称
        nodeName := "/ipns/" + ipns.NameFromPeer(nodes[0].PeerID()).String()

        # 无法解析名称，因为尚未发布
        res := nodes[1].RunIPFS("routing", "get", nodeName)
        require.Error(t, res.ExitErr)

        # 在节点 0 上发布记录
        path := "/ipfs/" + nodes[0].IPFSAddStr(testutils.RandomStr(1000))
        nodes[0].IPFS("name", "publish", "--allow-offline", path)

        # 在节点 1 上获取记录（无 DHT）
        res = nodes[1].IPFS("routing", "get", nodeName)
        # 解析 IPNS 记录
        record, err := ipns.UnmarshalRecord(res.Stdout.Bytes())
        require.NoError(t, err)
        value, err := record.Value()
        require.NoError(t, err)
        # 断言值是否与路径一致
        require.Equal(t, path, value.String())
    })
    # 测试 Kubo 是否能够通过 Routing V1 解析 IPNS 名称
    t.Run("Kubo can resolve IPNS name via Routing V1", func(t *testing.T) {
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 构建 IPNS 名称
        nodeName := "/ipns/" + ipns.NameFromPeer(nodes[0].PeerID()).String()

        # 无法解析名称，因为尚未发布
        res := nodes[1].RunIPFS("routing", "get", nodeName)
        require.Error(t, res.ExitErr)

        # 发布名称
        path := "/ipfs/" + nodes[0].IPFSAddStr(testutils.RandomStr(1000))
        nodes[0].IPFS("name", "publish", "--allow-offline", path)

        # 解析 IPNS 名称
        res = nodes[1].IPFS("name", "resolve", nodeName)
        require.Equal(t, path, res.Stdout.Trimmed())
    })

    # 测试 Kubo 是否能够通过 Routing V1 提供 IPNS 记录
    t.Run("Kubo can provide IPNS record via Routing V1", func(t *testing.T) {
        t.Parallel()
        # 设置节点
        nodes := setupNodes(t)

        # 在节点 1 上发布内容（没有 DHT）
        nodeName := "/ipns/" + ipns.NameFromPeer(nodes[1].PeerID()).String()
        path := "/ipfs/" + nodes[1].IPFSAddStr(testutils.RandomStr(1000))
        nodes[1].IPFS("name", "publish", "--allow-offline", path)

        # 通过节点 0 检索
        res := nodes[0].IPFS("routing", "get", nodeName)
        # 反序列化 IPNS 记录
        record, err := ipns.UnmarshalRecord(res.Stdout.Bytes())
        require.NoError(t, err)
        # 获取记录的值
        value, err := record.Value()
        require.NoError(t, err)
        require.Equal(t, path, value.String())
    })
# 闭合前面的函数定义
```