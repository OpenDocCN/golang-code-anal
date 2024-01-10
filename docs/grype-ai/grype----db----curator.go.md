# `grype\grype\db\curator.go`

```
package db

import (
    "crypto/tls"  // 导入加密/解密包
    "crypto/x509"  // 导入证书包
    "fmt"  // 导入格式化包
    "net/http"  // 导入网络包
    "os"  // 导入操作系统包
    "path"  // 导入路径包
    "strconv"  // 导入字符串转换包
    "time"  // 导入时间包

    "github.com/hako/durafmt"  // 导入时间格式化包
    cleanhttp "github.com/hashicorp/go-cleanhttp"  // 导入 HTTP 清洁包
    archiver "github.com/mholt/archiver/v3"  // 导入归档包
    "github.com/spf13/afero"  // 导入文件系统包
    partybus "github.com/wagoodman/go-partybus"  // 导入事件总线包
    progress "github.com/wagoodman/go-progress"  // 导入进度包

    grypeDB "github.com/anchore/grype/grype/db/v5"  // 导入 grype 数据库包
    "github.com/anchore/grype/grype/db/v5/store"  // 导入 grype 数据库存储包
    "github.com/anchore/grype/grype/event"  // 导入 grype 事件包
    "github.com/anchore/grype/grype/vulnerability"  // 导入 grype 漏洞包
    "github.com/anchore/grype/internal/bus"  // 导入 grype 内部事件总线包
    "github.com/anchore/grype/internal/file"  // 导入 grype 内部文件包
    "github.com/anchore/grype/internal/log"  // 导入 grype 内部日志包
)

const (
    FileName = grypeDB.VulnerabilityStoreFileName  // 定义文件名为漏洞存储文件名
)

type Config struct {
    DBRootDir           string  // 数据库根目录
    ListingURL          string  // 列表 URL
    CACert              string  // CA 证书
    ValidateByHashOnGet bool  // 获取时通过哈希验证
    ValidateAge         bool  // 验证年龄
    MaxAllowedBuiltAge  time.Duration  // 最大允许构建年龄
}

type Curator struct {
    fs                  afero.Fs  // 文件系统
    downloader          file.Getter  // 文件获取器
    targetSchema        int  // 目标模式
    dbDir               string  // 数据库目录
    dbPath              string  // 数据库路径
    listingURL          string  // 列表 URL
    validateByHashOnGet bool  // 获取时通过哈希验证
    validateAge         bool  // 验证年龄
    maxAllowedBuiltAge  time.Duration  // 最大允许构建年龄
}

func NewCurator(cfg Config) (Curator, error) {
    dbDir := path.Join(cfg.DBRootDir, strconv.Itoa(vulnerability.SchemaVersion))  // 构建数据库目录路径

    fs := afero.NewOsFs()  // 创建操作系统文件系统
    httpClient, err := defaultHTTPClient(fs, cfg.CACert)  // 获取默认的 HTTP 客户端
    if err != nil {
        return Curator{}, err  // 如果出错，返回空的 Curator 和错误
    }
    # 返回一个 Curator 结构体，包含以下字段
    return Curator{
        # 文件系统
        fs:                  fs,
        # 目标模式
        targetSchema:        vulnerability.SchemaVersion,
        # 下载器
        downloader:          file.NewGetter(httpClient),
        # 数据库目录
        dbDir:               dbDir,
        # 数据库路径
        dbPath:              path.Join(dbDir, FileName),
        # 列表 URL
        listingURL:          cfg.ListingURL,
        # 获取时通过哈希验证
        validateByHashOnGet: cfg.ValidateByHashOnGet,
        # 验证年龄
        validateAge:         cfg.ValidateAge,
        # 最大允许构建年龄
        maxAllowedBuiltAge:  cfg.MaxAllowedBuiltAge,
    }, nil
// 返回支持的模式
func (c Curator) SupportedSchema() int {
    return c.targetSchema
}

// 获取存储，返回存储读取器、数据库关闭器和错误
func (c *Curator) GetStore() (grypeDB.StoreReader, grypeDB.DBCloser, error) {
    // 确保数据库完整性
    _, err := c.validateIntegrity(c.dbDir)
    if err != nil {
        return nil, nil, fmt.Errorf("vulnerability database is invalid (run db update to correct): %+v", err)
    }

    // 创建存储对象
    s, err := store.New(c.dbPath, false)
    return s, s, err
}

// 返回当前状态
func (c *Curator) Status() Status {
    // 从目录中创建元数据
    metadata, err := NewMetadataFromDir(c.fs, c.dbDir)
    if err != nil {
        return Status{
            Err: fmt.Errorf("failed to parse database metadata (%s): %w", c.dbDir, err),
        }
    }
    if metadata == nil {
        return Status{
            Err: fmt.Errorf("database metadata not found at %q", c.dbDir),
        }
    }

    return Status{
        Built:         metadata.Built,
        SchemaVersion: metadata.Version,
        Location:      c.dbDir,
        Checksum:      metadata.Checksum,
        Err:           c.Validate(),
    }
}

// 删除特定模式的数据库和元数据文件
func (c *Curator) Delete() error {
    return c.fs.RemoveAll(c.dbDir)
}

// 更新现有数据库，返回是否有任何操作以及错误
func (c *Curator) Update() (bool, error) {
    // 通知消费者有可监控的事件（下载 + 导入阶段）
    importProgress := progress.NewManual(1)
    stage := progress.NewAtomicStage("checking for update")
    downloadProgress := progress.NewManual(1)
    aggregateProgress := progress.NewAggregator(progress.DefaultStrategy, downloadProgress, importProgress)

    // 发布事件
    bus.Publish(partybus.Event{
        Type: event.UpdateVulnerabilityDatabase,
        Value: progress.StagedProgressable(&struct {
            progress.Stager
            progress.Progressable
        }{
            Stager:       progress.Stager(stage),
            Progressable: progress.Progressable(aggregateProgress),
        }),
    })
}
    // 标记下载进度为已完成
    defer downloadProgress.SetCompleted()
    // 标记导入进度为已完成
    defer importProgress.SetCompleted()

    // 检查是否有更新可用，并获取更新元数据和更新条目
    updateAvailable, metadata, updateEntry, err := c.IsUpdateAvailable()
    if err != nil {
        // 如果无法检查更新，记录警告并继续执行
        log.Warnf("unable to check for vulnerability database update")
        log.Debugf("check for vulnerability update failed: %+v", err)
    }
    // 如果有更新可用
    if updateAvailable {
        // 记录下载新漏洞数据库的信息
        log.Infof("downloading new vulnerability DB")
        // 更新漏洞数据库，并传入下载进度、导入进度和阶段信息
        err = c.UpdateTo(updateEntry, downloadProgress, importProgress, stage)
        if err != nil {
            return false, fmt.Errorf("unable to update vulnerability database: %w", err)
        }

        // 如果有元数据，记录更新的漏洞数据库版本信息
        if metadata != nil {
            log.Infof(
                "updated vulnerability DB from version=%d built=%q to version=%d built=%q",
                metadata.Version,
                metadata.Built.String(),
                updateEntry.Version,
                updateEntry.Built.String(),
            )
            return true, nil
        }

        // 如果没有元数据，记录下载的新漏洞数据库版本信息
        log.Infof(
            "downloaded new vulnerability DB version=%d built=%q",
            updateEntry.Version,
            updateEntry.Built.String(),
        )
        return true, nil
    }

    // 设置阶段信息为“无可用更新”
    stage.Set("no update available")
    return false, nil
// IsUpdateAvailable 表示是否有新的更新可用，返回此模式的最新列表信息
func (c *Curator) IsUpdateAvailable() (bool, *Metadata, *ListingEntry, error) {
    // 输出调试信息
    log.Debugf("checking for available database updates")

    // 从 URL 获取列表信息
    listing, err := c.ListingFromURL()
    if err != nil {
        return false, nil, nil, err
    }

    // 获取最佳更新项
    updateEntry := listing.BestUpdate(c.targetSchema)
    if updateEntry == nil {
        return false, nil, nil, fmt.Errorf("no db candidates with correct version available (maybe there is an application update available?)")
    }
    log.Debugf("found database update candidate: %s", updateEntry)

    // 比较创建日期和当前数据库日期
    current, err := NewMetadataFromDir(c.fs, c.dbDir)
    if err != nil {
        return false, nil, nil, fmt.Errorf("current metadata corrupt: %w", err)
    }

    // 如果当前数据库被更新项取代，则返回更新信息
    if current.IsSupersededBy(updateEntry) {
        log.Debugf("database update available: %s", updateEntry)
        return true, current, updateEntry, nil
    }
    log.Debugf("no database update available")

    return false, nil, nil, nil
}

// UpdateTo 使用提供的列表项更新现有数据库
func (c *Curator) UpdateTo(listing *ListingEntry, downloadProgress, importProgress *progress.Manual, stage *progress.AtomicStage) error {
    stage.Set("downloading")
    // 注意：临时目录在下载/验证/激活失败时会被保留，以便进行调查
    tempDir, err := c.download(listing, downloadProgress)
    if err != nil {
        return err
    }

    stage.Set("validating integrity")
    _, err = c.validateIntegrity(tempDir)
    if err != nil {
        return err
    }

    stage.Set("importing")
    err = c.activate(tempDir)
    if err != nil {
        return err
    }
    stage.Set("updated")
    importProgress.Set(importProgress.Size())
    importProgress.SetCompleted()
}
    # 调用 fs 对象的 RemoveAll 方法，删除指定目录下的所有文件和子目录
    return c.fs.RemoveAll(tempDir)
// Validate函数检查当前数据库，以确保文件完整性，并检查它是否可以被应用程序的这个版本使用。
func (c *Curator) Validate() error {
    // 调用validateIntegrity函数验证数据库目录的完整性
    metadata, err := c.validateIntegrity(c.dbDir)
    if err != nil {
        return err
    }

    // 调用validateStaleness函数验证数据库的新旧程度
    return c.validateStaleness(metadata)
}

// ImportFrom函数接受一个DB归档文件，并将其导入到最终的DB位置。
func (c *Curator) ImportFrom(dbArchivePath string) error {
    // 注意：临时目录在下载/验证/激活失败时会被保留，以便进行调查
    tempDir, err := os.MkdirTemp("", "grype-import")
    if err != nil {
        return fmt.Errorf("无法创建DB临时目录：%w", err)
    }

    // 解压缩DB归档文件到临时目录
    err = archiver.Unarchive(dbArchivePath, tempDir)
    if err != nil {
        return err
    }

    // 验证临时目录的完整性
    _, err = c.validateIntegrity(tempDir)
    if err != nil {
        return err
    }

    // 激活临时目录
    err = c.activate(tempDir)
    if err != nil {
        return err
    }

    // 删除临时目录
    return c.fs.RemoveAll(tempDir)
}

// download函数从指定的URL下载数据库，并返回临时目录的路径
func (c *Curator) download(listing *ListingEntry, downloadProgress *progress.Manual) (string, error) {
    // 创建临时目录
    tempDir, err := os.MkdirTemp("", "grype-scratch")
    if err != nil {
        return "", fmt.Errorf("无法创建DB临时目录：%w", err)
    }

    // 下载数据库到临时目录
    url := listing.URL
    query := url.Query()
    query.Add("checksum", listing.Checksum)
    url.RawQuery = query.Encode()
    err = c.downloader.GetToDir(tempDir, listing.URL.String(), downloadProgress)
    if err != nil {
        return "", fmt.Errorf("无法下载DB：%w", err)
    }

    return tempDir, nil
}
// 根据构建时间验证数据库是否过期
func (c *Curator) validateStaleness(m Metadata) error {
    // 如果不需要验证数据库的年龄，则直接返回
    if !c.validateAge {
        return nil
    }

    // 获取当前的 UTC 时间
    now := time.Now().UTC()

    // 计算数据库构建时间与当前时间的差值
    age := now.Sub(m.Built)
    // 如果差值超过了允许的最大年龄，则返回错误
    if age > c.maxAllowedBuiltAge {
        return fmt.Errorf("the vulnerability database was built %s ago (max allowed age is %s)", durafmt.ParseShort(age), durafmt.ParseShort(c.maxAllowedBuiltAge))
    }

    return nil
}

// 验证数据库的完整性
func (c *Curator) validateIntegrity(dbDirPath string) (Metadata, error) {
    // 检查磁盘校验和是否与数据库有效载荷仍然匹配
    metadata, err := NewMetadataFromDir(c.fs, dbDirPath)
    if err != nil {
        return Metadata{}, fmt.Errorf("failed to parse database metadata (%s): %w", dbDirPath, err)
    }
    if metadata == nil {
        return Metadata{}, fmt.Errorf("database metadata not found: %s", dbDirPath)
    }

    // 如果需要在获取时通过哈希验证，则进行验证
    if c.validateByHashOnGet {
        dbPath := path.Join(dbDirPath, FileName)
        valid, actualHash, err := file.ValidateByHash(c.fs, dbPath, metadata.Checksum)
        if err != nil {
            return Metadata{}, err
        }
        if !valid {
            return Metadata{}, fmt.Errorf("bad db checksum (%s): %q vs %q", dbPath, metadata.Checksum, actualHash)
        }
    }

    // 如果目标数据库版本与元数据中的版本不匹配，则返回错误
    if c.targetSchema != metadata.Version {
        return Metadata{}, fmt.Errorf("unsupported database version: have=%d want=%d", metadata.Version, c.targetSchema)
    }

    // TODO: 在这里添加版本检查，以确保应用程序的这个版本可以使用数据库的这个版本（相对于数据库所说的，不仅仅是元数据！）

    return *metadata, nil
}

// 激活将下载的数据库切换到应用程序目录
func (c *Curator) activate(dbDirPath string) error {
    // 检查应用程序目录是否存在
    _, err := c.fs.Stat(c.dbDir)
}
    // 如果错误不是文件不存在错误，则执行下面的操作
    if !os.IsNotExist(err) {
        // 删除任何先前的数据库
        err = c.Delete()
        if err != nil {
            return fmt.Errorf("failed to purge existing database: %w", err)
        }
    }

    // 确保存在应用程序数据库目录
    err = c.fs.MkdirAll(c.dbDir, 0755)
    if err != nil {
        return fmt.Errorf("failed to create db directory: %w", err)
    }

    // 激活新的数据库缓存
    return file.CopyDir(c.fs, dbDirPath, c.dbDir)
// 从 URL 中加载一个 Listing
func (c Curator) ListingFromURL() (Listing, error) {
    // 创建临时文件用于存储下载的 listing 文件
    tempFile, err := afero.TempFile(c.fs, "", "grype-db-listing")
    if err != nil {
        return Listing{}, fmt.Errorf("unable to create listing temp file: %w", err)
    }
    // 延迟执行删除临时文件的操作
    defer func() {
        err := c.fs.RemoveAll(tempFile.Name())
        if err != nil {
            log.Errorf("failed to remove file (%s): %w", tempFile.Name(), err)
        }
    }()

    // 下载 listing 文件
    err = c.downloader.GetFile(tempFile.Name(), c.listingURL)
    if err != nil {
        return Listing{}, fmt.Errorf("unable to download listing: %w", err)
    }

    // 解析 listing 文件
    listing, err := NewListingFromFile(c.fs, tempFile.Name())
    if err != nil {
        return Listing{}, err
    }
    return listing, nil
}

// 创建默认的 HTTP 客户端
func defaultHTTPClient(fs afero.Fs, caCertPath string) (*http.Client, error) {
    // 使用 cleanhttp 包创建默认的 HTTP 客户端
    httpClient := cleanhttp.DefaultClient()
    // 如果提供了根证书路径
    if caCertPath != "" {
        // 创建根证书池
        rootCAs := x509.NewCertPool()

        // 读取根证书文件内容
        pemBytes, err := afero.ReadFile(fs, caCertPath)
        if err != nil {
            return nil, fmt.Errorf("unable to configure root CAs for curator: %w", err)
        }
        // 将 PEM 格式的证书内容添加到根证书池
        rootCAs.AppendCertsFromPEM(pemBytes)

        // 配置 HTTP 客户端的 TLS 设置
        httpClient.Transport.(*http.Transport).TLSClientConfig = &tls.Config{
            MinVersion: tls.VersionTLS12,
            RootCAs:    rootCAs,
        }
    }
    return httpClient, nil
}
```