# trojan-go源码解析 9

# `tunnel/dokodemo/server.go`

该代码定义了一个名为`Server`的`Server`结构体，用于创建一个服务器并管理服务器连接。以下是该结构体的关键字段及其作用：

- `tunnel.Server`：表示该结构体是一个服务器，用于监听来自客户端的连接。
- `net.Listener`：表示该结构体监听来自网络的连接，返回一个`net.Listener`对象。
- `udpListener`：表示该结构体监听来自网络的 UDP 连接，返回一个`net.PacketConn`对象。
- `packetChan`：表示服务器接收到的数据数据的通道，用于将数据数据发送到客户端。
- `timeout`：表示服务器连接超时的时间，超过该时间后将关闭连接并取消已连接的套接字。
- `targetAddr`：表示服务器发送数据数据的目标地址，用于将数据数据发送到该地址。
- `mappingLock`：表示一个互斥锁，用于保护服务器正在运行的映射数据，防止多个服务器同时发送数据数据。
- `mapping`：表示一个 map，用于存储服务器正在运行的映射数据。
- `ctx`：表示一个 context，用于在函数中获取的上下文。
- `cancel`：表示一个取消函数，用于在需要时取消正在运行的套接字。

通过使用以上关键字段，该结构体可以创建一个服务器并管理服务器连接，通过网络与客户端进行通信。


```go
package dokodemo

import (
	"context"
	"net"
	"sync"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Server struct {
	tunnel.Server
	tcpListener net.Listener
	udpListener net.PacketConn
	packetChan  chan tunnel.PacketConn
	timeout     time.Duration
	targetAddr  *tunnel.Address
	mappingLock sync.Mutex
	mapping     map[string]*PacketConn
	ctx         context.Context
	cancel      context.CancelFunc
}

```

这是一段 Go 语言代码，主要作用是处理 UDP 数据报连接。首先，根据给出的 IP 地址，判断数据报连接是否存在，并获取到连接对象。接着，如果连接对象处理完数据报连接，尝试从连接中读取数据报，并将数据存储到缓冲区中。然后，使用 Go 语言中的 `PacketConn` 类型创建一个新的数据报连接对象，并设置其输入和输出数据报缓冲区以及相关设置。最后，使用 Go 语言中的 `WithCancel` 函数取消连接操作，如果连接操作超时，则释放连接资源并关闭连接。


```go
func (s *Server) dispatchLoop() {
	fixedMetadata := &tunnel.Metadata{
		Address: s.targetAddr,
	}
	for {
		buf := make([]byte, MaxPacketSize)
		n, addr, err := s.udpListener.ReadFrom(buf)
		if err != nil {
			select {
			case <-s.ctx.Done():
			default:
				log.Fatal(common.NewError("dokodemo failed to read from udp socket").Base(err))
			}
			return
		}
		log.Debug("udp packet from", addr)
		s.mappingLock.Lock()
		if conn, found := s.mapping[addr.String()]; found {
			conn.input <- buf[:n]
			s.mappingLock.Unlock()
			continue
		}
		ctx, cancel := context.WithCancel(s.ctx)
		conn := &PacketConn{
			input:      make(chan []byte, 16),
			output:     make(chan []byte, 16),
			metadata:   fixedMetadata,
			src:        addr,
			PacketConn: s.udpListener,
			ctx:        ctx,
			cancel:     cancel,
		}
		s.mapping[addr.String()] = conn
		s.mappingLock.Unlock()

		conn.input <- buf[:n]
		s.packetChan <- conn

		go func(conn *PacketConn) {
			for {
				select {
				case payload := <-conn.output:
					// "Multiple goroutines may invoke methods on a Conn simultaneously."
					_, err := s.udpListener.WriteTo(payload, conn.src)
					if err != nil {
						log.Error(common.NewError("dokodemo udp write error").Base(err))
						return
					}
				case <-s.ctx.Done():
					return
				case <-time.After(s.timeout):
					s.mappingLock.Lock()
					delete(s.mapping, conn.src.String())
					s.mappingLock.Unlock()
					conn.Close()
					log.Debug("closing timeout packetConn")
					return
				}
			}
		}(conn)
	}
}

```

这两函数的作用是描述一个 TCP 服务器如何监听和接受连接以及处理接收到的数据包。

具体来说，`AcceptConn` 函数接收一个 `tunnel.Tunnel` 类型的参数，表示一个通过 TCP 连接到服务器的连接。函数首先尝试调用服务器 `tcpListener` 的 `Accept` 方法来获取新的连接，如果失败，则记录一个错误并返回。如果成功，则返回新连接的信息，包括连接的 `conn` 和目标元数据（例如 IP 地址和端口号）。

`AcceptPacket` 函数则接收一个 `tunnel.Tunnel` 类型的参数，表示一个通过 TCP 连接到服务器的数据包。函数会等待服务器接收到新的数据包，然后采取相应的行动。如果连接的 `conn` 参数不是一个有效的连接，函数将返回一个错误。如果服务器接收到数据包，函数返回一个指向 `conn` 的引用，表示这是一个有效的连接，同时还返回一个 `tunnel.PacketConn` 类型的数据，表示这是一个数据包连接。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := s.tcpListener.Accept()
	if err != nil {
		log.Fatal(common.NewError("dokodemo failed to accept connection").Base(err))
	}
	return &Conn{
		Conn: conn,
		targetMetadata: &tunnel.Metadata{
			Address: s.targetAddr,
		},
	}, nil
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	select {
	case conn := <-s.packetChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("dokodemo server closed")
	}
}

```

该代码是一个 Go 语言中的函数，定义了一个名为 "func" 的函数接收一个名为 "Server" 的结构体参数，并返回一个名为 "error" 的类型。

具体来说，函数的主要作用是关闭服务器，包括 TCP 和 UDP 监听器。创建服务器时，首先根据配置从 "config.FromContext" 函数中获取服务器配置，然后根据服务器配置创建一个 "Server" 实例，并设置服务器的目标地址和本地地址。接着分别创建 TCP 和 UDP 监听器，并设置好相关参数。最后在函数内部创建一个 "ctx" 上下文变量和一个 "cancel" 函数，用来在服务器关闭时执行一些清理操作，并进入一个循环，等待关闭服务器。

另外，该函数还定义了一个名为 "NewServer" 的函数，该函数接收一个 "Context" 类型的参数，根据该参数创建一个 "Server" 实例，并返回它。该函数使用了 "config.FromContext" 和 "tcp聆听器关闭" 和 "udp 聆听器关闭" 函数，这些函数没有在提供的上下文中定义，因此它们的实现不在本次解释的范围内。


```go
func (s *Server) Close() error {
	s.cancel()
	s.tcpListener.Close()
	s.udpListener.Close()
	return nil
}

func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	targetAddr := tunnel.NewAddressFromHostPort("tcp", cfg.TargetHost, cfg.TargetPort)
	listenAddr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)

	tcpListener, err := net.Listen("tcp", listenAddr.String())
	if err != nil {
		return nil, common.NewError("failed to listen tcp").Base(err)
	}
	udpListener, err := net.ListenPacket("udp", listenAddr.String())
	if err != nil {
		return nil, common.NewError("failed to listen udp").Base(err)
	}

	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		tcpListener: tcpListener,
		udpListener: udpListener,
		targetAddr:  targetAddr,
		mapping:     make(map[string]*PacketConn),
		packetChan:  make(chan tunnel.PacketConn, 32),
		timeout:     time.Second * time.Duration(cfg.UDPTimeout),
		ctx:         ctx,
		cancel:      cancel,
	}
	go server.dispatchLoop()
	return server, nil
}

```

# `tunnel/dokodemo/tunnel.go`

该代码定义了一个名为 "DOKODEMO" 的包，它从名为 "github.com/p4gefau1t/trojan-go" 的公共库中导入了一个名为 "trojan-go" 的包。然后，该包中的一个名为 "Tunnel" 的类型定义了一个 "Tunnel" 结构体，该结构体具有一个名为 "tunnel.Tunnel" 的成员 "隧道"。此外，该类型还具有一个名为 "Name" 的成员 "名称"，它的值为 "DOKODEMO"。


```go
package dokodemo

