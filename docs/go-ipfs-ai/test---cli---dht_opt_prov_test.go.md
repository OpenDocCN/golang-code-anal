# `kubo\test\cli\dht_opt_prov_test.go`

```
package cli

import (
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

func TestDHTOptimisticProvide(t *testing.T) {
    t.Parallel()  // 并行执行测试

    t.Run("optimistic provide smoke test", func(t *testing.T) {  // 运行一个测试子组
        nodes := harness.NewT(t).NewNodes(2).Init()  // 创建两个节点并初始化

        nodes[0].UpdateConfig(func(cfg *config.Config) {  // 更新节点0的配置
            cfg.Experimental.OptimisticProvide = true  // 设置实验性的乐观提供为真
        })

        nodes.StartDaemons().Connect()  // 启动节点并连接

        hash := nodes[0].IPFSAddStr(testutils.RandomStr(100))  // 生成一个随机字符串并将其添加到IPFS，返回哈希值
        nodes[0].IPFS("dht", "provide", hash)  // 节点0提供哈希值到DHT网络

        res := nodes[1].IPFS("routing", "findprovs", "--num-providers=1", hash)  // 节点1查找提供者
        assert.Equal(t, nodes[0].PeerID().String(), res.Stdout.Trimmed())  // 断言节点0的对等ID与结果的标准输出相等
    })
}
```