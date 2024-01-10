# `kubo\core\commands\swarm.go`

```
package commands

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "encoding/base64"  // base64 编码解码包
    "encoding/json"  // JSON 编解码包
    "errors"  // 错误处理包
    "fmt"  // 格式化包，用于打印输出
    "io"  // 输入输出包
    "path"  // 路径处理包
    "sort"  // 排序包
    "strconv"  // 字符串转换包
    "sync"  // 同步包，用于并发控制
    "text/tabwriter"  // 格式化输出包
    "time"  // 时间包

    "github.com/ipfs/kubo/commands"  // 导入自定义包
    "github.com/ipfs/kubo/config"  // 导入自定义包
    "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入自定义包
    "github.com/ipfs/kubo/core/node/libp2p"  // 导入自定义包
    "github.com/ipfs/kubo/repo"  // 导入自定义包
    "github.com/ipfs/kubo/repo/fsrepo"  // 导入自定义包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入第三方包
    ic "github.com/libp2p/go-libp2p/core/crypto"  // 导入第三方包
    inet "github.com/libp2p/go-libp2p/core/network"  // 导入第三方包
    "github.com/libp2p/go-libp2p/core/peer"  // 导入第三方包
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入第三方包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入第三方包
    ma "github.com/multiformats/go-multiaddr"  // 导入第三方包
    madns "github.com/multiformats/go-multiaddr-dns"  // 导入第三方包
    mamask "github.com/whyrusleeping/multiaddr-filter"  // 导入第三方包
)

const (
    dnsResolveTimeout = 10 * time.Second  // DNS 解析超时时间
)

type stringList struct {
    Strings []string  // 字符串列表结构体
}

type addrMap struct {
    Addrs map[string][]string  // 地址映射结构体
}

var SwarmCmd = &cmds.Command{  // SwarmCmd 命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Interact with the swarm.",  // 简短描述
        ShortDescription: `  // 简要描述
'ipfs swarm' is a tool to manipulate the network swarm. The swarm is the
component that opens, listens for, and maintains connections to other
ipfs peers in the internet.
`,
    },
    Subcommands: map[string]*cmds.Command{  // 子命令映射
        "addrs":      swarmAddrsCmd,  // 添加地址命令
        "connect":    swarmConnectCmd,  // 连接命令
        "disconnect": swarmDisconnectCmd,  // 断开连接命令
        "filters":    swarmFiltersCmd,  // 过滤器命令
        "peers":      swarmPeersCmd,  // 对等节点命令
        "peering":    swarmPeeringCmd,  // 对等连接命令
        "resources":  swarmResourcesCmd, // libp2p Network Resource Manager  // 资源管理命令

    },
}

const (
    swarmVerboseOptionName           = "verbose"  // 详细选项名称
    swarmStreamsOptionName           = "streams"  // 流选项名称
    swarmLatencyOptionName           = "latency"  // 延迟选项名称
    swarmDirectionOptionName         = "direction"  // 方向选项名称
    swarmResetLimitsOptionName       = "reset"  // 重置限制选项名称
    # 定义变量，表示用于计算资源使用百分比的名称
    swarmUsedResourcesPercentageName = "min-used-limit-perc"
    # 定义变量，表示用于标识选项的名称
    swarmIdentifyOptionName          = "identify"
// 定义结构体 peeringResult，包含 ID 和 Status 两个字段
type peeringResult struct {
    ID     peer.ID
    Status string
}

// 定义命令 swarmPeeringCmd
var swarmPeeringCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Modify the peering subsystem.",
        ShortDescription: `
'ipfs swarm peering' manages the peering subsystem.
Peers in the peering subsystem are maintained to be connected, reconnected
on disconnect with a back-off.
The changes are not saved to the config.
`,
    },
    Subcommands: map[string]*cmds.Command{
        "add": swarmPeeringAddCmd,  // 添加子命令 add
        "ls":  swarmPeeringLsCmd,   // 添加子命令 ls
        "rm":  swarmPeeringRmCmd,   // 添加子命令 rm
    },
}

// 定义命令 swarmPeeringAddCmd
var swarmPeeringAddCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Add peers into the peering subsystem.",
        ShortDescription: `
