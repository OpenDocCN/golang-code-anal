# go-ipfs 源码解析 31

# `core/node/libp2p/rcmgr_defaults.go`

这段代码是一个 Go 语言库 libp2p，它是 Go-libp2p 项目的核心库，提供了用于管理 IPFS（InterPlanetary File System）网络中的文件和点对点连接等功能。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为 "libp2p" 的包。
2. 导入了 libp2p 包的一些类型，如 "fmt" 类型用于输出信息。
3. 从 ipfs-kubo 包中导入 "config" 包，用于创建一个 IPFS 配置实例。
4. 从 ipfs-kubo 包中导入 "node" 包，用于在节点之间建立 P2P 连接。
5. 从 ipfs-kubo 包中导入 "resource-manager" 包，用于管理 IPFS 网络中的资源。
6. 从 memory 包中导入 "infiniteResourceLimits"，用于设置 IPFS 网络中的资源请求和允许。


```go
package libp2p

import (
	"fmt"

	"github.com/dustin/go-humanize"
	"github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core/node/libp2p/fd"
	"github.com/libp2p/go-libp2p"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	"github.com/pbnjay/memory"
)

var infiniteResourceLimits = rcmgr.InfiniteLimits.ToPartialLimitConfig().System

```

Autoscaling with Alibaba Cloud Dynos is a feature that allows the scaling limits for容器 services to be automatically adjusted based on the available resources, such as the number of connections and the amount of traffic. This allows for more efficient resource usage and can help prevent service outages.

To disable autoscaling for these services, you can set the `Autoscaling` setting for those services to `false`. This will override any auto scaling limits set by Alibaba Cloud Dynos.

Additionally, if you want to manually configure the scaling limits for these services, you can set the `Autoscaling` setting for those services to `true` and then set the `scalingLimitConfig` and `connmgr` settings as needed. This will override any auto scaling limits set by Alibaba Cloud Dynos, but will allow you to manually configure the scaling limits for these services.

You can find more information about setting the Autoscaling setting and the `scalingLimitConfig` and `connmgr` settings in the Alibaba Cloud Dynos documentation.


```go
// This file defines implicit limit defaults used when Swarm.ResourceMgr.Enabled

// createDefaultLimitConfig creates LimitConfig to pass to libp2p's resource manager.
// The defaults follow the documentation in docs/libp2p-resource-management.md.
// Any changes in the logic here should be reflected there.
func createDefaultLimitConfig(cfg config.SwarmConfig) (limitConfig rcmgr.ConcreteLimitConfig, logMessageForStartup string, err error) {
	maxMemoryDefaultString := humanize.Bytes(uint64(memory.TotalMemory()) / 2)
	maxMemoryString := cfg.ResourceMgr.MaxMemory.WithDefault(maxMemoryDefaultString)
	maxMemory, err := humanize.ParseBytes(maxMemoryString)
	if err != nil {
		return rcmgr.ConcreteLimitConfig{}, "", err
	}

	maxMemoryMB := maxMemory / (1024 * 1024)
	maxFD := int(cfg.ResourceMgr.MaxFileDescriptors.WithDefault(int64(fd.GetNumFDs()) / 2))

	// At least as of 2023-01-25, it's possible to open a connection that
	// doesn't ask for any memory usage with the libp2p Resource Manager/Accountant
	// (see https://github.com/libp2p/go-libp2p/issues/2010#issuecomment-1404280736).
	// As a result, we can't currently rely on Memory limits to full protect us.
	// Until https://github.com/libp2p/go-libp2p/issues/2010 is addressed,
	// we take a proxy now of restricting to 1 inbound connection per MB.
	// Note: this is more generous than go-libp2p's default autoscaled limits which do
	// 64 connections per 1GB
	// (see https://github.com/libp2p/go-libp2p/blob/master/p2p/host/resource-manager/limit_defaults.go#L357 ).
	systemConnsInbound := int(1 * maxMemoryMB)

	partialLimits := rcmgr.PartialLimitConfig{
		System: rcmgr.ResourceLimits{
			Memory: rcmgr.LimitVal64(maxMemory),
			FD:     rcmgr.LimitVal(maxFD),

			Conns:         rcmgr.Unlimited,
			ConnsInbound:  rcmgr.LimitVal(systemConnsInbound),
			ConnsOutbound: rcmgr.Unlimited,

			Streams:         rcmgr.Unlimited,
			StreamsOutbound: rcmgr.Unlimited,
			StreamsInbound:  rcmgr.Unlimited,
		},

		// Transient connections won't cause any memory to be accounted for by the resource manager/accountant.
		// Only established connections do.
		// As a result, we can't rely on System.Memory to protect us from a bunch of transient connection being opened.
		// We limit the same values as the System scope, but only allow the Transient scope to take 25% of what is allowed for the System scope.
		Transient: rcmgr.ResourceLimits{
			Memory: rcmgr.LimitVal64(maxMemory / 4),
			FD:     rcmgr.LimitVal(maxFD / 4),

			Conns:         rcmgr.Unlimited,
			ConnsInbound:  rcmgr.LimitVal(systemConnsInbound / 4),
			ConnsOutbound: rcmgr.Unlimited,

			Streams:         rcmgr.Unlimited,
			StreamsInbound:  rcmgr.Unlimited,
			StreamsOutbound: rcmgr.Unlimited,
		},

		// Lets get out of the way of the allow list functionality.
		// If someone specified "Swarm.ResourceMgr.Allowlist" we should let it go through.
		AllowlistedSystem: infiniteResourceLimits,

		AllowlistedTransient: infiniteResourceLimits,

		// Keep it simple by not having Service, ServicePeer, Protocol, ProtocolPeer, Conn, or Stream limits.
		ServiceDefault: infiniteResourceLimits,

		ServicePeerDefault: infiniteResourceLimits,

		ProtocolDefault: infiniteResourceLimits,

		ProtocolPeerDefault: infiniteResourceLimits,

		Conn: infiniteResourceLimits,

		Stream: infiniteResourceLimits,

		// Limit the resources consumed by a peer.
		// This doesn't protect us against intentional DoS attacks since an attacker can easily spin up multiple peers.
		// We specify this limit against unintentional DoS attacks (e.g., a peer has a bug and is sending too much traffic intentionally).
		// In that case we want to keep that peer's resource consumption contained.
		// To keep this simple, we only constrain inbound connections and streams.
		PeerDefault: rcmgr.ResourceLimits{
			Memory:          rcmgr.Unlimited64,
			FD:              rcmgr.Unlimited,
			Conns:           rcmgr.Unlimited,
			ConnsInbound:    rcmgr.DefaultLimit,
			ConnsOutbound:   rcmgr.Unlimited,
			Streams:         rcmgr.Unlimited,
			StreamsInbound:  rcmgr.DefaultLimit,
			StreamsOutbound: rcmgr.Unlimited,
		},
	}

	scalingLimitConfig := rcmgr.DefaultLimits
	libp2p.SetDefaultServiceLimits(&scalingLimitConfig)

	// Anything set above in partialLimits that had a value of rcmgr.DefaultLimit will be overridden.
	// Anything in scalingLimitConfig that wasn't defined in partialLimits above will be added (e.g., libp2p's default service limits).
	partialLimits = partialLimits.Build(scalingLimitConfig.Scale(int64(maxMemory), maxFD)).ToPartialLimitConfig()

	// Simple checks to override autoscaling ensuring limits make sense versus the connmgr values.
	// There are ways to break this, but this should catch most problems already.
	// We might improve this in the future.
	// See: https://github.com/ipfs/kubo/issues/9545
	if partialLimits.System.ConnsInbound > rcmgr.DefaultLimit && cfg.ConnMgr.Type.WithDefault(config.DefaultConnMgrType) != "none" {
		maxInboundConns := int64(partialLimits.System.ConnsInbound)
		if connmgrHighWaterTimesTwo := cfg.ConnMgr.HighWater.WithDefault(config.DefaultConnMgrHighWater) * 2; maxInboundConns < connmgrHighWaterTimesTwo {
			maxInboundConns = connmgrHighWaterTimesTwo
		}

		if maxInboundConns < config.DefaultResourceMgrMinInboundConns {
			maxInboundConns = config.DefaultResourceMgrMinInboundConns
		}

		// Scale System.StreamsInbound as well, but use the existing ratio of StreamsInbound to ConnsInbound
		if partialLimits.System.StreamsInbound > rcmgr.DefaultLimit {
			partialLimits.System.StreamsInbound = rcmgr.LimitVal(maxInboundConns * int64(partialLimits.System.StreamsInbound) / int64(partialLimits.System.ConnsInbound))
		}
		partialLimits.System.ConnsInbound = rcmgr.LimitVal(maxInboundConns)
	}

	msg := fmt.Sprintf(`
```

这段代码定义了一个名为`Computed default go-libp2p Resource Manager`的函数，它返回一个名为` partialLimits`的函数，该函数使用了`rcmgr.ConcreteLimitConfig`和`ipfs swarm resources`。

`Computed default go-libp2p Resource Manager`的作用是设置基于`Swarm.ResourceMgr.MaxMemory`和`Swarm.ResourceMgr.MaxFileDescriptors`设置的资源限制。这些限制可以被 inspection using 'ipfs swarm resources' as shown in the code snippet.

`partialLimits`函数的作用是创建一个空限制配置类，然后设置基于`rcmgr.ConcreteLimitConfig`设置的内存限制和文件描述符限制，最后它将创建好的限制配置类返回。这个函数的作用是确保在给定的内存和文件描述符限制内，返回一个符合条件的`Computed default go-libp2p Resource Manager`实例。


```go
Computed default go-libp2p Resource Manager limits based on:
    - 'Swarm.ResourceMgr.MaxMemory': %q
    - 'Swarm.ResourceMgr.MaxFileDescriptors': %d

