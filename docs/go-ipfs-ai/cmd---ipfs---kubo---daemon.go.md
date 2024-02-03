# `kubo\cmd\ipfs\kubo\daemon.go`

```go
package kubo

import (
    "errors"  // 导入 errors 包，用于处理错误
    _ "expvar"  // 导入 expvar 包，但不直接使用它
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net"  // 导入 net 包，用于网络操作
    "net/http"  // 导入 net/http 包，用于 HTTP 操作
    _ "net/http/pprof"  // 导入 net/http/pprof 包，但不直接使用它
    "os"  // 导入 os 包，用于操作系统功能
    "runtime"  // 导入 runtime 包，用于访问 Go 运行时环境
    "sort"  // 导入 sort 包，用于排序
    "sync"  // 导入 sync 包，用于同步操作
    "time"  // 导入 time 包，用于时间操作

    multierror "github.com/hashicorp/go-multierror"  // 导入第三方包 github.com/hashicorp/go-multierror，并重命名为 multierror

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入第三方包 github.com/ipfs/go-ipfs-cmds，并重命名为 cmds
    mprome "github.com/ipfs/go-metrics-prometheus"  // 导入第三方包 github.com/ipfs/go-metrics-prometheus，并重命名为 mprome
    version "github.com/ipfs/kubo"  // 导入第三方包 github.com/ipfs/kubo，并重命名为 version
    utilmain "github.com/ipfs/kubo/cmd/ipfs/util"  // 导入第三方包 github.com/ipfs/kubo/cmd/ipfs/util，并重命名为 utilmain
    oldcmds "github.com/ipfs/kubo/commands"  // 导入第三方包 github.com/ipfs/kubo/commands，并重命名为 oldcmds
    config "github.com/ipfs/kubo/config"  // 导入第三方包 github.com/ipfs/kubo/config，并重命名为 config
    cserial "github.com/ipfs/kubo/config/serialize"  // 导入第三方包 github.com/ipfs/kubo/config/serialize，并重命名为 cserial
    "github.com/ipfs/kubo/core"  // 导入第三方包 github.com/ipfs/kubo/core
    commands "github.com/ipfs/kubo/core/commands"  // 导入第三方包 github.com/ipfs/kubo/core/commands，并重命名为 commands
    "github.com/ipfs/kubo/core/coreapi"  // 导入第三方包 github.com/ipfs/kubo/core/coreapi
    corehttp "github.com/ipfs/kubo/core/corehttp"  // 导入第三方包 github.com/ipfs/kubo/core/corehttp，并重命名为 corehttp
    options "github.com/ipfs/kubo/core/coreiface/options"  // 导入第三方包 github.com/ipfs/kubo/core/coreiface/options，并重命名为 options
    corerepo "github.com/ipfs/kubo/core/corerepo"  // 导入第三方包 github.com/ipfs/kubo/core/corerepo，并重命名为 corerepo
    libp2p "github.com/ipfs/kubo/core/node/libp2p"  // 导入第三方包 github.com/ipfs/kubo/core/node/libp2p，并重命名为 libp2p
    nodeMount "github.com/ipfs/kubo/fuse/node"  // 导入第三方包 github.com/ipfs/kubo/fuse/node，并重命名为 nodeMount
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 导入第三方包 github.com/ipfs/kubo/repo/fsrepo，并重命名为 fsrepo
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // 导入第三方包 github.com/ipfs/kubo/repo/fsrepo/migrations
    "github.com/ipfs/kubo/repo/fsrepo/migrations/ipfsfetcher"  // 导入第三方包 github.com/ipfs/kubo/repo/fsrepo/migrations/ipfsfetcher
    goprocess "github.com/jbenet/goprocess"  // 导入第三方包 github.com/jbenet/goprocess，并重命名为 goprocess
    p2pcrypto "github.com/libp2p/go-libp2p/core/crypto"  // 导入第三方包 github.com/libp2p/go-libp2p/core/crypto，并重命名为 p2pcrypto
    pnet "github.com/libp2p/go-libp2p/core/pnet"  // 导入第三方包 github.com/libp2p/go-libp2p/core/pnet，并重命名为 pnet
    "github.com/libp2p/go-libp2p/core/protocol"  // 导入第三方包 github.com/libp2p/go-libp2p/core/protocol
    p2phttp "github.com/libp2p/go-libp2p/p2p/http"  // 导入第三方包 github.com/libp2p/go-libp2p/p2p/http，并重命名为 p2phttp
    sockets "github.com/libp2p/go-socket-activation"  // 导入第三方包 github.com/libp2p/go-socket-activation，并重命名为 sockets
    ma "github.com/multiformats/go-multiaddr"  // 导入第三方包 github.com/multiformats/go-multiaddr，并重命名为 ma
    manet "github.com/multiformats/go-multiaddr/net"  // 导入第三方包 github.com/multiformats/go-multiaddr/net，并重命名为 manet
    prometheus "github.com/prometheus/client_golang/prometheus"  // 导入第三方包 github.com/prometheus/client_golang/prometheus，并重命名为 prometheus
    promauto "github.com/prometheus/client_golang/prometheus/promauto"  // 导入第三方包 github.com/prometheus/client_golang/prometheus/promauto，并重命名为 promauto
)

const (
    adjustFDLimitKwd           = "manage-fdlimit"  // 声明常量 adjustFDLimitKwd，值为 "manage-fdlimit"
    enableGCKwd                = "enable-gc"  // 声明常量 enableGCKwd，值为 "enable-gc"
    initOptionKwd              = "init"  // 声明常量 initOptionKwd，值为 "init"
    initConfigOptionKwd        = "init-config"  // 声明常量 initConfigOptionKwd，值为 "init-config"
    initProfileOptionKwd       = "init-profile"  // 声明常量 initProfileOptionKwd，值为 "init-profile"
    ipfsMountKwd               = "mount-ipfs"  // 声明常量 ipfsMountKwd，值为 "mount-ipfs"
    ipnsMountKwd               = "mount-ipns"  // 声明常量 ipnsMountKwd，值为 "mount-ipns"
    migrateKwd                 = "migrate"  // 声明常量 migrateKwd，值为 "migrate"
    # 定义关键字 "mount"
    mountKwd                   = "mount"
    # 定义全局选项 "offline"
    offlineKwd                 = "offline" // global option
    # 定义路由选项关键字 "routing"
    routingOptionKwd           = "routing"
    # 定义超级节点路由选项关键字 "supernode"
    routingOptionSupernodeKwd  = "supernode"
    # 定义 DHT 客户端路由选项关键字 "dhtclient"
    routingOptionDHTClientKwd  = "dhtclient"
    # 定义 DHT 路由选项关键字 "dht"
    routingOptionDHTKwd        = "dht"
    # 定义 DHT 服务器路由选项关键字 "dhtserver"
    routingOptionDHTServerKwd  = "dhtserver"
    # 定义无路由选项关键字 "none"
    routingOptionNoneKwd       = "none"
    # 定义自定义路由选项关键字 "custom"
    routingOptionCustomKwd     = "custom"
    # 定义默认路由选项关键字 "default"
    routingOptionDefaultKwd    = "default"
    # 定义自动路由选项关键字 "auto"
    routingOptionAutoKwd       = "auto"
    # 定义自动客户端路由选项关键字 "autoclient"
    routingOptionAutoClientKwd = "autoclient"
    # 定义禁用传输加密选项关键字 "disable-transport-encryption"
    unencryptTransportKwd      = "disable-transport-encryption"
    # 定义无限制 API 访问选项关键字 "unrestricted-api"
    unrestrictedAPIAccessKwd   = "unrestricted-api"
    # 定义可写选项关键字 "writable"
    writableKwd                = "writable"
    # 定义启用发布订阅实验选项关键字 "enable-pubsub-experiment"
    enablePubSubKwd            = "enable-pubsub-experiment"
    # 定义启用 IPNS 发布订阅选项关键字 "enable-namesys-pubsub"
    enableIPNSPubSubKwd        = "enable-namesys-pubsub"
    # 定义启用多路复用实验选项关键字 "enable-mplex-experiment"
    enableMultiplexKwd         = "enable-mplex-experiment"
    # 定义代理版本后缀关键字 "agent-version-suffix"
    agentVersionSuffix         = "agent-version-suffix"
    # 注释掉的代码，暂时不使用
    // apiAddrKwd    = "address-api"
    // swarmAddrKwd  = "address-swarm".
# 定义一个名为daemonCmd的变量，类型为cmds.Command
var daemonCmd = &cmds.Command{
    # 帮助文本，包括一句标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Run a network-connected IPFS node.",
        ShortDescription: `
