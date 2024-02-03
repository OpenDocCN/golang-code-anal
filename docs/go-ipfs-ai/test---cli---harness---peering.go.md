# `kubo\test\cli\harness\peering.go`

```go
package harness

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "math/rand"  // 导入 math/rand 包，用于生成随机数
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/config"  // 导入外部包，用于配置
)

type Peering struct {
    From int  // 定义 Peering 结构体的属性 From，表示起始节点
    To   int  // 定义 Peering 结构体的属性 To，表示目标节点
}

func NewRandPort() int {
    n := rand.Int()  // 生成一个随机数
    return 3000 + (n % 1000)  // 返回一个随机端口号
}

func CreatePeerNodes(t *testing.T, n int, peerings []Peering) (*Harness, Nodes) {
    h := NewT(t)  // 创建测试对象
    nodes := h.NewNodes(n).Init()  // 初始化节点
    nodes.ForEachPar(func(node *Node) {  // 并行遍历节点
        node.UpdateConfig(func(cfg *config.Config) {  // 更新节点配置
            cfg.Routing.Type = config.NewOptionalString("none")  // 设置路由类型为 none
            cfg.Addresses.Swarm = []string{fmt.Sprintf("/ip4/127.0.0.1/tcp/%d", NewRandPort())}  // 设置 Swarm 地址
        })
    })

    for _, peering := range peerings {  // 遍历 peerings 切片
        nodes[peering.From].PeerWith(nodes[peering.To])  // 通过节点之间建立对等连接
    }

    return h, nodes  // 返回测试对象和节点
}
```