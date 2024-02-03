# `kubo\core\commands\p2p.go`

```go
package commands

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "errors"   // 错误处理包，用于处理错误信息
    "fmt"      // 格式化包，用于格式化输出
    "io"       // 输入输出包，用于处理输入输出流
    "strconv"  // 字符串转换包，用于字符串和数字之间的转换
    "strings"  // 字符串处理包，用于处理字符串
    "text/tabwriter"  // 文本格式化包，用于格式化输出文本
    "time"     // 时间包，用于处理时间

    core "github.com/ipfs/kubo/core"  // 导入名为 core 的包，并指定别名
    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入名为 cmdenv 的包
    p2p "github.com/ipfs/kubo/p2p"  // 导入名为 p2p 的包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入名为 cmds 的包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入名为 peer 的包
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入名为 pstore 的包
    protocol "github.com/libp2p/go-libp2p/core/protocol"  // 导入名为 protocol 的包
    ma "github.com/multiformats/go-multiaddr"  // 导入名为 ma 的包
    madns "github.com/multiformats/go-multiaddr-dns"  // 导入名为 madns 的包
)

// P2PProtoPrefix is the default required prefix for protocol names
const P2PProtoPrefix = "/x/"  // 定义常量 P2PProtoPrefix，表示协议名称的默认前缀

// P2PListenerInfoOutput is output type of ls command
type P2PListenerInfoOutput struct {
    Protocol      string  // 监听器信息输出类型的协议字段
    ListenAddress string  // 监听器信息输出类型的监听地址字段
    TargetAddress string  // 监听器信息输出类型的目标地址字段
}

// P2PStreamInfoOutput is output type of streams command
type P2PStreamInfoOutput struct {
    HandlerID     string  // 流信息输出类型的处理器 ID 字段
    Protocol      string  // 流信息输出类型的协议字段
    OriginAddress string  // 流信息输出类型的源地址字段
    TargetAddress string  // 流信息输出类型的目标地址字段
}

// P2PLsOutput is output type of ls command
type P2PLsOutput struct {
    Listeners []P2PListenerInfoOutput  // ls 命令的输出类型包含监听器信息输出类型的切片
}

// P2PStreamsOutput is output type of streams command
type P2PStreamsOutput struct {
    Streams []P2PStreamInfoOutput  // streams 命令的输出类型包含流信息输出类型的切片
}

const (
    allowCustomProtocolOptionName = "allow-custom-protocol"  // 允许自定义协议选项的名称
    reportPeerIDOptionName        = "report-peer-id"  // 报告对等节点 ID 选项的名称
)

var resolveTimeout = 10 * time.Second  // 解析超时时间为 10 秒

// P2PCmd is the 'ipfs p2p' command
var P2PCmd = &cmds.Command{  // 定义 P2PCmd 变量为 cmds.Command 类型
    Status: cmds.Experimental,  // 设置状态为实验性
    Helptext: cmds.HelpText{  // 设置帮助文本
        Tagline: "Libp2p stream mounting.",  // 简短描述
        ShortDescription: `  // 短描述
Create and use tunnels to remote peers over libp2p

Note: this command is experimental and subject to change as usecases and APIs
are refined`,
    },

    Subcommands: map[string]*cmds.Command{  // 子命令映射
        "stream":  p2pStreamCmd,  // 流命令
        "forward": p2pForwardCmd,  // 转发命令
        "listen":  p2pListenCmd,  // 监听命令
        "close":   p2pCloseCmd,  // 关闭命令
        "ls":      p2pLsCmd,  // ls 命令
    },
}
// 定义 p2pForwardCmd 命令，用于将连接转发到 libp2p 服务
var p2pForwardCmd = &cmds.Command{
    Status: cmds.Experimental, // 命令状态为实验性
    Helptext: cmds.HelpText{ // 帮助文本
        Tagline: "Forward connections to libp2p service.", // 简短描述
        ShortDescription: `
Forward connections made to <listen-address> to <target-address>.

<protocol> specifies the libp2p protocol name to use for libp2p
connections and/or handlers. It must be prefixed with '` + P2PProtoPrefix + `'.

