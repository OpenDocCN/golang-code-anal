# go-ipfs 源码解析 25

# `/opt/kubo/core/coreapi/pubsub.go`

这段代码定义了一个名为"coreapi"的包，它使用了多个其他包的依赖关系。

首先，它导入了coreiface、errors和tracing包，这些包用于与IPFS(InterPlanetary File System)的Box-O应用程序进行交互。

然后，它导入了caopts包，该包用于提供配置选项，以使应用程序更加灵活。

接着，它导入了pubsub包，该包用于与公众订阅服务进行交互。

然后，它导入了peer包，该包用于与对等网络中的节点进行交互。

接下来，它导入了routing包，该包用于定义路由，以使应用程序可以在不同的网络中进行通信。

最后，它导入了go-libp2p-pubsub包，该包实现了PubSub一路由的交互，用于在Box-O应用程序与对等网络节点之间传输数据。

此外，还导入了go-opentelemetry-io/otel包和go.opentelemetry.io/otel包，用于收集和跟踪分布式跟踪事务。


```
package coreapi

import (
	"context"
	"errors"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/kubo/tracing"
	pubsub "github.com/libp2p/go-libp2p-pubsub"
	peer "github.com/libp2p/go-libp2p/core/peer"
	routing "github.com/libp2p/go-libp2p/core/routing"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"
)

```

这段代码定义了一个名为`PubSubAPI`的类型，该类型有一个名为`pubSubSubscription`的结构体和一个名为`pubSubMessage`的结构体。

`pubSubSubscription`结构体包含一个指向`pubsub.Subscription`类型的引用，这个结构体可能用于订阅某个主题的消息。

`pubSubMessage`结构体包含一个指向`pubsub.Message`类型的引用，这个结构体可能用于发送消息。

该代码的目的是在`core-api`包中定义一个名为`Ls`的方法，该方法返回了当前运行时正在订阅的主题列表，如果订阅主题失败，则返回一个非空错误。

具体来说，该方法的实现包括以下步骤：

1. 检查当前运行时是否连接到Node.js服务，如果不是，则需要创建一个实例并连接。
2. 如果连接成功，则调用`pubsub.PubSubAPI`中的`checkNode`方法来检查是否存在任何错误。
3. 如果存在错误，则返回一个非空错误。
4. 如果没有任何错误，则使用`pubsub.PubSubAPI`中的`pubSub.GetTopics`方法获取订阅主题列表。
5. 将获取到的主题列表返回。


```
type PubSubAPI CoreAPI

type pubSubSubscription struct {
	subscription *pubsub.Subscription
}

type pubSubMessage struct {
	msg *pubsub.Message
}

func (api *PubSubAPI) Ls(ctx context.Context) ([]string, error) {
	_, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Ls")
	defer span.End()

	_, err := api.checkNode()
	if err != nil {
		return nil, err
	}

	return api.pubSub.GetTopics(), nil
}

```

此函数的作用是获取给定主题的信息服务器列表。它接收一个PubSubAPI类型的参数api，并传递一个或多个PubSubPeers选项。然后，它检查API的节点是否成功加载，并验证传递的选项。如果设置没有问题，它将设置属性的"topic"为设置的主题，并返回主题设置的信息服务器列表。函数使用了tracing中的Span，在操作失败时将输出错误。


```
func (api *PubSubAPI) Peers(ctx context.Context, opts ...caopts.PubSubPeersOption) ([]peer.ID, error) {
	_, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Peers")
	defer span.End()

	_, err := api.checkNode()
	if err != nil {
		return nil, err
	}

	settings, err := caopts.PubSubPeersOptions(opts...)
	if err != nil {
		return nil, err
	}

	span.SetAttributes(attribute.String("topic", settings.Topic))

	return api.pubSub.ListPeers(settings.Topic), nil
}

```

此代码定义了两个函数：func (api *PubSubAPI) Publish(ctx context.Context, topic string, data []byte) error 和 func (api *PubSubAPI) Subscribe(ctx context.Context, topic string, opts ...caopts.PubSubSubscribeOption) (coreiface.PubSubSubscription, error)。

函数Publish接收一个名为api的PubSubAPI实例，一个名为topic的主题和一个或多个名为data的数据字节数组。函数使用tracing库来记录性能和错误信息，并使用一个名为span的跨度来跟踪请求。在函数内部，首先检查node函数是否成功。如果成功，使用pubSub函数发布数据。如果失败，返回相应的错误。

函数Subscribe接收一个名为api的PubSubAPI实例，一个名为topic的主题和一个或多个名为opts的PubSubSubscribeOption选项数组。函数使用checkNode函数来检查网络连接。如果连接成功，使用pubSub函数订阅指定主题。如果订阅成功，返回一个名为pubSubSubscription的包装器类型，如果没有更多的配置选项，则返回一个核心iface.PubSubSubscription对象。如果订阅失败，返回一个错误。


```
func (api *PubSubAPI) Publish(ctx context.Context, topic string, data []byte) error {
	_, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Publish", trace.WithAttributes(attribute.String("topic", topic)))
	defer span.End()

	_, err := api.checkNode()
	if err != nil {
		return err
	}

	//nolint deprecated
	return api.pubSub.Publish(topic, data)
}

func (api *PubSubAPI) Subscribe(ctx context.Context, topic string, opts ...caopts.PubSubSubscribeOption) (coreiface.PubSubSubscription, error) {
	_, span := tracing.Span(ctx, "CoreAPI.PubSubAPI", "Subscribe", trace.WithAttributes(attribute.String("topic", topic)))
	defer span.End()

	// Parse the options to avoid introducing silent failures for invalid
	// options. However, we don't currently have any use for them. The only
	// subscription option, discovery, is now a no-op as it's handled by
	// pubsub itself.
	_, err := caopts.PubSubSubscribeOptions(opts...)
	if err != nil {
		return nil, err
	}

	_, err = api.checkNode()
	if err != nil {
		return nil, err
	}

	//nolint deprecated
	sub, err := api.pubSub.Subscribe(topic)
	if err != nil {
		return nil, err
	}

	return &pubSubSubscription{sub}, nil
}

```

此代码定义了两个函数：`func (api *PubSubAPI) checkNode() (routing.Routing, error)` 和 `func (sub *pubSubSubscription) Close() error`。

第一个函数 `func (api *PubSubAPI) checkNode() (routing.Routing, error)` 的作用是检查给定的 `api` 是否为 `PubSubAPI` 类型，如果不是，则返回 `nil` 和 `errors.New` 函数的错误消息。具体实现是首先检查 `api.pubSub` 是否为 `nil`，如果是，则直接返回 `nil` 和 `errors.New` 函数的错误消息。否则，调用 `api.checkOnline(false)` 函数，如果该函数返回错误，则将返回该错误。最后，返回 `api.routing` 和 `nil` 作为结果，或者返回 `nil` 和 `errors.New` 函数的错误消息。

第二个函数 `func (sub *pubSubSubscription) Close() error` 的作用是关闭 `sub` 实例所订阅的 `pubSubSubscription`。具体实现是调用 `sub.subscription.Cancel()` 函数取消订阅，并返回一个错误。如果取消订阅成功，则返回 `nil`。


```
func (api *PubSubAPI) checkNode() (routing.Routing, error) {
	if api.pubSub == nil {
		return nil, errors.New("experimental pubsub feature not enabled, run daemon with --enable-pubsub-experiment to use")
	}

	err := api.checkOnline(false)
	if err != nil {
		return nil, err
	}

	return api.routing, nil
}

func (sub *pubSubSubscription) Close() error {
	sub.subscription.Cancel()
	return nil
}

```

此代码定义了一个名为`func`的函数，接收一个名为`pubSubSubscription`的参数，返回一个名为`pubSubMessage`的类型和一个名为`error`的类型的变量。函数的作用如下：

1. 创建一个名为`ctx`的上下文对象，使用`tracing.Span`方法将其记录为"CoreAPI.PubSubSubscription.Next"，然后使用`defer`关键字通知函数在函数内部关闭 spans。
2. 调用`sub.subscription.Next`函数，将接收到的上下文对象`ctx`传递给函数，并记录跟踪 span。
3. 如果函数返回值不为`nil`，则创建一个名为`pubSubMessage`的变量，并从函数返回值中获取消息。
4. 函数返回一个`pubSubMessage`类型的变量，如果函数返回错误，则返回错误。