'ipfs swarm peering add' will add the new address to the peering subsystem as one that should always be connected to.
`,
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("address", true, true, "address of peer to add into the peering subsystem"),  // 添加参数 address
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        addrs := make([]ma.Multiaddr, len(req.Arguments))  // 创建一个 Multiaddr 数组

        for i, arg := range req.Arguments {  // 遍历参数
            addr, err := ma.NewMultiaddr(arg)  // 将参数转换为 Multiaddr
            if err != nil {
                return err
            }

            addrs[i] = addr  // 将转换后的 Multiaddr 存入数组
        }

        addInfos, err := peer.AddrInfosFromP2pAddrs(addrs...)  // 从 Multiaddr 数组中创建 AddrInfo 数组
        if err != nil {
            return err
        }

        node, err := cmdenv.GetNode(env)  // 从环境中获取节点信息
        if err != nil {
            return err
        }
        if !node.IsOnline {  // 如果节点不在线
            return ErrNotOnline  // 返回错误信息
        }

        for _, addrinfo := range addInfos {  // 遍历 AddrInfo 数组
            node.Peering.AddPeer(addrinfo)  // 将 AddrInfo 添加到节点的 Peering 中
            err = res.Emit(peeringResult{addrinfo.ID, "success"})  // 发送成功的结果
            if err != nil {
                return err
            }
        }
        return nil
    },
    # 创建 Encoders 对象，包含了不同类型命令的编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到特定的编码器函数
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, pr *peeringResult) error {
            # 将格式化的字符串写入到输出流中
            fmt.Fprintf(w, "add %s %s\n", pr.ID.String(), pr.Status)
            # 返回空值表示编码成功
            return nil
        }),
    },
    # 指定 Type 为 peeringResult 类型的对象
    Type: peeringResult{},
# 定义名为swarmPeeringLsCmd的命令对象，用于列出在对等子系统中注册的对等节点
var swarmPeeringLsCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List peers registered in the peering subsystem.",  # 帮助文本，简短描述列出在对等子系统中注册的对等节点
        ShortDescription: `
'ipfs swarm peering ls' lists the peers that are registered in the peering subsystem and to which the daemon is always connected.
`,  # 简短描述，描述列出在对等子系统中注册的对等节点，并且守护程序始终连接到这些节点
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        node, err := cmdenv.GetNode(env)  # 获取节点信息
        if err != nil {
            return err
        }
        if !node.IsOnline {  # 如果节点不在线
            return ErrNotOnline  # 返回错误信息
        }

        peers := node.Peering.ListPeers()  # 获取节点的对等节点列表
        return cmds.EmitOnce(res, addrInfos{Peers: peers})  # 一次性发送对等节点列表给响应输出
    },
    Type: addrInfos{},  # 类型为addrInfos结构体
    Encoders: cmds.EncoderMap{  # 编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ai *addrInfos) error {  # 文本编码器，将对等节点信息写入io.Writer
            for _, info := range ai.Peers {  # 遍历对等节点信息
                fmt.Fprintf(w, "%s\n", info.ID)  # 格式化输出对等节点ID
                for _, addr := range info.Addrs {  # 遍历对等节点地址
                    fmt.Fprintf(w, "\t%s\n", addr)  # 格式化输出对等节点地址
                }
            }
            return nil
        }),
    },
}

# 定义名为addrInfos的结构体，包含对等节点列表
type addrInfos struct {
    Peers []peer.AddrInfo  # 对等节点列表
}

# 定义名为swarmPeeringRmCmd的命令对象，用于从对等子系统中移除对等节点
var swarmPeeringRmCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Remove a peer from the peering subsystem.",  # 帮助文本，简短描述从对等子系统中移除对等节点
        ShortDescription: `
'ipfs swarm peering rm' will remove the given ID from the peering subsystem and remove it from the always-on connection.
`,  # 简短描述，描述将给定ID从对等子系统中移除，并从始终连接中移除
    },
    Arguments: []cmds.Argument{  # 参数列表
        cmds.StringArg("ID", true, true, "ID of peer to remove from the peering subsystem"),  # 字符串参数，表示要从对等子系统中移除的对等节点的ID
    },
}
    # Run 函数处理请求，接收请求、发送响应和环境变量
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境变量中获取节点信息
        node, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        # 如果节点不在线，则返回错误
        if !node.IsOnline {
            return ErrNotOnline
        }

        # 遍历请求参数
        for _, arg := range req.Arguments {
            # 解码对等节点的 ID
            id, err := peer.Decode(arg)
            if err != nil {
                return err
            }

            # 移除对等节点
            node.Peering.RemovePeer(id)
            # 发送移除对等节点的结果
            if err = res.Emit(peeringResult{id, "success"}); err != nil {
                return err
            }
        }
        # 返回空错误
        return nil
    },
    # 指定返回结果的类型
    Type: peeringResult{},
    # 指定编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, pr *peeringResult) error {
            # 将移除对等节点的结果格式化为文本并写入到输出流
            fmt.Fprintf(w, "remove %s %s\n", pr.ID.String(), pr.Status)
            return nil
        }),
    },
// 定义 swarmPeersCmd 命令，用于列出具有打开连接的对等节点
var swarmPeersCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List peers with open connections.", // 帮助文本的一句话描述
        ShortDescription: `
