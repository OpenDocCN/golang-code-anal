# trojan-go源码解析 6

# `proxy/stack.go`

这段代码定义了一个名为 "proxy" 的包，其中定义了一个名为 "Node" 的结构体节点类型。

节点类型包含以下字段：

- "Name": 节点的名称，用于标识节点。
- "Next": 节点next的映射，其中 next是键，值是下一个节点。
- "IsEndpoint": 指示节点是否为端点，如果为端点则不包括在next中。
- "Context": 上下文，用于记录与该节点相关的上下文。
- "tunnel.Server": 用于与服务器通信的代理服务器。
- "tunnel.Client": 用于与客户端通信的代理客户端。

这个包的作用是提供一个代理服务器，允许在发送消息到服务器时通过一个中转节点来保护消息的隐私，同时还可以在接收消息时提供更高的安全性。通过在 "Node" 类型中定义一个 "next" 字段，可以建立一个代理循环，允许节点将消息传递给下一个节点，直到达到 "IsEndpoint" 字段的设置，该节点将不再继续接收消息。


```go
package proxy

import (
	"context"

	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Node struct {
	Name       string
	Next       map[string]*Node
	IsEndpoint bool
	context.Context
	tunnel.Server
	tunnel.Client
}

```

该函数的作用是创建一个名为给定节点 `n` 的下一个节点，并返回它。

具体来说，函数首先检查给定的节点 `n` 的 `Next` 数组中是否存在名为给定名称 `name` 的节点。如果存在，就返回该节点。否则，函数调用 `tunnel.GetTunnel` 函数获取给定名称的隧道，并创建一个服务器对象 `s`。然后，将服务器对象用于创建一个新节点，并将其添加到 `n` 的 `Next` 数组中。最后，函数返回新节点。

函数的实现基于以下两个假设：

1. `n` 是一个 `Node` 类型的节点，它具有 `Next` 数组和 `Context` 字段，以及一个服务器对象 `s`。
2. `tunnel.GetTunnel` 函数用于获取给定名称的隧道。
3. `Node` 是一个 `Node` 类型，它包含一个名称字段和一个 `Next` 数组字段。

如果需要进一步解析函数的实现，请提供更多信息。


```go
func (n *Node) BuildNext(name string) *Node {
	if next, found := n.Next[name]; found {
		return next
	}
	t, err := tunnel.GetTunnel(name)
	if err != nil {
		log.Fatal(err)
	}
	s, err := t.NewServer(n.Context, n.Server)
	if err != nil {
		log.Fatal(err)
	}
	newNode := &Node{
		Name:    name,
		Next:    make(map[string]*Node),
		Context: n.Context,
		Server:  s,
	}
	n.Next[name] = newNode
	return newNode
}

```

该函数的作用是获取一个节点树中的两个节点之间的物理连接，并返回该连接的下一个节点。

具体实现包括以下几个步骤：

1. 如果已经访问过了目标节点的下一个节点，则直接返回该节点。
2. 如果还没有访问过目标节点，则创建一个新节点并将目标节点添加到节点的下一个节点中。
3. 如果尝试连接目标节点时出现错误，则记录错误并返回原节点，以便进行错误处理。
4. 通过调用t.GetTunnel函数获取目标节点的隧道，并使用t.NewServer函数创建一个新的服务器。
5. 将服务器连接到目标节点的上下文上下文中，并将目标节点的服务器设置为新的服务器。
6. 返回目标节点本身，以便后续操作使用。


```go
func (n *Node) LinkNextNode(next *Node) *Node {
	if next, found := n.Next[next.Name]; found {
		return next
	}
	n.Next[next.Name] = next
	t, err := tunnel.GetTunnel(next.Name)
	if err != nil {
		log.Fatal(err)
	}
	s, err := t.NewServer(next.Context, n.Server) // context of the child nodes have been initialized
	if err != nil {
		log.Fatal(err)
	}
	next.Server = s
	return next
}

```

这两段代码都是定义在函数中的函数，函数名为 `FindAllEndpoints` 。

函数的作用是返回一个数组，该数组包含了所有与给定根节点 `root` 相连接的端点的 `tunnel.Server` 。函数的实现包括以下步骤：

1. 通过 `make` 函数创建一个空列表 `list`，该列表将存储所有的端点。

2. 如果根节点 `root` 是一个端点或者它的 `Next` 字段为空，则将根节点 `root` 添加到列表中。

3. 对于每个连接到 `root` 的端点 `next`:

a. 将 `next` 添加到列表中。

b. 使用 `FindAllEndpoints` 函数递归地查找所有与 `next` 相连接的端点。

4. 返回列表 `list`。

函数 `CreateClientStack` 作用是创建一个客户端栈。函数的实现包括以下步骤：

1. 定义一个变量 `client` 并初始化为一个 tunnel.`Client`。

2. 通过循环遍历 `clientStack` 数组中的每个名称，使用 `tunnel.GetTunnel` 函数获取与该名称对应的 tunnel 对象，并使用 `NewClient` 函数创建一个新的 `Client` 对象。

3. 对于每个获取到的 tunnel 对象，使用 `tunnel.GetServer` 函数获取与该 tunnel 对象相连接的端点。

4. 将获取到的端点添加到 `client` 对象中。

5. 如果循环结束后仍然没有找到任何与给定根节点 `root` 相连接的端点，则返回一个 `nil` 表示错误。


```go
func FindAllEndpoints(root *Node) []tunnel.Server {
	list := make([]tunnel.Server, 0)
	if root.IsEndpoint || len(root.Next) == 0 {
		list = append(list, root.Server)
	}
	for _, next := range root.Next {
		list = append(list, FindAllEndpoints(next)...)
	}
	return list
}

// CreateClientStack create client tunnel stacks from lists
func CreateClientStack(ctx context.Context, clientStack []string) (tunnel.Client, error) {
	var client tunnel.Client
	for _, name := range clientStack {
		t, err := tunnel.GetTunnel(name)
		if err != nil {
			return nil, err
		}
		client, err = t.NewClient(ctx, client)
		if err != nil {
			return nil, err
		}
	}
	return client, nil
}

```

这段代码定义了一个名为CreateServerStack的函数，它接受一个字符串类型的服务器隧道栈列表作为参数，并返回一个隧道服务器和一个错误。

函数首先定义了一个名为server的变量，该变量类型为tunnel.Server，表示一个隧道服务器。然后，它遍历服务器隧道栈列表，对于每个条目，它首先尝试从上下文中获取隧道服务器，然后使用该服务器创建一个新的服务器。如果过程中出现任何错误，函数将返回一个非空错误。最后，函数返回服务器和错误都为 nil的情况。

总的来说，这段代码的主要目的是创建一个服务器隧道栈，并确保在创建服务器时不会出现错误。


```go
// CreateServerStack create server tunnel stack from list
func CreateServerStack(ctx context.Context, serverStack []string) (tunnel.Server, error) {
	var server tunnel.Server
	for _, name := range serverStack {
		t, err := tunnel.GetTunnel(name)
		if err != nil {
			return nil, err
		}
		server, err = t.NewServer(ctx, server)
		if err != nil {
			return nil, err
		}
	}
	return server, nil
}

```

# `proxy/client/client.go`

该代码包定义了一个名为 "client" 的包。它通过导入多个其他包中的类型实现了一个简单的代理结构。通过为不同代理提供不同的代理实现，该代码包使得可以创建不同类型的代理，从而可以对网络流量进行代理。

具体来说，该代码包中定义了一些类型，如 "Context"、"HTTPProxy"、"HTTPS代理"、"WebSocket代理" 和 "SSH代理"。这些类型实现了代理、代理实现和代理接口的功能。此外，该代码包中还定义了一些方法，用于设置和获取网络代理的配置和选项。

该代码包中的函数可以用来创建一个代理实例，指定代理的类型、目标 URL 和选项，然后使用该代理发送网络请求。通过使用不同的代理实现，可以对网络流量进行不同的处理，如加密、中转、加速等。


```go
package client

import (
	"context"

	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy"
	"github.com/p4gefau1t/trojan-go/tunnel/adapter"
	"github.com/p4gefau1t/trojan-go/tunnel/http"
	"github.com/p4gefau1t/trojan-go/tunnel/mux"
	"github.com/p4gefau1t/trojan-go/tunnel/router"
	"github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"
	"github.com/p4gefau1t/trojan-go/tunnel/simplesocks"
	"github.com/p4gefau1t/trojan-go/tunnel/socks"
	"github.com/p4gefau1t/trojan-go/tunnel/tls"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
	"github.com/p4gefau1t/trojan-go/tunnel/trojan"
	"github.com/p4gefau1t/trojan-go/tunnel/websocket"
)

```

这段代码定义了一个名为 `GenerateClientTree` 的函数，它接受六个参数，分别是 `transportPlugin`、`muxEnabled`、`wsEnabled`、`ssEnabled` 和 `routerEnabled`，它们都是布尔类型的变量。函数的作用是生成一个客户端树，将各种网络协议栈（transport、tls、websocket、shadowsocks、trojan 和 router）的名称添加到客户端树中。