'ipfs daemon' runs a persistent ipfs daemon that can serve commands
over the network. Most applications that use IPFS will do so by
communicating with a daemon over the HTTP API. While the daemon is
running, calls to 'ipfs' commands will be sent over the network to
the daemon.
`,
        # 长描述
        LongDescription: `
The daemon will start listening on ports on the network, which are
documented in (and can be modified through) 'ipfs config Addresses'.
For example, to change the 'Gateway' port:

  ipfs config Addresses.Gateway /ip4/127.0.0.1/tcp/8082

The RPC API address can be changed the same way:

  ipfs config Addresses.API /ip4/127.0.0.1/tcp/5002

Make sure to restart the daemon after changing addresses.

By default, the gateway is only accessible locally. To expose it to
other computers in the network, use 0.0.0.0 as the ip address:

  ipfs config Addresses.Gateway /ip4/0.0.0.0/tcp/8080

Be careful if you expose the RPC API. It is a security risk, as anyone could
control your node remotely. If you need to control the node remotely,
make sure to protect the port as you would other services or database
(firewall, authenticated proxy, etc).

HTTP Headers

ipfs supports passing arbitrary headers to the RPC API and Gateway. You can
do this by setting headers on the API.HTTPHeaders and Gateway.HTTPHeaders
keys:

  ipfs config --json API.HTTPHeaders.X-Special-Header "[\"so special :)\"]"
  ipfs config --json Gateway.HTTPHeaders.X-Special-Header "[\"so special :)\"]"

Note that the value of the keys is an _array_ of strings. This is because
headers can have more than one value, and it is convenient to pass through
to other libraries.

CORS Headers (for API)
# 设置 CORS 头部信息的方法
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin "[\"example.com\"]"
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods "[\"PUT\", \"GET\", \"POST\"]"
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Credentials "[\"true\"]"

# 关闭守护进程
要关闭守护进程，可以向其发送 SIGINT 信号（例如按下 'Ctrl-C'）
或者向其发送 SIGTERM 信号（例如使用 'kill' 命令）。守护进程可能需要一段时间来优雅地关闭，但可以通过发送第二个信号来强制杀死它。

