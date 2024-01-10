# `kubo\core\commands\pin\remotepin.go`

```
package pin

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "fmt"  // 格式化包，用于格式化输出
    "io"  // 输入输出包，提供了基本的输入输出功能
    "os"  // 操作系统包，提供了对操作系统的访问功能
    "sort"  // 排序包，提供了对切片和用户自定义集合的排序功能
    "strings"  // 字符串包，提供了对字符串的操作功能
    "text/tabwriter"  // 文本包，提供了格式化文本输出功能
    "time"  // 时间包，提供了时间的操作和格式化功能

    neturl "net/url"  // 网络URL包，提供了对URL的解析和生成功能
    gopath "path"  // 路径包，提供了对文件路径的操作功能

    "golang.org/x/sync/errgroup"  // 错误组包，提供了并发执行多个函数的能力

    pinclient "github.com/ipfs/boxo/pinning/remote/client"  // 远程固定客户端包，提供了远程固定服务的客户端功能
    cid "github.com/ipfs/go-cid"  // CID包，提供了CID（内容标识符）的功能
    cmds "github.com/ipfs/go-ipfs-cmds"  // 命令包，提供了命令行操作的功能
    logging "github.com/ipfs/go-log"  // 日志包，提供了日志记录和输出功能
    config "github.com/ipfs/kubo/config"  // 配置包，提供了配置管理功能
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 命令环境包，提供了命令环境的功能
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 文件系统仓库包，提供了文件系统仓库的功能
    "github.com/libp2p/go-libp2p/core/host"  // 主机包，提供了主机的功能
    peer "github.com/libp2p/go-libp2p/core/peer"  // 对等体包，提供了对等体的功能
)

var log = logging.Logger("core/commands/cmdenv")  // 定义日志记录器

var remotePinCmd = &cmds.Command{  // 定义远程固定命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Pin (and unpin) objects to remote pinning service.",  // 标语
    },

    Subcommands: map[string]*cmds.Command{  // 子命令
        "add":     addRemotePinCmd,  // 添加远程固定命令
        "ls":      listRemotePinCmd,  // 列出远程固定命令
        "rm":      rmRemotePinCmd,  // 删除远程固定命令
        "service": remotePinServiceCmd,  // 远程固定服务命令
    },
}

var remotePinServiceCmd = &cmds.Command{  // 定义远程固定服务命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Configure remote pinning services.",  // 标语
    },

    Subcommands: map[string]*cmds.Command{  // 子命令
        "add": addRemotePinServiceCmd,  // 添加远程固定服务命令
        "ls":  lsRemotePinServiceCmd,  // 列出远程固定服务命令
        "rm":  rmRemotePinServiceCmd,  // 删除远程固定服务命令
    },
}

const (  // 常量定义
    pinNameOptionName         = "name"  // 名称选项
    pinCIDsOptionName         = "cid"  // CID选项
    pinStatusOptionName       = "status"  // 状态选项
    pinServiceNameOptionName  = "service"  // 服务名称选项
    pinServiceNameArgName     = pinServiceNameOptionName  // 服务名称参数
    pinServiceEndpointArgName = "endpoint"  // 服务端点参数
    pinServiceKeyArgName      = "key"  // 服务密钥参数
    pinServiceStatOptionName  = "stat"  // 服务状态选项
    pinBackgroundOptionName   = "background"  // 后台选项
    pinForceOptionName        = "force"  // 强制选项
)

type RemotePinOutput struct {  // 远程固定输出结构体
    Status string  // 状态
    Cid    string  // CID
    Name   string  // 名称
}

func toRemotePinOutput(ps pinclient.PinStatusGetter) RemotePinOutput {  // 转换为远程固定输出
    # 返回一个 RemotePinOutput 结构体
    return RemotePinOutput{
        # 设置 Name 字段为 ps.GetPin() 的名称
        Name:   ps.GetPin().GetName(),
        # 设置 Status 字段为 ps.GetStatus() 的字符串表示
        Status: ps.GetStatus().String(),
        # 设置 Cid 字段为 ps.GetPin() 的 Cid 的字符串表示
        Cid:    ps.GetPin().GetCid().String(),
    }
// 打印远程固定的细节
func printRemotePinDetails(w io.Writer, out *RemotePinOutput) {
    // 创建一个新的 tabwriter.Writer，用于格式化输出
    tw := tabwriter.NewWriter(w, 0, 0, 1, ' ', 0)
    // 在函数返回时刷新 tabwriter.Writer
    defer tw.Flush()
    // 创建一个函数，用于将键值对格式化输出到 tabwriter.Writer
    fw := func(k string, v string) {
        fmt.Fprintf(tw, "%s:\t%s\n", k, v)
    }
    // 调用函数将 CID 输出到 tabwriter.Writer
    fw("CID", out.Cid)
    // 调用函数将 Name 输出到 tabwriter.Writer
    fw("Name", out.Name)
    // 调用函数将 Status 输出到 tabwriter.Writer
    fw("Status", out.Status)
}

// 远程固定命令

// 创建一个字符串选项，用于指定远程固定服务的名称
var pinServiceNameOption = cmds.StringOption(pinServiceNameOptionName, "Name of the remote pinning service to use (mandatory).")

// 创建一个命令，用于将对象固定到远程固定服务
var addRemotePinCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Pin object to remote pinning service.",
        ShortDescription: "Asks remote pinning service to pin an IPFS object from a given path.",
        LongDescription: `