具体来说，这段代码首先判断 `transportPlugin` 是否为真，如果是，则将 TLS 协议栈中的 `transport` 名称添加到客户端树中；如果不是，则将所有添加到客户端树中的协议栈名称都设置为 `transport`。接下来，函数会判断 `wsEnabled` 和 `ssEnabled` 是否为真，如果是，则将 WebSocket 和 Shadowsocks 协议栈中的 `websocket` 和 `shadowsocks` 名称添加到客户端树中；如果不是，则不添加。

接着，函数会判断 `muxEnabled` 和 `routerEnabled` 是否为真，如果是，则将 MaxMU 和路由器协议栈中的 `mux` 和 `router` 名称添加到客户端树中；如果不是，则不添加。

最后，函数会将所有生成的客户端树返回给调用者。


```go
const Name = "CLIENT"

// GenerateClientTree generate general outbound protocol stack
func GenerateClientTree(transportPlugin bool, muxEnabled bool, wsEnabled bool, ssEnabled bool, routerEnabled bool) []string {
	clientStack := []string{transport.Name}
	if !transportPlugin {
		clientStack = append(clientStack, tls.Name)
	}
	if wsEnabled {
		clientStack = append(clientStack, websocket.Name)
	}
	if ssEnabled {
		clientStack = append(clientStack, shadowsocks.Name)
	}
	clientStack = append(clientStack, trojan.Name)
	if muxEnabled {
		clientStack = append(clientStack, []string{mux.Name, simplesocks.Name}...)
	}
	if routerEnabled {
		clientStack = append(clientStack, router.Name)
	}
	return clientStack
}

```

这段代码定义了一个名为`init`的函数，用于创建一个代理对象。函数内部使用了一个代理创建器函数，接收一个上下文`ctx`和代理配置`config`作为参数。

在该函数内部，首先创建一个代理节点`root`，该节点使用代理创建器函数创建一个代理对象，并将其添加到代理对象树中的`next`字段中。`root`节点还设置其`IsEndpoint`字段为`false`，表示它不是一个端点，因此不会使用代理服务器进行流量转发。

然后，设置`root`节点的`next`字段为代理服务器地址和`socks`代理服务器的地址，这两个字段都是通过调用`adapter.NewServer`函数获得的。使用上下文`ctx`创建一个`代理服务器`并将其添加到`root.next`字段的`isEndpoint`字段中。

接下来，使用代理创建器函数中的配置`config`来创建一个`ClientStack`对象，并将其添加到`root.next`字段的`isEndpoint`字段中。`ClientStack`对象用于创建代理对象树中的客户端代理服务器。

最后，使用代理对象树中的`root`节点来获取所有的端点，并将其返回。如果创建代理对象树过程中出现错误，则返回一个`nil`表示出错。

整个函数的作用是创建一个代理对象树，并返回该对象。


```go
func init() {
	proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
		cfg := config.FromContext(ctx, Name).(*Config)
		adapterServer, err := adapter.NewServer(ctx, nil)
		if err != nil {
			return nil, err
		}
		ctx, cancel := context.WithCancel(ctx)

		root := &proxy.Node{
			Name:       adapter.Name,
			Next:       make(map[string]*proxy.Node),
			IsEndpoint: false,
			Context:    ctx,
			Server:     adapterServer,
		}

		root.BuildNext(http.Name).IsEndpoint = true
		root.BuildNext(socks.Name).IsEndpoint = true

		clientStack := GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, cfg.Router.Enabled)
		c, err := proxy.CreateClientStack(ctx, clientStack)
		if err != nil {
			cancel()
			return nil, err
		}
		s := proxy.FindAllEndpoints(root)
		return proxy.NewProxy(ctx, cancel, s, c), nil
	})
}

```

# `proxy/client/config.go`

这是一个 Go 语言的代码包 client，它定义了三个 struct 类型：MuxConfig、WebsocketConfig 和 RouterConfig。这些 struct 类型都包含一个名为 enabled 的布尔值，用于控制是否启用某种配置。

同时，该代码包引入了两个来自github.com/p4gefau1t/trojan-go的库：config和trojan-go/config。这两个库可能是一些配置文件的设置和读取的库。

另外，该代码包没有输出任何配置文件或者 config 文件的路径，也没有输出任何错误信息。

总的来说，该代码包可能是为了在 trojan-go 应用中设置一些网络或 WebSocket 相关的配置，然后在应用程序中根据配置文件的内容来决定是否启用这些配置。


```go
package client

import "github.com/p4gefau1t/trojan-go/config"

type MuxConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

type WebsocketConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

type RouterConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

```

这段代码定义了一个名为 Config 的结构体，它包含了以下字段：

* Enabled：一个布尔值，表示是否启用了 Shadowsocks 和 TransportPlugin。
* Mux：一个 MuxConfig 结构体，包含了启用 Multiplex 的配置。
* Websocket：一个 WebsocketConfig 结构体，包含了启用 WebSocket 的配置。
* Router：一个 RouterConfig 结构体，包含了路由器的配置。
* Shadowsocks：一个 ShadowsocksConfig 结构体，包含了 Shadowsocks 的配置。
* TransportPlugin：一个 TransportPluginConfig 结构体，包含了 TransportPlugin 的配置。

通过声明这些字段，这段代码定义了一个 Config 结构体，它代表了一个完整的 Web 应用程序集，包含了多个组件的配置，如 Multiplex、WebSocket 和路由器。


```go
type ShadowsocksConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

type TransportPluginConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

type Config struct {
	Mux             MuxConfig             `json:"mux" yaml:"mux"`
	Websocket       WebsocketConfig       `json:"websocket" yaml:"websocket"`
	Router          RouterConfig          `json:"router" yaml:"router"`
	Shadowsocks     ShadowsocksConfig     `json:"shadowsocks" yaml:"shadowsocks"`
	TransportPlugin TransportPluginConfig `json:"transport_plugin" yaml:"transport-plugin"`
}

```

这段代码是使用Go语言中的函数式编程范式(Functional Programming Paradigm)编写的。其作用是注册一个名为"init"的函数，用于初始化一个名为"config"的配置对象。

具体来说，该代码创建了一个名为"Config"的接口类型，并将其作为参数传入函数内部的一个匿名函数中。这个匿名函数返回一个指向Config类型对象的指针，也就是说，它返回了一个具体的Config实例。

然后，该函数将刚刚创建的匿名函数注册到了配置对象的"RegisterConfigCreator"方法中，使得当调用该函数时，可以通过传入一个名称参数来指定要注册的配置创建者。

这段代码主要目的是创建一个用于初始化配置对象的函数，该函数将在函数第一次调用时创建一个新的Config实例，并将其名称传递给配置对象的"RegisterConfigCreator"方法。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(Config)
	})
}

```

# `proxy/custom/config.go`

这段代码定义了一个名为 "custom" 的包，其中包含了一个名为 "NodeConfig" 的结构体类型，以及一个名为 "StackConfig" 的结构体类型。

具体来说，这段代码实现了以下功能：

1. 定义了一个名为 "CUSTOM" 的包。
2. 导入了 "github.com/p4gefau1t/trojan-go/config" 包，以便使用其中的配置文件。
3. 定义了一个名为 "Name" 的常量，用于指定该包的唯一名称。
4. 定义了一个名为 "NodeConfig" 的结构体类型，其中包含一个名为 "protocol" 的字段，它是一个字符串类型，用于指定节点的协议类型；另一个名为 "tag" 的字段，它也是一个字符串类型，用于指定节点的标签；第三个字段 "config" 是接口类型，用于指定节点的配置信息。
5. 定义了一个名为 "StackConfig" 的结构体类型，其中包含一个名为 "path" 的字段，它是一个数组，用于指定栈文件的路径；另一个字段 "node" 是一个数组，包含一个 "NodeConfig" 类型的字段，用于指定每个栈节点的配置信息。


```go
package custom

import "github.com/p4gefau1t/trojan-go/config"

const Name = "CUSTOM"

type NodeConfig struct {
	Protocol string      `json:"protocol" yaml:"protocol"`
	Tag      string      `json:"tag" yaml:"tag"`
	Config   interface{} `json:"config" yaml:"config"`
}

type StackConfig struct {
	Path [][]string   `json:"path" yaml:"path"`
	Node []NodeConfig `json:"node" yaml:"node"`
}

