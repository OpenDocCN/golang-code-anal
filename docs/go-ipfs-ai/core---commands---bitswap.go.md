# `kubo\core\commands\bitswap.go`

```
package commands

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv" // 导入自定义包 cmdenv
    e "github.com/ipfs/kubo/core/commands/e" // 导入自定义包 e

    humanize "github.com/dustin/go-humanize" // 导入第三方包 go-humanize，用于人性化显示数据大小
    bitswap "github.com/ipfs/boxo/bitswap" // 导入自定义包 bitswap
    "github.com/ipfs/boxo/bitswap/server" // 导入 bitswap 服务器包
    cidutil "github.com/ipfs/go-cidutil" // 导入自定义包 go-cidutil
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入自定义包 go-ipfs-cmds
    peer "github.com/libp2p/go-libp2p/core/peer" // 导入 libp2p 的 peer 包
)

var BitswapCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Interact with the bitswap agent.", // 设置命令的简短描述
        ShortDescription: ``, // 设置命令的短描述
    },

    Subcommands: map[string]*cmds.Command{ // 定义子命令
        "stat":      bitswapStatCmd, // 状态子命令
        "wantlist":  showWantlistCmd, // 显示 wantlist 子命令
        "ledger":    ledgerCmd, // 账本子命令
        "reprovide": reprovideCmd, // 重新提供子命令
    },
}

const (
    peerOptionName = "peer" // 定义 peer 选项的名称
)

var showWantlistCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Show blocks currently on the wantlist.", // 设置命令的简短描述
        ShortDescription: `Print out all blocks currently on the bitswap wantlist for the local peer.`, // 设置命令的短描述
    },
    Options: []cmds.Option{ // 定义命令的选项
        cmds.StringOption(peerOptionName, "p", "Specify which peer to show wantlist for. Default: self."), // 定义字符串类型的选项 peer
    },
    Type: KeyList{}, // 定义命令的类型为 KeyList
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 定义命令的执行函数
        nd, err := cmdenv.GetNode(env) // 获取节点信息
        if err != nil {
            return err
        }

        if !nd.IsOnline { // 如果节点不在线
            return ErrNotOnline // 返回不在线错误
        }

        bs, ok := nd.Exchange.(*bitswap.Bitswap) // 获取 bitswap 对象
        if !ok {
            return e.TypeErr(bs, nd.Exchange) // 返回类型错误
        }

        pstr, found := req.Options[peerOptionName].(string) // 获取 peer 选项的值
        if found {
            pid, err := peer.Decode(pstr) // 解码 peer ID
            if err != nil {
                return err
            }
            if pid != nd.Identity { // 如果 peer ID 不等于节点的身份
                return cmds.EmitOnce(res, &KeyList{bs.WantlistForPeer(pid)}) // 发送 wantlist 列表
            }
        }

        return cmds.EmitOnce(res, &KeyList{bs.GetWantlist()}) // 发送 wantlist 列表
    },
    # 创建一个编码器映射，将文本命令映射到一个特定的编码器函数
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到一个特定的编码器函数，该函数将请求、写入器和密钥列表作为参数，并返回错误
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *KeyList) error {
            # 获取低级 CID 编码器
            enc, err := cmdenv.GetLowLevelCidEncoder(req)
            if err != nil {
                return err
            }
            # 对密钥列表进行排序
            cidutil.Sort(out.Keys)
            # 遍历密钥列表，将每个密钥编码后写入到写入器中
            for _, key := range out.Keys {
                fmt.Fprintln(w, enc.Encode(key))
            }
            # 返回空错误表示编码成功
            return nil
        }),
    },
// 定义常量，用于设置选项名称
const (
    bitswapVerboseOptionName = "verbose" // 详细信息选项名称
    bitswapHumanOptionName   = "human"   // 人类可读格式选项名称
)

// 定义 bitswapStatCmd 命令
var bitswapStatCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Show some diagnostic information on the bitswap agent.", // 显示 bitswap 代理的一些诊断信息
        ShortDescription: ``,
    },
    Options: []cmds.Option{
        cmds.BoolOption(bitswapVerboseOptionName, "v", "Print extra information"), // 添加布尔类型选项，用于打印额外信息
        cmds.BoolOption(bitswapHumanOptionName, "Print sizes in human readable format (e.g., 1K 234M 2G)"), // 添加布尔类型选项，用于以人类可读格式打印大小（例如，1K 234M 2G）
    },
    Type: bitswap.Stat{}, // 设置类型为 bitswap.Stat 结构
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        nd, err := cmdenv.GetNode(env) // 获取节点信息
        if err != nil {
            return err
        }

        if !nd.IsOnline { // 如果节点不在线
            return cmds.Errorf(cmds.ErrClient, ErrNotOnline.Error()) // 返回错误信息
        }

        bs, ok := nd.Exchange.(*bitswap.Bitswap) // 获取 bitswap.Bitswap 对象
        if !ok {
            return e.TypeErr(bs, nd.Exchange) // 返回类型错误
        }

        st, err := bs.Stat() // 获取 bitswap 统计信息
        if err != nil {
            return err
        }

        return cmds.EmitOnce(res, st) // 发送统计信息
    },
}
    # 创建 Encoders 对象，包含不同类型命令的编码器
    Encoders: cmds.EncoderMap{
        # 为文本类型命令创建特定类型的编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, s *bitswap.Stat) error {
            # 获取低级别的 CID 编码器
            enc, err := cmdenv.GetLowLevelCidEncoder(req)
            if err != nil:
                return err
            # 获取是否启用了 bitswapVerboseOptionName 选项
            verbose, _ := req.Options[bitswapVerboseOptionName].(bool)
            # 获取是否启用了 bitswapHumanOptionName 选项
            human, _ := req.Options[bitswapHumanOptionName].(bool)

            # 输出 bitswap 状态信息
            fmt.Fprintln(w, "bitswap status")
            # 输出提供缓冲区的大小
            fmt.Fprintf(w, "\tprovides buffer: %d / %d\n", s.ProvideBufLen, bitswap.HasBlockBufferSize)
            # 输出接收到的块数
            fmt.Fprintf(w, "\tblocks received: %d\n", s.BlocksReceived)
            # 输出发送的块数
            fmt.Fprintf(w, "\tblocks sent: %d\n", s.BlocksSent)
            # 根据是否启用了 human 选项输出接收到的数据大小
            if human:
                fmt.Fprintf(w, "\tdata received: %s\n", humanize.Bytes(s.DataReceived))
                fmt.Fprintf(w, "\tdata sent: %s\n", humanize.Bytes(s.DataSent))
            else:
                fmt.Fprintf(w, "\tdata received: %d\n", s.DataReceived)
                fmt.Fprintf(w, "\tdata sent: %d\n", s.DataSent)
            # 输出接收到的重复块数
            fmt.Fprintf(w, "\tdup blocks received: %d\n", s.DupBlksReceived)
            # 根据是否启用了 human 选项输出接收到的重复数据大小
            if human:
                fmt.Fprintf(w, "\tdup data received: %s\n", humanize.Bytes(s.DupDataReceived))
            else:
                fmt.Fprintf(w, "\tdup data received: %d\n", s.DupDataReceived)
            # 输出 wantlist 的键数
            fmt.Fprintf(w, "\twantlist [%d keys]\n", len(s.Wantlist))
            # 遍历 wantlist 并输出每个键的编码值
            for _, k := range s.Wantlist:
                fmt.Fprintf(w, "\t\t%s\n", enc.Encode(k))
            # 输出 peers 的数量
            fmt.Fprintf(w, "\tpartners [%d]\n", len(s.Peers))
            # 根据是否启用了 verbose 选项输出每个 peer 的信息
            if verbose:
                for _, p := range s.Peers:
                    fmt.Fprintf(w, "\t\t%s\n", p)
            # 返回空值
            return nil
        }),
    },
// 定义名为ledgerCmd的命令，用于显示给定对等节点的当前分类帐
var ledgerCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Show the current ledger for a peer.", // 帮助文本的一句简短描述
        ShortDescription: `
