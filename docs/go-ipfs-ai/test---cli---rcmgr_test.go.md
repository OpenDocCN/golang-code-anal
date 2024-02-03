# `kubo\test\cli\rcmgr_test.go`

```go
package cli

import (
    "encoding/json"  // 导入 JSON 编解码包
    "testing"  // 导入测试包

    "github.com/ipfs/kubo/config"  // 导入配置包
    "github.com/ipfs/kubo/core/node/libp2p"  // 导入 libp2p 节点核心包
    "github.com/ipfs/kubo/test/cli/harness"  // 导入测试工具包
    "github.com/ipfs/kubo/test/cli/testutils"  // 导入测试工具包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入 libp2p 核心包
    "github.com/libp2p/go-libp2p/core/protocol"  // 导入 libp2p 协议包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入资源管理器包
    "github.com/stretchr/testify/assert"  // 导入测试断言包
    "github.com/stretchr/testify/require"  // 导入测试断言包
)

func TestRcmgr(t *testing.T) {
    t.Parallel()  // 并行执行测试

    t.Run("Resource manager disabled", func(t *testing.T) {
        t.Parallel()  // 并行执行测试
        node := harness.NewT(t).NewNode().Init()  // 创建测试节点
        node.UpdateConfig(func(cfg *config.Config) {  // 更新节点配置
            cfg.Swarm.ResourceMgr.Enabled = config.False  // 禁用资源管理器
        })

        node.StartDaemon()  // 启动守护进程

        t.Run("swarm resources should fail", func(t *testing.T) {
            res := node.RunIPFS("swarm", "resources")  // 运行 IPFS 命令
            assert.Equal(t, 1, res.ExitCode())  // 断言退出码为 1
            assert.Contains(t, res.Stderr.String(), "missing ResourceMgr")  // 断言标准错误输出包含指定字符串
        })
    })

    t.Run("Node with resource manager disabled", func(t *testing.T) {
        t.Parallel()  // 并行执行测试
        node := harness.NewT(t).NewNode().Init()  // 创建测试节点
        node.UpdateConfig(func(cfg *config.Config) {  // 更新节点配置
            cfg.Swarm.ResourceMgr.Enabled = config.False  // 禁用资源管理器
        })
        node.StartDaemon()  // 启动守护进程

        t.Run("swarm resources should fail", func(t *testing.T) {
            res := node.RunIPFS("swarm", "resources")  // 运行 IPFS 命令
            assert.Equal(t, 1, res.ExitCode())  // 断言退出码为 1
            assert.Contains(t, res.Stderr.String(), "missing ResourceMgr")  // 断言标准错误输出包含指定字符串
        })
    })
}
    # 运行测试，测试非常高的连接管理器高水位
    t.Run("Very high connmgr highwater", func(t *testing.T) {
        t.Parallel()
        # 创建新的测试节点
        node := harness.NewT(t).NewNode().Init()
        # 更新节点配置，设置连接管理器高水位为1000
        node.UpdateConfig(func(cfg *config.Config) {
            cfg.Swarm.ConnMgr.HighWater = config.NewOptionalInteger(1000)
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行IPFS命令，获取swarm资源的JSON格式输出
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 断言结果的退出码为0
        require.Equal(t, 0, res.ExitCode())
        # 解析资源限制
        limits := unmarshalLimits(t, res.Stdout.Bytes())

        # 将系统资源限制转换为资源限制对象
        rl := limits.System.ToResourceLimits()
        s := rl.Build(rcmgr.BaseLimit{})
        # 断言入站连接数大于或等于2000
        assert.GreaterOrEqual(t, s.ConnsInbound, 2000)
        # 断言入站流数大于或等于2000
        assert.GreaterOrEqual(t, s.StreamsInbound, 2000)
    })

    # 运行测试，测试无限制的系统入站连接
    t.Run("smoke test unlimited System inbounds", func(t *testing.T) {
        t.Parallel()
        # 创建新的测试节点
        node := harness.NewT(t).NewNode().Init()
        # 更新用户提供的资源管理器覆盖配置，设置系统入站流和连接为无限制
        node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
            overrides.System.StreamsInbound = rcmgr.Unlimited
            overrides.System.ConnsInbound = rcmgr.Unlimited
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行IPFS命令，获取swarm资源的JSON格式输出
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 解析资源限制
        limits := unmarshalLimits(t, res.Stdout.Bytes())

        # 断言系统入站连接为无限制
        assert.Equal(t, rcmgr.Unlimited, limits.System.ConnsInbound)
        # 断言系统入站流为无限制
        assert.Equal(t, rcmgr.Unlimited, limits.System.StreamsInbound)
    })

    # 运行测试，测试瞬态范围
    t.Run("smoke test transient scope", func(t *testing.T) {
        t.Parallel()
        # 创建新的测试节点
        node := harness.NewT(t).NewNode().Init()
        # 更新用户提供的资源管理器覆盖配置，设置瞬态内存为88888
        node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
            overrides.Transient.Memory = 88888
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行IPFS命令，获取swarm资源的JSON格式输出
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 解析资源限制
        limits := unmarshalLimits(t, res.Stdout.Bytes())
        # 断言瞬态内存为88888
        assert.Equal(t, rcmgr.LimitVal64(88888), limits.Transient.Memory)
    })
    # 运行服务范围的烟雾测试
    t.Run("smoke test service scope", func(t *testing.T) {
        # 设置测试并行运行
        t.Parallel()
        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新用户提供的资源管理器覆盖
        node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
            overrides.Service = map[string]rcmgr.ResourceLimits{"foo": {Memory: 77777}}
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行 IPFS 命令，获取资源限制信息
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 反序列化资源限制信息
        limits := unmarshalLimits(t, res.Stdout.Bytes())
        # 断言服务内存限制是否符合预期
        assert.Equal(t, rcmgr.LimitVal64(77777), limits.Services["foo"].Memory)
    })

    # 运行协议范围的烟雾测试
    t.Run("smoke test protocol scope", func(t *testing.T) {
        # 设置测试并行运行
        t.Parallel()
        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新用户提供的资源管理器覆盖
        node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
            overrides.Protocol = map[protocol.ID]rcmgr.ResourceLimits{"foo": {Memory: 66666}}
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行 IPFS 命令，获取资源限制信息
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 反序列化资源限制信息
        limits := unmarshalLimits(t, res.Stdout.Bytes())
        # 断言协议内存限制是否符合预期
        assert.Equal(t, rcmgr.LimitVal64(66666), limits.Protocols["foo"].Memory)
    })

    # 运行节点范围的烟雾测试
    t.Run("smoke test peer scope", func(t *testing.T) {
        # 设置测试并行运行
        t.Parallel()
        # 解码有效的对等节点 ID
        validPeerID, err := peer.Decode("QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN")
        assert.NoError(t, err)
        # 创建新的节点并初始化
        node := harness.NewT(t).NewNode().Init()
        # 更新用户提供的资源管理器覆盖
        node.UpdateUserSuppliedResourceManagerOverrides(func(overrides *rcmgr.PartialLimitConfig) {
            overrides.Peer = map[peer.ID]rcmgr.ResourceLimits{validPeerID: {Memory: 55555}}
        })
        # 启动守护进程
        node.StartDaemon()

        # 运行 IPFS 命令，获取资源限制信息
        res := node.RunIPFS("swarm", "resources", "--enc=json")
        # 反序列化资源限制信息
        limits := unmarshalLimits(t, res.Stdout.Bytes())
        # 断言对等节点内存限制是否符合预期
        assert.Equal(t, rcmgr.LimitVal64(55555), limits.Peers[validPeerID].Memory)
    })
# 结束当前函数的定义
}

# 解析传入的字节流数据，将其反序列化为 libp2p.LimitsConfigAndUsage 结构体指针
func unmarshalLimits(t *testing.T, b []byte) *libp2p.LimitsConfigAndUsage {
    # 创建一个 libp2p.LimitsConfigAndUsage 结构体指针
    limits := &libp2p.LimitsConfigAndUsage{}
    # 使用 JSON 解析器将字节流数据解析为 limits 结构体
    err := json.Unmarshal(b, limits)
    # 检查是否有错误发生，如果有则抛出测试失败的错误
    require.NoError(t, err)
    # 返回解析后的 limits 结构体指针
    return limits
}
```