```

这段代码定义了一个名为Config的结构体，包含了两个字段，分别是"inbound"和"outbound"，它们的JSON和YAML同义词分别是"inbound"和"outbound"。该结构体定义了一个Config类型的变量，用于保存网络的入站和出站设置。

在代码的初始化部分，使用RegisterConfigCreator函数将该Config类型的变量映射为Config类型的函数创建器，当创建器函数被调用时，将返回一个Config类型的实例，以便将其赋值给inbound和outbound变量。这个函数将该Config实例作为参数，并将其返回的Config类型实例作为函数的返回值，以便将其赋值给inbound和outbound变量。

如果网络配置被更改，入站和出站设置将被更新，并将新的设置存储在Config中。如果网络配置没有更改，该函数将返回现有的Config类型，以便将其赋值给inbound和outbound变量。


```go
type Config struct {
	Inbound  StackConfig `json:"inbound" yaml:"inbound"`
	Outbound StackConfig `json:"outbound" yaml:"outbound"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(Config)
	})
}

```

# `proxy/custom/custom.go`

这段代码定义了一个名为 "custom" 的包，其中包括了一些函数和变量。

import 函数从标准库 "gopkg.in/yaml.v3" 中导入了一个名为 "map" 的类型，并定义了一个名为 "convert" 的函数，该函数接受一个接口类型的参数 "i"。

convert 函数的实现比较复杂，如果 "i" 是一个包含键值对 "map" 类型的值，那么 convert 函数会将传入的每个键值对 "map" 类型中的键 "string" 进行转换，并返回一个新的 "map" 类型。如果 "i" 是一个包含 "[]interface{}" 类型的值，那么 convert 函数会将传入的每个 "interface{}" 类型中的值 "convert" 函数，并将结果返回。

最后，convert 函数返回原始的 "i" 类型，因为转换后的结果 "map" 类型与原始类型 "map" 类型相同。


```go
package custom

import (
	"context"
	"strings"

	"gopkg.in/yaml.v3"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

func convert(i interface{}) interface{} {
	switch x := i.(type) {
	case map[interface{}]interface{}:
		m2 := map[string]interface{}{}
		for k, v := range x {
			m2[k.(string)] = convert(v)
		}
		return m2
	case []interface{}:
		for i, v := range x {
			x[i] = convert(v)
		}
	}
	return i
}

```

这段代码定义了一个名为 `buildNodes` 的函数，它接受一个名为 `ctx` 的上下文对象和一个名为 `nodeConfigList` 的 slice 参数。

函数的主要目的是创建一个代理的 `Node` 对象，该对象包含一个键值映射，其中键是每个节点的标签（或名称），值是一个指向 `proxy.Node` 类型对象的引用。

函数的具体实现包括以下步骤：

1. 创建一个空代理的 `Node` 对象，并将其添加到 `nodes`  map 中。
2. 遍历 `nodeConfigList` slice，对于每个节点配置，执行以下操作：
a. 将节点的协议名称转换为大写，以增加安全性。
b. 获取隧道实例并尝试获取配置数据。
c. 将配置数据 YAML 编码并使用 `config.WithYAMLConfig` 函数解析。
d. 创建一个 `proxy.Node` 对象，并将其添加到 `nodes`  map 中，同时将 `Context` 字段设置为解析出的 `config` 实例。

函数的返回值是一个包含每个节点代理对象的 `nodes` map 和一个表示成功的 `error` 类型的空 `nil`。如果函数在执行过程中遇到任何错误，它将返回 `nil`，否则它将返回一个包含所有节点的代理对象的 `nodes`  map。


```go
func buildNodes(ctx context.Context, nodeConfigList []NodeConfig) (map[string]*proxy.Node, error) {
	nodes := make(map[string]*proxy.Node)
	for _, nodeCfg := range nodeConfigList {
		nodeCfg.Protocol = strings.ToUpper(nodeCfg.Protocol)
		if _, err := tunnel.GetTunnel(nodeCfg.Protocol); err != nil {
			return nil, common.NewError("invalid protocol name:" + nodeCfg.Protocol)
		}
		data, err := yaml.Marshal(nodeCfg.Config)
		common.Must(err)
		nodeContext, err := config.WithYAMLConfig(ctx, data)
		if err != nil {
			return nil, common.NewError("failed to parse config data for " + nodeCfg.Tag + " with protocol" + nodeCfg.Protocol).Base(err)
		}
		node := &proxy.Node{
			Name:    nodeCfg.Protocol,
			Next:    make(map[string]*proxy.Node),
			Context: nodeContext,
		}
		nodes[nodeCfg.Tag] = node
	}
	return nodes, nil
}

```

This is a Go language program that creates a proxy for a server that uses the TLS 1.2 protocol stack and routes traffic to the specified outbound protocol stack. It uses the built-in Go-as-告诉我(gRPC) package to create the gRPC server and gRPC client.

The program takes an option to specify the path of the outbound protocol stack. It assumes that the outbound stack is defined in a separate configuration file.

The program first sets up a server that listens for incoming traffic. It creates a loopback server that listens for incoming traffic and uses it to create a client to the TLS 1.2 protocol stack.

It then creates a gRPC server and client for the specified outbound protocol stack using the information from the TLS 1.2 protocol stack.

Finally, it starts the server and handles incoming traffic by matching it to the appropriate outbound node. It also checks if the outbound stack is configured correctly and returns an error if it's not.


```go
func init() {
	proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
		cfg := config.FromContext(ctx, Name).(*Config)

		ctx, cancel := context.WithCancel(ctx)
		success := false
		defer func() {
			if !success {
				cancel()
			}
		}()
		// inbound
		nodes, err := buildNodes(ctx, cfg.Inbound.Node)
		if err != nil {
			return nil, err
		}

		var root *proxy.Node
		// build server tree
		for _, path := range cfg.Inbound.Path {
			var lastNode *proxy.Node
			for _, tag := range path {
				if _, found := nodes[tag]; !found {
					return nil, common.NewError("invalid node tag: " + tag)
				}
				if lastNode == nil {
					if root == nil {
						lastNode = nodes[tag]
						root = lastNode
						t, err := tunnel.GetTunnel(root.Name)
						if err != nil {
							return nil, common.NewError("failed to find root tunnel").Base(err)
						}
						s, err := t.NewServer(root.Context, nil)
						if err != nil {
							return nil, common.NewError("failed to init root server").Base(err)
						}
						root.Server = s
					} else {
						lastNode = root
					}
				} else {
					lastNode = lastNode.LinkNextNode(nodes[tag])
				}
			}
			lastNode.IsEndpoint = true
		}

		servers := proxy.FindAllEndpoints(root)

		if len(cfg.Outbound.Path) != 1 {
			return nil, common.NewError("there must be only 1 path for outbound protocol stack")
		}

		// outbound
		nodes, err = buildNodes(ctx, cfg.Outbound.Node)
		if err != nil {
			return nil, err
		}

		// build client stack
		var client tunnel.Client
		for _, tag := range cfg.Outbound.Path[0] {
			if _, found := nodes[tag]; !found {
				return nil, common.NewError("invalid node tag: " + tag)
			}
			t, err := tunnel.GetTunnel(nodes[tag].Name)
			if err != nil {
				return nil, common.NewError("invalid tunnel name").Base(err)
			}
			client, err = t.NewClient(nodes[tag].Context, client)
			if err != nil {
				return nil, common.NewError("failed to create client").Base(err)
			}
		}

		success = true
		return proxy.NewProxy(ctx, cancel, servers, client), nil
	})
}

```

# `proxy/forward/forward.go`

这段代码是一个名为“forward”的包的初始化函数。它通过注册代理创建器来创建一个名为“FORWARD”的代理。

具体来说，这个初始化函数接收一个上下文对象和一个配置对象（来自命名空间“client”的配置结构）。它使用上下文对象和配置对象来创建代理实例，并使用配置对象中提供的选项来设置代理的连接。

它还创建了一个服务器链，用于在代理和客户端之间建立连接。在服务器链中，客户端将连接到代理，而代理将连接到客户端的服务器。

最后，它返回一个代理实例和一个表示服务器是否已经连接好的布尔值。如果代理成功创建并连接到服务器，则返回代理实例，否则返回 nil 和一个表示错误消息的错误。


```go
package forward

import (
	"context"

	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy"
	"github.com/p4gefau1t/trojan-go/proxy/client"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/dokodemo"
)

const Name = "FORWARD"

func init() {
	proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
		cfg := config.FromContext(ctx, Name).(*client.Config)
		ctx, cancel := context.WithCancel(ctx)
		serverStack := []string{dokodemo.Name}
		clientStack := client.GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, cfg.Router.Enabled)
		c, err := proxy.CreateClientStack(ctx, clientStack)
		if err != nil {
			cancel()
			return nil, err
		}
		s, err := proxy.CreateServerStack(ctx, serverStack)
		if err != nil {
			cancel()
			return nil, err
		}
		return proxy.NewProxy(ctx, cancel, []tunnel.Server{s}, c), nil
	})
}

```

这段代码定义了一个名为 "init" 的函数，它在函数内部创建了一个名为 "client" 的配置创建器。

具体来说，这段代码创建了一个名为 "client" 的局部变量，并将其与一个名为 "config" 的全局变量进行绑定。然后，它创建了一个名为 "Name" 的函数指针，并将其与 "client" 局部变量中的 "func" 键绑定。

接着，函数指针内部的代码创建了一个新的 "client.Config" 类型，并将其绑定到名为 "client" 的全局变量上。最后，它返回了一个指向 "client.Config" 的指针，这个指针类型就是我们所期望的接口类型。

总之，这段代码的主要作用是创建了一个名为 "client" 的配置创建器，该创建器由名为 "init" 的函数进行初始化。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(client.Config)
	})
}

