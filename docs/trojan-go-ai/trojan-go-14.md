# trojan-go源码解析 14

# `tunnel/transport/transport_test.go`

This is a Go program that creates a server and a client to communicate with it. The server listens on port 127.0.0.1, and the client can connect to the server by connecting to the same IP address on port 127.0.0.1.

The server is using the `tcp` protocol, and is picking a random port from the range of available ports on the IP address (127.0.0.1). The server is using the ` common.PickPort` method to pick a random port for the client to connect to.

The client, on the other hand, is also using the `tcp` protocol, and is connecting to the same IP address (127.0.0.1) and port (serverCfg.LocalPort) as the server. The client is using the ` common.PickPort` method to pick a random port for the connection.

There is a `freedom.Config` initialized inside the server, and it is used to configure the client and server to communicate with each other.


```go
package transport

import (
	"context"
	"net"
	"sync"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
)

func TestTransport(t *testing.T) {
	serverCfg := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  common.PickPort("tcp", "127.0.0.1"),
		RemoteHost: "127.0.0.1",
		RemotePort: common.PickPort("tcp", "127.0.0.1"),
	}
	clientCfg := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  common.PickPort("tcp", "127.0.0.1"),
		RemoteHost: "127.0.0.1",
		RemotePort: serverCfg.LocalPort,
	}
	freedomCfg := &freedom.Config{}
	sctx := config.WithConfig(context.Background(), Name, serverCfg)
	cctx := config.WithConfig(context.Background(), Name, clientCfg)
	cctx = config.WithConfig(cctx, freedom.Name, freedomCfg)

	s, err := NewServer(sctx, nil)
	common.Must(err)
	c, err := NewClient(cctx, nil)
	common.Must(err)

	wg := sync.WaitGroup{}
	wg.Add(1)
	var conn1, conn2 net.Conn
	go func() {
		conn2, err = s.AcceptConn(nil)
		common.Must(err)
		wg.Done()
	}()
	conn1, err = c.DialConn(nil, nil)
	common.Must(err)

	common.Must2(conn1.Write([]byte("12345678\r\n")))
	wg.Wait()
	buf := [10]byte{}
	conn2.Read(buf[:])
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}
	s.Close()
	c.Close()
}

```

该代码段是一个名为 `TestClientPlugin` 的函数，属于一个名为 `func` 的函数。它用于测试客户端插件，以验证是否符合预期的行为。

具体来说，该函数的作用如下：

1. 创建一个客户端配置对象 `clientCfg`，其中包含客户端的本地主机名、本地端口、远程主机名和远程端口。
2. 创建一个配置对象 `ctx`，其中包含客户端配置和上下文信息。
3. 设置客户端配置并创建一个配置对象 `ctx`，其中包含客户端配置和上下文信息。
4. 设置代理配置并创建一个配置对象 `freedomCfg`，其中包含代理的配置和上下文信息。
5. 设置客户端上下文信息并创建一个上下文信息 `ctx`，其中包含客户端上下文信息和配置。
6. 创建一个客户端实例 `c`，其中包含一个关闭的通道 `channel`。
7. 调用 `c.SetOption` 方法，设置一些选项，如 `SS_REMOTE_PORT`，然后关闭通道。
8. 调用 `c.Close` 方法，关闭通道。
9. 调用 `ctx.WithClient` 方法，设置客户端上下文信息，然后关闭上下文信息。
10. 调用 `ctx.WithServer` 方法，设置服务器上下文信息，然后关闭上下文信息。
11. 调用 `ctx.WithShadowsocks` 方法，设置服务器代理的类型为 `shadowsocks`，指定 `echo` 命令，然后关闭上下文信息。
12. 调用 `ctx.WithTcp` 方法，设置传输协议为 `tcp`，然后关闭上下文信息。
13. 调用 `ctx.WithUdp` 方法，设置传输协议为 `udp`，然后关闭上下文信息。
14. 设置 `clientCfg` 和 `ctx`，然后关闭上下文信息。
15. 调用 `config.WithFmt` 方法，格式化配置对象，然后关闭上下文信息。
16. 调用 `config.WithBc` 方法，设置选项 `郡博士`，然后关闭上下文信息。
17. 调用 `funcTestClientPlugin` 函数，测试客户端插件是否正确。


```go
func TestClientPlugin(t *testing.T) {
	clientCfg := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  common.PickPort("tcp", "127.0.0.1"),
		RemoteHost: "127.0.0.1",
		RemotePort: 12345,
		TransportPlugin: TransportPluginConfig{
			Enabled: true,
			Type:    "shadowsocks",
			Command: "echo $SS_REMOTE_PORT",
			Option:  "",
			Arg:     nil,
			Env:     nil,
		},
	}
	ctx := config.WithConfig(context.Background(), Name, clientCfg)
	freedomCfg := &freedom.Config{}
	ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)
	c, err := NewClient(ctx, nil)
	common.Must(err)
	c.Close()
}

```

这段代码是一个名为 `TestServerPlugin` 的函数，属于 `server` 包。它的作用是测试一个名为 `Shadowsocks` 的服务器插件。具体来说，这段代码的作用是创建一个服务器并将其启动，然后使用选定的端口和传输协议将一个从远程计算机发送到本地计算机的请求转发到服务器上，并从服务器接收一个响应。

具体地，这段代码具体执行以下操作：

1. 创建一个名为 `cfg` 的配置对象，其中包含服务器连接的本地主机名、本地端口、远程主机名和远程端口，以及用于连接的服务器配置（包括启用服务器、传输协议、命令、选项等）。

2. 使用 `WithConfig` 方法将配置对象与当前上下文（也就是 `ctx`）结合，这样就可以在当前上下文中应用配置了。

3. 创建一个名为 `freedomCfg` 的自由配置对象，它用于设置服务器在运行时需要的参数和选项。

4. 使用 `WithConfig` 方法将自由配置对象与当前上下文结合，这样就可以在当前上下文中应用配置了。

5. 使用 `NewServer` 方法创建一个服务器实例。

6. 调用 `服务器实例的关闭` 方法关闭服务器。

7. 调用 `发射一个请求并获取一个响应` 方法，将一个请求从本地计算机发送到远程计算机，并获取服务器返回的响应。

8. 调用 `关闭服务器` 方法关闭服务器。


```go
func TestServerPlugin(t *testing.T) {
	cfg := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  common.PickPort("tcp", "127.0.0.1"),
		RemoteHost: "127.0.0.1",
		RemotePort: 12345,
		TransportPlugin: TransportPluginConfig{
			Enabled: true,
			Type:    "shadowsocks",
			Command: "echo $SS_REMOTE_PORT",
			Option:  "",
			Arg:     nil,
			Env:     nil,
		},
	}
	ctx := config.WithConfig(context.Background(), Name, cfg)
	freedomCfg := &freedom.Config{}
	ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)
	s, err := NewServer(ctx, nil)
	common.Must(err)
	s.Close()
}

```

# `tunnel/transport/tunnel.go`

这段代码定义了一个名为 "transport" 的包，它导入了一个名为 "context" 的外设，以及一个名为 "trojan-go" 的包，然后定义了一个名为 "Tunnel" 的类型，该类型包含一个名为 "*Tunnel" 的指针类型的变量。

具体来说，该类型定义了一个名为 "Name" 的字段，其值为 "TRANSPORT"，然后定义了一个名为 "*Tunnel" 的指针类型的变量，它是一个空类型，没有实际的数据成员。

最后，该类型定义了一个名为 "Tunnel" 的函数，它没有参数，返回类型为 "Tunnel"，但可以在编译时产生一个函数声明，其名为 "transport.Tunnel"。


```go
package transport

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "TRANSPORT"

type Tunnel struct{}

func (*Tunnel) Name() string {
	return Name
}

```

这是一段使用Go编程语言编写的函数指针类型，定义了两个名为`NewClient`和`NewServer`的函数，以及一个名为`init`的函数。

