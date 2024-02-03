# `kubo\test\cli\stats_test.go`

```go
package cli

import (
    "testing"  // 导入测试包

    "github.com/stretchr/testify/assert"  // 导入断言包

    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
)

func TestStats(t *testing.T) {
    t.Parallel()  // 并行执行测试

    t.Run("stats dht", func(t *testing.T) {  // 运行名为 "stats dht" 的测试
        t.Parallel()  // 并行执行测试
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()  // 创建两个节点并初始化、启动守护进程、连接节点
        node1 := nodes[0]  // 获取第一个节点

        res := node1.IPFS("stats", "dht")  // 调用节点的 IPFS 方法获取 dht 统计信息
        assert.NoError(t, res.Err)  // 断言结果中没有错误
        assert.Equal(t, 0, len(res.Stderr.Lines()))  // 断言标准错误输出为空
        assert.NotEqual(t, 0, len(res.Stdout.Lines()))  // 断言标准输出不为空
    })
}
```