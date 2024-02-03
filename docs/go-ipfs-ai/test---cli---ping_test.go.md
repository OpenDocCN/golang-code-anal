# `kubo\test\cli\ping_test.go`

```go
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "strings"  // 导入 strings 包，用于处理字符串
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/test/cli/harness"  // 导入自定义包 harness，用于测试
    "github.com/stretchr/testify/assert"  // 导入 testify 包中的 assert，用于断言
)

func TestPing(t *testing.T) {  // 定义测试函数 TestPing
    t.Parallel()  // 声明测试函数可以并行执行

    t.Run("other", func(t *testing.T) {  // 运行子测试 "other"
        t.Parallel()  // 声明子测试可以并行执行
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()  // 创建两个节点并初始化，启动守护进程并连接
        node1 := nodes[0]  // 获取第一个节点
        node2 := nodes[1]  // 获取第二个节点

        node1.IPFS("ping", "-n", "2", "--", node2.PeerID().String())  // 节点1 ping 节点2
        node2.IPFS("ping", "-n", "2", "--", node1.PeerID().String())  // 节点2 ping 节点1
    })

    t.Run("ping unreachable peer", func(t *testing.T) {  // 运行子测试 "ping unreachable peer"
        t.Parallel()  // 声明子测试可以并行执行
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()  // 创建两个节点并初始化，启动守护进程并连接
        node1 := nodes[0]  // 获取第一个节点

        badPeer := "QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJx"  // 定义一个无法到达的对等节点
        res := node1.RunIPFS("ping", "-n", "2", "--", badPeer)  // 节点1 ping 无法到达的对等节点
        assert.Contains(t, res.Stdout.String(), fmt.Sprintf("Looking up peer %s", badPeer))  // 断言标准输出中包含查找对等节点的信息
        msg := res.Stderr.String()  // 获取标准错误输出
        assert.Truef(t, strings.HasPrefix(msg, "Error:"), "should fail got this instead: %q", msg)  // 断言标准错误输出以 "Error:" 开头，否则输出自定义错误信息
    })

    t.Run("self", func(t *testing.T) {  // 运行子测试 "self"
        t.Parallel()  // 声明子测试可以并行执行
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons()  // 创建两个节点并初始化，启动守护进程
        node1 := nodes[0]  // 获取第一个节点
        node2 := nodes[1]  // 获取第二个节点

        res := node1.RunIPFS("ping", "-n", "2", "--", node1.PeerID().String())  // 节点1 ping 自己
        assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())  // 断言命令的退出码为1
        assert.Contains(t, res.Stderr.String(), "can't ping self")  // 断言标准错误输出中包含 "can't ping self"

        res = node2.RunIPFS("ping", "-n", "2", "--", node2.PeerID().String())  // 节点2 ping 自己
        assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())  // 断言命令的退出码为1
        assert.Contains(t, res.Stderr.String(), "can't ping self")  // 断言标准错误输出中包含 "can't ping self"
    })
}
    # 运行测试用例 "0"
    t.Run("0", func(t *testing.T) {
        # 标记该测试用例可以并行执行
        t.Parallel()
        # 创建两个节点，并初始化、启动守护进程、连接节点
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()
        # 获取第一个节点
        node1 := nodes[0]
        # 获取第二个节点
        node2 := nodes[1]

        # 在节点1上执行 IPFS 命令 "ping"，指定 ping 次数为 0，目标节点为 node2 的 PeerID
        res := node1.RunIPFS("ping", "-n", "0", "--", node2.PeerID().String())
        # 断言命令执行结果的退出码为 1
        assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())
        # 断言标准错误输出包含指定的字符串 "ping count must be greater than 0"
        assert.Contains(t, res.Stderr.String(), "ping count must be greater than 0")
    })

    # 运行测试用例 "offline"
    t.Run("offline", func(t *testing.T) {
        # 标记该测试用例可以并行执行
        t.Parallel()
        # 创建两个节点，并初始化、启动守护进程、连接节点
        nodes := harness.NewT(t).NewNodes(2).Init().StartDaemons().Connect()
        # 获取第一个节点
        node1 := nodes[0]
        # 获取第二个节点
        node2 := nodes[1]

        # 停止节点2的守护进程
        node2.StopDaemon()

        # 在节点1上执行 IPFS 命令 "ping"，指定 ping 次数为 2，目标节点为 node2 的 PeerID
        res := node1.RunIPFS("ping", "-n", "2", "--", node2.PeerID().String())
        # 断言命令执行结果的退出码为 1
        assert.Equal(t, 1, res.Cmd.ProcessState.ExitCode())
        # 断言标准错误输出包含指定的字符串 "ping failed"
        assert.Contains(t, res.Stderr.String(), "ping failed")
    })
# 闭合前面的函数定义
```