import (
	"context"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "DOKODEMO"

type Tunnel struct{ tunnel.Tunnel }

func (*Tunnel) Name() string {
	return Name
}

```

这是一组使用Go语言编写的函数，定义了Tunnel类的创建服务器和客户端的接口。

在函数(*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error)中，使用了名为NewServer的函数，它接收一个带有上下文上下文的当前上下文和一个underlay类型的隧道服务器作为参数。函数返回一个新的隧道服务器和一个表示错误的错误对象。

在函数(*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error)中，同样使用了名为NewClient的函数，它接收一个带有上下文上下文的当前上下文和一个underlay类型的隧道客户端作为参数。函数返回一个表示错误的错误对象。

在函数初始化代码中，使用了名为Tunnel的类，它实现了接口tunnel.Server和tunnel.Client。函数初始化了一些函数注册器，其中包括名为Tunnel的函数注册为tunnel.Server类型。


```go
func (*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, underlay)
}

func (*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error) {
	return nil, common.NewError("not supported")
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/freedom/client.go`

这段代码定义了一个名为"freedom"的包，其中包括以下组件：

1. 导入了一些外部库：net、proxy、socks5、common、config和trojan-go。

2. 定义了一个名为"Client"的客户端结构体，其中包含以下字段：

	- preferIPv4: 是否更喜欢使用IPv4地址。
	- noDelay: 是否在发送数据时忽略延迟。
	- keepAlive: 是否保持TCP连接状态。
	- ctx: 当前上下文。
	- cancel: 取消某个网络请求的函数。
	- forwardProxy: 是否使用代理服务器。
	- proxyAddr: 如果使用代理服务器，则代理服务器的地址。
	- username: 用户名。
	- password: 密码。

3. 在"Client"结构体的初始化函数中，设置了一些默认值。

4. 定义了一个名为"connect"的函数，用于建立TCP连接并返回一个句柄。

5. 定义了一个名为"send"的函数，用于发送数据到指定的代理服务器或TCP连接。

6. 定义了一个名为"cancel"的函数，用于取消之前设置的连接或代理服务器。

7. 最后，定义了一个名为"main"的函数，用于获取命令行参数并使用"connect"函数建立TCP连接。

整个 package 的作用是提供一个用于建立代理服务器或TCP连接的简单客户端。


```go
package freedom

import (
	"context"
	"net"

	"github.com/txthinking/socks5"
	"golang.org/x/net/proxy"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Client struct {
	preferIPv4   bool
	noDelay      bool
	keepAlive    bool
	ctx          context.Context
	cancel       context.CancelFunc
	forwardProxy bool
	proxyAddr    *tunnel.Address
	username     string
	password     string
}

```

该函数名为`DialConn`，定义了一个名为`Client`的`c`参数和一个名为`tunnel`的`tunnel`参数。函数的作用是尝试通过代理和直接连接来建立与目标连接器的连接，并返回一个`tunnel.Conn`类型的结果。

具体来说，函数的实现分为以下几步：

1. 如果`c`参数中的`forwardProxy`为`true`，则表示使用代理建立连接。
2. 如果`c`参数中的`username`不为空，则设置用户名和密码作为代理的用户信息。
3. 如果`c`参数中的`proxyAddr`为`nil`，则表示不使用代理，直接使用直接连接。
4. 如果使用代理建立连接，使用代理的`SOCKS5`函数建立与目标服务器之间的连接，并尝试通过该连接进行数据传输。
5. 如果使用直接连接建立连接，使用`net.Dialer`类尝试根据指定的网络类型和目标地址建立一个TCP连接，并设置一些选项如`keepAlive`和`noDelay`。
6. 最后，返回一个`tunnel.Conn`类型的结果，该结果包含与目标连接器的连接信息。

如果函数在执行过程中遇到错误，则会返回一个相应的错误信息，并将其作为第一个参数传递给函数的`base`函数。


```go
func (c *Client) DialConn(addr *tunnel.Address, _ tunnel.Tunnel) (tunnel.Conn, error) {
	// forward proxy
	if c.forwardProxy {
		var auth *proxy.Auth
		if c.username != "" {
			auth = &proxy.Auth{
				User:     c.username,
				Password: c.password,
			}
		}
		dialer, err := proxy.SOCKS5("tcp", c.proxyAddr.String(), auth, proxy.Direct)
		if err != nil {
			return nil, common.NewError("freedom failed to init socks dialer")
		}
		conn, err := dialer.Dial("tcp", addr.String())
		if err != nil {
			return nil, common.NewError("freedom failed to dial target address via socks proxy " + addr.String()).Base(err)
		}
		return &Conn{
			Conn: conn,
		}, nil
	}
	network := "tcp"
	if c.preferIPv4 {
		network = "tcp4"
	}
	dialer := new(net.Dialer)
	tcpConn, err := dialer.DialContext(c.ctx, network, addr.String())
	if err != nil {
		return nil, common.NewError("freedom failed to dial " + addr.String()).Base(err)
	}

	tcpConn.(*net.TCPConn).SetKeepAlive(c.keepAlive)
	tcpConn.(*net.TCPConn).SetNoDelay(c.noDelay)
	return &Conn{
		Conn: tcpConn,
	}, nil
}

```

This appears to be a function that creates a TCP or UDP connection to a SOCKS server. It takes a connection address, port, and an optional username and password. It uses the `freedom` library to establish the connection and returns a `PacketConn` object if the connection was successful. If an error occurs, it returns a `common.Error` message.


```go
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	if c.forwardProxy {
		socksClient, err := socks5.NewClient(c.proxyAddr.String(), c.username, c.password, 0, 0)
		common.Must(err)
		if err := socksClient.Negotiate(&net.TCPAddr{}); err != nil {
			return nil, common.NewError("freedom failed to negotiate socks").Base(err)
		}
		a, addr, port, err := socks5.ParseAddress("1.1.1.1:53") // useless address
		common.Must(err)
		resp, err := socksClient.Request(socks5.NewRequest(socks5.CmdUDP, a, addr, port))
		if err != nil {
			return nil, common.NewError("freedom failed to dial udp to socks").Base(err)
		}
		// TODO fix hardcoded localhost
		packetConn, err := net.ListenPacket("udp", "127.0.0.1:0")
		if err != nil {
			return nil, common.NewError("freedom failed to listen udp").Base(err)
		}
		socksAddr, err := net.ResolveUDPAddr("udp", resp.Address())
		if err != nil {
			return nil, common.NewError("freedom recv invalid socks bind addr").Base(err)
		}
		return &SocksPacketConn{
			PacketConn:  packetConn,
			socksAddr:   socksAddr,
			socksClient: socksClient,
		}, nil
	}
	network := "udp"
	if c.preferIPv4 {
		network = "udp4"
	}
	udpConn, err := net.ListenPacket(network, "")
	if err != nil {
		return nil, common.NewError("freedom failed to listen udp socket").Base(err)
	}
	return &PacketConn{
		UDPConn: udpConn.(*net.UDPConn),
	}, nil
}

```

该函数名为 `func (c *Client) Close() error`，表示该函数的作用是关闭名为 `c` 的 `Client` 对象。

具体来说，该函数执行以下操作：

1. 调用 `c.cancel()`，取消与客户端的连接。
2. 返回 `nil`，表示成功关闭了客户端连接，没有错误发生。

该函数与 `func (c *Client) Connect() error` 类似，但是两者的作用域不同，因此可以单独调用该函数，而不必担心与另一个函数同时存在时可能会出现的命名冲突问题。


```go
func (c *Client) Close() error {
	c.cancel()
	return nil
}

func NewClient(ctx context.Context, _ tunnel.Client) (*Client, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	addr := tunnel.NewAddressFromHostPort("tcp", cfg.ForwardProxy.ProxyHost, cfg.ForwardProxy.ProxyPort)
	ctx, cancel := context.WithCancel(ctx)
	return &Client{
		ctx:          ctx,
		cancel:       cancel,
		noDelay:      cfg.TCP.NoDelay,
		keepAlive:    cfg.TCP.KeepAlive,
		preferIPv4:   cfg.TCP.PreferIPV4,
		forwardProxy: cfg.ForwardProxy.Enabled,
		proxyAddr:    addr,
		username:     cfg.ForwardProxy.Username,
		password:     cfg.ForwardProxy.Password,
	}, nil
}

```

# `tunnel/freedom/config.go`

该代码是一个Go语言的包，名为“freedom”。

它导入了自定义配置结构体“Config”以及一个名为“TCPConfig”的子类型。

“Config”结构体定义了服务器的本地地址、端口号、是否启用TCP和是否启用反向代理。

“TCPConfig”实现了“Config”结构体中“TCP”字段的定义。它定义了是否启用IPv4，是否启用保持活塞，以及是否启用延迟。

通过将“TCPConfig”和“Config”结构体一起使用，可以创建一个完整的Go服务器配置。


```go
package freedom

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
	LocalHost    string             `json:"local_addr" yaml:"local-addr"`
	LocalPort    int                `json:"local_port" yaml:"local-port"`
	TCP          TCPConfig          `json:"tcp" yaml:"tcp"`
	ForwardProxy ForwardProxyConfig `json:"forward_proxy" yaml:"forward-proxy"`
}

