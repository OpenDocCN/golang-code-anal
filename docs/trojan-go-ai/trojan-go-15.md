# trojan-go源码解析 15

# `tunnel/websocket/config.go`

这段代码定义了一个名为 "websocket" 的包，并包含了一个名为 "Config" 的结构体。该结构体包含三个字段，分别表示是否启用 WebSocket 连接、连接的主机名和端口号，以及 WebSocket 的配置。

具体来说，该代码将以下内容存储在 "Config" 结构体中，并将其导出为 "websocket.Config"：

json
{
 "enabled": true,
 "host": "https://example.com",
 "path": "/ws",
 "remote_addr": "127.0.0.1",
 "remote_port": 0,
 "websocket": {
   "name": "MyWebSocket",
   "version": "0.8.0",
   "type": "websocket"
 }
}


然后，该代码导出了一个名为 "websocket.Configuration" 的接口，该接口定义了如何设置 WebSocket 连接的参数。

最后，该代码导出了一个名为 "websocket" 的包，该包包含了一个名为 "Config" 的结构体，该结构体用于存储 WebSocket 的配置。


```go
package websocket

import "github.com/p4gefau1t/trojan-go/config"

type WebsocketConfig struct {
	Enabled bool   `json:"enabled" yaml:"enabled"`
	Host    string `json:"host" yaml:"host"`
	Path    string `json:"path" yaml:"path"`
}

type Config struct {
	RemoteHost string          `json:"remote_addr" yaml:"remote-addr"`
	RemotePort int             `json:"remote_port" yaml:"remote-port"`
	Websocket  WebsocketConfig `json:"websocket" yaml:"websocket"`
}

```

这段代码是使用Go语言中的函数式编程风格定义了一个名为`init`的函数。

具体来说，该函数接受一个参数`Config`，然后注册了一个名为`Name`的配置创建器函数。这个配置创建器函数内部创建了一个`Config`对象，并返回了这个`Config`对象。

这里的关键点是，`Name`只是一个参数，它被传递给`func()`后面的匿名函数。这个匿名函数返回了一个`interface{}`类型的变量，这个类型可以理解为任何类型的引用。由于`Name`是一个字符串，所以在这里被用作参数的`interface{}`变量实际上接收了一个字符串类型的参数。

这段代码的作用是创建了一个名为`init`的函数，该函数接受一个名为`Config`的参数，并返回一个配置创建器函数。这个配置创建器函数创建了一个`Config`对象并返回了这个对象，使得我们可以通过定义好的函数指针来调用该函数，并传递一个`Config`对象作为参数。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(Config)
	})
}

```

# `tunnel/websocket/conn.go`

这段代码定义了一个名为`OutboundConn`的结构体，它包含一个`*websocket.Conn`类型的网络连接和一个`net.Conn`类型的TCP连接。

首先，它导入了`websocket`包，这是Golang的一个用于与WebSocket服务器进行交互的库。它还导入了一个名为`net`的包，这是Golang的一个用于与网络进行交互的库。

接着，它定义了一个`OutboundConn`结构体，该结构体包含一个`*websocket.Conn`类型的网络连接和一个`net.Conn`类型的TCP连接。这个结构体有一个`网络连接`字段和一个`TCP连接`字段，它们都使用了`net.Conn`类型。

`*websocket.Conn`类型是一个实现了`net.Conn`接口的`*websocket.Conn`类型，它提供了一个与WebSocket服务器交互的连接。通过创建一个`*websocket.Conn`类型的实例并将其赋值给`OutboundConn`的结构体，我们可以获取该实例的网络连接。

`net.Conn`类型是一个实现了`net.Conn`接口的`net.Conn`类型，它提供了一个与网络进行交互的连接。通过创建一个`net.Conn`类型的实例并将其赋值给`OutboundConn`的结构体，我们可以获取该实例的TCP连接。

最后，`OutboundConn`结构体还包含一个名为`tcpConn`字段，它的类型与`OutboundConn`结构体中的`net.Conn`字段的类型相同。通过创建一个`OutboundConn`实例，我们可以创建一个与WebSocket服务器交互的TCP连接，并将其添加到该实例的`TCP连接`字段中。


```go
package websocket

