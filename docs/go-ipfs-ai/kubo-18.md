# go-ipfs 源码解析 18

# `core/commands/shutdown.go`

这段代码定义了一个命令行工具 called "daemonShutdownCmd"，用于关闭 IPFS(InterPlanetary File System) 的本地 daemon 进程。

具体来说，这个工具接受一个 CMDS (Command and Message System) request，请求的参数是一个表示是否要关闭 daemon 的布尔值，以及一个表示要关闭的 IPFS 节点 ID。

如果请求的参数为真，则会执行以下操作：

1. 获取要关闭的 IPFS 节点的环境(Environment)对象。
2. 关闭 IPFS 本地 daemon。
3. 返回没有错误。

如果请求的参数有误，则会返回一个错误消息。

此外，代码中还定义了一个辅助函数 cmdenv，该函数用于获取当前正在运行的 IPFS daemon 的节点 ID。


```go
package commands

import (
	cmds "github.com/ipfs/go-ipfs-cmds"
	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"
)

var daemonShutdownCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Shut down the IPFS daemon.",
	},
	Run: func(req *cmds.Request, re cmds.ResponseEmitter, env cmds.Environment) error {
		nd, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !nd.IsDaemon {
			return cmds.Errorf(cmds.ErrClient, "daemon not running")
		}

		if err := nd.Close(); err != nil {
			log.Error("error while shutting down ipfs daemon:", err)
		}

		return nil
	},
}

```

# `core/commands/stat.go`

这段代码定义了一个名为“commands”的包，它属于一个名为“ipfs-io”的包。通过导入其他一些包，如“fmt”、“io”、“os”和“time”，它实现了在命令行中执行IPFS命令的一些功能。

具体来说，这段代码的作用是：

1. 导入定义在“cmdenv”包中的“cmdenv”函数，以便在命令行中使用；
2. 通过导入定义在“humanize”包中的“humanize”函数，对命令行中的输出进行解析；
3. 通过导入定义在“cmds”包中的“cmds”函数，实现一些与IPFS命令行相关的操作；
4. 通过导入定义在“metrics”包中的“metrics”函数，实现一些与IPFS命令行相关的统计功能；
5. 通过导入定义在“peer”包中的“peer”函数，实现与IPFS-PEer 服务器通信的功能；
6. 通过导入定义在“protocol”包中的“protocol”函数，实现与IPFS服务器通信的功能。


```go
package commands

import (
	"fmt"
	"io"
	"os"
	"time"

	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"

	humanize "github.com/dustin/go-humanize"
	cmds "github.com/ipfs/go-ipfs-cmds"
	metrics "github.com/libp2p/go-libp2p/core/metrics"
	peer "github.com/libp2p/go-libp2p/core/peer"
	protocol "github.com/libp2p/go-libp2p/core/protocol"
)

```

此代码定义了一个名为 StatsCmd 的命令对象，用于执行 IPFS(InterPlanetary File System)统计查询。

该命令对象的 Helptext 属性指定了该命令的文档字符串，其中包含了该命令的短描述和 Long 描述。短描述和 Long 描述都描述了该命令的作用，即查询 IPFS 节点的统计信息。

该命令对象的 Subcommands 属性指定了该命令可以拥有的子命令。在这个例子中，该命令对象可以拥有 6 个子命令，分别为 "bw"、"repo"、"bitswap"、"dht" 和 "provide"。这些子命令都继承自 cmds.Command 类型，并且实现了命令的执行逻辑。

总的来说，这个代码定义了一个用于查询 IPFS 节点的统计信息的命令对象，可以接受一些子命令来实现不同的查询操作。


```go
var StatsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Query IPFS statistics.",
		ShortDescription: `'ipfs stats' is a set of commands to help look at statistics
for your IPFS node.
`,
		LongDescription: `'ipfs stats' is a set of commands to help look at statistics
for your IPFS node.`,
	},

	Subcommands: map[string]*cmds.Command{
		"bw":      statBwCmd,
		"repo":    repoStatCmd,
		"bitswap": bitswapStatCmd,
		"dht":     statDhtCmd,
		"provide": statProvideCmd,
	},
}

```

此代码定义了几个常量，用于配置 ipfs 命令行界面（CLI）的统计选项。

1. `statPeerOptionName`：统计同构类（peer）选项的名称。
2. `statProtoOptionName`：统计自定义协议（proto）选项的名称。
3. `statPollOptionName`：统计自定义轮询（poll）选项的名称。
4. `statIntervalOptionName`：统计自定义定时器（interval）选项的名称。

然后，定义了一个名为 `statBwCmd` 的变量，它是一个 `cmds.Command` 类型的变量，用于存储 ipfs 统计命令的实例。该命令的帮助文本显示了如何使用此命令来打印 ipfs 统计信息，包括带宽。

最后，导入了 `cmds.Group`、`cmds.Args` 和 `cmds.Color` 等 cmds 系统的类，以便在命令行中使用这些选项。


```go
const (
	statPeerOptionName     = "peer"
	statProtoOptionName    = "proto"
	statPollOptionName     = "poll"
	statIntervalOptionName = "interval"
)

var statBwCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Print IPFS bandwidth information.",
		ShortDescription: `'ipfs stats bw' prints bandwidth information for the ipfs daemon.
It displays: TotalIn, TotalOut, RateIn, RateOut.
		`,
		LongDescription: `'ipfs stats bw' prints bandwidth information for the ipfs daemon.
It displays: TotalIn, TotalOut, RateIn, RateOut.

```

该代码是一个命令行工具，用于显示基于IPFS（InterPlanetary File System）的对等网络中的带宽使用情况。它允许用户通过指定“peer”选项来将带宽限制到特定的对等节点，同时通过指定“proto”选项来限制显示的协议。

根据libp2p协议栈文档，该工具可以显示以下内容：

* 带宽使用情况（total bandwidth and usage）
* 当前网络中的IPFS客户端的带宽使用情况
* 限速到指定对等节点的带宽使用情况（peer选项）
* 显示特定的协议（proto选项）

通过“ipfs stats bw”命令，用户可以查看当前网络中所有对等节点的带宽使用情况。通过“peer”选项指定限制带宽的对等节点，通过“proto”选项指定限制显示的协议。例如，要限制带宽只显示HTTP协议，可以同时指定“peer”和“proto”选项。


```go
By default, overall bandwidth and all protocols are shown. To limit bandwidth
to a particular peer, use the 'peer' option along with that peer's multihash
id. To specify a specific protocol, use the 'proto' option. The 'peer' and
'proto' options cannot be specified simultaneously. The protocols that are
queried using this method are outlined in the specification:
https://github.com/libp2p/specs/blob/master/7-properties.md#757-protocol-multicodecs

Example protocol options:
  - /ipfs/id/1.0.0
  - /ipfs/bitswap
  - /ipfs/dht

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
```

This appears to be a Go program that烧录 Beats and collects metrics about them. It uses the `time.Chan` from the `time` package to handle the `<-time.After` and `<-req.Context.Done()` error cases.

The program has a single function, `doPoll`, which is a boolean that indicates whether to perform a恒定时器 Polling. If it is not `doPoll`, the program will return immediately if it is pressed. If `doPoll` is true, the program will perform a Polling操作 every `interval` seconds.

The program has a `PostRun` method, which is a Map of all the available `<-cmds.Response, <-reqs.ResponseEmitter<` commands that can be executed. The `<-cmds.Response` is a field that can be used to emit a `<-response.Response` response for each command, while the `<-reqs.ResponseEmitter` is a field that can be used to emit a `<-response.Response` response for each command.

The program starts by printing the statistics of all the Beats that are currently active. If `doPoll` is `true`, the program will start a Polling operation every `interval` seconds. For each beat, the program will print the statistics of the beat and perform any necessary Beats.

If `doPoll` is `false`, the program will return immediately if it is pressed. If `doPoll` is `true`, the program will perform a Polling operation every `interval` seconds.


```go
`,
	},
	Options: []cmds.Option{
		cmds.StringOption(statPeerOptionName, "p", "Specify a peer to print bandwidth for."),
		cmds.StringOption(statProtoOptionName, "t", "Specify a protocol to print bandwidth for."),
		cmds.BoolOption(statPollOptionName, "Print bandwidth at an interval."),
		cmds.StringOption(statIntervalOptionName, "i", `Time interval to wait between updating output, if 'poll' is true.

    This accepts durations such as "300s", "1.5h" or "2h45m". Valid time units are:
    "ns", "us" (or "µs"), "ms", "s", "m", "h".`).WithDefault("1s"),
	},

	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		nd, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		// Must be online!
		if !nd.IsOnline {
			return cmds.Errorf(cmds.ErrClient, ErrNotOnline.Error())
		}

		if nd.Reporter == nil {
			return fmt.Errorf("bandwidth reporter disabled in config")
		}

		pstr, pfound := req.Options[statPeerOptionName].(string)
		tstr, tfound := req.Options["proto"].(string)
		if pfound && tfound {
			return cmds.Errorf(cmds.ErrClient, "please only specify peer OR protocol")
		}

		var pid peer.ID
		if pfound {
			checkpid, err := peer.Decode(pstr)
			if err != nil {
				return err
			}
			pid = checkpid
		}

		timeS, _ := req.Options[statIntervalOptionName].(string)
		interval, err := time.ParseDuration(timeS)
		if err != nil {
			return err
		}

		doPoll, _ := req.Options[statPollOptionName].(bool)
		for {
			if pfound {
				stats := nd.Reporter.GetBandwidthForPeer(pid)
				if err := res.Emit(&stats); err != nil {
					return err
				}
			} else if tfound {
				protoID := protocol.ID(tstr)
				stats := nd.Reporter.GetBandwidthForProtocol(protoID)
				if err := res.Emit(&stats); err != nil {
					return err
				}
			} else {
				totals := nd.Reporter.GetBandwidthTotals()
				if err := res.Emit(&totals); err != nil {
					return err
				}
			}
			if !doPoll {
				return nil
			}
			select {
			case <-time.After(interval):
			case <-req.Context.Done():
				return req.Context.Err()
			}
		}
	},
	Type: metrics.Stats{},
	PostRun: cmds.PostRunMap{
		cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
			polling, _ := res.Request().Options[statPollOptionName].(bool)

			if polling {
				fmt.Fprintln(os.Stdout, "Total Up    Total Down  Rate Up     Rate Down")
			}
			for {
				v, err := res.Next()
				if err != nil {
					if err == io.EOF {
						return nil
					}
					return err
				}

				bs := v.(*metrics.Stats)

				if !polling {
					printStats(os.Stdout, bs)
					return nil
				}

				fmt.Fprintf(os.Stdout, "%8s    ", humanize.Bytes(uint64(bs.TotalOut)))
				fmt.Fprintf(os.Stdout, "%8s    ", humanize.Bytes(uint64(bs.TotalIn)))
				fmt.Fprintf(os.Stdout, "%8s/s  ", humanize.Bytes(uint64(bs.RateOut)))
				fmt.Fprintf(os.Stdout, "%8s/s      \r", humanize.Bytes(uint64(bs.RateIn)))
			}
		},
	},
}

```