type TCPConfig struct {
	PreferIPV4 bool `json:"prefer_ipv4" yaml:"prefer-ipv4"`
	KeepAlive  bool `json:"keep_alive" yaml:"keep-alive"`
	NoDelay    bool `json:"no_delay" yaml:"no-delay"`
}

```

这段代码定义了一个名为 "ForwardProxyConfig" 的结构体，它用于配置代理服务。这个结构体包含以下字段：

- "Enabled"：布尔值，表示是否启用代理服务。
- "ProxyHost"：代理服务器的 URL。
- "ProxyPort"：代理服务器的端口号。
- "Username"：远程用户的用户名。
- "Password"：远程用户的密码。

在代码的 "init" 函数中，这个结构体被注册为代理服务器的配置创建器。这意味着每次实例化代理服务器时，都可以通过调用这个函数来设置这些字段的值。


```go
type ForwardProxyConfig struct {
	Enabled   bool   `json:"enabled" yaml:"enabled"`
	ProxyHost string `json:"proxy_addr" yaml:"proxy-addr"`
	ProxyPort int    `json:"proxy_port" yaml:"proxy-port"`
	Username  string `json:"username" yaml:"username"`
	Password  string `json:"password" yaml:"password"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{
			TCP: TCPConfig{
				PreferIPV4: false,
				NoDelay:    true,
				KeepAlive:  true,
			},
		}
	})
}

```

# `tunnel/freedom/conn.go`

该代码是一个名为"freedom"的R network库，用于创建能够通过网络连接到指定服务器的有利通道。

首先，它导入了多个外部库，包括：

- "bytes"库：字节数组操作库，用于处理数据包的字节数据。
- "net"网络库：网络通信库，用于创建套接字并建立网络连接。
- "github.com/txthinking/socks5"库：用于创建和管理SOCKS5代理的库。
- "github.com/p4gefau1t/trojan-go/common"库：通用包，用于进行跨网络通信操作。
- "github.com/p4gefau1t/trojan-go/log"库：用于记录错误和信息的库。
- "github.com/p4gefau1t/trojan-go/tunnel"库：用于在本地网络和目标服务器之间建立隧道连接的库。

接着，定义了一个名为"MaxPacketSize"的变量，用于表示每个数据包的最大大小。

最后，该代码的主要函数包括：

- "createChannel"函数：用于创建一个通道，通过该渠道可以发送数据包到指定的服务器。
- "forwardPacket"函数：用于将数据包转发到指定的服务器，通过指定的代理服务器。
- "sendPacket"函数：用于将数据包发送到指定服务器的函数，可以通过指定的代理服务器。
- "closeChannel"函数：用于关闭一个打开的通道。

函数的实现略。


```go
package freedom

import (
	"bytes"
	"net"

	"github.com/txthinking/socks5"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

const MaxPacketSize = 1024 * 8

```

这段代码定义了一个名为 "Conn" 的 struct 类型，该类型包含一个 "net.Conn"。这个 "net.Conn" 是一个用于与网络连接进行交互的 "net" 包中的类，它可以封装一个网络连接并提供了各种与网络相关的方法。

接下来，定义了一个名为 "PacketsConn" 的 struct 类型，该类型也包含一个 "net.UDPConn"，该类型表示一个使用 UDP 协议进行数据传输的 "net.Conn"。

最后，定义了一个名为 "Metadata" 的 struct 类型，它是一个包含 "tunnel.Metadata" 类型的 "metadata" 类型的指针。

该代码的主要目的是创建一个 "Conn" 类型的变量 "c"，该变量包含一个 "net.Conn" 类型的实例。

"Metadata()" 函数返回一个指向 "tunnel.Metadata" 类型对象的 "metadata" 类型的指针，如果没有其他活动发生，该指针将返回一个空 "metadata" 类型。

"WriteWithMetadata(p []byte, m *tunnel.Metadata)" 函数接收一个字节数组 "p" 和一个指向 "tunnel.Metadata" 类型对象的 "m" 参数。它将 "p" 字节数组中的数据发送到 "m" 所指向的 "metadata" 类型对象的 "地址" 字段中，并返回两个参数中的任意一个，具体取决于是否有数据可以发送。

"WriteTo(p []byte, m *tunnel.Metadata)" 函数接收一个字节数组 "p" 和一个指向 "tunnel.Metadata" 类型对象的 "m" 参数。它将 "p" 字节数组中的数据发送到 "m" 所指向的 "metadata" 类型对象的 "地址" 字段中，并返回一个整数表示有多少数据可以发送，如果发送成功。


```go
type Conn struct {
	net.Conn
}

func (c *Conn) Metadata() *tunnel.Metadata {
	return nil
}

type PacketConn struct {
	*net.UDPConn
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
	return c.WriteTo(p, m.Address)
}

```

这两函数是PacketConn类的功能，作用如下：

func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
	n, addr, err := c.ReadFrom(p)
	if err != nil {
		return 0, nil, err
	}
	address, err := tunnel.NewAddressFromAddr("udp", addr.String())
	common.Must(err)
	metadata := &tunnel.Metadata{
		Address: address,
	}
	return n, metadata, nil
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error) {
	if udpAddr, ok := addr.(*net.UDPAddr); ok {
		return c.WriteToUDP(p, udpAddr)
	}
	ip, err := addr.(*tunnel.Address).ResolveIP()
	if err != nil {
		return 0, err
	}
	udpAddr := &net.UDPAddr{
		IP:   ip,
		Port: addr.(*tunnel.Address).Port,
	}
	return c.WriteToUDP(p, udpAddr)
}

PacketConn类是用于在UDP协议中传输数据的一个封装类，提供了一些基本的UDP函数。在这两个函数中，c是一个PacketConn实例，p是一个字节切片，addr是一个网络地址。这两个函数分别实现了ReadWithMetadata和WriteTo函数，用于从和向PacketConn实例发送数据。

在ReadWithMetadata函数中，c从地址和端口组成的一个IP数据报中读取数据，并尝试将其转换为tunnel.Metadata类型的数据，如果转换失败，则返回0并返回错误。如果成功，它返回读取到的数据包数量和tunnel.Metadata类型的数据。

在WriteTo函数中，c将一个字节切片p和一个网络地址addr作为参数，尝试向目标地址发送数据。如果使用UDP协议，它尝试将数据包发送到目标地址，并返回失败的结果和错误。如果UDP地址解析成功，它将构造一个udp数据报，设置为源地址为c的IP数据报发送。


```go
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
	n, addr, err := c.ReadFrom(p)
	if err != nil {
		return 0, nil, err
	}
	address, err := tunnel.NewAddressFromAddr("udp", addr.String())
	common.Must(err)
	metadata := &tunnel.Metadata{
		Address: address,
	}
	return n, metadata, nil
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error) {
	if udpAddr, ok := addr.(*net.UDPAddr); ok {
		return c.WriteToUDP(p, udpAddr)
	}
	ip, err := addr.(*tunnel.Address).ResolveIP()
	if err != nil {
		return 0, err
	}
	udpAddr := &net.UDPAddr{
		IP:   ip,
		Port: addr.(*tunnel.Address).Port,
	}
	return c.WriteToUDP(p, udpAddr)
}

```

该代码定义了一个名为SocksPacketConn的结构体，它包含一个Net packetConn实例和一个Socks5客户端实例。

函数WriteWithMetadata的作用是接收一个包含数据的字节数组和元数据对象，将其写入到SocksPacketConn的UDP套的数据中，并发送到目标IP地址和端口上。

具体来说，函数内部创建一个缓冲区(Buffer)并将其初始化为一个最大包大小，然后将元数据对象的地址写入缓冲区。接着，将数据(Payload)写入缓冲区，并使用SocksPacketConn的UDP套将其发送到目标IP地址和端口上。

如果发送过程中出现错误，函数将返回0并打印错误信息。


