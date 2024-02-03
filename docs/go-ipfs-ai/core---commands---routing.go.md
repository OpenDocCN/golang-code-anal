# `kubo\core\commands\routing.go`

```go
package commands

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "encoding/base64"  // base64 编码解码包
    "errors"  // 错误处理包
    "fmt"  // 格式化包，用于打印输出
    "io"  // 输入输出包
    "strings"  // 字符串处理包
    "time"  // 时间包

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入自定义包

    dag "github.com/ipfs/boxo/ipld/merkledag"  // 导入自定义包
    "github.com/ipfs/boxo/ipns"  // 导入自定义包
    cid "github.com/ipfs/go-cid"  // 导入自定义包
    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入自定义包
    ipld "github.com/ipfs/go-ipld-format"  // 导入自定义包
    iface "github.com/ipfs/kubo/core/coreiface"  // 导入自定义包
    "github.com/ipfs/kubo/core/coreiface/options"  // 导入自定义包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入自定义包
    routing "github.com/libp2p/go-libp2p/core/routing"  // 导入自定义包
)

var errAllowOffline = errors.New("can't put while offline: pass `--allow-offline` to override")  // 定义一个错误变量

const (
    dhtVerboseOptionName   = "verbose"  // 定义常量
    numProvidersOptionName = "num-providers"  // 定义常量
    allowOfflineOptionName = "allow-offline"  // 定义常量
)

var RoutingCmd = &cmds.Command{  // 定义命令
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline:          "Issue routing commands.",  // 命令的简短描述
        ShortDescription: ``,  // 命令的详细描述
    },

    Subcommands: map[string]*cmds.Command{  // 子命令映射
        "findprovs": findProvidersRoutingCmd,  // 查找提供者的子命令
        "findpeer":  findPeerRoutingCmd,  // 查找对等节点的子命令
        "get":       getValueRoutingCmd,  // 获取值的子命令
        "put":       putValueRoutingCmd,  // 存储值的子命令
        "provide":   provideRefRoutingCmd,  // 提供引用的子命令
    },
}

var findProvidersRoutingCmd = &cmds.Command{  // 查找提供者的命令
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline:          "Find peers that can provide a specific value, given a key.",  // 命令的简短描述
        ShortDescription: "Outputs a list of newline-delimited provider Peer IDs.",  // 命令的详细描述
    },

    Arguments: []cmds.Argument{  // 命令的参数
        cmds.StringArg("key", true, true, "The key to find providers for."),  // 字符串参数
    },
    Options: []cmds.Option{  // 命令的选项
        cmds.BoolOption(dhtVerboseOptionName, "v", "Print extra information."),  // 布尔选项
        cmds.IntOption(numProvidersOptionName, "n", "The number of providers to find.").WithDefault(20),  // 整数选项
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 如果节点不在线，则返回错误
        if !n.IsOnline {
            return ErrNotOnline
        }

        # 从请求选项中获取提供者数量
        numProviders, _ := req.Options[numProvidersOptionName].(int)
        # 如果提供者数量小于1，则返回错误
        if numProviders < 1 {
            return fmt.Errorf("number of providers must be greater than 0")
        }

        # 解析请求参数为CID
        c, err := cid.Parse(req.Arguments[0])
        if err != nil {
            return err
        }

        # 创建一个带有取消功能的上下文
        ctx, cancel := context.WithCancel(req.Context)
        # 注册查询事件的上下文和事件通道
        ctx, events := routing.RegisterForQueryEvents(ctx)

        # 启动一个 goroutine 处理异步查找提供者的操作
        go func() {
            defer cancel()
            # 从节点的路由表中异步查找提供者
            pchan := n.Routing.FindProvidersAsync(ctx, c, numProviders)
            for p := range pchan {
                np := p
                # 发布查询事件，包括提供者信息
                routing.PublishQueryEvent(ctx, &routing.QueryEvent{
                    Type:      routing.Provider,
                    Responses: []*peer.AddrInfo{&np},
                })
            }
        }()
        # 处理查询事件并将结果发送给响应处理器
        for e := range events {
            if err := res.Emit(e); err != nil {
                return err
            }
        }

        # 返回空错误，表示处理成功
        return nil
    },
    # 定义 Encoders，使用 cmds.EncoderMap 类型
    Encoders: cmds.EncoderMap{
        # 将 cmds.Text 与一个函数绑定，该函数接收请求、写入器和查询事件，并返回错误
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *routing.QueryEvent) error {
            # 定义 pfm 变量，类型为 pfuncMap，包含两个函数
            pfm := pfuncMap{
                # 当查询事件为 FinalPeer 时执行的函数
                routing.FinalPeer: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 如果 verbose 为真，则向输出写入最接近的对等点的信息
                    if verbose {
                        fmt.Fprintf(out, "* closest peer %s\n", obj.ID)
                    }
                    return nil
                },
                # 当查询事件为 Provider 时执行的函数
                routing.Provider: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 获取响应中的第一个提供者
                    prov := obj.Responses[0]
                    # 如果 verbose 为真，则向输出写入 "provider: "
                    if verbose {
                        fmt.Fprintf(out, "provider: ")
                    }
                    # 向输出写入提供者的 ID
                    fmt.Fprintf(out, "%s\n", prov.ID)
                    # 如果 verbose 为真，则遍历提供者的地址列表，向输出写入每个地址
                    if verbose {
                        for _, a := range prov.Addrs {
                            fmt.Fprintf(out, "\t%s\n", a)
                        }
                    }
                    return nil
                },
            }

            # 从请求选项中获取 dhtVerboseOptionName 对应的值，并将其转换为布尔类型
            verbose, _ := req.Options[dhtVerboseOptionName].(bool)
            # 调用 printEvent 函数，将输出、写入器、verbose 标志和 pfm 作为参数
            return printEvent(out, w, verbose, pfm)
        }),
    },
    # 定义 Type，类型为 routing.QueryEvent
    Type: routing.QueryEvent{},
# 定义常量，递归选项的名称为"recursive"
const (
    recursiveOptionName = "recursive"
)

# 定义命令对象provideRefRoutingCmd
var provideRefRoutingCmd = &cmds.Command{
    # 命令状态为实验性
    Status: cmds.Experimental,
    # 命令的帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Announce to the network that you are providing given values.",
    },

    # 命令的参数列表，包含一个字符串参数key，必填，支持多个值，启用标准输入
    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, true, "The key[s] to send provide records for.").EnableStdin(),
    },
    # 命令的选项列表，包含一个布尔选项dhtVerboseOptionName，简写为"v"，用于打印额外信息；一个布尔选项recursiveOptionName，简写为"r"，用于递归提供整个图
    Options: []cmds.Option{
        cmds.BoolOption(dhtVerboseOptionName, "v", "Print extra information."),
        cmds.BoolOption(recursiveOptionName, "r", "Recursively provide entire graph."),
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 如果节点不在线，则返回错误
        if !nd.IsOnline {
            return ErrNotOnline
        }

        # 如果节点没有连接的对等节点，则返回错误
        if len(nd.PeerHost.Network().Conns()) == 0 {
            return errors.New("cannot provide, no connected peers")
        }

        # 解析标准输入参数
        # TODO: 惰性加载
        err = req.ParseBodyArgs()
        if err != nil {
            return err
        }

        # 从请求选项中获取递归标志
        rec, _ := req.Options[recursiveOptionName].(bool)

        # 创建空的 CID 数组
        var cids []cid.Cid
        # 遍历请求参数中的 CID
        for _, arg := range req.Arguments {
            # 解码 CID
            c, err := cid.Decode(arg)
            if err != nil {
                return err
            }

            # 检查节点的块存储中是否存在该 CID
            has, err := nd.Blockstore.Has(req.Context, c)
            if err != nil {
                return err
            }

            # 如果不存在该 CID，则返回错误
            if !has {
                return fmt.Errorf("block %s not found locally, cannot provide", c)
            }

            # 将 CID 添加到数组中
            cids = append(cids, c)
        }

        # 创建上下文和取消函数
        ctx, cancel := context.WithCancel(req.Context)
        # 注册查询事件
        ctx, events := routing.RegisterForQueryEvents(ctx)

        # 声明提供错误变量
        var provideErr error
        # 异步执行提供操作
        go func() {
            defer cancel()
            # 如果是递归提供，则调用递归提供函数，否则调用普通提供函数
            if rec {
                provideErr = provideKeysRec(ctx, nd.Routing, nd.DAG, cids)
            } else {
                provideErr = provideKeys(ctx, nd.Routing, cids)
            }
            # 如果提供操作出现错误，则发布查询事件
            if provideErr != nil {
                routing.PublishQueryEvent(ctx, &routing.QueryEvent{
                    Type:  routing.QueryError,
                    Extra: provideErr.Error(),
                })
            }
        }()

        # 遍历事件并发送到结果流
        for e := range events {
            if err := res.Emit(e); err != nil {
                return err
            }
        }

        # 返回提供操作的错误
        return provideErr
    },
    # 创建一个名为Encoders的映射，将命令类型与编码器函数关联起来
    Encoders: cmds.EncoderMap{
        # 将文本命令与特定的编码器函数关联起来
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *routing.QueryEvent) error {
            # 创建一个函数映射，将最终对等点与处理函数关联起来
            pfm := pfuncMap{
                routing.FinalPeer: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 如果verbose为真，则向输出流写入提供者记录发送到对等点的消息
                    if verbose {
                        fmt.Fprintf(out, "sending provider record to peer %s\n", obj.ID)
                    }
                    return nil
                },
            }
            # 从请求的选项中获取verbose参数的值
            verbose, _ := req.Options[dhtVerboseOptionName].(bool)
            # 调用printEvent函数，将事件输出到指定的流中
            return printEvent(out, w, verbose, pfm)
        }),
    },
    # 创建一个类型为routing.QueryEvent的结构体实例
    Type: routing.QueryEvent{},
# 提供给定 CID 列表的数据给路由系统
func provideKeys(ctx context.Context, r routing.Routing, cids []cid.Cid) error:
    # 遍历给定的 CID 列表
    for _, c := range cids:
        # 调用路由系统的提供方法，将数据提供给其他节点
        err := r.Provide(ctx, c, true)
        # 如果出现错误，返回错误信息
        if err != nil:
            return err
    # 如果没有错误，返回空
    return nil

# 递归地提供给定 CID 列表的数据给路由系统
func provideKeysRec(ctx context.Context, r routing.Routing, dserv ipld.DAGService, cids []cid.Cid) error:
    # 创建一个已提供的 CID 集合
    provided := cid.NewSet()
    # 遍历给定的 CID 列表
    for _, c := range cids:
        # 创建一个新的 CID 集合
        kset := cid.NewSet()
        # 使用 DAG 服务获取直接链接，并遍历 CID 的链接
        err := dag.Walk(ctx, dag.GetLinksDirect(dserv), c, kset.Visit)
        # 如果出现错误，返回错误信息
        if err != nil:
            return err
        # 遍历链接集合中的键
        for _, k := range kset.Keys():
            # 如果已经提供过该键，继续下一个循环
            if provided.Has(k):
                continue
            # 调用路由系统的提供方法，将数据提供给其他节点
            err = r.Provide(ctx, k, true)
            # 如果出现错误，返回错误信息
            if err != nil:
                return err
            # 将已提供的键添加到集合中
            provided.Add(k)
    # 如果没有错误，返回空
    return nil

# 定义一个查找对等节点路由命令
var findPeerRoutingCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Find the multiaddresses associated with a Peer ID.",
        ShortDescription: "Outputs a list of newline-delimited multiaddresses.",
    },
    # 定义命令的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("peerID", true, true, "The ID of the peer to search for."),
    },
    # 定义命令的选项
    Options: []cmds.Option{
        cmds.BoolOption(dhtVerboseOptionName, "v", "Print extra information."),
    },
    # Run 函数处理命令请求，接收请求、发送响应、环境变量
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境变量中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 如果节点不在线，返回错误
        if !nd.IsOnline {
            return ErrNotOnline
        }

        # 解码请求参数中的对等节点 ID
        pid, err := peer.Decode(req.Arguments[0])
        if err != nil {
            return err
        }

        # 如果对等节点 ID 与当前节点 ID 相同，返回错误
        if pid == nd.Identity {
            return ErrSelfUnsupported
        }

        # 创建一个带有取消功能的上下文
        ctx, cancel := context.WithCancel(req.Context)
        # 注册查询事件
        ctx, events := routing.RegisterForQueryEvents(ctx)

        # 定义查找对等节点错误
        var findPeerErr error
        # 异步执行查找对等节点的操作
        go func() {
            defer cancel()
            var pi peer.AddrInfo
            pi, findPeerErr = nd.Routing.FindPeer(ctx, pid)
            if findPeerErr != nil {
                # 发布查询错误事件
                routing.PublishQueryEvent(ctx, &routing.QueryEvent{
                    Type:  routing.QueryError,
                    Extra: findPeerErr.Error(),
                })
                return
            }

            # 发布最终对等节点事件
            routing.PublishQueryEvent(ctx, &routing.QueryEvent{
                Type:      routing.FinalPeer,
                Responses: []*peer.AddrInfo{&pi},
            })
        }()

        # 循环处理查询事件
        for e := range events {
            if err := res.Emit(e); err != nil {
                return err
            }
        }

        # 返回查找对等节点的错误
        return findPeerErr
    },
    # 编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *routing.QueryEvent) error {
            # 定义打印事件的函数映射
            pfm := pfuncMap{
                routing.FinalPeer: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    pi := obj.Responses[0]
                    for _, a := range pi.Addrs {
                        fmt.Fprintf(out, "%s\n", a)
                    }
                    return nil
                },
            }

            # 获取 DHT 详细信息选项
            verbose, _ := req.Options[dhtVerboseOptionName].(bool)
            return printEvent(out, w, verbose, pfm)
        }),
    },
    # 创建一个名为QueryEvent的routing类型的空对象
    Type: routing.QueryEvent{},
# 定义一个名为 getValueRoutingCmd 的命令对象
var getValueRoutingCmd = &cmds.Command{
    # 命令对象的状态为实验性
    Status: cmds.Experimental,
    # 命令对象的帮助文本
    Helptext: cmds.HelpText{
        # 命令对象的简短描述
        Tagline: "Given a key, query the routing system for its best value.",
        # 命令对象的详细描述
        ShortDescription: `
