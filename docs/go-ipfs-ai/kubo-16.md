# go-ipfs 源码解析 16

# `/opt/kubo/core/commands/ping.go`

该代码是一个 Go 语言 package 中的函数命令，主要用于设置和管理社区成员资格（社区成员身份验证）。

具体来说，该代码实现了以下功能：

1. 导入所需的第三方库：cmdenv、errors、fmt、io、strings、time、github.com/ipfs/kubo/core/commands、github.com/ipfs/go-ipfs-cmds、github.com/libp2p/go-libp2p/core/peer、github.com/libp2p/go-libp2p/core/peerstore、github.com/libp2p/go-libp2p/p2p/protocol/ping、github.com/multiformats/go-multiaddr。

2. 定义了一个名为 "commands" 的常量，该常量用于以下库中的函数命令：cmdenv、格式化错误消息、设置社区成员资格相关参数的前缀。

3. 通过 "fmt" 函数，将社区成员资格设置的相关参数格式化，以便在命令行中使用时便于阅读。

4. 通过 "time" 函数，设置社区成员资格验证的时间间隔，以便在设置其它的社区成员资格时能够保证足够的时间间隔。

5. 通过 "import peer" 导入 peer 库中的 "peer" 包，用于设置社区成员资格验证的验证服务器。

6. 通过 "import pstore" 导入 peerstore 包中的 "pstore" 包，用于设置社区成员资格存储的数据结构。

7. 通过 "import cmds" 导入 cmdenv 包中的 "cmds" 函数，用于设置社区成员标识符（类似于互联网之花朵，IPFS 中的一个地址）。

8. 通过 "import math" 导入 "math" 包，用于计算社区成员资格验证后的得分。

9. 通过 "export peer" 导出 "peer" 包中的 "peer" 函数，用于设置社区成员资格验证的服务器。

10. 通过 "export pstore" 导出 "pstore" 包中的 "pstore" 函数，用于设置社区成员资格存储的数据结构。

11. 通过 "export cmds" 导出 "cmds" 函数，用于设置社区成员标识符。

12. 通过 "export patience" 导出 "patience" 函数，用于设置社区成员资格验证的时间间隔。

13. 通过 "export error" 导出 "error" 函数，用于设置社区成员资格验证出现错误时返回的错误信息。

14. 通过 "export run" 导出 "run" 函数，用于设置社区成员资格验证的配置。

15. 通过 "export usage" 导出 "usage" 函数，用于设置社区成员资格验证的使用说明。


```
package commands

import (
	"context"
	"errors"
	"fmt"
	"io"
	"strings"
	"time"

	"github.com/ipfs/kubo/core/commands/cmdenv"

	cmds "github.com/ipfs/go-ipfs-cmds"
	peer "github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	ping "github.com/libp2p/go-libp2p/p2p/protocol/ping"
	ma "github.com/multiformats/go-multiaddr"
)

```

这段代码定义了一个名为 `PingResult` 的结构体类型，用于表示网络包发送 `Ping` 请求的结果。该类型包含三个字段：`Success`（是否成功）、`Time`（请求执行时间）和 `Text`（错误信息，如果有的话）。

接下来，定义了一个名为 `PingCount` 的常量，它的作用是限制 `Ping` 请求的最大次数。当超过这个限制时，会抛出一个名为 `ErrPingSelf` 的错误。

最后，定义了一个 `PingTimeout` 常量，它的值为 `10 * time.Second`，表示允许的最短 `Ping` 请求超时时间。这个常量被用于 `PingCount` 的 `count` 选项中，当 `count` 设置为 `0` 时，允许 `Ping` 请求的最大次数为 `PingCount` 减一。


```
const kPingTimeout = 10 * time.Second

type PingResult struct {
	Success bool
	Time    time.Duration
	Text    string
}

const (
	pingCountOptionName = "count"
)

// ErrPingSelf is returned when the user attempts to ping themself.
var ErrPingSelf = errors.New("error: can't ping self")

```

This is a Go-like language and it looks like it implements theering.io API. The `res` slice seems to be a buffer of network requests and the `re` instance is a Receiver. It receives and emits events based on the received requests. The `PingResult` struct represents a received ping request and the `Emit` method sends the event to the receiver. The `Text` field of the `PingResult` struct is optional and contains the actual text to be included in the event log.

The code implements a simple ping server that listens for incoming pings and sends them a response. The server uses a buffer to store incoming requests and a receiver to handle the events emitted by the receiver. The receiver receives and processes each event, updating the total latency time received from the pings. If the receiver receives multiple pings, it average the latency time over the received pings and sends the information in a `PingResult` struct to the receiver.


```
var PingCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Send echo request packets to IPFS hosts.",
		ShortDescription: `
'ipfs ping' is a tool to test sending data to other nodes. It finds nodes
via the routing system, sends pings, waits for pongs, and prints out round-
trip latency information.
		`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("peer ID", true, true, "ID of peer to be pinged.").EnableStdin(),
	},
	Options: []cmds.Option{
		cmds.IntOption(pingCountOptionName, "n", "Number of ping messages to send.").WithDefault(10),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		// Must be online!
		if !n.IsOnline {
			return ErrNotOnline
		}

		addr, pid, err := ParsePeerParam(req.Arguments[0])
		if err != nil {
			return fmt.Errorf("failed to parse peer address '%s': %s", req.Arguments[0], err)
		}

		if pid == n.Identity {
			return ErrPingSelf
		}

		if addr != nil {
			n.Peerstore.AddAddr(pid, addr, pstore.TempAddrTTL) // temporary
		}

		numPings, _ := req.Options[pingCountOptionName].(int)
		if numPings <= 0 {
			return fmt.Errorf("ping count must be greater than 0, was %d", numPings)
		}

		if len(n.Peerstore.Addrs(pid)) == 0 {
			// Make sure we can find the node in question
			if err := res.Emit(&PingResult{
				Text:    fmt.Sprintf("Looking up peer %s", pid),
				Success: true,
			}); err != nil {
				return err
			}

			ctx, cancel := context.WithTimeout(req.Context, kPingTimeout)
			p, err := n.Routing.FindPeer(ctx, pid)
			cancel()
			if err != nil {
				return fmt.Errorf("peer lookup failed: %s", err)
			}
			n.Peerstore.AddAddrs(p.ID, p.Addrs, pstore.TempAddrTTL)
		}

		if err := res.Emit(&PingResult{
			Text:    fmt.Sprintf("PING %s.", pid),
			Success: true,
		}); err != nil {
			return err
		}

		ctx, cancel := context.WithTimeout(req.Context, kPingTimeout*time.Duration(numPings))
		defer cancel()
		pings := ping.Ping(ctx, n.PeerHost, pid)

		var (
			count int
			total time.Duration
		)
		ticker := time.NewTicker(time.Second)
		defer ticker.Stop()

		for i := 0; i < numPings; i++ {
			r, ok := <-pings
			if !ok {
				break
			}

			if r.Error != nil {
				err = res.Emit(&PingResult{
					Success: false,
					Text:    fmt.Sprintf("Ping error: %s", r.Error),
				})
			} else {
				count++
				total += r.RTT
				err = res.Emit(&PingResult{
					Success: true,
					Time:    r.RTT,
				})
			}
			if err != nil {
				return err
			}

			select {
			case <-ticker.C:
			case <-ctx.Done():
				return ctx.Err()
			}
		}
		if count == 0 {
			return fmt.Errorf("ping failed")
		}
		averagems := total.Seconds() * 1000 / float64(count)
		return res.Emit(&PingResult{
			Success: true,
			Text:    fmt.Sprintf("Average latency: %.2fms", averagems),
		})
	},
	Type: PingResult{},
	PostRun: cmds.PostRunMap{
		cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
			var (
				total time.Duration
				count int
			)

			for {
				event, err := res.Next()
				switch err {
				case nil:
				case io.EOF:
					return nil
				case context.Canceled, context.DeadlineExceeded:
					if count == 0 {
						return err
					}
					averagems := total.Seconds() * 1000 / float64(count)
					return re.Emit(&PingResult{
						Success: true,
						Text:    fmt.Sprintf("Average latency: %.2fms", averagems),
					})
				default:
					return err
				}

				pr := event.(*PingResult)
				if pr.Success && pr.Text == "" {
					total += pr.Time
					count++
				}
				err = re.Emit(event)
				if err != nil {
					return err
				}
			}
		},
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *PingResult) error {
			if len(out.Text) > 0 {
				fmt.Fprintln(w, out.Text)
			} else if out.Success {
				fmt.Fprintf(w, "Pong received: time=%.2f ms\n", out.Time.Seconds()*1000)
			} else {
				fmt.Fprintf(w, "Pong failed\n")
			}
			return nil
		}),
	},
}

```