'ipfs swarm peers' lists the set of peers this node is connected to.
`, // 帮助文本的简短描述
    },
    Options: []cmds.Option{ // 定义命令的选项
        cmds.BoolOption(swarmVerboseOptionName, "v", "display all extra information"), // 显示所有额外信息的选项
        cmds.BoolOption(swarmStreamsOptionName, "Also list information about open streams for each peer"), // 列出每个对等节点的打开流信息的选项
        cmds.BoolOption(swarmLatencyOptionName, "Also list information about latency to each peer"), // 列出到每个对等节点的延迟信息的选项
        cmds.BoolOption(swarmDirectionOptionName, "Also list information about the direction of connection"), // 列出连接方向信息的选项
        cmds.BoolOption(swarmIdentifyOptionName, "Also list information about peers identify"), // 列出对等节点标识信息的选项
    },
    Encoders: cmds.EncoderMap{ // 定义命令的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ci *connInfos) error { // 使用文本编码器
            pipfs := ma.ProtocolWithCode(ma.P_IPFS).Name // 获取 IPFS 协议名称
            for _, info := range ci.Peers { // 遍历对等节点信息
                fmt.Fprintf(w, "%s/%s/%s", info.Addr, pipfs, info.Peer) // 格式化输出对等节点地址、协议和对等节点信息
                if info.Latency != "" { // 如果存在延迟信息
                    fmt.Fprintf(w, " %s", info.Latency) // 输出延迟信息
                }

                if info.Direction != inet.DirUnknown { // 如果存在连接方向信息
                    fmt.Fprintf(w, " %s", directionString(info.Direction)) // 输出连接方向信息
                }
                fmt.Fprintln(w) // 输出空行

                for _, s := range info.Streams { // 遍历对等节点的流信息
                    if s.Protocol == "" { // 如果协议为空
                        s.Protocol = "<no protocol name>" // 设置协议名称为默认值
                    }

                    fmt.Fprintf(w, "  %s\n", s.Protocol) // 输出流的协议名称
                }
            }

            return nil
        }),
    },
    Type: connInfos{}, // 定义命令的类型为 connInfos 结构体
}

// 定义 swarmResourcesCmd 命令，用于获取 libp2p 资源管理器所记录的所有资源的摘要
var swarmResourcesCmd = &cmds.Command{
    Status: cmds.Experimental, // 命令状态为实验性
    Helptext: cmds.HelpText{
        Tagline: "Get a summary of all resources accounted for by the libp2p Resource Manager.", // 帮助文本的一句话描述
        LongDescription: ` // 帮助文本的长描述
