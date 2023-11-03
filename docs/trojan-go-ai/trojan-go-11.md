# trojan-go源码解析 11

# `tunnel/router/tunnel.go`

这段代码定义了一个名为 "router" 的包，其中包含一个名为 "Tunnel" 的结构体。

"Tunnel" 结构体包含一个名为 "Name" 的成员变量，其值为 "ROUTER"，用于标识这个结构体属于 "router" 包。

此外，导入了两个成员函数 "Tunnel" 和 "Context"，其中 "Tunnel" 函数没有定义任何成员变量或方法，而 "Context" 函数定义了一个名为 "Context" 的成员变量和一个名为 "Log" 的成员函数。


```go
package router

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "ROUTER"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}

```

这两函数是用于创建隧道客户端和隧道服务器的主要函数。在函数内部，首先创建一个新的客户端实例，然后使用该客户端创建一个新的隧道客户端。如果成功，将返回新的隧道客户端，否则返回一个错误。

如果没有其他函数进行注册，则会自动注册两个隧道客户端。注册时会根据隧道名称创建一个新的隧道实例，并将其添加到注册表中。注册后，如果调用上面两个函数创建新的隧道客户端，那么将会输出 "not supported"。


```go
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, client)
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	panic("not supported")
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/shadowsocks/client.go`

该代码是一个基于Shadowsocks2协议的Shadowsocks客户端。它实现了TrojanGo框架中的Client端。主要作用是实现网络通信的功能，包括建立与服务器的连接、发送数据、接收数据等。

具体来说，该代码的作用如下：

1. 导入必要的依赖：包括Shadowsocks2协议的规范、TrojanGo框架的相关库、Go语言的标准库等。

2. 定义一个名为Client的客户端结构体，包含一个底层网络客户端（tunnel.Client）和一个加密套件（core.Cipher）。

3. 通过创建一个Client实例，将加密套件设置为默认设置，然后设置连接服务器的信息（如服务器地址、端口、密码等）。

4. 实现Client的NetworkStreamRead和NetworkStreamWrite方法，分别用于读取和写入数据。这些方法使用Client实例底层网络客户端与服务器通信，并将数据加密后发送或接收。

5. 实现Client的Connect、SendMessage和ReceiveMessage方法，用于建立与服务器的连接、发送数据和接收数据。这些方法同样使用了Client实例底层网络客户端与服务器通信，并将数据加密后发送或接收。

6. 通过调用Client实例的NetworkStreamRead和NetworkStreamWrite方法，发送数据到服务器，并接收来自服务器的数据。

7. 在网络数据传输过程中，实现了一些辅助方法，如清理不必要的文件、记录日志等。

8. 通过使用TrojanGo框架提供的配置文件，设置一些具体的配置项，如连接服务器、加密方式等。

9. 最后，通过在客户端创建一个事件循环，确保在网络数据传输过程中，实时处理和回应。


```go
package shadowsocks

import (
	"context"

	"github.com/shadowsocks/go-shadowsocks2/core"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Client struct {
	underlay tunnel.Client
	core.Cipher
}

```

这两函数的作用是：

func (c *Client) DialConn(address *tunnel.Address, tunnel tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.underlay.DialConn(address, &Tunnel{})
	if err != nil {
		return nil, err
	}
	return &Conn{
		aeadConn: c.Cipher.StreamConn(conn),
		Conn:     conn,
	}, nil
}

func (c *Client) DialPacket(tunnel tunnel.Tunnel) (tunnel.PacketConn, error) {
	// Code not suitable for description
}

第一个函数 `DialConn` 接收一个 `tunnel.Address` 类型的参数和 `tunnel.Tunnel` 类型的参数，然后使用 `c.underlay.DialConn` 函数来建立一个安全套接字到远程设备的连接，并返回一个 `tunnel.Conn` 类型的变量和可能的错误。

第二个函数 `DialPacket` 没有接收任何参数，并返回一个 `tunnel.PacketConn` 类型的变量和可能的错误。由于该函数没有具体的实现，因此无法确定其作用。


```go
func (c *Client) DialConn(address *tunnel.Address, tunnel tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.underlay.DialConn(address, &Tunnel{})
	if err != nil {
		return nil, err
	}
	return &Conn{
		aeadConn: c.Cipher.StreamConn(conn),
		Conn:     conn,
	}, nil
}

func (c *Client) DialPacket(tunnel tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

```

这两段代码定义了一个名为`func`的函数，接收一个名为`c`的指针参数，代表一个`Client`实例。这个函数的实现是关闭该`Client`实例所代表的主机上的隧道服务。

这两段代码还定义了一个名为`func`的函数，接收一个名为`underlay`的`Client`实例参数，代表一个隧道服务客户端。这个函数的实现创建一个新的隧道服务客户端，使用通过调用`NewClient`函数选定的后置密码和隧道服务类型。如果选定的后置密码无效，函数将返回一个错误。否则，函数返回一个指向`Client`实例的`Client`实例和 nil。


```go
func (c *Client) Close() error {
	return c.underlay.Close()
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	cipher, err := core.PickCipher(cfg.Shadowsocks.Method, nil, cfg.Shadowsocks.Password)
	if err != nil {
		return nil, common.NewError("invalid shadowsocks cipher").Base(err)
	}
	log.Debug("shadowsocks client created")
	return &Client{
		underlay: underlay,
		Cipher:   cipher,
	}, nil
}

```

# `tunnel/shadowsocks/config.go`

这段代码定义了一个名为 "shadowsocks" 的包，其中包含一个名为 "Config" 的结构体，用于配置 Shadowsocks 服务器。

具体来说，这段代码定义了一个 "ShadowsocksConfig" 结构体，其中包含以下字段：

- "enabled"(enabled)：开启或关闭 Shadowsocks 服务器
- "method"(method):Shadowsocks 服务器使用的加密方法，可以是 "aes-256-cfb" 或 "aes-256-cfb-urp"
- "password"(password)：设置 Shadowsocks 服务器密码

然后，定义了一个 "Config" 结构体，其中包含以下字段：

- "remote-addr"(remote-addr)：服务器所在地址的 URL
- "remote-port"(remote-port)：服务器所在端口
- "shadowsocks"(shadowsocks)：配置了 Shadowsocks 服务器连接的配置信息，包括服务器地址、服务器端口、加密方法、密码等信息。

最后，通过创建一个 "Config" 结构体，将上述定义的配置信息集中存储到一个 "Config" 变量中，以便在需要时进行使用。


```go
package shadowsocks

import "github.com/p4gefau1t/trojan-go/config"

type ShadowsocksConfig struct {
	Enabled  bool   `json:"enabled" yaml:"enabled"`
	Method   string `json:"method" yaml:"method"`
	Password string `json:"password" yaml:"password"`
}

type Config struct {
	RemoteHost  string            `json:"remote_addr" yaml:"remote-addr"`
	RemotePort  int               `json:"remote_port" yaml:"remote-port"`
	Shadowsocks ShadowsocksConfig `json:"shadowsocks" yaml:"shadowsocks"`
}

```

这段代码是使用Go编程语言编写的，它描述了一个名为"init"的函数，该函数在函数初始化时执行。

函数的作用是注册一个名为"Name"的配置创建器，该创建器使用一个内部类型"ConfigCreator"来返回一个指向"Config"的引用。

"Config"是一个接口类型，它定义了一个"Shadowsocks"属性的配置对象，其中包含一个名为"Shadowsocks"的"Method"属性的值为"AES-128-GCM"。

总的来说，这段代码描述了一个配置创建器，用于创建一个具有指定方法的"Shadowsocks"配置对象。这个配置对象将作为函数返回，以便其他函数可以调用该函数来创建更多的配置对象。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{
			Shadowsocks: ShadowsocksConfig{
				Method: "AES-128-GCM",
			},
		}
	})
}

```

# `tunnel/shadowsocks/conn.go`

该代码定义了一个名为Shadowsocks的包，其中包含一个名为Conn的接口类型，以及一个名为Tunnel的接口类型。

该包使用Go标准库中的Net包来实现网络连接。通过使用Trojan网络代理库，该包创建一个包含两个接口的变量c，分别代表TCP类型的aeadConn和UDP类型的tunnel.Conn。

该包中的Conn接口定义了一个用于与远程服务器建立连接的接口，包括一个名为Read的函数，该函数使用aeadConn.Read函数将数据从aeadConn连接中读取。

该包中的Tunnel接口定义了一个用于通过Trojan网络代理库中Tunnel类型服务器连接远程服务器的方式。


```go
package shadowsocks

