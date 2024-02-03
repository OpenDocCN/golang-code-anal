# `kubo\repo\fsrepo\fsrepo.go`

```go
package fsrepo

import (
    "context"  // 上下文包，用于控制函数调用的超时、取消和截止
    "errors"  // 错误处理包，用于创建和处理错误
    "fmt"  // 格式化包，用于格式化输出
    "io"  // 输入输出包，提供了基本的输入输出功能
    "net"  // 网络包，提供了基本的网络功能
    "os"  // 操作系统包，提供了与操作系统交互的功能
    "path/filepath"  // 文件路径包，提供了处理文件路径的功能
    "strings"  // 字符串包，提供了处理字符串的功能
    "sync"  // 同步包，提供了并发编程的基本同步原语

    filestore "github.com/ipfs/boxo/filestore"  // 文件存储包，用于文件存储
    keystore "github.com/ipfs/boxo/keystore"  // 密钥存储包，用于密钥存储
    repo "github.com/ipfs/kubo/repo"  // 仓库包，用于仓库管理
    "github.com/ipfs/kubo/repo/common"  // 仓库公共包，提供了仓库管理的公共功能
    dir "github.com/ipfs/kubo/thirdparty/dir"  // 目录包，提供了目录操作的功能
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 资源管理包，用于资源管理

    util "github.com/ipfs/boxo/util"  // 实用工具包，提供了一些实用的工具函数
    ds "github.com/ipfs/go-datastore"  // 数据存储包，提供了数据存储的功能
    measure "github.com/ipfs/go-ds-measure"  // 测量包，用于测量数据存储的性能
    lockfile "github.com/ipfs/go-fs-lock"  // 文件锁包，用于文件锁定
    logging "github.com/ipfs/go-log"  // 日志包，用于日志记录
    config "github.com/ipfs/kubo/config"  // 配置包，用于配置管理
    serialize "github.com/ipfs/kubo/config/serialize"  // 序列化包，用于配置序列化
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // 仓库迁移包，用于仓库迁移
    homedir "github.com/mitchellh/go-homedir"  // 主目录包，用于获取用户主目录
    ma "github.com/multiformats/go-multiaddr"  // 多地址包，用于处理多格式地址
)

// LockFile is the filename of the repo lock, relative to config dir
// TODO rename repo lock and hide name.
const LockFile = "repo.lock"  // 仓库锁文件的文件名，相对于配置目录

var log = logging.Logger("fsrepo")  // 创建日志记录器

// RepoVersion is the version number that we are currently expecting to see.
var RepoVersion = 15  // 期望看到的仓库版本号

var migrationInstructions = `See https://github.com/ipfs/fs-repo-migrations/blob/master/run.md
Sorry for the inconvenience. In the future, these will run automatically.`  // 迁移说明，提供了迁移的链接和提示信息

var programTooLowMessage = `Your programs version (%d) is lower than your repos (%d).
Please update ipfs to a version that supports the existing repo, or run
a migration in reverse.

See https://github.com/ipfs/fs-repo-migrations/blob/master/run.md for details.`  // 程序版本过低的提示信息，提供了更新和迁移的链接和详细信息

var (
    ErrNoVersion     = errors.New("no version file found, please run 0-to-1 migration tool.\n" + migrationInstructions)  // 未找到版本文件的错误
    ErrOldRepo       = errors.New("ipfs repo found in old '~/.go-ipfs' location, please run migration tool.\n" + migrationInstructions)  // 在旧的 '~/.go-ipfs' 位置找到 ipfs 仓库的错误
    ErrNeedMigration = errors.New("ipfs repo needs migration, please run migration tool.\n" + migrationInstructions)  // ipfs 仓库需要迁移的错误
)

type NoRepoError struct {
    Path string  // 无仓库错误结构体，包含路径字段
}
// 定义一个名为 NoRepoError 的结构体类型，实现 Error 方法，返回错误信息
var _ error = NoRepoError{}

func (err NoRepoError) Error() string {
    return fmt.Sprintf("no IPFS repo found in %s.\nplease run: 'ipfs init'", err.Path)
}

// 定义常量
const (
    apiFile      = "api"
    gatewayFile  = "gateway"
    swarmKeyFile = "swarm.key"
)

