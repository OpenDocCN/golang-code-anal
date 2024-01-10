# `kubo\core\commands\dht.go`

```
package commands

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "errors"   // 错误处理包，用于处理错误信息
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，用于处理输入输出流

    cmds "github.com/ipfs/go-ipfs-cmds"  // IPFS 命令包，用于处理 IPFS 命令
    "github.com/ipfs/kubo/core/commands/cmdenv"  // IPFS Kubo 核心命令包，用于处理 IPFS Kubo 核心命令
    peer "github.com/libp2p/go-libp2p/core/peer"  // libp2p 核心节点包，用于处理 libp2p 核心节点
    routing "github.com/libp2p/go-libp2p/core/routing"  // libp2p 核心路由包，用于处理 libp2p 核心路由
)

var ErrNotDHT = errors.New("routing service is not a DHT")  // 定义一个错误变量，表示路由服务不是 DHT

var DhtCmd = &cmds.Command{  // 定义一个命令对象
    Helptext: cmds.HelpText{  // 命令的帮助文本
        Tagline:          "Issue commands directly through the DHT.",  // 命令的简短描述
        ShortDescription: ``,  // 命令的简短描述
    },

    Subcommands: map[string]*cmds.Command{  // 子命令映射
        "query":     queryDhtCmd,  // 查询 DHT 命令
        "findprovs": findProvidersDhtCmd,  // 查找提供者 DHT 命令
        "findpeer":  findPeerDhtCmd,  // 查找节点 DHT 命令
        "get":       getValueDhtCmd,  // 获取值 DHT 命令
        "put":       putValueDhtCmd,  // 存储值 DHT 命令
        "provide":   provideRefDhtCmd,  // 提供引用 DHT 命令
    },
}

var findProvidersDhtCmd = &cmds.Command{  // 查找提供者 DHT 命令
    Helptext:  findProvidersRoutingCmd.Helptext,  // 帮助文本
    Arguments: findProvidersRoutingCmd.Arguments,  // 参数
    Options:   findProvidersRoutingCmd.Options,  // 选项
    Run:       findProvidersRoutingCmd.Run,  // 运行
    Encoders:  findProvidersRoutingCmd.Encoders,  // 编码器
    Type:      findProvidersRoutingCmd.Type,  // 类型
    Status:    cmds.Deprecated,  // 状态为已弃用
}

var findPeerDhtCmd = &cmds.Command{  // 查找节点 DHT 命令
    Helptext:  findPeerRoutingCmd.Helptext,  // 帮助文本
    Arguments: findPeerRoutingCmd.Arguments,  // 参数
    Options:   findPeerRoutingCmd.Options,  // 选项
    Run:       findPeerRoutingCmd.Run,  // 运行
    Encoders:  findPeerRoutingCmd.Encoders,  // 编码器
    Type:      findPeerRoutingCmd.Type,  // 类型
    Status:    cmds.Deprecated,  // 状态为已弃用
}

var getValueDhtCmd = &cmds.Command{  // 获取值 DHT 命令
    Helptext:  getValueRoutingCmd.Helptext,  // 帮助文本
    Arguments: getValueRoutingCmd.Arguments,  // 参数
    Options:   getValueRoutingCmd.Options,  // 选项
    Run:       getValueRoutingCmd.Run,  // 运行
    Encoders:  getValueRoutingCmd.Encoders,  // 编码器
    Type:      getValueRoutingCmd.Type,  // 类型
    Status:    cmds.Deprecated,  // 状态为已弃用
}

var putValueDhtCmd = &cmds.Command{  // 存储值 DHT 命令
    Helptext:  putValueRoutingCmd.Helptext,  // 帮助文本
    Arguments: putValueRoutingCmd.Arguments,  // 参数
    Options:   putValueRoutingCmd.Options,  // 选项
    Run:       putValueRoutingCmd.Run,  // 运行
    # 将 putValueRoutingCmd.Encoders 赋值给 Encoders
    Encoders:  putValueRoutingCmd.Encoders,
    # 将 putValueRoutingCmd.Type 赋值给 Type
    Type:      putValueRoutingCmd.Type,
    # 将 cmds.Deprecated 赋值给 Status
    Status:    cmds.Deprecated,
// 创建一个名为 provideRefDhtCmd 的命令对象，继承自 cmds.Command
var provideRefDhtCmd = &cmds.Command{
    // 设置帮助文本为 provideRefRoutingCmd 的帮助文本
    Helptext:  provideRefRoutingCmd.Helptext,
    // 设置参数为 provideRefRoutingCmd 的参数
    Arguments: provideRefRoutingCmd.Arguments,
    // 设置选项为 provideRefRoutingCmd 的选项
    Options:   provideRefRoutingCmd.Options,
    // 设置运行函数为 provideRefRoutingCmd 的运行函数
    Run:       provideRefRoutingCmd.Run,
    // 设置编码器为 provideRefRoutingCmd 的编码器
    Encoders:  provideRefRoutingCmd.Encoders,
    // 设置类型为 provideRefRoutingCmd 的类型
    Type:      provideRefRoutingCmd.Type,
    // 设置状态为 Deprecated
    Status:    cmds.Deprecated,
}

// 定义 kademlia 接口，扩展了 routing 接口，包含一个用于获取最接近目标的对等节点的命令
type kademlia interface {
    routing.Routing
    GetClosestPeers(ctx context.Context, key string) ([]peer.ID, error)
}

// 创建一个名为 queryDhtCmd 的命令对象，继承自 cmds.Command
var queryDhtCmd = &cmds.Command{
    // 设置帮助文本
    Helptext: cmds.HelpText{
        Tagline:          "Find the closest Peer IDs to a given Peer ID by querying the DHT.",
        ShortDescription: "Outputs a list of newline-delimited Peer IDs.",
    },
    // 设置参数
    Arguments: []cmds.Argument{
        cmds.StringArg("peerID", true, true, "The peerID to run the query against."),
    },
    // 设置选项
    Options: []cmds.Option{
        cmds.BoolOption(dhtVerboseOptionName, "v", "Print extra information."),
    },
}
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 如果节点的 DHT 客户端为空，返回错误
        if nd.DHTClient == nil {
            return ErrNotDHT
        }

        # 解析请求参数中的对等节点 ID
        id, err := peer.Decode(req.Arguments[0])
        if err != nil {
            return cmds.ClientError("invalid peer ID")
        }

        # 创建一个带有取消函数的上下文
        ctx, cancel := context.WithCancel(req.Context)
        # 延迟调用取消函数
        defer cancel()
        # 注册查询事件的上下文
        ctx, events := routing.RegisterForQueryEvents(ctx)

        # 获取节点的 DHT 客户端
        client := nd.DHTClient
        # 如果客户端等于节点的 DHT，则将客户端设置为节点的 DHT 的广域网客户端
        if client == nd.DHT {
            client = nd.DHT.WAN
            # 如果节点的 DHT 的广域网客户端不活跃，则将客户端设置为节点的 DHT 的局域网客户端
            if !nd.DHT.WANActive() {
                client = nd.DHT.LAN
            }
        }

        # 如果客户端不是 kademlia 类型，返回错误
        if d, ok := client.(kademlia); !ok {
            return fmt.Errorf("dht client does not support GetClosestPeers")
        } else {
            # 创建一个错误通道
            errCh := make(chan error, 1)
            # 启动一个 goroutine
            go func() {
                defer close(errCh)
                defer cancel()
                # 获取最接近对等节点的节点列表
                closestPeers, err := d.GetClosestPeers(ctx, string(id))
                # 遍历最接近的节点列表，发布查询事件
                for _, p := range closestPeers {
                    routing.PublishQueryEvent(ctx, &routing.QueryEvent{
                        ID:   p,
                        Type: routing.FinalPeer,
                    })
                }

                # 如果发生错误，将错误发送到错误通道
                if err != nil {
                    errCh <- err
                    return
                }
            }()

            # 遍历事件通道，将事件发送给响应处理器
            for e := range events {
                if err := res.Emit(e); err != nil {
                    return err
                }
            }

            # 返回错误通道的结果
            return <-errCh
        }
    },
    # 创建 Encoders 对象，包含不同类型命令的编码器
    Encoders: cmds.EncoderMap{
        # 文本类型命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *routing.QueryEvent) error {
            # 创建 pfm 对象，包含不同类型的 pfuncMap
            pfm := pfuncMap{
                # 最终对等节点的 pfuncMap
                routing.FinalPeer: func(obj *routing.QueryEvent, out io.Writer, verbose bool) error {
                    # 将最终对等节点的 ID 写入输出流
                    fmt.Fprintf(out, "%s\n", obj.ID)
                    # 返回空值
                    return nil
                },
            }
            # 从请求选项中获取 dhtVerboseOptionName 对应的值，转换为布尔类型
            verbose, _ := req.Options[dhtVerboseOptionName].(bool)
            # 调用 printEvent 函数，打印事件信息
            return printEvent(out, w, verbose, pfm)
        }),
    },
    # 定义 Type 为 routing.QueryEvent 类型的空对象
    Type: routing.QueryEvent{},
# 闭合前面的函数定义
```