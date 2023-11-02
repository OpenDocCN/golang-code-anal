# trojan-go源码解析 10

# `tunnel/mux/config.go`

这段代码定义了一个名为 `MuxConfig` 的结构体，它表示一个支持 Mux API 的配置对象。这个配置对象包含三个字段：`enabled`、`idle_timeout` 和 `concurrency`。其中，`enabled` 是布尔值，表示是否启用 Mux API。`idle_timeout` 是整数，表示在空闲时应用户空闲一段时间后自动关闭连接的秒数。`concurrency` 是整数，表示允许同时创建的最大连接数。

这个配置对象有一个父类型 `Config`，它包含一个 `Mux` 字段，该字段表示一个指向 `MuxConfig` 类型的引用。这个 `Mux` 字段包含一个指向 `MuxConfig` 类型的指针，它包含了上面定义的三个字段。

最后，在 `init()` 函数中，使用 `config.RegisterConfigCreator()` 方法将 `MuxConfig` 类型的配置创建者注册到 `config` 上下文中。这个配置创建者函数将上面定义的 `MuxConfig` 类型中的字段映射到 `Mux` 字段中，并设置其 `enabled` 字段的值为 `false`，`idle_timeout` 字段的值为 `30`，`concurrency` 字段的值为 `8`。


```go
package mux

import "github.com/p4gefau1t/trojan-go/config"

type MuxConfig struct {
	Enabled     bool `json:"enabled" yaml:"enabled"`
	IdleTimeout int  `json:"idle_timeout" yaml:"idle-timeout"`
	Concurrency int  `json:"concurrency" yaml:"concurrency"`
}

type Config struct {
	Mux MuxConfig `json:"mux" yaml:"mux"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{
			Mux: MuxConfig{
				Enabled:     false,
				IdleTimeout: 30,
				Concurrency: 8,
			},
		}
	})
}

```

# `tunnel/mux/conn.go`

这段代码定义了一个名为`stickyConn`的结构体，它属于`mux`包。这个结构体包含两个channel，一个是`synQueue`，另一个是`finQueue`。这两个channel分别用于传输同步信息和最终消息。

`stickyConn`结构体中的`tunnel.Conn`使用Go标准库中的`net`类型，它代表一个网络连接。`io.IORef`类型代表一个可以操作I/O的指针，这使得`stickyConn`可以接收和发送字节流。

`synQueue`和`finQueue`都使用了Go标准库中的`sync.Queue`类型。`synQueue`用于传输同步信息，`finQueue`用于传输最终消息。这些信息是在网络中传输的，所以它们需要一个参照物来确保正确传输。

整个结构体通过`tunnel.Connect`方法与外部服务建立连接，将连接的同步信息发送给服务器，并将最终消息发送给客户端。


```go
package mux

import (
	"io"
	"math/rand"

	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type stickyConn struct {
	tunnel.Conn
	synQueue chan []byte
	finQueue chan []byte
}

```

此代码定义了一个名为`func`的函数，接收一个名为`c`的指针变量和一个字节数组`p`作为参数。

函数的作用是确保一个名为`stickyConn`的客户端与一个名为`payload`的推送服务器保持连接。为了实现这一目标，函数首先创建一个长度为`len(p)+16`的字节数组`buf`，然后将`p`字节数组与`buf`数组长度相等的字节添加到数组中。

接下来，函数通过一个循环来检测客户端是否接收到服务器发送的消息。如果是，函数将读取消息并将其存储在`buf`数组的结点中。如果未接收到消息，则函数跳转到`stick1`标签，然后将`p`字节数组与`buf`数组长度相等的字节添加到`stickyConn`的`finQueue`中。

在`finQueue`中，函数通过一个循环来检测是否接收到服务器发送的消息。如果是，函数将读取消息并将其存储在`buf`数组的结点中。如果未接收到消息，则函数跳转到`stick2`标签，然后继续循环直到收到一条服务器发送的消息。

总之，此函数的作用是确保客户端与服务器保持连接，并在客户端有新消息时通知它。


```go
func (c *stickyConn) stickToPayload(p []byte) []byte {
	buf := make([]byte, 0, len(p)+16)
	for {
		select {
		case header := <-c.synQueue:
			buf = append(buf, header...)
		default:
			goto stick1
		}
	}
stick1:
	buf = append(buf, p...)
	for {
		select {
		case header := <-c.finQueue:
			buf = append(buf, header...)
		default:
			goto stick2
		}
	}
```

这段代码是两个名为"stickyConn"和"write"的函数的组合。它们的作用是：

1. "stickyConn.Close()"函数的作用是关闭与 stickyConn 连接的通道，并返回通道关闭时产生的错误。

2. "write.Write(p)"函数的作用是向名为 "write" 的函数写入数据（此处以字节切片 "p" 为例）。首先，根据传入的数据长度，在通道缓冲区中获取对应长度的字节。然后，使用 "writeTo" 函数将数据写入到 "write" 函数的缓冲区中。如果数据长度为 8，则表示实现了 8 字节协议头，会向 "synQueue" 和 "finQueue" 中添加数据。如果数据长度不为 8，则按照需要写入数据。8 字节协议头中的数据类型分别为：

 - 1. "cmdSYN"
 - 2. "cmdFIN"

这两个函数的实现主要依赖于 stickyConn 提供的一个名为 "stickToPayload" 的函数，以及一个名为 "writeTo" 的内部函数。


```go
stick2:
	return buf
}

func (c *stickyConn) Close() error {
	const maxPaddingLength = 512
	padding := [maxPaddingLength + 8]byte{'A', 'B', 'C', 'D', 'E', 'F'} // for debugging
	buf := c.stickToPayload(nil)
	c.Write(append(buf, padding[:rand.Intn(maxPaddingLength)]...))
	return c.Conn.Close()
}

func (c *stickyConn) Write(p []byte) (int, error) {
	if len(p) == 8 {
		if p[0] == 1 || p[0] == 2 { // smux 8 bytes header
			switch p[1] {
			// THE CONTENT OF THE BUFFER MIGHT CHANGE
			// NEVER STORE THE POINTER TO HEADER, COPY THE HEADER INSTEAD
			case 0:
				// cmdSYN
				header := make([]byte, 8)
				copy(header, p)
				c.synQueue <- header
				return 8, nil
			case 1:
				// cmdFIN
				header := make([]byte, 8)
				copy(header, p)
				c.finQueue <- header
				return 8, nil
			}
		} else {
			log.Debug("other 8 bytes header")
		}
	}
	_, err := c.Conn.Write(c.stickToPayload(p))
	return len(p), err
}

```

该代码定义了一个名为`newStickyConn`的函数，它接受一个名为`conn`的TCP套接字参数。函数返回一个指向`stickyConn`结构体的指针。

`stickyConn`结构体包含一个TCP套接字连接、一个发送Syn消息的通道和一个发送Fin消息的通道。

函数的实现主要分为两步：

1. 创建一个名为`make(chan []byte, 128)`的通道，用于接收来自`conn`套接字的Syn消息。
2. 创建一个名为`make(chan []byte, 128)`的通道，用于接收来自`conn`套接字的Fin消息。

这两步都通过`make`函数创建一个大小为128字节（或128个字节）的通道，当有新的消息产生时，通道将收到消息并将其加入到相应的通道中。

函数的最后一部分，将创建的`stickyConn`结构体返回，以便后续使用。


```go
func newStickyConn(conn tunnel.Conn) *stickyConn {
	return &stickyConn{
		Conn:     conn,
		synQueue: make(chan []byte, 128),
		finQueue: make(chan []byte, 128),
	}
}

type Conn struct {
	rwc io.ReadWriteCloser
	tunnel.Conn
}

func (c *Conn) Read(p []byte) (int, error) {
	return c.rwc.Read(p)
}

```

这两段代码是 Go 语言中连接套接字（也就是 HTTP 连接）的相关操作。

第一段代码 `func (c *Conn) Write(p []byte) (int, error)` 表示将一个 byte 数组 `p` 写入到连接套接字 `c` 的第二个读写通道（也就是 `rwc` 通道）中，并返回写入成功和错误的信息。具体实现方式如下：

1. `c.rwc.Write` 是一个内部方法，它将 `c` 的第二个读写通道 `rwc` 中的数据写入到缓冲区 `p` 中。
2. `(c.rwc.Write)` 是一个接受一个字节数组 `p` 的函数，它返回一个整数类型的写入结果和一个错误信息。如果写入成功，则返回写入字节数组的大小 `p.len`，否则返回一个错误。

第二段代码 `func (c *Conn) Close() error` 表示关闭连接套接字 `c`，并返回一个错误信息。具体实现方式如下：

1. `c.rwc.Close` 是一个内部方法，它关闭连接套接字 `c` 的第二个读写通道 `rwc`，并返回一个错误信息。
2. `(c.rwc.Close)` 是一个接受一个错误信息的函数，它关闭连接套接字 `c`，并返回一个错误信息。


```go
func (c *Conn) Write(p []byte) (int, error) {
	return c.rwc.Write(p)
}

func (c *Conn) Close() error {
	return c.rwc.Close()
}

```

# `tunnel/mux/mux_test.go`

This is a Go program that sets up a超高性能网络信道（UPN），主要用于在不同实体之间进行数据传输。它的目的是在数据传输之前创建一个TCP连接，然后使用数据传输来建立一个TCP二进制数据流，以便在两个实体之间传输数据。它支持并发8个连接，可以设置一个超时为60秒的闲着超时。

首先，它会配置一些网络参数，如TCP连接的本地和远程主机以及端口号，然后使用transport.名称和freedom.名称配置传输协议和服务器。接下来，它通过transport.NewClient和transport.NewServer构建TCP客户端和服务器。

然后，它设置一个TCP信道，通过使用muxTunnel.新客户端和muxTunnel.新服务端来设置网络传输。接下来，它会通过一个TCP客户端和一个TCP服务器来设置UPN的传输。最后，它会通过一个TCP客户端和一个TCP服务器来设置UPN的传输。

如果您有疑问，或者需要更详细的解释，请提供更多信息，我会尽力回答您的问题。


```go
package mux

import (
	"context"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

func TestMux(t *testing.T) {
	muxCfg := &Config{
		Mux: MuxConfig{
			Enabled:     true,
			Concurrency: 8,
			IdleTimeout: 60,
		},
	}
	ctx := config.WithConfig(context.Background(), Name, muxCfg)

	port := common.PickPort("tcp", "127.0.0.1")
	transportConfig := &transport.Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  port,
		RemoteHost: "127.0.0.1",
		RemotePort: port,
	}
	ctx = config.WithConfig(ctx, transport.Name, transportConfig)
	ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})

	tcpClient, err := transport.NewClient(ctx, nil)
	common.Must(err)
	tcpServer, err := transport.NewServer(ctx, nil)
	common.Must(err)

	common.Must(err)

	muxTunnel := Tunnel{}
	muxClient, _ := muxTunnel.NewClient(ctx, tcpClient)
	muxServer, _ := muxTunnel.NewServer(ctx, tcpServer)

	conn1, err := muxClient.DialConn(nil, nil)
	common.Must2(conn1.Write(util.GeneratePayload(1024)))
	common.Must(err)
	buf := [1024]byte{}
	conn2, err := muxServer.AcceptConn(nil)
	common.Must(err)
	common.Must2(conn2.Read(buf[:]))
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}
	conn1.Close()
	conn2.Close()
	muxClient.Close()
	muxServer.Close()
}

