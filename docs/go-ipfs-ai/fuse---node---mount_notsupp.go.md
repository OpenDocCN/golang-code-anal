# `kubo\fuse\node\mount_notsupp.go`

```
// 根据条件构建标签，表示在非 nofuse 并且在 openbsd 或者非 nofuse 并且在 netbsd 或者非 nofuse 并且在 plan9 的情况下编译
// +build !nofuse,openbsd !nofuse,netbsd !nofuse,plan9

// 定义包名为 node
package node

// 导入 errors 包
import (
    "errors"

    // 导入 core 包，并重命名为 core
    core "github.com/ipfs/kubo/core"
)

// 定义 Mount 函数，接收一个 IpfsNode 对象和两个字符串参数，返回 error 类型
func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
    // 返回一个错误，表示在 OpenBSD 或者 NetBSD 上不支持 FUSE。参见 issue #5334 (https://github.com/ipfs/kubo/issues/5334)
    return errors.New("FUSE not supported on OpenBSD or NetBSD. See #5334 (https://github.com/ipfs/kubo/issues/5334).")
}
```