`pubSubMessage`是一个用于在生产者和订阅者之间传递消息的`coreiface.PubSubMessage`类型，包含了接收到的消息和消息的元数据（如消息ID、发送者ID等）。

`From`方法的目的是获取`pubSubMessage`类型中`msg.msg.From`属性的值，即消息的发送者ID。


```
func (sub *pubSubSubscription) Next(ctx context.Context) (coreiface.PubSubMessage, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.PubSubSubscription", "Next")
	defer span.End()

	msg, err := sub.subscription.Next(ctx)
	if err != nil {
		return nil, err
	}

	return &pubSubMessage{msg}, nil
}

func (msg *pubSubMessage) From() peer.ID {
	return peer.ID(msg.msg.From)
}

```

这是一段 Go 语言中的函数指针变量（函数指针）类型的代码。它定义了一个名为 func 的函数，接收一个名为 msg 的整型指针参数，并返回一个字节数组类型的数据。

具体来说，这段代码实现了一个简单的数据序列化功能，将一个 pubSubMessage 类型的数据结构（可能是物联网设备传输的消息）封装成一个字节数组，并提供了访问数据结构中每个元素的方法。

在函数实现中，首先定义了一个名为 Data 的函数，接收一个名为 msg 的整型指针参数，然后返回一个字节数组类型的数据。这个函数的作用是获取并返回数据结构中的数据部分，即消息内容。

接着定义了一个名为 Seq 的函数，接收一个名为 msg 的整型指针参数，然后返回一个字节数组类型的数据。这个函数的作用是获取并返回数据结构中的序列号部分，即消息的唯一标识。

最后定义了一个名为 Topics 的函数，接收一个名为 msg 的整型指针参数，然后尝试获取并返回数据结构中的主题部分，即消息关联的主题名称。这个函数在没有主题参数的情况下，返回一个空字符串[]，作为没有主题的消息默认的处理方式。

整段代码的作用是获取并返回一个名为 msg 的 pubSubMessage 类型的数据结构，包括数据部分、序列号部分和主题部分，并将这些部分字节数组返回。


```
func (msg *pubSubMessage) Data() []byte {
	return msg.msg.Data
}

func (msg *pubSubMessage) Seq() []byte {
	return msg.msg.Seqno
}

func (msg *pubSubMessage) Topics() []string {
	// TODO: handle breaking downstream changes by returning a single string.
	if msg.msg.Topic == nil {
		return nil
	}
	return []string{*msg.msg.Topic}
}

```

# `/opt/kubo/core/coreapi/routing.go`

这段代码定义了一个名为 `RoutingAPI` 的 `CoreAPI` 类型，它使用 `CoreIPFS` 包作为其数据存储的后端，使用了 `strings` 包对输入的参数进行校验，并使用了 `peer` 包与对等网络中的 `peer` 进行通信。

`RoutingAPI` 实现了 `CoreAPI`  trait，提供了 `Get` 方法来获取指定键的值，该方法使用了 `Routing` 链中的路由器，根据输入的 `key` 在对等网络中查找相应的节点，然后返回该节点的 `value`。

如果网络连接不可用，则此方法返回 `coreiface.ErrOffline`。

另外，`RoutingAPI` 还实现了 `CA开放选项` 和 `peer.Transport开放选项` 以及 `RoutingAPI` 与其他 `CoreAPI` 类型的 `Combined` 方法。


```
package coreapi

import (
	"context"
	"errors"
	"strings"

	coreiface "github.com/ipfs/boxo/coreiface"
	caopts "github.com/ipfs/boxo/coreiface/options"
	peer "github.com/libp2p/go-libp2p/core/peer"
)

type RoutingAPI CoreAPI

func (r *RoutingAPI) Get(ctx context.Context, key string) ([]byte, error) {
	if !r.nd.IsOnline {
		return nil, coreiface.ErrOffline
	}

	dhtKey, err := normalizeKey(key)
	if err != nil {
		return nil, err
	}

	return r.routing.GetValue(ctx, dhtKey)
}

```

这段代码定义了一个名为 func 的函数，接收一个名为 r 的引用，以及一个字符串键值对和多个选项 (caopts.RoutingPutOption)。函数的作用是在不计超时时将一个字符串值通过键名为该函数传递给一个名为 routing 的路由器实例的 put 方法。

具体来说，这段代码实现以下步骤：

1. 通过调用 caopts.RoutingPutOptions 函数，接收一个名为 opts 的参数，该参数是一个包含了多个选项的选项切片。然后，函数检查选项是否都正确，如果错误，函数返回。
2. 如果所有选项都正确，函数执行以下操作：
	1. 通过调用 r.checkOnline 函数，检查路由器是否在线，如果是，函数继续执行。
	2. 如果路由器不在在线，函数执行以下操作：
		1. 通过调用 normalizeKey 函数，将传入的字符串键进行规范化处理，以便使用正常。
		2. 通过调用 r.routing.PutValue 函数，将字符串值和传递给路由器的选项作为参数，执行 put 操作。
	3. 返回前驱操作的错误。

这段代码中，normalizeKey 函数用于将传入的字符串键进行规范化处理，以保持与其他的 RoutingAPI 函数的一致性。r.routing.PutValue 函数是一个已经定义在前面答案中的函数，它是将传入的值和路由策略一起发送到路由器。


```
func (r *RoutingAPI) Put(ctx context.Context, key string, value []byte, opts ...caopts.RoutingPutOption) error {
	options, err := caopts.RoutingPutOptions(opts...)
	if err != nil {
		return err
	}

	err = r.checkOnline(options.AllowOffline)
	if err != nil {
		return err
	}

	dhtKey, err := normalizeKey(key)
	if err != nil {
		return err
	}

	return r.routing.PutValue(ctx, dhtKey, value)
}

```

该函数的作用是验证一个字符串是否可以作为一个键值对存储，键值对由三个部分组成，且必须包含"ipns"或"pk"之一。

函数首先将输入的字符串使用"/"为分隔符拆分成三个部分，并保存在一个名为"parts"的变量中。然后，函数检查"parts"是否包含三个部分，以及是否包含"ipns"或"pk"之一。如果不满足任何一个条件，函数将返回一个错误并退出。

如果函数成功通过上述检查，它将尝试使用"peer.Decode"函数将"ipns"或"pk"编码为实际的主机名。如果编码成功，函数将返回该主机名的字符串，否则返回一个错误并返回。

最后，函数将三个部分的字符串连接起来，如果成功，则返回，否则返回一个空字符串。


```
func normalizeKey(s string) (string, error) {
	parts := strings.Split(s, "/")
	if len(parts) != 3 ||
		parts[0] != "" ||
		!(parts[1] == "ipns" || parts[1] == "pk") {
		return "", errors.New("invalid key")
	}

	k, err := peer.Decode(parts[2])
	if err != nil {
		return "", err
	}
	return strings.Join(append(parts[:2], string(k)), "/"), nil
}

```

# `/opt/kubo/core/coreapi/swarm.go`

该代码是一个 Go 语言项目，名为 "coreapi"，它实现了 libp2p 库中的相关功能。

首先，它导入了多个依赖项：coreiface、sort、time、tracing、inet、peer、peerstore、protocol、swarm、ma 和 go-multiaddr。

接着，它定义了一个名为 "coreapi" 的包。

下面是 "coreapi" 包中定义的一些函数和变量：

1. coreiface. DialContextFn：该函数是一个上下文调用的函数，接收一个上下文（Context）和一系列选项（Options）。

2. coreiface. eventln：该函数将给定的任意式转换为 json 事件的字符串表示。

3. sort.Run：该函数是 sort 函数的别名，用于对给定的一组元素进行排序并返回排序后的结果。

4. time.毫秒：创建一个毫秒数。

5. time.秒：创建一个秒。

6. inet.千兆网络：通过 IP 地址和端口号创建一个 Inet 网络实例。

7. inet.IPV4 Tracer：设置跟踪 Inet 网络中数据包的发送者和接收者。

