# `kubo\core\core.go`

```
/*
Package core implements the IpfsNode object and related methods.

Packages underneath core/ provide a (relatively) stable, low-level API
to carry out most IPFS-related tasks.  For more details on the other
interfaces and how core/... fits into the bigger IPFS picture, see:

    $ godoc github.com/ipfs/go-ipfs
*/
package core

import (
    "context"  // 导入 context 包，用于处理请求的取消、超时等
    "encoding/json"  // 导入 encoding/json 包，用于 JSON 数据的编解码
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "time"  // 导入 time 包，用于处理时间相关的操作

    "github.com/ipfs/boxo/filestore"  // 导入文件存储相关的包
    pin "github.com/ipfs/boxo/pinning/pinner"  // 导入 pinning/pinner 包，并将其命名为 pin
    "github.com/ipfs/go-datastore"  // 导入数据存储相关的包

    bserv "github.com/ipfs/boxo/blockservice"  // 导入 blockservice 包，并将其命名为 bserv
    bstore "github.com/ipfs/boxo/blockstore"  // 导入 blockstore 包，并将其命名为 bstore
    exchange "github.com/ipfs/boxo/exchange"  // 导入 exchange 包
    "github.com/ipfs/boxo/fetcher"  // 导入 fetcher 包
    mfs "github.com/ipfs/boxo/mfs"  // 导入 mfs 包
    pathresolver "github.com/ipfs/boxo/path/resolver"  // 导入 path/resolver 包
    provider "github.com/ipfs/boxo/provider"  // 导入 provider 包
    ipld "github.com/ipfs/go-ipld-format"  // 导入 go-ipld-format 包
    logging "github.com/ipfs/go-log"  // 导入 go-log 包
    goprocess "github.com/jbenet/goprocess"  // 导入 goprocess 包
    ddht "github.com/libp2p/go-libp2p-kad-dht/dual"  // 导入 go-libp2p-kad-dht/dual 包
    pubsub "github.com/libp2p/go-libp2p-pubsub"  // 导入 go-libp2p-pubsub 包
    psrouter "github.com/libp2p/go-libp2p-pubsub-router"  // 导入 go-libp2p-pubsub-router 包
    record "github.com/libp2p/go-libp2p-record"  // 导入 go-libp2p-record 包
    connmgr "github.com/libp2p/go-libp2p/core/connmgr"  // 导入 core/connmgr 包
    ic "github.com/libp2p/go-libp2p/core/crypto"  // 导入 core/crypto 包
    p2phost "github.com/libp2p/go-libp2p/core/host"  // 导入 core/host 包
    metrics "github.com/libp2p/go-libp2p/core/metrics"  // 导入 core/metrics 包
    "github.com/libp2p/go-libp2p/core/network"  // 导入 core/network 包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入 core/peer 包
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 core/peerstore 包
    routing "github.com/libp2p/go-libp2p/core/routing"  // 导入 core/routing 包
    "github.com/libp2p/go-libp2p/p2p/discovery/mdns"  // 导入 p2p/discovery/mdns 包
    p2pbhost "github.com/libp2p/go-libp2p/p2p/host/basic"  // 导入 p2p/host/basic 包
    ma "github.com/multiformats/go-multiaddr"  // 导入 go-multiaddr 包
    madns "github.com/multiformats/go-multiaddr-dns"  // 导入 go-multiaddr-dns 包

    "github.com/ipfs/boxo/bootstrap"  // 导入 bootstrap 包
    "github.com/ipfs/boxo/namesys"  // 导入 namesys 包
    ipnsrp "github.com/ipfs/boxo/namesys/republisher"  // 导入 namesys/republisher 包，并将其命名为 ipnsrp
    "github.com/ipfs/boxo/peering"  // 导入 peering 包
    "github.com/ipfs/kubo/config"  // 导入 config 包
    # 导入 kubo/core/node 模块
    "github.com/ipfs/kubo/core/node"
    # 导入 kubo/core/node/libp2p 模块
    "github.com/ipfs/kubo/core/node/libp2p"
    # 导入 kubo/fuse/mount 模块
    "github.com/ipfs/kubo/fuse/mount"
    # 导入 kubo/p2p 模块
    "github.com/ipfs/kubo/p2p"
    # 导入 kubo/repo 模块
    "github.com/ipfs/kubo/repo"
    # 导入 kubo/routing 模块，并重命名为 irouting
    irouting "github.com/ipfs/kubo/routing"
// 创建名为log的日志记录器
var log = logging.Logger("core")

// IpfsNode是IPFS核心模块。它代表一个IPFS实例。
type IpfsNode struct {
    // Self
    Identity peer.ID // 本地节点的身份标识

    Repo repo.Repo // IPFS节点的数据存储库

    // Local node
    Pinning         pin.Pinner             // pinning管理器
    Mounts          Mounts                 `optional:"true"` // 当前挂载状态（如果有的话）
    PrivateKey      ic.PrivKey             `optional:"true"` // 本地节点的私钥
    PNetFingerprint libp2p.PNetFingerprint `optional:"true"` // 私有网络的指纹

    // Services
    Peerstore                   pstore.Peerstore          `optional:"true"` // 存储其他对等节点实例的存储库
    Blockstore                  bstore.GCBlockstore       // 块存储（较低级别）
    Filestore                   *filestore.Filestore      `optional:"true"` // 文件存储块存储
    BaseBlocks                  node.BaseBlocks           // 原始块存储，没有文件存储包装
    GCLocker                    bstore.GCLocker           // 在垃圾收集期间保护块存储的锁
    Blocks                      bserv.BlockService        // 块服务，获取/添加块
    DAG                         ipld.DAGService           // Merkle DAG服务，获取/添加对象
    IPLDFetcherFactory          fetcher.Factory           `name:"ipldFetcher"`          // 在IPLD数据模型上进行路径处理的获取器
    UnixFSFetcherFactory        fetcher.Factory           `name:"unixfsFetcher"`        // 解释UnixFS数据的获取器
    OfflineIPLDFetcherFactory   fetcher.Factory           `name:"offlineIpldFetcher"`   // 在不获取新块的情况下在IPLD数据模型上进行路径处理的获取器
    OfflineUnixFSFetcherFactory fetcher.Factory           `name:"offlineUnixfsFetcher"` // 在不获取新块的情况下解释UnixFS数据的获取器
    Reporter                    *metrics.BandwidthCounter `optional:"true"`
    // 发现服务，使用 mdns 协议，可选
    Discovery                   mdns.Service              `optional:"true"`
    // 文件系统根目录
    FilesRoot                   *mfs.Root
    // 记录验证器
    RecordValidator             record.Validator

    // 在线
    // 网络主机，包括服务器和客户端，可选
    PeerHost                  p2phost.Host               `optional:"true"`
    // 对等网络服务
    Peering                   *peering.PeeringService    `optional:"true"`
    // 地址过滤器
    Filters                   *ma.Filters                `optional:"true"`
    // 定期引导程序
    Bootstrapper              io.Closer                  `optional:"true"`
    // 路由系统，推荐使用 ipfs-dht
    Routing                   irouting.ProvideManyRouter `optional:"true"`
    // DNS 解析器
    DNSResolver               *madns.Resolver            // the DNS resolver
    // IPLD 路径解析器
    IPLDPathResolver          pathresolver.Resolver      `name:"ipldPathResolver"`          // The IPLD path resolver
    // UnixFS 路径解析器
    UnixFSPathResolver        pathresolver.Resolver      `name:"unixFSPathResolver"`        // The UnixFS path resolver
    // 离线 IPLD 路径解析器，仅使用本地可用块
    OfflineIPLDPathResolver   pathresolver.Resolver      `name:"offlineIpldPathResolver"`   // The IPLD path resolver that uses only locally available blocks
    // 离线 UnixFS 路径解析器，仅使用本地可用块
    OfflineUnixFSPathResolver pathresolver.Resolver      `name:"offlineUnixFSPathResolver"` // The UnixFS path resolver that uses only locally available blocks
    // 区块交换 + 策略 (bitswap)
    Exchange                  exchange.Interface         // the block exchange + strategy (bitswap)
    // 名称系统，将路径解析为哈希
    Namesys                   namesys.NameSystem         // the name system, resolves paths to hashes
    // 值提供者系统
    Provider                  provider.System            // the value provider system
    // IpnsRepub 发布器，可选
    IpnsRepub                 *ipnsrp.Republisher        `optional:"true"`
    // 资源管理器，可选
    ResourceManager           network.ResourceManager    `optional:"true"`

    // 发布订阅系统
    PubSub   *pubsub.PubSub             `optional:"true"`
    // 发布订阅路由
    PSRouter *psrouter.PubsubValueStore `optional:"true"`

    // 分布式哈希表
    DHT       *ddht.DHT       `optional:"true"`
    // DHT 客户端
    DHTClient routing.Routing `name:"dhtc" optional:"true"`

    // P2P 网络
    P2P *p2p.P2P `optional:"true"`
    // 创建一个名为 goprocess.Process 的变量，用于处理进程
    Process goprocess.Process
    // 创建一个名为 ctx 的变量，用于存储上下文信息
    ctx     context.Context

    // 创建一个名为 stop 的函数变量，用于停止进程并返回错误信息
    stop func() error

    // 标志
    // 创建一个名为 IsOnline 的布尔类型变量，可选，表示网络是否已启用
    IsOnline bool `optional:"true"` 
    // 创建一个名为 IsDaemon 的布尔类型变量，可选，表示是否在长时间运行的守护进程上运行
    IsDaemon bool `optional:"true"` 
// Mounts 定义了节点的挂载状态。这可能应该移动到守护程序或挂载。它在这里是因为需要在守护程序请求中访问。
type Mounts struct {
    Ipfs mount.Mount
    Ipns mount.Mount
}

// Close 在 App 对象上调用 Close() 方法
func (n *IpfsNode) Close() error {
    return n.stop()
}

// Context 返回 IpfsNode 的上下文
func (n *IpfsNode) Context() context.Context {
    if n.ctx == nil {
        n.ctx = context.TODO()
    }
    return n.ctx
}

// Bootstrap 将设置并调用 IpfsNode 的引导程序函数。
func (n *IpfsNode) Bootstrap(cfg bootstrap.BootstrapConfig) error {
    // TODO 在离线模式下应该返回什么值？
    if n.Routing == nil {
        return nil
    }

    if n.Bootstrapper != nil {
        n.Bootstrapper.Close() // 停止先前的引导过程。
    }

    // 如果调用者没有指定引导对等函数，则从配置中获取最新的引导对等。这响应实时更改。
    if cfg.BootstrapPeers == nil {
        cfg.BootstrapPeers = func() []peer.AddrInfo {
            ps, err := n.loadBootstrapPeers()
            if err != nil {
                log.Warn("failed to parse bootstrap peers from config")
                return nil
            }
            return ps
        }
    }
    if load, _ := cfg.BackupPeers(); load == nil {
        save := func(ctx context.Context, peerList []peer.AddrInfo) {
            err := n.saveTempBootstrapPeers(ctx, peerList)
            if err != nil {
                log.Warnf("saveTempBootstrapPeers failed: %s", err)
                return
            }
        }
        load = func(ctx context.Context) []peer.AddrInfo {
            peerList, err := n.loadTempBootstrapPeers(ctx)
            if err != nil {
                log.Warnf("loadTempBootstrapPeers failed: %s", err)
                return nil
            }
            return peerList
        }
        cfg.SetBackupPeers(load, save)
    }
}
    // 从存储库中获取配置信息
    repoConf, err := n.Repo.Config()
    // 如果获取配置信息出错，则返回错误
    if err != nil {
        return err
    }
    // 如果存储库配置中存在内部备份引导间隔，则将其赋值给cfg.BackupBootstrapInterval
    if repoConf.Internal.BackupBootstrapInterval != nil {
        cfg.BackupBootstrapInterval = repoConf.Internal.BackupBootstrapInterval.WithDefault(time.Hour)
    }

    // 使用配置信息进行引导
    n.Bootstrapper, err = bootstrap.Bootstrap(n.Identity, n.PeerHost, n.Routing, cfg)
    // 返回引导过程中的错误
    return err
# 定义了一个名为 TempBootstrapPeersKey 的变量，用于存储临时引导节点的数据存储键
var TempBootstrapPeersKey = datastore.NewKey("/local/temp_bootstrap_peers")

# 定义了一个名为 loadBootstrapPeers 的方法，用于加载引导节点信息
func (n *IpfsNode) loadBootstrapPeers() ([]peer.AddrInfo, error) {
    # 从存储库中获取配置信息
    cfg, err := n.Repo.Config()
    if err != nil {
        return nil, err
    }

    # 返回配置中的引导节点信息
    return cfg.BootstrapPeers()
}

# 定义了一个名为 saveTempBootstrapPeers 的方法，用于保存临时引导节点信息
func (n *IpfsNode) saveTempBootstrapPeers(ctx context.Context, peerList []peer.AddrInfo) error {
    # 获取数据存储
    ds := n.Repo.Datastore()
    # 将引导节点信息转换为 JSON 格式的字节流
    bytes, err := json.Marshal(config.BootstrapPeerStrings(peerList))
    if err != nil {
        return err
    }

    # 将转换后的引导节点信息存储到数据存储中
    if err := ds.Put(ctx, TempBootstrapPeersKey, bytes); err != nil {
        return err
    }
    # 同步数据存储
    return ds.Sync(ctx, TempBootstrapPeersKey)
}

# 定义了一个名为 loadTempBootstrapPeers 的方法，用于加载临时引导节点信息
func (n *IpfsNode) loadTempBootstrapPeers(ctx context.Context) ([]peer.AddrInfo, error) {
    # 获取数据存储
    ds := n.Repo.Datastore()
    # 从数据存储中获取临时引导节点信息的字节流
    bytes, err := ds.Get(ctx, TempBootstrapPeersKey)
    if err != nil {
        return nil, err
    }

    # 定义一个字符串数组，用于存储解析后的引导节点地址
    var addrs []string
    # 将获取的字节流解析为字符串数组
    if err := json.Unmarshal(bytes, &addrs); err != nil {
        return nil, err
    }
    # 解析字符串数组，返回引导节点信息
    return config.ParseBootstrapPeers(addrs)
}

# 定义了一个名为 ConstructPeerHostOpts 的结构体，用于存储构建对等主机的选项
type ConstructPeerHostOpts struct {
    AddrsFactory      p2pbhost.AddrsFactory
    DisableNatPortMap bool
    DisableRelay      bool
    EnableRelayHop    bool
    ConnectionManager connmgr.ConnManager
}
```