These can be inspected with 'ipfs swarm resources'.

`, maxMemoryString, maxFD)

	// We already have a complete value thus pass in an empty ConcreteLimitConfig.
	return partialLimits.Build(rcmgr.ConcreteLimitConfig{}), msg, nil
}

```

# `core/node/libp2p/rcmgr_logging.go`

这段代码定义了一个名为 libp2p 的包，该包实现了 libp2p 协议。通过导入不同的依赖项，实现了以下功能：

1. 定义了一个名为 Clock 的时钟类型，用于计时libp2p中的时间戳。
2. 定义了一个名为 errors 的错误类型，用于包含从libp2p中可能获得的错误信息。
3. 定义了一个名为 sync 的同步类型，用于 libp2p中的时钟同步。
4. 定义了一个名为 time 的时间类型，用于 libp2p中的时间操作。
5. 导入了一个名为 network 的网络类型，用于与远程服务器建立连接并执行操作。
6. 导入了一个名为 peer 的 peer 类型，用于与本地网络中的对等连接器进行通信。
7. 导入了一个名为 protocol 的 protocol 类型，用于 libp2p中的各种协议。
8. 导入了一个名为 rcmgr 的 resource-manager 类型，用于管理资源。
9. 导入了一个名为 multiformats 的 multi-format 类型，用于支持不同的数据格式。
10. 导入了一个名为 zap 的 zap 类型，用于记录错误信息。


```go
package libp2p

import (
	"context"
	"errors"
	"sync"
	"time"

	"github.com/benbjohnson/clock"
	"github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/protocol"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	ma "github.com/multiformats/go-multiaddr"
	"go.uber.org/zap"
)

```

该代码定义了一个名为 "loggingResourceManager" 的结构体，该结构体包含一个名为 "clock" 的时钟实例和一个名为 "logger" 的输出 "zap.SugaredLogger" 类型和一个名为 "delegate" 的指向 "network.ResourceManager" 的指针，还有一个名为 "logInterval" 的定时间隔，以及一个名为 "mut" 的 sync.Mutex 和一个名为 "limitExceededErrs" 的 map。

该结构体表示一个资源管理器，用于记录与资源相关的日志信息。该资源管理器使用一个输出日志 "zap.SugaredLogger" 将日志信息发送到 "logger" 输出，同时也允许 "delegate" 进行资源管理。此外，该资源管理器定期将超过限制的错误计数到 "countErrs" 函数 map 中，并使用 "mut" sync.Mutex 来保护计数器。


```go
type loggingResourceManager struct {
	clock       clock.Clock
	logger      *zap.SugaredLogger
	delegate    network.ResourceManager
	logInterval time.Duration

	mut               sync.Mutex
	limitExceededErrs map[string]int
}

type loggingScope struct {
	logger    *zap.SugaredLogger
	delegate  network.ResourceScope
	countErrs func(error)
}

```

该代码定义了一个名为`loggingResourceManager`的`NetworkResourceManager`类型的变量，该变量实现了`ResourceManager`和`ResourceManagerState`接口。

`start`函数作为`ResourceManager`的子函数，接收一个`Context`并返回。内部使用了`clock.Ticker`方法来定期轮询资源的使用情况，并在使用完毕后停止。

`for`循环用于定期检查是否有新的限制信息。如果是，则调用`n.logger.Warnf`函数来输出一条警告信息，并记录到`errs` map中。如果所有错误都存在，则输出更详细的提示信息并使用`n.logger.Warnf`函数来输出。最后，使用`n.mut.Unlock`函数来解锁`mut`锁，以便在循环结束后能够继续添加新的错误。


```go
var (
	_ network.ResourceManager    = (*loggingResourceManager)(nil)
	_ rcmgr.ResourceManagerState = (*loggingResourceManager)(nil)
)

func (n *loggingResourceManager) start(ctx context.Context) {
	logInterval := n.logInterval
	if logInterval == 0 {
		logInterval = 10 * time.Second
	}
	ticker := n.clock.Ticker(logInterval)
	go func() {
		defer ticker.Stop()
		for {
			select {
			case <-ticker.C:
				n.mut.Lock()
				errs := n.limitExceededErrs
				n.limitExceededErrs = make(map[string]int)

				for e, count := range errs {
					n.logger.Warnf("Protected from exceeding resource limits %d times.  libp2p message: %q.", count, e)
				}

				if len(errs) != 0 {
					n.logger.Warnf("Learn more about potential actions to take at: https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md")
				}

				n.mut.Unlock()
			case <-ctx.Done():
				return
			}
		}
	}()
}

```

此函数的作用是统计错误并记录到日志资源管理器中。它接受一个整数类型的参数n和一个错误类型的参数err。

首先，如果err是网络.ErrResourceLimitExceeded，函数将尝试锁定n.mut，以确保统计到的错误类型与网络资源限界相关。如果n.limitExceededErrs是空集合，则创建一个包含键类型为string的map，以存储达到的限界。

接下来，函数将尝试从错误类型err中解析出错误信息，并获取限界范围和所达限界类型。如果解析出错误信息，将计数器添加到n.limitExceededErrs中。

最后，函数释放n.mut的锁，如果n.limitExceededErrs有键，则执行n.limitExceededErrs的计数器加1操作。


```go
func (n *loggingResourceManager) countErrs(err error) {
	if errors.Is(err, network.ErrResourceLimitExceeded) {
		n.mut.Lock()
		if n.limitExceededErrs == nil {
			n.limitExceededErrs = make(map[string]int)
		}

		// we need to unwrap the error to get the limit scope and the kind of reached limit
		eout := errors.Unwrap(err)
		if eout != nil {
			n.limitExceededErrs[eout.Error()]++
		}

		n.mut.Unlock()
	}
}

```

这是一个 Go 语言中的函数指针类型，它定义了一个名为 "func" 的函数，它可以接受一个可以接受一个名为 "n" 的整数类型的参数，并返回一个可以接受一个名为 "f" 的函数类型参数的错误。

这个函数指针是一个指向一个接受 "n" 和 "f" 两个参数的函数的指针。通过指针访问的 n.delegate.ViewSystem(f) 和 n.delegate.ViewTransient(f) 分别返回一个代表 "n" 和 "f" 的函数类型，并返回一个代表 "ViewSystem"、"ViewTransient" 和 "ViewService" 函数的错误类型。

具体来说，当调用 func(n *loggingResourceManager) ViewSystem(f func(network.ResourceScope) error) error 时，会创建一个名为 "n" 的整数类型的变量，并将 f 函数赋值给 n.delegate.ViewSystem。调用 ViewSystem(f) 时，将传入的 f 函数作为第一个参数传递，并返回一个代表 "ViewSystem" 函数的错误类型。

类似地，当调用 func(n *loggingResourceManager) ViewTransient(f func(network.ResourceScope) error) error 时，会创建一个名为 "n" 的整数类型的变量，并将 f 函数赋值给 n.delegate.ViewTransient。调用 ViewTransient(f) 时，将传入的 f 函数作为第一个参数传递，并将 s 作为第二个参数传递，并将 f 函数的返回值作为第三个参数传递。返回一个代表 "ViewTransient" 函数的错误类型。

当调用 func(n *loggingResourceManager) ViewService(svc string, f func(network.ServiceScope) error) error 时，会创建一个名为 "n" 的整数类型的变量，并将 f 函数赋值给 n.delegate.ViewService。调用 ViewService(f) 时，将传入的 f 函数作为第一个参数传递，并将 svc 作为第二个参数传递。调用 ViewService(f) 时，将传入的 f 函数的返回值作为第三个参数传递。返回一个代表 "ViewService" 函数的错误类型。


```go
func (n *loggingResourceManager) ViewSystem(f func(network.ResourceScope) error) error {
	return n.delegate.ViewSystem(f)
}

func (n *loggingResourceManager) ViewTransient(f func(network.ResourceScope) error) error {
	return n.delegate.ViewTransient(func(s network.ResourceScope) error {
		return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
	})
}

func (n *loggingResourceManager) ViewService(svc string, f func(network.ServiceScope) error) error {
	return n.delegate.ViewService(svc, func(s network.ServiceScope) error {
		return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
	})
}

```

这是一个 Go 语言中的函数指针，它定义了三个名为 "func" 的函数，它们都接受一个名为 "n" 的参数，表示一个名为 "loggingResourceManager" 的实例。

第一个函数 "ViewProtocol" 接受一个名为 "p" 的协议 ID 和一个名为 "f" 的函数作为参数。它返回一个错误 "ViewProtocol" 类型，这个函数的实现会在 "ViewProtocol" 函数内部进行。

第二个函数 "ViewPeer" 同样接受一个名为 "p" 的 peers ID 和一个名为 "f" 的函数作为参数。它返回一个错误 "ViewPeer" 类型，这个函数的实现会在 "ViewPeer" 函数内部进行。

第三个函数 "OpenConnection" 接受一个名为 "dir" 的网络方向和一个名为 "usefd" 的布尔参数，以及一个名为 "remote" 的名为 "ma.Multiaddr" 的参数。它返回一个名为 "connMgmtScope" 的网络连接管理范围和一个名为 "err" 的错误。这个函数的实现会在 "OpenConnection" 函数内部进行。