```go
type SocksPacketConn struct {
	net.PacketConn
	socksAddr   *net.UDPAddr
	socksClient *socks5.Client
}

func (c *SocksPacketConn) WriteWithMetadata(payload []byte, metadata *tunnel.Metadata) (int, error) {
	buf := bytes.NewBuffer(make([]byte, 0, MaxPacketSize))
	buf.Write([]byte{0, 0, 0}) // RSV, FRAG
	common.Must(metadata.Address.WriteTo(buf))
	buf.Write(payload)
	_, err := c.PacketConn.WriteTo(buf.Bytes(), c.socksAddr)
	if err != nil {
		return 0, err
	}
	log.Debug("sent udp packet to " + c.socksAddr.String() + " with metadata " + metadata.String())
	return len(payload), nil
}

```

该函数的作用是读取一个SocksPacketConn类型的数据连接发送的数据，并返回两个值：从数据中读取的UDP数据包的编号，以及在数据中包含的元数据。

具体来说，函数接收一个长度为MaxPacketSize的缓冲区作为输入，然后使用SocksPacketConn的PacketConn.ReadFrom函数读取数据连接中的数据。如果读取过程中出现错误，函数将返回0，表示没有成功读取数据。

如果读取成功，函数将继续处理从数据中读取的UDP数据包。首先，函数创建一个tunnel.Address类型的变量r，并使用bytes.NewBuffer函数从数据中读取。如果读取过程中出现错误，函数将返回一个错误对象。如果读取成功，函数将返回UDP数据包的编号和包含在数据中的元数据。

函数的参数c是一个SocksPacketConn类型的指针，用于与数据连接进行交互。函数返回值中包含的第一个值是UDP数据包的编号，第二个值是包含在数据中的元数据。


```go
func (c *SocksPacketConn) ReadWithMetadata(payload []byte) (int, *tunnel.Metadata, error) {
	buf := make([]byte, MaxPacketSize)
	n, from, err := c.PacketConn.ReadFrom(buf)
	if err != nil {
		return 0, nil, err
	}
	log.Debug("recv udp packet from " + from.String())
	addr := new(tunnel.Address)
	r := bytes.NewBuffer(buf[3:n])
	if err := addr.ReadFrom(r); err != nil {
		return 0, nil, common.NewError("socks5 failed to parse addr in the packet").Base(err)
	}
	length, err := r.Read(payload)
	if err != nil {
		return 0, nil, err
	}
	return length, &tunnel.Metadata{
		Address: addr,
	}, nil
}

```

此代码是使用 Go 语言编写的函数，名为 "Close"，接收一个名为 "c" 的 SocksPacketConn 类型的参数。

函数的作用是关闭与 "c" 指向的 SocksPacketConn 对象的连接，并返回关闭连接时可能产生的错误。

具体来说，函数首先调用 c.socksClient.Close() 方法，关闭 SocksPacketConn 中的 SocsClient 连接。然后，函数调用 c.PacketConn.Close() 方法，关闭 SocksPacketConn 中的网络连接。

函数的实现确保了当 SocksPacketConn 对象关闭时，所有连接都已经关闭，从而可以确保网络安全。


```go
func (c *SocksPacketConn) Close() error {
	c.socksClient.Close()
	return c.PacketConn.Close()
}

```

# `tunnel/freedom/freedom_test.go`

这段代码定义了一个名为"freedom"的包，其中包含了一些函数来演示网络编程的基本用法。以下是这段代码的主要作用和一些重要函数：

1. 导入一些必要的库：

	* "bytes": 用于创建和操作字节切片
	* "context": 用于上下文操作
	* "fmt": 用于格式化字符串
	* "testing": 用于测试
	* "time": 用于时间操作

	* "github.com/txthinking/socks5": 用于支持Socks5协议的库
	* "github.com/p4gefau1t/trojan-go/common": 用于通用的工具函数
	* "github.com/p4gefau1t/trojan-go/test/util": 用于测试的库
	* "github.com/p4gefau1t/trojan-go/tunnel": 用于创建和管理网络连接的库

2. 定义了一些函数：

	* "createTunnel": 创建一个Tunnel连接
	* "connectTunnel": 连接到创建好的Tunnel连接
	* "sendData": 向目标服务器发送数据
	* "receiveData": 接收来自目标服务器的数据
	* "forwardTunnel": 在两个Tunnel连接之间进行数据转发
	* "decryptTunnel": 解码通过Tunnel传输的数据

3. 在一些测试函数中使用了这些函数：

	* "testTunnelConnect": 用于测试Tunnel连接的基本功能
	* "testTunnelForward": 用于测试Tunnel转发数据的功能
	* "testTunnelDecrypt": 用于测试Tunnel解码数据的功能

	注意，除了上述列出的函数之外，还有其他函数被用来创建测试环境、设置时钟等。


```go
package freedom

import (
	"bytes"
	"context"
	"fmt"
	"testing"
	"time"

	"github.com/txthinking/socks5"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码是使用Go标准库中的testing包测试一个名为TestConn的函数。函数名为func TestConn，它接收一个testing.T类型的参数，代表一个测试。函数的作用是测试一个名为"测试TCP连接"的函数的正确性。

具体来说，函数首先创建一个名为"ctx"的上下文，和一个名为"cancel"的取消信号，上下文和取消信号都指向函数内部执行的代码块的外部。然后，创建一个名为"client"的Client实例，其中包含一个TCP上下文，上下文是一个与当前上下文相同的名为"ctx"的上下文，以及一个名为"cancel"的取消信号。

接下来，使用新创建的"client"实例中的"ctx"上下文创建一个名为"addr"的地址，并使用"NewAddressFromAddr"函数从"echo"地址创建一个TCP地址。创建地址的函数必须返回一个零错误，否则代码块会崩溃。

然后，使用"client.DialConn"函数使用刚刚创建的"client"实例中的"ctx"上下文和"addr"地址创建一个TCP连接，并返回一个成功和失败的结果。如果连接建立成功，则TCP连接的套接字将返回数据可用性。如果连接失败，则代码块将崩溃。

接下来，使用"util.GeneratePayload"函数生成一个长度为1024的负载数据。然后，使用"client.Write"函数将负载数据发送到刚刚创建的TCP连接中，并返回数据写入的错误。最后，使用"client.Read"函数从TCP连接中读取数据，并存储在"recvBuf"中。然后，使用"bytes.Equal"函数比较发送的负载数据和接收的负载数据，如果它们不同，则代码块将崩溃。最后，使用"client.Close"函数关闭TCP连接。


```go
func TestConn(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	client := &Client{
		ctx:    ctx,
		cancel: cancel,
	}
	addr, err := tunnel.NewAddressFromAddr("tcp", util.EchoAddr)
	common.Must(err)
	conn1, err := client.DialConn(addr, nil)
	common.Must(err)

	sendBuf := util.GeneratePayload(1024)
	recvBuf := [1024]byte{}

	common.Must2(conn1.Write(sendBuf))
	common.Must2(conn1.Read(recvBuf[:]))

	if !bytes.Equal(sendBuf, recvBuf[:]) {
		t.Fail()
	}
	client.Close()
}

```

该代码是一个名为 "TestPacket" 的函数，属于 "net" 包测试。其作用是测试一个 UDP 数据包通过一个经过确认的中间 UDP 服务器发送并接收是否能够到达目的地。

具体地，该函数创建了一个 "ctx" 上下文并挂载到测试上下文，然后使用一个经过确认的中间 UDP 地址 "udp" 创建了一个 "Client" 对象并配置了与上下文相关的 "cancel" 信号。

接着使用 "NewAddressFromAddr" 函数创建一个中间 UDP 地址，并使用 "DialPacket" 函数尝试使用该地址与客户端建立连接。

然后使用 "generatePayload" 函数生成一个 1024 字节的负载数据，并使用 "conn1.WriteTo" 函数将数据负载写入客户端的输入输出流中，同时使用 "conn1.ReadFrom" 函数从客户端接收并存储接收到的数据。

最后，比较客户端接收到的数据与发送的数据是否相同。若不同，则表示数据包在传输过程中发生了差错，从而导致失败。


```go
func TestPacket(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	client := &Client{
		ctx:    ctx,
		cancel: cancel,
	}
	addr, err := tunnel.NewAddressFromAddr("udp", util.EchoAddr)
	common.Must(err)
	conn1, err := client.DialPacket(nil)
	common.Must(err)

	sendBuf := util.GeneratePayload(1024)
	recvBuf := [1024]byte{}

	common.Must2(conn1.WriteTo(sendBuf, addr))
	_, _, err = conn1.ReadFrom(recvBuf[:])
	common.Must(err)

	if !bytes.Equal(sendBuf, recvBuf[:]) {
		t.Fail()
	}
}