这段代码定义了一个名为 `printStats` 的函数，接受两个参数：`out` 表示输出写入的文件对象，`bs` 表示要打印的统计数据。

函数内部先输出 "Bandwidth" 字段，然后输出两个字符串，分别表示 `bs.TotalIn` 和 `bs.TotalOut` 的值，其中 `humanize.Bytes` 函数用于将 `uint64` 类型的参数转换为人类可读的字符。

接着输出 "RateIn" 和 "RateOut" 字段，同样使用 `humanize.Bytes` 函数将 `uint64` 类型的参数转换为人类可读的字符。这里，`/s` 表示输出速率，即每秒钟的比特数。

最后，函数返回了参数 `out`。


```go
func printStats(out io.Writer, bs *metrics.Stats) {
	fmt.Fprintln(out, "Bandwidth")
	fmt.Fprintf(out, "TotalIn: %s\n", humanize.Bytes(uint64(bs.TotalIn)))
	fmt.Fprintf(out, "TotalOut: %s\n", humanize.Bytes(uint64(bs.TotalOut)))
	fmt.Fprintf(out, "RateIn: %s/s\n", humanize.Bytes(uint64(bs.RateIn)))
	fmt.Fprintf(out, "RateOut: %s/s\n", humanize.Bytes(uint64(bs.RateOut)))
}

```

# `core/commands/stat_dht.go`

该代码的作用是定义了一个名为 "commands" 的包，其中定义了一些命令，用于在基于 IPFS 的 Kubernetes Cluster 中执行操作。

具体来说，该代码实现了以下功能：

1. 导入了一些必要的库，包括 "fmt"、"io"、"text/tabwriter"、"time" 和 "github.com/ipfs/kubo/core/commands/cmdenv"、"github.com/ipfs/go-ipfs-cmds"、"github.com/libp2p/go-libp2p-kad-dht" 和 "github.com/libp2p/go-libp2p-kad-dht/fullrt"、"github.com/kbucket/kbucket" 和 "github.com/libp2p/go-libp2p"，以及 "github.com/ipfs/go-ipfs-io" 和 "github.com/ipfs/go-ipfs-sub" 库。

2. 实现了一个名为 "cmdenv" 的命令，用于创建和配置基于 IPFS 的 Kubernetes Cluster。该命令提供了创建一个自定义的 Kubernetes Cluster 选项卡，用户可以通过执行该命令的指定选项来设置。

3. 实现了一个名为 "cmds" 的包，用于管理基于 IPFS 的 Kubernetes Cluster 中的命令。该包提供了一个 "形式化接口" 形式的接口，允许用户通过编写命令行工具来定义如何执行常见的 Kubernetes 操作。

4. 导入了一个名为 "dht" 的库，该库实现了基于 IPFS 的 Kubernetes Cluster 中的点对点分布式哈希网络(DHT)服务。

5. 导入了一个名为 "kbucket" 的库，该库实现了基于 IPFS 的 Kubernetes Cluster 中的本地存储服务。

6. 导入了一个名为 "pstore" 的库，该库实现了基于 IPFS 的 Kubernetes Cluster 中的分布式存储系统。

7. 导入了一个名为 "kernel" 的库，该库实现了基于 IPFS 的 Kubernetes Cluster 中的一个通用控制器(controller)。

8. 实现了一个名为 "kubo" 的命令，用于管理基于 IPFS 的 Kubernetes Cluster。该命令提供了一个 "形式化接口" 形式的接口，允许用户通过编写命令行工具来定义如何执行常见的 Kubernetes 操作。

9. 实现了一个名为 "createio" 的命令，用于在基于 IPFS 的 Kubernetes Cluster 中创建自定义 IO 绑定。

10. 实现了一个名为 "bindnet" 的命令，用于在基于 IPFS 的 Kubernetes Cluster 中创建自定义网络。


```go
package commands

import (
	"fmt"
	"io"
	"text/tabwriter"
	"time"

	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"

	cmds "github.com/ipfs/go-ipfs-cmds"
	dht "github.com/libp2p/go-libp2p-kad-dht"
	"github.com/libp2p/go-libp2p-kad-dht/fullrt"
	kbucket "github.com/libp2p/go-libp2p-kbucket"
	"github.com/libp2p/go-libp2p/core/network"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
)

```

这是一个描述DHT（分布式哈希表）数据结构的一些类型定义。

`dhtPeerInfo` 类型表示 DHT 对等机信息，包括连接的客户端版本，最近使用的时间戳和最后的查询时间戳。

`dhtStat` 类型表示 DHT 统计信息，包括名称和哈希表。

`dhtBucket` 类型表示哈希表的一个数据分片，包括最后一个刷新时间和分片内的对等机列表。

这个代码定义了 DHT 中的三个关键类型：对等机信息、统计信息和数据分片。这些类型以及它们所代表的含义可以帮助理解 DHT 的底层架构和数据结构。


```go
type dhtPeerInfo struct {
	ID            string
	Connected     bool
	AgentVersion  string
	LastUsefulAt  string
	LastQueriedAt string
}

type dhtStat struct {
	Name    string
	Buckets []dhtBucket
}

type dhtBucket struct {
	LastRefresh string
	Peers       []dhtPeerInfo
}

```

This appears to be a DHT (Distributed Hash Table) status log for a Bucket. The log entries are written for each bucket and include the bucket ID, the number of peers, the timestamp of the last refresh, and a list of the peers that have been refreshed, along with information about the state of each peer (e.g. whether it is connected and what the Agent Version is). The log also includes a timestamp for when the refresh first occurred and a timestamp for when the last refresh occurred for each peer.


```go
var statDhtCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Returns statistics about the node's DHT(s).",
		ShortDescription: `
Returns statistics about the DHT(s) the node is participating in.