函数内部使用了一个名为 "delegate" 的字段，它是一个匿名类型，表示一个代理对象，这个对象在 "ViewProtocol" 和 "ViewPeer" 函数中被调用，用于执行实际的网络协议访问操作。函数还使用了一个名为 "logger" 的字段，表示一个日志定制的 "Logger" 对象，这个对象在所有函数内部都被用来记录协议或 peer 执行请求时的信息。最后，函数还使用了一个名为 "countErrs" 的字段，表示一个计数器，这个计数器在所有函数内部都被用来记录发生的错误数量。


```go
func (n *loggingResourceManager) ViewProtocol(p protocol.ID, f func(network.ProtocolScope) error) error {
	return n.delegate.ViewProtocol(p, func(s network.ProtocolScope) error {
		return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
	})
}

func (n *loggingResourceManager) ViewPeer(p peer.ID, f func(network.PeerScope) error) error {
	return n.delegate.ViewPeer(p, func(s network.PeerScope) error {
		return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
	})
}

func (n *loggingResourceManager) OpenConnection(dir network.Direction, usefd bool, remote ma.Multiaddr) (network.ConnManagementScope, error) {
	connMgmtScope, err := n.delegate.OpenConnection(dir, usefd, remote)
	n.countErrs(err)
	return connMgmtScope, err
}

```

这段代码定义了三个函数，分别作用于一个名为 `loggingResourceManager` 的类型，下面是详细的解释：

1. `func (n *loggingResourceManager) OpenStream(p peer.ID, dir network.Direction) (network.StreamManagementScope, error)`：
这个函数接收两个参数，一个是 `peer.ID`，另一个是 `network.Direction`，然后返回一个名为 `network.StreamManagementScope` 的类型和一个名为 `error` 的类型的变量。

这个函数的作用是：执行 `n.delegate` 的函数调用，并且传递 `peer.ID` 和 `dir` 参数，然后返回一个名为 `network.StreamManagementScope` 的类型和一个名为 `error` 的类型的变量。

2. `func (n *loggingResourceManager) Close() error`：
这个函数接收一个参数，返回一个名为 `error` 的类型的变量。

这个函数的作用是：执行 `n.delegate` 的函数调用，然后返回一个名为 `error` 的类型的变量。

3. `func (n *loggingResourceManager) ListServices() []string`：
这个函数接收一个参数，返回一个名为 `[]string` 的类型的数组。

这个函数的作用是：执行 `n.delegate` 的函数调用，然后返回一个名为 `[]string` 的类型的数组。


```go
func (n *loggingResourceManager) OpenStream(p peer.ID, dir network.Direction) (network.StreamManagementScope, error) {
	connMgmtScope, err := n.delegate.OpenStream(p, dir)
	n.countErrs(err)
	return connMgmtScope, err
}

func (n *loggingResourceManager) Close() error {
	return n.delegate.Close()
}

func (n *loggingResourceManager) ListServices() []string {
	rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
	if !ok {
		return nil
	}

	return rapi.ListServices()
}

```

这两位作者共同实现了两个函数，`ListProtocols()` 和 `ListPeers()`。这两个函数接收一个 `loggingResourceManager` 类型的参数 `n`。

这两位作者确保 `n` 存在，然后获取其 `delegate` 字段，如果 `delegate` 字段不存在，则返回 `nil`。接着，这两位作者调用 `ListProtocols()` 函数，并将其返回结果赋给变量 `rapi`。最后，这两位作者再次调用 `ListProtocols()` 函数，并将其返回结果赋给变量 `rapi`，将 `ok` 的值设置为 `true`。

这两位作者还确保 `rapi` 存在，然后调用 `ListPeers()` 函数，并将其返回结果赋给变量 `rapi`。最后，这两位作者再次调用 `ListPeers()` 函数，并将其返回结果赋给变量 `rapi`，将 `ok` 的值设置为 `true`。


```go
func (n *loggingResourceManager) ListProtocols() []protocol.ID {
	rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
	if !ok {
		return nil
	}

	return rapi.ListProtocols()
}

func (n *loggingResourceManager) ListPeers() []peer.ID {
	rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
	if !ok {
		return nil
	}

	return rapi.ListPeers()
}

```

这段代码定义了两个函数，一个是`func`，一个是`s`.这两个函数的主要作用是确保在函数调用时，所有的错误都正确地传递给调用者。

`func`函数接收一个`loggingResourceManager`类型的参数`n`，并返回一个`rcmgr.ResourceManagerStat`类型的`rcmgr.ResourceManagerStat`作为结果。函数的作用是将`n`代理的`rcmgr.ResourceManagerState`类型调用`rcmgr.ResourceManagerStat`类型的方法，并返回该方法的结果。如果`n`代理的`rcmgr.ResourceManagerState`类型调用`rcmgr.ResourceManagerStat`类型的方法时，传递的参数`rcmgr.ResourceManagerStat{}`，那么该函数将直接返回`rcmgr.ResourceManagerStat{}`。

`s`函数接收一个`loggingScope`类型的参数`s`，并返回一个`error`类型的`ReserveMemory`函数作为结果。函数的作用是执行`s.delegate.ReserveMemory`函数，并返回一个`error`类型的结果。如果`s.delegate.ReserveMemory`函数在尝试分配内存时遇到任何错误，那么函数将返回该错误。


```go
func (n *loggingResourceManager) Stat() rcmgr.ResourceManagerStat {
	rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
	if !ok {
		return rcmgr.ResourceManagerStat{}
	}

	return rapi.Stat()
}

func (s *loggingScope) ReserveMemory(size int, prio uint8) error {
	err := s.delegate.ReserveMemory(size, prio)
	s.countErrs(err)
	return err
}

```

此代码定义了一个名为 "loggingScope" 的名为 "s" 的指针变量，该变量代表一个名为 "NetworkScope" 的类型。

此代码定义了三个函数，名为 "ReleaseMemory"，"Stat" 和 "BeginSpan"。

"ReleaseMemory" 函数接收一个名为 "size" 的整数参数，然后调用 "delegate" 变量所定义的 "ReleaseMemory" 函数，该函数释放指定大小的内存。

"Stat" 函数返回一个名为 "network.ScopeStat" 的名为 "s" 的指针变量所代表 "NetworkScope" 类型的统计信息，该函数使用 "delegate" 变量所定义的 "Stat" 函数来获取统计信息。

"BeginSpan" 函数接收一个名为 "s" 的指针变量，返回一个名为 "network.ResourceScopeSpan" 和一个名为 "error" 的名为 "done" 的错误类型的 "BeginSpan" 函数返回值，该函数使用 "delegate" 变量所定义的 "BeginSpan" 函数来开始或结束一个 span，并提供 "Done" 函数，该函数使用 "BeginSpan" 函数返回的 "done" 错误类型的变量来关闭 span。

"Done" 函数接收一个名为 "done" 错误类型的参数，然后执行与该参数相关的 "BeginSpan" 函数 "Done" 定义的 "Done" 函数，使用传入的 "done" 参数来关闭 span。


```go
func (s *loggingScope) ReleaseMemory(size int) {
	s.delegate.ReleaseMemory(size)
}

func (s *loggingScope) Stat() network.ScopeStat {
	return s.delegate.Stat()
}

func (s *loggingScope) BeginSpan() (network.ResourceScopeSpan, error) {
	return s.delegate.BeginSpan()
}

func (s *loggingScope) Done() {
	s.delegate.(network.ResourceScopeSpan).Done()
}

```

此代码定义了一个名为 `loggingScope` 的类型，它是一个名为 `Scope` 的接口的实现。这个接口定义了 `PeerScope` 和 `ProtocolScope` 两个类型，分别表示对 `Peer` 和 `Protocol` 属性的访问。

`loggingScope` 类型包含四个方法：`Name()`、`Protocol()`、`Peer()` 和 `PeerScope()`。以下是这些方法的实现：

1. `Name()` 方法返回一个字符串，表示 `loggingScope` 的名称。
2. `Protocol()` 方法返回一个 `ID` 类型的 `Protocol` 属性的值，表示 `loggingScope` 所代表的网络协议 scope 的 `Protocol()` 方法返回的协议 ID。
3. `Peer()` 方法返回一个 `peer.ID` 类型的 `Peer` 属性的值，表示 `loggingScope` 所代表的网络协议 scope 的 `Peer()` 方法返回的对等者。
4. `PeerScope()` 方法返回一个 `network.PeerScope` 类型的 `PeerScope` 属性的值，表示 `loggingScope` 所代表的网络协议 scope 的 `PeerScope()` 方法返回的对等者。

这个实现的作用是，通过 `delegate` 和一系列方法，使得 `loggingScope` 类型可以被用来访问其 `ProtocolScope` 和 `PeerScope` 类型的属性和方法，从而实现对网络协议 scope 的操作。


```go
func (s *loggingScope) Name() string {
	return s.delegate.(network.ServiceScope).Name()
}

func (s *loggingScope) Protocol() protocol.ID {
	return s.delegate.(network.ProtocolScope).Protocol()
}

func (s *loggingScope) Peer() peer.ID {
	return s.delegate.(network.PeerScope).Peer()
}

func (s *loggingScope) PeerScope() network.PeerScope {
	return s.delegate.(network.PeerScope)
}

```

此代码定义了一个名为 `loggingScope` 的类型，该类型代表了一个网络日志扫描场景。

函数 `SetPeer` 接收一个 `peer.ID` 类型的参数 `p`，表示要连接的远程服务器。它将调用 `delegate` 指针所代表的方法 `SetPeer`，该方法使用 `network.ConnManagementScope` 来设置对远程服务器的连接，并返回一个错误。