Asks remote pinning service to pin an IPFS object from a given path or a CID.

To pin CID 'bafkqaaa' to service named 'mysrv' under a pin named 'mypin':

  $ ipfs pin remote add --service=mysrv --name=mypin bafkqaaa

The above command will block until remote service returns 'pinned' status,
which may take time depending on the size and available providers of the pinned
data.

If you prefer to not wait for pinning confirmation and return immediately
after remote service confirms 'queued' status, add the '--background' flag:

  $ ipfs pin remote add --service=mysrv --name=mypin --background bafkqaaa

Status of background pin requests can be inspected with the 'ls' command.

To list all pins for the CID across all statuses:

  $ ipfs pin remote ls --service=mysrv --cid=bafkqaaa --status=queued \
      --status=pinning --status=pinned --status=failed

NOTE: a comma-separated notation is supported in CLI for convenience:

  $ ipfs pin remote ls --service=mysrv --cid=bafkqaaa --status=queued,pinning,pinned,failed

`,
    },

    // 创建一个字符串参数，用于指定要固定的 CID 或路径
    Arguments: []cmds.Argument{
        cmds.StringArg("ipfs-path", true, false, "CID or Path to be pinned."),
    },
    # 选项列表，包含多个命令选项
    Options: []cmds.Option{
        # 添加服务名称选项
        pinServiceNameOption,
        # 添加一个字符串选项，用于指定固定名称
        cmds.StringOption(pinNameOptionName, "An optional name for the pin."),
        # 添加一个布尔选项，用于指定是否在远程服务上添加到队列并立即返回（不等待固定状态）
        cmds.BoolOption(pinBackgroundOptionName, "Add to the queue on the remote service and return immediately (does not wait for pinned status).").WithDefault(false),
    },
    # 类型为 RemotePinOutput 的对象
    Type: RemotePinOutput{},
    },
    # 编码器映射，将文本编码器与函数绑定
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *RemotePinOutput) error {
            # 打印远程固定的详细信息
            printRemotePinDetails(w, out)
            # 返回空值
            return nil
        }),
    },
}
// 定义 listRemotePinCmd 命令
var listRemotePinCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "List objects pinned to remote pinning service.", // 标签行
        ShortDescription: `
Returns a list of objects that are pinned to a remote pinning service.
`, // 简短描述
        LongDescription: `
Returns a list of objects that are pinned to a remote pinning service.

