# `kubo\core\commands\id.go`

```
package commands

import (
    "encoding/base64" // 导入 base64 编码包
    "encoding/json" // 导入 JSON 编码包
    "errors" // 导入错误处理包
    "fmt" // 导入格式化包
    "io" // 导入输入输出包
    "sort" // 导入排序包
    "strings" // 导入字符串处理包

    version "github.com/ipfs/kubo" // 导入版本包
    "github.com/ipfs/kubo/core" // 导入核心包
    "github.com/ipfs/kubo/core/commands/cmdenv" // 导入命令环境包

    cmds "github.com/ipfs/go-ipfs-cmds" // 导入命令包
    ke "github.com/ipfs/kubo/core/commands/keyencode" // 导入密钥编码包
    kb "github.com/libp2p/go-libp2p-kbucket" // 导入 kbucket 包
    ic "github.com/libp2p/go-libp2p/core/crypto" // 导入核心加密包
    "github.com/libp2p/go-libp2p/core/host" // 导入主机核心包
    "github.com/libp2p/go-libp2p/core/peer" // 导入对等体包
    pstore "github.com/libp2p/go-libp2p/core/peerstore" // 导入对等体存储包
    "github.com/libp2p/go-libp2p/core/protocol" // 导入协议包
)

const offlineIDErrorMessage = "'ipfs id' cannot query information on remote peers without a running daemon; if you only want to convert --peerid-base, pass --offline option" // 定义离线 ID 错误消息

type IdOutput struct { // nolint
    ID           string // ID 字符串
    PublicKey    string // 公钥字符串
    Addresses    []string // 地址列表
    AgentVersion string // 代理版本字符串
    Protocols    []protocol.ID // 协议列表
}

const (
    formatOptionName   = "format" // 格式选项名称
    idFormatOptionName = "peerid-base" // ID 格式选项名称
)

var IDCmd = &cmds.Command{ // 定义 ID 命令
    Helptext: cmds.HelpText{ // 帮助文本
        Tagline: "Show IPFS node id info.", // 标语
        ShortDescription: ` // 简短描述
Prints out information about the specified peer.
If no peer is specified, prints out information for local peers.

'ipfs id' supports the format option for output with the following keys:
<id> : The peers id.
<aver>: Agent version.
<pver>: Protocol version.
<pubkey>: Public key.
<addrs>: Addresses (newline delimited).
<protocols>: Libp2p Protocol registrations (newline delimited).

EXAMPLE:

    ipfs id Qmece2RkXhsKe5CRooNisBTh4SK119KrXXGmoK6V3kb8aH -f="<addrs>\n"