```

# `proxy/nat/nat.go`

这段代码是一个 Go 语言编写的 build 脚本，用于在 Go 环境中编译一个名为 "nat" 的 package。它包含两个部分：

1. //go:build linux
这是一个表示为 "go build" 使用的指令，用于生成一个名为 "nat_linux.go" 的文件。这个文件会被包含在构建生成的原生 linux  executable 中。
2. //+build linux
这个指令表示为 "build" 使用的指令，用于告诉 Go 编译器生成名为 "nat_linux.go" 的文件。这个文件会被包含在构建生成的原生 linux executable 中。

通过执行这个 build 脚本，可以生成一个名为 "nat_linux.go" 的文件，这个文件将包含在原生 linux executable 中，用于在 Linux 系统上运行代理程序。


```go
//go:build linux
// +build linux

package nat

import (
	"context"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy"
	"github.com/p4gefau1t/trojan-go/proxy/client"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/tproxy"
)

```

这段代码定义了一个名为"NAT"的代理配置类，并在初始化函数中创建了一个代理实例。

在该代理实例的初始化函数中，首先从配置类中读取相关配置信息，然后检查是否启用了路由器，如果是，就返回错误。否则，创建一个客户端堆栈，并从堆栈中创建一个服务器堆栈。最后，将客户端堆栈和服务器堆栈连接起来，并返回代理实例和 nil，表示成功创建代理实例。


```go
const Name = "NAT"

func init() {
	proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
		cfg := config.FromContext(ctx, Name).(*client.Config)
		if cfg.Router.Enabled {
			return nil, common.NewError("router is not allowed in nat mode")
		}
		ctx, cancel := context.WithCancel(ctx)
		serverStack := []string{tproxy.Name}
		clientStack := client.GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, false)
		c, err := proxy.CreateClientStack(ctx, clientStack)
		if err != nil {
			cancel()
			return nil, err
		}
		s, err := proxy.CreateServerStack(ctx, serverStack)
		if err != nil {
			cancel()
			return nil, err
		}
		return proxy.NewProxy(ctx, cancel, []tunnel.Server{s}, c), nil
	})
}

```

这段代码定义了一个名为 "init" 的函数，该函数在函数开始时执行。函数内部包含一个配置创建器，用于将一个自定义配置创建者注册到配置中。

配置创建器函数的参数是一个字符串 "Name"，用于指定创建配置的名称。函数内部的回调函数接收一个 "func()" 类型参数，该函数返回一个接口 "interface{}"。然后，函数创建一个新的 "client.Config" 对象，将其赋给接口类型的参数，并返回该对象的引用。

这段代码的作用是创建一个配置创建器，该创建器可以用来设置客户端的配置。配置创建器可以用来注册自己的配置创建者，也可以用来注册默认的配置创建者。注册创建器后，可以使用 config.Set来设置客户端的配置。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(client.Config)
	})
}

```

# `proxy/nat/nat_stub.go`



这段代码定义了一个名为 "nat" 的命名空间，其中包含了一些默认成员函数和变量，以及一些语法定义。

具体来说，这个命名空间定义了一个名为 "P" 的函数，它是一个默认的函数，可以用来声明任何类型的变量。这个函数没有参数，返回类型为 "nat"。

还定义了一个名为 "nat_str" 的变量，它是一个 "nat" 命名空间中的默认变量，值为 "默认的 "。

最后，定义了一个名为 "nat_int" 的变量，它也是一个 "nat" 命名空间中的默认变量，值为 42。

还可以使用以下代码创建一个名为 "nat" 的命名空间：


package lat

namespace nat


这样，就可以在程序中使用上面定义的函数和变量了。


```go
package nat

```

# `proxy/server/config.go`

这段代码是一个Go语言的包，名为"server"，旨在提供代理功能。它主要通过以下两个步骤来实现这一目的：

1. 导入所需的第三方库：
	* "github.com/p4gefau1t/trojan-go/config"：用于设置服务器配置；
	* "github.com/p4gefau1t/trojan-go/proxy/client"：用于与远程服务器通信的客户端库。
2. 初始化代理服务器：
	* 在"server"包中，定义了一个名为"init"的函数；
	* 该函数首先调用了"config.RegisterConfigCreator"函数，该函数接收一个名为"Name"的参数，用于指定创建配置创建器函数的名称；
	* 然后定义了一个名为"new"的函数，该函数接收一个客户端代理的实例，将其存储在名为"client"的变量中；
	* 最后，将上述两个函数组合在一起，并传入一个代表"true"的整数，表示启用代理功能。

通过这些函数，可以创建一个服务器代理，允许对远程服务器进行代理。


```go
package server

import (
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy/client"
)

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(client.Config)
	})
}

```

# `proxy/server/server.go`

这段代码定义了一个名为“server”的包。它从多个外部库中导入了一些依赖项，然后定义了一些接口，以及一些代表不同功能的路由器。

具体来说，这段代码：

1. 通过导入“context”和“proxy”等外部库，可以创建一个HTTP代理，即一个可以拦截HTTP请求并转发到后端的服务。

2. 通过导入“client”和“tls”等外部库，可以创建一个HTTP客户端，即一个可以发起HTTP请求并使用TLS加密的客户端。

3. 通过导入“freedom”和“mux”等外部库，可以创建一个支持HTTP和WebSocket的代理，即一个可以同时处理HTTP请求和WebSocket连接的代理。

4. 通过导入“shadowsocks”和“simplesocks”等外部库，可以创建一个支持SSH和Socket的代理，即一个可以同时处理SSH和Socket连接的代理。

5. 通过导入“troll”和“tcp”等外部库，可以创建一个可以发送Troll隧道数据的代理，即一个可以发送Troll隧道数据的代理。

6. 通过导入“transport”等外部库，可以创建一个支持各种 transport协议的代理。

7. 通过导入“trojan”和“tls”等外部库，可以创建一个支持Troll隧道数据和TLS加密的代理，即一个可以同时处理Troll隧道数据和TLS加密的代理。

8. 通过导入“websocket”等外部库，可以创建一个支持WebSocket连接的代理。


```go
package server

import (
	"context"

	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/proxy"
	"github.com/p4gefau1t/trojan-go/proxy/client"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/mux"
	"github.com/p4gefau1t/trojan-go/tunnel/router"
	"github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"
	"github.com/p4gefau1t/trojan-go/tunnel/simplesocks"
	"github.com/p4gefau1t/trojan-go/tunnel/tls"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
	"github.com/p4gefau1t/trojan-go/tunnel/trojan"
	"github.com/p4gefau1t/trojan-go/tunnel/websocket"
)

```

This is a Go language program that creates a proxy server to bypass internet censorship using the `shadowsocks` and `trojan` transport plugins.

It starts by building a proxy node that handles all incoming and outgoing connections. It then checks if the configuration is enabled for shadowsocks and, if it is, creates a subtree for it.

Finally, it sets up a simple HTTP server for handling incoming client requests.

The program uses the `freedom` and `proxy.Node` packages from the `go-args` package to handle the configuration-related work.


```go
const Name = "SERVER"

func init() {
	proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
		cfg := config.FromContext(ctx, Name).(*client.Config)
		ctx, cancel := context.WithCancel(ctx)
		transportServer, err := transport.NewServer(ctx, nil)
		if err != nil {
			cancel()
			return nil, err
		}
		clientStack := []string{freedom.Name}
		if cfg.Router.Enabled {
			clientStack = []string{freedom.Name, router.Name}
		}

		root := &proxy.Node{
			Name:       transport.Name,
			Next:       make(map[string]*proxy.Node),
			IsEndpoint: false,
			Context:    ctx,
			Server:     transportServer,
		}

		if !cfg.TransportPlugin.Enabled {
			root = root.BuildNext(tls.Name)
		}

		trojanSubTree := root
		if cfg.Shadowsocks.Enabled {
			trojanSubTree = trojanSubTree.BuildNext(shadowsocks.Name)
		}
		trojanSubTree.BuildNext(trojan.Name).BuildNext(mux.Name).BuildNext(simplesocks.Name).IsEndpoint = true
		trojanSubTree.BuildNext(trojan.Name).IsEndpoint = true

		wsSubTree := root.BuildNext(websocket.Name)
		if cfg.Shadowsocks.Enabled {
			wsSubTree = wsSubTree.BuildNext(shadowsocks.Name)
		}
		wsSubTree.BuildNext(trojan.Name).BuildNext(mux.Name).BuildNext(simplesocks.Name).IsEndpoint = true
		wsSubTree.BuildNext(trojan.Name).IsEndpoint = true

		serverList := proxy.FindAllEndpoints(root)
		clientList, err := proxy.CreateClientStack(ctx, clientStack)
		if err != nil {
			cancel()
			return nil, err
		}
		return proxy.NewProxy(ctx, cancel, serverList, clientList), nil
	})
}

```

# `redirector/redirector.go`

