# `kubo\fuse\node\mount_windows.go`

```go
// 导入 node 包和 core 包
package node

import (
    "github.com/ipfs/kubo/core"
)

// 挂载函数，接受一个 IpfsNode 对象和两个字符串参数，返回错误
func Mount(node *core.IpfsNode, fsdir, nsdir string) error {
    // TODO
    // 目前是一个空操作，但我们不想返回错误
    return nil
}
```