# IPFS_PATH 环境变量
ipfs 在本地文件系统中使用存储库。默认情况下，存储库位于 ~/.ipfs。要更改存储库位置，请设置 $IPFS_PATH 环境变量：
export IPFS_PATH=/path/to/ipfsrepo

# 弃用通知
以前，ipfs 使用了如下所示的环境变量：
export API_ORIGIN="http://localhost:8888/"
这已被弃用。在此版本中仍然有效，但将在将来的版本中删除，以及此通知。请转而设置 HTTP 头部。

# defaultMux 告诉 mux 使用默认的 muxer 来服务路径。这在大多数情况下用于连接注册在默认 muxer 中的东西，并且不提供方便的 http.Handler 入口点，例如 expvar 和 http/pprof。
func defaultMux(path string) corehttp.ServeOption {
    return func(node *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        mux.Handle(path, http.DefaultServeMux)
        return mux, nil
    }
}

# daemonFunc 是守护进程的函数
func daemonFunc(req *cmds.Request, re cmds.ResponseEmitter, env cmds.Environment) (_err error) {
    # 在执行任何操作之前注入指标
    err := mprome.Inject()
    // 如果发生错误，记录错误消息
    if err != nil {
        log.Errorf("Injecting prometheus handler for metrics failed with message: %s\n", err.Error())
    }

    // 让用户知道我们正在进行初始化
    fmt.Printf("Initializing daemon...\n")

    // 延迟函数，用于在函数返回前执行一些操作
    defer func() {
        if _err != nil {
            // 在任何错误之前打印一个额外的空行。这可能放在命令库中，但对于所有命令来说并不是很有意义。
            fmt.Println()
        }
    }()

    // 打印 IPFS 版本
    printVersion()

    // 检查是否需要管理文件描述符限制
    managefd, _ := req.Options[adjustFDLimitKwd].(bool)
    if managefd {
        if _, _, err := utilmain.ManageFdLimit(); err != nil {
            log.Errorf("setting file descriptor limit: %s", err)
        }
    }

    // 获取环境上下文
    cctx := env.(*oldcmds.Context)

    // 检查传输加密标志
    unencrypted, _ := req.Options[unencryptTransportKwd].(bool)
    if unencrypted {
        log.Warnf(`Running with --%s: All connections are UNENCRYPTED.
        You will not be able to connect to regular encrypted networks.`, unencryptTransportKwd)
    }

    // 首先，检查用户是否提供了初始化标志。我们可能处于未初始化状态。
    initialize, _ := req.Options[initOptionKwd].(bool)
    // 如果初始化标志为真，并且文件系统存储库未初始化
    if initialize && !fsrepo.IsInitialized(cctx.ConfigRoot) {
        // 从请求选项中获取初始化配置的位置和配置文件
        cfgLocation, _ := req.Options[initConfigOptionKwd].(string)
        profiles, _ := req.Options[initProfileOptionKwd].(string)
        var conf *config.Config

        // 如果配置文件位置不为空
        if cfgLocation != "" {
            // 加载配置文件
            if conf, err = cserial.Load(cfgLocation); err != nil {
                return err
            }
        }

        // 如果配置文件为空
        if conf == nil {
            // 创建身份并初始化配置
            identity, err := config.CreateIdentity(os.Stdout, []options.KeyGenerateOption{
                options.Key.Type(algorithmDefault),
            })
            if err != nil {
                return err
            }
            conf, err = config.InitWithIdentity(identity)
            if err != nil {
                return err
            }
        }

        // 执行初始化操作
        if err = doInit(os.Stdout, cctx.ConfigRoot, false, profiles, conf); err != nil {
            return err
        }
    }

    // 定义缓存迁移和固定迁移的布尔值，以及迁移获取器
    var cacheMigrations, pinMigrations bool
    var fetcher migrations.Fetcher

    // 在构建节点之前获取存储库锁，确保可以访问资源（数据存储等）
    repo, err := fsrepo.Open(cctx.ConfigRoot)
    switch err {
    default:
        return err
    case nil:
        break
    }

    // 节点也会关闭存储库，但在到达那之前可能会失败多次。关闭两次也无妨。
    defer repo.Close()

    // 从请求选项中获取离线标志、IPNS 发布订阅标志和发布订阅标志
    offline, _ := req.Options[offlineKwd].(bool)
    ipnsps, ipnsPsSet := req.Options[enableIPNSPubSubKwd].(bool)
    pubsub, psSet := req.Options[enablePubSubKwd].(bool)

    // 如果请求选项中有启用多路复用关键字
    if _, hasMplex := req.Options[enableMultiplexKwd]; hasMplex {
        log.Errorf("The mplex multiplexer has been enabled by default and the experimental %s flag has been removed.")
        log.Errorf("To disable this multiplexer, please configure `Swarm.Transports.Multiplexers'.")
    }

    // 获取存储库配置
    cfg, err := repo.Config()
    if err != nil {
        return err
    }
    // 如果未设置 pubsub 标志，则根据配置文件设置 pubsub 变量
    if !psSet {
        pubsub = cfg.Pubsub.Enabled.WithDefault(false)
    }
    // 如果未设置 ipnsPsSet 标志，则根据配置文件设置 ipnsps 变量
    if !ipnsPsSet {
        ipnsps = cfg.Ipns.UsePubsub.WithDefault(false)
    }

    // 开始组装节点配置
    ncfg := &core.BuildCfg{
        Repo:                        repo,
        Permanent:                   true, // 临时标志，表示节点是永久的
        Online:                      !offline,
        DisableEncryptedConnections: unencrypted,
        ExtraOpts: map[string]bool{
            "pubsub": pubsub,
            "ipnsps": ipnsps,
        },
        // TODO(Kubuxu): 通过添加 Permanent vs Ephemeral 来重构 Online vs Offline
    }

    // 从请求选项中获取路由选项
    routingOption, _ := req.Options[routingOptionKwd].(string)
    // 如果路由选项为默认关键字，则根据配置文件设置路由选项
    if routingOption == routingOptionDefaultKwd {
        routingOption = cfg.Routing.Type.WithDefault(routingOptionAutoKwd)
        if routingOption == "" {
            routingOption = routingOptionAutoKwd
        }
    }

    // 私有设置无法利用默认的 IPNIs 返回的对等体，将它们切换到仅使用 DHT
    // 为了避免破坏现有设置，将它们切换到仅使用 DHT
    if routingOption == routingOptionAutoKwd {
        if key, _ := repo.SwarmKey(); key != nil || pnet.ForcePrivateNetwork {
            log.Error("私有网络 (swarm.key / LIBP2P_FORCE_PNET) 无法与启用 Routing.Type=auto 的公共 HTTP IPNIs 配合使用。Kubo 将使用 Routing.Type=dht。更新配置以删除此消息。")
            routingOption = routingOptionDHTKwd
        }
    }

    // 根据路由选项进行不同的操作
    switch routingOption {
    case routingOptionSupernodeKwd:
        return errors.New("超级节点路由从未完全实现，并已被移除")
    case routingOptionDefaultKwd, routingOptionAutoKwd:
        ncfg.Routing = libp2p.ConstructDefaultRouting(cfg, libp2p.DHTOption)
    case routingOptionAutoClientKwd:
        ncfg.Routing = libp2p.ConstructDefaultRouting(cfg, libp2p.DHTClientOption)
    // 根据路由选项设置节点的路由类型
    case routingOptionDHTClientKwd:
        ncfg.Routing = libp2p.DHTClientOption
    // 根
// 警告信息，提醒用户使用了被 Ed25519 取代的 RSA Peer ID，建议更改公钥类型
// 提供更改公钥类型的命令
// 更改公钥类型后，需要重启节点使更改生效

})