此代码定义了一个名为 `ParsePeerParam` 的函数，其接收一个字符串参数 `text`。这个函数的作用是解析 `text` 中的 peer 地址信息，并返回一个匹配的 `MA` 对象、一个 `peer.ID` 值和一个错误对象。

具体来说，函数的实现分为以下几个步骤：

1. 如果 `text` 的开头是 `/`，则将 `text` 解析为一个 `MA` 对象。
2. 如果 `text` 不是 `/` 开头的字符串，尝试使用 `strings.HasPrefix` 函数检查 `text` 是否以 `/` 开头。如果是，则执行以下操作：

a. 尝试使用 `ma.NewMultiaddr` 函数创建一个 `MA` 对象。如果失败，返回 `nil` 和一个错误对象。

b. 使用 `peer.SplitAddr` 函数将 `MA` 对象的分发路由器 `transport` 和 `id` 分离出来。

c. 如果 `id` 是空字符串，使用 `peer.ErrInvalidAddr` 错误对象返回。

d. 如果 `text` 不是以 `/` 开头的字符串，执行以下操作：

e. 返回 `nil` 和一个错误对象。

f. 返回 `text`。

函数的作用是接收一个字符串参数 `text`，解析 `text` 中的 peer 地址信息，并返回一个匹配的 `MA` 对象、一个 `peer.ID` 值和一个错误对象。


```
func ParsePeerParam(text string) (ma.Multiaddr, peer.ID, error) {
	// Multiaddr
	if strings.HasPrefix(text, "/") {
		maddr, err := ma.NewMultiaddr(text)
		if err != nil {
			return nil, "", err
		}
		transport, id := peer.SplitAddr(maddr)
		if id == "" {
			return nil, "", peer.ErrInvalidAddr
		}
		return transport, id, nil
	}
	// Raw peer ID
	p, err := peer.Decode(text)
	return nil, p, err
}

```

# `/opt/kubo/core/commands/profile.go`

这段代码是一个 Go 语言编写的命令行工具，它主要用于在本地机器上安装并配置 IPFS（InterPlanetary File System）库。IPFS 是一个去中心化的点对点分布式文件系统，可以快速、安全地共享文件。

以下是这段代码的作用：

1. 导入需要的库：首先，导入 "archive/zip"、"fmt"、"io"、"os" 和 "strings"，这些库分别用于处理压缩文件、格式化字符串、输入输出和字符串操作。

2. 加载 IPFS 配置：从主配置文件（如 "ipfs.json"）中读取 IPFS 配置，并将其存储在 "config" 变量中。我们假设 IPFS 已经安装在系统中，并且已经配置好了主节点和人类节点。

3. 创建输出目录：如果当前目录不存在，就创建一个输出目录。

4. 压缩文件：使用 IPFS 提供的 "compress" 和 "decompress" 函数对文件进行压缩和解压缩。注意，这些函数的实现可能因具体需求而异。压缩后的文件将命名为 "output.zip"。

5. 压缩文件到 IPFS：将当前文件压缩为 IPFS 支持的压缩格式，并将其存储在 "output.zip" 中。

6. 等待压缩完成：在文件进行压缩时，使用 "time.Sleep" 函数暂停时间，以避免在漫长的压缩过程中导致程序卡死。

7. 打印进度：在文件开始压缩时，打印 "正在压缩..."，在文件压缩完成后，打印 "压缩完成，共压缩了 "，以及一个数字表示已压缩的文件数量。

8. 使用 GitHub Actions：最后，通过运行 GitHub Actions 来自动化部署这个命令行工具，以便在代码仓库的页面中显示 ipfs 密度的额外信息。


```
package commands

import (
	"archive/zip"
	"fmt"
	"io"
	"os"
	"strings"
	"time"

	cmds "github.com/ipfs/go-ipfs-cmds"
	"github.com/ipfs/kubo/core/commands/e"
	"github.com/ipfs/kubo/profile"
)

```

此代码是一个 Go 语言中的函数，它对 `time.RFC3339` 中的时间格式进行了修改，使其能够适用于在 Windows 文件中的命名。

具体来说，函数首先使用 `strings.ReplaceAll` 函数从 `time.RFC3339` 中删除 `:` 并添加 `_`，然后将其返回。接下来，定义了一个名为 `profileResult` 的结构体，其中包含一个名为 `File` 的字段。

然后，定义了一个名为 `collectorsOptionName`、`profileTimeOption`、`mutexProfileFractionOption` 和 `blockProfileRateOption` 的常量。

最后，定义了一个名为 `sysProfileCmd` 的常量，该常量指向一个名为 `cmds.Command` 的类型，该类型包含一个可以帮助使用 `sysProfileCmd` 的 `Helptext` 字段。


```
// time format that works in filenames on windows.
var timeFormat = strings.ReplaceAll(time.RFC3339, ":", "_")

type profileResult struct {
	File string
}

const (
	collectorsOptionName       = "collectors"
	profileTimeOption          = "profile-time"
	mutexProfileFractionOption = "mutex-profile-fraction"
	blockProfileRateOption     = "block-profile-rate"
)

var sysProfileCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "Collect a performance profile for debugging.",
		ShortDescription: `
```

这段代码的作用是将从运行中的 go-ipfs 守护程序中收集到的配置文件夹到单独的 zip 文件中，并尝试包括一个运行中的 go-ipfs  binary 的副本。这个命令旨在帮助进行调试，同时也可以在需要时方便地查看运行中的 go-ipfs  binary 的堆栈跟踪。

具体来说，这个命令会运行以下命令并传递一些参数给它们：


go run collect_profiles.go


其中，`collect_profiles.go` 是包含上述注释的文件，它定义了一个函数 `collect_profiles`，它会将从守护程序中收集到的配置文件夹压缩到一个名为 `collect_profiles.zip` 的文件中，并设置 `--no_ipfs_ binary` 和 `--profile_path` 选项以指定要包括的二进制文件和配置文件夹的路径。

`--profile_path` 选项指定要包括的二进制文件路径，如果指定了此路径，则会覆盖 `collect_profiles.zip` 文件中的同名部分，并将其重命名为 `<base_profile_name>.ipfs`。

此外，`go tool pprof` 可以用来查看生成的堆栈跟踪，它会在运行期间输出一系列类似于下面这样的条目：


2121111121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212121212


```
Collects profiles from a running go-ipfs daemon into a single zip file.
To aid in debugging, this command also attempts to include a copy of
the running go-ipfs binary.
`,
		LongDescription: `
Collects profiles from a running go-ipfs daemon into a single zipfile.
To aid in debugging, this command also attempts to include a copy of
the running go-ipfs binary.