import (
	"context"
	"net"

	"golang.org/x/net/websocket"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

type OutboundConn struct {
	*websocket.Conn
	tcpConn net.Conn
}

```

该代码定义了两个名为`Outbound`和`Inbound`的类型的`OutboundConn`结构体。

`func (c *OutboundConn) Metadata() *tunnel.Metadata {
	return nil
}`函数返回一个`Metadata`类型的指针，它代表连接的元数据。此函数没有实现，因为它返回了一个`nil`值，表示没有元数据可返回。

`func (c *OutboundConn) RemoteAddr() net.Addr {
	// 覆盖了websocket.Conn的RemoteAddr函数，或者它将返回一些来自"Origin"的URL。
	return c.tcpConn.RemoteAddr()
}`函数返回一个`net.Addr`类型的指针，它代表连接的远程地址。此函数覆盖了`OutboundConn`结构体中的`RemoteAddr`函数，因此它返回了与原始连接相同的远程地址。

`type InboundConn struct {
	OutboundConn
	ctx    context.Context
	cancel context.CancelFunc
}`定义了一个`InboundConn`结构体，它代表一个入站连接。该结构体包含一个`OutboundConn`实例和一个`ctx`上下文变量以及一个`cancel`函数。

`Outbound`和`Inbound`结构体都实现了`OutboundConn`接口，因此它们都实现了`c.SetMetadata`和`c.SetRemoteAddr`函数。


```go
func (c *OutboundConn) Metadata() *tunnel.Metadata {
	return nil
}

func (c *OutboundConn) RemoteAddr() net.Addr {
	// override RemoteAddr of websocket.Conn, or it will return some url from "Origin"
	return c.tcpConn.RemoteAddr()
}

type InboundConn struct {
	OutboundConn
	ctx    context.Context
	cancel context.CancelFunc
}

```

此代码是一个函数，接收一个名为InboundConn的协程作为参数。函数的作用是关闭传入的协程，并返回一个错误。

具体来说，首先调用c.cancel()函数来取消正在等待的连接，然后执行c.Conn.Close()函数来关闭连接。由于这两个函数都使用并发编程的方式，因此不会输出任何错误信息。


```go
func (c *InboundConn) Close() error {
	c.cancel()
	return c.Conn.Close()
}

```

# `tunnel/websocket/server.go`

这段代码是一个 WebSocket 库，实现了 WebSocket 的创建、连接、消息处理等功能。

具体来说，它包括以下组件：

1. 导入了必要的包：`websocket`、`bufio`、`context`、`math/rand`、`net`、`net/http`、`strings`、`time`、`golang.org/x/net/websocket`。

2. 定义了一个名为 `websocket` 的包，它包含了所有与 WebSocket 相关的函数和类型。

3. 导入了一个名为 ` common` 的包，它可能包含了一些通用的功能和工具。

4. 导入了一个名为 `config` 的包，它可能包含了一些配置相关的功能和类型。

5. 导入了一个名为 `log` 的包，它可能包含了一些日志相关的功能和类型。

6. 导入了一个名为 `redirector` 的包，它可能包含了一些 redirection 相关的功能和类型。

7. 导入了一个名为 `tunnel` 的包，它可能包含了一些 tunnel 相关的功能和类型。

8. 定义了一个名为 `Websocket` 的类型，它可能是一个 WebSocket 客户端或服务器。

9. 定义了一个名为 `connect` 的函数，它接收一个 WebSocket 目标 URI 和一些参数，并尝试连接到服务器。

10. 定义了一个名为 `send` 的函数，它接收一个 WebSocket 消息和一些参数，并尝试发送到服务器。

11. 定义了一个名为 `parseMessage` 的函数，它接收一个 WebSocket 消息，并解析成普通字符串。

12. 定义了一个名为 `handleError` 的函数，它接收一个 WebSocket 错误，并采取相应的措施。

13. 定义了一个名为 `logOnce` 的函数，它接收一个日志消息和一个配置项，并记录一次日志。

14. 定义了一个名为 `tlsAuthenticator` 的类型，它可能是一个使用 TLS 身份验证的函数。

15. 定义了一个名为 `tlsListen` 的函数，它接收一个 TLS 配置项和一个 WebSocket 服务器 URI，并启动一个 TLS 监听器。

16. 定义了一个名为 `setReady` 的函数，它接收一个 WebSocket 服务器 URI 和一些参数，并设置服务器为准备就绪状态。

17. 定义了一个名为 `start` 的函数，它接收一个 WebSocket 服务器 URI 和一些参数，并启动服务器。

18. 定义了一个名为 `stop` 的函数，它接收一个 WebSocket 服务器 URI 和一些参数，并停止服务器。

19. 定义了一个名为 ` WeSocketConfig `的结构体，它包含了所有 WebSocket 配置相关的字段。

20. 定义了一个名为 `WeSocketServer` 的类型，它包含了所有与 WebSocket 服务器相关的方法。

21. 定义了一个名为 `ListenAndServe` 的函数，它接收一个 URI 和一些配置项，并启动一个 HTTP 和 WebSocket 服务器。


```go
package websocket

import (
	"bufio"
	"context"
	"math/rand"
	"net"
	"net/http"
	"strings"
	"time"

	"golang.org/x/net/websocket"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/redirector"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码定义了一个名为 `fakeHTTPResponseWriter` 的 struct，它是一个模仿 HTTP 响应writer的山姆会员。它的作用是模拟 HTTP 请求并返回一个 HTTP 响应。

`fakeHTTPResponseWriter` 的 `Hijack` 方法使用 Hijack 方法从 `http.ResponseWriter` 类型中获取 `ReadWriter` 字段。`ReadWriter` 字段是一个 `bufio.ReadWriter` 类型，表示一个缓冲区IO套筒，用于向 `http.ResponseWriter` 输出内容。`Hijack` 方法返回一个 `net.Conn` 类型和一个 `bufio.ReadWriter` 类型的组合，以及一个表示是否成功完成写入的 `error` 类型。如果调用 `Hijack` 方法时没有错误，则创建一个新的 `net.Conn` 类型和一个 `bufio.ReadWriter` 类型的组合，并将 `ReadWriter` 字段的值设置为 `http.ResponseWriter` 类型中的 `Hijack` 字段。

`fakeHTTPResponseWriter` 的 `Server` 结构体定义了一个服务器的基本设置，包括设置服务器名称、端口、是否启用 WebSocket 等等。它的 `underlay` 字段是一个山姆会员，用于在 `fakeHTTPResponseWriter` 和 `http.ResponseWriter` 之间传输数据。它的 `hostname` 字段指定了服务器的主机名，`path` 字段指定了服务器接受的路径，`enabled` 字段指定了是否启用 WebSocket。

`fakeHTTPResponseWriter` 的 `cancel` 字段是一个取消信号，用于取消已经预订的 `Underlay` 通道。`cancel` 字段是一个 `context.CancelFunc` 类型，表示一个函数，用于处理已经取消的 `Underlay` 通道。

`fakeHTTPResponseWriter` 的 `timeout` 字段指定了服务器连接超时的时间。如果 `timeout` 字段没有被设置，则服务器将一直运行，直到被手动关闭。


```go
// Fake response writer
// Websocket ServeHTTP method uses Hijack method to get the ReadWriter
type fakeHTTPResponseWriter struct {
	http.Hijacker
	http.ResponseWriter

	ReadWriter *bufio.ReadWriter
	Conn       net.Conn
}

func (w *fakeHTTPResponseWriter) Hijack() (net.Conn, *bufio.ReadWriter, error) {
	return w.Conn, w.ReadWriter, nil
}

type Server struct {
	underlay  tunnel.Server
	hostname  string
	path      string
	enabled   bool
	redirAddr net.Addr
	redir     *redirector.Redirector
	ctx       context.Context
	cancel    context.CancelFunc
	timeout   time.Duration
}

```

This is a Rust implementation of a websocket server that listens for incoming connections and handles the websocket connection by opening a connection to a remotewebsocket server using a provided websocket configuration.

The websocket server listens for incoming connections on port 8080 and uses the provided websocket configuration to connect to the remotewebsocket server. The websocket server creates a connection to the remotewebsocket server and passes the provided websocket configuration to the `websocket.NewConfig` function.

The remotewebsocket server is created using the `websocket.NewConfig` function and a URL and an origin from an HTTP request. The websocket server listens for incoming connections and passes the provided websocket configuration to the `websocket.NewConfig` function.

The websocket server uses the `websocket` implementation to handle incoming connections and the `fakeHTTPResponseWriter` to write the response to the client.

The websocket server has a timeout set to 30 seconds and if the connection is closed within that time, the server will cancel the connection and return an error.

The websocket server is using a connection pool to handle the connection pool.

The input parameters for the function `InboundConn` is the `OutboundConn` and the context.

The `OutboundConn` is an instance of `OutboundConn` which is using `tcpConn` to connect to the remotewebsocket server.

The `InboundConn` is using a connection pool to handle the connection pool.

The function returns an instance of `InboundConn` if the connection was closed within the timeout or an error.


```go
func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := s.underlay.AcceptConn(&Tunnel{})
	if err != nil {
		return nil, common.NewError("websocket failed to accept connection from underlying server")
	}
	if !s.enabled {
		s.redir.Redirect(&redirector.Redirection{
			InboundConn: conn,
			RedirectTo:  s.redirAddr,
		})
		return nil, common.NewError("websocket is disabled. redirecting http request from " + conn.RemoteAddr().String())
	}
	rewindConn := common.NewRewindConn(conn)
	rewindConn.SetBufferSize(512)
	defer rewindConn.StopBuffering()
	rw := bufio.NewReadWriter(bufio.NewReader(rewindConn), bufio.NewWriter(rewindConn))
	req, err := http.ReadRequest(rw.Reader)
	if err != nil {
		log.Debug("invalid http request")
		rewindConn.Rewind()
		rewindConn.StopBuffering()
		s.redir.Redirect(&redirector.Redirection{
			InboundConn: rewindConn,
			RedirectTo:  s.redirAddr,
		})
		return nil, common.NewError("not a valid http request: " + conn.RemoteAddr().String()).Base(err)
	}
	if strings.ToLower(req.Header.Get("Upgrade")) != "websocket" || req.URL.Path != s.path {
		log.Debug("invalid http websocket handshake request")
		rewindConn.Rewind()
		rewindConn.StopBuffering()
		s.redir.Redirect(&redirector.Redirection{
			InboundConn: rewindConn,
			RedirectTo:  s.redirAddr,
		})
		return nil, common.NewError("not a valid websocket handshake request: " + conn.RemoteAddr().String()).Base(err)
	}

	handshake := make(chan struct{})

	url := "wss://" + s.hostname + s.path
	origin := "https://" + s.hostname
	wsConfig, err := websocket.NewConfig(url, origin)
	if err != nil {
		return nil, common.NewError("failed to create websocket config").Base(err)
	}
	var wsConn *websocket.Conn
	ctx, cancel := context.WithCancel(s.ctx)

	wsServer := websocket.Server{
		Config: *wsConfig,
		Handler: func(conn *websocket.Conn) {
			wsConn = conn                              // store the websocket after handshaking
			wsConn.PayloadType = websocket.BinaryFrame // treat it as a binary websocket

			log.Debug("websocket obtained")
			handshake <- struct{}{}
			// this function SHOULD NOT return unless the connection is ended
			// or the websocket will be closed by ServeHTTP method
			<-ctx.Done()
			log.Debug("websocket closed")
		},
		Handshake: func(wsConfig *websocket.Config, httpRequest *http.Request) error {
			log.Debug("websocket url", httpRequest.URL, "origin", httpRequest.Header.Get("Origin"))
			return nil
		},
	}

	respWriter := &fakeHTTPResponseWriter{
		Conn:       conn,
		ReadWriter: rw,
	}
	go wsServer.ServeHTTP(respWriter, req)

	select {
	case <-handshake:
	case <-time.After(s.timeout):
	}

	if wsConn == nil {
		cancel()
		return nil, common.NewError("websocket failed to handshake")
	}

	return &InboundConn{
		OutboundConn: OutboundConn{
			tcpConn: conn,
			Conn:    wsConn,
		},
		ctx:    ctx,
		cancel: cancel,
	}, nil
}

```

此代码定义了两个函数，函数一是`AcceptPacket`，函数二是`NewServer`。

函数一：`AcceptPacket`接收一个`tunnel.Tunnel`类型的参数，代表一个网络隧道。函数的返回值是一个`tunnel.PacketConn`类型，代表一个连接到该网络隧道的数据报。函数的错误处理是一个`common.NewError`类型的实例，代表一个自定义的错误，例如：

not supported：网络隧道不支持某种功能

函数二：`NewServer`接收一个`context.Context`类型的参数，一个`tunnel.Server`类型的参数和一个`config.Config`类型的参数。函数的返回值是一个`*Server`类型的指针，代表一个可以管理网络隧道的服务器实例。函数的错误处理是一个`common.NewError`类型的实例，代表一个自定义的错误，例如：

websocket path must start with "/”：WebSocket路径必须以"/"开头

`RemoteHost`和`RemotePort`字段在函数中被定义为服务器配置中的`RemoteHost`和`RemotePort`参数。如果这些参数为空，函数将会记录一个警告信息。如果服务器配置中`Websocket`字段启用，那么函数将会创建一个WebSocket服务器，并尝试通过该服务器接收数据报。如果WebSocket服务器成功创建，那么函数将会返回一个`tunnel.Server`类型的实例，代表一个可以管理网络隧道的服务器实例。如果WebSocket服务器失败，那么函数将会返回一个`nil`值。


```go
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	return nil, common.NewError("not supported")
}

func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	if cfg.Websocket.Enabled {
		if !strings.HasPrefix(cfg.Websocket.Path, "/") {
			return nil, common.NewError("websocket path must start with \"/\"")
		}
	}
	if cfg.RemoteHost == "" {
		log.Warn("empty websocket redirection hostname")
		cfg.RemoteHost = cfg.Websocket.Host
	}
	if cfg.RemotePort == 0 {
		log.Warn("empty websocket redirection port")
		cfg.RemotePort = 80
	}
	ctx, cancel := context.WithCancel(ctx)
	log.Debug("websocket server created")
	return &Server{
		enabled:   cfg.Websocket.Enabled,
		hostname:  cfg.Websocket.Host,
		path:      cfg.Websocket.Path,
		ctx:       ctx,
		cancel:    cancel,
		underlay:  underlay,
		timeout:   time.Second * time.Duration(rand.Intn(10)+5),
		redir:     redirector.NewRedirector(ctx),
		redirAddr: tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort),
	}, nil
}

```

# `tunnel/websocket/tunnel.go`

这段代码定义了一个名为 "websocket" 的包，它包含了以下内容：

1. 导入了一个名为 "context" 的的外部包，它可能用于在代码中处理连接上下文。
2. 导入了名为 "websocket" 的包，它自己定义了一个名为 "Tunnel" 的类型，该类型表示一个 WebSocket 通道。
3. 在 "Tunnel" 类型的定义中，定义了一个名为 "*Tunnel" 的指针类型，该指针类型包含了一个名为 "Name" 的字段，它的值为 "WEBSOCKET"。
4. 在 "Tunnel" 类型中，定义了一个名为 "(*Tunnel)Name()" 的名为 "Name" 的方法，该方法返回一个字符串 "WEBSOCKET"。
5. 在 "websocket" 包的其他部分，可能还会定义其他方法和函数，用于在 WebSocket 通道中处理其他操作。

由于没有提供具体的代码内容，因此无法进一步解释这些代码的作用。


```go
package websocket

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "WEBSOCKET"

type Tunnel struct{}

func (*Tunnel) Name() string {
	return Name
}

```

这是一段使用Go编程语言编写的函数指针类型，定义了两个名为NewServer和NewClient的函数，用于创建基于Underlay的Tunnel服务器和客户端。

函数的作用如下：

1. `NewServer`函数接收一个上下文上下文(Context)和一个基于Underlay的Server对象作为参数，返回一个新创建的Server对象，不返回错误信息。

2. `NewClient`函数接收一个上下文上下文(Context)和一个基于Underlay的Client对象作为参数，返回一个新创建的Client对象，不返回错误信息。

3. `init`函数用于初始化Tunnel类型的函数指针。

函数指针类型是一种特殊类型的函数指针，可以用来创建一个Tunnel类型的函数指针实例，而不需要显式地指定函数的实际实现。在这种情况下，上述函数指针允许您创建一个名为NewServer的函数指针实例，该实例将调用Underlay中的名为NewServer的函数，以创建一个新的基于Underlay的Tunnel服务器。同样，您可以创建一个名为NewClient的函数指针实例，该实例将调用Underlay中的名为NewClient的函数，以创建一个新的基于Underlay的Tunnel客户端。


```go
func (*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, underlay)
}

func (*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, underlay)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/websocket/websocket_test.go`

这段代码定义了一个名为 "websocket" 的包，它包含了使用 WebSocket 进行网络通信的相关代码。

该包通过导入 "net"、"strings" 和 "testing" 包来使用网络、字符串和testing包。它还通过导入 "websocket" 包本身来使用 WebSocket协议。

接下来，该包定义了一个名为 "TooManyTokens" 的函数，它从命令行接收一个 URL 和一个令牌数。如果 URL 和令牌数正确，该函数会尝试连接到指定的 WebSocket 服务器，并返回一个确认消息。如果 URL 或令牌数有误，该函数会将错误信息打印到控制台。

此外，该包还定义了一个名为 "Dialogue" 的函数，该函数在尝试连接到 WebSocket 服务器之后，如果连接成功，它会进入一个无限循环，等待消息。如果连接失败，该函数会等待一段时间后重试，也可能会发送一些确认消息给服务器以进行错误提示。


```go
package websocket

import (
	"context"
	"fmt"
	"net"
	"strings"
	"sync"
	"testing"
	"time"

	"golang.org/x/net/websocket"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This appears to be a Go program that sets up a reverse proxy and a server using the FreeEthernet network on a Linux system. The program uses the Echo server and client to demonstrate how to establish a connection between the proxy and the client.

The program sets up a connection between the proxy and the client using a remote port specified by the `-1` or `--remote-port` option. If the specified port is not already in use, the program creates a new port and sets it as the remote port for the proxy.

The program also sets up a tunnel connection between the client and the server using the `-t` or `--tcp-transport` option. This allows the client to send traffic to the server through the tunnel, which will be received by the server instead of the original client.

The program uses the `transport.NewClient` and `transport.NewServer` functions to create a new `transport.Client` and `transport.Server` instance, respectively. These instances are used to establish a connection to the remote server and create a new `transport.Conn` instance for the tunnel connection, respectively.

The program uses a `sync.WaitGroup` to synchronize the completion of the `transport.Client` and `transport.Server` connections. It also uses a `time.Sleep` to wait for a second before closing the connection to the server.

The program checks if the remote connection is a WebSocket (`-w`) connection by examining the `remoteAddr` property of the `transport.Conn` instance. If the connection is a WebSocket, the program checks if the remote hostname uses the "ws" protocol. If the connection is not a WebSocket, the program checks if the connection has been closed correctly.

If the connection is a WebSocket and the remote hostname uses the "ws" protocol, the program checks if the connection has been closed correctly. If the connection has not been closed correctly, the program fails with an error. If the connection is not a WebSocket or the connection has been closed correctly, the program does not fail with an error.


```go
func TestWebsocket(t *testing.T) {
	cfg := &Config{
		Websocket: WebsocketConfig{
			Enabled: true,
			Host:    "localhost",
			Path:    "/ws",
		},
	}

	ctx := config.WithConfig(context.Background(), Name, cfg)

	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  port,
		RemoteHost: "127.0.0.1",
		RemotePort: port,
	}
	freedomCfg := &freedom.Config{}
	ctx = config.WithConfig(ctx, transport.Name, transportConfig)
	ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)
	tcpClient, err := transport.NewClient(ctx, nil)
	common.Must(err)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	c, err := NewClient(ctx, tcpClient)
	common.Must(err)
	s, err := NewServer(ctx, tcpServer)
	var conn2 tunnel.Conn
	wg := sync.WaitGroup{}
	wg.Add(1)
	go func() {
		conn2, err = s.AcceptConn(nil)
		common.Must(err)
		wg.Done()
	}()
	time.Sleep(time.Second)
	conn1, err := c.DialConn(nil, nil)
	common.Must(err)
	wg.Wait()
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}

	if strings.HasPrefix(conn1.RemoteAddr().String(), "ws") {
		t.Fail()
	}
	if strings.HasPrefix(conn2.RemoteAddr().String(), "ws") {
		t.Fail()
	}

	conn1.Close()
	conn2.Close()
	s.Close()
	c.Close()
}