函数 `ProtocolScope` 返回一个名为 `network.ProtocolScope` 的类型，表示对远程服务器套接字层的协议。它将调用 `delegate` 指针所代表的方法 `ProtocolScope`，该方法返回一个 `network.ProtocolScope` 类型的值。

函数 `SetProtocol` 接收一个名为 `protocol.ID` 类型的参数 `proto`，表示要设置的对远程服务器套接字层的协议ID。它将调用 `delegate` 指针所代表的方法 `SetProtocol`，该方法使用 `network.StreamManagementScope` 来设置对远程服务器套接字层的协议，并返回一个错误。

函数 `SetPeer` 和 `SetProtocol` 都使用了 `delegate` 指针，该指针保存了一个网络 `Logger` 实例，可以用来记录函数调用过程中的错误信息。 `delegate` 函数会在函数调用时执行相应的 `SetPeer` 和 `SetProtocol` 方法，并将结果返回。


```go
func (s *loggingScope) SetPeer(p peer.ID) error {
	err := s.delegate.(network.ConnManagementScope).SetPeer(p)
	s.countErrs(err)
	return err
}

func (s *loggingScope) ProtocolScope() network.ProtocolScope {
	return s.delegate.(network.ProtocolScope)
}

func (s *loggingScope) SetProtocol(proto protocol.ID) error {
	err := s.delegate.(network.StreamManagementScope).SetProtocol(proto)
	s.countErrs(err)
	return err
}

```

这段代码定义了一个名为`loggingScope`的`*`类型变量，它包含一个指向`loggingScope`类型的指针变量`s`。

通过`s.delegate.(network.ServiceScope)`将`loggingScope`的指针转换为`network.ServiceScope`类型，然后返回该类型的`ServiceScope`对象。

通过`s.SetService(srv string)`函数，可以将指定的服务名称`srv`设置为`loggingScope`的`ServiceScope`对象。注意，这个函数有一个`Error`类型的参数，如果设置失败，函数返回该参数作为错误信息。

通过`s.countErrs(err)`函数，在设置服务名称失败时，记录错误信息并返回。

通过`s.limit()`函数，可以将指定的资源限制器`rcmgr.ResourceScopeLimiter`设置为指定的`loggingScope`的`Limit`对象。

通过`s.SetLimit(limit rcmgr.Limit)`函数，可以将指定的资源限制器`rcmgr.ResourceScopeLimiter`设置为指定的`loggingScope`的`Limit`对象。


```go
func (s *loggingScope) ServiceScope() network.ServiceScope {
	return s.delegate.(network.ServiceScope)
}

func (s *loggingScope) SetService(srv string) error {
	err := s.delegate.(network.StreamManagementScope).SetService(srv)
	s.countErrs(err)
	return err
}

func (s *loggingScope) Limit() rcmgr.Limit {
	return s.delegate.(rcmgr.ResourceScopeLimiter).Limit()
}

func (s *loggingScope) SetLimit(limit rcmgr.Limit) {
	s.delegate.(rcmgr.ResourceScopeLimiter).SetLimit(limit)
}

```

# `core/node/libp2p/rcmgr_logging_test.go`

这段代码定义了一个名为"libp2p"的包，其中包含了以下几个主要组件：

1. 一个名为"clock"的时钟上下文，它提供了一个全局的定时器，可以在代码中使用。
2. 一个名为"testing"的测试功能，提供了用于测试的函数和测试case。
3. 一个名为"time"的time包，提供了一些与时间相关的函数和常量。
4. 一个名为"resource-manager"的"host"包，可能是用于管理节点资源的组件。
5. 一个名为"multiformats"的"go-multiaddr"包，可能用于在网络中传输多地址。
6. 一个名为"stretchr"的"go-zap"包，可能用于代码的代码片段。
7. 一个名为"zap"的zap库。
8. 一个名为"zaptest"的zap测试框架的测试插件。
9. 一个名为"observer"的zap测试框架的观察者插件。

此代码的作用是定义了一个名为"libp2p"的包，其中包含了一些用于编写单元测试和时钟驱动程序的组件。具体来说，它定义了一些函数和常量，用于定义测试函数和观察者模式，以及定义了一些用于与时间相关的函数。此外，它还定义了一些用于管理节点资源的组件。


```go
package libp2p

import (
	"context"
	"testing"
	"time"

	"github.com/benbjohnson/clock"
	"github.com/libp2p/go-libp2p/core/network"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"
	ma "github.com/multiformats/go-multiaddr"
	"github.com/stretchr/testify/require"
	"go.uber.org/zap"
	"go.uber.org/zap/zaptest/observer"
)

```

This is a Go program that performs a network test to determine if it can reserve the specified number of inbound connections while an application runs. The program uses the Go-zled令牌桶限制库和一个自定义的日志观察者来收集限制条件下的日志。

程序首先创建了一个资源管理器(rm)和一个日志观察者(oLogger)。然后，它创建了一个网络观察者(lrm)和一个日志资源管理器(logRM)，用于管理观察者。

接下来，程序创建了一个名为"core"的观察者和一个名为"logs"的观察者。然后，程序使用这两个观察者开始等待限制条件下的日志。如果观察者接收到限制违反的日志，程序将跳转到计时器并继续等待。

程序还创建了一个名为"protected"的观察者，用于在资源限制超过限制数量时记录日志。最后，程序通过循环3次来模拟超过限制数量的情况。在循环内，程序使用观察者打开一个TCP连接并将其重置为无限。

如果程序在循环内两次接收到违反限制的日志，则它将停止并输出"Protected from exceeding resource limits 2 times.  libp2p message: \"system: cannot reserve inbound connection: resource limit exceeded\""。


```go
func TestLoggingResourceManager(t *testing.T) {
	clock := clock.NewMock()
	orig := rcmgr.DefaultLimits.AutoScale()
	limits := orig.ToPartialLimitConfig()
	limits.System.Conns = 1
	limits.System.ConnsInbound = 1
	limits.System.ConnsOutbound = 1
	limiter := rcmgr.NewFixedLimiter(limits.Build(orig))
	rm, err := rcmgr.NewResourceManager(limiter)
	if err != nil {
		t.Fatal(err)
	}

	oCore, oLogs := observer.New(zap.WarnLevel)
	oLogger := zap.New(oCore)
	lrm := &loggingResourceManager{
		clock:       clock,
		logger:      oLogger.Sugar(),
		delegate:    rm,
		logInterval: 1 * time.Second,
	}

	// 2 of these should result in resource limit exceeded errors and subsequent log messages
	for i := 0; i < 3; i++ {
		_, _ = lrm.OpenConnection(network.DirInbound, false, ma.StringCast("/ip4/127.0.0.1/tcp/1234"))
	}

	// run the logger which will write an entry for those errors
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	lrm.start(ctx)
	clock.Add(3 * time.Second)

	timer := time.NewTimer(1 * time.Second)
	for {
		select {
		case <-timer.C:
			t.Fatalf("expected logs never arrived")
		default:
			if oLogs.Len() == 0 {
				continue
			}
			require.Equal(t, "Protected from exceeding resource limits 2 times.  libp2p message: \"system: cannot reserve inbound connection: resource limit exceeded\".", oLogs.All()[0].Message)
			return
		}
	}
}

```

# `core/node/libp2p/rcmgr_metrics.go`

这段代码定义了一个名为 "libp2p" 的包，其中包含了一些通用的功能。

1. "errors" 导入了一个名为 "errors" 的错误类，这可能用于在代码中处理错误。
2. "strconv" 导入了一个名为 "strconv" 的函数，这可能用于进行字符串操作。
3. "github.com/libp2p/go-libp2p/core/network""导入了一个名为 "network" 的包，这可能用于管理网络连接。
4. "github.com/libp2p/go-libp2p/core/peer""导入了一个名为 "peer" 的包，这可能用于与对等体进行通信。
5. "github.com/libp2p/go-libp2p/core/protocol"导入了一个名为 "protocol" 的包，这可能用于在应用程序中使用协议。
6. "github.com/prometheus/client_golang/prometheus"导入了一个名为 "prometheus" 的包，这可能用于在应用程序中使用 Prometheus 客户端。
7. "都必须注册" 函数 "mustRegister" 接受一个名为 "c" 的参数，这可能是用于计数的上下文。
8. "errs" 字段 "errs" 是一个名为 "errors" 的错误类的实例。
9. "错误" 字段 "err" 将存储给 "errs" 字段。
10. "记账" 函数 "记账" 没有具体的实现，它可能是一个通用的记账函数。
11. "初始化" 函数 "初始化" 没有具体的实现，它可能是一个通用的初始化函数。
12. "关闭" 函数 "关闭" 没有具体的实现，它可能是一个通用的关闭函数。
13. "日志" 函数 "日志" 没有具体的实现，它可能是一个通用的日志函数。
14. "设置超时" 函数 "设置超时" 没有具体的实现，它可能是一个通用的设置超时函数。


```go
package libp2p

import (
	"errors"
	"strconv"

	"github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/protocol"
	rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"

	"github.com/prometheus/client_golang/prometheus"
)

func mustRegister(c prometheus.Collector) {
	err := prometheus.Register(c)
	are := prometheus.AlreadyRegisteredError{}
	if errors.As(err, &are) {
		return
	}
	if err != nil {
		panic(err)
	}
}

```

This is a Go contract for Prometheus that provides metrics for a Libp2p RCMG (Remote controlled Memory Manager) system. The metrics are divided into different categories such asconn,conn\_blocked,stream,stream\_blocked,peer,peer\_blocked,protocol,protocol\_blocked,protocol\_peer\_blocked,service,service\_blocked,service\_peer\_blocked,memory,memory\_blocked. These metrics are useful to track the health and performance of the Libp2p RCMM system.