```

This is a Go program that sets up a Telnet server using the `net Telnet` package. The server will监听来自127.0.0.1的TCP连接，创建一个基于Telnet的客户端，并设置代理连接。

The program first sets up a Telnet server using the `udp` protocol and the port 127.0.0.1. The server will create a `Client` object, which will be used to establish the Telnet connection with the client.

The server then sets up a reverse proxy for the client, which will forward the client's traffic to the target address specified by the `TunnelTarget` parameter when建立TCP connections.

The server then enters a loop that listens for incoming connections. When a connection is established, the server reads the payload of the incoming message and sends it back to the client.

The server also sets up a Telnet client to connect to the target through the reverse proxy.

The program uses the `ClientDial` method to establish the initial connection with the target, and the `ClientDialPacket` method to send packets to the target through the reverse proxy.

The `TelnetMetadata` struct is used to include the target address in the packet sent to the target.

The program uses the `TunnelTarget` parameter to specify the target address for the reverse proxy. This can be either a IP address or a port number.


```go
func TestSocks(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())

	socksAddr := tunnel.NewAddressFromHostPort("udp", "127.0.0.1", common.PickPort("udp", "127.0.0.1"))
	client := &Client{
		ctx:          ctx,
		cancel:       cancel,
		proxyAddr:    socksAddr,
		forwardProxy: true,
		noDelay:      true,
	}
	target, err := tunnel.NewAddressFromAddr("tcp", util.EchoAddr)
	common.Must(err)
	s, _ := socks5.NewClassicServer(socksAddr.String(), "127.0.0.1", "", "", 0, 0)
	s.Handle = &socks5.DefaultHandle{}
	go s.RunTCPServer()
	go s.RunUDPServer()

	time.Sleep(time.Second * 2)
	conn, err := client.DialConn(target, nil)
	common.Must(err)
	payload := util.GeneratePayload(1024)
	common.Must2(conn.Write(payload))

	recvBuf := [1024]byte{}
	conn.Read(recvBuf[:])
	if !bytes.Equal(recvBuf[:], payload) {
		t.Fail()
	}
	conn.Close()

	packet, err := client.DialPacket(nil)
	common.Must(err)
	common.Must2(packet.WriteWithMetadata(payload, &tunnel.Metadata{
		Address: target,
	}))

	recvBuf = [1024]byte{}
	n, m, err := packet.ReadWithMetadata(recvBuf[:])
	common.Must(err)

	if n != 1024 || !bytes.Equal(recvBuf[:], payload) {
		t.Fail()
	}

	fmt.Println(m)
	packet.Close()
	client.Close()
}

```

# `tunnel/freedom/tunnel.go`

这是一个使用 Go 语言编写的名为 "freedom" 的库，它包含了一个名为 "Tunnel" 的结构体。根据该库的官方文档，这个库可能用于在 Go 应用程序之间建立网络隧道。在这个示例中，我们创建了一个名为 "FREEDOM" 的命名管道，该命名管道代表这个库。

"Tunnel" 结构体包含了一个名为 "Name" 的字段，该字段用于存储该命名管道的外部名称。

此外，该库使用 "github.com/p4gefau1t/trojan-go/tunnel" 作为其 "import" 语句的导入来源。这表明它可能使用 "trojan-go" 库来与 Go 应用程序进行交互，并使用 "github.com/p4gefau1t" 的 "tunnel" 包来建立网络隧道。


```go
package freedom

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "FREEDOM"

type Tunnel struct{}

func (*Tunnel) Name() string {
	return Name
}

```

这是一段使用 Go 编写的代码，它实现了一个简单的隧道客户端和服务器。在这段代码中，我们定义了一个名为 Tunnel 的接口，用于定义客户端和服务器，该接口包含两个方法：NewClient 和 NewServer。

具体来说，这段代码实现了一个 Tunnel 类型的接口，其中 NewClient 方法接收一个上下文上下文（Context）和一个隧道客户端（client），返回一个隧道客户端和一个成功创建客户端的错误。NewServer 方法则没有实现，因为它被留空了。

该代码的作用是：

1. 定义了一个名为 Tunnel 的接口，包含 NewClient 和 NewServer 方法，用于创建隧道客户端和服务器。
2. 在初始化函数 init 中，注册了一个名为 Name 的隧道名称，并将其映射到 Tunnel 接口上，使得我们可以在其他地方使用 Tunnel 名称来调用该接口。
3. 实现了 Tunnel 接口，以便创建和管理隧道客户端和服务器。


```go
func (*Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, client)
}

func (*Tunnel) NewServer(ctx context.Context, client tunnel.Server) (tunnel.Server, error) {
	panic("not supported")
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/http/http_test.go`

这段代码是一个 Go 语言中的 package，名为 "http"。它定义了一系列用于测试 HTTP 客户端和服务器工具的函数和接口。

具体来说，这段代码包含以下几个主要部分：

1. 导入一些必要的库：
	* "bufio": 用于字符缓冲区的库，提供了 `Reader` 和 `Writer` 接口，方便读写缓冲区。
	* "context": 用于上下文管理的库，提供了 `Chan`、`Duration` 等接口，方便各种上下文的管理。
	* "fmt": 用于格式化的库，定义了 `Fmtln` 函数，用于打印格式化的输出。
	* "net": 用于网络通信的库，定义了 `TcpListener`、`TcpReader`、`TcpWriter` 等接口，以及用于设置 HTTP 客户端和服务器超时时间的 `time.Duration` 类型。
	* "testing": 用于测试的库，定义了 `Run` 函数，方便在测试中运行代码。
	* "trojan-go": 该库的依赖库，定义了一些常用的函数和接口，如 ` common`、`config`、`test/util` 等。
	* "github.com/p4gefau1t/trojan-go/common": 该库的私有依赖库，定义了一些通用的函数和接口，如 `http


```go
package http

import (
	"bufio"
	"context"
	"fmt"
	"io/ioutil"
	"net"
	"net/http"
	"testing"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This appears to be a Go program that simulates a connection to a Google website using the `net/http` and `io/ioutil` packages.

The program first establishes a connection to a server running on the local machine (at IP address `127.0.0.1` and port `443`). It then sends an HTTP request to that server to create a new connection.

If the connection is successful (HTTP/1.1 200 OK), the program reads all the data from the request body and sends a response back to the server. It then closes the connection and stops the program.

The program also checks that the connection is established correctly before sending the request. If the connection is not established correctly, the program will fail and end up.


```go
func TestHTTP(t *testing.T) {
	port := common.PickPort("tcp", "127.0.0.1")
	ctx := config.WithConfig(context.Background(), transport.Name, &transport.Config{
		LocalHost: "127.0.0.1",
		LocalPort: port,
	})

	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)
	s, err := NewServer(ctx, tcpServer)
	common.Must(err)

	for i := 0; i < 10; i++ {
		go func() {
			resp, err := http.Get(fmt.Sprintf("http://127.0.0.1:%d", port))
			common.Must(err)
			defer resp.Body.Close()
		}()
		time.Sleep(time.Microsecond * 10)
		conn, err := s.AcceptConn(nil)
		common.Must(err)
		bufReader := bufio.NewReader(bufio.NewReader(conn))
		req, err := http.ReadRequest(bufReader)
		common.Must(err)
		fmt.Println(req)
		ioutil.ReadAll(req.Body)
		req.Body.Close()
		resp, err := http.Get("http://127.0.0.1:" + util.HTTPPort)
		common.Must(err)
		defer resp.Body.Close()
		err = resp.Write(conn)
		common.Must(err)
		buf := [100]byte{}
		_, err = conn.Read(buf[:])
		if err == nil {
			t.Fail()
		}
		conn.Close()
	}

	req, err := http.NewRequest(http.MethodConnect, "https://google.com:443", nil)
	common.Must(err)
	conn1, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))
	common.Must(err)
	go func() {
		common.Must(req.Write(conn1))
	}()

	conn2, err := s.AcceptConn(nil)
	common.Must(err)

	if conn2.Metadata().Port != 443 || conn2.Metadata().DomainName != "google.com" {
		t.Fail()
	}

	connResp := "HTTP/1.1 200 Connection established\r\n\r\n"
	buf := make([]byte, len(connResp))
	_, err = conn1.Read(buf)
	common.Must(err)
	if string(buf) != connResp {
		t.Fail()
	}

	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}

	conn1.Close()
	conn2.Close()
	s.Close()
}