import (
	"net"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Conn struct {
	aeadConn net.Conn
	tunnel.Conn
}

func (c *Conn) Read(p []byte) (n int, err error) {
	return c.aeadConn.Read(p)
}

```

该代码定义了两个函数，一个是`func (c *Conn) Write(p []byte) (n int, err error)`，另一个是`func (c *Conn) Close() error`。

这两个函数都在一个名为`func (c *Conn)`的函数里，但需要注意的是，这两个函数都是作为一个名为`c`的参数传递给一个名为`Write`和`Close`的函数。因此，我们可以推断出，这个函数接收一个`*Conn`类型的参数。

接下来，我们分别来看这两个函数的实现：

1. `func (c *Conn) Write(p []byte) (n int, err error)`

这个函数接收一个`*byte`类型的参数`p`，并返回`n`和`err`类型的变量。根据题目描述，我们可以知道这个函数是在`c.aeadConn.Write`上进行写入操作，因此，这个函数实际上是调用了`c.aeadConn`的`Write`方法，并返回了写入的字节数`n`和错误情况`err`。具体实现如下：


func (c *Conn) Write(p []byte) (n int, err error) {
   return c.aeadConn.Write(p)
}


2. `func (c *Conn) Close() error`

这个函数接收一个`error`类型的参数，表示关闭连接时可能出现的错误。根据题目描述，我们可以知道这个函数实际上是关闭了`c.Conn`和`c.aeadConn`，因此，这个函数可能会出现以下几种情况：

- 如果连接没有打开，那么没有关闭操作。
- 如果连接已经打开，但是没有提供数据的输入或输出通道，那么关闭操作不会生效。
- 如果提供了数据的输入或输出通道，但是没有提供数据的输入或输出流，那么关闭操作不会生效。

具体实现如下：


func (c *Conn) Close() error {
   c.Conn.Close()
   c.aeadConn.Close()
   return nil
}


3. `func (c *Conn) Metadata() *tunnel.Metadata`

这个函数返回了一个名为`tunnel.Metadata`的类型，并且返回了一个指向该类型的指针。根据题目描述，我们可以知道这个函数是在`c.Conn.Metadata`上进行获取，因此，这个函数实际上是获取了连接的元数据，并返回了一个指向该元数据的指针。具体实现如下：


func (c *Conn) Metadata() *tunnel.Metadata {
   return c.Conn.Metadata()
}


综上所述，以上三个函数都是和一个名为`c`的参数有关，且都接收了一个`*byte`类型的参数，返回了一个整型和一个错误类型。


```go
func (c *Conn) Write(p []byte) (n int, err error) {
	return c.aeadConn.Write(p)
}

func (c *Conn) Close() error {
	c.Conn.Close()
	return c.aeadConn.Close()
}

func (c *Conn) Metadata() *tunnel.Metadata {
	return c.Conn.Metadata()
}

```

# `tunnel/shadowsocks/server.go`

该代码是一个名为"shadowsocks"的包，它import了多个其他包以及自定义的"shadowsocks2"核心。它似乎是一个用于创建和管理Shadowsocks网络代理的库。通过导入其他必要的组件，该库可以提供各种功能，包括创建和管理代理实例、设置代理的自动启动、设置代理的连接信息等等。


```go
package shadowsocks

import (
	"context"
	"net"

	"github.com/shadowsocks/go-shadowsocks2/core"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/redirector"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

此代码定义了一个名为Server的SSR服务器结构体，它实现了Istanbul-style的Shadowsocks服务。

服务器的主要组件包括：

1. 一个名为core.Cipher的加密套接字，用于处理数据传输过程中的加密和解密。
2. 一个名为redirector.Redirector的代理服务器，用于将客户端连接到代理服务器，再由代理服务器转发到后端服务器。
3. 一个名为underlay的隧道.Server，用于与后端服务器建立隧道，将数据传输加密并转发。
4. 一个名为net.Addr的网络地址类型，用于将服务器地址与连接的客户端进行映射。

服务器还实现了一个名为AcceptConn的函数，用于接受客户端连接并返回一个已经连接的tunnel.Conn类型。函数接受一个名为overlay的隧道.Server作为底层通道，然后使用underlay与后端服务器建立连接，最后返回客户端连接到后端服务器的tunnel.Conn类型。

AcceptConn函数的具体实现如下：

go
func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := s.underlay.AcceptConn(&Tunnel{})
	if err != nil {
		return nil, common.NewError("shadowsocks failed to accept connection from underlying tunnel").Base(err)
	}
	rewindConn := common.NewRewindConn(conn)
	rewindConn.SetBufferSize(1024)
	defer rewindConn.StopBuffering()

	// try to read something from this connection
	buf := [1024]byte{}
	testConn := s.Cipher.StreamConn(rewindConn)
	if _, err := testConn.Read(buf[:]); err != nil {
		// we are under attack
		log.Error(common.NewError("shadowsocks failed to decrypt").Base(err))
		rewindConn.Rewind()
		rewindConn.StopBuffering()
		s.Redirect(&redirector.Redirection{
			RedirectTo:  s.redirAddr,
			InboundConn: rewindConn,
		})
		return nil, common.NewError("invalid aead payload")
	}
	rewindConn.Rewind()
	rewindConn.StopBuffering()

	return &Conn{
		aeadConn: s.Cipher.StreamConn(rewindConn),
		Conn:     conn,
	}, nil
}


这个函数的作用是接受一个底层隧道，然后尝试通过该隧道与后端服务器建立连接。如果连接建立成功，就返回客户端连接到后端服务器的tunnel.Conn类型。如果连接建立失败，函数将执行重放攻击操作，然后停止接受客户端连接，防止进一步攻击。


```go
type Server struct {
	core.Cipher
	*redirector.Redirector
	underlay  tunnel.Server
	redirAddr net.Addr
}

func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := s.underlay.AcceptConn(&Tunnel{})
	if err != nil {
		return nil, common.NewError("shadowsocks failed to accept connection from underlying tunnel").Base(err)
	}
	rewindConn := common.NewRewindConn(conn)
	rewindConn.SetBufferSize(1024)
	defer rewindConn.StopBuffering()

	// try to read something from this connection
	buf := [1024]byte{}
	testConn := s.Cipher.StreamConn(rewindConn)
	if _, err := testConn.Read(buf[:]); err != nil {
		// we are under attack
		log.Error(common.NewError("shadowsocks failed to decrypt").Base(err))
		rewindConn.Rewind()
		rewindConn.StopBuffering()
		s.Redirect(&redirector.Redirection{
			RedirectTo:  s.redirAddr,
			InboundConn: rewindConn,
		})
		return nil, common.NewError("invalid aead payload")
	}
	rewindConn.Rewind()
	rewindConn.StopBuffering()

	return &Conn{
		aeadConn: s.Cipher.StreamConn(rewindConn),
		Conn:     conn,
	}, nil
}

```

这段代码定义了两个函数，一个是`AcceptPacket`，另一个是`Close`。

`AcceptPacket`函数接收一个`tunnel.Tunnel`类型的参数，代表一个通过隧道传输的数据包。该函数 panics（抛出）并返回一个错误类型的变量。这里，函数处理失败的情况，并没有做具体的错误处理，需要用户在调用时进行处理。

`Close`函数接收一个`error`类型的参数，代表一个错误类型的变量。函数关闭服务器连接，将其返回。

接着，代码定义了一个名为`NewServer`的函数，该函数接收一个`tunnel.Server`类型的参数，并返回一个`Server`类型的指针。函数的实现看起来像是创建一个服务器实例，但是没有做具体的处理，需要用户在使用时进行处理。

`NewServer`函数的实现主要分为以下几个步骤：

1. 根据配置文件加载服务器配置信息。
2. 选择一个加密算法，并创建一个加密器。
3. 选择一个目标服务器，创建一个指向该服务器的`tunnel.Server`类型的变量。
4. 创建一个用于存储服务器地址和端口的字段。
5. 创建一个指向加密器的`tunnel.Server`类型的变量。
6. 创建一个用于存储目标服务器地址和端口的字段。
7. 创建一个用于存储服务器实例的字段。
8. 将服务器实例的加密器设置为加密器，并将目标服务器地址和端口设置为字段中存储的目标服务器地址和端口。
9. 返回服务器实例的指针。


```go
func (s *Server) AcceptPacket(t tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

func (s *Server) Close() error {
	return s.underlay.Close()
}

func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	cipher, err := core.PickCipher(cfg.Shadowsocks.Method, nil, cfg.Shadowsocks.Password)
	if err != nil {
		return nil, common.NewError("invalid shadowsocks cipher").Base(err)
	}
	if cfg.RemoteHost == "" {
		return nil, common.NewError("invalid shadowsocks redirection address")
	}
	if cfg.RemotePort == 0 {
		return nil, common.NewError("invalid shadowsocks redirection port")
	}
	log.Debug("shadowsocks client created")
	return &Server{
		underlay:   underlay,
		Cipher:     cipher,
		Redirector: redirector.NewRedirector(ctx),
		redirAddr:  tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort),
	}, nil
}

```

# `tunnel/shadowsocks/shadowsocks_test.go`

该代码的作用是实现一个名为"shadowsocks"的包，用于在Shadowsocks服务器和客户端之间建立连接并传输数据。

具体来说，该包通过以下方式实现：

1. 导入一些必要的网络功能，包括网络和字符串操作。
2. 导入一些测试相关的包，用于测试。
3. 定义了一个名为"Tunnel"的接口，用于在连接中建立数据隧道。
4. 实现了一个名为"ShadowsocksServer"的类，用于在服务器上接收来自客户端的连接请求并返回数据。
5. 实现了一个名为"ShadowsocksClient"的类，用于在客户端上接收来自服务器的高清数据并传输给服务器。
6. 在"ShadowsocksServer"和"ShadowsocksClient"之间设置一个锁，以保证数据的同步。
7. 在客户端和服务器之间建立连接并保持活动状态，以保证连接不会中断。
8. 通过"Tunnel"接口实现数据传输，包括加密和传输。
9. 在客户端进行测试时，会对一些链接进行处理，以防止恶意攻击。




```go
package shadowsocks

import (
	"context"
	"fmt"
	"net"
	"strconv"
	"strings"
	"sync"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This is a Go test that checks the ability of a client and a server to establish a connection and send data between each other. The client is using the `tcpClient` from the `github.com/stretchr/testify/asserts` package and the server is using the `tcpServer` from the same package.

The `tcpClient` and `tcpServer` packages are designed to be used in Go and arecarried out by the contemporary Go programming language, this example is to check if the connection can be established between the client and the server, if the connection is established then it will print "connect" and if not it will print "not connect"

The `util.GeneratePayload` function generates a random 1024-byte payload and the `common.CheckConn` function checks if the connection is established or not.


```go
func TestShadowsocks(t *testing.T) {
	p, err := strconv.ParseInt(util.HTTPPort, 10, 32)
	common.Must(err)

	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  port,
		RemoteHost: "127.0.0.1",
		RemotePort: port,
	}
	ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
	ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
	tcpClient, err := transport.NewClient(ctx, nil)
	common.Must(err)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	cfg := &Config{
		RemoteHost: "127.0.0.1",
		RemotePort: int(p),
		Shadowsocks: ShadowsocksConfig{
			Enabled:  true,
			Method:   "AES-128-GCM",
			Password: "password",
		},
	}
	ctx = config.WithConfig(ctx, Name, cfg)

	c, err := NewClient(ctx, tcpClient)
	common.Must(err)
	s, err := NewServer(ctx, tcpServer)
	common.Must(err)

	wg := sync.WaitGroup{}
	wg.Add(2)
	var conn1, conn2 net.Conn
	go func() {
		var err error
		conn1, err = c.DialConn(nil, nil)
		common.Must(err)
		conn1.Write(util.GeneratePayload(1024))
		wg.Done()
	}()
	go func() {
		var err error
		conn2, err = s.AcceptConn(nil)
		common.Must(err)
		buf := [1024]byte{}
		conn2.Read(buf[:])
		wg.Done()
	}()
	wg.Wait()
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}

	go func() {
		var err error
		conn2, err = s.AcceptConn(nil)
		if err == nil {
			t.Fail()
		}
	}()

	// test redirection
	conn3, err := tcpClient.DialConn(nil, nil)
	common.Must(err)
	n, err := conn3.Write(util.GeneratePayload(1024))
	common.Must(err)
	fmt.Println("write:", n)
	buf := [1024]byte{}
	n, err = conn3.Read(buf[:])
	common.Must(err)
	fmt.Println("read:", n)
	if !strings.Contains(string(buf[:n]), "Bad Request") {
		t.Fail()
	}
	conn1.Close()
	conn3.Close()
	c.Close()
	s.Close()
}