// 获取 libp2p 资源管理器所记录的所有资源的摘要
// 包括限制和对这些限制的使用情况
// 可以输出人类可读的表格和 JSON 编码
`,
Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
    // 从环境中获取节点
    node, err := cmdenv.GetNode(env)
    if err != nil {
        return err
    }

    // 如果节点的资源管理器为空，则返回错误
    if node.ResourceManager == nil {
        return libp2p.ErrNoResourceMgr
    }

    // 从节点的存储库中获取配置信息
    cfg, err := node.Repo.Config()
    if err != nil {
        return err
    }

    // 从节点的存储库中获取用户资源覆盖信息
    userResourceOverrides, err := node.Repo.UserResourceOverrides()
    if err != nil {
        return err
    }

    // FIXME: 我们不应该重新计算限制，应该保存它们或从 libp2p 加载它们 (https://github.com/libp2p/go-libp2p/issues/2166)
    // 计算限制配置
    limitConfig, _, err := libp2p.LimitConfig(cfg.Swarm, userResourceOverrides)
    if err != nil {
        return err
    }

    // 获取节点的资源管理器状态
    rapi, ok := node.ResourceManager.(rcmgr.ResourceManagerState)
    if !ok { // NullResourceManager
        return libp2p.ErrNoResourceMgr
    }

    // 将限制配置和资源管理器状态合并，并发射一次结果
    return cmds.EmitOnce(res, libp2p.MergeLimitsAndStatsIntoLimitsConfigAndUsage(limitConfig, rapi.Stat()))
},
    # 创建一个编码器映射，将不同类型的命令编码为对应的编码器函数
    Encoders: cmds.EncoderMap{
        # JSON 编码器
        cmds.JSON: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, limitsAndUsage libp2p.LimitsConfigAndUsage) error {
            # 使用 JSON 编码器将限制配置和使用信息编码到输出流中
            return json.NewEncoder(w).Encode(limitsAndUsage)
        }),
        # 文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, limitsAndUsage libp2p.LimitsConfigAndUsage) error {
            # 创建一个制表符写入器，用于格式化输出
            tw := tabwriter.NewWriter(w, 20, 8, 0, '\t', 0)
            # 在函数结束时刷新并关闭制表符写入器
            defer tw.Flush()

            # 格式化输出表头
            fmt.Fprintf(tw, "%s\t%s\t%s\t%s\t%s\t\n", "Scope", "Limit Name", "Limit Value", "Limit Usage Amount", "Limit Usage Percent")
            # 遍历限制配置和使用信息，将其格式化输出到制表符写入器中
            for _, ri := range libp2p.LimitConfigsToInfo(limitsAndUsage) {
                var limit, percentage string
                # 根据限制值和当前使用量计算限制和百分比
                switch ri.LimitValue {
                case rcmgr.Unlimited64:
                    limit = "unlimited"
                    percentage = "n/a"
                case rcmgr.BlockAllLimit64:
                    limit = "blockAll"
                    percentage = "n/a"
                default:
                    limit = strconv.FormatInt(int64(ri.LimitValue), 10)
                    if ri.CurrentUsage == 0 {
                        percentage = "0%"
                    } else {
                        percentage = strconv.FormatFloat(float64(ri.CurrentUsage)/float64(ri.LimitValue)*100, 'f', 1, 64) + "%"
                    }
                }
                # 将格式化后的信息输出到制表符写入器中
                fmt.Fprintf(tw, "%s\t%s\t%s\t%d\t%s\t\n",
                    ri.ScopeName,
                    ri.LimitName,
                    limit,
                    ri.CurrentUsage,
                    percentage,
                )
            }

            # 返回空值，表示编码成功
            return nil
        }),
    },
    # 初始化一个空的限制配置和使用信息对象
    Type: libp2p.LimitsConfigAndUsage{},
}
// 定义流信息结构体
type streamInfo struct {
    Protocol string
}

// 定义连接信息结构体
type connInfo struct {
    Addr      string         `json:",omitempty"`
    Peer      string         `json:",omitempty"`
    Latency   string         `json:",omitempty"`
    Muxer     string         `json:",omitempty"`
    Direction inet.Direction `json:",omitempty"`
    Streams   []streamInfo   `json:",omitempty"`
    Identify  IdOutput       `json:",omitempty"`
}

// 比较函数，用于排序
func (ci *connInfo) Less(i, j int) bool {
    return ci.Streams[i].Protocol < ci.Streams[j].Protocol
}

// 返回长度
func (ci *connInfo) Len() int {
    return len(ci.Streams)
}

// 交换元素
func (ci *connInfo) Swap(i, j int) {
    ci.Streams[i], ci.Streams[j] = ci.Streams[j], ci.Streams[i]
}

// 定义连接信息集合结构体
type connInfos struct {
    Peers []connInfo
}

// 比较函数，用于排序
func (ci connInfos) Less(i, j int) bool {
    return ci.Peers[i].Addr < ci.Peers[j].Addr
}

// 返回长度
func (ci connInfos) Len() int {
    return len(ci.Peers)
}

// 交换元素
func (ci connInfos) Swap(i, j int) {
    ci.Peers[i], ci.Peers[j] = ci.Peers[j], ci.Peers[i]
}

// 根据对等节点信息获取标识信息
func (ci *connInfo) identifyPeer(ps pstore.Peerstore, p peer.ID) (IdOutput, error) {
    var info IdOutput
    info.ID = p.String()

    // 获取公钥并编码
    if pk := ps.PubKey(p); pk != nil {
        pkb, err := ic.MarshalPublicKey(pk)
        if err != nil {
            return IdOutput{}, err
        }
        info.PublicKey = base64.StdEncoding.EncodeToString(pkb)
    }

    // 获取地址信息并排序
    addrInfo := ps.PeerInfo(p)
    addrs, err := peer.AddrInfoToP2pAddrs(&addrInfo)
    if err != nil {
        return IdOutput{}, err
    }

    for _, a := range addrs {
        info.Addresses = append(info.Addresses, a.String())
    }
    sort.Strings(info.Addresses)

    // 获取协议信息并排序
    if protocols, err := ps.GetProtocols(p); err == nil {
        info.Protocols = append(info.Protocols, protocols...)
        sort.Slice(info.Protocols, func(i, j int) bool { return info.Protocols[i] < info.Protocols[j] })
    }

    // 获取代理版本信息
    if v, err := ps.Get(p, "AgentVersion"); err == nil {
        if vs, ok := v.(string); ok {
            info.AgentVersion = vs
        }
    }
}
    # 返回变量 info 和 nil
    return info, nil
// directionString函数将inet.Direction类型转换为字符串
func directionString(d inet.Direction) string {
    // 使用switch语句根据inet.Direction类型的不同返回不同的字符串
    switch d {
    case inet.DirInbound:
        return "inbound"
    case inet.DirOutbound:
        return "outbound"
    default:
        return ""
    }
}

// swarmAddrsCmd是一个命令对象，用于列出已知地址，用于调试
var swarmAddrsCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "List known addresses. Useful for debugging.",
        ShortDescription: `
