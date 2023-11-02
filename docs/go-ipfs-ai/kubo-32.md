# go-ipfs 源码解析 32

# `/opt/kubo/core/node/libp2p/routingopt.go`

该代码是一个 Go 语言库中的 `libp2p` 包，其中包括了用于实现分布式哈希表 (DHT) 的各种组件。具体来说，该库提供了包括 DHT 路由、DHT 客户端、KV 存储、KV 路由、DHT 客户端、以及一些与 DHT 相关的工具函数。

下面是 `libp2p` 包中的一些主要函数和其作用：

- `DHTPeerService`：实现了一个 DHT 存储服务，可以用来创建和管理 DHT 节点，并提供了使用户通过 `DHTPeerService` 进行 DHT 存储的 API。
- `DHTNodeService`：实现了 DHT 存储服务的一个实例，可以用来创建和管理 DHT 节点，并提供了使用户通过 `DHTNodeService` 进行 DHT 存储的 API。
- `DHTKeyB镜像`：是一个 DHT 存储的键值对，它将一个键映射到一个值，并返回一个 `DHTKeyB` 类型的值，该值可以用来获取键对应的值。
- `DHTValueB镜像`：是一个 DHT 存储的键值对，它将一个键映射到一个值，并返回一个 `DHTValueB` 类型的值，该值可以用来获取键对应的值。
- `DHTDHTf`：是一个 DHT 存储的哈希函数，可以通过 `DHTDHTf` 将一个键映射到一个值，并返回一个 `DHTNodeService` 类型的值，该值可以用来获取 DHT 存储的实例。
- `DHTISStopWatch`：是一个 DHT 存储的时钟，可以用来获取 DHT 存储的实例，并停止和开始 DHT 存储的实例。
- `DHTPeriodicFlush`：是一个周期性执行的函数，用于将 DHT 存储的实例刷到本地磁盘，并清除了一些古老的 DHT 实例。
- `DHTRequestSubmitter`：实现了 `IpfsRequestSubmitter` 接口，可以用来向 DHT 存储提交请求，并获取一个 `DHTNodeService` 类型的实例，该实例可以用来获取 DHT 存储的实例。
- `DHTRouteableHook`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。
- `DHTRouteableHookEHCI`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。
- `DHTRouteableHookLegacy`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。
- `DHTRouteableHookPathConfirmation`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。
- `DHTRouteableHookSnapshotConfirmation`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。
- `DHTRouteableHookSleep`：是一个 DHT 路由的钩子函数，可以用来获取 DHT 路由的实例，并执行路由的相关操作。


```
package libp2p

import (
	"context"
	"os"
	"strings"
	"time"

	"github.com/ipfs/go-datastore"
	"github.com/ipfs/kubo/config"
	irouting "github.com/ipfs/kubo/routing"
	dht "github.com/libp2p/go-libp2p-kad-dht"
	dual "github.com/libp2p/go-libp2p-kad-dht/dual"
	record "github.com/libp2p/go-libp2p-record"
	routinghelpers "github.com/libp2p/go-libp2p-routing-helpers"
	host "github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/core/peer"
	routing "github.com/libp2p/go-libp2p/core/routing"
)

```

这段代码定义了一个名为 `RoutingOptionArgs` 的结构体，用于表示路由器选项的输入参数。

该结构体包含以下字段：

- `Ctx`：上下文句柄，用于在计算路由时获取远程数据源的上下文。
- `Host`：目标主机，用于在路由器中指定目标主机。
- `Datastore`：用于数据存储的库，可能是用于缓存数据。
- `Validator`：验证器，用于检查数据的有效性。
- `BootstrapPeers`：用于发现主机的Peer路由器。
- `OptimisticProvide`：用于 Optimistic 提供的支持。
- `OptimisticProvideJobsPoolSize`：用于 Optimistic 提供的静态 jobs 池大小。

此外，该代码还定义了一个名为 `RoutingOption` 的函数，接受一个名为 `RoutingOptionArgs` 的参数，用于设置路由器的选项，并返回一个路由器和错误信息。

该函数使用一个名为 `defaultHTTPRouters` 的数组，其中包含一些默认 HTTP 路由器，这些路由器在路由器类型为 "auto" 时使用。此外，该函数还定义了一个名为 `peer.AddrInfo` 的用于获取远程主机信息的类型，该类型可能用于在路由器中指定目标主机。


```
type RoutingOptionArgs struct {
	Ctx                           context.Context
	Host                          host.Host
	Datastore                     datastore.Batching
	Validator                     record.Validator
	BootstrapPeers                []peer.AddrInfo
	OptimisticProvide             bool
	OptimisticProvideJobsPoolSize int
}

type RoutingOption func(args RoutingOptionArgs) (routing.Routing, error)

// Default HTTP routers used in parallel to DHT when Routing.Type = "auto"
var defaultHTTPRouters = []string{
	"https://cid.contact", // https://github.com/ipfs/kubo/issues/9422#issuecomment-1338142084
	// TODO: add an independent router from Cloudflare
}

```

This code checks if any custom HTTP routers were passed via an environment variable, and if so, itSplits them by whitespace and appends them to a default list of routers. The routers are then constructed using the `irouting.ConstructHTTPRouter` function, which composes the router into a `routinghelpers.ParallelRouter` object. This object is then appended to the list of routers, along with any additional HTTP routers constructed from the environment variable. The `DefaultHTTPRouters` function has been tested in this environment.


```
func init() {
	// Override HTTP routers if custom ones were passed via env
	if routers := os.Getenv("IPFS_HTTP_ROUTERS"); routers != "" {
		defaultHTTPRouters = strings.Split(routers, " ")
	}
}

func constructDefaultHTTPRouters(cfg *config.Config) ([]*routinghelpers.ParallelRouter, error) {
	var routers []*routinghelpers.ParallelRouter
	// Append HTTP routers for additional speed
	for _, endpoint := range defaultHTTPRouters {
		httpRouter, err := irouting.ConstructHTTPRouter(endpoint, cfg.Identity.PeerID, httpAddrsFromConfig(cfg.Addresses), cfg.Identity.PrivKey)
		if err != nil {
			return nil, err
		}

		r := &irouting.Composer{
			GetValueRouter:      routinghelpers.Null{},
			PutValueRouter:      routinghelpers.Null{},
			ProvideRouter:       routinghelpers.Null{}, // modify this when indexers supports provide
			FindPeersRouter:     routinghelpers.Null{},
			FindProvidersRouter: httpRouter,
		}

		routers = append(routers, &routinghelpers.ParallelRouter{
			Router:                  r,
			IgnoreError:             true,             // https://github.com/ipfs/kubo/pull/9475#discussion_r1042507387
			Timeout:                 15 * time.Second, // 5x server value from https://github.com/ipfs/kubo/pull/9475#discussion_r1042428529
			DoNotWaitForSearchValue: true,
			ExecuteAfter:            0,
		})
	}
	return routers, nil
}

```

这段代码定义了一个名为 `ConstructDefaultRouting` 的函数，它接受一个名为 `cfg` 的 `Config` 对象和一个名为 `routingOpt` 的 `RoutingOption` 参数。函数的作用是在 `Routing.Type` 设置为未指定或设置为 "auto" 时，返回用于构建路由的 `Router`。

函数内部首先定义了一个名为 `routers` 的字符数组，用于存储定义的路由器。接着，函数调用了名为 `routingOpt` 的 `RoutingOption` 函数，并且如果出现错误，返回一个空字符串。如果 `routingOpt` 返回的值包含有效的路由器，则将其添加到 `routers` 数组中。最后，函数创建了一个名为 `routing` 的路由器对象，使用 `routers` 数组中的路由器，并将 `IgnoreError` 和 `ExecuteAfter` 设置为 0，表示路由器不会等待搜索值的到来，而是立即返回。函数返回这个路由器对象，并且如果出现错误，返回一个空字符串。


```
// ConstructDefaultRouting returns routers used when Routing.Type is unset or set to "auto"
func ConstructDefaultRouting(cfg *config.Config, routingOpt RoutingOption) RoutingOption {
	return func(args RoutingOptionArgs) (routing.Routing, error) {
		// Defined routers will be queried in parallel (optimizing for response speed)
		// Different trade-offs can be made by setting Routing.Type = "custom" with own Routing.Routers
		var routers []*routinghelpers.ParallelRouter

		dhtRouting, err := routingOpt(args)
		if err != nil {
			return nil, err
		}
		routers = append(routers, &routinghelpers.ParallelRouter{
			Router:                  dhtRouting,
			IgnoreError:             false,
			DoNotWaitForSearchValue: true,
			ExecuteAfter:            0,
		})

		httpRouters, err := constructDefaultHTTPRouters(cfg)
		if err != nil {
			return nil, err
		}

		routers = append(routers, httpRouters...)

		routing := routinghelpers.NewComposableParallel(routers)
		return routing, nil
	}
}

```

这段代码定义了一个名为 `constructDHTRouting` 的函数，用于在 `Routing.Type` 为 "dht" 时创建 DHT 路由。

该函数接收一个名为 `mode` 的选项参数，该参数表示 DHT 路由的模式，可以取 `dht.ModeOpt`、`dht.Mode` 或 `dht.CustomMode` 之一。函数内部根据所选模式计算出所需的最小权限，并返回一个名为 `Dual` 的路由选项对象。

具体来说，函数内部执行以下步骤：

1. 如果所选模式是 `dht.ModeOpt`，则创建一个名为 `dhtOpts` 的选项对象，其中包含以下选项：

		- `dht.Concurrency(10)`: 设置 DHT 并行度，确保在高并发时不会影响性能。
		- `dht.Mode(mode)`: 设置 DHT 路由的模式为指定的 `mode`。
		- `dht.Datastore(args.Datastore)`: 设置 DHT 数据存储，用于保存路由信息。
		- `dht.Validator(args.Validator)`: 设置 DHT 验证器，确保路由是有效的。

		如果需要支持乐观提供(using optimistic provide)，则执行以下操作：

			- `dhtOpts = append(dhtOpts, dht.EnableOptimisticProvide())`: 将 `dht.EnableOptimisticProvide()` 添加到 `dhtOpts` 中。
			- `args.OptimisticProvideJobsPoolSize`: 如果 `args.OptimisticProvideJobsPoolSize` 不为 0，则执行以下操作：

				- `dhtOpts = append(dhtOpts, dht.OptimisticProvideJobsPoolSize(args.OptimisticProvideJobsPoolSize))`: 将 `dht.OptimisticProvideJobsPoolSize()` 添加到 `dhtOpts` 中。

2. 如果所选模式是 `dht.CustomMode`，则执行以下操作：

		- `dhtOpts = append(dhtOpts, dht.CustomMode(args.CustomMode))`: 将 `dht.CustomMode()` 添加到 `dhtOpts` 中。

		- `args.Ctx, args.Host`: 保存上下文，并获取路由信息。

		- `return dual.New(args.Ctx, args.Host, ...)`: 返回一个名为 `Dual` 的路由选项对象。其中， `...` 表示该路由选项对象可能包含的其他选项，但不会输出。


```
// constructDHTRouting is used when Routing.Type = "dht"
func constructDHTRouting(mode dht.ModeOpt) RoutingOption {
	return func(args RoutingOptionArgs) (routing.Routing, error) {
		dhtOpts := []dht.Option{
			dht.Concurrency(10),
			dht.Mode(mode),
			dht.Datastore(args.Datastore),
			dht.Validator(args.Validator),
		}
		if args.OptimisticProvide {
			dhtOpts = append(dhtOpts, dht.EnableOptimisticProvide())
		}
		if args.OptimisticProvideJobsPoolSize != 0 {
			dhtOpts = append(dhtOpts, dht.OptimisticProvideJobsPoolSize(args.OptimisticProvideJobsPoolSize))
		}
		return dual.New(
			args.Ctx, args.Host,
			dual.DHTOption(dhtOpts...),
			dual.WanDHTOption(dht.BootstrapPeers(args.BootstrapPeers...)),
		)
	}
}

```

这段代码定义了一个名为"ConstructDelegatedRouting"的函数，用于在路由器类型为"custom"时执行路由。

函数接受四个参数：路由器配置、方法配置、对等ID和私钥。函数内部使用`irouting.Parse`函数将路由器和路由器参数解析为`irouting.Routing`和`irouting.Error`类型的数据，然后使用这些数据创建一个实现了`RoutingOption`接口的函数，该函数返回一个`RoutingOption`对象和可能的错误。

`RoutingOptionArgs`是一个代表路由器选项的`RoutingOptionArgs`类型的参数，其中包含了路由器的一些设置，如启动链路、主机、验证器和数据存储等。函数内部将这些设置转换为`irouting.ExtraDHTParams`和`irouting.ExtraHTTPParams`类型的数据，然后将这些设置传递给`irouting.Parse`函数。

最后，函数内部调用了`irouting.Irouter.Create`函数，该函数创建一个实现了`Irouter`接口的`Router`对象，并返回该对象的`Create`方法。通过调用这个`Create`方法，可以创建一个新的路由器实例，从而实现路由器的功能。


```
// ConstructDelegatedRouting is used when Routing.Type = "custom"
func ConstructDelegatedRouting(routers config.Routers, methods config.Methods, peerID string, addrs config.Addresses, privKey string) RoutingOption {
	return func(args RoutingOptionArgs) (routing.Routing, error) {
		return irouting.Parse(routers, methods,
			&irouting.ExtraDHTParams{
				BootstrapPeers: args.BootstrapPeers,
				Host:           args.Host,
				Validator:      args.Validator,
				Datastore:      args.Datastore,
				Context:        args.Ctx,
			},
			&irouting.ExtraHTTPParams{
				PeerID:     peerID,
				Addrs:      httpAddrsFromConfig(addrs),
				PrivKeyB64: privKey,
			},
		)
	}
}

```

这段代码定义了一个名为 `constructNilRouting` 的函数，它接受一个名为 `RoutingOptionArgs` 的参数。函数的作用是创建一个非空的路由实例，如果失败，则返回一个空错误。

函数的实现主要分两个步骤：

1. 根据传入的 `RoutingOptionArgs` 创建指定的路由实例。具体类型取决于 `RoutingOptionArgs` 的值，例如 `constructDHTRouting` 和 `constructDHTRouting`。这些函数的实现可以参考之前的回答。
2. 如果 `RoutingOptionArgs` 中没有指定任何特定的路由类型，则函数将创建一个空的 `Routing` 实例，同时返回一个空错误。

此外，函数还定义了一个名为 `httpAddrsFromConfig` 的函数，它接受一个名为 `cfgAddrs` 的配置参数。这个函数的作用是将配置中指定的地址列表返回给 HTTP 代理，覆盖了默认的 Swarm 地址列表。函数的实现可以简单理解为根据配置参数中的 `Announce` 和 `NoAnnounce` 字段来决定是否使用 Swarm 或者不使用 Swarm 路由。如果 `Announce` 字段指定，则使用 Swarm 路由；否则，如果 `NoAnnounce` 字段指定，则过滤掉 Swarm 路由，使用 `DHTServerOption` 和 `DHTClientOption` 中指定的路由。最后，函数返回一个包含所有配置中指定的地址列表的切片。


```
func constructNilRouting(_ RoutingOptionArgs) (routing.Routing, error) {
	return routinghelpers.Null{}, nil
}

var (
	DHTOption       RoutingOption = constructDHTRouting(dht.ModeAuto)
	DHTClientOption               = constructDHTRouting(dht.ModeClient)
	DHTServerOption               = constructDHTRouting(dht.ModeServer)
	NilRouterOption               = constructNilRouting
)

// httpAddrsFromConfig creates a list of addresses from the provided configuration to be used by HTTP delegated routers.
func httpAddrsFromConfig(cfgAddrs config.Addresses) []string {
	// Swarm addrs are announced by default
	addrs := cfgAddrs.Swarm
	// if Announce addrs are specified - override Swarm
	if len(cfgAddrs.Announce) > 0 {
		addrs = cfgAddrs.Announce
	} else if len(cfgAddrs.NoAnnounce) > 0 {
		// if Announce adds are not specified - filter Swarm addrs with NoAnnounce list
		maddrs := map[string]struct{}{}
		for _, addr := range addrs {
			maddrs[addr] = struct{}{}
		}
		for _, addr := range cfgAddrs.NoAnnounce {
			delete(maddrs, addr)
		}
		addrs = make([]string, 0, len(maddrs))
		for k := range maddrs {
			addrs = append(addrs, k)
		}
	}
	// append AppendAnnounce addrs to the result list
	if len(cfgAddrs.AppendAnnounce) > 0 {
		addrs = append(addrs, cfgAddrs.AppendAnnounce...)
	}
	return addrs
}

```

# `/opt/kubo/core/node/libp2p/routingopt_test.go`

This is a Go test that tests the `httpAddrsFromConfig` function, which parses an HTTP address configuration string and returns the corresponding IP addresses for a given list of addresses.

The test includes several scenarios, such as a) testing the behavior of `httpAddrsFromConfig` when the `Swarm` address config is not specified, b) testing the behavior of `httpAddrsFromConfig` when the `NoAnnounce` address config is specified, and c) testing the behavior of `httpAddrsFromConfig` when both `Swarm` and `NoAnnounce` address configs are specified.

The expected outcome of each test is that the function returns the correct IP addresses for the given configuration settings.