```

这段代码是一个 Go 语言中的测试函数，名为 "TestRedirect"。它的作用是测试一个名为 "Redirect" 的功能，即当客户端连接到服务器时，服务器会通过一个 Redirect 重定向到另一个服务器，测试服务器是否能够正确地工作。

具体来说，这段代码以下的几个步骤：

1. 配置服务器连接参数，包括远程主机（127.0.0.1）、端口（通过 HTTP 80 协议获取）、WebSocket 配置等。
2. 读取客户端 HTTP 80 协议获取的端口号，并将其存储到 "cfg.RemotePort" 字段中。
3. 创建一个名为 "transportConfig" 的传输配置对象，包括本地主机（127.0.0.1）和本地端口（通过 HTTP 80 协议获取的端口号），用于创建一个传输上下文。
4. 创建一个名为 "tcpServer" 的传输服务器实例，用于创建一个 HTTP 服务器，并将上面创建的传输上下文传递给它。
5. 创建一个名为 "NewServer" 的函数，用于创建一个 HTTP 服务器实例，它接收一个传输上下文作为参数，并返回一个空的 HTTP 服务器实例。
6. 创建一个名为 "AcceptConn" 的函数，用于接受客户端连接，然后等待客户端发送数据。
7. 创建一个名为 "Run" 的函数，用于模拟客户端连接到服务器的操作，创建一个 HTTP 连接，并发送一个请求。
8. 调用 "Run" 函数，模拟客户端连接到服务器的操作，并发送一个请求。
9. 如果服务器接收到数据，就关闭连接并返回，否则阻塞等待，直到客户端连接断开。
10. 循环等待客户端连接，然后关闭连接并打印错误信息。

"Redirect" 功能的具体实现并不包括在这段代码中，而是由调用方通过调用 "https://localhost/wrong-path" 来实现。这段代码的作用是测试 "Redirect" 功能是否正常工作。


```go
func TestRedirect(t *testing.T) {
	cfg := &Config{
		RemoteHost: "127.0.0.1",
		Websocket: WebsocketConfig{
			Enabled: true,
			Host:    "localhost",
			Path:    "/ws",
		},
	}
	fmt.Sscanf(util.HTTPPort, "%d", &cfg.RemotePort)
	ctx := config.WithConfig(context.Background(), Name, cfg)

	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost: "127.0.0.1",
		LocalPort: port,
	}
	ctx = config.WithConfig(ctx, transport.Name, transportConfig)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	s, err := NewServer(ctx, tcpServer)
	common.Must(err)

	go func() {
		_, err := s.AcceptConn(nil)
		if err == nil {
			t.Fail()
		}
	}()
	time.Sleep(time.Second)
	conn, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))
	common.Must(err)
	url := "wss://localhost/wrong-path"
	origin := "https://localhost"
	wsConfig, err := websocket.NewConfig(url, origin)
	common.Must(err)
	_, err = websocket.NewClient(wsConfig, conn)
	if err == nil {
		t.Fail()
	}
	conn.Close()

	s.Close()
}

```

# `url/option.go`

这段代码定义了一个名为 "url" 的包，它import了以下几个外部库：

- "encoding/json"：用于将JSON数据解析成普通字符串。
- "flag"：用于解析命令行参数。
- "net"：用于访问网络。
- "strconv"：用于字符串转义。
- "strings"：用于截取字符串中的任何字符。

然后，它导入了以下类型：

- "encoding/json.秤"：类型为 "encoding/json" 的秤，用于将JSON数据解析成普通字符串。
- "flag.解析器"：类型为 "flag.解析器" 的解析器，用于解析命令行参数。
- "net.URL"：类型为 "net.URL" 的 URL 类，用于访问网络。
- "strconv.殴打者"：类型为 "strconv.殴打者" 的字符串转义函数。
- "strings.工具"：类型为 "strings.工具" 的字符串截取函数。

最后，在 "url" 包内部，定义了一些函数和方法，用于处理 URL 相关操作。例如，定义了一个名为 "FetchURL" 的函数，用于获取 URL 中的资源。


```go
package url

import (
	"encoding/json"
	"flag"
	"net"
	"strconv"
	"strings"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/option"
	"github.com/p4gefau1t/trojan-go/proxy"
)