```

# `tunnel/shadowsocks/tunnel.go`

这段代码定义了一个名为 "shadowsocks" 的包，其中包含一个名为 "Tunnel" 的结构体。


package shadowsocks

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "SHADOWSOCKS"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}


结构体 "Tunnel" 包含一个名为 "Name" 的字段，其值为 "SHADOWSOCKS"。


func (t *Tunnel) Name() string {
	return Name
}


这个结构体有一个 "Name" 字段，类型为 "string"，没有其它方法。

另外，在 "Tunnel" 结构体内部，有一个 "github.com/p4gefau1t/trojan-go/tunnel" 导入，这个库可能用来在暗网中建立连接。


```go
package shadowsocks

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "SHADOWSOCKS"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}

```

这两函数是用来创建和初始化一个Tunnel实例的。Tunnel是一个用于封装隧道客户端和隧道服务器的数据结构。通过这两函数，我们可以创建一个新的Tunnel实例，并设置其客户端和/或服务器。

第一个函数是在客户端创建一个新客户端，第二个函数是在服务器创建一个新服务器。这两个函数都使用了Tunnel.Context上下文来获取和设置创建实例所需的上下文信息，并返回一个新的Tunnel实例和一个错误。

初始化函数告诉我们的注册信息，告诉tunnel.RegisterTunnel函数。


```go
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	return NewClient(ctx, client)
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, server)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/simplesocks/client.go`

这段代码定义了一个名为"simplesocks"的包，其中包括以下几个主要函数和常量：

- `import` 函数：引入了"github.com/p4gefau1t/trojan-go"包，可能用于从该包中导入一些函数或类型。

- `package` 函数：定义了该包的名称。

- `const` 函数：定义了两个名为`Connect`和`Associate`的常量，分别表示tunnel.Command类型的命令。

- `func` 函数：定义了一个名为`Main`的函数，没有参数，返回值类型为int，可能用于主函数。

- `func` 函数：定义了一个名为`Client`的函数，返回值类型为tuple[int, error]*trojan.Client，可能用于客户端与服务器之间的通信。

- `func` 函数：定义了一个名为`Server`的函数，返回值类型为tuple[int, error]*trojan.Server，可能用于服务器与客户端之间的通信。

- `func` 函数：定义了一个名为`Context`的函数，返回值类型为context.Context，可能用于与上下文相关的操作。

- `func` 函数：定义了一个名为`NewContext`的函数，返回值类型为context.Context，可能用于创建一个新的上下文。

- `func` 函数：定义了一个名为`Log`的函数，可能用于记录一些信息。

- `func` 函数：定义了一个名为`Tunnel`的函数，可能用于创建一个隧道对象。

- `func` 函数：定义了一个名为`Trojan`的函数，可能用于创建一个trojan客户端或服务器。

- `func` 函数：定义了一个名为`Connect`的函数，可能用于建立tunnel.Command类型的连接。

- `func` 函数：定义了一个名为`Associate`的函数，可能用于建立tunnel.Command类型的连接。


```go
package simplesocks