```
package libp2p

import (
	"testing"

	config "github.com/ipfs/kubo/config"
	"github.com/stretchr/testify/require"
)

func TestHttpAddrsFromConfig(t *testing.T) {
	require.Equal(t, []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
		httpAddrsFromConfig(config.Addresses{
			Swarm: []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
		}), "Swarm addrs should be taken by default")

	require.Equal(t, []string{"/ip4/192.168.0.1/tcp/4001"},
		httpAddrsFromConfig(config.Addresses{
			Swarm:    []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
			Announce: []string{"/ip4/192.168.0.1/tcp/4001"},
		}), "Announce addrs should override Swarm if specified")

	require.Equal(t, []string{"/ip4/0.0.0.0/udp/4001/quic-v1"},
		httpAddrsFromConfig(config.Addresses{
			Swarm:      []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
			NoAnnounce: []string{"/ip4/0.0.0.0/tcp/4001"},
		}), "Swarm addrs should not contain NoAnnounce addrs")

	require.Equal(t, []string{"/ip4/192.168.0.1/tcp/4001", "/ip4/192.168.0.2/tcp/4001"},
		httpAddrsFromConfig(config.Addresses{
			Swarm:          []string{"/ip4/0.0.0.0/tcp/4001", "/ip4/0.0.0.0/udp/4001/quic-v1"},
			Announce:       []string{"/ip4/192.168.0.1/tcp/4001"},
			AppendAnnounce: []string{"/ip4/192.168.0.2/tcp/4001"},
		}), "AppendAnnounce addrs should be included if specified")
}

```

# `/opt/kubo/core/node/libp2p/sec.go`

这段代码是一个 Go-IPFS 库中的包，它的作用是引入了 libp2p、go-libp2p 和 tls 等依赖，用于实现了一个基于 libp2p 的加密和安全性支持。

具体来说，这段代码：

1. 引入了 libp2p、go-libp2p 和 tls 等依赖，以便在项目中使用。
2. 定义了一个名为 "libp2p" 的常量，用于指代 libp2p 库。
3. 导入了 "github.com/ipfs/kubo/config" 和 "github.com/libp2p/go-libp2p/p2p/security/noise" 和 "github.com/libp2p/go-libp2p/p2p/security/tls" 等包，以便在项目中使用。
4. 定义了一个名为 "secioEnabledWarning" 的警告，用于在使用 SECIO 安全传输时发出警告，因为 SECIO 安全传输已经在Go-IPFS 0.7 中被移除，而在 Go-IPFS 0.9 中仍然支持。


```
package libp2p

import (
	"github.com/ipfs/kubo/config"

	"github.com/libp2p/go-libp2p"
	"github.com/libp2p/go-libp2p/p2p/security/noise"
	tls "github.com/libp2p/go-libp2p/p2p/security/tls"
)

const secioEnabledWarning = `The SECIO security transport was enabled in the config but is no longer supported.

SECIO disabled by default in go-ipfs 0.7 removed in go-ipfs 0.9. Please remove
Swarm.Transports.Security.SECIO from your IPFS config.`

```

这是一个名为`Security`的函数，它接受两个参数：`enabled`布尔值和`tptConfig`的`Transports`参数。函数返回一个实现了`Libp2pOpts`接口的`opts`参数。

函数首先判断`enabled`是否为`true`，如果是，则执行以下操作：

1. 如果未配置安全选项，函数将返回一个`Libp2pOpts`类型的函数，其中包含`noSecurity`选项，然后将`noSecurity`选项添加到`opts`参数中，并返回。
2. 如果已经配置了安全选项，函数将打印一个警告消息，然后尝试使用新的配置选项。

函数的具体实现包括以下步骤：

1. 如果未配置安全选项，函数将返回一个`Libp2pOpts`类型的函数，其中包含`noSecurity`选项，然后将`noSecurity`选项添加到`opts`参数中，并返回。这个函数在未配置安全选项时不会执行实际的配置操作，而仅仅是返回一个包含了安全选项的`opts`参数。
2. 如果已经配置了安全选项，函数将打印一个警告消息，`secioEnabledWarning`函数将打印一个警告消息，表示已配置的安全选项将会生效。然后函数将尝试使用新的配置选项。函数的`secioEnabledWarning`函数使用`tptConfig.Security.SECIO.WithDefault`函数来设置默认的安全选项。如果已经配置了`WithDefault`选项，函数将使用该选项中指定的安全选项，否则将尝试使用新的安全选项。函数的具体实现可能因为具体的`tptConfig`参数而有所不同。


```
func Security(enabled bool, tptConfig config.Transports) interface{} {
	if !enabled {
		return func() (opts Libp2pOpts) {
			log.Errorf(`Your IPFS node has been configured to run WITHOUT ENCRYPTED CONNECTIONS.
		You will not be able to connect to any nodes configured to use encrypted connections`)
			opts.Opts = append(opts.Opts, libp2p.NoSecurity)
			return opts
		}
	}

	if _, enabled := tptConfig.Security.SECIO.WithDefault(config.Disabled); enabled {
		log.Error(secioEnabledWarning)
	}

	// Using the new config options.
	return func() (opts Libp2pOpts) {
		opts.Opts = append(opts.Opts, prioritizeOptions([]priorityOption{{
			priority:        tptConfig.Security.TLS,
			defaultPriority: 200,
			opt:             libp2p.Security(tls.ID, tls.New),
		}, {
			priority:        tptConfig.Security.Noise,
			defaultPriority: 100,
			opt:             libp2p.Security(noise.ID, noise.New),
		}}))
		return opts
	}
}

```

# `/opt/kubo/core/node/libp2p/smux.go`

This is a function that configures a Chain of上坡 plants using the `Swarm.Transports.Multiplexers` library. It takes a priority option and an optional array of `libp2p.Option` structs containing the configurations for each muxer.

The function first checks if the `LIBP2P_MUX_PREFS` environment variable is not empty. If it's not, it logs an error and uses the `Swarm.Transports.Multiplexers` configuration field.

If the `LIBP2P_MUX_PREFS` environment variable is empty or the `Swarm.Transports.Multiplexers` configuration field is not specified, the function returns an error and mentions the `libp2p.Muxer` function as a fallback.

If the `Swarm.Transports.Multiplexers` configuration field is specified, the function creates an options struct that contains the configuration parameters for each muxer. It then checks if the `LIBP2P_MUX_PREFS` environment variable is still empty and if the `Swarm.Transports.Multiplexers` configuration field is specified. If the environment variable is still empty and the `Swarm.Transports.Multiplexers` configuration field is specified, the function returns an error.

If the `Swarm.Transports.Multiplexers` configuration field is specified, the function returns the `ChainOptions` method of the `libp2p.Chain` class with the configurations specified in the `opts` array.


```
package libp2p

import (
	"fmt"
	"os"
	"strings"

	"github.com/ipfs/kubo/config"

	"github.com/ipfs/kubo/core/node/libp2p/internal/mplex"
	"github.com/libp2p/go-libp2p"
	"github.com/libp2p/go-libp2p/p2p/muxer/yamux"
)

func makeSmuxTransportOption(tptConfig config.Transports) (libp2p.Option, error) {
	if prefs := os.Getenv("LIBP2P_MUX_PREFS"); prefs != "" {
		// Using legacy LIBP2P_MUX_PREFS variable.
		log.Error("LIBP2P_MUX_PREFS is now deprecated.")
		log.Error("Use the `Swarm.Transports.Multiplexers' config field.")
		muxers := strings.Fields(prefs)
		enabled := make(map[string]bool, len(muxers))

		var opts []libp2p.Option
		for _, tpt := range muxers {
			if enabled[tpt] {
				return nil, fmt.Errorf(
					"duplicate muxer found in LIBP2P_MUX_PREFS: %s",
					tpt,
				)
			}
			switch tpt {
			case yamux.ID:
				opts = append(opts, libp2p.Muxer(tpt, yamux.DefaultTransport))
			case mplex.ID:
				opts = append(opts, libp2p.Muxer(tpt, mplex.DefaultTransport))
			default:
				return nil, fmt.Errorf("unknown muxer: %s", tpt)
			}
		}
		return libp2p.ChainOptions(opts...), nil
	}
	return prioritizeOptions([]priorityOption{{
		priority:        tptConfig.Multiplexers.Yamux,
		defaultPriority: 100,
		opt:             libp2p.Muxer(yamux.ID, yamux.DefaultTransport),
	}, {
		priority:        tptConfig.Multiplexers.Mplex,
		defaultPriority: config.Disabled,
		opt:             libp2p.Muxer(mplex.ID, mplex.DefaultTransport),
	}}), nil
}

```

此代码定义了一个名为 "SmapuxTransport" 的函数，其接收一个名为 "config.Transports" 的参数。函数内部定义了一个内部函数 "func SmaxTransport"，该函数将接收的参数 "config.Transports" 传递给 "makeSmuxTransportOption" 函数。如果 "makeSmuxTransportOption" 函数返回的值包含错误，则将此错误与 "func()" 函数返回的选项 "opts" 中的第一个元素联系起来，导致函数返回。如果 "makeSmuxTransportOption" 函数返回的值是有效的，则将其封装在 "opts" 类型的变量中，并将其返回。

具体来说，此函数的作用是创建一个名为 "SmapuxTransport" 的函数，该函数接收一个名为 "config.Transports" 的参数，然后使用接收到的参数调用 "makeSmuxTransportOption" 函数，获取一个选项对象。如果 "makeSmuxTransportOption" 函数返回的值包含错误，则将其封装在 "opts" 类型的变量中，并返回。如果 "makeSmuxTransportOption" 函数返回的值是有效的，则将其封装在 "opts" 类型的变量中，并返回。


```
func SmuxTransport(tptConfig config.Transports) func() (opts Libp2pOpts, err error) {
	return func() (opts Libp2pOpts, err error) {
		res, err := makeSmuxTransportOption(tptConfig)
		if err != nil {
			return opts, err
		}
		opts.Opts = append(opts.Opts, res)
		return opts, nil
	}
}

```

# `/opt/kubo/core/node/libp2p/topicdiscovery.go`

该代码定义了一个名为 TopicDiscovery 的函数，它返回一个主题发现的元组。具体来说，它实现了基于随机化的 P2P 路由协议中的主题发现功能。

该函数接收两个参数：一个 P2P 主机（host）和一个内容路由路由器（routing.ContentRouting）。函数返回一个发现服务（discovery.Discovery）和一个错误（err）。发现服务用于在发现可用主题时返回，而错误是可能发生的。

函数内部，首先创建一个底层的主题发现路由器（discovery.NewRoutingDiscovery）。接着，实现了一个基于最大后退时间（maxBackoff）的 exponential 后退算法，用于在主题发现失败时进行重试。如果成功发现主题，该函数将返回发现服务。


```
package libp2p

import (
	"math/rand"
	"time"

	"github.com/libp2p/go-libp2p/core/discovery"
	"github.com/libp2p/go-libp2p/core/host"
	"github.com/libp2p/go-libp2p/p2p/discovery/backoff"
	disc "github.com/libp2p/go-libp2p/p2p/discovery/routing"

	"github.com/libp2p/go-libp2p/core/routing"
)

func TopicDiscovery() interface{} {
	return func(host host.Host, cr routing.ContentRouting) (service discovery.Discovery, err error) {
		baseDisc := disc.NewRoutingDiscovery(cr)
		minBackoff, maxBackoff := time.Second*60, time.Hour
		rng := rand.New(rand.NewSource(rand.Int63()))
		d, err := backoff.NewBackoffDiscovery(
			baseDisc,
			backoff.NewExponentialBackoff(minBackoff, maxBackoff, backoff.FullJitter, time.Second, 5.0, 0, rng),
		)
		if err != nil {
			return nil, err
		}

		return d, nil
	}
}

```

# `/opt/kubo/core/node/libp2p/transport.go`

这段代码定义了一个名为 "libp2p" 的包，其中包含了一些用于实现 QuIC、TCP 和 WebSocket 网络传输协议的函数和变量。

具体来说，这段代码实现了一个名为 "transport" 的包，它包含了多种不同的网络传输协议实现，包括 QuIC、TCP 和 WebSocket。这些函数和变量可以被用来创建和管理 QuIC 或 TCP 连接，或者创建 WebSocket 连接。

此外，这段代码还定义了一个名为 "metrics" 的包，其中包含了一些用于收集和统计网络传输协议的统计数据。这些数据可以被用来监控网络传输的性能和问题，例如延迟、吞吐量、成功率和失败率等。

最后，这段代码还定义了一个名为 "quic" 和 "tcp" 的函数指针，它们分别对应 QuIC 和 TCP 传输协议的实现。这些函数指针可以被用来创建和管理与远程服务器或客户端的 QuIC 或 TCP 连接。


```
package libp2p

import (
	"fmt"

	"github.com/ipfs/kubo/config"
	"github.com/libp2p/go-libp2p"
	"github.com/libp2p/go-libp2p/core/metrics"
	quic "github.com/libp2p/go-libp2p/p2p/transport/quic"
	"github.com/libp2p/go-libp2p/p2p/transport/tcp"
	"github.com/libp2p/go-libp2p/p2p/transport/websocket"
	webtransport "github.com/libp2p/go-libp2p/p2p/transport/webtransport"

	"go.uber.org/fx"
)

```

这段代码定义了一个名为 "Transports" 的函数，接收一个名为 "config.Transports" 的参数，并返回一个名为 "func" 的接口，该接口定义了一个接收一个 "struct" 类型的参数 "pnet" 和一个 "opts" 参数的函数，返回一个 "Libp2pOpts" 类型的选项 "opts"，或者一个错误 "err"。

函数的作用是设置传入 "pnet" 结构的网络传输配置，包括 TCP、WebSocket 和 UDP 穿越。具体设置步骤如下：

1. 根据 "config.Transports" 的网络类型设置 TCP 传输配置，如果 "privateNetworkEnabled" 设置为 true，则使用 "private" 类型的 TCP 传输，并开启 Metrics 记录。
2. 如果 "privateNetworkEnabled" 设置为 false，则直接使用 "public" 类型的 TCP 传输。
3. 如果 "config.Network.WEBSocket" 的默认设置为 true，则设置 WebSocket 传输。
4. 如果 "config.Network.QUIC" 的默认设置为 true，则设置 QUIC 传输，但仅限于非私人物流网络。
5. 如果 "config.Network.WEBTransport" 的默认设置为 true，则设置 WebTransport 传输，但仅限于非私人物流网络。
6. 如果 "config.Network.SWAMP.WEBTransport" 的默认设置为 true，则设置 WebTransport 传输，但仅限于 Swarm 网络。
7. 如果 "privateNetworkEnabled" 是 true 并且 "config.Network.TCP.WithDefault(true)" 是 false，则设置 TCP 传输为使用 Metrics 记录。
8. 如果 "privateNetworkEnabled" 是 false 并且 "config.Network.TCP.WithDefault(true)" 是 true，则设置 TCP 传输为使用 Private 类型。
9. 如果 "privateNetworkEnabled" 是 false 并且 "config.Network.WEBSocket.WithDefault(true)" 是 false，则设置 WebSocket 传输为使用 Metrics 记录。
10. 如果 "privateNetworkEnabled" 是 true 并且 "config.Network.WEBSocket.WithDefault(true)" 是 true，则设置 WebSocket 传输为使用 Private 类型。
11. 如果 "privateNetworkEnabled" 是 true 并且 "config.Network.QUIC.WithDefault(!privateNetworkEnabled)" 是 false，则设置 QUIC 传输为使用 Metrics 记录。
12. 如果 "privateNetworkEnabled" 是 false 并且 "config.Network.QUIC.WithDefault(!privateNetworkEnabled)" 是 true，则设置 QUIC 传输为使用 Private 类型。


```
func Transports(tptConfig config.Transports) interface{} {
	return func(pnet struct {
		fx.In
		Fprint PNetFingerprint `optional:"true"`
	},
	) (opts Libp2pOpts, err error) {
		privateNetworkEnabled := pnet.Fprint != nil

		if tptConfig.Network.TCP.WithDefault(true) {
			// TODO(9290): Make WithMetrics configurable
			opts.Opts = append(opts.Opts, libp2p.Transport(tcp.NewTCPTransport, tcp.WithMetrics()))
		}

		if tptConfig.Network.Websocket.WithDefault(true) {
			opts.Opts = append(opts.Opts, libp2p.Transport(websocket.New))
		}

		if tptConfig.Network.QUIC.WithDefault(!privateNetworkEnabled) {
			if privateNetworkEnabled {
				return opts, fmt.Errorf(
					"QUIC transport does not support private networks, please disable Swarm.Transports.Network.QUIC",
				)
			}
			opts.Opts = append(opts.Opts, libp2p.Transport(quic.NewTransport))
		}

		if tptConfig.Network.WebTransport.WithDefault(!privateNetworkEnabled) {
			if privateNetworkEnabled {
				return opts, fmt.Errorf(
					"WebTransport transport does not support private networks, please disable Swarm.Transports.Network.WebTransport",
				)
			}
			opts.Opts = append(opts.Opts, libp2p.Transport(webtransport.New))
		}

		return opts, nil
	}
}

```

该函数名为`BandwidthCounter`，返回值为`opts`和`reporter`，函数内部使用了以下步骤：

1. 创建一个名为`metrics.BandwidthCounter`的`reporter`实例，并将其赋值给`reporter`变量。
2. 将`libp2p.BandwidthReporter`类的实例创建并传入`reporter`实例，将其添加到`opts`数组中。
3. 返回`opts`和`reporter`，即将创建的`BandwidthCounter`实例返回给调用者。

该函数的作用是创建一个可以测量网络带宽的`BandwidthCounter`实例，并将其作为参数传递给其他函数或使用。通过使用`libp2p.BandwidthReporter`来报告网络带宽，可以确保在函数中进行网络带宽测量。


```
func BandwidthCounter() (opts Libp2pOpts, reporter *metrics.BandwidthCounter) {
	reporter = metrics.NewBandwidthCounter()
	opts.Opts = append(opts.Opts, libp2p.BandwidthReporter(reporter))
	return opts, reporter
}

