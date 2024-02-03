# `kubo\core\commands\repo.go`

```go
package commands

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "errors"   // 错误处理包，用于处理错误信息
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，用于处理输入输出流
    "os"       // 操作系统包，用于操作系统相关功能
    "runtime"  // 运行时包，用于访问运行时环境的信息
    "strings"  // 字符串包，用于处理字符串
    "sync"     // 同步包，用于并发控制
    "text/tabwriter"  // 文本包，用于格式化文本输出

    oldcmds "github.com/ipfs/kubo/commands"  // 导入外部包
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入外部包
    corerepo "github.com/ipfs/kubo/core/corerepo"  // 导入外部包
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 导入外部包
    "github.com/ipfs/kubo/repo/fsrepo/migrations"  // 导入外部包
    "github.com/ipfs/kubo/repo/fsrepo/migrations/ipfsfetcher"  // 导入外部包

    humanize "github.com/dustin/go-humanize"  // 导入外部包
    bstore "github.com/ipfs/boxo/blockstore"  // 导入外部包
    cid "github.com/ipfs/go-cid"  // 导入外部包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入外部包
)

type RepoVersion struct {
    Version string  // 定义结构体 RepoVersion，包含 Version 字段
}

var RepoCmd = &cmds.Command{  // 定义变量 RepoCmd，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置变量 RepoCmd 的 Helptext 字段
        Tagline: "Manipulate the IPFS repo.",  // 设置变量 RepoCmd 的 Tagline 字段
        ShortDescription: `  // 设置变量 RepoCmd 的 ShortDescription 字段
'ipfs repo' is a plumbing command used to manipulate the repo.
`,  // 设置变量 RepoCmd 的 ShortDescription 字段
    },

    Subcommands: map[string]*cmds.Command{  // 设置变量 RepoCmd 的 Subcommands 字段
        "stat":    repoStatCmd,  // 设置 Subcommands 字段的 "stat" 键对应的值
        "gc":      repoGcCmd,  // 设置 Subcommands 字段的 "gc" 键对应的值
        "version": repoVersionCmd,  // 设置 Subcommands 字段的 "version" 键对应的值
        "verify":  repoVerifyCmd,  // 设置 Subcommands 字段的 "verify" 键对应的值
        "migrate": repoMigrateCmd,  // 设置 Subcommands 字段的 "migrate" 键对应的值
        "ls":      RefsLocalCmd,  // 设置 Subcommands 字段的 "ls" 键对应的值
    },
}

// GcResult is the result returned by "repo gc" command.
type GcResult struct {
    Key   cid.Cid  // 定义结构体 GcResult，包含 Key 字段
    Error string `json:",omitempty"`  // 定义结构体 GcResult，包含 Error 字段
}

const (
    repoStreamErrorsOptionName   = "stream-errors"  // 定义常量 repoStreamErrorsOptionName
    repoQuietOptionName          = "quiet"  // 定义常量 repoQuietOptionName
    repoSilentOptionName         = "silent"  // 定义常量 repoSilentOptionName
    repoAllowDowngradeOptionName = "allow-downgrade"  // 定义常量 repoAllowDowngradeOptionName
)

var repoGcCmd = &cmds.Command{  // 定义变量 repoGcCmd，类型为 *cmds.Command
    Helptext: cmds.HelpText{  // 设置变量 repoGcCmd 的 Helptext 字段
        Tagline: "Perform a garbage collection sweep on the repo.",  // 设置变量 repoGcCmd 的 Tagline 字段
        ShortDescription: `  // 设置变量 repoGcCmd 的 ShortDescription 字段
