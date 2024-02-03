# `kubo\core\commands\bootstrap.go`

```go
package commands

import (
    "errors"  // 导入错误处理包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "sort"  // 导入排序包

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入命令环境包
    repo "github.com/ipfs/kubo/repo"  // 导入存储库包
    fsrepo "github.com/ipfs/kubo/repo/fsrepo"  // 导入文件系统存储库包

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入命令包
    config "github.com/ipfs/kubo/config"  // 导入配置包
    peer "github.com/libp2p/go-libp2p/core/peer"  // 导入对等节点包
    ma "github.com/multiformats/go-multiaddr"  // 导入多地址包
)

type BootstrapOutput struct {
    Peers []string  // 定义输出结构体，包含对等节点的字符串数组
}

var peerOptionDesc = "A peer to add to the bootstrap list (in the format '<multiaddr>/<peerID>')"  // 定义对等节点描述信息

var BootstrapCmd = &cmds.Command{  // 定义命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Show or edit the list of bootstrap peers.",  // 简短描述
        ShortDescription: `
Running 'ipfs bootstrap' with no arguments will run 'ipfs bootstrap list'.
` + bootstrapSecurityWarning,  // 短描述，包含安全警告
    },

    Run:      bootstrapListCmd.Run,  // 运行命令
    Encoders: bootstrapListCmd.Encoders,  // 编码器
    Type:     bootstrapListCmd.Type,  // 类型

    Subcommands: map[string]*cmds.Command{  // 子命令
        "list": bootstrapListCmd,  // 列出命令
        "add":  bootstrapAddCmd,  // 添加命令
        "rm":   bootstrapRemoveCmd,  // 移除命令
    },
}

const (
    defaultOptionName = "default"  // 默认选项名称
)

var bootstrapAddCmd = &cmds.Command{  // 添加命令
    Helptext: cmds.HelpText{  // 帮助文本
        Tagline: "Add peers to the bootstrap list.",  // 简短描述
        ShortDescription: `Outputs a list of peers that were added (that weren't already
in the bootstrap list).
` + bootstrapSecurityWarning,  // 短描述，包含安全警告
    },

    Arguments: []cmds.Argument{  // 参数
        cmds.StringArg("peer", false, true, peerOptionDesc).EnableStdin(),  // 字符串参数
    },

    Options: []cmds.Option{  // 选项
        cmds.BoolOption(defaultOptionName, "Add default bootstrap nodes. (Deprecated, use 'default' subcommand instead)"),  // 布尔选项
    },
    Subcommands: map[string]*cmds.Command{  // 子命令
        "default": bootstrapAddDefaultCmd,  // 默认添加命令
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求选项中获取默认值，如果不存在则使用默认值
        deflt, _ := req.Options[defaultOptionName].(bool)

        # 设置输入对等节点的默认值为配置文件中的默认引导地址
        inputPeers := config.DefaultBootstrapAddresses
        # 如果不是默认值
        if !deflt {
            # 解析请求的主体参数
            if err := req.ParseBodyArgs(); err != nil {
                return err
            }
            # 使用请求参数作为输入对等节点
            inputPeers = req.Arguments
        }

        # 如果输入对等节点数量为0，则返回错误
        if len(inputPeers) == 0 {
            return errors.New("no bootstrap peers to add")
        }

        # 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        # 打开文件系统存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        # 延迟关闭文件系统存储库
        defer r.Close()
        # 获取存储库配置
        cfg, err := r.Config()
        if err != nil {
            return err
        }

        # 添加引导节点
        added, err := bootstrapAdd(r, cfg, inputPeers)
        if err != nil {
            return err
        }

        # 发送一次性结果
        return cmds.EmitOnce(res, &BootstrapOutput{added})
    },
    # 类型为 BootstrapOutput 结构
    Type: BootstrapOutput{},
    # 编码器映射
    Encoders: cmds.EncoderMap{
        # 文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *BootstrapOutput) error {
            return bootstrapWritePeers(w, "added ", out.Peers)
        }),
    },
}

// 定义一个名为bootstrapAddDefaultCmd的命令对象
var bootstrapAddDefaultCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Add default peers to the bootstrap list.", // 简短描述
        ShortDescription: `Outputs a list of peers that were added (that weren't already
in the bootstrap list).`, // 详细描述
    },
    // 运行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        // 打开文件系统存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }

        // 延迟关闭文件系统存储库
        defer r.Close()
        // 获取配置信息
        cfg, err := r.Config()
        if err != nil {
            return err
        }

        // 添加默认引导地址
        added, err := bootstrapAdd(r, cfg, config.DefaultBootstrapAddresses)
        if err != nil {
            return err
        }

        // 发射一次结果
        return cmds.EmitOnce(res, &BootstrapOutput{added})
    },
    Type: BootstrapOutput{}, // 类型
    Encoders: cmds.EncoderMap{ // 编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *BootstrapOutput) error {
            return bootstrapWritePeers(w, "added ", out.Peers)
        }),
    },
}

// 常量
const (
    bootstrapAllOptionName = "all"
)

// 定义一个名为bootstrapRemoveCmd的命令对象
var bootstrapRemoveCmd = &cmds.Command{
    // 帮助文本
    Helptext: cmds.HelpText{
        Tagline: "Remove peers from the bootstrap list.", // 简短描述
        ShortDescription: `Outputs the list of peers that were removed.