`NewClient`函数接受一个名为`ctx`的上下文句柄和一个名为`client`的匿名客户端参数，并返回一个名为`tunnel.Client`的客户端对象和一个名为`error`的错误对象。函数实现了一个简单的工厂模式，创建一个新的客户端实例并返回。

`NewServer`函数与`NewClient`类似，只是将参数类型改为服务器对象，并返回一个名为`tunnel.Server`的服务器对象和一个名为`error`的错误对象。函数同样实现了一个简单的工厂模式，创建一个新的服务器实例并返回。

`init`函数用于初始化命名空间。在函数内部，创建了一个名为`Tunnel`的名为`tunnel`的匿名导入，并将其注册为命名空间。


```go
func (*Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, client)
}

func (*Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, server)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/trojan/client.go`

这段代码是一个名为 "trojan" 的 Go 包，包含了一些用于创建和管理网络隧道的功能。下面是对代码中的一些重要部分的解释：

1. "import (": 这是一个导入语句，用于导入其他包。这里列出了 "bytes"、"context"、"net"、"sync" 和 "time" 包。

2. "importing bytes": 这一行用于导入 "bytes" 包，该包用于操作字节切片。

3. "importing context": 这一行用于导入 "context" 包，该包用于获取上下文信息。

4. "importing net": 这一行用于导入 "net" 包，该包用于网络通信。

5. "importing sync": 这一行用于导入 "sync" 包，该包用于并发操作的封装。

6. "importing time": 这一行用于导入 "time" 包，该包用于操作时间。

7. "importing (": 这是一个导入语句，用于导入其他包。这里列出了 "github.com/p4gefau1t/trojan-go/api"、"github.com/p4gefau1t/trojan-go/common"、"github.com/p4gefau1t/trojan-go/config" 和 "github.com/p4gefau1t/trojan-go/log"。

8. "importing api": 这一行用于导入 "github.com/p4gefau1t/trojan-go/api" 包，该包用于与远程服务器通信。

9. "importing common": 这一行用于导入 "github.com/p4gefau1t/trojan-go/common" 包，该包包含一些通用的功能。

10. "importing config": 这一行用于导入 "github.com/p4gefau1t/trojan-go/config" 包，该包用于设置和读取配置文件。

11. "importing log": 这一行用于导入 "github.com/p4gefau1t/trojan-go/log" 包，该包用于记录日志。

12. "importing time": 这一行用于导入 "time" 包，该包用于操作时间。

13. "importing memory": 这一行用于导入 "github.com/p4gefau1t/trojan-go/memory" 包，该包用于管理内存。

14. "importing statistic": 这一行用于导入 "github.com/p4gefau1t/trojan-go/statistic" 包，该包用于收集统计信息。

15. "importing statistic/memory": 这一行用于导入 "github.com/p4gefau1t/trojan-go/statistic/memory" 包，该包用于收集内存中的统计信息。

16. "importing tunnel": 这一行用于导入 "github.com/p4gefau1t/trojan-go/tunnel" 包，该包用于创建和管理网络隧道。

17. "importing tunnel/mux": 这一行用于导入 "github.com/p4gefau1t/trojan-go/tunnel/mux" 包，该包用于在隧道中发送消息。


```go
package trojan

import (
	"bytes"
	"context"
	"net"
	"sync"
	"sync/atomic"
	"time"

	"github.com/p4gefau1t/trojan-go/api"
	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/statistic"
	"github.com/p4gefau1t/trojan-go/statistic/memory"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/mux"
)

```

这段代码定义了一个名为`OutboundConn`的结构体，表示一个出站的网络连接。这个连接使用Go语言中的Go-IPR基于IP协议栈的库实现。

以下是这个结构体中的一些关键字段：


const MaxPacketSize = 1024 * 8


这个常量是一个8位的无符号整数，定义了最大数据包大小。


const Connect = 1
const Associate = 3
const Mux = 0x7f


这三个常量定义了这个连接的一些操作。