```

# `tunnel/http/server.go`

这段代码是一个 Go 语言中的 package，名为 "http"。它定义了一系列用于创建 HTTP 网络连接的函数和结构体。

具体来说，这段代码以下几个主要组件：

1. 通过导入 "net" 和 "io" 包，可以访问全局的 HTTP 客户端和服务器 API。
2. 通过导入 "http" 包，可以定义一个 HTTP 连接的实例。
3. 通过使用 "fmt" 函数，将错误消息打印到控制台。
4. 通过使用 "io/ioutil" 包，将 HTTP 请求主体和响应 body 的内容读取并关闭。
5. 通过使用 "net/http" 包，可以创建 HTTP 连接。
6. 通过使用 "strings" 包，对 HTTP 请求的 URI 进行解析。
7. 通过使用 "trojan-go/common" 和 "trojan-go/log" 包，可以设置一些全局的选项，如是否启用日志记录，日志格式等。
8. 通过使用 "tunnel" 包，可以创建一个隧道，允许在不安全的网络连接中进行安全的通信。


```go
package http

import (
	"bufio"
	"context"
	"fmt"
	"io"
	"io/ioutil"
	"net"
	"net/http"
	"strings"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码定义了一个名为 `ConnectConn` 的结构体，它包含一个网络连接 `net.Conn` 和一个元数据 `metadata` 字段。

在 `ConnectConn` 结构体的 `Metadata` 函数中，返回了该结构体的 `metadata` 字段，以便在调用方进行使用。

另外，定义了一个名为 `OtherConn` 的结构体，它包含一个网络连接 `net.Conn`、一个元数据 `metadata` 字段，以及一个 `reqReader`、一个 `respWriter` 和一个 `ctx` 字段。

`OtherConn` 结构体定义了另一个连接的实现，这个连接的 `metadata` 字段在 `Connect` 函数中设置为固定值，即常量值。同时，该连接还有一个 `reqReader` 和一个 `respWriter`，分别用于读取和写入数据，以及一个 `ctx` 字段用于跟踪连接上下文。此外，还定义了一个 `cancel` 字段，用于取消连接。


```go
type ConnectConn struct {
	net.Conn
	metadata *tunnel.Metadata
}

func (c *ConnectConn) Metadata() *tunnel.Metadata {
	return c.metadata
}

type OtherConn struct {
	net.Conn
	metadata   *tunnel.Metadata // fixed
	reqReader  *io.PipeReader
	respWriter *io.PipeWriter
	ctx        context.Context
	cancel     context.CancelFunc
}

```

这段代码定义了两个函数，一个是`func (c *OtherConn) Metadata() *tunnel.Metadata`，返回一个指向`tunnel.Metadata`类型的指针；另一个是`func (c *OtherConn) Read(p []byte) (int, error)`，返回从连接中读取数据的能力。

具体来说，第一个函数`func (c *OtherConn) Metadata() *tunnel.Metadata`是一个简单的函数，它返回`c.metadata`的值，其中`c.metadata`是一个指向`OtherConn`类型的指针，由于这个函数没有对`c.metadata`进行修改，因此返回的值也是不变的。

第二个函数`func (c *OtherConn) Read(p []byte) (int, error)`则是一个比较复杂的函数。它接收一个字节切片`p`，然后通过调用`c.reqReader`的方法来读取数据。如果读取过程中出现错误，例如读取到了末尾却还没有数据可读，那么函数的行为和错误信息有关，但通常情况下会返回一个非零的整数和一个错误对象。如果连接正常结束，则返回读取的数据长度。由于这个函数使用了多线程和/或上下文感知，因此它返回的值可能会因为连接状态的变化而有所不同，但是通常情况下不会有其他的问题。


```go
func (c *OtherConn) Metadata() *tunnel.Metadata {
	return c.metadata
}

func (c *OtherConn) Read(p []byte) (int, error) {
	n, err := c.reqReader.Read(p)
	if err == io.EOF {
		if n != 0 {
			panic("non zero")
		}
		for range c.ctx.Done() {
			return 0, common.NewError("http conn closed")
		}
	}
	return n, err
}

```

这段代码定义了一个名为`Server`的结构体，表示一个服务器。这个服务器有两个主要方法：`func`和`func`。

`func`方法：


func (c *OtherConn) Write(p []byte) (int, error) {
	return c.respWriter.Write(p)
}

func (c *OtherConn) Close() error {
	c.cancel()
	c.reqReader.Close()
	c.respWriter.Close()
	return nil
}


1. `func (c *OtherConn) Write(p []byte) (int, error)` 是一个方法，接收一个字节数组（`p`）。它将调用`c.respWriter.Write`，这个方法接收一个字节数组并返回在通道（`c.connChan`）中的写入操作。

2. `func (c *OtherConn) Close() error` 是另一个方法，它会在关闭服务器时执行。它首先调用`c.cancel`，这个方法会在服务器实例中执行 cancel 操作。然后它关闭 `c.reqReader` 和 `c.respWriter`，这两个方法是用于从和向客户端发送数据的。最后，它返回一个`nil`作为结果，表示操作成功。

整个 `Server` 类用于管理服务器与客户端之间的通信。通过 `Server` 实例，客户端可以与服务器建立连接，并发送和接收数据。


```go
func (c *OtherConn) Write(p []byte) (int, error) {
	return c.respWriter.Write(p)
}

func (c *OtherConn) Close() error {
	c.cancel()
	c.reqReader.Close()
	c.respWriter.Close()
	return nil
}

type Server struct {
	underlay tunnel.Server
	connChan chan tunnel.Conn
	ctx      context.Context
	cancel   context.CancelFunc
}

```

This is a Go program that uses the go-http-server library to handle HTTP requests and responses. It creates a new HTTP connection using the `net/http` package and passes it to the `RelayConn` instance, which is then passed to the `RelayHttp` struct.

The `RelayHttp` struct represents an HTTP connection pool that can handle a large number of incoming connections. It includes fields such as `addr`, `ctx`, `cancel`, `reqReader`, `respWriter`, and `connChan`.

The `addr` field is the IP address or name of the server to which the connection will be established. The `ctx` field is the context used by the remote connection. `cancel` is a boolean that is used to indicate whether the connection should be closed by the caller. `reqReader` and `respWriter` are instances of `Reader` and `Writer` that are used to read and write HTTP requests and responses, respectively.

The `connChan` field is a channel that is used to send and receive requests and responses between the `RelayHttp` struct and the `RelayConn` instance.

The `relay` function is used to establish the new connection and the `relayRead` function is used to read the response from the remote server.


```go
func (s *Server) acceptLoop() {
	for {
		conn, err := s.underlay.AcceptConn(&Tunnel{})
		if err != nil {
			select {
			case <-s.ctx.Done():
				log.Error(common.NewError("http closed"))
				return
			default:
				log.Error(common.NewError("http failed to accept connection").Base(err))
				continue
			}
		}

		go func(conn net.Conn) {
			reqBufReader := bufio.NewReader(ioutil.NopCloser(conn))
			req, err := http.ReadRequest(reqBufReader)
			if err != nil {
				log.Error(common.NewError("not a valid http request").Base(err))
				return
			}

			if strings.ToUpper(req.Method) == "CONNECT" { // CONNECT
				addr, err := tunnel.NewAddressFromAddr("tcp", req.Host)
				if err != nil {
					log.Error(common.NewError("invalid http dest address").Base(err))
					conn.Close()
					return
				}
				resp := fmt.Sprintf("HTTP/%d.%d 200 Connection established\r\n\r\n", req.ProtoMajor, req.ProtoMinor)
				_, err = conn.Write([]byte(resp))
				if err != nil {
					log.Error("http failed to respond connect request")
					conn.Close()
					return
				}
				s.connChan <- &ConnectConn{
					Conn: conn,
					metadata: &tunnel.Metadata{
						Address: addr,
					},
				}
			} else { // GET, POST, PUT...
				defer conn.Close()
				for {
					reqReader, reqWriter := io.Pipe()
					respReader, respWriter := io.Pipe()
					var addr *tunnel.Address
					if addr, err = tunnel.NewAddressFromAddr("tcp", req.Host); err != nil {
						addr = tunnel.NewAddressFromHostPort("tcp", req.Host, 80)
					}
					log.Debug("http dest", addr)

					ctx, cancel := context.WithCancel(s.ctx)
					newConn := &OtherConn{
						Conn: conn,
						metadata: &tunnel.Metadata{
							Address: addr,
						},
						ctx:        ctx,
						cancel:     cancel,
						reqReader:  reqReader,
						respWriter: respWriter,
					}
					s.connChan <- newConn // pass this http session connection to proxy.RelayConn

					err = req.Write(reqWriter) // write request to the remote
					if err != nil {
						log.Error(common.NewError("http failed to write http request").Base(err))
						return
					}

					respBufReader := bufio.NewReader(ioutil.NopCloser(respReader)) // read response from the remote
					resp, err := http.ReadResponse(respBufReader, req)
					if err != nil {
						log.Error(common.NewError("http failed to read http response").Base(err))
						return
					}
					err = resp.Write(conn) // send the response back to the local
					if err != nil {
						log.Error(common.NewError("http failed to write the response back").Base(err))
						return
					}
					newConn.Close()
					req.Body.Close()
					resp.Body.Close()

					req, err = http.ReadRequest(reqBufReader) // read the next http request from local
					if err != nil {
						log.Error(common.NewError("http failed to the read request from local").Base(err))
						return
					}
				}
			}
		}(conn)
	}
}

```

这段代码定义了三个函数，作用如下：

1. `AcceptConn`函数接收一个`Server`类型的参数`s`，以及一个`Tunnel`类型的参数`tunnel`。该函数的作用是在`Server`中接受一个连接请求，并返回一个`Tunnel`类型的`conn`和一个`error`类型的`err`。

2. `AcceptPacket`函数同样接收一个`Server`类型的参数`s`，以及一个`Tunnel`类型的参数`tunnel`。该函数的作用是在`Server`中接收一个数据包请求，并返回一个`Tunnel`类型的`packetConn`和一个`error`类型的`err`。

3. `Close`函数接收一个`Server`类型的参数`s`。该函数的作用是关闭由`s`实例管理的整个网络堆栈，并返回一个错误。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	select {
	case conn := <-s.connChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("http server closed")
	}
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	<-s.ctx.Done()
	return nil, common.NewError("http server closed")
}

