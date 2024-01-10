# `kubo\core\commands\stat.go`

```
package commands

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "io" // 导入 io 包，用于实现 I/O 操作
    "os" // 导入 os 包，提供对操作系统功能的访问
    "time" // 导入 time 包，用于处理时间

    cmdenv "github.com/ipfs/kubo/core/commands/cmdenv" // 导入自定义包 cmdenv

    humanize "github.com/dustin/go-humanize" // 导入第三方包，用于格式化数据大小
    cmds "github.com/ipfs/go-ipfs-cmds" // 导入自定义包 cmds
    metrics "github.com/libp2p/go-libp2p/core/metrics" // 导入第三方包，用于获取 libp2p 核心度量
    peer "github.com/libp2p/go-libp2p/core/peer" // 导入第三方包，用于处理 libp2p 核心对等节点
    protocol "github.com/libp2p/go-libp2p/core/protocol" // 导入第三方包，用于处理 libp2p 核心协议
)

var StatsCmd = &cmds.Command{ // 定义名为 StatsCmd 的命令
    Helptext: cmds.HelpText{ // 命令的帮助文本
        Tagline: "Query IPFS statistics.", // 命令的简短描述
        ShortDescription: `'ipfs stats' is a set of commands to help look at statistics
for your IPFS node.
`, // 命令的简短描述
        LongDescription: `'ipfs stats' is a set of commands to help look at statistics
for your IPFS node.`, // 命令的详细描述
    },

    Subcommands: map[string]*cmds.Command{ // 定义子命令
        "bw":      statBwCmd, // 子命令 "bw" 对应 statBwCmd
        "repo":    repoStatCmd, // 子命令 "repo" 对应 repoStatCmd
        "bitswap": bitswapStatCmd, // 子命令 "bitswap" 对应 bitswapStatCmd
        "dht":     statDhtCmd, // 子命令 "dht" 对应 statDhtCmd
        "provide": statProvideCmd, // 子命令 "provide" 对应 statProvideCmd
    },
}

const ( // 定义常量
    statPeerOptionName     = "peer" // 定义名为 statPeerOptionName 的常量
    statProtoOptionName    = "proto" // 定义名为 statProtoOptionName 的常量
    statPollOptionName     = "poll" // 定义名为 statPollOptionName 的常量
    statIntervalOptionName = "interval" // 定义名为 statIntervalOptionName 的常量
)

var statBwCmd = &cmds.Command{ // 定义名为 statBwCmd 的命令
    Helptext: cmds.HelpText{ // 命令的帮助文本
        Tagline: "Print IPFS bandwidth information.", // 命令的简短描述
        ShortDescription: `'ipfs stats bw' prints bandwidth information for the ipfs daemon.
It displays: TotalIn, TotalOut, RateIn, RateOut.
        `, // 命令的简短描述
        LongDescription: `'ipfs stats bw' prints bandwidth information for the ipfs daemon.
It displays: TotalIn, TotalOut, RateIn, RateOut.

By default, overall bandwidth and all protocols are shown. To limit bandwidth
to a particular peer, use the 'peer' option along with that peer's multihash
id. To specify a specific protocol, use the 'proto' option. The 'peer' and
'proto' options cannot be specified simultaneously. The protocols that are
queried using this method are outlined in the specification:
https://github.com/libp2p/specs/blob/master/7-properties.md#757-protocol-multicodecs
`, // 命令的详细描述
    },
}
Example protocol options:
  - /ipfs/id/1.0.0
  - /ipfs/bitswap
  - /ipfs/dht


示例协议选项，包括/ipfs/id/1.0.0、/ipfs/bitswap和/ipfs/dht。


Example:

    > ipfs stats bw -t /ipfs/bitswap
    Bandwidth
    TotalIn: 5.0MB
    TotalOut: 0B
    RateIn: 343B/s
    RateOut: 0B/s
    > ipfs stats bw -p QmepgFW7BHEtU4pZJdxaNiv75mKLLRQnPi1KaaXmQN4V1a
    Bandwidth
    TotalIn: 4.9MB
    TotalOut: 12MB
    RateIn: 0B/s
    RateOut: 0B/s


示例用法，展示了使用ipfs stats bw命令来查看带宽使用情况的示例。


Options: []cmds.Option{
    cmds.StringOption(statPeerOptionName, "p", "Specify a peer to print bandwidth for."),
    cmds.StringOption(statProtoOptionName, "t", "Specify a protocol to print bandwidth for."),
    cmds.BoolOption(statPollOptionName, "Print bandwidth at an interval."),
    cmds.StringOption(statIntervalOptionName, "i", `Time interval to wait between updating output, if 'poll' is true.

    This accepts durations such as "300s", "1.5h" or "2h45m". Valid time units are:
    "ns", "us" (or "µs"), "ms", "s", "m", "h".`).WithDefault("1s"),
},


命令选项，包括指定对特定对等节点打印带宽、指定对特定协议打印带宽、在间隔期间打印带宽以及指定更新输出之间的时间间隔。


Type: metrics.Stats{},


类型定义，表示metrics.Stats类型的空实例。
    # 定义 PostRunMap 结构体的 CLI 字段，对应的值是一个匿名函数
    PostRun: cmds.PostRunMap{
        cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
            # 从请求的选项中获取 statPollOptionName 对应的值，转换为布尔类型
            polling, _ := res.Request().Options[statPollOptionName].(bool)

            # 如果需要轮询，则在标准输出中打印标题
            if polling {
                fmt.Fprintln(os.Stdout, "Total Up    Total Down  Rate Up     Rate Down")
            }
            # 循环处理响应中的数据
            for {
                v, err := res.Next()
                # 如果出现错误
                if err != nil {
                    # 如果是文件结束错误，则返回空
                    if err == io.EOF {
                        return nil
                    }
                    # 否则返回错误
                    return err
                }

                # 将数据转换为 metrics.Stats 类型
                bs := v.(*metrics.Stats)

                # 如果不需要轮询，则在标准输出中打印统计数据并返回空
                if !polling {
                    printStats(os.Stdout, bs)
                    return nil
                }

                # 如果需要轮询，则在标准输出中打印统计数据
                fmt.Fprintf(os.Stdout, "%8s    ", humanize.Bytes(uint64(bs.TotalOut)))
                fmt.Fprintf(os.Stdout, "%8s    ", humanize.Bytes(uint64(bs.TotalIn)))
                fmt.Fprintf(os.Stdout, "%8s/s  ", humanize.Bytes(uint64(bs.RateOut)))
                fmt.Fprintf(os.Stdout, "%8s/s      \r", humanize.Bytes(uint64(bs.RateIn)))
            }
        },
    },
# 打印统计信息到指定的输出流
func printStats(out io.Writer, bs *metrics.Stats) {
    # 打印标题 "Bandwidth"
    fmt.Fprintln(out, "Bandwidth")
    # 打印总输入数据量
    fmt.Fprintf(out, "TotalIn: %s\n", humanize.Bytes(uint64(bs.TotalIn)))
    # 打印总输出数据量
    fmt.Fprintf(out, "TotalOut: %s\n", humanize.Bytes(uint64(bs.TotalOut)))
    # 打印输入速率
    fmt.Fprintf(out, "RateIn: %s/s\n", humanize.Bytes(uint64(bs.RateIn)))
    # 打印输出速率
    fmt.Fprintf(out, "RateOut: %s/s\n", humanize.Bytes(uint64(bs.RateOut)))
}
```