Example:
  ipfs p2p forward ` + P2PProtoPrefix + `myproto /ip4/127.0.0.1/tcp/4567 /p2p/QmPeer
    - Forward connections to 127.0.0.1:4567 to '` + P2PProtoPrefix + `myproto' service on /p2p/QmPeer

`, // 详细描述
    },
    Arguments: []cmds.Argument{ // 命令参数
        cmds.StringArg("protocol", true, false, "Protocol name."), // 协议名称
        cmds.StringArg("listen-address", true, false, "Listening endpoint."), // 监听地址
        cmds.StringArg("target-address", true, false, "Target endpoint."), // 目标地址
    },
    Options: []cmds.Option{ // 命令选项
        cmds.BoolOption(allowCustomProtocolOptionName, "Don't require /x/ prefix"), // 允许自定义协议选项
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 执行函数
        n, err := p2pGetNode(env) // 获取 libp2p 节点
        if err != nil {
            return err
        }

        protoOpt := req.Arguments[0] // 获取协议参数
        listenOpt := req.Arguments[1] // 获取监听地址参数
        targetOpt := req.Arguments[2] // 获取目标地址参数

        proto := protocol.ID(protoOpt) // 将协议参数转换为协议 ID

        listen, err := ma.NewMultiaddr(listenOpt) // 解析监听地址
        if err != nil {
            return err
        }

        targets, err := parseIpfsAddr(targetOpt) // 解析目标地址
        if err != nil {
            return err
        }

        allowCustom, _ := req.Options[allowCustomProtocolOptionName].(bool) // 获取是否允许自定义协议选项

        if !allowCustom && !strings.HasPrefix(string(proto), P2PProtoPrefix) { // 如果不允许自定义协议且协议不以指定前缀开头
            return errors.New("protocol name must be within '" + P2PProtoPrefix + "' namespace") // 返回错误信息
        }

        return forwardLocal(n.Context(), n.P2P, n.Peerstore, proto, listen, targets) // 转发本地连接
    },
}

// parseIpfsAddr 是一个函数，接受地址字符串并返回 ipfsAddrs
func parseIpfsAddr(addr string) (*peer.AddrInfo, error) {
    // 解析给定的地址字符串为多地址对象
    multiaddr, err := ma.NewMultiaddr(addr)
    if err != nil {
        return nil, err
    }

    // 从多地址对象中提取 peer.AddrInfo 对象
    pi, err := peer.AddrInfoFromP2pAddr(multiaddr)
    if err == nil {
        return pi, nil
    }

    // 解析不是 ma.P_IPFS 协议的多地址
    ctx, cancel := context.WithTimeout(context.Background(), resolveTimeout)
    defer cancel()
    addrs, err := madns.Resolve(ctx, multiaddr)
    if err != nil {
        return nil, err
    }
    if len(addrs) == 0 {
        return nil, errors.New("fail to resolve the multiaddr:" + multiaddr.String())
    }
    var info peer.AddrInfo
    for _, addr := range addrs {
        taddr, id := peer.SplitAddr(addr)
        if id == "" {
            // 不是 IPFS 地址，跳过
            continue
        }
        switch info.ID {
        case "":
            info.ID = id
        case id:
        default:
            return nil, fmt.Errorf(
                "ambiguous multiaddr %s could refer to %s or %s",
                multiaddr,
                info.ID,
                id,
            )
        }
        info.Addrs = append(info.Addrs, taddr)
    }
    return &info, nil
}

var p2pListenCmd = &cmds.Command{
    Status: cmds.Experimental,
    Helptext: cmds.HelpText{
        Tagline: "Create libp2p service.",
        ShortDescription: `
Create libp2p service and forward connections made to <target-address>.

<protocol> specifies the libp2p handler name. It must be prefixed with '` + P2PProtoPrefix + `'.

Example:
  ipfs p2p listen ` + P2PProtoPrefix + `myproto /ip4/127.0.0.1/tcp/1234
    - Forward connections to 'myproto' libp2p service to 127.0.0.1:1234

