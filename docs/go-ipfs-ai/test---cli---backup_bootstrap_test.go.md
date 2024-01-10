# `kubo\test\cli\backup_bootstrap_test.go`

```
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，用于处理时间

    "github.com/ipfs/kubo/config"  // 导入 config 包，用于处理配置
    "github.com/ipfs/kubo/test/cli/harness"  // 导入 harness 包，用于测试 CLI
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于断言
)

func TestBackupBootstrapPeers(t *testing.T) {
    nodes := harness.NewT(t).NewNodes(3).Init()  // 创建 3 个节点并初始化
    nodes.ForEachPar(func(n *harness.Node) {  // 并行遍历每个节点
        n.UpdateConfig(func(cfg *config.Config) {  // 更新节点配置
            cfg.Bootstrap = []string{}  // 设置引导节点为空
            cfg.Addresses.Swarm = []string{fmt.Sprintf("/ip4/127.0.0.1/tcp/%d", harness.NewRandPort())}  // 设置 Swarm 地址
            cfg.Discovery.MDNS.Enabled = false  // 禁用 MDNS
            cfg.Internal.BackupBootstrapInterval = config.NewOptionalDuration(250 * time.Millisecond)  // 设置备份引导间隔时间
        })
    })

    // 启动所有节点并确保它们都没有对等节点
    nodes.StartDaemons()
    nodes.ForEachPar(func(n *harness.Node) {  // 并行遍历每个节点
        assert.Len(t, n.Peers(), 0)  // 断言节点的对等节点数量为 0
    })

    // 连接节点 0 和 1，并确保它们彼此知道
    nodes[0].Connect(nodes[1])
    assert.Len(t, nodes[0].Peers(), 1)  // 断言节点 0 的对等节点数量为 1
    assert.Len(t, nodes[1].Peers(), 1)  // 断言节点 1 的对等节点数量为 1
    assert.Len(t, nodes[2].Peers(), 0)  // 断言节点 2 的对等节点数量为 0

    // 等待一段时间以确保 0 和 1 保存了它们的临时引导备份
    time.Sleep(time.Millisecond * 500)
    nodes.StopDaemons()

    // 启动 1 和 2。2 目前还不知道任何节点
    nodes[1].StartDaemon()
    nodes[2].StartDaemon()
    assert.Len(t, nodes[1].Peers(), 0)  // 断言节点 1 的对等节点数量为 0
    assert.Len(t, nodes[2].Peers(), 0)  // 断言节点 2 的对等节点数量为 0

    // 连接 1 和 2，并确保它们彼此知道
    nodes[1].Connect(nodes[2])
    assert.Len(t, nodes[1].Peers(), 1)  // 断言节点 1 的对等节点数量为 1
    assert.Len(t, nodes[2].Peers(), 1)  // 断言节点 2 的对等节点数量为 1

    // 启动 0，等待一段时间。应该连接到 1，然后通过备份引导节点发现 2
    nodes[0].StartDaemon()
    time.Sleep(time.Millisecond * 500)

    // 检查它们是否都连接上了
    assert.Len(t, nodes[0].Peers(), 2)  // 断言节点 0 的对等节点数量为 2
    assert.Len(t, nodes[1].Peers(), 2)  // 断言节点 1 的对等节点数量为 2
    assert.Len(t, nodes[2].Peers(), 2)  // 断言节点 2 的对等节点数量为 2
}
```