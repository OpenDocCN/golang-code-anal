# `kubo\core\corerepo\gc.go`

```
package corerepo

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于处理上下文
    "errors"  // 导入 errors 包，用于处理错误
    "time"  // 导入 time 包，用于处理时间

    "github.com/ipfs/kubo/core"  // 导入 core 包
    "github.com/ipfs/kubo/gc"  // 导入 gc 包
    "github.com/ipfs/kubo/repo"  // 导入 repo 包

    "github.com/dustin/go-humanize"  // 导入 go-humanize 包，用于格式化数据大小
    "github.com/ipfs/boxo/mfs"  // 导入 mfs 包
    "github.com/ipfs/go-cid"  // 导入 cid 包
    logging "github.com/ipfs/go-log"  // 导入 logging 包，用于日志记录
)

var log = logging.Logger("corerepo")  // 定义名为 log 的日志记录器

var ErrMaxStorageExceeded = errors.New("maximum storage limit exceeded. Try to unpin some files")  // 定义名为 ErrMaxStorageExceeded 的错误变量

type GC struct {
    Node       *core.IpfsNode  // 定义名为 Node 的指向 core.IpfsNode 结构体的指针
    Repo       repo.Repo  // 定义名为 Repo 的 repo.Repo 类型变量
    StorageMax uint64  // 定义名为 StorageMax 的无符号整数类型变量
    StorageGC  uint64  // 定义名为 StorageGC 的无符号整数类型变量
    SlackGB    uint64  // 定义名为 SlackGB 的无符号整数类型变量
    Storage    uint64  // 定义名为 Storage 的无符号整数类型变量
}

func NewGC(n *core.IpfsNode) (*GC, error) {
    r := n.Repo  // 将 n 的 Repo 字段赋值给 r
    cfg, err := r.Config()  // 调用 r 的 Config 方法，返回配置和错误
    if err != nil {  // 如果错误不为空
        return nil, err  // 返回空和错误
    }

    // check if cfg has these fields initialized
    // TODO: there should be a general check for all of the cfg fields
    // maybe distinguish between user config file and default struct?
    if cfg.Datastore.StorageMax == "" {  // 如果 cfg 的 Datastore.StorageMax 字段为空
        if err := r.SetConfigKey("Datastore.StorageMax", "10GB"); err != nil {  // 调用 r 的 SetConfigKey 方法，设置 Datastore.StorageMax 字段为 "10GB"，返回错误
            return nil, err  // 返回空和错误
        }
        cfg.Datastore.StorageMax = "10GB"  // 将 cfg 的 Datastore.StorageMax 字段设置为 "10GB"
    }
    if cfg.Datastore.StorageGCWatermark == 0 {  // 如果 cfg 的 Datastore.StorageGCWatermark 字段为 0
        if err := r.SetConfigKey("Datastore.StorageGCWatermark", 90); err != nil {  // 调用 r 的 SetConfigKey 方法，设置 Datastore.StorageGCWatermark 字段为 90，返回错误
            return nil, err  // 返回空和错误
        }
        cfg.Datastore.StorageGCWatermark = 90  // 将 cfg 的 Datastore.StorageGCWatermark 字段设置为 90
    }

    storageMax, err := humanize.ParseBytes(cfg.Datastore.StorageMax)  // 调用 humanize 的 ParseBytes 方法，解析 cfg 的 Datastore.StorageMax 字段，返回存储大小和错误
    if err != nil {  // 如果错误不为空
        return nil, err  // 返回空和错误
    }
    storageGC := storageMax * uint64(cfg.Datastore.StorageGCWatermark) / 100  // 计算存储 GC 大小
    // calculate the slack space between StorageMax and StorageGCWatermark
    // used to limit GC duration
    slackGB := (storageMax - storageGC) / 10e9  // 计算 StorageMax 和 StorageGCWatermark 之间的空闲空间，用于限制 GC 的持续时间
    if slackGB < 1 {  // 如果 slackGB 小于 1
        slackGB = 1  // 将 slackGB 设置为 1
    }

    return &GC{  // 返回 GC 结构体指针
        Node:       n,  // 设置 Node 字段为 n
        Repo:       r,  // 设置 Repo 字段为 r
        StorageMax: storageMax,  // 设置 StorageMax 字段为 storageMax
        StorageGC:  storageGC,  // 设置 StorageGC 字段为 storageGC
        SlackGB:    slackGB,  // 设置 SlackGB 字段为 slackGB
    }, nil  // 返回 GC 结构体指针和空
}

func BestEffortRoots(filesRoot *mfs.Root) ([]cid.Cid, error) {
    // 从文件根节点获取目录节点
    rootDag, err := filesRoot.GetDirectory().GetNode()
    // 如果出现错误，返回空和错误信息
    if err != nil {
        return nil, err
    }

    // 返回根节点的 CID
    return []cid.Cid{rootDag.Cid()}, nil
}
// 垃圾回收函数，接收一个 IPFS 节点和上下文作为参数，返回错误
func GarbageCollect(n *core.IpfsNode, ctx context.Context) error {
    // 获取最佳努力获取根节点
    roots, err := BestEffortRoots(n.FilesRoot)
    if err != nil {
        return err
    }
    // 调用垃圾回收模块的 GC 函数，传入上下文、块存储、数据存储、固定和根节点
    rmed := gc.GC(ctx, n.Blockstore, n.Repo.Datastore(), n.Pinning, roots)

    // 调用 CollectResult 函数，传入上下文、垃圾回收结果和回调函数
    return CollectResult(ctx, rmed, nil)
}

// CollectResult 函数用于收集垃圾回收运行的输出，并对每个删除的对象调用给定的回调函数。
// 它还将所有错误收集到一个 MultiError 中，在完成垃圾回收后返回。
func CollectResult(ctx context.Context, gcOut <-chan gc.Result, cb func(cid.Cid)) error {
    var errors []error
loop:
    for {
        select {
        case res, ok := <-gcOut:
            if !ok {
                break loop
            }
            if res.Error != nil {
                errors = append(errors, res.Error)
            } else if res.KeyRemoved.Defined() && cb != nil {
                cb(res.KeyRemoved)
            }
        case <-ctx.Done():
            errors = append(errors, ctx.Err())
            break loop
        }
    }

    switch len(errors) {
    case 0:
        return nil
    case 1:
        return errors[0]
    default:
        return NewMultiError(errors...)
    }
}

// NewMultiError 从给定的错误切片中创建一个新的 MultiError 对象。
func NewMultiError(errs ...error) *MultiError {
    return &MultiError{errs[:len(errs)-1], errs[len(errs)-1]}
}

// MultiError 包含多个错误的结果。
type MultiError struct {
    Errors  []error
    Summary error
}

// Error 方法用于返回 MultiError 对象的错误字符串表示形式。
func (e *MultiError) Error() string {
    var buf bytes.Buffer
    for _, err := range e.Errors {
        buf.WriteString(err.Error())
        buf.WriteString("; ")
    }
    buf.WriteString(e.Summary.Error())
    return buf.String()
}

// GarbageCollectAsync 函数用于异步执行垃圾回收，接收一个 IPFS 节点和上下文作为参数，返回一个垃圾回收结果的通道。
func GarbageCollectAsync(n *core.IpfsNode, ctx context.Context) <-chan gc.Result {
    // 获取最佳努力获取根节点
    roots, err := BestEffortRoots(n.FilesRoot)
    # 如果错误不为空
    if err != nil:
        # 创建一个通道用于传输gc.Result类型的数据
        out := make(chan gc.Result)
        # 向通道中发送包含错误信息的gc.Result类型数据
        out <- gc.Result{Error: err}
        # 关闭通道
        close(out)
        # 返回通道
        return out

    # 调用GC函数，传入上下文、块存储、数据存储、固定、根节点参数，并返回结果
    return gc.GC(ctx, n.Blockstore, n.Repo.Datastore(), n.Pinning, roots)
// 定期执行垃圾回收，根据配置的时间间隔执行垃圾回收操作
func PeriodicGC(ctx context.Context, node *core.IpfsNode) error {
    // 获取节点的配置信息
    cfg, err := node.Repo.Config()
    if err != nil {
        return err
    }

    // 如果未配置垃圾回收时间间隔，则设置默认值为1小时
    if cfg.Datastore.GCPeriod == "" {
        cfg.Datastore.GCPeriod = "1h"
    }

    // 解析配置的时间间隔
    period, err := time.ParseDuration(cfg.Datastore.GCPeriod)
    if err != nil {
        return err
    }
    // 如果时间间隔为0，则表示垃圾回收被禁用
    if int64(period) == 0 {
        return nil
    }

    // 创建垃圾回收对象
    gc, err := NewGC(node)
    if err != nil {
        return err
    }

    // 循环执行垃圾回收操作
    for {
        select {
        case <-ctx.Done():
            return nil
        case <-time.After(period):
            // 调用私有方法maybeGC，执行垃圾回收操作
            if err := gc.maybeGC(ctx, 0); err != nil {
                log.Error(err)
            }
        }
    }
}

// 根据偏移量执行条件性垃圾回收
func ConditionalGC(ctx context.Context, node *core.IpfsNode, offset uint64) error {
    // 创建垃圾回收对象
    gc, err := NewGC(node)
    if err != nil {
        return err
    }
    // 调用maybeGC方法执行垃圾回收操作
    return gc.maybeGC(ctx, offset)
}

// 执行条件性垃圾回收操作
func (gc *GC) maybeGC(ctx context.Context, offset uint64) error {
    // 获取存储使用情况
    storage, err := gc.Repo.GetStorageUsage(ctx)
    if err != nil {
        return err
    }

    // 如果存储使用量超过阈值，则执行垃圾回收操作
    if storage+offset > gc.StorageGC {
        if storage+offset > gc.StorageMax {
            log.Warnf("pre-GC: %s", ErrMaxStorageExceeded)
        }

        // 执行垃圾回收操作
        log.Info("Watermark exceeded. Starting repo GC...")

        if err := GarbageCollect(gc.Node, ctx); err != nil {
            return err
        }
        log.Infof("Repo GC done. See `ipfs repo stat` to see how much space got freed.\n")
    }
    return nil
}
```