const specFn = "datastore_spec"

var (
    // packageLock 用于在修改 FSRepo 的状态字段时保持锁定
    packageLock sync.Mutex

    // onlyOne 用于跟踪打开的 FSRepo 实例
    onlyOne repo.OnlyOne
)

// FSRepo 表示一个 IPFS 文件系统 Repo，可以被多个调用者安全使用
type FSRepo struct {
    // 表示 Close 方法是否已经被调用
    closed bool
    // path 是文件系统路径
    path string
    // configFilePath 是配置文件的路径
    configFilePath string
    // lockfile 是文件系统锁，用于防止其他人同时打开相同的 fsrepo 路径
    lockfile              io.Closer
    config                *config.Config
    userResourceOverrides rcmgr.PartialLimitConfig
    ds                    repo.Datastore
    keystore              keystore.Keystore
    filemgr               *filestore.FileManager
}

// 实现 Repo 接口
var _ repo.Repo = (*FSRepo)(nil)

// 打开指定路径的 FSRepo，如果 repo 未初始化，则返回错误
func Open(repoPath string) (repo.Repo, error) {
    fn := func() (repo.Repo, error) {
        return open(repoPath, "")
    }
    # 调用onlyOne对象的Open方法，传入repoPath和fn作为参数，并返回结果
    return onlyOne.Open(repoPath, fn)
// OpenWithUserConfig函数是等同于上面Open函数的函数，但是可以设置配置文件路径而不是使用默认值。
func OpenWithUserConfig(repoPath string, userConfigFilePath string) (repo.Repo, error) {
    // 创建一个闭包函数fn，用于调用open函数
    fn := func() (repo.Repo, error) {
        return open(repoPath, userConfigFilePath)
    }
    // 调用onlyOne包的Open函数，传入repoPath和fn函数
    return onlyOne.Open(repoPath, fn)
}

// open函数用于打开仓库，接收仓库路径和用户配置文件路径作为参数
func open(repoPath string, userConfigFilePath string) (repo.Repo, error) {
    // 加锁
    packageLock.Lock()
    // 延迟解锁
    defer packageLock.Unlock()

    // 创建一个新的文件系统仓库r
    r, err := newFSRepo(repoPath, userConfigFilePath)
    if err != nil {
        return nil, err
    }

    // 检查仓库是否已初始化
    if err := checkInitialized(r.path); err != nil {
        return nil, err
    }

    // 锁定文件，如果出错则返回错误
    r.lockfile, err = lockfile.Lock(r.path, LockFile)
    if err != nil {
        return nil, err
    }
    keepLocked := false
    // 延迟函数，用于在出错时解锁文件，成功时保持锁定状态
    defer func() {
        if !keepLocked {
            r.lockfile.Close()
        }
    }()

    // 检查仓库版本，如果不匹配则返回错误
    ver, err := migrations.RepoVersion(r.path)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, ErrNoVersion
        }
        return nil, err
    }

    if RepoVersion > ver {
        return nil, ErrNeedMigration
    } else if ver > RepoVersion {
        return nil, fmt.Errorf(programTooLowMessage, RepoVersion, ver)
    }

    // 检查仓库路径，然后检查所有组成部分
    if err := dir.Writable(r.path); err != nil {
        return nil, err
    }

    // 打开配置文件
    if err := r.openConfig(); err != nil {
        return nil, err
    }

    // 打开用户资源覆盖
    if err := r.openUserResourceOverrides(); err != nil {
        return nil, err
    }

    // 打开数据存储
    if err := r.openDatastore(); err != nil {
        return nil, err
    }

    // 打开密钥存储
    if err := r.openKeystore(); err != nil {
        return nil, err
    }
}
    # 如果文件存储或 URL 存储功能已启用
    if r.config.Experimental.FilestoreEnabled || r.config.Experimental.UrlstoreEnabled:
        # 创建文件管理器对象，传入数据存储对象和路径
        r.filemgr = filestore.NewFileManager(r.ds, filepath.Dir(r.path))
        # 设置文件管理器允许文件存储的状态
        r.filemgr.AllowFiles = r.config.Experimental.FilestoreEnabled
        # 设置文件管理器允许 URL 存储的状态
        r.filemgr.AllowUrls = r.config.Experimental.UrlstoreEnabled

    # 设置 keepLocked 变量为 true
    keepLocked = true
    # 返回 r 和 nil
    return r, nil
}
// 创建一个新的文件系统仓库
func newFSRepo(rpath string, userConfigFilePath string) (*FSRepo, error) {
    // 将路径进行清理和展开
    expPath, err := homedir.Expand(filepath.Clean(rpath))
    if err != nil {
        return nil, err
    }

    // 获取配置文件路径
    configFilePath, err := config.Filename(rpath, userConfigFilePath)
    if err != nil {
        // FIXME: 当用户配置路径为""时，个性化处理此处
        return nil, fmt.Errorf("finding config filepath from repo %s and user config %s: %w",
            rpath, userConfigFilePath, err)
    }
    return &FSRepo{path: expPath, configFilePath: configFilePath}, nil
}