type OutboundConn struct {
	// WARNING: do not change the order of these fields.
	// 64-bit fields that use `sync/atomic` package functions
	// must be 64-bit aligned on 32-bit systems.
	// Reference: https://github.com/golang/go/issues/599
	// Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
	sent uint64
	recv uint64


这个结构体中包含两个32位的无符号整数字段，`sent`表示发送的数据包数量，`recv`表示收到的数据包数量。


	metadata          *tunnel.Metadata
	user              statistic.User
	headerWrittenOnce sync.Once
	net.Conn


这个结构体中包含一些配套的字段，如`metadata`类型字段表示元数据，`user`类型字段表示用户信息，`headerWrittenOnce`类型字段表示是否已经向发送的数据包写入过`header`字段，`net.Conn`类型字段表示网络连接的底层实现。

总的来说，这个结构体表示一个出站的网络连接，包含了连接的相关信息以及一些配套的字段。


```go
const (
	MaxPacketSize = 1024 * 8
)

const (
	Connect   tunnel.Command = 1
	Associate tunnel.Command = 3
	Mux       tunnel.Command = 0x7f
)

type OutboundConn struct {
	// WARNING: do not change the order of these fields.
	// 64-bit fields that use `sync/atomic` package functions
	// must be 64-bit aligned on 32-bit systems.
	// Reference: https://github.com/golang/go/issues/599
	// Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
	sent uint64
	recv uint64

	metadata          *tunnel.Metadata
	user              statistic.User
	headerWrittenOnce sync.Once
	net.Conn
}

```

该函数的作用是返回一个指向名为"Metadata"的结构的指针，该结构体代表了一个网络连接的元数据。

具体来说，该函数接收一个指向名为"OutboundConn"的结构体的指针，该结构体代表了一个网络连接的出方向的操作。

函数首先通过"WriteHeader"函数将数据帧的头部发送到出方向，然后执行一个名为"Metadata Write"的职能。

在该职能中，首先创建一个名为"buf"的字符缓冲区，用于存储接收到的数据。然后，构造一个名为"crlf"的字符串，并将其添加到缓冲区中。

接着，使用"Metadata WriteTo"函数将元数据写入缓冲区中。如果传入了任何数据，则使用"c.Conn.Write"函数将数据发送到连接的下一跳。

最后，使用"buf.Write"函数将数据写入接收端的缓冲区中，并设置"written"变量为真，以及错误变量为空。

函数返回两个值，第一个值是布尔值表示是否成功写入元数据，第二个值是错误。


```go
func (c *OutboundConn) Metadata() *tunnel.Metadata {
	return c.metadata
}

func (c *OutboundConn) WriteHeader(payload []byte) (bool, error) {
	var err error
	written := false
	c.headerWrittenOnce.Do(func() {
		hash := c.user.Hash()
		buf := bytes.NewBuffer(make([]byte, 0, MaxPacketSize))
		crlf := []byte{0x0d, 0x0a}
		buf.Write([]byte(hash))
		buf.Write(crlf)
		c.metadata.WriteTo(buf)
		buf.Write(crlf)
		if payload != nil {
			buf.Write(payload)
		}
		_, err = c.Conn.Write(buf.Bytes())
		if err == nil {
			written = true
		}
	})
	return written, err
}

```

这两函数的作用如下：

1. func (c *OutboundConn) Write(p []byte) (int, error) {
这个函数接收一个OutboundConn类型的指针c，以及一个字节数组p。函数的作用是在这个OutboundConn上写入p中的字节数组。它返回写入的的字节数和错误。

函数首先使用c.WriteHeader(p)将p中的字节数组设置为写入的头部信息，包括帧头。然后，它检查错误并输出written。如果written为true，则说明成功写入了p中的字节数组。函数中的错误处理是：trojan failed to flush header with payload（已发送数据无法刷新帧头）。

2. func (c *OutboundConn) Read(p []byte) (int, error) {
这个函数接收一个OutboundConn类型的指针c，以及一个字节数组p。函数的作用是在这个OutboundConn上读取p中的字节数组。它返回读取的字节数和错误。

函数首先使用c.Read(p)从OutboundConn的套接字中读取p中的字节数组。然后，它增加c.user的流量，并将n存储在内部状态中。最后，函数将n转换为原子整数并存储到c.sent中。函数中的错误处理是：trojan failed to read from net（无法从网络中读取数据）。


```go
func (c *OutboundConn) Write(p []byte) (int, error) {
	written, err := c.WriteHeader(p)
	if err != nil {
		return 0, common.NewError("trojan failed to flush header with payload").Base(err)
	}
	if written {
		return len(p), nil
	}
	n, err := c.Conn.Write(p)
	c.user.AddTraffic(n, 0)
	atomic.AddUint64(&c.sent, uint64(n))
	return n, err
}

func (c *OutboundConn) Read(p []byte) (int, error) {
	n, err := c.Conn.Read(p)
	c.user.AddTraffic(0, n)
	atomic.AddUint64(&c.recv, uint64(n))
	return n, err
}

```

此代码定义了一个名为`func`的函数，接收一个名为`OutboundConn`的协程作为参数。函数的作用是关闭与该协程连接的套接字，并返回一个错误。具体来说，函数执行以下操作：

1. 记录连接信息和收发数据：函数创建一个名为`c`的变量，该变量引用一个名为`OutboundConn`的协程。函数还创建了两个名为`sent`和`recv`的整数变量，分别表示发送和接收的数据包数量。
go
log.Info("connection to", c.metadata, "closed", "sent:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.sent)), "recv:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.recv)))

2. 关闭套接字：函数使用`c.Conn.Close`方法关闭与协程的连接。
go
return c.underlay.Close()

3. 取消定时器：如果已经创建了定时器，函数将取消它。
go
c.cancel()

4. 返回错误：函数返回一个错误，该错误可能是由于关闭套接字时出现的问题导致的。

该函数的作用是关闭与协程的连接，并确保所有数据包已经被接收或发送完毕。


```go
func (c *OutboundConn) Close() error {
	log.Info("connection to", c.metadata, "closed", "sent:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.sent)), "recv:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.recv)))
	return c.Conn.Close()
}

type Client struct {
	underlay tunnel.Client
	user     statistic.User
	ctx      context.Context
	cancel   context.CancelFunc
}

func (c *Client) Close() error {
	c.cancel()
	return c.underlay.Close()
}

```

该函数名为 `DialConn`，它接受一个指向客户端 `Client` 的指针 `c`，以及一个指向隧道 `Tunnel` 的指针 `overlay`。它的作用是使客户端与远程服务器建立连接，并在连接建立后执行一个名为 `Mux` 的操作，将隧道类型更改为 `Mux`。

函数内部首先尝试使用客户端 `c` 的 `underlay` 部分调用 `DialConn` 函数，如果执行成功，则返回连接信息和错误信息。如果 `DialConn` 函数返回错误信息，函数将返回一个空 `OutboundConn` 类型的变量，并将其作为函数的返回值。

如果 `overlay` 是介于 `Tunnel` 和 `Mux` 之间的类型，函数将执行在函数内部的一个 `go` 循环，该循环将尝试调用 `newConn.WriteHeader` 函数，将任何缓冲的 Trojan 头数据发送到远程服务器。如果 `overlay` 是 `Tunnel` 类型，函数将忽略该循环。

最后，函数返回新创建的 `OutboundConn` 类型的变量，它连接到远程服务器，执行了名为 `Mux` 的操作。


```go
func (c *Client) DialConn(addr *tunnel.Address, overlay tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.underlay.DialConn(addr, &Tunnel{})
	if err != nil {
		return nil, err
	}
	newConn := &OutboundConn{
		Conn: conn,
		user: c.user,
		metadata: &tunnel.Metadata{
			Command: Connect,
			Address: addr,
		},
	}
	if _, ok := overlay.(*mux.Tunnel); ok {
		newConn.metadata.Command = Mux
	}

	go func(newConn *OutboundConn) {
		// if the trojan header is still buffered after 100 ms, the client may expect data from the server
		// so we flush the trojan header
		time.Sleep(time.Millisecond * 100)
		newConn.WriteHeader(nil)
	}(newConn)
	return newConn, nil
}

```

该函数名为 `DialPacket`，它接收一个名为 `c` 的 UDP 客户端参数，并返回一个名为 `PacketConn` 的 UDP 连接和一个名为 `error` 的错误。

函数的作用是：连接到指定的隧道，并将数据包发送到该隧道中。如果连接成功，则返回一个 `PacketConn` 实例，其中包含有关连接的信息。如果连接失败，则返回一个 `error` 消息。

具体来说，函数的实现包括以下步骤：

1. 创建一个名为 `fakeAddr` 的 UDP 地址结构体，该地址用于连接到隧道。该地址包含一个域名，用于标识数据包发送或接收的服务器，以及一个地址类型，该类型用于将数据包发送或接收的服务器与客户端的 IP 地址区分开来。
2. 使用 `c.underlay.DialConn` 函数拨打该地址，并获取一个 `DialResult` 对象。如果该函数返回一个错误，则调用 `DialResult.Listen` 函数重新尝试拨打该地址。
3. 如果 `DialResult.Listen` 函数返回一个有效的 `Listen` 结果，则使用 `c.underlay.DialStream` 函数将数据包发送到服务器。
4. 创建一个名为 `PacketConn` 的 UDP 连接结构体，该结构体包含有关连接的信息，如客户端的 `user` 字段用于指定客户端的本地 UDP 地址，以及用于标识数据包发送或接收的地址。
5. 如果调用 `DialResult.Listen` 函数时没有错误，则返回一个 `PacketConn` 实例，其中包含有关连接的信息。
6. 如果调用 `DialResult.Listen` 函数时出现了错误，则返回一个 `error` 消息。


```go
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	fakeAddr := &tunnel.Address{
		DomainName:  "UDP_CONN",
		AddressType: tunnel.DomainName,
	}
	conn, err := c.underlay.DialConn(fakeAddr, &Tunnel{})
	if err != nil {
		return nil, err
	}
	return &PacketConn{
		Conn: &OutboundConn{
			Conn: conn,
			user: c.user,
			metadata: &tunnel.Metadata{
				Command: Associate,
				Address: fakeAddr,
			},
		},
	}, nil
}

```

该函数名为 `NewClient`，其作用是在上下文上下文中创建一个新的客户端实例，并返回其值或错误。

具体来说，它使用了 `context.WithCancel` 函数来确保在函数中的任何副作用操作都得到了正确的关闭。同时，它还创建了一个自定义的身份验证器 `statistic.NewAuthenticator`，该身份验证器通过 `memory.Name` 进行身份验证。

接下来，它读取配置文件中的客户端配置，如果配置中指定了 `api.Enabled` 字段为 `true`，那么它将创建一个名为 `Name_CLIENT` 的服务，并使用身份验证器通过 `ctx` 和 `client` 参数传递给 `api.RunService` 函数。

此外，它还创建了一个 `statistic.User` 类型的变量 `user`，用于存储身份验证器中发现的用户之一。如果 `user` 为 `nil`，则说明没有找到任何有效的用户，函数将返回并取消任何操作。

最后，它输出一条日志消息，表示已创建一个新的客户端。


```go
func NewClient(ctx context.Context, client tunnel.Client) (*Client, error) {
	ctx, cancel := context.WithCancel(ctx)
	auth, err := statistic.NewAuthenticator(ctx, memory.Name)
	if err != nil {
		cancel()
		return nil, err
	}

	cfg := config.FromContext(ctx, Name).(*Config)
	if cfg.API.Enabled {
		go api.RunService(ctx, Name+"_CLIENT", auth)
	}

	var user statistic.User
	for _, u := range auth.ListUsers() {
		user = u
		break
	}
	if user == nil {
		cancel()
		return nil, common.NewError("no valid user found")
	}

	log.Debug("trojan client created")
	return &Client{
		underlay: client,
		ctx:      ctx,
		user:     user,
		cancel:   cancel,
	}, nil
}

```

# `tunnel/trojan/config.go`

这段代码是一个 Go 语言编写的 Trojan 框架包，其中定义了一个名为 Config 的结构体，表示配置信息。该 Config 结构体包含了远程服务器和客户端的配置信息，如 IP 地址、端口号、用户名、密码等，以及是否启用 HTTP 验证等选项。

具体来说，该代码实现了一个简单的配置存储和读取功能，将配置信息存储在 Config 结构体中，并支持 JSON 和 YAML 两种形式的配置文件读取。通过 Config 结构体，可以方便地读取和设置应用程序的配置信息，从而使应用程序的行为更加灵活和可扩展。


```go
package trojan

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
	LocalHost        string      `json:"local_addr" yaml:"local-addr"`
	LocalPort        int         `json:"local_port" yaml:"local-port"`
	RemoteHost       string      `json:"remote_addr" yaml:"remote-addr"`
	RemotePort       int         `json:"remote_port" yaml:"remote-port"`
	DisableHTTPCheck bool        `json:"disable_http_check" yaml:"disable-http-check"`
	MySQL            MySQLConfig `json:"mysql" yaml:"mysql"`
	API              APIConfig   `json:"api" yaml:"api"`
}

type MySQLConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

```

这段代码定义了一个名为APIConfig的结构体类型，其中包含一个名为Enabled的布尔值。这个结构体类型包含两个字段，一个是json字段enabled，另一个是yaml字段enabled，它们都使用了大括号{}包裹。

在代码的初始化部分，使用了一个名为init的函数。这个函数接受一个参数，代表一个名为Name的参数。函数内部创建了一个名为APIConfig的配置创建器函数，这个函数返回一个指向Config类型的变量。然后将这个函数的返回值赋值给APIConfig类型的字段enabled，最后将Enabled字段的值设置为true。

整个函数的作用是创建一个名为APIConfig的配置创建器，并将之注册到名为config的配置上下文中。如果配置上下文中未指定具体的配置名称，则默认的配置名就是"default"。


```go
type APIConfig struct {
	Enabled bool `json:"enabled" yaml:"enabled"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{}
	})
}

