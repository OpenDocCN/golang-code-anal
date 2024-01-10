# `kubo\core\commands\version.go`

```
package commands

import (
    "errors"  // 导入 errors 包
    "fmt"  // 导入 fmt 包
    "io"  // 导入 io 包
    "runtime/debug"  // 导入 runtime/debug 包

    version "github.com/ipfs/kubo"  // 导入版本信息包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令包
)

const (
    versionNumberOptionName = "number"  // 定义版本号选项名称
    versionCommitOptionName = "commit"  // 定义提交哈希选项名称
    versionRepoOptionName   = "repo"  // 定义仓库版本选项名称
    versionAllOptionName    = "all"  // 定义显示所有版本信息选项名称
)

var VersionCmd = &cmds.Command{  // 定义版本命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline:          "Show IPFS version information.",  // 标语
        ShortDescription: "Returns the current version of IPFS and exits.",  // 简短描述
    },
    Subcommands: map[string]*cmds.Command{  // 子命令
        "deps": depsVersionCommand,  // 依赖版本命令
    },

    Options: []cmds.Option{  // 选项
        cmds.BoolOption(versionNumberOptionName, "n", "Only show the version number."),  // 版本号选项
        cmds.BoolOption(versionCommitOptionName, "Show the commit hash."),  // 提交哈希选项
        cmds.BoolOption(versionRepoOptionName, "Show repo version."),  // 仓库版本选项
        cmds.BoolOption(versionAllOptionName, "Show all version information"),  // 显示所有版本信息选项
    },
    // must be permitted to run before init
    Extra: CreateCmdExtras(SetDoesNotUseRepo(true), SetDoesNotUseConfigAsInput(true)),  // 额外设置
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 运行函数
        return cmds.EmitOnce(res, version.GetVersionInfo())  // 发射版本信息
    },
    # 创建 Encoders 对象，包含不同类型的编码器
    Encoders: cmds.EncoderMap{
        # 创建文本类型的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, version *version.VersionInfo) error {
            # 从请求参数中获取 versionAllOptionName 对应的布尔值
            all, _ := req.Options[versionAllOptionName].(bool)
            # 如果 all 为 true，则输出完整版本信息
            if all {
                # 拼接版本信息字符串
                ver := version.Version
                if version.Commit != "" {
                    ver += "-" + version.Commit
                }
                out := fmt.Sprintf("Kubo version: %s\n"+
                    "Repo version: %s\nSystem version: %s\nGolang version: %s\n",
                    ver, version.Repo, version.System, version.Golang)
                # 将版本信息输出到指定的 io.Writer
                fmt.Fprint(w, out)
                return nil
            }

            # 从请求参数中获取 versionCommitOptionName 对应的布尔值
            commit, _ := req.Options[versionCommitOptionName].(bool)
            commitTxt := ""
            # 如果 commit 为 true 且版本信息中包含 commit，则设置 commitTxt
            if commit && version.Commit != "" {
                commitTxt = "-" + version.Commit
            }

            # 从请求参数中获取 versionRepoOptionName 对应的布尔值
            repo, _ := req.Options[versionRepoOptionName].(bool)
            # 如果 repo 为 true，则输出版本的 Repo 信息
            if repo {
                fmt.Fprintln(w, version.Repo)
                return nil
            }

            # 从请求参数中获取 versionNumberOptionName 对应的布尔值
            number, _ := req.Options[versionNumberOptionName].(bool)
            # 如果 number 为 true，则输出版本号信息
            if number {
                fmt.Fprintln(w, version.Version+commitTxt)
                return nil
            }

            # 输出默认的版本信息
            fmt.Fprintf(w, "ipfs version %s%s\n", version.Version, commitTxt)
            return nil
        }),
    },
    # 创建 Type 对象，包含版本信息
    Type: version.VersionInfo{},
// 定义依赖结构体，包含路径、版本、替换信息和校验和
type Dependency struct {
    Path       string
    Version    string
    ReplacedBy string
    Sum        string
}

// 定义包版本格式字符串
const pkgVersionFmt = "%s@%s"

// 创建命令对象，用于显示构建所使用的依赖信息
var depsVersionCommand = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Shows information about dependencies used for build.",
        ShortDescription: `
Print out all dependencies and their versions.`,
    },
    Type: Dependency{}, // 设置命令对象的类型为依赖结构体

    // 定义命令执行的函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 读取构建信息
        info, ok := debug.ReadBuildInfo()
        if !ok {
            return errors.New("no embedded dependency information")
        }
        // 将 debug.Module 转换为 Dependency 结构体
        toDependency := func(mod *debug.Module) (dep Dependency) {
            dep.Path = mod.Path
            dep.Version = mod.Version
            dep.Sum = mod.Sum
            if repl := mod.Replace; repl != nil {
                dep.ReplacedBy = fmt.Sprintf(pkgVersionFmt, repl.Path, repl.Version)
            }
            return
        }
        // 发送主模块的依赖信息
        if err := res.Emit(toDependency(&info.Main)); err != nil {
            return err
        }
        // 发送所有依赖模块的信息
        for _, dep := range info.Deps {
            if err := res.Emit(toDependency(dep)); err != nil {
                return err
            }
        }
        return nil
    },
    // 设置命令的编码器，用于将依赖信息编码为文本格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, dep Dependency) error {
            fmt.Fprintf(w, pkgVersionFmt, dep.Path, dep.Version)
            if dep.ReplacedBy != "" {
                fmt.Fprintf(w, " => %s", dep.ReplacedBy)
            }
            fmt.Fprintf(w, "\n")
            return nil
        }),
    },
}
```