Profiles can be examined using 'go tool pprof', some tips can be found at
https://github.com/ipfs/kubo/blob/master/docs/debug-guide.md.

Privacy Notice:

The output file includes:

```

这段代码实现了一个对Go-IPFS进行监控和分析的工具，提供了运行中的 Goroutines、CPU 分配情况、堆内存使用情况、堆内存分配情况、互斥锁使用情况、块级分配情况以及 Go-IPFS 的版本信息。它并不包含任何与 Go-IPFS 数据或元数据相关的信息，也不包含您的配置文件或私钥，以及您的计算机的内存、文件系统等内部信息。


```
- A list of running goroutines.
- A CPU profile.
- A heap inuse profile.
- A heap allocation profile.
- A mutex profile.
- A block profile.
- Your copy of go-ipfs.
- The output of 'ipfs version --all'.

It does not include:

- Any of your IPFS data or metadata.
- Your config or private key.
- Your IP address.
- The contents of your computer's memory, filesystem, etc.

```

This appears to be a Go language implementation of an IPFS (InterPlanetary File System) profile. The `profile.Options` parameter is a dictionary that can be used to configure the IPFS profile.

The `archive.Close()` call at the end of the `PostRun` block is a mistake, as it seems to be discarding the contents of the IPFS profile. It is not clear why this is the case, as the code does not reference the contents of the profile.

It is also worth noting that the code does not handle errors that may occur during profile creation. For example, the file may not have the necessary permissions, or the profile may already exist. It is possible that these errors will cause the profile to fail and should be handled appropriately.


```
However, it could reveal:

- Your build path, if you built go-ipfs yourself.
- If and how a command/feature is being used (inferred from running functions).
- Memory offsets of various data structures.
- Any modifications you've made to go-ipfs.
`,
	},
	NoLocal: true,
	Options: []cmds.Option{
		cmds.StringOption(outputOptionName, "o", "The path where the output .zip should be stored. Default: ./ipfs-profile-[timestamp].zip"),
		cmds.DelimitedStringsOption(",", collectorsOptionName, "The list of collectors to use for collecting diagnostic data.").
			WithDefault([]string{
				profile.CollectorGoroutinesStack,
				profile.CollectorGoroutinesPprof,
				profile.CollectorVersion,
				profile.CollectorHeap,
				profile.CollectorAllocs,
				profile.CollectorBin,
				profile.CollectorCPU,
				profile.CollectorMutex,
				profile.CollectorBlock,
			}),
		cmds.StringOption(profileTimeOption, "The amount of time spent profiling. If this is set to 0, then sampling profiles are skipped.").WithDefault("30s"),
		cmds.IntOption(mutexProfileFractionOption, "The fraction 1/n of mutex contention events that are reported in the mutex profile.").WithDefault(4),
		cmds.StringOption(blockProfileRateOption, "The duration to wait between sampling goroutine-blocking events for the blocking profile.").WithDefault("1ms"),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		collectors := req.Options[collectorsOptionName].([]string)

		profileTimeStr, _ := req.Options[profileTimeOption].(string)
		profileTime, err := time.ParseDuration(profileTimeStr)
		if err != nil {
			return fmt.Errorf("failed to parse profile duration %q: %w", profileTimeStr, err)
		}

		blockProfileRateStr, _ := req.Options[blockProfileRateOption].(string)
		blockProfileRate, err := time.ParseDuration(blockProfileRateStr)
		if err != nil {
			return fmt.Errorf("failed to parse block profile rate %q: %w", blockProfileRateStr, err)
		}

		mutexProfileFraction, _ := req.Options[mutexProfileFractionOption].(int)

		r, w := io.Pipe()

		go func() {
			archive := zip.NewWriter(w)
			err = profile.WriteProfiles(req.Context, archive, profile.Options{
				Collectors:           collectors,
				ProfileDuration:      profileTime,
				MutexProfileFraction: mutexProfileFraction,
				BlockProfileRate:     blockProfileRate,
			})
			archive.Close()
			_ = w.CloseWithError(err)
		}()
		return res.Emit(r)
	},
	PostRun: cmds.PostRunMap{
		cmds.CLI: func(res cmds.Response, re cmds.ResponseEmitter) error {
			v, err := res.Next()
			if err != nil {
				return err
			}

			outReader, ok := v.(io.Reader)
			if !ok {
				return e.New(e.TypeErr(outReader, v))
			}

			outPath, _ := res.Request().Options[outputOptionName].(string)
			if outPath == "" {
				outPath = "ipfs-profile-" + time.Now().Format(timeFormat) + ".zip"
			}
			fi, err := os.Create(outPath)
			if err != nil {
				return err
			}
			defer fi.Close()

			_, err = io.Copy(fi, outReader)
			if err != nil {
				return err
			}
			return re.Emit(&profileResult{File: outPath})
		},
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *profileResult) error {
			fmt.Fprintf(w, "Wrote profiles to: %s\n", out.File)
			return nil
		}),
	},
}

```

# `/opt/kubo/core/commands/pubsub.go`

这段代码定义了一个名为"commands"的包，其中定义了一些用于管理IPFS(InterPlanetary File System)和其他网络服务的命令。具体来说，这些命令包括：

1. "fmt"：用于格式化输入/输出。
2. "io"、"net/http" 和 "sort"：这三个库分别用于文件I/O、网络HTTP请求和排序。
3. "github.com/ipfs/kubo/core/commands/cmdenv"：通过导入 "github.com/ipfs/kubo/core/commands/cmdenv" 来使用Kubernetes(k8s)命令行工具中的"cmdenv"命令。
4. "github.com/multiformats/go-multibase"：通过导入 "github.com/multiformats/go-multibase" 来支持Multi-formats(MF)格式的数据。
5. "github.com/pkg/errors"：通过导入 "github.com/pkg/errors" 来处理错误。
6. "github.com/ipfs/boxo/coreiface/options"：通过导入 "github.com/ipfs/boxo/coreiface/options" 来支持IPFS的选项。
7. "github.com/ipfs/go-ipfs-cmds"：通过导入 "github.com/ipfs/go-ipfs-cmds" 来支持IPFS命令行工具中的"go-ipfs-cmds"命令。


```
package commands

import (
	"context"
	"fmt"
	"io"
	"net/http"
	"sort"

	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"
	mbase "github.com/multiformats/go-multibase"
	"github.com/pkg/errors"

	options "github.com/ipfs/boxo/coreiface/options"
	cmds "github.com/ipfs/go-ipfs-cmds"
)

```

该代码定义了一个名为 PubsubCmd 的命令对象，它实现了 PubSub 系统。PubSub 是一个基于 IPFS(InterPlanetary File System)的实验性发布订阅系统。该系统允许用户在给定的主题下发布消息并订阅新的消息。