This interface is not stable and may change from release to release.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("dht", false, true, "The DHT whose table should be listed (wanserver, lanserver, wan, lan). "+
			"wan and lan refer to client routing tables. When using the experimental DHT client only WAN is supported. Defaults to wan and lan."),
	},
	Options: []cmds.Option{},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		nd, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !nd.IsOnline {
			return ErrNotOnline
		}

		if nd.DHT == nil {
			return ErrNotDHT
		}

		id := kbucket.ConvertPeerID(nd.Identity)

		dhts := req.Arguments
		if len(dhts) == 0 {
			dhts = []string{"wan", "lan"}
		}

	dhttypeloop:
		for _, name := range dhts {
			var dht *dht.IpfsDHT

			var separateClient bool
			if nd.DHTClient != nd.DHT {
				separateClient = true
			}

			switch name {
			case "wan":
				if separateClient {
					client, ok := nd.DHTClient.(*fullrt.FullRT)
					if !ok {
						return cmds.Errorf(cmds.ErrClient, "could not generate stats for the WAN DHT client type")
					}
					peerMap := client.Stat()
					buckets := make([]dhtBucket, 1)
					b := &dhtBucket{}
					for _, p := range peerMap {
						info := dhtPeerInfo{ID: p.String()}

						if ver, err := nd.Peerstore.Get(p, "AgentVersion"); err == nil {
							info.AgentVersion, _ = ver.(string)
						} else if err == pstore.ErrNotFound {
							// ignore
						} else {
							// this is a bug, usually.
							log.Errorw(
								"failed to get agent version from peerstore",
								"error", err,
							)
						}

						info.Connected = nd.PeerHost.Network().Connectedness(p) == network.Connected
						b.Peers = append(b.Peers, info)
					}
					buckets[0] = *b

					if err := res.Emit(dhtStat{
						Name:    name,
						Buckets: buckets,
					}); err != nil {
						return err
					}
					continue dhttypeloop
				}
				fallthrough
			case "wanserver":
				dht = nd.DHT.WAN
			case "lan":
				if separateClient {
					return cmds.Errorf(cmds.ErrClient, "no LAN client found")
				}
				fallthrough
			case "lanserver":
				dht = nd.DHT.LAN
			default:
				return cmds.Errorf(cmds.ErrClient, "unknown dht type: %s", name)
			}

			rt := dht.RoutingTable()
			lastRefresh := rt.GetTrackedCplsForRefresh()
			infos := rt.GetPeerInfos()
			buckets := make([]dhtBucket, 0, len(lastRefresh))
			for _, pi := range infos {
				cpl := kbucket.CommonPrefixLen(id, kbucket.ConvertPeerID(pi.Id))
				if len(buckets) <= cpl {
					buckets = append(buckets, make([]dhtBucket, 1+cpl-len(buckets))...)
				}

				info := dhtPeerInfo{ID: pi.Id.String()}

				if ver, err := nd.Peerstore.Get(pi.Id, "AgentVersion"); err == nil {
					info.AgentVersion, _ = ver.(string)
				} else if err == pstore.ErrNotFound {
					// ignore
				} else {
					// this is a bug, usually.
					log.Errorw(
						"failed to get agent version from peerstore",
						"error", err,
					)
				}
				if !pi.LastUsefulAt.IsZero() {
					info.LastUsefulAt = pi.LastUsefulAt.Format(time.RFC3339)
				}

				if !pi.LastSuccessfulOutboundQueryAt.IsZero() {
					info.LastQueriedAt = pi.LastSuccessfulOutboundQueryAt.Format(time.RFC3339)
				}

				info.Connected = nd.PeerHost.Network().Connectedness(pi.Id) == network.Connected

				buckets[cpl].Peers = append(buckets[cpl].Peers, info)
			}
			for i := 0; i < len(buckets) && i < len(lastRefresh); i++ {
				refreshTime := lastRefresh[i]
				if !refreshTime.IsZero() {
					buckets[i].LastRefresh = refreshTime.Format(time.RFC3339)
				}
			}
			if err := res.Emit(dhtStat{
				Name:    name,
				Buckets: buckets,
			}); err != nil {
				return err
			}
		}

		return nil
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out dhtStat) error {
			tw := tabwriter.NewWriter(w, 4, 4, 2, ' ', 0)
			defer tw.Flush()

			// Formats a time into XX ago and remove any decimal
			// parts. That is, change "2m3.00010101s" to "2m3s ago".
			now := time.Now()
			since := func(t time.Time) string {
				return now.Sub(t).Round(time.Second).String() + " ago"
			}

			count := 0
			for _, bucket := range out.Buckets {
				count += len(bucket.Peers)
			}

			fmt.Fprintf(tw, "DHT %s (%d peers):\t\t\t\n", out.Name, count)

			for i, bucket := range out.Buckets {
				lastRefresh := "never"
				if bucket.LastRefresh != "" {
					t, err := time.Parse(time.RFC3339, bucket.LastRefresh)
					if err != nil {
						return err
					}
					lastRefresh = since(t)
				}
				fmt.Fprintf(tw, "  Bucket %2d (%d peers) - refreshed %s:\t\t\t\n", i, len(bucket.Peers), lastRefresh)
				fmt.Fprintln(tw, "    Peer\tlast useful\tlast queried\tAgent Version")

				for _, p := range bucket.Peers {
					lastUseful := "never"
					if p.LastUsefulAt != "" {
						t, err := time.Parse(time.RFC3339, p.LastUsefulAt)
						if err != nil {
							return err
						}
						lastUseful = since(t)
					}

					lastQueried := "never"
					if p.LastUsefulAt != "" {
						t, err := time.Parse(time.RFC3339, p.LastQueriedAt)
						if err != nil {
							return err
						}
						lastQueried = since(t)
					}

					state := " "
					if p.Connected {
						state = "@"
					}
					fmt.Fprintf(tw, "  %s %s\t%s\t%s\t%s\n", state, p.ID, lastUseful, lastQueried, p.AgentVersion)
				}
				fmt.Fprintln(tw, "\t\t\t")
			}
			return nil
		}),
	},
	Type: dhtStat{},
}

```

# `core/commands/stat_provide.go`

该代码的作用是定义了一个名为"cmds"的包。它使用了多个外部库：

- "fmt"库：用于格式化字符串
- "io"库：用于输入/输出文件
- "text/tabwriter"库：用于以制表符分隔并写入多个字符串
- "time"库：用于获取当前时间
- "github.com/dustin/go-humanize"库：用于将复杂数字格式化为人可读的格式
- "github.com/ipfs/boxo/provider"库：用于提供Boxo提供商
- "github.com/ipfs/go-ipfs-cmds"库：用于提供IPFS命令行工具
- "github.com/ipfs/kubo/core/commands/cmdenv"库：用于提供Kubernetes命令行工具
- "github.com/ipfs/kubo/userguide/generate"库：用于生成Kubernetes命令行工具的使用说明

该代码还定义了一些常量：

- "fmt.printf"，"%s%c[%s%c]%s%c[%s%c]%s%c%c%s"，表示输出字符串中的百分号、加号、减号、括号和空格。
- "constraints"，约束条件的常量

通过使用这些库和定义，该代码实现了一个功能齐全的命令行工具。


```go
package commands

import (
	"fmt"
	"io"
	"text/tabwriter"
	"time"

	humanize "github.com/dustin/go-humanize"
	"github.com/ipfs/boxo/provider"
	cmds "github.com/ipfs/go-ipfs-cmds"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"golang.org/x/exp/constraints"
)

```

This is a Go program that provides statistics about the content node is advertising. This interface may change from release to release. The program takes no arguments and has no options.

The program first sets up an environment and then retrieves the node's statistics from the provider. If there is an error, it returns an error. If the node is online, the program emits the statistics using `res.Emit(stats)`.

The program uses the ` humanNumber` and ` humanDuration` functions to display the statistics in a human-readable format. The ` humanNumber` function takes a `Provider` struct and returns a human-readable number, such as "10 providers". The ` humanDuration` function takes a `Provider` struct and returns the duration in human time units, such as "1 minute".

The program also defines the data type for the `Provider` struct as `provider.ProviderStats`.


```go
var statProvideCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Returns statistics about the node's (re)provider system.",
		ShortDescription: `
Returns statistics about the content the node is advertising.

This interface is not stable and may change from release to release.
`,
	},
	Arguments: []cmds.Argument{},
	Options:   []cmds.Option{},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		nd, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !nd.IsOnline {
			return ErrNotOnline
		}

		stats, err := nd.Provider.Stat()
		if err != nil {
			return err
		}

		if err := res.Emit(stats); err != nil {
			return err
		}

		return nil
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, s *provider.ReproviderStats) error {
			wtr := tabwriter.NewWriter(w, 1, 2, 1, ' ', 0)
			defer wtr.Flush()

			fmt.Fprintf(wtr, "TotalProvides:\t%s\n", humanNumber(s.TotalProvides))
			fmt.Fprintf(wtr, "AvgProvideDuration:\t%s\n", humanDuration(s.AvgProvideDuration))
			fmt.Fprintf(wtr, "LastReprovideDuration:\t%s\n", humanDuration(s.LastReprovideDuration))
			fmt.Fprintf(wtr, "LastReprovideBatchSize:\t%s\n", humanNumber(s.LastReprovideBatchSize))
			return nil
		}),
	},
	Type: provider.ReproviderStats{},
}

```

这两段代码定义了两个函数，分别是 "func humanDuration(val time.Duration) string" 和 "func humanNumber[T constraints.Float | constraints.Integer](n T) string"。

函数 "func humanDuration(val time.Duration) string" 的作用是返回一个将 "time.Duration" 类型中的值截断到 "time.Microsecond" 精度的字符串。函数的实现中，首先调用 "Truncate" 函数将 "time.Duration" 中的值截断到 "time.Microsecond" 精度，然后调用 "String" 函数将截断后的值转换为字符串类型。

函数 "func humanNumber[T constraints.Float | constraints.Integer](n T) string" 的作用是返回一个将 "T" 类型中的整数截断到 "humanNumber" 函数中的浮点数 "n" 并返回对应的字符串。函数的实现中，首先调用 "humanSI" 函数将 "n" 截断到 "humanNumber" 函数中的浮点数 "n"，然后调用 "fmt.Sprintf" 函数将 "n" 和对应的字符串 "str""组合在一起，最后调用 "humanFull" 函数将 "str" 中的字符串转换为字符串类型。


```go
func humanDuration(val time.Duration) string {
	return val.Truncate(time.Microsecond).String()
}

func humanNumber[T constraints.Float | constraints.Integer](n T) string {
	nf := float64(n)
	str := humanSI(nf, 0)
	fullStr := humanFull(nf, 0)
	if str != fullStr {
		return fmt.Sprintf("%s\t(%s)", str, fullStr)
	}
	return str
}

func humanSI(val float64, decimals int) string {
	v, unit := humanize.ComputeSI(val)
	return fmt.Sprintf("%s%s", humanFull(v, decimals), unit)
}

```

该函数名为 `humanFull`，它接受两个参数 `val` 和 `decimals`，它们都是 `float64` 类型的浮点数。函数返回一个字符串，它是通过调用 `humanize.CommafWithDigits` 函数并传入 `val` 和 `decimals` 参数得到的。

`humanize.CommafWithDigits` 函数的作用是将传入的 `val` 参数的数字格式化为一个字符串，其中数字使用逗号分隔，小数点使用指定的 `decimals` 参数数量级别的精度。具体来说，如果 `decimals` 参数为 0，则保留两位小数，否则保留更多的小数位。

因此，函数 `humanFull` 的作用是将传入的 `float64` 类型的数值格式化为一个以逗号分隔的数字字符串，其中数字使用指定的精度。


```go
func humanFull(val float64, decimals int) string {
	return humanize.CommafWithDigits(val, decimals)
}