8. inet.IPV6 Tracer：设置跟踪 Inet 网络中数据包的发送者和接收者。

9. libp2p.CoreIPPServer：表示一个 Core IPP 服务器。

10. libp2p.CoreIPPServerpeer：表示一个 Core IPP 服务器中的对等方。

11. libp2p.PeerID：表示一个用于标识特定系统中 Core IPP 实体的 UUID。

12. libp2p.PeerStore：表示一个 Core IPP 实体的状态存储。

13. libp2p.Protocol：表示一个用于在 Core IPP 网络中传递数据包的协议。

14. libp2p.Swarm：表示一个用于在 Core IPP 网络中产生数据包的 Swarm。

15. libp2p.ma：表示一个用于在 libp2p 库中管理 Multi-Address 的人类接口。

16. libp2p.trace：表示一个用于跟踪数据包发送/接收事件的 libp2p 库。

17. libp2p.trace.Attribute：表示一个可以添加到跟踪事件的 Go 类型。

18. libp2p.trace.Span：表示一个跨函数调用的 Git 类型。


```
package coreapi

import (
	"context"
	"sort"
	"time"

	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/kubo/tracing"
	inet "github.com/libp2p/go-libp2p/core/network"
	"github.com/libp2p/go-libp2p/core/peer"
	pstore "github.com/libp2p/go-libp2p/core/peerstore"
	"github.com/libp2p/go-libp2p/core/protocol"
	"github.com/libp2p/go-libp2p/p2p/net/swarm"
	ma "github.com/multiformats/go-multiaddr"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"
)

```

这段代码定义了一个名为SwarmAPI的类型，该类型包含一个名为CoreAPI的类型。

接下来，定义了一个名为connInfo的结构体，其中包含一个peerstore类型类型的变量，一个inet.Conn类型类型的变量和一个int类型的变量，用于表示连接方向(目录方向默认为inout)。另外，还包含一个ma.Multiaddr类型的变量，用于表示连接地址，以及一个peer类型类型的变量，用于表示连接对等方ID。

接着，定义了一个名为connectionManagerTag的标签，其值为"user-connect"，用于在连接管理器中记录连接的标签，以便在日志中输出连接信息。定义了一个名为connectionManagerWeight的标签，其值为100，用于表示连接管理器在选择连接目标时的权重，值为更高的值将选择更快的连接。

最后，没有定义任何函数或其他变量，直接定义了需要使用的类型和标签。


```
type SwarmAPI CoreAPI

type connInfo struct {
	peerstore pstore.Peerstore
	conn      inet.Conn
	dir       inet.Direction

	addr ma.Multiaddr
	peer peer.ID
}

// tag used in the connection manager when explicitly connecting to a peer.
const (
	connectionManagerTag    = "user-connect"
	connectionManagerWeight = 100
)

```

此函数的作用是使SwarmAPI连接到远程主机，并设置连接参数。函数接受一个名为api的SwarmAPI实例和一个名为pi的PeerInfo结构体。

函数首先使用tracing工具链设置一个名为"CoreAPI.SwarmAPI"的跟踪，并设置一个名为"Connect"的子跟踪，使用当前上下文和PeerInfo实例的ID作为跟踪属性。然后检查swarm.Swarm类型对象的peerHost属性是否为nil，如果是，则函数返回一个名为"coreiface.ErrOffline"的错误。

接下来，函数检查swarm.Swarm类型对象中与peerHost关联的网络是否为 nil，然后使用该网络的Backoff清除方法清除peerHost.ID。然后，如果成功连接到远程主机，函数将使用peerHost.Connect方法连接到远程主机，并设置连接参数。最后，函数使用api.peerHost.ConnManager设置一个名为"connectionManagerTag"的标签，并设置连接管理器的权重，以便连接在网络中的其他地方。函数还返回一个nil，表示没有错误发生。


```
func (api *SwarmAPI) Connect(ctx context.Context, pi peer.AddrInfo) error {
	ctx, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Connect", trace.WithAttributes(attribute.String("peerid", pi.ID.String())))
	defer span.End()

	if api.peerHost == nil {
		return coreiface.ErrOffline
	}

	if swrm, ok := api.peerHost.Network().(*swarm.Swarm); ok {
		swrm.Backoff().Clear(pi.ID)
	}

	if err := api.peerHost.Connect(ctx, pi); err != nil {
		return err
	}

	api.peerHost.ConnManager().TagPeer(pi.ID, connectionManagerTag, connectionManagerWeight)
	return nil
}

```

这段代码是一个名为`SwarmAPI`的函数，接收一个`ctx`上下文和一个`ma.Multiaddr`类型的参数`addr`。函数的作用是断开与给定地址的`SwarmAPI`的连接，并返回错误。以下是函数的详细解释：

1. 函数接收两个参数：一个`SwarmAPI`类型的`api`和一个`ma.Multiaddr`类型的`addr`。这两个参数将在函数内部使用。
2. 函数开始时，使用`tracing.Span`函数创建一个名为"CoreAPI.SwarmAPI"的跟踪，并设置一个名为"Disconnect"的错误类型。错误将包含`addr`属性的值。
3. 如果`api.peerHost`为`nil`，函数将返回`coreiface.ErrOffline`错误。这是因为`peerHost`是远程主机，如果它为`nil`，那么函数无法断开与它的连接。
4. 如果`api.peerHost`不等于`nil`，函数将确定与给定地址的`SwarmAPI`的连接。首先，使用`peer.SplitAddr`函数将`addr`拆分为两个参数：`taddr`和`id`。然后，如果`id`为空，函数将返回`peer.ErrInvalidAddr`错误。否则，函数将设置`taddr`和`id`属性，以便跟踪器跟踪。
5. 接下来，函数创建一个`net`字段，使用`api.peerHost.Network()`获取与给定地址的`SwarmAPI`的网络。
6. 如果给定地址的连接性为`false`（即未连接或已断开），函数将尝试关闭套接字并返回错误。然后，如果发生错误，函数将返回错误。
7. 最后，函数遍历与给定地址的连接，如果连接不匹配，函数将返回错误。
8. 函数返回一个错误对象，其错误类型为`coreiface.ErrConnected`。


```
func (api *SwarmAPI) Disconnect(ctx context.Context, addr ma.Multiaddr) error {
	_, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Disconnect", trace.WithAttributes(attribute.String("addr", addr.String())))
	defer span.End()

	if api.peerHost == nil {
		return coreiface.ErrOffline
	}

	taddr, id := peer.SplitAddr(addr)
	if id == "" {
		return peer.ErrInvalidAddr
	}

	span.SetAttributes(attribute.String("peerid", id.String()))

	net := api.peerHost.Network()
	if taddr == nil {
		if net.Connectedness(id) != inet.Connected {
			return coreiface.ErrNotConnected
		}
		if err := net.ClosePeer(id); err != nil {
			return err
		}
		return nil
	}
	for _, conn := range net.ConnsToPeer(id) {
		if !conn.RemoteMultiaddr().Equal(taddr) {
			continue
		}

		return conn.Close()
	}
	return coreiface.ErrConnNotFound
}

```

此代码定义了一个名为"KnownAddrs"的函数，接收一个名为"api"的整数类型的指针和上下文上下文。函数返回一个名为"addrs"的"map"类型的值，其中包含与给定ID的"Multiaddr"列表，以及一个表示错误的"error"类型。

函数的主要部分如下：

1. 如果给定的"peerHost"为空，函数返回一个"error"。
2. 创建一个名为"addrs"的"map"类型的值，其中包含与给定ID的"Multiaddr"列表。
3. 对于给定的"peerHost"，通过调用"network().Peerstore()"获取与其相关的"ps"实例。
4. 通过遍历"ps.Peers()"返回的"Peer"结构，获取与给定ID相关的"Addr"结构，并将其添加到相应的"addrs" map中。
5. 对"addrs" map中的"Multiaddr"列表进行排序，以便按字符串顺序进行查找。

函数的作用是获取给定ID的"Multiaddr"列表，如果没有指定"peerHost"，则返回一个错误。


