# `kubo\cmd\ipfs\kubo\ipfs.go`

```
// 导入 kubo 包
package kubo

// 导入 IPFS 相关的包
import (
    commands "github.com/ipfs/kubo/core/commands"
    cmds "github.com/ipfs/go-ipfs-cmds"
)

// Root 是 CLI 的根命令，用于执行 CLI 客户端可访问的命令。
// 一些子命令（如 'ipfs daemon' 或 'ipfs init'）只能在这里访问，不能通过 HTTP API 调用。
var Root = &cmds.Command{
    Options:  commands.Root.Options,  // 设置命令的选项
    Helptext: commands.Root.Helptext,  // 设置命令的帮助文本
}

// commandsClientCmd 是本地 CLI 的 "ipfs commands" 命令。
var commandsClientCmd = commands.CommandsCmd(Root)

// localCommands 中的命令应始终在本地运行（即使守护程序正在运行）。
// 它们可以通过定义具有相同名称的子命令来覆盖 commands.Root 中的子命令。
var localCommands = map[string]*cmds.Command{
    "daemon":   daemonCmd,  // 守护程序命令
    "init":     initCmd,  // 初始化命令
    "commands": commandsClientCmd,  // 命令客户端命令
}

func init() {
    // 在这里设置而不是在文字中以防止初始化循环
    // （一些命令引用了 Root）
    Root.Subcommands = localCommands  // 设置根命令的子命令

    for k, v := range commands.Root.Subcommands {
        if _, found := Root.Subcommands[k]; !found {
            Root.Subcommands[k] = v
        }
    }
}
```