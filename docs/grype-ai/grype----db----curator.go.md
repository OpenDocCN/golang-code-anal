# `grype\grype\db\curator.go`

```
// 导入 db 包
package db

// 导入所需的包
import (
	"crypto/tls" // 导入加密/解密包
	"crypto/x509" // 导入证书操作包
	"fmt" // 导入格式化包
	"net/http" // 导入 HTTP 包
	"os" // 导入操作系统包
	"path" // 导入路径操作包
	"strconv" // 导入字符串转换包
	"time" // 导入时间包

	"github.com/hako/durafmt" // 导入时间格式化包
	cleanhttp "github.com/hashicorp/go-cleanhttp" // 导入 HTTP 清洁包
	archiver "github.com/mholt/archiver/v3" // 导入归档包
	"github.com/spf13/afero" // 导入文件系统操作包
	partybus "github.com/wagoodman/go-partybus" // 导入事件总线包
	progress "github.com/wagoodman/go-progress" // 导入进度条包

	grypeDB "github.com/anchore/grype/grype/db/v5" // 导入 grypeDB 包
)
// 导入存储模块
"github.com/anchore/grype/grype/db/v5/store"
// 导入事件模块
"github.com/anchore/grype/grype/event"
// 导入漏洞模块
"github.com/anchore/grype/grype/vulnerability"
// 导入总线模块
"github.com/anchore/grype/internal/bus"
// 导入文件模块
"github.com/anchore/grype/internal/file"
// 导入日志模块
"github.com/anchore/grype/internal/log"
)

// 定义常量文件名为漏洞存储文件名
const (
	FileName = grypeDB.VulnerabilityStoreFileName
)

// 定义配置结构体
type Config struct {
	// 数据库根目录
	DBRootDir           string
	// 列表 URL
	ListingURL          string
	// CA 证书
	CACert              string
	// 获取时验证哈希
	ValidateByHashOnGet bool
	// 验证年龄
	ValidateAge         bool
	// 最大允许构建年龄
	MaxAllowedBuiltAge  time.Duration
}
// 定义了一个名为 Curator 的结构体，包含了文件系统、文件下载器、目标模式、数据库目录、数据库路径、列表 URL、获取时是否通过哈希验证、验证年龀、最大允许构建年龀等字段
type Curator struct {
	fs                  afero.Fs
	downloader          file.Getter
	targetSchema        int
	dbDir               string
	dbPath              string
	listingURL          string
	validateByHashOnGet bool
	validateAge         bool
	maxAllowedBuiltAge  time.Duration
}

// 创建一个新的 Curator 对象，根据传入的配置信息
func NewCurator(cfg Config) (Curator, error) {
	// 根据配置中的数据库根目录和漏洞模式版本号，拼接出数据库目录路径
	dbDir := path.Join(cfg.DBRootDir, strconv.Itoa(vulnerability.SchemaVersion))

	// 创建一个新的操作系统文件系统对象
	fs := afero.NewOsFs()
	
	// 根据文件系统和配置中的 CA 证书路径，创建一个默认的 HTTP 客户端
	httpClient, err := defaultHTTPClient(fs, cfg.CACert)
	if err != nil {
		// 如果创建失败，则返回空的 Curator 对象和错误信息
		return Curator{}, err
# 返回一个Curator对象，包含了文件系统、目标模式、下载器、数据库目录、数据库路径、列表URL、获取时的哈希验证、验证年龄、允许的最大构建年龄等属性
	return Curator{
		fs:                  fs,
		targetSchema:        vulnerability.SchemaVersion,
		downloader:          file.NewGetter(httpClient),
		dbDir:               dbDir,
		dbPath:              path.Join(dbDir, FileName),
		listingURL:          cfg.ListingURL,
		validateByHashOnGet: cfg.ValidateByHashOnGet,
		validateAge:         cfg.ValidateAge,
		maxAllowedBuiltAge:  cfg.MaxAllowedBuiltAge,
	}, nil
}

# 返回Curator对象的支持模式
func (c Curator) SupportedSchema() int {
	return c.targetSchema
}

# 获取存储器的读取器、数据库关闭器和错误信息
func (c *Curator) GetStore() (grypeDB.StoreReader, grypeDB.DBCloser, error) {
// 确保数据库状态正常
_, err := c.validateIntegrity(c.dbDir)
if err != nil {
    return nil, nil, fmt.Errorf("vulnerability database is invalid (run db update to correct): %+v", err)
}

// 创建一个新的存储对象，用于访问数据库
s, err := store.New(c.dbPath, false)
return s, s, err
}

// 获取当前数据库的状态
func (c *Curator) Status() Status {
    // 从数据库目录中获取元数据
    metadata, err := NewMetadataFromDir(c.fs, c.dbDir)
    if err != nil {
        return Status{
            Err: fmt.Errorf("failed to parse database metadata (%s): %w", c.dbDir, err),
        }
    }
    // 如果未找到元数据，则返回错误状态
    if metadata == nil {
        return Status{
            Err: fmt.Errorf("database metadata not found at %q", c.dbDir),
// 返回包含元数据信息的状态对象
return Status{
    Built:         metadata.Built,  // 设置状态对象的构建时间
    SchemaVersion: metadata.Version,  // 设置状态对象的模式版本
    Location:      c.dbDir,  // 设置状态对象的数据库目录
    Checksum:      metadata.Checksum,  // 设置状态对象的校验和
    Err:           c.Validate(),  // 设置状态对象的错误信息
}

// 删除特定模式的数据库和元数据文件
func (c *Curator) Delete() error {
    return c.fs.RemoveAll(c.dbDir)  // 删除数据库目录及其内容
}

// 更新现有数据库，返回是否有任何操作以及可能的错误
func (c *Curator) Update() (bool, error) {
    // 通知消费者可监控的事件（下载和导入阶段）
# 导入 progress 包，并创建一个手动进度条，初始值为 1
importProgress := progress.NewManual(1)
# 创建一个原子阶段，用于表示检查更新的阶段
stage := progress.NewAtomicStage("checking for update")
# 创建一个手动进度条，初始值为 1，用于表示下载进度
downloadProgress := progress.NewManual(1)
# 创建一个进度聚合器，使用默认策略，将下载进度和导入进度进行聚合
aggregateProgress := progress.NewAggregator(progress.DefaultStrategy, downloadProgress, importProgress)

# 发布一个事件，类型为更新漏洞数据库，值为分阶段的进度条
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

# 延迟设置下载进度条为已完成
defer downloadProgress.SetCompleted()
# 延迟设置导入进度条为已完成
defer importProgress.SetCompleted()

# 调用 c.IsUpdateAvailable() 方法，获取更新是否可用、元数据、更新条目和错误信息
updateAvailable, metadata, updateEntry, err := c.IsUpdateAvailable()
	if err != nil {
		// 如果发生错误，记录警告信息并继续执行
		log.Warnf("unable to check for vulnerability database update")
		log.Debugf("check for vulnerability update failed: %+v", err)
	}
	if updateAvailable {
		// 如果有更新可用，记录信息并执行更新操作
		log.Infof("downloading new vulnerability DB")
		err = c.UpdateTo(updateEntry, downloadProgress, importProgress, stage)
		if err != nil {
			// 如果更新操作失败，返回错误信息
			return false, fmt.Errorf("unable to update vulnerability database: %w", err)
		}

		if metadata != nil {
			// 如果有元数据，记录更新后的版本信息
			log.Infof(
				"updated vulnerability DB from version=%d built=%q to version=%d built=%q",
				metadata.Version,
				metadata.Built.String(),
				updateEntry.Version,
				updateEntry.Built.String(),
			)
```
// 返回 true 和 nil，表示没有更新可用
return true, nil