```

# `tunnel/mux/server.go`

这段代码定义了一个名为`Server`的服务器结构体，包含了与服务器相关的字段和方法。

该结构体使用了`smux`库，它是Kubernetes中的一个流量管理库，用于创建和管理网络代理，可以在容器中转发网络流量。

该服务器使用了`underlay`作为底层网络代理，`underlay`通过一个`Server`实例来管理网络连接和流量。

该服务器连接到客户端的通道被存储在一个名为`connChan`的通道上，每个客户端连接都是独立的。

该服务器包含了与服务器上下文相关的字段，包括`ctx`表示当前上下文，`cancel`是一个用于取消上下文的函数，该函数在客户端连接断开时被调用。

该服务器通过`trojan-go`库与`trojan-go`隧道进行交互。通过`trojan-go`隧道，可以创建隧道并管理隧道中的流量，可以用于在容器之间传输数据和在容器之间进行通信。


```go
package mux

import (
	"context"

	"github.com/xtaci/smux"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

// Server is a smux server
type Server struct {
	underlay tunnel.Server
	connChan chan tunnel.Conn
	ctx      context.Context
	cancel   context.CancelFunc
}

```

这段代码定义了一个名为`acceptConnWorker`的函数，属于一个名为`Server`的类的实例。

该函数接受一个`conn`参数，然后进行一系列操作，最终返回。下面是具体的实现步骤：

1. 循环等待直到有新的连接到达。

2. 接受一个连接并将其传递给一个名为`smux`的函数处理。

3. 在`smux`函数中，通过调用`AcceptConn`函数尝试接受一个连接。如果接受成功，将创建一个`smuxSession`并返回，这个`smuxSession`与下面将要创建的`smux`函数的`Session`参数重叠，所以将接受到新的连接。

4. 在`smux`函数中，创建一个名为`smuxConfig`的配置实例。

5. 使用`smuxConfig`中的`KeepAliveDisabled`选项设置为`true`。

6. 在`smux`函数中，创建一个名为`smuxSession`的`smux.Session`实例。

7. 在`smux`函数中，创建一个名为`smuxSession`的`smux.Session`实例。

8. 在`smux`函数中，通过调用`AcceptStream`函数尝试获取可用的流。

9. 在`smux`函数中，循环等待直到有新的数据可用。

10. 如果连接成功，将连接发送到`s.connChan`。

11. 如果所有的连接都关闭了，则返回。

12. 循环继续等待新的连接，直到所有的连接都关闭。


```go
func (s *Server) acceptConnWorker() {
	for {
		conn, err := s.underlay.AcceptConn(&Tunnel{})
		if err != nil {
			log.Debug(err)
			select {
			case <-s.ctx.Done():
				return
			default:
			}
			continue
		}
		go func(conn tunnel.Conn) {
			smuxConfig := smux.DefaultConfig()
			// smuxConfig.KeepAliveDisabled = true
			smuxSession, err := smux.Server(conn, smuxConfig)
			if err != nil {
				log.Error(err)
				return
			}
			go func(session *smux.Session, conn tunnel.Conn) {
				defer session.Close()
				defer conn.Close()
				for {
					stream, err := session.AcceptStream()
					if err != nil {
						log.Error(err)
						return
					}
					select {
					case s.connChan <- &Conn{
						rwc:  stream,
						Conn: conn,
					}:
					case <-s.ctx.Done():
						log.Debug("exiting")
						return
					}
				}
			}(smuxSession, conn)
		}(conn)
	}
}

```

这是一段使用Go的函数式编程风格实现的服务器端代码。

第一个函数`AcceptConn`接收一个`Server`结构体类型的参数`s`和一个`Tunnel`结构体类型的参数`tunnel`，并返回一个`Tunnel`结构体类型的结果`conn`和一个`error`类型的结果`e`。函数使用了`select`语句来处理客户端连接请求。如果连接成功，则返回客户端的`conn`，否则返回一个`error`。

第二个函数`AcceptPacket`同样接收一个`Server`结构体类型的参数`s`和一个`Tunnel`结构体类型的参数`tunnel`，并返回一个`Tunnel`结构体类型的结果`conn`和一个`error`类型的结果`e`。这个函数没有做任何特殊处理，只是简单的返回了一个`error`。

第三个函数`Close`接收一个`Server`结构体类型的参数`s`，并返回一个`error`类型的结果`e`。函数实现了关闭服务器端口的功能，通过调用`underlay.Close()`方法来实现。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	select {
	case conn := <-s.connChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("mux server closed")
	}
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

func (s *Server) Close() error {
	s.cancel()
	return s.underlay.Close()
}

```

这段代码定义了一个名为NewServer的函数，它接受一个上下文上下文和一个新的服务器作为参数，并返回一个Server类型的指针和一个错误。

具体来说，这段代码创建了一个名为Server的实例，该实例包含一个underlay类型的服务器，该服务器连接到指定的上下文上下文，还设置了一个cancel信号以便在需要时取消创建服务器。另外，还创建了一个名为connChan的通道，用于在服务器接受连接时接收来自下层网络的连接。最后，在函数的返回点上，将创建好的服务器返回，如果没有错误则不会返回错误。

总的来说，这段代码定义了一个用于创建服务器并将其连接到上下文上下文的函数，该函数可以用于开始接受来自下层网络的连接。


```go
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		underlay: underlay,
		ctx:      ctx,
		cancel:   cancel,
		connChan: make(chan tunnel.Conn, 32),
	}
	go server.acceptConnWorker()
	log.Debug("mux server created")
	return server, nil
}

```

# `tunnel/mux/tunnel.go`

这段代码定义了一个名为 "MUX" 的命名上下文，并实现了 "MUX" 包的功能。

具体来说，代码中定义了一个名为 "Tunnel" 的 struct 类型，该类型包含一个名为 "Name" 的字段，其值为 "MUX"。

此外，代码还定义了一个名为 "Tunnel" 的普通函数，该函数的实现为 "(*Tunnel)Name()" 函数，返回参数 "Tunnel" 的 "Name" 字段的值，即 "MUX"。

最后，在 "Tunnel" 函数的定义中，使用了 "github.com/p4gefau1t/trojan-go/tunnel" 导入了一来自 "trojan-go" 库的 "Tunnel" 实例，该实例可能被用于创建和管理网络隧道。


```go
package mux

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "MUX"

type Tunnel struct{}

func (*Tunnel) Name() string {
	return Name
}

```

这两行代码定义了两个名为NewClient和NewServer的函数，它们用于创建隧道客户端和隧道服务器。具体来说，这两行代码实现了一个名为Tunnel的接口，并在该接口上注册了一个名为Tunnel的函数。

在函数内部，这两行代码首先创建一个名为client的Tunnel客户端实例，然后使用返回值类型tunnel.Client指针变量来获取它。接下来，这两行代码会尝试使用NewClient函数创建另一个名为client的Tunnel客户端实例。如果第一个尝试创建失败，则返回一个error类型的变量。

对于第二个函数，这两行代码创建一个名为server的Tunnel服务器实例，然后使用返回值类型tunnel.Server指针变量来获取它。接下来，这两行代码会尝试使用NewServer函数创建另一个名为server的Tunnel服务器实例。如果第一个尝试创建失败，则返回一个error类型的变量。

最后，这两行代码定义了一个名为init的函数，该函数初始化了一切Tunnel相关的函数。


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

# `tunnel/router/client.go`

该代码的作用是定义了一个名为 "router" 的包，其中定义了一些工具函数和类型变量。

具体来说，该包定义了一个 "v2router" 类型，它是一个 v2ray-core 库的路由器实现。

该包还定义了一些函数和类型变量，包括：

- "import": 导入了一些来自 "net"、"regexp" 和 "runtime" 包的函数和类型变量。
- "v2router": 定义了一个名为 "v2router" 的类型变量，它是一个 "router" 的实例。
- "ipAddr": 定义了一个名为 "ipAddr" 的类型变量，它是一个 "net" 包中的 "IPAddr" 类型。
- "Mask": 定义了一个名为 "Mask" 的类型变量，它是一个 "regexp" 包中的 "Mask" 类型。
- "Ctx": 定义了一个名为 "Ctx" 的类型变量，它是一个 "runtime" 包中的 "Context" 类型。
- "strconv": 定义了一个名为 "strconv" 的类型变量，它是一个 "strings" 包中的 "StringConv" 类型。
- "regexp": 定义了一个名为 "regexp" 的类型变量，它是一个 "regexp" 包中的 "RegExp" 类型。
- "ipnet": 定义了一个名为 "ipnet" 的类型变量，它是一个 "net" 包中的 "IPNet" 类型。
- "ipBlock": 定义了一个名为 "ipBlock" 的类型变量，它是一个 "net" 包中的 "IPBlock" 类型。
- "rgb": 定义了一个名为 "rgb" 的类型变量，它是一个 "math" 包中的 "RGBA" 类型。
- "crypto": 定义了一个名为 "crypto" 的类型变量，它是一个 "crypto" 包中的 "Crypto" 类型。
- " freedom": 定义了一个名为 "freedom" 的类型变量，它是一个 "freedom" 包中的 "Freedom" 类型。
- "transport": 定义了一个名为 "transport" 的类型变量，它是一个 "transport" 包中的 "Transport" 类型。


```go
package router

import (
	"context"
	"net"
	"regexp"
	"runtime"
	"strconv"
	"strings"

	v2router "github.com/v2fly/v2ray-core/v4/app/router"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/common/geodata"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/transport"
)

```

This appears to be a Python script that contains two functions, `matchDomain` and `ipOnDemand`. The `matchDomain` function takes a list of domain names and a target string, and returns a boolean indicating whether the domain name is equal to the target string. The `ipOnDemand` function is a placeholder for a function that is to be defined later.

The `matchDomain` function takes advantage of the fact that the domain names are being passed as strings, and not the actual domain objects. It converts the strings to domain objects using the `v2router.Domain_Full`, `v2router.Domain_Domain`, and `v2router.Domain_Plain` methods, and then checks if the target string is a prefix or suffix of the domain name. If it is, the function returns `true`. Otherwise, it returns `false`.

It is important to note that the `v2router` package is not defined in this script, and it is recommended to install it before running this script.


```go
const (
	Block  = 0
	Bypass = 1
	Proxy  = 2
)

const (
	AsIs         = 0
	IPIfNonMatch = 1
	IPOnDemand   = 2
)

const MaxPacketSize = 1024 * 8

func matchDomain(list []*v2router.Domain, target string) bool {
	for _, d := range list {
		switch d.GetType() {
		case v2router.Domain_Full:
			domain := d.GetValue()
			if domain == target {
				log.Tracef("domain %s hit domain(full) rule: %s", target, domain)
				return true
			}
		case v2router.Domain_Domain:
			domain := d.GetValue()
			if strings.HasSuffix(target, domain) {
				idx := strings.Index(target, domain)
				if idx == 0 || target[idx-1] == '.' {
					log.Tracef("domain %s hit domain rule: %s", target, domain)
					return true
				}
			}
		case v2router.Domain_Plain:
			// keyword
			if strings.Contains(target, d.GetValue()) {
				log.Tracef("domain %s hit keyword rule: %s", target, d.GetValue())
				return true
			}
		case v2router.Domain_Regex:
			matched, err := regexp.Match(d.GetValue(), []byte(target))
			if err != nil {
				log.Error("invalid regex", d.GetValue())
				return false
			}
			if matched {
				log.Tracef("domain %s hit regex rule: %s", target, d.GetValue())
				return true
			}
		default:
			log.Debug("unknown rule type:", d.GetType().String())
		}
	}
	return false
}

```

该函数接受一个IPv6地址目标和一个IPv4地址作为参数，并返回一个布尔值以表示是否匹配。函数首先检查目标IPv4地址是否为空，如果是，则将函数转换为IPv6地址并检查其是否为空。如果不是，则函数将返回false。接下来，函数遍历一个IPv6地址列表，并检查每个IPv6地址是否与目标IPv4地址匹配。如果匹配，则函数返回true，否则返回false。


```go
func matchIP(list []*v2router.CIDR, target net.IP) bool {
	isIPv6 := true
	len := net.IPv6len
	if target.To4() != nil {
		len = net.IPv4len
		isIPv6 = false
	}
	for _, c := range list {
		n := int(c.GetPrefix())
		mask := net.CIDRMask(n, 8*len)
		cidrIP := net.IP(c.GetIp())
		if cidrIP.To4() != nil { // IPv4 CIDR
			if isIPv6 {
				continue
			}
		} else { // IPv6 CIDR
			if !isIPv6 {
				continue
			}
		}
		subnet := &net.IPNet{IP: cidrIP.Mask(mask), Mask: mask}
		if subnet.Contains(target) {
			return true
		}
	}
	return false
}

```

这段代码定义了一个名为 `newIPAddress` 的函数，接受一个名为 `tunnel.Address` 的指针参数。函数的作用是尝试使用 `address.ResolveIP` 函数来获取目标 IP 地址，如果失败，则返回一个 `tunnel.Address` 类型的空指针，并记录错误信息。如果成功获取 IP 地址，则创建一个 `tunnel.Address` 类型的变量 `newAddress`，并设置其 IP 地址为获取到的 IP 地址，设置其地址类型为根据 IP 协议类型（IPv4 或 IPv6）进行判断，最后返回 `newAddress` 和一个表示成功或错误信息的错误对象。

这里 `tunnel.Address` 是一个用于封装目标网络地址的接口类型，它提供了一些方便的方法来设置或获取目标地址。而 `tunnel.IP` 是一个用于解析目标 IP 地址的接口类型，它提供了一些方便的方法来获取目标地址的 IP 地址。


```go
func newIPAddress(address *tunnel.Address) (*tunnel.Address, error) {
	ip, err := address.ResolveIP()
	if err != nil {
		return nil, common.NewError("router failed to resolve ip").Base(err)
	}
	newAddress := &tunnel.Address{
		IP:   ip,
		Port: address.Port,
	}
	if ip.To4() != nil {
		newAddress.AddressType = tunnel.IPv4
	} else {
		newAddress.AddressType = tunnel.IPv6
	}
	return newAddress, nil
}

```

该代码定义了一个名为 Client 的结构体，表示一个 V2Router 的客户端。该客户端包含了以下字段：

* domains：一个 3 行 *v2router.Domain 类型的数组，表示该客户端支持的所有域名。
* cidrs：一个 3 行 *v2router.CIDR 类型的数组，表示该客户端使用的 CIDR 格式的域名。
* defaultPolicy：一个整数类型的字段，表示默认策略，如果匹配的域名或 CIDR 格式不匹配，客户端将使用默认策略。
* domainStrategy：一个整数类型的字段，表示域名策略，可能的取值包括 IPOnDemand 和 IPIfNonMatch。
* underlay：一个 tunnel.Client 类型的字段，表示是否使用Underlay网络代理。
* direct：一个 freeway.Client 类型的字段，表示是否使用Direct网络代理。
* ctx：一个 context.Context 类型的字段，表示请求上下文。
* cancel：一个 context.CancelFunc 类型的字段，表示请求取消函数。

该 Client 结构体定义了一个 Route 方法，用于确定要下一跳的地址。如果请求的地址类型是域名，则该方法根据设定的域名策略选择下一跳，如果无法匹配，则尝试使用默认策略。如果请求的地址类型是 CIDR，则该方法根据设定的 CIDR 格式策略选择下一跳。如果使用了 Underlay 代理，则该方法使用 Underlay 的路由，如果使用了 Direct 代理，则该方法使用 Direct 的路由。在该方法中，如果使用了 Direct 的代理，则直接创建一个 freeway.Client 类型的实例并设置为 true，否则创建一个 tunnel.Client 类型的实例并设置为 true。


```go
type Client struct {
	domains        [3][]*v2router.Domain
	cidrs          [3][]*v2router.CIDR
	defaultPolicy  int
	domainStrategy int
	underlay       tunnel.Client
	direct         *freedom.Client
	ctx            context.Context
	cancel         context.CancelFunc
}

func (c *Client) Route(address *tunnel.Address) int {
	if address.AddressType == tunnel.DomainName {
		if c.domainStrategy == IPOnDemand {
			resolvedIP, err := newIPAddress(address)
			if err == nil {
				for i := Block; i <= Proxy; i++ {
					if matchIP(c.cidrs[i], resolvedIP.IP) {
						return i
					}
				}
			}
		}
		for i := Block; i <= Proxy; i++ {
			if matchDomain(c.domains[i], address.DomainName) {
				return i
			}
		}
		if c.domainStrategy == IPIfNonMatch {
			resolvedIP, err := newIPAddress(address)
			if err == nil {
				for i := Block; i <= Proxy; i++ {
					if matchIP(c.cidrs[i], resolvedIP.IP) {
						return i
					}
				}
			}
		}
	} else {
		for i := Block; i <= Proxy; i++ {
			if matchIP(c.cidrs[i], address.IP) {
				return i
			}
		}
	}
	return c.defaultPolicy
}

```

此函数名为 "DialConn"，定义了一个名为 "c" 的指针变量，该指针变量代表一个名为 "Client" 的客户端，该客户端有一个名为 "address" 的 *tunnel.Address 类型的参数和一个名为 "overlay" 的 tunnel.Tunnel 类型的参数。函数返回一个 tunnel.Conn 类型的结果和一个可能的 error。

函数的实现是通过对 address 和 overlay 参数的处理来确定安全连接的策略，并基于策略返回可能的结果。如果传入的策略是 "Proxy"，则返回客户端的 underlay 实例的 DialConn 函数，如果策略是 "Block"，则返回一个名为 "router blocked address" 的错误。如果策略是 "Bypass"，则尝试直接连接到客户端的 direct 实例，并返回一个 tunnel.Conn 类型的结果。如果函数在处理任何策略时遇到错误，则会输出一个 "unknown policy" 的错误。


```go
func (c *Client) DialConn(address *tunnel.Address, overlay tunnel.Tunnel) (tunnel.Conn, error) {
	policy := c.Route(address)
	switch policy {
	case Proxy:
		return c.underlay.DialConn(address, overlay)
	case Block:
		return nil, common.NewError("router blocked address: " + address.String())
	case Bypass:
		conn, err := c.direct.DialConn(address, &Tunnel{})
		if err != nil {
			return nil, common.NewError("router dial error").Base(err)
		}
		return &transport.Conn{
			Conn: conn,
		}, nil
	}
	panic("unknown policy")
}

```

该函数名为“DialPacket”，它接收一个名为“c”的指针参数，代表一个客户端客户端，并传递一个名为“overlay”的名为“tunnel.Tunnel”的参数。

函数实现首先尝试使用直接连接到路由器，如果失败，则返回一个错误。然后，它尝试使用代理连接到路由器，如果失败，则返回一个错误。

接下来，函数创建一个名为“ctx”的上下文变量，用于取消取消信号。然后创建一个名为“conn”的包装器对象，该对象包含客户端、数据连接、代理客户端和取消信号上下文。最后，创建一个名为“packetChan”的通道，用于接收和处理数据包。

函数内部还有一个名为“packetLoop”的循环，它会不断地从“packetChan”通道中接收数据包，并执行“conn.packetLoop()”函数中的代码。而“conn.packetLoop()”函数会循环处理每一个数据包，对数据包信息进行处理，并使用“c.underlay.DialPacket”函数发送数据包。


```go
func (c *Client) DialPacket(overlay tunnel.Tunnel) (tunnel.PacketConn, error) {
	directConn, err := net.ListenPacket("udp", "")
	if err != nil {
		return nil, common.NewError("router failed to dial udp (direct)").Base(err)
	}
	proxy, err := c.underlay.DialPacket(overlay)
	if err != nil {
		return nil, common.NewError("router failed to dial udp (proxy)").Base(err)
	}
	ctx, cancel := context.WithCancel(c.ctx)
	conn := &PacketConn{
		Client:     c,
		PacketConn: directConn,
		proxy:      proxy,
		cancel:     cancel,
		ctx:        ctx,
		packetChan: make(chan *packetInfo, 16),
	}
	go conn.packetLoop()
	return conn, nil
}

```

此代码定义了一个名为func的函数，接收一个名为c的客户端引用作为参数，并返回一个Close()错误类型的整数。

func函数的作用是关闭底层代理关闭的请求，并返回一个错误。当函数被调用时，它会执行以下操作：

1. 调用c.cancel()方法来取消任何正在进行的取消操作。
2. 调用c.underlay.Close()方法来关闭底层代理关闭的请求。
3. 返回前两步操作的结果。

函数的实现依赖于Config对象和Router对象。如果Config对象中包含了代理和 bypass配置，那么函数将关注基于prefix规则的前缀部分，并在找到匹配的代理或规则时停止递归。

如果Config对象中包含了块配置，那么函数将关注基于prefix规则的前缀部分，并在找到匹配的规则时停止递归。

此函数对于每个规则都执行了相同的操作，只是对不同的前缀部分做出了一些不同的事情。通过使用c.underlay.Close()方法来确保所有代理都关闭，即使前缀部分中没有定义的规则也应该关闭。


```go
func (c *Client) Close() error {
	c.cancel()
	return c.underlay.Close()
}

type codeInfo struct {
	code     string
	strategy int
}

func loadCode(cfg *Config, prefix string) []codeInfo {
	codes := []codeInfo{}
	for _, s := range cfg.Router.Proxy {
		if strings.HasPrefix(s, prefix) {
			if left := s[len(prefix):]; len(left) > 0 {
				codes = append(codes, codeInfo{
					code:     left,
					strategy: Proxy,
				})
			} else {
				log.Warn("invalid empty rule:", s)
			}
		}
	}
	for _, s := range cfg.Router.Bypass {
		if strings.HasPrefix(s, prefix) {
			if left := s[len(prefix):]; len(left) > 0 {
				codes = append(codes, codeInfo{
					code:     left,
					strategy: Bypass,
				})
			} else {
				log.Warn("invalid empty rule:", s)
			}
		}
	}
	for _, s := range cfg.Router.Block {
		if strings.HasPrefix(s, prefix) {
			if left := s[len(prefix):]; len(left) > 0 {
				codes = append(codes, codeInfo{
					code:     left,
					strategy: Block,
				})
			} else {
				log.Warn("invalid empty rule:", s)
			}
		}
	}
	return codes
}

```

This is a Go function that creates a local router and adds routes to it. It takes in a list of all the available IP addresses in the network and a list of all the prefixes for those IP addresses. It then creates a list of all the possible routes for each prefix and calculates the metric for each route. Finally, it adds all the routes to the router and updates the metric for each route.


```go
func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
	m1 := runtime.MemStats{}
	m2 := runtime.MemStats{}
	m3 := runtime.MemStats{}
	m4 := runtime.MemStats{}

	cfg := config.FromContext(ctx, Name).(*Config)
	var cancel context.CancelFunc
	ctx, cancel = context.WithCancel(ctx)

	direct, err := freedom.NewClient(ctx, nil)
	if err != nil {
		cancel()
		return nil, common.NewError("router failed to initialize raw client").Base(err)
	}

	client := &Client{
		domains:  [3][]*v2router.Domain{},
		cidrs:    [3][]*v2router.CIDR{},
		underlay: underlay,
		direct:   direct,
		ctx:      ctx,
		cancel:   cancel,
	}
	switch strings.ToLower(cfg.Router.DomainStrategy) {
	case "as_is", "as-is", "asis":
		client.domainStrategy = AsIs
	case "ip_if_non_match", "ip-if-non-match", "ipifnonmatch":
		client.domainStrategy = IPIfNonMatch
	case "ip_on_demand", "ip-on-demand", "ipondemand":
		client.domainStrategy = IPOnDemand
	default:
		return nil, common.NewError("unknown strategy: " + cfg.Router.DomainStrategy)
	}

	switch strings.ToLower(cfg.Router.DefaultPolicy) {
	case "proxy":
		client.defaultPolicy = Proxy
	case "bypass":
		client.defaultPolicy = Bypass
	case "block":
		client.defaultPolicy = Block
	default:
		return nil, common.NewError("unknown strategy: " + cfg.Router.DomainStrategy)
	}

	runtime.ReadMemStats(&m1)

	geodataLoader := geodata.NewGeodataLoader()

	ipCode := loadCode(cfg, "geoip:")
	for _, c := range ipCode {
		code := c.code
		cidrs, err := geodataLoader.LoadIP(cfg.Router.GeoIPFilename, code)
		if err != nil {
			log.Error(err)
		} else {
			log.Infof("geoip:%s loaded", code)
			client.cidrs[c.strategy] = append(client.cidrs[c.strategy], cidrs...)
		}
	}

	runtime.ReadMemStats(&m2)

	siteCode := loadCode(cfg, "geosite:")
	for _, c := range siteCode {
		code := c.code
		attrWanted := ""
		// Test if user wants domains that have an attribute
		if attrIdx := strings.Index(code, "@"); attrIdx > 0 {
			if !strings.HasSuffix(code, "@") {
				code = c.code[:attrIdx]
				attrWanted = c.code[attrIdx+1:]
			} else { // "geosite:google@" is invalid
				log.Warnf("geosite:%s invalid", code)
				continue
			}
		} else if attrIdx == 0 { // "geosite:@cn" is invalid
			log.Warnf("geosite:%s invalid", code)
			continue
		}

		domainList, err := geodataLoader.LoadSite(cfg.Router.GeoSiteFilename, code)
		if err != nil {
			log.Error(err)
		} else {
			found := false
			if attrWanted != "" {
				for _, domain := range domainList {
					for _, attr := range domain.GetAttribute() {
						if strings.EqualFold(attrWanted, attr.GetKey()) {
							client.domains[c.strategy] = append(client.domains[c.strategy], domain)
							found = true
						}
					}
				}
			} else {
				client.domains[c.strategy] = append(client.domains[c.strategy], domainList...)
				found = true
			}
			if found {
				log.Infof("geosite:%s loaded", c.code)
			} else {
				log.Errorf("geosite:%s not found", c.code)
			}
		}
	}

	runtime.ReadMemStats(&m3)

	domainInfo := loadCode(cfg, "domain:")
	for _, info := range domainInfo {
		client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
			Type:      v2router.Domain_Domain,
			Value:     strings.ToLower(info.code),
			Attribute: nil,
		})
	}

	keywordInfo := loadCode(cfg, "keyword:")
	for _, info := range keywordInfo {
		client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
			Type:      v2router.Domain_Plain,
			Value:     strings.ToLower(info.code),
			Attribute: nil,
		})
	}

	regexInfo := loadCode(cfg, "regex:")
	for _, info := range regexInfo {
		if _, err := regexp.Compile(info.code); err != nil {
			return nil, common.NewError("invalid regular expression: " + info.code).Base(err)
		}
		client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
			Type:      v2router.Domain_Regex,
			Value:     info.code,
			Attribute: nil,
		})
	}

	// Just for compatibility with V2Ray rule type `regexp`
	regexpInfo := loadCode(cfg, "regexp:")
	for _, info := range regexpInfo {
		if _, err := regexp.Compile(info.code); err != nil {
			return nil, common.NewError("invalid regular expression: " + info.code).Base(err)
		}
		client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
			Type:      v2router.Domain_Regex,
			Value:     info.code,
			Attribute: nil,
		})
	}

	fullInfo := loadCode(cfg, "full:")
	for _, info := range fullInfo {
		client.domains[info.strategy] = append(client.domains[info.strategy], &v2router.Domain{
			Type:      v2router.Domain_Full,
			Value:     strings.ToLower(info.code),
			Attribute: nil,
		})
	}

	cidrInfo := loadCode(cfg, "cidr:")
	for _, info := range cidrInfo {
		tmp := strings.Split(info.code, "/")
		if len(tmp) != 2 {
			return nil, common.NewError("invalid cidr: " + info.code)
		}
		ip := net.ParseIP(tmp[0])
		if ip == nil {
			return nil, common.NewError("invalid cidr ip: " + info.code)
		}
		prefix, err := strconv.ParseInt(tmp[1], 10, 32)
		if err != nil {
			return nil, common.NewError("invalid prefix").Base(err)
		}
		client.cidrs[info.strategy] = append(client.cidrs[info.strategy], &v2router.CIDR{
			Ip:     ip,
			Prefix: uint32(prefix),
		})
	}

	log.Info("router client created")

	runtime.ReadMemStats(&m4)

	log.Debugf("GeoIP rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m2.Alloc-m1.Alloc), common.HumanFriendlyTraffic(m2.TotalAlloc-m1.TotalAlloc))
	log.Debugf("GeoSite rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m3.Alloc-m2.Alloc), common.HumanFriendlyTraffic(m3.TotalAlloc-m2.TotalAlloc))
	log.Debugf("Plaintext rules -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m4.Alloc-m3.Alloc), common.HumanFriendlyTraffic(m4.TotalAlloc-m3.TotalAlloc))
	log.Debugf("Total(router) -> Alloc: %s; TotalAlloc: %s", common.HumanFriendlyTraffic(m4.Alloc-m1.Alloc), common.HumanFriendlyTraffic(m4.TotalAlloc-m1.TotalAlloc))

	return client, nil
}

```

# `tunnel/router/config.go`

这段代码定义了一个名为 "router" 的包，并定义了一个 "Config" 类型和一个 "RouterConfig" 类型。

"Config" 类型表示一个路由器的配置，包括启用路由、绕过、代理、阻止访问、采用默认策略以及根据域名策略等。这个配置被发送到 "router" 包中的 "RouterConfig" 类型变量。

"RouterConfig" 类型变量定义了一个路由器的详细配置，包括启用路由、代理、阻止访问、默认策略以及根据域名策略。这个配置是发送到 "router" 包中的 "Config" 类型变量的主机名。


```go
package router

import (
	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
)

type Config struct {
	Router RouterConfig `json:"router" yaml:"router"`
}

type RouterConfig struct {
	Enabled         bool     `json:"enabled" yaml:"enabled"`
	Bypass          []string `json:"bypass" yaml:"bypass"`
	Proxy           []string `json:"proxy" yaml:"proxy"`
	Block           []string `json:"block" yaml:"block"`
	DomainStrategy  string   `json:"domain_strategy" yaml:"domain-strategy"`
	DefaultPolicy   string   `json:"default_policy" yaml:"default-policy"`
	GeoIPFilename   string   `json:"geoip" yaml:"geoip"`
	GeoSiteFilename string   `json:"geosite" yaml:"geosite"`
}

```

这段代码定义了一个名为 "init" 的函数，该函数在函数开始时执行。函数内部创建了一个名为 "config.RegisterConfigCreator" 的配置创建者函数，该函数接受一个接口 "cfg" 作为参数。

配置创建者函数内部创建了一个 "cfg" 对象，该对象包含用于配置 Next.js 的路由配置。具体来说，该配置创建者设置了一个名为 "Router" 的路由器配置对象，其中包含一个 "proxy" 默认策略，以及一个 "domainStrategy" 将域名映射为 IP 地址的策略。还设置了一些文件名，如 "geoip.dat" 和 "geotime.dat"。

配置创建者函数返回生成的配置对象 "cfg"，这个对象可以被接下来的 Next.js 应用程序注册和使用。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		cfg := &Config{
			Router: RouterConfig{
				DefaultPolicy:   "proxy",
				DomainStrategy:  "as_is",
				GeoIPFilename:   common.GetAssetLocation("geoip.dat"),
				GeoSiteFilename: common.GetAssetLocation("geosite.dat"),
			},
		}
		return cfg
	})
}