```
func (api *SwarmAPI) KnownAddrs(ctx context.Context) (map[peer.ID][]ma.Multiaddr, error) {
	_, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "KnownAddrs")
	defer span.End()

	if api.peerHost == nil {
		return nil, coreiface.ErrOffline
	}

	addrs := make(map[peer.ID][]ma.Multiaddr)
	ps := api.peerHost.Network().Peerstore()
	for _, p := range ps.Peers() {
		addrs[p] = append(addrs[p], ps.Addrs(p)...)
		sort.Slice(addrs[p], func(i, j int) bool {
			return addrs[p][i].String() < addrs[p][j].String()
		})
	}

	return addrs, nil
}

```

这两函数接收一个 SwarmAPI 类型的参数。它们的作用是让 SwarmAPI 与本地网络中的多地址参与组保持连接。

第一个函数 `LocalAddrs` 接收一个 `context.Context`。它由两个参数组成：一个是传递给该函数的上下文，另一个是字符串 "CoreAPI.SwarmAPI"。函数内部使用 `tracing.Span` 类型来记录请求的会话ID。函数主要的作用是检查 SwarmAPI 的 `peerHost` 字段是否为空。如果是，函数将返回一个空列表并抛出 `coreiface.ErrOffline` 错误。否则，函数返回 SwarmAPI 的 `peerHost` 的多地址，并将其返回。

第二个函数 `ListenAddrs` 与第一个函数类似，但它是通过监听而不是连接到本地网络来获取多地址。函数同样使用一个 `tracing.Span` 来记录请求的会话ID。函数的作用是获取 SwarmAPI 的 `peerHost` 并返回其网络中的接口列表。这个接口列表将包含可用于连接到 `peerHost` 的多地址。如果 `peerHost` 是空的，函数将返回一个空列表并抛出 `coreiface.ErrOffline` 错误。


```
func (api *SwarmAPI) LocalAddrs(ctx context.Context) ([]ma.Multiaddr, error) {
	_, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "LocalAddrs")
	defer span.End()

	if api.peerHost == nil {
		return nil, coreiface.ErrOffline
	}

	return api.peerHost.Addrs(), nil
}

func (api *SwarmAPI) ListenAddrs(ctx context.Context) ([]ma.Multiaddr, error) {
	_, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "ListenAddrs")
	defer span.End()

	if api.peerHost == nil {
		return nil, coreiface.ErrOffline
	}

	return api.peerHost.Network().InterfaceListenAddresses()
}

```

此代码定义了一个名为`Peers`的函数，接收一个名为`api`的`SwarmAPI`类型参数，返回一个包含`coreiface.ConnectionInfo`类型的切片和可能的错误。函数使用了`tracing.Span`和`defer`来自行踪跟踪和资源回收，函数的作用域是`func`。

函数的作用是获取与给定`SwarmAPI`的`peerHost`相连接的`coreiface.ConnectionInfo`。如果`peerHost`为空，函数返回一个空切片并抛出`coreiface.ErrOffline`异常。

函数内部首先检查`peerHost`是否为空，如果是，则直接返回一个空切片并抛出`coreiface.ErrOffline`异常。然后，使用一个循环遍历`peerHost`所连接的`coreiface.ConnectionInfo`。在循环内部，首先检查给定的`SwarmAPI`是否是一个`swarm.Conn`，如果是，则将其`muxer`字段设置为该`swarm.Conn`的`StreamConn()`方法返回的`conn`对象。然后，将该`swarm.Conn`的`StreamConn()`方法返回的`conn`对象添加到结果切片中。

最后，函数返回结果切片和可能的错误。


```
func (api *SwarmAPI) Peers(ctx context.Context) ([]coreiface.ConnectionInfo, error) {
	_, span := tracing.Span(ctx, "CoreAPI.SwarmAPI", "Peers")
	defer span.End()

	if api.peerHost == nil {
		return nil, coreiface.ErrOffline
	}

	conns := api.peerHost.Network().Conns()

	out := make([]coreiface.ConnectionInfo, 0, len(conns))
	for _, c := range conns {

		ci := &connInfo{
			peerstore: api.peerstore,
			conn:      c,
			dir:       c.Stat().Direction,

			addr: c.RemoteMultiaddr(),
			peer: c.RemotePeer(),
		}

		/*
			// FIXME(steb):
			swcon, ok := c.(*swarm.Conn)
			if ok {
				ci.muxer = fmt.Sprintf("%T", swcon.StreamConn().Conn())
			}
		*/

		out = append(out, ci)
	}

	return out, nil
}

```

这段代码定义了三个函数，接收一个名为ci的二维结构体作为参数。

ID()函数返回一个名为ci.peer的peer.ID结构体，表示与该连接相关的对等网络中的发送方ID。

Address()函数返回一个名为ci.addr的ma.Multiaddr结构体，表示与该连接相关的所有可到达的地址。

Direction()函数返回一个名为ci.dir的inet.Direction结构体，表示与该连接相关的网络方向。

Latency()函数返回一个名为ci.peerstore的错误类型的time.Duration结构体，表示与该连接相关的对等网络中的延迟。如果出现错误，该函数将返回nil。

函数的作用是将连接信息中的参数ci作为输入，并输出与该连接相关的信息，包括发送方ID、对等网络中的地址、网络方向和延迟。


```
func (ci *connInfo) ID() peer.ID {
	return ci.peer
}

func (ci *connInfo) Address() ma.Multiaddr {
	return ci.addr
}

func (ci *connInfo) Direction() inet.Direction {
	return ci.dir
}

func (ci *connInfo) Latency() (time.Duration, error) {
	return ci.peerstore.LatencyEWMA(peer.ID(ci.ID())), nil
}

```

此函数接收一个名为 ci 的连接信息参数，并返回一个包含协议 ID 的切片（即网络数据流）和错误。

它首先使用 ci.conn.GetStreams() 获取与连接相关的流。然后，使用 for 循环遍历所获得的流并将其存储在 out 切片中。最后，它返回 out 切片和 nil 错误。

函数的作用是获取与连接相关的流，并将其存储在一个名为 out 的数组中，该数组包含协议 ID。


```
func (ci *connInfo) Streams() ([]protocol.ID, error) {
	streams := ci.conn.GetStreams()

	out := make([]protocol.ID, len(streams))
	for i, s := range streams {
		out[i] = s.Protocol()
	}

	return out, nil
}

```

# `/opt/kubo/core/coreapi/unixfs.go`

该代码的作用是定义了一个名为 "coreapi" 的包，该包包含了一些与 Kubernetes 客户端和服务器相关的功能。

具体来说，该包通过导入不同的第三方库和函数，实现了以下功能：

1. 定义了一些类型变量，如 "ctx" 表示上下文，"fmt" 用于格式化输出，"sync" 用于并发，"trace" 用于记录跟踪信息等。

2. 通过导入 "github.com/ipfs/kubo/core" 和 "github.com/ipfs/kubo/tracing"，实现了与 Kubernetes API 服务器进行交互的功能。

3. 通过导入 "github.com/ipfs/boxo/blockservice" 和 "github.com/ipfs/boxo/blockstore"，实现了与分布式 storage 服务进行交互的功能。

4. 通过导入 "github.com/ipfs/boxo/coreiface" 和 "github.com/ipfs/boxo/options"，实现了与 Kubernetes 客户端进行交互的功能。

5. 通过导入 "github.com/ipfs/boxo/files" 和 "github.com/ipfs/boxo/filestore"，实现了文件和文件存储服务的管理功能。

6. 通过导入 "github.com/ipfs/boxo/merkledag" 和 "github.com/ipfs/boxo/ipld/merkledag/test"，实现了分布式数据结构的操作。

7. 通过导入 "github.com/ipfs/boxo/ipld/unixfs" 和 "github.com/ipfs/boxo/ipld/unixfs/file"，实现了文件系统的管理功能。

8. 通过导入 "github.com/ipfs/boxo/ipld/unixfs/io" 和 "github.com/ipfs/boxo/ipld/unixfs/file"，实现了与文件系统进行交互的功能。

9. 通过导入 "github.com/ipfs/boxo/mfs" 和 "github.com/ipfs/boxo/mfs/file"，实现了文件系统的管理功能。

