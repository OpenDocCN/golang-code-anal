# `kubo\core\commands\stat_dht.go`

```
package commands

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于实现 I/O 操作
    "text/tabwriter"  // 导入 text/tabwriter 包，用于格式化输出表格
    "time"  // 导入 time 包，用于处理时间

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"  // 导入自定义包 cmdenv

    cmds "github.com/ipfs/go-ipfs-cmds"  // 导入第三方包 go-ipfs-cmds
    dht "github.com/libp2p/go-libp2p-kad-dht"  // 导入第三方包 go-libp2p-kad-dht
    "github.com/libp2p/go-libp2p-kad-dht/fullrt"  // 导入 fullrt 包
    kbucket "github.com/libp2p/go-libp2p-kbucket"  // 导入第三方包 go-libp2p-kbucket
    "github.com/libp2p/go-libp2p/core/network"  // 导入 core/network 包
    pstore "github.com/libp2p/go-libp2p/core/peerstore"  // 导入 core/peerstore 包
)

type dhtPeerInfo struct {
    ID            string  // 定义 dhtPeerInfo 结构体的 ID 字段，表示节点 ID
    Connected     bool  // 定义 dhtPeerInfo 结构体的 Connected 字段，表示节点是否连接
    AgentVersion  string  // 定义 dhtPeerInfo 结构体的 AgentVersion 字段，表示节点的代理版本
    LastUsefulAt  string  // 定义 dhtPeerInfo 结构体的 LastUsefulAt 字段，表示节点最后一次有用的时间
    LastQueriedAt string  // 定义 dhtPeerInfo 结构体的 LastQueriedAt 字段，表示节点最后一次查询的时间
}

type dhtStat struct {
    Name    string  // 定义 dhtStat 结构体的 Name 字段，表示 DHT 的名称
    Buckets []dhtBucket  // 定义 dhtStat 结构体的 Buckets 字段，表示 DHT 的桶信息
}

type dhtBucket struct {
    LastRefresh string  // 定义 dhtBucket 结构体的 LastRefresh 字段，表示桶的最后刷新时间
    Peers       []dhtPeerInfo  // 定义 dhtBucket 结构体的 Peers 字段，表示桶内节点的信息
}

var statDhtCmd = &cmds.Command{
    Helptext: cmds.HelpText{  // 定义 statDhtCmd 命令的帮助文本
        Tagline: "Returns statistics about the node's DHT(s).",  // 简短的描述节点 DHT 的统计信息
        ShortDescription: `  // 详细描述节点参与的 DHT 的统计信息
Returns statistics about the DHT(s) the node is participating in.

This interface is not stable and may change from release to release.
`,
    },
    Arguments: []cmds.Argument{  // 定义 statDhtCmd 命令的参数
        cmds.StringArg("dht", false, true, "The DHT whose table should be listed (wanserver, lanserver, wan, lan). "+
            "wan and lan refer to client routing tables. When using the experimental DHT client only WAN is supported. Defaults to wan and lan."),  // 定义名为 dht 的字符串参数
    },
    Options: []cmds.Option{},  // 定义 statDhtCmd 命令的选项
    Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {  // 定义 statDhtCmd 命令的运行函数
        nd, err := cmdenv.GetNode(env)  // 获取环境中的节点信息
        if err != nil {
            return err  // 如果出现错误，返回错误信息
        }

        if !nd.IsOnline {
            return ErrNotOnline  // 如果节点不在线，返回不在线错误
        }

        if nd.DHT == nil {
            return ErrNotDHT  // 如果节点的 DHT 为空，返回不是 DHT 错误
        }

        id := kbucket.ConvertPeerID(nd.Identity)  // 将节点的标识转换为对应的 peer ID

        dhts := req.Arguments  // 获取请求中的参数
        if len(dhts) == 0 {
            dhts = []string{"wan", "lan"}  // 如果参数为空，默认为 wan 和 lan
        }

    },
    },
    Type: dhtStat{},  // 定义 statDhtCmd 命令的类型为 dhtStat
}
```