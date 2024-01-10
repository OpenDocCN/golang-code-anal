# `kubo\test\cli\gateway_range_test.go`

```
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net/http"  // 导入 net/http 包，用于处理 HTTP 请求
    "os"  // 导入 os 包，用于操作系统功能
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/test/cli/harness"  // 导入自定义包 harness，用于测试辅助功能
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于断言测试结果
)

func TestGatewayHAMTDirectory(t *testing.T) {
    t.Parallel()  // 并行执行测试函数

    const (
        // HAMT 分片目录的 CID，包含 10k 个项目
        hamtCid = "bafybeiggvykl7skb2ndlmacg2k5modvudocffxjesexlod2pfvg5yhwrqm"

        // fixtureCid 是 hamtCid DAG 的子集的根节点的 CID，表示目录列表所需的最小块集。
        // 还包括一个 "files_refs" 文件，其中列出了我们不需要获取的引用（目录内的文件）
        fixtureCid = "bafybeig3yoibxe56aolixqa4zk55gp5sug3qgaztkakpndzk2b2ynobd4i"
    )

    // 启动节点
    h := harness.NewT(t)
    node := h.NewNode().Init("--empty-repo", "--profile=test").StartDaemon("--offline")
    client := node.GatewayClient()  // 获取节点的网关客户端

    // 导入测试数据
    r, err := os.Open("./fixtures/TestGatewayHAMTDirectory.car")  // 打开测试数据文件
    assert.NoError(t, err)  // 断言打开文件操作没有错误
    defer r.Close()  // 延迟关闭文件
    err = node.IPFSDagImport(r, fixtureCid)  // 导入测试数据到 IPFS 节点
    assert.NoError(t, err)  // 断言导入操作没有错误

    // 获取 HAMT 目录，使用最小引用成功
    resp := client.Get(fmt.Sprintf("/ipfs/%s/", hamtCid))  // 发起 GET 请求获取 HAMT 目录
    assert.Equal(t, http.StatusOK, resp.StatusCode)  // 断言响应状态码为 200
}

func TestGatewayHAMTRanges(t *testing.T) {
    t.Parallel()  // 并行执行测试函数

    const (
        // 大型 HAMT 分片文件的 CID
        fileCid = "bafybeiae5abzv6j3ucqbzlpnx3pcqbr2otbnpot7d2k5pckmpymin4guau"

        // fixtureCid 是 fileCid DAG 的子集的根节点的 CID，表示简单字节范围请求所需的最小块集。
        fixtureCid = "bafybeicgsg3lwyn3yl75lw7sn4zhyj5dxtb7wfxwscpq6yzippetmr2w3y"
    )

    // 启动节点
    h := harness.NewT(t)
    node := h.NewNode().Init("--empty-repo", "--profile=test").StartDaemon("--offline")
    client := node.GatewayClient()  // 获取节点的网关客户端
    // 导入测试数据
    r, err := os.Open("./fixtures/TestGatewayMultiRange.car")  // 打开测试文件
    assert.NoError(t, err)  // 断言没有错误发生
    defer r.Close()  // 延迟关闭文件
    err = node.IPFSDagImport(r, fixtureCid)  // 使用测试数据导入 IPFS DAG
    assert.NoError(t, err)  // 断言没有错误发生

    t.Run("Succeeds Fetching Range", func(t *testing.T) {
        t.Parallel()

        resp := client.Get(fmt.Sprintf("/ipfs/%s", fileCid), func(r *http.Request) {
            r.Header.Set("Range", "bytes=1276-1279")  // 设置请求头的 Range 字段
        })
        assert.Equal(t, http.StatusPartialContent, resp.StatusCode)  // 断言响应状态码为部分内容
        assert.Equal(t, "bytes 1276-1279/109266405", resp.Headers.Get("Content-Range"))  // 断言响应头的 Content-Range 字段
        assert.Equal(t, "iana", resp.Body)  // 断言响应体内容为 "iana"
    })

    // ... (以下两个测试用例同上，只是请求头的 Range 字段不同)
# 闭合前面的函数定义
```