```go
func createRcmgrMetrics() rcmgr.MetricsReporter {
	const (
		direction = "direction"
		usesFD    = "usesFD"
		protocol  = "protocol"
		service   = "service"
	)

	connAllowed := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_conns_allowed_total",
			Help: "allowed connections",
		},
		[]string{direction, usesFD},
	)
	mustRegister(connAllowed)

	connBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_conns_blocked_total",
			Help: "blocked connections",
		},
		[]string{direction, usesFD},
	)
	mustRegister(connBlocked)

	streamAllowed := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_streams_allowed_total",
			Help: "allowed streams",
		},
		[]string{direction},
	)
	mustRegister(streamAllowed)

	streamBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_streams_blocked_total",
			Help: "blocked streams",
		},
		[]string{direction},
	)
	mustRegister(streamBlocked)

	peerAllowed := prometheus.NewCounter(prometheus.CounterOpts{
		Name: "libp2p_rcmgr_peers_allowed_total",
		Help: "allowed peers",
	})
	mustRegister(peerAllowed)

	peerBlocked := prometheus.NewCounter(prometheus.CounterOpts{
		Name: "libp2p_rcmgr_peer_blocked_total",
		Help: "blocked peers",
	})
	mustRegister(peerBlocked)

	protocolAllowed := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_protocols_allowed_total",
			Help: "allowed streams attached to a protocol",
		},
		[]string{protocol},
	)
	mustRegister(protocolAllowed)

	protocolBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_protocols_blocked_total",
			Help: "blocked streams attached to a protocol",
		},
		[]string{protocol},
	)
	mustRegister(protocolBlocked)

	protocolPeerBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_protocols_for_peer_blocked_total",
			Help: "blocked streams attached to a protocol for a specific peer",
		},
		[]string{protocol},
	)
	mustRegister(protocolPeerBlocked)

	serviceAllowed := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_services_allowed_total",
			Help: "allowed streams attached to a service",
		},
		[]string{service},
	)
	mustRegister(serviceAllowed)

	serviceBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_services_blocked_total",
			Help: "blocked streams attached to a service",
		},
		[]string{service},
	)
	mustRegister(serviceBlocked)

	servicePeerBlocked := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "libp2p_rcmgr_service_for_peer_blocked_total",
			Help: "blocked streams attached to a service for a specific peer",
		},
		[]string{service},
	)
	mustRegister(servicePeerBlocked)

	memoryAllowed := prometheus.NewCounter(prometheus.CounterOpts{
		Name: "libp2p_rcmgr_memory_allocations_allowed_total",
		Help: "allowed memory allocations",
	})
	mustRegister(memoryAllowed)

	memoryBlocked := prometheus.NewCounter(prometheus.CounterOpts{
		Name: "libp2p_rcmgr_memory_allocations_blocked_total",
		Help: "blocked memory allocations",
	})
	mustRegister(memoryBlocked)

	return rcmgrMetrics{
		connAllowed,
		connBlocked,
		streamAllowed,
		streamBlocked,
		peerAllowed,
		peerBlocked,
		protocolAllowed,
		protocolBlocked,
		protocolPeerBlocked,
		serviceAllowed,
		serviceBlocked,
		servicePeerBlocked,
		memoryAllowed,
		memoryBlocked,
	}
}

```

该代码定义了一个名为 `rcmgrMetrics` 的结构体，其中包含了一些与 P2P 资源管理器(rcmgr)相关的指标。

该 `rcmgrMetrics` 结构体定义了 8 个指标：

- `connAllowed`：连接允许计数
- `connBlocked`：连接阻塞计数
- `streamAllowed`：流允许计数
- `streamBlocked`：流阻塞计数
- `peerAllowed`：对等方允许计数
- `peerBlocked`：对等方阻塞计数
- `protocolAllowed`：协议允许计数
- `protocolBlocked`：协议禁止计数
- `protocolPeerBlocked`：协议对等方允许计数
- `serviceAllowed`：服务允许计数
- `serviceBlocked`：服务阻塞计数
- `servicePeerBlocked`：服务对等方允许计数
- `memoryAllowed`：内存允许计数
- `memoryBlocked`：内存阻塞计数

这些指标被存储在一个名为 `rcmgrMetrics` 的结构体中，该结构体定义了连接到对等方和服务的允许/阻塞计数，以及一些其他 P2P 资源管理器指标。

由于 `rcmgrMetrics` 是使用 `rcmgr` 包进行管理的，因此它依赖于 `rcmgr` 包的接口实现。如果 `rcmgr` 的实现未实现该接口，该代码将无法编译或运行。


```go
// Failsafe to ensure interface from go-libp2p-resource-manager is implemented
var _ rcmgr.MetricsReporter = rcmgrMetrics{}

type rcmgrMetrics struct {
	connAllowed         *prometheus.CounterVec
	connBlocked         *prometheus.CounterVec
	streamAllowed       *prometheus.CounterVec
	streamBlocked       *prometheus.CounterVec
	peerAllowed         prometheus.Counter
	peerBlocked         prometheus.Counter
	protocolAllowed     *prometheus.CounterVec
	protocolBlocked     *prometheus.CounterVec
	protocolPeerBlocked *prometheus.CounterVec
	serviceAllowed      *prometheus.CounterVec
	serviceBlocked      *prometheus.CounterVec
	servicePeerBlocked  *prometheus.CounterVec
	memoryAllowed       prometheus.Counter
	memoryBlocked       prometheus.Counter
}

```

此代码定义了一个名为getDirection的函数，它接收一个网络Direction类型的参数，并返回一个字符串表示与该方向相关的方向。如果传入的Direction类型无法匹配任何预定义的方向，函数将返回空字符串。

接下来是一个名为allowConn的函数，它接收一个网络Direction类型的参数，使用rmc Gather Metrics的connAllowed方法来记录允许的连接方向，并使用getDirection函数来根据传入的Direction类型返回一个方向字符串。

connAllowed方法使用WithLabelValues方法来设置计数器，该方法的第一个参数是一个Map，它将获取getDirection函数返回的方向字符串，第二个参数是布尔值，表示是否启用统计函数。每次调用该方法时，将更新计数器的值，并将方向字符串作为键的值返回。

allowConn函数还定义了一个名为func的匿名函数，该函数接受一个网络Direction类型的参数和一个布尔值，用于设置是否允许连接。函数将在函数内部执行getDirection和rmc Gather Metrics的connAllowed方法，并将结果返回。


```go
func getDirection(d network.Direction) string {
	switch d {
	default:
		return ""
	case network.DirInbound:
		return "inbound"
	case network.DirOutbound:
		return "outbound"
	}
}

func (r rcmgrMetrics) AllowConn(dir network.Direction, usefd bool) {
	r.connAllowed.WithLabelValues(getDirection(dir), strconv.FormatBool(usefd)).Inc()
}

```

这段代码定义了四个函数，分别是BlockConn、AllowStream、BlockStream和AllowPeer。这些函数都与网络连通性相关。

具体来说，BlockConn函数用于记录当前网络连接是否被阻塞，并使用getDirection函数获取网络连通性方向（如INetworking.Upward或INetworking.Downward）。如果使用usefd参数为true，那么函数还会在 connBlocked 键上添加一个 label 值，表示使用 fd 打开的端口。

AllowStream函数用于记录当前网络连接是否允许接收数据流，并使用 getDirection 函数获取网络连通性方向。函数还会在 streamAllowed 键上添加一个 label 值，表示允许的流类型。

BlockStream函数用于记录当前网络连接是否允许发送数据流，并使用 getDirection 函数获取网络连通性方向。函数还会在 streamBlocked 键上添加一个 label 值，表示当前允许的流类型。

AllowPeer函数用于允许指定给定的PeerID的连接。函数会直接增加peerAllowed计数。


```go
func (r rcmgrMetrics) BlockConn(dir network.Direction, usefd bool) {
	r.connBlocked.WithLabelValues(getDirection(dir), strconv.FormatBool(usefd)).Inc()
}

func (r rcmgrMetrics) AllowStream(_ peer.ID, dir network.Direction) {
	r.streamAllowed.WithLabelValues(getDirection(dir)).Inc()
}

func (r rcmgrMetrics) BlockStream(_ peer.ID, dir network.Direction) {
	r.streamBlocked.WithLabelValues(getDirection(dir)).Inc()
}

func (r rcmgrMetrics) AllowPeer(_ peer.ID) {
	r.peerAllowed.Inc()
}

```

这段代码定义了一个名为func的函数，它接收两个参数：

1. r：代表一个Blocks metrics类型的变量，它存储了一个带有标签的值，每行都是一个peer.ID，表示这个值属于一个已经存在过的peer对象。
2. rcmgrMetrics：代表一个Blocks metrics类型的变量，它存储了一个带有标签的值，每行都是一个protocol.ID，表示这个值属于一个已经存在过的protocol对象。

函数可以执行以下操作：

1. 对r中的每个peer.ID添加一个Block标志，Inc()方法会增加该标志的计数。
2. 对rcmgrMetrics中的每个protocol.ID添加一个AllowProtocol标签，Inc()方法会增加该标签的计数。
3. 对rcmgrMetrics中的每个protocol.ID添加一个BlockProtocol标签，Inc()方法会增加该标签的计数。
4. 对rcmgrMetrics中的每个protocol.ID添加一个BlockProtocolPeer标签，Inc()方法会增加该标签的计数。