```

# `tunnel/trojan/packet.go`

该代码定义了一个名为`PacketConn`的结构体，表示一个在Trojan协议中使用的数据连接。

该结构体包含以下字段：

* `tunnel.Conn`字段，表示与隧道连接的底层`tunnel.Conn`实例。
* ` common.Capability`字段，表示该连接是否具有发送或接收数据的能力。
* ` encoding.UTF8`字段，表示数据编码使用的UTF-8编码。
* ` net.IP`字段，表示连接的网络接口。

该结构体用于在Trojan协议中创建一个数据连接，提供了一个统一的方式来封装网络层和数据层之间的通信，使得在Trojan中，不同协议之间的数据传输变得简单和统一。


```go
package trojan

import (
	"bytes"
	"encoding/binary"
	"io"
	"io/ioutil"
	"net"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type PacketConn struct {
	tunnel.Conn
}

```

这两函数是定义在PacketConn结构的体中，它们用于数据传输过程中的数据接收和发送。

具体来说，ReadFrom函数接收一个字节数组（payload）并返回两个值：读取的字节数和错误。它使用c.ReadWithMetadata函数读取数据，并传递给c.ReadWithMetadata的第二个参数，该参数包含一个字节数组和一系列元数据（Metadata）。函数在调用时可能会失败，比如如果读取的数据长度比期望的数据长度要长，则会产生错误。

WriteTo函数同样接收一个字节数组（payload）和一个网络地址（addr）。它使用c.WriteWithMetadata函数将数据发送到目标地址，并传递给c.WriteWithMetadata的第二个参数，该参数包含一个字节数组和一系列元数据（Metadata）。函数在调用时可能会失败，比如如果目标地址无效或者数据发送过程中出错，则会产生错误。

总的来说，这两函数是用于封装PacketConn的数据读写功能，让开发者可以方便地进行数据传输过程中的数据接收和发送操作。


```go
func (c *PacketConn) ReadFrom(payload []byte) (int, net.Addr, error) {
	return c.ReadWithMetadata(payload)
}

func (c *PacketConn) WriteTo(payload []byte, addr net.Addr) (int, error) {
	address, err := tunnel.NewAddressFromAddr("udp", addr.String())
	if err != nil {
		return 0, err
	}
	m := &tunnel.Metadata{
		Address: address,
	}
	return c.WriteWithMetadata(payload, m)
}

```

该函数的作用是封装一个名为PacketConn的网络连接对象，以向远程主机发送数据包并添加元数据。它将构造一个最大长度为MaxPacketSize的内存缓冲区，将元数据中的地址信息写入其中，然后将整个数据包发送到目标连接上。

以下是函数的实现步骤：

1. 创建一个最大长度为MaxPacketSize的内存缓冲区packet，并将其初始化为空。
2. 创建一个名为w的bytes.Buffer缓冲区，用于存储数据包中的数据。
3. 创建一个包含0x0d和0x0a字节的数据包头部，并将其写入w。
4. 创建一个包含长度B字符串和crlf字节的数据包尾部分并将其写入w。
5. 将数据包 payload 中的字节添加到w中。
6. 通过conn.Write函数将数据包发送到远程主机。
7. 在函数中添加日志输出，以便在发生错误时进行调试。
8. 返回数据包的长度和错误。


```go
func (c *PacketConn) WriteWithMetadata(payload []byte, metadata *tunnel.Metadata) (int, error) {
	packet := make([]byte, 0, MaxPacketSize)
	w := bytes.NewBuffer(packet)
	metadata.Address.WriteTo(w)

	length := len(payload)
	lengthBuf := [2]byte{}
	crlf := [2]byte{0x0d, 0x0a}

	binary.BigEndian.PutUint16(lengthBuf[:], uint16(length))
	w.Write(lengthBuf[:])
	w.Write(crlf[:])
	w.Write(payload)

	_, err := c.Conn.Write(w.Bytes())

	log.Debug("udp packet remote", c.RemoteAddr(), "metadata", metadata, "size", length)
	return len(payload), err
}

```

这段代码的作用是实现了一个名为"ReadWithMetadata"的函数，接收一个名为"PacketConn"的UDP协议套接字，并接收一个字节数组"payload"。函数返回两个值：

1. 一个整数类型的"length"（实际收到的数据长度），它是通过调用"ReadFrom"函数获取的。如果该函数出现错误，则返回0并打印错误信息。
2. 一个指向"Metadata"类型的指针类型的"tunnel.Metadata"（实际收到的数据中包含的元数据），它是通过调用"ReadFrom"函数获取的。如果该函数执行成功，则返回一个包含"address"字段的元数据对象，其中包含发送方地址的"Address"字段和数据长度。如果数据长度超过了"MaxPacketSize"（设置的最大数据长度），函数将使用"Drain"函数将多余的数据从接收端的套接字中读取并返回。

函数中首先检查传入的字节数组"payload"是否足够大，如果是，则说明函数收到的数据长度已经超出了允许的最大长度，这时函数将使用"Drain"函数将多余的数据读取并返回。


```go
func (c *PacketConn) ReadWithMetadata(payload []byte) (int, *tunnel.Metadata, error) {
	addr := &tunnel.Address{
		NetworkType: "udp",
	}
	if err := addr.ReadFrom(c.Conn); err != nil {
		return 0, nil, common.NewError("failed to parse udp packet addr").Base(err)
	}
	lengthBuf := [2]byte{}
	if _, err := io.ReadFull(c.Conn, lengthBuf[:]); err != nil {
		return 0, nil, common.NewError("failed to read length")
	}
	length := int(binary.BigEndian.Uint16(lengthBuf[:]))

	crlf := [2]byte{}
	if _, err := io.ReadFull(c.Conn, crlf[:]); err != nil {
		return 0, nil, common.NewError("failed to read crlf")
	}

	if len(payload) < length || length > MaxPacketSize {
		io.CopyN(ioutil.Discard, c.Conn, int64(length)) // drain the rest of the packet
		return 0, nil, common.NewError("incoming packet size is too large")
	}

	if _, err := io.ReadFull(c.Conn, payload[:length]); err != nil {
		return 0, nil, common.NewError("failed to read payload")
	}

	log.Debug("udp packet from", c.RemoteAddr(), "metadata", addr.String(), "size", length)
	return length, &tunnel.Metadata{
		Address: addr,
	}, nil
}

```

# `tunnel/trojan/server.go`

这段代码是一个 Go 语言编写的 Trojan 木马程序。Trojan 是一个用于远程执行代码的工具，可以用来进行各种恶意操作，如文件上传、注册表修改等。

该程序的作用是下载并执行一个名为 "trojan.exe" 的木马程序，该程序将在本地计算机上创建一个 "trojan.exe" 文件，并下载到 "C:\trojan.exe" 的路径。下载过程会使用 HTTP 请求下载该文件，并使用字符串转义来将文件名转换为 "trojan.exe"。

该程序还包含一个用于显示下载进度的 "./display-进度.png" 文件，以及一个用于在下载完成后启动 "trojan.exe" 的新窗口的 "./启动-trojan.png" 文件。

下载的 "trojan.exe" 木马程序可能是一个恶意软件，具有破坏计算机系统、窃取敏感信息等危险行为。使用此类程序请谨慎，并确保从可信的来源下载并执行。


```go
package trojan

import (
	"context"
	"fmt"
	"io"
	"net"
	"sync/atomic"

	"github.com/p4gefau1t/trojan-go/api"
	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/redirector"
	"github.com/p4gefau1t/trojan-go/statistic"
	"github.com/p4gefau1t/trojan-go/statistic/memory"
	"github.com/p4gefau1t/trojan-go/statistic/mysql"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/mux"
)

```

该代码定义了一个名为InboundConn的结构体，表示一个入站连接。该结构体实现了以下功能：

1. 定义了64位字段sent和recv，表示发送和接收的数据量。
2. 实现了一个atomic类型的InboundConn，以确保这些字段的值能够正确地同步到系统外。
3. 使用了sync/atomic包中的函数，确保这些函数需要64位整数大小。
4. 包含了一个net.Conn类型的字段，表示入站连接的网络套接字。
5. 包含了一个auth类型的字段，表示用于身份验证的认证器。
6. 包含了一个user类型的字段，表示用于身份验证的用户名。
7. 包含了一个hash类型的字段，表示用于加密的哈希函数。
8. 包含了一个metadata类型的字段，表示用于存储隧道元数据的代理。
9. 包含了一个ip类型的字段，表示入站连接的IP地址。

该代码定义了一个用于入站连接的结构体，该结构体用于管理入站连接的信息，包括网络套接字、认证器、用户名、密码哈希、隧道元数据和IP地址等。该结构体还实现了atomic类型，以确保这些字段的值能够正确地同步到系统外。


```go
// InboundConn is a trojan inbound connection
type InboundConn struct {
	// WARNING: do not change the order of these fields.
	// 64-bit fields that use `sync/atomic` package functions
	// must be 64-bit aligned on 32-bit systems.
	// Reference: https://github.com/golang/go/issues/599
	// Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
	sent uint64
	recv uint64

	net.Conn
	auth     statistic.Authenticator
	user     statistic.User
	hash     string
	metadata *tunnel.Metadata
	ip       string
}

```

这段代码是一个名为`InboundConn`的函数指针，它接受一个入站连接`InboundConn`类型的参数。函数指针返回一个指向`Metadata`类型的指针，该类型可以用来检索隧道中正在传输的数据。

具体来说，函数指针`func (c *InboundConn) Metadata() *tunnel.Metadata`返回的是入站连接`InboundConn`所关联的隧道中的`Metadata`结构体。该函数指针返回的`Metadata`类型是一个用于存储隧道中数据的结构体，它包含了以下字段：

- `c.metadata`：该字段存储了入站连接的`Metadata`结构体中存储的数据。
- `c.sent`：该字段记录了入站连接已经发送的数据量。
- `c.recv`：该字段记录了入站连接已经接收的数据量。
- `c.conn`：该字段存储了入站连接的底层网络连接对象。

函数指针`func (c *InboundConn) Write(p []byte) (int, error)`接受一个字节数组`p`作为参数，并返回入站连接`InboundConn`向底层网络写入数据的成功次数`n`和错误信息`err`。

函数指针`func (c *InboundConn) Read(p []byte) (int, error)`接受一个字节数组`p`作为参数，并返回入站连接`InboundConn`从底层网络读取数据的成功次数`n`和错误信息`err`。

函数指针`func (c *InboundConn) Metadata() *tunnel.Metadata`是入站连接`InboundConn`的`Metadata`函数的返回值，而`Write`和`Read`函数则是入站连接的底层网络连接对象的函数，它们用于向底层网络写入或读取数据。


```go
func (c *InboundConn) Metadata() *tunnel.Metadata {
	return c.metadata
}

func (c *InboundConn) Write(p []byte) (int, error) {
	n, err := c.Conn.Write(p)
	atomic.AddUint64(&c.sent, uint64(n))
	c.user.AddTraffic(n, 0)
	return n, err
}

func (c *InboundConn) Read(p []byte) (int, error) {
	n, err := c.Conn.Read(p)
	atomic.AddUint64(&c.recv, uint64(n))
	c.user.AddTraffic(0, n)
	return n, err
}

```

This is a Go function that defines an `InboundConn` struct that connects to a remote server using a `Tunnel` instance.

The `InboundConn` struct has several fields:

* `c.hash`: a hash of the remote server's public key
* `c.metadata`: the metadata associated with the remote server
* `c.user`: a user object representing the remote server's identity
* `c.ip`: the IP address of the remote server
* `c.sent`: the number of packets sent from the `InboundConn` to the remote server
* `c.recv`: the number of packets received from the remote server by the `InboundConn`

The `Auth()` method is called by the `InboundConn` struct to authenticate with the remote server before establishing the tunnel. It reads the remote server's public key, checks if it's valid, and updates the `c.hash` field with the hash of the remote server's public key. It also updates the `c.user` field with the identity of the remote server and adds the IP address of the remote server to the `c.metadata` field.

The `AuthUser()` method is defined by the `InboundConn` struct and is used to authenticate a user with the remote server. It takes a username and password and updates the `c.hash` field with the hash of the remote server's public key, if the password is correct. It also updates the `c.user` field with the identity of the user and adds the IP address of the user to the `c.metadata` field.

The `Close()` method is defined by the `InboundConn` struct to close the connection to the remote server when it's finished.

The `AWSON鼓起業績主義开始的SET}/\*'use magnautitential <a href='/igLomp='结'>立馬 LEADS <a href='/igLomp='結'>强壮 LEADS <a href='/igLomp='結'>bandwidth LEADS <a href='/igLomp='結'>contentious LEADS <a href='/igLomp='結'>counterculture LEADS <a href='/igLomp='結'>emotional <a href='/igLomp='結'>leaderboard <a href='/igLomp='結'>quick <a href='/igLomp='結'>sign up'><button>Sign Up</button></a>"/>：服飾 LEADS <a href='/igLomp='結'>MORE LEADS <a href='/igLomp='結'>Connect with us</a></div>'}}表示測試台灣之光 PS4 Pro LEADS <a href='/igLomp='結'>拔刀 LEADS <a href='/igLomp='結'>射飛 LEADS <a href='/igLomp='結'> demon <a href='/igLomp='結'>弱點 LEADS <a href='/igLomp='結'>people <a href='/igLomp='結'>such as me</a>.</a>"/>';text-align:center;visibility:hidden}}'</div>武道LEADS <a href='/igLomp='結'>中和 LEADS <a href='/igLomp='結'>頁面 LEADS <a href='/igLomp='結'>by <a href='/igLomp='結'>PEOPLE </a>for <a href='/igLomp='結'>北境 LEADS <a href='/igLomp='結'>SECURITY</a>('.',',' []LEADS <a href='/igLomp='結'>它們 LEADS <a href='/igLomp='結'>'</a>');text-align:center;visibility:hidden}}'</div>''}}​?relaxedCSS= Raises <a href='/igLomp='結'>挑战</a> <a href='/igLomp='結'>也让 <a href='/igLomp='結'>我們</a> <a href='/igLomp='結'>一起</a> <a href='/igLomp='結'>成長</a> <a href='/igLomp='結'>並且</a> <a href='/igLomp='結'>為 <a href='/igLomp='結'>更好的明天</a> <a href='/igLomp='結'>而</a> <a href='/igLomp='結'>在這個 <a href='/igLomp='結'>世界</a> <a href='/igLomp='結'>中</a> <a href='/igLomp='結'>一起</a> <a href='/igLomp='結'>創造</a> <a href='/igLomp='結'>更美好的未來</a> <a href='/igLomp='結'>。',ThelenF抽，萃取E,text-align:center,horizontal-align:center,dfn:F,logger:aHR0cDovL3d3L8WlUz8eL7吡握肛一條項目的縮寫搞搞豫 psychologically百分之百content17,text-align:center,horizontal-align:center,dfn:F,logger:aHR0cDovL3d3L8WlUz8eL7download洛LEADS <a href='/igLomp='結'>下载</a> <a href='/igLomp='結'>这个</a> <a href='/igLomp='結'>网站</a> <a href='/igLomp='結'>上</a> <a href='/igLomp='結'>安<a href='/igLomp='結'>排<a href='/igLomp='結'>排</a> <a href='/igLomp='結'>容<a href='/igLomp='結'>大<a href='/igLomp='結'>並<a href='/igLomp='結'>將</a> <a href='/igLomp='結'>這<a href='/igLomp='結'>個</a> <a href='/igLomp='結'>计<a href='/igLomp='結'>算</a> <a href='/igLomp='結'>得</a> <a href='/igLomp='結'>以</a> <a href='/igLomp='結'>上</a> <a href='/igLomp='結'>三<a href='/igLomp='結'>大</a> <a href='/igLomp='結'>流<a href='/igLomp='結'>表<a href='/igL