'ipfs swarm addrs' lists all addresses this node is aware of.
`,
    },
    Subcommands: map[string]*cmds.Command{
        "local":  swarmAddrsLocalCmd, // 子命令"local"对应swarmAddrsLocalCmd
        "listen": swarmAddrsListenCmd, // 子命令"listen"对应swarmAddrsListenCmd
    },
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 从环境中获取api对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        // 获取节点已知的地址
        addrs, err := api.Swarm().KnownAddrs(req.Context)
        if err != nil {
            return err
        }

        // 创建一个map用于存储地址
        out := make(map[string][]string)
        for p, paddrs := range addrs {
            s := p.String()
            for _, a := range paddrs {
                out[s] = append(out[s], a.String())
            }
        }

        // 发送地址map给输出流
        return cmds.EmitOnce(res, &addrMap{Addrs: out})
    },
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, am *addrMap) error {
            // 对地址进行排序
            ids := make([]string, 0, len(am.Addrs))
            for p := range am.Addrs {
                ids = append(ids, p)
            }
            sort.Strings(ids)

            // 将排序后的地址输出到流中
            for _, p := range ids {
                paddrs := am.Addrs[p]
                fmt.Fprintf(w, "%s (%d)\n", p, len(paddrs))
                for _, addr := range paddrs {
                    fmt.Fprintf(w, "\t"+addr+"\n")
                }
            }

            return nil
        }),
    },
    Type: addrMap{}, // 设置命令的类型为addrMap
}

// swarmAddrsLocalCmd是一个命令对象
var swarmAddrsLocalCmd = &cmds.Command{
    # 定义帮助文本结构体，包含标语和简短描述
    Helptext: cmds.HelpText{
        # 设置标语为"List local addresses."
        Tagline: "List local addresses.",
        # 设置简短描述为多行字符串
        ShortDescription: `
// 'ipfs swarm addrs local' 列出所有本地监听地址并向网络宣布。
// 定义命令的选项
// -id 选项用于显示地址中的对等节点 ID
// 执行命令的函数
// 从环境中获取 API，并检查错误
// 获取是否显示 ID 的选项值
// 获取本地节点的密钥信息
// 获取本地节点的监听地址
// 定义地址列表变量
// 获取 P2P 协议的名称
// 遍历监听地址列表，根据是否显示 ID，拼接地址并添加到地址列表中
// 对地址列表进行排序
// 返回地址列表
// 定义命令的类型
// 定义命令的编码器
// 'ipfs swarm addrs listen' 列出节点正在监听的接口地址。
// 从环境中获取 API，并检查错误
// 定义地址列表变量
// 获取节点监听的地址
// 遍历监听地址列表，将地址添加到地址列表中
// 对地址列表进行排序
// 返回地址列表
// 定义命令的类型
    # 创建一个名为Encoders的映射，包含一个键值对
    Encoders: cmds.EncoderMap{
        # 键为cmds.Text，值为调用cmds.MakeTypedEncoder函数并传入safeTextListEncoder作为参数的结果
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
# 定义名为 swarmConnectCmd 的命令对象
var swarmConnectCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Open connection to a given peer.",
        ShortDescription: `
'ipfs swarm connect' attempts to ensure a connection to a given peer.

Multiaddresses given are advisory, for example the node may already be aware of other addresses for a given peer or may already have an established connection to the peer.

The address format is a libp2p multiaddr:

ipfs swarm connect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
`,
    },
    # 参数列表，包括地址参数
    Arguments: []cmds.Argument{
        cmds.StringArg("address", true, true, "Address of peer to connect to.").EnableStdin(),
    },
    # 运行函数，处理连接到给定地址的逻辑
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 获取节点对象
        node, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 获取 API 对象
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }

        # 解析地址参数
        addrs := req.Arguments
        pis, err := parseAddresses(req.Context, addrs, node.DNSResolver)
        if err != nil {
            return err
        }

        # 创建输出列表
        output := make([]string, len(pis))
        # 遍历解析后的地址列表
        for i, pi := range pis {
            # 构建连接信息
            output[i] = "connect " + pi.ID.String()
            # 尝试连接到地址
            err := api.Swarm().Connect(req.Context, pi)
            if err != nil {
                return fmt.Errorf("%s failure: %s", output[i], err)
            }
            output[i] += " success"
        }

        # 发送输出列表
        return cmds.EmitOnce(res, &stringList{output})
    },
    # 编码器映射，指定文本编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
    # 类型为字符串列表
    Type: stringList{},
}

