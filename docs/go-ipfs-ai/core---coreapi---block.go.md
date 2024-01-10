# `kubo\core\coreapi\block.go`

```
package coreapi

import (
    "bytes" // 导入 bytes 包，用于操作字节流
    "context" // 导入 context 包，用于处理上下文
    "errors" // 导入 errors 包，用于处理错误
    "io" // 导入 io 包，用于进行输入输出操作

    "github.com/ipfs/boxo/path" // 导入路径相关的包
    pin "github.com/ipfs/boxo/pinning/pinner" // 导入 pinning 相关的包
    blocks "github.com/ipfs/go-block-format" // 导入区块格式相关的包
    cid "github.com/ipfs/go-cid" // 导入 CID 相关的包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入核心接口相关的包
    caopts "github.com/ipfs/kubo/core/coreiface/options" // 导入选项相关的包
    "go.opentelemetry.io/otel/attribute" // 导入属性相关的包
    "go.opentelemetry.io/otel/trace" // 导入跟踪相关的包

    util "github.com/ipfs/kubo/blocks/blockstoreutil" // 导入块存储工具相关的包
    "github.com/ipfs/kubo/tracing" // 导入跟踪相关的包
)

type BlockAPI CoreAPI // 定义 BlockAPI 类型为 CoreAPI 类型

type BlockStat struct { // 定义 BlockStat 结构体
    path path.ImmutablePath // 包含路径的不可变路径
    size int // 大小
}

func (api *BlockAPI) Put(ctx context.Context, src io.Reader, opts ...caopts.BlockPutOption) (coreiface.BlockStat, error) { // 定义 Put 方法
    ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Put") // 创建跟踪 span
    defer span.End() // 延迟结束 span

    settings, err := caopts.BlockPutOptions(opts...) // 获取块放置选项
    if err != nil { // 如果出现错误
        return nil, err // 返回空和错误
    }

    data, err := io.ReadAll(src) // 读取输入流的所有数据
    if err != nil { // 如果出现错误
        return nil, err // 返回空和错误
    }

    bcid, err := settings.CidPrefix.Sum(data) // 计算数据的 CID
    if err != nil { // 如果出现错误
        return nil, err // 返回空和错误
    }

    b, err := blocks.NewBlockWithCid(data, bcid) // 创建带有 CID 的新块
    if err != nil { // 如果出现错误
        return nil, err // 返回空和错误
    }

    if settings.Pin { // 如果设置了 Pin
        defer api.blockstore.PinLock(ctx).Unlock(ctx) // 延迟解锁 PinLock
    }

    err = api.blocks.AddBlock(ctx, b) // 添加块到块存储
    if err != nil { // 如果出现错误
        return nil, err // 返回空和错误
    }

    if settings.Pin { // 如果设置了 Pin
        if err = api.pinning.PinWithMode(ctx, b.Cid(), pin.Recursive, ""); err != nil { // 使用递归模式进行 Pin
            return nil, err // 返回空和错误
        }
        if err := api.pinning.Flush(ctx); err != nil { // 刷新 Pin
            return nil, err // 返回空和错误
        }
    }

    return &BlockStat{path: path.FromCid(b.Cid()), size: len(data)}, nil // 返回块的路径和大小
}

func (api *BlockAPI) Get(ctx context.Context, p path.Path) (io.Reader, error) { // 定义 Get 方法
    ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Get", trace.WithAttributes(attribute.String("path", p.String()))) // 创建跟踪 span
    defer span.End() // 延迟结束 span
    rp, _, err := api.core().ResolvePath(ctx, p) // 解析路径
    # 如果错误不为空，则返回空和错误
    if err != nil:
        return nil, err

    # 通过 API 获取块数据，并检查是否有错误
    b, err := api.blocks.GetBlock(ctx, rp.RootCid())
    if err != nil:
        return nil, err

    # 返回块数据的字节流和空错误
    return bytes.NewReader(b.RawData()), nil
}
// 删除指定路径的块数据
func (api *BlockAPI) Rm(ctx context.Context, p path.Path, opts ...caopts.BlockRmOption) error {
    // 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Rm", trace.WithAttributes(attribute.String("path", p.String())))
    defer span.End()

    // 解析指定路径
    rp, _, err := api.core().ResolvePath(ctx, p)
    if err != nil {
        return err
    }

    // 获取块删除的设置选项
    settings, err := caopts.BlockRmOptions(opts...)
    if err != nil {
        return err
    }
    cids := []cid.Cid{rp.RootCid()}
    o := util.RmBlocksOpts{Force: settings.Force}

    // 删除指定块
    out, err := util.RmBlocks(ctx, api.blockstore, api.pinning, cids, o)
    if err != nil {
        return err
    }

    // 从输出通道中读取结果
    select {
    case res, ok := <-out:
        if !ok {
            return nil
        }

        remBlock, ok := res.(*util.RemovedBlock)
        if !ok {
            return errors.New("got unexpected output from util.RmBlocks")
        }

        if remBlock.Error != nil {
            return remBlock.Error
        }
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

// 获取指定路径块的统计信息
func (api *BlockAPI) Stat(ctx context.Context, p path.Path) (coreiface.BlockStat, error) {
    // 创建一个跟踪 span，并在函数结束时结束该 span
    ctx, span := tracing.Span(ctx, "CoreAPI.BlockAPI", "Stat", trace.WithAttributes(attribute.String("path", p.String())))
    defer span.End()

    // 解析指定路径
    rp, _, err := api.core().ResolvePath(ctx, p)
    if err != nil {
        return nil, err
    }

    // 获取指定块的数据
    b, err := api.blocks.GetBlock(ctx, rp.RootCid())
    if err != nil {
        return nil, err
    }

    // 返回块的统计信息
    return &BlockStat{
        path: path.FromCid(b.Cid()),
        size: len(b.RawData()),
    }, nil
}

// 获取块的大小
func (bs *BlockStat) Size() int {
    return bs.size
}

// 获取块的路径
func (bs *BlockStat) Path() path.ImmutablePath {
    return bs.path
}

// 获取核心 API
func (api *BlockAPI) core() coreiface.CoreAPI {
    return (*CoreAPI)(api)
}
```