// 使用日志记录已下载的新漏洞数据库版本和构建日期
log.Infof(
    "downloaded new vulnerability DB version=%d built=%q",
    updateEntry.Version,
    updateEntry.Built.String(),
)
return true, nil

// 设置阶段为“没有可用的更新”
stage.Set("no update available")
return false, nil

// IsUpdateAvailable 函数指示是否有新的更新可用，作为布尔值，并返回此模式的最新列表信息
func (c *Curator) IsUpdateAvailable() (bool, *Metadata, *ListingEntry, error) {
// 使用日志记录检查是否有可用的数据库更新
log.Debugf("checking for available database updates")
	// 从URL获取列表，如果出现错误则返回错误信息
	listing, err := c.ListingFromURL()
	if err != nil {
		return false, nil, nil, err
	}

	// 从列表中找到最佳的更新条目
	updateEntry := listing.BestUpdate(c.targetSchema)
	if updateEntry == nil {
		// 如果没有找到正确版本的数据库候选项，则返回错误信息
		return false, nil, nil, fmt.Errorf("no db candidates with correct version available (maybe there is an application update available?)")
	}
	// 打印找到的数据库更新候选项
	log.Debugf("found database update candidate: %s", updateEntry)

	// 比较创建的数据与当前数据库日期
	current, err := NewMetadataFromDir(c.fs, c.dbDir)
	if err != nil {
		// 如果当前元数据损坏，则返回错误信息
		return false, nil, nil, fmt.Errorf("current metadata corrupt: %w", err)
	}

	// 如果当前数据库被更新候选项取代，则打印数据库更新可用信息，并返回相应的结果
	if current.IsSupersededBy(updateEntry) {
		log.Debugf("database update available: %s", updateEntry)
		return true, current, updateEntry, nil
	}
	}
	// 记录调试信息，表示没有可用的数据库更新
	log.Debugf("no database update available")

	// 返回 false 和四个 nil 值
	return false, nil, nil, nil
}

