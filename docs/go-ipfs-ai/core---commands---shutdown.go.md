# `kubo\core\commands\shutdown.go`

```go
package commands

import (
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令行操作库
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入环境命令库
)

var daemonShutdownCmd = &cmds.Command{  // 定义关闭守护进程的命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Shut down the IPFS daemon.",  // 简短描述
    },
    Run: func(req *cmds.Request, re cmds.ResponseEmitter, env cmds.Environment) error {  // 运行函数
        nd, err := cmdenv.GetNode(env)  // 获取节点
        if err != nil {  // 如果出现错误
            return err  // 返回错误
        }

        if !nd.IsDaemon {  // 如果节点不是守护进程
            return cmds.Errorf(cmds.ErrClient, "daemon not running")  // 返回错误信息
        }

        if err := nd.Close(); err != nil {  // 如果关闭节点出现错误
            log.Error("error while shutting down ipfs daemon:", err)  // 记录错误日志
        }

        return nil  // 返回空
    },
}
```