// 检查仓库是否已初始化
func checkInitialized(path string) error {
    if !isInitializedUnsynced(path) {
        // 替换路径中的".ipfs"为".go-ipfs"
        alt := strings.Replace(path, ".ipfs", ".go-ipfs", 1)
        if isInitializedUnsynced(alt) {
            return ErrOldRepo
        }
        return NoRepoError{Path: path}
    }
    return nil
}

// configIsInitialized 在提供的路径上返回true，如果仓库已初始化
func configIsInitialized(path string) bool {
    configFilename, err := config.Filename(path, "")
    if err != nil {
        return false
    }
    if !util.FileExists(configFilename) {
        return false
    }
    return true
}

// 初始化配置
func initConfig(path string, conf *config.Config) error {
    if configIsInitialized(path) {
        return nil
    }
    configFilename, err := config.Filename(path, "")
    if err != nil {
        return err
    }
    // 初始化是唯一一次可以在不读取磁盘上的配置并合并任何可能存在的用户提供的键的情况下写入配置的时机
    if err := serialize.WriteConfigFile(configFilename, conf); err != nil {
        return err
    }

    return nil
}

// 初始化规范
func initSpec(path string, conf map[string]interface{}) error {
    fn, err := config.Path(path, specFn)
    if err != nil {
        return err
    }

    if util.FileExists(fn) {
        return nil
    }

    dsc, err := AnyDatastoreConfig(conf)
    if err != nil {
        return err
    }
}
    # 获取磁盘规格的字节数据
    bytes := dsc.DiskSpec().Bytes()
    # 将字节数据写入文件，设置文件权限为600
    return os.WriteFile(fn, bytes, 0o600)
// Init函数用于在给定路径上使用提供的配置初始化一个新的FSRepo。
// TODO 添加对自定义数据存储的支持。
func Init(repoPath string, conf *config.Config) error {
    // 必须持有packageLock以确保仓库不会被初始化多次。
    packageLock.Lock()
    defer packageLock.Unlock()

    if isInitializedUnsynced(repoPath) {
        return nil
    }

    if err := initConfig(repoPath, conf); err != nil {
        return err
    }

    if err := initSpec(repoPath, conf.Datastore.Spec); err != nil {
        return err
    }

    if err := migrations.WriteRepoVersion(repoPath, RepoVersion); err != nil {
        return err
    }

    return nil
}

// LockedByOtherProcess函数在FSRepo被其他进程锁定时返回true。
// 如果为true，则该仓库无法被本进程打开。
func LockedByOtherProcess(repoPath string) (bool, error) {
    repoPath = filepath.Clean(repoPath)
    locked, err := lockfile.Locked(repoPath, LockFile)
    if locked {
        log.Debugf("(%t)<->Lock is held at %s", locked, repoPath)
    }
    return locked, err
}