10. 通过导入 "github.com/ipfs/boxo/cid" 和 "github.com/ipfs/go-cidutil"，实现了对 CID 对象的封装和管理。

11. 通过导入 "github.com/ipfs/boxo/cidutil/测试"，实现了对 CID 对象的测试功能。

12. 通过导入 "github.com/ipfs/boxo/ipld/ipld/merkledag"，实现了 Merkledag 对象的封装和管理。

13. 通过导入 "github.com/ipfs/boxo/ipld/ipld/merkledag/test"，实现了 Merkledag 对象的测试功能。

14. 通过导入 "github.com/ipfs/boxo/ipld/unixfs"，实现了与 Unix 文件系统进行交互的功能。

15. 通过导入 "github.com/ipfs/boxo/ipld/unixfs/file"，实现了与 Unix 文件系统进行交互的功能。

16. 通过导入 "github.com/ipfs/boxo/ipld/unixfs/io"，实现了与 Unix 文件系统进行交互的功能。

17. 通过导入 "github.com/ipfs/boxo/mfs"，实现了与文件系统进行交互的功能。

18. 通过导入 "github.com/ipfs/boxo/cid"，实现了对 CID 对象的封装和管理。


```
package coreapi

import (
	"context"
	"fmt"
	"sync"

	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/tracing"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/trace"

	"github.com/ipfs/kubo/core/coreunix"

	blockservice "github.com/ipfs/boxo/blockservice"
	bstore "github.com/ipfs/boxo/blockstore"
	coreiface "github.com/ipfs/boxo/coreiface"
	options "github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/files"
	filestore "github.com/ipfs/boxo/filestore"
	merkledag "github.com/ipfs/boxo/ipld/merkledag"
	dagtest "github.com/ipfs/boxo/ipld/merkledag/test"
	ft "github.com/ipfs/boxo/ipld/unixfs"
	unixfile "github.com/ipfs/boxo/ipld/unixfs/file"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"
	mfs "github.com/ipfs/boxo/mfs"
	"github.com/ipfs/boxo/path"
	cid "github.com/ipfs/go-cid"
	cidutil "github.com/ipfs/go-cidutil"
	ipld "github.com/ipfs/go-ipld-format"
)

```

这段代码定义了一个名为"UnixfsAPI CoreAPI"的类型，该类型包含两个方法：getOrCreateNilNode()和Once。

getOrCreateNilNode()方法的实现与题目描述中的代码相似，主要作用是创建一个空的IpfsNode实例，并在需要时创建实际的节点。其具体实现包括：检查是否已经创建了节点，如果已经创建，则返回现有的节点；如果尚未创建节点，则在背景上下文中创建一个新的节点，并将其返回。

Once方法的实现包含两个步骤：确保创建节点时已经设置好NilRepo字段，并且在创建失败时记录错误。这个字段用于指定Ipfs是否应该在内存中缓存已经创建的节点，以便在稍后调用已经创建的节点。


```
type UnixfsAPI CoreAPI

var (
	nilNode *core.IpfsNode
	once    sync.Once
)

func getOrCreateNilNode() (*core.IpfsNode, error) {
	once.Do(func() {
		if nilNode != nil {
			return
		}
		node, err := core.NewNode(context.Background(), &core.BuildCfg{
			// TODO: need this to be true or all files
			// hashed will be stored in memory!
			NilRepo: true,
		})
		if err != nil {
			panic(err)
		}
		nilNode = node
	})

	return nilNode, nil
}

```

This appears to be a function that creates a file adapter for a file system. The function takes a series of configuration settings, including a layout for the file system (such as balanced or trickle), and an option to enable or disable copy-on-write.

The function first checks the settings and creates a file adapter based on the selected layout. If the layout is not known, the function returns an error.

The function then creates the file system root if the `Inline` option is enabled, and adds all the files in the specified directory to the file system root.

Note that the function uses the `fileAdder.CidBuilder` to store a reference to the file system root, which is updated by the `fileAdder.AddAllAndPin` function if the files are added and pinned successfully.

The function returns the path to the new file system root, or an error if any issues arise during the creation process.


```
// Add builds a merkledag node from a reader, adds it to the blockstore,
// and returns the key representing that node.
func (api *UnixfsAPI) Add(ctx context.Context, files files.Node, opts ...options.UnixfsAddOption) (path.ImmutablePath, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Add")
	defer span.End()

	settings, prefix, err := options.UnixfsAddOptions(opts...)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	span.SetAttributes(
		attribute.String("chunker", settings.Chunker),
		attribute.Int("cidversion", settings.CidVersion),
		attribute.Bool("inline", settings.Inline),
		attribute.Int("inlinelimit", settings.InlineLimit),
		attribute.Bool("rawleaves", settings.RawLeaves),
		attribute.Bool("rawleavesset", settings.RawLeavesSet),
		attribute.Int("layout", int(settings.Layout)),
		attribute.Bool("pin", settings.Pin),
		attribute.Bool("onlyhash", settings.OnlyHash),
		attribute.Bool("fscache", settings.FsCache),
		attribute.Bool("nocopy", settings.NoCopy),
		attribute.Bool("silent", settings.Silent),
		attribute.Bool("progress", settings.Progress),
	)

	cfg, err := api.repo.Config()
	if err != nil {
		return path.ImmutablePath{}, err
	}

	// check if repo will exceed storage limit if added
	// TODO: this doesn't handle the case if the hashed file is already in blocks (deduplicated)
	// TODO: conditional GC is disabled due to it is somehow not possible to pass the size to the daemon
	//if err := corerepo.ConditionalGC(req.Context(), n, uint64(size)); err != nil {
	//	res.SetError(err, cmds.ErrNormal)
	//	return
	//}

	if settings.NoCopy && !(cfg.Experimental.FilestoreEnabled || cfg.Experimental.UrlstoreEnabled) {
		return path.ImmutablePath{}, fmt.Errorf("either the filestore or the urlstore must be enabled to use nocopy, see: https://github.com/ipfs/kubo/blob/master/docs/experimental-features.md#ipfs-filestore")
	}

	addblockstore := api.blockstore
	if !(settings.FsCache || settings.NoCopy) {
		addblockstore = bstore.NewGCBlockstore(api.baseBlocks, api.blockstore)
	}
	exch := api.exchange
	pinning := api.pinning

	if settings.OnlyHash {
		node, err := getOrCreateNilNode()
		if err != nil {
			return path.ImmutablePath{}, err
		}
		addblockstore = node.Blockstore
		exch = node.Exchange
		pinning = node.Pinning
	}

	bserv := blockservice.New(addblockstore, exch) // hash security 001
	dserv := merkledag.NewDAGService(bserv)

	// add a sync call to the DagService
	// this ensures that data written to the DagService is persisted to the underlying datastore
	// TODO: propagate the Sync function from the datastore through the blockstore, blockservice and dagservice
	var syncDserv *syncDagService
	if settings.OnlyHash {
		syncDserv = &syncDagService{
			DAGService: dserv,
			syncFn:     func() error { return nil },
		}
	} else {
		syncDserv = &syncDagService{
			DAGService: dserv,
			syncFn: func() error {
				ds := api.repo.Datastore()
				if err := ds.Sync(ctx, bstore.BlockPrefix); err != nil {
					return err
				}
				return ds.Sync(ctx, filestore.FilestorePrefix)
			},
		}
	}

	fileAdder, err := coreunix.NewAdder(ctx, pinning, addblockstore, syncDserv)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	fileAdder.Chunker = settings.Chunker
	if settings.Events != nil {
		fileAdder.Out = settings.Events
		fileAdder.Progress = settings.Progress
	}
	fileAdder.Pin = settings.Pin && !settings.OnlyHash
	fileAdder.Silent = settings.Silent
	fileAdder.RawLeaves = settings.RawLeaves
	fileAdder.NoCopy = settings.NoCopy
	fileAdder.CidBuilder = prefix

	switch settings.Layout {
	case options.BalancedLayout:
		// Default
	case options.TrickleLayout:
		fileAdder.Trickle = true
	default:
		return path.ImmutablePath{}, fmt.Errorf("unknown layout: %d", settings.Layout)
	}

	if settings.Inline {
		fileAdder.CidBuilder = cidutil.InlineBuilder{
			Builder: fileAdder.CidBuilder,
			Limit:   settings.InlineLimit,
		}
	}

	if settings.OnlyHash {
		md := dagtest.Mock()
		emptyDirNode := ft.EmptyDirNode()
		// Use the same prefix for the "empty" MFS root as for the file adder.
		err := emptyDirNode.SetCidBuilder(fileAdder.CidBuilder)
		if err != nil {
			return path.ImmutablePath{}, err
		}
		mr, err := mfs.NewRoot(ctx, md, emptyDirNode, nil)
		if err != nil {
			return path.ImmutablePath{}, err
		}

		fileAdder.SetMfsRoot(mr)
	}

	nd, err := fileAdder.AddAllAndPin(ctx, files)
	if err != nil {
		return path.ImmutablePath{}, err
	}

	if !settings.OnlyHash {
		if err := api.provider.Provide(nd.Cid()); err != nil {
			return path.ImmutablePath{}, err
		}
	}

	return path.FromCid(nd.Cid()), nil
}

```