# 定义名为 swarmDisconnectCmd 的命令对象
var swarmDisconnectCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Close connection to a given address.",
        ShortDescription: `
'ipfs swarm disconnect' closes a connection to a peer address. The address
format is an IPFS multiaddr:
// 断开与指定地址的 IPFS 节点的连接，断开后并不是永久性的；如果 IPFS 需要与该地址通信，它会重新连接。
ipfs swarm disconnect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ

// 定义了一个 IPFS 命令的结构体，包含了命令的名称、帮助信息、参数、运行函数、编码器等
cmd := &cmds.Command{
    // 命令的名称
    Name: "swarm disconnect",
    // 命令的帮助信息
    Help: "Close the connection to a given address.",
    // 命令的参数，包括参数名、是否必需、是否可选、参数描述等
    Arguments: []cmds.Argument{
        cmds.StringArg("address", true, true, "Address of peer to disconnect from.").EnableStdin(),
    },
    // 命令的运行函数，处理实际的断开连接操作
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取当前节点的信息
        node, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        // 获取 API 接口
        api, err := cmdenv.GetApi(env, req)
        if err != nil {
            return err
        }
        // 解析地址信息
        addrs, err := parseAddresses(req.Context, req.Arguments, node.DNSResolver)
        if err != nil {
            return err
        }
        // 初始化输出结果的数组
        output := make([]string, 0, len(addrs))
        // 遍历地址信息，执行断开连接操作
        for _, ainfo := range addrs {
            maddrs, err := peer.AddrInfoToP2pAddrs(&ainfo)
            if err != nil {
                return err
            }
            // FIXME: This will print: ...
            // 对每个地址执行断开连接操作，并将结果添加到输出数组中
            for _, addr := range maddrs {
                msg := "disconnect " + ainfo.ID.String()
                if err := api.Swarm().Disconnect(req.Context, addr); err != nil {
                    msg += " failure: " + err.Error()
                } else {
                    msg += " success"
                }
                output = append(output, msg)
            }
        }
        // 将输出结果发送给响应处理器
        return cmds.EmitOnce(res, &stringList{output})
    },
    // 定义命令的编码器，用于将结果编码成文本格式
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
    // 定义命令的类型
    Type: stringList{},
}
// parseAddresses is a function that takes in a slice of string peer addresses
// (multiaddr + peerid) and returns a slice of properly constructed peers
func parseAddresses(ctx context.Context, addrs []string, rslv *madns.Resolver) ([]peer.AddrInfo, error) {
    // resolve addresses
    maddrs, err := resolveAddresses(ctx, addrs, rslv)  // 调用 resolveAddresses 函数解析地址
    if err != nil {
        return nil, err  // 如果解析出错，返回错误
    }

    return peer.AddrInfosFromP2pAddrs(maddrs...)  // 返回解析后的地址信息
}

// resolveAddresses resolves addresses parallelly
func resolveAddresses(ctx context.Context, addrs []string, rslv *madns.Resolver) ([]ma.Multiaddr, error) {
    ctx, cancel := context.WithTimeout(ctx, dnsResolveTimeout)  // 设置解析超时时间
    defer cancel()  // 在函数返回前取消超时

    var maddrs []ma.Multiaddr  // 创建存放解析后地址的切片
    var wg sync.WaitGroup  // 创建同步等待组
    resolveErrC := make(chan error, len(addrs))  // 创建存放解析错误的通道

    maddrC := make(chan ma.Multiaddr)  // 创建存放解析后地址的通道

    for _, addr := range addrs {  // 遍历输入的地址
        maddr, err := ma.NewMultiaddr(addr)  // 将字符串地址转换为多地址对象
        if err != nil {
            return nil, err  // 如果转换出错，返回错误
        }

        // check whether address ends in `ipfs/Qm...`
        if _, last := ma.SplitLast(maddr); last.Protocol().Code == ma.P_IPFS {  // 检查地址是否以 `ipfs/Qm...` 结尾
            maddrs = append(maddrs, maddr)  // 如果是，将地址添加到解析后地址的切片中
            continue
        }
        wg.Add(1)  // 增加等待组计数
        go func(maddr ma.Multiaddr) {  // 启动一个 goroutine 进行地址解析
            defer wg.Done()  // 在函数结束时减少等待组计数
            raddrs, err := rslv.Resolve(ctx, maddr)  // 调用 Resolver 对象的 Resolve 方法解析地址
            if err != nil {
                resolveErrC <- err  // 如果解析出错，将错误发送到解析错误通道
                return
            }
            // filter out addresses that still doesn't end in `ipfs/Qm...`
            found := 0  // 初始化找到的地址数量
            for _, raddr := range raddrs {  // 遍历解析后的地址
                if _, last := ma.SplitLast(raddr); last != nil && last.Protocol().Code == ma.P_IPFS {  // 检查地址是否以 `ipfs/Qm...` 结尾
                    maddrC <- raddr  // 如果是，将地址发送到解析后地址通道
                    found++  // 增加找到的地址数量
                }
            }
            if found == 0 {
                resolveErrC <- fmt.Errorf("found no ipfs peers at %s", maddr)  // 如果找到的地址数量为0，发送错误信息到解析错误通道
            }
        }(maddr)
    }
    go func() {
        wg.Wait()  // 等待所有地址解析完成
        close(maddrC)  // 关闭解析后地址通道
    }()
    # 遍历通道 maddrC 中的值，将其添加到 maddrs 切片中
    for maddr := range maddrC:
        maddrs = append(maddrs, maddr)

    # 从 resolveErrC 通道中接收错误信息
    # 如果有错误信息，则返回 nil 和错误信息
    select {
    case err := <-resolveErrC:
        return nil, err
    # 如果没有错误信息，则继续执行下面的代码
    default:
    }

    # 返回 maddrs 切片和 nil，表示没有错误
    return maddrs, nil
# 定义名为 swarmFiltersCmd 的命令对象
var swarmFiltersCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Manipulate address filters.",
        ShortDescription: `