```

# `core/commands/swarm.go`

这段代码定义了一个名为 "commands" 的包，其中定义了一些与 Go-IPFS 相关的命令。

具体来说，这段代码：

1. 导入了一些必要的库：base64、json、fmt、strconv、path、sort、strconv、sync、text/tabwriter、time、github.com/ipfs/kubo/commands、github.com/ipfs/kubo/config、github.com/ipfs/kubo/core/commands/cmdenv、github.com/ipfs/kubo/core/node/libp2p、github.com/ipfs/kubo/repo、github.com/ipfs/kubo/repo/fsrepo、github.com/ipfs/go-ipfs-cmds、github.com/libp2p/go-libp2p/core/crypto、github.com/libp2p/go-libp2p/core/network、github.com/libp2p/go-libp2p/core/peer、github.com/libp2p/go-libp2p/p2p/host/resource-manager、github.com/multiformats/go-multiaddr、github.com/multiformats/go-multiaddr-dns、github.com/whyrusleeping/multiaddr-filter。

2. 定义了一些常量：一个用于存储桶标识的变量、用于在控制台上输出消息的变量、用于将字符串解码为 JSON 对象的编码器等等。

3. 定义了一些函数：其中一些函数与 Go-IPFS 相关，其他函数则与 Go-IPFS 无关。这些函数的实现包括：addKubernetesConfig、cliEnter、cmderive、denv、do-auth、do-fetch、do-send、do-todo、fmtarg、getEnv、getStaticValue、getValue、io的write、json、makeRequest、net/http的getContext、net/http的startTLS、net/http的startTrading、net/ipfs的io等。


```go
package commands

import (
	"context"
	"encoding/base64"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"path"
	"sort"
	"strconv"
	"sync"
	"text/tabwriter"
	"time"

	"github.com/ipfs/kubo/commands"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/repo"
	"github.com/ipfs/kubo/repo/fsrepo"

	cmds "github.com/ipfs/go-ipfs-cmds"
	ic "github.com/libp2p/go-libp2p/core/crypto"
	inet "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	ma "github.com/multiformats/go-multiaddr"
	madns "github.com/multiformats/go-multiaddr-dns"
	mamask "github.com/whyrusleeping/multiaddr-filter"
)

```

This code defines several types and a function signature in Go.

The `dnsResolveTimeout` constant is set to 10 seconds.

The `stringList` type is defined as an ordered slice of strings.

The `addrMap` type is defined as a map with key-value pairs where the keys are strings and the values are arrays of strings.

The `SwarmCmd` function signature is defined as a command with a help text of "Interact with the swarm." and a short description.

The code does not include any function or variable usage, so it is unclear what the `SwarmCmd` function or the `addrMap` type is intended to do when called.


```go
const (
	dnsResolveTimeout = 10 * time.Second
)

type stringList struct {
	Strings []string
}

type addrMap struct {
	Addrs map[string][]string
}

var SwarmCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Interact with the swarm.",
		ShortDescription: `
```

该代码是一个名为"ipfs swarm"的工具，用于操作网络 Swarm。Swarm 是组件，打开、监听并维护与互联网上其他 ipfs 节点的连接。

该代码定义了一个名为"Subcommands"的 map，它包含了一些 ipfs 命令，用于添加、连接、断开、筛选和获取与节点共享的资源。这些命令通过 "swarm" 名称进行调用，swarm 是一个代表整个工具操作的术语。

具体来说，这些命令包括：

- "addrs": 这个命令用于添加 ipfs 节点地址。
- "connect": 这个命令用于连接到其他 ipfs 节点。
- "disconnect": 这个命令用于断开与当前节点的连接。
- "filters": 这个命令可以用于添加、删除、设置 ipfs 过滤器。
- "peers": 这个命令可以用于获取与当前节点连接的其它节点。
- "peering": 这个命令可以用于设置或获取与当前节点之间的对等连接。
- "resources": 这个命令可以用于设置或获取与当前节点共享的资源。libp2p 网络资源管理器（swarmResourcesCmd）可能也需要使用这个命令进行设置。


```go
'ipfs swarm' is a tool to manipulate the network swarm. The swarm is the
component that opens, listens for, and maintains connections to other
ipfs peers in the internet.
`,
	},
	Subcommands: map[string]*cmds.Command{
		"addrs":      swarmAddrsCmd,
		"connect":    swarmConnectCmd,
		"disconnect": swarmDisconnectCmd,
		"filters":    swarmFiltersCmd,
		"peers":      swarmPeersCmd,
		"peering":    swarmPeeringCmd,
		"resources":  swarmResourcesCmd, // libp2p Network Resource Manager

	},
}

```

此代码定义了一个名为`swarmStreamsOptionName`的选项组，其中包括了`swarmStreamsOptionName`、`swarmLatencyOptionName`、`swarmDirectionOptionName`和`swarmResetLimitsOptionName`。这些选项用于配置一个名为`swarmStreams`的选项。

此选项组中的每个选项都使用了`const`定义的常量，可以在代码中根据需要进行赋值。例如，可以在某个地方定义一个`swarmStreams`变量，并使用上面定义的选项进行初始化。

`swarmStreamsOptionName`是一个字符串类型的选项，根据定义可以知道它的值为`"streams"`。这个选项用于配置一个swarm网络中`swarmStreams`选项的值，让该网络根据设置的值工作。

`swarmStreamsOptionName`是上述定义的选项中最后一个，用于配置swarm网络的延迟选项。根据设置的值，网络将使用该延迟选项来工作，以保证网络的性能。延迟选项的值可以是`"低"`、`"medium"`或`"high"`。


```go
const (
	swarmVerboseOptionName           = "verbose"
	swarmStreamsOptionName           = "streams"
	swarmLatencyOptionName           = "latency"
	swarmDirectionOptionName         = "direction"
	swarmResetLimitsOptionName       = "reset"
	swarmUsedResourcesPercentageName = "min-used-limit-perc"
	swarmIdentifyOptionName          = "identify"
)

type peeringResult struct {
	ID     peer.ID
	Status string
}

```

该代码定义了一个名为`swarmPeeringCmd`的命令对象，用于管理一个名为`swarmPeering`的子系统。

该命令的`Helptext`字段包含命令的帮助信息，其中包含一个`Tagline`和一个`ShortDescription`字段。`Tagline`字段提供了一个简洁的命令描述，`ShortDescription`字段则提供了命令的简短说明。

该命令的`Subcommands`字段定义了与该命令相关的命令对象。在这个列表中，定义了4个命令对象，分别是`swarmPeeringAddCmd`、`swarmPeeringLsCmd`、`swarmPeeringRmCmd`和`swarmPeeringConfigCmd`。

`swarmPeeringAddCmd`用于添加一个新的`swarmPeering`节点。

`swarmPeeringLsCmd`用于列出所有已知的`swarmPeering`节点及其相关信息。

`swarmPeeringRmCmd`用于删除指定的`swarmPeering`节点。

`swarmPeeringConfigCmd`用于设置或取消指定的`swarmPeering`节点的配置，例如设置指定节点的端口、设置标签等。


```go
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
		"add": swarmPeeringAddCmd,
		"ls":  swarmPeeringLsCmd,
		"rm":  swarmPeeringRmCmd,
	},
}

```

This is a command line interface (CLI) command for adding a new IPFS node address to the peering subsystem. The `ipfs swarm peering add` command takes an argument specifying the address of the new node to be added, and returns a response indicating whether the operation was successful or not. If the operation was successful, the response will include the ID of the newly added node and a message indicating that the operation was successful. If the operation was not successful, the response will include an error message.


```go
var swarmPeeringAddCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Add peers into the peering subsystem.",
		ShortDescription: `
'ipfs swarm peering add' will add the new address to the peering subsystem as one that should always be connected to.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("address", true, true, "address of peer to add into the peering subsystem"),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		addrs := make([]ma.Multiaddr, len(req.Arguments))

		for i, arg := range req.Arguments {
			addr, err := ma.NewMultiaddr(arg)
			if err != nil {
				return err
			}

			addrs[i] = addr
		}

		addInfos, err := peer.AddrInfosFromP2pAddrs(addrs...)
		if err != nil {
			return err
		}

		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}
		if !node.IsOnline {
			return ErrNotOnline
		}

		for _, addrinfo := range addInfos {
			node.Peering.AddPeer(addrinfo)
			err = res.Emit(peeringResult{addrinfo.ID, "success"})
			if err != nil {
				return err
			}
		}
		return nil
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, pr *peeringResult) error {
			fmt.Fprintf(w, "add %s %s\n", pr.ID.String(), pr.Status)
			return nil
		}),
	},
	Type: peeringResult{},
}

```

该代码是一个 Go 语言中的函数，用于列出注册在交换子系统中的对端。函数接受一个 `cmds.Request`、一个 `cmds.ResponseEmitter` 和一个 `cmds.Environment`。

函数的作用如下：

1. 首先获取一个 `node` 实例，如果 `node` 存在且在线，则执行以下操作：
   a. 获取注册在对端的服务器端 ID。
   b. 通过调用 `node.Peering.ListPeers()` 获取注册在对端的客户端 ID。
   c. 返回客户端 ID 并添加到结果映射中，使用 `fmt.EmitOnce` 函数确保只返回一次。

2. 如果 `node` 存在，但不是在线状态，则返回错误并使用 `fmt.EmitOnce` 函数确保只返回一次。