```

这段代码定义了一个名为 "Name" 的常量，其值为 "URL"。

定义了一个名为 "Websocket" 的类型，该类型有一个名为 "enabled" 的布尔字段，其值为 true。

定义了一个名为 "TLS" 的类型，该类型有一个名为 "SNI" 的字符串字段，其值为 SNI。

定义了一个名为 "Shadowsocks" 的类型，该类型有一个名为 "enabled" 的布尔字段，其值为 true。还有一个名为 "Method" 的字段，其值为一个字符串，表示使用的方法。还有一个名为 "Password" 的字段，其值为一个字符串。

最后，将这三个类型都定义为了 JSON 序列化接口的子类型，以便可以使用它们来生成 JSON 序列化的数据。


```go
const Name = "URL"

type Websocket struct {
	Enabled bool   `json:"enabled"`
	Host    string `json:"host"`
	Path    string `json:"path"`
}

type TLS struct {
	SNI string `json:"sni"`
}

type Shadowsocks struct {
	Enabled  bool   `json:"enabled"`
	Method   string `json:"method"`
	Password string `json:"password"`
}

```

这段代码定义了一个名为 `Mux` 的数据结构体，它包含一个名为 `Enabled` 的布尔值，该值表示是否启用某种功能。这个数据结构体使用了 JSON 序列化技术，以便在将来的应用程序中交换数据时进行序列化和反序列化。

接着，定义了一个名为 `API` 的数据结构体，它包含一个名为 `Enabled` 的布尔值，一个名为 `APIHost` 的字符串，一个名为 `APIPort` 的整数。这个数据结构体同样使用了 JSON 序列化技术，以便在将来的应用程序中交换数据时进行序列化和反序列化。

接下来，定义了一个名为 `UrlConfig` 的数据结构体，它包含一系列关于 URL 配置的属性，包括 `RunType`、`LocalAddr`、`LocalPort`、`RemoteAddr`、`RemotePort`、`Password`、`Websocket`、`Shadowsocks` 和 `TLS`。这些属性同样使用了 JSON 序列化技术，以便在将来的应用程序中交换数据时进行序列化和反序列化。

然后，定义了一个名为 `Mux` 的数据结构体，它包含一个名为 `API` 的数据结构体，一个名为 `UrlConfig` 的数据结构体和一个名为 `Mux` 的数据结构体。这个数据结构体使用了 JSON 序列化技术，以便在将来的应用程序中交换数据时进行序列化和反序列化。

最后，定义了一个名为 `API` 的数据结构体，它包含一个名为 `API` 的字符串，一个名为 `Enabled` 的布尔值和一个名为 `APIHost` 的字符串。这个数据结构体同样使用了 JSON 序列化技术，以便在将来的应用程序中交换数据时进行序列化和反序列化。


```go
type Mux struct {
	Enabled bool `json:"enabled"`
}

type API struct {
	Enabled bool   `json:"enabled"`
	APIHost string `json:"api_addr"`
	APIPort int    `json:"api_port"`
}

type UrlConfig struct {
	RunType     string   `json:"run_type"`
	LocalAddr   string   `json:"local_addr"`
	LocalPort   int      `json:"local_port"`
	RemoteAddr  string   `json:"remote_addr"`
	RemotePort  int      `json:"remote_port"`
	Password    []string `json:"password"`
	Websocket   `json:"websocket"`
	Shadowsocks `json:"shadowsocks"`
	TLS         `json:"ssl"`
	Mux         `json:"mux"`
	API         `json:"api"`
}

```

This is a Go language function that configures a proxy. It takes a single argument, which is a JSON-formatted string representing the proxy configuration. The function parse


```go
type url struct {
	url    *string
	option *string
}

func (u *url) Name() string {
	return Name
}

func (u *url) Handle() error {
	if u.url == nil || *u.url == "" {
		return common.NewError("")
	}
	info, err := NewShareInfoFromURL(*u.url)
	if err != nil {
		log.Fatal(err)
	}
	wsEnabled := false
	if info.Type == ShareInfoTypeWebSocket {
		wsEnabled = true
	}
	ssEnabled := false
	ssPassword := ""
	ssMethod := ""
	if strings.HasPrefix(info.Encryption, "ss;") {
		ssEnabled = true
		ssConfig := strings.Split(info.Encryption[3:], ":")
		if len(ssConfig) != 2 {
			log.Fatalf("invalid shadowsocks config: %s", info.Encryption)
		}
		ssMethod = ssConfig[0]
		ssPassword = ssConfig[1]
	}
	muxEnabled := false
	listenHost := "127.0.0.1"
	listenPort := 1080

	apiEnabled := false
	apiHost := "127.0.0.1"
	apiPort := 10000

	options := strings.Split(*u.option, ";")
	for _, o := range options {
		key := ""
		val := ""
		l := strings.Split(o, "=")
		if len(l) != 2 {
			log.Fatal("option format error, no \"key=value\" pair found:", o)
		}
		key = l[0]
		val = l[1]
		switch key {
		case "mux":
			muxEnabled, err = strconv.ParseBool(val)
			if err != nil {
				log.Fatal(err)
			}
		case "listen":
			h, p, err := net.SplitHostPort(val)
			if err != nil {
				log.Fatal(err)
			}
			listenHost = h
			lp, err := strconv.Atoi(p)
			if err != nil {
				log.Fatal(err)
			}
			listenPort = lp
		case "api":
			apiEnabled = true
			h, p, err := net.SplitHostPort(val)
			if err != nil {
				log.Fatal(err)
			}
			apiHost = h
			lp, err := strconv.Atoi(p)
			if err != nil {
				log.Fatal(err)
			}
			apiPort = lp
		default:
			log.Fatal("invalid option", o)
		}
	}
	config := UrlConfig{
		RunType:    "client",
		LocalAddr:  listenHost,
		LocalPort:  listenPort,
		RemoteAddr: info.TrojanHost,
		RemotePort: int(info.Port),
		Password:   []string{info.TrojanPassword},
		TLS: TLS{
			SNI: info.SNI,
		},
		Websocket: Websocket{
			Enabled: wsEnabled,
			Path:    info.Path,
			Host:    info.Host,
		},
		Mux: Mux{
			Enabled: muxEnabled,
		},
		Shadowsocks: Shadowsocks{
			Enabled:  ssEnabled,
			Password: ssPassword,
			Method:   ssMethod,
		},
		API: API{
			Enabled: apiEnabled,
			APIHost: apiHost,
			APIPort: apiPort,
		},
	}
	data, err := json.Marshal(&config)
	if err != nil {
		log.Fatal(err)
	}
	log.Debug(string(data))
	client, err := proxy.NewProxyFromConfigData(data, true)
	if err != nil {
		log.Fatal(err)
	}
	return client.Run()
}

```

这段代码定义了一个名为“func”的函数，接受一个名为“url”的指针参数，并返回一个整数类型的“Priority”函数。

具体来说，函数在内部使用了以下步骤：

1. 将10赋值给整型变量“return-value”，并将其返回。
2. 在函数内部使用“option.RegisterHandler”方法，将一个新注册的“url”类型的Handler与“url”指针参数关联起来。
3. 在“handler”内部，使用“url”指针参数，通过“flag.String”方法设置一个选项“url”，通过“option.String”方法设置一个选项“url-option”。
4. 通过调用“option.RegisterHandler”方法，将该Handler注册到“option”选项中，从而将设置的选项传递给“func”函数。
5. 在“func”函数内部，直接返回10。


```go
func (u *url) Priority() int {
	return 10
}

func init() {
	option.RegisterHandler(&url{
		url:    flag.String("url", "", "Setup trojan-go client with a url link"),
		option: flag.String("url-option", "mux=true;listen=127.0.0.1:1080", "URL mode options"),
	})
}

```

# `url/option_test.go`

This is a Go test case that tests the `Url_Handle` function of the `proxy/client` package. The function has several options that can be used to configure the URL handle, including the proxy mode (`mux`), SSL/TLS encryption (`encryption`), and HTTP/Websocket support (`type`).

The `Url_Handle` function takes a single argument, which is a `url` struct that contains the base URL and any options passed to it. The function then calls the `Handle` method on the `url` struct, which should delegate to the `url.Client` backend and make the request to the base URL.

The `url.Client` backend should return an error in case there was an issue with the request, such as an HTTP error or a timeout. The error should be caught by the `errChan` channel and printed to the `t.Fatal` function if it occurs within a certain amount of time.

The `urlCases` array contains a list of base URLs and options that the `Url_Handle` function can handle. The `optionCases` array contains a list of options that can be used to configure the `url` struct.

This test case can be run using the `go test` command and should provide a good coverage of the different URL options that can be used.


```go
package url

import (
	"testing"
	"time"

	_ "github.com/p4gefau1t/trojan-go/proxy/client"
)

func TestUrl_Handle(t *testing.T) {
	urlCases := []string{
		"trojan-go://password@server.com",
		"trojan-go://password@server.com/?type=ws&host=baidu.com&path=%2fwspath",
		"trojan-go://password@server.com/?encryption=ss%3baes-256-gcm%3afuckgfw",
		"trojan-go://password@server.com/?type=ws&host=baidu.com&path=%2fwspath&encryption=ss%3Baes-256-gcm%3Afuckgfw",
	}
	optionCases := []string{
		"mux=true;listen=127.0.0.1:0",
		"mux=false;listen=127.0.0.1:0",
		"mux=false;listen=127.0.0.1:0;api=127.0.0.1:0",
	}

	for _, s := range urlCases {
		for _, option := range optionCases {
			s := s
			option := option
			u := &url{
				url:    &s,
				option: &option,
			}
			u.Name()
			u.Priority()

			errChan := make(chan error, 1)
			go func() {
				errChan <- u.Handle()
			}()

			select {
			case err := <-errChan:
				t.Fatal(err)
			case <-time.After(time.Second * 1):
			}
		}
	}
}

