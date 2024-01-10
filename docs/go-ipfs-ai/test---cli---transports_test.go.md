# `kubo\test\cli\transports_test.go`

```
package cli

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "os"  // 导入 os 包，用于操作系统功能
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/config"  // 导入 config 包，用于处理配置
    "github.com/ipfs/kubo/test/cli/harness"  // 导入 harness 包，用于测试工具
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入 testutils 包，用于测试工具
    "github.com/stretchr/testify/assert"  // 导入 assert 包，用于断言
    "github.com/stretchr/testify/require"  // 导入 require 包，用于测试需求
)

func TestTransports(t *testing.T) {
    disableRouting := func(nodes harness.Nodes) {  // 定义一个函数 disableRouting，接收 harness.Nodes 参数
        nodes.ForEachPar(func(n *harness.Node) {  // 对 nodes 中的每个节点执行并行操作
            n.UpdateConfig(func(cfg *config.Config) {  // 更新节点的配置
                cfg.Routing.Type = config.NewOptionalString("none")  // 设置路由类型为 "none"
                cfg.Bootstrap = nil  // 清空引导节点
            })
        })
    }
    checkSingleFile := func(nodes harness.Nodes) {  // 定义一个函数 checkSingleFile，接收 harness.Nodes 参数
        s := testutils.RandomStr(100)  // 生成一个长度为 100 的随机字符串
        hash := nodes[0].IPFSAddStr(s)  // 将字符串添加到 IPFS，并获取其哈希值
        nodes.ForEachPar(func(n *harness.Node) {  // 对 nodes 中的每个节点执行并行操作
            val := n.IPFS("cat", hash).Stdout.String()  // 从 IPFS 中获取哈希值对应的内容
            assert.Equal(t, s, val)  // 断言获取的内容与原始字符串相等
        })
    }
    checkRandomDir := func(nodes harness.Nodes) {  // 定义一个函数 checkRandomDir，接收 harness.Nodes 参数
        randDir := filepath.Join(nodes[0].Dir, "foobar")  // 生成一个随机目录路径
        require.NoError(t, os.Mkdir(randDir, 0o777))  // 创建随机目录，并检查是否出错
        rf := testutils.NewRandFiles()  // 创建一个新的随机文件对象
        rf.FanoutDirs = 3  // 设置随机文件对象的目录分支数
        rf.FanoutFiles = 6  // 设置随机文件对象的文件分支数
        require.NoError(t, rf.WriteRandomFiles(randDir, 4))  // 在随机目录中写入随机文件，并检查是否出错

        hash := nodes[1].IPFS("add", "-r", "-Q", randDir).Stdout.Trimmed()  // 将随机目录添加到 IPFS，并获取其哈希值
        nodes.ForEachPar(func(n *harness.Node) {  // 对 nodes 中的每个节点执行并行操作
            res := n.RunIPFS("refs", "-r", hash)  // 运行 IPFS 命令，查找哈希值的引用
            assert.Equal(t, 0, res.ExitCode())  // 断言命令执行结果的退出码为 0
        })
    }

    runTests := func(nodes harness.Nodes) {  // 定义一个函数 runTests，接收 harness.Nodes 参数
        checkSingleFile(nodes)  // 执行 checkSingleFile 函数
        checkRandomDir(nodes)  // 执行 checkRandomDir 函数
    }
}
    # 定义一个名为tcpNodes的函数，用于创建并配置TCP节点
    tcpNodes := func(t *testing.T) harness.Nodes {
        # 使用harness.NewT(t)创建测试实例，然后创建两个节点并初始化
        nodes := harness.NewT(t).NewNodes(2).Init()
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置，设置Swarm地址为本地TCP地址，关闭其他网络传输协议
            n.UpdateConfig(func(cfg *config.Config) {
                cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/tcp/0"}
                cfg.Swarm.Transports.Network.QUIC = config.False
                cfg.Swarm.Transports.Network.Relay = config.False
                cfg.Swarm.Transports.Network.WebTransport = config.False
                cfg.Swarm.Transports.Network.WebRTCDirect = config.False
                cfg.Swarm.Transports.Network.Websocket = config.False
            })
        })
        # 禁用节点之间的路由
        disableRouting(nodes)
        # 返回配置好的节点
        return nodes
    }
    
    # 运行名为"tcp"的测试
    t.Run("tcp", func(t *testing.T) {
        t.Parallel()
        # 获取配置好的TCP节点，启动守护进程并连接节点
        nodes := tcpNodes(t).StartDaemons().Connect()
        # 运行测试
        runTests(nodes)
    })
    
    # 运行名为"tcp with NOISE"的测试
    t.Run("tcp with NOISE", func(t *testing.T) {
        t.Parallel()
        # 获取配置好的TCP节点
        nodes := tcpNodes(t)
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置，禁用TLS安全传输
            n.UpdateConfig(func(cfg *config.Config) {
                cfg.Swarm.Transports.Security.TLS = config.Disabled
            })
        })
        # 启动守护进程并连接节点
        nodes.StartDaemons().Connect()
        # 运行测试
        runTests(nodes)
    })
    
    # 运行名为"QUIC"的测试
    t.Run("QUIC", func(t *testing.T) {
        t.Parallel()
        # 使用harness.NewT(t)创建测试实例，然后创建五个节点并初始化
        nodes := harness.NewT(t).NewNodes(5).Init()
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置，设置Swarm地址为本地QUIC地址，关闭TCP传输
            cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/udp/0/quic-v1"}
            cfg.Swarm.Transports.Network.QUIC = config.True
            cfg.Swarm.Transports.Network.TCP = config.False
        })
        # 禁用节点之间的路由
        disableRouting(nodes)
        # 启动守护进程并连接节点
        nodes.StartDaemons().Connect()
        # 运行测试
        runTests(nodes)
    }
    # 运行名为 "QUIC" 的测试函数
    t.Run("QUIC", func(t *testing.T) {
        # 标记该测试函数可以并行执行
        t.Parallel()
        # 创建5个节点并初始化
        nodes := harness.NewT(t).NewNodes(5).Init()
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置信息
            n.UpdateConfig(func(cfg *config.Config) {
                # 设置节点的Swarm地址为指定的QUIC地址
                cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/udp/0/quic-v1/webtransport"}
                # 启用QUIC传输协议
                cfg.Swarm.Transports.Network.QUIC = config.True
                # 启用WebTransport传输协议
                cfg.Swarm.Transports.Network.WebTransport = config.True
            })
        })
        # 禁用节点之间的路由
        disableRouting(nodes)
        # 启动节点的守护进程并连接节点
        nodes.StartDaemons().Connect()
        # 运行测试函数
        runTests(nodes)
    })

    # 运行名为 "QUIC connects with non-dialable transports" 的测试函数
    t.Run("QUIC connects with non-dialable transports", func(t *testing.T) {
        # 该测试函数目标特定的Kubo内部，可能会在后续更改。这里检查我们是否能够宣布一个我们不监听的地址，然后能够通过另一个可用的地址进行连接。
        t.Parallel()
        # 创建5个节点并初始化
        nodes := harness.NewT(t).NewNodes(5).Init()
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置信息
            n.UpdateConfig(func(cfg *config.Config) {
                # 生成一个随机端口用于宣布
                port := harness.NewRandPort()
                quicAddr := fmt.Sprintf("/ip4/127.0.0.1/udp/%d/quic-v1", port)
                # 设置节点的Swarm地址和宣布地址
                cfg.Addresses.Swarm = []string{quicAddr}
                cfg.Addresses.Announce = []string{quicAddr, quicAddr + "/webtransport"}
            })
        })
        # 禁用节点之间的路由
        disableRouting(nodes)
        # 启动节点的守护进程并连接节点
        nodes.StartDaemons().Connect()
        # 运行测试函数
        runTests(nodes)
    })
    # 运行名为 "WebRTC Direct" 的测试
    t.Run("WebRTC Direct", func(t *testing.T) {
        # 启用并行测试
        t.Parallel()
        # 创建包含5个节点的测试环境
        nodes := harness.NewT(t).NewNodes(5).Init()
        # 对每个节点并行执行以下操作
        nodes.ForEachPar(func(n *harness.Node) {
            # 更新节点的配置信息
            n.UpdateConfig(func(cfg *config.Config) {
                # 设置节点的Swarm地址为 "/ip4/127.0.0.1/udp/0/webrtc-direct"
                cfg.Addresses.Swarm = []string{"/ip4/127.0.0.1/udp/0/webrtc-direct"}
                # 禁用TCP传输
                cfg.Swarm.Transports.Network.TCP = config.False
                # 禁用QUIC传输
                cfg.Swarm.Transports.Network.QUIC = config.False
                # 禁用Web传输
                cfg.Swarm.Transports.Network.WebTransport = config.False
                # 启用WebRTC直连传输
                cfg.Swarm.Transports.Network.WebRTCDirect = config.True
            })
        })
        # 禁用节点之间的路由
        disableRouting(nodes)
        # 启动节点的守护进程并连接节点
        nodes.StartDaemons().Connect()
        # 运行测试
        runTests(nodes)
    })
# 闭合前面的函数定义
```