// 延迟执行的函数，用于关闭节点
defer func() {
    // 等待节点关闭，因为节点可能有子节点需要等待关闭，比如 API 服务器
    node.Close()

    select {
    case <-req.Context.Done():
        log.Info("Gracefully shut down daemon")
    default:
    }
}()

cctx.ConstructNode = func() (*core.IpfsNode, error) {
    return node, nil
}

// 启动 "core" 插件，在启动 HTTP API 之前执行，因为用户可能依赖这些插件
err = cctx.Plugins.Start(node)
if err != nil {
    return err
}
node.Process.AddChild(goprocess.WithTeardown(cctx.Plugins.Close))

// 构建 API 端点 - 每次都执行
apiErrc, err := serveHTTPApi(req, cctx)
if err != nil {
    return err
}

// 构建 fuse 挂载点 - 如果用户提供了 --mount 标志
mount, _ := req.Options[mountKwd].(bool)
if mount && offline {
    return cmds.Errorf(cmds.ErrClient, "mount is not currently supported in offline mode")
}
if mount {
    if err := mountFuse(req, cctx); err != nil {
        return err
    }
}

// 仓库块存储 GC - 如果存在 --enable-gc 标志
gcErrc, err := maybeRunGC(req, node)
if err != nil {
    return err
}