这段代码定义了一个名为 "redirector" 的包，其中包含了一些名为 "Dial" 的函数，以及一个名为 "defaultDial" 的函数。

"Dial" 函数接受一个网络地址参数 "net.Addr"，并返回一个网络连接和可能的错误。函数内部使用 "net.Dial" 函数，根据传入的网络地址类型设置连接类型为 TCP。

"defaultDial" 函数与 "Dial" 函数不同，它直接返回一个网络连接实例，没有参数。这个函数接受一个字符串参数，表示目标服务器，根据这个字符串创建一个默认的网络连接。

整个 package 的目的是提供一个可以用来创建 Redirector(重定向者)的函数，它可以创建一个或多个网络连接，并在这些连接上执行映射，将请求转发到目标 URL。这个 package 可能会在需要时被用于网络代理、负载均衡或其他网络相关任务。


```go
package redirector

import (
	"context"
	"io"
	"net"
	"reflect"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
)

type Dial func(net.Addr) (net.Conn, error)

func defaultDial(addr net.Addr) (net.Conn, error) {
	return net.Dial("tcp", addr.String())
}

```

这段代码定义了一个名为Redirectation的结构体，该结构体包含以下字段：

* Dial：电话上下文
* RedirectTo：目标地址，格式为net.Addr类型
* InboundConn：入站连接，格式为net.Conn类型

另外，还定义了一个名为Redirector的结构体，该结构体包含一个名为Redirect的函数，以及一个名为ctx的上下文变量。

Redirector的Redirect函数接收一个Redirection结构体和一个上下文变量。该函数的行为取决于传入的redirection实例是否与当前正在处理的目标redirection实例匹配。如果不匹配，则函数将按照默认方式处理，输出一条日志消息。如果匹配，则函数将继续处理redirection实例，不会输出任何消息。

整个代码的目的是创建一个Redirector实例，用于管理URL重定向。该实例可以让您创建和发送redirection请求，并在请求处理成功或失败时执行相应的操作。


```go
type Redirection struct {
	Dial
	RedirectTo  net.Addr
	InboundConn net.Conn
}

type Redirector struct {
	ctx             context.Context
	redirectionChan chan *Redirection
}

func (r *Redirector) Redirect(redirection *Redirection) {
	select {
	case r.redirectionChan <- redirection:
		log.Debug("redirect request")
	case <-r.ctx.Done():
		log.Debug("exiting")
	}
}

```

This is a Go program that handles the configuration of a Redirector, which is a piece of software that manages the way that web applications handle incoming connections from clients.

The program has several options that can be configured through a configuration file, which is typically stored in the `redirection.yaml` file. These options include the IP address of the Redirector, the port number to use for the Redirect, and the IP address of the target system that the connection should be redirected to.

If the Redirector is configured to use a specific IP address for the Redirect, it is added to the `redirection.ip` map, which is defined in the `ip.yaml` file. This map maps each IP address to a list of destinations that the connection should be routed to, including the IP address of the target system.

If the Redirector is configured to use a specific port number for the Redirect, it is added to the `redirection.port` map, which is defined in the `port.yaml` file. This map maps each port number to a list of destinations that the connection should be routed to, including the IP address of the target system.

If the Redirector is not configured to use a specific IP address for the Redirect or port number, it will use the default IP address (0.0.0.0) and port number (8080) for the Redirect. If the Redirector is not configured to use any IP address for the Redirect, it will automatically use the default IP address (0.0.0.0) for the Redirect.

If the Redirector is configured to use a specific URL for the Redirect, it is added to the `urls.yaml` file. This file maps each URL to a list of destinations that the connection should be routed to, including the IP address of the target system.

If the Redirector is configured to use a specific URL for the Redirect, it is added to the `urls.ip` file. This file maps each URL to a list of IP addresses that the connection should be routed to, including the IP address of the target system.

If the Redirector is not configured to use any URL for the Redirect, it will automatically use the default URL for the Redirect, which is `http://localhost:8080/`. If the Redirector is not configured to use any URL for the Redirect, it will automatically use the default URL for the Redirect, which is `http://localhost:8080/`.


```go
func (r *Redirector) worker() {
	for {
		select {
		case redirection := <-r.redirectionChan:
			handle := func(redirection *Redirection) {
				if redirection.InboundConn == nil || reflect.ValueOf(redirection.InboundConn).IsNil() {
					log.Error("nil inbound conn")
					return
				}
				defer redirection.InboundConn.Close()
				if redirection.RedirectTo == nil || reflect.ValueOf(redirection.RedirectTo).IsNil() {
					log.Error("nil redirection addr")
					return
				}
				if redirection.Dial == nil {
					redirection.Dial = defaultDial
				}
				log.Warn("redirecting connection from", redirection.InboundConn.RemoteAddr(), "to", redirection.RedirectTo.String())
				outboundConn, err := redirection.Dial(redirection.RedirectTo)
				if err != nil {
					log.Error(common.NewError("failed to redirect to target address").Base(err))
					return
				}
				defer outboundConn.Close()
				errChan := make(chan error, 2)
				copyConn := func(a, b net.Conn) {
					_, err := io.Copy(a, b)
					errChan <- err
				}
				go copyConn(outboundConn, redirection.InboundConn)
				go copyConn(redirection.InboundConn, outboundConn)
				select {
				case err := <-errChan:
					if err != nil {
						log.Error(common.NewError("failed to redirect").Base(err))
					}
					log.Info("redirection done")
				case <-r.ctx.Done():
					log.Debug("exiting")
					return
				}
			}
			go handle(redirection)
		case <-r.ctx.Done():
			log.Debug("shutting down redirector")
			return
		}
	}
}

```

该代码定义了一个名为NewRedirector的函数，它返回一个名为Redirector的Redirector类型的变量。

函数体中，首先创建了一个名为r的Redirector类型变量，并初始化为一个名为Redirector的匿名变量，该匿名变量创建了一个容量为64的通道，用于存储Redirection类型的数据。

该函数还创建了一个名为r.worker的Go函数，该函数将作为Redirector类型的协程，在函数执行时运行。

最后，该函数返回r，以便在函数调用时使用它。

整个函数的作用是创建一个Redirector类型的实例，该实例可以使用Go函数r.worker来管理Redirection类型的数据。


```go
func NewRedirector(ctx context.Context) *Redirector {
	r := &Redirector{
		ctx:             ctx,
		redirectionChan: make(chan *Redirection, 64),
	}
	go r.worker()
	return r
}

```

# `redirector/redirector_test.go`

这段代码是一个 Go 语言编写的 redirector（重定向工具）package，主要作用是实现 HTTP 重定向。其具体功能如下：

1. 定义了一个名为 "redirector" 的 package，包含了一些与 HTTP 重定向相关的函数和类型。

2. 导入了 "net" 和 "testing" 包，因为我们需要使用 HTTP 网络连接和断言测试代码。

3. 定义了一个名为 "redirector" 的 struct 类型，包含了一些与 HTTP 重定向相关的成员变量，如客户端 HTTP 连接、目标 URL 等。

4. 定义了一个名为 "main" 的函数，它是应用程序的入口点。在函数中，创建一个 HTTP 客户端并设置目标 URL，然后调用 "redirector.Redirect" 函数将重定向请求发送到目标 URL。

5. 在 "redirector.Redirect" 函数中，实现了一个 HTTP 重定向。首先，创建一个 HTTP 客户端并设置目标 URL。然后，设置断言测试中请求的时间超时，如果请求超时，则发送一个 HTTP 状态码 501（与非 200 OK 状态码相同）。接着，调用 "redirector.ServeHTTP" 函数将重定向请求发送到目标 URL。最后，等待目标 URL 返回响应，然后清除断言测试中的 HTTP 客户端。

6. 在 "test/redirector_test.go" 文件中，实现了一个断言测试，用于测试 redirector 的正确性。具体来说，创建一个 HTTP 客户端并设置目标 URL，然后使用 "Redirector" 函数进行重定向，断言测试会检查 HTTP 状态码是否为 200 OK。

7. 在 "redirector/redirector.go" 文件中，定义了一个名为 "Redirector" 的函数，它接受一个 HTTP 客户端和一个目标 URL。函数首先创建一个 HTTP 客户端并设置目标 URL，然后调用 "ServeHTTP" 函数将重定向请求发送到目标 URL。最后，如果目标 URL 返回响应，则清除断言测试中的 HTTP 客户端。

8. 在 "redirector/example.go" 文件中，实现了一个 HTTP 服务器，用于演示 redirector 的正确性。该服务器会响应所有来自客户端的 HTTP GET 请求，并将其重定向到 "redirector.Redirect" 函数返回的目标 URL。


```go
package redirector

import (
	"context"
	"fmt"
	"net"
	"net/http"
	"strings"
	"testing"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/test/util"
)

```

This is a function that sets up a redirect server using the `net/redis` package. A redirect server is a server that routes requests from one service to another. In this case, the server is set up to forward all incoming connections from the localhost on port 0 to a connection on the same port as the client.