```go
func (c *InboundConn) Close() error {
	log.Info("user", c.hash, "from", c.Conn.RemoteAddr(), "tunneling to", c.metadata.Address, "closed",
		"sent:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.sent)), "recv:", common.HumanFriendlyTraffic(atomic.LoadUint64(&c.recv)))
	c.user.DelIP(c.ip)
	return c.Conn.Close()
}

func (c *InboundConn) Auth() error {
	userHash := [56]byte{}
	n, err := c.Conn.Read(userHash[:])
	if err != nil || n != 56 {
		return common.NewError("failed to read hash").Base(err)
	}

	valid, user := c.auth.AuthUser(string(userHash[:]))
	if !valid {
		return common.NewError("invalid hash:" + string(userHash[:]))
	}
	c.hash = string(userHash[:])
	c.user = user

	ip, _, err := net.SplitHostPort(c.Conn.RemoteAddr().String())
	if err != nil {
		return common.NewError("failed to parse host:" + c.Conn.RemoteAddr().String()).Base(err)
	}

	c.ip = ip
	ok := user.AddIP(ip)
	if !ok {
		return common.NewError("ip limit reached")
	}

	crlf := [2]byte{}
	_, err = io.ReadFull(c.Conn, crlf[:])
	if err != nil {
		return err
	}

	c.metadata = &tunnel.Metadata{}
	if err := c.metadata.ReadFrom(c.Conn); err != nil {
		return err
	}

	_, err = io.ReadFull(c.Conn, crlf[:])
	if err != nil {
		return err
	}
	return nil
}

```