这段代码的具体作用是用于记录函数调用，对于每个函数调用，会创建一个Blocks metrics类型的变量来存储函数的执行结果，然后对于函数中的每个操作，增加该变量的相应标签的计数。


```go
func (r rcmgrMetrics) BlockPeer(_ peer.ID) {
	r.peerBlocked.Inc()
}

func (r rcmgrMetrics) AllowProtocol(proto protocol.ID) {
	r.protocolAllowed.WithLabelValues(string(proto)).Inc()
}

func (r rcmgrMetrics) BlockProtocol(proto protocol.ID) {
	r.protocolBlocked.WithLabelValues(string(proto)).Inc()
}

func (r rcmgrMetrics) BlockProtocolPeer(proto protocol.ID, _ peer.ID) {
	r.protocolPeerBlocked.WithLabelValues(string(proto)).Inc()
}

```

这段代码定义了四个函数，它们都是针对`rcmgrMetrics`类型的`r`对象进行的操作。具体来说，这些函数的主要作用是记录一些与服务相关的指标，例如服务的允许情况、服务的阻止情况、服务之间的阻止情况以及内存使用情况。

具体来说，这段代码如下：


func (r rcmgrMetrics) AllowService(svc string) {
	r.serviceAllowed.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) BlockService(svc string) {
	r.serviceBlocked.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) BlockServicePeer(svc string, _ peer.ID) {
	r.servicePeerBlocked.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) AllowMemory(_ int) {
	r.memoryAllowed.Inc()
}


其中，`rcmgrMetrics`是一个`rcmgrMetrics`类型的变量，它可能是从某些地方获取来的，也可能是自己计算出来的。在这段代码中，我们只是简单地将这些函数定义为`r`对象的方法。

函数的具体实现包括：

1. `AllowService`函数：允许指定的服务。它的具体实现是，在`rcmgrMetrics`类型的`r`对象中，将`serviceAllowed`字段所包含的键值对中的键值对，添加一个新的键值对，并设置键的标签值为`svc`。

2. `BlockService`函数：阻止指定的服务。它的具体实现与`AllowService`函数相反。

3. `BlockServicePeer`函数：阻止指定服务对之间的服务。它的具体实现与`BlockService`函数相同，但需要传递一个`_ peer.ID`类型的参数。

4. `AllowMemory`函数：允许指定数量的内存。它的具体实现与`AllowService`函数不同，需要传递一个整数类型的参数。


```go
func (r rcmgrMetrics) AllowService(svc string) {
	r.serviceAllowed.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) BlockService(svc string) {
	r.serviceBlocked.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) BlockServicePeer(svc string, _ peer.ID) {
	r.servicePeerBlocked.WithLabelValues(svc).Inc()
}

func (r rcmgrMetrics) AllowMemory(_ int) {
	r.memoryAllowed.Inc()
}

```

这是一段C语言函数，名为"BlockMemory"。函数接收两个整数参数，一个是整型变量"r"，另一个是"rcmgrMetrics"指针类型。函数的作用是统计并记录函数中每个线程块(Block)的内存占用情况，并返回内存占用情况的变化数(Inc)。

函数的具体实现包括以下几个步骤：

1. 调用另一个函数"__Metrics_"使用 rcmgrMetrics 作为参数，传入一个整数 5。函数"__Metrics_"返回一个整型变量 r，表示线程块的最大内存占用量。

2. 创建一个名为 "memoryBlocked" 的计数器，并将其递增。

3. 输出统计结果，并返回内存占用情况的变化数。函数返回值为-1，表示函数操作失败。

该函数的作用是统计并记录每个线程块的内存占用情况，并输出线程块最大内存占用量。它可能被用于研究程序中的内存问题，并为程序优化提供参考。


```go
func (r rcmgrMetrics) BlockMemory(_ int) {
	r.memoryBlocked.Inc()
}

```

# `core/node/libp2p/relay.go`

这段代码定义了一个名为`RelayTransport`的函数，其作用是创建一个libp2p的relay传输，用于在localhost上搭建基于libp2p的传输系统。

具体来说，该函数实现了以下功能：

1. 判断`enableRelay`是否为`true`，如果是，则启用relay传输，否则禁用。
2. 创建一个`RelayTransport`实例，并将libp2p的`EnableRelay`选项添加到`opts`结构体中。
3. 创建一个`Libp2pOpts`实例，其中包含libp2p的`Transport`选项，并将`EnableRelay`选项设置为`true`。
4. 创建一个`relay`路由器实例，将其配置为`Autorelay`类型，用于在接收到数据时立即发送数据。
5. 将`relay`路由器实例的`Peer`选项设置为`host.DummyPeer`，用于创建一个接受数据但不会做任何处理的peer。
6. 返回`relay`路由器和`Autorelay`路由器的`Options`，以及一个错误。

该函数可以被用来创建一个libp2p的relay传输实例，从而实现数据的传输和接收。


```go
package libp2p

import (
	"context"

	"github.com/ipfs/kubo/config"
	"github.com/libp2p/go-libp2p"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/p2p/host/autorelay"
	"github.com/libp2p/go-libp2p/p2p/protocol/circuitv2/relay"
	"go.uber.org/fx"
)

func RelayTransport(enableRelay bool) func() (opts Libp2pOpts, err error) {
	return func() (opts Libp2pOpts, err error) {
		if enableRelay {
			opts.Opts = append(opts.Opts, libp2p.EnableRelay())
		} else {
			opts.Opts = append(opts.Opts, libp2p.DisableRelay())
		}
		return
	}
}

```

This appears to be a function definition for the `libp2p` package. It appears to be defining a method for enabling or disabling the default relay service in Go-libp2p, and the method returns an `Libp2pOpts` object and an error.

The `Libp2pOpts` object appears to be a collection of选项 for the relayed service, including values for enabling or disabling the service, as well as limits on the number of circuits, the buffer size, and other settings.

The function also appears to be defining a default set of values for the relayed service, and any user configuration options specified by the user will be applied to this default set.


```go
func RelayService(enable bool, relayOpts config.RelayService) func() (opts Libp2pOpts, err error) {
	return func() (opts Libp2pOpts, err error) {
		if enable {
			def := relay.DefaultResources()
			// Real defaults live in go-libp2p.
			// Here we apply any overrides from user config.
			opts.Opts = append(opts.Opts, libp2p.EnableRelayService(relay.WithResources(relay.Resources{
				Limit: &relay.RelayLimit{
					Data:     relayOpts.ConnectionDataLimit.WithDefault(def.Limit.Data),
					Duration: relayOpts.ConnectionDurationLimit.WithDefault(def.Limit.Duration),
				},
				MaxCircuits:            int(relayOpts.MaxCircuits.WithDefault(int64(def.MaxCircuits))),
				BufferSize:             int(relayOpts.BufferSize.WithDefault(int64(def.BufferSize))),
				ReservationTTL:         relayOpts.ReservationTTL.WithDefault(def.ReservationTTL),
				MaxReservations:        int(relayOpts.MaxReservations.WithDefault(int64(def.MaxReservations))),
				MaxReservationsPerIP:   int(relayOpts.MaxReservationsPerIP.WithDefault(int64(def.MaxReservationsPerIP))),
				MaxReservationsPerPeer: int(relayOpts.MaxReservationsPerPeer.WithDefault(int64(def.MaxReservationsPerPeer))),
				MaxReservationsPerASN:  int(relayOpts.MaxReservationsPerASN.WithDefault(int64(def.MaxReservationsPerASN))),
			})))
		}
		return
	}
}

```

This is a Go function that initializes the peers of a local Libp2p node.

It creates a peer channel for receiving the address information of the peers, and an options struct for the Libp2p peers.

The function uses the provided `fx.Provide` method to initialize the options, and the `libp2p.EnableAutoRelayWithPeerSource` method to enable the autorelay feature with the peers.

It starts a background task that listens for new peers and adds their address information to the peer channel.

It also initializes an autoRelay feeder that sends a message to the peers when a new configuration is received, and an autoRelay timeout that controls the interval between the last autoRelay message and the current time.

It returns the options struct, the peerChannel, and the autoRelay feeder.


```go
func MaybeAutoRelay(staticRelays []string, cfgPeering config.Peering, enabled bool) fx.Option {
	if !enabled {
		return fx.Options()
	}

	if len(staticRelays) > 0 {
		return fx.Provide(func() (opts Libp2pOpts, err error) {
			if len(staticRelays) > 0 {
				static := make([]peer.AddrInfo, 0, len(staticRelays))
				for _, s := range staticRelays {
					var addr *peer.AddrInfo
					addr, err = peer.AddrInfoFromString(s)
					if err != nil {
						return
					}
					static = append(static, *addr)
				}
				opts.Opts = append(opts.Opts, libp2p.EnableAutoRelayWithStaticRelays(static))
			}
			return
		})
	}

	peerChan := make(chan peer.AddrInfo)
	return fx.Options(
		// Provide AutoRelay option
		fx.Provide(func() (opts Libp2pOpts, err error) {
			opts.Opts = append(opts.Opts,
				libp2p.EnableAutoRelayWithPeerSource(
					func(ctx context.Context, numPeers int) <-chan peer.AddrInfo {
						// TODO(9257): make this code smarter (have a state and actually try to grow the search outward) instead of a long running task just polling our K cluster.
						r := make(chan peer.AddrInfo)
						go func() {
							defer close(r)
							for ; numPeers != 0; numPeers-- {
								select {
								case v, ok := <-peerChan:
									if !ok {
										return
									}
									select {
									case r <- v:
									case <-ctx.Done():
										return
									}
								case <-ctx.Done():
									return
								}
							}
						}()
						return r
					},
					autorelay.WithMinInterval(0),
				))
			return
		}),
		autoRelayFeeder(cfgPeering, peerChan),
	)
}

```