// 添加迁移下载的任何文件
    // 如果需要缓存迁移或者固定迁移
    if cacheMigrations || pinMigrations {
        // 向 IPFS 添加迁移，如果出错则打印错误信息
        err = addMigrations(cctx.Context(), node, fetcher, pinMigrations)
        if err != nil {
            fmt.Fprintln(os.Stderr, "Could not add migration to IPFS:", err)
        }
        // 删除下载目录，以防守护进程的生命周期中保留它，或者在守护进程强制退出时留下它
        os.RemoveAll(migrations.DownloadDirectory)
        migrations.DownloadDirectory = ""
    }
    // 如果存在 fetcher
    if fetcher != nil {
        // 如果关闭 IpfsFetcher 出现错误，则打印错误，但不因此而失败
        err = fetcher.Close()
        if err != nil {
            log.Errorf("error closing IPFS fetcher: %s", err)
        }
    }

    // 构建 HTTP 网关
    gwErrc, err := serveHTTPGateway(req, cctx)
    if err != nil {
        return err
    }

    // 添加基于 libp2p 的不可信任网关
    p2pGwErrc, err := serveTrustlessGatewayOverLibp2p(cctx)
    if err != nil {
        return err
    }

    // 将 IPFS 版本信息添加到 Prometheus 指标
    ipfsInfoMetric := promauto.NewGaugeVec(prometheus.GaugeOpts{
        Name: "ipfs_info",
        Help: "IPFS version information.",
    }, []string{"version", "commit"})

    // 将其设置为 1，以便我们可以将其与其他统计数据相乘，以添加版本标签
    ipfsInfoMetric.With(prometheus.Labels{
        "version": version.CurrentVersionNumber,
        "commit":  version.CurrentCommit,
    }).Set(1)

    // TODO（9285）：使指标更具可配置性
    // 初始化指标收集器
    prometheus.MustRegister(&corehttp.IpfsNodeCollector{Node: node})

    // 启动 MFS 固定线程
    startPinMFS(daemonConfigPollInterval, cctx, &ipfsPinMFSNode{node})

    // 守护进程 *终于* 准备好了。
    fmt.Printf("Daemon is ready\n")
    notifyReady()

    // 当用户按下 C-c 时，给予用户一些即时反馈
    // 创建一个 goroutine，监听请求的上下文是否被取消，如果取消则通知停止
    go func() {
        <-req.Context.Done()
        notifyStopping()
        fmt.Println("Received interrupt signal, shutting down...")
        fmt.Println("(Hit ctrl-c again to force-shutdown the daemon.)")
    }()

    // 如果不是离线模式，1 分钟后给用户一个提示，如果在线模式下没有对等节点
    if !offline {
        time.AfterFunc(1*time.Minute, func() {
            // 获取配置信息
            cfg, err := cctx.GetConfig()
            if err != nil {
                log.Errorf("failed to access config: %s", err)
            }
            // 如果 Bootstrap 和 Peering 列表都为空，则跳过对等节点检查
            if len(cfg.Bootstrap) == 0 && len(cfg.Peering.Peers) == 0 {
                log.Warn("skipping bootstrap: empty Bootstrap and Peering lists")
                return
            }
            // 创建 CoreAPI 对象
            ipfs, err := coreapi.NewCoreAPI(node)
            if err != nil {
                log.Errorf("failed to access CoreAPI: %v", err)
            }
            // 获取对等节点信息
            peers, err := ipfs.Swarm().Peers(cctx.Context())
            if err != nil {
                log.Errorf("failed to read swarm peers: %v", err)
            }
            // 如果没有对等节点，则输出警告信息
            if len(peers) == 0 {
                log.Error("failed to bootstrap (no peers found): consider updating Bootstrap or Peering section of your config")
            }
        })
    }

    // 如果环境变量 IPFS_REUSEPORT 存在，则输出错误信息
    if flag := os.Getenv("IPFS_REUSEPORT"); flag != "" {
        log.Fatal("Support for IPFS_REUSEPORT was removed. Use LIBP2P_TCP_REUSEPORT instead.")
    }

    // 收集长时间运行的错误并阻塞等待关闭
    // TODO(cryptix): 我们的 fuse 目前不遵循此模式进行优雅关闭
    var errs error
    for err := range merge(apiErrc, gwErrc, gcErrc, p2pGwErrc) {
        if err != nil {
            errs = multierror.Append(errs, err)
        }
    }

    return errs
// serveHTTPApi函数收集选项，创建监听器，打印状态消息并开始处理请求。
func serveHTTPApi(req *cmds.Request, cctx *oldcmds.Context) (<-chan error, error) {
    // 获取配置信息
    cfg, err := cctx.GetConfig()
    if err != nil {
        return nil, fmt.Errorf("serveHTTPApi: GetConfig() failed: %s", err)
    }

    // 获取监听器
    listeners, err := sockets.TakeListeners("io.ipfs.api")
    if err != nil {
        return nil, fmt.Errorf("serveHTTPApi: socket activation failed: %s", err)
    }

    // 创建API地址列表
    apiAddrs := make([]string, 0, 2)
    apiAddr, _ := req.Options[commands.ApiOption].(string)
    if apiAddr == "" {
        apiAddrs = cfg.Addresses.API
    } else {
        apiAddrs = append(apiAddrs, apiAddr)
    }

    // 创建监听器地址映射
    listenerAddrs := make(map[string]bool, len(listeners))
    for _, listener := range listeners {
        listenerAddrs[string(listener.Multiaddr().Bytes())] = true
    }

    // 遍历API地址列表，创建监听器
    for _, addr := range apiAddrs {
        apiMaddr, err := ma.NewMultiaddr(addr)
        if err != nil {
            return nil, fmt.Errorf("serveHTTPApi: invalid API address: %q (err: %s)", addr, err)
        }
        if listenerAddrs[string(apiMaddr.Bytes())] {
            continue
        }

        apiLis, err := manet.Listen(apiMaddr)
        if err != nil {
            return nil, fmt.Errorf("serveHTTPApi: manet.Listen(%s) failed: %s", apiMaddr, err)
        }

        listenerAddrs[string(apiMaddr.Bytes())] = true
        listeners = append(listeners, apiLis)
    }

    // 打印RPC API访问受API.Authorizations定义的规则限制
    if len(cfg.API.Authorizations) > 0 && len(listeners) > 0 {
        fmt.Printf("RPC API access is limited by the rules defined in API.Authorizations\n")
    }
}
    // 遍历监听器列表
    for _, listener := range listeners {
        // 打印 RPC API 服务器监听的地址
        fmt.Printf("RPC API server listening on %s\n", listener.Multiaddr())
        // 检查监听器的网络类型，如果是 TCP 则输出 WebUI 的地址
        switch listener.Addr().Network() {
        case "tcp", "tcp4", "tcp6":
            fmt.Printf("WebUI: http://%s/webui\n", listener.Addr())
        }
    }

    // 默认情况下，不允许通过 API 加载任意的 ipfs 对象，因为这会导致脚本漏洞
    // 只允许加载 webui 对象
    // 如果你知道你在做什么，可以传递 --unrestricted-api 参数
    unrestricted, _ := req.Options[unrestrictedAPIAccessKwd].(bool)
    // 设置网关选项
    gatewayOpt := corehttp.GatewayOption(corehttp.WebUIPaths...)
    if unrestricted {
        gatewayOpt = corehttp.GatewayOption("/ipfs", "/ipns")
    }

    // 设置服务选项
    opts := []corehttp.ServeOption{
        corehttp.MetricsCollectionOption("api"),
        corehttp.MetricsOpenCensusCollectionOption(),
        corehttp.MetricsOpenCensusDefaultPrometheusRegistry(),
        corehttp.CheckVersionOption(),
        corehttp.CommandsOption(*cctx),
        corehttp.WebUIOption,
        gatewayOpt,
        corehttp.VersionOption(),
        defaultMux("/debug/vars"),
        defaultMux("/debug/pprof/"),
        defaultMux("/debug/stack"),
        corehttp.MutexFractionOption("/debug/pprof-mutex/"),
        corehttp.BlockProfileRateOption("/debug/pprof-block/"),
        corehttp.MetricsScrapingOption("/debug/metrics/prometheus"),
        corehttp.LogOption(),
    }

    // 如果网关的根重定向不为空，则添加重定向选项
    if len(cfg.Gateway.RootRedirect) > 0 {
        opts = append(opts, corehttp.RedirectOption("", cfg.Gateway.RootRedirect))
    }

    // 构建节点
    node, err := cctx.ConstructNode()
    if err != nil {
        return nil, fmt.Errorf("serveHTTPApi: ConstructNode() failed: %s", err)
    }
    // 如果监听器列表的长度大于0
    if len(listeners) > 0 {
        // 只有在 API 运行时才添加一个 API 文件
        if err := node.Repo.SetAPIAddr(rewriteMaddrToUseLocalhostIfItsAny(listeners[0].Multiaddr())); err != nil {
            return nil, fmt.Errorf("serveHTTPApi: SetAPIAddr() failed: %w", err)
        }
    }

    // 创建一个错误通道
    errc := make(chan error)
    // 创建一个同步等待组
    var wg sync.WaitGroup
    // 遍历监听器列表
    for _, apiLis := range listeners {
        // 增加等待组的计数
        wg.Add(1)
        // 启动一个 goroutine 处理 HTTP 服务
        go func(lis manet.Listener) {
            // 减少等待组的计数
            defer wg.Done()
            // 将错误发送到错误通道
            errc <- corehttp.Serve(node, manet.NetListener(lis), opts...)
        }(apiLis)
    }

    // 启动一个 goroutine 等待所有处理完成
    go func() {
        // 等待等待组中的所有 goroutine 完成
        wg.Wait()
        // 关闭错误通道
        close(errc)
    }()

    // 返回错误通道和空值
    return errc, nil