NOTE: By default, it will only show matching objects in 'pinned' state.
Pass '--status=queued,pinning,pinned,failed' to list pins in all states.
`, // 长描述
    },
    // 参数
    Arguments: []cmds.Argument{},
    // 选项
    Options: []cmds.Option{
        pinServiceNameOption, // pin 服务名称选项
        cmds.StringOption(pinNameOptionName, "Return pins with names that contain the value provided (case-sensitive, exact match)."), // 返回包含指定值的名称的 pin
        cmds.DelimitedStringsOption(",", pinCIDsOptionName, "Return pins for the specified CIDs (comma-separated)."), // 返回指定 CID 的 pin
        cmds.DelimitedStringsOption(",", pinStatusOptionName, "Return pins with the specified statuses (queued,pinning,pinned,failed).").WithDefault([]string{"pinned"}), // 返回指定状态的 pin，默认为 "pinned"
    },
    // 运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        ctx, cancel := context.WithCancel(req.Context) // 创建上下文
        defer cancel() // 延迟取消

        c, err := getRemotePinServiceFromRequest(req, env) // 从请求和环境中获取远程 pin 服务
        if err != nil {
            return err
        }

        psCh, errCh, err := lsRemote(ctx, req, c) // 调用 lsRemote 函数获取远程 pin 服务的信息
        if err != nil {
            return err
        }

        for ps := range psCh {
            if err := res.Emit(toRemotePinOutput(ps)); err != nil { // 将远程 pin 服务的信息转换为输出并发送
                return err
            }
        }

        return <-errCh // 返回错误通道的值
    },
    Type: RemotePinOutput{}, // 类型为 RemotePinOutput
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *RemotePinOutput) error {
            // pin remote ls produces a flat output similar to legacy pin ls
            fmt.Fprintf(w, "%s\t%s\t%s\n", out.Cid, out.Status, cmdenv.EscNonPrint(out.Name)) // 格式化输出
            return nil
        }),
    },
}
// 执行 GET /pins/?query-with-filters
func lsRemote(ctx context.Context, req *cmds.Request, c *pinclient.Client) (chan pinclient.PinStatusGetter, chan error, error) {
    // 初始化选项数组
    opts := []pinclient.LsOption{}
    // 如果请求中包含 pinNameOptionName，将其添加到选项数组中
    if name, nameFound := req.Options[pinNameOptionName]; nameFound {
        nameStr := name.(string)
        opts = append(opts, pinclient.PinOpts.FilterName(nameStr))
    }

    // 如果请求中包含 pinCIDsOptionName，将其添加到选项数组中
    if cidsRaw, cidsFound := req.Options[pinCIDsOptionName]; cidsFound {
        cidsRawArr := cidsRaw.([]string)
        var parsedCIDs []cid.Cid
        for _, rawCID := range cidsRawArr {
            parsedCID, err := cid.Decode(rawCID)
            if err != nil {
                return nil, nil, fmt.Errorf("CID %q cannot be parsed: %v", rawCID, err)
            }
            parsedCIDs = append(parsedCIDs, parsedCID)
        }
        opts = append(opts, pinclient.PinOpts.FilterCIDs(parsedCIDs...))
    }

    // 如果请求中包含 pinStatusOptionName，将其添加到选项数组中
    if statusRaw, statusFound := req.Options[pinStatusOptionName]; statusFound {
        statusRawArr := statusRaw.([]string)
        var parsedStatuses []pinclient.Status
        for _, rawStatus := range statusRawArr {
            s := pinclient.Status(rawStatus)
            if s.String() == string(pinclient.StatusUnknown) {
                return nil, nil, fmt.Errorf("status %q is not valid", rawStatus)
            }
            parsedStatuses = append(parsedStatuses, s)
        }
        opts = append(opts, pinclient.PinOpts.FilterStatus(parsedStatuses...))
    }

    // 调用远程服务的 Ls 方法，传入选项数组
    psCh, errCh := c.Ls(ctx, opts...)

    return psCh, errCh, nil
}

// rmRemotePinCmd 命令
var rmRemotePinCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Remove pins from remote pinning service.",
        ShortDescription: "Removes the remote pin, allowing it to be garbage-collected if needed.",
        LongDescription: `
Removes remote pins, allowing them to be garbage-collected if needed.

This command accepts the same search query parameters as 'ls', and it is good
# 在执行 'rm' 命令之前，先执行 'ls' 命令确认要移除的 pin 列表。

# 为特定的 CID 移除单个 pin：
  $ ipfs pin remote ls --service=mysrv --cid=bafkqaaa
  $ ipfs pin remote rm --service=mysrv --cid=bafkqaaa

# 当远程服务上有多个 pin 与查询匹配时，会返回错误。要确认移除多个 pin，需要传入 '--force' 参数：
  $ ipfs pin remote ls --service=mysrv --name=popular-name
  $ ipfs pin remote rm --service=mysrv --name=popular-name --force