具体来说，PubSubCmd 的作用是帮助用户在 IPFS 上进行发布、订阅和查看消息，其中包括发布到指定主题的消息和订阅到指定主题的新消息。此外，该系统还提供了一个 DEPRECATED FEATURE( see https://github.com/ipfs/kubo/issues/9717)的实现，即在生产环境中使用时需要运行的命令行选项。

该命令对象通过 map 函数与几个 CMDS(命令对象模型)关联，包括 PubSubPubCmd、PubsubSubCmd、PubsubLsCmd 和 PubsubPeersCmd。这些函数一起实现了 PubSub 系统的核心功能。


```
var PubsubCmd = &cmds.Command{
	Status: cmds.Deprecated,
	Helptext: cmds.HelpText{
		Tagline: "An experimental publish-subscribe system on ipfs.",
		ShortDescription: `
ipfs pubsub allows you to publish messages to a given topic, and also to
subscribe to new messages on a given topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.
`,
	},
	Subcommands: map[string]*cmds.Command{
		"pub":   PubsubPubCmd,
		"sub":   PubsubSubCmd,
		"ls":    PubsubLsCmd,
		"peers": PubsubPeersCmd,
	},
}

```

这是一个Go语言中的结构体定义，表示一个PubSubMessage类型。PubSubMessage结构体包含了从哪里发送消息、消息内容以及消息在哪个主题上发送的一些信息。

接下来是一个Go语言中的命令行脚本，它使用了cmds.Command，输出了一个订阅消息到指定主题的命令行工具。它接收一些参数，用于指定要订阅的主题、要发送消息的内容，以及消息序列号。

最后，这个脚本定义了一个名为 PubsubSubCmd 的常量，它是一个cmds.Command类型的变量，用于存储 cmds.Command 的实例。


```
type pubsubMessage struct {
	From     string   `json:"from,omitempty"`
	Data     string   `json:"data,omitempty"`
	Seqno    string   `json:"seqno,omitempty"`
	TopicIDs []string `json:"topicIDs,omitempty"`
}

var PubsubSubCmd = &cmds.Command{
	Status: cmds.Deprecated,
	Helptext: cmds.HelpText{
		Tagline: "Subscribe to messages on a given topic.",
		ShortDescription: `
ipfs pubsub sub subscribes to messages on a given topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

```

This is a Go routine that defines a data serializer for a PubSubMessage type. The serializer encodes the message data into text format and emits it over a publish channel.

The serializer has an encoder for the message's data and a decoder for the text format. The encoder uses the undocumented format you described, while the decoder uses the json format.

The serializer also supports a custom encryption/decryption mechanism using the "ndpayload" and "lenpayload" formats. These formats are not recommended for use, as they were removed in newer versions of the Go standard.

The data serializer can be used in conjunction with the "text" or "json" formats, as follows:
php
req, err := stream.Reader() // Create a reader from the incoming stream
if err != nil {
	return err
}

psm, err := stream.Decode(req, &psm) // Decode the message data from the reader
if err != nil {
	return err
}

data, err := stream.Decode(psm.Data(), &data) // Decode the data from the message
if err != nil {
	return err
}

stream, err := data.Encode(stream) // Encode the data to the output stream
if err != nil {
	return err
}

res, err := stream.Emit(psm) // Emit the message data to the output stream
if err != nil {
	return err
}

This code snippet demonstrates how to use the data serializer to serialize a PubSubMessage to the output stream. The serializer can then be used with the "text" or "json" formats as described in the Go standard.


```
PEER ENCODING

  Peer IDs in From fields are encoded using the default text representation
  from go-libp2p. This ensures the same string values as in 'ipfs pubsub peers'.

TOPIC AND DATA ENCODING

  Topics, Data and Seqno are binary data. To ensure all bytes are transferred
  correctly the RPC client and server will use multibase encoding behind
  the scenes.

  You can inspect the format by passing --enc=json. The ipfs multibase commands
  can be used for encoding/decoding multibase strings in the userland.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("topic", true, false, "Name of topic to subscribe to (multibase encoded when sent over HTTP RPC)."),
	},
	PreRun: urlArgsEncoder,
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}
		if err := urlArgsDecoder(req, env); err != nil {
			return err
		}

		topic := req.Arguments[0]

		sub, err := api.PubSub().Subscribe(req.Context, topic)
		if err != nil {
			return err
		}
		defer sub.Close()

		if f, ok := res.(http.Flusher); ok {
			f.Flush()
		}

		for {
			msg, err := sub.Next(req.Context)
			if err == io.EOF || err == context.Canceled {
				return nil
			} else if err != nil {
				return err
			}

			// turn bytes into strings
			encoder, _ := mbase.EncoderByName("base64url")
			psm := pubsubMessage{
				Data:  encoder.Encode(msg.Data()),
				From:  msg.From().String(),
				Seqno: encoder.Encode(msg.Seq()),
			}
			for _, topic := range msg.Topics() {
				psm.TopicIDs = append(psm.TopicIDs, encoder.Encode([]byte(topic)))
			}
			if err := res.Emit(&psm); err != nil {
				return err
			}
		}
	},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
			_, dec, err := mbase.Decode(psm.Data)
			if err != nil {
				return err
			}
			_, err = w.Write(dec)
			return err
		}),
		// DEPRECATED, undocumented format we used in tests, but not anymore
		// <message.payload>\n<message.payload>\n
		"ndpayload": cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
			return errors.New("--enc=ndpayload was removed, use --enc=json instead")
		}),
		// DEPRECATED, uncodumented format we used in tests, but not anymore
		// <varint-len><message.payload><varint-len><message.payload>
		"lenpayload": cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, psm *pubsubMessage) error {
			return errors.New("--enc=lenpayload was removed, use --enc=json instead")
		}),
	},
	Type: pubsubMessage{},
}

