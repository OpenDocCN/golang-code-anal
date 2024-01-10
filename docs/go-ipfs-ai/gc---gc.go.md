# `kubo\gc\gc.go`

```
// Package gc provides garbage collection for go-ipfs.
package gc

import (
    "context" // 导入上下文包，用于控制程序的执行流程
    "errors" // 导入错误处理包，用于处理错误信息
    "fmt" // 导入格式化包，用于格式化输出
    "strings" // 导入字符串处理包，用于字符串操作

    bserv "github.com/ipfs/boxo/blockservice" // 导入块服务包
    bstore "github.com/ipfs/boxo/blockstore" // 导入块存储包
    offline "github.com/ipfs/boxo/exchange/offline" // 导入离线交换包
    dag "github.com/ipfs/boxo/ipld/merkledag" // 导入默克尔有向无环图包
    pin "github.com/ipfs/boxo/pinning/pinner" // 导入固定包
    "github.com/ipfs/boxo/verifcid" // 导入验证 CID 包
    cid "github.com/ipfs/go-cid" // 导入 CID 包
    dstore "github.com/ipfs/go-datastore" // 导入数据存储包
    ipld "github.com/ipfs/go-ipld-format" // 导入 IPLD 格式包
    logging "github.com/ipfs/go-log" // 导入日志包
)

var log = logging.Logger("gc") // 定义日志记录器

// Result represents an incremental output from a garbage collection
// run.  It contains either an error, or the cid of a removed object.
type Result struct {
    KeyRemoved cid.Cid // 表示被移除对象的 CID
    Error      error // 表示可能出现的错误信息
}

// converts a set of CIDs with different codecs to a set of CIDs with the raw codec.
func toRawCids(set *cid.Set) (*cid.Set, error) {
    newSet := cid.NewSet() // 创建一个新的 CID 集合
    err := set.ForEach(func(c cid.Cid) error {
        newSet.Add(cid.NewCidV1(cid.Raw, c.Hash())) // 将不同编解码的 CID 转换为原始编解码的 CID
        return nil
    })
    return newSet, err
}

// GC performs a mark and sweep garbage collection of the blocks in the blockstore
// first, it creates a 'marked' set and adds to it the following:
// - all recursively pinned blocks, plus all of their descendants (recursively)
// - bestEffortRoots, plus all of its descendants (recursively)
// - all directly pinned blocks
// - all blocks utilized internally by the pinner
//
// The routine then iterates over every block in the blockstore and
// deletes any block that is not found in the marked set.
func GC(ctx context.Context, bs bstore.GCBlockstore, dstor dstore.Datastore, pn pin.Pinner, bestEffortRoots []cid.Cid) <-chan Result {
    ctx, cancel := context.WithCancel(ctx) // 创建一个带有取消功能的上下文

    unlocker := bs.GCLock(ctx) // 获取垃圾回收锁

    bsrv := bserv.New(bs, offline.Exchange(bs)) // 创建块服务
    ds := dag.NewDAGService(bsrv) // 创建 DAG 服务

    output := make(chan Result, 128) // 创建一个带缓冲的结果通道
    // 延迟执行取消操作
    defer cancel()
    // 延迟关闭输出通道
    defer close(output)
    // 延迟释放锁
    defer unlocker.Unlock(ctx)

    // 调用 ColoredSet 函数，获取结果集和可能的错误
    gcs, err := ColoredSet(ctx, pn, ds, bestEffortRoots, output)
    // 如果有错误，将错误信息发送到输出通道，或者在上下文取消时退出
    if err != nil {
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
        }
        return
    }

    // 块存储报告原始块，需要从 CID 中移除编解码器
    gcs, err = toRawCids(gcs)
    // 如果有错误，将错误信息发送到输出通道，或者在上下文取消时退出
    if err != nil {
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
        }
        return
    }

    // 获取所有密钥的通道
    keychan, err := bs.AllKeysChan(ctx)
    // 如果有错误，将错误信息发送到输出通道，或者在上下文取消时退出
    if err != nil {
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
        }
        return
    }

    // 初始化错误标志和已移除的计数
    errors := false
    var removed uint64
    # 循环直到出现错误
    loop:
        # 当上下文没有错误时执行，避免 select 无法察觉到我们已经“完成”。
        for ctx.Err() == nil {
            # 选择一个 case 执行
            select {
            # 从 keychan 通道接收键值
            case k, ok := <-keychan:
                # 如果通道关闭，则跳出循环
                if !ok {
                    break loop
                }
                # 注意：假设 keychan 返回的所有 CID 都是原始的 CIDv1 CID。
                # 这意味着我们可以在任何地方保留块，只要我们想要它（CIDv1、CIDv0、原始、其他...）。
                if !gcs.Has(k) {
                    # 删除块并增加已删除计数
                    err := bs.DeleteBlock(ctx, k)
                    removed++
                    # 如果出现错误
                    if err != nil {
                        errors = true
                        # 选择一个 case 执行
                        select {
                        # 发送无法删除块的错误到 output 通道
                        case output <- Result{Error: &CannotDeleteBlockError{k, err}}:
                        # 当上下文被取消时跳出循环
                        case <-ctx.Done():
                            break loop
                        }
                        # 继续循环，因为错误是非致命的
                        continue loop
                    }
                    # 选择一个 case 执行
                    select {
                    # 发送已删除的键到 output 通道
                    case output <- Result{KeyRemoved: k}:
                    # 当上下文被取消时跳出循环
                    case <-ctx.Done():
                        break loop
                    }
                }
            # 当上下文被取消时跳出循环
            case <-ctx.Done():
                break loop
            }
        }
        # 如果出现错误
        if errors {
            # 选择一个 case 执行
            select {
            # 发送无法删除某些块的错误到 output 通道
            case output <- Result{Error: ErrCannotDeleteSomeBlocks}:
            # 当上下文被取消时返回
            case <-ctx.Done():
                return
            }
        }

        # 尝试将 dstor 转换为 GCDatastore 接口
        gds, ok := dstor.(dstore.GCDatastore)
        # 如果转换失败则返回
        if !ok {
            return
        }

        # 执行垃圾回收
        err = gds.CollectGarbage(ctx)
        # 如果出现错误
        if err != nil {
            # 选择一个 case 执行
            case output <- Result{Error: err}:
            # 当上下文被取消时返回
            case <-ctx.Done():
            }
            return
        }
    }()
    # 返回 output
    return output
// Descendants函数递归查找给定根节点的所有后代，并将它们添加到给定的cid.Set中，使用提供的dag.GetLinks函数来遍历树。
func Descendants(ctx context.Context, getLinks dag.GetLinks, set *cid.Set, roots <-chan pin.StreamedPin) error {
    // 验证getLinks函数的返回值是否合法
    verifyGetLinks := func(ctx context.Context, c cid.Cid) ([]*ipld.Link, error) {
        // 验证Cid是否合法
        err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c)
        if err != nil {
            return nil, err
        }

        return getLinks(ctx, c)
    }

    // 处理CID错误并输出详细信息
    verboseCidError := func(err error) error {
        if strings.Contains(err.Error(), verifcid.ErrBelowMinimumHashLength.Error()) ||
            strings.Contains(err.Error(), verifcid.ErrPossiblyInsecureHashFunction.Error()) {
            err = fmt.Errorf("\"%s\"\nPlease run 'ipfs pin verify'"+ // nolint
                " to list insecure hashes. If you want to read them,"+
                " please downgrade your go-ipfs to 0.4.13\n", err)
            log.Error(err)
        }
        return err
    }

    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case wrapper, ok := <-roots:
            if !ok {
                return nil
            }
            if wrapper.Err != nil {
                return wrapper.Err
            }

            // 递归遍历dag并将键添加到给定的set中
            err := dag.Walk(ctx, verifyGetLinks, wrapper.Pin.Key, func(k cid.Cid) bool {
                return set.Visit(toCidV1(k))
            }, dag.Concurrent())
            if err != nil {
                err = verboseCidError(err)
                return err
            }
        }
    }
}

// 将任何CIDv0转换为CIDv1
func toCidV1(c cid.Cid) cid.Cid {
    if c.Version() == 0 {
        return cid.NewCidV1(c.Type(), c.Hash())
    }
    return c
}

// ColoredSet计算图中由给定pinner中的pin固定的节点集合
func ColoredSet(ctx context.Context, pn pin.Pinner, ng ipld.NodeGetter, bestEffortRoots []cid.Cid, output chan<- Result) (*cid.Set, error) {
    // KeySet当前在内存中实现，将来可能会改为布隆过滤器或磁盘存储以节省内存。
    errors := false
    // 创建一个空的CID集合
    gcs := cid.NewSet()
    // 定义一个函数，用于获取指定CID的链接
    getLinks := func(ctx context.Context, cid cid.Cid) ([]*ipld.Link, error) {
        // 调用ipld包中的GetLinks函数获取指定CID的链接
        links, err := ipld.GetLinks(ctx, ng, cid)
        // 如果出现错误
        if err != nil {
            // 设置错误标志为true
            errors = true
            // 使用select语句发送错误结果到output通道，或者在ctx.Done()时返回上下文错误
            select {
            case output <- Result{Error: &CannotFetchLinksError{cid, err}}:
            case <-ctx.Done():
                return nil, ctx.Err()
            }
        }
        // 返回获取的链接
        return links, nil
    }
    // 获取非递归键集合
    rkeys := pn.RecursiveKeys(ctx, false)
    // 调用Descendants函数获取节点的后代节点，并将结果存入gcs中
    err := Descendants(ctx, getLinks, gcs, rkeys)
    // 如果出现错误
    if err != nil {
        // 设置错误标志为true
        errors = true
        // 使用select语句发送错误结果到output通道，或者在ctx.Done()时返回上下文错误
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
            return nil, ctx.Err()
        }
    }

    // 定义一个函数，用于尝试获取指定CID的链接
    bestEffortGetLinks := func(ctx context.Context, cid cid.Cid) ([]*ipld.Link, error) {
        // 尝试获取指定CID的链接
        links, err := ipld.GetLinks(ctx, ng, cid)
        // 如果出现错误且不是未找到错误
        if err != nil && !ipld.IsNotFound(err) {
            // 设置错误标志为true
            errors = true
            // 使用select语句发送错误结果到output通道，或者在ctx.Done()时返回上下文错误
            select {
            case output <- Result{Error: &CannotFetchLinksError{cid, err}}:
            case <-ctx.Done():
                return nil, ctx.Err()
            }
        }
        // 返回获取的链接
        return links, nil
    }
    // 创建一个用于存储最佳尝试根节点的通道
    bestEffortRootsChan := make(chan pin.StreamedPin)
    // 启动一个goroutine，用于向bestEffortRootsChan通道发送最佳尝试根节点
    go func() {
        defer close(bestEffortRootsChan)
        for _, root := range bestEffortRoots {
            select {
            case <-ctx.Done():
                return
            case bestEffortRootsChan <- pin.StreamedPin{Pin: pin.Pinned{Key: root}}:
            }
        }
    }()
    // 调用Descendants函数获取最佳尝试根节点的后代节点，并将结果存入gcs中
    err = Descendants(ctx, bestEffortGetLinks, gcs, bestEffortRootsChan)
}
    # 如果发生错误
    if err != nil:
        # 设置错误标志为true
        errors = true
        # 选择操作：如果output通道可用，则发送Result{Error: err}；如果ctx.Done()通道可用，则返回nil和ctx.Err()
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
            return nil, ctx.Err()
        }
    
    # 获取直接键集合
    dkeys := pn.DirectKeys(ctx, false)
    # 遍历直接键集合
    for k := range dkeys:
        # 如果存在错误，则返回nil和错误
        if k.Err != nil:
            return nil, k.Err
        # 将直接键转换为CID V1格式并添加到gcs中
        gcs.Add(toCidV1(k.Pin.Key))
    
    # 获取内部PIN集合
    ikeys := pn.InternalPins(ctx, false)
    # 获取链接的后代
    err = Descendants(ctx, getLinks, gcs, ikeys)
    # 如果发生错误
    if err != nil:
        # 设置错误标志为true
        errors = true
        # 选择操作：如果output通道可用，则发送Result{Error: err}；如果ctx.Done()通道可用，则返回nil和ctx.Err()
        select {
        case output <- Result{Error: err}:
        case <-ctx.Done():
            return nil, ctx.Err()
        }
    
    # 如果存在错误
    if errors:
        # 返回错误信息
        return nil, ErrCannotFetchAllLinks
    
    # 返回gcs和nil
    return gcs, nil
// 定义一个错误变量，表示垃圾回收过程中因为找不到后代而创建标记集失败
var ErrCannotFetchAllLinks = errors.New("garbage collection aborted: could not retrieve some links")

// 定义一个错误变量，表示垃圾回收过程中删除标记的块失败
var ErrCannotDeleteSomeBlocks = errors.New("garbage collection incomplete: could not delete some blocks")

// 定义一个结构体类型，提供关于无法获取链接的详细信息，并可以作为GC输出通道中的一个结果
type CannotFetchLinksError struct {
    Key cid.Cid
    Err error
}

// 为CannotFetchLinksError类型实现错误接口，返回一个有用的错误信息
func (e *CannotFetchLinksError) Error() string {
    return fmt.Sprintf("could not retrieve links for %s: %s", e.Key, e.Err)
}

// 定义一个结构体类型，提供关于无法删除块的详细信息，并可以作为GC输出通道中的一个结果
type CannotDeleteBlockError struct {
    Key cid.Cid
    Err error
}

// 为CannotDeleteBlockError类型实现错误接口，返回一个有用的错误信息
func (e *CannotDeleteBlockError) Error() string {
    return fmt.Sprintf("could not remove %s: %s", e.Key, e.Err)
}
```