'ipfs repo gc' is a plumbing command that will sweep the local
set of stored objects and remove ones that are not pinned in
order to reclaim hard disk space.
`,  // 设置变量 repoGcCmd 的 ShortDescription 字段
    },
    # 定义一个选项数组，包含三个命令行选项
    Options: []cmds.Option{
        # 定义一个布尔类型的选项，用于控制是否输出错误信息
        cmds.BoolOption(repoStreamErrorsOptionName, "Stream errors."),
        # 定义一个布尔类型的选项，用于控制是否输出最小化的输出
        cmds.BoolOption(repoQuietOptionName, "q", "Write minimal output."),
        # 定义一个布尔类型的选项，用于控制是否不输出任何信息
        cmds.BoolOption(repoSilentOptionName, "Write no output."),
    },
    # 定义一个运行函数，接受请求、响应发射器和环境作为参数
    Run: func(req *cmds.Request, re cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 从请求中获取repoSilentOptionName选项的布尔值
        silent, _ := req.Options[repoSilentOptionName].(bool)
        # 从请求中获取repoStreamErrorsOptionName选项的布尔值
        streamErrors, _ := req.Options[repoStreamErrorsOptionName].(bool)

        # 异步执行垃圾回收操作，返回一个通道
        gcOutChan := corerepo.GarbageCollectAsync(n, req.Context)

        # 如果streamErrors为true，则遍历gcOutChan通道
        if streamErrors {
            errs := false
            for res := range gcOutChan {
                # 如果结果中包含错误，则将错误信息发射出去
                if res.Error != nil {
                    if err := re.Emit(&GcResult{Error: res.Error.Error()}); err != nil {
                        return err
                    }
                    errs = true
                } else {
                    # 否则将移除的键发射出去
                    if err := re.Emit(&GcResult{Key: res.KeyRemoved}); err != nil {
                        return err
                    }
                }
            }
            # 如果遍历过程中出现了错误，则返回错误信息
            if errs {
                return errors.New("encountered errors during gc run")
            }
        } else {
            # 否则调用CollectResult函数处理gcOutChan通道中的结果
            err := corerepo.CollectResult(req.Context, gcOutChan, func(k cid.Cid) {
                # 如果silent为true，则直接返回
                if silent {
                    return
                }
                # 否则发射移除的键
                _ = re.Emit(&GcResult{Key: k})
            })
            # 如果出现错误，则返回错误信息
            if err != nil {
                return err
            }
        }

        # 执行完毕，返回nil
        return nil
    },
    # 定义类型为GcResult的结构
    Type: GcResult{},
    # 创建一个编码器映射，将命令类型与对应的编码器函数关联起来
    Encoders: cmds.EncoderMap{
        # 将文本类型命令与特定的编码器函数关联起来
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, gcr *GcResult) error {
            # 从请求参数中获取安静和静默选项的布尔值
            quiet, _ := req.Options[repoQuietOptionName].(bool)
            silent, _ := req.Options[repoSilentOptionName].(bool)

            # 如果是静默模式，则直接返回
            if silent {
                return nil
            }

            # 如果存在错误信息，则将错误信息写入到输出流中
            if gcr.Error != "" {
                _, err := fmt.Fprintf(w, "Error: %s\n", gcr.Error)
                return err
            }

            # 根据安静选项确定输出前缀
            prefix := "removed "
            if quiet {
                prefix = ""
            }

            # 将前缀和键值写入到输出流中
            _, err := fmt.Fprintf(w, "%s%s\n", prefix, gcr.Key)
            return err
        }),
    },
// 定义常量，用于存储选项名称
const (
    repoSizeOnlyOptionName = "size-only" // 仅报告 RepoSize 和 StorageMax
    repoHumanOptionName    = "human"     // 以人类可读的格式打印大小（例如，1K 234M 2G）
)

// 定义命令
var repoStatCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Get stats for the currently used repo.", // 获取当前使用的存储库的统计信息
        ShortDescription: `
'ipfs repo stat' provides information about the local set of
stored objects. It outputs:

RepoSize        int Size in bytes that the repo is currently taking. // 存储库当前占用的字节数
StorageMax      string Maximum datastore size (from configuration) // 最大数据存储大小（来自配置）
NumObjects      int Number of objects in the local repo. // 本地存储库中的对象数量
RepoPath        string The path to the repo being currently used. // 当前使用的存储库的路径
Version         string The repo version. // 存储库版本
`,
    },
    Options: []cmds.Option{
        cmds.BoolOption(repoSizeOnlyOptionName, "s", "Only report RepoSize and StorageMax."), // 仅报告 RepoSize 和 StorageMax
        cmds.BoolOption(repoHumanOptionName, "H", "Print sizes in human readable format (e.g., 1K 234M 2G)"), // 以人类可读的格式打印大小（例如，1K 234M 2G）
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        n, err := cmdenv.GetNode(env) // 获取节点
        if err != nil {
            return err
        }

        sizeOnly, _ := req.Options[repoSizeOnlyOptionName].(bool) // 获取选项值
        if sizeOnly {
            sizeStat, err := corerepo.RepoSize(req.Context, n) // 获取存储库大小
            if err != nil {
                return err
            }
            return cmds.EmitOnce(res, &corerepo.Stat{
                SizeStat: sizeStat, // 发送存储库大小统计信息
            })
        }

        stat, err := corerepo.RepoStat(req.Context, n) // 获取存储库统计信息
        if err != nil {
            return err
        }

        return cmds.EmitOnce(res, &stat) // 发送存储库统计信息
    },
    Type: &corerepo.Stat{}, // 存储库统计信息类型
}
    # 创建一个命令编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, stat *corerepo.Stat) error {
            # 创建一个 tabwriter，用于格式化输出
            wtr := tabwriter.NewWriter(w, 0, 0, 1, ' ', 0)
            # 在函数结束时刷新并关闭 tabwriter
            defer wtr.Flush()

            # 从请求选项中获取是否为人类可读的标志
            human, _ := req.Options[repoHumanOptionName].(bool)
            # 从请求选项中获取是否仅显示大小的标志
            sizeOnly, _ := req.Options[repoSizeOnlyOptionName].(bool)

            # 定义一个打印大小的函数
            printSize := func(name string, size uint64) {
                # 将大小转换为字符串
                sizeStr := fmt.Sprintf("%d", size)
                # 如果是人类可读的，则使用 humanize 库将大小转换为可读格式
                if human {
                    sizeStr = humanize.Bytes(size)
                }

                # 格式化输出文件名和大小
                fmt.Fprintf(wtr, "%s:\t%s\n", name, sizeStr)
            }

            # 如果不仅显示大小，则打印对象数量
            if !sizeOnly {
                fmt.Fprintf(wtr, "NumObjects:\t%d\n", stat.NumObjects)
            }

            # 打印仓库大小和最大存储大小
            printSize("RepoSize", stat.RepoSize)
            printSize("StorageMax", stat.StorageMax)

            # 如果不仅显示大小，则打印仓库路径和版本
            if !sizeOnly {
                fmt.Fprintf(wtr, "RepoPath:\t%s\n", stat.RepoPath)
                fmt.Fprintf(wtr, "Version:\t%s\n", stat.Version)
            }

            # 返回空值
            return nil
        }),
    },
// 定义用于验证进度的结构体，包含消息和进度两个字段
type VerifyProgress struct {
    Msg      string
    Progress int
}

// 验证工作协程函数，接收上下文、等待组、Cid通道、结果通道和块存储作为参数
func verifyWorkerRun(ctx context.Context, wg *sync.WaitGroup, keys <-chan cid.Cid, results chan<- string, bs bstore.Blockstore) {
    // 在函数退出时执行等待组的Done方法
    defer wg.Done()

    // 遍历Cid通道中的键
    for k := range keys {
        // 从块存储中获取指定Cid的块数据
        _, err := bs.Get(ctx, k)
        // 如果出现错误
        if err != nil {
            // 选择发送错误消息到结果通道或者退出
            select {
            case results <- fmt.Sprintf("block %s was corrupt (%s)", k, err):
            case <-ctx.Done():
                return
            }
            // 继续下一次循环
            continue
        }
        // 选择发送空消息到结果通道或者退出
        select {
        case results <- "":
        case <-ctx.Done():
            return
        }
    }
}

// 验证结果通道函数，接收上下文、Cid通道和块存储作为参数，返回结果通道
func verifyResultChan(ctx context.Context, keys <-chan cid.Cid, bs bstore.Blockstore) <-chan string {
    // 创建结果通道
    results := make(chan string)

    // 启动协程
    go func() {
        // 在函数退出时关闭结果通道
        defer close(results)

        // 创建等待组
        var wg sync.WaitGroup

        // 根据CPU核心数启动验证工作协程
        for i := 0; i < runtime.NumCPU()*2; i++ {
            // 增加等待组计数
            wg.Add(1)
            // 启动验证工作协程
            go verifyWorkerRun(ctx, &wg, keys, results, bs)
        }

        // 等待所有工作协程结束
        wg.Wait()
    }()

    // 返回结果通道
    return results
}

// 定义repoVerifyCmd命令
var repoVerifyCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Verify all blocks in repo are not corrupted.",
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 创建新的块存储，并设置在读取时进行哈希
        bs := bstore.NewBlockstore(nd.Repo.Datastore())
        bs.HashOnRead(true)

        # 获取所有块的键的通道
        keys, err := bs.AllKeysChan(req.Context)
        if err != nil {
            log.Error(err)
            return err
        }

        # 使用键的通道和块存储创建验证结果的通道
        results := verifyResultChan(req.Context, keys, bs)

        # 初始化失败计数和进度计数
        var fails int
        var i int
        # 遍历验证结果通道
        for msg := range results {
            # 如果消息不为空
            if msg != "" {
                # 发送验证进度消息
                if err := res.Emit(&VerifyProgress{Msg: msg}); err != nil {
                    return err
                }
                # 增加失败计数
                fails++
            }
            # 增加进度计数
            i++
            # 发送验证进度消息
            if err := res.Emit(&VerifyProgress{Progress: i}); err != nil {
                return err
            }
        }

        # 检查请求上下文是否有错误
        if err := req.Context.Err(); err != nil {
            return err
        }

        # 如果有失败的块，则返回错误
        if fails != 0 {
            return errors.New("verify complete, some blocks were corrupt")
        }

        # 发送验证完成消息
        return res.Emit(&VerifyProgress{Msg: "verify complete, all blocks validated."})
    },
    # 定义类型为 VerifyProgress
    Type: &VerifyProgress{},
    # 设置编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, obj *VerifyProgress) error {
            # 如果消息包含 "was corrupt"，则打印到标准输出
            if strings.Contains(obj.Msg, "was corrupt") {
                fmt.Fprintln(os.Stdout, obj.Msg)
                return nil
            }

            # 如果消息不为空
            if obj.Msg != "" {
                # 如果消息长度小于20，则补充空格
                if len(obj.Msg) < 20 {
                    obj.Msg += "             "
                }
                # 将消息写入到输出流
                fmt.Fprintln(w, obj.Msg)
                return nil
            }

            # 将块处理进度写入到输出流
            fmt.Fprintf(w, "%d blocks processed.\r", obj.Progress)
            return nil
        }),
    },
# 定义名为repoVersionCmd的命令对象，用于显示仓库版本信息
var repoVersionCmd = &cmds.Command{
    # 帮助文本，包括简短描述和短描述
    Helptext: cmds.HelpText{
        Tagline: "Show the repo version.",  # 显示仓库版本
        ShortDescription: `