func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

```

这段代码定义了一个名为NewServer的函数，它接受一个上下文上下文和一个通道.Server类型的参数。

函数内部创建了一个Server对象，该对象包含一个通道.Underlay类型，一个通道.Conn类型，以及一个取消通道.Context类型和一个取消通道.Conn类型的上下文上下文。

函数还启动了一个接受连接的循环，该循环一直运行 until 函数返回，以确保服务器一直处于连接状态。

最终，函数返回了一个Server对象和一个 nil 错误，表示创建成功。


```go
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		underlay: underlay,
		connChan: make(chan tunnel.Conn, 32),
		ctx:      ctx,
		cancel:   cancel,
	}
	go server.acceptLoop()
	return server, nil
}

```

# `tunnel/http/tunnel.go`

这段代码定义了一个名为 "http" 的包，该包包含一个名为 "Tunnel" 的 struct 类型。这个 struct 类型包含一个名为 "Name" 的字段，其值为 "HTTP"。此外，该 struct 类型还有一个名为 "Tunnel" 的字段，该字段是一个匿名类型，它将 "Name" 的值保留，但不会创建一个名为 "Tunnel" 的实例。


```go
package http

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "HTTP"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}

```

这两位代码定义了两个名为NewClient和NewServer的函数，用于创建隧道客户端和隧道服务器。

具体来说，函数接收两个参数：

- `t`：是一个指向Tunnel类型的变量，可能会用于后续的数据传递。
- `ctx`：是一个带有上下文参数的Context，可以用于在函数内部设置、获取或取消上下文。
- `client`：是一个Tunnel.Client类型的变量，用于创建新的隧道客户端。

函数内部使用了 panics() 函数来输出一条错误消息，但并不会进行实际的错误处理。如果在函数外部调用了这些函数，并且错误消息没有在必要时进行输出，那么程序可能会崩溃。

函数的实现是为了创建一个名为Tunnel的服务，可以用于在代理和原始之间建立隧道，允许在代理一侧通过发送RPC请求来访问原始一侧的数据。


```go
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	panic("not supported")
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, server)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/mux/client.go`

这段代码是一个名为 "mux" 的 packages，其中包含了以下几个部分：

1. 导入了一些外部的库：
python
import (
	"context"
	"fmt"
	"math/rand"
	"sync"
	"time"

	"github.com/xtaci/smux"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

2. 导入了 "mux/channels" 和 "mux/topics" 包：
python
from mux import channels, topics

3. 通过 `fmt.Printf` 函数输出 "mux" 包的版本信息：
python
fmt.Printf("mux version %s\n", mux.Version)

4. 通过 `time.rand` 函数生成一个随机数，并将其作为 `math.rand.Intn` 函数的参数：
python
math.rand.seed(time.Now().UnixNano())

5. 通过 `sync.NewMutex` 函数创建一个 `sync.WaitGroup`，并将其加入 `sync.WaitGroup` 列表中：
python
group := sync.NewMutex()
ch := make(chan bool)

group.Add(1)

6. 通过 `trojan.config.LoadConfig` 函数加载配置文件中的配置信息：
python
config.Init(config.ConfigFile())

7. 通过 `log.SetLevel` 函数设置日志输出的最低级别：
python
log.SetLevel(log.LogLevelDebug)

8. 通过 `trojan.core. traceback` 函数输出调用栈跟踪：
python
trojan.core.traceback.Start()

9. 通过 `trojan.core.可选` 函数创建一个可选的通道：
python
channel, err := channels.NewDirectionChannel(<channel_id>)
if err != nil {
	log.Fatalf("Failed to create channel: %v", err)
}

10. 通过 `trojan.core.write` 函数向选定的方向写入数据：
python
data := []byte("Hello, world!")
channel.Write(data)



```go
package mux