// 重新编写 maddr，如果是任何地址，则使用 localhost
func rewriteMaddrToUseLocalhostIfItsAny(maddr ma.Multiaddr) ma.Multiaddr {
    // 分离 maddr 的第一个地址和剩余部分
    first, rest := ma.SplitFirst(maddr)

    switch {
    // 如果第一个地址是 IPv4 未指定地址，则返回 IPv4 回环地址加上剩余部分
    case first.Equal(manet.IP4Unspecified):
        return manet.IP4Loopback.Encapsulate(rest)
    // 如果第一个地址是 IPv6 未指定地址，则返回 IPv6 回环地址加上剩余部分
    case first.Equal(manet.IP6Unspecified):
        return manet.IP6Loopback.Encapsulate(rest)
    // 如果不是 IP 地址，则直接返回 maddr
    default:
        return maddr // not ip
    }
}

// 打印主机的 Swarm 地址
func printSwarmAddrs(node *core.IpfsNode) {
    // 如果节点不在线，则打印信息并返回
    if !node.IsOnline {
        fmt.Println("Swarm not listening, running in offline mode.")
        return
    }

    // 获取节点的接口地址
    ifaceAddrs, err := node.PeerHost.Network().InterfaceListenAddresses()
    if err != nil {
        log.Errorf("failed to read listening addresses: %s", err)
    }
    // 将接口地址转换为字符串并排序
    lisAddrs := make([]string, len(ifaceAddrs))
    for i, addr := range ifaceAddrs {
        lisAddrs[i] = addr.String()
    }
    sort.Strings(lisAddrs)
    // 打印 Swarm 监听地址
    for _, addr := range lisAddrs {
        fmt.Printf("Swarm listening on %s\n", addr)
    }

    // 获取节点的地址并排序
    nodePhostAddrs := node.PeerHost.Addrs()
    addrs := make([]string, len(nodePhostAddrs))
    for i, addr := range nodePhostAddrs {
        addrs[i] = addr.String()
    }
    sort.Strings(addrs)
    // 打印 Swarm 公布地址
    for _, addr := range addrs {
        fmt.Printf("Swarm announcing %s\n", addr)
    }
}