` + bootstrapSecurityWarning, // 详细描述
    },
    // 参数
    Arguments: []cmds.Argument{
        cmds.StringArg("peer", false, true, peerOptionDesc).EnableStdin(),
    },
    // 选项
    Options: []cmds.Option{
        cmds.BoolOption(bootstrapAllOptionName, "Remove all bootstrap peers. (Deprecated, use 'all' subcommand)"),
    },
    // 子命令
    Subcommands: map[string]*cmds.Command{
        "all": bootstrapRemoveAllCmd,
    },
    # 运行函数，处理请求并返回结果
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        # 从请求选项中获取是否为全部引导的标志
        all, _ := req.Options[bootstrapAllOptionName].(bool)

        # 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        # 打开文件系统存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        # 延迟关闭文件系统存储库
        defer r.Close()
        # 获取存储库配置
        cfg, err := r.Config()
        if err != nil {
            return err
        }

        # 初始化一个空的字符串数组来存储被移除的内容
        var removed []string
        # 如果是全部引导，则调用bootstrapRemoveAll函数
        if all {
            removed, err = bootstrapRemoveAll(r, cfg)
        } else {
            # 否则解析请求的主体参数
            if err := req.ParseBodyArgs(); err != nil {
                return err
            }
            # 调用bootstrapRemove函数来移除指定的内容
            removed, err = bootstrapRemove(r, cfg, req.Arguments)
        }
        # 如果有错误则返回错误
        if err != nil {
            return err
        }

        # 发射一次结果
        return cmds.EmitOnce(res, &BootstrapOutput{removed})
    },
    # 类型为BootstrapOutput
    Type: BootstrapOutput{},
    # 编码器映射
    Encoders: cmds.EncoderMap{
        # 文本编码器
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *BootstrapOutput) error {
            return bootstrapWritePeers(w, "removed ", out.Peers)
        }),
    },
// 定义一个命令，用于从引导列表中移除所有对等点
var bootstrapRemoveAllCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Remove all peers from the bootstrap list.", // 简短描述：从引导列表中移除所有对等点
        ShortDescription: `Outputs the list of peers that were removed.`, // 输出已移除对等点的列表
    },

    // 执行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        // 打开文件系统存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        defer r.Close() // 延迟关闭存储库
        cfg, err := r.Config() // 获取存储库配置
        if err != nil {
            return err
        }

        // 移除引导对等点
        removed, err := bootstrapRemoveAll(r, cfg)
        if err != nil {
            return err
        }

        // 发射一次结果
        return cmds.EmitOnce(res, &BootstrapOutput{removed})
    },
    Type: BootstrapOutput{}, // 类型为 BootstrapOutput
    Encoders: cmds.EncoderMap{ // 编码器映射
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *BootstrapOutput) error {
            return bootstrapWritePeers(w, "removed ", out.Peers) // 写入已移除对等点
        }),
    },
}

// 定义一个命令，用于显示引导列表中的对等点
var bootstrapListCmd = &cmds.Command{
    Helptext: cmds.HelpText{
        Tagline:          "Show peers in the bootstrap list.", // 简短描述：显示引导列表中的对等点
        ShortDescription: "Peers are output in the format '<multiaddr>/<peerID>'.", // 对等点以'<multiaddr>/<peerID>'格式输出
    },

    // 执行函数
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
        // 获取配置根目录
        cfgRoot, err := cmdenv.GetConfigRoot(env)
        if err != nil {
            return err
        }

        // 打开文件系统存储库
        r, err := fsrepo.Open(cfgRoot)
        if err != nil {
            return err
        }
        defer r.Close() // 延迟关闭存储库
        cfg, err := r.Config() // 获取存储库配置
        if err != nil {
            return err
        }

        // 获取引导对等点
        peers, err := cfg.BootstrapPeers()
        if err != nil {
            return err
        }

        // 发射一次结果
        return cmds.EmitOnce(res, &BootstrapOutput{config.BootstrapPeerStrings(peers)})
    },
    Type: BootstrapOutput{}, // 类型为 BootstrapOutput
}
    # 创建 Encoders 对象，包含不同类型命令的编码器映射
    Encoders: cmds.EncoderMap{
        # 将文本命令映射到特定的编码器函数
        cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *BootstrapOutput) error {
            # 调用编码器函数，将输出对等点写入到指定的写入器中
            return bootstrapWritePeers(w, "", out.Peers)
        }),
    },
}
// 将对等节点列表按字母顺序排序
func bootstrapWritePeers(w io.Writer, prefix string, peers []string) error {
    sort.Stable(sort.StringSlice(peers))
    // 遍历对等节点列表，将每个节点写入到指定的写入器中
    for _, peer := range peers {
        _, err := w.Write([]byte(prefix + peer + "\n"))
        if err != nil {
            return err
        }
    }
    return nil
}

// 向存储库添加引导节点
func bootstrapAdd(r repo.Repo, cfg *config.Config, peers []string) ([]string, error) {
    // 遍历引导节点列表，验证地址格式和协议
    for _, p := range peers {
        m, err := ma.NewMultiaddr(p)
        if err != nil {
            return nil, err
        }
        tpt, p2ppart := ma.SplitLast(m)
        if p2ppart == nil || p2ppart.Protocol().Code != ma.P_P2P {
            return nil, fmt.Errorf("invalid bootstrap address: %s", p)
        }
        if tpt == nil {
            return nil, fmt.Errorf("bootstrap address without a transport: %s", p)
        }
    }

    // 初始化已添加的引导节点映射和列表
    addedMap := map[string]struct{}{}
    addedList := make([]string, 0, len(peers)

    // 保存配置中的引导节点，以便去重
    bpeers := cfg.Bootstrap
    cfg.Bootstrap = nil

    // 添加新的引导节点
    for _, s := range peers {
        if _, found := addedMap[s]; found {
            continue
        }

        cfg.Bootstrap = append(cfg.Bootstrap, s)
        addedList = append(addedList, s)
        addedMap[s] = struct{}{}
    }

    // 将原始引导节点重新添加到配置中，以便按顺序输出
    for _, s := range bpeers {
        if _, found := addedMap[s]; found {
            continue
        }

        cfg.Bootstrap = append(cfg.Bootstrap, s)
        addedMap[s] = struct{}{}
    }

    // 将更新后的配置保存到存储库中
    if err := r.SetConfig(cfg); err != nil {
        return nil, err
    }

    return addedList, nil
}

// 从存储库中移除引导节点
func bootstrapRemove(r repo.Repo, cfg *config.Config, toRemove []string) ([]string, error) {
    // 初始化已移除的节点列表和保留的节点列表
    removed := make([]peer.AddrInfo, 0, len(toRemove))
    keep := make([]peer.AddrInfo, 0, len(cfg.Bootstrap))

    // 解析要移除的引导节点地址
    toRemoveAddr, err := config.ParseBootstrapPeers(toRemove)
    if err != nil {
        return nil, err
    }
    // 创建一个映射，用于存储需要移除的对等节点ID和对应的多地址列表
    toRemoveMap := make(map[peer.ID][]ma.Multiaddr, len(toRemoveAddr))
    // 遍历需要移除的地址列表，将对等节点ID和多地址列表存储到映射中
    for _, addr := range toRemoveAddr {
        toRemoveMap[addr.ID] = addr.Addrs
    }

    // 获取引导节点列表
    peers, err := cfg.BootstrapPeers()
    if err != nil {
        return nil, err
    }

    // 遍历引导节点列表
    for _, p := range peers {
        // 检查对等节点是否在移除集合中
        addrs, ok := toRemoveMap[p.ID]
        // 如果不在移除集合中，则保留该对等节点
        if !ok {
            keep = append(keep, p)
            continue
        }
        // 如果需要移除整个对等节点
        if len(addrs) == 0 {
            removed = append(removed, p)
            continue
        }
        var keptAddrs, removedAddrs []ma.Multiaddr
        // 遍历对等节点的地址列表，移除特定的地址
    filter:
        for _, addr := range p.Addrs {
            for _, addr2 := range addrs {
                if addr.Equal(addr2) {
                    removedAddrs = append(removedAddrs, addr)
                    continue filter
                }
            }
            keptAddrs = append(keptAddrs, addr)
        }
        // 如果有移除的地址，则将对等节点ID和移除的地址列表添加到移除列表中
        if len(removedAddrs) > 0 {
            removed = append(removed, peer.AddrInfo{ID: p.ID, Addrs: removedAddrs})
        }
        // 如果有保留的地址，则将对等节点ID和保留的地址列表添加到保留列表中
        if len(keptAddrs) > 0 {
            keep = append(keep, peer.AddrInfo{ID: p.ID, Addrs: keptAddrs})
        }
    }
    // 设置新的引导节点列表
    cfg.SetBootstrapPeers(keep)

    // 更新配置
    if err := r.SetConfig(cfg); err != nil {
        return nil, err
    }

    // 返回移除的引导节点列表和错误为nil
    return config.BootstrapPeerStrings(removed), nil
// 从存储库中删除所有引导节点，并返回被删除的引导节点列表
func bootstrapRemoveAll(r repo.Repo, cfg *config.Config) ([]string, error) {
    // 获取配置文件中的引导节点列表
    removed, err := cfg.BootstrapPeers()
    if err != nil {
        return nil, err
    }

    // 清空配置文件中的引导节点列表
    cfg.Bootstrap = nil
    // 更新存储库中的配置文件
    if err := r.SetConfig(cfg); err != nil {
        return nil, err
    }
    // 返回被删除的引导节点列表
    return config.BootstrapPeerStrings(removed), nil
}

// 引导安全警告信息
const bootstrapSecurityWarning = `
SECURITY WARNING:

The bootstrap command manipulates the "bootstrap list", which contains
the addresses of bootstrap nodes. These are the *trusted peers* from
which to learn about other peers in the network. Only edit this list
if you understand the risks of adding or removing nodes from this list.

`
```