```

# `url/share_link.go`

这段代码是一个 Go 语言中的 `url` 包，作用是实现 WebSocket 服务器和客户端之间的交互。它主要实现了以下功能：

1. 定义了两个内置变量 `ShareInfoTypeOriginal` 和 `ShareInfoTypeWebSocket`，用于存储 WebSocket 类型和原始类型。

2. 导入了自定义的 `neturl` 包，用于实现网络 URL 解析和构建。

3. 定义了一个名为 `fmt.Printf` 的函数，用于将字符串和错误信息格式化并输出。

4. 定义了一个名为 `strings.ToLower` 的函数，用于将字符串转换为小写。

5. 定义了一个名为 `CreateShareInfo` 的函数，用于创建一个新的 `ShareInfo` 对象，它支持原始类型和 WebSocket 类型。

6. 定义了一个名为 `ListenAndServe` 的函数，用于监听端口并处理客户端的请求。它通过调用 `CreateShareInfo` 函数来创建新的 `ShareInfo` 对象，并根据客户端请求的类型选择使用哪种类型构建。

7. 最后，通过组合这些函数，实现了 WebSocket 服务器和客户端之间的交互，当客户端发送请求时，服务器会根据请求类型选择对应的 `ShareInfo` 对象，然后返回给客户端。


```go
package url

import (
	"errors"
	"fmt"
	neturl "net/url"
	"strconv"
	"strings"
)

const (
	ShareInfoTypeOriginal  = "original"
	ShareInfoTypeWebSocket = "ws"
)

```

这段代码定义了两个哈希表 validTypes 和 validEncryptionProviders，分别存储了 ShareInfoTypeOriginal 和 ShareInfoTypeWebSocket 类型的数据。

哈希表是一种数据结构，它通过哈希函数将键映射到值。在 Go 中，哈希表通常是基于 key 和值的键值对。哈希表可以用来快速查找和插入数据。

在 Go 语言中，map 是哈希表的標準類型。它允许您键入和获取键值对。通过 map[string]struct{}{，您可以将字符串键映射到自定义结构体类型的值。

var validTypes = map[string]struct{}{
	ShareInfoTypeOriginal:  {},
	ShareInfoTypeWebSocket: {},
}

var validEncryptionProviders = map[string]struct{}{
	"ss":   {},
	"none": {},
}

var validSSEncryptionMap = map[string]struct{}{
	"aes-128-gcm":            {},
	"aes-256-gcm":            {},
	"chacha20-ietf-poly1305": {},
}

这段代码主要作用是定义了两个哈希表 validTypes 和 validEncryptionProviders，存储了 ShareInfoTypeOriginal 和 ShareInfoTypeWebSocket 类型的数据。validSSEncryptionMap 是 map[string]struct{}{中的一个哈希表，存储了 AES-128-GCM、AES-256-GCM 和 Chacha20-IETF-Poly1305 等类型的加密提供者。


```go
var validTypes = map[string]struct{}{
	ShareInfoTypeOriginal:  {},
	ShareInfoTypeWebSocket: {},
}

var validEncryptionProviders = map[string]struct{}{
	"ss":   {},
	"none": {},
}

var validSSEncryptionMap = map[string]struct{}{
	"aes-128-gcm":            {},
	"aes-256-gcm":            {},
	"chacha20-ietf-poly1305": {},
}

```

这是一个结构体类型，表示一个与节点进行通信的ShareInfo对象。

这个结构体定义了以下字段：

* TrojanHost：节点的IP地址或域名。
* Port：节点端口号。
* TrojanPassword：用于身份验证的密码。
* SNI：安全名称标识（SNI）。
* Type：数据类型，可能是用户名或服务器名。
* Host：HTTP主机头。
* Path：WebSocket或H2路径。
* Encryption：是否启用数据加密。
* Plugin：插件设置。
* Description：节点说明。

这个结构体可以用来创建一个ShareInfo实例，并通过一些字段来设置其属性。然后，可以使用一些方法来访问和修改这个实例，例如，设置其SNI、Type、Host等字段。


```go
type ShareInfo struct {
	TrojanHost     string // 节点 IP / 域名
	Port           uint16 // 节点端口
	TrojanPassword string // Trojan 密码

	SNI  string // SNI
	Type string // 类型
	Host string // HTTP Host Header

	Path       string // WebSocket / H2 Path
	Encryption string // 额外加密
	Plugin     string // 插件设定

	Description string // 节点说明
}

```

This is a Go function that handles the initialization of an SSH client connection. It takes in information about the connection, such as the hostname, username, and encryption algorithm, and performs the following steps:

1. Validates the provided encryption algorithm against a list of supported encryption providers.
2. If the encryption provider is valid, validates the provided password against the list of valid SSH passwords for that provider.
3. If the encryption provider is "ss", performs additional SSH-specific validations.
4. Plugins are an optional field, but if present, the function will apply the corresponding plugin to the connection.
5. Generates a description of the connection.
6. Returns an error if any step failed.

Here is a more detailed explanation of the function:

- The `initialize` function takes a connection object as input, which includes the hostname, username, and encryption algorithm.
- The first step is to validate the encryption algorithm against a list of supported encryption providers. This is done using the `strings.SplitN` function, which splits the input string by the `;` character and returns an array with the first element of each partition or the second element if the partition is empty. The function takes two arguments, the first one being the string to be split and the second one being the array size. This function is applied to the encryption algorithm parameter, resulting in an array containing the supported encryption algorithms. The algorithm is then compared to the list of supported algorithms to determine if it is valid.
- If the encryption provider is valid, the next step is to validate the provided password against the list of valid SSH passwords for that provider. This is done using the `contrib.sessions.ValidPassword` function, which is a built-in function in the `contrib.sessions` package. This function takes a password string as input and returns a boolean indicating if the password is valid.
- If the encryption provider is "ss", the connection is required to use SSH and the `ss-agent` is installed on the system. This is done by first checking if the `ss-agent` is present, and if it is not, installing it using the `setup.sh` command. Then, the function checks the required SSH method (`'m'` or `'p'`) and the required password for that method.
- If the connection information is valid and the encryption provider is valid, the function will apply the corresponding plugin to the connection. This is done using the `plugins` parameter of the connection object, which is a list of supported plugins for the connection. The `plugins` field is an optional field and if not provided, the function will not apply any plugins.
- The function then generates a description of the connection using the `parse.Fragment` function, which is a part of the `text/i` package. This function takes a string as input and returns the first fragment of the text that cannot be fragmented any further.
- Finally, the function returns an error if any step failed, or if the connection cannot be established.

Note: The code in the function assumes that the initialization of the connection was successful and the user provided the correct information. The error handling in case of invalid inputs or unsupported plugins is not covered in the code provided.


```go
func NewShareInfoFromURL(shareLink string) (info ShareInfo, e error) {
	// share link must be valid url
	parse, e := neturl.Parse(shareLink)
	if e != nil {
		e = fmt.Errorf("invalid url: %s", e.Error())
		return
	}

	// share link must have `trojan-go://` scheme
	if parse.Scheme != "trojan-go" {
		e = errors.New("url does not have a trojan-go:// scheme")
		return
	}

	// password
	if info.TrojanPassword = parse.User.Username(); info.TrojanPassword == "" {
		e = errors.New("no password specified")
		return
	} else if _, hasPassword := parse.User.Password(); hasPassword {
		e = errors.New("password possibly missing percentage encoding for colon")
		return
	}

	// trojanHost: not empty & strip [] from IPv6 addresses
	if info.TrojanHost = parse.Hostname(); info.TrojanHost == "" {
		e = errors.New("host is empty")
		return
	}

	// port
	if info.Port, e = handleTrojanPort(parse.Port()); e != nil {
		return
	}

	// strictly parse the query
	query, e := neturl.ParseQuery(parse.RawQuery)
	if e != nil {
		return
	}

	// sni
	if SNIs, ok := query["sni"]; !ok {
		info.SNI = info.TrojanHost
	} else if len(SNIs) > 1 {
		e = errors.New("multiple SNIs")
		return
	} else if info.SNI = SNIs[0]; info.SNI == "" {
		e = errors.New("empty SNI")
		return
	}

	// type
	if types, ok := query["type"]; !ok {
		info.Type = ShareInfoTypeOriginal
	} else if len(types) > 1 {
		e = errors.New("multiple transport types")
		return
	} else if info.Type = types[0]; info.Type == "" {
		e = errors.New("empty transport type")
		return
	} else if _, ok := validTypes[info.Type]; !ok {
		e = fmt.Errorf("unknown transport type: %s", info.Type)
		return
	}

	// host
	if hosts, ok := query["host"]; !ok {
		info.Host = info.TrojanHost
	} else if len(hosts) > 1 {
		e = errors.New("multiple hosts")
		return
	} else if info.Host = hosts[0]; info.Host == "" {
		e = errors.New("empty host")
		return
	}

	// path
	if info.Type == ShareInfoTypeWebSocket {
		if paths, ok := query["path"]; !ok {
			e = errors.New("path is required in websocket")
			return
		} else if len(paths) > 1 {
			e = errors.New("multiple paths")
			return
		} else if info.Path = paths[0]; info.Path == "" {
			e = errors.New("empty path")
			return
		}

		if !strings.HasPrefix(info.Path, "/") {
			e = errors.New("path must start with /")
			return
		}
	}

	// encryption
	if encryptionArr, ok := query["encryption"]; !ok {
		// no encryption. that's okay.
	} else if len(encryptionArr) > 1 {
		e = errors.New("multiple encryption fields")
		return
	} else if info.Encryption = encryptionArr[0]; info.Encryption == "" {
		e = errors.New("empty encryption")
		return
	} else {
		encryptionParts := strings.SplitN(info.Encryption, ";", 2)
		encryptionProviderName := encryptionParts[0]

		if _, ok := validEncryptionProviders[encryptionProviderName]; !ok {
			e = fmt.Errorf("unsupported encryption provider name: %s", encryptionProviderName)
			return
		}

		var encryptionParams string
		if len(encryptionParts) >= 2 {
			encryptionParams = encryptionParts[1]
		}

		if encryptionProviderName == "ss" {
			ssParams := strings.SplitN(encryptionParams, ":", 2)
			if len(ssParams) < 2 {
				e = errors.New("missing ss password")
				return
			}

			ssMethod, ssPassword := ssParams[0], ssParams[1]
			if _, ok := validSSEncryptionMap[ssMethod]; !ok {
				e = fmt.Errorf("unsupported ss method: %s", ssMethod)
				return
			}

			if ssPassword == "" {
				e = errors.New("ss password cannot be empty")
				return
			}
		}
	}

	// plugin
	if plugins, ok := query["plugin"]; !ok {
		// no plugin. that's okay.
	} else if len(plugins) > 1 {
		e = errors.New("multiple plugins")
		return
	} else if info.Plugin = plugins[0]; info.Plugin == "" {
		e = errors.New("empty plugin")
		return
	}

	// description
	info.Description = parse.Fragment

	return
}

