# `kubo\test\cli\routing_dht_test.go`

```go
package cli

import (
    "fmt"
    "testing"

    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/stretchr/testify/assert"  // 导入断言包
    "github.com/stretchr/testify/require"  // 导入断言包
)

func testRoutingDHT(t *testing.T, enablePubsub bool) {  // 定义测试函数，测试路由DHT
    // 测试逻辑
}

func testSelfFindDHT(t *testing.T) {  // 定义测试函数，测试自身查找DHT
    t.Run("ipfs routing findpeer fails for self", func(t *testing.T) {  // 运行子测试，测试IPFS路由查找自身失败
        t.Parallel()  // 并行运行子测试
        nodes := harness.NewT(t).NewNodes(1).Init()  // 创建测试节点
        nodes.ForEachPar(func(node *harness.Node) {  // 遍历节点并并行执行
            node.IPFS("config", "Routing.Type", "dht")  // 设置节点的IPFS配置
        })

        nodes.StartDaemons()  // 启动节点守护进程

        res := nodes[0].RunIPFS("dht", "findpeer", nodes[0].PeerID().String())  // 运行IPFS命令查找节点自身
        assert.Equal(t, 1, res.ExitCode())  // 断言结果是否符合预期
    })
}

func TestRoutingDHT(t *testing.T) {  // 定义测试函数，测试路由DHT
    testRoutingDHT(t, false)  // 执行路由DHT测试
    testRoutingDHT(t, true)  // 执行路由DHT测试
    testSelfFindDHT(t)  // 执行自身查找DHT测试
}
```