'ipfs swarm filters' will list out currently applied filters. Its subcommands
can be used to add or remove said filters. Filters are specified using the
multiaddr-filter format:
        `,
    },
    # 子命令对象的映射
    Subcommands: map[string]*cmds.Command{
        "add": swarmFiltersAddCmd,  # 添加子命令对象 swarmFiltersAddCmd
        "rm":  swarmFiltersRmCmd,   # 添加子命令对象 swarmFiltersRmCmd
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点对象
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        # 如果节点不在线，则返回错误
        if !n.IsOnline {
            return ErrNotOnline
        }
        # 初始化输出字符串切片
        var output []string
        # 遍历节点的过滤器列表，获取拒绝动作的过滤器
        for _, f := range n.Filters.FiltersForAction(ma.ActionDeny) {
            # 将过滤器转换为字符串形式
            s, err := mamask.ConvertIPNet(&f)
            if err != nil {
                return err
            }
            # 将转换后的字符串添加到输出切片中
            output = append(output, s)
        }
        # 发送一次性的结果到响应发射器
        return cmds.EmitOnce(res, &stringList{output})
    },
    # 编码器映射，指定文本编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
    # 类型为 stringList 结构
    Type: stringList{},
}

# 定义名为 swarmFiltersAddCmd 的命令对象
var swarmFiltersAddCmd = &cmds.Command{
    # 帮助文本，包括标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Add an address filter.",
        ShortDescription: `
'ipfs swarm filters add' will add an address filter to the daemons swarm.
        `,
    },
    # 参数列表，包括一个必需的字符串参数
    Arguments: []cmds.Argument{
        cmds.StringArg("address", true, true, "Multiaddr to filter.").EnableStdin(),
    },
}
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

        # 如果请求参数为空，则返回错误
        if len(req.Arguments) == 0 {
            return errors.New("no filters to add")
        }

        # 打开环境配置的文件系统存储库
        r, err := fsrepo.Open(env.(*commands.Context).ConfigRoot)
        if err != nil {
            return err
        }
        # 延迟关闭文件系统存储库
        defer r.Close()
        # 获取文件系统存储库的配置信息
        cfg, err := r.Config()
        if err != nil {
            return err
        }

        # 遍历请求参数，创建过滤器并添加到节点
        for _, arg := range req.Arguments {
            mask, err := mamask.NewMask(arg)
            if err != nil {
                return err
            }
            n.Filters.AddFilter(*mask, ma.ActionDeny)
        }

        # 调用 filtersAdd 函数，添加过滤器
        added, err := filtersAdd(r, cfg, req.Arguments)
        if err != nil {
            return err
        }

        # 发送一次性结果
        return cmds.EmitOnce(res, &stringList{added})
    },
    # 编码器映射
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
    # 类型为字符串列表
    Type: stringList{},
# 定义一个名为swarmFiltersRmCmd的命令对象
var swarmFiltersRmCmd = &cmds.Command{
    # 帮助文本，包括一句简短的标语和简短描述
    Helptext: cmds.HelpText{
        Tagline: "Remove an address filter.",
        ShortDescription: `
