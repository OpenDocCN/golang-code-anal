# `kubo\test\cli\swarm_test.go`

```
package cli

import (
    "encoding/json"  // 导入 JSON 编解码包
    "fmt"  // 导入格式化输出包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试辅助包

    "github.com/stretchr/testify/assert"  // 导入断言包
)

// TODO: Migrate the rest of the sharness swarm test.
func TestSwarm(t *testing.T) {
    type identifyType struct {  // 定义 identifyType 结构体
        ID           string
        PublicKey    string
        Addresses    []string
        AgentVersion string
        Protocols    []string
    }
    type peer struct {  // 定义 peer 结构体
        Identify identifyType
    }
    type expectedOutputType struct {  // 定义 expectedOutputType 结构体
        Peers []peer
    }

    t.Parallel()  // 并行执行测试

    t.Run("ipfs swarm peers returns empty peers when a node is not connected to any peers", func(t *testing.T) {
        t.Parallel()  // 并行执行子测试
        node := harness.NewT(t).NewNode().Init().StartDaemon()  // 创建测试节点并初始化
        res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")  // 运行 IPFS 命令获取 swarm peers 信息
        var output expectedOutputType  // 定义变量 output 为 expectedOutputType 类型
        err := json.Unmarshal(res.Stdout.Bytes(), &output)  // 解析命令输出为 JSON 格式
        assert.NoError(t, err)  // 断言解析过程无错误
        assert.Equal(t, 0, len(output.Peers))  // 断言输出的 Peers 数量为 0
    })
}
    # 使用测试框架运行测试，验证IPFS节点的swarm peers功能是否输出预期的关于连接对等节点的识别信息
    t.Run("ipfs swarm peers with flag identify outputs expected identify information about connected peers", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建新的IPFS节点并初始化，启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 创建另一个新的IPFS节点并初始化，启动守护进程
        otherNode := harness.NewT(t).NewNode().Init().StartDaemon()
        # 将当前节点连接到另一个节点
        node.Connect(otherNode)

        # 运行IPFS命令，获取输出结果
        res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")
        # 定义变量output，用于存储预期的输出类型
        var output expectedOutputType
        # 将命令输出解析为JSON格式，存储到output变量中
        err := json.Unmarshal(res.Stdout.Bytes(), &output)
        # 断言解析过程中是否出现错误
        assert.NoError(t, err)
        # 获取实际的节点ID
        actualID := output.Peers[0].Identify.ID
        # 获取实际的公钥
        actualPublicKey := output.Peers[0].Identify.PublicKey
        # 获取实际的代理版本
        actualAgentVersion := output.Peers[0].Identify.AgentVersion
        # 获取实际的地址
        actualAdresses := output.Peers[0].Identify.Addresses
        # 获取实际的协议
        actualProtocols := output.Peers[0].Identify.Protocols

        # 获取预期的节点ID
        expectedID := otherNode.PeerID().String()
        # 获取预期的地址列表
        expectedAddresses := []string{fmt.Sprintf("%s/p2p/%s", otherNode.SwarmAddrs()[0], actualID)}

        # 断言实际的节点ID与预期的节点ID是否相等
        assert.Equal(t, actualID, expectedID)
        # 断言实际的公钥是否不为空
        assert.NotNil(t, actualPublicKey)
        # 断言实际的代理版本是否不为空
        assert.NotNil(t, actualAgentVersion)
        # 断言实际的地址列表长度为1
        assert.Len(t, actualAdresses, 1)
        # 断言实际的地址与预期的地址是否相等
        assert.Equal(t, expectedAddresses[0], actualAdresses[0])
        # 断言实际的协议列表长度大于0
        assert.Greater(t, len(actualProtocols), 0)
    })
    # 使用测试框架运行一个测试，验证ipfs swarm peers命令与标识标志一起输出的Identify字段，其数据与调用ipfs id命令的对等节点的数据匹配
    t.Run("ipfs swarm peers with flag identify outputs Identify field with data that matches calling ipfs id on a peer", func(t *testing.T) {
        # 并行执行测试
        t.Parallel()
        # 创建一个新的节点，并初始化并启动守护进程
        node := harness.NewT(t).NewNode().Init().StartDaemon()
        # 创建另一个新的节点，并初始化并启动守护进程
        otherNode := harness.NewT(t).NewNode().Init().StartDaemon()
        # 将当前节点连接到另一个节点
        node.Connect(otherNode)

        # 获取另一个节点的ID响应，格式为JSON
        otherNodeIDResponse := otherNode.RunIPFS("id", "--enc=json")
        var otherNodeIDOutput identifyType
        # 将ID响应解析为identifyType结构
        err := json.Unmarshal(otherNodeIDResponse.Stdout.Bytes(), &otherNodeIDOutput)
        assert.NoError(t, err)
        # 运行ipfs swarm peers命令，格式为JSON，并包含标识信息
        res := node.RunIPFS("swarm", "peers", "--enc=json", "--identify")

        var output expectedOutputType
        # 将命令输出解析为expectedOutputType结构
        err = json.Unmarshal(res.Stdout.Bytes(), &output)
        assert.NoError(t, err)
        # 获取输出中的Identify字段
        outputIdentify := output.Peers[0].Identify

        # 验证输出中的ID与另一个节点的ID相等
        assert.Equal(t, outputIdentify.ID, otherNodeIDOutput.ID)
        # 验证输出中的PublicKey与另一个节点的PublicKey相等
        assert.Equal(t, outputIdentify.PublicKey, otherNodeIDOutput.PublicKey)
        # 验证输出中的AgentVersion与另一个节点的AgentVersion相等
        assert.Equal(t, outputIdentify.AgentVersion, otherNodeIDOutput.AgentVersion)
        # 验证输出中的Addresses与另一个节点的Addresses相匹配
        assert.ElementsMatch(t, outputIdentify.Addresses, otherNodeIDOutput.Addresses)
        # 验证输出中的Protocols与另一个节点的Protocols相匹配
        assert.ElementsMatch(t, outputIdentify.Protocols, otherNodeIDOutput.Protocols)
    })
# 闭合前面的函数定义
```