import (
	"context"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

const (
	Connect   tunnel.Command = 1
	Associate tunnel.Command = 3
)

```

该代码定义了一个名为 Client 的结构体，其中包含一个名为 underlay 的字段，该字段是一个 tunnel.Client 类型的变量。

该代码还定义了一个名为 DialConn 的函数，该函数接受一个名为 addr 的 tunnel.Address 类型的变量和一个名为 t 的 tunnel.Tunnel 类型的变量。函数内部使用 c.underlay.DialConn 函数来尝试连接到远程服务器，如果连接建立成功，则返回该连接对象，否则返回一个错误对象。

DialConn 函数的参数包括：

- c:a指针变量，包含 c.underlay 结构体。
- addr:nil 指针变量，包含远程服务器地址。
- t:nil 指针变量，包含一个 tunnel.Tunnel 类型的变量，用于连接到远程服务器。

如果 DialConn 函数返回 nil，则表示在连接到远程服务器时出现错误。如果返回的是一个有效的连接对象，则该连接对象包含以下信息：

- conn：一个 tunnel.Conn 类型的变量，表示与远程服务器成功建立连接。
- isOutbound：一个布尔类型的变量，表示该连接是出站连接。
- metadata：一个 tunnel.Metadata 类型的变量，包含连接的元数据。

总结起来，该代码定义了一个名为 Client 的结构体，其中包含一个名为 underlay 的字段，该字段是一个 tunnel.Client 类型的变量。还定义了一个名为 DialConn 的函数，用于尝试连接到远程服务器并返回一个有效的连接对象。


```go
type Client struct {
	underlay tunnel.Client
}

func (c *Client) DialConn(addr *tunnel.Address, t tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.underlay.DialConn(nil, &Tunnel{})
	if err != nil {
		return nil, common.NewError("simplesocks failed to dial using underlying tunnel").Base(err)
	}
	return &Conn{
		Conn:       conn,
		isOutbound: true,
		metadata: &tunnel.Metadata{
			Command: Connect,
			Address: addr,
		},
	}, nil
}

```

该函数名为`DialPacket`，它接受一个名为`c`的指针变量和一个名为`t`的匿名类型参数。函数返回一个名为`tunnel.PacketConn`的类型，它表示一个套接字到数据报文的连通性信息，以及一个名为`error`的类型参数，用于返回调用该函数时发生的错误。

函数的作用是：通过使用一个名为`c`的指针变量，它与一个匿名类型参数`t`，根据需要设置一个名为`tunnel.Metadata`的类型变量，其中包含一个用于发送数据包到远点的指针，一个目标服务器，以及一些其他的设置。

函数首先使用`c.underlay.DialConn`函数构造一个套接字到远程服务器，通过传递一个空指针`nil`和一个代表远程服务器的类型`tunnel.Server`，然后返回该套接字。如果该构造函数产生任何错误，函数将返回一个错误对象，并指出错误类型。

接下来，函数使用`metadata.WriteTo`函数将设置好的元数据写入到已经连接的套接字中。如果写入过程中出现错误，函数将返回一个错误对象，并指出错误类型。

最后，函数创建一个名为`tunnel.PacketConn`的类型，该类型包含一个指向套接字的指针`conn`，以及一个用于连接数据报文到该套接字的指针`packet`。函数返回这个类型，表示构造了一个可以用于数据包传输的套接字。如果调用该函数时没有错误，函数将返回`nil`表示成功，或者通过错误对象返回具体的错误信息。


```go
func (c *Client) DialPacket(t tunnel.Tunnel) (tunnel.PacketConn, error) {
	conn, err := c.underlay.DialConn(nil, &Tunnel{})
	if err != nil {
		return nil, common.NewError("simplesocks failed to dial using underlying tunnel").Base(err)
	}
	metadata := &tunnel.Metadata{
		Command: Associate,
		Address: &tunnel.Address{
			DomainName:  "UDP_CONN",
			AddressType: tunnel.DomainName,
		},
	}
	if err := metadata.WriteTo(conn); err != nil {
		return nil, common.NewError("simplesocks failed to write udp associate").Base(err)
	}
	return &PacketConn{
		PacketConn: trojan.PacketConn{
			Conn: conn,
		},
	}, nil
}

```

这两段代码定义了一个名为`Client`的结构体，该结构体代表了一个SimpleRSA clients的客户端。这个客户端在关闭时会通知所有连通的简签客户端进行关闭。同时，这两段代码还定义了两个函数：

1. `func (c *Client) Close() error`，该函数接受一个`Client`类型的参数，并返回一个`error`类型的结果。函数的作用是关闭客户端并返回一个错误。在这个函数中，使用`c.underlay.Close()`方法来关闭客户端。如果关闭过程中出现任何错误，该函数将返回一个`error`。

2. `func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error)`，该函数接收一个`Context`类型的参数和一个`tunnel.Client`类型的参数。函数的作用是在上下文上下文和协程上下文中创建一个`Client`实例，并返回该实例和错误。函数创建的客户端使用提供的`underlay`参数作为客户端底层连接。


```go
func (c *Client) Close() error {
	return c.underlay.Close()
}

func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
	log.Debug("simplesocks client created")
	return &Client{
		underlay: underlay,
	}, nil
}

```

# `tunnel/simplesocks/conn.go`

这段代码定义了一个名为 simplesocks 的包，它包含了用于建立 SimplexSocks 连接的代码。

首先，它导入了三个必要的库：

1. bytes：用于字节切片操作
2. trojan.io：用于创建和处理 Trojan 对象
3. trojan.io/trojan：用于管理隧道连接

接下来，定义了一个名为 Conn 的结构体，它代表了 SimplexSocks 连接的元数据和状态。

这个结构体包含以下字段：

1. tunnel.Conn：代表连接到服务器或客户端的隧道协程。
2. metadata：代表连接的元数据，包括服务器地址、端口号、用户名、密码等信息。
3. isOutbound：代表是否是从服务器到客户端的连接。
4. headerWritten：代表是否已经写入了 HTTP 头部，这样就可以在收到数据时立即进行解码。

最后，定义了 Trojan 连接的创建函数和一些方法，包括一些常用的功能，如连接配置、隧道创建、数据接收等。


```go
package simplesocks