```

这段代码是一个名为"PubsubPubCmd"的命令对象，属于名为"cmds"的包。它的作用是发布数据到指定的pubsub主题。

具体来说，它通过读取二进制数据(可能来自标准输入或文件)并将消息发送到指定主题来实现这一目的。在代码中，使用了一个名为"ipfs pubsub"的命令来执行动作，这需要使用"--enable-pubsub-experiment"选项来启用。此外，代码还使用了来自"ipfs"包的"publishes a message to a specified topic"来表示发布消息到主题的作用。

由于该代码处于被 deprecated(已过时)的状态，因此它的实现被弃用，不应该在生产环境中使用。


```
var PubsubPubCmd = &cmds.Command{
	Status: cmds.Deprecated,
	Helptext: cmds.HelpText{
		Tagline: "Publish data to a given pubsub topic.",
		ShortDescription: `
ipfs pubsub pub publishes a message to a specified topic.
It reads binary data from stdin or a file.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

HTTP RPC ENCODING

  The data to be published is sent in HTTP request body as multipart/form-data.

  Topic names are binary data too. To ensure all bytes are transferred
  correctly via URL params, the RPC client and server will use multibase
  encoding behind the scenes.

```

该代码是一个 Go 语言中的命令行工具，名为 "pubsub"，用于在本地或远程服务器上发布主题到订阅者。

具体来说，该工具接受一个参数 "topic"，表示要发布到的主题，可以通过多基础因式编码发送通过 HTTP RPC。还有一个参数 "data"，表示要发布到服务器的数据。

在运行时，该工具首先尝试从请求中读取有关 "topic" 和 "data" 的参数。如果从请求中读取参数有错误，如 "topic" 或 "data" 没有提供足够的信息，则会返回并丢弃这些参数。

接着，该工具尝试从请求中提供的文件中读取数据。如果从请求的文件中读取数据有错误，如文件不存在或数据无法读取，则会返回并丢弃这些文件。

最后，该工具使用服务器 API 发布主题到订阅者。主题是通过多基础因式编码的，这使得主题可以作为 JSON 键的一部分在 HTTP RPC 中传输。

注意，该代码中的所有 API 调用都是基于 Go 的云函数，并且使用了 Go 的标准库 "fmt"，"os"，"strings"，"unicode"，"net/http"，"io/ioutil"，"golang.org/x/net/context"，以及 "golang.org/x/os/err" 等。


```
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("topic", true, false, "Topic to publish to (multibase encoded when sent over HTTP RPC)."),
		cmds.FileArg("data", true, false, "The data to be published.").EnableStdin(),
	},
	PreRun: urlArgsEncoder,
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}
		if err := urlArgsDecoder(req, env); err != nil {
			return err
		}

		topic := req.Arguments[0]

		// read data passed as a file
		file, err := cmdenv.GetFileArg(req.Files.Entries())
		if err != nil {
			return err
		}
		defer file.Close()
		data, err := io.ReadAll(file)
		if err != nil {
			return err
		}

		// publish
		return api.PubSub().Publish(req.Context, topic, data)
	},
}

```

这段代码是一个名为 PubsubLsCmd 的命令行工具，它旨在列出用户当前订阅的主题名称。该命令使用户能够通过使用 "ipfs pubsub ls" 命令来查看他们订阅的主题列表。

该命令使用一个名为 cmds 的包中的命令行工具类，该类提供了一些基本的命令行接口。在该命令中，变量 PubsubLsCmd 是一个名为 cmds.Command 的引用，该引用包含命令的一些基本信息，如状态（cmds.Deprecated）和帮助文本。

Helptext 变量包含一个使用新主题名称格式帮助文本，该文本将在用户运行命令时显示。DEPRECATED FEATURE 变量提醒用户该命令在发布环境中不应使用，因为它已被弃用。

在帮助文本中，有一个名为 TopicEncoding 的短描述，它指出了该命令使用的主题名称编码方式。该描述还提到了一个名为 TopicEncodings 的变量，它包含一个 JSON 格式的主题编码格式。


```
var PubsubLsCmd = &cmds.Command{
	Status: cmds.Deprecated,
	Helptext: cmds.HelpText{
		Tagline: "List subscribed topics by name.",
		ShortDescription: `
ipfs pubsub ls lists out the names of topics you are currently subscribed to.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

TOPIC ENCODING

  Topic names are a binary data. To ensure all bytes are transferred
  correctly RPC client and server will use multibase encoding behind
  the scenes.

  You can inspect the format by passing --enc=json. ipfs multibase commands
  can be used for encoding/decoding multibase strings in the userland.
```

这段代码是一个 Go 语言编写的命令行工具的实现，它主要用于在讨论组中发布主题。具体来说，它的作用是将一组主题发送到指定的讨论组，并将它们编码成“base64url”格式的字节。

下面是具体代码的功能解释：

1. `Run` 函数接受一个请求参数 `req`，一个响应参数 `res`，以及一个环境参数 `env`。它的作用是在环境 `env` 下获取指定的 API，并使用它来获取讨论组。如果获取失败，函数返回一个错误。
2. 在获取到 API 之后，函数调用 `api.PubSub().Ls` 方法，并传递 `req.Context` 作为参数。这个方法返回一个指向讨论组名称的指针，如果有错误，函数将返回。
3. 对于每个主题，函数使用一个名为 `base64url` 的编码器来编码它，并将编码后的主题存储在一个名为 `l` 的列表中。
4. 最后，函数使用 `cmds.EmitOnce` 函数将编码好的主题列表发送到指定的响应 `res` 对象中，每个主题都会被封装成一个独立的负载数据。
5. 函数的类型设置为 `stringList`，它要求传入的参数列表必须包含一个或多个字符串。
6. 函数的编码器映射使用了一个名为 `multibaseDecodedStringListEncoder` 的自定义编码器，它将编码后的主题列表解码成对应的原始消息主题。

整个工具的作用是将一组主题发送到指定的讨论组，并将它们编码成“base64url”格式的字节。


```
`,
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		l, err := api.PubSub().Ls(req.Context)
		if err != nil {
			return err
		}

		// emit topics encoded in multibase
		encoder, _ := mbase.EncoderByName("base64url")
		for n, topic := range l {
			l[n] = encoder.Encode([]byte(topic))
		}

		return cmds.EmitOnce(res, stringList{l})
	},
	Type: stringList{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(multibaseDecodedStringListEncoder),
	},
}

```

这两段代码都是函数，它们的作用是帮助处理请求参数 "req"、输出 "w" 和输入参数 "list"，并在处理过程中执行安全检查。

第一段代码 "multibaseDecodedStringListEncoder" 接收一个 "Request" 参数，一个 "Writer" 参数和一个字符串数组 "list"。它遍历输入参数 "list" 中的每个元素，并将每个元素的字符串编码为 "mb" 并存储到 "list.Strings" 数组中。然后，它使用 "safeTextListEncoder" 函数将数组中的字符串转换为文本表示形式，这样可以避免非打印able/unsafe 字符集对输出结果的损害，同时也可以保护终端输出不受到非 ASCII 主题名称的损害。

第二段代码 "safeTextListEncoder" 接收一个 "Request" 参数、一个 "Writer" 参数和一个字符串数组 "list"。它遍历输入参数 "list" 中的每个元素，并使用 "fmt" 函数将每个元素的字符串编码为字符串，并在编码过程中使用 "cmdenv.EscNonPrint" 函数将非打印able字符集中的字符进行转义。最后，它使用 "safeTextListEncoder" 函数将编码后的字符串打印到输出 "Writer" 上。


```
func multibaseDecodedStringListEncoder(req *cmds.Request, w io.Writer, list *stringList) error {
	for n, mb := range list.Strings {
		_, data, err := mbase.Decode(mb)
		if err != nil {
			return err
		}
		list.Strings[n] = string(data)
	}
	return safeTextListEncoder(req, w, list)
}

// converts list of strings to text representation where each string is placed
// in separate line with non-printable/unsafe characters escaped
// (this protects terminal output from being mangled by non-ascii topic names)
func safeTextListEncoder(req *cmds.Request, w io.Writer, list *stringList) error {
	for _, str := range list.Strings {
		_, err := fmt.Fprintf(w, "%s\n", cmdenv.EscNonPrint(str))
		if err != nil {
			return err
		}
	}
	return nil
}

```

此代码是一个PowerShell脚本，它定义了一个名为"PubsubPeersCmd"的命令对象。这个命令对象表示一个可以用来列出当前正在使用PubSub代理与我们连接的端点的工具。

具体来说，这个脚本实现了以下功能：

1. 获取与PubSub服务器连接的所有端点。
2. 通过传递主题参数，列出与指定主题订阅的端点。
3. 输出连接到主题的端点，以便用户监视他们正在订阅的主题。

值得注意的是，这个脚本还输出了一个DEPRECATED FEATURE警告，因为这个功能在将来不应该被用于生产环境。在当前状态下，它可能已经被测试用于开发和调试目的。


```
var PubsubPeersCmd = &cmds.Command{
	Status: cmds.Deprecated,
	Helptext: cmds.HelpText{
		Tagline: "List peers we are currently pubsubbing with.",
		ShortDescription: `
ipfs pubsub peers with no arguments lists out the pubsub peers you are
currently connected to. If given a topic, it will list connected peers who are
subscribed to the named topic.

DEPRECATED FEATURE (see https://github.com/ipfs/kubo/issues/9717)

  It is not intended in its current state to be used in a production
  environment.  To use, the daemon must be run with
  '--enable-pubsub-experiment'.

```

此代码定义了一个用于在命令行界面（CLI）中指定主题并输出连接的IoTFS客户端和服务器之间的多基质数据编码的`TOPIC`命令。

具体来说，此命令使用`--enc=json`选项向用户输出使用JSON格式的数据。`TOPIC`参数是一个字符串，用于指定要列出连接的IoTFS主题。如果只有一项参数，则将其作为主题传递给命令。

在函数内部，使用`api.PubSub().Peers`函数获取IoTFS客户端和服务器之间的连接。如果此操作成功，则执行以下操作：

1. 从连接中获取主题列表。
2. 对主题列表进行排序。
3. 将主题列表编码为JSON格式并输出。

如果出现任何错误，例如从客户端收到的数据不可用或IoTFS服务器返回的错误消息，则返回错误。


```
TOPIC AND DATA ENCODING

  Topic names are a binary data. To ensure all bytes are transferred
  correctly RPC client and server will use multibase encoding behind
  the scenes.

  You can inspect the format by passing --enc=json. ipfs multibase commands
  can be used for encoding/decoding multibase strings in the userland.
`,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("topic", false, false, "Topic to list connected peers of."),
	},
	PreRun: urlArgsEncoder,
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}
		if err := urlArgsDecoder(req, env); err != nil {
			return err
		}

		var topic string
		if len(req.Arguments) == 1 {
			topic = req.Arguments[0]
		}

		peers, err := api.PubSub().Peers(req.Context, options.PubSub.Topic(topic))
		if err != nil {
			return err
		}

		list := &stringList{make([]string, 0, len(peers))}

		for _, peer := range peers {
			list.Strings = append(list.Strings, peer.String())
		}
		sort.Strings(list.Strings)
		return cmds.EmitOnce(res, list)
	},
	Type: stringList{},
	Encoders: cmds.EncoderMap{
		cmds.Text: cmds.MakeTypedEncoder(safeTextListEncoder),
	},
}

```

这段代码定义了两个函数，分别是 `urlArgsEncoder` 和 `urlArgsDecoder`。它们的作用是将传入的 binary 数据编码成可以作为多基数字符串传递给 URL 参数的格式。

具体来说，这两函数接受一个 `cmds.Request` 类型的请求对象和一个 `cmds.Environment` 类型的环境对象作为参数。在函数内部，通过调用 `mbase.EncoderByName("base64url")` 函数，将传入的 binary 数据编码成 Base64 格式，并返回结果。在另一个函数中，通过遍历请求的 URL 参数，将编码好的 binary 数据作为参数传递给 `mbase.Decode` 函数。如果 `mbase.Decode` 函数返回的错误信息中包含 "URL arg must be multibase encoded" 或 "URL arg must be base64url encoded"，则会返回一个错误并传入相应的错误信息。否则，正确地设置 `encoding` 为 `mbase.Base64url`，并将解码后的数据作为结果返回。

在函数内部，由于 `mbase.Decode` 函数需要一个编码作为参数，而 `urlArgsEncoder` 和 `urlArgsDecoder` 函数需要将 binary 数据作为参数传递，因此它们需要对 `mbase.Decode` 函数进行修改，以便正确地接收和处理多基数字符串。具体来说，这两个函数需要在返回前对 `mbase.Decode` 函数的错误信息进行处理，以使其能够正确地处理 `urlArgsEncoder` 和 `urlArgsDecoder` 函数传入的数据。


```
// TODO: move to cmdenv?
// Encode binary data to be passed as multibase string in URL arguments.
// (avoiding issues described in https://github.com/ipfs/kubo/issues/7939)
func urlArgsEncoder(req *cmds.Request, env cmds.Environment) error {
	encoder, _ := mbase.EncoderByName("base64url")
	for n, arg := range req.Arguments {
		req.Arguments[n] = encoder.Encode([]byte(arg))
	}
	return nil
}

// Decode binary data passed as multibase string in URL arguments.
// (avoiding issues described in https://github.com/ipfs/kubo/issues/7939)
func urlArgsDecoder(req *cmds.Request, env cmds.Environment) error {
	for n, arg := range req.Arguments {
		encoding, data, err := mbase.Decode(arg)
		if err != nil {
			return errors.Wrap(err, "URL arg must be multibase encoded")
		}

		// Enforce URL-safe encoding is used for data passed via URL arguments
		// - without this we get data corruption similar to https://github.com/ipfs/kubo/issues/7939
		// - we can't just deny base64, because there may be other bases that
		//   are not URL-safe – better to force base64url which is known to be
		//   safe in URL context
		if encoding != mbase.Base64url {
			return errors.New("URL arg must be base64url encoded")
		}

		req.Arguments[n] = string(data)
	}
	return nil
}

```

# `/opt/kubo/core/commands/refs.go`

这段代码定义了一个名为 "commands" 的包，其中定义了一些与命令行相关的常量和函数。

具体来说，这个包定义了以下常量：

- `strings.Join()` 函数用于将多个字符串连接成一个字符串。
- `fmt.Printf()` 函数用于格式化输出。
- `context.Context()` 函数用于获取当前操作上下文。

此外，这个包还定义了一些函数：

- `io.ReadCloser()` 函数用于关闭读取操作。
- `io.ReadOnceCloser()` 函数用于读取一次并关闭读取操作。
- `strings.ToLower()` 函数用于将字符串转换为小写。
- `strings.ToUpper()` 函数用于将字符串转换为大写。
- `strings.Trim()` 函数用于删除字符串中的前缀和后缀。
- `fmt.Print()` 函数用于输出格式化的字符串。
- `fmt.Printf()` 函数用于格式化输出字符串。
- `fmt.Println()` 函数用于输出到标准输出流并格式化输出字符串。

最后，这个包还定义了一个名为 "ipfs-cmds" 的函数指针类型，它是从 "ipfs/go-ipfs-cmds" 包中导入的。


```
package commands

import (
	"context"
	"errors"
	"fmt"
	"io"
	"strings"

	cmdenv "github.com/ipfs/kubo/core/commands/cmdenv"
	"github.com/ipfs/kubo/core/commands/cmdutils"

	iface "github.com/ipfs/boxo/coreiface"
	merkledag "github.com/ipfs/boxo/ipld/merkledag"
	cid "github.com/ipfs/go-cid"
	cidenc "github.com/ipfs/go-cidutil/cidenc"
	cmds "github.com/ipfs/go-ipfs-cmds"
	ipld "github.com/ipfs/go-ipld-format"
)

```

这段代码定义了一个名为 `refsEncoderMap` 的结构体，它代表了一个命令的编码器映射。

结构体中的 `cmds.Text` 字段是一个函数指针，它使用了 `cmds.MakeTypedEncoder` 函数来创建一个编码器。

如果 `out` 变量出现错误，该函数将返回一个 `fmt.Errorf` 类型的错误。如果所有都没有错误，它将打印 `out` 变量的引用。

`refsEncoderMap` 还定义了一个名为 `KeyList` 的结构体，它代表了一个输出列表键的通用类型。

最后，该代码没有做其他事情，所以它不会输出任何值。


```
var refsEncoderMap = cmds.EncoderMap{
	cmds.Text: cmds.MakeTypedEncoder(func(req *cmds.Request, w io.Writer, out *RefWrapper) error {
		if out.Err != "" {
			return fmt.Errorf(out.Err)
		}
		fmt.Fprintln(w, out.Ref)

		return nil
	}),
}

// KeyList is a general type for outputting lists of keys
type KeyList struct {
	Keys []cid.Cid
}

```

此代码定义了一个名为 "format", "edges", "unique" 和 "recursive" 的选项，它们都是 "ipfs refs" 命令选项。

具体来说，这些选项可以用于指定在 IPFS 或 IPNS 对象中列出链接的格式。例如，"format" 选项可以用于以 "base58" 格式列出链接，"edges" 选项可以用于列出对象的边缘，"unique" 选项可以用于列出对象中唯一的链接，"recursive" 选项可以用于列出对象的嵌套深度。

另外，最后一个选项 "max-depth" 可以用于指定在 "ipfs refs" 命令中列出链接的最大深度，即对象中链接不能再嵌套多少层。


```
const (
	refsFormatOptionName    = "format"
	refsEdgesOptionName     = "edges"
	refsUniqueOptionName    = "unique"
	refsRecursiveOptionName = "recursive"
	refsMaxDepthOptionName  = "max-depth"
)

// RefsCmd is the `ipfs refs` command
var RefsCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List links (references) from an object.",
		ShortDescription: `
Lists the hashes of all the links an IPFS or IPNS object(s) contains,
with the following format:

  <link base58 hash>

```

This is a Go language program that implements a Cross-Refence Manager (CRM) using the CentOS Linux operating system.

It creates a Cross-Refence Manager (CRM) using the CentOS Linux operating system. The program takes a required input file, a reference graph, and a set of reference parameters. The reference graph is represented as an object graph where each node represents a unique resource, and each edge represents a reference from one unique resource to another.

The program uses the `go-glsql` package to connect to the database, and the `github.com/jmoiron/color-query` package to display the encoded references in a more readable format.

The program outputs a JSON file with the encoded references.

The program can handle a maximum depth of 10, and supports both direct and indirect references.

The program uses a recursive function to traverse the object graph, and outputs the encoded references for all unique resources that are at a depth of 1 or greater.


```
List all references recursively by using the flag '-r'.

NOTE: Like most other commands, Kubo will try to fetch the blocks of the passed path if they can't be found in the local store if it is running in online mode.
`,
	},
	Subcommands: map[string]*cmds.Command{
		"local": RefsLocalCmd,
	},
	Arguments: []cmds.Argument{
		cmds.StringArg("ipfs-path", true, true, "Path to the object(s) to list refs from.").EnableStdin(),
	},
	Options: []cmds.Option{
		cmds.StringOption(refsFormatOptionName, "Emit edges with given format. Available tokens: <src> <dst> <linkname>.").WithDefault("<dst>"),
		cmds.BoolOption(refsEdgesOptionName, "e", "Emit edge format: `<from> -> <to>`."),
		cmds.BoolOption(refsUniqueOptionName, "u", "Omit duplicate refs from output."),
		cmds.BoolOption(refsRecursiveOptionName, "r", "Recursively list links of child nodes."),
		cmds.IntOption(refsMaxDepthOptionName, "Only for recursive refs, limits fetch and listing to the given depth").WithDefault(-1),
	},
	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		err := req.ParseBodyArgs()
		if err != nil {
			return err
		}

		ctx := req.Context
		api, err := cmdenv.GetApi(env, req)
		if err != nil {
			return err
		}

		enc, err := cmdenv.GetCidEncoder(req)
		if err != nil {
			return err
		}

		unique, _ := req.Options[refsUniqueOptionName].(bool)
		recursive, _ := req.Options[refsRecursiveOptionName].(bool)
		maxDepth, _ := req.Options[refsMaxDepthOptionName].(int)
		edges, _ := req.Options[refsEdgesOptionName].(bool)
		format, _ := req.Options[refsFormatOptionName].(string)

		if !recursive {
			maxDepth = 1 // write only direct refs
		}

		if edges {
			if format != "<dst>" {
				return errors.New("using format argument with edges is not allowed")
			}

			format = "<src> -> <dst>"
		}

		// TODO: use session for resolving as well.
		objs, err := objectsForPaths(ctx, api, req.Arguments)
		if err != nil {
			return err
		}

		rw := RefWriter{
			res:      res,
			DAG:      merkledag.NewSession(ctx, api.Dag()),
			Ctx:      ctx,
			Unique:   unique,
			PrintFmt: format,
			MaxDepth: maxDepth,
		}

		for _, o := range objs {
			if _, err := rw.WriteRefs(o, enc); err != nil {
				if err := res.Emit(&RefWrapper{Err: err.Error()}); err != nil {
					return err
				}
			}
		}

		return nil
	},
	Encoders: refsEncoderMap,
	Type:     RefWrapper{},
}

```

该代码定义了一个名为 "RefsLocalCmd" 的命令对象，用于列出所有本地对象的哈希值。该命令通过调用 "n.Blockstore.AllKeysChan" 函数获取当前节点的一个或多个哈希值，并使用 "res.Emitter" 输出结果。

该命令会遍历所有哈希值，并将每个哈希值转换为一个 "RefWrapper" 对象，这个对象包含该哈希值的引用和相关的信息，例如该哈希值所在的块和该哈希值的数据类型。

通过使用 "refsEncoderMap" 和其他函数，该命令可以正确处理不同本地对象的哈希值，并在遍历哈希值时输出结果。


```
var RefsLocalCmd = &cmds.Command{
	Helptext: cmds.HelpText{
		Tagline: "List all local references.",
		ShortDescription: `
Displays the hashes of all local objects. NOTE: This treats all local objects as "raw blocks" and returns CIDv1-Raw CIDs.
`,
	},

	Run: func(req *cmds.Request, res cmds.ResponseEmitter, env cmds.Environment) error {
		ctx := req.Context
		n, err := cmdenv.GetNode(env)
		if err != nil {
			return err
		}

		// todo: make async
		allKeys, err := n.Blockstore.AllKeysChan(ctx)
		if err != nil {
			return err
		}

		for k := range allKeys {
			err := res.Emit(&RefWrapper{Ref: k.String()})
			if err != nil {
				return err
			}
		}

		return nil
	},
	Encoders: refsEncoderMap,
	Type:     RefWrapper{},
}

```

该函数的作用是获取给定路径列表中的所有根节点，并将它们返回。函数接收一个上下文上下文、一个表示要获取的接口API和一个字符串数组，然后使用cmdutils.PathOrCidPath函数将每个路径的根节点获取出来，并将获取到的根节点存储在一个数组中。接着使用n.ResolvePath函数，将每个根节点作为路径的根节点，递归地获取其它的根节点，并将它们的根节点ID存储在一个哈希表中。最后，函数返回根节点ID数组和错误，如果在获取过程中出现错误，则返回nil。


```
func objectsForPaths(ctx context.Context, n iface.CoreAPI, paths []string) ([]cid.Cid, error) {
	roots := make([]cid.Cid, len(paths))
	for i, sp := range paths {
		p, err := cmdutils.PathOrCidPath(sp)
		if err != nil {
			return nil, err
		}
		o, _, err := n.ResolvePath(ctx, p)
		if err != nil {
			return nil, err
		}
		roots[i] = o.RootCid()
	}
	return roots, nil
}

```

这段代码定义了一个名为 RefWrapper 的结构体，它包含一个名为 Ref 的字符串字段和一个名为 Err 的字符串字段。

接着定义了一个名为 RefWriter 的结构体，它包含一个名为 res 的 cmds.ResponseEmitter 字段、一个名为 DAG 的 ipld.NodeGetter 字段、一个名为 ctx 的 context.Context 字段和一个名为 Unique 的布尔字段。

然后定义了一个名为 PrintFmt 的字符串字段，用于在输出中使用格式化字符串。

接着定义了一个名为 seen 的 map[string]int 字段，用于存储已经被访问过的标识符。

最后，在 RefWrapper 和 RefWriter 的构造函数中，初始化了上述定义的字符串字段和标识符，并将 RefWrapper 的 ref 和 Err 字段设置为空，将 RefWriter 的 res 和 DAG 字段设置为 res.Response 和 ipld.NodeGetter.Node，将 RefWriter 的 Unique 字段设置为 false，将 PrintFmt 字段设置为 ""。


```
type RefWrapper struct {
	Ref string
	Err string
}

type RefWriter struct {
	res cmds.ResponseEmitter
	DAG ipld.NodeGetter
	Ctx context.Context

	Unique   bool
	MaxDepth int
	PrintFmt string

	seen map[string]int
}

```

This is a function that is part of a拓扑排序算法的预处理 step。它的作用是处理每个节点的局部拓扑结构，即将每个节点需要打印的边连接起来。它通过两种方式获取每个节点：一是使用 depth 计算方法获取，二是通过 rw.writeEdge 函数获取。

首先，对于每个节点，它需要判断自己是否需要打印。如果该节点是新的或者还没有被打印过，那么它需要被打印。如果节点已经打印过了，并且它没有未访问过的子节点，那么也不需要打印。如果节点需要被打印，那么 rw.writeEdge 函数会被调用，以将该节点和所有从它的邻居节点到达的边都写入。

在获取每个节点时，函数使用了 depth 计算方法。该方法会遍历所有与节点 i 相关的边，并计算出节点 i 的深度。如果深度为 0，则说明节点 i 还没有被打印过，因此需要继续递归执行。如果深度已经达到 maxDeep，则停止执行，因为在该深度下节点集合中已经有无限个节点了。

另外，函数还使用了两种不同的链深度的计算方式。一种是基于 depth 计算，另一种是基于 Go 函数 depth 计算。基于 Go 函数的计算方式可能会在某些情况下导致错误的深度，因此在实际使用中，更应当使用基于 depth 计算的方式。

总体来说，该函数的主要作用是处理每个节点的局部拓扑结构，并确保节点集合中每个节点都被正确地打印出来。


```
// WriteRefs writes refs of the given object to the underlying writer.
func (rw *RefWriter) WriteRefs(c cid.Cid, enc cidenc.Encoder) (int, error) {
	n, err := rw.DAG.Get(rw.Ctx, c)
	if err != nil {
		return 0, err
	}
	return rw.writeRefsRecursive(n, 0, enc)
}

func (rw *RefWriter) writeRefsRecursive(n ipld.Node, depth int, enc cidenc.Encoder) (int, error) {
	nc := n.Cid()

	var count int
	for i, ng := range ipld.GetDAG(rw.Ctx, rw.DAG, n) {
		lc := n.Links()[i].Cid
		goDeeper, shouldWrite := rw.visit(lc, depth+1) // The children are at depth+1

		// Avoid "Get()" on the node and continue with next Link.
		// We can do this if:
		// - We printed it before (thus it was already seen and
		//   fetched with Get()
		// - AND we must not go deeper.
		// This is an optimization for pruned branches which have been
		// visited before.
		if !shouldWrite && !goDeeper {
			continue
		}

		// We must Get() the node because:
		// - it is new (never written)
		// - OR we need to go deeper.
		// This ensures printed refs are always fetched.
		nd, err := ng.Get(rw.Ctx)
		if err != nil {
			return count, err
		}

		// Write this node if not done before (or !Unique)
		if shouldWrite {
			if err := rw.WriteEdge(nc, lc, n.Links()[i].Name, enc); err != nil {
				return count, err
			}
			count++
		}

		// Keep going deeper. This happens:
		// - On unexplored branches
		// - On branches not explored deep enough
		// Note when !Unique, branches are always considered
		// unexplored and only depth limits apply.
		if goDeeper {
			c, err := rw.writeRefsRecursive(nd, depth+1, enc)
			count += c
			if err != nil {
				return count, err
			}
		}
	}

	return count, nil
}

```

This function appears to determine whether a depth recorder (`rw`) can retrieve a CPL subgraph from a root node (`c`) with the specified depth limit (`--maxDepth`). It does this by first checking if the depth limit is already reached. If it is, the function returns `false` and `false`, as it is not possible to have a depth limit that is greater than or equal to the maximum depth. If the depth limit is not reached, the function checks if the input graph is already unique. If it is not, the function starts tracking the CPL subgraph by traversing the graph and counting the number of seen CPL instances. If the depth limit is already reached or the graph is already unique, the function returns `false` and `false`, as it is not possible to retrieve a CPL subgraph from a root node with a depth that is greater than or equal to the maximum depth. If the depth limit is not reached and the graph is not already unique, the function returns `true` and `true`, as it is possible to retrieve a CPL subgraph from a root node with the specified depth limit.


```
// visit returns two values:
// - the first boolean is true if we should keep traversing the DAG
// - the second boolean is true if we should print the CID
//
// visit will do branch pruning depending on rw.MaxDepth, previously visited
// cids and whether rw.Unique is set. i.e. rw.Unique = false and
// rw.MaxDepth = -1 disables any pruning. But setting rw.Unique to true will
// prune already visited branches at the cost of keeping as set of visited
// CIDs in memory.
func (rw *RefWriter) visit(c cid.Cid, depth int) (bool, bool) {
	atMaxDepth := rw.MaxDepth >= 0 && depth == rw.MaxDepth
	overMaxDepth := rw.MaxDepth >= 0 && depth > rw.MaxDepth

	// Shortcut when we are over max depth. In practice, this
	// only applies when calling refs with --maxDepth=0, as root's
	// children are already over max depth. Otherwise nothing should
	// hit this.
	if overMaxDepth {
		return false, false
	}

	// We can shortcut right away if we don't need unique output:
	//   - we keep traversing when not atMaxDepth
	//   - always print
	if !rw.Unique {
		return !atMaxDepth, true
	}

	// Unique == true from this point.
	// Thus, we keep track of seen Cids, and their depth.
	if rw.seen == nil {
		rw.seen = make(map[string]int)
	}
	key := string(c.Bytes())
	oldDepth, ok := rw.seen[key]

	// Unique == true && depth < MaxDepth (or unlimited) from this point

	// Branch pruning cases:
	// - We saw the Cid before and either:
	//   - Depth is unlimited (MaxDepth = -1)
	//   - We saw it higher (smaller depth) in the DAG (means we must have
	//     explored deep enough before)
	// Because we saw the CID, we don't print it again.
	if ok && (rw.MaxDepth < 0 || oldDepth <= depth) {
		return false, false
	}

	// Final case, we must keep exploring the DAG from this CID
	// (unless we hit the depth limit).
	// We note down its depth because it was either not seen
	// or is lower than last time.
	// We print if it was not seen.
	rw.seen[key] = depth
	return !atMaxDepth, !ok
}

```

这段代码定义了一个名为 `WriteEdge` 的函数，属于名为 `RefWriter` 的接口。

该函数接受四个参数：

- `from`：要输出对像的 CID。
- `to`：的目标 CID。
- `linkname`：链接名称，用于在输出中注明来源和目标。
- `enc`：用于编码 CID 的实现了。

函数的作用是：

- 如果 `rw` 参数所关联的上下文还活着（即，`rw.Ctx` 还没有完成），则执行以下操作：
  select`
  - 如果 `rw.PrintFmt` 字段不为空，则执行以下操作：
    - 读取 `rw.PrintFmt` 中定义好的模板字符串，并将其 `<src>`、`<dst>` 和 `<linkname>` 部分替换为对应的 CID。
    - 如果 `rw.PrintFmt` 中 `<src>`、`<dst>` 和 `<linkname>` 部分模板字符串中的字符串是空字符串，则将其补充为 `linkname`。
    - 返回 `rw.res.Emit(&RefWrapper{Ref: "<linkname>"})` 返回结果。
    - 否则，直接返回 `rw.Ctx.Err()`，因为 `rw.Ctx` 已经 `Done()`。

- 如果 `rw.Ctx` 已经 `Done()`，则执行以下操作：
   select`
   - 返回 `rw.Ctx.Err()`，因为 `rw.Ctx` 已经 `Done()`。


该函数主要实现了 `WriteOneEdge` 接口，支持输出一个边（` edge` 参数）。


```
// Write one edge
func (rw *RefWriter) WriteEdge(from, to cid.Cid, linkname string, enc cidenc.Encoder) error {
	if rw.Ctx != nil {
		select {
		case <-rw.Ctx.Done(): // just in case.
			return rw.Ctx.Err()
		default:
		}
	}

	var s string
	switch {
	case rw.PrintFmt != "":
		s = rw.PrintFmt
		s = strings.Replace(s, "<src>", enc.Encode(from), -1)
		s = strings.Replace(s, "<dst>", enc.Encode(to), -1)
		s = strings.Replace(s, "<linkname>", linkname, -1)
	default:
		s += enc.Encode(to)
	}

	return rw.res.Emit(&RefWrapper{Ref: s})
}

```