# `kubo\core\corerepo\stat.go`

```
package corerepo

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "math" // 导入 math 包，用于数学计算

    context "context" // 导入 context 包，用于处理上下文

    "github.com/ipfs/kubo/core" // 导入 core 包
    fsrepo "github.com/ipfs/kubo/repo/fsrepo" // 导入 fsrepo 包

    humanize "github.com/dustin/go-humanize" // 导入 humanize 包，用于格式化文件大小
)

// SizeStat wraps information about the repository size and its limit.
type SizeStat struct {
    RepoSize   uint64 // size in bytes，存储库的大小（以字节为单位）
    StorageMax uint64 // size in bytes，存储库的最大限制（以字节为单位）
}

// Stat wraps information about the objects stored on disk.
type Stat struct {
    SizeStat
    NumObjects uint64 // number of objects stored on disk，存储在磁盘上的对象数量
    RepoPath   string // path to the repository，存储库的路径
    Version    string // version of the repository，存储库的版本
}

// NoLimit represents the value for unlimited storage
const NoLimit uint64 = math.MaxUint64 // NoLimit 表示无限存储的值

// RepoStat returns a *Stat object with all the fields set.
func RepoStat(ctx context.Context, n *core.IpfsNode) (Stat, error) {
    sizeStat, err := RepoSize(ctx, n) // 获取存储库大小信息
    if err != nil {
        return Stat{}, err
    }

    allKeys, err := n.Blockstore.AllKeysChan(ctx) // 获取所有密钥的通道
    if err != nil {
        return Stat{}, err
    }

    count := uint64(0) // 初始化对象数量为 0
    for range allKeys { // 遍历所有密钥
        count++ // 计算对象数量
    }

    path, err := fsrepo.BestKnownPath() // 获取最佳已知路径
    if err != nil {
        return Stat{}, err
    }

    return Stat{ // 返回 Stat 对象
        SizeStat: SizeStat{ // 设置 SizeStat 字段
            RepoSize:   sizeStat.RepoSize, // 存储库大小
            StorageMax: sizeStat.StorageMax, // 存储库最大限制
        },
        NumObjects: count, // 对象数量
        RepoPath:   path, // 存储库路径
        Version:    fmt.Sprintf("fs-repo@%d", fsrepo.RepoVersion), // 存储库版本
    }, nil
}

// RepoSize returns a *Stat object with the RepoSize and StorageMax fields set.
func RepoSize(ctx context.Context, n *core.IpfsNode) (SizeStat, error) {
    r := n.Repo // 获取存储库

    cfg, err := r.Config() // 获取存储库配置
    if err != nil {
        return SizeStat{}, err
    }

    usage, err := r.GetStorageUsage(ctx) // 获取存储使用情况
    if err != nil {
        return SizeStat{}, err
    }

    storageMax := NoLimit // 初始化存储最大限制为无限
    if cfg.Datastore.StorageMax != "" { // 如果存储最大限制不为空
        storageMax, err = humanize.ParseBytes(cfg.Datastore.StorageMax) // 解析存储最大限制
        if err != nil {
            return SizeStat{}, err
        }
    }
    # 返回一个 SizeStat 结构体，包含 RepoSize 和 StorageMax 两个字段
    return SizeStat{
        RepoSize:   usage,   # 使用参数 usage 初始化 RepoSize 字段
        StorageMax: storageMax,   # 使用参数 storageMax 初始化 StorageMax 字段
    }, nil   # 返回值为一个空值
# 闭合前面的函数定义
```