// APIAddr函数根据fsrepo中的api文件返回注册的API地址。
// 这是一个并发操作，意味着任何进程都可以读取此文件。
// 因此，修改此文件应该使用"mv"来替换整个文件，并避免交错的读/写。
func APIAddr(repoPath string) (ma.Multiaddr, error) {
    repoPath = filepath.Clean(repoPath)
    apiFilePath := filepath.Join(repoPath, apiFile)

    // 如果没有文件，则假定没有api地址。
    f, err := os.Open(apiFilePath)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, repo.ErrApiNotRunning
        }
        return nil, err
    }
    defer f.Close()

    // 读取最多2048字节。io.ReadAll存在漏洞，因为某人可以通过在那里放置一个大文件来破坏进程。
    //
    // 设置一个注释，提醒后续维护者注意之前的注释可能存在问题，但保留了限制值
    // 1. 保证不会读取太少的数据
    // 2. 避免截断数据并且成功返回
    // 从文件流中读取最多 2048 字节的数据到缓冲区中
    buf, err := io.ReadAll(io.LimitReader(f, 2048))
    // 如果读取过程中出现错误，则返回 nil 和错误信息
    if err != nil {
        return nil, err
    }
    // 如果缓冲区的长度等于 2048，则返回 nil 和错误信息，说明 API 文件太大，必须小于 2048 字节
    if len(buf) == 2048 {
        return nil, fmt.Errorf("API file too large, must be <2048 bytes long: %s", apiFilePath)
    }
    
    // 将缓冲区的内容转换为字符串
    s := string(buf)
    // 去除字符串两端的空白字符
    s = strings.TrimSpace(s)
    // 根据处理后的字符串创建并返回一个新的 Multiaddr 对象
    return ma.NewMultiaddr(s)
// 返回存储库的密钥库
func (r *FSRepo) Keystore() keystore.Keystore {
    return r.keystore
}

// 返回存储库的路径
func (r *FSRepo) Path() string {
    return r.path
}

// SetAPIAddr 将 API 地址写入 /api 文件
func (r *FSRepo) SetAPIAddr(addr ma.Multiaddr) error {
    // 创建临时文件以写入地址，以防程序在创建文件后崩溃而留下空文件
    f, err := os.Create(filepath.Join(r.path, "."+apiFile+".tmp"))
    if err != nil {
        return err
    }

    if _, err = f.WriteString(addr.String()); err != nil {
        return err
    }
    if err = f.Close(); err != nil {
        return err
    }

    // 原子性地将临时文件重命名为正确的文件名
    if err = os.Rename(filepath.Join(r.path, "."+apiFile+".tmp"), filepath.Join(r.path,
        apiFile)); err == nil {
        return nil
    }
    // 在重命名返回错误时删除临时文件
    if err1 := os.Remove(filepath.Join(r.path, "."+apiFile+".tmp")); err1 != nil {
        return fmt.Errorf("file Rename error: %s, file remove error: %s", err.Error(),
            err1.Error())
    }
    return err
}