此代码定义了一个名为"CoreAPI.UnixfsAPI"的函数，它接收一个名为"api"的指针参数和一个名为"p"的路径参数，并返回一个名为"files.Node"的文件节点和一个名为"error"的错误。

函数的核心逻辑是先获取一个与给定路径相关的ActiveSession，然后使用该ActiveSession获取指定路径的Node。如果ActiveSession或获取Node的过程出现错误，函数返回一个错误对象。

具体来说，函数的实现包括以下步骤：

1. 创建一个名为"CoreAPI.UnixfsAPI"的函数，并设置其参数为"api"和"p"。
2. 在函数内部，创建一个名为"ctx"的上下文，一个名为"span"的跟踪 spans，并设置其行为为" tracing.Span"。
3. 创建一个名为"ses"的名为"api.core().getSession"的函数，接收一个上下文上下文并返回一个名为"ActiveSession"的函数，并设置其行为为" tracing.Span"。
4. 使用"ActiveSession.ResolveNode"函数获取指定路径的节点，设置其行为为"unixfile.NewUnixfsFile"，并返回。
5. 如果步骤3或4中的任何一个出现错误，创建并返回一个"error"对象。


```
func (api *UnixfsAPI) Get(ctx context.Context, p path.Path) (files.Node, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Get", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	ses := api.core().getSession(ctx)

	nd, err := ses.ResolveNode(ctx, p)
	if err != nil {
		return nil, err
	}

	return unixfile.NewUnixfsFile(ctx, ses.dag, nd)
}

// Ls returns the contents of an IPFS or IPNS object(s) at path p, with the format:
```

这段代码定义了一个名为 `UnixfsAPI` 的接口，实现了 Unix 文件系统的递归列表 (ls) 功能。

在函数 `Ls` 中，首先调用父接口 `coreiface.DirEntry` 的 `Liste食人鱼` 方法，获取目录内容。然后设置 `resolvechildren` 选项为 `true`，以支持递归目录列表。接着获取当前会话设置并设置属性。然后从当前目录开始遍历设置的链接，如果链接无效，则返回 `uses.lsFromLinks` 函数。如果链接有效，则递归调用 `uses.lsFromLinksAsync` 函数，该函数会异步等待链接结果并返回。

函数参数：

- `ctx`：表示使用上下文；
- `p`：要列出目录内容的路径；
- `opts`：传递给 `options.UnixfsLsOptions` 的选项；
- `api`：`UnixfsAPI` 实例；
- `dagnode`：指定目录的父节点；
- `uses`：`UnixfsAPI` 实例；
- `ses`：Unix 文件系统会话；
- `dir`：指定当前目录。


```
// `<link base58 hash> <link size in bytes> <link name>`
func (api *UnixfsAPI) Ls(ctx context.Context, p path.Path, opts ...options.UnixfsLsOption) (<-chan coreiface.DirEntry, error) {
	ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "Ls", trace.WithAttributes(attribute.String("path", p.String())))
	defer span.End()

	settings, err := options.UnixfsLsOptions(opts...)
	if err != nil {
		return nil, err
	}

	span.SetAttributes(attribute.Bool("resolvechildren", settings.ResolveChildren))

	ses := api.core().getSession(ctx)
	uses := (*UnixfsAPI)(ses)

	dagnode, err := ses.ResolveNode(ctx, p)
	if err != nil {
		return nil, err
	}

	dir, err := uio.NewDirectoryFromNode(ses.dag, dagnode)
	if err == uio.ErrNotADir {
		return uses.lsFromLinks(ctx, dagnode.Links(), settings)
	}
	if err != nil {
		return nil, err
	}

	return uses.lsFromLinksAsync(ctx, dir, settings)
}

```

This is a Rust function that parses a `lnk` record from the `linkres` struct received from the `coreiface.DirEntry.ListChildren` method.

The `lnk` record has the following fields:

* `Name`: the name of the file or directory.
* `Cid`: a unique identifier for the file or directory.
* `Type`: the type of the file or directory, such as `coreiface.TFile`, `coreiface.TRaw`, `coreiface.TSDirectory`, or `coreiface.TSymlink`.
* `Size`: the size of the file or directory, if applicable.
* `Resource`: additional metadata associated with the file or directory, such as a symlink's target.

The function first checks whether the `lnk` record has a raw link, and if so, it retrieves the `lnk` record's parent node in the linked list using the `GetNode` method.

Then, it checks whether the `lnk` record's parent node is a `merkledag.ProtoNode` and, if it is, it attempts to retrieve the file's data from the `Data` method.

If the `lnk` record is a directory, the function checks whether `settings.ResolveChildren` is `true` and retrieves the linked children of the directory using the `GetChildren` method.

If the `lnk` record is a file, the function checks the type of the file, the size, and, if `settings.UseCumulativeSize` is `true`, it calculates the cumulative size.

The function then returns the `lnk` record.


```
func (api *UnixfsAPI) processLink(ctx context.Context, linkres ft.LinkResult, settings *options.UnixfsLsSettings) coreiface.DirEntry {
	ctx, span := tracing.Span(ctx, "CoreAPI.UnixfsAPI", "ProcessLink")
	defer span.End()
	if linkres.Link != nil {
		span.SetAttributes(attribute.String("linkname", linkres.Link.Name), attribute.String("cid", linkres.Link.Cid.String()))
	}

	if linkres.Err != nil {
		return coreiface.DirEntry{Err: linkres.Err}
	}

	lnk := coreiface.DirEntry{
		Name: linkres.Link.Name,
		Cid:  linkres.Link.Cid,
	}

	switch lnk.Cid.Type() {
	case cid.Raw:
		// No need to check with raw leaves
		lnk.Type = coreiface.TFile
		lnk.Size = linkres.Link.Size
	case cid.DagProtobuf:
		if settings.ResolveChildren {
			linkNode, err := linkres.Link.GetNode(ctx, api.dag)
			if err != nil {
				lnk.Err = err
				break
			}

			if pn, ok := linkNode.(*merkledag.ProtoNode); ok {
				d, err := ft.FSNodeFromBytes(pn.Data())
				if err != nil {
					lnk.Err = err
					break
				}
				switch d.Type() {
				case ft.TFile, ft.TRaw:
					lnk.Type = coreiface.TFile
				case ft.THAMTShard, ft.TDirectory, ft.TMetadata:
					lnk.Type = coreiface.TDirectory
				case ft.TSymlink:
					lnk.Type = coreiface.TSymlink
					lnk.Target = string(d.Data())
				}
				if !settings.UseCumulativeSize {
					lnk.Size = d.FileSize()
				}
			}
		}

		if settings.UseCumulativeSize {
			lnk.Size = linkres.Link.Size
		}
	}

	return lnk
}

```

此代码定义了一个名为 `lsFromLinksAsync` 的函数，接收三个参数：

1. `api`：代表一个 `UnixfsAPI` 类型的变量，该函数使用它来对目录 `dir` 中的链接进行递归扫描。
2. `ctx`：代表一个 `context.Context` 类型的变量，用于在函数执行期间管理任务。
3. `dir`：代表一个 `uio.Directory` 类型的变量，用于指定要递归扫描的目录。
4. `settings`：代表一个 `options.UnixfsLsSettings` 类型的变量，用于设置递归扫描的超时时间和不透明度设置。