这段代码定义了一个名为`HolePunching`的函数，其作用是 enabling 或disabling hole punching（悬空打孔）功能，具体实现如下：

1. 函数有两个参数，`config.Flag`是配置标记，`hasRelayClient`是一个布尔值，表示是否启用了代理客户端。

2. 函数内部有一个if语句，判断`config.Flag`是否为`true`，如果是，表示已经开启了悬空打孔功能。否则，函数内部会执行一些日志记录。

3. 如果`hasRelayClient`为`true`，那么函数会执行`if`语句，判断`config.Flag`是否为`true`。如果是，那么函数会执行一些日志记录。否则，函数不会执行任何操作，直接返回。

4. 如果`config.Flag`为`true`，但`hasRelayClient`为`false`，那么函数会执行以下操作：

	* 创建一个`Libp2pOpts`类型的变量`opts`。
	* 如果`config.Flag`为`true`，那么函数会执行`if`语句，判断`hasRelayClient`是否为`false`。
	* 如果`hasRelayClient`为`false`，那么函数会执行以下操作：
		- 将`opts`数组中的所有元素都设置为`libp2p.EnableHolePunching()`。
		- 返回`opts`和`0`。

这段代码的主要作用是实现了一个hole punching（悬空打孔）功能，可以设置代理客户端是否启用悬空打孔功能。如果设置了代理客户端并且启用了悬空打孔功能，那么当没有启用代理客户端时，函数会输出一条错误日志。如果设置了代理客户端并且禁用了悬空打孔功能，那么函数不会执行任何操作，直接返回。


```go
func HolePunching(flag config.Flag, hasRelayClient bool) func() (opts Libp2pOpts, err error) {
	return func() (opts Libp2pOpts, err error) {
		if flag.WithDefault(true) {
			if !hasRelayClient {
				// If hole punching is explicitly enabled but the relay client is disabled then panic,
				// otherwise just silently disable hole punching
				if flag != config.Default {
					log.Fatal("Failed to enable `Swarm.EnableHolePunching`, it requires `Swarm.RelayClient.Enabled` to be true.")
				} else {
					log.Info("HolePunching has been disabled due to the RelayClient being disabled.")
				}
				return
			}
			opts.Opts = append(opts.Opts, libp2p.EnableHolePunching())
		}
		return
	}
}

```

# `core/node/libp2p/routing.go`

该代码的作用是定义了一个名为 "libp2p" 的包，其中定义了一些定义、函数和变量，以及如何导入其他依赖库。

具体来说，这些函数和变量包括：

-定义了一个名为 "context" 的上下文类，用于在代码中处理错误和网络延迟。

-定义了一个名为 "fmt" 的函数，用于将 "offroute" 中的路由配置转换为字符串格式。

-定义了一个名为 "runtime.debug" 的函数，用于捕获调试信息并输出到控制台。

-定义了一个名为 "sort" 的函数，用于对 "ds" 中的链表进行排序。

-定义了一个名为 "time" 的函数，用于获取当前时间的秒数。

-定义了一个名为 "github.com/cenkalti/backoff/v4" 的导入，用于从 "offroute" 包中导入 "offline" 函数。

-定义了一个名为 "github.com/ipfs/boxo/routing/offline" 的导入，用于从 "offline" 包中导入 "offline" 函数。

-定义了一个名为 "github.com/ipfs/go-datastore" 的导入，用于从 "ds" 包中导入 "ds" 函数。

-定义了一个名为 "github.com/ipfs/go-libp2p-kad-dht" 的导入，用于从 "dht" 包中导入 "dht" 函数。

-定义了一个名为 "github.com/ipfs/go-libp2p-kad-dht/dual" 的导入，用于从 "dht" 包中导入 "dual" 函数。

-定义了一个名为 "github.com/ipfs/go-libp2p-kad-dht/fullrt" 的导入，用于从 "dht" 包中导入 "fullrt" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p-kad-dht/pubsub" 的导入，用于从 "pubsub" 包中导入 "pubsub" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p-pubsub-router" 的导入，用于从 "routinghelpers" 包中导入 "routinghelpers" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p-pubsub-router/pubsubrouter" 的导入，用于从 "pubsub" 包中导入 "pubsubrouter" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p-pubsub/record" 的导入，用于从 "record" 包中导入 "record" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p/core/host" 的导入，用于从 "host" 包中导入 "host" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p/core/peer" 的导入，用于从 "peer" 包中导入 "peer" 函数。

-定义了一个名为 "github.com/libp2p/go-libp2p/core/routing" 的导入，用于从 "routinghelpers"


```go
package libp2p

import (
	"context"
	"fmt"
	"runtime/debug"
	"sort"
	"time"

	"github.com/cenkalti/backoff/v4"
	offroute "github.com/ipfs/boxo/routing/offline"
	ds "github.com/ipfs/go-datastore"
	dht "github.com/libp2p/go-libp2p-kad-dht"
	ddht "github.com/libp2p/go-libp2p-kad-dht/dual"
	"github.com/libp2p/go-libp2p-kad-dht/fullrt"
	pubsub "github.com/libp2p/go-libp2p-pubsub"
	namesys "github.com/libp2p/go-libp2p-pubsub-router"
	record "github.com/libp2p/go-libp2p-record"
	routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"
	"github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/core/peer"
	"github.com/libp2p/go-libp2p/core/routing"
	"go.uber.org/fx"

	config "github.com/ipfs/kubo/config"
	"github.com/ipfs/kubo/core/node/helpers"
	"github.com/ipfs/kubo/repo"
	irouting "github.com/ipfs/kubo/routing"
)

```

这是一段 Go 语言中的代码，定义了一个名为 `Router` 的结构体，它实现了依赖关系映射。同时，还定义了一个名为 `p2pRouterOut` 的结构体，用于将路由信息导出到名为 "routers" 的 group 中。

`Router` 结构体包含了路由的相关信息，包括路由类型（ routes、type Router struct ）和路由的优先级（ Priority、int ）。

`p2pRouterOut` 结构体用于将 `Router` 实例导出到 "routers" group 中，以便于其他依赖关系图的消费。

`processInitialRoutingIn` 是 `InitialRoutingIn` 处理器的初始化操作，包含了一些设置 DHT 客户端的实验性配置，包括设置实验性客户端的 IP 地址、用户名和密码。


```go
type Router struct {
	routing.Routing

	Priority int // less = more important
}

type p2pRouterOut struct {
	fx.Out

	Router Router `group:"routers"`
}

type processInitialRoutingIn struct {
	fx.In

	Router routing.Routing `name:"initialrouting"`

	// For setting up experimental DHT client
	Host      host.Host
	Repo      repo.Repo
	Validator record.Validator
}

```

This appears to be a Go program that initializes aado(k) as a K-dimensional HTTP data router. It appears to handle initial configuration of the router, including setting up the default HTTP routers and the FullRT client, and setting the priority and bucket size of the router. It also appears to be using a parallel router to handle multiple search requests, and a hook to handle the HTTP router's OnStop method when it stops.

It looks like the program has a number of dependencies, including the dependency "github.com/spf13/packages-linux/repos/ent-dimension/webserver-release/v0.3.4" and the dependency "github.com/webrTC/webrTC".


```go
type processInitialRoutingOut struct {
	fx.Out

	Router        Router                 `group:"routers"`
	ContentRouter routing.ContentRouting `group:"content-routers"`

	DHT       *ddht.DHT
	DHTClient routing.Routing `name:"dhtc"`
}

type AddrInfoChan chan peer.AddrInfo

func BaseRouting(cfg *config.Config) interface{} {
	return func(lc fx.Lifecycle, in processInitialRoutingIn) (out processInitialRoutingOut, err error) {
		var dualDHT *ddht.DHT
		if dht, ok := in.Router.(*ddht.DHT); ok {
			dualDHT = dht

			lc.Append(fx.Hook{
				OnStop: func(ctx context.Context) error {
					return dualDHT.Close()
				},
			})
		}

		if cr, ok := in.Router.(routinghelpers.ComposableRouter); ok {
			for _, r := range cr.Routers() {
				if dht, ok := r.(*ddht.DHT); ok {
					dualDHT = dht
					lc.Append(fx.Hook{
						OnStop: func(ctx context.Context) error {
							return dualDHT.Close()
						},
					})
					break
				}
			}
		}

		if dualDHT != nil && cfg.Routing.AcceleratedDHTClient {
			cfg, err := in.Repo.Config()
			if err != nil {
				return out, err
			}
			bspeers, err := cfg.BootstrapPeers()
			if err != nil {
				return out, err
			}

			fullRTClient, err := fullrt.NewFullRT(in.Host,
				dht.DefaultPrefix,
				fullrt.DHTOption(
					dht.Validator(in.Validator),
					dht.Datastore(in.Repo.Datastore()),
					dht.BootstrapPeers(bspeers...),
					dht.BucketSize(20),
				),
			)
			if err != nil {
				return out, err
			}

			lc.Append(fx.Hook{
				OnStop: func(ctx context.Context) error {
					return fullRTClient.Close()
				},
			})

			// we want to also use the default HTTP routers, so wrap the FullRT client
			// in a parallel router that calls them in parallel
			httpRouters, err := constructDefaultHTTPRouters(cfg)
			if err != nil {
				return out, err
			}
			routers := []*routinghelpers.ParallelRouter{
				{Router: fullRTClient, DoNotWaitForSearchValue: true},
			}
			routers = append(routers, httpRouters...)
			router := routinghelpers.NewComposableParallel(routers)

			return processInitialRoutingOut{
				Router: Router{
					Priority: 1000,
					Routing:  router,
				},
				DHT:           dualDHT,
				DHTClient:     fullRTClient,
				ContentRouter: fullRTClient,
			}, nil
		}

		return processInitialRoutingOut{
			Router: Router{
				Priority: 1000,
				Routing:  in.Router,
			},
			DHT:           dualDHT,
			DHTClient:     dualDHT,
			ContentRouter: in.Router,
		}, nil
	}
}

```