该代码定义了一个名为Server的 struct 类型，表示一个令牌隧道服务器。该服务器包含以下成员变量：

- `auth`：一个名为auth的统计器，用于记录客户端的认证信息。
- `redir`：一个名为redirector的redirector，用于处理客户端的redirect请求。
- `redirAddr`：一个名为tunnel.Address的tunnel.Address类型的变量，用于存储服务器监听的端口。
- `underlay`：一个名为tunnel.Server的tunnel.Server类型的变量，表示一个下层服务器，用于处理连接到令牌隧道中的客户端。
- `connChan`：一个名为tunnel.Conn的tunnel.Conn类型的变量，用于存储连接客户端的通道。
- `muxChan`：一个名为tunnel.PacketConn的tunnel.PacketConn类型的变量，用于存储连接客户端的通道，用于传输数据 packets。
- `packetChan`：一个名为tunnel.PacketConn的tunnel.PacketConn类型的变量，用于存储连接客户端的通道，用于传输数据 packets。
- `ctx`：一个名为context.Context的context.Context类型的变量，用于存储当前上下文。
- `cancel`：一个名为context.CancelFunc的context.CancelFunc类型的变量，用于取消连接。

该服务器还包含以下方法：

- `Close`：关闭服务器并尝试关闭任何未关闭的连接。

此外，该服务器还实现了两个闭包函数：

- `auth.Close`：用于关闭auth统计器。
- `tunnel.Server.Close`：用于关闭下层服务器。


```go
// Server is a trojan tunnel server
type Server struct {
	auth       statistic.Authenticator
	redir      *redirector.Redirector
	redirAddr  *tunnel.Address
	underlay   tunnel.Server
	connChan   chan tunnel.Conn
	muxChan    chan tunnel.Conn
	packetChan chan tunnel.PacketConn
	ctx        context.Context
	cancel     context.CancelFunc
}

func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

```

This appears to be a Go program that handles incoming connections to a web application. It establishes a connection with a Mux server, either by default or by specifying the Mux domain name in the `Connect` metadata field.

The program also supports the `Associate` metadata field, which allows it to connect to a remote trojan for the purpose of authentication. When the `Associate` metadata field is used, the program creates a `PacketConn` struct to communicate with the remote trojan.

The program uses a connection pool to handle the reused connections. When a connection is closed, it rewinds the connection and returns to the connection pool. If an incoming connection cannot be established, the program redirects the connection to the default redirect address or returns an error.

The program also handles the case where the incoming connection is a direct connection to the Mux server. It will establish a `Mux` connection and log any Mux-specific events in its logs.

Overall, the program appears to handle incoming connections to a Mux server in a functional manner.


```go
func (s *Server) acceptLoop() {
	for {
		conn, err := s.underlay.AcceptConn(&Tunnel{})
		if err != nil { // Closing
			log.Error(common.NewError("trojan failed to accept conn").Base(err))
			select {
			case <-s.ctx.Done():
				return
			default:
			}
			continue
		}
		go func(conn tunnel.Conn) {
			rewindConn := common.NewRewindConn(conn)
			rewindConn.SetBufferSize(128)
			defer rewindConn.StopBuffering()

			inboundConn := &InboundConn{
				Conn: rewindConn,
				auth: s.auth,
			}

			if err := inboundConn.Auth(); err != nil {
				rewindConn.Rewind()
				rewindConn.StopBuffering()
				log.Warn(common.NewError("connection with invalid trojan header from " + rewindConn.RemoteAddr().String()).Base(err))
				s.redir.Redirect(&redirector.Redirection{
					RedirectTo:  s.redirAddr,
					InboundConn: rewindConn,
				})
				return
			}

			rewindConn.StopBuffering()
			switch inboundConn.metadata.Command {
			case Connect:
				if inboundConn.metadata.DomainName == "MUX_CONN" {
					s.muxChan <- inboundConn
					log.Debug("mux(r) connection")
				} else {
					s.connChan <- inboundConn
					log.Debug("normal trojan connection")
				}

			case Associate:
				s.packetChan <- &PacketConn{
					Conn: inboundConn,
				}
				log.Debug("trojan udp connection")
			case Mux:
				s.muxChan <- inboundConn
				log.Debug("mux connection")
			default:
				log.Error(common.NewError(fmt.Sprintf("unknown trojan command %d", inboundConn.metadata.Command)))
			}
		}(conn)
	}
}

```