import (
	"context"
	"fmt"
	"math/rand"
	"sync"
	"time"

	"github.com/xtaci/smux"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码定义了一个名为 `muxID` 的类型为 `uint32` 的枚举类型。该类型定义了一个名为 `generateMuxID` 的函数，用于生成一个随机的 `muxID` 值。

接下来的 `generateMuxID` 函数使用了 `rand.Uint32()` 生成一个 `uint32` 类型的随机整数，并将其转换为 `muxID` 类型，作为函数的返回值。

然后，定义了一个名为 `smuxClientInfo` 的结构体类型，该结构体包含一个名为 `id` 的 `muxID` 类型字段、一个名为 `client` 的 `smux.Session` 类型字段、一个名为 `lastActiveTime` 的 `time.Time` 类型字段和一个名为 `underlayConn` 的 `tunnel.Conn` 类型字段。

接着，定义了一个名为 `Client` 的结构体类型，该结构体包含一个名为 `clientPoolLock` 的 `sync.Mutex` 类型的字段、一个名为 `clientPool` 的 `map[muxID]*smuxClientInfo` 类型的字段和一个名为 `underlay` 的 `tunnel.Client` 类型的字段。该结构体还包含一个名为 `concurrency` 的 `int` 类型的字段、一个名为 `timeout` 的 `time.Duration` 类型的字段和一个名为 `ctx` 的 `context.Context` 类型的字段、一个名为 `cancel` 的 `context.CancelFunc` 类型的字段和一个名为 `generateMuxID` 的函数。

最后，该函数没有输出任何值，返回了一个 `muxID` 类型的值，用于定义 `Client` 类型的实例。


```go
type muxID uint32

func generateMuxID() muxID {
	return muxID(rand.Uint32())
}

type smuxClientInfo struct {
	id             muxID
	client         *smux.Session
	lastActiveTime time.Time
	underlayConn   tunnel.Conn
}

// Client is a smux client
type Client struct {
	clientPoolLock sync.Mutex
	clientPool     map[muxID]*smuxClientInfo
	underlay       tunnel.Client
	concurrency    int
	timeout        time.Duration
	ctx            context.Context
	cancel         context.CancelFunc
}

```

This is a Go program that performs a constant time check on the number of active MX clients in a circuit breaker. It uses the `time.Since()` function to check how much time has passed since the last active MX client, and the `is.Closed()` method from the `mux` package to check if the client is closed or not.

The program takes an initial timeout value (`c.timeout`) and a check interval (`c.checkDuration`) in seconds. The check interval is divided into four 25-second intervals.

The program enters a loop that repeatedly checks for active MX clients by closed or not. The client is considered active if it has at least one stream and the client is not closed or if the timeout has passed.

For each active client, the program sends a request to the server to check the client's status. If the client is not active anymore or the timeout has passed, the client is closed.

The program also logs the number of active clients by printing the number of active clients in the `mux` package.

The program uses the `select` statement to wait for new client requests. The program returns after waiting for any new client requests for a period of time equal to the `checkDuration` (in seconds).


```go
func (c *Client) Close() error {
	c.cancel()
	c.clientPoolLock.Lock()
	defer c.clientPoolLock.Unlock()
	for id, info := range c.clientPool {
		info.client.Close()
		log.Debug("mux client", id, "closed")
	}
	return nil
}

func (c *Client) cleanLoop() {
	var checkDuration time.Duration
	if c.timeout <= 0 {
		checkDuration = time.Second * 10
		log.Warn("negative mux timeout")
	} else {
		checkDuration = c.timeout / 4
	}
	log.Debug("check duration:", checkDuration.Seconds(), "s")
	for {
		select {
		case <-time.After(checkDuration):
			c.clientPoolLock.Lock()
			for id, info := range c.clientPool {
				if info.client.IsClosed() {
					info.client.Close()
					info.underlayConn.Close()
					delete(c.clientPool, id)
					log.Info("mux client", id, "is dead")
				} else if info.client.NumStreams() == 0 && time.Since(info.lastActiveTime) > c.timeout {
					info.client.Close()
					info.underlayConn.Close()
					delete(c.clientPool, id)
					log.Info("mux client", id, "is closed due to inactivity")
				}
			}
			log.Debug("current mux clients: ", len(c.clientPool))
			for id, info := range c.clientPool {
				log.Debug(fmt.Sprintf("  - %x: %d/%d", id, info.client.NumStreams(), c.concurrency))
			}
			c.clientPoolLock.Unlock()
		case <-c.ctx.Done():
			log.Debug("shutting down mux cleaner..")
			c.clientPoolLock.Lock()
			for id, info := range c.clientPool {
				info.client.Close()
				info.underlayConn.Close()
				delete(c.clientPool, id)
				log.Debug("mux client", id, "closed")
			}
			c.clientPoolLock.Unlock()
			return
		}
	}
}

```

这段代码定义了一个名为 `func` 的函数，它接收一个名为 `c` 的指向 `Client` 结构体的指针，并返回一个名为 `*smuxClientInfo` 的指针和一个名为 `error` 的类型。

该函数的主要作用是确保每个 `Client` 实例都对应一个唯一的 `id`。具体实现包括以下几个步骤：

1. 生成一个唯一的 `id`，如果 `c` 中已经存在具有相同 `id` 的实例，则返回错误。
2. 创建一个 `tunnel.Address` 类型的变量 `fakeAddr`，并使用 `c.underlay.DialConn` 函数将其与一个默认的 `tunnel` 连接器建立连接。
3. 使用 `newStickyConn` 函数将连接器与 `tunnel` 连接器建立持久连接。
4. 创建一个默认的 `smuxConfig` 结构体，并将其设置为 `smux.DefaultConfig`。
5. 使用 `c.underlay.DialConn` 函数创建一个 `smux` 连接器实例，并将其设置为 `smuxConfig`。
6. 将 `id` 和 `lastActiveTime` 设置为当前时间，以确保 `Client` 实例具有相同的 `lastActiveTime`。
7. 将 `c.clientPool` 结构体中的 `id` 键设置为生成的 `id`，并将 `info` 设置为创建的 `smuxClientInfo` 实例。
8. 返回 `info`，如果没有错误，则返回 `nil`。


```go
func (c *Client) newMuxClient() (*smuxClientInfo, error) {
	// The mutex should be locked when this function is called
	id := generateMuxID()
	if _, found := c.clientPool[id]; found {
		return nil, common.NewError("duplicated id")
	}

	fakeAddr := &tunnel.Address{
		DomainName:  "MUX_CONN",
		AddressType: tunnel.DomainName,
	}
	conn, err := c.underlay.DialConn(fakeAddr, &Tunnel{})
	if err != nil {
		return nil, common.NewError("mux failed to dial").Base(err)
	}
	conn = newStickyConn(conn)

	smuxConfig := smux.DefaultConfig()
	// smuxConfig.KeepAliveDisabled = true
	client, _ := smux.Client(conn, smuxConfig)
	info := &smuxClientInfo{
		client:         client,
		underlayConn:   conn,
		id:             id,
		lastActiveTime: time.Now(),
	}
	c.clientPool[id] = info
	return info, nil
}

```

该函数名为 `func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error)`，它接收两个参数，一个是 `c` 指向的 `Client` 结构体的引用，另一个是 `tunnel.Address` 和 `tunnel.Tunnel` 类型的参数。

函数的主要作用是连接到远程服务器，并在连接成功后返回一个 `tunnel.Conn` 类型的实例，同时也可以处理连接失败的情况。

具体实现可以分为以下几个步骤：

1. 创建一个新的 `tunnel.Conn` 实例，或者是通过调用 `c.newMuxClient()` 函数来获取一个可用的 `tunnel.Client` 实例。
2. 如果 `c` 指向的 `Client` 是一个关闭的连接，那么关闭它并删除它所指向的 `tunnel.Client` 实例，同时记录当前时间作为 `info.lastActiveTime` 的值。
3. 如果 `c` 指向的 `Client` 没有连接或者连接不足，那么使用 `createNewConn()` 函数来尝试创建一个新的连接并返回。
4. 使用 `c.clientPoolLock.Lock()` 和 `c.clientPoolLock.Unlock()` 函数来确保在 `c.clientPool` 映射中锁定 `c` 指向的 `Client` 实例，并在循环过程中检查它是否有效。
5. 如果 `c` 指向的 `Client` 是一个关闭的连接，那么在循环结束后将 `info.underlayConn` 设置为 `nil`，并记录关闭时间。
6. 如果 `c` 指向的 `Client` 没有连接或者连接不足，那么在循环结束后返回 `createNewConn(info)` 函数返回的 `tunnel.Conn` 实例，其中 `info` 是从 `c.clientPool` 中获取到的 `smuxClientInfo` 类型的参数。


```go
func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
	createNewConn := func(info *smuxClientInfo) (tunnel.Conn, error) {
		rwc, err := info.client.Open()
		info.lastActiveTime = time.Now()
		if err != nil {
			info.underlayConn.Close()
			info.client.Close()
			delete(c.clientPool, info.id)
			return nil, common.NewError("mux failed to open stream from client").Base(err)
		}
		return &Conn{
			rwc:  rwc,
			Conn: info.underlayConn,
		}, nil
	}

	c.clientPoolLock.Lock()
	defer c.clientPoolLock.Unlock()
	for _, info := range c.clientPool {
		if info.client.IsClosed() {
			delete(c.clientPool, info.id)
			log.Info(fmt.Sprintf("Mux client %x is closed", info.id))
			continue
		}
		if info.client.NumStreams() < c.concurrency || c.concurrency <= 0 {
			return createNewConn(info)
		}
	}

	info, err := c.newMuxClient()
	if err != nil {
		return nil, common.NewError("no available mux client found").Base(err)
	}
	return createNewConn(info)
}

```

这两段代码定义了一个名为"DialPacket"的函数类型，代表一个名为"Client"的接口的函数，该函数接受一个名为"tunnel.Tunnel"的参数，并返回一个名为"tunnel.PacketConn"的类型，或者是"error"类型的错误。

这两段代码还定义了一个名为"NewClient"的函数，该函数接受一个名为"ctx.Context"的上下文参数，和一个名为"underlay"的名为"tunnel.Client"的参数，并返回一个名为"Client"的类型，或者是"error"类型的错误。

NewClient函数的实现者在函数内部创建了一个名为"client"的客户端对象，该客户端对象实现了"Client"接口，包括实现了"DialPacket"函数类型的"DialPacket"函数，以及实现了"smuxClientInfo"接口的"client.cleanLoop"函数。client对象还实现了"cleanLoop"函数类型的"client.cleanLoop"函数，该函数用于清理客户端池中的连接。


```go
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
	clientConfig := config.FromContext(ctx, Name).(*Config)
	ctx, cancel := context.WithCancel(ctx)
	client := &Client{
		underlay:    underlay,
		concurrency: clientConfig.Mux.Concurrency,
		timeout:     time.Duration(clientConfig.Mux.IdleTimeout) * time.Second,
		ctx:         ctx,
		cancel:      cancel,
		clientPool:  make(map[muxID]*smuxClientInfo),
	}
	go client.cleanLoop()
	log.Debug("mux client created")
	return client, nil
}

```