# 注意：当没有传入 '--status' 参数时，隐式使用 '--status=pinned'。要列出然后移除所有待定的 pin 请求，需要传入一个显式的状态列表：
  $ ipfs pin remote ls --service=mysrv --status=queued,pinning,failed
  $ ipfs pin remote rm --service=mysrv --status=queued,pinning,failed --force
    // Run 函数接收请求、响应和环境参数，执行远程 pin 的删除操作
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 使用请求的上下文创建一个新的上下文，并返回一个取消函数
        ctx, cancel := context.WithCancel(req.Context)
        // 在函数返回时调用取消函数，确保上下文被正确释放
        defer cancel()

        // 从请求和环境参数中获取远程 pin 服务
        c, err := getRemotePinServiceFromRequest(req, env)
        // 如果获取远程 pin 服务出错，直接返回错误
        if err != nil {
            return err
        }

        // 初始化一个空的远程 pin ID 列表
        rmIDs := []string{}
        // 如果请求参数为空，则列出远程 pin 服务中的所有 pin
        if len(req.Arguments) == 0 {
            // 调用 lsRemote 函数列出远程 pin 服务中的 pin，并返回 pin 列表和错误通道
            psCh, errCh, err := lsRemote(ctx, req, c)
            // 如果列出远程 pin 服务中的 pin 出错，直接返回错误
            if err != nil {
                return err
            }
            // 遍历 pin 列表通道，将 pin ID 添加到 rmIDs 中
            for ps := range psCh {
                rmIDs = append(rmIDs, ps.GetRequestId())
            }
            // 从错误通道中获取错误，如果有错误则返回错误信息
            if err = <-errCh; err != nil {
                return fmt.Errorf("error while listing remote pins: %v", err)
            }

            // 如果远程 pin ID 列表长度大于1且未设置强制删除选项，则返回错误信息
            if len(rmIDs) > 1 && !req.Options[pinForceOptionName].(bool) {
                return fmt.Errorf("multiple remote pins are matching this query, add --force to confirm the bulk removal")
            }
        } else {
            // 如果请求参数不为空，则返回意外的参数错误信息
            return fmt.Errorf("unexpected argument %q", req.Arguments[0])
        }

        // 遍历远程 pin ID 列表，逐个删除远程 pin
        for _, rmID := range rmIDs {
            if err := c.DeleteByID(ctx, rmID); err != nil {
                return fmt.Errorf("removing pin identified by requestid=%q failed: %v", rmID, err)
            }
        }
        // 操作成功，返回空错误
        return nil
    },
// 远程服务命令

// 定义添加远程固定服务的命令
var addRemotePinServiceCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline:          "Add remote pinning service.", // 简短描述
        ShortDescription: "Add credentials for access to a remote pinning service.", // 简要描述
        LongDescription: `
Add credentials for access to a remote pinning service and store them in the
config under Pinning.RemoteServices map.

TIP:

  To add services and test them by fetching pin count stats:

  $ ipfs pin remote service add goodsrv https://pin-api.example.com secret-key
  $ ipfs pin remote service add badsrv  https://bad-api.example.com invalid-key
  $ ipfs pin remote service ls --stat
  goodsrv   https://pin-api.example.com 0/0/0/0
  badsrv    https://bad-api.example.com invalid

`, // 详细描述
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.StringArg(pinServiceNameArgName, true, false, "Service name."), // 服务名称
        cmds.StringArg(pinServiceEndpointArgName, true, false, "Service endpoint."), // 服务端点
        cmds.StringArg(pinServiceKeyArgName, true, false, "Service key."), // 服务密钥
    },
    Type: nil, // 类型
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }
        # 打开存储库
        repo, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        # 延迟关闭存储库
        defer repo.Close()

        # 检查参数数量是否正确
        if len(req.Arguments) < 3 {
            return fmt.Errorf("expecting three arguments: service name, endpoint and key")
        }

        # 获取参数值
        name := req.Arguments[0]
        # 标准化端点
        endpoint, err := normalizeEndpoint(req.Arguments[1])
        if err != nil {
            return err
        }
        key := req.Arguments[2]

        # 获取存储库配置
        cfg, err := repo.Config()
        if err != nil {
            return err
        }
        # 检查远程服务是否存在
        if cfg.Pinning.RemoteServices != nil {
            if _, present := cfg.Pinning.RemoteServices[name]; present {
                return fmt.Errorf("service already present")
            }
        } else {
            cfg.Pinning.RemoteServices = map[string]config.RemotePinningService{}
        }

        # 设置远程服务配置
        cfg.Pinning.RemoteServices[name] = config.RemotePinningService{
            API: config.RemotePinningServiceAPI{
                Endpoint: endpoint,
                Key:      key,
            },
            Policies: config.RemotePinningServicePolicies{},
        }

        # 设置存储库配置
        return repo.SetConfig(cfg)
    },