// 服务 HTTP 网关，收集选项，创建监听器，打印状态消息并开始处理请求
func serveHTTPGateway(req *cmds.Request, cctx *oldcmds.Context) (<-chan error, error) {
    // 获取配置信息
    cfg, err := cctx.GetConfig()
    if err != nil {
        return nil, fmt.Errorf("serveHTTPGateway: GetConfig() failed: %s", err)
    }

    // 获取 writable 选项，如果不存在则使用默认值
    writable, writableOptionFound := req.Options[writableKwd].(bool)
    if !writableOptionFound {
        writable = cfg.Gateway.Writable.WithDefault(false)
    }
}
    // 如果可写标志为真，则输出错误信息并终止程序
    if writable {
        log.Fatalf("Support for Gateway.Writable and --writable has been REMOVED. Please remove it from your config file or CLI. Modern replacement tracked in https://github.com/ipfs/specs/issues/375")
    }

    // 从指定名称空间获取监听器
    listeners, err := sockets.TakeListeners("io.ipfs.gateway")
    if err != nil {
        return nil, fmt.Errorf("serveHTTPGateway: socket activation failed: %s", err)
    }

    // 创建监听器地址的映射
    listenerAddrs := make(map[string]bool, len(listeners))
    for _, listener := range listeners {
        listenerAddrs[string(listener.Multiaddr().Bytes())] = true
    }

    // 获取网关地址并进行处理
    gatewayAddrs := cfg.Addresses.Gateway
    for _, addr := range gatewayAddrs {
        gatewayMaddr, err := ma.NewMultiaddr(addr)
        if err != nil {
            return nil, fmt.Errorf("serveHTTPGateway: invalid gateway address: %q (err: %s)", addr, err)
        }

        // 如果监听器地址已存在，则继续下一次循环
        if listenerAddrs[string(gatewayMaddr.Bytes())] {
            continue
        }

        // 监听网关地址
        gwLis, err := manet.Listen(gatewayMaddr)
        if err != nil {
            return nil, fmt.Errorf("serveHTTPGateway: manet.Listen(%s) failed: %s", gatewayMaddr, err)
        }
        listenerAddrs[string(gatewayMaddr.Bytes())] = true
        listeners = append(listeners, gwLis)
    }

    // 输出网关服务器监听的地址
    for _, listener := range listeners {
        fmt.Printf("Gateway server listening on %s\n", listener.Multiaddr())
    }

    // 如果配置中暴露路由API，则输出路由V1 API的地址
    if cfg.Gateway.ExposeRoutingAPI.WithDefault(config.DefaultExposeRoutingAPI) {
        for _, listener := range listeners {
            fmt.Printf("Routing V1 API exposed at http://%s/routing/v1\n", listener.Addr())
        }
    }

    // 复制上下文并设置网关标志为真
    cmdctx := *cctx
    cmdctx.Gateway = true
    // 创建一个选项列表，用于配置HTTP服务器
    opts := []corehttp.ServeOption{
        // 添加指标收集选项，用于监控网关性能
        corehttp.MetricsCollectionOption("gateway"),
        // 添加主机名选项，用于配置主机名
        corehttp.HostnameOption(),
        // 添加网关选项，指定IPFS和IPNS的网关路径
        corehttp.GatewayOption("/ipfs", "/ipns"),
        // 添加版本选项，用于返回IPFS节点的版本信息
        corehttp.VersionOption(),
        // 添加版本检查选项，用于检查IPFS节点的版本是否最新
        corehttp.CheckVersionOption(),
        // 添加只读命令选项，用于配置只读命令上下文
        corehttp.CommandsROOption(cmdctx),
    }

    // 如果启用了实验性的P2P HTTP代理，则添加P2P代理选项
    if cfg.Experimental.P2pHttpProxy {
        opts = append(opts, corehttp.P2PProxyOption())
    }

    // 如果配置中暴露了路由API，则添加路由选项
    if cfg.Gateway.ExposeRoutingAPI.WithDefault(config.DefaultExposeRoutingAPI) {
        opts = append(opts, corehttp.RoutingOption())
    }

    // 如果配置中设置了根重定向路径，则添加重定向选项
    if len(cfg.Gateway.RootRedirect) > 0 {
        opts = append(opts, corehttp.RedirectOption("", cfg.Gateway.RootRedirect))
    }

    // 如果配置中设置了自定义的Gateway.PathPrefixes，则输出错误信息并终止程序
    if len(cfg.Gateway.PathPrefixes) > 0 {
        log.Fatal("Support for custom Gateway.PathPrefixes was removed: https://github.com/ipfs/go-ipfs/issues/7702")
    }

    // 构建IPFS节点
    node, err := cctx.ConstructNode()
    if err != nil {
        return nil, fmt.Errorf("serveHTTPGateway: ConstructNode() failed: %s", err)
    }

    // 如果有监听器，则设置网关地址
    if len(listeners) > 0 {
        addr, err := manet.ToNetAddr(rewriteMaddrToUseLocalhostIfItsAny(listeners[0].Multiaddr()))
        if err != nil {
            return nil, fmt.Errorf("serveHTTPGateway: manet.ToIP() failed: %w", err)
        }
        if err := node.Repo.SetGatewayAddr(addr); err != nil {
            return nil, fmt.Errorf("serveHTTPGateway: SetGatewayAddr() failed: %w", err)
        }
    }

    // 创建一个错误通道和等待组
    errc := make(chan error)
    var wg sync.WaitGroup
    // 遍历监听器，为每个监听器启动一个goroutine
    for _, lis := range listeners {
        wg.Add(1)
        go func(lis manet.Listener) {
            defer wg.Done()
            errc <- corehttp.Serve(node, manet.NetListener(lis), opts...)
        }(lis)
    }

    // 启动一个goroutine等待所有监听器的goroutine结束
    go func() {
        wg.Wait()
        close(errc)
    }()

    // 返回错误通道和nil
    return errc, nil
// 定义网关协议的ID
const gatewayProtocolID protocol.ID = "/ipfs/gateway" // FIXME: specify https://github.com/ipfs/specs/issues/433

