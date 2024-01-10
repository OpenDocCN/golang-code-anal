# `kubo\core\coreiface\block.go`

```
package iface

import (
    "context"  // 上下文包，用于处理请求的上下文信息
    "io"  // 输入输出包，用于处理输入输出流

    "github.com/ipfs/boxo/path"  // 导入路径包，用于处理路径信息
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入选项包，用于处理核心接口的选项
)

// BlockStat contains information about a block
type BlockStat interface {
    // Size is the size of a block
    Size() int  // 返回块的大小

    // Path returns path to the block
    Path() path.ImmutablePath  // 返回块的路径
}

// BlockAPI specifies the interface to the block layer
type BlockAPI interface {
    // Put imports raw block data, hashing it using specified settings.
    Put(context.Context, io.Reader, ...options.BlockPutOption) (BlockStat, error)  // 导入原始块数据，并使用指定的设置进行哈希处理

    // Get attempts to resolve the path and return a reader for data in the block
    Get(context.Context, path.Path) (io.Reader, error)  // 尝试解析路径并返回块中数据的读取器

    // Rm removes the block specified by the path from local blockstore.
    // By default an error will be returned if the block can't be found locally.
    //
    // NOTE: If the specified block is pinned it won't be removed and no error
    // will be returned
    Rm(context.Context, path.Path, ...options.BlockRmOption) error  // 从本地块存储中删除指定路径的块

    // Stat returns information on
    Stat(context.Context, path.Path) (BlockStat, error)  // 返回有关块的信息
}
```