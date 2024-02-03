# `kubo\core\commands\diag.go`

```go
# 导入 commands 包和 cmds 包
import (
    cmds "github.com/ipfs/go-ipfs-cmds"
)

# 定义 DiagCmd 变量，类型为 *cmds.Command
var DiagCmd = &cmds.Command{
    # 帮助文本，包括一句话描述
    Helptext: cmds.HelpText{
        Tagline: "Generate diagnostic reports.",
    },

    # 子命令，包括 "sys"、"cmds"、"profile" 三个子命令
    Subcommands: map[string]*cmds.Command{
        "sys":     sysDiagCmd,      # "sys" 子命令对应 sysDiagCmd
        "cmds":    ActiveReqsCmd,   # "cmds" 子命令对应 ActiveReqsCmd
        "profile": sysProfileCmd,   # "profile" 子命令对应 sysProfileCmd
    },
}
```