// 在Libp2p上提供无信任网关服务
func serveTrustlessGatewayOverLibp2p(cctx *oldcmds.Context) (<-chan error, error) {
    // 构建节点
    node, err := cctx.ConstructNode()
    if err != nil {
        return nil, fmt.Errorf("serveHTTPGatewayOverLibp2p: ConstructNode() failed: %s", err)
    }
    // 读取配置
    cfg, err := node.Repo.Config()
    if err != nil {
        return nil, fmt.Errorf("could not read config: %w", err)
    }

    // 如果配置中未启用Libp2p网关，则返回空通道和nil
    if !cfg.Experimental.GatewayOverLibp2p {
        errCh := make(chan error)
        close(errCh)
        return errCh, nil
    }

    // 设置选项
    opts := []corehttp.ServeOption{
        corehttp.MetricsCollectionOption("libp2p-gateway"),
        corehttp.Libp2pGatewayOption(),
        corehttp.VersionOption(),
    }

    // 创建处理程序
    handler, err := corehttp.MakeHandler(node, nil, opts...)
    if err != nil {
        return nil, err
    }

    // 设置Libp2p主机
    h := p2phttp.Host{
        StreamHost: node.PeerHost,
    }

    // 设置临时协议和处理程序
    tmpProtocol := protocol.ID("/kubo/delete-me")
    h.SetHTTPHandler(tmpProtocol, http.NotFoundHandler())
    h.WellKnownHandler.RemoveProtocolMeta(tmpProtocol)

    // 添加网关协议和处理程序
    h.WellKnownHandler.AddProtocolMeta(gatewayProtocolID, p2phttp.ProtocolMeta{Path: "/"})
    h.ServeMux = http.NewServeMux()
    h.ServeMux.Handle("/", handler)

    // 创建错误通道
    errc := make(chan error, 1)
    // 启动服务
    go func() {
        defer close(errc)
        errc <- h.Serve()
    }()

    // 监听节点关闭事件，关闭Libp2p主机
    go func() {
        <-node.Process.Closing()
        h.Close()
    }()

    return errc, nil
}

// 收集选项并打开Fuse挂载点
func mountFuse(req *cmds.Request, cctx *oldcmds.Context) error {
    // 获取配置
    cfg, err := cctx.GetConfig()
    if err != nil {
        return fmt.Errorf("mountFuse: GetConfig() failed: %s", err)
    }

    // 获取IPFS挂载目录
    fsdir, found := req.Options[ipfsMountKwd].(string)
    if !found {
        fsdir = cfg.Mounts.IPFS
    }

    // 获取IPNS挂载目录
    nsdir, found := req.Options[ipnsMountKwd].(string)
    if !found {
        nsdir = cfg.Mounts.IPNS
    }
    // 使用 cctx 构造一个节点
    node, err := cctx.ConstructNode()
    // 如果构造节点出现错误，则返回错误信息
    if err != nil {
        return fmt.Errorf("mountFuse: ConstructNode() failed: %s", err)
    }

    // 将节点挂载到指定的文件系统目录和命名空间目录
    err = nodeMount.Mount(node, fsdir, nsdir)
    // 如果挂载出现错误，则返回错误信息
    if err != nil {
        return err
    }
    // 打印 IPFS 挂载的目录
    fmt.Printf("IPFS mounted at: %s\n", fsdir)
    // 打印 IPNS 挂载的目录
    fmt.Printf("IPNS mounted at: %s\n", nsdir)
    // 没有错误发生，返回空
    return nil
}

// maybeRunGC 根据请求和节点执行垃圾回收
func maybeRunGC(req *cmds.Request, node *core.IpfsNode) (<-chan error, error) {
    // 从请求选项中获取 enableGCKwd 对应的布尔值
    enableGC, _ := req.Options[enableGCKwd].(bool)
    // 如果 enableGC 为 false，则返回空通道和空错误
    if !enableGC {
        return nil, nil
    }

    // 创建一个错误通道
    errc := make(chan error)
    // 启动一个 goroutine 执行 PeriodicGC，并将结果发送到 errc 通道
    go func() {
        errc <- corerepo.PeriodicGC(req.Context, node)
        close(errc)
    }()
    return errc, nil
}

// merge 执行多个只读错误通道的扇入
// 参考自 http://blog.golang.org/pipelines
func merge(cs ...<-chan error) <-chan error {
    var wg sync.WaitGroup
    out := make(chan error)

    // 为每个输入通道启动一个输出 goroutine。output 从 c 复制值到 out，直到 c 关闭，然后调用 wg.Done。
    output := func(c <-chan error) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    for _, c := range cs {
        if c != nil {
            wg.Add(1)
            go output(c)
        }
    }

    // 启动一个 goroutine 在所有输出 goroutine 完成后关闭 out。这必须在 wg.Add 调用之后开始。
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

// YesNoPrompt 提示用户输入 yes 或 no
func YesNoPrompt(prompt string) bool {
    var s string
    for i := 0; i < 3; i++ {
        fmt.Printf("%s ", prompt)
        fmt.Scanf("%s", &s)
        switch s {
        case "y", "Y":
            return true
        case "n", "N":
            return false
        case "":
            return false
        }
        fmt.Println("Please press either 'y' or 'n'")
    }

    return false
}

// printVersion 打印版本信息
func printVersion() {
    v := version.CurrentVersionNumber
    if version.CurrentCommit != "" {
        v += "-" + version.CurrentCommit
    }
    fmt.Printf("Kubo version: %s\n", v)
    fmt.Printf("Repo version: %d\n", fsrepo.RepoVersion)
    fmt.Printf("System version: %s\n", runtime.GOARCH+"/"+runtime.GOOS)
    fmt.Printf("Golang version: %s\n", runtime.Version())
}
```