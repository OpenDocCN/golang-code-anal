# `kubo\blocks\blockstoreutil\remove.go`

```go
// Package blockstoreutil provides utility functions for Blockstores.
package blockstoreutil

import (
    "context"  // 导入上下文包，用于控制函数调用的上下文
    "errors"   // 导入错误包，用于处理错误
    "fmt"      // 导入格式化包，用于格式化输出

    bs "github.com/ipfs/boxo/blockstore"  // 导入别名为 bs 的 blockstore 包
    pin "github.com/ipfs/boxo/pinning/pinner"  // 导入别名为 pin 的 pinner 包
    cid "github.com/ipfs/go-cid"  // 导入 cid 包
    format "github.com/ipfs/go-ipld-format"  // 导入格式化包
)

// RemovedBlock is used to represent the result of removing a block.
// If a block was removed successfully, then the Error will be empty.
// If a block could not be removed, then Error will contain the
// reason the block could not be removed.  If the removal was aborted
// due to a fatal error, Hash will be empty, Error will contain the
// reason, and no more results will be sent.
type RemovedBlock struct {
    Hash  string  // 用于表示移除块的哈希值
    Error error   // 用于表示移除块时的错误信息
}

// RmBlocksOpts is used to wrap options for RmBlocks().
type RmBlocksOpts struct {
    Prefix string  // 用于包装 RmBlocks() 的选项前缀
    Quiet  bool    // 用于包装 RmBlocks() 的安静选项
    Force  bool    // 用于包装 RmBlocks() 的强制选项
}

// RmBlocks removes the blocks provided in the cids slice.
// It returns a channel where objects of type RemovedBlock are placed, when
// not using the Quiet option. Block removal is asynchronous and will
// skip any pinned blocks.
func RmBlocks(ctx context.Context, blocks bs.GCBlockstore, pins pin.Pinner, cids []cid.Cid, opts RmBlocksOpts) (<-chan interface{}, error) {
    // make the channel large enough to hold any result to avoid
    // blocking while holding the GCLock
    out := make(chan interface{}, len(cids))  // 创建一个通道，用于存放移除块的结果，避免在持有 GCLock 时阻塞
    # 创建一个匿名函数并立即执行
    go func() {
        # 延迟关闭输出通道，确保在函数退出时关闭通道
        defer close(out)

        # 获取全局锁，并在函数退出时释放锁
        unlocker := blocks.GCLock(ctx)
        defer unlocker.Unlock(ctx)

        # 过滤出仍然存在的固定块
        stillOkay := FilterPinned(ctx, pins, out, cids)

        # 遍历仍然存在的固定块
        for _, c := range stillOkay {
            # 为了向后兼容性而保留。将来可能会在某个时候删除这个。
            # 检查块是否存在
            has, err := blocks.Has(ctx, c)
            if err != nil {
                # 如果出现错误，向输出通道发送已移除的块信息
                out <- &RemovedBlock{Hash: c.String(), Error: err}
                continue
            }
            # 如果块不存在且不是强制删除，则向输出通道发送已移除的块信息
            if !has && !opts.Force {
                out <- &RemovedBlock{Hash: c.String(), Error: format.ErrNotFound{Cid: c}}
                continue
            }

            # 删除块
            err = blocks.DeleteBlock(ctx, c)
            if err != nil {
                # 如果删除出现错误，则向输出通道发送已移除的块信息
                out <- &RemovedBlock{Hash: c.String(), Error: err}
            } else if !opts.Quiet {
                # 如果删除成功且不是安静模式，则向输出通道发送已移除的块信息
                out <- &RemovedBlock{Hash: c.String()}
            }
        }
    }()
    # 返回输出通道和空错误
    return out, nil
// FilterPinned函数接受一个Cids切片，并返回一个移除了已固定的Cids的切片。
// 如果一个Cid被固定，它将在给定的out通道中放置RemovedBlock对象，其中包含指示该Cid已固定的错误。
// 此函数用于在RmBlocks中过滤掉不应该被移除的任何块（因为它们被固定）。
func FilterPinned(ctx context.Context, pins pin.Pinner, out chan<- interface{}, cids []cid.Cid) []cid.Cid {
    // 创建一个空的Cid切片，用于存放仍然可以保留的Cids
    stillOkay := make([]cid.Cid, 0, len(cids))
    // 检查Cids是否被固定
    res, err := pins.CheckIfPinned(ctx, cids...)
    // 如果检查出现错误，将错误信息放入out通道，并返回空切片
    if err != nil {
        out <- &RemovedBlock{Error: fmt.Errorf("pin check failed: %w", err)}
        return nil
    }
    // 遍历检查结果
    for _, r := range res {
        // 如果Cid未被固定，将其添加到stillOkay切片中
        if !r.Pinned() {
            stillOkay = append(stillOkay, r.Key)
        } else {
            // 如果Cid被固定，将其信息放入out通道
            out <- &RemovedBlock{
                Hash:  r.Key.String(),
                Error: errors.New(r.String()),
            }
        }
    }
    // 返回仍然可以保留的Cids切片
    return stillOkay
}
```