Outputs the best value for the given key.

There may be several different values for a given key stored in the routing
system; in this context 'best' means the record that is most desirable. There is
no one metric for 'best': it depends entirely on the key type. For IPNS, 'best'
is the record that is both valid and has the highest sequence number (freshest).
Different key types can specify other 'best' rules.
`,
    },
    # 命令对象的参数
    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, true, "The key to find a value for."),
    },
    # 命令对象的运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        # 调用路由系统的 Get 方法获取给定 key 的最佳值
        r, err := api.Routing().Get(req.Context, req.Arguments[0])
        if err != nil {
            return err
        }
        # 发送查询事件到响应流
        return res.Emit(routing.QueryEvent{
            Extra: base64.StdEncoding.EncodeToString(r),
            Type:  routing.Value,
        })
    },
    # 命令对象的编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, obj *routing.QueryEvent) error {
            res, err := base64.StdEncoding.DecodeString(obj.Extra)
            if err != nil {
                return err
            }
            _, err = w.Write(res)
            return err
        }),
    },
    # 命令对象的类型
    Type: routing.QueryEvent{},
}

# 定义一个名为 putValueRoutingCmd 的命令对象
var putValueRoutingCmd = &cmds.Command{
    # 命令对象的状态为实验性
    Status: cmds.Experimental,
    # 命令对象的帮助文本
    Helptext: cmds.HelpText{
        # 命令对象的简短描述
        Tagline: "Write a key/value pair to the routing system.",
        # 命令对象的详细描述
        ShortDescription: `
