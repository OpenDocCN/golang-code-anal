# `kubo\cmd\ipfs\kubo\add_migrations.go`

```
package kubo

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "errors"   // 错误处理包，用于处理错误信息
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，用于处理输入输出操作
    "os"       // 操作系统包，用于操作系统相关功能
    "path/filepath"  // 文件路径包，用于处理文件路径

    "github.com/ipfs/boxo/files"  // IPFS文件包，用于处理IPFS文件
    "github.com/ipfs/boxo/path"   // IPFS路径包，用于处理IPFS路径
    "github.com/ipfs/kubo/core"   // IPFS核心包，用于IPFS核心功能
    "github.com/ipfs/kubo/core/coreapi"  // IPFS核心API包，用于IPFS核心API功能
    coreiface "github.com/ipfs/kubo/core/coreiface"  // IPFS核心接口包，用于IPFS核心接口功能
    "github.com/ipfs/kubo/core/coreiface/options"  // IPFS核心接口选项包，用于IPFS核心接口选项功能
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // IPFS存储库迁移包，用于IPFS存储库迁移功能
    "github.com/ipfs/kubo/repo/fsrepo/migrations/ipfsfetcher"  // IPFS存储库迁移IPFS获取器包，用于IPFS存储库迁移IPFS获取器功能
    "github.com/libp2p/go-libp2p/core/peer"  // libp2p核心对等体包，用于libp2p核心对等体功能
)

// addMigrations adds any migration downloaded by the fetcher to the IPFS node.
// addMigrations函数用于将获取器下载的任何迁移添加到IPFS节点
func addMigrations(ctx context.Context, node *core.IpfsNode, fetcher migrations.Fetcher, pin bool) error {
    var fetchers []migrations.Fetcher  // 定义迁移获取器切片
    if mf, ok := fetcher.(*migrations.MultiFetcher); ok {  // 判断获取器是否为多获取器类型
        fetchers = mf.Fetchers()  // 如果是多获取器类型，则获取所有获取器
    } else {
        fetchers = []migrations.Fetcher{fetcher}  // 如果不是多获取器类型，则将获取器放入切片中
    }
    // 遍历 fetchers 列表中的每个 fetcher
    for _, fetcher := range fetchers {
        // 根据 fetcher 的类型进行不同的处理
        switch f := fetcher.(type) {
        case *ipfsfetcher.IpfsFetcher:
            // 如果是 IpfsFetcher 类型，则通过连接到临时节点并从 IPFS 获取迁移路径来添加迁移
            err := addMigrationPaths(ctx, node, f.AddrInfo(), f.FetchedPaths(), pin)
            if err != nil {
                return err
            }
        case *migrations.HttpFetcher, *migrations.RetryFetcher: // https://github.com/ipfs/kubo/issues/8780
            // 如果是 HttpFetcher 或 RetryFetcher 类型，则直接添加已下载的迁移文件
            if migrations.DownloadDirectory != "" {
                // 获取下载目录中的文件路径列表
                var paths []string
                err := filepath.Walk(migrations.DownloadDirectory, func(filePath string, info os.FileInfo, err error) error {
                    if info.IsDir() {
                        return nil
                    }
                    paths = append(paths, filePath)
                    return nil
                })
                if err != nil {
                    return err
                }
                // 将文件路径列表添加到迁移文件中
                err = addMigrationFiles(ctx, node, paths, pin)
                if err != nil {
                    return err
                }
            }
        default:
            // 如果 fetcher 类型未知，则返回错误
            return errors.New("cannot get migrations from unknown fetcher type")
        }
    }
    // 处理完所有 fetchers 后返回 nil
    return nil
// addMigrationFiles函数将路径中的文件添加到IPFS，可选择是否将其固定。
func addMigrationFiles(ctx context.Context, node *core.IpfsNode, paths []string, pin bool) error {
    // 如果路径为空，则返回nil
    if len(paths) == 0 {
        return nil
    }
    // 创建IPFS核心API
    ifaceCore, err := coreapi.NewCoreAPI(node)
    if err != nil {
        return err
    }
    // 获取Unixfs接口
    ufs := ifaceCore.Unixfs()

    // 添加迁移文件
    for _, filePath := range paths {
        // 打开文件
        f, err := os.Open(filePath)
        if err != nil {
            return err
        }

        // 获取文件信息
        fi, err := f.Stat()
        if err != nil {
            return err
        }

        // 将文件添加到IPFS，并可选择固定
        ipfsPath, err := ufs.Add(ctx, files.NewReaderStatFile(f, fi), options.Unixfs.Pin(pin))
        if err != nil {
            return err
        }
        fmt.Printf("Added migration file %q: %s\n", filepath.Base(filePath), ipfsPath)
    }

    return nil
}

// addMigrationPaths函数将路径中的文件添加到IPFS，可选择是否将其固定。这是在连接到对等节点之后完成的。
func addMigrationPaths(ctx context.Context, node *core.IpfsNode, peerInfo peer.AddrInfo, paths []path.Path, pin bool) error {
    // 如果路径为空，则返回错误
    if len(paths) == 0 {
        return errors.New("nothing downloaded by ipfs fetcher")
    }
    // 如果对等节点没有本地swarm地址，则返回错误
    if len(peerInfo.Addrs) == 0 {
        return errors.New("no local swarm address for migration node")
    }

    // 创建IPFS核心API
    ipfs, err := coreapi.NewCoreAPI(node)
    if err != nil {
        return err
    }

    // 连接到临时节点
    if err := ipfs.Swarm().Connect(ctx, peerInfo); err != nil {
        return fmt.Errorf("could not connect to migration peer %q: %s", peerInfo.ID, err)
    }
    fmt.Printf("connected to migration peer %q\n", peerInfo)

    // 如果固定为true，则添加并固定迁移文件
    if pin {
        pinAPI := ipfs.Pin()
        for _, ipfsPath := range paths {
            err := pinAPI.Add(ctx, ipfsPath)
            if err != nil {
                return err
            }
            fmt.Printf("Added and pinned migration file: %q\n", ipfsPath)
        }
        return nil
    }

    // 获取Unixfs接口
    ufs := ipfs.Unixfs()
}
    // 遍历路径列表，依次获取对应的 IPFS 文件
    for _, ipfsPath := range paths {
        // 调用 ipfsGet 函数获取指定路径的 IPFS 文件
        err = ipfsGet(ctx, ufs, ipfsPath)
        // 如果获取过程中出现错误，则返回该错误
        if err != nil {
            return err
        }
    }
    // 如果没有出现错误，则返回空值
    return nil
# 定义一个名为 ipfsGet 的函数，接收上下文、UnixfsAPI 接口和路径作为参数，返回错误
func ipfsGet(ctx context.Context, ufs coreiface.UnixfsAPI, ipfsPath path.Path) error {
    # 使用 UnixfsAPI 接口获取指定路径的节点对象和可能的错误
    nd, err := ufs.Get(ctx, ipfsPath)
    # 如果出现错误，则返回错误
    if err != nil {
        return err
    }
    # 延迟关闭节点对象
    defer nd.Close()

    # 尝试将节点对象转换为文件对象
    fnd, ok := nd.(files.File)
    # 如果无法转换，则返回错误
    if !ok {
        return fmt.Errorf("not a file node: %q", ipfsPath)
    }
    # 将文件对象的内容拷贝到 io.Discard 中，返回拷贝过程中可能出现的错误
    _, err = io.Copy(io.Discard, fnd)
    if err != nil {
        return fmt.Errorf("cannot read migration: %w", err)
    }
    # 打印已添加的迁移文件路径
    fmt.Printf("Added migration file: %q\n", ipfsPath)
    # 返回空错误，表示操作成功
    return nil
}
```