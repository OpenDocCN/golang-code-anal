# `kubo\core\commands\mount_nofuse.go`

```go
// 如果不是在 Windows 平台并且没有启用 nofuse 标志，则编译此代码
// +build !windows,nofuse

// 声明 commands 包
package commands

// 导入 cmds 包，用于处理命令
import (
    cmds "github.com/ipfs/go-ipfs-cmds"
)

// MountCmd 是一个命令对象，用于挂载 ipfs 到文件系统（已禁用）
var MountCmd = &cmds.Command{
    // 声明该命令是实验性质的
    Status: cmds.Experimental,
    // 帮助文本
    Helptext: cmds.HelpText{
        // 一句话描述
        Tagline: "Mounts ipfs to the filesystem (disabled).",
        // 简短描述
        ShortDescription: `
This version of ipfs is compiled without fuse support, which is required
for mounting. If you'd like to be able to mount, please use a version of
ipfs compiled with fuse.

For the latest instructions, please check the project's repository:
  http://github.com/ipfs/go-ipfs
`,
    },
}
```