3. 返回结果映射中的客户端 ID，使用 `fmt.EmitOnce` 函数确保只返回一次。

4. 返回 `cmds.Success` 作为结果，如果没有错误则返回 `cmds.NoOp` 作为结果。


```go
var swarmPeeringLsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List peers registered in the peering subsystem.",
		ShortDescription: `
'ipfs swarm peering ls' lists the peers that are registered in the peering subsystem and to which the daemon is always connected.
`,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}
		if !node.IsOnline {
			return ErrNotOnline
		}

		peers := node.Peering.ListPeers()
		return cmds.EmitOnce(res, addrInfos{Peers: peers})
	},
	Type: addrInfos{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ai *addrInfos) error {
			for _, info := range ai.Peers {
				fmt.Fprintf(w, "%s\n", info.ID)
				for _, addr := range info.Addrs {
					fmt.Fprintf(w, "\t%s\n", addr)
				}
			}
			return nil
		}),
	},
}

```

此代码定义了一个名为`addrInfos`的结构体，其中包含一个名为`Peers`的数组，该数组包含一个`peer.AddrInfo`类型的变量。

接下来的代码定义了一个名为`swarmPeeringRmCmd`的命令，该命令具有如下帮助文本：


Remove a peer from the peering subsystem.


该命令需要传递一个`ID`参数，作为要删除的ID。该命令会在删除ID后从`peering subsystem`中将其从`always-on connection`中移除。

该命令的实现如下：


func (c *cmds.Command) Run(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
	node, err := cmdenv.GetNode(env)
	if err != nil {
		return err
	}
	if !node.IsOnline {
		return errNotOnline
	}

	for _, arg := range req.Arguments {
		id, err := peer.Decode(arg)
		if err != nil {
			return err
		}

		node.Peering.RemovePeer(id)
		if err = res.Emit(peeringResult{id, "success"}); err != nil {
			return err
		}
	}

	return nil
}


该命令实现了两个函数：

* `PeerDrop`函数：从`addrInfos`数组中删除给定ID的`peer.AddrInfo`，并从`always-on connection`中将其从`swarm`中移除。
* `Run`函数：实现命令行命令的实现。接收请求参数，创建或删除指定ID的`peer.AddrInfo`，然后将其从`swarm`中删除，并从`always-on connection`中删除指定的ID。最后，将结果返回给调用者。


```go
type addrInfos struct {
	Peers []peer.AddrInfo
}

var swarmPeeringRmCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Remove a peer from the peering subsystem.",
		ShortDescription: `
'ipfs swarm peering rm' will remove the given ID from the peering subsystem and remove it from the always-on connection.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("ID", true, true, "ID of peer to remove from the peering subsystem"),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}
		if !node.IsOnline {
			return ErrNotOnline
		}

		for _, arg := range req.Arguments {
			id, err := peer.Decode(arg)
			if err != nil {
				return err
			}

			node.Peering.RemovePeer(id)
			if err = res.Emit(peeringResult{id, "success"}); err != nil {
				return err
			}
		}
		return nil
	},
	Type: peeringResult{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, pr *peeringResult) error {
			fmt.Fprintf(w, "remove %s %s\n", pr.ID.String(), pr.Status)
			return nil
		}),
	},
}

```

This is a Go code that implements a command-line interface (CLI) for connecting to a geth cluster and connecting to a identify peer. The `ci.IdentifyPeer` function is used to establish a connection with the identify peer and the `ci.Identify` field is set to the result of the `ci.identifyPeer` function.

The `cli` package was used to create the CLI based on the `-h` flag, and the `-v` flag for verbose output. `identifyPeer` function is used to identify the identity of the peer, and if `identify` flag is set, it will use this function.

The output of the ` identifyPeer` function is compared to the input to determine the next action to take. If the `identify` flag is set, it will use the `ci.identifyPeer` function to establish a connection with the identify peer and the `ci.Identify` field will be set to the result of the `ci.identifyPeer` function. If the `verbose` flag is set, the output will be verbose.

The peers are connected to the cluster and their information is sent to the output. The peers are sorted by the `ci` struct based on the peers and the output is sent to the input.


```go
var swarmPeersCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List peers with open connections.",
		ShortDescription: `
'ipfs swarm peers' lists the set of peers this node is connected to.
`,
	},
	Options: []cmds.Option{
		cmds.BoolOption(swarmVerboseOptionName, "v", "display all extra information"),
		cmds.BoolOption(swarmStreamsOptionName, "Also list information about open streams for each peer"),
		cmds.BoolOption(swarmLatencyOptionName, "Also list information about latency to each peer"),
		cmds.BoolOption(swarmDirectionOptionName, "Also list information about the direction of connection"),
		cmds.BoolOption(swarmIdentifyOptionName, "Also list information about peers identify"),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}
		verbose, _ := req.Options[swarmVerboseOptionName].(bool)
		latency, _ := req.Options[swarmLatencyOptionName].(bool)
		streams, _ := req.Options[swarmStreamsOptionName].(bool)
		direction, _ := req.Options[swarmDirectionOptionName].(bool)
		identify, _ := req.Options[swarmIdentifyOptionName].(bool)

		conns, err := api.Swarm().Peers(req.Context)
		if err != nil {
			return err
		}

		var out connInfos
		for _, c := range conns {
			ci := connInfo{
				Addr: c.Address().String(),
				Peer: c.ID().String(),
			}

			if verbose || direction {
				// set direction
				ci.Direction = c.Direction()
			}

			if verbose || latency {
				lat, err := c.Latency()
				if err != nil {
					return err
				}

				if lat == 0 {
					ci.Latency = "n/a"
				} else {
					ci.Latency = lat.String()
				}
			}
			if verbose || streams {
				strs, err := c.Streams()
				if err != nil {
					return err
				}

				for _, s := range strs {
					ci.Streams = append(ci.Streams, streamInfo{Protocol: string(s)})
				}
			}

			if verbose || identify {
				n, err := cmdenv.GetNode(env)
				if err != nil {
					return err
				}
				identifyResult, _ := ci.identifyPeer(n.Peerstore, c.ID())
				ci.Identify = identifyResult
			}
			sort.Sort(&ci)
			out.Peers = append(out.Peers, ci)
		}

		sort.Sort(&out)
		return cmds.EmitOnce(res, &out)
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, ci *connInfos) error {
			pipfs := ma.ProtocolWithCode(ma.P_IPFS).Name
			for _, info := range ci.Peers {
				fmt.Fprintf(w, "%s/%s/%s", info.Addr, pipfs, info.Peer)
				if info.Latency != "" {
					fmt.Fprintf(w, " %s", info.Latency)
				}

				if info.Direction != inet.DirUnknown {
					fmt.Fprintf(w, " %s", directionString(info.Direction))
				}
				fmt.Fprintln(w)

				for _, s := range info.Streams {
					if s.Protocol == "" {
						s.Protocol = "<no protocol name>"
					}

					fmt.Fprintf(w, "  %s\n", s.Protocol)
				}
			}

			return nil
		}),
	},
	Type: connInfos{},
}

```

This appears to be a Go program that formats the usage of a Linux limit system into a tab-separated file. The program takes two arguments: a writer and a limiter configuration object. The writer is used to write the data to the file, and the limiter configuration object is used to specify the format of the data to be written.

The program then loops through each usage limit in the limiter configuration object, formatting the data for each limit and writing it to the writer. The data is formatted as a table with columns for the scope name, limit name, limit value, and usage amount, as well as columns for the percentage of the limit value that represents the usage.

The program uses the `fmt.Fprintf` function to write the data to the file, and the `defer` directive to automatically flush the writer to disk when the program exits. The program also imports the `libp2p.LimitConfigsToInfo` function to help with parsing the limiter configuration object.


```go
var swarmResourcesCmd = &cmds.Command{
	Status: cmds.Experimental,
	Helptext: cmds.HelpText{
		Tagline: "Get a summary of all resources accounted for by the libp2p Resource Manager.",
		LongDescription: `
Get a summary of all resources accounted for by the libp2p Resource Manager.
This includes the limits and the usage against those limits.
This can output a human readable table and JSON encoding.
`,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if node.ResourceManager == nil {
			return libp2p.ErrNoResourceMgr
		}

		cfg, err := node.Repo.Config()
		if err != nil {
			return err
		}

		userResourceOverrides, err := node.Repo.UserResourceOverrides()
		if err != nil {
			return err
		}

		// FIXME: we shouldn't recompute limits, either save them or load them from libp2p (https://github.com/libp2p/go-libp2p/issues/2166)
		limitConfig, _, err := libp2p.LimitConfig(cfg.Swarm, userResourceOverrides)
		if err != nil {
			return err
		}

		rapi, ok := node.ResourceManager.(rcmgr.ResourceManagerState)
		if !ok { // NullResourceManager
			return libp2p.ErrNoResourceMgr
		}

		return cmds.EmitOnce(res, libp2p.MergeLimitsAndStatsIntoLimitsConfigAndUsage(limitConfig, rapi.Stat()))
	},
	Encoders: cmds.EncoderMap{
		cmds.JSON: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, limitsAndUsage libp2p.LimitsConfigAndUsage) error {
			return json.NewEncoder(w).Encode(limitsAndUsage)
		}),
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, limitsAndUsage libp2p.LimitsConfigAndUsage) error {
			tw := tabwriter.NewWriter(w, 20, 8, 0, '\t', 0)
			defer tw.Flush()

			fmt.Fprintf(tw, "%s\t%s\t%s\t%s\t%s\t\n", "Scope", "Limit Name", "Limit Value", "Limit Usage Amount", "Limit Usage Percent")
			for _, ri := range libp2p.LimitConfigsToInfo(limitsAndUsage) {
				var limit, percentage string
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
				fmt.Fprintf(tw, "%s\t%s\t%s\t%d\t%s\t\n",
					ri.ScopeName,
					ri.LimitName,
					limit,
					ri.CurrentUsage,
					percentage,
				)
			}

			return nil
		}),
	},
	Type: libp2p.LimitsConfigAndUsage{},
}