// UpdateTo 使用提供的列表条目更新现有的数据库。
func (c *Curator) UpdateTo(listing *ListingEntry, downloadProgress, importProgress *progress.Manual, stage *progress.AtomicStage) error {
	// 设置阶段为“downloading”
	stage.Set("downloading")
	// 注意：临时目录在下载/验证/激活失败时会被保留，以便进行调查
	tempDir, err := c.download(listing, downloadProgress)
	if err != nil {
		return err
	}

	// 设置阶段为“validating integrity”
	stage.Set("validating integrity")
	_, err = c.validateIntegrity(tempDir)
	if err != nil {
		return err
	}
// 设置阶段为“导入中”
stage.Set("importing")
// 激活临时目录
err = c.activate(tempDir)
// 如果出现错误，返回错误信息
if err != nil {
    return err
}
// 设置阶段为“已更新”
stage.Set("updated")
// 设置导入进度为总大小
importProgress.Set(importProgress.Size())
// 设置导入进度为已完成
importProgress.SetCompleted()

// 删除临时目录
return c.fs.RemoveAll(tempDir)
}

// Validate 检查当前数据库以确保文件完整性，并检查是否可以被应用程序的这个版本使用
func (c *Curator) Validate() error {
    // 验证数据库完整性，并获取元数据
    metadata, err := c.validateIntegrity(c.dbDir)
    // 如果出现错误，返回错误信息
    if err != nil {
        return err
    }
// validateStaleness 方法用于验证元数据的新旧程度
return c.validateStaleness(metadata)
}

// ImportFrom 方法用于从 DB 存档文件中导入数据到最终的 DB 位置
func (c *Curator) ImportFrom(dbArchivePath string) error {
	// 注意：临时目录在下载/验证/激活失败时会被保留，以便进行调查
	// 创建临时目录用于解压缩存档文件
	tempDir, err := os.MkdirTemp("", "grype-import")
	if err != nil {
		return fmt.Errorf("unable to create db temp dir: %w", err)
	}

	// 解压缩存档文件到临时目录
	err = archiver.Unarchive(dbArchivePath, tempDir)
	if err != nil {
		return err
	}

	// 验证临时目录的完整性
	_, err = c.validateIntegrity(tempDir)
	if err != nil {
		return err
	}
// 使用临时目录激活下载
err = c.activate(tempDir)
if err != nil {
    return err
}

// 删除临时目录
return c.fs.RemoveAll(tempDir)
}

// 下载函数，传入列表和下载进度，返回临时目录和错误
func (c *Curator) download(listing *ListingEntry, downloadProgress *progress.Manual) (string, error) {
    // 创建临时目录
    tempDir, err := os.MkdirTemp("", "grype-scratch")
    if err != nil {
        return "", fmt.Errorf("unable to create db temp dir: %w", err)
    }

    // 下载数据库到临时目录
    url := listing.URL

    // 从 go-getter 中，添加一个校验和作为查询字符串，用于在下载后验证有效载荷
    // 注意：校验和查询参数不会发送到服务器
	query := url.Query()
	// 创建一个新的查询参数对象

	query.Add("checksum", listing.Checksum)
	// 向查询参数对象中添加名为"checksum"的参数，值为listing.Checksum

	url.RawQuery = query.Encode()
	// 将查询参数对象编码并设置为URL的原始查询部分

	// go-getter will automatically extract all files within the archive to the temp dir
	// 使用go-getter自动将存档中的所有文件提取到临时目录

	err = c.downloader.GetToDir(tempDir, listing.URL.String(), downloadProgress)
	// 使用下载器将listing.URL指定的文件下载到临时目录，并提供下载进度回调函数downloadProgress

	if err != nil {
		return "", fmt.Errorf("unable to download db: %w", err)
		// 如果下载出现错误，则返回错误信息
	}

	return tempDir, nil
	// 返回临时目录路径和空错误值
}

// validateStaleness ensures the vulnerability database has not passed
// the max allowed age, calculated from the time it was built until now.
// validateStaleness函数确保漏洞数据库的年龄未超过允许的最大值，从构建时间到当前时间计算。

