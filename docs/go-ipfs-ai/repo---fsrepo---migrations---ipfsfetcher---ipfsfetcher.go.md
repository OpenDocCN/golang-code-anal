# `kubo\repo\fsrepo\migrations\ipfsfetcher\ipfsfetcher.go`

```go
package ipfsfetcher

import (
    "context"  // 上下文包，用于控制函数调用的取消、超时等
    "encoding/json"  // 用于 JSON 数据的编码和解码
    "fmt"  // 用于格式化输出
    "io"  // 提供了基本的接口和函数，用于读写数据
    "net/url"  // 用于 URL 解析
    "os"  // 提供了操作系统功能的函数
    gopath "path"  // 提供了对路径操作的函数
    "strings"  // 提供了对字符串操作的函数
    "sync"  // 提供了同步原语的包

    "github.com/ipfs/boxo/files"  // IPFS 相关包
    "github.com/ipfs/boxo/path"  // IPFS 相关包
    "github.com/ipfs/kubo/config"  // IPFS 相关包
    "github.com/ipfs/kubo/core"  // IPFS 相关包
    "github.com/ipfs/kubo/core/coreapi"  // IPFS 相关包
    iface "github.com/ipfs/kubo/core/coreiface"  // IPFS 相关包
    "github.com/ipfs/kubo/core/coreiface/options"  // IPFS 相关包
    "github.com/ipfs/kubo/core/node/libp2p"  // IPFS 相关包
    "github.com/ipfs/kubo/repo/fsrepo"  // IPFS 相关包
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // IPFS 相关包
    peer "github.com/libp2p/go-libp2p/core/peer"  // libp2p 相关包
)

const (
    // 默认的最大下载大小
    defaultFetchLimit = 1024 * 1024 * 512

    tempNodeTCPAddr = "/ip4/127.0.0.1/tcp/0"  // 临时节点的 TCP 地址
)

type IpfsFetcher struct {
    distPath       string  // 分发路径
    limit          int64  // 下载限制
    repoRoot       *string  // 仓库根目录
    userConfigFile string  // 用户配置文件

    openOnce  sync.Once  // 一次性执行的同步原语
    openErr   error  // 打开错误
    closeOnce sync.Once  // 一次性执行的同步原语
    closeErr  error  // 关闭错误

    ipfs         iface.CoreAPI  // IPFS 核心 API
    ipfsTmpDir   string  // IPFS 临时目录
    ipfsStopFunc func()  // IPFS 停止函数

    fetched []path.Path  // 已获取的路径
    mutex   sync.Mutex  // 互斥锁
}

var _ migrations.Fetcher = (*IpfsFetcher)(nil)  // 实现 Fetcher 接口

// NewIpfsFetcher 创建一个新的 IpfsFetcher
//
// 如果 distPath 为 ""，则设置默认的 IPNS 路径。
// 如果 fetchLimit 为 0，则设置默认值，-1 表示无限制。
//
// 从 repoRoot 中读取 IPFS 配置文件中的引导和对等信息，除非 repoRoot 为 nil。
// 如果 repoRoot 为空（""），则从默认的 IPFS 目录中读取配置。
func NewIpfsFetcher(distPath string, fetchLimit int64, repoRoot *string, userConfigFile string) *IpfsFetcher {
    f := &IpfsFetcher{
        limit:          defaultFetchLimit,  // 设置默认的下载限制
        distPath:       migrations.LatestIpfsDist,  // 设置默认的分发路径
        repoRoot:       repoRoot,  // 设置仓库根目录
        userConfigFile: userConfigFile,  // 设置用户配置文件
    }

    if distPath != "" {
        if !strings.HasPrefix(distPath, "/") {
            distPath = "/" + distPath  // 如果 distPath 不是以 "/" 开头，则添加 "/"
        }
        f.distPath = distPath  // 设置分发路径为 distPath
    }
    # 如果fetchLimit不等于0
    if fetchLimit != 0:
        # 如果fetchLimit小于0，则将fetchLimit设置为0
        if fetchLimit < 0:
            fetchLimit = 0
        # 将f对象的limit属性设置为fetchLimit
        f.limit = fetchLimit
    # 返回f对象
    return f
// Close 方法用于关闭 IPFS 节点，确保只执行一次
func (f *IpfsFetcher) Close() error {
    f.closeOnce.Do(func() {
        if f.ipfsStopFunc != nil {
            // 告诉 IPFS 节点停止，并等待其停止
            f.ipfsStopFunc()
        }

        if f.ipfsTmpDir != "" {
            // 删除临时的 IPFS 目录
            f.closeErr = os.RemoveAll(f.ipfsTmpDir)
        }
    })
    return f.closeErr
}

// AddrInfo 方法返回 IPFS 节点的地址信息
func (f *IpfsFetcher) AddrInfo() peer.AddrInfo {
    return f.addrInfo
}

// FetchedPaths 方法返回此获取器获取的所有项目的 IPFS 路径
func (f *IpfsFetcher) FetchedPaths() []path.Path {
    f.mutex.Lock()
    defer f.mutex.Unlock()
    return f.fetched
}

// recordFetched 方法用于记录已获取的 IPFS 路径
func (f *IpfsFetcher) recordFetched(fetchedPath path.Path) {
    // 互斥锁用于保护免受并发调用Fetch的更新
    f.mutex.Lock()  // 获取互斥锁
    defer f.mutex.Unlock()  // 在函数返回时释放互斥锁
    f.fetched = append(f.fetched, fetchedPath)  // 将fetchedPath添加到f.fetched切片中
// 初始化临时节点，传入上下文、引导节点和对等节点信息，返回临时节点的目录路径和可能的错误
func initTempNode(ctx context.Context, bootstrap []string, peers []peer.AddrInfo) (string, error) {
    // 创建身份，将生成的密钥输出到 io.Discard，使用 options.KeyGenerateOption 配置生成的密钥类型为 options.Ed25519Key
    identity, err := config.CreateIdentity(io.Discard, []options.KeyGenerateOption{
        options.Key.Type(options.Ed25519Key),
    })
    if err != nil {
        return "", err
    }
    // 使用生成的身份初始化配置
    cfg, err := config.InitWithIdentity(identity)
    if err != nil {
        return "", err
    }

    // 创建临时的 ipfs 目录
    dir, err := os.MkdirTemp("", "ipfs-temp")
    if err != nil {
        return "", fmt.Errorf("failed to get temp dir: %s", err)
    }

    // 配置临时节点
    cfg.Routing.Type = config.NewOptionalString("dhtclient")

    // 禁用监听传入连接
    cfg.Addresses.Gateway = []string{}
    cfg.Addresses.API = []string{}
    cfg.Addresses.Swarm = []string{tempNodeTCPAddr}

    // 如果引导节点不为空，则配置引导节点
    if len(bootstrap) != 0 {
        cfg.Bootstrap = bootstrap
    }

    // 如果对等节点不为空，则配置对等节点
    if len(peers) != 0 {
        cfg.Peering.Peers = peers
    }

    // 假设仓库插件已经加载，初始化仓库
    err = fsrepo.Init(dir, cfg)
    if err != nil {
        os.RemoveAll(dir)
        return "", fmt.Errorf("failed to initialize ephemeral node: %s", err)
    }

    return dir, nil
}

// 启动临时节点，传入上下文，返回可能的错误
func (f *IpfsFetcher) startTempNode(ctx context.Context) error {
    // 打开仓库
    r, err := fsrepo.Open(f.ipfsTmpDir)
    if err != nil {
        return err
    }

    // 创建一个新的生命周期上下文，用于停止临时 ipfs 节点
    ctxIpfsLife, cancel := context.WithCancel(context.Background())

    // 构建节点
    node, err := core.NewNode(ctxIpfsLife, &core.BuildCfg{
        Online:  true,
        Routing: libp2p.DHTClientOption,
        Repo:    r,
    })
    if err != nil {
        cancel()
        r.Close()
        return err
    }

    // 创建 ipfs 核心 API
    ipfs, err := coreapi.NewCoreAPI(node)
    if err != nil {
        cancel()
        return err
    }
    stopFunc := func() {
        // Tell ipfs to stop
        cancel()
        // Wait until ipfs is stopped
        <-node.Context().Done()
    }

    addrs, err := ipfs.Swarm().LocalAddrs(ctx)
    if err != nil {
        // 如果获取本地 swarm 地址失败，只意味着无法通过临时节点获取已下载的迁移。
        // 因此，打印错误消息并继续执行。
        fmt.Fprintln(os.Stderr, "cannot get local swarm address:", err)
    }

    f.addrInfo = peer.AddrInfo{
        ID:    node.Identity,
        Addrs: addrs,
    }

    f.ipfs = ipfs
    f.ipfsStopFunc = stopFunc

    return nil
// 解析给定的路径字符串，返回对应的路径对象和可能的错误
func parsePath(fetchPath string) (path.Path, error) {
    // 尝试根据给定的路径字符串创建新的路径对象
    if ipfsPath, err := path.NewPath(fetchPath); err == nil {
        return ipfsPath, nil
    }

    // 如果创建路径对象失败，则尝试解析 URL
    u, err := url.Parse(fetchPath)
    if err != nil {
        return nil, fmt.Errorf("%q could not be parsed: %s", fetchPath, err)
    }

    // 根据 URL 的 scheme 进行不同的处理
    switch proto := u.Scheme; proto {
    case "ipfs", "ipld", "ipns":
        return path.NewPath(gopath.Join("/", proto, u.Host, u.Path))
    default:
        return nil, fmt.Errorf("%q is not an IPFS path", fetchPath)
    }
}

// 读取 IPFS 配置文件，返回引导节点地址和对等节点信息
func readIpfsConfig(repoRoot *string, userConfigFile string) (bootstrap []string, peers []peer.AddrInfo) {
    // 如果存储库根目录为空，则直接返回
    if repoRoot == nil {
        return
    }

    // 构建配置文件路径
    cfgPath, err := config.Filename(*repoRoot, userConfigFile)
    if err != nil {
        fmt.Fprintln(os.Stderr, err)
        return
    }

    // 打开配置文件
    cfgFile, err := os.Open(cfgPath)
    if err != nil {
        fmt.Fprintln(os.Stderr, err)
        return
    }
    defer cfgFile.Close()

    // 尝试读取引导节点地址
    var bootstrapCfg struct {
        Bootstrap []string
    }
    err = json.NewDecoder(cfgFile).Decode(&bootstrapCfg)
    if err != nil {
        fmt.Fprintln(os.Stderr, "cannot read bootstrap peers from config")
    } else {
        bootstrap = bootstrapCfg.Bootstrap
    }

    // 尝试读取对等节点信息
    if _, err = cfgFile.Seek(0, 0); err != nil {
        fmt.Fprintln(os.Stderr, err)
    }

    var peeringCfg struct {
        Peering config.Peering
    }
    err = json.NewDecoder(cfgFile).Decode(&peeringCfg)
    if err != nil {
        fmt.Fprintln(os.Stderr, "cannot read peering from config")
    } else {
        peers = peeringCfg.Peering.Peers
    }

    return
}
```