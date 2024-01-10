# `kubo\repo\repo.go`

```
package repo

import (
    "context" // 导入上下文包
    "errors" // 导入错误处理包
    "io" // 导入输入输出包
    "net" // 导入网络包

    filestore "github.com/ipfs/boxo/filestore" // 导入文件存储包
    keystore "github.com/ipfs/boxo/keystore" // 导入密钥存储包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager" // 导入资源管理包

    ds "github.com/ipfs/go-datastore" // 导入数据存储包
    config "github.com/ipfs/kubo/config" // 导入配置包
    ma "github.com/multiformats/go-multiaddr" // 导入多地址包
)

var ErrApiNotRunning = errors.New("api not running") //nolint // 定义一个错误变量

// Repo represents all persistent data of a given ipfs node.
type Repo interface {
    // Config returns the ipfs configuration file from the repo. Changes made
    // to the returned config are not automatically persisted.
    Config() (*config.Config, error) // 返回存储库中的IPFS配置文件

    // UserResourceOverrides returns optional user resource overrides for the
    // libp2p resource manager.
    UserResourceOverrides() (rcmgr.PartialLimitConfig, error) // 返回用户资源覆盖项

    // BackupConfig creates a backup of the current configuration file using
    // the given prefix for naming.
    BackupConfig(prefix string) (string, error) // 创建当前配置文件的备份

    // SetConfig persists the given configuration struct to storage.
    SetConfig(*config.Config) error // 将给定的配置结构持久化到存储中

    // SetConfigKey sets the given key-value pair within the config and persists it to storage.
    SetConfigKey(key string, value interface{}) error // 设置给定键值对到配置中并将其持久化到存储中

    // GetConfigKey reads the value for the given key from the configuration in storage.
    GetConfigKey(key string) (interface{}, error) // 从存储中读取给定键的值

    // Datastore returns a reference to the configured data storage backend.
    Datastore() Datastore // 返回配置的数据存储后端的引用

    // GetStorageUsage returns the number of bytes stored.
    GetStorageUsage(context.Context) (uint64, error) // 返回存储的字节数

    // Keystore returns a reference to the key management interface.
    Keystore() keystore.Keystore // 返回密钥管理接口的引用

    // FileManager returns a reference to the filestore file manager.
    FileManager() *filestore.FileManager // 返回文件存储文件管理器的引用

    // SetAPIAddr sets the API address in the repo.
    SetAPIAddr(addr ma.Multiaddr) error // 设置存储库中的API地址

    // SetGatewayAddr sets the Gateway address in the repo.
    # 设置网关地址，接受一个 net.Addr 类型的参数，返回一个错误
    SetGatewayAddr(addr net.Addr) error
    
    # 返回私有网络功能配置的共享对称密钥
    SwarmKey() ([]byte, error)
    
    # 实现了关闭方法的接口
    io.Closer
// Datastore 是一个接口，FSRepo 需要一个符合该接口的数据存储来进行操作
type Datastore interface {
    ds.Batching // 必须是线程安全的
}
```