`,
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("protocol", true, false, "Protocol name."),
        cmds.StringArg("target-address", true, false, "Target endpoint."),
    },
}
    # 定义一个包含两个命令选项的选项数组
    Options: []cmds.Option{
        # 定义一个布尔类型的选项，用于指示是否允许自定义协议前缀
        cmds.BoolOption(allowCustomProtocolOptionName, "Don't require /x/ prefix"),
        # 定义一个布尔类型的选项，用于指示是否在建立新连接时向目标发送远程的base58对等标识
        cmds.BoolOption(reportPeerIDOptionName, "r", "Send remote base58 peerid to target when a new connection is established"),
    },
    # 定义一个运行函数，接收请求、响应发射器和环境作为参数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取P2P节点
        n, err := p2pGetNode(env)
        if err != nil {
            return err
        }

        # 获取请求参数中的协议选项和目标选项
        protoOpt := req.Arguments[0]
        targetOpt := req.Arguments[1]

        # 将协议选项转换为协议ID
        proto := protocol.ID(protoOpt)

        # 将目标选项转换为多地址
        target, err := ma.NewMultiaddr(targetOpt)
        if err != nil {
            return err
        }

        # 检查目标地址的端口是否为0
        if err := checkPort(target); err != nil {
            return err
        }

        # 获取allowCustom和reportPeerID选项的布尔值
        allowCustom, _ := req.Options[allowCustomProtocolOptionName].(bool)
        reportPeerID, _ := req.Options[reportPeerIDOptionName].(bool)

        # 如果不允许自定义协议且协议不以P2PProtoPrefix开头，则返回错误
        if !allowCustom && !strings.HasPrefix(string(proto), P2PProtoPrefix) {
            return errors.New("protocol name must be within '" + P2PProtoPrefix + "' namespace")
        }

        # 将远程协议和目标地址转发到P2P节点
        _, err = n.P2P.ForwardRemote(n.Context(), proto, target, reportPeerID)
        return err
    },
// checkPort检查目标multiaddr是否包含tcp或udp协议，以及端口是否等于0
func checkPort(target ma.Multiaddr) error {
    // 从multiaddr中获取tcp或udp端口
    getPort := func() (string, error) {
        sport, _ := target.ValueForProtocol(ma.P_TCP)
        if sport != "" {
            return sport, nil
        }

        sport, _ = target.ValueForProtocol(ma.P_UDP)
        if sport != "" {
            return sport, nil
        }
        return "", fmt.Errorf("address does not contain tcp or udp protocol")
    }

    sport, err := getPort()
    if err != nil {
        return err
    }

    port, err := strconv.Atoi(sport)
    if err != nil {
        return err
    }

    if port == 0 {
        return fmt.Errorf("port can not be 0")
    }

    return nil
}

// forwardLocal将本地连接转发到libp2p服务
func forwardLocal(ctx context.Context, p *p2p.P2P, ps pstore.Peerstore, proto protocol.ID, bindAddr ma.Multiaddr, addr *peer.AddrInfo) error {
    ps.AddAddrs(addr.ID, addr.Addrs, pstore.TempAddrTTL)
    // TODO: 返回一些信息
    _, err := p.ForwardLocal(ctx, addr.ID, proto, bindAddr)
    return err
}

const (
    p2pHeadersOptionName = "headers"
)

var p2pLsCmd = &cmds.Command{
    Status: cmds.Experimental,
    Helptext: cmds.HelpText{
        Tagline: "List active p2p listeners.",
    },
    Options: []cmds.Option{
        cmds.BoolOption(p2pHeadersOptionName, "v", "Print table headers (Protocol, Listen, Target)."),
    },
    # 运行函数，处理请求并返回响应
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取 P2P 节点
        n, err := p2pGetNode(env)
        # 如果获取节点失败，则返回错误
        if err != nil {
            return err
        }

        # 创建 P2PLsOutput 结构体
        output := &P2PLsOutput{}

        # 锁定本地监听器列表
        n.P2P.ListenersLocal.Lock()
        # 遍历本地监听器列表，将信息添加到 output.Listeners 中
        for _, listener := range n.P2P.ListenersLocal.Listeners {
            output.Listeners = append(output.Listeners, P2PListenerInfoOutput{
                Protocol:      string(listener.Protocol()),
                ListenAddress: listener.ListenAddress().String(),
                TargetAddress: listener.TargetAddress().String(),
            })
        }
        # 解锁本地监听器列表
        n.P2P.ListenersLocal.Unlock()

        # 锁定 P2P 监听器列表
        n.P2P.ListenersP2P.Lock()
        # 遍历 P2P 监听器列表，将信息添加到 output.Listeners 中
        for _, listener := range n.P2P.ListenersP2P.Listeners {
            output.Listeners = append(output.Listeners, P2PListenerInfoOutput{
                Protocol:      string(listener.Protocol()),
                ListenAddress: listener.ListenAddress().String(),
                TargetAddress: listener.TargetAddress().String(),
            })
        }
        # 解锁 P2P 监听器列表
        n.P2P.ListenersP2P.Unlock()

        # 发送一次响应
        return cmds.EmitOnce(res, output)
    },
    # 定义类型为 P2PLsOutput
    Type: P2PLsOutput{},
    # 设置编码器映射
    Encoders: cmds.EncoderMap{
        # 设置文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *P2PLsOutput) error {
            # 获取请求中的选项 p2pHeadersOptionName，并转换为布尔值
            headers, _ := req.Options[p2pHeadersOptionName].(bool)
            # 创建 tabwriter
            tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
            # 遍历监听器列表，将信息格式化输出到 tabwriter
            for _, listener := range out.Listeners {
                if headers {
                    fmt.Fprintln(tw, "Protocol\tListen Address\tTarget Address")
                }

                fmt.Fprintf(tw, "%s\t%s\t%s\n", listener.Protocol, listener.ListenAddress, listener.TargetAddress)
            }
            tw.Flush()

            return nil
        }),
    },
# 定义常量，表示不同的选项名称
const (
    p2pAllOptionName           = "all"  # 表示关闭所有监听器的选项名称
    p2pProtocolOptionName      = "protocol"  # 表示匹配协议名称的选项名称
    p2pListenAddressOptionName = "listen-address"  # 表示匹配监听地址的选项名称
    p2pTargetAddressOptionName = "target-address"  # 表示匹配目标地址的选项名称
)

# 定义 p2pCloseCmd 变量，表示关闭命令
var p2pCloseCmd = &cmds.Command{
    Status: cmds.Experimental,  # 设置命令的状态为实验性
    Helptext: cmds.HelpText{  # 设置命令的帮助文本
        Tagline: "Stop listening for new connections to forward.",  # 设置命令的简短描述
    },
    Options: []cmds.Option{  # 设置命令的选项
        cmds.BoolOption(p2pAllOptionName, "a", "Close all listeners."),  # 添加布尔类型选项，表示关闭所有监听器
        cmds.StringOption(p2pProtocolOptionName, "p", "Match protocol name"),  # 添加字符串类型选项，表示匹配协议名称
        cmds.StringOption(p2pListenAddressOptionName, "l", "Match listen address"),  # 添加字符串类型选项，表示匹配监听地址
        cmds.StringOption(p2pTargetAddressOptionName, "t", "Match target address"),  # 添加字符串类型选项，表示匹配目标地址
    },
    # 定义一个名为 Run 的函数，接收请求、响应和环境参数，并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境参数中获取节点信息
        n, err := p2pGetNode(env)
        # 如果获取节点信息出错，则返回错误
        if err != nil {
            return err
        }

        # 从请求参数中获取是否关闭所有连接的选项
        closeAll, _ := req.Options[p2pAllOptionName].(bool)
        # 从请求参数中获取协议选项
        protoOpt, p := req.Options[p2pProtocolOptionName].(string)
        # 从请求参数中获取监听地址选项
        listenOpt, l := req.Options[p2pListenAddressOptionName].(string)
        # 从请求参数中获取目标地址选项
        targetOpt, t := req.Options[p2pTargetAddressOptionName].(string)

        # 将协议选项转换为协议 ID
        proto := protocol.ID(protoOpt)

        # 定义目标地址和监听地址的多重地址变量
        var target, listen ma.Multiaddr

        # 如果存在监听地址选项，则解析为多重地址
        if l {
            listen, err = ma.NewMultiaddr(listenOpt)
            # 如果解析出错，则返回错误
            if err != nil {
                return err
            }
        }

        # 如果存在目标地址选项，则解析为多重地址
        if t {
            target, err = ma.NewMultiaddr(targetOpt)
            # 如果解析出错，则返回错误
            if err != nil {
                return err
            }
        }

        # 如果没有任何匹配的选项，则返回错误
        if !(closeAll || p || l || t) {
            return errors.New("no matching options given")
        }

        # 如果同时存在关闭所有连接选项和其他匹配选项，则返回错误
        if closeAll && (p || l || t) {
            return errors.New("can't combine --all with other matching options")
        }

        # 定义一个匹配函数，用于判断是否符合关闭条件
        match := func(listener p2p.Listener) bool {
            if closeAll {
                return true
            }
            if p && proto != listener.Protocol() {
                return false
            }
            if l && !listen.Equal(listener.ListenAddress()) {
                return false
            }
            if t && !target.Equal(listener.TargetAddress()) {
                return false
            }
            return true
        }

        # 关闭本地监听器和 P2P 监听器，并返回关闭的连接数
        done := n.P2P.ListenersLocal.Close(match)
        done += n.P2P.ListenersP2P.Close(match)

        # 将关闭的连接数发送给响应
        return cmds.EmitOnce(res, done)
    },
    # 定义类型为整数
    Type: int(0),
    # 定义编码器映射
    Encoders: cmds.EncoderMap{
        # 使用文本编码器，将结果输出到写入器中
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out int) error {
            fmt.Fprintf(w, "Closed %d stream(s)\n", out)
            return nil
        }),
    },
// Stream
//

// p2pStreamCmd is the 'ipfs p2p stream' command
// 定义 p2pStreamCmd 命令，用于管理 P2P 流
var p2pStreamCmd = &cmds.Command{
    Status: cmds.Experimental,
    Helptext: cmds.HelpText{
        Tagline:          "P2P stream management.",
        ShortDescription: "Create and manage p2p streams",
    },

    Subcommands: map[string]*cmds.Command{
        "ls":    p2pStreamLsCmd,  // 子命令 ls，用于列出活动的 P2P 流
        "close": p2pStreamCloseCmd,  // 子命令 close，用于关闭 P2P 流
    },
}

// 定义 p2pStreamLsCmd 命令，用于列出活动的 P2P 流
var p2pStreamLsCmd = &cmds.Command{
    Status: cmds.Experimental,
    Helptext: cmds.HelpText{
        Tagline: "List active p2p streams.",  // 帮助文本，用于描述命令的作用
    },
    Options: []cmds.Option{
        cmds.BoolOption(p2pHeadersOptionName, "v", "Print table headers (ID, Protocol, Local, Remote)."),  // 命令选项，用于控制输出内容
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        n, err := p2pGetNode(env)  // 获取 P2P 节点
        if err != nil {
            return err
        }

        output := &P2PStreamsOutput{}  // 创建 P2P 流输出对象

        n.P2P.Streams.Lock()  // 锁定 P2P 流
        for id, s := range n.P2P.Streams.Streams {  // 遍历 P2P 流
            output.Streams = append(output.Streams, P2PStreamInfoOutput{  // 将 P2P 流信息添加到输出对象中
                HandlerID: strconv.FormatUint(id, 10),  // 将处理程序 ID 转换为字符串格式

                Protocol: string(s.Protocol),  // 获取协议名称

                OriginAddress: s.OriginAddr.String(),  // 获取源地址
                TargetAddress: s.TargetAddr.String(),  // 获取目标地址
            })
        }
        n.P2P.Streams.Unlock()  // 解锁 P2P 流

        return cmds.EmitOnce(res, output)  // 发送一次输出
    },
    Type: P2PStreamsOutput{},  // 指定输出类型为 P2PStreamsOutput
}
    # 创建一个命令编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *P2PStreamsOutput) error {
            # 从请求选项中获取 p2pHeadersOptionName 对应的布尔值
            headers, _ := req.Options[p2pHeadersOptionName].(bool)
            # 创建一个新的 tabwriter，用于格式化输出
            tw := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
            # 遍历输出流列表
            for _, stream := range out.Streams {
                # 如果 headers 为真，则输出表头
                if headers {
                    fmt.Fprintln(tw, "ID\tProtocol\tOrigin\tTarget")
                }
                # 格式化输出流的信息到 tabwriter
                fmt.Fprintf(tw, "%s\t%s\t%s\t%s\n", stream.HandlerID, stream.Protocol, stream.OriginAddress, stream.TargetAddress)
            }
            # 刷新 tabwriter
            tw.Flush()

            # 返回空值
            return nil
        }),
    },
# 定义一个命令对象，用于关闭活动的点对点流
var p2pStreamCloseCmd = &cmds.Command{
    Status: cmds.Experimental,  # 设置命令状态为实验性
    Helptext: cmds.HelpText{  # 设置命令的帮助文本
        Tagline: "Close active p2p stream.",  # 设置命令的简短描述
    },
    Arguments: []cmds.Argument{  # 定义命令的参数
        cmds.StringArg("id", false, false, "Stream identifier"),  # 设置一个字符串类型的参数，用于指定流的标识符
    },
    Options: []cmds.Option{  # 定义命令的选项
        cmds.BoolOption(p2pAllOptionName, "a", "Close all streams."),  # 设置一个布尔类型的选项，用于指定是否关闭所有流
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  # 定义命令的运行函数
        n, err := p2pGetNode(env)  # 调用p2pGetNode函数获取IpfsNode对象
        if err != nil {  # 如果出现错误，则返回错误
            return err
        }

        closeAll, _ := req.Options[p2pAllOptionName].(bool)  # 获取是否关闭所有流的选项值
        var handlerID uint64  # 定义一个流的标识符变量

        if !closeAll {  # 如果不关闭所有流
            if len(req.Arguments) == 0 {  # 如果参数列表为空
                return errors.New("no id specified")  # 返回错误信息
            }

            handlerID, err = strconv.ParseUint(req.Arguments[0], 10, 64)  # 将参数转换为无符号整数
            if err != nil {  # 如果出现错误，则返回错误
                return err
            }
        }

        toClose := make([]*p2p.Stream, 0, 1)  # 创建一个用于存储要关闭的流的切片
        n.P2P.Streams.Lock()  # 锁定P2P流
        for id, stream := range n.P2P.Streams.Streams {  # 遍历P2P流
            if !closeAll && handlerID != id {  # 如果不关闭所有流且流的标识符不匹配
                continue  # 继续下一次循环
            }
            toClose = append(toClose, stream)  # 将流添加到要关闭的切片中
            if !closeAll {  # 如果不关闭所有流
                break  # 退出循环
            }
        }
        n.P2P.Streams.Unlock()  # 解锁P2P流

        for _, s := range toClose {  # 遍历要关闭的流
            n.P2P.Streams.Reset(s)  # 重置流
        }

        return nil  # 返回空值
    },
}

# 定义一个函数，用于获取IpfsNode对象
func p2pGetNode(env cmds.Environment) (*core.IpfsNode, error) {
    nd, err := cmdenv.GetNode(env)  # 调用cmdenv.GetNode函数获取IpfsNode对象
    if err != nil {  # 如果出现错误，则返回错误
        return nil, err
    }

    config, err := nd.Repo.Config()  # 获取节点的配置信息
    if err != nil {  # 如果出现错误，则返回错误
        return nil, err
    }

    if !config.Experimental.Libp2pStreamMounting {  # 如果实验性的Libp2p流挂载未启用
        return nil, errors.New("libp2p stream mounting not enabled")  # 返回错误信息
    }

    if !nd.IsOnline {  # 如果节点不在线
        return nil, ErrNotOnline  # 返回错误信息
    }

    return nd, nil  # 返回IpfsNode对象和空值
}
```