The function first sets up a connection to the localhost on port 0 and sets the IP address of the client to `nil`. It then sets up a redirect table using the `redis` package, with the keys being the network addresses and the values being nil.

The function then sets up a redirect session with the address `nil` and the port `nil`. It sets up a connection on the same port as the client and then reads data from the client and prints it to stdout.

If the data received from the client does not contain the expected HTTP/1.1 200 OK status code, the function prints a message and fails. Otherwise, it closes the connection to the client and closes the connection to the server.


```go
func TestRedirector(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	redir := NewRedirector(ctx)
	redir.Redirect(&Redirection{
		Dial:        nil,
		RedirectTo:  nil,
		InboundConn: nil,
	})
	var fakeAddr net.Addr
	var fakeConn net.Conn
	redir.Redirect(&Redirection{
		Dial:        nil,
		RedirectTo:  fakeAddr,
		InboundConn: fakeConn,
	})
	redir.Redirect(&Redirection{
		Dial:        nil,
		RedirectTo:  nil,
		InboundConn: fakeConn,
	})
	redir.Redirect(&Redirection{
		Dial:        nil,
		RedirectTo:  fakeAddr,
		InboundConn: nil,
	})
	l, err := net.Listen("tcp", "127.0.0.1:0")
	common.Must(err)
	conn1, err := net.Dial("tcp", l.Addr().String())
	common.Must(err)
	conn2, err := l.Accept()
	common.Must(err)
	redirAddr, err := net.ResolveTCPAddr("tcp", util.HTTPAddr)
	common.Must(err)
	redir.Redirect(&Redirection{
		Dial:        nil,
		RedirectTo:  redirAddr,
		InboundConn: conn2,
	})
	time.Sleep(time.Second)
	req, err := http.NewRequest("GET", "http://localhost/", nil)
	common.Must(err)
	req.Write(conn1)
	buf := make([]byte, 1024)
	conn1.Read(buf)
	fmt.Println(string(buf))
	if !strings.HasPrefix(string(buf), "HTTP/1.1 200 OK") {
		t.Fail()
	}
	cancel()
	conn1.Close()
	conn2.Close()
}

```

# `statistic/statistics.go`

这段代码定义了一个名为 "statistic" 的包，其中定义了一个名为 "TrafficMeter" 的接口，以及一些相关的函数和类型。

具体来说，这个包旨在实现一个计数器，可以记录通过网络传输的数据包数量以及每个数据包的大小。通过这个计数器，可以对网络流量进行统计和分析，以便了解网络的运行状况和瓶颈。

具体实现中，这个包使用了 Go 标准库中的 "io"、"context" 和 "sync" 包，同时也定义了一些名为 "TrafficMeter" 的函数，这些函数实现了计数器的基本功能，包括添加数据包、获取数据包数量、设置数据包数量上限、获取当前数据包速度和设置数据包速度上限等。

这个包主要用于提供一个简单的计数器，用于统计网络流量，可以帮助开发者更好地了解网络的运行状况和瓶颈，从而优化网络性能。


```go
package statistic

import (
	"context"
	"io"
	"strings"
	"sync"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
)

type TrafficMeter interface {
	io.Closer
	Hash() string
	AddTraffic(sent, recv int)
	GetTraffic() (sent, recv uint64)
	SetTraffic(sent, recv uint64)
	ResetTraffic() (sent, recv uint64)
	GetSpeed() (sent, recv uint64)
	GetSpeedLimit() (sent, recv int)
	SetSpeedLimit(sent, recv int)
}

```

这段代码定义了三个不同类型的接口：IPRecorder、User 和 Authenticator。

IPRecorder 接口表示一个记录 IP 地址的方法，它实现了 AddIP、DelIP 和 GetIP 方法。AddIP 方法接受一个字符串参数，将其添加到 IP 记录器中；DelIP 方法同样接受一个字符串参数，将其从 IP 记录器中删除；GetIP 方法返回 IP 记录器中的当前 IP 地址；SetIPLimit 方法接受一个整数参数，设置 IP 限制，方法本身不提供任何操作。

User 接口表示一个记录用户行为的方法，它实现了 TrafficMeter 和 IPRecorder 接口。可能实现了 AddUser 和 DelUser 方法，具体实现了哪些方法可以在后续的代码中进行补充。

Authenticator 接口表示一个进行身份验证的用户认证器，它实现了 io.Closer 和 AuthUser 方法。其中 AuthUser 方法接收一个哈希密码和用户数据，验证用户是否有效，并返回验证结果；AddUser 方法接收一个哈希密码和用户数据，将其添加到用户注册列表中；DelUser 方法删除用户注册列表中的用户。ListUsers 方法返回注册列表中的所有用户。


```go
type IPRecorder interface {
	AddIP(string) bool
	DelIP(string) bool
	GetIP() int
	SetIPLimit(int)
	GetIPLimit() int
}

type User interface {
	TrafficMeter
	IPRecorder
}

type Authenticator interface {
	io.Closer
	AuthUser(hash string) (valid bool, user User)
	AddUser(hash string) error
	DelUser(hash string) error
	ListUsers() []User
}

```

这段代码定义了一个名为 `Creator` 的接口，以及一个名为 `RegisterAuthenticatorCreator` 的函数和名为 `NewAuthenticator` 的函数。

`RegisterAuthenticatorCreator` 函数用于注册一个名为 `name` 的认证器创建者，可以将创建者传递给 `make` 函数来创建一个新创建者。

`NewAuthenticator` 函数用于创建一个认证器并将结果存储在 `createdAuth`  map 中，也可以用于创建一个未认证的认证器并将结果存储在 `createdAuth`  map 中。

对于每个认证器，它首先检查是否已经创建了一个具有相同名称的认证器。如果是，将该认证器标记为已创建，并返回它。否则，它将尝试从 `authCreators` 地图中获取一个名为 `name` 的创建者，如果找不到，将使用错误消息。

然后，使用创建者提供的 `(ctx context.Context)` 函数，创建一个新认证器并将它添加到 `createdAuth` map 中。

最后，返回创建的认证器。


```go
type Creator func(ctx context.Context) (Authenticator, error)

var (
	createdAuthLock sync.Mutex
	authCreators    = make(map[string]Creator)
	createdAuth     = make(map[context.Context]Authenticator)
)

func RegisterAuthenticatorCreator(name string, creator Creator) {
	authCreators[name] = creator
}

func NewAuthenticator(ctx context.Context, name string) (Authenticator, error) {
	// allocate a unique authenticator for each context
	createdAuthLock.Lock() // avoid concurrent map read/write
	defer createdAuthLock.Unlock()
	if auth, found := createdAuth[ctx]; found {
		log.Debug("authenticator has been created:", name)
		return auth, nil
	}
	creator, found := authCreators[strings.ToUpper(name)]
	if !found {
		return nil, common.NewError("auth driver name " + name + " not found")
	}
	auth, err := creator(ctx)
	if err != nil {
		return nil, err
	}
	createdAuth[ctx] = auth
	return auth, err
}

```

# `statistic/memory/config.go`

这段代码定义了一个名为"memory"的包，其中包含一个名为"Config"的结构体类型，该类型包含两个字段：一个字符串类型的"passwords"数组，和一个字符串类型的"password"字段，分别使用了JSON和YAML格式的配置文件输入。

此外，该代码还定义了一个名为"init"的函数，该函数在函数内部创建了一个名为"memory"的配置创建器，用于初始化该包的配置。该函数使用了名为"github.com/p4gefau1t/trojan-go/config"的依赖，该依赖提供了用于解析配置文件的功能。

最后，该代码没有输出任何配置文件或参数，也没有进行任何错误处理。


```go
package memory

import (
	"github.com/p4gefau1t/trojan-go/config"
)

type Config struct {
	Passwords []string `json:"password" yaml:"password"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{}
	})
}

```

# `statistic/memory/memory.go`

这段代码是一个 Go 语言编写的 package 名为 "memory" 的库，其中包含了一些用于管理内存的数据结构和函数。

具体来说，这段代码以下几种方式来实现对内存的管理：

1. 对一个 synchronization event 进行锁上下文，保证在多线程的情况下对共享资源的互斥访问。
2. 使用一个 atomic 整型变量，对一个 "延时执行" 的操作进行全局锁，避免了多线程之间的竞争条件。
3. 通过 rate 函数，定性地定期执行一些系统任务，例如清理不必要的内存，或者每隔一段时间就扫描整个系统内存的使用情况。
4. 通过 common 包中的 context，将一些系统级的操作与业务逻辑分离，使得这些操作可以更加灵活地被其他业务逻辑调用。
5. 通过 log 包，输出一些信息，例如错误信息或者警告信息，以便开发人员调试和解决问题。
6. 通过 statistic 包，输出一些统计信息，例如系统内存使用情况的数据。

这份代码的主要作用是作为一个内存管理的工具库，提供了对共享资源的管理手段，以提高系统的性能和稳定性。


```go
package memory

import (
	"context"
	"sync"
	"sync/atomic"
	"time"

	"golang.org/x/time/rate"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/statistic"
)

