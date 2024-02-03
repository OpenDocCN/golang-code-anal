# `kubo\routing\delegated.go`

```go
package routing

import (
    "context" // 引入上下文包，用于控制请求的生命周期
    "encoding/base64" // 引入 base64 编码包，用于对数据进行 base64 编码
    "errors" // 引入错误处理包
    "fmt" // 引入格式化包，用于格式化输出
    "net/http" // 引入 HTTP 包，用于处理 HTTP 请求

    drclient "github.com/ipfs/boxo/routing/http/client" // 引入 HTTP 客户端包
    "github.com/ipfs/boxo/routing/http/contentrouter" // 引入 HTTP 内容路由包
    "github.com/ipfs/go-datastore" // 引入数据存储包
    logging "github.com/ipfs/go-log" // 引入日志包
    version "github.com/ipfs/kubo" // 引入版本包
    "github.com/ipfs/kubo/config" // 引入配置包
    dht "github.com/libp2p/go-libp2p-kad-dht" // 引入 Kademlia 分布式哈希表包
    "github.com/libp2p/go-libp2p-kad-dht/dual" // 引入双重 Kademlia 分布式哈希表包
    "github.com/libp2p/go-libp2p-kad-dht/fullrt" // 引入全路由 Kademlia 分布式哈希表包
    record "github.com/libp2p/go-libp2p-record" // 引入记录包
    routinghelpers "github.com/libp2p/go-libp2p-routing-helpers" // 引入路由助手包
    ic "github.com/libp2p/go-libp2p/core/crypto" // 引入核心加密包
    host "github.com/libp2p/go-libp2p/core/host" // 引入核心主机包
    "github.com/libp2p/go-libp2p/core/peer" // 引入核心对等体包
    "github.com/libp2p/go-libp2p/core/routing" // 引入核心路由包
    ma "github.com/multiformats/go-multiaddr" // 引入多格式地址包
    "go.opencensus.io/stats/view" // 引入统计视图包
)

var log = logging.Logger("routing/delegated") // 创建日志记录器

func Parse(routers config.Routers, methods config.Methods, extraDHT *ExtraDHTParams, extraHTTP *ExtraHTTPParams) (routing.Routing, error) {
    if err := methods.Check(); err != nil { // 检查方法是否有效
        return nil, err // 返回错误信息
    }

    createdRouters := make(map[string]routing.Routing) // 创建路由器映射

    finalRouter := &Composer{} // 创建组合路由器对象

    // Create all needed routers from method names
    # 遍历 methods 列表，获取索引和值
    for mn, m := range methods {
        # 调用 parse 函数解析路由器，返回路由器对象和错误信息
        router, err := parse(make(map[string]bool), createdRouters, m.RouterName, routers, extraDHT, extraHTTP)
        # 如果解析出错，返回空和错误信息
        if err != nil {
            return nil, err
        }

        # 根据不同的方法名，将解析得到的路由器对象赋值给 finalRouter 对应的字段
        switch mn {
        case config.MethodNamePutIPNS:
            finalRouter.PutValueRouter = router
        case config.MethodNameGetIPNS:
            finalRouter.GetValueRouter = router
        case config.MethodNameFindPeers:
            finalRouter.FindPeersRouter = router
        case config.MethodNameFindProviders:
            finalRouter.FindProvidersRouter = router
        case config.MethodNameProvide:
            finalRouter.ProvideRouter = router
        }

        # 记录使用的方法和对应的路由器名称
        log.Info("using method ", mn, " with router ", m.RouterName)
    }

    # 返回最终的路由器对象和空错误信息
    return finalRouter, nil
}
// 解析函数，根据给定参数创建路由对象
func parse(visited map[string]bool,
    createdRouters map[string]routing.Routing,
    routerName string,
    routersCfg config.Routers,
    extraDHT *ExtraDHTParams,
    extraHTTP *ExtraHTTPParams,
) (routing.Routing, error) {
    // 检查是否已经创建了该路由对象
    r, ok := createdRouters[routerName]
    if ok {
        return r, nil
    }

    // 检查是否存在依赖循环
    if visited[routerName] {
        return nil, fmt.Errorf("dependency loop creating router with name %q", routerName)
    }

    // 将节点标记为已访问
    visited[routerName] = true

    // 获取路由器配置
    cfg, ok := routersCfg[routerName]
    if !ok {
        return nil, fmt.Errorf("config for router with name %q not found", routerName)
    }

    var router routing.Routing
    var err error
    switch cfg.Type {
    case config.RouterTypeHTTP:
        router, err = httpRoutingFromConfig(cfg.Router, extraHTTP)
    case config.RouterTypeDHT:
        router, err = dhtRoutingFromConfig(cfg.Router, extraDHT)
    case config.RouterTypeParallel:
        crp := cfg.Parameters.(*config.ComposableRouterParams)
        var pr []*routinghelpers.ParallelRouter
        for _, cr := range crp.Routers {
            ri, err := parse(visited, createdRouters, cr.RouterName, routersCfg, extraDHT, extraHTTP)
            if err != nil {
                return nil, err
            }

            pr = append(pr, &routinghelpers.ParallelRouter{
                Router:                  ri,
                IgnoreError:             cr.IgnoreErrors,
                DoNotWaitForSearchValue: true,
                Timeout:                 cr.Timeout.Duration,
                ExecuteAfter:            cr.ExecuteAfter.WithDefault(0),
            })

        }

        router = routinghelpers.NewComposableParallel(pr)
    // 根据配置中的路由器类型选择不同的处理方式
    case config.RouterTypeSequential:
        // 将参数转换为 ComposableRouterParams 类型
        crp := cfg.Parameters.(*config.ComposableRouterParams)
        // 创建一个空的 SequentialRouter 切片
        var sr []*routinghelpers.SequentialRouter
        // 遍历配置中的路由器列表
        for _, cr := range crp.Routers {
            // 解析路由器配置并返回路由器实例
            ri, err := parse(visited, createdRouters, cr.RouterName, routersCfg, extraDHT, extraHTTP)
            // 如果解析出错，则返回错误
            if err != nil {
                return nil, err
            }
            // 将解析出的路由器实例添加到 SequentialRouter 切片中
            sr = append(sr, &routinghelpers.SequentialRouter{
                Router:      ri,
                IgnoreError: cr.IgnoreErrors,
                Timeout:     cr.Timeout.Duration,
            })
        }
        // 创建一个 ComposableSequential 路由器实例
        router = routinghelpers.NewComposableSequential(sr)
    // 如果路由器类型未知，则返回错误
    default:
        return nil, fmt.Errorf("unknown router type %q", cfg.Type)
    }

    // 如果在处理过程中出现错误，则返回错误
    if err != nil {
        return nil, err
    }

    // 将创建的路由器实例添加到已创建路由器的映射中
    createdRouters[routerName] = router

    // 记录创建的路由器信息
    log.Info("created router ", routerName, " with params ", cfg.Parameters)

    // 返回创建的路由器实例和空错误
    return router, nil
}
// 定义一个结构体，包含额外的 HTTP 参数
type ExtraHTTPParams struct {
    PeerID     string
    Addrs      []string
    PrivKeyB64 string
}

// 构造 HTTP 路由器
func ConstructHTTPRouter(endpoint string, peerID string, addrs []string, privKey string) (routing.Routing, error) {
    // 从配置中创建 HTTP 路由器
    return httpRoutingFromConfig(
        config.Router{
            Type: "http",
            Parameters: &config.HTTPRouterParams{
                Endpoint: endpoint,
            },
        },
        &ExtraHTTPParams{
            PeerID:     peerID,
            Addrs:      addrs,
            PrivKeyB64: privKey,
        },
    )
}

// 从配置中创建 HTTP 路由器
func httpRoutingFromConfig(conf config.Router, extraHTTP *ExtraHTTPParams) (routing.Routing, error) {
    // 获取 HTTP 路由器参数
    params := conf.Parameters.(*config.HTTPRouterParams)
    // 如果端点为空，则返回参数所需错误
    if params.Endpoint == "" {
        return nil, NewParamNeededErr("Endpoint", conf.Type)
    }

    // 填充默认参数
    params.FillDefaults()

    // 增加每个主机的连接池，因为我们正在进行大量并发请求
    transport := http.DefaultTransport.(*http.Transport).Clone()
    transport.MaxIdleConns = 500
    transport.MaxIdleConnsPerHost = 100

    // 创建委托的 HTTP 客户端
    delegateHTTPClient := &http.Client{
        Transport: &drclient.ResponseBodyLimitedTransport{
            RoundTripper: transport,
            LimitBytes:   1 << 20,
        },
    }

    // 解码私钥
    key, err := decodePrivKey(extraHTTP.PrivKeyB64)
    if err != nil {
        return nil, err
    }

    // 创建地址信息
    addrInfo, err := createAddrInfo(extraHTTP.PeerID, extraHTTP.Addrs)
    if err != nil {
        return nil, err
    }

    // 创建新的客户端
    cli, err := drclient.New(
        params.Endpoint,
        drclient.WithHTTPClient(delegateHTTPClient),
        drclient.WithIdentity(key),
        drclient.WithProviderInfo(addrInfo.ID, addrInfo.Addrs),
        drclient.WithUserAgent(version.GetUserAgentVersion()),
    )
    if err != nil {
        return nil, err
    }
    // 创建一个新的内容路由客户端
    cr := contentrouter.NewContentRoutingClient(
        cli,
        contentrouter.WithMaxProvideBatchSize(params.MaxProvideBatchSize),  // 设置最大提供批量大小
        contentrouter.WithMaxProvideConcurrency(params.MaxProvideConcurrency),  // 设置最大提供并发数
    )

    // 注册 HTTP 委托路由视图
    err = view.Register(drclient.OpenCensusViews...)
    if err != nil {
        return nil, fmt.Errorf("registering HTTP delegated routing views: %w", err)
    }

    // 返回一个包含内容路由、对等路由、数值存储和提供多个路由的 HTTP 路由包装器
    return &httpRoutingWrapper{
        ContentRouting:    cr,
        PeerRouting:       cr,
        ValueStore:        cr,
        ProvideManyRouter: cr,
    }, nil
}

// 根据 base64 编码的私钥字符串解码私钥
func decodePrivKey(keyB64 string) (ic.PrivKey, error) {
    // 使用 base64 解码私钥字符串
    pk, err := base64.StdEncoding.DecodeString(keyB64)
    if err != nil {
        return nil, err
    }

    // 反序列化私钥
    return ic.UnmarshalPrivateKey(pk)
}

// 创建地址信息对象
func createAddrInfo(peerID string, addrs []string) (peer.AddrInfo, error) {
    // 解码对等节点 ID
    pID, err := peer.Decode(peerID)
    if err != nil {
        return peer.AddrInfo{}, err
    }

    var mas []ma.Multiaddr
    // 遍历地址列表，创建多地址对象
    for _, a := range addrs {
        m, err := ma.NewMultiaddr(a)
        if err != nil {
            return peer.AddrInfo{}, err
        }

        mas = append(mas, m)
    }

    // 返回地址信息对象
    return peer.AddrInfo{
        ID:    pID,
        Addrs: mas,
    }, nil
}

// DHT 参数结构体
type ExtraDHTParams struct {
    BootstrapPeers []peer.AddrInfo
    Host           host.Host
    Validator      record.Validator
    Datastore      datastore.Batching
    Context        context.Context
}

// 根据配置创建 DHT 路由
func dhtRoutingFromConfig(conf config.Router, extra *ExtraDHTParams) (routing.Routing, error) {
    params, ok := conf.Parameters.(*config.DHTRouterParams)
    if !ok {
        return nil, errors.New("incorrect params for DHT router")
    }

    if params.AcceleratedDHTClient {
        return createFullRT(extra)
    }

    var mode dht.ModeOpt
    switch params.Mode {
    case config.DHTModeAuto:
        mode = dht.ModeAuto
    case config.DHTModeClient:
        mode = dht.ModeClient
    case config.DHTModeServer:
        mode = dht.ModeServer
    default:
        return nil, fmt.Errorf("invalid DHT mode: %q", params.Mode)
    }

    return createDHT(extra, params.PublicIPNetwork, mode)
}

// 创建 DHT 路由
func createDHT(params *ExtraDHTParams, public bool, mode dht.ModeOpt) (routing.Routing, error) {
    var opts []dht.Option

    if public {
        opts = append(opts, dht.QueryFilter(dht.PublicQueryFilter),
            dht.RoutingTableFilter(dht.PublicRoutingTableFilter),
            dht.RoutingTablePeerDiversityFilter(dht.NewRTPeerDiversityFilter(params.Host, 2, 3)))
    } else {
        // 如果条件不成立，执行以下操作
        opts = append(opts, dht.ProtocolExtension(dual.LanExtension),
            dht.QueryFilter(dht.PrivateQueryFilter),
            dht.RoutingTableFilter(dht.PrivateRoutingTableFilter))
    }

    // 将参数添加到 opts 切片中
    opts = append(opts,
        dht.Concurrency(10),
        dht.Mode(mode),
        dht.Datastore(params.Datastore),
        dht.Validator(params.Validator),
        dht.BootstrapPeers(params.BootstrapPeers...))

    // 创建并返回一个新的 DHT 对象
    return dht.New(
        params.Context, params.Host, opts...,
    )
# 创建一个完整的路由对象，并返回该对象和可能的错误
func createFullRT(params *ExtraDHTParams) (routing.Routing, error) {
    # 使用参数中的主机和默认前缀创建一个新的完整路由对象
    return fullrt.NewFullRT(params.Host,
        dht.DefaultPrefix,
        # 使用参数中的选项配置完整路由对象
        fullrt.DHTOption(
            dht.Validator(params.Validator),  # 设置验证器
            dht.Datastore(params.Datastore),  # 设置数据存储
            dht.BootstrapPeers(params.BootstrapPeers...),  # 设置引导节点
            dht.BucketSize(20),  # 设置桶大小
        ),
    )
}
```