```

这段代码定义了一个名为 `connInfo` 的结构体类型，该类型包含一个名为 `Addr` 的字符串字段，一个名为 `Peer` 的字符串字段，一个名为 `Latency` 的字符串字段和一个名为 `Muxer` 的字符串字段。

此外，该类型还包含一个名为 `Direction` 的字符串字段，该字段表示输入/输出方向，以及一个名为 `Streams` 的数组字段，该字段包含一个名为 `Protocol` 的字符串字段，该字段表示每个流媒体的协议类型。

最后，该类型还包含一个名为 `Identify` 的字段，该字段表示用于标识连接的名称。

另外，该代码实现了一个名为 `Less` 的函数，该函数比较两个 `connInfo` 类型的实例 `ci` 和 `i` 的 `Streams` 字段，并返回一个名为 `ci` 的实例比上 `i` 的实例更年轻。


```go
type streamInfo struct {
	Protocol string
}

type connInfo struct {
	Addr      string         `json:",omitempty"`
	Peer      string         `json:",omitempty"`
	Latency   string         `json:",omitempty"`
	Muxer     string         `json:",omitempty"`
	Direction inet.Direction `json:",omitempty"`
	Streams   []streamInfo   `json:",omitempty"`
	Identify  IdOutput       `json:",omitempty"`
}

func (ci *connInfo) Less(i, j int) bool {
	return ci.Streams[i].Protocol < ci.Streams[j].Protocol
}

```

这段代码定义了一个名为 connInfos 的 struct 类型，它包含一个名为 Peers 的数组，数组长度为 16。

func (ci connInfos) Len() int {
	return len(ci.Peers)
}

func (ci connInfos) Swap(i, j int) {
	ci.Peers[i], ci.Peers[j] = ci.Peers[j], ci.Peers[i]
}

func (ci connInfos) Less(i, j int) bool {
	return ci.Peers[i].Addr < ci.Peers[j].Addr
}

这个函数CI 是接收一个 connInfos 类型的参数，并返回一个 int 类型的变量，该变量表示连接信息的长度。

这个函数CI Len() 是返回ci.Peers 的长度，该函数ci.Swap() 是交换 ci.Peers 和 ci.Peers。

这个函数CI Less() 是用来比较ci.Peers 的 Addr 是否比ci.Peers 的 Addr 小。


```go
func (ci *connInfo) Len() int {
	return len(ci.Streams)
}

func (ci *connInfo) Swap(i, j int) {
	ci.Streams[i], ci.Streams[j] = ci.Streams[j], ci.Streams[i]
}

type connInfos struct {
	Peers []connInfo
}

func (ci connInfos) Less(i, j int) bool {
	return ci.Peers[i].Addr < ci.Peers[j].Addr
}

```

该代码定义了两个函数，一个是`Len()`函数，返回`ci.Peers`的长度；另一个是`Swap()`函数，接收两个整数参数`i`和`j`，交换`ci.Peers`[`i`]和`ci.Peers`[`j`]的值。

函数`identifyPeer()`接收一个`pstore.Peerstore`类型的`ps`和一个`peer.ID`类型的`p`，返回一个`IdOutput`类型的数据，该数据包含`p`的ID作为`info.ID`，以及一个包含`p`的公共关键字的字节数组作为`info.PublicKey`，或者在失败时返回一个错误。

函数`identifyPeer()`中包含以下步骤：

1. 获取`ps`中与`p`交易的PubKey。
2. 将获取的PubKey编码为Base64字符串，并将其存储为`info.PublicKey`。
3. 获取`ps`中与`p`交易的PeerInfo，并将其存储为`info.Addresses`。
4. 对`info.Addresses`进行排序。
5. 获取`ps`中与`p`支持的所有协议，并将其存储为`info.Protocols`。
6. 对`info.Protocols`进行排序。
7. 获取`ps`中与`p`交易的AgentVersion，并将其存储为`info.AgentVersion`。

函数`Len()`没有返回值，因为它只是返回了`ci.Peers`的长度。


```go
func (ci connInfos) Len() int {
	return len(ci.Peers)
}

func (ci connInfos) Swap(i, j int) {
	ci.Peers[i], ci.Peers[j] = ci.Peers[j], ci.Peers[i]
}

func (ci *connInfo) identifyPeer(ps pstore.Peerstore, p peer.ID) (IdOutput, error) {
	var info IdOutput
	info.ID = p.String()

	if pk := ps.PubKey(p); pk != nil {
		pkb, err := ic.MarshalPublicKey(pk)
		if err != nil {
			return IdOutput{}, err
		}
		info.PublicKey = base64.StdEncoding.EncodeToString(pkb)
	}

	addrInfo := ps.PeerInfo(p)
	addrs, err := peer.AddrInfoToP2pAddrs(&addrInfo)
	if err != nil {
		return IdOutput{}, err
	}

	for _, a := range addrs {
		info.Addresses = append(info.Addresses, a.String())
	}
	sort.Strings(info.Addresses)

	if protocols, err := ps.GetProtocols(p); err == nil {
		info.Protocols = append(info.Protocols, protocols...)
		sort.Slice(info.Protocols, func(i, j int) bool { return info.Protocols[i] < info.Protocols[j] })
	}

	if v, err := ps.Get(p, "AgentVersion"); err == nil {
		if vs, ok := v.(string); ok {
			info.AgentVersion = vs
		}
	}

	return info, nil
}

```

This code defines a function called `directionString()` that takes an `inet.Direction` value as an input and returns a corresponding string representation of that direction.

The function has a single argument of type `inet.Direction`, which represents the direction. The function then uses a `switch` statement to check the input direction and return the corresponding string representation if the input direction is recognized, otherwise it returns an empty string.

The `swarmAddrsCmd` variable is of type `cmds.Command` and has a `Helptext` field with a description of the command's purpose, and a `ShortDescription` field that is a brief description of the command. It does not seem to have any direct functionality, but it is used in the `ListKnownAddresses()` function call.


```go
// directionString transfers to string
func directionString(d inet.Direction) string {
	switch d {
	case inet.DirInbound:
		return "inbound"
	case inet.DirOutbound:
		return "outbound"
	default:
		return ""
	}
}

var swarmAddrsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List known addresses. Useful for debugging.",
		ShortDescription: `
```

This is a Go template for a command line interface (CLI) that adds a "local" and "listen" command to a Swarm. The Swarm is specified by the environment in which the CLI is running.

The "local" command is used to add a known data service address to the Swarm's local address段 while the "listen" command is used to advertise the Swarm's local address range to remote servers.

The CLI output is a map that maps Swarm node IDs to a list of available data service endpoints, with the endpoints being separated by a delimiter.


```go
'ipfs swarm addrs' lists all addresses this node is aware of.
`,
	},
	Subcommands: map[string]*cmds.Command{
		"local":  swarmAddrsLocalCmd,
		"listen": swarmAddrsListenCmd,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		addrs, err := api.Swarm().KnownAddrs(req.Context)
		if err != nil {
			return err
		}

		out := make(map[string][]string)
		for p, paddrs := range addrs {
			s := p.String()
			for _, a := range paddrs {
				out[s] = append(out[s], a.String())
			}
		}

		return cmds.EmitOnce(res, &addrMap{Addrs: out})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, am *addrMap) error {
			// sort the ids first
			ids := make([]string, 0, len(am.Addrs))
			for p := range am.Addrs {
				ids = append(ids, p)
			}
			sort.Strings(ids)

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
	Type: addrMap{},
}

```

该代码定义了一个名为`swarmAddrsLocalCmd`的命令对象。

该命令的帮助下，用户可以列出本地主机的IPFS服务器。该命令通过调用`api.Swarm().LocalAddrs`方法获取到主机的`localAddrs`，然后遍历该结果，将每个`localAddr`的`String()`值存储在`addrs`数组中。最后，对`addrs`数组进行排序并输出。

该命令的`options`选项中包含一个名为`id`的`bool`选项。如果选中此选项，它将输出经过P2P协议的主机ID，而不是IPFS本地地址。


```go
var swarmAddrsLocalCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List local addresses.",
		ShortDescription: `
'ipfs swarm addrs local' lists all local listening addresses announced to the network.
`,
	},
	Options: []cmds.Option{
		cmds.BoolOption("id", "Show peer ID in addresses."),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		showid, _ := req.Options["id"].(bool)
		self, err := api.Key().Self(req.Context)
		if err != nil {
			return err
		}

		maddrs, err := api.Swarm().LocalAddrs(req.Context)
		if err != nil {
			return err
		}

		var addrs []string
		p2pProtocolName := ma.ProtocolWithCode(ma.P_P2P).Name
		for _, addr := range maddrs {
			saddr := addr.String()
			if showid {
				saddr = path.Join(saddr, p2pProtocolName, self.ID().String())
			}
			addrs = append(addrs, saddr)
		}
		sort.Strings(addrs)
		return cmds.EmitOnce(res, &stringList{addrs})
	},
	Type: stringList{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
}

