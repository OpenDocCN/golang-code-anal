# `kubo\test\cli\dht_autoclient_test.go`

```go
package cli
# 导入所需的包

import (
    "bytes"
    "testing"

    "github.com/ipfs/kubo/test/cli/harness"
    "github.com/ipfs/kubo/test/cli/testutils"
    "github.com/stretchr/testify/assert"
)

func TestDHTAutoclient(t *testing.T) {
    t.Parallel()
    # 创建测试函数，允许并行执行
    nodes := harness.NewT(t).NewNodes(10).Init()
    # 初始化测试节点
    harness.Nodes(nodes[8:]).ForEachPar(func(node *harness.Node) {
        node.IPFS("config", "Routing.Type", "autoclient")
    })
    # 对节点进行配置
    nodes.StartDaemons().Connect()
    # 启动守护进程并连接节点

    t.Run("file added on node in client mode is retrievable from node in client mode", func(t *testing.T) {
        t.Parallel()
        # 创建测试子函数，允许并行执行
        randomBytes := testutils.RandomBytes(1000)
        # 生成随机字节
        randomBytes = append(randomBytes, '\r')
        # 在随机字节后追加回车符
        hash := nodes[8].IPFSAdd(bytes.NewReader(randomBytes))
        # 在第8个节点上添加文件，并获取其哈希值

        res := nodes[9].IPFS("cat", hash)
        # 从第9个节点上获取文件内容
        assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
        # 断言获取的文件内容与随机字节相等
    })

    t.Run("file added on node in server mode is retrievable from all nodes", func(t *testing.T) {
        t.Parallel()
        # 创建测试子函数，允许并行执行
        randomBytes := testutils.RandomBytes(1000)
        # 生成随机字节
        hash := nodes[0].IPFSAdd(bytes.NewReader(randomBytes))
        # 在第0个节点上添加文件，并获取其哈希值

        for i := 0; i < 10; i++ {
            # 循环遍历所有节点
            res := nodes[i].IPFS("cat", hash)
            # 从每个节点上获取文件内容
            assert.Equal(t, randomBytes, []byte(res.Stdout.Trimmed()))
            # 断言获取的文件内容与随机字节相等
        }
    })
}
```