```

# `tunnel/router/conn.go`

这段代码定义了一个名为 "router" 的包，它提供了用于在 Trojan 网络代理中转发数据包的功能。

它导入了以下外部库：

- "net"：用于创建套接字并发送数据包。
- "io"：用于读写文件。
- "github.com/p4gefau1t/trojan-go"：包含 Trojan 的基本类和函数。
- "github.com/p4gefau1t/trojan-go/common"：提供一些通用的工具函数。
- "github.com/p4gefau1t/trojan-go/log"：用于记录错误信息。
- "github.com/p4gefau1t/trojan-go/tunnel"：提供隧道协议的实现。

然后，它定义了一个名为 "packetInfo" 的结构体，用于存储数据包的信息。

接着，它定义了一个名为 "Router" 的接口，用于在 Trojan 网络代理中转发数据包。

最后，它创建了一个名为 "RouterImpl" 的实现了 "Router" 接口的类，它使用了 "RouterImpl#Default" 函数来创建一个默认的 "RouterImpl" 实例，然后使用了 "AddListenPort" 和 "AddRoute" 函数来设置 "RouterImpl" 实例的监听端口和路由规则。


```go
package router

import (
	"context"
	"io"
	"net"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

type packetInfo struct {
	src     *tunnel.Metadata
	payload []byte
}

```

该代码定义了一个名为 PacketConn 的 struct 类型，它表示一个在代理隧道中传输数据的数据报连接。

该 struct 包含一个名为 proxy 的 tunnel.PacketConn 类型成员，它表示一个代理隧道，用于在底层网络协议和应用程序之间传输数据。

还有名为 net.PacketConn 的成员，它表示一个普通的网络数据报连接，用于在网络中传输数据。

然后，该 struct 包含一个名为 packetChan 的 channel，用于在代理隧道中接收数据报。

还有名为 Client 的成员，它表示一个用于发送数据报的客户端。

该 struct 的 packetLoop 函数用于在代理隧道中循环接收数据报，并在收到数据报时执行相应的操作。

函数的实现分为两个部分：

1. 循环读取数据报：在循环中，该函数会从代理隧道中读取一个数据报。如果读取数据报时遇到错误，函数将使用 cancel 函数取消该操作并返回。

2. 接收处理数据报：在循环中，如果接收到数据报，该函数将提取数据报的源地址和数据，并将其存储在 packetInfo 结构体中。然后，该函数使用一个名为 tunnel.Metadata 的类型的变量，将数据源地址添加到该结构体中。

该 struct 的 packetLoop 函数作为代理隧道中的一个重要组件，可以在传输数据之前或之后对数据进行处理。


```go
type PacketConn struct {
	proxy tunnel.PacketConn
	net.PacketConn
	packetChan chan *packetInfo
	*Client
	ctx    context.Context
	cancel context.CancelFunc
}

func (c *PacketConn) packetLoop() {
	go func() {
		for {
			buf := make([]byte, MaxPacketSize)
			n, addr, err := c.proxy.ReadWithMetadata(buf)
			if err != nil {
				select {
				case <-c.ctx.Done():
					return
				default:
					log.Error("router packetConn error", err)
					continue
				}
			}
			c.packetChan <- &packetInfo{
				src:     addr,
				payload: buf[:n],
			}
		}
	}()
	for {
		buf := make([]byte, MaxPacketSize)
		n, addr, err := c.PacketConn.ReadFrom(buf)
		if err != nil {
			select {
			case <-c.ctx.Done():
				return
			default:
				log.Error("router packetConn error", err)
				continue
			}
		}
		address, _ := tunnel.NewAddressFromAddr("udp", addr.String())
		c.packetChan <- &packetInfo{
			src: &tunnel.Metadata{
				Address: address,
			},
			payload: buf[:n],
		}
	}
}

```

此代码定义了三个函数，分别用于关闭数据传输协议连接、从数据包中读取数据和向数据包中写入数据。

1. `func (c *PacketConn) Close() error`，该函数用于关闭数据传输协议连接。首先调用`c.cancel()`清空连接上下文，然后调用`c.proxy.Close()`关闭代理连接，最后调用`c.PacketConn.Close()`关闭主连接。这个函数可能会抛出错误，需要进行错误处理。

2. `func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error)`，该函数用于从数据包中读取数据。由于没有进行实现，因此不会抛出错误。

3. `func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error)`，该函数用于向数据包中写入数据。由于没有进行实现，因此不会抛出错误。

4. `func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error)`，该函数用于向数据包中写入带有元数据的报文。首先调用`c.Route(m.Address)`根据目的IP地址选择一个路由，然后调用`c.proxy.WriteWithMetadata(p, m)`从代理连接中写入数据和元数据。如果路由类型为`Proxy`，则会从代理连接中读取数据和元数据；如果路由类型为`Block`，则会返回一个错误，因为路由被屏蔽了；如果路由类型为`Bypass`，则会尝试解析目的IP地址，并从主连接中写入数据和元数据。


```go
func (c *PacketConn) Close() error {
	c.cancel()
	c.proxy.Close()
	return c.PacketConn.Close()
}

