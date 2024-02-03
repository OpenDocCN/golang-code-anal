# `kubo\thirdparty\verifbs\verifbs.go`

```go
package verifbs

import (
    "context"

    bstore "github.com/ipfs/boxo/blockstore"  // 导入 blockstore 包，起别名为 bstore
    "github.com/ipfs/boxo/verifcid"  // 导入 verifcid 包
    blocks "github.com/ipfs/go-block-format"  // 导入 go-block-format 包，起别名为 blocks
    cid "github.com/ipfs/go-cid"  // 导入 go-cid 包，起别名为 cid
)

type VerifBSGC struct {
    bstore.GCBlockstore  // 定义 VerifBSGC 结构体，继承自 GCBlockstore
}

func (bs *VerifBSGC) Put(ctx context.Context, b blocks.Block) error {
    // 验证块的 CID 是否在默认允许列表中
    if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
        return err
    }
    return bs.GCBlockstore.Put(ctx, b)  // 调用 GCBlockstore 的 Put 方法
}

func (bs *VerifBSGC) PutMany(ctx context.Context, blks []blocks.Block) error {
    for _, b := range blks {
        // 验证块的 CID 是否在默认允许列表中
        if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
            return err
        }
    }
    return bs.GCBlockstore.PutMany(ctx, blks)  // 调用 GCBlockstore 的 PutMany 方法
}

func (bs *VerifBSGC) Get(ctx context.Context, c cid.Cid) (blocks.Block, error) {
    // 验证 CID 是否在默认允许列表中
    if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c); err != nil {
        return nil, err
    }
    return bs.GCBlockstore.Get(ctx, c)  // 调用 GCBlockstore 的 Get 方法
}

type VerifBS struct {
    bstore.Blockstore  // 定义 VerifBS 结构体，继承自 Blockstore
}

func (bs *VerifBS) Put(ctx context.Context, b blocks.Block) error {
    // 验证块的 CID 是否在默认允许列表中
    if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
        return err
    }
    return bs.Blockstore.Put(ctx, b)  // 调用 Blockstore 的 Put 方法
}

func (bs *VerifBS) PutMany(ctx context.Context, blks []blocks.Block) error {
    for _, b := range blks {
        // 验证块的 CID 是否在默认允许列表中
        if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, b.Cid()); err != nil {
            return err
        }
    }
    return bs.Blockstore.PutMany(ctx, blks)  // 调用 Blockstore 的 PutMany 方法
}

func (bs *VerifBS) Get(ctx context.Context, c cid.Cid) (blocks.Block, error) {
    // 验证 CID 是否在默认允许列表中
    if err := verifcid.ValidateCid(verifcid.DefaultAllowlist, c); err != nil {
        return nil, err
    }
    return bs.Blockstore.Get(ctx, c)  // 调用 Blockstore 的 Get 方法
}
```