函数的主要逻辑如下：

1. 创建一个名为 `out` 的通道，用于异步存储目录 `dir` 中的链接。
2. 调用 `dir.EnumLinksAsync` 函数，获取目录 `dir` 中所有链接的个数，并将其存储在通道 `out` 上。
3. 循环处理目录 `dir` 中的所有链接。
4. 遍历链接时，使用 `select` 语句将链接 `l` 发送到 `api.processLink` 函数，如果链接处理成功，则将其添加到 `out` 通道中。
5. 如果 `ctx.Done()` 信号被触发，函数将立即返回，不处理任何链接。

函数的实现使用了 `select` 语句和 `channel` 包来实现异步 I/O。`select` 语句允许在数据准备好后从不同的通道中读取数据，即使一些通道可能还没有准备好。`channel` 包允许在 `channel` 类型的通道上发送和接收数据，即使数据流量不确定。


```
func (api *UnixfsAPI) lsFromLinksAsync(ctx context.Context, dir uio.Directory, settings *options.UnixfsLsSettings) (<-chan coreiface.DirEntry, error) {
	out := make(chan coreiface.DirEntry, uio.DefaultShardWidth)

	go func() {
		defer close(out)
		for l := range dir.EnumLinksAsync(ctx) {
			select {
			case out <- api.processLink(ctx, l, settings): // TODO: perf: processing can be done in background and in parallel
			case <-ctx.Done():
				return
			}
		}
	}()

	return out, nil
}

```

该函数的作用是从给定的链接中列出子目录。它将每个链接作为参数传递给 `api.processLink` 函数，该函数将在当前上下文中处理链接并将其返回。

函数将返回一个通道，其中包含coreiface.DirEntry类型的目录项。如果任何错误发生，函数将返回 nil。

函数使用一个循环来遍历给定的链接。对于每个链接，它使用 `ft.LinkResult` 类型来获取链接信息，并将其作为参数传递给 `api.processLink` 函数。如果设置 `settings.Async` 为 true，则可以启用异步处理链接。

函数还使用 `close` 函数来关闭链接通道。

最后，函数返回一个指向 `CoreAPI` 类型的指针，该指针将代表一个核心 Unixfs API 实例。


```
func (api *UnixfsAPI) lsFromLinks(ctx context.Context, ndlinks []*ipld.Link, settings *options.UnixfsLsSettings) (<-chan coreiface.DirEntry, error) {
	links := make(chan coreiface.DirEntry, len(ndlinks))
	for _, l := range ndlinks {
		lr := ft.LinkResult{Link: &ipld.Link{Name: l.Name, Size: l.Size, Cid: l.Cid}}

		links <- api.processLink(ctx, lr, settings) // TODO: can be parallel if settings.Async
	}
	close(links)
	return links, nil
}

func (api *UnixfsAPI) core() *CoreAPI {
	return (*CoreAPI)(api)
}

```

这是一段使用Go中的`ipld.DAGService`类型来实现的同步有向无环图（DAG）服务的代码。`ipld.DAGService`类型提供了一些用于管理有向无环图的函数和标记，例如添加有向无环图、创建有向无环图实例等。

此处定义了一个名为`syncDagService`的结构体，它包含一个`ipld.DAGService`实例和一个名为`syncFn`的函数。`syncFn`函数用于返回一个阻塞错误的恢复操作，该操作可以在有向无环图的分支上执行。

`syncDagService`实例中的`syncFn`函数的作用是确保有向无环图的每个分支都被持久化到底层的数据存储系统。具体来说，当有向无环图的分支发生变化时，使用`syncFn`函数来通知底层数据存储系统进行更新。这样，即使是在应用程序内部的数据更新操作之后，底层数据存储系统也可以确保有向无环图的分支仍然与应用程序看到的是一致的。


```
// syncDagService is used by the Adder to ensure blocks get persisted to the underlying datastore
type syncDagService struct {
	ipld.DAGService
	syncFn func() error
}

func (s *syncDagService) Sync() error {
	return s.syncFn()
}

```

# `/opt/kubo/core/coreapi/test/api_test.go`

该代码包是一个 Go 语言项目，它旨在实现一个分布式哈希库（DHS）和数据存储库的测试框架。它包括以下主要部分：

1. 导入必要的库：通过导入 "testing"，我们可以使用一系列的测试框架，如 "range"，"unittest" 等。此外，还会导入其他必要的库，如 "base64" 和 "path/filepath"，这些库用于处理字符串和文件路径相关的问题。

2. 定义测试项目结构：通过定义一个名为 "test" 的包，我们可以将测试代码组织为一系列独立的测试函数。同时，定义了一个 "package" 定义，其中定义了 "test" 包的导入路径。

3. 导入共同的库：通过导入 "github.com/ipfs/boxo/bootstrap"，"github.com/ipfs/boxo/filestore" 和 "github.com/ipfs/boxo/keystore"，我们可以导入一系列与 DHS 和数据存储库相关的库。

4. 导入数据存储库的库：通过导入 "github.com/ipfs/boxo/node"，"github.com/ipfs/boxo/coreiface" 和 "github.com/ipfs/boxo/repo"，我们可以导入与 DHS 存储库相关的库。

5. 设置测试环境：通过定义一个名为 "testenv" 的结构体，我们可以设置测试的一些环境变量，如 "os" 设置我们想要运行测试的操作系统，"testing" 设置测试框架为 "unittest" 等。

6. 导入用于测试的库：通过导入 "github.com/ipfs/kubo/core"，"github.com/ipfs/kubo/core/coreapi" 和 "github.com/ipfs/kubo/node/libp2p"，我们可以导入一系列与 DHS 和数据存储库操作相关的库。

7. 定义测试函数：通过定义一个名为 "test" 的函数，我们可以编写测试用例。在本函数中，我们首先会调用 "boxo.Parse" 函数，将数据存储库的名称解析为一个 "bootstrap.Bootstrap" 对象。然后，我们可以设置一些实验参数，如 "bootstrap.膏复杂工夫"，"file.nal复合利率" 和 "file.章程目标基金费率" 等。接下来，我们可以使用 "盒子里走路" 的设计模式，测试如何创建一个自定义 "DHS" 实例并设置一些参数。最后，我们可以使用 "基准测试" 来评估 "DHS" 的性能。

8. 导出测试报告：通过定义一个名为 "testreport" 的函数，我们可以创建一个测试报告，评估基准测试的分数。

9.


```
package test

import (
	"context"
	"encoding/base64"
	"fmt"
	"os"
	"path/filepath"
	"testing"

	"github.com/ipfs/boxo/bootstrap"
	"github.com/ipfs/boxo/filestore"
	keystore "github.com/ipfs/boxo/keystore"
	"github.com/ipfs/kubo/core"
	"github.com/ipfs/kubo/core/coreapi"
	mock "github.com/ipfs/kubo/core/mock"
	"github.com/ipfs/kubo/core/node/libp2p"
	"github.com/ipfs/kubo/repo"

	coreiface "github.com/ipfs/boxo/coreiface"
	"github.com/ipfs/boxo/coreiface/tests"
	"github.com/ipfs/go-datastore"
	syncds "github.com/ipfs/go-datastore/sync"
	"github.com/ipfs/kubo/config"
	"github.com/libp2p/go-libp2p/core/crypto"
	"github.com/libp2p/go-libp2p/core/peer"
	mocknet "github.com/libp2p/go-libp2p/p2p/net/mock"
)

```

This is a Go program that configures a节点进行 PeerDiscovery with a Swarm DNS server. It uses the `coreapi` and `datastore` packages to interact with the Kubernetes repo system and file system, respectively.

The program creates a `Mock` object that implements the `coreapi.Node` interface. It has a `C` field that is used to configure the node's settings, such as the DNS server and the file store, and an `Experimental` field that enables the use of experimental features.

It also creates a `repo.Mock` object that implements the `datastore.Node` interface. This object is used to interact with the file system.

