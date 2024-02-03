# `kubo\core\node\libp2p\routingopt.go`

```go
package libp2p

import (
    "context" // 引入上下文包
    "os" // 引入操作系统包
    "strings" // 引入字符串处理包
    "time" // 引入时间包

    "github.com/ipfs/go-datastore" // 引入数据存储包
    "github.com/ipfs/kubo/config" // 引入配置包
    irouting "github.com/ipfs/kubo/routing" // 引入路由包
    dht "github.com/libp2p/go-libp2p-kad-dht" // 引入 DHT 包
    dual "github.com/libp2p/go-libp2p-kad-dht/dual" // 引入双重 DHT 包
    record "github.com/libp2p/go-libp2p-record" // 引入记录包
    routinghelpers "github.com/libp2p/go-libp2p-routing-helpers" // 引入路由辅助包
    host "github.com/libp2p/go-libp2p/core/host" // 引入主机包
    "github.com/libp2p/go-libp2p/core/peer" // 引入对等节点包
    routing "github.com/libp2p/go-libp2p/core/routing" // 引入路由包
)

type RoutingOptionArgs struct {
    Ctx                           context.Context // 上下文
    Host                          host.Host // 主机
    Datastore                     datastore.Batching // 数据存储
    Validator                     record.Validator // 记录验证器
    BootstrapPeers                []peer.AddrInfo // 引导节点信息
    OptimisticProvide             bool // 乐观提供
    OptimisticProvideJobsPoolSize int // 乐观提供作业池大小
}

type RoutingOption func(args RoutingOptionArgs) (routing.Routing, error) // 路由选项函数

// Default HTTP routers used in parallel to DHT when Routing.Type = "auto"
var defaultHTTPRouters = []string{
    "https://cid.contact", // 默认的 HTTP 路由
    // TODO: add an independent router from Cloudflare
}

func init() {
    // Override HTTP routers if custom ones were passed via env
    if routers := os.Getenv("IPFS_HTTP_ROUTERS"); routers != "" {
        defaultHTTPRouters = strings.Split(routers, " ") // 如果通过环境变量传递了自定义的路由器，则覆盖默认的 HTTP 路由
    }
}

func constructDefaultHTTPRouters(cfg *config.Config) ([]*routinghelpers.ParallelRouter, error) {
    var routers []*routinghelpers.ParallelRouter // 创建并初始化路由器数组
    // Append HTTP routers for additional speed
    // 遍历默认的 HTTP 路由器列表
    for _, endpoint := range defaultHTTPRouters {
        // 构建 HTTP 路由器
        httpRouter, err := irouting.ConstructHTTPRouter(endpoint, cfg.Identity.PeerID, httpAddrsFromConfig(cfg.Addresses), cfg.Identity.PrivKey)
        // 如果构建过程中出现错误，则返回 nil 和错误信息
        if err != nil {
            return nil, err
        }

        // 创建路由器组合对象
        r := &irouting.Composer{
            GetValueRouter:      routinghelpers.Null{},
            PutValueRouter:      routinghelpers.Null{},
            ProvideRouter:       routinghelpers.Null{}, // 当索引器支持提供时修改此处
            FindPeersRouter:     routinghelpers.Null{},
            FindProvidersRouter: httpRouter,
        }

        // 将路由器组合对象添加到路由器列表中
        routers = append(routers, &routinghelpers.ParallelRouter{
            Router:                  r,
            IgnoreError:             true,             // https://github.com/ipfs/kubo/pull/9475#discussion_r1042507387
            Timeout:                 15 * time.Second, // 从服务器值 https://github.com/ipfs/kubo/pull/9475#discussion_r1042428529 的5倍
            DoNotWaitForSearchValue: true,
            ExecuteAfter:            0,
        })
    }
    // 返回路由器列表和空的错误信息
    return routers, nil
// ConstructDefaultRouting returns routers used when Routing.Type is unset or set to "auto"
func ConstructDefaultRouting(cfg *config.Config, routingOpt RoutingOption) RoutingOption {
    return func(args RoutingOptionArgs) (routing.Routing, error) {
        // Defined routers will be queried in parallel (optimizing for response speed)
        // Different trade-offs can be made by setting Routing.Type = "custom" with own Routing.Routers
        var routers []*routinghelpers.ParallelRouter

        // Call the provided routing option function to get the DHT routing and handle any error
        dhtRouting, err := routingOpt(args)
        if err != nil {
            return nil, err
        }
        // Append the DHT routing to the routers slice
        routers = append(routers, &routinghelpers.ParallelRouter{
            Router:                  dhtRouting,
            IgnoreError:             false,
            DoNotWaitForSearchValue: true,
            ExecuteAfter:            0,
        })

        // Call the function to construct default HTTP routers and handle any error
        httpRouters, err := constructDefaultHTTPRouters(cfg)
        if err != nil {
            return nil, err
        }

        // Append the HTTP routers to the routers slice
        routers = append(routers, httpRouters...)

        // Create a new composable parallel routing using the routers slice
        routing := routinghelpers.NewComposableParallel(routers)
        return routing, nil
    }
}

// constructDHTRouting is used when Routing.Type = "dht"
func constructDHTRouting(mode dht.ModeOpt) RoutingOption {
    return func(args RoutingOptionArgs) (routing.Routing, error) {
        // Define the DHT options based on the provided arguments
        dhtOpts := []dht.Option{
            dht.Concurrency(10),
            dht.Mode(mode),
            dht.Datastore(args.Datastore),
            dht.Validator(args.Validator),
        }
        // Add optimistic provide options if specified in the arguments
        if args.OptimisticProvide {
            dhtOpts = append(dhtOpts, dht.EnableOptimisticProvide())
        }
        if args.OptimisticProvideJobsPoolSize != 0 {
            dhtOpts = append(dhtOpts, dht.OptimisticProvideJobsPoolSize(args.OptimisticProvideJobsPoolSize))
        }
        // Create a new dual routing with the DHT options and bootstrap peers
        return dual.New(
            args.Ctx, args.Host,
            dual.DHTOption(dhtOpts...),
            dual.WanDHTOption(dht.BootstrapPeers(args.BootstrapPeers...)),
        )
    }
}
// ConstructDelegatedRouting用于当Routing.Type = "custom"时
func ConstructDelegatedRouting(routers config.Routers, methods config.Methods, peerID string, addrs config.Addresses, privKey string) RoutingOption {
    // 返回一个函数，该函数接受RoutingOptionArgs参数并返回routing.Routing和error
    return func(args RoutingOptionArgs) (routing.Routing, error) {
        // 调用irouting.Parse函数，传入routers、methods和ExtraDHTParams结构体
        // ExtraDHTParams结构体包含BootstrapPeers、Host、Validator、Datastore和Context字段
        // 同时传入ExtraHTTPParams结构体，包含PeerID、Addrs和PrivKeyB64字段
        return irouting.Parse(routers, methods,
            &irouting.ExtraDHTParams{
                BootstrapPeers: args.BootstrapPeers,
                Host:           args.Host,
                Validator:      args.Validator,
                Datastore:      args.Datastore,
                Context:        args.Ctx,
            },
            &irouting.ExtraHTTPParams{
                PeerID:     peerID,
                Addrs:      httpAddrsFromConfig(addrs),
                PrivKeyB64: privKey,
            },
        )
    }
}

// constructNilRouting用于返回一个空的routing.Routing和nil的error
func constructNilRouting(_ RoutingOptionArgs) (routing.Routing, error) {
    return routinghelpers.Null{}, nil
}

// 定义变量DHTOption，其值为constructDHTRouting(dht.ModeAuto)返回的RoutingOption
var (
    DHTOption       RoutingOption = constructDHTRouting(dht.ModeAuto)
    DHTClientOption               = constructDHTRouting(dht.ModeClient)
    DHTServerOption               = constructDHTRouting(dht.ModeServer)
    NilRouterOption               = constructNilRouting
)

// httpAddrsFromConfig从提供的配置中创建地址列表，用于HTTP委托路由器
func httpAddrsFromConfig(cfgAddrs config.Addresses) []string {
    // 默认使用Swarm地址
    addrs := cfgAddrs.Swarm
    // 如果指定了Announce地址，则覆盖Swarm地址
    if len(cfgAddrs.Announce) > 0 {
        addrs = cfgAddrs.Announce
    }
}
    } else if len(cfgAddrs.NoAnnounce) > 0 {
        // 如果未指定Announce地址 - 使用NoAnnounce列表过滤Swarm地址
        maddrs := map[string]struct{}{}
        // 将地址列表转换为map，方便进行删除操作
        for _, addr := range addrs {
            maddrs[addr] = struct{}{}
        }
        // 从map中删除NoAnnounce列表中的地址
        for _, addr := range cfgAddrs.NoAnnounce {
            delete(maddrs, addr)
        }
        // 将map中的地址重新转换为列表
        addrs = make([]string, 0, len(maddrs))
        for k := range maddrs {
            addrs = append(addrs, k)
        }
    }
    // 将AppendAnnounce地址追加到结果列表中
    if len(cfgAddrs.AppendAnnounce) > 0 {
        addrs = append(addrs, cfgAddrs.AppendAnnounce...)
    }
    // 返回最终的地址列表
    return addrs
# 闭合前面的函数定义
```