```

这段代码定义了一个名为handleTrojanPort的函数，用于处理传入的传输协议（TCP或UDP）和端口号。其作用如下：

1. 如果传入的字符串为空，函数返回443和Nil值。
2. 如果传入的端口号解析为无效的整数，函数返回，并附带一个错误信息。
3. 如果传入的端口号在1到65535之间，函数返回一个有效的端口号。
4. 如果传入的端口号不在1到65535之间，函数返回并附带一个错误信息。

总结起来，这段代码定义了一个函数，可以接受一个传输协议（TCP或UDP）和一个端口号，对传入的参数进行验证，并在需要时返回相应的错误信息。


```go
func handleTrojanPort(p string) (port uint16, e error) {
	if p == "" {
		return 443, nil
	}

	portParsed, e := strconv.Atoi(p)
	if e != nil {
		return
	}

	if portParsed < 1 || portParsed > 65535 {
		e = fmt.Errorf("invalid port %d", portParsed)
		return
	}

	port = uint16(portParsed)
	return
}

```

# `url/share_link_test.go`

这段代码是一个 Go 语言 package，名为 "url"，用于测试网站管道（URL）的相关功能。它主要实现了两个测试函数：TestHandleTrojanPortDefault 和 TestGetTrojanPortDefault。

1. TestHandleTrojanPortDefault 函数测试默认情况下处理 Trojan 端口。其作用是：接收一个空字符串（""）并尝试将其转换为端口号，然后输出结果。

2. TestGetTrojanPortDefault 函数测试从预定义的几个 Trojan 端口号（8080，8081，8082）中获取随机端口号。其作用是：尝试从预定义的端口号中随机选择一个并返回，然后输出结果。

这两个测试函数的实现主要涉及以下步骤：

1. 导入必要的库：crand、fmt 和 testing。
2. 实现 TestHandleTrojanPortDefault 和 TestGetTrojanPortDefault 函数。
3. 在函数内部处理从外部传入的参数。
4. 在测试时，通过调用 handleTrojanPort 函数，传入不同的参数，检查其是否能够正确处理 empty 和从预定义的端口号中随机选择几个不同的结果。

handleTrojanPort 的实现细节并不在上述代码中，这个函数的具体实现可能因实际需求而异。在这个例子中，我们只是简单地实现了两个测试函数。


```go
package url

import (
	crand "crypto/rand"
	"fmt"
	"io"
	"io/ioutil"
	"testing"

	"github.com/stretchr/testify/assert"
)

func TestHandleTrojanPort_Default(t *testing.T) {
	port, e := handleTrojanPort("")
	assert.Nil(t, e, "empty port should not error")
	assert.EqualValues(t, 443, port, "empty port should fallback to 443")
}

```

这两段代码是测试用例，分别对handleTrojanPort函数进行测试。handleTrojanPort函数的作用是处理通过trojan-port 工具实现的TCP连接，该工具可以在不关闭控制台的情况下监听telnet连接。

具体来说，如果传入的端口号是一个数字，则handleTrojanPort函数会尝试使用该端口与远程主机建立连接，如果连接成功，则说明该端口是有效的。如果传入的端口号不是数字，或者不是有效的端口号，则会输出错误信息，并保住原始的测试用例。

在测试中，handleTrojanPort函数分别对“443”、“8080”、“10086”、“80”、“65535”、“1”这六个端口进行了测试，以验证其有效性。如果任何测试用例的运行结果表明了这些端口都能正常使用，则这些端口被认为是有效的。


```go
func TestHandleTrojanPort_NotNumber(t *testing.T) {
	_, e := handleTrojanPort("fuck")
	assert.Error(t, e, "non-numerical port should error")
}

func TestHandleTrojanPort_GoodNumber(t *testing.T) {
	testCases := []string{"443", "8080", "10086", "80", "65535", "1"}
	for _, testCase := range testCases {
		_, e := handleTrojanPort(testCase)
		assert.Nil(t, e, "good port %s should not error", testCase)
	}
}

func TestHandleTrojanPort_InvalidNumber(t *testing.T) {
	testCases := []string{"443.0", "443.000", "8e2", "3.5", "9.99", "-1", "-65535", "65536", "0"}

	for _, testCase := range testCases {
		_, e := handleTrojanPort(testCase)
		assert.Error(t, e, "invalid number port %s should error", testCase)
	}
}

```

The provided code appears to be a ShareInfo类 testing implementation for the Twitter platform. It defines a test function `testTrojanShareInfo` which takes an array of test cases and loops through each test case. For each test case, it uses the `NewShareInfoFromURL` function to create a `ShareInfo` object from the URL in the test case. It then asserts that an error occurs when processing the ShareInfo object, and finally it prints out the test case.

The test case passed to `testTrojanShareInfo` is:
sql
trojan://what.ever@www.twitter.com:443?allowInsecure=1&allowInsecureHostname=1&allowInsecureCertificate=1&sessionTicket=0&tfo=1#some-trojan

This appears to be a valid Twitter share link. The function `testTrojanShareInfo` should be able to correctly process this link and print out the test case without any errors.


```go
func TestNewShareInfoFromURL_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("")
	assert.Error(t, e, "empty link should lead to error")
}

func TestNewShareInfoFromURL_RandomCrap(t *testing.T) {
	for i := 0; i < 100; i++ {
		randomCrap, _ := ioutil.ReadAll(io.LimitReader(crand.Reader, 10))
		_, e := NewShareInfoFromURL(string(randomCrap))
		assert.Error(t, e, "random crap %v should lead to error", randomCrap)
	}
}

func TestNewShareInfoFromURL_NotTrojanGo(t *testing.T) {
	testCases := []string{
		"trojan://what.ever@www.twitter.com:443?allowInsecure=1&allowInsecureHostname=1&allowInsecureCertificate=1&sessionTicket=0&tfo=1#some-trojan",
		"ssr://d3d3LnR3aXR0ZXIuY29tOjgwOmF1dGhfc2hhMV92NDpjaGFjaGEyMDpwbGFpbjpZbkpsWVd0M1lXeHMvP29iZnNwYXJhbT0mcmVtYXJrcz02TC1INXB5ZjVwZTI2WmUwNzd5YU1qQXlNQzB3TnkweE9DQXhNam8xTlRveU1RJmdyb3VwPVEzUkRiRzkxWkNCVFUxSQ",
		"vmess://eyJhZGQiOiJtb3RoZXIuZnVja2VyIiwiYWlkIjowLCJpZCI6IjFmYzI0NzVmLThmNDMtM2FlYi05MzUyLTU2MTFhZjg1NmQyOSIsIm5ldCI6InRjcCIsInBvcnQiOjEwMDg2LCJwcyI6Iui/h+acn+aXtumXtO+8mjIwMjAtMDYtMjMiLCJ0bHMiOiJub25lIiwidHlwZSI6Im5vbmUiLCJ2IjoyfQ==",
	}

	for _, testCase := range testCases {
		_, e := NewShareInfoFromURL(testCase)
		assert.Error(t, e, "non trojan-go link %s should not decode", testCase)
	}
}

```

这两段代码是在测试一个名为 `TestNewShareInfoFromURL_EmptyTrojanHost` 的函数。函数的作用是测试 `NewShareInfoFromURL` 函数，当它接受一个空字符串作为参数时，它是否能够正确地将信息解析回正常的字符串。

具体来说，这两段代码实现了以下功能：

1. 定义了一个名为 `func TestNewShareInfoFromURL_EmptyTrojanHost` 的函数；
2. 定义了一个名为 `func TestNewShareInfoFromURL_BadPassword` 的函数；
3. 对于这两个函数，它们都接受一个字符串参数 `testCases`，其中包含多个测试用例；
4. 对于每个测试用例，它们都会使用 `NewShareInfoFromURL` 函数来获取一个字符串，并将它们传递给 `assert.Error` 函数；
5. 如果 `assert.Error` 函数返回一个空字符串，那么说明 `NewShareInfoFromURL` 函数能够正确地解析空字符串，否则就会抛出一个错误。

因此，这两个函数分别测试了 `func TestNewShareInfoFromURL_EmptyTrojanHost` 和 `func TestNewShareInfoFromURL_BadPassword` 函数的行为，以验证它们是否按照预期工作。


```go
func TestNewShareInfoFromURL_EmptyTrojanHost(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://fuckyou@:443/")
	assert.Error(t, e, "empty host should not decode")
}