此代码定义了一个名为 p2pOnlineContentRoutingIn 的结构体，它包含了内容路由器（ContentRouter）列表、路由前缀（pathPrefix）和路由委托（routing.ContentRouting）字段。

在 ContentRouting 函数中，首先定义了一个内部变量 routers，它用于存储所有内容路由器。然后，通过遍历 ContentRouter 字段，将每个路由器添加到 routers 列表中。

最后，将 routers 列表返回，作为名为 ContentRouting 的路由委托类型，用于在运行时计算内容路由器。


```go
type p2pOnlineContentRoutingIn struct {
	fx.In

	ContentRouter []routing.ContentRouting `group:"content-routers"`
}

// ContentRouting will get all routers that can do contentRouting and add them
// all together using a TieredRouter. It will be used for topic discovery.
func ContentRouting(in p2pOnlineContentRoutingIn) routing.ContentRouting {
	var routers []routing.Routing
	for _, cr := range in.ContentRouter {
		routers = append(routers,
			&routinghelpers.Compose{
				ContentRouting: cr,
			},
		)
	}

	return routinghelpers.Tiered{
		Routers: routers,
	}
}

```

这段代码定义了一个名为 `p2pOnlineRoutingIn` 的结构体，它包含一个名为 `Routers` 的 slice，以及一个名为 `Validator` 的 record。

`Routing` 函数接受一个 `p2pOnlineRoutingIn` 类型的参数，并返回一个名为 `irouting.ProvideManyRouter` 的函数类型。函数的具体实现如下：

1. 从 `Routers` slice 中获取所有路由器，并使用一个名为 `TieredRouter` 的队列排序它们。根据实际使用情况，可能还需要根据路由器的优先级对路由器进行排序。
2. 创建一个名为 `cRouters` 的 slice，用于存储所有 `ParallelRouter` 类型的实例。这些实例将忽略错误并不要等待搜索值，而是直接使用路由器实例。
3. 创建一个名为 `routingHelpers` 的命名空间，它包含一个名为 `NewComposableParallel` 的函数类型，用于创建一个可组合的并行路由器。最后，返回这个并行路由器。

`Routing` 函数的作用是帮助用户构建一个 P2P 网络中的路由表。通过使用 `TieredRouter` 来排序路由器，可以让系统在路由器数量较多时仍然能够支持高效的负载均衡。


```go
type p2pOnlineRoutingIn struct {
	fx.In

	Routers   []Router `group:"routers"`
	Validator record.Validator
}

// Routing will get all routers obtained from different methods
// (delegated routers, pub-sub, and so on) and add them all together
// using a TieredRouter.
func Routing(in p2pOnlineRoutingIn) irouting.ProvideManyRouter {
	routers := in.Routers

	sort.SliceStable(routers, func(i, j int) bool {
		return routers[i].Priority < routers[j].Priority
	})

	var cRouters []*routinghelpers.ParallelRouter
	for _, v := range routers {
		cRouters = append(cRouters, &routinghelpers.ParallelRouter{
			IgnoreError:             true,
			DoNotWaitForSearchValue: true,
			Router:                  v.Routing,
		})
	}

	return routinghelpers.NewComposableParallel(cRouters)
}

```

该代码定义了一个名为`OfflineRouting`的函数，它接收两个参数：一个`datastore`和一个`validator`。函数返回一个作为`p2pRouterOut`类型的参数，该类型的定义没有在这里明确说明，但可以推断出它需要一个`p2pRouter`和一个`validator`作为其内部状态。

`datastore`参数是一个`ds.Datastore`类型，它表示要存储在线下数据存储中的数据。

`validator`参数是一个`record.Validator`类型，它表示要验证的记录的`authorizer`。

函数返回的`p2pRouterOut`类型是一个实现了`p2pRouter`接口的`p2pRouter`和一个作为选项的`pubsub.PubSub`类型的字段，用于在路由器上接收和发送pubsub消息。

此外，该函数还定义了一个`p2pPSRoutingIn`类型，该类型包含一个`fx.In`类型的字段，它表示需要从路由器输入的pubsub消息。


```go
// OfflineRouting provides a special Router to the routers list when we are creating a offline node.
func OfflineRouting(dstore ds.Datastore, validator record.Validator) p2pRouterOut {
	return p2pRouterOut{
		Router: Router{
			Routing:  offroute.NewOfflineRouter(dstore, validator),
			Priority: 10000,
		},
	}
}

type p2pPSRoutingIn struct {
	fx.In

	Validator record.Validator
	Host      host.Host
	PubSub    *pubsub.PubSub `optional:"true"`
}

```

此代码定义了一个名为 `PubsubRouter` 的函数，接收三个参数：`mctx`、`lc` 和 `in`。函数返回一个名为 `p2pRouterOut` 的 `p2pRouter` 类型的结构体，以及一个指向 `namesys.PubsubValueStore` 的引用和一个指向错误的引用。

具体来说，函数内部执行以下操作：

1. 创建一个名为 `psRouter` 的 `namesys.PubsubValueStore` 类型的实例，并将其作为参数传递给 `namesys.NewPubsubValueStore` 函数。该函数使用 `lc` 和 `in.PubSub` 作为其输入参数，并设置psRouter作为ValueStore的值。此外，函数还设置了一个 rebroadcastInterval，用于指定发布者发布消息的间隔时间。

2. 如果以上操作成功，函数将返回一个 `p2pRouterOut` 类型的结构体，其中包含一个 `Router` 字段，该字段定义了路由器的行为。路由器的行为由一个包含两个字段（`ValueStore` 和 `Priority`）的 `Routing` 字段指定。`ValueStore` 字段是一个 `namesys.LimitedValueStore` 类型的实例，它用于存储路由器从 `psRouter` 获取的值。`Priority` 字段定义了路由器的优先级，值越小越优先。

3. 如果出现错误，函数将返回一个非 ` nil` 的错误对象，以便调用者能够处理。


```go
func PubsubRouter(mctx helpers.MetricsCtx, lc fx.Lifecycle, in p2pPSRoutingIn) (p2pRouterOut, *namesys.PubsubValueStore, error) {
	psRouter, err := namesys.NewPubsubValueStore(
		helpers.LifecycleCtx(mctx, lc),
		in.Host,
		in.PubSub,
		in.Validator,
		namesys.WithRebroadcastInterval(time.Minute),
	)
	if err != nil {
		return p2pRouterOut{}, nil, err
	}

	return p2pRouterOut{
		Router: Router{
			Routing: &routinghelpers.Compose{
				ValueStore: &routinghelpers.LimitedValueStore{
					ValueStore: psRouter,
					Namespaces: []string{"ipns"},
				},
			},
			Priority: 100,
		},
	}, psRouter, nil
}

```

This is a Go module that implements the initial configuration of a Go network. It contains the implementation for the peers



```go
func autoRelayFeeder(cfgPeering config.Peering, peerChan chan<- peer.AddrInfo) fx.Option {
	return fx.Invoke(func(lc fx.Lifecycle, h host.Host, dht *ddht.DHT) {
		ctx, cancel := context.WithCancel(context.Background())
		done := make(chan struct{})

		defer func() {
			if r := recover(); r != nil {
				fmt.Println("Recovering from unexpected error in AutoRelayFeeder:", r)
				debug.PrintStack()
			}
		}()
		go func() {
			defer close(done)

			// Feed peers more often right after the bootstrap, then backoff
			bo := backoff.NewExponentialBackOff()
			bo.InitialInterval = 15 * time.Second
			bo.Multiplier = 3
			bo.MaxInterval = 1 * time.Hour
			bo.MaxElapsedTime = 0 // never stop
			t := backoff.NewTicker(bo)
			defer t.Stop()
			for {
				select {
				case <-t.C:
				case <-ctx.Done():
					return
				}

				// Always feed trusted IDs (Peering.Peers in the config)
				for _, trustedPeer := range cfgPeering.Peers {
					if len(trustedPeer.Addrs) == 0 {
						continue
					}
					select {
					case peerChan <- trustedPeer:
					case <-ctx.Done():
						return
					}
				}

				// Additionally, feed closest peers discovered via DHT
				if dht == nil {
					/* noop due to missing dht.WAN. happens in some unit tests,
					   not worth fixing as we will refactor this after go-libp2p 0.20 */
					continue
				}
				closestPeers, err := dht.WAN.GetClosestPeers(ctx, h.ID().String())
				if err != nil {
					// no-op: usually 'failed to find any peer in table' during startup
					continue
				}
				for _, p := range closestPeers {
					addrs := h.Peerstore().Addrs(p)
					if len(addrs) == 0 {
						continue
					}
					dhtPeer := peer.AddrInfo{ID: p, Addrs: addrs}
					select {
					case peerChan <- dhtPeer:
					case <-ctx.Done():
						return
					}
				}
			}
		}()

		lc.Append(fx.Hook{
			OnStop: func(_ context.Context) error {
				cancel()
				<-done
				return nil
			},
		})
	})
}

```