func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
	panic("implement me")
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
	panic("implement me")
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
	policy := c.Route(m.Address)
	switch policy {
	case Proxy:
		return c.proxy.WriteWithMetadata(p, m)
	case Block:
		return 0, common.NewError("router blocked address (udp): " + m.Address.String())
	case Bypass:
		ip, err := m.Address.ResolveIP()
		if err != nil {
			return 0, common.NewError("router failed to resolve udp address").Base(err)
		}
		return c.PacketConn.WriteTo(p, &net.UDPAddr{
			IP:   ip,
			Port: m.Address.Port,
		})
	default:
		panic("unknown policy")
	}
}

```

这是一段 Go 语言中的函数指针类型，表示了一个名为 `ReadWithMetadata` 的函数，接受一个名为 `PacketConn` 的接收者类型的参数 `c`，并返回一个名为 `int` 的整数类型、一个名为 `Metadata` 的接收者类型的指针类型、一个名为 `error` 的接收者类型的指针类型。

函数的作用是：接收一个字节切片类型的参数 `p` ，并从接收者 `c` 的 `packetChan` 事件中读取相关信息，然后将读取到的字节切片 `info.payload` 中的数据复制到 `p` 中。如果读取操作成功，函数将返回读取的字节数 `n`、接收者 `info.src` 的地址以及 `n` 没有复制到的字节数为 0，否则返回 0 和错误。

函数的实现中，使用了 Go 语言中的 `select` 语句，可以在事件触发时执行相应的代码。当 `c.packetChan` 事件触发时，程序会进入 `select` 语句，等待事件数据到来。如果 `info` 是一个有效的接收者，函数会将 `info.payload` 中的数据复制到 `p` 中，然后返回 `n` 和 `info.src` 的地址，如果 `n` 没有复制到 `p` 中，函数将返回 0 和错误。如果 `c.ctx.Done()` 事件触发，函数将返回 0 和错误。


```go
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
	select {
	case info := <-c.packetChan:
		n := copy(p, info.payload)
		return n, info.src, nil
	case <-c.ctx.Done():
		return 0, nil, io.EOF
	}
}