```

该代码定义了一个名为 "MEMORY" 的常量类型变量 "Name"，以及一个 "User" 结构体。该结构体包含了一些用于同步原子操作的 64 位数据成员，如 "sent"、"recv"、"lastSent" 和 "lastRecv"，以及 "sendSpeed" 和 "recvSpeed" 成员。

这个结构体的定义和使用旨在实现一个高性能、低延迟的 HTTP 客户端。它的一些关键特性包括：

1. 使用 sync/atomic 包中的原子操作来保证 64 位数据成员的访问顺序和系统并行度。
2. 定义了一个 cancel 函数，用于在客户端关闭连接时安全地取消请求。
3. 通过 ipTable 同步Map中的数据，该Map是所有并发访问它的读写锁都持有。
4. 通过 rate.Limiter 对发送和接收数据速度进行限制，默认值为 100 速率和 0.1 GPR。
5. 支持最后一个发送数据包发送确认，允许客户端发送 EOF 数据，并不允许客户端发送没有数据的数据包，避免了不必要的网络流量和延迟。


```go
const Name = "MEMORY"

type User struct {
	// WARNING: do not change the order of these fields.
	// 64-bit fields that use `sync/atomic` package functions
	// must be 64-bit aligned on 32-bit systems.
	// Reference: https://github.com/golang/go/issues/599
	// Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
	sent      uint64
	recv      uint64
	lastSent  uint64
	lastRecv  uint64
	sendSpeed uint64
	recvSpeed uint64

	hash        string
	ipTable     sync.Map
	ipNum       int32
	maxIPNum    int
	limiterLock sync.RWMutex
	sendLimiter *rate.Limiter
	recvLimiter *rate.Limiter
	ctx         context.Context
	cancel      context.CancelFunc
}

```

该代码定义了两个函数，分别为：

1. func (u *User) Close() error {
该函数接收一个名为u的*User类型的参数，并返回一个错误值。函数内部先调用u的ResetTraffic()和u的cancel()函数，然后执行一个nil返回的匿名函数，返回nil表示成功关闭网络连接。

2. func (u *User) AddIP(ip string) bool {
该函数接收一个名为u的*User类型的参数，并返回一个布尔值。函数内部首先检查u的最大IP数量是否为0，如果是，则允许添加IP。然后尝试从u的ip表中加载IP地址，如果成功，则返回true。如果尝试加载的IP地址数量超过了u的最大IP数量，则返回false。最后，将IP地址添加到u的ip表中，并将u的ip数量自增1。


```go
func (u *User) Close() error {
	u.ResetTraffic()
	u.cancel()
	return nil
}

func (u *User) AddIP(ip string) bool {
	if u.maxIPNum <= 0 {
		return true
	}
	_, found := u.ipTable.Load(ip)
	if found {
		return true
	}
	if int(u.ipNum)+1 > u.maxIPNum {
		return false
	}
	u.ipTable.Store(ip, true)
	atomic.AddInt32(&u.ipNum, 1)
	return true
}

```

此代码定义了两个函数：`func (u *User) DelIP(ip string) bool` 和 `func (u *User) GetIP() int`。

函数 `func (u *User) DelIP(ip string) bool` 接收一个字符串参数 `ip`，然后判断是否可以删除该 IP 地址。函数首先检查 `u.maxIPNum` 变量，如果它为 0，则表示可以删除任何 IP 地址，因此直接返回 `true`。接下来，函数调用 `u.ipTable.Load(ip)` 函数，尝试从键 `ip` 对应的值中查找是否已存在。如果查找成功，则返回 `false`，否则执行删除操作并更新 `u.ipNum` 变量。最后，返回 `true`，表示成功删除 IP 地址。

函数 `func (u *User) GetIP() int` 接收一个 `*User` 类型的 `u` 变量，然后返回 `u.ipNum` 变量的值。

函数的作用是，`u.ipTable.Load(ip)` 可以用来在 IP 缓存中查找是否存在给定的 IP 地址，`u.ipTable.Delete(ip)` 可以用来删除该 IP 地址，`atomic.AddInt32(&u.ipNum, -1)` 可以用来计数 IP 地址的数量。


```go
func (u *User) DelIP(ip string) bool {
	if u.maxIPNum <= 0 {
		return true
	}
	_, found := u.ipTable.Load(ip)
	if !found {
		return false
	}
	u.ipTable.Delete(ip)
	atomic.AddInt32(&u.ipNum, -1)
	return true
}

func (u *User) GetIP() int {
	return int(u.ipNum)
}

```

这段代码定义了两个函数，分别接收一个 `User` 类型的指针变量 `u`，并实现了 `SetIPLimit` 和 `GetIPLimit` 功能。

具体来说，`SetIPLimit` 函数接收一个 `int` 类型的参数 `n`，将其设置为 `u` 指向的 `User` 类型中的 `maxIPNum` 字段。这个字段是一个限制用户发送的最大 IP 数量，当用户发送超过这个限制时，该字段会被更新。

`GetIPLimit` 函数返回 `User` 类型中的 `maxIPNum` 字段，用于获取限制的最大 IP 数量。

另外，`AddTraffic` 函数接收 `int` 类型的参数 `sent` 和 `recv`，对发送到的数据进行了限制。首先，调用 `u.limiterLock.RLock()`，获得互斥锁，确保在 `AddTraffic` 函数内部对网络请求的并发访问。然后，在获取锁的安全子句中，分别调用 `u.sendLimiter.WaitN(u.ctx, sent)` 和 `u.recvLimiter.WaitN(u.ctx, recv)`，这两个函数会在获取到数据后通知用户。最后，在 `原子` 保证组中增加 `sent` 和 `recv` 两个成员的计数。


```go
func (u *User) SetIPLimit(n int) {
	u.maxIPNum = n
}

func (u *User) GetIPLimit() int {
	return u.maxIPNum
}

