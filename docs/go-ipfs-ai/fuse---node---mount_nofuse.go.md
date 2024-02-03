# `kubo\fuse\node\mount_nofuse.go`

```go
// 如果不是在 Windows 平台并且不使用 nofuse 标志构建，则执行下面的代码
// 构建条件：不是在 Windows 平台并且不使用 nofuse 标志
package node

import (
    "errors"

    core "github.com/ipfs/kubo/core"
)

// 挂载函数，接受 IPFS 节点、文件系统目录和命名空间目录作为参数，返回错误
func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
    // 返回一个错误，表示未编译进来
    return errors.New("not compiled in")
}
```