// 定义一个命令，用于移除远程固定服务的凭据
var rmRemotePinServiceCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Remove remote pinning service.", // 命令简要描述
        ShortDescription: "Remove credentials for access to a remote pinning service.", // 命令的简短描述
    },
    Arguments: []cmds.Argument{ // 定义命令的参数
        cmds.StringArg(pinServiceNameOptionName, true, false, "Name of remote pinning service to remove."), // 远程固定服务的名称参数
    },
    Options: []cmds.Option{}, // 定义命令的选项
    Type:    nil, // 定义命令的类型
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 定义命令的执行函数
        cfgRoot, err := cmdenv.GetConfigRoot(env) // 获取配置根目录
        if err != nil {
            return err
        }
        repo, err := fsrepo.Open(cfgRoot) // 打开文件系统存储库
        if err != nil {
            return err
        }
        defer repo.Close() // 延迟关闭存储库

        if len(req.Arguments) != 1 { // 如果参数数量不等于1
            return fmt.Errorf("expecting one argument: name") // 返回错误信息
        }
        name := req.Arguments[0] // 获取参数值

        cfg, err := repo.Config() // 获取存储库配置
        if err != nil {
            return err
        }
        if cfg.Pinning.RemoteServices != nil { // 如果远程服务不为空
            delete(cfg.Pinning.RemoteServices, name) // 删除指定名称的远程服务
        }
        return repo.SetConfig(cfg) // 设置存储库配置
    },
}

// 定义一个命令，用于列出远程固定服务
var lsRemotePinServiceCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "List remote pinning services.", // 命令简要描述
        ShortDescription: "List remote pinning services.", // 命令的简短描述
        LongDescription: ` // 命令的长描述
List remote pinning services.

By default, only a name and an endpoint are listed; however, one can pass
'--stat' to test each endpoint by fetching pin counts for each state:

  $ ipfs pin remote service ls --stat
  goodsrv   https://pin-api.example.com 0/0/0/0
  badsrv    https://bad-api.example.com invalid

TIP: pass '--enc=json' for more useful JSON output.
`,
    },
    Arguments: []cmds.Argument{}, // 定义命令的参数
    Options: []cmds.Option{ // 定义命令的选项
        cmds.BoolOption(pinServiceStatOptionName, "Try to fetch and display current pin count on remote service (queued/pinning/pinned/failed).").WithDefault(false), // 尝试获取并显示远程服务的当前固定计数
    },
    Type: PinServicesList{}, // 定义命令的类型
}
    # 创建一个命令编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, list *PinServicesList) error {
            # 创建一个 tabwriter 对象，用于格式化输出
            tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
            # 检查是否需要包含统计信息
            withStat := req.Options[pinServiceStatOptionName].(bool)
            # 遍历远程服务列表
            for _, s := range list.RemoteServices {
                # 如果需要包含统计信息
                if withStat {
                    # 获取服务的状态信息
                    stat := s.Stat.Status
                    pc := s.Stat.PinCount
                    # 如果服务的 pinCount 不为空
                    if s.Stat.PinCount != nil {
                        # 格式化 pinCount 信息
                        stat = fmt.Sprintf("%d/%d/%d/%d", pc.Queued, pc.Pinning, pc.Pinned, pc.Failed)
                    }
                    # 将服务信息和统计信息写入 tabwriter
                    fmt.Fprintf(tw, "%s\t%s\t%s\n", s.Service, s.ApiEndpoint, stat)
                } else {
                    # 如果不需要包含统计信息，只将服务信息写入 tabwriter
                    fmt.Fprintf(tw, "%s\t%s\n", s.Service, s.ApiEndpoint)
                }
            }
            # 刷新 tabwriter 缓冲区
            tw.Flush()
            # 返回空值
            return nil
        }),
    },
// 定义 ServiceDetails 结构体，包含服务名称、API 端点和 Stat 结构体指针
type ServiceDetails struct {
    Service     string
    ApiEndpoint string //nolint
    Stat        *Stat  `json:",omitempty"` // 当未传入 --stat 参数时才存在
}