```

这段代码定义了一个名为`swarmAddrsListenCmd`的命令，它使用了`cmds.Command`类型。这个命令的作用是获取节点当前监听的接口地址列表，并将它们返回给用户。

具体来说，这个命令首先获取了当前环境中的`cmdenv`包的`api`字段，如果获取失败则返回。然后，使用`api.Swarm().ListenAddrs(req.Context)`方法获取了节点当前监听的接口地址列表，如果这个方法返回错误，则也会返回。

获取到接口地址列表后，命令将这些地址进行排序，然后将它们打包成一个字符串列表，最后使用`cmds.Text`类型的编码器将这个列表格式的数据输出给用户。

这个命令的输入参数是一个`stringList`类型的参数，其中包含节点当前监听的所有接口地址。


```go
var swarmAddrsListenCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List interface listening addresses.",
		ShortDescription: `
'ipfs swarm addrs listen' lists all interface addresses the node is listening on.
`,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		var addrs []string
		maddrs, err := api.Swarm().ListenAddrs(req.Context)
		if err != nil {
			return err
		}

		for _, addr := range maddrs {
			addrs = append(addrs, addr.String())
		}
		sort.Strings(addrs)

		return cmds.EmitOnce(res, &stringList{addrs})
	},
	Type: stringList{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
}

```

This is a description of a p2p (peer-to-peer) address format for libp2p, which is a networking technology for peer-to-peer applications.

The address format is represented by the multiaddr:

ipfs swarm connect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ

This format includes the IPv4 address of the peer to connect to (IPv4 address 104.131.131.82), a TCP port number (port 4001), and the path of the connection ( "/ip4/104.131.131.82/tcp/4001").

The address can be aware of other addresses for a given peer or may already have an established connection to the peer.

The address format is a libp2p multiaddr:

ipfs swarm connect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ

This format includes the IPv4 address of the peer to connect to (IPv4 address 104.131.131.82), a TCP port number (port 4001), and the path of the connection ( "/ip4/104.131.131.82/tcp/4001").

The address can be aware of other addresses for a given peer or may already have an established connection to the peer.

The address format is a libp2p multiaddr:

ipfs swarm connect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8Scb


```go
var swarmConnectCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Open connection to a given peer.",
		ShortDescription: `
'ipfs swarm connect' attempts to ensure a connection to a given peer.

Multiaddresses given are advisory, for example the node may already be aware of other addresses for a given peer or may already have an established connection to the peer.

The address format is a libp2p multiaddr:

ipfs swarm connect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("address", true, true, "Address of peer to connect to.").EnableStdin(),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		addrs := req.Arguments

		pis, err := parseAddresses(req.Context, addrs, node.DNSResolver)
		if err != nil {
			return err
		}

		output := make([]string, len(pis))
		for i, pi := range pis {
			output[i] = "connect " + pi.ID.String()

			err := api.Swarm().Connect(req.Context, pi)
			if err != nil {
				return fmt.Errorf("%s failure: %s", output[i], err)
			}
			output[i] += " success"
		}

		return cmds.EmitOnce(res, &stringList{output})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
	Type: stringList{},
}

```

This is a Go function that defines the "response-emitter" for a "node" service. It takes an "environment" and a "response-code-name" as input arguments, and returns a "response-emitter" that can be used to emit a response using the "response-code-name" as a prefix.

The function first checks that the "node" service has been initialized correctly, and then retrieves the "api" service from the "environment". It then retrieves the "node-dns-resolver" service from the "environment", and passes an "AddrInfo" struct and a "Node" struct to the "peer.AddrInfoToP2pAddrs" function.

It then iterates through the "AddrInfo" struct and the "Node" struct, and for each "AddrInfo" struct, it retrieves a "PeerAddrInfo" struct and uses the "Swarm" method to disconnect the peer, returning the error and the message in the response if the disconnect was successful, or the message in case of failure.

Finally, it returns a "response-emitter" that emits a list of strings based on the prefix of the "response-code-name", and returns the list after the initial emission.

Note that the behavior of this function is not well tested and may be changed in a future release.


```go
var swarmDisconnectCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Close connection to a given address.",
		ShortDescription: `
'ipfs swarm disconnect' closes a connection to a peer address. The address
format is an IPFS multiaddr:

ipfs swarm disconnect /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ

The disconnect is not permanent; if ipfs needs to talk to that address later,
it will reconnect.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("address", true, true, "Address of peer to disconnect from.").EnableStdin(),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		node, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		addrs, err := parseAddresses(req.Context, req.Arguments, node.DNSResolver)
		if err != nil {
			return err
		}

		output := make([]string, 0, len(addrs))
		for _, ainfo := range addrs {
			maddrs, err := peer.AddrInfoToP2pAddrs(&ainfo)
			if err != nil {
				return err
			}
			// FIXME: This will print:
			//
			//   disconnect QmFoo success
			//   disconnect QmFoo success
			//   ...
			//
			// Once per address specified. However, I'm not sure of
			// a good backwards compat solution. Right now, I'm just
			// preserving the current behavior.
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
		return cmds.EmitOnce(res, &stringList{output})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
	Type: stringList{},
}

```

This is a Rust function that resolves a list of IP addresses to their corresponding peerless addresses (i.e., `ipfs/Qm` addresses) on the IPFS network. It takes a list of IP addresses and resolves them one by one, checking if their peerless address can be found in the `ipfs/Qm` space. If it can't, it returns an error. If it can, it adds the address to a channel `maddrC` and returns it.

The function also handles the case where the input IP addresses are not in the `ipfs/Qm` space, which is not an error but instead returns the IP addresses immediately.

The function uses a goroutine that runs the `rslv.Resolve` function to resolve each IP address to its corresponding peerless address, and then filters out any IP addresses that still don't end in the `ipfs/Qm` space. If it can't resolve an IP address, it sends a message to the `resolveErrC` channel and returns an error.

The function also has a `main` function that creates a `select` statement that waits until the `maddrC` channel has data and returns it. This allows the main function to run before the `resolveIPFSPeerless` function, which uses `select` to wait for the `maddrC` channel to have data before returning.


```go
// parseAddresses is a function that takes in a slice of string peer addresses
// (multiaddr + peerid) and returns a slice of properly constructed peers
func parseAddresses(ctx context.Context, addrs []string, rslv *madns.Resolver) ([]peer.AddrInfo, error) {
	// resolve addresses
	maddrs, err := resolveAddresses(ctx, addrs, rslv)
	if err != nil {
		return nil, err
	}

	return peer.AddrInfosFromP2pAddrs(maddrs...)
}

// resolveAddresses resolves addresses parallelly
func resolveAddresses(ctx context.Context, addrs []string, rslv *madns.Resolver) ([]ma.Multiaddr, error) {
	ctx, cancel := context.WithTimeout(ctx, dnsResolveTimeout)
	defer cancel()

	var maddrs []ma.Multiaddr
	var wg sync.WaitGroup
	resolveErrC := make(chan error, len(addrs))

	maddrC := make(chan ma.Multiaddr)

	for _, addr := range addrs {
		maddr, err := ma.NewMultiaddr(addr)
		if err != nil {
			return nil, err
		}

		// check whether address ends in `ipfs/Qm...`
		if _, last := ma.SplitLast(maddr); last.Protocol().Code == ma.P_IPFS {
			maddrs = append(maddrs, maddr)
			continue
		}
		wg.Add(1)
		go func(maddr ma.Multiaddr) {
			defer wg.Done()
			raddrs, err := rslv.Resolve(ctx, maddr)
			if err != nil {
				resolveErrC <- err
				return
			}
			// filter out addresses that still doesn't end in `ipfs/Qm...`
			found := 0
			for _, raddr := range raddrs {
				if _, last := ma.SplitLast(raddr); last != nil && last.Protocol().Code == ma.P_IPFS {
					maddrC <- raddr
					found++
				}
			}
			if found == 0 {
				resolveErrC <- fmt.Errorf("found no ipfs peers at %s", maddr)
			}
		}(maddr)
	}
	go func() {
		wg.Wait()
		close(maddrC)
	}()

	for maddr := range maddrC {
		maddrs = append(maddrs, maddr)
	}

	select {
	case err := <-resolveErrC:
		return nil, err
	default:
	}

	return maddrs, nil
}

```

这段代码定义了一个名为swarmFiltersCmd的命令对象，该对象继承自cmds.Command类。

在该命令对象中，包含了两个方法，分别是helptext和subcommands。其中，helptext方法用于显示该命令的帮助信息，而subcommands方法用于返回该命令的所有子命令。

在该命令的helptext中，使用了一个使用单引号标记的短描述，来描述该命令的作用。描述的内容是"Manipulate address filters."。

在该命令的子命令中，使用了manipulate命令和ipfs命令。其中，ipfs命令的作用是列出当前已经应用的过滤器，而manipulate命令的作用是添加或删除指定的过滤器。

通过这些方法的调用，可以方便地管理和添加/删除系统中的地址过滤器，从而对系统中的网络流量进行更加精确的控制。


```go
var swarmFiltersCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Manipulate address filters.",
		ShortDescription: `
'ipfs swarm filters' will list out currently applied filters. Its subcommands
can be used to add or remove said filters. Filters are specified using the
multiaddr-filter format:

Example:

    /ip4/192.168.0.0/ipcidr/16

Where the above is equivalent to the standard CIDR:

    192.168.0.0/16

```

这段代码是一个滤波器的作用，其作用是过滤那些在 "Swarm.AddrFilters" 配置键下指定的过滤器。