func TestNewShareInfoFromURL_BadPassword(t *testing.T) {
	testCases := []string{
		"trojan-go://we:are:the:champion@114514.go",
		"trojan-go://evilpassword:@1919810.me",
		"trojan-go://evilpassword::@1919810.me",
		"trojan-go://@password.404",
		"trojan-go://mother.fuck#yeah",
	}

	for _, testCase := range testCases {
		_, e := NewShareInfoFromURL(testCase)
		assert.Error(t, e, "bad password link %s should not decode", testCase)
	}
}

```

该代码测试 `NewShareInfoFromURL` 函数，用于从 URL 获取信息并验证其是否正确。

函数内部首先定义了一个字符串数组 `testCases`，其中包含了一些测试用例。

接着，在主函数中使用一个循环遍历 `testCases`，为每个测试用例调用 `NewShareInfoFromURL` 函数，然后使用 `assert` 函数验证函数返回值是否为 `nil`，即是否出现了错误。

如果返回值不是 `nil`，说明函数执行成功，则执行循环体内的代码，否则输出错误信息。


```go
func TestNewShareInfoFromURL_GoodPassword(t *testing.T) {
	testCases := []string{
		"trojan-go://we%3Aare%3Athe%3Achampion@114514.go",
		"trojan-go://evilpassword%3A@1919810.me",
		"trojan-go://passw0rd-is-a-must@password.200",
	}

	for _, testCase := range testCases {
		_, e := NewShareInfoFromURL(testCase)
		assert.Nil(t, e, "good password link %s should decode", testCase)
	}
}

func TestNewShareInfoFromURL_BadPort(t *testing.T) {
	testCases := []string{
		"trojan-go://pswd@example.com:114514",
		"trojan-go://pswd@example.com:443.0",
		"trojan-go://pswd@example.com:-1",
		"trojan-go://pswd@example.com:8e2",
		"trojan-go://pswd@example.com:65536",
	}

	for _, testCase := range testCases {
		_, e := NewShareInfoFromURL(testCase)
		assert.Error(t, e, "decode url %s with invalid port should error", testCase)
	}
}

```

这段代码是一个 Go 语言中的测试函数，名为 `TestNewShareInfoFromURL_BadQuery`。函数的作用是测试 `NewShareInfoFromURL` 函数在处理 bad query 时是否会产生错误。

函数内部定义了一个字符串数组 `testCases`，其中包含了两个测试用例。

第一个测试用例是 `trojan-go://cao@ni.ma?NMSL=%CG%GE%CAONIMA`，第二个测试用例是 `trojan-go://ni@ta.ma:13/?#%2e%fu`。这两个测试用例都会通过 `NewShareInfoFromURL` 函数，然后会通过 `assert.Error` 函数输出一个错误信息。

第一个测试用例的错误信息将会打印出来，例如：

Error: parse bad query should error

第二个测试用例的错误信息将会留空，因为 `assert.Error` 函数并不会为空错误信息。

需要注意的是，`trojan-go://` 和 `ni.ma?NMSL=` 是网址，而 `?NMSL=` 中的 `?NMSL=` 参数并不是 URL 的一部分，所以当 `trojan-go://` 和 `ni.ma?NMSL=` 作为网址使用时，`testCases` 数组中的第二个测试用例不会产生错误信息。


```go
func TestNewShareInfoFromURL_BadQuery(t *testing.T) {
	testCases := []string{
		"trojan-go://cao@ni.ma?NMSL=%CG%GE%CAONIMA",
		"trojan-go://ni@ta.ma:13/?#%2e%fu",
	}

	for _, testCase := range testCases {
		_, e := NewShareInfoFromURL(testCase)
		assert.Error(t, e, "parse bad query should error")
	}
}

func TestNewShareInfoFromURL_SNI_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?sni=")
	assert.Error(t, e, "empty SNI should not be allowed")
}

```

这是一段 Go 语言测试代码，旨在测试 `NewShareInfoFromURL` 函数。通过调用该函数，我们可以测试它是否能够正确地从 URL 中解析出相关信息，包括 trojan hostname 和 SNI。

具体来说，这段代码分为三个测试用例。第一个测试用例 `TestNewShareInfoFromURL_SNI_Default` 旨在测试 `NewShareInfoFromURL` 函数在只传递了一个 trojan hostname 时，是否能够正确地返回信息。第二个测试用例 `TestNewShareInfoFromURL_SNI_Multiple` 旨在测试 `NewShareInfoFromURL` 函数在传递多个 SNI 时，是否能够正确地返回信息。第三个测试用例 `TestNewShareInfoFromURL_Type_Empty` 旨在测试 `NewShareInfoFromURL` 函数在传递一个空类型时，是否能够正确地返回信息。


```go
func TestNewShareInfoFromURL_SNI_Default(t *testing.T) {
	info, e := NewShareInfoFromURL("trojan-go://a@b.c")
	assert.Nil(t, e)
	assert.Equal(t, info.TrojanHost, info.SNI, "default sni should be trojan hostname")
}

func TestNewShareInfoFromURL_SNI_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?sni=a&sni=b&sni=c")
	assert.Error(t, e, "multiple SNIs should not be allowed")
}

func TestNewShareInfoFromURL_Type_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=")
	assert.Error(t, e, "empty type should not be allowed")
}

```

这篇代码是一个 Go 语言测试中的函数，用于测试 `NewShareInfoFromURL` 函数在不同输入参数的情况下是否正确。

首先，我们需要为测试定义一个函数 `TestNewShareInfoFromURL_Type_Default`，它接收一个指向 `testing.T` 类型的变量 `t`，作为测试框架的输入。

接下来，我们定义另一个函数 `TestNewShareInfoFromURL_Type_Invalid`，它接收一个字符串数组 `invalidTypes`，作为测试框架的输入。

接着，我们定义一个名为 `TestNewShareInfoFromURL_Type_Multiple` 的函数，但这个函数没有具体的实现，因为它只是用来演示 `TestNewShareInfoFromURL_Type_Invalid` 函数的多个输入参数情况。

现在，我们可以编写测试用例来测试 `NewShareInfoFromURL` 函数：

go
package main

import (
	"testing"
)

func TestNewShareInfoFromURL_Type_Default(t *testing.T) {
	info, e := NewShareInfoFromURL("trojan-go://a@b.c")
	assert.Nil(t, e)
	assert.Equal(t, ShareInfoTypeOriginal, info.Type, "default type should be original")
}

func TestNewShareInfoFromURL_Type_Invalid(t *testing.T) {
	invalidTypes := []string{"nmsl", "dio"}
	for _, invalidType := range invalidTypes {
		_, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?type=%s", invalidType))
		assert.Error(t, e, "%s should not be a valid type", invalidType)
	}
}

func TestNewShareInfoFromURL_Type_Multiple(t *testing.T) {
	assert.Error(t, NewShareInfoFromURL("trojan-go://a@b.c?type=a&type=b&type=c"), "multiple types should not be allowed")
}


首先，我们为每个测试函数提供了一个测试用例。例如，对于 `TestNewShareInfoFromURL_Type_Default` 函数，我们测试了将 `"trojan-go://a@b.c"` 作为输入参数时，函数是否能够正常工作，并检查返回的 `ShareInfo` 对象是否具有 `ShareInfoTypeOriginal` 类型。

对于 `TestNewShareInfoFromURL_Type_Invalid` 函数，我们测试了几个无效输入参数，如 `"nmsl"` 和 `"dio"`。我们期望函数在尝试解析无效输入参数时能够抛出异常，并检查异常的详细信息。

对于 `TestNewShareInfoFromURL_Type_Multiple` 函数，我们测试了在一个 URL 中使用多个无效输入参数的情况。我们期望函数能够抛出异常，并检查异常的详细信息。


```go
func TestNewShareInfoFromURL_Type_Default(t *testing.T) {
	info, e := NewShareInfoFromURL("trojan-go://a@b.c")
	assert.Nil(t, e)
	assert.Equal(t, ShareInfoTypeOriginal, info.Type, "default type should be original")
}

func TestNewShareInfoFromURL_Type_Invalid(t *testing.T) {
	invalidTypes := []string{"nmsl", "dio"}
	for _, invalidType := range invalidTypes {
		_, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?type=%s", invalidType))
		assert.Error(t, e, "%s should not be a valid type", invalidType)
	}
}

func TestNewShareInfoFromURL_Type_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=a&type=b&type=c")
	assert.Error(t, e, "multiple types should not be allowed")
}

```

这段代码是对“trojan-go://a@b.c”这个URL进行了测试，主要测试以下三个场景：

1. 空格字符不存在的场景。
2. 换行符“\n”不存在的场景。
3. 多个空格字符和不同空格字符的组合。

`assert.Error`函数是断言测试中返回的错误信息，如果测试失败，则会输出错误信息，否则不会输出任何信息。


```go
func TestNewShareInfoFromURL_Host_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?host=")
	assert.Error(t, e, "empty host should not be allowed")
}

func TestNewShareInfoFromURL_Host_Default(t *testing.T) {
	info, e := NewShareInfoFromURL("trojan-go://a@b.c")
	assert.Nil(t, e)
	assert.Equal(t, info.TrojanHost, info.Host, "default host should be trojan hostname")
}

func TestNewShareInfoFromURL_Host_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?host=a&host=b&host=c")
	assert.Error(t, e, "multiple hosts should not be allowed")
}

```

这段代码对`NewShareInfoFromURL`函数进行了三个测试用例。具体解释如下：

