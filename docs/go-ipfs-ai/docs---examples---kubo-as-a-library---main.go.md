# `kubo\docs\examples\kubo-as-a-library\main.go`

```
package main

import (
    "context" // 上下文包，用于控制取消操作
    "flag" // 命令行参数解析包
    "fmt" // 格式化包
    "io" // 输入输出包
    "log" // 日志包
    "os" // 操作系统功能包
    "path/filepath" // 文件路径包
    "strings" // 字符串处理包
    "sync" // 同步包

    "github.com/ipfs/boxo/files" // IPFS文件包
    "github.com/ipfs/boxo/path" // IPFS路径包
    icore "github.com/ipfs/kubo/core/coreiface" // IPFS核心接口包
    ma "github.com/multiformats/go-multiaddr" // 多格式地址包

    "github.com/ipfs/kubo/config" // IPFS配置包
    "github.com/ipfs/kubo/core" // IPFS核心包
    "github.com/ipfs/kubo/core/coreapi" // IPFS核心API包
    "github.com/ipfs/kubo/core/node/libp2p" // IPFS节点libp2p包
    "github.com/ipfs/kubo/plugin/loader" // 需要此包以便自动加载所有预加载的插件
    "github.com/ipfs/kubo/repo/fsrepo" // IPFS存储库包
    "github.com/libp2p/go-libp2p/core/peer" // libp2p核心对等体包
)

/// ------ Setting up the IPFS Repo

func setupPlugins(externalPluginsPath string) error {
    // 如果可用，加载任何外部插件
    plugins, err := loader.NewPluginLoader(filepath.Join(externalPluginsPath, "plugins"))
    if err != nil {
        return fmt.Errorf("error loading plugins: %s", err)
    }

    // 加载预加载和外部插件
    if err := plugins.Initialize(); err != nil {
        return fmt.Errorf("error initializing plugins: %s", err)
    }

    if err := plugins.Inject(); err != nil {
        return fmt.Errorf("error initializing plugins: %s", err)
    }

    return nil
}

func createTempRepo() (string, error) {
    repoPath, err := os.MkdirTemp("", "ipfs-shell") // 创建临时目录
    if err != nil {
        return "", fmt.Errorf("failed to get temp dir: %s", err)
    }

    // 使用默认选项和2048位密钥创建配置
    cfg, err := config.Init(io.Discard, 2048)
    if err != nil {
        return "", err
    }

    // 创建存储库时，可以定义存储库上的自定义设置，例如启用实验性功能（参见实验性功能.md）或自定义网关端点。
    // 要执行此类操作，应修改变量`cfg`。例如：
    // 如果标志位 *flagExp 为真，则启用实验性特性
    if *flagExp {
        // 启用文件存储实验性特性
        cfg.Experimental.FilestoreEnabled = true
        // 启用 URL 存储实验性特性
        cfg.Experimental.UrlstoreEnabled = true
        // 启用 Libp2p 流挂载实验性特性
        cfg.Experimental.Libp2pStreamMounting = true
        // 启用 P2P HTTP 代理实验性特性
        cfg.Experimental.P2pHttpProxy = true
        // 参考：https://github.com/ipfs/kubo/blob/master/docs/config.md
        // 以及：https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md
    }

    // 使用配置信息创建存储库
    err = fsrepo.Init(repoPath, cfg)
    if err != nil {
        // 如果创建存储库失败，则返回错误信息
        return "", fmt.Errorf("failed to init ephemeral node: %s", err)
    }

    // 返回存储库路径
    return repoPath, nil
}

/// ------ Spawning the node

// 创建一个 IPFS 节点并返回其核心 API。
func createNode(ctx context.Context, repoPath string) (*core.IpfsNode, error) {
    // 打开存储库
    repo, err := fsrepo.Open(repoPath)
    if err != nil {
        return nil, err
    }

    // 构建节点

    nodeOptions := &core.BuildCfg{
        Online:  true,
        Routing: libp2p.DHTOption, // 此选项将节点设置为完整的 DHT 节点（用于获取和存储 DHT 记录）
        // Routing: libp2p.DHTClientOption, // 此选项将节点设置为客户端 DHT 节点（仅用于获取记录）
        Repo: repo,
    }

    return core.NewNode(ctx, nodeOptions)
}

var loadPluginsOnce sync.Once

// 生成一个节点，仅用于此运行（即创建一个临时存储库）。
func spawnEphemeral(ctx context.Context) (icore.CoreAPI, *core.IpfsNode, error) {
    var onceErr error
    loadPluginsOnce.Do(func() {
        onceErr = setupPlugins("")
    })
    if onceErr != nil {
        return nil, nil, onceErr
    }

    // 创建临时存储库
    repoPath, err := createTempRepo()
    if err != nil {
        return nil, nil, fmt.Errorf("failed to create temp repo: %s", err)
    }

    node, err := createNode(ctx, repoPath)
    if err != nil {
        return nil, nil, err
    }

    api, err := coreapi.NewCoreAPI(node)

    return api, node, err
}

func connectToPeers(ctx context.Context, ipfs icore.CoreAPI, peers []string) error {
    var wg sync.WaitGroup
    peerInfos := make(map[peer.ID]*peer.AddrInfo, len(peers))
    for _, addrStr := range peers {
        addr, err := ma.NewMultiaddr(addrStr)
        if err != nil {
            return err
        }
        pii, err := peer.AddrInfoFromP2pAddr(addr)
        if err != nil {
            return err
        }
        pi, ok := peerInfos[pii.ID]
        if !ok {
            pi = &peer.AddrInfo{ID: pii.ID}
            peerInfos[pi.ID] = pi
        }
        pi.Addrs = append(pi.Addrs, pii.Addrs...)
    }
    # 根据 peerInfos 的数量设置 WaitGroup 的计数
    wg.Add(len(peerInfos))
    # 遍历 peerInfos 中的每个 peerInfo
    for _, peerInfo := range peerInfos:
        # 启动一个 goroutine 来连接到 peerInfo
        go func(peerInfo *peer.AddrInfo):
            # 在函数退出时减少 WaitGroup 的计数
            defer wg.Done()
            # 使用 ipfs.Swarm() 来连接到 peerInfo
            err := ipfs.Swarm().Connect(ctx, *peerInfo)
            # 如果连接失败，记录错误信息
            if err != nil:
                log.Printf("failed to connect to %s: %s", peerInfo.ID, err)
        # 传入当前的 peerInfo
        }(peerInfo)
    # 等待所有 goroutine 完成
    wg.Wait()
    # 返回空值
    return nil
// 获取指定路径的 Unixfs 节点，如果路径不存在则返回错误
func getUnixfsNode(path string) (files.Node, error) {
    // 获取指定路径的文件信息
    st, err := os.Stat(path)
    if err != nil {
        return nil, err
    }

    // 使用指定路径创建一个新的序列化文件
    f, err := files.NewSerialFile(path, false, st)
    if err != nil {
        return nil, err
    }

    // 返回创建的文件节点
    return f, nil
}

/// -------

// 定义一个布尔类型的命令行标志，用于启用实验性功能
var flagExp = flag.Bool("experimental", false, "enable experimental features")

// 主函数
func main() {
    // 解析命令行标志
    flag.Parse()

    /// --- Part I: Getting a IPFS node running

    // 打印信息
    fmt.Println("-- Getting an IPFS node running -- ")

    // 创建一个带有取消功能的上下文
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // 生成一个本地对等节点，用于测试目的
    ipfsA, nodeA, err := spawnEphemeral(ctx)
    if err != nil {
        panic(fmt.Errorf("failed to spawn peer node: %s", err))
    }

    // 将字符串转换为字节流并添加到 IPFS 节点
    peerCidFile, err := ipfsA.Unixfs().Add(ctx,
        files.NewBytesFile([]byte("hello from ipfs 101 in Kubo")))
    if err != nil {
        panic(fmt.Errorf("could not add File: %s", err))
    }

    // 打印添加文件的节点 CID
    fmt.Printf("Added file to peer with CID %s\n", peerCidFile.String())

    // 生成一个临时路径的节点，创建一个临时的存储库
    fmt.Println("Spawning Kubo node on a temporary repo")
    ipfsB, _, err := spawnEphemeral(ctx)
    if err != nil {
        panic(fmt.Errorf("failed to spawn ephemeral node: %s", err))
    }

    // 打印信息
    fmt.Println("IPFS node is running")

    /// --- Part II: Adding a file and a directory to IPFS

    // 打印信息
    fmt.Println("\n-- Adding and getting back files & directories --")

    // 设置输入基本路径和文件路径
    inputBasePath := "../example-folder/"
    inputPathFile := inputBasePath + "ipfs.paper.draft3.pdf"
    inputPathDirectory := inputBasePath + "test-dir"

    // 获取指定文件的 Unixfs 节点
    someFile, err := getUnixfsNode(inputPathFile)
    if err != nil {
        panic(fmt.Errorf("could not get File: %s", err))
    }

    // 将文件添加到 IPFS 节点
    cidFile, err := ipfsB.Unixfs().Add(ctx, someFile)
    if err != nil {
        panic(fmt.Errorf("could not add File: %s", err))
    }

    // 打印添加文件的节点 CID
    fmt.Printf("Added file to IPFS with CID %s\n", cidFile.String())
}
    // 通过输入路径获取Unixfs节点
    someDirectory, err := getUnixfsNode(inputPathDirectory)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not get File: %s", err))
    }

    // 将Unixfs节点添加到IPFS，并获取CID
    cidDirectory, err := ipfsB.Unixfs().Add(ctx, someDirectory)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not add Directory: %s", err))
    }

    // 打印添加到IPFS的目录CID
    fmt.Printf("Added directory to IPFS with CID %s\n", cidDirectory.String())

    /// --- Part III: Getting the file and directory you added back

    // 创建临时输出目录
    outputBasePath, err := os.MkdirTemp("", "example")
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not create output dir (%v)", err))
    }
    // 打印输出目录路径
    fmt.Printf("output folder: %s\n", outputBasePath)
    // 构建输出文件路径
    outputPathFile := outputBasePath + strings.Split(cidFile.String(), "/")[2]
    // 构建输出目录路径
    outputPathDirectory := outputBasePath + strings.Split(cidDirectory.String(), "/")[2]

    // 通过CID获取文件的Unixfs节点
    rootNodeFile, err := ipfsB.Unixfs().Get(ctx, cidFile)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not get file with CID: %s", err))
    }

    // 将获取的文件写入输出文件路径
    err = files.WriteTo(rootNodeFile, outputPathFile)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not write out the fetched CID: %s", err))
    }

    // 打印从IPFS获取文件并写入输出文件路径的信息
    fmt.Printf("got file back from IPFS (IPFS path: %s) and wrote it to %s\n", cidFile.String(), outputPathFile)

    // 通过CID获取目录的Unixfs节点
    rootNodeDirectory, err := ipfsB.Unixfs().Get(ctx, cidDirectory)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not get file with CID: %s", err))
    }

    // 将获取的目录写入输出目录路径
    err = files.WriteTo(rootNodeDirectory, outputPathDirectory)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not write out the fetched CID: %s", err))
    }

    // 打印从IPFS获取目录并写入输出目录路径的信息
    fmt.Printf("Got directory back from IPFS (IPFS path: %s) and wrote it to %s\n", cidDirectory.String(), outputPathDirectory)

    /// --- Part IV: Getting a file from the IPFS Network

    // 打印连接到网络中几个节点作为引导节点的信息
    fmt.Println("\n-- Going to connect to a few nodes in the Network as bootstrappers --")

    // 格式化节点A的地址信息
    peerMa := fmt.Sprintf("/ip4/127.0.0.1/udp/4010/p2p/%s", nodeA.Identity.String())
    bootstrapNodes := []string{
        // IPFS Bootstrapper nodes.
        // IPFS引导节点
        // "/dnsaddr/bootstrap.libp2p.io/p2p/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN",
        // "/dnsaddr/bootstrap.libp2p.io/p2p/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa",
        // "/dnsaddr/bootstrap.libp2p.io/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb",
        // "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",

        // IPFS Cluster Pinning nodes
        // IPFS集群固定节点
        // "/ip4/138.201.67.219/tcp/4001/p2p/QmUd6zHcbkbcs7SMxwLs48qZVX3vpcM8errYS7xEczwRMA",
        // "/ip4/138.201.67.219/udp/4001/quic/p2p/QmUd6zHcbkbcs7SMxwLs48qZVX3vpcM8errYS7xEczwRMA",
        // "/ip4/138.201.67.220/tcp/4001/p2p/QmNSYxZAiJHeLdkBg38roksAR9So7Y5eojks1yjEcUtZ7i",
        // "/ip4/138.201.67.220/udp/4001/quic/p2p/QmNSYxZAiJHeLdkBg38roksAR9So7Y5eojks1yjEcUtZ7i",
        // "/ip4/138.201.68.74/tcp/4001/p2p/QmdnXwLrC8p1ueiq2Qya8joNvk3TVVDAut7PrikmZwubtR",
        // "/ip4/138.201.68.74/udp/4001/quic/p2p/QmdnXwLrC8p1ueiq2Qya8joNvk3TVVDAut7PrikmZwubtR",
        // "/ip4/94.130.135.167/tcp/4001/p2p/QmUEMvxS2e7iDrereVYc5SWPauXPyNwxcy9BXZrC1QTcHE",
        // "/ip4/94.130.135.167/udp/4001/quic/p2p/QmUEMvxS2e7iDrereVYc5SWPauXPyNwxcy9BXZrC1QTcHE",

        // You can add more nodes here, for example, another IPFS node you might have running locally, mine was:
        // 你可以在这里添加更多节点，例如，你可能在本地运行另一个IPFS节点，我的是：
        // "/ip4/127.0.0.1/tcp/4010/p2p/QmZp2fhDLxjYue2RiUvLwT9MWdnbDxam32qYFnGmxZDh5L",
        // "/ip4/127.0.0.1/udp/4010/quic/p2p/QmZp2fhDLxjYue2RiUvLwT9MWdnbDxam32qYFnGmxZDh5L",
        peerMa,
    }

    go func() {
        // 连接到节点
        err := connectToPeers(ctx, ipfsB, bootstrapNodes)
        if err != nil {
            log.Printf("failed connect to peers: %s", err)
        }
    }()

    exampleCIDStr := peerCidFile.RootCid().String()

    // 从网络中获取具有CID的文件
    fmt.Printf("Fetching a file from the network with CID %s\n", exampleCIDStr)
    outputPath := outputBasePath + exampleCIDStr
    testCID := path.FromCid(peerCidFile.RootCid())
    // 通过CID获取Unixfs节点
    rootNode, err := ipfsB.Unixfs().Get(ctx, testCID)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not get file with CID: %s", err))
    }

    // 将获取到的节点写入到指定的输出路径
    err = files.WriteTo(rootNode, outputPath)
    // 如果出现错误，抛出异常
    if err != nil {
        panic(fmt.Errorf("could not write out the fetched CID: %s", err))
    }

    // 打印输出文件的路径
    fmt.Printf("Wrote the file to %s\n", outputPath)

    // 打印完成信息
    fmt.Println("\nAll done! You just finalized your first tutorial on how to use Kubo as a library")
# 闭合前面的函数定义
```