```

# `tunnel/router/data.go`



这段代码定义了一个名为 `router` 的包，其中包含了一些路由相关的类和函数。具体来说，这个包可能用于在应用程序中进行路由管理和决策，根据用户提供的 URL 或其他元数据，将请求路由到相应的控制器或服务。

在这个包中，一些具体的类可能会有更具体的实现，例如 `MyController` 和 `MyService` 类，它们可以用来处理应用程序中的具体业务逻辑和交互。

通常情况下，应用程序需要一个路由器来定义如何处理特定的 URL 或元数据。使用这个包，开发者可以使用标准库方法来定义和实现路由，然后在整个应用程序中进行路由匹配和决策。


```go
package router

```

# `tunnel/router/router_test.go`

该代码是一个Router包，用于实现网络代理功能。具体来说，该包通过在代理连接上执行一系列的处理来对数据进行修改，包括对数据进行加密、压缩、发送等操作。以下是该代码的一些主要功能：

1. 创建一个TCP代理连接：通过使用net包创建一个TCP代理连接，该连接的本地IP地址为127.0.0.1，端口号为8888。

2. 设置连接超时时间：使用 config.Set(config.FlowControlGrowth)，设置代理连接超时时间为30秒。

3. 设置日志输出：使用 config.Set(config.Logger)，设置代理连接的日志输出级别为info，输出代理连接的所有信息。

4. 对数据进行处理：通过使用 common包中的Func，对数据进行处理。具体来说，对数据进行字符串比较，如果两个字符串相同，则执行Func中的代码。在此代码中，对数据进行了一些常见的操作，如对数据进行加密、对数据进行压缩、发送等操作。

5. 设置计费策略：通过使用 config.Set(config.ChargeStrategy)，设置计费策略。具体来说，该策略会对数据进行计费，当数据传输的距离超过1G时，会对数据进行计费。计费策略的计费方式为按流量计费，即按照数据传输的流量大小进行计费。

6. 设置隧道模式：通过使用 config.Set(config.TunnelMode)，设置隧道模式。具体来说，该模式会创建一个隧道，在隧道内部传输的数据不会被代理，代理仅会对隧道外部传输的数据进行处理。

该代码是一个代理代理，其作用是对数据进行处理，包括对数据进行加密、对数据进行压缩、发送等操作。该代理会在代理连接上执行上述操作，代理连接的本地IP地址为127.0.0.1，端口号为8888。该代理会在计费模式下对数据进行计费，计费策略为按流量计费，隧道模式为开启。


```go
package router