1. `TestNewShareInfoFromURL_Type_WS_Multiple`测试了输入参数中的"multiple paths"是否正确。函数中的`NewShareInfoFromURL`函数会尝试使用"trojan-go://a@b.c?type=ws&path=a&path=b&path=c"这样的输入参数，但是这将允许通过。该测试用例会验证函数在允许的情况下，抛出一个异常，以验证函数是否正确地处理了包含多个路径的输入参数。
2. `TestNewShareInfoFromURL_Path_WS_None`测试了输入参数中的"ws"是否正确。函数中的`NewShareInfoFromURL`函数会尝试使用"trojan-go://a@b.c?type=ws"这样的输入参数，但是这将允许通过。该测试用例会验证函数在允许的情况下，抛出一个异常，以验证函数是否正确地处理了只包含一个路径的输入参数。
3. `TestNewShareInfoFromURL_Path_WS_Empty`测试了输入参数中的"ws"是否正确。函数中的`NewShareInfoFromURL`函数会尝试使用"trojan-go://a@b.c?type=ws&path="这样的输入参数，但是这将允许通过。该测试用例会验证函数在允许的情况下，抛出一个异常，以验证函数是否正确地处理了一个空路径的输入参数。


```go
func TestNewShareInfoFromURL_Type_WS_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws&path=a&path=b&path=c")
	assert.Error(t, e, "multiple paths should not be allowed in wss")
}

func TestNewShareInfoFromURL_Path_WS_None(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws")
	assert.Error(t, e, "ws should require path")
}

func TestNewShareInfoFromURL_Path_WS_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=ws&path=")
	assert.Error(t, e, "empty path should not be allowed in ws")
}

```

这段代码是一个测试用例，主要是针对函数 `NewShareInfoFromURL` 的作用进行测试。具体内容如下：

1. 首先定义了一个字符串数组 `invalidPaths`，包含三个路径，分别是以 `../`、`.+!` 和空格结尾的路径。

2. 以 `for` 循环结构 `for _, invalidPath := range invalidPaths` 遍历 `invalidPaths` 数组中的每一个元素，并将其存储到 `e` 变量中。

3. 对于循环结构中的每个元素，调用 `NewShareInfoFromURL` 函数，并使用 `fmt.Sprintf` 函数将传入的路径作为参数，构建一个请求的 URL。然后将构建好的 URL 参数作为参数，调用 `NewShareInfoFromURL` 函数，并将结果存储到 `e` 变量中。

4. 分别对循环结构中的每个元素进行断言，即检查返回值是否为 `nil`，如果不是，说明函数能够正确处理无效路径，并输出一条相应的错误信息。

5. 最后一个测试用例 `TestNewShareInfoFromURL_Path_Plain_Empty` 对 `NewShareInfoFromURL` 函数的另一个参数 `path` 进行测试，测试其是否能够正确处理路径为空的情况。如果返回值 `e` 为 `nil`，说明函数能够正确处理路径为空的情况，不会产生任何错误。


```go
func TestNewShareInfoFromURL_Path_WS_Invalid(t *testing.T) {
	invalidPaths := []string{"../", ".+!", " "}
	for _, invalidPath := range invalidPaths {
		_, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?type=ws&path=%s", invalidPath))
		assert.Error(t, e, "%s should not be a valid path in ws", invalidPath)
	}
}

func TestNewShareInfoFromURL_Path_Plain_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?type=original&path=")
	assert.Nil(t, e, "empty path should be ignored in original mode")
}

func TestNewShareInfoFromURL_Encryption_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=")
	assert.Error(t, e, "encryption should not be empty")
}

```

这段代码在测试中验证了三种不同的加密模式：SS-OAEP、RC4 和 RSA。它通过调用 NewShareInfoFromURL 函数，传入不同的 URL，来测试服务器是否可以正确地支持这些加密模式。

具体来说，这段代码主要关注以下几点：

1.测试函数 TestNewShareInfoFromURL_Encryption_Unknown，验证服务器是否支持 "unknown" 加密模式。
2.测试函数 TestNewShareInfoFromURL_Encryption_None，验证服务器是否支持 "none" 加密模式。
3.测试函数 TestNewShareInfoFromURL_Encryption_SS_NotSupportedMethods，验证服务器是否支持 SS-OAEP、RC4 和 RSA 等不支持的加密模式。

如果服务器成功支持这些加密模式，测试代码将会输出 "ok"。如果不能支持，则会输出错误信息。


```go
func TestNewShareInfoFromURL_Encryption_Unknown(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=motherfucker")
	assert.Error(t, e, "unknown encryption should not be supported")
}

func TestNewShareInfoFromURL_Encryption_None(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://what@ever.me?encryption=none")
	assert.Nil(t, e, "should support none encryption")
}

func TestNewShareInfoFromURL_Encryption_SS_NotSupportedMethods(t *testing.T) {
	invalidMethods := []string{"rc4-md5", "rc4", "des-cfb", "table", "salsa20-ctr"}
	for _, invalidMethod := range invalidMethods {
		_, e := NewShareInfoFromURL(fmt.Sprintf("trojan-go://a@b.c?encryption=ss%%3B%s%%3Ashabi", invalidMethod))
		assert.Error(t, e, "encryption %s should not be supported by ss", invalidMethod)
	}
}

```

该代码是一个 Go 语言中的函数测试类，名为 `TestNewShareInfoFromURL_Encryption_SS_NoPassword`、`TestNewShareInfoFromURL_Encryption_SS_BadParams` 和 `TestNewShareInfoFromURL_Encryption_Multiple`。它们用于对 `NewShareInfoFromURL` 函数进行测试，以验证在不同的参数输入下，函数的行为是否符合预期。

具体来说，每个测试用例都会创建一个空的 `testing.T` 环境，并使用不同的输入参数来调用 `NewShareInfoFromURL` 函数。如果函数能够正确处理所有输入参数，测试将会通过，否则将会抛出错误并指出函数的缺陷。

例如，在第一个测试用例中，测试用语 `func TestNewShareInfoFromURL_Encryption_SS_NoPassword(t *testing.T)` 将创建一个测试函数 `TestNewShareInfoFromURL_Encryption_SS_NoPassword`，并使用 `t.Run` 函数为其命名，参数指示使用 `testing.T` 环境。这个测试函数将尝试使用 `trojan-go://a@b.c?encryption=ss%3Baes-256-gcm%3A` 作为输入参数调用 `NewShareInfoFromURL` 函数，然后验证函数是否能够正确处理这个输入参数。如果函数能够正确处理，测试将会通过，否则将会抛出错误并指出函数的缺陷。

同样地，在第二个测试用例中，测试用语 `func TestNewShareInfoFromURL_Encryption_SS_BadParams(t *testing.T)` 将创建一个测试函数 `TestNewShareInfoFromURL_Encryption_SS_BadParams`，并使用 `t.Run` 函数为其命名，参数指示使用 `testing.T` 环境。这个测试函数将尝试使用 `trojan-go://a@b.c?encryption=ss%3Ba` 作为输入参数调用 `NewShareInfoFromURL` 函数，然后验证函数是否能够正确处理这个输入参数。如果函数能够正确处理，测试将会通过，否则将会抛出错误并指出函数的缺陷。

在第三个测试用例中，测试用语 `func TestNewShareInfoFromURL_Encryption_Multiple(t *testing.T)` 将创建一个测试函数 `TestNewShareInfoFromURL_Encryption_Multiple`，并使用 `t.Run` 函数为其命名，参数指示使用 `testing.T` 环境。这个测试函数将尝试使用 `trojan-go://a@b.c?encryption=a&encryption=b&encryption=c` 作为输入参数调用 `NewShareInfoFromURL` 函数，然后验证函数是否能够正确处理这个输入参数。如果函数能够正确处理，测试将会通过，否则将会抛出错误并指出函数的缺陷。


```go
func TestNewShareInfoFromURL_Encryption_SS_NoPassword(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=ss%3Baes-256-gcm%3A")
	assert.Error(t, e, "empty ss password should not be allowed")
}

func TestNewShareInfoFromURL_Encryption_SS_BadParams(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=ss%3Ba")
	assert.Error(t, e, "broken ss param should not be allowed")
}

func TestNewShareInfoFromURL_Encryption_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?encryption=a&encryption=b&encryption=c")
	assert.Error(t, e, "multiple encryption should not be allowed")
}

```

这两个测试函数分别测试了两种不同情况。

第一个测试函数的作用是测试 `NewShareInfoFromURL` 函数，当传入一个不包含 "plugin=" 的 URL 时，该函数会引发异常。具体来说，函数先尝试使用 "plugin=" 参数的 URL，然后使用 "?plugin=" 参数的 URL，如果这两个 URL 都无法正常返回，那么就会引发异常。这个测试是为了验证 `NewShareInfoFromURL` 函数在遇到空格或其他非法字符串时能够正确地引发异常，从而确保了函数的稳定性和可靠性。

第二个测试函数的作用是测试 `NewShareInfoFromURL` 函数，当传入一个包含多个 "plugin=" 参数的 URL 时，该函数会引发异常。具体来说，函数会检查传入的 URL 是否包含多个 "plugin=" 参数，如果是，那么就会引发异常。这个测试是为了验证 `NewShareInfoFromURL` 函数在遇到多个 "plugin=" 参数的 URL 时能够正确地引发异常，从而确保了函数的稳定性和可靠性。


```go
func TestNewShareInfoFromURL_Plugin_Empty(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?plugin=")
	assert.Error(t, e, "plugin should not be empty")
}

func TestNewShareInfoFromURL_Plugin_Multiple(t *testing.T) {
	_, e := NewShareInfoFromURL("trojan-go://a@b.c?plugin=a&plugin=b&plugin=c")
	assert.Error(t, e, "multiple plugin should not be allowed")
}

```