The program uses the `core.NewNode` function to create a new node and the `nodes` slice to store the nodes. It uses the `repo.LinkAll` function to link all the nodes together and the `keystore.NewMemKeystore` and `filestore.NewFileManager` functions to create the file system.

It also creates a new `repo.Mock` object for each node and uses the `coreapi.NewCoreAPI` function to create a new API for the node.

The program uses the `bsinf` configuration with the `nodes[0].Peerstore.PeerInfo` field to specify the DNS server and the `nodes[0].Identity` field to specify the file store. It uses the `for` loop to iterate over the nodes and the `if err` statement to handle any errors.


```
const testPeerID = "QmTFauExutTsy4XP6JbMFcw2Wa9645HJt2bTqL6qYDCKfe"

type NodeProvider struct{}

func (NodeProvider) MakeAPISwarm(t *testing.T, ctx context.Context, fullIdentity bool, online bool, n int) ([]coreiface.CoreAPI, error) {
	mn := mocknet.New()

	nodes := make([]*core.IpfsNode, n)
	apis := make([]coreiface.CoreAPI, n)

	for i := 0; i < n; i++ {
		var ident config.Identity
		if fullIdentity {
			sk, pk, err := crypto.GenerateKeyPair(crypto.RSA, 2048)
			if err != nil {
				return nil, err
			}

			id, err := peer.IDFromPublicKey(pk)
			if err != nil {
				return nil, err
			}

			kbytes, err := crypto.MarshalPrivateKey(sk)
			if err != nil {
				return nil, err
			}

			ident = config.Identity{
				PeerID:  id.String(),
				PrivKey: base64.StdEncoding.EncodeToString(kbytes),
			}
		} else {
			ident = config.Identity{
				PeerID: testPeerID,
			}
		}

		c := config.Config{}
		c.Addresses.Swarm = []string{fmt.Sprintf("/ip4/18.0.%d.1/tcp/4001", i)}
		c.Identity = ident
		c.Experimental.FilestoreEnabled = true

		ds := syncds.MutexWrap(datastore.NewMapDatastore())
		r := &repo.Mock{
			C: c,
			D: ds,
			K: keystore.NewMemKeystore(),
			F: filestore.NewFileManager(ds, filepath.Dir(os.TempDir())),
		}

		node, err := core.NewNode(ctx, &core.BuildCfg{
			Routing: libp2p.DHTServerOption,
			Repo:    r,
			Host:    mock.MockHostOption(mn),
			Online:  online,
			ExtraOpts: map[string]bool{
				"pubsub": true,
			},
		})
		if err != nil {
			return nil, err
		}
		nodes[i] = node
		apis[i], err = coreapi.NewCoreAPI(node)
		if err != nil {
			return nil, err
		}
	}

	err := mn.LinkAll()
	if err != nil {
		return nil, err
	}

	if online {
		bsinf := bootstrap.BootstrapConfigWithPeers(
			[]peer.AddrInfo{
				nodes[0].Peerstore.PeerInfo(nodes[0].Identity),
			},
		)

		for _, n := range nodes[1:] {
			if err := n.Bootstrap(bsinf); err != nil {
				return nil, err
			}
		}
	}

	return apis, nil
}

```

该代码是一个名为 "TestIface" 的函数，它属于 "testing" 包。它定义了一个函数 "TestIface"，用于对 "NodeProvider" 接口的测试。

具体来说，该函数接收一个参数 "t"，它代表测试框架中的 "transaction" 参数。然后，它使用 "tests.TestApi" 函数对 "NodeProvider" 接口进行测试，并将测试结果存储在变量 "t" 的引用 "t" 本身。

总而言之，该代码的作用是定义并实现了一个对 "NodeProvider" 接口的测试函数，用于在测试框架中测试 "NodeProvider" 接口的实现。


```
func TestIface(t *testing.T) {
	tests.TestApi(NodeProvider{})(t)
}

```

# `/opt/kubo/core/coreapi/test/path_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Boxo 项目中的 ipld 包。具体来说，这段代码：

1. 导入了所需的包。
2. 定义了一个名为 "test" 的包。
3. 导入了 "context"、"strconv"、"testing" 和 "time" 包。
4. 定义了一个名为 "testcase" 的函数，它是 "testing.T" 函数的子函数。
5. 导入 "github.com/ipfs/boxo/coreiface/options" 和 "github.com/ipfs/boxo/files" 包。
6. 导入 "github.com/ipfs/boxo/ipld/merkledag" 和 "github.com/ipfs/boxo/ipld/unixfs/io" 包。
7. 导入 "github.com/ipfs/boxo/path" 包。
8. 导入 "github.com/ipfs/boxo/ipld/go-ipld-prime" 包。
9. 定义了一个名为 "testcase" 的函数，该函数实现了 ipld 包中的一个名为 "fixture" 的函数。
10. 在 "testcase" 函数中，使用了 Go 标准库中的 "context" 和 "strconv" 函数。
11. 在 "testcase" 函数中，使用了一定量的 "time" 参数。
12. 在 "test" 包的 "main" 函数中，将所有 "testcase" 函数作为参数传递给 "testing.T" 函数。


```
package test

import (
	"context"
	"strconv"
	"testing"
	"time"

	"github.com/ipfs/boxo/coreiface/options"
	"github.com/ipfs/boxo/files"
	"github.com/ipfs/boxo/ipld/merkledag"
	uio "github.com/ipfs/boxo/ipld/unixfs/io"
	"github.com/ipfs/boxo/path"
	"github.com/ipld/go-ipld-prime"
)

```

This is a Go program that appears to be testing the resolving of a Merkle datacenter block failure. It appears to be using the UnixFS filesystem to store and retrieve files, and is using the IPLD (Internet Time猴驱动程序) to perform the file operations. It uses the ipld.Node type to represent the DagPB node that represents the file, and uses the *merkledag.ProtoNode to represent the IPLD struct that contains the file metadata. It appears to be trying to resolve each entry in the sharded directory which will result in pathing over the missing block by performing a search across the network.


```
func TestPathUnixFSHAMTPartial(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// Create a node
	apis, err := NodeProvider{}.MakeAPISwarm(t, ctx, true, true, 1)
	if err != nil {
		t.Fatal(err)
	}
	a := apis[0]

	// Setting this after instantiating the swarm so that it's not clobbered by loading the go-ipfs config
	prevVal := uio.HAMTShardingSize
	uio.HAMTShardingSize = 1
	defer func() {
		uio.HAMTShardingSize = prevVal
	}()

	// Create and add a sharded directory
	dir := make(map[string]files.Node)
	// Make sure we have at least two levels of sharding
	for i := 0; i < uio.DefaultShardWidth+1; i++ {
		dir[strconv.Itoa(i)] = files.NewBytesFile([]byte(strconv.Itoa(i)))
	}

	r, err := a.Unixfs().Add(ctx, files.NewMapDirectory(dir), options.Unixfs.Pin(false))
	if err != nil {
		t.Fatal(err)
	}

	// Get the root of the directory
	nd, err := a.Dag().Get(ctx, r.RootCid())
	if err != nil {
		t.Fatal(err)
	}

	// Make sure the root is a DagPB node (this API might change in the future to account for ADLs)
	_ = nd.(ipld.Node)
	pbNode := nd.(*merkledag.ProtoNode)

	// Remove one of the sharded directory blocks
	if err := a.Block().Rm(ctx, path.FromCid(pbNode.Links()[0].Cid)); err != nil {
		t.Fatal(err)
	}

	// Try and resolve each of the entries in the sharded directory which will result in pathing over the missing block
	//
	// Note: we could just check a particular path here, but it would require either greater use of the HAMT internals
	// or some hard coded values in the test both of which would be a pain to follow.
	for k := range dir {
		// The node will go out to the (non-existent) network looking for the missing block. Make sure we're erroring
		// because we exceeded the timeout on our query
		timeoutCtx, timeoutCancel := context.WithTimeout(ctx, time.Second*1)
		newPath, err := path.Join(r, k)
		if err != nil {
			t.Fatal(err)
		}

		_, err = a.ResolveNode(timeoutCtx, newPath)
		if err != nil {
			if timeoutCtx.Err() == nil {
				t.Fatal(err)
			}
		}
		timeoutCancel()
	}
}

```