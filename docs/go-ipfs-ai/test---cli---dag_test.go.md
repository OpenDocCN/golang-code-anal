# `kubo\test\cli\dag_test.go`

```go
package cli

import (
    "encoding/json"  // 导入 JSON 编解码包
    "io"  // 导入 IO 包
    "os"  // 导入操作系统相关包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试相关包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/stretchr/testify/assert"  // 导入断言包
)

const (
    fixtureFile    = "./fixtures/TestDagStat.car"  // 定义常量 fixtureFile，表示测试用的文件路径
    textOutputPath = "./fixtures/TestDagStatExpectedOutput.txt"  // 定义常量 textOutputPath，表示测试输出文件路径
    node1Cid       = "bafyreibmdfd7c5db4kls4ty57zljfhqv36gi43l6txl44pi423wwmeskwy"  // 定义常量 node1Cid，表示节点1的 CID
    node2Cid       = "bafyreie3njilzdi4ixumru4nzgecsnjtu7fzfcwhg7e6s4s5i7cnbslvn4"  // 定义常量 node2Cid，表示节点2的 CID
    fixtureCid     = "bafyreifrm6uf5o4dsaacuszf35zhibyojlqclabzrms7iak67pf62jygaq"  // 定义常量 fixtureCid，表示测试用的 CID
)

type DagStat struct {
    Cid       string `json:"Cid"`  // 定义结构体 DagStat，包含 CID 字段
    Size      int    `json:"Size"`  // 定义结构体 DagStat，包含 Size 字段
    NumBlocks int    `json:"NumBlocks"`  // 定义结构体 DagStat，包含 NumBlocks 字段
}

type Data struct {
    UniqueBlocks int       `json:"UniqueBlocks"`  // 定义结构体 Data，包含 UniqueBlocks 字段
    TotalSize    int       `json:"TotalSize"`  // 定义结构体 Data，包含 TotalSize 字段
    SharedSize   int       `json:"SharedSize"`  // 定义结构体 Data，包含 SharedSize 字段
    Ratio        float64   `json:"Ratio"`  // 定义结构体 Data，包含 Ratio 字段
    DagStats     []DagStat `json:"DagStats"`  // 定义结构体 Data，包含 DagStats 字段
}

// The Fixture file represents a dag where 2 nodes of size = 46B each, have a common child of 7B
// when traversing the DAG from the root's children (node1 and node2) we count (46 + 7)x2 bytes (counting redundant bytes) = 106
// since both nodes share a common child of 7 bytes we actually had to read (46)x2 + 7 =  99 bytes
// we should get a dedup ratio of 106/99 that results in approximatelly 1.0707071

func TestDag(t *testing.T) {
    t.Parallel()  // 并行执行测试
    # 运行测试函数 "ipfs dag stat --enc=json"
    t.Run("ipfs dag stat --enc=json", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建新的测试节点
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 导入测试数据
        r, err := os.Open(fixtureFile)
        assert.Nil(t, err)
        defer r.Close()
        err = node.IPFSDagImport(r, fixtureCid)
        assert.NoError(t, err)
        # 运行 IPFS 命令 "dag stat"，返回 JSON 格式的结果
        stat := node.RunIPFS("dag", "stat", "--progress=false", "--enc=json", node1Cid, node2Cid)
        var data Data
        # 解析 JSON 格式的结果
        err = json.Unmarshal(stat.Stdout.Bytes(), &data)
        assert.NoError(t, err)

        # 期望的唯一块数
        expectedUniqueBlocks := 3
        # 期望的共享大小
        expectedSharedSize := 7
        # 期望的总大小
        expectedTotalSize := 99
        # 期望的比率
        expectedRatio := float64(expectedSharedSize+expectedTotalSize) / float64(expectedTotalSize)
        # 期望的 DAG 统计长度
        expectedDagStatsLength := 2
        # 验证唯一块数
        assert.Equal(t, expectedUniqueBlocks, data.UniqueBlocks)
        # 验证共享大小
        assert.Equal(t, expectedSharedSize, data.SharedSize)
        # 验证总大小
        assert.Equal(t, expectedTotalSize, data.TotalSize)
        # 验证比率
        assert.Equal(t, testutils.FloatTruncate(expectedRatio, 4), testutils.FloatTruncate(data.Ratio, 4))

        # 验证 DAG 统计
        assert.Equal(t, expectedDagStatsLength, len(data.DagStats))
        node1Output := data.DagStats[0]
        node2Output := data.DagStats[1]

        # 验证节点1的输出
        assert.Equal(t, node1Output.Cid, node1Cid)
        # 验证节点2的输出
        assert.Equal(t, node2Output.Cid, node2Cid)

        # 期望的节点1大小
        expectedNode1Size := (expectedTotalSize + expectedSharedSize) / 2
        # 期望的节点2大小
        expectedNode2Size := (expectedTotalSize + expectedSharedSize) / 2
        # 验证节点1的大小
        assert.Equal(t, expectedNode1Size, node1Output.Size)
        # 验证节点2的大小
        assert.Equal(t, expectedNode2Size, node2Output.Size)

        # 期望的节点1块数
        expectedNode1Blocks := 2
        # 期望的节点2块数
        expectedNode2Blocks := 2
        # 验证节点1的块数
        assert.Equal(t, expectedNode1Blocks, node1Output.NumBlocks)
        # 验证节点2的块数
        assert.Equal(t, expectedNode2Blocks, node2Output.NumBlocks)
    })
    # 运行测试函数 "ipfs dag stat"
    t.Run("ipfs dag stat", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建新的节点，并初始化并启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 打开指定的文件，返回文件对象和可能的错误
        r, err := os.Open(fixtureFile)
        assert.NoError(t, err)
        # 延迟关闭文件
        defer r.Close()
        # 打开指定的文件，返回文件对象和可能的错误
        f, err := os.Open(textOutputPath)
        assert.NoError(t, err)
        # 延迟关闭文件
        defer f.Close()
        # 读取文件的所有内容
        content, err := io.ReadAll(f)
        assert.NoError(t, err)
        # 使用 IPFS 导入数据到 DAG
        err = node.IPFSDagImport(r, fixtureCid)
        assert.NoError(t, err)
        # 运行 IPFS 命令 "dag stat"，并禁用进度显示
        stat := node.RunIPFS("dag", "stat", "--progress=false", node1Cid, node2Cid)
        # 断言内容与输出的标准输出字节相等
        assert.Equal(t, content, stat.Stdout.Bytes())
    })
# 闭合前面的函数定义
```