import (
	"bytes"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

// Conn is a simplesocks connection
type Conn struct {
	tunnel.Conn
	metadata      *tunnel.Metadata
	isOutbound    bool
	headerWritten bool
}

```

这段代码定义了两个函数，一个是`func (c *Conn) Metadata() *tunnel.Metadata {
	return c.metadata
}`，返回一个指向`tunnel.Metadata`类型的指针，这个类型在接下来的代码中被使用过。

另一个是`func (c *Conn) Write(payload []byte) (int, error)`，用于向连接的下一跳发送数据。函数有两个参数，一个是接收者`c.Conn`，第二个是数据要发送的`payload`。

函数实现如下：

1. 如果`c`是出行的`true`并且`c.metadata`缓冲区未填充，则创建一个缓冲区`buf`，将`c.metadata`的内容写入并加上`payload`的长度，然后将`buf`的内容写入到接收者`c.Conn`的写入缓冲区中。如果写入失败，返回一个错误。如果写入成功，则将`c.metadata`的内容设置为`true`，表明已经写入了`simplesocks`的头部，使得`c.Write`函数可以继续使用。

2. 如果`c`是出行的`false`或者已经发送了`simplesocks`的头部，则直接调用`c.Write`函数，将`payload`的长度作为参数传入，并将结果返回。如果`c.Write`函数在这个过程中出现错误，则返回一个错误。


```go
func (c *Conn) Metadata() *tunnel.Metadata {
	return c.metadata
}

func (c *Conn) Write(payload []byte) (int, error) {
	if c.isOutbound && !c.headerWritten {
		buf := bytes.NewBuffer(make([]byte, 0, 4096))
		c.metadata.WriteTo(buf)
		buf.Write(payload)
		_, err := c.Conn.Write(buf.Bytes())
		if err != nil {
			return 0, common.NewError("failed to write simplesocks header").Base(err)
		}
		c.headerWritten = true
		return len(payload), nil
	}
	return c.Conn.Write(payload)
}

```

这段代码定义了一个名为PacketConn的结构体，它继承自trojan.PacketConn。PacketConn用于实现通过网络发送数据。

具体来说，这段代码主要实现了以下功能：

1. 定义了一个PacketConn类型，该类型包含一个PacketConn结构体。
2. 实现了一个PacketConn的结构体，该结构体包含一个名为trojan.PacketConn的类型字段。
3. 在PacketConn结构体的初始化函数中，调用了trojan.PacketConn的构造函数，从而实现了PacketConn的初始化。
4. 实现了数据发送功能，通过成员函数sendData可以将数据发送到目标端口。
5. 实现了数据接收功能，通过成员函数receiveData可以接收来自目标端口的数据。

通过这些功能，这段代码可以实现一个简单的网络数据传输，例如在双方之间进行数据传输、发送文件等操作。


```go
// PacketConn is a simplesocks packet connection
// The header syntax is the same as trojan's
type PacketConn struct {
	trojan.PacketConn
}

```

# `tunnel/simplesocks/server.go`

该代码定义了一个名为 "simplesocks" 的包，其中包含了一个名为 "Server" 的结构体，它表示一个 simpleocks 服务器。

该结构体实现了一个 "Server" 类型的服务，可以处理连接、数据包和消息，具体实现如下：

1. 定义了一个 "underlay" 字段，该字段是一个 tunnel.Server 类型，代表该服务器使用的隧道技术的服务器。

2. 定义了一个 "connChan" 字段，代表该服务器连接的通道。

3. 定义了一个 "packetChan" 字段，代表该服务器接收的数据包的通道。

4. 定义了一个 "ctx" 字段，代表上下文信息，该字段在关闭服务器时被使用。

5. 定义了一个 "cancel" 字段，代表取消函数，该字段在取消连接时被使用。

6. 在构造函数中，初始化了上述字段，并返回了一个 instance of "Server" 类型的变量，该变量表示该服务器的实例。

该代码提供了一个 simpleocks 服务器的核心部分，可以用于与客户端建立连接，接收和发送数据包。


```go
package simplesocks

import (
	"context"
	"fmt"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

// Server is a simplesocks server
type Server struct {
	underlay   tunnel.Server
	connChan   chan tunnel.Conn
	packetChan chan tunnel.PacketConn
	ctx        context.Context
	cancel     context.CancelFunc
}

```

这段代码是双向日月亭(Two-Sunny-Tsentennen)服务器的相关功能实现。下面是具体的解释：

1. `func (s *Server) Close() error` 函数的作用是关闭服务器并返回结果。具体来说，这个函数执行以下操作：

  - `s.cancel()`: 取消已经连接的客户端的会话。
  - `s.underlay.Close()`: 关闭服务器底层的 WebSocket 服务器。

  如果执行以上操作时出现任何错误，函数将返回一个非空错误对象。

2. `func (s *Server) acceptLoop()` 函数的作用是等待客户端连接并返回结果。具体来说，这个函数会无限循环地等待客户端连接，然后对连接的客户端执行以下操作：

  - `s.underlay.AcceptConn(&Tunnel{})`: 接受一个 WebSocket 连接并将其传递给 `s.underlay`。
  - `metadata := new(tunnel.Metadata)`: 创建一个 `tunnel.Metadata` 对象。
  - `err := metadata.ReadFrom(conn)`: 从当前连接的客户端读取元数据。
  - `switch metadata.Command {`: 检查客户端发送的命令。
  - `case Connect:`: 当客户端发送 `Connect` 命令时，执行以下操作：
  - `s.connChan <- &Conn{metadata: metadata,Conn: conn}`: 将客户端的元数据和当前连接的 `conn` 信息传递给 `s.connChan`。
  - `case Associate:`: 当客户端发送 `Associate` 命令时，执行以下操作：
  - `s.packetChan <- &PacketConn{conn: conn, PacketFunc: packetfunc}`: 将客户端的元数据和 `packetfunc` 传递给 `s.packetChan`。
  - `default:`: 当客户端发送不是 `Connect` 或 `Associate` 命令时，执行以下操作：
  - `log.Error(common.NewError(fmt.Sprintf("simplesocks unknown command %d", metadata.Command)))`: 输出错误消息，并关闭客户端连接。
  - `conn.Close()`: 关闭当前连接。

3. `func (s *Server) tunnelUnderlay(tunnel *tunnel.Tunnel) error` 函数的作用是在 `s.underlay` 和 `tunnel` 之间建立连接。具体来说，这个函数将尝试使用 `tunnel` 中的 WebSocket 服务器与 `s.underlay` 中的 WebSocket 服务器进行通信。

4. `func (s *Server) cmdloop()` 函数是一个命令循环，它将无限循环地等待客户端连接并接收来自客户端的命令。具体来说，这个函数会等待以下客户端连接并接收命令：

  - `s.underlay.AcceptConn(&Tunnel{})`: 接受一个 WebSocket 连接并将其传递给 `s.underlay`。
  - `s.underlay.Close()`: 关闭服务器底层的 WebSocket 服务器。
  - `s.connChan <- &Conn{metadata: metadata,Conn: conn}`: 将客户端的元数据和当前连接的 `conn` 信息传递给 `s.connChan`。
  - `s.packetChan <- &PacketConn{conn: conn, PacketFunc: packetfunc}`: 将客户端的元数据和 `packetfunc` 传递给 `s.packetChan`。
  - `s.underlay.Connect()`: 尝试使用 `s.underlay` 中的 WebSocket 服务器与 `tunnel` 中的 WebSocket 服务器进行通信。

  如果以上操作都成功，函数将返回一个非空错误对象。


```go
func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

func (s *Server) acceptLoop() {
	for {
		conn, err := s.underlay.AcceptConn(&Tunnel{})
		if err != nil {
			log.Error(common.NewError("simplesocks failed to accept connection from underlying tunnel").Base(err))
			select {
			case <-s.ctx.Done():
				return
			default:
			}
			continue
		}
		metadata := new(tunnel.Metadata)
		if err := metadata.ReadFrom(conn); err != nil {
			log.Error(common.NewError("simplesocks server faield to read header").Base(err))
			conn.Close()
			continue
		}
		switch metadata.Command {
		case Connect:
			s.connChan <- &Conn{
				metadata: metadata,
				Conn:     conn,
			}
		case Associate:
			s.packetChan <- &PacketConn{
				PacketConn: trojan.PacketConn{
					Conn: conn,
				},
			}
		default:
			log.Error(common.NewError(fmt.Sprintf("simplesocks unknown command %d", metadata.Command)))
			conn.Close()
		}
	}
}

```

这两函数是 Simplesocks 服务器的高可用机制，用来处理客户端与服务器的连接和数据接收。

1. AcceptConn 函数接受一个 Tunnel 类型的参数，代表一个客户端连接，函数内部使用 select 语句监听客户端连接通道，当接收到客户端连接时，函数返回客户端的套接字（conn）和 nil 错误。当客户端连接关闭时，函数返回 nil 错误。

2. AcceptPacket 函数与 AcceptConn 类似，但使用的套接字类型是 Tunnel 类型的 Packet 类型，代表一个数据接收，函数内部使用 select 语句监听数据接收通道，当接收到数据时，函数返回对应的 Packet 类型客户端的套接字和 nil 错误。当客户端关闭或数据接收失败时，函数返回 nil 错误。

作用：这两函数实现了客户端与服务器之间的连接和数据接收，可以在客户端与服务器断开连接后，继续监听客户端的连接，接收之前发送的数据，确保客户端与服务器的数据交互正常进行。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	select {
	case conn := <-s.connChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("simplesocks server closed")
	}
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	select {
	case packetConn := <-s.packetChan:
		return packetConn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("simplesocks server closed")
	}
}

```

这段代码定义了一个名为NewServer的函数，它接受一个上下文上下文和一个隧道服务器作为参数，并返回一个服务器实例和错误。

具体来说，这段代码创建了一个服务器实例，该实例包含一个隧道服务器和一个上下文上下文。上下文上下文是一个 cancel 事件，用于在服务器接受连接时取消上下文上下文。服务器实例还包含一个连接通道和一个数据通道，用于接收和发送数据。

函数的实现还创建了一个循环，该循环在每个连接上一直运行，直到服务器接收到连接请求。当服务器接收到连接请求时，函数将发送一个新连接请求到服务器，然后继续等待下一个连接请求。

最后，函数还使用了一个简单的日志记录，用于在创建服务器时记录一条日志消息。


```go
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		underlay:   underlay,
		ctx:        ctx,
		connChan:   make(chan tunnel.Conn, 32),
		packetChan: make(chan tunnel.PacketConn, 32),
		cancel:     cancel,
	}
	go server.acceptLoop()
	log.Debug("simplesocks server created")
	return server, nil
}

```

# `tunnel/simplesocks/simplesocks_test.go`

该代码的作用是定义一个名为 "simplesocks" 的 packages。它从两个依赖库开始导入： "github.com/p4gefau1t/trojan-go" 和 "github.com/stretchr/testify/assert"。然后，它定义了一个名为 "simplesocks" 的包。在定义 "simplesocks" 包的同时，它还定义了三个函数： "fmt"、"testing" 和 "util"。接下来，我们通过分析函数的使用来进一步了解 "simplesocks" 包的作用。

"fmt" 函数用于输出一个格式化字符串。在这个例子中，它没有做任何特别的处理，只是简单地将一个字符串输出。

"testing" 函数用于提供测试函数的输入和预期的输出。这个函数可能被用于编写测试用例，确保 "simplesocks" 包正常工作。

"util" 函数用于提供一些通用的工具函数。这些函数可能包括与 "simplesocks" 包相关的函数，用于帮助进行测试或设置环境等。

然后，它通过导入一些外部库来导入 "github.com/p4gefau1t/trojan-go" 和 "github.com/stretchr/testify/assert" 库。这些库可能被用于在 "simplesocks" 包中进行更具体的测试。


```go
package simplesocks

import (
	"context"
	"fmt"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This is a Go program that creates a TCP server that listens for incoming connections and sends a response packet to the client for a given input packet. The program uses the `transport` package to create the TCP server and the `net` package to create the TCP connection to the server.

The program first creates a TCP server by creating a new TransportContext for the server, which is then used to create a TCP server using the `transport.NewServer` function. This server is then bound to a local address and a timeout of 10 seconds using the `transport.Bind` function.

The program then creates a TCP client and a TCP server and uses the `transport.Dial` and `transport.Listen` functions to create a new connection to the server and start listening for incoming connections, respectively.

The program then enters a loop that listens for incoming connections. When a connection is accepted, the program reads the incoming data using the `transport.Read` function and sends a response packet using the `transport.Write` function. The response packet is sent to the client using the `transport.Connect` function.

The program uses the `util.GeneratePayload` function to generate a random payload for the response packet, and the `util.Check` function to check if the connection is successful. If the connection is successful, the program prints the received data to the console using the `fmt.Println` function.

Finally, the program closes the connection to the server using the `transport.Close` function and closes the connection to the client using the `transport.Close` function.


```go
func TestSimpleSocks(t *testing.T) {
	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  port,
		RemoteHost: "127.0.0.1",
		RemotePort: port,
	}
	ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)
	ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})
	tcpClient, err := transport.NewClient(ctx, nil)
	common.Must(err)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	c, err := NewClient(ctx, tcpClient)
	common.Must(err)
	s, err := NewServer(ctx, tcpServer)
	common.Must(err)

	conn1, err := c.DialConn(&tunnel.Address{
		DomainName:  "www.baidu.com",
		AddressType: tunnel.DomainName,
		Port:        443,
	}, nil)
	common.Must(err)
	defer conn1.Close()
	conn1.Write(util.GeneratePayload(1024))
	conn2, err := s.AcceptConn(nil)
	common.Must(err)
	defer conn2.Close()
	buf := [1024]byte{}
	common.Must2(conn2.Read(buf[:]))
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}

	packet1, err := c.DialPacket(nil)
	common.Must(err)
	packet1.WriteWithMetadata([]byte("12345678"), &tunnel.Metadata{
		Address: &tunnel.Address{
			DomainName:  "test.com",
			AddressType: tunnel.DomainName,
			Port:        443,
		},
	})
	defer packet1.Close()
	packet2, err := s.AcceptPacket(nil)
	common.Must(err)
	defer packet2.Close()
	_, m, err := packet2.ReadWithMetadata(buf[:])
	common.Must(err)
	fmt.Println(m)

	if !util.CheckPacketOverConn(packet1, packet2) {
		t.Fail()
	}
	s.Close()
	c.Close()
}