`, // 示例
    },
    Arguments: []cmds.Argument{ // 参数
        cmds.StringArg("peerid", false, false, "Peer.ID of node to look up."), // 字符串参数
    },
    # 定义命令行选项，包括两个字符串选项
    Options: []cmds.Option{
        cmds.StringOption(formatOptionName, "f", "Optional output format."),
        cmds.StringOption(idFormatOptionName, "Encoding used for peer IDs: Can either be a multibase encoded CID or a base58btc encoded multihash. Takes {b58mh|base36|k|base32|b...}.").WithDefault("b58mh"),
    },
    # 定义命令行执行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求中获取 peer ID 编码格式
        keyEnc, err := ke.KeyEncoderFromString(req.Options[idFormatOptionName].(string))
        if err != nil {
            return err
        }

        # 从环境中获取节点信息
        n, err := cmdenv.GetNode(env)
        if err != nil {
            return err
        }

        # 定义 peer ID 变量
        var id peer.ID
        # 如果命令行参数中包含 peer ID，则解析并赋值给 id
        if len(req.Arguments) > 0 {
            var err error
            id, err = peer.Decode(req.Arguments[0])
            if err != nil {
                return fmt.Errorf("invalid peer id")
            }
        } else {
            # 否则使用节点自身的 peer ID
            id = n.Identity
        }

        # 如果 peer ID 与节点自身的 peer ID 相同，则打印节点自身信息
        if id == n.Identity {
            output, err := printSelf(keyEnc, n)
            if err != nil {
                return err
            }
            return cmds.EmitOnce(res, output)
        }

        # 从请求中获取 offline 选项，并检查节点是否在线
        offline, _ := req.Options[OfflineOption].(bool)
        if !offline && !n.IsOnline {
            return errors.New(offlineIDErrorMessage)
        }

        # 如果不是离线模式，则需要连接到指定的 peer
        if !offline {
            # 需要实际连接以执行 identify 操作
            err = n.PeerHost.Connect(req.Context, peer.AddrInfo{ID: id})
            switch err {
            case nil:
            case kb.ErrLookupFailure:
                return errors.New(offlineIDErrorMessage)
            default:
                return err
            }
        }

        # 打印指定 peer 的信息
        output, err := printPeer(keyEnc, n.Peerstore, id)
        if err != nil {
            return err
        }
        return cmds.EmitOnce(res, output)
    },
    # 创建 Encoders 对象，包含了不同类型的编码器
    Encoders: cmds.EncoderMap{
        # 使用文本类型创建编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *IdOutput) error {
            # 从请求的选项中获取格式信息
            format, found := req.Options[formatOptionName].(string)
            # 如果找到了格式信息
            if found {
                # 根据格式信息替换输出中的占位符
                output := format
                output = strings.Replace(output, "<id>", out.ID, -1)
                output = strings.Replace(output, "<aver>", out.AgentVersion, -1)
                output = strings.Replace(output, "<pubkey>", out.PublicKey, -1)
                output = strings.Replace(output, "<addrs>", strings.Join(out.Addresses, "\n"), -1)
                output = strings.Replace(output, "<protocols>", strings.Join(protocol.ConvertToStrings(out.Protocols), "\n"), -1)
                output = strings.Replace(output, "\\n", "\n", -1)
                output = strings.Replace(output, "\\t", "\t", -1)
                # 将格式化后的输出写入到指定的写入器中
                fmt.Fprint(w, output)
            } else {
                # 如果没有找到格式信息，则将输出对象转换为 JSON 格式并进行缩进处理
                marshaled, err := json.MarshalIndent(out, "", "\t")
                if err != nil {
                    return err
                }
                # 在 JSON 格式后添加换行符
                marshaled = append(marshaled, byte('\n'))
                # 将格式化后的 JSON 输出写入到指定的写入器中
                fmt.Fprintln(w, string(marshaled))
            }
            # 返回空值表示没有错误发生
            return nil
        }),
    },
    # 设置 Type 为 IdOutput 类型的对象
    Type: IdOutput{},
// 打印对等节点信息，包括公钥、地址、协议和代理版本
func printPeer(keyEnc ke.KeyEncoder, ps pstore.Peerstore, p peer.ID) (interface{}, error) {
    // 如果对等节点为空，则返回错误
    if p == "" {
        return nil, errors.New("attempted to print nil peer")
    }

    // 创建一个新的 IdOutput 结构体
    info := new(IdOutput)
    // 将对等节点的 ID 格式化并赋值给 info.ID
    info.ID = keyEnc.FormatID(p)

    // 获取对等节点的公钥并编码为 base64 字符串
    if pk := ps.PubKey(p); pk != nil {
        pkb, err := ic.MarshalPublicKey(pk)
        if err != nil {
            return nil, err
        }
        info.PublicKey = base64.StdEncoding.EncodeToString(pkb)
    }

    // 获取对等节点的地址信息并转换为 P2P 地址
    addrInfo := ps.PeerInfo(p)
    addrs, err := peer.AddrInfoToP2pAddrs(&addrInfo)
    if err != nil {
        return nil, err
    }

    // 将地址信息添加到 info.Addresses 中并排序
    for _, a := range addrs {
        info.Addresses = append(info.Addresses, a.String())
    }
    sort.Strings(info.Addresses)

    // 获取对等节点支持的协议并添加到 info.Protocols 中并排序
    protocols, _ := ps.GetProtocols(p) // 不关心这里的错误
    info.Protocols = append(info.Protocols, protocols...)
    sort.Slice(info.Protocols, func(i, j int) bool { return info.Protocols[i] < info.Protocols[j] })

    // 获取对等节点的代理版本信息
    if v, err := ps.Get(p, "AgentVersion"); err == nil {
        if vs, ok := v.(string); ok {
            info.AgentVersion = vs
        }
    }

    // 返回对等节点信息
    return info, nil
}

// 打印自身节点信息，包括公钥
func printSelf(keyEnc ke.KeyEncoder, node *core.IpfsNode) (interface{}, error) {
    // 创建一个新的 IdOutput 结构体
    info := new(IdOutput)
    // 将节点的 ID 格式化并赋值给 info.ID
    info.ID = keyEnc.FormatID(node.Identity)

    // 获取节点的公钥并编码为 base64 字符串
    pk := node.PrivateKey.GetPublic()
    pkb, err := ic.MarshalPublicKey(pk)
    if err != nil {
        return nil, err
    }
    info.PublicKey = base64.StdEncoding.EncodeToString(pkb)
    # 如果节点的对等主机不为空
    if node.PeerHost != nil:
        # 将对等主机信息转换为P2P地址
        addrs, err := peer.AddrInfoToP2pAddrs(host.InfoFromHost(node.PeerHost))
        # 如果转换出错，则返回空和错误
        if err != nil:
            return nil, err
        # 遍历P2P地址列表，将地址字符串添加到信息结构体的地址列表中
        for _, a := range addrs:
            info.Addresses = append(info.Addresses, a.String())
        # 对地址列表进行排序
        sort.Strings(info.Addresses)
        # 获取对等主机的多路复用协议
        info.Protocols = node.PeerHost.Mux().Protocols()
        # 对协议列表进行排序
        sort.Slice(info.Protocols, func(i, j int) bool { return info.Protocols[i] < info.Protocols[j] })
    # 获取用户代理版本信息
    info.AgentVersion = version.GetUserAgentVersion()
    # 返回信息结构体和空错误
    return info, nil
# 闭合前面的函数定义
```