```

# `/opt/kubo/core/node/libp2p/fd/sys_not_unix.go`

这段代码是一个 Go 语言编写的工具函数，它的作用是返回操作系统上所有文件描述符（FD）的数量。然后将其输出到控制台。

首先，该代码使用 Go 语言的 `build` 命令为该文件编写了 `linux`、`darwin` 和 `windows` 平台的支持。这样做之后，它将生成支持这些操作系统的构建图，从而使得该工具函数可以被在任何支持这些操作系统的平台上的 Go 语言开发人员使用。

然后，该代码导入了自定义的 `fd` 包，并定义了一个名为 `GetNumFDs` 的函数，该函数返回操作系统上所有文件描述符的数量，结果为 0。

最后，该代码导入了 `fmt` 包的 `Printf` 函数，并在控制台上输出字符串 "所有文件描述符的数量"。


```
//go:build !linux && !darwin && !windows

package fd

func GetNumFDs() int {
	return 0
}

```

# `/opt/kubo/core/node/libp2p/fd/sys_unix.go`

这段代码是一个用于在Go语言中编译Linux和Darwin操作系统的脚本。它使用了Go语言的`build`包来编译这个脚本。

具体来说，第一个导入语句 `//go:build linux || darwin` 是用于定义编译选项。`||` 表示如果第一个编译选项失败，则使用第二个编译选项编译。这个脚本会编译成一个名为 `fd` 的包。

接下来是两个导入语句，它们用于定义 `GetNumFDs` 和 `unix.Rlimit` 函数的定义。

`GetNumFDs` 函数的实现使用 `unix.Rlimit` 类型来获取文件描述符的数量。如果当前操作系统的限制与操作系统的版本不匹配，函数的行为将不可预测。

`unix.Getrlimit` 函数的实现使用 `unix.RLIMIT_NOFILE` 标志来设置文件描述符的数量，如果没有设置此标志，它的值将等于 `unix.Rlimit_硬` 常量的值，它的值为32767。`unix.RLIMIT_NOFILE` 标志用于限制文件描述符的数量，确保不会创建过多的进程。

最后，没有其他代码在导入或定义任何函数，所以函数的行为将不可预测。


```
//go:build linux || darwin
// +build linux darwin

package fd

import (
	"golang.org/x/sys/unix"
)

func GetNumFDs() int {
	var l unix.Rlimit
	if err := unix.Getrlimit(unix.RLIMIT_NOFILE, &l); err != nil {
		return 0
	}
	return int(l.Cur)
}

```

# `/opt/kubo/core/node/libp2p/fd/sys_windows.go`

这段代码是一个 Go 语言编写的 Go 语言函数，它返回了一个 `math.MaxInt`，主要用于返回两个整数的最大值。

具体来说，这段代码定义了一个名为 `GetNumFDs` 的函数，它返回一个整数，这个整数是 `math.MaxInt` 的值。 `math.MaxInt` 是一个在 Go 语言中定义的常量，它返回两个整数中的最大值。

此外，该函数没有返回任何其他类型的值，它仅仅返回一个整数。


```
//go:build windows

package fd

import (
	"math"
)

func GetNumFDs() int {
	return math.MaxInt
}

```

# `/opt/kubo/core/node/libp2p/internal/mplex/conn.go`

该代码定义了一个名为 "mplex" 的 "conn" 类型，它使用名为 "mp" 的 "Multiplex" 类型来支持 "network.MuxedConn" 接口。

在这个 "conn" 类型中，我们定义了一个名为 "Multiplex" 的接口，它代表了一个支持多种协议的复用连接。然后，我们定义了一个名为 "conn" 的类型，它实现了 "Multiplex" 接口，提供了一个 "network.MuxedConn" 类型的接口，该接口允许在同时使用多个网络连接的情况下，仍然能够进行实时的数据传输。

通过使用 "mp" "Multiplex" 类型，我们可以实现多个底层网络的复用，从而实现在一个应用程序中同时使用不同的网络连接，例如 TCP 和 UDP。而 "conn" 类型中的 "network.MuxedConn" 接口允许我们对复用的连接进行统一的管理和配置，使得我们能够更加方便地实现实时的数据传输。


```
// Code copied from https://github.com/libp2p/go-libp2p/blob/9bd85029550a084fca63ec6ff9184122cdf06591/p2p/muxer/mplex/conn.go
package mplex

import (
	"context"

	"github.com/libp2p/go-libp2p/core/network"

	mp "github.com/libp2p/go-mplex"
)

type conn mp.Multiplex

var _ network.MuxedConn = &conn{}

```

这段代码定义了一个名为NewMuxedConn的函数，它从一个名为mp.Multiplex的接口中创建一个新的网络.MuxedConn实例。

函数有两个闭包函数，分别是Close和IsClosed，它们分别是与连接相关的操作，可以确保在正确关闭连接时返回true或false。

函数还有一个名为OpenStream的函数，用于创建一个新的网络.MuxedStream实例。它接收一个上下文事件，并在该上下文中创建一个新的mp.Multiplex连接并返回它。如果连接创建成功，函数返回一个非空MuxedStream，否则返回错误。

总的来说，这段代码定义了一个用于创建网络.MuxedConn实例的函数，该函数创建的连接可以通过Close和IsClosed函数进行管理，并可以使用OpenStream函数来打开或关闭连接。


```
// NewMuxedConn constructs a new Conn from a *mp.Multiplex.
func NewMuxedConn(m *mp.Multiplex) network.MuxedConn {
	return (*conn)(m)
}

func (c *conn) Close() error {
	return c.mplex().Close()
}

func (c *conn) IsClosed() bool {
	return c.mplex().IsClosed()
}

// OpenStream creates a new stream.
func (c *conn) OpenStream(ctx context.Context) (network.MuxedStream, error) {
	s, err := c.mplex().NewStream(ctx)
	if err != nil {
		return nil, err
	}
	return (*stream)(s), nil
}

```

该代码的作用是实现了一个接受其他端点发送的流（如文件、网络数据等）的AcceptStream函数。

首先，该函数的参数c是一个接收网络数据流和错误值的指针，接收流可以是已经打开的文件句柄或网络套接字等。函数内部首先尝试使用c.mplex()函数将流接受进来，如果失败，则返回 nil 和错误。如果成功，则返回一个接受流的指针和一个 nil 表示成功接收流。

函数内部还定义了一个名为mplex的函数，该函数接收一个Multiplex对象和一个网络连接对象，并返回一个Multiplex类型的接受流。Multiplex对象代表一个支持多种流接受（如文件、网络数据等）的套接字，通过该对象可以设置或获取接受哪些流。


```
// AcceptStream accepts a stream opened by the other side.
func (c *conn) AcceptStream() (network.MuxedStream, error) {
	s, err := c.mplex().Accept()
	if err != nil {
		return nil, err
	}
	return (*stream)(s), nil
}

func (c *conn) mplex() *mp.Multiplex {
	return (*mp.Multiplex)(c)
}

```

# `/opt/kubo/core/node/libp2p/internal/mplex/stream.go`

该代码是一个名为"mplex"的包，它实现了"network.MuxedStream"接口，用于在基于Mux的流中传输数据。具体来说，这个包实现了一个名为"Stream"的类型，该类型代表了一个可以传输数据的流。

在代码中，首先导入了"time"和"github.com/libp2p/go-libp2p/core/network"三个包。其中，"time"包提供了时间相关的函数和类型，用于在代码中处理时间相关的操作；而"github.com/libp2p/go-libp2p/core/network"包则提供了与网络相关的接口和函数，以便在代码中与各种网络进行交互。

接着，定义了一个名为"stream"的类型，该类型代表了一个可以传输数据的流。接着，通过使用"github.com/libp2p/go-mplex"包中提供的"mp"类型，将"Stream"类型转换为"mp.Stream"类型，以便在代码中可以更方便地使用该流。

最后，定义了一个名为"mpStream"的函数，该函数接受一个"stream"类型的参数，并返回一个指向该流对象的指针。通过创建这个指针，就可以使得"network.MuxedStream"接口在代码中生效，从而实现对数据流的支持。


```
// Code copied from https://github.com/libp2p/go-libp2p/blob/9bd85029550a084fca63ec6ff9184122cdf06591/p2p/muxer/mplex/stream.go
package mplex

import (
	"time"

	"github.com/libp2p/go-libp2p/core/network"

	mp "github.com/libp2p/go-mplex"
)

// stream implements network.MuxedStream over mplex.Stream.
type stream mp.Stream

var _ network.MuxedStream = &stream{}

```

这两函数是用来对一个网络流进行读写操作的。

具体来说，`func (s *stream) Read(b []byte) (n int, err error)`函数接收一个字节切片（[]byte），然后将字节流（s.mplex().）进行缓冲区读取（Read）操作。如果读取操作成功，将返回读取的字节数（n）和状态（err）。如果状态错误，将返回一个非空（n为0，err为非空值）错误。这里的状态判断中，使用了`mp.ErrStreamReset`错误。

`func (s *stream) Write(b []byte) (n int, err error)`函数接收一个字节切片（[]byte），然后将字节流（s.mplex().）进行缓冲区写入（Write）操作。如果写入操作成功，将返回写入的字节数（n）和错误（err）。如果错误，将返回一个非空（n为0，err为非空值）错误。这里的状态判断中，使用了`net.ErrReset`错误。


```
func (s *stream) Read(b []byte) (n int, err error) {
	n, err = s.mplex().Read(b)
	if err == mp.ErrStreamReset {
		err = network.ErrReset
	}

	return n, err
}

func (s *stream) Write(b []byte) (n int, err error) {
	n, err = s.mplex().Write(b)
	if err == mp.ErrStreamReset {
		err = network.ErrReset
	}

	return n, err
}

```

这段代码定义了四个函数，它们都是接受一个名为s的整数类型的指针参数。

第一个函数名为func，接收的参数是一个整数类型的指针变量，没有返回值。函数的作用是关闭流中的数据读写操作。具体实现是通过调用s.mplex().Close()方法来实现的。

第二个函数名为func，同样接收一个整数类型的指针变量，也没有返回值。函数的作用是关闭流中的数据读写操作。具体实现是通过调用s.mplex().CloseWrite()方法来实现的。

第三个函数名为func，同样接收一个整数类型的指针变量，也没有返回值。函数的作用是关闭流中的数据读写操作。具体实现是通过调用s.mplex().CloseRead()方法来实现的。

第四个函数名为func，同样接收一个整数类型的指针变量，也没有返回值。函数的作用是关闭流中的数据读写操作。具体实现是通过调用s.mplex().Reset()方法来实现的。

这些函数的作用都是关闭流中的数据读写操作，但是它们实现在方法上略有不同。具体来说，第一个函数会尝试关闭流中的数据读写操作，如果关闭失败，则会返回一个错误；而其他函数会尝试关闭流中的数据读写操作，如果关闭成功，则不会返回任何错误。


```
func (s *stream) Close() error {
	return s.mplex().Close()
}

func (s *stream) CloseWrite() error {
	return s.mplex().CloseWrite()
}

func (s *stream) CloseRead() error {
	return s.mplex().CloseRead()
}

func (s *stream) Reset() error {
	return s.mplex().Reset()
}

```

这段代码定义了一个名为 `func` 的函数，它接受一个名为 `s` 的指针参数，代表一个 `mp.Stream` 类型。

这个函数可以用来设置 `s` 对象的三个 Deadline，分别是读取、写入和执行。通过调用 `s.mplex().SetDeadline` 和 `s.mplex().SetReadDeadline` 和 `s.mplex().SetWriteDeadline` 来设置这些 Deadline。

`mplex` 函数也定义在函数内部，它是通过 `(*mp.Stream)(s)` 返回的 `mp.Stream` 类型指针。这个指针可能用来从 `s` 对象中获取 `mp.Stream` 类型的实例。


```
func (s *stream) SetDeadline(t time.Time) error {
	return s.mplex().SetDeadline(t)
}

func (s *stream) SetReadDeadline(t time.Time) error {
	return s.mplex().SetReadDeadline(t)
}

func (s *stream) SetWriteDeadline(t time.Time) error {
	return s.mplex().SetWriteDeadline(t)
}

func (s *stream) mplex() *mp.Stream {
	return (*mp.Stream)(s)
}

```

# `/opt/kubo/core/node/libp2p/internal/mplex/transport.go`

这段代码定义了一个名为 "mplex" 的包，其作用是实现一个名为 "mplex.Transport" 的接口，该接口负责在 MPEG transport协议中实现与网络的交互。

首先，它导入了自 "libp2p/go-libp2p" 库中的 "net" 和 "mp" 包，分别用于网络和 "mplex" 包的依赖。

接着，它定义了一个名为 "DefaultTransport" 的变量，该变量为一个名为 "Transport" 的接口的实例，该接口没有具体的实现，因此可以理解为 "Transport" 的默认实现。

然后，它定义了一个名为 "ID" 的常量，该常量表示一个保留字，用于标识 "mplex" 包中的内容。

最后，它导入了 "mp" 包中的 "Transport" 接口，该接口用于实现 MPEG transport协议与网络的交互。


```
// Code copied from https://github.com/libp2p/go-libp2p/blob/9bd85029550a084fca63ec6ff9184122cdf06591/p2p/muxer/mplex/transport.go
package mplex

import (
	"net"

	"github.com/libp2p/go-libp2p/core/network"

	mp "github.com/libp2p/go-mplex"
)

// DefaultTransport has default settings for Transport
var DefaultTransport = &Transport{}

const ID = "/mplex/6.7.0"

```

这段代码定义了一个名为`Transport`的`mux.Multiplexer`类型，表示了一个实现`muxed`连接的`Transport`实例。

`Transport`结构体包含一个`NewConn`函数，该函数接收一个`net.Conn`对象和一个`isServer`布尔值和一个`scope`网络作用域。该函数使用`mp.NewMultiplex`函数创建一个实现了`mplex-backed`连接的`Multiplexer`实例，并返回一个`network.MuxedConn`对象和一个错误。

如果调用`NewConn`时传入的`net.Conn`对象是已关闭的或未连接的，则函数将返回一个`nil`值。如果`isServer`为`true`，则函数将返回一个` network.MuxedServer`类型的`Transport`实例，而不是一个`Transport`实例。如果`scope`网络作用域包含` PeerScope`选项，则`NewMultiplexer`将尝试与远程对端建立双向多路复用连接。

由于 `Transport` 实体的具体实现没有明确的定义，因此无法提供更多细节。


```
var _ network.Multiplexer = &Transport{}

// Transport implements mux.Multiplexer that constructs
// mplex-backed muxed connections.
type Transport struct{}

func (t *Transport) NewConn(nc net.Conn, isServer bool, scope network.PeerScope) (network.MuxedConn, error) {
	m, err := mp.NewMultiplex(nc, isServer, scope)
	if err != nil {
		return nil, err
	}
	return NewMuxedConn(m), nil
}

```

# `/opt/kubo/core/node/libp2p/internal/mplex/transport_test.go`

这段代码是一个 Go 语言编写的测试用例，用于测试 Libp2p 中的 Mplex 传输层的默认实现。主要作用是创建一个 Mplex 传输实例，然后使用该实例进行测试。

具体来说，代码包括以下几个部分：

1. 导入必要的库：
	* "errors"
	* "net"
	* "testing"
	* "github.com/libp2p/go-libp2p/core/network"
	* "github.com/libp2p/go-libp2p/p2p/muxer/testsuite"
2. 定义一个名为 "TestDefaultTransport" 的测试函数：
	* "t *testing.T"表示该函数属于 "testing" 包；
	* "err *errors.Error"表示该函数需要接收一个 "errors.Error" 类型的参数；
	* "net net"表示该函数需要接收一个 "net" 类型的参数；
	* "testing testing"表示该函数需要接收一个 "testing" 类型的参数；
	* "github.com/libp2p/go-libp2p/core/network" "github.com/libp2p/go-libp2p/p2p/muxer/testsuite"表示该函数需要从 "github.com/libp2p/go-libp2p/core/network" 和 "github.com/libp2p/go-libp2p/p2p/muxer/testsuite" 导入。
3. 调用 "DefaultTransport" 函数进行测试：
	* "test.SubtestAll" 表示该函数需要调用 "DefaultTransport" 函数的所有测试子函数；
	* "testsuite.TestCases" 表示该函数需要使用 "github.com/libp2p/go-libp2p/p2p/muxer/testsuite/TestCases" 函数来创建测试实例。


```
// Code copied from https://github.com/libp2p/go-libp2p/blob/9bd85029550a084fca63ec6ff9184122cdf06591/p2p/muxer/mplex/transport_test.go
package mplex

import (
	"errors"
	"net"
	"testing"

	"github.com/libp2p/go-libp2p/core/network"
	test "github.com/libp2p/go-libp2p/p2p/muxer/testsuite"
)

func TestDefaultTransport(t *testing.T) {
	test.SubtestAll(t, DefaultTransport)
}

```

这段代码定义了一个名为`memoryScope`的结构体，它包含以下字段：

- `network.PeerScope`：这是一个`network.PeerScope`类型的字段，它表示该内存区域是为与其他peer设备共享而设定的。
- `limit`：这是一个`int`类型的字段，表示该内存区域的最大大小。
- `reserved`：这是一个`int`类型的字段，表示该内存区域保留的额外大小。

代码中定义了一个名为`(m * memoryScope)`的`*memoryScope`类型的指针变量`m`，它的作用是：

- `m. ReserveMemory`：该函数接收一个`size`参数和一个`prio`参数，表示要保留的内存大小。函数首先检查`m.reserved`是否足够大来容纳`size`的内存，如果是，则返回一个`nil`表示成功。否则，函数返回一个`errors.New`错误，指出内存不足。
- `m. ReleaseMemory`：该函数接收一个`size`参数，表示要释放的内存大小。函数首先减少`m.reserved`的大小，但如果`m.reserved`已经小于`0`，则函数会 panic并退出。

该内存区域的作用是管理一个共享内存区域，可以根据需要对其进行保留或释放。


