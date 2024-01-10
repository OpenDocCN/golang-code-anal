# `kubo\core\mock\mock.go`

```
package coremock

import (
    "context" // 引入上下文包，用于处理请求的取消、超时等
    "fmt" // 引入格式化包，用于格式化输出

    "io" // 引入输入输出包，用于处理输入输出流

    libp2p2 "github.com/ipfs/kubo/core/node/libp2p" // 引入 libp2p2 包，用于构建 libp2p 节点

    "github.com/ipfs/kubo/commands" // 引入命令包，用于处理命令相关操作
    "github.com/ipfs/kubo/core" // 引入核心包，用于构建 IPFS 节点
    "github.com/ipfs/kubo/repo" // 引入仓库包，用于处理 IPFS 仓库相关操作

    "github.com/ipfs/go-datastore" // 引入数据存储包，用于处理数据存储
    syncds "github.com/ipfs/go-datastore/sync" // 引入同步数据存储包，用于处理同步数据存储
    config "github.com/ipfs/kubo/config" // 引入配置包，用于处理配置相关操作

    "github.com/libp2p/go-libp2p" // 引入 libp2p 包，用于构建 libp2p 节点
    testutil "github.com/libp2p/go-libp2p-testing/net" // 引入测试工具包，用于测试网络相关操作
    "github.com/libp2p/go-libp2p/core/host" // 引入主机包，用于构建 libp2p 主机
    "github.com/libp2p/go-libp2p/core/peer" // 引入节点包，用于构建 libp2p 节点
    pstore "github.com/libp2p/go-libp2p/core/peerstore" // 引入节点存储包，用于处理节点存储
    mocknet "github.com/libp2p/go-libp2p/p2p/net/mock" // 引入模拟网络包，用于模拟网络操作
)

// NewMockNode constructs an IpfsNode for use in tests.
func NewMockNode() (*core.IpfsNode, error) {
    // effectively offline, only peer in its network
    // 构建一个用于测试的 IpfsNode 节点
    return core.NewNode(context.Background(), &core.BuildCfg{
        Online: true, // 设置节点在线状态为 true
        Host:   MockHostOption(mocknet.New()), // 设置节点的主机选项为模拟网络的主机选项
    })
}

func MockHostOption(mn mocknet.Mocknet) libp2p2.HostOption {
    return func(id peer.ID, ps pstore.Peerstore, opts ...libp2p.Option) (host.Host, error) {
        var cfg libp2p.Config // 定义 libp2p 配置对象
        if err := cfg.Apply(opts...); err != nil { // 如果应用配置出错
            return nil, err // 返回错误
        }

        // The mocknet does not use the provided libp2p.Option. This options include
        // the listening addresses we want our peer listening on. Therefore, we have
        // to manually parse the configuration and add them here.
        // 模拟网络不使用提供的 libp2p.Option。这些选项包括我们希望对等体侦听的侦听地址。因此，我们必须手动解析配置并在此处添加它们。
        ps.AddAddrs(id, cfg.ListenAddrs, pstore.PermanentAddrTTL) // 将监听地址添加到节点存储中
        return mn.AddPeerWithPeerstore(id, ps) // 使用节点存储添加对等体
    }
}

func MockCmdsCtx() (commands.Context, error) {
    // Generate Identity
    ident, err := testutil.RandIdentity() // 生成身份标识
    if err != nil { // 如果生成身份标识出错
        return commands.Context{}, err // 返回错误
    }
    p := ident.ID() // 获取身份标识的 ID

    conf := config.Config{ // 构建配置对象
        Identity: config.Identity{ // 设置身份标识
            PeerID: p.String(), // 设置对等体 ID
        },
    }

    r := &repo.Mock{ // 创建模拟仓库对象
        D: syncds.MutexWrap(datastore.NewMapDatastore()), // 设置数据存储
        C: conf, // 设置配置
    }
    // 使用 core.NewNode 方法创建一个新的节点，传入上下文和构建配置
    node, err := core.NewNode(context.Background(), &core.BuildCfg{
        Repo: r,
    })
    // 如果创建节点时发生错误，则返回空的上下文和错误信息
    if err != nil {
        return commands.Context{}, err
    }

    // 返回一个包含配置根路径和构造节点函数的上下文
    return commands.Context{
        ConfigRoot: "/tmp/.mockipfsconfig",
        ConstructNode: func() (*core.IpfsNode, error) {
            return node, nil
        },
    }, nil
// 模拟公共节点，接受上下文和模拟网络作为参数，返回IPFS节点和错误
func MockPublicNode(ctx context.Context, mn mocknet.Mocknet) (*core.IpfsNode, error) {
    // 创建一个互斥锁封装的MapDatastore
    ds := syncds.MutexWrap(datastore.NewMapDatastore())
    // 初始化配置，将日志输出到io.Discard，设置初始大小为2048
    cfg, err := config.Init(io.Discard, 2048)
    if err != nil {
        return nil, err
    }
    // 获取模拟网络中的对等节点数量
    count := len(mn.Peers())
    // 设置Swarm地址为模拟网络中对等节点数量的IP地址和端口
    cfg.Addresses.Swarm = []string{
        fmt.Sprintf("/ip4/18.0.%d.%d/tcp/4001", count>>16, count&0xFF),
    }
    // 设置数据存储为默认配置
    cfg.Datastore = config.Datastore{}
    // 创建并返回一个新的IPFS节点
    return core.NewNode(ctx, &core.BuildCfg{
        Online:  true,
        Routing: libp2p2.DHTServerOption,
        Repo: &repo.Mock{
            C: *cfg,
            D: ds,
        },
        Host: MockHostOption(mn),
    })
}
```