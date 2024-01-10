# `kubo\repo\fsrepo\migrations\migrations.go`

```
package migrations

import (
    "context" // 上下文包，用于控制goroutine的执行
    "encoding/json" // JSON编解码包
    "errors" // 错误处理包
    "fmt" // 格式化包
    "log" // 日志包
    "net/url" // URL解析包
    "os" // 操作系统功能包
    "os/exec" // 执行外部命令包
    "path" // 路径操作包
    "runtime" // 运行时信息包
    "strings" // 字符串操作包
    "sync" // 同步包

    config "github.com/ipfs/kubo/config" // 导入自定义的config包
)

const (
    // Migrations subdirectory in distribution. Empty for root (no subdir).
    distMigsRoot = "" // 分发中的迁移子目录。根目录为空（没有子目录）。
    distFSRM     = "fs-repo-migrations" // 分发中的fs-repo-migrations目录
)

// RunMigration finds, downloads, and runs the individual migrations needed to
// migrate the repo from its current version to the target version.
func RunMigration(ctx context.Context, fetcher Fetcher, targetVer int, ipfsDir string, allowDowngrade bool) error {
    ipfsDir, err := CheckIpfsDir(ipfsDir) // 检查IPFS目录是否存在
    if err != nil {
        return err
    }
    fromVer, err := RepoVersion(ipfsDir) // 获取IPFS目录的版本号
    if err != nil {
        return fmt.Errorf("could not get repo version: %w", err) // 返回获取版本号错误
    }
    if fromVer == targetVer {
        // repo already at target version number
        return nil
    }
    if fromVer > targetVer && !allowDowngrade {
        return fmt.Errorf("downgrade not allowed from %d to %d", fromVer, targetVer) // 返回不允许降级的错误
    }

    logger := log.New(os.Stdout, "", 0) // 创建日志记录器

    logger.Print("Looking for suitable migration binaries.") // 打印日志信息

    migrations, binPaths, err := findMigrations(ctx, fromVer, targetVer) // 查找迁移文件
    if err != nil {
        return err
    }

    // Download migrations that were not found
    # 如果二进制路径数量小于迁移数量
    if len(binPaths) < len(migrations) {
        # 创建一个空的字符串切片，用于存储缺失的迁移
        missing := make([]string, 0, len(migrations)-len(binPaths))
        # 遍历迁移列表，检查是否存在对应的二进制路径，如果不存在则添加到缺失列表中
        for _, mig := range migrations {
            if _, ok := binPaths[mig]; !ok {
                missing = append(missing, mig)
            }
        }

        # 打印需要下载的迁移数量
        logger.Println("Need", len(missing), "migrations, downloading.")

        # 创建临时目录用于存储下载的迁移文件
        tmpDir, err := os.MkdirTemp("", "migrations")
        if err != nil {
            return err
        }
        defer os.RemoveAll(tmpDir)  # 在函数返回前删除临时目录

        # 从远程获取缺失的迁移文件
        fetched, err := fetchMigrations(ctx, fetcher, missing, tmpDir, logger)
        if err != nil {
            logger.Print("Failed to download migrations.")
            return err
        }

        # 将下载的迁移文件路径添加到二进制路径字典中
        for i := range missing {
            binPaths[missing[i]] = fetched[i]
        }
    }

    # 判断是否需要回滚迁移
    var revert bool
    if fromVer > targetVer {
        revert = true
    }
    # 遍历迁移列表，执行每个迁移
    for _, migration := range migrations {
        logger.Println("Running migration", migration, "...")
        # 执行迁移操作
        err = runMigration(ctx, binPaths[migration], ipfsDir, revert, logger)
        if err != nil {
            return fmt.Errorf("migration %s failed: %w", migration, err)
        }
    }
    # 打印迁移成功信息
    logger.Printf("Success: fs-repo migrated to version %d.\n", targetVer)

    return nil
// NeedMigration 检查是否需要进行迁移，比较当前版本号和目标版本号，返回是否需要迁移和可能的错误
func NeedMigration(target int) (bool, error) {
    // 获取仓库版本号
    vnum, err := RepoVersion("")
    if err != nil {
        return false, fmt.Errorf("could not get repo version: %w", err)
    }

    // 比较版本号，判断是否需要迁移
    return vnum != target, nil
}

// ExeName 根据操作系统返回可执行文件名
func ExeName(name string) string {
    // 如果是 Windows 系统，返回带有 .exe 后缀的文件名
    if runtime.GOOS == "windows" {
        return name + ".exe"
    }
    // 否则返回原文件名
    return name
}

// ReadMigrationConfig 读取 IPFS 配置文件中的迁移部分，避免读取除迁移部分以外的内容
func ReadMigrationConfig(repoRoot string, userConfigFile string) (*config.Migration, error) {
    // 定义配置结构
    var cfg struct {
        Migration config.Migration
    }

    // 获取配置文件路径
    cfgPath, err := config.Filename(repoRoot, userConfigFile)
    if err != nil {
        return nil, err
    }

    // 打开配置文件
    cfgFile, err := os.Open(cfgPath)
    if err != nil {
        return nil, err
    }
    defer cfgFile.Close()

    // 解析配置文件中的迁移部分
    err = json.NewDecoder(cfgFile).Decode(&cfg)
    if err != nil {
        return nil, err
    }

    // 根据配置值进行处理
    switch cfg.Migration.Keep {
    case "":
        cfg.Migration.Keep = config.DefaultMigrationKeep
    case "discard", "cache", "keep":
    default:
        return nil, errors.New("unknown config value, Migrations.Keep must be 'cache', 'pin', or 'discard'")
    }

    // 如果下载源为空，则使用默认下载源
    if len(cfg.Migration.DownloadSources) == 0 {
        cfg.Migration.DownloadSources = config.DefaultMigrationDownloadSources
    }

    // 返回迁移配置
    return &cfg.Migration, nil
}

// GetMigrationFetcher 根据下载源创建一个或多个 fetcher
func GetMigrationFetcher(downloadSources []string, distPath string, newIpfsFetcher func(string) Fetcher) (Fetcher, error) {
    // 定义常量
    const httpUserAgent = "kubo/migration"
    const numTriesPerHTTP = 3

    // 定义 fetchers 切片
    var fetchers []Fetcher
    // 遍历下载源列表
    for _, src := range downloadSources {
        // 去除源字符串两端的空格
        src := strings.TrimSpace(src)
        // 根据不同的源类型进行处理
        switch src {
        // 处理 HTTP 和 HTTPS 类型的源
        case "HTTPS", "https", "HTTP", "http":
            // 将 HTTP 和 HTTPS 类型的源添加到 fetchers 中
            fetchers = append(fetchers, &RetryFetcher{NewHttpFetcher(distPath, "", httpUserAgent, 0), numTriesPerHTTP})
        // 处理 IPFS 类型的源
        case "IPFS", "ipfs":
            // 如果存在 IPFS 类型的处理器，则将其添加到 fetchers 中
            if newIpfsFetcher != nil {
                fetchers = append(fetchers, newIpfsFetcher(distPath))
            }
        // 处理空字符串的情况
        case "":
            // 忽略空字符串
        // 处理其他类型的源
        default:
            // 解析源字符串为 URL
            u, err := url.Parse(src)
            // 如果解析出错，则返回错误信息
            if err != nil {
                return nil, fmt.Errorf("bad gateway address: %w", err)
            }
            // 根据 URL 的 scheme 进行处理
            switch u.Scheme {
            // 如果 scheme 为空，则设置为 https
            case "":
                u.Scheme = "https"
            // 如果 scheme 为 http 或 https，则不做处理
            case "https", "http":
            // 如果 scheme 为其他值，则返回错误信息
            default:
                return nil, errors.New("bad gateway address: url scheme must be http or https")
            }
            // 将处理后的 URL 添加到 fetchers 中
            fetchers = append(fetchers, &RetryFetcher{NewHttpFetcher(distPath, u.String(), httpUserAgent, 0), numTriesPerHTTP})
        }
    }

    // 根据 fetchers 的长度进行处理
    switch len(fetchers) {
    // 如果 fetchers 为空，则返回错误信息
    case 0:
        return nil, errors.New("no sources specified")
    // 如果 fetchers 只有一个元素，则直接返回该元素
    case 1:
        return fetchers[0], nil
    }

    // 将 fetchers 包装在 MultiFetcher 中，按顺序尝试它们
    return NewMultiFetcher(fetchers...), nil
// 定义一个函数，用于生成迁移的名称，格式为"fs-repo-起始版本号-to-目标版本号"
func migrationName(from, to int) string {
    return fmt.Sprintf("fs-repo-%d-to-%d", from, to)
}

// 查找迁移，返回按顺序应用的迁移列表，以及找到的任何迁移二进制文件的位置映射
func findMigrations(ctx context.Context, from, to int) ([]string, map[string]string, error) {
    step := 1
    count := to - from
    if from > to {
        step = -1
        count = from - to
    }

    // 初始化迁移列表
    migrations := make([]string, 0, count)
    // 初始化迁移二进制文件位置映射
    binPaths := make(map[string]string, count)

    // 循环生成迁移列表和迁移二进制文件位置映射
    for cur := from; cur != to; cur += step {
        // 检查上下文是否出错
        if ctx.Err() != nil {
            return nil, nil, ctx.Err()
        }
        var migName string
        // 根据步长确定迁移名称
        if step == -1 {
            migName = migrationName(cur+step, cur)
        } else {
            migName = migrationName(cur, cur+step)
        }
        // 将迁移名称添加到迁移列表
        migrations = append(migrations, migName)
        // 查找迁移二进制文件的路径
        bin, err := exec.LookPath(migName)
        if err != nil {
            continue
        }
        // 将迁移名称和对应的二进制文件路径添加到位置映射中
        binPaths[migName] = bin
    }
    return migrations, binPaths, nil
}

// 执行迁移，运行指定的二进制文件，传入IPFS目录路径和是否回滚的标志
func runMigration(ctx context.Context, binPath, ipfsDir string, revert bool, logger *log.Logger) error {
    // 格式化IPFS目录路径参数
    pathArg := fmt.Sprintf("-path=%s", ipfsDir)
    var cmd *exec.Cmd
    // 根据是否回滚，构建执行命令
    if revert {
        logger.Println("  => Running:", binPath, pathArg, "-verbose=true -revert")
        cmd = exec.CommandContext(ctx, binPath, pathArg, "-verbose=true", "-revert")
    } else {
        logger.Println("  => Running:", binPath, pathArg, "-verbose=true")
        cmd = exec.CommandContext(ctx, binPath, pathArg, "-verbose=true")
    }
    // 设置命令的标准输出和标准错误输出
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    // 运行命令
    return cmd.Run()
}

// 获取迁移，下载所需的迁移，并返回每个二进制文件的路径的切片，与所需的顺序相同
func fetchMigrations(ctx context.Context, fetcher Fetcher, needed []string, destDir string, logger *log.Logger) ([]string, error) {
    // 获取操作系统名称和变体
    osv, err := osWithVariant()
    // 如果获取失败，返回空和错误
    if err != nil {
        return nil, err
    }
    // 如果操作系统变体为 "linux-musl"，返回错误
    if osv == "linux-musl" {
        return nil, fmt.Errorf("linux-musl not supported, you must build the binary from source for your platform")
    }

    // 创建一个同步等待组
    var wg sync.WaitGroup
    // 增加等待组的计数
    wg.Add(len(needed))
    // 创建一个字符串切片用于存储下载的文件路径
    bins := make([]string, len(needed))
    // 并发下载和解压所有请求的迁移
    for i, name := range needed {
        // 打印日志，显示正在下载的迁移名称
        logger.Printf("Downloading migration: %s...", name)
        // 启动一个 goroutine 执行下载和解压操作
        go func(i int, name string) {
            // 函数执行完毕后减少等待组的计数
            defer wg.Done()
            // 拼接迁移文件的路径
            dist := path.Join(distMigsRoot, name)
            // 获取迁移的最新版本
            ver, err := LatestDistVersion(ctx, fetcher, dist, false)
            // 如果获取失败，打印错误日志并返回
            if err != nil {
                logger.Printf("could not get latest version of migration %s: %s", name, err)
                return
            }
            // 下载迁移文件
            loc, err := FetchBinary(ctx, fetcher, dist, ver, name, destDir)
            // 如果下载失败，打印错误日志并返回
            if err != nil {
                logger.Printf("could not download %s: %s", name, err)
                return
            }
            // 打印日志，显示已下载和解压的迁移文件路径和版本
            logger.Printf("Downloaded and unpacked migration: %s (%s)", loc, ver)
            // 将下载的文件路径存入对应的位置
            bins[i] = loc
        }(i, name)
    }
    // 等待所有 goroutine 完成
    wg.Wait()

    // 创建一个失败的迁移文件名切片
    var fails []string
    // 遍历下载的文件路径切片，将下载失败的文件名存入失败切片
    for i := range bins {
        if bins[i] == "" {
            fails = append(fails, needed[i])
        }
    }
    // 如果有下载失败的文件，返回错误
    if len(fails) != 0 {
        err = fmt.Errorf("failed to download migrations: %s", strings.Join(fails, " "))
        // 如果上下文发生错误，将上下文错误和下载错误合并返回
        if ctx.Err() != nil {
            err = fmt.Errorf("%s, %w", ctx.Err(), err)
        }
        return nil, err
    }

    // 返回下载成功的文件路径切片和空错误
    return bins, nil
# 闭合前面的函数定义
```