func (u *User) AddTraffic(sent, recv int) {
	u.limiterLock.RLock()
	defer u.limiterLock.RUnlock()

	if u.sendLimiter != nil && sent >= 0 {
		u.sendLimiter.WaitN(u.ctx, sent)
	} else if u.recvLimiter != nil && recv >= 0 {
		u.recvLimiter.WaitN(u.ctx, recv)
	}
	atomic.AddUint64(&u.sent, uint64(sent))
	atomic.AddUint64(&u.recv, uint64(recv))
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `u` 的指针和一个整数 `send` 和一个整数 `recv` 作为参数。

函数的作用是设置用户发送和接收数据的速度限制。具体来说，函数首先尝试获取发送速度限制并设置它为 `nil`，如果发送数据为空，则设置为没有限制。然后，函数根据发送数据的大小计算一个新的 `sendLimiter` 变量，使用 `rate.NewLimiter` 函数创建一个自定义的速率限制器，基于发送数据的大小设置一个限制，然后将该限制器设置为 `send` 乘以 `2`。

接下来，函数处理接收数据的速度限制。函数首先尝试获取接收速度限制并设置它为 `nil`，如果接收数据为空，则设置为没有限制。然后，函数根据接收数据的大小计算一个新的 `recvLimiter` 变量，使用 `rate.NewLimiter` 函数创建一个自定义的速率限制器，基于接收数据的大小设置一个限制，然后将该限制器设置为 `recv` 乘以 `2`。

函数的作用是确保发送和接收数据的速度不会超出设置的限制，从而保证系统能够处理数据的速度不会过快或过慢。


```go
func (u *User) SetSpeedLimit(send, recv int) {
	u.limiterLock.Lock()
	defer u.limiterLock.Unlock()

	if send <= 0 {
		u.sendLimiter = nil
	} else {
		u.sendLimiter = rate.NewLimiter(rate.Limit(send), send*2)
	}
	if recv <= 0 {
		u.recvLimiter = nil
	} else {
		u.recvLimiter = rate.NewLimiter(rate.Limit(recv), recv*2)
	}
}

```

这段代码定义了两个函数：

1. `func (u *User) GetSpeedLimit() (send, recv int)`，接收一个指向 `User` 类型对象的 `u` 变量，并返回两个整数 `send` 和 `recv`，用于获取发送和接收速度限制。函数内部首先获取 `u.limiterLock` 对象的锁，然后使用 `RLock()` 函数获取 `u.sendLimiter` 和 `u.recvLimiter` 对象的引用，最后根据限速是否可用判断函数是否需要返回限制值，然后返回限制值。

2. `func (u *User) Hash() string`，接收一个指向 `User` 类型对象的 `u` 变量，并返回一个字符串类型的 `u.hash` 对象的哈希值，这里没有具体的实现，只是定义了一个名为 `Hash` 的函数，接收一个 `User` 对象并返回其哈希值，但未定义哈希函数的具体实现。


```go
func (u *User) GetSpeedLimit() (send, recv int) {
	u.limiterLock.RLock()
	defer u.limiterLock.RUnlock()

	if u.sendLimiter != nil {
		send = int(u.sendLimiter.Limit())
	}
	if u.recvLimiter != nil {
		recv = int(u.recvLimiter.Limit())
	}
	return
}

func (u *User) Hash() string {
	return u.hash
}

```

这段代码定义了三个函数，分别接收一个 `User` 类型的参数 `u`，并实现了数据类型转换、原子操作和函数调用。

1. `func (u *User) SetTraffic(send, recv uint64)`：设置流量。

该函数接收两个 `uint64` 类型的参数 `send` 和 `recv`，然后通过原子存储器 `&u.sent` 和 `&u.recv` 来存储它们。这样，这两个参数将作为全局变量被所有调用此函数的 `User` 实例所共享。

1. `func (u *User) GetTraffic() (uint64, uint64)`：获取流量。

该函数返回两个全局变量 `sent` 和 `recv` 的值，这些值是调用 `SetTraffic` 函数时存储在全局变量中的。

1. `func (u *User) ResetTraffic() (uint64, uint64)`：重置流量。

该函数返回两个全局变量 `sent` 和 `recv` 的值，这些值是调用 `SetTraffic` 函数之前存储在全局变量中的，以及在调用 `GetTraffic` 函数时存储在全局变量中的最后一个 `sent` 和 `recv` 值的相反数。


```go
func (u *User) SetTraffic(send, recv uint64) {
	atomic.StoreUint64(&u.sent, send)
	atomic.StoreUint64(&u.recv, recv)
}

func (u *User) GetTraffic() (uint64, uint64) {
	return atomic.LoadUint64(&u.sent), atomic.LoadUint64(&u.recv)
}

func (u *User) ResetTraffic() (uint64, uint64) {
	sent := atomic.SwapUint64(&u.sent, 0)
	recv := atomic.SwapUint64(&u.recv, 0)
	atomic.StoreUint64(&u.lastSent, 0)
	atomic.StoreUint64(&u.lastRecv, 0)
	return sent, recv
}

```

该函数的作用是提高系统的吞吐量，通过定期发送数据来通知客户端是否有新数据，从而避免在等待数据时产生阻塞。

具体来说，该函数创建了一个定时器，在每次时间间隔内检查是否有新数据。如果是新数据，则函数会将数据发送给客户端，并更新客户端的发送速度和接收速度。同时，函数还会更新客户端最近一次发送数据和接收数据的时间，以便在有新数据时通知客户端。

该函数的实现还可以进一步优化，例如使用协程来发送数据，或者使用非阻塞I/O来处理网络请求。


```go
func (u *User) speedUpdater() {
	ticker := time.NewTicker(time.Second)
	for {
		select {
		case <-u.ctx.Done():
			return
		case <-ticker.C:
			sent, recv := u.GetTraffic()
			atomic.StoreUint64(&u.sendSpeed, sent-u.lastSent)
			atomic.StoreUint64(&u.recvSpeed, recv-u.lastRecv)
			atomic.StoreUint64(&u.lastSent, sent)
			atomic.StoreUint64(&u.lastRecv, recv)
		}
	}
}

```

该代码定义了一个名为Authenticator的结构体，代表一个认证器组件。该组件包含两个方法：

1. `func (a *Authenticator) AuthUser(hash string) (bool, statistic.User)`：用于验证用户身份。接收一个哈希值，首先检查该用户是否存在于`users`映射中。如果用户存在，则返回`true`和用户对象`user`的引用。否则，返回`false`和`nil`。

2. `func (a *Authenticator) GetSpeed() (uint64, uint64)`：用于获取用户速度。返回客户端发送速度和接收速度，两者都返回为`uint64`类型。

从代码中可以看出，该组件的作用是作为一个简单的认证器，验证用户是否存在，并返回客户端发送速度和接收速度。


```go
func (u *User) GetSpeed() (uint64, uint64) {
	return atomic.LoadUint64(&u.sendSpeed), atomic.LoadUint64(&u.recvSpeed)
}

type Authenticator struct {
	users sync.Map
	ctx   context.Context
}

func (a *Authenticator) AuthUser(hash string) (bool, statistic.User) {
	if user, found := a.users.Load(hash); found {
		return true, user.(*User)
	}
	return false, nil
}

```

此函数的作用是向认证器中添加一个用户。它接收一个哈希值（string类型）作为参数，首先检查哈希值是否已存在于环境中。如果是，函数将返回一个错误，否则将创建一个新用户并将其存储在认证器中。

以下是函数的更详细解释：

1. 首先，函数检查传入的哈希值是否已存在于环境中。如果是，函数将直接返回，表示已经存在另一个具有相同哈希值的实例。

2. 如果哈希值不会在环境中存在，函数会创建一个新用户对象，并将其存储在认证器中的哈希值字段中。

3. 接下来，函数使用一个与传入的上下文上下文一起取消的上下文（使用 `context.WithCancel`）来运行一个 goroutine，该 goroutine 会尝试调用 `meter.speedUpdater()` 函数来升级计数器（未在代码中定义，但可以合理地推断为计数器，根据名称和周围的函数）。

4. 最后，函数使用一个Go 中的 `fmt.Println()` 函数来输出一条错误消息，其中的 `%s` 格式化字符串将哈希值作为字符串输入。


```go
func (a *Authenticator) AddUser(hash string) error {
	if _, found := a.users.Load(hash); found {
		return common.NewError("hash " + hash + " is already exist")
	}
	ctx, cancel := context.WithCancel(a.ctx)
	meter := &User{
		hash:   hash,
		ctx:    ctx,
		cancel: cancel,
	}
	go meter.speedUpdater()
	a.users.Store(hash, meter)
	return nil
}

```

这两段代码定义了两个函数，一个是 `DelUser`，另一个是 `ListUsers`。

1. `DelUser` 函数接收一个哈希字符串作为参数，首先从传递进来的一维哈希表 `a.users` 中查找该哈希字符串，如果查找成功，则将其关闭并从哈希表中删除，最后返回一个 `nil` 表示操作成功。如果查找失败，函数将返回一个自定义错误，具体错误信息是 "hash 他指定的哈希字符串 not found"。

2. `ListUsers` 函数返回一个字符串数组，该数组包含了传递进来的一维哈希表 `a.users` 中所有用户的统计信息，包括用户 ID、最后一个登录时间戳（或 None）等。函数使用了 `range` 关键词，遍历哈希表中的每个 key，返回了对应的用户对象，然后将相同ID的用户对象添加到结果字符数组中。


```go
func (a *Authenticator) DelUser(hash string) error {
	meter, found := a.users.Load(hash)
	if !found {
		return common.NewError("hash " + hash + " not found")
	}
	meter.(*User).Close()
	a.users.Delete(hash)
	return nil
}

func (a *Authenticator) ListUsers() []statistic.User {
	result := make([]statistic.User, 0)
	a.users.Range(func(k, v interface{}) bool {
		result = append(result, v.(*User))
		return true
	})
	return result
}

```

这段代码定义了一个名为`func`的函数，它接受一个名为`Authenticator`的参数，该函数返回一个`nil`值，表示代码没有返回任何错误。

接下来，代码定义了一个名为`NewAuthenticator`的函数，它接受一个名为`ctx`的上下文参数，并返回一个`statistic.Authenticator`类型和一个错误类型的`map`值，表示根据提供的配置创建一个新的`Authenticator`实例。

在`NewAuthenticator`函数内部，代码创建了一个`Authenticator`实例，该实例包含了当前上下文`ctx`，以及一个包含密码列表的变量`passwords`。然后，代码遍历`passwords`列表中的每个密码，并使用`common.SHA224String`函数将密码哈希为字符串，以便在添加用户时进行比较。最后，代码将创建的`Authenticator`实例添加到本地内存中，并输出一条日志消息。

由于该函数没有返回任何具体的结果，因此无法确定它是否会成功创建新的`Authenticator`实例。


```go
func (a *Authenticator) Close() error {
	return nil
}

func NewAuthenticator(ctx context.Context) (statistic.Authenticator, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	u := &Authenticator{
		ctx: ctx,
	}
	for _, password := range cfg.Passwords {
		hash := common.SHA224String(password)
		u.AddUser(hash)
	}
	log.Debug("memory authenticator created")
	return u, nil
}

```

这段代码定义了一个名为 "init" 的函数，该函数接受一个参数 "Name" 和一个名为 "NewAuthenticator" 的函数指针。函数的作用是将 "Name" 赋值给一个名为 "statistic" 的标识符，并将 "NewAuthenticator" 的函数指针作为参数传递给 "RegisterAuthenticatorCreator" 函数。

具体来说，"RegisterAuthenticatorCreator" 函数是一个通用的函数，它接受一个名字参数 "Name"，用于将创建的认证器注册到与其名称相同的统计数据中。函数指针 "NewAuthenticator" 是一个函数，它接受一个名为 "Name" 的参数，用于创建一个新的认证器实例。通过将 "NewAuthenticator" 的函数指针传递给 "RegisterAuthenticatorCreator" 函数，可以确保每次调用该函数时都会创建一个新的认证器实例，并在统计数据中注册新的认证器。


```go
func init() {
	statistic.RegisterAuthenticatorCreator(Name, NewAuthenticator)
}

```