```

# `tunnel/simplesocks/tunnel.go`

该代码定义了一个名为 "simplesocks" 的包，其中包含一个名为 "Tunnel" 的类型，以及一个名为 "Tunnel" 的名为 "*Tunnel" 的指针类型。

具体来说，该代码实现了一个抽象类型 "Tunnel"，该类型有一个名为 "Name" 的字段，其值为 "SIMPLESOCKS"。此外，该类型还有一个名为 "*Tunnel" 的指针类型，用于访问 "Tunnel" 类型的实例。

在 "simplesocks" 包的其他定义中，还定义了一个名为 "Tunnel" 的具体类型，该类型包含一个名为 "*Tunnel" 的指针类型，用于访问 "Tunnel" 类型的实例。该具体类型有一个名为 "Name" 的字段，其值为 "SIMPLESOCKS"。

总体来说，该代码定义了一个 "Tunnel" 类型，该类型包含一个名为 "Name" 的字段和一个名为 "*Tunnel" 的指针类型，用于访问 "Tunnel" 类型的实例。


```go
package simplesocks

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "SIMPLESOCKS"

type Tunnel struct{}

func (*Tunnel) Name() string {
	return Name
}

```

这两函数是在定义一个名为Tunnel的接口，并实现其NewServer和NewClient方法的过程。

具体来说，这两函数是在创建一个名为Tunnel的服务器和客户端实例，通过在上下文中使用调用<tunnel.Tunnel：RegisterTunnel>来注册该接口的实现，并在注册成功后返回服务器实例和调用者的上下文上下文。同时，这两函数还负责处理错误，当出现错误时返回一个Error类型的答案。


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

# `tunnel/socks/config.go`

这段代码定义了一个名为 "socks" 的包，其中包含一个名为 "Config" 的结构体类型。这个 Config 结构体类型包含三个字段：local_addr、local_port 和 udp_timeout，分别表示本地主机地址、本地端口和 UDP 超时时间。

该代码使用了一个名为 "github.com/p4gefau1t/trojan-go/config" 的第三方库来解析配置文件。这个库需要先通过引入其所需的配置类型来初始化 config.RegisterConfigCreator 函数。

在 Config 结构体的初始化函数中，使用了一个简单的函数体，它返回了一个实现了 Config 类型接口的对象，并将这个对象作为参数传递给 config.RegisterConfigCreator 函数。这个函数将创建一个 Config 对象，然后将其注册到 config.Register 上下文中，以便在需要时动态加载配置。

最后，该代码导入了 "socks" 包，但没有其他操作，因此它的作用仅限于初始化一个名为 "config" 的配置对象，其中包含了上述三个字段。


```go
package socks

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
	LocalHost  string `json:"local_addr" yaml:"local-addr"`
	LocalPort  int    `json:"local_port" yaml:"local-port"`
	UDPTimeout int    `json:"udp_timeout" yaml:"udp-timeout"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{
			UDPTimeout: 60,
		}
	})
}

```

# `tunnel/socks/conn.go`

这段代码定义了一个名为`socks`的包，其中包括了以下两个主要函数：

1. `createConnection`函数，它接受一个网络连接上下文和一个metadata参数。它创建一个新的`tunnel.Conn`实例，并将`metadata`设置为`tunnel.Metadata`类型的`nil`。它返回一个`tunnel.Conn`实例和`metadata`。

2. `connect`函数，它接收一个网络连接上下文和一个目标URL参数。它使用`createConnection`函数创建一个新的`tunnel.Conn`实例，并使用该实例连接到目标URL。它返回一个`tunnel.Conn`实例。

`socks`包的主要作用是创建和管理网络连接。通过`createConnection`函数，可以创建一个连接并设置它的元数据，而通过`connect`函数，可以连接到指定的目标URL。


```go
package socks

import (
	"context"
	"net"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Conn struct {
	net.Conn
	metadata *tunnel.Metadata
}

```

该函数定义了一个名为`func`的函数接收一个名为`c`的`conn`参数，并返回一个名为`tunnel.Metadata`的类型，该类型包含一个指向`tunnel.Metadata`类型的指针。

函数的实现是为了让`func`可以创建一个可以获取`conn`的`Metadata`信息的函数。

`packetInfo`定义了一个包含`metadata`和`payload`字段的`packetInfo`结构体。

`PacketConn`定义了一个`PacketConn`结构体，该结构体实现了`net.PacketConn`和`net.Addr`接口。该结构体包含一个名为`input`的通道用于接收数据，一个名为`output`的通道用于发送数据，以及一个名为`src`的字段用于指定数据源的地址，一个名为`ctx`的字段用于存储输入和输出 contexts，一个名为`cancel`的函数用于取消输入数据连接。


```go
func (c *Conn) Metadata() *tunnel.Metadata {
	return c.metadata
}

type packetInfo struct {
	metadata *tunnel.Metadata
	payload  []byte
}

type PacketConn struct {
	net.PacketConn
	input  chan *packetInfo
	output chan *packetInfo
	src    net.Addr
	ctx    context.Context
	cancel context.CancelFunc
}

```

这是一段 Go 语言中的函数指针类型，定义了三个名为 `ReadFrom`,`WriteTo` 和 `Close` 的函数，以及一个名为 `WriteWithMetadata` 的函数。

函数 `ReadFrom` 接收一个字节切片 `p` 作为参数，并返回读取到的字节数 `n` 和添加的 IP 地址 `addr` 的 `net.Addr` 结构体，同时还可能有错误 `err` 的参数。函数在调用时会抛出 `panic` 函数，需要进行实现在调用时的心情。

函数 `WriteTo` 接收一个字节切片 `p` 和一个网络地址 `addr` 作为参数，并返回写入的字节数 `n` 和可能的错误 `err` 的参数。函数在调用时会抛出 `panic` 函数，需要进行实现在调用时的心情。

函数 `Close` 是一个没有参数的函数，返回一个关闭错误 `nil` 的参数。函数在调用时会抛出 `close` 函数，需要进行实现在调用时的心情。

