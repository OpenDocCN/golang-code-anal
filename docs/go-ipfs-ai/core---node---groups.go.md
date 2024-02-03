# `kubo\core\node\groups.go`

```go
package node

import (
    "context" // 上下文包，用于控制程序的执行流程
    "errors" // 错误处理包，用于处理错误信息
    "fmt" // 格式化包，用于格式化输出
    "time" // 时间包，用于处理时间相关操作

    "github.com/dustin/go-humanize" // 第三方库，用于格式化数据大小
    blockstore "github.com/ipfs/boxo/blockstore" // 第三方库，用于块存储
    offline "github.com/ipfs/boxo/exchange/offline" // 第三方库，用于离线交换
    uio "github.com/ipfs/boxo/ipld/unixfs/io" // 第三方库，用于UnixFS I/O操作
    util "github.com/ipfs/boxo/util" // 第三方库，用于工具函数
    "github.com/ipfs/go-log" // IPFS日志包
    "github.com/ipfs/kubo/config" // Kubo配置包
    "github.com/ipfs/kubo/core/node/libp2p" // Kubo核心节点libp2p包
    "github.com/ipfs/kubo/p2p" // Kubo P2P包
    pubsub "github.com/libp2p/go-libp2p-pubsub" // PubSub包
    "github.com/libp2p/go-libp2p-pubsub/timecache" // PubSub时间缓存包
    "github.com/libp2p/go-libp2p/core/peer" // Libp2p核心节点peer包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager" // Libp2p主机资源管理包
    "go.uber.org/fx" // Uber的fx包
)

var logger = log.Logger("core:constructor") // 定义日志记录器

var BaseLibP2P = fx.Options( // 定义BaseLibP2P选项
    fx.Provide(libp2p.PNet), // 提供PNet
    fx.Provide(libp2p.ConnectionManager), // 提供连接管理器
    fx.Provide(libp2p.Host), // 提供主机
    fx.Provide(libp2p.MultiaddrResolver), // 提供多地址解析器

    fx.Provide(libp2p.DiscoveryHandler), // 提供发现处理器

    fx.Invoke(libp2p.PNetChecker), // 调用PNetChecker
)

func LibP2P(bcfg *BuildCfg, cfg *config.Config, userResourceOverrides rcmgr.PartialLimitConfig) fx.Option { // 定义LibP2P函数
    var connmgr fx.Option // 定义连接管理器选项

    // 根据Swarm.ConnMgr.Type设置连接管理器
    connMgrType := cfg.Swarm.ConnMgr.Type.WithDefault(config.DefaultConnMgrType)
    switch connMgrType {
    case "none":
        connmgr = fx.Options() // 空操作
    case "", "basic":
        grace := cfg.Swarm.ConnMgr.GracePeriod.WithDefault(config.DefaultConnMgrGracePeriod)
        low := int(cfg.Swarm.ConnMgr.LowWater.WithDefault(config.DefaultConnMgrLowWater))
        high := int(cfg.Swarm.ConnMgr.HighWater.WithDefault(config.DefaultConnMgrHighWater))
        connmgr = fx.Provide(libp2p.ConnectionManager(low, high, grace)) // 提供连接管理器
    default:
        return fx.Error(fmt.Errorf("unrecognized Swarm.ConnMgr.Type: %q", connMgrType)) // 返回错误信息
    }

    // 解析PubSub配置

    ps, disc := fx.Options(), fx.Options() // 定义ps和disc选项
    # 如果配置中包含pubsub或ipnsps选项，则执行以下代码块
    if bcfg.getOpt("pubsub") || bcfg.getOpt("ipnsps") {
        # 使用fx.Provide函数提供TopicDiscovery服务
        disc = fx.Provide(libp2p.TopicDiscovery())

        # 创建pubsubOptions切片用于存储pubsub选项
        var pubsubOptions []pubsub.Option
        # 将pubsub选项追加到pubsubOptions切片中
        pubsubOptions = append(
            pubsubOptions,
            pubsub.WithMessageSigning(!cfg.Pubsub.DisableSigning),
            pubsub.WithSeenMessagesTTL(cfg.Pubsub.SeenMessagesTTL.WithDefault(pubsub.TimeCacheDuration)),
        )

        # 创建seenMessagesStrategy变量用于存储消息查看策略
        var seenMessagesStrategy timecache.Strategy
        # 获取配置中的SeenMessagesStrategy，如果未设置则使用默认值
        configSeenMessagesStrategy := cfg.Pubsub.SeenMessagesStrategy.WithDefault(config.DefaultSeenMessagesStrategy)
        # 根据配置的SeenMessagesStrategy设置seenMessagesStrategy
        switch configSeenMessagesStrategy {
        case config.LastSeenMessagesStrategy:
            seenMessagesStrategy = timecache.Strategy_LastSeen
        case config.FirstSeenMessagesStrategy:
            seenMessagesStrategy = timecache.Strategy_FirstSeen
        default:
            return fx.Error(fmt.Errorf("unsupported Pubsub.SeenMessagesStrategy %q", configSeenMessagesStrategy))
        }
        # 将消息查看策略追加到pubsubOptions切片中
        pubsubOptions = append(pubsubOptions, pubsub.WithSeenMessagesStrategy(seenMessagesStrategy))

        # 根据配置中的Pubsub.Router选择不同的pubsub路由
        switch cfg.Pubsub.Router {
        case "":
            fallthrough
        case "gossipsub":
            # 使用fx.Provide函数提供GossipSub服务，并传入pubsubOptions切片
            ps = fx.Provide(libp2p.GossipSub(pubsubOptions...))
        case "floodsub":
            # 使用fx.Provide函数提供FloodSub服务，并传入pubsubOptions切片
            ps = fx.Provide(libp2p.FloodSub(pubsubOptions...))
        default:
            return fx.Error(fmt.Errorf("unknown pubsub router %s", cfg.Pubsub.Router))
        }
    }

    # 创建autonat变量用于存储自动NAT服务选项
    autonat := fx.Options()

    # 根据配置中的AutoNAT.ServiceMode选择不同的自动NAT服务模式
    switch cfg.AutoNAT.ServiceMode {
    default:
        # 抛出错误，表示未处理的自动NAT服务模式
        panic("BUG: unhandled autonat service mode")
    case config.AutoNATServiceDisabled:
    case config.AutoNATServiceUnset:
        # TODO
        #
        // 我们目前默认在所有节点上启用AutoNAT服务。
        //
        // 如果dht设置为dhtclient，我们应该考虑默认禁用它。
        fallthrough
    // 根据配置文件中的AutoNATServiceEnabled字段，提供自动NAT服务
    case config.AutoNATServiceEnabled:
        autonat = fx.Provide(libp2p.AutoNATService(cfg.AutoNAT.Throttle))
    }

    // 根据配置文件中的Relay相关字段，设置Relay传输是否启用，默认为true
    enableRelayTransport := cfg.Swarm.Transports.Network.Relay.WithDefault(true) // nolint
    // 根据配置文件中的RelayService相关字段，设置Relay服务是否启用，默认与Relay传输一致
    enableRelayService := cfg.Swarm.RelayService.Enabled.WithDefault(enableRelayTransport)
    // 根据配置文件中的RelayClient相关字段，设置Relay客户端是否启用，默认与Relay传输一致
    enableRelayClient := cfg.Swarm.RelayClient.Enabled.WithDefault(enableRelayTransport)

    // 当Relay传输未启用时，记录错误日志并终止程序
    if !enableRelayTransport {
        if enableRelayService {
            logger.Fatal("Failed to enable `Swarm.RelayService`, it requires `Swarm.Transports.Network.Relay` to be true.")
        }
        if enableRelayClient {
            logger.Fatal("Failed to enable `Swarm.RelayClient`, it requires `Swarm.Transports.Network.Relay` to be true.")
        }
    }

    // 强制用户迁移旧配置，记录错误日志并终止程序
    // nolint
    if cfg.Swarm.DisableRelay {
        logger.Fatal("The 'Swarm.DisableRelay' config field was removed." +
            "Use the 'Swarm.Transports.Network.Relay' instead.")
    }
    // nolint
    if cfg.Swarm.EnableAutoRelay {
        logger.Fatal("The 'Swarm.EnableAutoRelay' config field was removed." +
            "Use the 'Swarm.RelayClient.Enabled' instead.")
    }
    // nolint
    if cfg.Swarm.EnableRelayHop {
        logger.Fatal("The `Swarm.EnableRelayHop` config field was removed.\n" +
            "Use `Swarm.RelayService` to configure the circuit v2 relay.\n" +
            "If you want to continue running a circuit v1 relay, please use the standalone relay daemon: https://dist.ipfs.tech/#libp2p-relay-daemon (with RelayV1.Enabled: true)")
    }

    // 收集所有选项
    # 创建一个选项列表，包含各种配置和服务
    opts := fx.Options(
        BaseLibP2P,  # 基础的 LibP2P 配置

        # 获取 identify 模块的 AgentVersion（包括可选的 agent-version-suffix）
        fx.Provide(libp2p.UserAgent()),

        # 服务（资源管理）
        fx.Provide(libp2p.ResourceManager(cfg.Swarm, userResourceOverrides)),
        fx.Provide(libp2p.AddrFilters(cfg.Swarm.AddrFilters)),
        fx.Provide(libp2p.AddrsFactory(cfg.Addresses.Announce, cfg.Addresses.AppendAnnounce, cfg.Addresses.NoAnnounce)),
        fx.Provide(libp2p.SmuxTransport(cfg.Swarm.Transports)),
        fx.Provide(libp2p.RelayTransport(enableRelayTransport)),
        fx.Provide(libp2p.RelayService(enableRelayService, cfg.Swarm.RelayService)),
        fx.Provide(libp2p.Transports(cfg.Swarm.Transports)),
        fx.Provide(libp2p.ListenOn(cfg.Addresses.Swarm)),
        fx.Invoke(libp2p.SetupDiscovery(cfg.Discovery.MDNS.Enabled)),
        fx.Provide(libp2p.ForceReachability(cfg.Internal.Libp2pForceReachability)),
        fx.Provide(libp2p.HolePunching(cfg.Swarm.EnableHolePunching, enableRelayClient)),

        fx.Provide(libp2p.Security(!bcfg.DisableEncryptedConnections, cfg.Swarm.Transports)),

        fx.Provide(libp2p.Routing),  # 提供路由服务
        fx.Provide(libp2p.ContentRouting),  # 提供内容路由服务

        fx.Provide(libp2p.BaseRouting(cfg)),  # 提供基础路由服务
        maybeProvide(libp2p.PubsubRouter, bcfg.getOpt("ipnsps")),  # 可能提供 pubsub 路由服务

        maybeProvide(libp2p.BandwidthCounter, !cfg.Swarm.DisableBandwidthMetrics),  # 可能提供带宽计数器
        maybeProvide(libp2p.NatPortMap, !cfg.Swarm.DisableNatPortMap),  # 可能提供 NAT 端口映射
        libp2p.MaybeAutoRelay(cfg.Swarm.RelayClient.StaticRelays, cfg.Peering, enableRelayClient),  # 可能自动中继
        autonat,  # 自动 NAT 穿透
        connmgr,  # 连接管理
        ps,  # 发布/订阅服务
        disc,  # 发现服务
    )

    return opts  # 返回配置选项列表
// Storage函数用于设置数据存储和块存储层的持久性
func Storage(bcfg *BuildCfg, cfg *config.Config) fx.Option {
    // 设置块存储的缓存选项
    cacheOpts := blockstore.DefaultCacheOpts()
    cacheOpts.HasBloomFilterSize = cfg.Datastore.BloomFilterSize
    // 如果不是永久性存储，则将布隆过滤器大小设置为0
    if !bcfg.Permanent {
        cacheOpts.HasBloomFilterSize = 0
    }

    // 提供GcBlockstoreCtor函数作为最终的块存储
    finalBstore := fx.Provide(GcBlockstoreCtor)
    // 如果启用了实验性的文件存储或URL存储，则提供FilestoreBlockstoreCtor函数作为最终的块存储
    if cfg.Experimental.FilestoreEnabled || cfg.Experimental.UrlstoreEnabled {
        finalBstore = fx.Provide(FilestoreBlockstoreCtor)
    }

    // 返回存储相关的依赖注入选项
    return fx.Options(
        fx.Provide(RepoConfig),
        fx.Provide(Datastore),
        fx.Provide(BaseBlockstoreCtor(cacheOpts, bcfg.NilRepo, cfg.Datastore.HashOnRead)),
        finalBstore,
    )
}

// Identity函数用于提供加密身份相关的依赖注入选项
func Identity(cfg *config.Config) fx.Option {
    // 获取对等节点的ID
    cid := cfg.Identity.PeerID
    // 如果对等节点ID为空，则返回错误
    if cid == "" {
        return fx.Error(errors.New("identity was not set in config (was 'ipfs init' run?)"))
    }
    // 如果对等节点ID长度为0，则返回错误
    if len(cid) == 0 {
        return fx.Error(errors.New("no peer ID in config! (was 'ipfs init' run?)"))
    }

    // 解码对等节点ID
    id, err := peer.Decode(cid)
    if err != nil {
        return fx.Error(fmt.Errorf("peer ID invalid: %s", err))
    }

    // 如果身份配置中的私钥为空，则提供PeerID和Peerstore函数
    if cfg.Identity.PrivKey == "" {
        return fx.Options(
            fx.Provide(PeerID(id)),
            fx.Provide(libp2p.Peerstore),
        )
    }

    // 解码私钥并提供完整的身份相关的依赖注入选项
    sk, err := cfg.Identity.DecodePrivateKey("passphrase todo!")
    if err != nil {
        return fx.Error(err)
    }

    return fx.Options(
        fx.Provide(PeerID(id)),
        fx.Provide(PrivateKey(sk)),
        fx.Provide(libp2p.Peerstore),
        fx.Invoke(libp2p.PstoreAddSelfKeys),
    )
}

// IPNS用于提供与命名系统相关的依赖注入选项
var IPNS = fx.Options(
    fx.Provide(RecordValidator),
)

// Online用于提供仅在线相关的依赖注入选项
func Online(bcfg *BuildCfg, cfg *config.Config, userResourceOverrides rcmgr.PartialLimitConfig) fx.Option {
    // Namesys params

    // 从配置中获取 IPNS 解析缓存大小
    ipnsCacheSize := cfg.Ipns.ResolveCacheSize
    // 如果未设置缓存大小，则使用默认值
    if ipnsCacheSize == 0 {
        ipnsCacheSize = DefaultIpnsCacheSize
    }
    // 如果缓存大小为负数，则返回错误
    if ipnsCacheSize < 0 {
        return fx.Error(fmt.Errorf("cannot specify negative resolve cache size"))
    }

    // Republisher params

    var repubPeriod, recordLifetime time.Duration

    // 如果配置中设置了 IPNS 重新发布周期
    if cfg.Ipns.RepublishPeriod != "" {
        // 解析配置中的时间周期
        d, err := time.ParseDuration(cfg.Ipns.RepublishPeriod)
        // 如果解析失败，则返回错误
        if err != nil {
            return fx.Error(fmt.Errorf("failure to parse config setting IPNS.RepublishPeriod: %s", err))
        }
        // 如果不是调试模式，并且时间周期不在1分钟到1天之间，则返回错误
        if !util.Debug && (d < time.Minute || d > (time.Hour*24)) {
            return fx.Error(fmt.Errorf("config setting IPNS.RepublishPeriod is not between 1min and 1day: %s", d))
        }
        // 设置重新发布周期
        repubPeriod = d
    }

    // 如果配置中设置了 IPNS 记录生命周期
    if cfg.Ipns.RecordLifetime != "" {
        // 解析配置中的时间周期
        d, err := time.ParseDuration(cfg.Ipns.RecordLifetime)
        // 如果解析失败，则返回错误
        if err != nil {
            return fx.Error(fmt.Errorf("failure to parse config setting IPNS.RecordLifetime: %s", err))
        }
        // 设置记录生命周期
        recordLifetime = d
    }

    /* don't provide from bitswap when the strategic provider service is active */
    // 当策略提供服务处于活动状态时，不提供来自 bitswap 的内容
    shouldBitswapProvide := !cfg.Experimental.StrategicProviding
}
    # 返回一个包含各种选项的对象
    return fx.Options(
        # 提供 Bitswap 的选项
        fx.Provide(BitswapOptions(cfg, shouldBitswapProvide)),
        # 提供在线交换的选项
        fx.Provide(OnlineExchange()),
        # 提供 DNS 解析器
        fx.Provide(DNSResolver),
        # 提供 Namesys，设置 IPNS 缓存大小
        fx.Provide(Namesys(ipnsCacheSize)),
        # 提供对等节点
        fx.Provide(Peering),
        # 与指定的对等节点建立连接
        PeerWith(cfg.Peering.Peers...),
        # 调用 IpnsRepublisher，设置重新发布周期和记录生命周期
        fx.Invoke(IpnsRepublisher(repubPeriod, recordLifetime)),
        # 提供 P2P 新实例
        fx.Provide(p2p.New),
        # 创建 LibP2P 实例，设置配置、用户资源覆盖
        LibP2P(bcfg, cfg, userResourceOverrides),
        # 在线提供者，设置实验性提供策略、重新提供策略、重新提供间隔、加速 DHT 客户端
        OnlineProviders(
            cfg.Experimental.StrategicProviding,
            cfg.Reprovider.Strategy.WithDefault(config.DefaultReproviderStrategy),
            cfg.Reprovider.Interval.WithDefault(config.DefaultReproviderInterval),
            cfg.Routing.AcceleratedDHTClient,
        ),
    )
}

// Offline函数返回一个fx.Option，包含了离线环境下的IPFS基本服务
func Offline(cfg *config.Config) fx.Option {
    return fx.Options(
        fx.Provide(offline.Exchange), // 提供离线交换服务
        fx.Provide(DNSResolver), // 提供DNS解析服务
        fx.Provide(Namesys(0)), // 提供名称系统服务
        fx.Provide(libp2p.Routing), // 提供libp2p路由服务
        fx.Provide(libp2p.ContentRouting), // 提供libp2p内容路由服务
        fx.Provide(libp2p.OfflineRouting), // 提供libp2p离线路由服务
        OfflineProviders(), // 提供离线服务提供者
    )
}

// Core包含了基本的IPFS服务
var Core = fx.Options(
    fx.Provide(BlockService), // 提供块服务
    fx.Provide(Dag), // 提供DAG服务
    fx.Provide(FetcherConfig), // 提供获取器配置服务
    fx.Provide(PathResolverConfig), // 提供路径解析器配置服务
    fx.Provide(Pinning), // 提供固定服务
    fx.Provide(Files), // 提供文件服务
)

func Networked(bcfg *BuildCfg, cfg *config.Config, userResourceOverrides rcmgr.PartialLimitConfig) fx.Option {
    if bcfg.Online {
        return Online(bcfg, cfg, userResourceOverrides) // 如果在线，返回Online函数的结果
    }
    return Offline(cfg) // 否则返回Offline函数的结果
}

// IPFS函数基于传入的BuildCfg构建了一组fx选项
func IPFS(ctx context.Context, bcfg *BuildCfg) fx.Option {
    if bcfg == nil {
        bcfg = new(BuildCfg)
    }

    bcfgOpts, cfg := bcfg.options(ctx) // 获取BuildCfg的选项和配置
    if cfg == nil {
        return bcfgOpts // 如果配置为空，返回错误
    }

    userResourceOverrides, err := bcfg.Repo.UserResourceOverrides() // 获取用户资源覆盖
    if err != nil {
        return fx.Error(err) // 如果出错，返回错误
    }

    // 自动分片设置
    shardSizeString := cfg.Internal.UnixFSShardingSizeThreshold.WithDefault("256kiB") // 获取分片大小字符串
    shardSizeInt, err := humanize.ParseBytes(shardSizeString) // 解析分片大小字符串
    if err != nil {
        return fx.Error(err) // 如果出错，返回错误
    }
    uio.HAMTShardingSize = int(shardSizeInt) // 设置HAMT分片大小

    // 迁移已弃用的Experimental.ShardingEnabled标志的用户
    if cfg.Experimental.ShardingEnabled {
        logger.Fatal("The `Experimental.ShardingEnabled` field is no longer used, please remove it from the config.\n" +
            "go-ipfs now automatically shards when directory block is bigger than  `" + shardSizeString + "`.\n" +
            "If you need to restore the old behavior (sharding everything) set `Internal.UnixFSShardingSizeThreshold` to `1B`.\n") // 如果启用了分片，输出错误信息
    # 返回一个包含各种选项的对象，这些选项包括 bcfgOpts、baseProcess、Storage、Identity、IPNS、Networked 和 Core
    return fx.Options(
        bcfgOpts,  # 传入 bcfgOpts 作为选项
        fx.Provide(baseProcess),  # 使用 fx.Provide 提供 baseProcess 作为选项
        Storage(bcfg, cfg),  # 使用 bcfg 和 cfg 创建 Storage 对象，并作为选项
        Identity(cfg),  # 使用 cfg 创建 Identity 对象，并作为选项
        IPNS,  # 将 IPNS 作为选项
        Networked(bcfg, cfg, userResourceOverrides),  # 使用 bcfg、cfg 和 userResourceOverrides 创建 Networked 对象，并作为选项
        Core,  # 将 Core 作为选项
    )
# 闭合前面的函数定义
```