// 定义 Stat 结构体，包含状态和 PinCount 结构体指针
type Stat struct {
    Status   string
    PinCount *PinCount `json:",omitempty"` // 当传入 --stat 参数但服务离线时缺失
}

// 定义 PinCount 结构体，包含排队数、正在固定数、已固定数和失败数
type PinCount struct {
    Queued  int
    Pinning int
    Pinned  int
    Failed  int
}

// 定义由 ipfs pin remote service ls --enc=json | jq 返回的 PinServicesList 结构体
type PinServicesList struct {
    RemoteServices []ServiceDetails
}

// 实现 Len 方法，返回 RemoteServices 切片的长度
func (l PinServicesList) Len() int {
    return len(l.RemoteServices)
}

// 实现 Swap 方法，交换 RemoteServices 切片中的两个元素
func (l PinServicesList) Swap(i, j int) {
    s := l.RemoteServices
    s[i], s[j] = s[j], s[i]
}

// 实现 Less 方法，比较 RemoteServices 切片中的两个元素
func (l PinServicesList) Less(i, j int) bool {
    s := l.RemoteServices
    return s[i].Service < s[j].Service
}

// 从请求中获取远程固定服务的信息
func getRemotePinServiceFromRequest(req *cmds.Request, env cmds.Environment) (*pinclient.Client, error) {
    service, serviceFound := req.Options[pinServiceNameOptionName]
    if !serviceFound {
        return nil, fmt.Errorf("a service name must be passed")
    }

    serviceStr := service.(string)
    var err error
    c, err := getRemotePinService(env, serviceStr)
    if err != nil {
        return nil, err
    }

    return c, nil
}

// 获取远程固定服务的信息
func getRemotePinService(env cmds.Environment, name string) (*pinclient.Client, error) {
    if name == "" {
        return nil, fmt.Errorf("remote pinning service name not specified")
    }
    endpoint, key, err := getRemotePinServiceInfo(env, name)
    if err != nil {
        return nil, err
    }
    return pinclient.NewClient(endpoint, key), nil
}

// 获取远程固定服务的端点和密钥信息
func getRemotePinServiceInfo(env cmds.Environment, name string) (endpoint, key string, err error) {
    cfgRoot, err := cmdenv.GetConfigRoot(env)
    if err != nil {
        return "", "", err
    }
    repo, err := fsrepo.Open(cfgRoot)
    if err != nil {
        return "", "", err
    }
    defer repo.Close()
    cfg, err := repo.Config()
}
    # 如果错误不为空，则返回空字符串和错误
    if err != nil {
        return "", "", err
    }
    # 如果远程服务未知，则返回空字符串和错误
    if cfg.Pinning.RemoteServices == nil {
        return "", "", fmt.Errorf("service not known")
    }
    # 从配置中获取指定名称的远程服务
    service, present := cfg.Pinning.RemoteServices[name]
    # 如果远程服务不存在，则返回空字符串和错误
    if !present {
        return "", "", fmt.Errorf("service not known")
    }
    # 标准化远程服务的 API 端点
    endpoint, err = normalizeEndpoint(service.API.Endpoint)
    # 如果标准化 API 端点出错，则返回空字符串和错误
    if err != nil {
        return "", "", err
    }
    # 返回标准化后的 API 端点和服务的 API 密钥
    return endpoint, service.API.Key, nil
// 标准化端点，确保其符合要求的格式
func normalizeEndpoint(endpoint string) (string, error) {
    // 解析端点字符串为 URI 对象
    uri, err := neturl.ParseRequestURI(endpoint)
    // 如果解析失败或者协议不是 http 或 https，则返回错误
    if err != nil || !(uri.Scheme == "http" || uri.Scheme == "https") {
        return "", fmt.Errorf("service endpoint must be a valid HTTP URL")
    }

    // 清理末尾和重复的斜杠
    uri.Path = gopath.Clean(uri.Path)
    uri.Path = strings.TrimSuffix(uri.Path, ".")
    uri.Path = strings.TrimSuffix(uri.Path, "/")

    // 移除任何查询参数
    if uri.RawQuery != "" {
        return "", fmt.Errorf("service endpoint should be provided without any query parameters")
    }

    // 如果路径以 "/pins" 结尾，则返回错误
    if strings.HasSuffix(uri.Path, "/pins") {
        return "", fmt.Errorf("service endpoint should be provided without the /pins suffix")
    }

    // 返回标准化后的端点字符串
    return uri.String(), nil
}
```