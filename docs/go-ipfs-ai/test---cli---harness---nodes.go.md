# `kubo\test\cli\harness\nodes.go`

```go
package harness

import (
    "sync"  // 导入 sync 包，用于实现并发操作

    . "github.com/ipfs/kubo/test/cli/testutils"  // 导入 testutils 包，并将其导出的所有符号都导入当前包的命名空间
    "github.com/multiformats/go-multiaddr"  // 导入 go-multiaddr 包
    "golang.org/x/sync/errgroup"  // 导入 errgroup 包
)

// Nodes is a collection of Kubo nodes along with operations on groups of nodes.
type Nodes []*Node  // 定义 Nodes 类型为 Node 指针的切片

func (n Nodes) Init(args ...string) Nodes {  // 定义 Nodes 类型的方法 Init，接收可变数量的字符串参数，返回 Nodes 类型
    ForEachPar(n, func(node *Node) { node.Init(args...) })  // 对 Nodes 切片中的每个节点并行执行 Init 方法
    return n  // 返回执行后的 Nodes 切片
}

func (n Nodes) ForEachPar(f func(*Node)) {  // 定义 Nodes 类型的方法 ForEachPar，接收一个函数参数
    group := &errgroup.Group{}  // 创建一个 errgroup.Group 对象
    for _, node := range n {  // 遍历 Nodes 切片
        node := node  // 创建局部变量 node，避免闭包中的竞争条件
        group.Go(func() error {  // 向 errgroup.Group 对象中添加一个并发执行的函数
            f(node)  // 调用传入的函数参数
            return nil  // 返回 nil
        })
    }
    err := group.Wait()  // 等待所有并发函数执行完毕
    if err != nil {  // 如果有错误发生
        panic(err)  // 抛出异常
    }
}

func (n Nodes) Connect() Nodes {  // 定义 Nodes 类型的方法 Connect，返回 Nodes 类型
    wg := sync.WaitGroup{}  // 创建一个 WaitGroup 对象
    for i, node := range n {  // 遍历 Nodes 切片
        for j, otherNode := range n {  // 再次遍历 Nodes 切片
            if i == j {  // 如果索引相同
                continue  // 继续下一次循环
            }
            node := node  // 创建局部变量 node，避免闭包中的竞争条件
            otherNode := otherNode  // 创建局部变量 otherNode，避免闭包中的竞争条件
            wg.Add(1)  // WaitGroup 加一
            go func() {  // 启动一个并发函数
                defer wg.Done()  // 在函数结束时调用 WaitGroup 完成方法
                node.Connect(otherNode)  // 调用节点的 Connect 方法
            }()
        }
    }
    wg.Wait()  // 等待所有并发函数执行完毕
    for _, node := range n {  // 遍历 Nodes 切片
        firstPeer := node.Peers()[0]  // 获取节点的第一个对等节点
        if _, err := firstPeer.ValueForProtocol(multiaddr.P_P2P); err != nil {  // 获取对等节点的 P2P 协议值
            log.Panicf("unexpected state for node %d with peer ID %s: %s", node.ID, node.PeerID(), err)  // 如果出现错误，抛出异常
        }
    }
    return n  // 返回 Nodes 切片
}

func (n Nodes) StartDaemons(args ...string) Nodes {  // 定义 Nodes 类型的方法 StartDaemons，接收可变数量的字符串参数，返回 Nodes 类型
    ForEachPar(n, func(node *Node) { node.StartDaemon(args...) })  // 对 Nodes 切片中的每个节点并行执行 StartDaemon 方法
    return n  // 返回执行后的 Nodes 切片
}

func (n Nodes) StopDaemons() Nodes {  // 定义 Nodes 类型的方法 StopDaemons，返回 Nodes 类型
    ForEachPar(n, func(node *Node) { node.StopDaemon() })  // 对 Nodes 切片中的每个节点并行执行 StopDaemon 方法
    return n  // 返回执行后的 Nodes 切片
}
```