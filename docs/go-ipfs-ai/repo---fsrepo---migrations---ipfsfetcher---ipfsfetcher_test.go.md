# `kubo\repo\fsrepo\migrations\ipfsfetcher\ipfsfetcher_test.go`

```
package ipfsfetcher

import (
    "bufio"  // 导入 bufio 包，用于缓冲读取
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "context"  // 导入 context 包，用于控制goroutine的生命周期
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "os"  // 导入 os 包，提供了一些与操作系统交互的函数
    "path/filepath"  // 导入 filepath 包，用于处理文件路径
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/ipfs/kubo/plugin/loader"  // 导入插件加载器
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // 导入文件系统存储库迁移
)

func init() {
    err := setupPlugins()  // 调用 setupPlugins 函数初始化插件
    if err != nil {
        panic(err)  // 如果初始化失败，触发 panic
    }
}

func TestIpfsFetcher(t *testing.T) {
    skipUnlessEpic(t)  // 调用 skipUnlessEpic 函数，如果条件不满足则跳过测试

    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    defer cancel()  // 延迟调用取消函数

    fetcher := NewIpfsFetcher("", 0, nil, "")  // 创建一个新的 IpfsFetcher 实例
    defer fetcher.Close()  // 延迟调用关闭函数

    out, err := fetcher.Fetch(ctx, "go-ipfs/versions")  // 调用 Fetch 方法获取数据
    if err != nil {
        t.Fatal(err)  // 如果出现错误，触发测试失败
    }

    var lines []string  // 声明一个字符串切片
    scan := bufio.NewScanner(bytes.NewReader(out))  // 创建一个新的 Scanner 对象
    for scan.Scan() {  // 循环读取数据
        lines = append(lines, scan.Text())  // 将读取的数据添加到字符串切片中
    }
    err = scan.Err()  // 获取可能出现的错误
    if err != nil {
        t.Fatal("could not read versions:", err)  // 如果出现错误，触发测试失败
    }

    if len(lines) < 6 {  // 检查读取的数据长度是否小于6
        t.Fatal("do not get all expected data")  // 如果长度小于6，触发测试失败
    }
    if lines[0] != "v0.3.2" {  // 检查第一行数据是否为 "v0.3.2"
        t.Fatal("expected v1.0.0 as first line, got", lines[0])  // 如果不是，触发测试失败
    }

    // Check not found
    if _, err = fetcher.Fetch(ctx, "/no_such_file"); err == nil {  // 检查是否能够获取指定文件，如果能够获取，触发测试失败
        t.Fatal("expected error 404")
    }
}

func TestInitIpfsFetcher(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    defer cancel()  // 延迟调用取消函数

    f := NewIpfsFetcher("", 0, nil, "")  // 创建一个新的 IpfsFetcher 实例
    defer f.Close()  // 延迟调用关闭函数

    // Init ipfs repo
    f.ipfsTmpDir, f.openErr = initTempNode(ctx, nil, nil)  // 初始化 ipfs 仓库
    if f.openErr != nil {
        t.Fatalf("failed to initialize ipfs node: %s", f.openErr)  // 如果初始化失败，触发测试失败
    }

    // Start ipfs node
    f.openErr = f.startTempNode(ctx)  // 启动 ipfs 节点
    if f.openErr != nil {
        t.Errorf("failed to start ipfs node: %s", f.openErr)  // 如果启动失败，记录错误信息
        return
    }

    var stopFuncCalled bool  // 声明一个布尔变量
    stopFunc := f.ipfsStopFunc  // 备份 ipfsStopFunc 函数
    f.ipfsStopFunc = func() {  // 重写 ipfsStopFunc 函数
        stopFuncCalled = true  // 标记为调用过
        stopFunc()  // 调用备份的函数
    }

    addrInfo := f.AddrInfo()  // 获取地址信息
    if string(addrInfo.ID) == "" {  // 检查地址信息中的 ID 是否为空
        t.Error("AddInfo ID not set")  // 如果为空，记录错误信息
    }
}
    # 如果地址信息中的地址数量为0，则输出错误信息
    if len(addrInfo.Addrs) == 0 {
        t.Error("AddInfo Addrs not set")
    }
    # 输出临时节点监听的地址信息
    t.Log("Temp node listening on:", addrInfo.Addrs)

    # 关闭文件
    err := f.Close()
    # 如果关闭文件时出现错误，则输出错误信息
    if err != nil {
        t.Fatalf("failed to close fetcher: %s", err)
    }

    # 如果停止函数不为空且停止函数未被调用，则输出错误信息
    if stopFunc != nil && !stopFuncCalled {
        t.Error("Close did not call stop function")
    }

    # 再次关闭文件
    err = f.Close()
    # 如果再次关闭文件时出现错误，则输出错误信息
    if err != nil {
        t.Fatalf("failed to close fetcher 2nd time: %s", err)
    }
# 测试读取 IPFS 配置的函数
func TestReadIpfsConfig(t *testing.T) {
    # 测试用的 IPFS 配置
    testConfig := `
{
    "Bootstrap": [
        "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
        "/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
    ],
    "Migration": {
        "DownloadSources": ["IPFS", "HTTP", "127.0.0.1", "https://127.0.1.1"],
        "Keep": "cache"
    },
    "Peering": {
        "Peers": [
            {
                "ID": "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5",
                "Addrs": ["/ip4/127.0.0.1/tcp/4001", "/ip4/127.0.0.1/udp/4001/quic"]
            }
        ]
    }
}
`
    # 不存在的目录
    noSuchDir := "no_such_dir-5953aa51-1145-4efd-afd1-a069075fcf76"
    # 读取 IPFS 配置
    bootstrap, peers := readIpfsConfig(&noSuchDir, "")
    # 检查 bootstrap 是否为 nil
    if bootstrap != nil {
        t.Error("expected nil bootstrap")
    }
    # 检查 peers 是否为 nil
    if peers != nil {
        t.Error("expected nil peers")
    }

    # 创建临时目录
    tmpDir := makeConfig(t, testConfig)

    # 读取 IPFS 配置
    bootstrap, peers = readIpfsConfig(nil, "")
    # 检查 bootstrap 和 peers 是否为 nil
    if bootstrap != nil || peers != nil {
        t.Fatal("expected nil ipfs config items")
    }

    # 读取 IPFS 配置
    bootstrap, peers = readIpfsConfig(&tmpDir, "")
    # 检查 bootstrap 地址数量是否为 2
    if len(bootstrap) != 2 {
        t.Fatal("wrong number of bootstrap addresses")
    }
    # 检查第一个 bootstrap 地址是否正确
    if bootstrap[0] != "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt" {
        t.Fatal("wrong bootstrap address")
    }

    # 检查 peers 数量是否为 1
    if len(peers) != 1 {
        t.Fatal("wrong number of peers")
    }

    # 获取第一个 peer
    peer := peers[0]
    # 检查第一个 peer 的 ID 是否正确
    if peer.ID.String() != "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5" {
        t.Errorf("wrong ID for first peer")
    }
    # 检查第一个 peer 的地址数量是否为 2
    if len(peer.Addrs) != 2 {
        t.Error("wrong number of addrs for first peer")
    }
}

# 测试错误的 IPFS 配置
func TestBadBootstrappingIpfsConfig(t *testing.T) {
    # 错误的 IPFS 配置
    const configBadBootstrap = `
{
    "Bootstrap": "unreadable",
    "Migration": {
        "DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
        "Keep": "cache"
    },
    # 定义了一个名为 "Peering" 的字典，包含了 "Peers" 键对应的值
    "Peering": {
        # "Peers" 键对应的值是一个列表，包含了一个字典
        "Peers": [
            {
                # 字典中的 "ID" 键对应的值是一个字符串
                "ID": "12D3KooWGC6TvWhfapngX6wvJHMYvKpDMXPb3ZnCZ6dMoaMtimQ5",
                # 字典中的 "Addrs" 键对应的值是一个列表，包含了两个字符串
                "Addrs": ["/ip4/127.0.0.1/tcp/4001", "/ip4/127.0.0.1/udp/4001/quic"]
            }
        ]
    }
func TestBadBootstrapIpfsConfig(t *testing.T) {
    // 定义包含错误引导配置的 JSON 字符串
    const configBadBootstrap = `
{
    "Bootstrap": "Unreadable-data",
    "Peering": {
        "Peers": ["/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt"]
    }
}
`
    // 创建临时目录并返回其路径
    tmpDir := makeConfig(t, configBadBootstrap)

    // 读取 IPFS 配置文件中的引导节点和对等节点
    bootstrap, peers := readIpfsConfig(&tmpDir, "")
    // 检查引导节点是否为 nil
    if bootstrap != nil {
        t.Fatal("expected nil bootstrap")
    }
    // 检查对等节点的数量是否为 1
    if len(peers) != 1 {
        t.Fatal("wrong number of peers")
    }
    // 检查第一个对等节点的地址数量是否为 2
    if len(peers[0].Addrs) != 2 {
        t.Error("wrong number of addrs for first peer")
    }
    // 删除临时目录及其内容
    os.RemoveAll(tmpDir)
}

func TestBadPeersIpfsConfig(t *testing.T) {
    // 定义包含错误对等节点配置的 JSON 字符串
    const configBadPeers = `
{
    "Bootstrap": [
        "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
        "/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
    ],
    "Migration": {
        "DownloadSources": ["IPFS", "HTTP", "127.0.0.1"],
        "Keep": "cache"
    },
    "Peering": "Unreadable-data"
}
`
    // 创建临时目录并返回其路径
    tmpDir := makeConfig(t, configBadPeers)

    // 读取 IPFS 配置文件中的引导节点和对等节点
    bootstrap, peers := readIpfsConfig(&tmpDir, "")
    // 检查对等节点是否为 nil
    if peers != nil {
        t.Fatal("expected nil peers")
    }
    // 检查引导节点的数量是否为 2
    if len(bootstrap) != 2 {
        t.Fatal("wrong number of bootstrap addresses")
    }
    // 检查第一个引导节点的地址是否符合预期
    if bootstrap[0] != "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt" {
        t.Fatal("wrong bootstrap address")
    }
}

// 创建临时配置文件并返回其路径
func makeConfig(t *testing.T, configData string) string {
    tmpDir := t.TempDir()

    // 创建配置文件并写入配置数据
    cfgFile, err := os.Create(filepath.Join(tmpDir, "config"))
    if err != nil {
        t.Fatal(err)
    }
    if _, err = cfgFile.Write([]byte(configData)); err != nil {
        t.Fatal(err)
    }
    // 关闭配置文件
    if err = cfgFile.Close(); err != nil {
        t.Fatal(err)
    }
    // 返回临时目录路径
    return tmpDir
}

// 如果未设置 IPFS_EPIC_TEST 环境变量，则跳过测试
func skipUnlessEpic(t *testing.T) {
    if os.Getenv("IPFS_EPIC_TEST") == "" {
        t.SkipNow()
    }
}

// 设置插件
func setupPlugins() error {
    // 获取默认路径
    defaultPath, err := migrations.IpfsDir("")
    if err != nil {
        return err
    }

    // 加载插件，如果不可用则跳过存储库
    plugins, err := loader.NewPluginLoader(filepath.Join(defaultPath, "plugins"))
    // 如果 err 不为 nil，则返回加载插件时的错误
    if err != nil {
        return fmt.Errorf("error loading plugins: %w", err)
    }

    // 如果初始化插件时出现错误，则需要忽略，因为在 ipfs 守护程序运行时可能已经加载了插件
    if err := plugins.Initialize(); err != nil {
        // Need to ignore errors here because plugins may already be loaded when
        // run from ipfs daemon.
        return fmt.Errorf("error initializing plugins: %w", err)
    }

    // 如果注入插件时出现错误，则需要忽略，因为在 ipfs 守护程序运行时可能已经加载了插件
    if err := plugins.Inject(); err != nil {
        // Need to ignore errors here because plugins may already be loaded when
        // run from ipfs daemon.
        return fmt.Errorf("error injecting plugins: %w", err)
    }

    // 如果没有错误发生，则返回 nil
    return nil
# 闭合前面的函数定义
```