```
type memoryScope struct {
	network.PeerScope
	limit    int
	reserved int
}

func (m *memoryScope) ReserveMemory(size int, prio uint8) error {
	if m.reserved+size > m.limit {
		return errors.New("too much")
	}
	m.reserved += size
	return nil
}

func (m *memoryScope) ReleaseMemory(size int) {
	m.reserved -= size
	if m.reserved < 0 {
		panic("too much memory released")
	}
}

```

该代码定义了一个名为`memoryLimitedTransport`的结构体，它包含一个名为`Transport`的`Transport`类型字段。

接着，该结构体定义了一个名为`NewConn`的函数，该函数接收一个`net.Conn`对象和一个`bool`类型的参数`isServer`，以及一个`net.PeerScope`类型的参数`scope`。

函数内部首先创建一个`memoryLimitedTransport`实例的`Transport`对象，然后使用`Transport`对象的`NewConn`函数创建一个新的`net.Conn`对象。接着，创建新的`net.Conn`对象之后，函数内部使用`memoryScope`约束快速创建一个`net.PeerScope`对象，该对象包含`limit`字段，其值为`3 * 1 << 20`，表示允许此客户端连接上的服务器的内存大小为`3MB`。最后，函数返回创建的`net.MuxedConn`对象和创建`net.PeerScope`对象之后的结果。

该代码的主要目的是创建一个可以设置最大内存限制的传输协议，以便在网络应用程序中对客户端和服务器之间的连接设置硬限制。通过`memoryLimitedTransport`，我们可以创建一个客户端和服务器之间的连接，使得服务器能够在一定的情况下允许客户端使用尽可能多的内存，但客户端的内存使用不会超出设置的最大限制。


```
type memoryLimitedTransport struct {
	Transport
}

func (t *memoryLimitedTransport) NewConn(nc net.Conn, isServer bool, scope network.PeerScope) (network.MuxedConn, error) {
	return t.Transport.NewConn(nc, isServer, &memoryScope{
		limit:     3 * 1 << 20,
		PeerScope: scope,
	})
}

func TestDefaultTransportWithMemoryLimit(t *testing.T) {
	test.SubtestAll(t, &memoryLimitedTransport{
		Transport: *DefaultTransport,
	})
}

```

# `/opt/kubo/coverage/main/main.go`

这段代码是一个 Go 语言程序，它执行以下操作：

1. 使用 `build` 命令构建 Go 语言项目。
2. 使用 `testrunmain` 构建运行 `main` 函数的程序（即 `main.go`）。
3. 使用 `fmt` 打印输出 `main.go` 包的名称。
4. 使用 `io` 用于输入/输出操作，包括从标准输入（通常是终端）读取用户输入，并向标准输出（通常是屏幕）输出信息。
5. 使用 `os` 用于操作系统功能，包括文件操作，如创建文件，删除文件，以及获取操作系统中可用的命令行参数。
6. 使用 `os/exec` 包在后台执行命令。
7. 使用 `syscall` 包实现操作系统调用，用于获取操作系统实时的信号（如 SIGTERM、SIGKILL 等）。
8. 将第 5 步和第 6 步的结果传递给第 3 步，以便输出结果。


```
//go:build testrunmain
// +build testrunmain

package main

import (
	"fmt"
	"io"
	"os"
	"os/exec"
	"os/signal"
	"strconv"
	"syscall"
)

```

This appears to be a Go program that uses the `ipfs-test-cover` tool to generate a cover file for the `ipfs-test-run` program. The cover file is generated from the `^TestRunMain^` function, which appears to be the entry point for the `ipfs-test` CLI.

The program takes several arguments, including the name of the cover file to generate and the name of the test. It then sets up an execution context for the `ipfs-test-cover` tool, passes the specified arguments to it, and watches for the exit signal of the tool.

If any errors occur during the execution of the program, it prints an error message and exits with a non-zero status code. Otherwise, it returns 0 to indicate successful execution.


```
func main() {
	coverDir := os.Getenv("IPFS_COVER_DIR")
	if len(coverDir) == 0 {
		fmt.Println("IPFS_COVER_DIR not defined")
		os.Exit(1)
	}
	coverFile, err := os.CreateTemp(coverDir, "coverage-")
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}

	retFile, err := os.CreateTemp("", "cover-ret-file")
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}

	args := []string{"-test.run", "^TestRunMain$", "-test.coverprofile=" + coverFile.Name(), "--"}
	args = append(args, os.Args[1:]...)

	p := exec.Command("ipfs-test-cover", args...)
	p.Stdin = os.Stdin
	p.Stdout = os.Stdout
	p.Stderr = os.Stderr
	p.Env = append(os.Environ(), "IPFS_COVER_RET_FILE="+retFile.Name())

	p.SysProcAttr = &syscall.SysProcAttr{
		Pdeathsig: syscall.SIGTERM,
	}

	sig := make(chan os.Signal, 10)
	start := make(chan struct{})
	go func() {
		<-start
		for {
			p.Process.Signal(<-sig)
		}
	}()

	signal.Notify(sig, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM)

	err = p.Start()
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}

	close(start)

	err = p.Wait()
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}

	b, err := io.ReadAll(retFile)
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}
	b = b[:len(b)-1]
	d, err := strconv.Atoi(string(b))
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(1)
	}
	os.Exit(d)
}

```

# IPFS : The `Add` command demystified

The goal of this document is to capture the code flow for adding a file (see the `coreapi` package) using the IPFS CLI, in the process exploring some datastructures and packages like `ipld.Node` (aka `dagnode`), `FSNode`, `MFS`, etc.