func (c *Curator) validateStaleness(m Metadata) error {
	// 如果不需要验证年龄，则直接返回nil
	if !c.validateAge {
		return nil
	}
// 获取当前时间，并转换为UTC时间
now := time.Now().UTC()

// 计算构建时间与当前时间的差值
age := now.Sub(m.Built)
// 如果差值超过最大允许的构建时间，则返回错误信息
if age > c.maxAllowedBuiltAge {
    return fmt.Errorf("the vulnerability database was built %s ago (max allowed age is %s)", durafmt.ParseShort(age), durafmt.ParseShort(c.maxAllowedBuiltAge))
}

// 验证数据库完整性
func (c *Curator) validateIntegrity(dbDirPath string) (Metadata, error) {
    // 检查磁盘校验和是否与数据库有效负载仍然匹配
    metadata, err := NewMetadataFromDir(c.fs, dbDirPath)
    if err != nil {
        return Metadata{}, fmt.Errorf("failed to parse database metadata (%s): %w", dbDirPath, err)
    }
    // 如果未找到数据库元数据，则返回错误信息
    if metadata == nil {
        return Metadata{}, fmt.Errorf("database metadata not found: %s", dbDirPath)
}
	}

	// 如果设置了根据哈希值验证，则验证数据库文件的哈希值
	if c.validateByHashOnGet {
		// 获取数据库文件路径
		dbPath := path.Join(dbDirPath, FileName)
		// 验证数据库文件的哈希值是否匹配
		valid, actualHash, err := file.ValidateByHash(c.fs, dbPath, metadata.Checksum)
		if err != nil {
			return Metadata{}, err
		}
		// 如果哈希值不匹配，则返回错误
		if !valid {
			return Metadata{}, fmt.Errorf("bad db checksum (%s): %q vs %q", dbPath, metadata.Checksum, actualHash)
		}
	}

	// 如果目标数据库版本与元数据中的版本不匹配，则返回错误
	if c.targetSchema != metadata.Version {
		return Metadata{}, fmt.Errorf("unsupported database version: have=%d want=%d", metadata.Version, c.targetSchema)
	}

	// TODO: 在这里添加版本检查，以确保应用程序的这个版本可以使用数据库的这个版本（相对于数据库所说的版本，而不仅仅是元数据！）

	// 返回元数据和空错误
	return *metadata, nil
// activate 方法用于激活已下载的数据库到应用程序目录
func (c *Curator) activate(dbDirPath string) error {
    // 检查数据库目录是否存在，如果存在则删除之前的数据库
    _, err := c.fs.Stat(c.dbDir)
    if !os.IsNotExist(err) {
        err = c.Delete()
        if err != nil {
            return fmt.Errorf("failed to purge existing database: %w", err)
        }
    }

    // 确保应用程序数据库目录存在，如果不存在则创建
    err = c.fs.MkdirAll(c.dbDir, 0755)
    if err != nil {
        return fmt.Errorf("failed to create db directory: %w", err)
    }

    // 激活新的数据库缓存
}
// 将文件夹复制到指定路径
return file.CopyDir(c.fs, dbDirPath, c.dbDir)
}

// 从URL加载一个Listing
func (c Curator) ListingFromURL() (Listing, error) {
    // 创建临时文件
    tempFile, err := afero.TempFile(c.fs, "", "grype-db-listing")
    if err != nil {
        return Listing{}, fmt.Errorf("unable to create listing temp file: %w", err)
    }
    // 延迟执行，确保在函数返回时删除临时文件
    defer func() {
        err := c.fs.RemoveAll(tempFile.Name())
        if err != nil {
            log.Errorf("failed to remove file (%s): %w", tempFile.Name(), err)
        }
    }()

    // 下载listing文件
    err = c.downloader.GetFile(tempFile.Name(), c.listingURL)
    if err != nil {
        return Listing{}, fmt.Errorf("unable to download listing: %w", err)
```

	}

	// 解析列表文件
	listing, err := NewListingFromFile(c.fs, tempFile.Name())
	if err != nil {
		return Listing{}, err
	}
	return listing, nil
}

func defaultHTTPClient(fs afero.Fs, caCertPath string) (*http.Client, error) {
	// 创建一个默认的 HTTP 客户端
	httpClient := cleanhttp.DefaultClient()
	// 如果提供了 CA 证书路径，则配置根证书
	if caCertPath != "" {
		// 创建一个新的证书池
		rootCAs := x509.NewCertPool()

		// 从文件系统中读取 PEM 格式的证书
		pemBytes, err := afero.ReadFile(fs, caCertPath)
		if err != nil {
			return nil, fmt.Errorf("unable to configure root CAs for curator: %w", err)
		}
		// 将 PEM 格式的证书添加到证书池中
		rootCAs.AppendCertsFromPEM(pemBytes)
# 设置 HTTP 客户端的传输层安全配置，指定最小的 TLS 版本为 TLS 1.2，并指定根证书
httpClient.Transport.(*http.Transport).TLSClientConfig = &tls.Config{
    MinVersion: tls.VersionTLS12,
    RootCAs:    rootCAs,
}
# 返回配置好的 HTTP 客户端
return httpClient, nil
```