'ipfs swarm filters rm' will remove an address filter from the daemons swarm.
`,
    },
    # 参数列表，包括一个必填的字符串参数
    Arguments: []cmds.Argument{
        cmds.StringArg("address", true, true, "Multiaddr filter to remove.").EnableStdin(),
    },
    # 运行函数，处理命令请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }
        # 如果节点不在线，则返回错误
        if !n.IsOnline {
            return ErrNotOnline
        }
        # 打开存储库
        r, err := fsrepo.Open(env.(*commands.Context).ConfigRoot)
        if err != nil {
            return err
        }
        defer r.Close() # 延迟关闭存储库
        cfg, err := r.Config() # 获取存储库配置
        if err != nil {
            return err
        }
        # 如果参数为"all"或"*"，则移除所有过滤器
        if req.Arguments[0] == "all" || req.Arguments[0] == "*" {
            fs := n.Filters.FiltersForAction(ma.ActionDeny) # 获取所有拒绝动作的过滤器
            for _, f := range fs {
                n.Filters.RemoveLiteral(f) # 移除过滤器
            }
            removed, err := filtersRemoveAll(r, cfg) # 移除所有过滤器
            if err != nil {
                return err
            }
            return cmds.EmitOnce(res, &stringList{removed}) # 返回移除的过滤器列表
        }
        # 遍历参数列表中的每个参数
        for _, arg := range req.Arguments {
            mask, err := mamask.NewMask(arg) # 创建一个新的掩码
            if err != nil {
                return err
            }
            n.Filters.RemoveLiteral(*mask) # 移除过滤器
        }
        removed, err := filtersRemove(r, cfg, req.Arguments) # 移除指定的过滤器
        if err != nil {
            return err
        }
        return cmds.EmitOnce(res, &stringList{removed}) # 返回移除的过滤器列表
    },
    # 编码器映射，指定文本编码器
    Encoders: cmds.EncoderMap{
        cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
    },
    # 类型为字符串列表
    Type: stringList{},
}

# 定义一个名为filtersAdd的函数，用于向存储库添加过滤器
func filtersAdd(r repo.Repo, cfg *config.Config, filters []string) ([]string, error) {
    addedMap := map[string]struct{}{} # 创建一个空的字符串结构体映射
    addedList := make([]string, 0, len(filters)) # 创建一个空的字符串列表，预分配足够的空间
    // 重新添加配置的Swarm地址过滤器以删除重复项
    oldFilters := cfg.Swarm.AddrFilters  // 保存旧的地址过滤器配置

    cfg.Swarm.AddrFilters = nil  // 清空地址过滤器配置

    // 添加新的过滤器
    for _, filter := range filters {
        if _, found := addedMap[filter]; found {  // 检查是否已经添加过该过滤器
            continue
        }

        cfg.Swarm.AddrFilters = append(cfg.Swarm.AddrFilters, filter)  // 将新过滤器添加到配置中
        addedList = append(addedList, filter)  // 将新过滤器添加到已添加列表中
        addedMap[filter] = struct{}{}  // 将新过滤器添加到已添加映射中
    }

    // 恢复原始过滤器。按照这个顺序添加，以便输出它们。
    for _, filter := range oldFilters {
        if _, found := addedMap[filter]; found {  // 检查是否已经添加过该过滤器
            continue
        }

        cfg.Swarm.AddrFilters = append(cfg.Swarm.AddrFilters, filter)  // 将原始过滤器添加到配置中
        addedMap[filter] = struct{}{}  // 将原始过滤器添加到已添加映射中
    }

    if err := r.SetConfig(cfg); err != nil {  // 设置配置并检查是否出错
        return nil, err  // 如果出错，返回空和错误
    }

    return addedList, nil  // 返回已添加的过滤器列表和空
# 从配置中移除所有地址过滤器
func filtersRemoveAll(r repo.Repo, cfg *config.Config) ([]string, error) {
    # 保存要移除的地址过滤器
    removed := cfg.Swarm.AddrFilters
    # 清空地址过滤器列表
    cfg.Swarm.AddrFilters = nil

    # 将更新后的配置写入存储库
    if err := r.SetConfig(cfg); err != nil {
        return nil, err
    }

    # 返回被移除的地址过滤器列表
    return removed, nil
}

# 从配置中移除指定的地址过滤器
func filtersRemove(r repo.Repo, cfg *config.Config, toRemoveFilters []string) ([]string, error) {
    # 保存被移除的地址过滤器
    removed := make([]string, 0, len(toRemoveFilters))
    # 保存保留的地址过滤器
    keep := make([]string, 0, len(cfg.Swarm.AddrFilters))

    # 保存原始的地址过滤器列表
    oldFilters := cfg.Swarm.AddrFilters

    # 遍历原始地址过滤器列表
    for _, oldFilter := range oldFilters:
        # 初始化找到标志
        found := false
        # 遍历要移除的地址过滤器列表
        for _, toRemoveFilter := range toRemoveFilters:
            # 如果找到要移除的地址过滤器
            if oldFilter == toRemoveFilter:
                found = true
                # 将其添加到被移除列表中
                removed = append(removed, toRemoveFilter)
                break
            # 如果未找到要移除的地址过滤器
            if !found:
                # 将其添加到保留列表中
                keep = append(keep, oldFilter)
    # 更新配置中的地址过滤器列表为保留列表
    cfg.Swarm.AddrFilters = keep

    # 将更新后的配置写入存储库
    if err := r.SetConfig(cfg); err != nil {
        return nil, err
    }

    # 返回被移除的地址过滤器列表
    return removed, nil
}
```