## Concepts
- [Files](https://github.com/ipfs/docs/issues/133)

--- 

**Try this yourself**
> 
> ```
> # Convert a file to the IPFS format.
> echo "Hello World" > new-file
> ipfs add new-file
> added QmWATWQ7fVPP2EFGu71UkfnqhYXDYH566qy47CnJDgvs8u new-file
> 12 B / 12 B [=========================================================] 100.00%
>
> # Add a file to the MFS.
> NEW_FILE_HASH=$(ipfs add new-file -Q)
> ipfs files cp /ipfs/$NEW_FILE_HASH /new-file
> 
> # Get information from the file in MFS.
> ipfs files stat /new-file
> # QmWATWQ7fVPP2EFGu71UkfnqhYXDYH566qy47CnJDgvs8u
> # Size: 12
> # CumulativeSize: 20
> # ChildBlocks: 0
> # Type: file
> 
> # Retrieve the contents.
> ipfs files read /new-file
> # Hello World
> ```

## Code Flow

**[`UnixfsAPI.Add()`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreapi/unixfs.go#L31)** - *Entrypoint into the `Unixfs` package*

The `UnixfsAPI.Add()` acts on the input data or files, to build a _merkledag_ node (in essence it is the entire tree represented by the root node) and adds it to the _blockstore_.
Within the function, a new `Adder` is created with the configured `Blockstore` and __DAG service__`. 

- **[`adder.AddAllAndPin(files)`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L403)** - *Entrypoint to the `Add` logic*
    encapsulates a lot of the underlying functionality that will be investigated in the following sections. 

    Our focus will be on the simplest case, a single file, handled by `Adder.addFile(file files.File)`. 

  - **[`adder.addFile(file files.File)`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L450)** - *Create the _DAG_ and add to `MFS`*

      The `addFile(file)` method takes the data and converts it into a __DAG__ tree and adds the root of the tree into the `MFS`.

      https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L508-L521

      There are two main methods to focus on -

      1. **[`adder.add(io.Reader)`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L115)** - *Create and return the **root** __DAG__ node*

          This method converts the input data (`io.Reader`) to a __DAG__ tree, by splitting the data into _chunks_ using the `Chunker` and organizing them in to a __DAG__ (with a *trickle* or *balanced* layout. See [balanced](https://github.com/ipfs/go-unixfs/blob/6b769632e7eb8fe8f302e3f96bf5569232e7a3ee/importer/balanced/builder.go) for more info). 

          The method returns the **root** `ipld.Node` of the __DAG__.

      2. **[`adder.addNode(ipld.Node, path)`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L366)** - *Add **root** __DAG__ node to the `MFS`*

          Now that we have the **root** node of the `DAG`, this needs to be added to the `MFS` file system. 
          Fetch (or create, if doesn't already exist) the `MFS` **root** using `mfsRoot()`. 

          > NOTE: The `MFS` **root** is an ephemeral root, created and destroyed solely for the `add` functionality.

          Assuming the directory already exists in the MFS file system, (if it doesn't exist it will be created using `mfs.Mkdir()`), the **root** __DAG__ node is added to the `MFS` File system using the `mfs.PutNode()` function.

          - **[MFS] [`PutNode(mfs.Root, path, ipld.Node)`](https://github.com/ipfs/go-mfs/blob/v0.1.18/ops.go#L86)** - *Insert node at path into given `MFS`*

              The `path` param is used to determine the `MFS Directory`, which is first looked up in the `MFS` using `lookupDir()` function. This is followed by adding the **root** __DAG__ node (`ipld.Node`) in to this `Directory` using `directory.AddChild()` method.

          - **[MFS] Add Child To `UnixFS`**
            - **[`directory.AddChild(filename, ipld.Node)`](https://github.com/ipfs/go-mfs/blob/v0.1.18/dir.go#L350)** - *Add **root** __DAG__ node under this directory*

                Within this method the node is added to the `Directory`'s __DAG service__ using the `dserv.Add()` method, followed by adding the **root** __DAG__ node with the given name, in the `directory.addUnixFSChild(directory.child{name, ipld.Node})` method.

            - **[MFS] [`directory.addUnixFSChild(child)`](https://github.com/ipfs/go-mfs/blob/v0.1.18/dir.go#L375)** - *Add child to inner UnixFS Directory*

                The node is then added as a child to the inner `UnixFS` directory using the `(BasicDirectory).AddChild()` method.

                > NOTE: This is not to be confused with the `directory.AddChild(filename, ipld.Node)`, as this operates on the `UnixFS` `BasicDirectory` object.

            - **[UnixFS] [`(BasicDirectory).AddChild(ctx, name, ipld.Node)`](https://github.com/ipfs/go-unixfs/blob/v1.1.16/io/directory.go#L137)** - *Add child to `BasicDirectory`*

                > IMPORTANT: It should be noted that the `BasicDirectory` object uses the `ProtoNode` type object which is an implementation of the `ipld.Node` interface, seen and used throughout this document. Ideally the `ipld.Node` should always be used, unless we need access to specific functions from `ProtoNode` (like `Copy()`) that are not available in the interface.

                This method first attempts to remove any old links (`ProtoNode.RemoveNodeLink(name)`) to the `ProtoNode` prior to adding a link to the newly added `ipld.Node`, using `ProtoNode.AddNodeLink(name, ipld.Node)`.

                - **[Merkledag] [`AddNodeLink()`](https://github.com/ipfs/go-merkledag/blob/v1.1.15/node.go#L99)**

                  The `AddNodeLink()` method is where an `ipld.Link` is created with the `ipld.Node`'s `CID` and size in the `ipld.MakeLink(ipld.Node)` method, and is then appended to the `ProtoNode`'s links in the `ProtoNode.AddRawLink(name)` method.

  - **[`adder.Finalize()`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L200)** - *Fetch and return the __DAG__ **root** from the `MFS` and `UnixFS` directory*

      The `Finalize` method returns the `ipld.Node` from the `UnixFS` `Directory`.

  - **[`adder.PinRoot()`](https://github.com/ipfs/go-ipfs/blob/v0.4.18/core/coreunix/add.go#L171)** - *Pin all files under the `MFS` **root***

    The whole process ends with `PinRoot` recursively pinning all the files under the `MFS` **root**

# Command Completion

Shell command completions can be generated by running one of the `ipfs commands completions`
sub-commands.

The simplest way to "eval" the completions logic:

```bash
> eval "$(ipfs commands completion bash)"
```

To install the completions permanently, they can be moved to
`/etc/bash_completion.d` or sourced from your `~/.bashrc` file.

## Fish

The fish shell is also supported:

The simplest way to use the completions logic:

```bash
> ipfs commands completion fish | source
```

To install the completions permanently, they can be moved to
`/etc/fish/completions` or `~/.config/fish/completions` or sourced from your `~/.config/fish/config.fish` file.

## ZSH

The zsh shell is also supported:

The simplest way to "eval" the completions logic:

```bash
> eval "$(ipfs commands completion zsh)"
```

To install the completions permanently, they can be moved to
`/etc/bash_completion.d` or sourced from your `~/.zshrc` file.


# The Kubo config file

The Kubo (go-ipfs) config file is a JSON document located at `$IPFS_PATH/config`. It
is read once at node instantiation, either for an offline command, or when
starting the daemon. Commands that execute on a running daemon do not read the
config file at runtime.

# Table of Contents

- [The Kubo config file](#the-kubo-config-file)
- [Table of Contents](#table-of-contents)
  - [Profiles](#profiles)
  - [Types](#types)
    - [`flag`](#flag)
    - [`priority`](#priority)
    - [`strings`](#strings)
    - [`duration`](#duration)
    - [`optionalInteger`](#optionalinteger)
    - [`optionalBytes`](#optionalbytes)
    - [`optionalString`](#optionalstring)
    - [`optionalDuration`](#optionalduration)
  - [`Addresses`](#addresses)
    - [`Addresses.API`](#addressesapi)
    - [`Addresses.Gateway`](#addressesgateway)
    - [`Addresses.Swarm`](#addressesswarm)
    - [`Addresses.Announce`](#addressesannounce)
    - [`Addresses.AppendAnnounce`](#addressesappendannounce)
    - [`Addresses.NoAnnounce`](#addressesnoannounce)
  - [`API`](#api)
    - [`API.HTTPHeaders`](#apihttpheaders)
  - [`AutoNAT`](#autonat)
    - [`AutoNAT.ServiceMode`](#autonatservicemode)
    - [`AutoNAT.Throttle`](#autonatthrottle)
    - [`AutoNAT.Throttle.GlobalLimit`](#autonatthrottlegloballimit)
    - [`AutoNAT.Throttle.PeerLimit`](#autonatthrottlepeerlimit)
    - [`AutoNAT.Throttle.Interval`](#autonatthrottleinterval)
  - [`Bootstrap`](#bootstrap)
  - [`Datastore`](#datastore)
    - [`Datastore.StorageMax`](#datastorestoragemax)
    - [`Datastore.StorageGCWatermark`](#datastorestoragegcwatermark)
    - [`Datastore.GCPeriod`](#datastoregcperiod)
    - [`Datastore.HashOnRead`](#datastorehashonread)
    - [`Datastore.BloomFilterSize`](#datastorebloomfiltersize)
    - [`Datastore.Spec`](#datastorespec)
  - [`Discovery`](#discovery)
    - [`Discovery.MDNS`](#discoverymdns)
      - [`Discovery.MDNS.Enabled`](#discoverymdnsenabled)
      - [`Discovery.MDNS.Interval`](#discoverymdnsinterval)
  - [`Experimental`](#experimental)
  - [`Gateway`](#gateway)
    - [`Gateway.NoFetch`](#gatewaynofetch)
    - [`Gateway.NoDNSLink`](#gatewaynodnslink)
    - [`Gateway.DeserializedResponses`](#gatewaydeserializedresponses)
    - [`Gateway.DisableHTMLErrors`](#gatewaydisablehtmlerrors)
    - [`Gateway.ExposeRoutingAPI`](#gatewayexposeroutingapi)
    - [`Gateway.HTTPHeaders`](#gatewayhttpheaders)
    - [`Gateway.RootRedirect`](#gatewayrootredirect)
    - [`Gateway.FastDirIndexThreshold`](#gatewayfastdirindexthreshold)
    - [`Gateway.Writable`](#gatewaywritable)
    - [`Gateway.PathPrefixes`](#gatewaypathprefixes)
    - [`Gateway.PublicGateways`](#gatewaypublicgateways)
      - [`Gateway.PublicGateways: Paths`](#gatewaypublicgateways-paths)
      - [`Gateway.PublicGateways: UseSubdomains`](#gatewaypublicgateways-usesubdomains)
      - [`Gateway.PublicGateways: NoDNSLink`](#gatewaypublicgateways-nodnslink)
      - [`Gateway.PublicGateways: InlineDNSLink`](#gatewaypublicgateways-inlinednslink)
      - [`Gateway.PublicGateways: DeserializedResponses`](#gatewaypublicgateways-deserializedresponses)
      - [Implicit defaults of `Gateway.PublicGateways`](#implicit-defaults-of-gatewaypublicgateways)
    - [`Gateway` recipes](#gateway-recipes)
  - [`Identity`](#identity)
    - [`Identity.PeerID`](#identitypeerid)
    - [`Identity.PrivKey`](#identityprivkey)
  - [`Internal`](#internal)
    - [`Internal.Bitswap`](#internalbitswap)
      - [`Internal.Bitswap.TaskWorkerCount`](#internalbitswaptaskworkercount)
      - [`Internal.Bitswap.EngineBlockstoreWorkerCount`](#internalbitswapengineblockstoreworkercount)
      - [`Internal.Bitswap.EngineTaskWorkerCount`](#internalbitswapenginetaskworkercount)
      - [`Internal.Bitswap.MaxOutstandingBytesPerPeer`](#internalbitswapmaxoutstandingbytesperpeer)
    - [`Internal.Bitswap.ProviderSearchDelay`](#internalbitswapprovidersearchdelay)
    - [`Internal.UnixFSShardingSizeThreshold`](#internalunixfsshardingsizethreshold)
  - [`Ipns`](#ipns)
    - [`Ipns.RepublishPeriod`](#ipnsrepublishperiod)
    - [`Ipns.RecordLifetime`](#ipnsrecordlifetime)
    - [`Ipns.ResolveCacheSize`](#ipnsresolvecachesize)
    - [`Ipns.UsePubsub`](#ipnsusepubsub)
  - [`Migration`](#migration)
    - [`Migration.DownloadSources`](#migrationdownloadsources)
    - [`Migration.Keep`](#migrationkeep)
  - [`Mounts`](#mounts)
    - [`Mounts.IPFS`](#mountsipfs)
    - [`Mounts.IPNS`](#mountsipns)
    - [`Mounts.FuseAllowOther`](#mountsfuseallowother)
  - [`Pinning`](#pinning)
    - [`Pinning.RemoteServices`](#pinningremoteservices)
      - [`Pinning.RemoteServices: API`](#pinningremoteservices-api)
        - [`Pinning.RemoteServices: API.Endpoint`](#pinningremoteservices-apiendpoint)
        - [`Pinning.RemoteServices: API.Key`](#pinningremoteservices-apikey)
      - [`Pinning.RemoteServices: Policies`](#pinningremoteservices-policies)
        - [`Pinning.RemoteServices: Policies.MFS`](#pinningremoteservices-policiesmfs)
          - [`Pinning.RemoteServices: Policies.MFS.Enabled`](#pinningremoteservices-policiesmfsenabled)
          - [`Pinning.RemoteServices: Policies.MFS.PinName`](#pinningremoteservices-policiesmfspinname)
          - [`Pinning.RemoteServices: Policies.MFS.RepinInterval`](#pinningremoteservices-policiesmfsrepininterval)
  - [`Pubsub`](#pubsub)
    - [`Pubsub.Enabled`](#pubsubenabled)
    - [`Pubsub.Router`](#pubsubrouter)
    - [`Pubsub.DisableSigning`](#pubsubdisablesigning)
    - [`Pubsub.SeenMessagesTTL`](#pubsubseenmessagesttl)
    - [`Pubsub.SeenMessagesStrategy`](#pubsubseenmessagesstrategy)
  - [`Peering`](#peering)
    - [`Peering.Peers`](#peeringpeers)
  - [`Reprovider`](#reprovider)
    - [`Reprovider.Interval`](#reproviderinterval)
    - [`Reprovider.Strategy`](#reproviderstrategy)
  - [`Routing`](#routing)
    - [`Routing.Type`](#routingtype)
    - [`Routing.AcceleratedDHTClient`](#routingaccelerateddhtclient)
    - [`Routing.Routers`](#routingrouters)
      - [`Routing.Routers: Type`](#routingrouters-type)
      - [`Routing.Routers: Parameters`](#routingrouters-parameters)
    - [`Routing: Methods`](#routing-methods)
  - [`Swarm`](#swarm)
    - [`Swarm.AddrFilters`](#swarmaddrfilters)
    - [`Swarm.DisableBandwidthMetrics`](#swarmdisablebandwidthmetrics)
    - [`Swarm.DisableNatPortMap`](#swarmdisablenatportmap)
    - [`Swarm.EnableHolePunching`](#swarmenableholepunching)
    - [`Swarm.EnableAutoRelay`](#swarmenableautorelay)
    - [`Swarm.RelayClient`](#swarmrelayclient)
      - [`Swarm.RelayClient.Enabled`](#swarmrelayclientenabled)
      - [`Swarm.RelayClient.StaticRelays`](#swarmrelayclientstaticrelays)
    - [`Swarm.RelayService`](#swarmrelayservice)
      - [`Swarm.RelayService.Enabled`](#swarmrelayserviceenabled)
      - [`Swarm.RelayService.Limit`](#swarmrelayservicelimit)
        - [`Swarm.RelayService.ConnectionDurationLimit`](#swarmrelayserviceconnectiondurationlimit)
        - [`Swarm.RelayService.ConnectionDataLimit`](#swarmrelayserviceconnectiondatalimit)
      - [`Swarm.RelayService.ReservationTTL`](#swarmrelayservicereservationttl)
      - [`Swarm.RelayService.MaxReservations`](#swarmrelayservicemaxreservations)
      - [`Swarm.RelayService.MaxCircuits`](#swarmrelayservicemaxcircuits)
      - [`Swarm.RelayService.BufferSize`](#swarmrelayservicebuffersize)
      - [`Swarm.RelayService.MaxReservationsPerPeer`](#swarmrelayservicemaxreservationsperpeer)
      - [`Swarm.RelayService.MaxReservationsPerIP`](#swarmrelayservicemaxreservationsperip)
      - [`Swarm.RelayService.MaxReservationsPerASN`](#swarmrelayservicemaxreservationsperasn)
    - [`Swarm.EnableRelayHop`](#swarmenablerelayhop)
    - [`Swarm.DisableRelay`](#swarmdisablerelay)
    - [`Swarm.EnableAutoNATService`](#swarmenableautonatservice)
    - [`Swarm.ConnMgr`](#swarmconnmgr)
      - [`Swarm.ConnMgr.Type`](#swarmconnmgrtype)
      - [Basic Connection Manager](#basic-connection-manager)
        - [`Swarm.ConnMgr.LowWater`](#swarmconnmgrlowwater)
        - [`Swarm.ConnMgr.HighWater`](#swarmconnmgrhighwater)
        - [`Swarm.ConnMgr.GracePeriod`](#swarmconnmgrgraceperiod)
    - [`Swarm.ResourceMgr`](#swarmresourcemgr)
      - [`Swarm.ResourceMgr.Enabled`](#swarmresourcemgrenabled)
      - [`Swarm.ResourceMgr.MaxMemory`](#swarmresourcemgrmaxmemory)
      - [`Swarm.ResourceMgr.MaxFileDescriptors`](#swarmresourcemgrmaxfiledescriptors)
      - [`Swarm.ResourceMgr.Allowlist`](#swarmresourcemgrallowlist)
    - [`Swarm.Transports`](#swarmtransports)
    - [`Swarm.Transports.Network`](#swarmtransportsnetwork)
      - [`Swarm.Transports.Network.TCP`](#swarmtransportsnetworktcp)
      - [`Swarm.Transports.Network.Websocket`](#swarmtransportsnetworkwebsocket)
      - [`Swarm.Transports.Network.QUIC`](#swarmtransportsnetworkquic)
      - [`Swarm.Transports.Network.Relay`](#swarmtransportsnetworkrelay)
      - [`Swarm.Transports.Network.WebTransport`](#swarmtransportsnetworkwebtransport)
    - [`Swarm.Transports.Security`](#swarmtransportssecurity)
      - [`Swarm.Transports.Security.TLS`](#swarmtransportssecuritytls)
      - [`Swarm.Transports.Security.SECIO`](#swarmtransportssecuritysecio)
      - [`Swarm.Transports.Security.Noise`](#swarmtransportssecuritynoise)
    - [`Swarm.Transports.Multiplexers`](#swarmtransportsmultiplexers)
    - [`Swarm.Transports.Multiplexers.Yamux`](#swarmtransportsmultiplexersyamux)
    - [`Swarm.Transports.Multiplexers.Mplex`](#swarmtransportsmultiplexersmplex)
  - [`DNS`](#dns)
    - [`DNS.Resolvers`](#dnsresolvers)
    - [`DNS.MaxCacheTTL`](#dnsmaxcachettl)

## Profiles

Configuration profiles allow to tweak configuration quickly. Profiles can be
applied with the `--profile` flag to `ipfs init` or with the `ipfs config profile
apply` command. When a profile is applied a backup of the configuration file
will be created in `$IPFS_PATH`.

The available configuration profiles are listed below. You can also find them
documented in `ipfs config profile --help`.

- `server`

  Disables local host discovery, recommended when
  running IPFS on machines with public IPv4 addresses.

- `randomports`

  Use a random port number for the incoming swarm connections.

- `default-datastore`

  Configures the node to use the default datastore (flatfs).

  Read the "flatfs" profile description for more information on this datastore.

  This profile may only be applied when first initializing the node.

- `local-discovery`

  Enables local discovery (enabled by default). Useful to re-enable local discovery after it's
  disabled by another profile (e.g., the server profile).

- `test`

  Reduces external interference of IPFS daemon, this
  is useful when using the daemon in test environments.

- `default-networking`

  Restores default network settings.
  Inverse profile of the test profile.

- `flatfs`

  Configures the node to use the flatfs datastore. Flatfs is the default datastore.

  This is the most battle-tested and reliable datastore.
  You should use this datastore if:

  - You need a very simple and very reliable datastore, and you trust your
    filesystem. This datastore stores each block as a separate file in the
    underlying filesystem so it's unlikely to lose data unless there's an issue
    with the underlying file system.
  - You need to run garbage collection in a way that reclaims free space as soon as possible.
  - You want to minimize memory usage.
  - You are ok with the default speed of data import, or prefer to use `--nocopy`.

  This profile may only be applied when first initializing the node.


- `badgerds`

  Configures the node to use the experimental badger datastore. Keep in mind that this **uses an outdated badger 1.x**.

  Use this datastore if some aspects of performance,
  especially the speed of adding many gigabytes of files, are critical. However, be aware that:

  - This datastore will not properly reclaim space when your datastore is
    smaller than several gigabytes. If you run IPFS with `--enable-gc`, you plan on storing very little data in
    your IPFS node, and disk usage is more critical than performance, consider using
    `flatfs`.
  - This datastore uses up to several gigabytes of memory.
  - Good for medium-size datastores, but may run into performance issues if your dataset is bigger than a terabyte.
  - The current implementation is based on old badger 1.x which is no longer supported by the upstream team.

  This profile may only be applied when first initializing the node.

- `lowpower`

  Reduces daemon overhead on the system. Affects node
  functionality - performance of content discovery and data
  fetching may be degraded. Local data won't be announced on routing systems like Amino DHT.

  - `Swarm.ConnMgr` set to maintain minimum number of p2p connections at a time.
  - Disables [`Reprovider`](#reprovider) service → no CID will be announced on Amino DHT and other routing systems(!)
  - Disables AutoNAT.

  Use this profile with caution.

## Types

This document refers to the standard JSON types (e.g., `null`, `string`,
`number`, etc.), as well as a few custom types, described below.

### `flag`

Flags allow enabling and disabling features. However, unlike simple booleans,
they can also be `null` (or omitted) to indicate that the default value should
be chosen. This makes it easier for Kubo to change the defaults in the
future unless the user _explicitly_ sets the flag to either `true` (enabled) or
`false` (disabled). Flags have three possible states:

- `null` or missing (apply the default value).
- `true` (enabled)
- `false` (disabled)

### `priority`

Priorities allow specifying the priority of a feature/protocol and disabling the
feature/protocol. Priorities can take one of the following values:

- `null`/missing (apply the default priority, same as with flags)
- `false` (disabled)
- `1 - 2^63` (priority, lower is preferred)

### `strings`

Strings is a special type for conveniently specifying a single string, an array
of strings, or null:

- `null`
- `"a single string"`
- `["an", "array", "of", "strings"]`

### `duration`

Duration is a type for describing lengths of time, using the same format go
does (e.g, `"1d2h4m40.01s"`).

### `optionalInteger`

Optional integers allow specifying some numerical value which has
an implicit default when missing from the config file:

- `null`/missing will apply the default value defined in Kubo sources (`.WithDefault(value)`)
- an integer between `-2^63` and `2^63-1` (i.e. `-9223372036854775808` to `9223372036854775807`)

### `optionalBytes`

Optional Bytes allow specifying some number of bytes which has
an implicit default when missing from the config file:

- `null`/missing (apply the default value defined in Kubo sources)
- a string value indicating the number of bytes, including human readable representations:
  - [SI sizes](https://en.wikipedia.org/wiki/Metric_prefix#List_of_SI_prefixes) (metric units, powers of 1000), e.g. `1B`, `2kB`, `3MB`, `4GB`, `5TB`, …)
  - [IEC sizes](https://en.wikipedia.org/wiki/Binary_prefix#IEC_prefixes) (binary units, powers of 1024), e.g. `1B`, `2KiB`, `3MiB`, `4GiB`, `5TiB`, …)

### `optionalString`

Optional strings allow specifying some string value which has
an implicit default when missing from the config file:

- `null`/missing will apply the default value defined in Kubo sources (`.WithDefault("value")`)
- a string

### `optionalDuration`

Optional durations allow specifying some duration value which has
an implicit default when missing from the config file:

- `null`/missing will apply the default value defined in Kubo sources (`.WithDefault("1h2m3s")`)
- a string with a valid [go duration](#duration)  (e.g, `"1d2h4m40.01s"`).

## `Addresses`

Contains information about various listener addresses to be used by this node.

### `Addresses.API`

Multiaddr or array of multiaddrs describing the address to serve the local HTTP
API on.

Supported Transports:

* tcp/ip{4,6} - `/ipN/.../tcp/...`
* unix - `/unix/path/to/socket`

Default: `/ip4/127.0.0.1/tcp/5001`

Type: `strings` (multiaddrs)

### `Addresses.Gateway`

Multiaddr or array of multiaddrs describing the address to serve the local
gateway on.

Supported Transports:

* tcp/ip{4,6} - `/ipN/.../tcp/...`
* unix - `/unix/path/to/socket`

Default: `/ip4/127.0.0.1/tcp/8080`

Type: `strings` (multiaddrs)

### `Addresses.Swarm`

An array of multiaddrs describing which addresses to listen on for p2p swarm
connections.

Supported Transports:

* tcp/ip{4,6} - `/ipN/.../tcp/...`
* websocket - `/ipN/.../tcp/.../ws`
* quicv1 (RFC9000) - `/ipN/.../udp/.../quic-v1` - can share the same two tuple with `/quic-v1/webtransport`
* webtransport `/ipN/.../udp/.../quic-v1/webtransport` - can share the same two tuple with `/quic-v1`

Note that quic (Draft-29) used to be supported with the format `/ipN/.../udp/.../quic`, but has since been [removed](https://github.com/libp2p/go-libp2p/releases/tag/v0.30.0).

Default:
```json
[
  "/ip4/0.0.0.0/tcp/4001",
  "/ip6/::/tcp/4001",
  "/ip4/0.0.0.0/udp/4001/quic-v1",
  "/ip4/0.0.0.0/udp/4001/quic-v1/webtransport",
  "/ip6/::/udp/4001/quic-v1",
  "/ip6/::/udp/4001/quic-v1/webtransport"
]
```

Type: `array[string]` (multiaddrs)

### `Addresses.Announce`

If non-empty, this array specifies the swarm addresses to announce to the
network. If empty, the daemon will announce inferred swarm addresses.

Default: `[]`

Type: `array[string]` (multiaddrs)

### `Addresses.AppendAnnounce`

Similar to [`Addresses.Announce`](#addressesannounce) except this doesn't
override inferred swarm addresses if non-empty.

Default: `[]`

Type: `array[string]` (multiaddrs)

### `Addresses.NoAnnounce`

An array of swarm addresses not to announce to the network.
Takes precedence over `Addresses.Announce` and `Addresses.AppendAnnounce`.

Default: `[]`

Type: `array[string]` (multiaddrs)

## `API`
Contains information used by the API gateway.

### `API.HTTPHeaders`
Map of HTTP headers to set on responses from the API HTTP server.

Example:
```json
{
	"Foo": ["bar"]
}
```

Default: `null`

Type: `object[string -> array[string]]` (header names -> array of header values)

## `AutoNAT`

Contains the configuration options for the AutoNAT service. The AutoNAT service
helps other nodes on the network determine if they're publicly reachable from
the rest of the internet.

### `AutoNAT.ServiceMode`

When unset (default), the AutoNAT service defaults to _enabled_. Otherwise, this
field can take one of two values:

* "enabled" - Enable the service (unless the node determines that it, itself,
  isn't reachable by the public internet).
* "disabled" - Disable the service.

Additional modes may be added in the future.

Type: `string` (one of `"enabled"` or `"disabled"`)

### `AutoNAT.Throttle`

When set, this option configures the AutoNAT services throttling behavior. By
default, Kubo will rate-limit the number of NAT checks performed for other
nodes to 30 per minute, and 3 per peer.

### `AutoNAT.Throttle.GlobalLimit`

Configures how many AutoNAT requests to service per `AutoNAT.Throttle.Interval`.

Default: 30

Type: `integer` (non-negative, `0` means unlimited)

### `AutoNAT.Throttle.PeerLimit`

Configures how many AutoNAT requests per-peer to service per `AutoNAT.Throttle.Interval`.

Default: 3

Type: `integer` (non-negative, `0` means unlimited)

### `AutoNAT.Throttle.Interval`

Configures the interval for the above limits.

Default: 1 Minute

Type: `duration` (when `0`/unset, the default value is used)

## `Bootstrap`

Bootstrap is an array of multiaddrs of trusted nodes that your node connects to, to fetch other nodes of the network on startup.

Default: The ipfs.io bootstrap nodes

Type: `array[string]` (multiaddrs)

## `Datastore`

Contains information related to the construction and operation of the on-disk
storage system.

### `Datastore.StorageMax`

A soft upper limit for the size of the ipfs repository's datastore. With `StorageGCWatermark`,
is used to calculate whether to trigger a gc run (only if `--enable-gc` flag is set).

Default: `"10GB"`

Type: `string` (size)

### `Datastore.StorageGCWatermark`

The percentage of the `StorageMax` value at which a garbage collection will be
triggered automatically if the daemon was run with automatic gc enabled (that
option defaults to false currently).

Default: `90`

Type: `integer` (0-100%)

### `Datastore.GCPeriod`

A time duration specifying how frequently to run a garbage collection. Only used
if automatic gc is enabled.

Default: `1h`

Type: `duration` (an empty string means the default value)

### `Datastore.HashOnRead`

A boolean value. If set to true, all block reads from the disk will be hashed and
verified. This will cause increased CPU utilization.

Default: `false`

Type: `bool`

### `Datastore.BloomFilterSize`

A number representing the size in bytes of the blockstore's [bloom
filter](https://en.wikipedia.org/wiki/Bloom_filter). A value of zero represents
the feature is disabled.

This site generates useful graphs for various bloom filter values:
<https://hur.st/bloomfilter/?n=1e6&p=0.01&m=&k=7> You may use it to find a
preferred optimal value, where `m` is `BloomFilterSize` in bits. Remember to
convert the value `m` from bits, into bytes for use as `BloomFilterSize` in the
config file. For example, for 1,000,000 blocks, expecting a 1% false-positive
rate, you'd end up with a filter size of 9592955 bits, so for `BloomFilterSize`
we'd want to use 1199120 bytes. As of writing, [7 hash
functions](https://github.com/ipfs/go-ipfs-blockstore/blob/547442836ade055cc114b562a3cc193d4e57c884/caching.go#L22)
are used, so the constant `k` is 7 in the formula.

Default: `0` (disabled)

Type: `integer` (non-negative, bytes)

### `Datastore.Spec`

Spec defines the structure of the ipfs datastore. It is a composable structure,
where each datastore is represented by a json object. Datastores can wrap other
datastores to provide extra functionality (eg metrics, logging, or caching).

This can be changed manually, however, if you make any changes that require a
different on-disk structure, you will need to run the [ipfs-ds-convert
tool](https://github.com/ipfs/ipfs-ds-convert) to migrate data into the new
structures.

For more information on possible values for this configuration option, see
[docs/datastores.md](datastores.md)

Default:
```
{
  "mounts": [
	{
	  "child": {
		"path": "blocks",
		"shardFunc": "/repo/flatfs/shard/v1/next-to-last/2",
		"sync": true,
		"type": "flatfs"
	  },
	  "mountpoint": "/blocks",
	  "prefix": "flatfs.datastore",
	  "type": "measure"
	},
	{
	  "child": {
		"compression": "none",
		"path": "datastore",
		"type": "levelds"
	  },
	  "mountpoint": "/",
	  "prefix": "leveldb.datastore",
	  "type": "measure"
	}
  ],
  "type": "mount"
}
```

Type: `object`

## `Discovery`

Contains options for configuring IPFS node discovery mechanisms.

### `Discovery.MDNS`

Options for [ZeroConf](https://github.com/libp2p/zeroconf#readme) Multicast DNS-SD peer discovery.

#### `Discovery.MDNS.Enabled`

A boolean value for whether or not Multicast DNS-SD should be active.

Default: `true`

Type: `bool`

#### `Discovery.MDNS.Interval`

**REMOVED:**  this is not configurable anymore
in the [new mDNS implementation](https://github.com/libp2p/zeroconf#readme).

## `Experimental`

Toggle and configure experimental features of Kubo. Experimental features are listed [here](./experimental-features.md).

## `Gateway`

Options for the HTTP gateway.

### `Gateway.NoFetch`

When set to true, the gateway will only serve content already in the local repo
and will not fetch files from the network.

Default: `false`

Type: `bool`

### `Gateway.NoDNSLink`

A boolean to configure whether DNSLink lookup for value in `Host` HTTP header
should be performed.  If DNSLink is present, the content path stored in the DNS TXT
record becomes the `/` and the respective payload is returned to the client.

Default: `false`

Type: `bool`

### `Gateway.DeserializedResponses`

An optional flag to explicitly configure whether this gateway responds to deserialized
requests, or not. By default, it is enabled. When disabling this option, the gateway
operates as a Trustless Gateway only: https://specs.ipfs.tech/http-gateways/trustless-gateway/.

Default: `true`

Type: `flag`

### `Gateway.DisableHTMLErrors`

An optional flag to disable the pretty HTML error pages of the gateway. Instead,
a `text/plain` page will be returned with the raw error message from Kubo.

It is useful for whitelabel or middleware deployments that wish to avoid
`text/html` responses with IPFS branding and links on error pages in browsers.

Default: `false`

Type: `flag`

### `Gateway.ExposeRoutingAPI`

An optional flag to expose Kubo `Routing` system on the gateway port as a [Routing
V1](https://specs.ipfs.tech/routing/routing-v1/) endpoint.  This only affects your
local gateway, at `127.0.0.1`.

This endpoint can be used by other Kubo instance, as illustrated in [`delegated_routing_v1_http_proxy_test.go`](https://github.com/ipfs/kubo/blob/master/test/cli/delegated_routing_v1_http_proxy_test.go).

Default: `false`

Type: `flag`

### `Gateway.HTTPHeaders`

Headers to set on gateway responses.

Default: `{}` + implicit CORS headers from `boxo/gateway#AddAccessControlHeaders` and [ipfs/specs#423](https://github.com/ipfs/specs/issues/423)

Type: `object[string -> array[string]]`

### `Gateway.RootRedirect`

A url to redirect requests for `/` to.

Default: `""`

Type: `string` (url)

### `Gateway.FastDirIndexThreshold`

**REMOVED**: this option is [no longer necessary](https://github.com/ipfs/kubo/pull/9481). Ignored since  [Kubo 0.18](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.18.md).

### `Gateway.Writable`

**REMOVED**: this option no longer available as of [Kubo 0.20](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.20.md).

We are working on developing a modern replacement. To support our efforts, please leave a comment describing your use case in [ipfs/specs#375](https://github.com/ipfs/specs/issues/375).

### `Gateway.PathPrefixes`

**REMOVED:** see [go-ipfs#7702](https://github.com/ipfs/go-ipfs/issues/7702)

### `Gateway.PublicGateways`

`PublicGateways` is a dictionary for defining gateway behavior on specified hostnames.

Hostnames can optionally be defined with one or more wildcards.

Examples:
- `*.example.com` will match requests to `http://foo.example.com/ipfs/*` or `http://{cid}.ipfs.bar.example.com/*`.
- `foo-*.example.com` will match requests to `http://foo-bar.example.com/ipfs/*` or `http://{cid}.ipfs.foo-xyz.example.com/*`.

#### `Gateway.PublicGateways: Paths`

An array of paths that should be exposed on the hostname.

Example:
```json
{
  "Gateway": {
    "PublicGateways": {
      "example.com": {
        "Paths": ["/ipfs", "/ipns"],
      }
    }
  }
}
```

Above enables `http://example.com/ipfs/*` and `http://example.com/ipns/*` but not `http://example.com/api/*`

Default: `[]`

Type: `array[string]`

#### `Gateway.PublicGateways: UseSubdomains`

A boolean to configure whether the gateway at the hostname provides [Origin isolation](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
between content roots.

- `true` - enables [subdomain gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway) at `http://*.{hostname}/`
    - **Requires whitelist:** make sure respective `Paths` are set.
      For example, `Paths: ["/ipfs", "/ipns"]` are required for `http://{cid}.ipfs.{hostname}` and `http://{foo}.ipns.{hostname}` to work:
        ```json
        "Gateway": {
            "PublicGateways": {
                "dweb.link": {
                    "UseSubdomains": true,
                    "Paths": ["/ipfs", "/ipns"],
                }
            }
        }
        ```
    - **Backward-compatible:** requests for content paths such as `http://{hostname}/ipfs/{cid}` produce redirect to `http://{cid}.ipfs.{hostname}`
    - **API:** if `/api` is on the `Paths` whitelist, `http://{hostname}/api/{cmd}` produces redirect to `http://api.{hostname}/api/{cmd}`

- `false` - enables [path gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#path-gateway) at `http://{hostname}/*`
  - Example:
    ```json
    "Gateway": {
        "PublicGateways": {
            "ipfs.io": {
                "UseSubdomains": false,
                "Paths": ["/ipfs", "/ipns", "/api"],
            }
        }
    }
    ```

Default: `false`

Type: `bool`

#### `Gateway.PublicGateways: NoDNSLink`

A boolean to configure whether DNSLink for hostname present in `Host`
HTTP header should be resolved. Overrides global setting.
If `Paths` are defined, they take priority over DNSLink.

Default: `false` (DNSLink lookup enabled by default for every defined hostname)

Type: `bool`

#### `Gateway.PublicGateways: InlineDNSLink`

An optional flag to explicitly configure whether subdomain gateway's redirects
(enabled by `UseSubdomains: true`) should always inline a DNSLink name (FQDN)
into a single DNS label:

```
//example.com/ipns/example.net → HTTP 301 → //example-net.ipns.example.com
```

DNSLink name inlining allows for HTTPS on public subdomain gateways with single
label wildcard TLS certs (also enabled when passing `X-Forwarded-Proto: https`),
and provides disjoint Origin per root CID when special rules like
https://publicsuffix.org, or a custom localhost logic in browsers like Brave
has to be applied.

Default: `false`

Type: `flag`

#### `Gateway.PublicGateways: DeserializedResponses`

An optional flag to explicitly configure whether this gateway responds to deserialized
requests, or not. By default, it is enabled. When disabling this option, the gateway
operates as a Trustless Gateway only: https://specs.ipfs.tech/http-gateways/trustless-gateway/.

Default: same as global `Gateway.DeserializedResponses`

Type: `flag`

#### Implicit defaults of `Gateway.PublicGateways`

Default entries for `localhost` hostname and loopback IPs are always present.
If additional config is provided for those hostnames, it will be merged on top of implicit values:
```json
{
  "Gateway": {
    "PublicGateways": {
      "localhost": {
        "Paths": ["/ipfs", "/ipns"],
        "UseSubdomains": true
      }
    }
  }
}
```

It is also possible to remove a default by setting it to `null`.

For example, to disable subdomain gateway on `localhost`
and make that hostname act the same as `127.0.0.1`:

```console
$ ipfs config --json Gateway.PublicGateways '{"localhost": null }'
```

### `Gateway` recipes

Below is a list of the most common public gateway setups.

* Public [subdomain gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway) at `http://{cid}.ipfs.dweb.link` (each content root gets its own Origin)
   ```console
   $ ipfs config --json Gateway.PublicGateways '{
       "dweb.link": {
         "UseSubdomains": true,
         "Paths": ["/ipfs", "/ipns"]
       }
     }'
   ```
   - **Backward-compatible:** this feature enables automatic redirects from content paths to subdomains:

     `http://dweb.link/ipfs/{cid}` → `http://{cid}.ipfs.dweb.link`

   - **X-Forwarded-Proto:** if you run Kubo behind a reverse proxy that provides TLS, make it add a `X-Forwarded-Proto: https` HTTP header to ensure users are redirected to `https://`, not `http://`. It will also ensure DNSLink names are inlined to fit in a single DNS label, so they work fine with a wildcart TLS cert ([details](https://github.com/ipfs/in-web-browsers/issues/169)). The NGINX directive is `proxy_set_header X-Forwarded-Proto "https";`.:

     `http://dweb.link/ipfs/{cid}` → `https://{cid}.ipfs.dweb.link`

     `http://dweb.link/ipns/your-dnslink.site.example.com` → `https://your--dnslink-site-example-com.ipfs.dweb.link`

   - **X-Forwarded-Host:** we also support `X-Forwarded-Host: example.com` if you want to override subdomain gateway host from the original request:

     `http://dweb.link/ipfs/{cid}` → `http://{cid}.ipfs.example.com`


* Public [path gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#path-gateway) at `http://ipfs.io/ipfs/{cid}` (no Origin separation)
   ```console
   $ ipfs config --json Gateway.PublicGateways '{
       "ipfs.io": {
         "UseSubdomains": false,
         "Paths": ["/ipfs", "/ipns", "/api"]
       }
     }'
   ```

* Public [DNSLink](https://dnslink.io/) gateway resolving every hostname passed in `Host` header.
  ```console
  $ ipfs config --json Gateway.NoDNSLink false
  ```
  * Note that `NoDNSLink: false` is the default (it works out of the box unless set to `true` manually)

* Hardened, site-specific [DNSLink gateway](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#dnslink-gateway).

  Disable fetching of remote data (`NoFetch: true`) and resolving DNSLink at unknown hostnames (`NoDNSLink: true`).
  Then, enable DNSLink gateway only for the specific hostname (for which data
  is already present on the node), without exposing any content-addressing `Paths`:

   ```console
   $ ipfs config --json Gateway.NoFetch true
   $ ipfs config --json Gateway.NoDNSLink true
   $ ipfs config --json Gateway.PublicGateways '{
       "en.wikipedia-on-ipfs.org": {
         "NoDNSLink": false,
         "Paths": []
       }
     }'
   ```

## `Identity`

### `Identity.PeerID`

The unique PKI identity label for this configs peer. Set on init and never read,
it's merely here for convenience. Ipfs will always generate the peerID from its
keypair at runtime.

Type: `string` (peer ID)

### `Identity.PrivKey`

The base64 encoded protobuf describing (and containing) the node's private key.

Type: `string` (base64 encoded)

## `Internal`

This section includes internal knobs for various subsystems to allow advanced users with big or private infrastructures to fine-tune some behaviors without the need to recompile Kubo.

**Be aware that making informed change here requires in-depth knowledge and most users should leave these untouched. All knobs listed here are subject to breaking changes between versions.**

### `Internal.Bitswap`

`Internal.Bitswap` contains knobs for tuning bitswap resource utilization.
The knobs (below) document how their value should related to each other.
Whether their values should be raised or lowered should be determined
based on the metrics `ipfs_bitswap_active_tasks`, `ipfs_bitswap_pending_tasks`,
`ipfs_bitswap_pending_block_tasks` and `ipfs_bitswap_active_block_tasks`
reported by bitswap.

These metrics can be accessed as the Prometheus endpoint at `{Addresses.API}/debug/metrics/prometheus` (default: `http://127.0.0.1:5001/debug/metrics/prometheus`)

The value of `ipfs_bitswap_active_tasks` is capped by `EngineTaskWorkerCount`.

The value of `ipfs_bitswap_pending_tasks` is generally capped by the knobs below,
however its exact maximum value is hard to predict as it depends on task sizes
as well as number of requesting peers. However, as a rule of thumb,
during healthy operation this value should oscillate around a "typical" low value
(without hitting a plateau continuously).

If `ipfs_bitswap_pending_tasks` is growing while `ipfs_bitswap_active_tasks` is at its maximum then
the node has reached its resource limits and new requests are unable to be processed as quickly as they are coming in.
Raising resource limits (using the knobs below) could help, assuming the hardware can support the new limits.

The value of `ipfs_bitswap_active_block_tasks` is capped by `EngineBlockstoreWorkerCount`.

The value of `ipfs_bitswap_pending_block_tasks` is indirectly capped by `ipfs_bitswap_active_tasks`, but can be hard to
predict as it depends on the number of blocks involved in a peer task which can vary.

If the value of `ipfs_bitswap_pending_block_tasks` is observed to grow,
while `ipfs_bitswap_active_block_tasks` is at its maximum, there is indication that the number of
available block tasks is creating a bottleneck (either due to high-latency block operations,
or due to high number of block operations per bitswap peer task).
In such cases, try increasing the `EngineBlockstoreWorkerCount`.
If this adjustment still does not increase the throughput of the node, there might
be hardware limitations like I/O or CPU.

#### `Internal.Bitswap.TaskWorkerCount`

Number of threads (goroutines) sending outgoing messages.
Throttles the number of concurrent send operations.

Type: `optionalInteger` (thread count, `null` means default which is 8)

#### `Internal.Bitswap.EngineBlockstoreWorkerCount`

Number of threads for blockstore operations.
Used to throttle the number of concurrent requests to the block store.
The optimal value can be informed by the metrics `ipfs_bitswap_pending_block_tasks` and `ipfs_bitswap_active_block_tasks`.
This would be a number that depends on your hardware (I/O and CPU).

Type: `optionalInteger` (thread count, `null` means default which is 128)

#### `Internal.Bitswap.EngineTaskWorkerCount`

Number of worker threads used for preparing and packaging responses before they are sent out.
This number should generally be equal to `TaskWorkerCount`.

Type: `optionalInteger` (thread count, `null` means default which is 8)

#### `Internal.Bitswap.MaxOutstandingBytesPerPeer`

Maximum number of bytes (across all tasks) pending to be processed and sent to any individual peer.
This number controls fairness and can vary from 250Kb (very fair) to 10Mb (less fair, with more work
dedicated to peers who ask for more). Values below 250Kb could cause thrashing.
Values above 10Mb open the potential for aggressively-wanting peers to consume all resources and
deteriorate the quality provided to less aggressively-wanting peers.

Type: `optionalInteger` (byte count, `null` means default which is 1MB)

### `Internal.Bitswap.ProviderSearchDelay`

This parameter determines how long to wait before looking for providers outside of bitswap.
Other routing systems like the Amino DHT are able to provide results in less than a second, so lowering
this number will allow faster peers lookups in some cases.

Type: `optionalDuration` (`null` means default which is 1s)

### `Internal.UnixFSShardingSizeThreshold`

The sharding threshold used internally to decide whether a UnixFS directory should be sharded or not.
This value is not strictly related to the size of the UnixFS directory block and any increases in
the threshold should come with being careful that block sizes stay under 2MiB in order for them to be
reliably transferable through the networking stack (IPFS peers on the public swarm tend to ignore requests for blocks bigger than 2MiB).

Decreasing this value to 1B is functionally equivalent to the previous experimental sharding option to
shard all directories.

Type: `optionalBytes` (`null` means default which is 256KiB)

## `Ipns`

### `Ipns.RepublishPeriod`

A time duration specifying how frequently to republish ipns records to ensure
they stay fresh on the network.

Default: 4 hours.

Type: `interval` or an empty string for the default.

### `Ipns.RecordLifetime`

A time duration specifying the value to set on ipns records for their validity
lifetime.

Default: 24 hours.

Type: `interval` or an empty string for the default.

### `Ipns.ResolveCacheSize`

The number of entries to store in an LRU cache of resolved ipns entries. Entries
will be kept cached until their lifetime is expired.

Default: `128`

Type: `integer` (non-negative, 0 means the default)

### `Ipns.UsePubsub`

Enables IPFS over pubsub experiment for publishing IPNS records in real time.

**EXPERIMENTAL:**  read about current limitations at [experimental-features.md#ipns-pubsub](./experimental-features.md#ipns-pubsub).

Default: `disabled`

Type: `flag`

## `Migration`

Migration configures how migrations are downloaded and if the downloads are added to IPFS locally.

### `Migration.DownloadSources`

Sources in order of preference, where "IPFS" means use IPFS and "HTTPS" means use default gateways. Any other values are interpreted as hostnames for custom gateways. An empty list means "use default sources".

Default: `["HTTPS", "IPFS"]`

### `Migration.Keep`

Specifies whether or not to keep the migration after downloading it. Options are "discard", "cache", "pin". Empty string for default.

Default: `cache`

## `Mounts`

**EXPERIMENTAL:** read about current limitations at [fuse.md](./fuse.md).

FUSE mount point configuration options.

### `Mounts.IPFS`

Mountpoint for `/ipfs/`.

Default: `/ipfs`

Type: `string` (filesystem path)

### `Mounts.IPNS`

Mountpoint for `/ipns/`.

Default: `/ipns`

Type: `string` (filesystem path)

### `Mounts.FuseAllowOther`

Sets the 'FUSE allow other'-option on the mount point.

## `Pinning`

Pinning configures the options available for pinning content
(i.e. keeping content longer-term instead of as temporarily cached storage).

### `Pinning.RemoteServices`

`RemoteServices` maps a name for a remote pinning service to its configuration.

A remote pinning service is a remote service that exposes an API for managing
that service's interest in long-term data storage.

The exposed API conforms to the specification defined at
https://ipfs.github.io/pinning-services-api-spec/

#### `Pinning.RemoteServices: API`

Contains information relevant to utilizing the remote pinning service

Example:
```json
{
  "Pinning": {
    "RemoteServices": {
      "myPinningService": {
        "API" : {
          "Endpoint" : "https://pinningservice.tld:1234/my/api/path",
          "Key" : "someOpaqueKey"
				}
      }
    }
  }
}
```

##### `Pinning.RemoteServices: API.Endpoint`

The HTTP(S) endpoint through which to access the pinning service

Example: "https://pinningservice.tld:1234/my/api/path"

Type: `string`

##### `Pinning.RemoteServices: API.Key`

The key through which access to the pinning service is granted

Type: `string`

#### `Pinning.RemoteServices: Policies`

Contains additional opt-in policies for the remote pinning service.

##### `Pinning.RemoteServices: Policies.MFS`

When this policy is enabled, it follows changes to MFS
and updates the pin for MFS root on the configured remote service.

A pin request to the remote service is sent only when MFS root CID has changed
and enough time has passed since the previous request (determined by `RepinInterval`).

One can observe MFS pinning details by enabling debug via `ipfs log level remotepinning/mfs debug` and switching back to `error` when done.

###### `Pinning.RemoteServices: Policies.MFS.Enabled`

Controls if this policy is active.

Default: `false`

Type: `bool`

###### `Pinning.RemoteServices: Policies.MFS.PinName`

Optional name to use for a remote pin that represents the MFS root CID.
When left empty, a default name will be generated.

Default: `"policy/{PeerID}/mfs"`, e.g. `"policy/12.../mfs"`

Type: `string`

###### `Pinning.RemoteServices: Policies.MFS.RepinInterval`

Defines how often (at most) the pin request should be sent to the remote service.
If left empty, the default interval will be used. Values lower than `1m` will be ignored.

Default: `"5m"`

Type: `duration`

## `Pubsub`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Pubsub configures the `ipfs pubsub` subsystem. To use, it must be enabled by
passing the `--enable-pubsub-experiment` flag to the daemon
or via the `Pubsub.Enabled` flag below.

### `Pubsub.Enabled`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Enables the pubsub system.

Default: `false`

Type: `flag`

### `Pubsub.Router`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Sets the default router used by pubsub to route messages to peers. This can be one of:

* `"floodsub"` - floodsub is a basic router that simply _floods_ messages to all
  connected peers. This router is extremely inefficient but _very_ reliable.
* `"gossipsub"` - [gossipsub][] is a more advanced routing algorithm that will
  build an overlay mesh from a subset of the links in the network.

Default: `"gossipsub"`

Type: `string` (one of `"floodsub"`, `"gossipsub"`, or `""` (apply default))

[gossipsub]: https://github.com/libp2p/specs/tree/master/pubsub/gossipsub

### `Pubsub.DisableSigning`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Disables message signing and signature verification. Enable this option if
you're operating in a completely trusted network.

It is _not_ safe to disable signing even if you don't care _who_ sent the
message because spoofed messages can be used to silence real messages by
intentionally re-using the real message's message ID.

Default: `false`

Type: `bool`

### `Pubsub.SeenMessagesTTL`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Controls the time window within which duplicate messages, identified by Message
ID, will be identified and won't be emitted again.

A smaller value for this parameter means that Pubsub messages in the cache will
be garbage collected sooner, which can result in a smaller cache. At the same
time, if there are slower nodes in the network that forward older messages,
this can cause more duplicates to be propagated through the network.

Conversely, a larger value for this parameter means that Pubsub messages in the
cache will be garbage collected later, which can result in a larger cache for
the same traffic pattern. However, it is less likely that duplicates will be
propagated through the network.

Default: see `TimeCacheDuration` from [go-libp2p-pubsub](https://github.com/libp2p/go-libp2p-pubsub)

Type: `optionalDuration`

### `Pubsub.SeenMessagesStrategy`

**DEPRECATED**: See [#9717](https://github.com/ipfs/kubo/issues/9717)

Determines how the time-to-live (TTL) countdown for deduplicating Pubsub
messages is calculated.

The Pubsub seen messages cache is a LRU cache that keeps messages for up to a
specified time duration. After this duration has elapsed, expired messages will
be purged from the cache.

The `last-seen` cache is a sliding-window cache. Every time a message is seen
again with the SeenMessagesTTL duration, its timestamp slides forward. This
keeps frequently occurring messages cached and prevents them from being
continually propagated, especially because of issues that might increase the
number of duplicate messages in the network.

The `first-seen` cache will store new messages and purge them after the
SeenMessagesTTL duration, even if they are seen multiple times within this
duration.

Default: `last-seen` (see [go-libp2p-pubsub](https://github.com/libp2p/go-libp2p-pubsub))

Type: `optionalString`

## `Peering`

Configures the peering subsystem. The peering subsystem configures Kubo to
connect to, remain connected to, and reconnect to a set of nodes. Nodes should
use this subsystem to create "sticky" links between frequently useful peers to
improve reliability.

Use-cases:

* An IPFS gateway connected to an IPFS cluster should peer to ensure that the
  gateway can always fetch content from the cluster.
* A dapp may peer embedded Kubo nodes with a set of pinning services or
  textile cafes/hubs.
* A set of friends may peer to ensure that they can always fetch each other's
  content.

When a node is added to the set of peered nodes, Kubo will:

1. Protect connections to this node from the connection manager. That is,
   Kubo will never automatically close the connection to this node and
   connections to this node will not count towards the connection limit.
2. Connect to this node on startup.
3. Repeatedly try to reconnect to this node if the last connection dies or the
   node goes offline. This repeated re-connect logic is governed by a randomized
   exponential backoff delay ranging from ~5 seconds to ~10 minutes to avoid
   repeatedly reconnect to a node that's offline.

Peering can be asymmetric or symmetric:

* When symmetric, the connection will be protected by both nodes and will likely
  be very stable.
* When asymmetric, only one node (the node that configured peering) will protect
  the connection and attempt to re-connect to the peered node on disconnect. If
  the peered node is under heavy load and/or has a low connection limit, the
  connection may flap repeatedly. Be careful when asymmetrically peering to not
  overload peers.

### `Peering.Peers`

The set of peers with which to peer.

```json
{
  "Peering": {
    "Peers": [
      {
        "ID": "QmPeerID1",
        "Addrs": ["/ip4/18.1.1.1/tcp/4001"]
      },
      {
        "ID": "QmPeerID2",
        "Addrs": ["/ip4/18.1.1.2/tcp/4001", "/ip4/18.1.1.2/udp/4001/quic-v1"]
      }
    ]
  }
  ...
}
```

Where `ID` is the peer ID and `Addrs` is a set of known addresses for the peer. If no addresses are specified, the Amino DHT will be queried.

Additional fields may be added in the future.

Default: empty.

Type: `array[peering]`

## `Reprovider`

### `Reprovider.Interval`

Sets the time between rounds of reproviding local content to the routing
system.

- If unset, it uses the implicit safe default.
- If set to the value `"0"` it will disable content reproviding.

Note: disabling content reproviding will result in other nodes on the network
not being able to discover that you have the objects that you have. If you want
to have this disabled and keep the network aware of what you have, you must
manually announce your content periodically.

Default: `22h` (`DefaultReproviderInterval`)

Type: `optionalDuration` (unset for the default)

### `Reprovider.Strategy`

Tells reprovider what should be announced. Valid strategies are:

- `"all"` - announce all CIDs of stored blocks
- `"pinned"` - only announce pinned CIDs recursively (both roots and child blocks)
- `"roots"` - only announce the root block of explicitly pinned CIDs

Default: `"all"`

Type: `optionalString` (unset for the default)

## `Routing`

Contains options for content, peer, and IPNS routing mechanisms.

### `Routing.Type`

There are multiple routing options: "auto", "autoclient", "none", "dht", "dhtclient", and "custom".

* **DEFAULT:** If unset, or set to "auto", your node will use the public IPFS DHT (aka "Amino")
  and parallel HTTP routers listed below for additional speed.

* If set to "autoclient", your node will behave as in "auto" but without running a DHT server.

* If set to "none", your node will use _no_ routing system. You'll have to
  explicitly connect to peers that have the content you're looking for.

* If set to "dht" (or "dhtclient"/"dhtserver"), your node will ONLY use the Amino DHT (no HTTP routers).

* If set to "custom", all default routers are disabled, and only ones defined in `Routing.Routers` will be used.

When the DHT is enabled, it can operate in two modes: client and server.

* In server mode, your node will query other peers for DHT records, and will
  respond to requests from other peers (both requests to store records and
  requests to retrieve records).

* In client mode, your node will query the DHT as a client but will not respond
  to requests from other peers. This mode is less resource-intensive than server
  mode.

When `Routing.Type` is set to `auto` or `dht`, your node will start as a DHT client, and
switch to a DHT server when and if it determines that it's reachable from the
public internet (e.g., it's not behind a firewall).

To force a specific Amino DHT-only mode, client or server, set `Routing.Type` to
`dhtclient` or `dhtserver` respectively. Please do not set this to `dhtserver`
unless you're sure your node is reachable from the public network.

When `Routing.Type` is set to `auto` or `autoclient` your node will accelerate some types of routing
by leveraging HTTP endpoints compatible with [IPIP-337](https://github.com/ipfs/specs/pull/337)
in addition to the IPFS DHT.
By default, an instance of [IPNI](https://github.com/ipni/specs/blob/main/IPNI.md#readme)
at https://cid.contact is used.
Alternative routing rules can be configured in `Routing.Routers` after setting `Routing.Type` to `custom`.

Default: `auto` (DHT + IPNI)

Type: `optionalString` (`null`/missing means the default)


### `Routing.AcceleratedDHTClient`

This alternative Amino DHT client with a Full-Routing-Table strategy will
do a complete scan of the DHT every hour and record all nodes found.
Then when a lookup is tried instead of having to go through multiple Kad hops it
is able to find the 20 final nodes by looking up the in-memory recorded network table.

This means sustained higher memory to store the routing table
and extra CPU and network bandwidth for each network scan.
However the latency of individual read/write operations should be ~10x faster
and the provide throughput up to 6 million times faster on larger datasets!

This is not compatible with `Routing.Type` `custom`. If you are using composable routers
you can configure this individually on each router.

When it is enabled:
- Client DHT operations (reads and writes) should complete much faster
- The provider will now use a keyspace sweeping mode allowing to keep alive
  CID sets that are multiple orders of magnitude larger.
  - The standard Bucket-Routing-Table DHT will still run for the DHT server (if
    the DHT server is enabled). This means the classical routing table will
    still be used to answer other nodes.
    This is critical to maintain to not harm the network.
- The operations `ipfs stats dht` will default to showing information about the accelerated DHT client

**Caveats:**
1. Running the accelerated client likely will result in more resource consumption (connections, RAM, CPU, bandwidth)
   - Users that are limited in the number of parallel connections their machines/networks can perform will likely suffer
   - The resource usage is not smooth as the client crawls the network in rounds and reproviding is similarly done in rounds
   - Users who previously had a lot of content but were unable to advertise it on the network will see an increase in
     egress bandwidth as their nodes start to advertise all of their CIDs into the network. If you have lots of data
     entering your node that you don't want to advertise, then consider using [Reprovider Strategies](#reproviderstrategy)
     to reduce the number of CIDs that you are reproviding. Similarly, if you are running a node that deals mostly with
     short-lived temporary data (e.g. you use a separate node for ingesting data then for storing and serving it) then
     you may benefit from using [Strategic Providing](experimental-features.md#strategic-providing) to prevent advertising
     of data that you ultimately will not have.
2. Currently, the DHT is not usable for queries for the first 5-10 minutes of operation as the routing table is being
prepared. This means operations like searching the DHT for particular peers or content will not work initially.
   - You can see if the DHT has been initially populated by running `ipfs stats dht`
3. Currently, the accelerated DHT client is not compatible with LAN-based DHTs and will not perform operations against
them

Default: `false`

Type: `bool` (missing means `false`)

### `Routing.Routers`

**EXPERIMENTAL: `Routing.Routers` configuration may change in future release**

Map of additional Routers.

Allows for extending the default routing (Amino DHT) with alternative Router
implementations.

The map key is a name of a Router, and the value is its configuration.

Default: `{}`

Type: `object[string->object]`

#### `Routing.Routers: Type`

**EXPERIMENTAL: `Routing.Routers` configuration may change in future release**

It specifies the routing type that will be created.

Currently supported types:

- `http` simple delegated routing based on HTTP protocol from [IPIP-337](https://github.com/ipfs/specs/pull/337)
- `dht` provides decentralized routing based on [libp2p's kad-dht](https://github.com/libp2p/specs/tree/master/kad-dht)
- `parallel` and `sequential`: Helpers that can be used to run several routers sequentially or in parallel.

Type: `string`

#### `Routing.Routers: Parameters`

**EXPERIMENTAL: `Routing.Routers` configuration may change in future release**

Parameters needed to create the specified router. Supported params per router type:

HTTP:
  - `Endpoint` (mandatory): URL that will be used to connect to a specified router.
  - `MaxProvideBatchSize`: This number determines the maximum amount of CIDs sent per batch. Servers might not accept more than 100 elements per batch. 100 elements by default.
  - `MaxProvideConcurrency`: It determines the number of threads used when providing content. GOMAXPROCS by default.

DHT:
  - `"Mode"`: Mode used by the Amino DHT. Possible values: "server", "client", "auto"
  - `"AcceleratedDHTClient"`: Set to `true` if you want to use the acceleratedDHT.
  - `"PublicIPNetwork"`: Set to `true` to create a `WAN` DHT. Set to `false` to create a `LAN` DHT.

Parallel:
  - `Routers`: A list of routers that will be executed in parallel:
    - `Name:string`: Name of the router. It should be one of the previously added to `Routers` list.
    - `Timeout:duration`: Local timeout. It accepts strings compatible with Go `time.ParseDuration(string)` (`10s`, `1m`, `2h`). Time will start counting when this specific router is called, and it will stop when the router returns, or we reach the specified timeout.
    - `ExecuteAfter:duration`: Providing this param will delay the execution of that router at the specified time. It accepts strings compatible with Go `time.ParseDuration(string)` (`10s`, `1m`, `2h`).
    - `IgnoreErrors:bool`: It will specify if that router should be ignored if an error occurred.
  - `Timeout:duration`: Global timeout.  It accepts strings compatible with Go `time.ParseDuration(string)` (`10s`, `1m`, `2h`).

Sequential:
  - `Routers`: A list of routers that will be executed in order:
    - `Name:string`: Name of the router. It should be one of the previously added to `Routers` list.
    - `Timeout:duration`: Local timeout. It accepts strings compatible with Go `time.ParseDuration(string)`. Time will start counting when this specific router is called, and it will stop when the router returns, or we reach the specified timeout.
    - `IgnoreErrors:bool`: It will specify if that router should be ignored if an error occurred.
  - `Timeout:duration`: Global timeout.  It accepts strings compatible with Go `time.ParseDuration(string)`.

Default: `{}` (use the safe implicit defaults)

Type: `object[string->string]`

### `Routing: Methods`

`Methods:map` will define which routers will be executed per method. The key will be the name of the method: `"provide"`, `"find-providers"`, `"find-peers"`, `"put-ipns"`, `"get-ipns"`. All methods must be added to the list.

The value will contain:
- `RouterName:string`: Name of the router. It should be one of the previously added to `Routing.Routers` list.

Type: `object[string->object]`

**Examples:**

Complete example using 2 Routers, Amino DHT (LAN/WAN) and parallel.

```
$ ipfs config Routing.Type --json '"custom"'

$ ipfs config Routing.Routers.WanDHT --json '{
  "Type": "dht",
  "Parameters": {
    "Mode": "auto",
    "PublicIPNetwork": true,
    "AcceleratedDHTClient": false
  }
}'

$ ipfs config Routing.Routers.LanDHT --json '{
  "Type": "dht",
  "Parameters": {
    "Mode": "auto",
    "PublicIPNetwork": false,
    "AcceleratedDHTClient": false
  }
}'

$ ipfs config Routing.Routers.ParallelHelper --json '{
  "Type": "parallel",
  "Parameters": {
    "Routers": [
        {
        "RouterName" : "LanDHT",
        "IgnoreErrors" : true,
        "Timeout": "3s"
        },
        {
        "RouterName" : "WanDHT",
        "IgnoreErrors" : false,
        "Timeout": "5m",
        "ExecuteAfter": "2s"
        }
    ]
  }
}'

ipfs config Routing.Methods --json '{
      "find-peers": {
        "RouterName": "ParallelHelper"
      },
      "find-providers": {
        "RouterName": "ParallelHelper"
      },
      "get-ipns": {
        "RouterName": "ParallelHelper"
      },
      "provide": {
        "RouterName": "ParallelHelper"
      },
      "put-ipns": {
        "RouterName": "ParallelHelper"
      }
    }'

```

## `Swarm`

Options for configuring the swarm.

### `Swarm.AddrFilters`

An array of addresses (multiaddr netmasks) to not dial. By default, IPFS nodes
advertise _all_ addresses, even internal ones. This makes it easier for nodes on
the same network to reach each other. Unfortunately, this means that an IPFS
node will try to connect to one or more private IP addresses whenever dialing
another node, even if this other node is on a different network. This may
trigger netscan alerts on some hosting providers or cause strain in some setups.

The `server` configuration profile fills up this list with sensible defaults,
preventing dials to all non-routable IP addresses (e.g., `/ip4/192.168.0.0/ipcidr/16`,
which is the multiaddress representation of `192.168.0.0/16`) but you should always
check settings against your own network and/or hosting provider.

Default: `[]`

Type: `array[string]`

### `Swarm.DisableBandwidthMetrics`

A boolean value that when set to true, will cause ipfs to not keep track of
bandwidth metrics. Disabling bandwidth metrics can lead to a slight performance
improvement, as well as a reduction in memory usage.

Default: `false`

Type: `bool`

### `Swarm.DisableNatPortMap`

Disable automatic NAT port forwarding.

When not disabled (default), Kubo asks NAT devices (e.g., routers), to open
up an external port and forward it to the port Kubo is running on. When this
works (i.e., when your router supports NAT port forwarding), it makes the local
Kubo node accessible from the public internet.

Default: `false`

Type: `bool`

### `Swarm.EnableHolePunching`

Enable hole punching for NAT traversal
when port forwarding is not possible.

When enabled, Kubo will coordinate with the counterparty using
a [relayed connection](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md),
to [upgrade to a direct connection](https://github.com/libp2p/specs/blob/master/relay/DCUtR.md)
through a NAT/firewall whenever possible.
This feature requires `Swarm.RelayClient.Enabled` to be set to `true`.

Default: `true`

Type: `flag`

### `Swarm.EnableAutoRelay`

**REMOVED**

See `Swarm.RelayClient` instead.

### `Swarm.RelayClient`

Configuration options for the relay client to use relay services.

Default: `{}`

Type: `object`

#### `Swarm.RelayClient.Enabled`

Enables "automatic relay user" mode for this node.

Your node will automatically _use_ public relays from the network if it detects
that it cannot be reached from the public internet (e.g., it's behind a
firewall) and get a `/p2p-circuit` address from a public relay.

Default: `true`

Type: `flag`

#### `Swarm.RelayClient.StaticRelays`

Your node will use these statically configured relay servers
instead of discovering public relays ([Circuit Relay v2](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md)) from the network.

Default: `[]`

Type: `array[string]`

### `Swarm.RelayService`

Configuration options for the relay service that can be provided to _other_ peers
on the network ([Circuit Relay v2](https://github.com/libp2p/specs/blob/master/relay/circuit-v2.md)).

Default: `{}`

Type: `object`

#### `Swarm.RelayService.Enabled`

Enables providing `/p2p-circuit` v2 relay service to other peers on the network.

NOTE: This is the service/server part of the relay system.
Disabling this will prevent this node from running as a relay server.
Use [`Swarm.RelayClient.Enabled`](#swarmrelayclientenabled) for turning your node into a relay user.

Default: `true`

Type: `flag`

#### `Swarm.RelayService.Limit`

Limits applied to every relayed connection.

Default: `{}`

Type: `object[string -> string]`

##### `Swarm.RelayService.ConnectionDurationLimit`

Time limit before a relayed connection is reset.

Default: `"2m"`

Type: `duration`

##### `Swarm.RelayService.ConnectionDataLimit`

Limit of data relayed (in each direction) before a relayed connection is reset.

Default: `131072` (128 kb)

Type: `optionalInteger`


#### `Swarm.RelayService.ReservationTTL`

Duration of a new or refreshed reservation.

Default: `"1h"`

Type: `duration`


#### `Swarm.RelayService.MaxReservations`

Maximum number of active relay slots.

Default: `128`

Type: `optionalInteger`


#### `Swarm.RelayService.MaxCircuits`

Maximum number of open relay connections for each peer.

Default: `16`

Type: `optionalInteger`


#### `Swarm.RelayService.BufferSize`

Size of the relayed connection buffers.

Default: `2048`

Type: `optionalInteger`


#### `Swarm.RelayService.MaxReservationsPerPeer`

Maximum number of reservations originating from the same peer.

Default: `4`

Type: `optionalInteger`


#### `Swarm.RelayService.MaxReservationsPerIP`

Maximum number of reservations originating from the same IP.

Default: `8`

Type: `optionalInteger`

#### `Swarm.RelayService.MaxReservationsPerASN`

Maximum number of reservations originating from the same ASN.

Default: `32`

Type: `optionalInteger`

### `Swarm.EnableRelayHop`

**REMOVED**

Replaced with [`Swarm.RelayService.Enabled`](#swarmrelayserviceenabled).

### `Swarm.DisableRelay`

**REMOVED**

Set `Swarm.Transports.Network.Relay` to `false` instead.

### `Swarm.EnableAutoNATService`

**REMOVED**

Please use [`AutoNAT.ServiceMode`](#autonatservicemode).

### `Swarm.ConnMgr`

The connection manager determines which and how many connections to keep and can
be configured to keep. Kubo currently supports two connection managers:

* none: never close idle connections.
* basic: the default connection manager.

By default, this section is empty and the implicit defaults defined below
are used.

#### `Swarm.ConnMgr.Type`

Sets the type of connection manager to use, options are: `"none"` (no connection
management) and `"basic"`.

Default: "basic".

Type: `optionalString` (default when unset or empty)

#### Basic Connection Manager

The basic connection manager uses a "high water", a "low water", and internal
scoring to periodically close connections to free up resources. When a node
using the basic connection manager reaches `HighWater` idle connections, it will
close the least useful ones until it reaches `LowWater` idle connections.

The connection manager considers a connection idle if:

* It has not been explicitly _protected_ by some subsystem. For example, Bitswap
  will protect connections to peers from which it is actively downloading data,
  the DHT will protect some peers for routing, and the peering subsystem will
  protect all "peered" nodes.
* It has existed for longer than the `GracePeriod`.

**Example:**

```json
{
  "Swarm": {
    "ConnMgr": {
      "Type": "basic",
      "LowWater": 100,
      "HighWater": 200,
      "GracePeriod": "30s"
    }
  }
}
```

##### `Swarm.ConnMgr.LowWater`

LowWater is the number of connections that the basic connection manager will
trim down to.

Default: `32`

Type: `optionalInteger`

##### `Swarm.ConnMgr.HighWater`

HighWater is the number of connections that, when exceeded, will trigger a
connection GC operation. Note: protected/recently formed connections don't count
towards this limit.

Default: `96`

Type: `optionalInteger`

##### `Swarm.ConnMgr.GracePeriod`

GracePeriod is a time duration that new connections are immune from being closed
by the connection manager.

Default: `"20s"`

Type: `optionalDuration`

### `Swarm.ResourceMgr`

Learn more about Kubo's usage of libp2p Network Resource Manager
in the [dedicated resource management docs](./libp2p-resource-management.md).

#### `Swarm.ResourceMgr.Enabled`

Enables the libp2p Resource Manager using limits based on the defaults and/or other configuration as discussed in [libp2p resource management](./libp2p-resource-management.md).

Default: `true`
Type: `flag`

#### `Swarm.ResourceMgr.MaxMemory`

This is the max amount of memory to allow libp2p to use.
libp2p's resource manager will prevent additional resource creation while this limit is reached.
This value is also used to scale the limit on various resources at various scopes
when the default limits (discussed in [libp2p resource management](./libp2p-resource-management.md)) are used.
For example, increasing this value will increase the default limit for incoming connections.

It is possible to inspect the runtime limits via `ipfs swarm resources --help`.

Default: `[TOTAL_SYSTEM_MEMORY]/2`
Type: `optionalBytes`

#### `Swarm.ResourceMgr.MaxFileDescriptors`

This is the maximum number of file descriptors to allow libp2p to use.
libp2p's resource manager will prevent additional file descriptor consumption while this limit is reached.

This param is ignored on Windows.

Default `[TOTAL_SYSTEM_FILE_DESCRIPTORS]/2`
Type: `optionalInteger`

#### `Swarm.ResourceMgr.Allowlist`

A list of multiaddrs that can bypass normal system limits (but are still limited by the allowlist scope).
Convenience config around [go-libp2p-resource-manager#Allowlist.Add](https://pkg.go.dev/github.com/libp2p/go-libp2p/p2p/host/resource-manager#Allowlist.Add).

Default: `[]`

Type: `array[string]` (multiaddrs)

### `Swarm.Transports`

Configuration section for libp2p transports. An empty configuration will apply
the defaults.

### `Swarm.Transports.Network`

Configuration section for libp2p _network_ transports. Transports enabled in
this section will be used for dialing. However, to receive connections on these
transports, multiaddrs for these transports must be added to `Addresses.Swarm`.

Supported transports are: QUIC, TCP, WS, Relay and WebTransport.

Each field in this section is a `flag`.

#### `Swarm.Transports.Network.TCP`

[TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) is a simple
and widely deployed transport, it should be compatible with most implementations
and network configurations.  TCP doesn't directly support encryption and/or
multiplexing, so libp2p will layer a security & multiplexing transport over it.

Default: Enabled

Type: `flag`

Listen Addresses:
* /ip4/0.0.0.0/tcp/4001 (default)
* /ip6/::/tcp/4001 (default)

#### `Swarm.Transports.Network.Websocket`

[Websocket](https://en.wikipedia.org/wiki/WebSocket) is a transport usually used
to connect to non-browser-based IPFS nodes from browser-based js-ipfs nodes.

While it's enabled by default for dialing, Kubo doesn't listen on this
transport by default.

Default: Enabled

Type: `flag`

Listen Addresses:
* /ip4/0.0.0.0/tcp/4002/ws
* /ip6/::/tcp/4002/ws

#### `Swarm.Transports.Network.QUIC`

[QUIC](https://en.wikipedia.org/wiki/QUIC) is the most widely used transport by
Kubo nodes. It is a UDP-based transport with built-in encryption and
multiplexing. The primary benefits over TCP are:

1. It takes 1 round trip to establish a connection (our TCP transport
   currently takes 4).
2. No [Head-of-Line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking).
3. It doesn't require a file descriptor per connection, easing the load on the OS.

Default: Enabled

Type: `flag`

Listen Addresses:
* /ip4/0.0.0.0/udp/4001/quic-v1 (default)
* /ip6/::/udp/4001/quic-v1 (default)

#### `Swarm.Transports.Network.Relay`

[Libp2p Relay](https://github.com/libp2p/specs/tree/master/relay) proxy
transport that forms connections by hopping between multiple libp2p nodes.
Allows IPFS node to connect to other peers using their `/p2p-circuit`
multiaddrs.  This transport is primarily useful for bypassing firewalls and
NATs.

See also:
- Docs: [Libp2p Circuit Relay](https://docs.libp2p.io/concepts/circuit-relay/)
- [`Swarm.RelayClient.Enabled`](#swarmrelayclientenabled) for getting a public
-  `/p2p-circuit` address when behind a firewall.
  - [`Swarm.EnableHolePunching`](#swarmenableholepunching) for direct connection upgrade through relay
- [`Swarm.RelayService.Enabled`](#swarmrelayserviceenabled) for becoming a
  limited relay for other peers

Default: Enabled

Type: `flag`

Listen Addresses:
* This transport is special. Any node that enables this transport can receive
  inbound connections on this transport, without specifying a listen address.


#### `Swarm.Transports.Network.WebTransport`

A new feature of [`go-libp2p`](https://github.com/libp2p/go-libp2p/releases/tag/v0.23.0)
is the [WebTransport](https://github.com/libp2p/go-libp2p/issues/1717) transport.

This is a spiritual descendant of WebSocket but over `HTTP/3`.
Since this runs on top of `HTTP/3` it uses `QUIC` under the hood.
We expect it to perform worst than `QUIC` because of the extra overhead,
this transport is really meant at agents that cannot do `TCP` or `QUIC` (like browsers).

WebTransport is a new transport protocol currently under development by the IETF and the W3C, and already implemented by Chrome.
Conceptually, it’s like WebSocket run over QUIC instead of TCP. Most importantly, it allows browsers to establish (secure!) connections to WebTransport servers without the need for CA-signed certificates,
thereby enabling any js-libp2p node running in a browser to connect to any kubo node, with zero manual configuration involved.

The previous alternative is websocket secure, which require installing a reverse proxy and TLS certificates manually.

Default: Enabled

Type: `flag`

### `Swarm.Transports.Security`

Configuration section for libp2p _security_ transports. Transports enabled in
this section will be used to secure unencrypted connections.

This does not concern all the QUIC transports which use QUIC's builtin encryption.

Security transports are configured with the `priority` type.

When establishing an _outbound_ connection, Kubo will try each security
transport in priority order (lower first), until it finds a protocol that the
receiver supports. When establishing an _inbound_ connection, Kubo will let
the initiator choose the protocol, but will refuse to use any of the disabled
transports.

Supported transports are: TLS (priority 100) and Noise (priority 300).

No default priority will ever be less than 100.

#### `Swarm.Transports.Security.TLS`

[TLS](https://github.com/libp2p/specs/tree/master/tls) (1.3) is the default
security transport as of Kubo 0.5.0. It's also the most scrutinized and
trusted security transport.

Default: `100`

Type: `priority`

#### `Swarm.Transports.Security.SECIO`

Support for SECIO has been removed. Please remove this option from your config.

#### `Swarm.Transports.Security.Noise`

[Noise](https://github.com/libp2p/specs/tree/master/noise) is slated to replace
TLS as the cross-platform, default libp2p protocol due to ease of
implementation. It is currently enabled by default but with low priority as it's
not yet widely supported.

Default: `300`

Type: `priority`

### `Swarm.Transports.Multiplexers`

Configuration section for libp2p _multiplexer_ transports. Transports enabled in
this section will be used to multiplex duplex connections.

This does not concern all the QUIC transports which use QUIC's builtin muxing.

Multiplexer transports are configured the same way security transports are, with
the `priority` type. Like with security transports, the initiator gets their
first choice.

Supported transport is only: Yamux (priority 100)

No default priority will ever be less than 100.

### `Swarm.Transports.Multiplexers.Yamux`

Yamux is the default multiplexer used when communicating between Kubo nodes.

Default: `100`

Type: `priority`

### `Swarm.Transports.Multiplexers.Mplex`

**DEPRECATED**: See https://github.com/ipfs/kubo/issues/9958

Mplex is deprecated, this is because it is unreliable and
randomly drop streams when sending data *too fast*.

New pieces of code rely on backpressure, that means the stream will dynamically
slow down the sending rate if data is getting backed up.
Backpressure is provided by **Yamux** and **QUIC**.

If you want to turn it back on make sure to have a higher (lower is better)
priority than `Yamux`, you don't want your Kubo to start defaulting to Mplex.

Default: `200`

Type: `priority`

## `DNS`

Options for configuring DNS resolution for [DNSLink](https://docs.ipfs.tech/concepts/dnslink/) and `/dns*` [Multiaddrs](https://github.com/multiformats/multiaddr/).

### `DNS.Resolvers`

Map of [FQDNs](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) to custom resolver URLs.

This allows for overriding the default DNS resolver provided by the operating system,
and using different resolvers per domain or TLD (including ones from alternative, non-ICANN naming systems).

Example:
```json
{
  "DNS": {
    "Resolvers": {
      "eth.": "https://dns.eth.limo/dns-query",
      "crypto.": "https://resolver.unstoppable.io/dns-query",
      "libre.": "https://ns1.iriseden.fr/dns-query",
      ".": "https://cloudflare-dns.com/dns-query"
    }
  }
}
```

Be mindful that:
- Currently only `https://` URLs for [DNS over HTTPS (DoH)](https://en.wikipedia.org/wiki/DNS_over_HTTPS) endpoints are supported as values.
- The default catch-all resolver is the cleartext one provided by your operating system. It can be overridden by adding a DoH entry for the DNS root indicated by  `.` as illustrated above.
- Out-of-the-box support for selected decentralized TLDs relies on a [centralized service which is provided on best-effort basis](https://www.cloudflare.com/distributed-web-gateway-terms/). The implicit DoH resolvers are:
  ```json
  {
    "eth.": "https://resolver.cloudflare-eth.com/dns-query",
    "crypto.": "https://resolver.cloudflare-eth.com/dns-query"
  }
  ```
  To get all the benefits of a decentralized naming system we strongly suggest setting DoH endpoint to an empty string and running own decentralized resolver as catch-all one on localhost.

Default: `{}`

Type: `object[string -> string]`

### `DNS.MaxCacheTTL`

Maximum duration for which entries are valid in the DoH cache.

This allows you to cap the Time-To-Live suggested by the DNS response ([RFC2181](https://datatracker.ietf.org/doc/html/rfc2181#section-8)).
If present, the upper bound is applied to DoH resolvers in [`DNS.Resolvers`](#dnsresolvers).

Note: this does NOT work with Go's default DNS resolver. To make this a global setting, add a `.` entry to `DNS.Resolvers` first.

**Examples:**
* `"5m"` DNS entries are kept for 5 minutes or less.
* `"0s"` DNS entries expire as soon as they are retrieved.

Default: Respect DNS Response TTL

Type: `optionalDuration`