函数 `WriteWithMetadata` 接收一个字节切片 `p` 和一个隧道 `Metadata` 类型的参数 `m` 作为参数，并返回写入的字节数 `n` 和可能的错误 `err` 的参数。函数使用了 `select` 语句，如果 `c.output` 缓存了 `m`，则直接返回 `len(p)` 和 `nil`，否则会等待 `c.ctx.Done()` 并返回一个错误。


```go
func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
	panic("implement me")
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
	panic("implement me")
}

func (c *PacketConn) Close() error {
	c.cancel()
	return nil
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
	select {
	case c.output <- &packetInfo{
		metadata: m,
		payload:  p,
	}:
		return len(p), nil
	case <-c.ctx.Done():
		return 0, common.NewError("socks packet conn closed")
	}
}

```

此函数`ReadWithMetadata`接收一个`PacketConn`类型的参数`c`和一个字节切片`p`，并返回一个整数类型的变量`n`，一个指向`Metadata`类型的指针`tunnel.Metadata`以及一个错误类型的变量`error`。

函数的作用如下：

1. 通过调用`c.input`并使用`info.payload`从`PacketConn`中读取数据，并将读取到的数据复制到一个字节切片`p`中。
2. 如果读取过程中遇到错误，将返回一个非零值，否则返回0。
3. 如果`PacketConn`已经关闭，无论是否已经完成读取操作，都会返回一个非零值。

函数的实现非常简单，主要涉及到了字节切片和指针类型的变量。通过`PacketConn`的`Read`方法读取数据，并使用指针类型来访问`Metadata`类型的数据，实现了数据的逐个读取。同时，在函数中使用了一个`select`语句来处理可能出现的错误情况，并使用了`common.NewError`函数来返回错误类型的值。


```go
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
	select {
	case info := <-c.input:
		n := copy(p, info.payload)
		return n, info.metadata, nil
	case <-c.ctx.Done():
		return 0, nil, common.NewError("socks packet conn closed")
	}
}

```

# `tunnel/socks/server.go`

这段代码定义了一个名为 "socks" 的包，它提供了用于创建和管理网络代理的函数。具体来说，这个包通过导入其他包中的函数和变量，实现了以下功能：

1. 定义了一个名为 "bytes" 的常量，它表示一个字节序列。
2. 定义了一个名为 "context" 的常量，它表示一个上下文。
3. 定义了一个名为 "fmt" 的常量，它表示一个格式化字符串。
4. 定义了一个名为 "net" 的常量，它表示一个网络接口。
5. 定义了一个名为 "time" 的常量，它表示一个时间。
6. 定义了一个名为 "github.com/p4gefau1t/trojan-go/common" 的常量，它表示一个公共包的实例。
7. 定义了一个名为 "github.com/p4gefau1t/trojan-go/config" 的常量，它表示一个配置文件读取器实例。
8. 定义了一个名为 "github.com/p4gefau1t/trojan-go/log" 的常量，它表示一个日志记录器实例。
9. 定义了一个名为 "github.com/p4gefau1t/trojan-go/tunnel" 的常量，它表示一个隧道实例。

此外，还定义了一些函数，包括创建一个名为 "Proxy" 的类型，它实现了代理模式；创建一个名为 "Server" 的类型，它实现了服务器模式；以及一些用于设置和取消代理模式的功能。


```go
package socks

import (
	"bytes"
	"context"
	"fmt"
	"io"
	"io/ioutil"
	"net"
	"sync"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码定义了一个名为Server的Server结构体，用于在网络中传输数据。下面是它的主要部分功能说明：

1. 定义了两个名为Connect和Associate的tunnel.Command结构体，它们都接受一个int类型的参数，用于指定在TCP套接字上使用的命令。

2. 定义了一个名为MaxPacketSize的int类型结构体，用于指定可以接受的最大数据包大小。

3. 在Server结构体中定义了三个类型的变量：

	- Server struct中有一个名为connChan的类型为the这些东西的channell类型的变量，用于存储客户端连接的通道。

	- Server struct中有一个名为packetChan的类型为the这些东西的channell类型的变量，用于存储接收到的数据包的通道。

	- Server struct中有一个名为underlay的类型为the这些东西的Server结构体类型的变量，用于在客户端和服务器之间传递数据。

	- Server struct中有一个名为localHost的类型为the这些东西的string类型的变量，用于存储服务器的主机名。

	- Server struct中有一个名为localPort的类型为the这些东西的int类型的变量，用于存储服务器套接字的主机端口号。

	- Server struct中有一个名为timeout的类型为the这些东西的time.Duration类型的变量，用于指定超时的时间。

	- Server struct中有一个名为listenPacketConn的类型为the这些东西的tunnel.Server的类型定义的变量，用于存储接受客户端数据包的Server结构体类型的变量。

	- Server struct中有一个名为mapping的类型为the这些东西的map[string:]*PacketConn的类型定义的变量，用于存储服务器端口号和客户端连接的映射。

	- Server struct中有一个名为mappingLock的类型为the这些东西的sync.RWMutex的类型定义的变量，用于锁定映射字段以防止读写冲突。

	- Server struct中有一个名为ctx的类型为the这些东西的context.Context的类型定义的变量，用于存储当前上下文。

	- Server struct中有一个名为cancel的类型为the这些东西的context.CancelFunc的类型定义的变量，用于存储取消已发送的数据包的回调函数。

	- 在Make sure connChan,packetChan,underlay,localHost,localPort,timeout,listenPacketConn,mapping,mappingLock,ctx,cancel这些都是常量，在代码中直接赋值。


```go
const (
	Connect   tunnel.Command = 1
	Associate tunnel.Command = 3
)

const (
	MaxPacketSize = 1024 * 8
)

type Server struct {
	connChan         chan tunnel.Conn
	packetChan       chan tunnel.PacketConn
	underlay         tunnel.Server
	localHost        string
	localPort        int
	timeout          time.Duration
	listenPacketConn tunnel.PacketConn
	mapping          map[string]*PacketConn
	mappingLock      sync.RWMutex
	ctx              context.Context
	cancel           context.CancelFunc
}

```

这两函数的作用是处理服务器端的连接和数据接收。

`AcceptConn`函数接收一个`socks`服务器的`connChan`通道，然后等待一个连接并返回它。如果服务器连接成功，则返回客户端的`conn`字段，否则返回一个`nil`表示出错。

`AcceptPacket`函数接收一个`socks`服务器的`packetChan`通道，然后等待一个数据包并返回它。如果服务器接收到数据包，则返回客户端的`conn`字段，否则返回一个自定义错误。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	select {
	case conn := <-s.connChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("socks server closed")
	}
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	select {
	case conn := <-s.packetChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("socks server closed")
	}
}

```

此代码定义了两个函数，一个是`Close()`函数，另一个是`handshake()`函数。

`Close()`函数的参数为`s`指针，代表一个`Server`结构体。此函数的作用是取消服务器连接并关闭底层网络传输层。具体实现包括取消订阅服务器连接、关闭底层传输层以及关闭服务器套接字。

`handshake()`函数的参数为`conn`指针，代表一个`net.Conn`结构体。此函数的作用是在客户端与服务器建立连接时进行握手。具体实现包括读取客户端发送的`socks version`字段、读取客户端发送的`NMETHODS`字段、读取客户端发送的命令、关闭底层传输层并关闭服务器套接字。

`handshake()`函数的实现非常复杂，涉及了多个网络库的交互。在此不再详细解释，以免影响解题效率。


```go
func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

func (s *Server) handshake(conn net.Conn) (*Conn, error) {
	version := [1]byte{}
	if _, err := conn.Read(version[:]); err != nil {
		return nil, common.NewError("failed to read socks version").Base(err)
	}
	if version[0] != 5 {
		return nil, common.NewError(fmt.Sprintf("invalid socks version %d", version[0]))
	}
	nmethods := [1]byte{}
	if _, err := conn.Read(nmethods[:]); err != nil {
		return nil, common.NewError("failed to read NMETHODS")
	}
	if _, err := io.CopyN(ioutil.Discard, conn, int64(nmethods[0])); err != nil {
		return nil, common.NewError("socks failed to read methods").Base(err)
	}
	if _, err := conn.Write([]byte{0x5, 0x0}); err != nil {
		return nil, common.NewError("failed to respond auth").Base(err)
	}

	buf := [3]byte{}
	if _, err := conn.Read(buf[:]); err != nil {
		return nil, common.NewError("failed to read command")
	}

	addr := new(tunnel.Address)
	if err := addr.ReadFrom(conn); err != nil {
		return nil, err
	}

	return &Conn{
		metadata: &tunnel.Metadata{
			Command: tunnel.Command(buf[1]),
			Address: addr,
		},
		Conn: conn,
	}, nil
}

```