该函数`AcceptConn`接收一个`Server`类型的参数`s`和一个`nextTunnel`类型的参数`nextTunnel`。`nextTunnel`是一个`Tunnel`类型的参数，`Tunnel`是一个`mux.Tunnel`类型，代表一个支持`mux`协议的隧道。

函数的作用是接受一个连接请求，并根据所提供的连接类型选择如何处理。

具体来说，如果`nextTunnel`是一个`mux.Tunnel`，则函数将接收一个连接并尝试通过该连接建立一个会话。如果通过连接建立了会话，函数将返回该会话，否则返回一个错误。

如果`nextTunnel`是一个`Tunnel`类型的参数，则函数将尝试通过服务器通道`s.connChan`或`s.muxChan`中的连接建立一个会话。如果通过连接建立了会话，函数将返回该会话，否则返回一个错误。注意，如果服务器通道关闭或没有连接，函数将返回一个错误，而不是空或`nil`。

如果`nextTunnel`是一个`mux.Tunnel`和一个`Trojan`实例，则函数将尝试通过服务器通道`s.connChan`或`s.muxChan`中的连接建立一个会话。如果通过连接建立了会话，函数将返回该会话，否则返回一个错误。注意，如果服务器通道关闭或没有连接，函数将返回一个错误，而不是空或`nil`。


```go
func (s *Server) AcceptConn(nextTunnel tunnel.Tunnel) (tunnel.Conn, error) {
	switch nextTunnel.(type) {
	case *mux.Tunnel:
		select {
		case t := <-s.muxChan:
			return t, nil
		case <-s.ctx.Done():
			return nil, common.NewError("trojan client closed")
		}
	default:
		select {
		case t := <-s.connChan:
			return t, nil
		case <-s.ctx.Done():
			return nil, common.NewError("trojan client closed")
		}
	}
}

```

This is a function that creates a TCP server using the specified configuration. It does this by connecting to a MySQL database (if specified) and creating a new authenticator that can authenticate users. It then starts listening for incoming connections and using the specified HTTP check to check if the connection is allowed.

The function also creates a `Server` struct that contains several fields such as `underlay`, `auth`, `redirAddr`, `connChan`, `muxChan`, `packetChan`, `ctx`, `cancel`, `redir`, and `acceptLoop`. The `underlay` field is an instance of the `Underlay` class that will handle the connection backlog, while `auth` is an instance of the `Authenticator` class that will handle the authentication process. `redirAddr` is a string that specifies the host port of the redirection server, and `connChan`, `muxChan`, `packetChan`, `ctx`, `cancel`, `redir`, and `acceptLoop` are all fields that correspond to the `Server` struct.

If the configuration is not specified when the function is called, it will exit with an error. Additionally, if the MySQL database is not specified, it will also exit with an error.


```go
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	select {
	case t := <-s.packetChan:
		return t, nil
	case <-s.ctx.Done():
		return nil, common.NewError("trojan client closed")
	}
}

func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	ctx, cancel := context.WithCancel(ctx)

	// TODO replace this dirty code
	var auth statistic.Authenticator
	var err error
	if cfg.MySQL.Enabled {
		log.Debug("mysql enabled")
		auth, err = statistic.NewAuthenticator(ctx, mysql.Name)
	} else {
		log.Debug("auth by config file")
		auth, err = statistic.NewAuthenticator(ctx, memory.Name)
	}
	if err != nil {
		cancel()
		return nil, common.NewError("trojan failed to create authenticator")
	}

	if cfg.API.Enabled {
		go api.RunService(ctx, Name+"_SERVER", auth)
	}

	redirAddr := tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)
	s := &Server{
		underlay:   underlay,
		auth:       auth,
		redirAddr:  redirAddr,
		connChan:   make(chan tunnel.Conn, 32),
		muxChan:    make(chan tunnel.Conn, 32),
		packetChan: make(chan tunnel.PacketConn, 32),
		ctx:        ctx,
		cancel:     cancel,
		redir:      redirector.NewRedirector(ctx),
	}

	if !cfg.DisableHTTPCheck {
		redirConn, err := net.Dial("tcp", redirAddr.String())
		if err != nil {
			cancel()
			return nil, common.NewError("invalid redirect address. check your http server: " + redirAddr.String()).Base(err)
		}
		redirConn.Close()
	}

	go s.acceptLoop()
	log.Debug("trojan server created")
	return s, nil
}

```

# `tunnel/trojan/trojan_test.go`

这段代码是一个Go语言编写的Trojan框架包。它定义了一些接口、函数和变量，用于支持编写和运行网络代理程序。具体来说，它实现了以下功能：

1. 定义了`trojan.http.Client`接口，用于创建HTTP客户端并发送请求。
2. 定义了`trojan.http.Server`接口，用于创建HTTP服务器并监听来自客户端的请求。
3. 实现了一个`trojan.core.Chain`类，它实现了HTTP链的接口。
4. 实现了一个`trojan.core.Credentials`类，它实现了用于身份验证的接口。
5. 实现了一个`trojan.core.事件`类型，它用于表示代理事件。
6. 实现了一个`trojan.core.HTTP2`类型，它实现了使用HTTP/2协议传输数据。
7. 实现了一个`trojan.core.Transport`类型，它实现了代理传输协议的接口。
8. 实现了一个`trojan.test.Main`类，用于设置 up测试套件、处理选项、打印测试结果等。
9. 实现了一个`trojan.test.Trojan`类，用于实现Trojan的基本功能。
10. 实现了一个`trojan.test.internal`类型，它用于包含与测试相关的函数和变量。


```go
package trojan

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"net"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/statistic/memory"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This appears to be a Go program that performs a redirect connection with a remote server. Here's a high-level overview of what it does:

1.Accepts a connection from a remote server.
2.Reads data from the server.
3.Calls the AcceptPacket method to accept a packet from the server.
4.Calls the AcceptConn method to accept a connection from the server.
5.Reads data from the server and stores it in a buffer.
6.Sends a packet from the client to the server using the generated payload and metadata.
7.Reads data from the server and stores it in a buffer.
8.Checks if the packet sent from the client and received from the server is equal.
9.Calls the Close method to close the connection between the client and server.
10.Calls the Close method to close the connection between the client and server.


```go
func TestTrojan(t *testing.T) {
	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  port,
		RemoteHost: "127.0.0.1",
		RemotePort: port,
	}
	ctx, cancel := context.WithCancel(context.Background())
	ctx = config.WithConfig(ctx, transport.Name, transportConfig)
	ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
	tcpClient, err := transport.NewClient(ctx, nil)
	common.Must(err)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	serverPort := common.PickPort("tcp", "127.0.0.1")
	authConfig := &memory.Config{Passwords: []string{"password"}}
	clientConfig := &Config{
		RemoteHost: "127.0.0.1",
		RemotePort: serverPort,
	}
	serverConfig := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  serverPort,
		RemoteHost: "127.0.0.1",
		RemotePort: util.EchoPort,
	}

	ctx = config.WithConfig(ctx, memory.Name, authConfig)
	clientCtx := config.WithConfig(ctx, Name, clientConfig)
	serverCtx := config.WithConfig(ctx, Name, serverConfig)
	c, err := NewClient(clientCtx, tcpClient)
	common.Must(err)
	s, err := NewServer(serverCtx, tcpServer)
	common.Must(err)
	conn1, err := c.DialConn(&tunnel.Address{
		DomainName:  "example.com",
		AddressType: tunnel.DomainName,
	}, nil)
	common.Must(err)
	common.Must2(conn1.Write([]byte("87654321")))
	conn2, err := s.AcceptConn(nil)
	common.Must(err)
	buf := [8]byte{}
	conn2.Read(buf[:])
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}

	packet1, err := c.DialPacket(nil)
	common.Must(err)
	packet1.WriteWithMetadata([]byte("12345678"), &tunnel.Metadata{
		Address: &tunnel.Address{
			DomainName:  "example.com",
			AddressType: tunnel.DomainName,
			Port:        80,
		},
	})
	packet2, err := s.AcceptPacket(nil)
	common.Must(err)

	_, m, err := packet2.ReadWithMetadata(buf[:])
	common.Must(err)

	fmt.Println(m)

	if !util.CheckPacketOverConn(packet1, packet2) {
		t.Fail()
	}

	// redirecting
	conn, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))
	common.Must(err)
	sendBuf := util.GeneratePayload(1024)
	recvBuf := [1024]byte{}
	common.Must2(conn.Write(sendBuf))
	common.Must2(io.ReadFull(conn, recvBuf[:]))
	if !bytes.Equal(sendBuf, recvBuf[:]) {
		fmt.Println(sendBuf)
		fmt.Println(recvBuf[:])
		t.Fail()
	}
	conn1.Close()
	conn2.Close()
	packet1.Close()
	packet2.Close()
	conn.Close()
	c.Close()
	s.Close()
	cancel()
}

