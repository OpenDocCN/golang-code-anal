# `kubo\core\commands\cmdutils\utils.go`

```go
package cmdutils

import (
    "fmt" // 导入 fmt 包

    cmds "github.com/ipfs/go-ipfs-cmds" // 导入 cmds 包

    "github.com/ipfs/boxo/path" // 导入 path 包
    "github.com/ipfs/go-cid" // 导入 cid 包
    coreiface "github.com/ipfs/kubo/core/coreiface" // 导入 coreiface 包
)

const (
    AllowBigBlockOptionName = "allow-big-block" // 定义常量 AllowBigBlockOptionName
    SoftBlockLimit          = 1024 * 1024 // 定义常量 SoftBlockLimit，表示 1MiB
)

var AllowBigBlockOption cmds.Option // 定义变量 AllowBigBlockOption

func init() {
    AllowBigBlockOption = cmds.BoolOption(AllowBigBlockOptionName, "Disable block size check and allow creation of blocks bigger than 1MiB. WARNING: such blocks won't be transferable over the standard bitswap.").WithDefault(false) // 初始化 AllowBigBlockOption
}

func CheckCIDSize(req *cmds.Request, c cid.Cid, dagAPI coreiface.APIDagService) error {
    n, err := dagAPI.Get(req.Context, c) // 通过 dagAPI 获取 CID 对应的节点
    if err != nil {
        return fmt.Errorf("CheckCIDSize: getting dag: %w", err) // 返回错误信息
    }

    nodeSize, err := n.Size() // 获取节点的大小
    if err != nil {
        return fmt.Errorf("CheckCIDSize: getting node size: %w", err) // 返回错误信息
    }

    return CheckBlockSize(req, nodeSize) // 调用 CheckBlockSize 函数检查节点大小
}

func CheckBlockSize(req *cmds.Request, size uint64) error {
    allowAnyBlockSize, _ := req.Options[AllowBigBlockOptionName].(bool) // 从请求参数中获取是否允许任意块大小

    if allowAnyBlockSize {
        return nil // 如果允许任意块大小，则返回空
    }

    // 如果节点大小超过 SoftBlockLimit，返回错误信息
    if size > SoftBlockLimit {
        return fmt.Errorf("produced block is over 1MiB: big blocks can't be exchanged with other peers. consider using UnixFS for automatic chunking of bigger files, or pass --allow-big-block to override")
    }
    return nil // 否则返回空
}

// PathOrCidPath returns a path.Path built from the argument. It keeps the old
// behaviour by building a path from a CID string.
func PathOrCidPath(str string) (path.Path, error) {
    p, err := path.NewPath(str) // 从字符串构建 path.Path
    if err == nil {
        return p, nil // 如果没有错误，返回构建的 path.Path
    }
    // 如果有错误，继续执行其他逻辑
}
    // 如果 p, err := path.NewPath("/ipfs/" + str) 成功执行，将 p 返回并且 err 返回 nil
    if p, err := path.NewPath("/ipfs/" + str); err == nil {
        return p, nil
    }

    // 如果 path.NewPath("/ipfs/" + str) 执行失败，将原始的 err 返回
    // Send back original err.
    return nil, err
# 闭合前面的函数定义
```