This is a Go function that handles the input of a UDP packet from a Socks5 proxy server to a Socks5 proxy client. It uses the `github.com/markbates/tunnel` package to handle the隧道建立和数据传输。

The function takes an incoming UDP packet from the Socks5 proxy server, reads the packet data and checks for certain fields such as the metadata field and the destination address. If the packet is not a valid Socks5 packet, or if it does not contain a valid destination address, the function logs an error and returns.

If the packet is a valid Socks5 packet, the function extracts the destination address from the packet data and creates a new Socks5 connection using the Socks5 protocol to establish a new UDP session with the Socks5 proxy client. The function then continues to read the packet data from the input and passes it on to the Socks5 connection.

The function also handle the case where the Socks5 proxy server may not respond to the incoming packet, if that happens, the function will wait for some time and try again. If the Socks5 proxy server does not respond, the connection will be closed by the Socks5 server.

The function is using a client-server architecture, where the Socks5 proxy server is the server and the Socks5 proxy client is the client. The Socks5 proxy server is using a IP:port format and the Socks5 proxy client is using an IP:port format.


```go
func (s *Server) connect(conn net.Conn) error {
	_, err := conn.Write([]byte{0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00})
	return err
}

func (s *Server) associate(conn net.Conn, addr *tunnel.Address) error {
	buf := bytes.NewBuffer([]byte{0x05, 0x00, 0x00})
	common.Must(addr.WriteTo(buf))
	_, err := conn.Write(buf.Bytes())
	return err
}

func (s *Server) packetDispatchLoop() {
	for {
		buf := make([]byte, MaxPacketSize)
		n, src, err := s.listenPacketConn.ReadFrom(buf)
		if err != nil {
			select {
			case <-s.ctx.Done():
				log.Debug("exiting")
				return
			default:
				continue
			}
		}
		log.Debug("socks recv udp packet from", src)
		s.mappingLock.RLock()
		conn, found := s.mapping[src.String()]
		s.mappingLock.RUnlock()
		if !found {
			ctx, cancel := context.WithCancel(s.ctx)
			conn = &PacketConn{
				input:      make(chan *packetInfo, 128),
				output:     make(chan *packetInfo, 128),
				ctx:        ctx,
				cancel:     cancel,
				PacketConn: s.listenPacketConn,
				src:        src,
			}
			go func(conn *PacketConn) {
				defer conn.Close()
				for {
					select {
					case info := <-conn.output:
						buf := bytes.NewBuffer(make([]byte, 0, MaxPacketSize))
						buf.Write([]byte{0, 0, 0}) // RSV, FRAG
						common.Must(info.metadata.Address.WriteTo(buf))
						buf.Write(info.payload)
						_, err := s.listenPacketConn.WriteTo(buf.Bytes(), conn.src)
						if err != nil {
							log.Error("socks failed to respond packet to", src)
							return
						}
						log.Debug("socks respond udp packet to", src, "metadata", info.metadata)
					case <-time.After(time.Second * 5):
						log.Info("socks udp session timeout, closed")
						s.mappingLock.Lock()
						delete(s.mapping, src.String())
						s.mappingLock.Unlock()
						return
					case <-conn.ctx.Done():
						log.Info("socks udp session closed")
						return
					}
				}
			}(conn)

			s.mappingLock.Lock()
			s.mapping[src.String()] = conn
			s.mappingLock.Unlock()

			s.packetChan <- conn
			log.Info("socks new udp session from", src)
		}
		r := bytes.NewBuffer(buf[3:n])
		address := new(tunnel.Address)
		if err := address.ReadFrom(r); err != nil {
			log.Error(common.NewError("socks failed to parse incoming packet").Base(err))
			continue
		}
		payload := make([]byte, MaxPacketSize)
		length, _ := r.Read(payload)
		select {
		case conn.input <- &packetInfo{
			metadata: &tunnel.Metadata{
				Address: address,
			},
			payload: payload[:length],
		}:
		default:
			log.Warn("socks udp queue full")
		}
	}
}

```

This is ago program that performs an iterative loop to establish a connection with a TCP connection to a SOCKS server. It uses the `socksdb` and `socks为人民` libraries to handle the communication with the server and the `net` package to handle the TCP connection.

The program is structured as follows:

1. The program defines a`loop()`函数， which performs an iterative loop to connect to a TCP connection.
2. The loop() function uses the`for` loop to iterate over all the connections that are available on the server.
3. Inside the loop() function, the program calls the`AcceptConn()` method of the`socksunderlay` object to accept a new connection.
4. If the connection is accepted successfully, the program calls the `handshake()` method of the`socksunderlay` object to establish a connection with the client.
5. If there is an error during the handshake, the program logs it and returns, returning the connection to the server.
6. If the connection is established successfully, the program reads data from the client and sends it to the server using the`connect()` method of the`socksunderlay` object.
7. If there is an error during the`connect()` method, the program logs it and returns, closing the connection.
8. If the connection is closed, the program closes all the connections that are currently open.

The program also includes a helper function`connect()` which connects to a server using the`tcpconnect` method provided by the`net` package. This function takes a connection address and a timeout value as arguments and returns a connection object that can be used to send and receive data.


```go
func (s *Server) acceptLoop() {
	for {
		conn, err := s.underlay.AcceptConn(&Tunnel{})
		if err != nil {
			log.Error(common.NewError("socks accept err").Base(err))
			return
		}
		go func(conn net.Conn) {
			newConn, err := s.handshake(conn)
			if err != nil {
				log.Error(common.NewError("socks failed to handshake with client").Base(err))
				return
			}
			log.Info("socks connection from", conn.RemoteAddr(), "metadata", newConn.metadata.String())
			switch newConn.metadata.Command {
			case Connect:
				if err := s.connect(newConn); err != nil {
					log.Error(common.NewError("socks failed to respond CONNECT").Base(err))
					newConn.Close()
					return
				}
				s.connChan <- newConn
				return
			case Associate:
				defer newConn.Close()
				associateAddr := tunnel.NewAddressFromHostPort("udp", s.localHost, s.localPort)
				if err := s.associate(newConn, associateAddr); err != nil {
					log.Error(common.NewError("socks failed to respond to associate request").Base(err))
					return
				}
				buf := [16]byte{}
				newConn.Read(buf[:])
				log.Debug("socks udp session ends")
			default:
				log.Error(common.NewError(fmt.Sprintf("unknown socks command %d", newConn.metadata.Command)))
				newConn.Close()
			}
		}(conn)
	}
}

```

这段代码定义了一个名为NewServer的函数，该函数创建一个基于 SOCKS5 连接的服务器。函数接受一个underlay服务器作为参数，并返回一个表示服务器的SocksServer对象和可能的错误。

函数首先从配置上下文中读取服务器配置，然后使用AcceptPacket函数尝试从底层服务器接收数据包。如果遇到错误，函数将返回一个错误信息。

接下来，函数创建一个Server对象，该对象包含服务器的底层连接、取消信号、连接 channel、数据包 channel和一些与服务器连接相关的配置参数。

最后，函数启动服务器接受数据包的循环和数据包处理循环，并将一些关键配置参数映射到一个名为PacketConn的结构中。

函数使用go server.acceptLoop()和go server.packetDispatchLoop()来运行服务器的主循环，并在两个循环中处理接收到的数据包。

函数的目的是创建一个基于 SOCKS5 连接的服务器，以便在底层服务器不可用时提供数据传输服务。


```go
// NewServer create a socks server
func NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	listenPacketConn, err := underlay.AcceptPacket(&Tunnel{})
	if err != nil {
		return nil, common.NewError("socks failed to listen packet from underlying server")
	}
	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		underlay:         underlay,
		ctx:              ctx,
		cancel:           cancel,
		connChan:         make(chan tunnel.Conn, 32),
		packetChan:       make(chan tunnel.PacketConn, 32),
		localHost:        cfg.LocalHost,
		localPort:        cfg.LocalPort,
		timeout:          time.Duration(cfg.UDPTimeout) * time.Second,
		listenPacketConn: listenPacketConn,
		mapping:          make(map[string]*PacketConn),
	}
	go server.acceptLoop()
	go server.packetDispatchLoop()
	log.Debug("socks server created")
	return server, nil
}

```