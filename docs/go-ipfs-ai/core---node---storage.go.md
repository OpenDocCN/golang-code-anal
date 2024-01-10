# `kubo\core\node\storage.go`

```
package node

import (
    blockstore "github.com/ipfs/boxo/blockstore"  // 导入 blockstore 包
    "github.com/ipfs/go-datastore"  // 导入 go-datastore 包
    config "github.com/ipfs/kubo/config"  // 导入 config 包
    "go.uber.org/fx"  // 导入 fx 包

    "github.com/ipfs/boxo/filestore"  // 导入 filestore 包
    "github.com/ipfs/kubo/core/node/helpers"  // 导入 helpers 包
    "github.com/ipfs/kubo/repo"  // 导入 repo 包
    "github.com/ipfs/kubo/thirdparty/verifbs"  // 导入 verifbs 包
)

// RepoConfig loads configuration from the repo
func RepoConfig(repo repo.Repo) (*config.Config, error) {
    cfg, err := repo.Config()  // 从 repo 中加载配置
    return cfg, err  // 返回配置和错误
}

// Datastore provides the datastore
func Datastore(repo repo.Repo) datastore.Datastore {
    return repo.Datastore()  // 返回数据存储
}

// BaseBlocks is the lower level blockstore without GC or Filestore layers
type BaseBlocks blockstore.Blockstore  // 定义 BaseBlocks 类型为 blockstore.Blockstore

// BaseBlockstoreCtor creates cached blockstore backed by the provided datastore
func BaseBlockstoreCtor(cacheOpts blockstore.CacheOpts, nilRepo bool, hashOnRead bool) func(mctx helpers.MetricsCtx, repo repo.Repo, lc fx.Lifecycle) (bs BaseBlocks, err error) {
    return func(mctx helpers.MetricsCtx, repo repo.Repo, lc fx.Lifecycle) (bs BaseBlocks, err error) {
        // hash security
        bs = blockstore.NewBlockstore(repo.Datastore())  // 创建基于提供的数据存储的缓存块存储
        bs = &verifbs.VerifBS{Blockstore: bs}  // 使用 verifbs.VerifBS 对象包装 bs

        if !nilRepo {
            bs, err = blockstore.CachedBlockstore(helpers.LifecycleCtx(mctx, lc), bs, cacheOpts)  // 如果不是空的 repo，则创建缓存块存储
            if err != nil {
                return nil, err  // 如果出错，返回 nil 和错误
            }
        }

        bs = blockstore.NewIdStore(bs)  // 创建基于 bs 的 ID 存储

        if hashOnRead { // 如果需要在读取时进行哈希
            bs.HashOnRead(true)  // 设置在读取时进行哈希
        }

        return  // 返回 bs 和错误
    }
}

// GcBlockstoreCtor wraps the base blockstore with GC and Filestore layers
func GcBlockstoreCtor(bb BaseBlocks) (gclocker blockstore.GCLocker, gcbs blockstore.GCBlockstore, bs blockstore.Blockstore) {
    gclocker = blockstore.NewGCLocker()  // 创建 GC 锁
    gcbs = blockstore.NewGCBlockstore(bb, gclocker)  // 创建带有 GC 和 Filestore 层的基本块存储

    bs = gcbs  // 将 bs 设置为 gcbs
    return  // 返回 gclocker, gcbs 和 bs
}
// GcBlockstoreCtor wraps GcBlockstore and adds Filestore support
// 定义一个函数，用于创建包含文件存储支持的 GcBlockstore
func FilestoreBlockstoreCtor(repo repo.Repo, bb BaseBlocks) (gclocker blockstore.GCLocker, gcbs blockstore.GCBlockstore, bs blockstore.Blockstore, fstore *filestore.Filestore) {
    // 创建一个新的 GCLocker 对象
    gclocker = blockstore.NewGCLocker()

    // 创建一个新的 Filestore 对象，使用基础块存储和存储库的文件管理器
    fstore = filestore.NewFilestore(bb, repo.FileManager())
    // 创建一个新的 GCBlockstore 对象，使用 Filestore 和 GCLocker
    gcbs = blockstore.NewGCBlockstore(fstore, gclocker)
    // 创建一个 VerifBSGC 对象，包含 GCBlockstore
    gcbs = &verifbs.VerifBSGC{GCBlockstore: gcbs}

    // 将 GCBlockstore 赋值给 Blockstore
    bs = gcbs
    // 返回结果
    return
}
```