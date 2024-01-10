# `kubo\core\node\libp2p\routing.go`

```
package libp2p

import (
    "context" // 上下文包，用于控制函数调用的超时、取消和截止
    "fmt" // 格式化包，用于格式化输出
    "runtime/debug" // 运行时包，用于获取调用栈信息
    "sort" // 排序包，用于对切片进行排序
    "time" // 时间包，用于处理时间相关操作

    "github.com/cenkalti/backoff/v4" // 重试包，用于实现指数退避算法
    offroute "github.com/ipfs/boxo/routing/offline" // 离线路由包
    ds "github.com/ipfs/go-datastore" // 数据存储包，用于数据存储和检索
    dht "github.com/libp2p/go-libp2p-kad-dht" // Kademlia DHT 包
    ddht "github.com/libp2p/go-libp2p-kad-dht/dual" // 双重 Kademlia DHT 包
    "github.com/libp2p/go-libp2p-kad-dht/fullrt" // 完整路由表包
    pubsub "github.com/libp2p/go-libp2p-pubsub" // 发布订阅包
    namesys "github.com/libp2p/go-libp2p-pubsub-router" // 发布订阅路由包
    record "github.com/libp2p/go-libp2p-record" // 记录包，用于记录和检索数据
    routinghelpers "github.com/libp2p/go-libp2p-routing-helpers" // 路由助手包
    "github.com/libp2p/go-libp2p/core/host" // 主机核心包，用于构建和管理 libp2p 主机
    "github.com/libp2p/go-libp2p/core/peer" // 对等体包，用于表示 libp2p 网络中的节点
    "github.com/libp2p/go-libp2p/core/routing" // 路由核心包，用于构建和管理 libp2p 路由

    "go.uber.org/fx" // 依赖注入框架

    config "github.com/ipfs/kubo/config" // 配置包，用于处理配置信息
    "github.com/ipfs/kubo/core/node/helpers" // 节点助手包，用于处理节点相关操作
    "github.com/ipfs/kubo/repo" // 仓库包，用于处理仓库相关操作
    irouting "github.com/ipfs/kubo/routing" // 路由接口包
)

// Router 结构体，实现了 Routing 接口，用于表示 libp2p 路由器
type Router struct {
    routing.Routing // 继承自 Routing 接口

    Priority int // 优先级，数值越小表示越重要
}

// p2pRouterOut 结构体，用于输出 Router 实例
type p2pRouterOut struct {
    fx.Out // 依赖注入输出

    Router Router `group:"routers"` // 输出 Router 实例到 routers 组
}

// processInitialRoutingIn 结构体，用于处理初始路由输入
type processInitialRoutingIn struct {
    fx.In // 依赖注入输入

    Router routing.Routing `name:"initialrouting"` // 输入 initialrouting 名称的 Routing 实例

    // 用于设置实验性 DHT 客户端
    Host      host.Host // 主机实例
    Repo      repo.Repo // 仓库实例
    Validator record.Validator // 记录验证器
}

// processInitialRoutingOut 结构体，用于输出初始路由处理结果
type processInitialRoutingOut struct {
    fx.Out // 依赖注入输出

    Router        Router                 `group:"routers"` // 输出 Router 实例到 routers 组
    ContentRouter routing.ContentRouting `group:"content-routers"` // 输出 ContentRouting 实例到 content-routers 组

    DHT       *ddht.DHT // 双重 Kademlia DHT 实例
    DHTClient routing.Routing `name:"dhtc"` // 名称为 dhtc 的 Routing 实例
}

// AddrInfoChan 类型，用于表示 peer.AddrInfo 的通道
type AddrInfoChan chan peer.AddrInfo

// BaseRouting 函数，用于创建基础路由
func BaseRouting(cfg *config.Config) interface{} {
    // TODO: 实现基础路由的创建
}

// p2pOnlineContentRoutingIn 结构体，用于处理在线内容路由输入
type p2pOnlineContentRoutingIn struct {
    fx.In // 依赖注入输入

    ContentRouter []routing.ContentRouting `group:"content-routers"` // 输入 content-routers 组的 ContentRouting 实例
}

// ContentRouting 函数，用于获取所有可以执行内容路由的路由器，并使用 TieredRouter 将它们全部添加在一起
// 用于主题发现
func ContentRouting(in p2pOnlineContentRoutingIn) routing.ContentRouting {
    // TODO: 实现内容路由的获取和添加
}
    # 声明一个空的 routing.Routing 类型的切片
    var routers []routing.Routing
    # 遍历输入的 ContentRouter 切片
    for _, cr := range in.ContentRouter:
        # 将 ContentRouter 转换为 routinghelpers.Compose 类型，并添加到 routers 切片中
        routers = append(routers,
            &routinghelpers.Compose{
                ContentRouting: cr,
            },
        )
    # 返回一个 routinghelpers.Tiered 类型，其中包含 routers 切片
    return routinghelpers.Tiered{
        Routers: routers,
    }
// 结构体定义，包含了依赖注入的路由器列表和记录验证器
type p2pOnlineRoutingIn struct {
    fx.In

    Routers   []Router `group:"routers"`
    Validator record.Validator
}

// Routing 函数将从不同方法获取的所有路由器（委托路由器、发布-订阅等）汇总到一起，使用 TieredRouter
func Routing(in p2pOnlineRoutingIn) irouting.ProvideManyRouter {
    // 获取路由器列表
    routers := in.Routers

    // 按照优先级对路由器列表进行排序
    sort.SliceStable(routers, func(i, j int) bool {
        return routers[i].Priority < routers[j].Priority
    })

    // 创建并组合并行路由器
    var cRouters []*routinghelpers.ParallelRouter
    for _, v := range routers {
        cRouters = append(cRouters, &routinghelpers.ParallelRouter{
            IgnoreError:             true,
            DoNotWaitForSearchValue: true,
            Router:                  v.Routing,
        })
    }

    return routinghelpers.NewComposableParallel(cRouters)
}

// OfflineRouting 在创建离线节点时，向路由器列表提供一个特殊的路由器
func OfflineRouting(dstore ds.Datastore, validator record.Validator) p2pRouterOut {
    return p2pRouterOut{
        Router: Router{
            Routing:  offroute.NewOfflineRouter(dstore, validator),
            Priority: 10000,
        },
    }
}

// 结构体定义，包含了依赖注入的记录验证器、主机和发布-订阅对象
type p2pPSRoutingIn struct {
    fx.In

    Validator record.Validator
    Host      host.Host
    PubSub    *pubsub.PubSub `optional:"true"`
}

// PubsubRouter 函数用于创建 Pubsub 路由器
func PubsubRouter(mctx helpers.MetricsCtx, lc fx.Lifecycle, in p2pPSRoutingIn) (p2pRouterOut, *namesys.PubsubValueStore, error) {
    // 创建 PubsubValueStore 对象
    psRouter, err := namesys.NewPubsubValueStore(
        helpers.LifecycleCtx(mctx, lc),
        in.Host,
        in.PubSub,
        in.Validator,
        namesys.WithRebroadcastInterval(time.Minute),
    )
    if err != nil {
        return p2pRouterOut{}, nil, err
    }
    # 返回一个p2pRouterOut结构体，包含Router字段和psRouter字段，以及一个空的错误值
    return p2pRouterOut{
        # 继承Router结构体的字段
        Router: Router{
            # 设置路由的具体实现，这里使用Compose来组合多个路由
            Routing: &routinghelpers.Compose{
                # 设置值存储的具体实现，这里使用LimitedValueStore来限制存储的命名空间
                ValueStore: &routinghelpers.LimitedValueStore{
                    # 设置值存储的具体实现，这里使用psRouter
                    ValueStore: psRouter,
                    # 设置允许的命名空间
                    Namespaces: []string{"ipns"},
                },
            },
            # 设置路由的优先级
            Priority: 100,
        },
    }, psRouter, nil
# 定义一个名为autoRelayFeeder的函数，接受config.Peering类型的参数cfgPeering和一个用于发送peer.AddrInfo类型数据的通道peerChan
func autoRelayFeeder(cfgPeering config.Peering, peerChan chan<- peer.AddrInfo) fx.Option {
    # 函数体开始
    # 函数体结束
    })
}
```