具体来说，这段代码定义了一个名为 "swarmFilters" 的函数，它接收一个 "req" 参数代表请求，一个 "res" 参数代表响应，一个 "env" 参数代表一个 CMD 环境。函数内部首先检查传入的 "n" 是否在线，如果不在线，则返回 "errNotOnline" 的错误。然后，函数遍历 "n.Filters.FiltersForAction(ma.ActionDeny)" 返回的过滤器，将每个过滤器的网络输入转换为字符串并添加到 "output" 数组中。最后，函数使用 "cmds.EmitOnce" 函数将 "output" 数组中的内容输出到 "res" 参数所代表的响应中，并使用 "cmds.Text" 和 "cmds.MakeTypedEncoder" 函数来对 "output" 数组中的内容进行编码。


```go
Filters default to those specified under the "Swarm.AddrFilters" config key.
`,
	},
	Subcommands: map[string]*cmds.Command{
		"add": swarmFiltersAddCmd,
		"rm":  swarmFiltersRmCmd,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !n.IsOnline {
			return ErrNotOnline
		}

		var output []string
		for _, f := range n.Filters.FiltersForAction(ma.ActionDeny) {
			s, err := mamask.ConvertIPNet(&f)
			if err != nil {
				return err
			}
			output = append(output, s)
		}
		return cmds.EmitOnce(res, &stringList{output})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
	Type: stringList{},
}

```

This is a command that adds an address filter to the daemons swarm of IPFS-SMB. The filter will match all transactions containing a specific address.

The command is used by running `ipfs swarm filters add <multiaddr>`. The `<multiaddr>` is a string that specifies the address(es) to filter.

To use this command, you first need to make sure that the `ipfs-SMB` daemon is running on your system. Then, you can run the command with the `ipfs swarm filters add` syntax, followed by the address(es) you want to filter by.


```go
var swarmFiltersAddCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Add an address filter.",
		ShortDescription: `
'ipfs swarm filters add' will add an address filter to the daemons swarm.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("address", true, true, "Multiaddr to filter.").EnableStdin(),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !n.IsOnline {
			return ErrNotOnline
		}

		if len(req.Arguments) == 0 {
			return errors.New("no filters to add")
		}

		r, err := fsrepo.Open(env.(*commands.Context).ConfigRoot)
		if err != nil {
			return err
		}
		defer r.Close()
		cfg, err := r.Config()
		if err != nil {
			return err
		}

		for _, arg := range req.Arguments {
			mask, err := mamask.NewMask(arg)
			if err != nil {
				return err
			}

			n.Filters.AddFilter(*mask, ma.ActionDeny)
		}

		added, err := filtersAdd(r, cfg, req.Arguments)
		if err != nil {
			return err
		}

		return cmds.EmitOnce(res, &stringList{added})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
	Type: stringList{},
}

```

This is a Go function that takes a request object, a response object, and an environment object, and returns an error or a success message.

The function takes an initial request object and an initial response object. The initial request object is passed to the function using the `func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error` parameter.

The function checks if the initial node is online. If the node is not online, the function returns an error. If the node is online, the function opens a file system repository and configures it using the `fsrepo.Open` and `fsrepo.Config` functions.

The function then reads the configuration from the file system repository and applies a policy to remove all rules that match the `ma.ActionDeny` action.

If any errors occur during these steps, the function returns an error and emits a response with a success message using the `cmds.ResponseEmitter.Emit` method.

If the initial request is successful, the function applies the same policy to the initial response object and emits a response with a success message using the `cmds.ResponseEmitter.Emit` method.

If any errors occur during these steps, the function returns an error and emits a response with a failure message using the `cmds.ResponseEmitter.Emit` method.


```go
var swarmFiltersRmCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Remove an address filter.",
		ShortDescription: `
'ipfs swarm filters rm' will remove an address filter from the daemons swarm.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("address", true, true, "Multiaddr filter to remove.").EnableStdin(),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		if !n.IsOnline {
			return ErrNotOnline
		}

		r, err := fsrepo.Open(env.(*commands.Context).ConfigRoot)
		if err != nil {
			return err
		}
		defer r.Close()
		cfg, err := r.Config()
		if err != nil {
			return err
		}

		if req.Arguments[0] == "all" || req.Arguments[0] == "*" {
			fs := n.Filters.FiltersForAction(ma.ActionDeny)
			for _, f := range fs {
				n.Filters.RemoveLiteral(f)
			}

			removed, err := filtersRemoveAll(r, cfg)
			if err != nil {
				return err
			}

			return cmds.EmitOnce(res, &stringList{removed})
		}

		for _, arg := range req.Arguments {
			mask, err := mamask.NewMask(arg)
			if err != nil {
				return err
			}

			n.Filters.RemoveLiteral(*mask)
		}

		removed, err := filtersRemove(r, cfg, req.Arguments)
		if err != nil {
			return err
		}

		return cmds.EmitOnce(res, &stringList{removed})
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
	Type: stringList{},
}

```

这段代码是一个名为 `filtersAdd` 的函数，它接受一个 `repo.Repo` 类型的参数、一个 `config.Config` 类型的参数和一个字符串数组 `filters`。它的作用是添加一些配置相关的过滤器，并将它们添加到 `rm` 状态的 `repo.Repo` 对象中。

该函数的具体实现包括以下几个步骤：

1. 初始化一个名为 `addedMap` 的 map，以及一个空字符串数组 `addedList`。
2. 设置一个名为 `oldFilters` 的字符串数组，它是之前 `cfg.Swarm.AddrFilters` 的值。
3. 遍历 `filters` 数组，对于每个过滤器，执行以下操作：
	1. 如果已经添加过这个过滤器，跳过。
	2. 将 `cfg.Swarm.AddrFilters` 设置为这个过滤器的值，并将 `addedList` 字符串数组中相应位置的值更新为 `true`。
	3. 将 `oldFilters` 数组中相应的过滤器添加到 `cfg.Swarm.AddrFilters` 数组中。
4. 如果配置文件设置失败，返回 `nil`。
5. 返回 `addedList` 和 `nil`，分别代表添加的过滤器和设置的配置文件是否成功。


```go
func filtersAdd(r repo.Repo, cfg *config.Config, filters []string) ([]string, error) {
	addedMap := map[string]struct{}{}
	addedList := make([]string, 0, len(filters))

	// re-add cfg swarm filters to rm dupes
	oldFilters := cfg.Swarm.AddrFilters
	cfg.Swarm.AddrFilters = nil

	// add new filters
	for _, filter := range filters {
		if _, found := addedMap[filter]; found {
			continue
		}

		cfg.Swarm.AddrFilters = append(cfg.Swarm.AddrFilters, filter)
		addedList = append(addedList, filter)
		addedMap[filter] = struct{}{}
	}

	// add back original filters. in this order so that we output them.
	for _, filter := range oldFilters {
		if _, found := addedMap[filter]; found {
			continue
		}

		cfg.Swarm.AddrFilters = append(cfg.Swarm.AddrFilters, filter)
		addedMap[filter] = struct{}{}
	}

	if err := r.SetConfig(cfg); err != nil {
		return nil, err
	}

	return addedList, nil
}

```

这两函数的作用是用于移除Repo中的地址过滤器。通过配置错误的配置项，我们可以输出所有地址过滤器，并将其移除，以避免在部署到生产环境时出现问题。

这两个函数的实现比较复杂，但可以提供比函数更明确的说明。下面是每个函数的更详细的解释：

filtersRemoveAll`

函数接收两个参数：`r` 和 `cfg`。`r` 是 `repo` 实例，`cfg` 是 `Config` 配置项的实例。此函数的作用是设置 `cfg` 中的 `swarm.addr_filters` 字段为 `nil`，并返回移除的地址列表和错误。

函数首先设置 `r.setConfig(cfg)` 函数，如果设置失败，将返回 `nil` 和错误。设置完成后，函数将返回移除的地址列表 `removed` 和错误 `nil`。

filtersRemove`

函数接收三个参数：`r`,`cfg`，和 `toRemoveFilters`。`r` 是 `repo` 实例，`cfg` 是 `Config` 配置项的实例，`toRemoveFilters` 是要删除的地址过滤器列表。此函数的作用是移除 `toRemoveFilters` 中的地址过滤器并将结果存储在 `removed` 数组中，如果发生错误，返回结果和错误。

函数首先设置 `r.setConfig(cfg)` 函数，如果设置失败，将返回 `nil` 和错误。设置完成后，函数将返回移除的地址列表 `removed` 和错误 `nil`。


```go
func filtersRemoveAll(r repo.Repo, cfg *config.Config) ([]string, error) {
	removed := cfg.Swarm.AddrFilters
	cfg.Swarm.AddrFilters = nil

	if err := r.SetConfig(cfg); err != nil {
		return nil, err
	}

	return removed, nil
}

func filtersRemove(r repo.Repo, cfg *config.Config, toRemoveFilters []string) ([]string, error) {
	removed := make([]string, 0, len(toRemoveFilters))
	keep := make([]string, 0, len(cfg.Swarm.AddrFilters))

	oldFilters := cfg.Swarm.AddrFilters

	for _, oldFilter := range oldFilters {
		found := false
		for _, toRemoveFilter := range toRemoveFilters {
			if oldFilter == toRemoveFilter {
				found = true
				removed = append(removed, toRemoveFilter)
				break
			}
		}

		if !found {
			keep = append(keep, oldFilter)
		}
	}
	cfg.Swarm.AddrFilters = keep

	if err := r.SetConfig(cfg); err != nil {
		return nil, err
	}

	return removed, nil
}

```