The Bitswap decision engine tracks the number of bytes exchanged between IPFS
nodes, and stores this information as a collection of ledgers. This command
prints the ledger associated with a given peer.
`, // 帮助文本的详细描述
    },
    Arguments: []cmds.Argument{
        cmds.StringArg("peer", true, false, "The PeerID (B58) of the ledger to inspect."), // 命令的参数，包括参数名、是否必需、是否可变、参数描述
    },
    Type: server.Receipt{}, // 命令的类型
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error { // 命令的执行函数
        nd, err := cmdenv.GetNode(env) // 获取节点
        if err != nil {
            return err
        }

        if !nd.IsOnline { // 如果节点不在线
            return ErrNotOnline
        }

        bs, ok := nd.Exchange.(*bitswap.Bitswap) // 获取并检查节点的交换引擎类型
        if !ok {
            return e.TypeErr(bs, nd.Exchange)
        }

        partner, err := peer.Decode(req.Arguments[0]) // 解码给定的对等节点ID
        if err != nil {
            return err
        }

        return cmds.EmitOnce(res, bs.LedgerForPeer(partner)) // 发送一次命令的执行结果
    },
    Encoders: cmds.EncoderMap{ // 命令的编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *server.Receipt) error { // 文本编码器
            fmt.Fprintf(w, "Ledger for %s\n"+
                "Debt ratio:\t%f\n"+
                "Exchanges:\t%d\n"+
                "Bytes sent:\t%d\n"+
                "Bytes received:\t%d\n\n",
                out.Peer, out.Value, out.Exchanged,
                out.Sent, out.Recv) // 格式化输出结果
            return nil
        }),
    },
}

// 定义名为reprovideCmd的命令，用于触发重新提供程序
var reprovideCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline: "Trigger reprovider.", // 帮助文本的一句简短描述
        ShortDescription: `
Trigger reprovider to announce our data to network.
`, // 帮助文本的详细描述
    },
}
    # 定义一个名为 Run 的方法，接收请求、响应和环境参数，并返回错误
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从环境中获取节点信息
        nd, err := cmdenv.GetNode(env)
        # 如果获取节点信息出错，则返回错误
        if err != nil {
            return err
        }

        # 如果节点不在线，则返回错误
        if !nd.IsOnline {
            return ErrNotOnline
        }

        # 重新提供节点服务
        err = nd.Provider.Reprovide(req.Context)
        # 如果重新提供服务出错，则返回错误
        if err != nil {
            return err
        }

        # 如果没有出错，则返回空
        return nil
    },
# 闭合前面的函数定义
```