# `kubo\core\commands\mount_unix.go`

```go
// 如果不是在 Windows 平台并且不是在 nofuse 模式下构建，则包含该文件
// +build !windows,!nofuse

package commands

import (
    "fmt"
    "io"

    oldcmds "github.com/ipfs/kubo/commands"  // 导入旧的命令包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境包
    nodeMount "github.com/ipfs/kubo/fuse/node"  // 导入节点挂载包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令包
    config "github.com/ipfs/kubo/config"  // 导入配置包
)

const (
    mountIPFSPathOptionName = "ipfs-path"  // 定义 IPFS 路径选项名称
    mountIPNSPathOptionName = "ipns-path"  // 定义 IPNS 路径选项名称
)

var MountCmd = &cmds.Command{  // 定义 MountCmd 命令
    Status: cmds.Experimental,  // 设置命令状态为实验性
    Helptext: cmds.HelpText{  // 设置命令的帮助文本
        Tagline: "Mounts IPFS to the filesystem (read-only).",  // 设置命令的一句话描述
        ShortDescription: `  // 设置命令的简短描述
Mount IPFS at a read-only mountpoint on the OS (default: /ipfs and /ipns).
All IPFS objects will be accessible under that directory. Note that the
root will not be listable, as it is virtual. Access known paths directly.

You may have to create /ipfs and /ipns before using 'ipfs mount':

> sudo mkdir /ipfs /ipns
> sudo chown $(whoami) /ipfs /ipns
> ipfs daemon &
> ipfs mount
`,
        LongDescription: `  // 设置命令的详细描述
Mount IPFS at a read-only mountpoint on the OS. The default, /ipfs and /ipns,
are set in the configuration file, but can be overridden by the options.
All IPFS objects will be accessible under this directory. Note that the
root will not be listable, as it is virtual. Access known paths directly.

You may have to create /ipfs and /ipns before using 'ipfs mount':

> sudo mkdir /ipfs /ipns
> sudo chown $(whoami) /ipfs /ipns
> ipfs daemon &
> ipfs mount

Example:

# setup
> mkdir foo
> echo "baz" > foo/bar
> ipfs add -r foo
added QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR foo/bar
added QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC foo
> ipfs ls QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC
QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR 12 bar
> ipfs cat QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR
baz

# mount
> ipfs daemon &
> ipfs mount
IPFS mounted at: /ipfs
IPNS mounted at: /ipns
> cd /ipfs/QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC
> ls
bar
> cat bar
`,
    },
)
// 定义一个名为 baz 的结构体，包含两个字段
baz
> cat /ipfs/QmSh5e7S6fdcu75LAbXNZAFY2nGyZUJXyLCJDvn2zRkWyC/bar
baz
> cat /ipfs/QmWLdkp93sNxGRjnFHPaYg8tCQ35NBY3XPn6KiETd3Z4WR
baz
`,
    },
    // 定义 Options 字段，包含两个 StringOption 类型的选项
    Options: []cmds.Option{
        cmds.StringOption(mountIPFSPathOptionName, "f", "The path where IPFS should be mounted."),
        cmds.StringOption(mountIPNSPathOptionName, "n", "The path where IPNS should be mounted."),
    },
    // 定义 Run 方法，接收请求、响应和环境作为参数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 从环境中获取配置信息
        cfg, err := env.(*oldcmds.Context).GetConfig()
        if err != nil {
            return err
        }

        // 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        // 如果节点不处于在线模式，则返回错误
        if !nd.IsOnline {
            return ErrNotOnline
        }

        // 获取 IPFS 的挂载路径，如果未指定则使用默认值
        fsdir, found := req.Options[mountIPFSPathOptionName].(string)
        if !found {
            fsdir = cfg.Mounts.IPFS // use default value
        }

        // 获取 IPNS 的挂载路径，如果未指定则使用默认值
        nsdir, found := req.Options[mountIPNSPathOptionName].(string)
        if !found {
            nsdir = cfg.Mounts.IPNS // NB: be sure to not redeclare!
        }

        // 挂载 IPFS 和 IPNS
        err = nodeMount.Mount(nd, fsdir, nsdir)
        if err != nil {
            return err
        }

        // 构造输出结构体并返回
        var output config.Mounts
        output.IPFS = fsdir
        output.IPNS = nsdir
        return cmds.EmitOnce(res, &output)
    },
    // 定义 Type 字段，类型为 config.Mounts
    Type: config.Mounts{},
    // 定义 Encoders 字段，包含一个 Text 类型的编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, mounts *config.Mounts) error {
            // 将 IPFS 和 IPNS 的挂载路径输出到文本中
            fmt.Fprintf(w, "IPFS mounted at: %s\n", cmdenv.EscNonPrint(mounts.IPFS))
            fmt.Fprintf(w, "IPNS mounted at: %s\n", cmdenv.EscNonPrint(mounts.IPNS))

            return nil
        }),
    },
}
```