// SetGatewayAddr 将网关地址写入 /gateway 文件
func (r *FSRepo) SetGatewayAddr(addr net.Addr) error {
    // 创建临时文件以写入地址，以防程序在创建文件后崩溃而留下空文件
    tmpPath := filepath.Join(r.path, "."+gatewayFile+".tmp")
    f, err := os.Create(tmpPath)
    if err != nil {
        return err
    }
    var good bool
    // 在最坏的情况下使用延迟静默删除
    defer func() {
        if !good {
            os.Remove(tmpPath)
        }
    }()
    defer f.Close()

    if _, err := fmt.Fprintf(f, "http://%s", addr.String()); err != nil {
        return err
    }
    if err := f.Close(); err != nil {
        return err
    }

    // 原子性地将临时文件重命名为正确的文件名
    err = os.Rename(tmpPath, filepath.Join(r.path, gatewayFile))
    // 检查错误是否为空，如果为空则表示操作成功
    good = err == nil
    // 如果操作成功，则返回空错误
    if good {
        return nil
    }
    // 当重命名操作返回错误时，删除临时文件
    if err1 := os.Remove(tmpPath); err1 != nil {
        // 返回包含重命名错误和删除文件错误的格式化错误
        return fmt.Errorf("file Rename error: %w, file remove error: %s", err, err1.Error())
    }
    // 返回原始错误
    return err
// openConfig函数会打开配置文件，如果文件不存在则返回错误。
func (r *FSRepo) openConfig() error {
    // 使用serialize.Load函数加载配置文件，将结果赋值给conf和err
    conf, err := serialize.Load(r.configFilePath)
    // 如果加载配置文件出错，则返回错误
    if err != nil {
        return err
    }
    // 将加载的配置文件赋值给r.config
    r.config = conf
    // 返回nil
    return nil
}

// openUserResourceOverrides函数会打开用户资源覆盖文件，如果文件不存在则移除所有覆盖，如果解码失败则返回错误。
func (r *FSRepo) openUserResourceOverrides() error {
    // 从指定路径读取配置文件到r.userResourceOverrides，如果文件未初始化则返回nil
    err := serialize.ReadConfigFile(filepath.Join(r.path, "libp2p-resource-limit-overrides.json"), &r.userResourceOverrides)
    if errors.Is(err, serialize.ErrNotInitialized) {
        err = nil
    }
    // 返回err
    return err
}

// openKeystore函数会打开密钥库，如果出错则返回错误。
func (r *FSRepo) openKeystore() error {
    // 将密钥库路径和错误信息赋值给ksp和err
    ksp := filepath.Join(r.path, "keystore")
    ks, err := keystore.NewFSKeystore(ksp)
    // 如果出错则返回错误
    if err != nil {
        return err
    }
    // 将密钥库赋值给r.keystore
    r.keystore = ks
    // 返回nil
    return nil
}

// openDatastore函数会打开数据存储，如果配置文件不存在则返回错误。
func (r *FSRepo) openDatastore() error {
    // 如果配置文件中的数据存储类型和路径不为空，则返回错误
    if r.config.Datastore.Type != "" || r.config.Datastore.Path != "" {
        return fmt.Errorf("old style datatstore config detected")
    } else if r.config.Datastore.Spec == nil {
        return fmt.Errorf("required Datastore.Spec entry missing from config file")
    }
    // 如果配置文件中的数据存储NoSync为true，则输出警告信息
    if r.config.Datastore.NoSync {
        log.Warn("NoSync is now deprecated in favor of datastore specific settings. If you want to disable fsync on flatfs set 'sync' to false. See https://github.com/ipfs/kubo/blob/master/docs/datastores.md#flatfs.")
    }
    // 从配置文件中获取数据存储配置，并赋值给dsc和err
    dsc, err := AnyDatastoreConfig(r.config.Datastore.Spec)
    // 如果出错则返回错误
    if err != nil {
        return err
    }
    // 获取数据存储的规格
    spec := dsc.DiskSpec()
    // 读取旧的数据存储规格，并赋值给oldSpec和err
    oldSpec, err := r.readSpec()
    // 如果出错则返回错误
    if err != nil {
        return err
    }
    // 如果旧的数据存储规格和当前规格不一致，则返回错误
    if oldSpec != spec.String() {
        return fmt.Errorf("datastore configuration of '%s' does not match what is on disk '%s'",
            oldSpec, spec.String())
    }
}
    // 调用 Create 方法创建一个数据存储对象，并将返回的对象赋值给变量 d，同时检查是否有错误发生
    d, err := dsc.Create(r.path)
    // 如果有错误发生，则返回该错误
    if err != nil {
        return err
    }
    // 将创建的数据存储对象赋值给 r.ds
    r.ds = d

    // 使用 measure 包对数据存储对象进行包装，以便进行指标收集
    prefix := "ipfs.fsrepo.datastore"
    r.ds = measure.New(prefix, r.ds)

    // 返回空值，表示没有错误发生
    return nil
// Close 方法关闭 FSRepo，释放持有的资源。
func (r *FSRepo) Close() error {
    // 获取包锁
    packageLock.Lock()
    // 延迟释放包锁
    defer packageLock.Unlock()

    // 如果已经关闭，则返回错误信息
    if r.closed {
        return errors.New("repo is closed")
    }

    // 删除 api 文件，如果出错且不是文件不存在的错误，则记录警告
    err := os.Remove(filepath.Join(r.path, apiFile))
    if err != nil && !os.IsNotExist(err) {
        log.Warn("error removing api file: ", err)
    }

    // 删除网关文件，如果出错且不是文件不存在的错误，则记录警告
    err = os.Remove(filepath.Join(r.path, gatewayFile))
    if err != nil && !os.IsNotExist(err) {
        log.Warn("error removing gateway file: ", err)
    }

    // 关闭数据存储
    if err := r.ds.Close(); err != nil {
        return err
    }

    // 标记为已关闭
    r.closed = true
    // 关闭锁文件
    return r.lockfile.Close()
}

// Config 返回当前配置。该函数不会复制配置。调用者在修改配置之前必须先调用 `Clone`。
//
// 当未打开时的结果是未定义的。如果愿意，该方法可能会引发 panic。
func (r *FSRepo) Config() (*config.Config, error) {
    // 由于存储库处于已打开状态，因此不需要持有包锁。包锁不是用来确保存储库是线程安全的。包锁仅用于防止删除和协调锁文件。但是，我们提供线程安全以保持简单。
    packageLock.Lock()
    // 延迟释放包锁
    defer packageLock.Unlock()
    # 如果资源已关闭，则返回空和错误信息
    if r.closed {
        return nil, errors.New("cannot access config, repo not open")
    }
    # 否则返回资源的配置信息和空错误
    return r.config, nil
// 获取用户资源覆盖配置，如果存储库未打开，则返回错误
func (r *FSRepo) UserResourceOverrides() (rcmgr.PartialLimitConfig, error) {
    // 不需要持有包锁，因为存储库处于打开状态。包锁不是用来确保存储库是线程安全的。包锁只是用来防止删除和协调锁文件。但是，我们提供线程安全以保持简单。
    packageLock.Lock()
    defer packageLock.Unlock()

    // 如果存储库已关闭，则返回错误
    if r.closed {
        return rcmgr.PartialLimitConfig{}, errors.New("cannot access config, repo not open")
    }
    // 返回用户资源覆盖配置
    return r.userResourceOverrides, nil
}

// 返回文件管理器
func (r *FSRepo) FileManager() *filestore.FileManager {
    return r.filemgr
}

// 备份配置文件
func (r *FSRepo) BackupConfig(prefix string) (string, error) {
    // 创建临时文件
    temp, err := os.CreateTemp(r.path, "config-"+prefix)
    if err != nil {
        return "", err
    }
    defer temp.Close()

    // 打开原始配置文件
    orig, err := os.OpenFile(r.configFilePath, os.O_RDONLY, 0o600)
    if err != nil {
        return "", err
    }
    defer orig.Close()

    // 将原始配置文件内容拷贝到临时文件
    _, err = io.Copy(temp, orig)
    if err != nil {
        return "", err
    }

    // 返回原始配置文件的名称
    return orig.Name(), nil
}

// 设置存储库的配置
// 用户在调用此方法后不得修改配置对象
// 存在一个固有的矛盾，即将非用户生成的 Go config.Config 结构存储为用户生成的 JSON 嵌套映射。这体现在字段的 `omitempty` 属性上，这些字段未被用户定义，但 Go 仍然需要将它们初始化为默认值（这不会反映在存储库的配置文件中，详见 https://github.com/ipfs/kubo/issues/8088 了解更多详情）。
// 通常情况下，我们应该使用 JSON 嵌套映射作为参数调用此 API（`map[string]interface{}`）。许多对此函数的调用被迫从可用的 JSON 映射中合成 config.Config 结构，以满足这一点（导致不兼容性，如上面提到的 `omitempty` 问题）。
// SetConfig方法用于更新配置，采用了JSON映射的变体
func (r *FSRepo) SetConfig(updated *config.Config) error {
    // packageLock用于提供线程安全性
    packageLock.Lock()
    defer packageLock.Unlock()

    // 为了避免覆盖用户提供的键，必须从磁盘读取配置作为映射，将更新的结构值写入映射，然后将映射写入磁盘
    var mapconf map[string]interface{}
    if err := serialize.ReadConfigFile(r.configFilePath, &mapconf); err != nil {
        return err
    }
    m, err := config.ToMap(updated)
    if err != nil {
        return err
    }
    mergedMap := common.MapMergeDeep(mapconf, m)
    if err := serialize.WriteConfigFile(r.configFilePath, mergedMap); err != nil {
        return err
    }
    // 不要使用`*r.config = ...`，这将修改`r.Config`返回的*shared*配置
    r.config = updated
    return nil
}

// GetConfigKey方法用于检索特定键的值
func (r *FSRepo) GetConfigKey(key string) (interface{}, error) {
    packageLock.Lock()
    defer packageLock.Unlock()

    if r.closed {
        return nil, errors.New("repo is closed")
    }

    var cfg map[string]interface{}
    if err := serialize.ReadConfigFile(r.configFilePath, &cfg); err != nil {
        return nil, err
    }
    return common.MapGetKV(cfg, key)
}

// SetConfigKey方法用于写入特定键的值
func (r *FSRepo) SetConfigKey(key string, value interface{}) error {
    packageLock.Lock()
    defer packageLock.Unlock()

    if r.closed {
        return errors.New("repo is closed")
    }

    // 加载到映射中，以防止将任何额外的默认值写入配置文件
    var mapconf map[string]interface{}
    if err := serialize.ReadConfigFile(r.configFilePath, &mapconf); err != nil {
        return err
    }

    // 加载私钥以防止其被覆盖
    // NOTE: 这是一个临时措施，用于保护该字段，直到我们将密钥移出配置文件。
    // 从配置映射中获取特定键的值
    pkval, err := common.MapGetKV(mapconf, config.PrivKeySelector)
    // 如果出现错误，则返回错误
    if err != nil {
        return err
    }

    // 在映射中设置键值对
    if err := common.MapSetKV(mapconf, key, value); err != nil {
        return err
    }

    // 替换私钥，以防它被覆盖
    if err := common.MapSetKV(mapconf, config.PrivKeySelector, pkval); err != nil {
        return err
    }

    // 这一步同时用于验证映射与结构体之间的匹配性
    // 将映射转换为配置对象
    conf, err := config.FromMap(mapconf)
    // 如果出现错误，则返回错误
    if err != nil {
        return err
    }
    // 将配置对象赋值给 r.config
    r.config = conf

    // 将映射序列化为配置文件并写入
    if err := serialize.WriteConfigFile(r.configFilePath, mapconf); err != nil {
        return err
    }

    // 返回空值
    return nil
// Datastore返回一个属于repo的数据存储。如果FSRepo已关闭，则返回值是未定义的。
func (r *FSRepo) Datastore() repo.Datastore {
    // 锁定packageLock以确保线程安全
    packageLock.Lock()
    // 将数据存储赋值给变量d
    d := r.ds
    // 解锁packageLock
    packageLock.Unlock()
    // 返回数据存储
    return d
}

// GetStorageUsage计算存储库占用的存储空间（以字节为单位）。
func (r *FSRepo) GetStorageUsage(ctx context.Context) (uint64, error) {
    // 调用ds.DiskUsage方法计算存储使用量
    return ds.DiskUsage(ctx, r.Datastore())
}

func (r *FSRepo) SwarmKey() ([]byte, error) {
    // 清理repo路径
    repoPath := filepath.Clean(r.path)
    // 拼接得到swarmKey文件的路径
    spath := filepath.Join(repoPath, swarmKeyFile)

    // 打开文件
    f, err := os.Open(spath)
    // 如果出现错误
    if err != nil {
        // 如果文件不存在，则将err置为nil
        if os.IsNotExist(err) {
            err = nil
        }
        // 返回nil和错误
        return nil, err
    }
    // 延迟关闭文件
    defer f.Close()

    // 读取文件内容并返回
    return io.ReadAll(f)
}

// 声明FSRepo实现了io.Closer和repo.Repo接口
var (
    _ io.Closer = &FSRepo{}
    _ repo.Repo = &FSRepo{}
)

// 如果提供的路径已初始化，则返回true。
func IsInitialized(path string) bool {
    // 锁定packageLock以确保线程安全
    packageLock.Lock()
    // 延迟解锁packageLock
    defer packageLock.Unlock()

    // 调用isInitializedUnsynced方法检查repo是否已初始化
    return isInitializedUnsynced(path)
}

// 以下是私有方法。注意：调用者必须持有packageLock。

// isInitializedUnsynced报告repo是否已初始化。调用者必须持有packageLock。
func isInitializedUnsynced(repoPath string) bool {
    // 调用configIsInitialized方法检查repo是否已初始化
    return configIsInitialized(repoPath)
}
```