import (
	"context"
	"net"
	"strconv"
	"strings"
	"testing"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这是一个Go语言中定义的MockClient结构体，表示一个抽象的代理客户端，用于演示代理网络传输的过程。

具体来说，该MockClient结构体有以下方法：

1. DialConn：用于建立与远程服务器连接的代理连接，但并不实际建立网络传输通道。该方法接收一个目标地址和一个数据隧道参数t，然后返回一个 nil 表示成功连接，以及一个 common.NewError "mockproxy" 的错误。

2. DialPacket：用于发送一个数据包到目标地址，并返回一个 MockPacketConn 类型的包装对象，该对象表示已经发送了数据包，但数据包并没有真正被发送出去。该方法同样接收一个数据隧道参数t，然后返回一个 nil 表示成功发送，以及一个 nil 表示没有错误。

3. Close：关闭与远程服务器的连接。该方法不执行任何操作，直接返回一个 nil 表示成功关闭。


```go
type MockClient struct{}

func (m *MockClient) DialConn(address *tunnel.Address, t tunnel.Tunnel) (tunnel.Conn, error) {
	return nil, common.NewError("mockproxy")
}

func (m *MockClient) DialPacket(t tunnel.Tunnel) (tunnel.PacketConn, error) {
	return MockPacketConn{}, nil
}

func (m MockClient) Close() error {
	return nil
}

type MockPacketConn struct{}

```

这是一组定义在 MockPacketConn struct 上的函数，用于实现数据包的读取、写入和关闭操作。具体来说，这些函数接收一个名为 m 的参数，该参数是一个 MockPacketConn 类型的实例，然后执行一系列panic 函数来模拟错误情况。

函数的作用如下：

- func(m MockPacketConn) ReadFrom(p []byte): 读取数据包并返回读取的位数、地址和错误。
- func(m MockPacketConn) WriteTo(p []byte, addr net.Addr): 写入数据包并指定目标地址，返回写入的位数和错误。
- func(m MockPacketConn) Close(): 关闭数据包连接并返回错误。
- func(m MockPacketConn) LocalAddr(): 获取模擬器中的本地地址，并返回该地址。


```go
func (m MockPacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
	panic("implement me")
}

func (m MockPacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
	panic("implement me")
}

func (m MockPacketConn) Close() error {
	panic("implement me")
}

func (m MockPacketConn) LocalAddr() net.Addr {
	panic("implement me")
}

```

这段代码定义了三个函数，分别用于设置MockPacketConn的写入、读取和设置超时时间。同时，这些函数都会抛出一个panic，用于在发生错误时输出错误信息。

具体来说，当调用这些函数时，如果出现错误，函数会输出一条错误信息并暂停执行，而如果没有错误，则会直接返回0。

由于没有提供具体的实现，因此无法确定这些函数的行为。


```go
func (m MockPacketConn) SetDeadline(t time.Time) error {
	panic("implement me")
}

func (m MockPacketConn) SetReadDeadline(t time.Time) error {
	panic("implement me")
}

func (m MockPacketConn) SetWriteDeadline(t time.Time) error {
	panic("implement me")
}

func (m MockPacketConn) WriteWithMetadata(bytes []byte, metadata *tunnel.Metadata) (int, error) {
	return 0, common.NewError("mockproxy")
}

```

该代码定义了一个名为func的函数，接受一个名为m的参数，并返回一个int类型的参数、一个指向tunnel.Metadata类型的指针和一个指向common.Error类型的错误。函数的作用是通过一个MockPacketConn类型的参数m来对数据包进行读取，并返回一个元数据。

该函数的实现比较简单，直接通过New的函数来返回一个常见的错误，并检查该函数的实现，如果函数没有返回元数据或者返回的元数据为空，则会输出该函数。


```go
func (m MockPacketConn) ReadWithMetadata(bytes []byte) (int, *tunnel.Metadata, error) {
	return 0, nil, common.NewError("mockproxy")
}

func TestRouter(t *testing.T) {
	data := `
router:
    enabled: true
    bypass: 
    - "regex:bypassreg(.*)"
    - "full:bypassfull"
    - "full:localhost"
    - "domain:bypass.com"
    block:
    - "regexp:blockreg(.*)"
    - "full:blockfull"
    - "domain:block.com"
    proxy:
    - "regexp:proxyreg(.*)"
    - "full:proxyfull"
    - "domain:proxy.com"
    - "cidr:192.168.1.1/16"
```

This is a test function that simulates a connection to a proxy server. The connection is established using the "Dial" method provided by the Go-代理库。

The function takes as input an "tunnel" struct that contains the address information for the proxy server (IPv4 or DomainName) and the port number. The function then establishes a connection to the proxy server using the "Dial" method, passing in the appropriate address information.

The function then sends a packet to the proxy server using the "DialPacket" method. The packet is sent with metadata that includes the address information for the proxy server and the port number.

Finally, the function checks if the connection was successful using the "strings.Contains" method, which tests if the error message from the "Dial" method includes the string "block". If the connection was successful, the function prints the port number returned by the "Dial" method. If the connection failed or the error message included the string "block", the function prints a failure message.


```go
`
	ctx, err := config.WithYAMLConfig(context.Background(), []byte(data))
	common.Must(err)
	client, err := NewClient(ctx, &MockClient{})
	common.Must(err)
	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.DomainName,
		DomainName:  "proxy.com",
		Port:        80,
	}, nil)
	if err.Error() != "mockproxy" {
		t.Fatal(err)
	}
	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.DomainName,
		DomainName:  "proxyreg123456",
		Port:        80,
	}, nil)
	if err.Error() != "mockproxy" {
		t.Fatal(err)
	}
	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.DomainName,
		DomainName:  "proxyfull",
		Port:        80,
	}, nil)
	if err.Error() != "mockproxy" {
		t.Fatal(err)
	}

	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.IPv4,
		IP:          net.ParseIP("192.168.123.123"),
		Port:        80,
	}, nil)
	if err.Error() != "mockproxy" {
		t.Fatal(err)
	}

	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.DomainName,
		DomainName:  "block.com",
		Port:        80,
	}, nil)
	if !strings.Contains(err.Error(), "block") {
		t.Fatal("block??")
	}
	port, err := strconv.Atoi(util.HTTPPort)
	common.Must(err)

	_, err = client.DialConn(&tunnel.Address{
		AddressType: tunnel.DomainName,
		DomainName:  "localhost",
		Port:        port,
	}, nil)
	if err != nil {
		t.Fatal("dial http failed", err)
	}

	packet, err := client.DialPacket(nil)
	common.Must(err)
	buf := [10]byte{}
	_, err = packet.WriteWithMetadata(buf[:], &tunnel.Metadata{
		Address: &tunnel.Address{
			AddressType: tunnel.DomainName,
			DomainName:  "proxyfull",
			Port:        port,
		},
	})
	if err.Error() != "mockproxy" {
		t.Fail()
	}
}

```