```

# `tunnel/trojan/tunnel.go`

这段代码定义了一个名为"trojan"的包，其中包含一个名为"Tunnel"的 struct 类型。这个 struct 类型有一个名为"Name"的成员变量，该成员变量返回一个字符串常量"TROJAN"，即"trojan"的名称。

此外，代码还导入了一个名为"context"的包，但没有将其用于任何地方。最后，代码导入了另一个名为"github.com/p4gefau1t/trojan-go/tunnel"的包，但同样没有对其进行任何使用。


```go
package trojan

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "TROJAN"

type Tunnel struct{}

func (c *Tunnel) Name() string {
	return Name
}

```

这两函数是用于创建Tunnel的`Tunnel`类型和对应的`Tunnel.Client`和`Tunnel.Server`类型的函数。

具体来说，这两函数接受一个`c`参数（代表一个`Tunnel`类型的实例）和另一个`ctx`参数（代表一个`context.Context`），并返回一个`tunnel.Client`类型的实例和一个`tunnel.Server`类型的实例，或者错误。

这两函数实现的作用如下：

1. `NewClient`函数接收一个`c`参数和一个`client`参数，返回一个`tunnel.Client`类型的实例。
2. `NewServer`函数接收一个`c`参数和一个`server`参数，返回一个`tunnel.Server`类型的实例。


```go
func (c *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, client)
}

func (c *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, server)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/websocket/client.go`

这段代码定义了一个名为 "websocket" 的包，它包含了一个名为 "Client" 的结构体，该结构体使用了一个名为 "tunnel" 的隧道客户端与 WebSocket 服务器进行通信。

下面是对该 "Client" 结构体的定义的详细解释：

go
type Client struct {
	underlay tunnel.Client
	hostname string
	path     string
}


该结构体定义了一个名为 "Client" 的变量，它使用了一个名为 "underlay" 的隧道客户端对象，该客户端对象使用了一个名为 "tunnel" 的隧道客户端，可以与任何 WebSocket 服务器进行通信。

该结构体定义了一个名为 "hostname" 的变量，它用于设置与 WebSocket 服务器通信时使用的默认主机名称。

该结构体定义了一个名为 "path" 的变量，它用于设置 WebSocket 服务器的路径，如果该路径不存在，则会使用默认的路径。

在函数内部，该结构体使用 "underlay.Connect" 方法连接到 WebSocket 服务器。然后，它使用 "underlay.Text" 方法发送消息到服务器。

go
func Connect(hostname string, path string) error {
	return underlay.Connect(hostname, path)
}


该函数 "Connect" 接收两个参数，第一个参数是主机名称，第二个参数是服务器的路径。它使用 "underlay.Connect" 方法连接到服务器，该函数返回一个 "error" 类型的变量，表示连接是否成功。如果连接成功，则返回一个 Nil 值。

websocket
func text(message string) error {
	return underlay.Text(message)
}


该函数 "text" 接收一个参数 "message"，它是一个字符串，代表消息。它使用 "underlay.Text" 方法将消息发送到服务器，并返回一个 "error" 类型的变量，表示是否成功发送消息。

websocket
func listen(hostname string, path string) (<-t.Msg, error) {
	return underlay.Listen(hostname, path)
}


该函数 "listen" 接收两个参数，第一个参数是主机名称，第二个参数是服务器的路径。它使用 "underlay.Listen" 方法监听服务器，接收一个 "<-t.Msg, error" 类型的变量和一个 "error" 类型的变量，表示是否成功监听服务器。如果成功监听服务器，则返回一个 "<-t.Msg, error" 类型的变量，表示一个可以发送消息的 "Msg" 类型的变量和一个 "error" 类型的变量，表示是否成功监听服务器。

websocket
func connect(hostname string, path string) error {
	return underlay.Connect(hostname, path)
}


该函数 "connect" 接收两个参数，第一个参数是主机名称，第二个参数是服务器的路径。它使用 "underlay.Connect" 方法连接到服务器，该函数返回一个 "error" 类型的变量，表示连接是否成功。如果连接成功，则返回一个 Nil 值。


```go
package websocket

import (
	"context"
	"strings"

	"golang.org/x/net/websocket"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Client struct {
	underlay tunnel.Client
	hostname string
	path     string
}

```

该函数名为`func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error)`，它接收一个`Client`类型的参数`c`，然后执行以下操作：

1. 使用`c.underlay.DialConn`函数尝试与 underlying 网络客户建立连接。如果失败，函数将返回一个空指针并抛出`common.NewError`类型的错误。
2. 将连接的 URL 和 origin 设置为 "wss://" 和 "https://" 以及客户端主机名和路径，创建一个`websocket.Config`对象和一个`websocket.Client`对象。
3. 如果配置或客户端连接出现错误，函数将抛出`common.NewError`类型的错误。
4. 创建一个名为`OutboundConn`的输出级连接对象，包含`wsConn`和`tcpConn`字段，其中`wsConn`是通过调用`websocket.NewClient`函数创建的客户端 WebSocket 连接，`tcpConn`是通过调用`c.underlay.DialConn`函数创建的底层 TCP 连接。
5. 返回`OutboundConn`作为结果，不输出任何函数体。


```go
func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.underlay.DialConn(nil, &Tunnel{})
	if err != nil {
		return nil, common.NewError("websocket cannot dial with underlying client").Base(err)
	}
	url := "wss://" + c.hostname + c.path
	origin := "https://" + c.hostname
	wsConfig, err := websocket.NewConfig(url, origin)
	if err != nil {
		return nil, common.NewError("invalid websocket config").Base(err)
	}
	wsConn, err := websocket.NewClient(wsConfig, conn)
	if err != nil {
		return nil, common.NewError("websocket failed to handshake with server").Base(err)
	}
	return &OutboundConn{
		Conn:    wsConn,
		tcpConn: conn,
	}, nil
}

```

该代码定义了两个函数，一个是名为 "DialPacket"，另一个是名为 "Close"。

"DialPacket" 函数接收一个名为 "c" 的指针，代表一个客户端，以及一个名为 "tunnel" 的管道。该函数尝试调用 "websocket" 路由，并返回一个 "tunnel.PacketConn" 类型的数据，或者是 "error" 类型的数据。

"Close" 函数接收一个名为 "c" 的指针，代表一个客户端，并尝试关闭该客户端与底层网络的连接。


```go
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	return nil, common.NewError("not supported by websocket")
}

func (c *Client) Close() error {
	return c.underlay.Close()
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	if !strings.HasPrefix(cfg.Websocket.Path, "/") {
		return nil, common.NewError("websocket path must start with \"/\"")
	}
	if cfg.Websocket.Host == "" {
		cfg.Websocket.Host = cfg.RemoteHost
		log.Warn("empty websocket hostname")
	}
	log.Debug("websocket client created")
	return &Client{
		hostname: cfg.Websocket.Host,
		path:     cfg.Websocket.Path,
		underlay: underlay,
	}, nil
}

```