'ipfs repo version' returns the current repo version.  # 'ipfs repo version' 返回当前仓库版本
`,
    },

    # 选项，包括repoQuietOptionName选项
    Options: []cmds.Option{
        cmds.BoolOption(repoQuietOptionName, "q", "Write minimal output."),  # 布尔选项，用于控制输出的最小化
    },

    # 运行函数，处理命令请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        return cmds.EmitOnce(res, &RepoVersion{  # 一次性发射结果
            Version: fmt.Sprint(fsrepo.RepoVersion),  # 版本信息
        })
    },
    Type: RepoVersion{},  # 类型为RepoVersion
    Encoders: cmds.EncoderMap{  # 编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *RepoVersion) error {  # 文本编码器
            quiet, _ := req.Options[repoQuietOptionName].(bool)  # 获取repoQuietOptionName选项的布尔值

            if quiet {  # 如果是安静模式
                fmt.Fprintf(w, "fs-repo@%s\n", out.Version)  # 输出最小化信息
            } else {
                fmt.Fprintf(w, "ipfs repo version fs-repo@%s\n", out.Version)  # 输出完整信息
            }
            return nil
        }),
    },
}

# 定义名为repoMigrateCmd的命令对象，用于应用仓库的未完成迁移
var repoMigrateCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Apply any outstanding migrations to the repo.",  # 应用仓库的未完成迁移
    },
    Options: []cmds.Option{
        cmds.BoolOption(repoAllowDowngradeOptionName, "Allow downgrading to a lower repo version"),  # 布尔选项，允许降级到较低的仓库版本
    },
    NoRemote: true,  # 不允许远程操作
}
```