Given a key of the form /foo/bar and a valid value for that key, this will write
that value to the routing system with that key.
`,
    },
}
# 键有两部分：键类型（foo）和键名（bar）。IPNS 使用 /ipns 键类型，并期望键名是对等节点 ID。IPNS 条目具体格式化（协议缓冲区）。
# 只能使用 ipfs 二进制文件支持的键类型：目前只有 /ipns。除非您对 go-ipfs 路由内部有相对深入的了解，否则您可能希望使用 'ipfs name publish' 而不是这个。
# 值必须是给定键类型的有效值。例如，如果键是 /ipns/QmFoo，则值必须是使用 QmFoo 标识的键签名的 IPNS 记录（协议缓冲区）。



    Arguments: []cmds.Argument{
        cmds.StringArg("key", true, false, "The key to store the value at."),
        cmds.FileArg("value-file", true, false, "A path to a file containing the value to store.").EnableStdin(),
    },



    Options: []cmds.Option{
        cmds.BoolOption(allowOfflineOptionName, "When offline, save the IPNS record to the the local datastore without broadcasting to the network instead of simply failing."),
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 从请求的文件中获取文件对象
        file, err := cmdenv.GetFileArg(req.Files.Entries())
        if err != nil {
            return err
        }
        # 延迟关闭文件
        defer file.Close()

        # 读取文件中的数据
        data, err := io.ReadAll(file)
        if err != nil {
            return err
        }

        # 从请求的选项中获取是否允许离线操作
        allowOffline, _ := req.Options[allowOfflineOptionName].(bool)

        # 设置路由操作的选项
        opts := []options.RoutingPutOption{
            options.Put.AllowOffline(allowOffline),
        }

        # 从请求参数中获取 IPNS 名称
        ipnsName, err := ipns.NameFromString(req.Arguments[0])
        if err != nil {
            return err
        }

        # 执行路由操作，将数据放入路由系统
        err = api.Routing().Put(req.Context, req.Arguments[0], data, opts...)
        if err != nil {
            # 如果出现离线错误，则返回允许离线的错误
            if err == iface.ErrOffline {
                err = errAllowOffline
            }
            return err
        }

        # 返回路由查询事件
        return res.Emit(routing.QueryEvent{
            Type: routing.Value,
            ID:   ipnsName.Peer(),
        })
    },
    # 编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *routing.QueryEvent) error {
            # 定义路由查询事件的处理函数映射
            pfm := pfuncMap{
                routing.FinalPeer: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 如果需要详细信息，则输出最接近的对等点
                    if verbose {
                        fmt.Fprintf(out, "* closest peer %s\n", obj.ID)
                    }
                    return nil
                },
                routing.Value: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 输出对等点的 ID
                    fmt.Fprintf(out, "%s\n", obj.ID)
                    return nil
                },
            }

            # 从请求选项中获取是否详细输出信息
            verbose, _ := req.Options[dhtVerboseOptionName].(bool)

            # 打印事件信息
            return printEvent(out, w, verbose, pfm)
        }),
    },
    # 路由查询事件类型
    Type: routing.QueryEvent{},
}

这是一个类型定义的结束标记


type (
    printFunc func(obj *routing.QueryEvent, out io.Writer, verbose bool) error
    pfuncMap  map[routing.QueryEventType]printFunc
)

定义了两个类型：printFunc 和 pfuncMap


func printEvent(obj *routing.QueryEvent, out io.Writer, verbose bool, override pfuncMap) error {

定义了一个名为printEvent的函数，接受routing.QueryEvent类型的obj、io.Writer类型的out、bool类型的verbose和pfuncMap类型的override参数，并返回error类型的结果


if verbose {
    fmt.Fprintf(out, "%s: ", time.Now().Format("15:04:05.000"))
}

如果verbose为true，则向out写入当前时间的格式化字符串


if override != nil {
    if pf, ok := override[obj.Type]; ok {
        return pf(obj, out, verbose)
    }
}

如果override不为nil，且override[obj.Type]存在，则调用override[obj.Type]函数并返回结果


switch obj.Type {
case routing.SendingQuery:
    if verbose {
        fmt.Fprintf(out, "* querying %s\n", obj.ID)
    }
case routing.Value:
    if verbose {
        fmt.Fprintf(out, "got value: '%s'\n", obj.Extra)
    } else {
        fmt.Fprint(out, obj.Extra)
    }
case routing.PeerResponse:
    if verbose {
        fmt.Fprintf(out, "* %s says use ", obj.ID)
        for _, p := range obj.Responses {
            fmt.Fprintf(out, "%s ", p.ID)
        }
        fmt.Fprintln(out)
    }
case routing.QueryError:
    if verbose {
        fmt.Fprintf(out, "error: %s\n", obj.Extra)
    }
case routing.DialingPeer:
    if verbose {
        fmt.Fprintf(out, "dialing peer: %s\n", obj.ID)
    }
case routing.AddingPeer:
    if verbose {
        fmt.Fprintf(out, "adding peer to query: %s\n", obj.ID)
    }
case routing.FinalPeer:
default:
    if verbose {
        fmt.Fprintf(out, "unrecognized event type: %d\n", obj.Type)
    }
}

根据obj.Type的不同进行不同的处理


return nil

返回nil


func escapeDhtKey(s string) (string, error) {

定义了一个名为escapeDhtKey的函数，接受string类型的s参数，并返回string类型和error类型的结果


parts := strings.Split(s, "/")
if len(parts) != 3 ||
    parts[0] != "" ||
    !(parts[1] == "ipns" || parts[1] == "pk") {
    return "", errors.New("invalid key")
}

将s按"/"分割成多个部分，如果部分数量不等于3，或者第一个部分不为空，或者第二个部分不是"ipns"或"pk"，则返回一个错误


k, err := peer.Decode(parts[2])
if err != nil {
    return "", err
}

使用peer.Decode对parts[2]进行解码，如果出现错误则返回该错误


return strings.Join(append(parts[:2], string(k)), "/"), nil

将parts[:2]和string(k)连接成一个字符串并返回，同时返回nil作为错误结果
```