# trojan-go源码解析 13

# `tunnel/tls/tunnel.go`

这段代码定义了一个名为 "TLS" 的 TLS 套件，其中包含一个名为 "Tunnel" 的 struct 类型。这个 struct 类型有一个 "Name" 字段，其值为 "TLS"，类似于 Python 中的 "MyTLS" 类。

另外，代码中还导入了一个名为 "context" 的外部库，以及一个名为 "github.com/p4gefau1t/trojan-go/tunnel" 的外部库。

最后，代码没有做任何其他事情，直接返回了一个名为 "TLS" 的字符串常量。


```go
package tls

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "TLS"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}

```

这两位作者提供的代码定义了两个名为NewClient和NewServer的函数，以及一个名为init的函数。它们的作用是创建一个Tunnel类型的实例并返回一个Tunnel客户端或服务器。

具体来说，NewClient函数接收一个上下文上下文和一个Tunnel客户端，然后使用NewClient函数创建一个新的Tunnel客户端。NewServer函数接收一个上下文上下文和一个Tunnel服务器，然后使用NewServer函数创建一个新的Tunnel服务器。init函数在初始化Tunnel时被调用，然后注册了一个名为"Name"的Tunnel类型。


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

# `tunnel/tls/fingerprint/tls.go`

这段代码是一个 Go 语言中的工具函数，名为 `ParseCipher`。它旨在从一个或多个字符串（称为 "输入参数"）中解析出一个或多个 TLS 加密套（称为 "输出参数"）对应的 ID。它实现了对输入字符串中所有已知的密码套的匹配，然后返回匹配结果。

函数的实现主要分为两个步骤：

1. 遍历输入字符串中的每个字符，对其进行操作，以查找可能的密码套。
2. 如果找到了匹配的密码套，将其 ID 添加到结果列表中，并跳出循环。否则，在循环结束后，将打印一条警告消息，表示在当前字符上找不到匹配的密码套。

该函数的作用是帮助用户解析传入的一组字符，以便根据这些字符选择合适的加密套。这有助于在需要保护数据传输安全时，自动选择用于加密数据的开源密码库。


```go
package fingerprint

import (
	"crypto/tls"

	"github.com/p4gefau1t/trojan-go/log"
)

func ParseCipher(s []string) []uint16 {
	all := tls.CipherSuites()
	var result []uint16
	for _, p := range s {
		found := true
		for _, q := range all {
			if q.Name == p {
				result = append(result, q.ID)
				break
			}
			if !found {
				log.Warn("invalid cipher suite", p, "skipped")
			}
		}
	}
	return result
}

```

# `tunnel/tproxy/config.go`

这段代码是一个 Go 语言编写的交叉编译构建工具。它用于在 Go 和其他后置语言（如 Python、Ruby、Java 等）中编译对应的后置语言项目。

具体来说，这段代码的作用是将 Go 语言项目中的源代码编译成对应的后置语言代码，以便在运行时能够在其他语言环境中运行。这个工具使用了 Go 语言的 build 系统，通过在 Go 源代码上运行一系列的构建脚本来创建一个只包含 Go 语言源代码的构建文件。

然后，通过在 build 工具选项中指定需要编译的目标语言，将构建工具的使用范围扩大到了其他编程语言的支持。

最后，通过初始化时调用一个名为 init 的函数，设置了一些用于配置本地主机和端口的参数，这些参数会在构建过程中被丢失，然后就不再被使用了。


```go
//go:build linux
// +build linux

package tproxy

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

# `tunnel/tproxy/conn.go`

这段代码定义了一个名为`tproxy`的`trojan`包，其主要作用是创建和管理网络代理连接。具体来说，它实现了以下功能：

1. 在命令行参数中指定`--build`选项时，会在当前目录下建立一个名为`linux`的构建目录，并将构建过程中的输出重定向到该目录下。
2. 在定义了一个名为`tproxy`的`trojan`包的同时，也定义了一个名为`Conn`的`tproxy.Conn`结构体。这个结构体包含一个`net.Conn`类型的网络连接，以及一个`tunnel.Metadata`类型的元数据，用于保存代理连接的元数据。
3. `tproxy`包中提供的`create`函数用于创建一个新的`tproxy.Conn`实例。该函数的第一个参数是一个指向`net.Conn`类型的参数，用于设置代理连接的底层网络组件。第二个参数是一个指向`tunnel.Metadata`类型的参数，用于设置代理连接的元数据。
4. `tproxy`包中提供的`connect`函数用于连接到远程代理服务器。该函数需要一个网络代理服务器和代理连接的元数据作为输入参数。该函数使用`tproxy.Metadata`类型的元数据中保存的代理连接元数据，创建一个新的`tproxy.Conn`实例，并设置其连接到代理服务器的元数据。
5. `tproxy`包中提供的`listen`函数用于监听来自代理服务器的连接请求。该函数会创建一个新的`tproxy.Conn`实例，并将其设置为侦听来自代理服务器的连接请求。

由于该代码中的`tproxy`包主要是用于在Go应用程序中实现网络代理功能，因此并没有实现太多的业务逻辑。如果您需要更详细的解释，可以考虑提供更多的上下文信息，以便我为您提供更详细的解释。


```go
//go:build linux
// +build linux

package tproxy

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

这段代码定义了一个名为 PacketConn 的 struct 类型，该类型表示网络数据传输过程中的数据连接。

函数 Metadata() *tunnel.Metadata 返回一个指向隧道元数据的指针，其中包含了该数据连接的元数据信息。

定义了一个名为 packetInfo 的 struct 类型，该类型包含了一个隧道元数据对象和一个字节数组，用于存储数据。

函数 PacketMetadata returns a pointer to an instance of PacketInfo of type tunnel.Metadata。

该函数 PacketMetadata() returns a pointer to an instance of PacketInfo of type tunnel.Metadata. This function is used to return the metadata of a PacketConn object to other functions or gRPC services.

函数 PacketConnect creates and returns a new PacketConn instance. It captures the required input and output channels for the connection, as well as the address of the src package context.

该函数 PacketConnect creates and returns a new PacketConn instance. It captures the required input and output channels for the connection, as well as the address of the src package context.


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

这段代码定义了三个函数，分别用于读取数据、写入数据和关闭套接字连接。

1. `ReadFrom`函数接收一个字节切片（`p`）作为参数，然后尝试从连接中读取数据。如果没有数据可用，函数会抛出异常。函数的实现可能需要根据具体业务逻辑进行修改。

2. `WriteTo`函数接收一个字节切片（`p`）和一个目标网络地址（`addr`）。函数尝试将数据写入到连接中，但不会保证数据能够被准确发送。如果出现错误，函数会抛出异常。

3. `Close`函数尝试关闭套接字连接，然后返回一个非空错误。如果套接字连接已经关闭，函数不会做任何处理。


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

此函数名为`ReadWithMetadata`，其作用是读取一个二进制数据包并包含其中的元数据。函数接收一个字节切片（`p`）作为参数，然后执行一个选择器（`select`）以决定如何读取元数据。

如果选中`info`是一个有效的元数据，则函数将接收这个元数据，并将其复制到一个字节切片（`n`）中。然后，函数返回从输入中读取的数据（`n`）以及元数据（`info.metadata`）。

如果选中`c.ctx.Done()`是一个已经完成的上下文，那么函数将返回0，因为上下文已经完成。同时，如果出现任何错误，函数将返回一个错误对象（`common.NewError`）。


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

# `tunnel/tproxy/getsockopt.go`

这段代码是一个 Go 语言编写的交叉-compile命令，用于在 Linux 平台上编译 tproxy 包，并且在编译时禁用 386 架构。

具体来说，代码中包含了两个部分：

1. `//go:build linux && !386`：这是一个并行的构建命令，表示在编译时先编译基于 Linux 架构的代码，然后编译基于 386 架构的代码。这里使用了 `&&` 符号来组合两个条件，表示只有当前条件为真时才会执行相应的操作。在本例中，这个命令编译完基于 Linux 架构的代码后，再编译基于 386 架构的代码。

2. `package tproxy`：这是一个用于定义 tproxy 包的结构体。在 Go 语言中，结构体可以用 `package` 关键字来定义。

3. `func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error)`：这是一个函数，表示获取一个套接字(socket)的选项。函数使用了 `syscall.Syscall6` 系统调用，并使用了 `unsafe.Pointer` 类型的变量来传递选项的值。函数的参数包括一个 int 类型的文件描述符(fd)、一个 int 类型的套接字级别(level)、一个 int 类型的选项名称(optname)、一个 pointer 类型的选项值(optval)，以及一个指向 uint32 类型的选项长度的指针(optlen)。函数的返回值是一个 error 类型。

综合来看，这段代码的作用是编译 tproxy 包，并且在编译时禁用 386 架构，因此只有在基于 386 架构的机器上编译才会成功。


```go
//go:build linux && !386
// +build linux,!386

package tproxy

import (
	"syscall"
	"unsafe"
)

func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error) {
	_, _, e := syscall.Syscall6(
		syscall.SYS_GETSOCKOPT, uintptr(fd), uintptr(level), uintptr(optname),
		uintptr(optval), uintptr(unsafe.Pointer(optlen)), 0)
	if e != 0 {
		return e
	}
	return
}

```

# `tunnel/tproxy/getsockopt_i386.go`

这段代码定义了一个名为`tproxy`的`package`。在这个`package`中，包含了一个名为`getsockopt`的函数。

`getsockopt`函数的实现如下：


func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error) {
	_, _, e := syscall.Syscall6(
		GETSOCKOPT, uintptr(fd), uintptr(level), uintptr(optname),
		uintptr(optval), uintptr(unsafe.Pointer(optlen)), 0)
	if e != 0 {
		return e
	}
	return
}


函数参数说明：

- `fd`：文件描述符(int类型)
- `level`：选项名为`optname`的Socket Level选项。例如，`SO_REUSEADDR`是另外的选项。
- `optname`：选项名称，例如`SO_REUSEADDR`。
- `optval`：选项值，可以是`uintptr`、`intptr`或`uint32`类型。
- `optlen`：选项值的数量(必须是`uint32`类型)

函数实现中，使用`syscall.Syscall6`系统调用接口来调用Linux系统上的`getsockopt`函数。通过调用`GETSOCKOPT`系统调用接口，实现获取选项的行为。如果调用失败，函数返回一个错误。

该代码片段作为`tproxy`包中的一个函数，用于在`getsockopt`函数中实现对Socket的选项设置。


```go
//go:build linux && 386
// +build linux,386

package tproxy

import (
	"syscall"
	"unsafe"
)

const GETSOCKOPT = 15

func getsockopt(fd int, level int, optname int, optval unsafe.Pointer, optlen *uint32) (err error) {
	_, _, e := syscall.Syscall6(
		GETSOCKOPT, uintptr(fd), uintptr(level), uintptr(optname),
		uintptr(optval), uintptr(unsafe.Pointer(optlen)), 0)
	if e != 0 {
		return e
	}
	return
}

```

# `tunnel/tproxy/server.go`

这段代码是一个 Go 语言编写的 Go 框架，用于在 Linux 系统上构建并运行一个名为 "tproxy" 的工具。它通过 "trojan-go" 库提供了对网络代理的访问，通过 "github.com/p4gefau1t/trojan-go/common" 和 "github.com/p4gefau1t/trojan-go/config" 库提供了基本的配置和日志功能，通过 "github.com/p4gefau1t/trojan-go/log" 和 "github.com/p4gefau1t/trojan-go/tunnel" 库实现了网络代理的配置和运行。

具体来说，这段代码的作用是定义了一个名为 "tproxy" 的函数，它接受一个 "build" 参数。如果 "build" 参数为 "linux"，那么它将在当前目录下创建一个名为 "tproxy.linux" 的二进制文件。否则，它将执行一系列的操作，包括下载并编译 "tproxy-linux.go" 代码，并设置代理的配置和运行一些代理。最后，它还记录了代理的运行时间和结果，并允许您通过日志文件查看这些结果。


```go
//go:build linux
// +build linux

package tproxy

import (
	"context"
	"io"
	"net"
	"sync"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

这段代码定义了一个名为`Server`的结构体，表示服务器，用于处理网络连接和数据传输。

`const MaxPacketSize = 1024 * 8`定义了一个常量`MaxPacketSize`，表示每个数据包的最大尺寸，以字节为单位。

`type Server struct {`定义了服务器结构体`

`	tcpListener net.Listener`定义了一个TCP回射器`

`	udpListener *net.UDPConn`定义了一个UDP连接器`

`	packetChan  chan tunnel.PacketConn`定义了一个用于接收数据包的通道`

`	timeout     time.Duration`定义了一个用于连接超时的时间`

`	mappingLock sync.RWMutex`定义了一个互斥锁，用于同步TCP和UDP连接器`

`	mapping     map[string]*PacketConn`定义了一个用于存储TCP和UDP连接器的映射`

`	ctx         context.Context`定义了一个用于取消等待数据的上下文`

`	cancel      context.CancelFunc`定义了一个用于取消等待数据的函数`

`	//...`省略了部分实现细节`

`}`

该结构体包含了一些用于管理服务器连接的函数和变量。通过使用`net.Listen`函数，该结构体可以监听TCP或UDP端口，并相应地接收数据包。`packetChan`通道用于接收数据包，并持有`tcp探测器`和`udp探测器`实例，以便在接收到数据时采取行动。`timeout`变量用于设置连接超时的时间。`mapping`变量用于存储TCP和UDP连接器，其中键是连接器ID，值是`PacketConn`结构体类型。`ctx`变量用于在连接器关闭时取消等待数据。`cancel`函数用于取消等待数据，不过，该函数没有实现任何具体的操作。


```go
const MaxPacketSize = 1024 * 8

type Server struct {
	tcpListener net.Listener
	udpListener *net.UDPConn
	packetChan  chan tunnel.PacketConn
	timeout     time.Duration
	mappingLock sync.RWMutex
	mapping     map[string]*PacketConn
	ctx         context.Context
	cancel      context.CancelFunc
}

func (s *Server) Close() error {
	s.cancel()
	s.tcpListener.Close()
	return s.udpListener.Close()
}

```

此函数的作用是接受一个TCP连接并将其连接到服务器端的套接字上。它接收一个TCP套接字并尝试使用服务器端的套接字监听端口以接受连接。如果服务器端套接字无法接受连接，函数将返回一个Nil的错误并捕获一个错误对象。如果服务器端套接字成功接受连接，它将返回一个包含原始TCP连接信息和服务器端套接字地址的tunnel.Conn对象。函数使用tproxy库中的tcpListener函数来实现TCP连接的接受。


```go
func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := s.tcpListener.Accept()
	if err != nil {
		select {
		case <-s.ctx.Done():
		default:
			log.Fatal(common.NewError("tproxy failed to accept connection").Base(err))
		}
		return nil, common.NewError("tproxy failed to accept conn")
	}
	dst, err := getOriginalTCPDest(conn.(*net.TCPConn))
	if err != nil {
		return nil, common.NewError("tproxy failed to obtain original address of tcp socket").Base(err)
	}
	address, err := tunnel.NewAddressFromAddr("tcp", dst.String())
	common.Must(err)
	log.Info("tproxy connection from", conn.RemoteAddr().String(), "metadata", dst.String())
	return &Conn{
		metadata: &tunnel.Metadata{
			Address: address,
		},
		Conn: conn,
	}, nil
}

```

This is a Go program that uses the TProxy UDP proxy service to forward packets from a client to a TProxy UDP server. It uses the information you provided to make a connection to the server, establish a stream of packets from the client


```go
func (s *Server) packetDispatchLoop() {
	type tproxyPacketInfo struct {
		src     *net.UDPAddr
		dst     *net.UDPAddr
		payload []byte
	}
	packetQueue := make(chan *tproxyPacketInfo, 1024)

	go func() {
		for {
			buf := make([]byte, MaxPacketSize)
			n, src, dst, err := ReadFromUDP(s.udpListener, buf)
			if err != nil {
				select {
				case <-s.ctx.Done():
				default:
					log.Fatal(common.NewError("tproxy failed to read from udp socket").Base(err))
				}
				s.Close()
				return
			}
			log.Debug("udp packet from", src, "metadata", dst, "size", n)
			packetQueue <- &tproxyPacketInfo{
				src:     src,
				dst:     dst,
				payload: buf[:n],
			}
		}
	}()

	for {
		var info *tproxyPacketInfo
		select {
		case info = <-packetQueue:
		case <-s.ctx.Done():
			log.Debug("exiting")
			return
		}

		s.mappingLock.RLock()
		conn, found := s.mapping[info.src.String()]
		s.mappingLock.RUnlock()

		if !found {
			ctx, cancel := context.WithCancel(s.ctx)
			conn = &PacketConn{
				input:      make(chan *packetInfo, 128),
				output:     make(chan *packetInfo, 128),
				PacketConn: s.udpListener,
				ctx:        ctx,
				cancel:     cancel,
				src:        info.src,
			}

			s.mappingLock.Lock()
			s.mapping[info.src.String()] = conn
			s.mappingLock.Unlock()

			log.Info("new tproxy udp session from", info.src.String(), "metadata", info.dst.String())
			s.packetChan <- conn

			go func(conn *PacketConn) {
				defer conn.Close()
				log.Debug("udp packet daemon for", conn.src.String())
				for {
					select {
					case info := <-conn.output:
						if info.metadata.AddressType != tunnel.IPv4 &&
							info.metadata.AddressType != tunnel.IPv6 {
							log.Error("tproxy invalid response metadata address", info.metadata)
							continue
						}
						back, err := DialUDP(
							"udp",
							&net.UDPAddr{
								IP:   info.metadata.IP,
								Port: info.metadata.Port,
							},
							conn.src.(*net.UDPAddr),
						)
						if err != nil {
							log.Error(common.NewError("failed to dial tproxy udp").Base(err))
							return
						}
						n, err := back.Write(info.payload)
						if err != nil {
							log.Error(common.NewError("tproxy udp write error").Base(err))
							return
						}
						log.Debug("recv packet, send back to", conn.src, "payload", len(info.payload), "sent", n)
						back.Close()
					case <-s.ctx.Done():
						log.Debug("exiting")
						return
					case <-time.After(s.timeout):
						s.mappingLock.Lock()
						delete(s.mapping, conn.src.String())
						s.mappingLock.Unlock()
						log.Debug("packet session ", conn.src.String(), "timeout")
						return
					}
				}
			}(conn)
		}

		newInfo := &packetInfo{
			metadata: &tunnel.Metadata{
				Address: tunnel.NewAddressFromHostPort("udp", info.dst.IP.String(), info.dst.Port),
			},
			payload: info.payload,
		}

		select {
		case conn.input <- newInfo:
			log.Debug("tproxy packet sent with metadata", newInfo.metadata, "size", len(info.payload))
		default:
			// if we got too many packets, simply drop it
			log.Warn("tproxy udp relay queue full!")
		}
	}
}

```

This is a Go function that creates and starts a TProxy server. It takes a connection context and a tunnel server as inputs and returns the server or an error.

The function first sets up a connection to the TProxy configuration provided by the user. It creates a TCP listener and an UDP listener for the server and starts listening for incoming connections.

The function then enters a loop to receive and handle packets from the incoming connections. It uses the `packetDispatchLoop` method to handle the packets and the `tcpListener.Addr()` and `udpListener.LocalAddr()` methods to get the connection's IP and port, respectively.

Finally, it logs some information about the server and creates the server object, which is stored in the `server` variable, and returns it. If an error occurs, it returns `nil`.


```go
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	select {
	case conn := <-s.packetChan:
		log.Info("tproxy packet conn accepted")
		return conn, nil
	case <-s.ctx.Done():
		return nil, io.EOF
	}
}

func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	ctx, cancel := context.WithCancel(ctx)
	listenAddr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
	ip, err := listenAddr.ResolveIP()
	if err != nil {
		cancel()
		return nil, common.NewError("invalid tproxy local address").Base(err)
	}
	tcpListener, err := ListenTCP("tcp", &net.TCPAddr{
		IP:   ip,
		Port: cfg.LocalPort,
	})
	if err != nil {
		cancel()
		return nil, common.NewError("tproxy failed to listen tcp").Base(err)
	}

	udpListener, err := ListenUDP("udp", &net.UDPAddr{
		IP:   ip,
		Port: cfg.LocalPort,
	})
	if err != nil {
		cancel()
		return nil, common.NewError("tproxy failed to listen udp").Base(err)
	}

	server := &Server{
		tcpListener: tcpListener,
		udpListener: udpListener,
		ctx:         ctx,
		cancel:      cancel,
		timeout:     time.Duration(cfg.UDPTimeout) * time.Second,
		mapping:     make(map[string]*PacketConn),
		packetChan:  make(chan tunnel.PacketConn, 32),
	}
	go server.packetDispatchLoop()
	log.Info("tproxy server listening on", tcpListener.Addr(), "(tcp)", udpListener.LocalAddr(), "(udp)")
	log.Debug("tproxy server created")
	return server, nil
}

```

# `tunnel/tproxy/tcp.go`

这段代码定义了一个名为"tproxy"的包，其中的"Listener"类型通过一个名为"linux"的标记来定义。通过同时使用"build"和"+build"标记来定义该包的编译选项。编译选项中的"linux"表示编译为Linux平台。

该代码接下来定义了一个名为"Listener"的接口，它包含一个名为"fmt"的"fmt.Listener"类型字段，用于输出客户端连接的IP地址和端口号。

接着，该代码定义了一个名为"net"的"net.Listener"类型字段，用于定义TCP套接字监听器。

然后，该代码定义了一个名为"os"的"os.Listener"类型字段，用于定义System V装载模块的套接字监听器。

接下来，该代码定义了一个名为"syscall"的"syscall.Listener"类型字段，用于定义调用操作系统提供的系统调用函数的套接字监听器。

最后，该代码定义了一个名为"unsafe"的"unsafe.Listener"类型字段，用于定义使用操作系统C库中"listen"函数的套接字监听器。

该代码的作用是定义一个TCP或UDP套接字监听器，用于接收客户端连接并输出客户端的IP地址和端口号，支持在Linux平台上使用。


```go
//go:build linux
// +build linux

package tproxy

import (
	"fmt"
	"net"
	"os"
	"syscall"
	"unsafe"
)

// Listener describes a TCP Listener
// with the Linux IP_TRANSPARENT option defined
```

这段代码定义了一个名为 `Listener` 的 struct 类型，该类型继承自 `net.Listener` 类型，用于监听客户端连接到服务器。

在 `Listener` 结构体中，有一个名为 `Accept` 的方法，该方法接受一个 `net.TCPListener` 类型的 `base` 字段和一个返回值 `net.Conn` 和一个错误 `error`。

`Accept` 方法使用 `base.(*net.TCPListener).AcceptTCP()` 解引用 `base` 的 `net.TCPListener` 类型字段，并调用该字段的 `AcceptTCP` 方法，来接受一个客户端连接到服务器。如果该连接失败，则返回 `nil` 和一个错误。

该代码的主要目的是创建一个用于监听客户端连接到服务器的服务器 `Listener` 实例。当有客户端连接时，将接受客户端连接并返回一个 `net.TCPConn` 和 ` nil`，或者返回一个错误。


```go
// on the listening socket
type Listener struct {
	base net.Listener
}

// Accept waits for and returns
// the next connection to the listener.
//
// This command wraps the AcceptTProxy
// method of the Listener
func (listener *Listener) Accept() (net.Conn, error) {
	tcpConn, err := listener.base.(*net.TCPListener).AcceptTCP()
	if err != nil {
		return nil, err
	}

	return tcpConn, nil
}

```

这段代码定义了一个名为`Listener`的`net.Listener`类型，它用于监听TCP连接。

`Addr`函数返回的是一个`net.Addr`结构体，其中包含当前正在监听的TCP连接的IP地址和端口号。

`Close`函数返回的是一个`error`类型，用于在关闭TCP连接时执行的操作。如果关闭连接时正在监听连接，那么所有的连接都会被关闭，如果有任何阻塞的连接，那么连接将被关闭并解除阻塞。

`ListenTCP`函数用于构造一个新的TCP连接监听器。


```go
// Addr returns the network address
// the listener is accepting connections
// from
func (listener *Listener) Addr() net.Addr {
	return listener.base.Addr()
}

// Close will close the listener from accepting
// any more connections. Any blocked connections
// will unblock and close
func (listener *Listener) Close() error {
	return listener.base.Close()
}

// ListenTCP will construct a new TCP listener
```

该代码实现了一个TCP套接字倾听器，具有IP_TRANSPARENT选项。其目的是在底层TCP套接字上监听网络流量，并将接收到的数据传递给用户。以下是代码的主要步骤：

1. 设置监听器函数的参数：网络名称和监听的IP地址。
2. 调用net.ListenTCP函数来创建监听器。
3. 获取监听器返回的错误码。
4. 如果错误码为nil，则表示成功创建监听器。否则，从错误中获取错误信息并输出。
5.调用syscall.SetsockoptInt函数，设置IP_TRANSPARENT选项，并确保其值为1。
6. 返回监听器实例，表示成功创建监听器。

函数中的主要步骤如下：

1. 通过调用net.ListenTCP函数，创建一个TCP套接字倾听器，并指定监听的IP地址和网络名称。
2. 使用getFileDescriptor函数获取监听器返回的文件描述符，如果失败，则返回一个错误信息。
3. 确保文件描述符使用完毕后，调用close函数关闭文件描述符。
4. 如果错误码为nil，则调用成功创建了监听器，否则从错误信息中获取错误信息并输出。
5. 调用syscall.SetsockoptInt函数，设置IP_TRANSPARENT选项，并确保其值为1。
6. 返回监听器实例，表示成功创建监听器。


```go
// socket with the Linux IP_TRANSPARENT option
// set on the underlying socket
func ListenTCP(network string, laddr *net.TCPAddr) (net.Listener, error) {
	listener, err := net.ListenTCP(network, laddr)
	if err != nil {
		return nil, err
	}

	fileDescriptorSource, err := listener.File()
	if err != nil {
		return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("get file descriptor: %s", err)}
	}
	defer fileDescriptorSource.Close()

	if err = syscall.SetsockoptInt(int(fileDescriptorSource.Fd()), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
		return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}
	}

	return &Listener{listener}, nil
}

```

This is a function that tries to get a IPv6 address from a connection using the `conn.LocalAddr().(*net.TCPAddr).IP.To4()` method. If the IPv6 address can be obtained, the function returns a `net.TCPAddr` object with the IPv6 address and port information. If the IPv6 address cannot be obtained, the function returns a `net.TCPAddr` object with an empty IPv6 address and an error.


```go
const (
	IP6T_SO_ORIGINAL_DST = 80
	SO_ORIGINAL_DST      = 80
)

// getOriginalTCPDest retrieves the original destination address from
// NATed connection.  Currently, only Linux iptables using DNAT/REDIRECT
// is supported.  For other operating systems, this will just return
// conn.LocalAddr().
//
// Note that this function only works when nf_conntrack_ipv4 and/or
// nf_conntrack_ipv6 is loaded in the kernel.
func getOriginalTCPDest(conn *net.TCPConn) (*net.TCPAddr, error) {
	f, err := conn.File()
	if err != nil {
		return nil, err
	}
	defer f.Close()

	fd := int(f.Fd())
	// revert to non-blocking mode.
	// see http://stackoverflow.com/a/28968431/1493661
	if err = syscall.SetNonblock(fd, true); err != nil {
		return nil, os.NewSyscallError("setnonblock", err)
	}

	v6 := conn.LocalAddr().(*net.TCPAddr).IP.To4() == nil
	if v6 {
		var addr syscall.RawSockaddrInet6
		var len uint32
		len = uint32(unsafe.Sizeof(addr))
		err = getsockopt(fd, syscall.IPPROTO_IPV6, IP6T_SO_ORIGINAL_DST,
			unsafe.Pointer(&addr), &len)
		if err != nil {
			return nil, os.NewSyscallError("getsockopt", err)
		}
		ip := make([]byte, 16)
		for i, b := range addr.Addr {
			ip[i] = b
		}
		pb := *(*[2]byte)(unsafe.Pointer(&addr.Port))
		return &net.TCPAddr{
			IP:   ip,
			Port: int(pb[0])*256 + int(pb[1]),
		}, nil
	}

	// IPv4
	var addr syscall.RawSockaddrInet4
	var len uint32
	len = uint32(unsafe.Sizeof(addr))
	err = getsockopt(fd, syscall.IPPROTO_IP, SO_ORIGINAL_DST,
		unsafe.Pointer(&addr), &len)
	if err != nil {
		return nil, os.NewSyscallError("getsockopt", err)
	}
	ip := make([]byte, 4)
	for i, b := range addr.Addr {
		ip[i] = b
	}
	pb := *(*[2]byte)(unsafe.Pointer(&addr.Port))
	return &net.TCPAddr{
		IP:   ip,
		Port: int(pb[0])*256 + int(pb[1]),
	}, nil
}

```

# `tunnel/tproxy/tproxy_stub.go`

这段代码定义了一个名为"tproxy"的包，其中定义了一些函数和变量。

具体来说，这个包可能提供了一些与网络代理有关的功能，例如代理连接、发送请求、接收响应等等。但由于缺乏上下文和详细信息，我无法提供更多有关这个包的实际用途的更多信息。


```go
package tproxy

```

# `tunnel/tproxy/tunnel.go`

这段代码是一个Go语言编写的Makefile，用于编译和构建名为"tproxy"的trojan-go库。

首先，它定义了一个名为"TPROXY"的包别名。

接着，在TUNNEL.META.C（定义了TPROXY包的元数据）文件中，定义了一个名为"build"的函数，它是通过将GOOS设置为"linux"来实现的。因为GOOS可以通过GOOS环境变量来设置，所以这个函数可以被任何使用GOOS的环境变量运行。

然后，在TUNNEL.META.C文件中，定义了一个名为"+build"的函数，它是通过将GOOSS接为"linux"实现的，这个函数将会把GOOS设置为"linux"，从而编译出适用于Linux操作系统的可执行文件。

接下来，定义了一个名为"TUNNEL"的结构体，它表示一个TUNNEL连接。

最后，定义了一个名为"TPROXY"的包，它包含了一个名为"TUNNEL"的结构体，实现了trojan-go库的TUNNEL功能。


```go
//go:build linux
// +build linux

package tproxy

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "TPROXY"

type Tunnel struct{}

```

这是一段使用 Go 编写的函数式接口实现，主要作用是为了解决在 gRPC 架构中，服务实现类无法直接使用 "Name" 字段来获取服务名称的问题。

具体来说，这段代码实现了一个名为 "Tunnel" 的接口，提供了两个函数：

1. "Name" 函数：返回自身，即 "Tunnel" 的实例。
2. "NewClient" 和 "NewServer" 函数：不支持现有的 "tunnel.Client" 和 "tunnel.Server" 接口。其中 "Name" 函数的作用是让 "Tunnel" 的实例可以被创建并作为 "tunnel.Client" 和 "tunnel.Server" 的参数传入。
3. "init" 函数：初始化 "Tunnel" 接口的注册，将 "Name" 函数注册为 "Tunnel.Name" 接口的实现。


```go
func (t *Tunnel) Name() string {
	return Name
}

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

# `tunnel/tproxy/udp.go`

这段代码是一个 Go 语言编写的 build 脚本，用于在 Linux 系统上编译 tproxy 包。具体来说，它包括以下几个部分：

1. `//go:build linux`：这是一个 Go 语言自定义编译头，用于告诉编译器如何为 Linux 系统编译 tproxy 包。在 Go 语言中，可以通过在代码中使用 `//go:build` 注释来定义自定义编译头。
2. `//+build linux`：这是一个 Go 语言自定义编译后输出策略，用于告诉编译器如何为 Linux 系统编译 tproxy 包。在 Go 语言中，可以通过在代码中使用 `//go:run` 注释来定义自定义编译后输出策略。
3. `package tproxy`：这是一个 Go 语言定义的包名称，用于表示 tproxy 包的根目录。
4. `import (`：这是一个 Go 语言导入外部包的语句，用于引入 tproxy 包中定义的外部接口。
5. `bytes`：这是一个 Go 语言中用于表示字节切片的数据类型，用于表示从 tproxy 包中读取的数据。
6. `encoding/binary`：这是一个 Go 语言中用于表示二进制数据的编码类型，用于表示从 tproxy 包中读取的二进制数据。
7. `fmt`：这是一个 Go 语言中用于格式化字符串的函数，用于将 tproxy 包中定义的字符串格式化。
8. `net`：这是一个 Go 语言中用于表示网络的包，用于在 tproxy 包中定义的网络接口。
9. `os`：这是一个 Go 语言中用于表示操作系统的包，用于在 tproxy 包中定义的操作系统接口。
10. `strconv.Its`：这是一个 Go 语言中用于实现字符串转换的函数，用于从 tproxy 包中读取字符串并返回其 ASCII 值。
11. `syscall.Syscall`：这是一个 Go 语言中用于调用操作系统系统调用的函数，用于在 tproxy 包中调用操作系统中的 Syscall 函数。
12. `unsafe`：这是一个 Go 语言中用于操作 Unicode 字符串的函数，用于在 tproxy 包中处理 Unicode 字符串。

总之，这段代码定义了一个 tproxy 包，用于实现网络代理功能，通过在 Linux 系统上编译并运行 tproxy 包，可以实现网络代理、流量控制等功能。


```go
//go:build linux
// +build linux

package tproxy

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"net"
	"os"
	"strconv"
	"syscall"
	"unsafe"
)

```

该代码实现了一个 UDP 套接字监听器，用于监听来自特定网络主机上的 UDP 数据报。主要作用是创建一个新的 UDP 监听器并返回，如果监听失败则返回相应的错误信息。

具体来说，代码首先通过 `net.ListenUDP` 函数将 UDP 监听器构建出来，并指定监听的 IP 地址和端口。接着，通过 `net.ListenUDP` 函数返回一个新的 UDP 监听器对象。

如果通过 `fileDescriptorSource.File()` 函数获取到监听套接字的文件描述符，则通过 `syscall.SetsockoptInt` 函数设置 IPv4 套接字选项，指定为 IPv4 套接字并且使用无协议透明 (IPv4 Transparent) 选项。

如果通过 `syscall.SetsockoptInt` 函数设置套接字选项时出现错误，则返回一个相应的错误信息，并返回 `nil`。否则，返回 `listener`，表示成功创建了 UDP 监听器。


```go
// ListenUDP will construct a new UDP listener
// socket with the Linux IP_TRANSPARENT option
// set on the underlying socket
func ListenUDP(network string, laddr *net.UDPAddr) (*net.UDPConn, error) {
	listener, err := net.ListenUDP(network, laddr)
	if err != nil {
		return nil, err
	}

	fileDescriptorSource, err := listener.File()
	if err != nil {
		return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("get file descriptor: %s", err)}
	}
	defer fileDescriptorSource.Close()

	fileDescriptor := int(fileDescriptorSource.Fd())
	if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
		return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}
	}

	if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_RECVORIGDSTADDR, 1); err != nil {
		return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_RECVORIGDSTADDR: %s", err)}
	}

	return listener, nil
}

```

This is a function definition for `getOriginalAddress`. It appears to be a part of a larger program, although it's not clear how it's intended to be used.

The function appears to take in a `msg.Data` slice and return the original destination address of the `net.IPv4` or `net.IPv6` address that is returned by the function. The function reads the `originalDstRaw` field from the input message and attempts to determine the destination address based on the `Family` field of the `originalDstRaw` pointer.

If the function cannot determine the destination address, it will return an error. If it's able to determine the destination address, it will return the IP address, port number, and `Zone` (optional) of the destination address.

The function uses the `binary.Read` function to read the `originalDstRaw` field and the `syscall.RawSockaddrInet4` and `syscall.RawSockaddrInet6` functions to convert the IP address to a `net.IP` object. It then converts the port number from bytes to an `int` and uses it to create a `net.UDPAddr` object.

Overall, it looks like the function is intended to be a utility function for obtaining the original destination address of an IP address. However, it's difficult to provide more information without understanding the larger program and how it's being used.


```go
// ReadFromUDP reads a UDP packet from c, copying the payload into b.
// It returns the number of bytes copied into b and the return address
// that was on the packet.
//
// Out-of-band data is also read in so that the original destination
// address can be identified and parsed.
func ReadFromUDP(conn *net.UDPConn, b []byte) (int, *net.UDPAddr, *net.UDPAddr, error) {
	oob := make([]byte, 1024)
	n, oobn, _, addr, err := conn.ReadMsgUDP(b, oob)
	if err != nil {
		return 0, nil, nil, err
	}

	msgs, err := syscall.ParseSocketControlMessage(oob[:oobn])
	if err != nil {
		return 0, nil, nil, fmt.Errorf("parsing socket control message: %s", err)
	}

	var originalDst *net.UDPAddr
	for _, msg := range msgs {
		if (msg.Header.Level == syscall.SOL_IP || msg.Header.Level == syscall.SOL_IPV6) && msg.Header.Type == syscall.IP_RECVORIGDSTADDR {
			originalDstRaw := &syscall.RawSockaddrInet4{}
			if err = binary.Read(bytes.NewReader(msg.Data), binary.LittleEndian, originalDstRaw); err != nil {
				return 0, nil, nil, fmt.Errorf("reading original destination address: %s", err)
			}

			switch originalDstRaw.Family {
			case syscall.AF_INET:
				pp := (*syscall.RawSockaddrInet4)(unsafe.Pointer(originalDstRaw))
				p := (*[2]byte)(unsafe.Pointer(&pp.Port))
				originalDst = &net.UDPAddr{
					IP:   net.IPv4(pp.Addr[0], pp.Addr[1], pp.Addr[2], pp.Addr[3]),
					Port: int(p[0])<<8 + int(p[1]),
				}

			case syscall.AF_INET6:
				pp := (*syscall.RawSockaddrInet6)(unsafe.Pointer(originalDstRaw))
				p := (*[2]byte)(unsafe.Pointer(&pp.Port))
				originalDst = &net.UDPAddr{
					IP:   net.IP(pp.Addr[:]),
					Port: int(p[0])<<8 + int(p[1]),
					Zone: strconv.Itoa(int(pp.Scope_id)),
				}

			default:
				return 0, nil, nil, fmt.Errorf("original destination is an unsupported network family")
			}
		}
	}

	if originalDst == nil {
		return 0, nil, nil, fmt.Errorf("unable to obtain original destination: %s", err)
	}

	return n, addr, originalDst, nil
}

```

This is a function that sets up a Telnet or SSL/TLS connection to a remote host using the provided local IP address and port number, and a remote host that accepts incoming connections on a specific socket port. It takes as input a file descriptor that represents the socket, and a connection address for the remote host.

The function first sets the socket to non-transparent mode, which is required for TCP connections. It then sets the IP address of the socket to the local IP address, which is specified by the `syscall.SOL_IP` option.

The function then binds the socket to the remote host by sending a connection request to the `syscall.SOL_SOCKET` option, and then listens for incoming connections on the specified socket port by sending a `syscall.SOL_SOCKET` option to the `syscall.SOL_IP` option.

If the connection to the remote host is successful, the function returns a pointer to a `net.UDPConn` object, which represents the TCP connection to the remote host, and a `net.OpError` object with an error message explaining the failure.

Note that the functionClose() method is called to close the socket connection when it is no longer needed, and the `net.OpError` object is used to return an error message if an error occurs.


```go
// DialUDP connects to the remote address raddr on the network net,
// which must be "udp", "udp4", or "udp6".  If laddr is not nil, it is
// used as the local address for the connection.
func DialUDP(network string, laddr *net.UDPAddr, raddr *net.UDPAddr) (*net.UDPConn, error) {
	remoteSocketAddress, err := udpAddrToSocketAddr(raddr)
	if err != nil {
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("build destination socket address: %s", err)}
	}

	localSocketAddress, err := udpAddrToSocketAddr(laddr)
	if err != nil {
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("build local socket address: %s", err)}
	}

	fileDescriptor, err := syscall.Socket(udpAddrFamily(network, laddr, raddr), syscall.SOCK_DGRAM, 0)
	if err != nil {
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket open: %s", err)}
	}

	if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil {
		syscall.Close(fileDescriptor)
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("set socket option: SO_REUSEADDR: %s", err)}
	}

	if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
		syscall.Close(fileDescriptor)
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}
	}

	if err = syscall.Bind(fileDescriptor, localSocketAddress); err != nil {
		syscall.Close(fileDescriptor)
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket bind: %s", err)}
	}

	if err = syscall.Connect(fileDescriptor, remoteSocketAddress); err != nil {
		syscall.Close(fileDescriptor)
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket connect: %s", err)}
	}

	fdFile := os.NewFile(uintptr(fileDescriptor), fmt.Sprintf("net-udp-dial-%s", raddr.String()))
	defer fdFile.Close()

	remoteConn, err := net.FileConn(fdFile)
	if err != nil {
		syscall.Close(fileDescriptor)
		return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("convert file descriptor to connection: %s", err)}
	}

	return remoteConn.(*net.UDPConn), nil
}

```

这段代码定义了一个名为 `udpAddToSockerAddr` 的函数，它接受一个 `UDPAddr` 类型的参数，将其转换为可能用于连接和绑定的 `Sockaddr` 类型。

函数的实现根据输入的 `UDPAddr` 参数的类型和使用场景进行不同的转换，并返回一个 `Sockaddr` 类型。函数首先将输入 `UDPAddr` 的 IP 字段转换为一个 `[4]byte` 类型的IP地址，然后根据这个 IP 地址的类型选择正确的转换方式，如果 IP 地址的 IP 字段为 `nil`，则表示输入的地址不是一个有效的 IP 地址，函数将返回一个 `nil` 的错误。如果输入的地址可以转换为一个有效的 IP 地址，则函数将其转换为一个 `syscall.SockaddrInet4` 类型的 `Sockaddr` 实例，并将端口号作为参数传递给 `syscall.SockaddrInet4` 类型的函数返回，如果转换失败，则返回一个 `nil` 的错误。如果输入的地址包含一个有效的子网掩码（即 `nil` 和 `0` 组成的二进制字符串），则函数将其转换为一个 `syscall.SockaddrInet6` 类型的 `Sockaddr` 实例，并将端口号作为参数传递给 `syscall.SockaddrInet6` 类型的函数返回，如果转换失败，则返回一个 `nil` 的错误。


```go
// udpAddToSockerAddr will convert a UDPAddr
// into a Sockaddr that may be used when
// connecting and binding sockets
func udpAddrToSocketAddr(addr *net.UDPAddr) (syscall.Sockaddr, error) {
	switch {
	case addr.IP.To4() != nil:
		ip := [4]byte{}
		copy(ip[:], addr.IP.To4())

		return &syscall.SockaddrInet4{Addr: ip, Port: addr.Port}, nil

	default:
		ip := [16]byte{}
		copy(ip[:], addr.IP.To16())

		zoneID, err := strconv.ParseUint(addr.Zone, 10, 32)
		if err != nil {
			return nil, err
		}

		return &syscall.SockaddrInet6{Addr: ip, Port: addr.Port, ZoneId: uint32(zoneID)}, nil
	}
}

```

这段代码定义了一个名为udpAddrFamily的函数，用于根据网络和UDP地址确定IP地址家族。函数接收两个UDP地址和一个IP地址作为参数。首先，函数根据网络协议（如IPv4或IPv6）来确定输入的协议类型。然后，函数检查输入的IP地址是否为空指针，如果是，则尝试使用IPv4地址，否则使用IPv6地址。接下来，函数根据输入的IP地址是本地地址还是 remote 地址（即IPv4或IPv6）来确定最终的协议类型。最后，函数返回所确定的IP地址家族，值为AF_INET表示IPv4地址家族，值为AF_INET6表示IPv6地址家族。


```go
// udpAddrFamily will attempt to work
// out the address family based on the
// network and UDP addresses
func udpAddrFamily(net string, laddr, raddr *net.UDPAddr) int {
	switch net[len(net)-1] {
	case '4':
		return syscall.AF_INET
	case '6':
		return syscall.AF_INET6
	}

	if (laddr == nil || laddr.IP.To4() != nil) &&
		(raddr == nil || laddr.IP.To4() != nil) {
		return syscall.AF_INET
	}
	return syscall.AF_INET6
}

```

# `tunnel/transport/client.go`

这段代码是一个 Go 语言 package，名为 "transport"，定义了一些用于在本地host中建立与远程 host 之间的网络隧道的方法。具体来说，它实现了以下功能：

1. 导入了一些标准库和第三方库的依赖，包括 "os" 和 "os/exec" 用于操作系统相关的操作，以及 "github.com/p4gefau1t/trojan-go" 中的库。

2. 定义了一个名为 "transport" 的常量，初始化为默认值。

3. 定义了一个名为 "connect" 的函数，接受一个网络协议的名称、目标主机和端口号，返回一个连接上下文。

4. 定义了一个名为 "disconnect" 的函数，用于关闭当前的连接。

5. 定义了一个名为 "execute" 的函数，接受一个字符串，将其解析为可读的命令，并将它们执行掉。

6. 定义了一个名为 "get连接" 的函数，用于获取当前网络连接信息，包括目标主机、端口号、速度和安全套接字等。

7. 定义了一个名为 "get配置" 的函数，用于获取配置文件中的隧道信息，包括目标主机、端口号、速度和安全套接字等。

8. 定义了一个名为 "set速度" 的函数，用于设置指定通道中的速度。

9. 定义了一个名为 "tls" 的函数，用于设置 TLS 隧道。

10. 定义了一个名为 "create" 的函数，用于创建一个新通道。

11. 定义了一个名为 "info" 的函数，用于打印有关当前通道的信息。

12. 定义了一个名为 "log" 的函数，用于记录当前的日志信息。

13. 定义了一个名为 "start" 的函数，用于开始一个新通道。

14. 定义了一个名为 "stop" 的函数，用于停止所有正在运行的通道。

15. 定义了一个名为 "is running" 的函数，用于检查当前是否正在运行。

16. 在函数外部定义了一些函数，将它们作为 "transport" 包的函数。

17. 在 "transport" 包的函数中，定义了一些常量和函数，将它们作为 "transport" 包的函数。

18. 在 "connect"、"disconnect" 和 "execute" 函数中，通过操作系统调用实现了网络连接的功能。

19. 在 "get连接" 和 "get配置" 函数中，通过文件读写实现了配置文件中的隧道信息。

20. 在 "set速度" 和 "tls" 函数中，通过文件读写实现了对隧道速度和 TLS 隧道的设置。

21. 在 "create" 和 "start" 函数中，通过文件读写实现了隧道创建和开始功能。

22. 在 "stop" 和 "is running" 函数中，通过文件读写实现了停止和检查隧道状态功能。

23. 在 "execute" 函数中，通过文件执行实现了对命令的执行。


```go
package transport

import (
	"context"
	"os"
	"os/exec"
	"strconv"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
)

```

这段代码定义了一个名为 `Client` 的 struct 类型，表示一个客户端到服务器之间进行隧道连接的组件。

该 `Client` 类型包含以下字段：

- `serverAddress`：服务器地址，类型为 `tunnel.Address`。
- `cmd`：用于连接到服务器并执行命令的 `exec.Cmd` 实例。
- `ctx`：上下文栈，用于保存 cancel 函数的引用。
- `cancel`：用于取消上面创建的 `cmd`。
- `direct`：指向 `freedom.Client` 的指针，用于直接连接到服务器，而不是通过一个代理。

`Close` 函数用于关闭客户端连接并清理相关资源。具体来说，该函数执行以下操作：

- 调用 `cancel` 函数，取消 `cmd` 的运行状态。
- 如果 `cmd` 仍然有效且运行中，则调用 `cmd.Process.Kill` 函数，停止 `cmd` 的运行。
- 返回 `nil` 表示操作成功。


```go
// Client implements tunnel.Client
type Client struct {
	serverAddress *tunnel.Address
	cmd           *exec.Cmd
	ctx           context.Context
	cancel        context.CancelFunc
	direct        *freedom.Client
}

func (c *Client) Close() error {
	c.cancel()
	if c.cmd != nil && c.cmd.Process != nil {
		c.cmd.Process.Kill()
	}
	return nil
}

```

这两段代码定义了一个名为`DialPacket`的函数，接受一个名为`c`的`Client`类型的指针和一个名为`tunnel.Tunnel`类型的参数。这个函数的作用是：如果`c`是一个`Client`，那么它会执行`DialPacket`操作并返回结果；如果`c`是一个`Client`，那么它会执行`DialConn`操作并返回结果。

`DialPacket`函数的作用是：执行一个`Dial`操作并返回结果。它将创建一个`tunnel.PacketConn`类型的参数`tunnel.Tunnel`和一个`tunnel.Client`类型的参数`c`传递给`DialPacket`函数。如果`c`是一个`Client`，那么它会执行`DialConn`操作，直接拨打电话到远程服务器。如果`c`不是一个`Client`，那么它会执行`DialPacket`操作，创建一个`tunnel.PacketConn`类型的参数`tunnel.Tunnel`和一个`tunnel.Client`类型的参数`c`。无论哪种情况，`DialPacket`函数都会执行完其操作并返回结果。


```go
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

// DialConn implements tunnel.Client. It will ignore the params and directly dial to the remote server
func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
	conn, err := c.direct.DialConn(c.serverAddress, nil)
	if err != nil {
		return nil, common.NewError("transport failed to connect to remote server").Base(err)
	}
	return &Conn{
		Conn: conn,
	}, nil
}

```

This is a Go program that creates a secure connection to a remote host using an SSH tunnel. It supports various plugin types for configuration of the connection, such as chroot, SSH forwarding, and connection limits.

The program takes配置参数 like the remote host, port, and plugin options. It also creates a new SSH connection to the server and starts an SSHcmd command to establish the connection.

For each plugin type, it creates a new SSHcmd command with the corresponding configuration options. It then starts the SSHcmd command and returns the wrapped client object along with a check-pointer to indicate that the configuration was successful.

If the provided plugin type is not recognized or there is an error, it throws an common error.

SSHimate plugin support is also included to automatically configure the SSH connection based on the plugin type.


```go
// NewClient creates a transport layer client
func NewClient(ctx context.Context, _ tunnel.Client) (*Client, error) {
	cfg := config.FromContext(ctx, Name).(*Config)

	var cmd *exec.Cmd
	serverAddress := tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)

	if cfg.TransportPlugin.Enabled {
		log.Warn("trojan-go will use transport plugin and work in plain text mode")
		switch cfg.TransportPlugin.Type {
		case "shadowsocks":
			pluginHost := "127.0.0.1"
			pluginPort := common.PickPort("tcp", pluginHost)
			cfg.TransportPlugin.Env = append(
				cfg.TransportPlugin.Env,
				"SS_LOCAL_HOST="+pluginHost,
				"SS_LOCAL_PORT="+strconv.FormatInt(int64(pluginPort), 10),
				"SS_REMOTE_HOST="+cfg.RemoteHost,
				"SS_REMOTE_PORT="+strconv.FormatInt(int64(cfg.RemotePort), 10),
				"SS_PLUGIN_OPTIONS="+cfg.TransportPlugin.Option,
			)
			cfg.RemoteHost = pluginHost
			cfg.RemotePort = pluginPort
			serverAddress = tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort)
			log.Debug("plugin address", serverAddress.String())
			log.Debug("plugin env", cfg.TransportPlugin.Env)

			cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
			cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
			cmd.Stdout = os.Stdout
			cmd.Stderr = os.Stdout
			cmd.Start()
		case "other":
			cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
			cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
			cmd.Stdout = os.Stdout
			cmd.Stderr = os.Stdout
			cmd.Start()
		case "plaintext":
			// do nothing
		default:
			return nil, common.NewError("invalid plugin type: " + cfg.TransportPlugin.Type)
		}
	}

	direct, err := freedom.NewClient(ctx, nil)
	common.Must(err)
	ctx, cancel := context.WithCancel(ctx)
	client := &Client{
		serverAddress: serverAddress,
		cmd:           cmd,
		ctx:           ctx,
		cancel:        cancel,
		direct:        direct,
	}
	return client, nil
}

```

# `tunnel/transport/config.go`

这段代码定义了一个名为“transport”的包，其中定义了一个名为“Config”的结构体。

Config结构体包含三个键值对，分别是local_addr、local_port和remote_addr，它们用于配置到本地主机或远程主机的IP地址和端口号。

同时，Config结构体还包含一个名为transport_plugin的TransportPluginConfig结构体，用于配置传输层协议（如TCP、UDP等）的参数。

TransportPluginConfig结构体包含多个键值对，分别是enabled、type、command、option、arg和env，它们用于配置传输协议的开启、类型、命令、选项、参数和环境变量等。

通过这些结构体，可以方便地配置传输层的参数，以便在应用程序中使用。


```go
package transport

import (
	"github.com/p4gefau1t/trojan-go/config"
)

type Config struct {
	LocalHost       string                `json:"local_addr" yaml:"local-addr"`
	LocalPort       int                   `json:"local_port" yaml:"local-port"`
	RemoteHost      string                `json:"remote_addr" yaml:"remote-addr"`
	RemotePort      int                   `json:"remote_port" yaml:"remote-port"`
	TransportPlugin TransportPluginConfig `json:"transport_plugin" yaml:"transport-plugin"`
}

type TransportPluginConfig struct {
	Enabled bool     `json:"enabled" yaml:"enabled"`
	Type    string   `json:"type" yaml:"type"`
	Command string   `json:"command" yaml:"command"`
	Option  string   `json:"option" yaml:"option"`
	Arg     []string `json:"arg" yaml:"arg"`
	Env     []string `json:"env" yaml:"env"`
}

```

这段代码定义了一个名为 "init" 的函数，该函数在函数开始时执行。函数内部创建了一个名为 "Config" 的对象的配置创建器，并将其注册为一个名为 "Name" 的配置创建器的二元组。这个配置创建器函数返回一个接口 "func() interface{}"，并将其传入创建的配置创建器对象，这样 "Config" 对象就可以通过这个接口进行调用。


```go
func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(Config)
	})
}

```

# `tunnel/transport/conn.go`

这段代码定义了一个名为"transport"的包，其中包含一个名为"Conn"的类型，该类型包含一个net.Conn对象。

"Metadata"是"Metadata"类型的别名，用于存储与连接相关的元数据。

函数"Metadata"返回一个指向tunnel.Metadata对象的引用，其中元数据包含连接的元数据，如协议统计信息，开销和本地地址等信息。

函数"Connect"建立一个TCP连接，并返回一个包含连接对象和元数据的变量。函数使用了trojan-go库中的tunnel包，用于在两个受害者在之间建立网络隧道。


```go
package transport

import (
	"net"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

type Conn struct {
	net.Conn
}

func (c *Conn) Metadata() *tunnel.Metadata {
	return nil
}

```

# `tunnel/transport/server.go`

这段代码是一个 Go 语言的项目，名为 "transport"，旨在为网络编程提供一些通用的工具和函数。

具体来说，这个项目的代码包含以下几个主要部分：

1. HTTP 客户端：net/http，用于创建 HTTP 客户端连接，并发送和接收 HTTP 请求。
2. 标准输入和输出(stdio)：用于读写文件、URL 和端口等常用功能。
3. 字符串转换：strconv.MapStringToInt，用于将输入的字符串映射为整数。
4. 上下文(context)：用于异步编程中的数据传递和事件通知。
5. 网络套接字(net)：用于创建网络套接字并监听端口。
6. 互斥锁(sync)：用于 synchronize 操作，例如同时访问一个共享的内存区域。
7. 函数式编程(Functional Programming)：通过一些函数式编程技巧来简化代码，例如 Map、Filter 和 Reduce。
8. 日志(log):log.Logger，用于记录和输出调试信息。
9. 网络隧道(Tunnel)：通过 Internet 隧道传输数据。

此外，项目还包含一些名为 transport 的包，这些包可以通过遥调用 Lambda 函数来扩展功能。


```go
package transport

import (
	"bufio"
	"context"
	"net"
	"net/http"
	"os"
	"os/exec"
	"strconv"
	"sync"
	"time"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

```

该代码定义了一个名为Server的 struct 类型，表示服务器应用程序。该 struct 包含以下字段：

- tcpListener：表示服务器监听的传输层协议套接字。
- cmd：表示一个执行命令的上下文命令对象。
- connChan：表示一个通道，用于从客户端连接到服务器。
- wsChan：表示一个通道，用于从客户端连接到 WebSocket 服务器。
- httpLock：是一个互斥锁，用于确保在所有客户端连接到服务器时，只有一个 HTTP 连接。
- nextHTTP：一个布尔值，表示是否正在等待下一个 HTTP 连接。
- ctx：表示一个上下文。
- cancel：是一个取消函数，用于取消正在等待的连接。

其中，Server  struct 的 Close() 方法有以下行为：

- 关闭服务器监听的传输层协议套接字。
- 停止正在运行的执行命令。
- 关闭服务器监听的 HTTP 连接。
- 如果下一个 HTTP 连接还没有准备好，则取消正在等待的连接。

注意，由于该代码没有进行测试，因此不能确定其所有行为的正确性。


```go
// Server is a server of transport layer
type Server struct {
	tcpListener net.Listener
	cmd         *exec.Cmd
	connChan    chan tunnel.Conn
	wsChan      chan tunnel.Conn
	httpLock    sync.RWMutex
	nextHTTP    bool
	ctx         context.Context
	cancel      context.CancelFunc
}

func (s *Server) Close() error {
	s.cancel()
	if s.cmd != nil && s.cmd.Process != nil {
		s.cmd.Process.Kill()
	}
	return s.tcpListener.Close()
}

```

This is a function that implements the accept function for an HTTP connection in a transport layer. The function is responsible for receiving a connection from the client, and either passing it to the webSocket protocol layer or the TCP protocol layer for further inspection, or sending it a malicious request as a boolean flag.

It is protected by a lock to ensure that multiple connections can only be made at a time, and the锁 is released after the connection has been passed to the webSocket protocol layer or the TCP protocol layer.

The function uses a variable `tcpConn` to store the connection, and a variable `s.httpLock` to synchronize access to the connection. The variable `s.nextHTTP` is set to `false` to indicate that there is no HTTP request being made.

The function first checks if the connection has already been passed to the webSocket protocol layer, and if it has, it returns immediately without doing anything.

If the connection is not a HTTP request, the function uses a `NewRewindConn` function to close the connection and returns immediately.

If the connection is an HTTP request, the function reads the request using a `bufio.NewReader` and a `httpReq` variable, and passes it to the `http.ReadRequest` function.

If there is an error, the function logs it and returns after waiting for 1 second.

If the connection is a HTTP request, the function passes it to the ` websocket. prototype.http.Request` function, and if it is not a request, it sends a `conn` message to the `s.connChan` channel.

If the connection is a HTTP request, the function logs it and passes it to the ` websocket.prototype.conn.Conn` function, which returns the connection object.

The function also uses a `bufio.NewReader` to read the connection's buffer, which is `512 bytes` in this case.


```go
func (s *Server) acceptLoop() {
	for {
		tcpConn, err := s.tcpListener.Accept()
		if err != nil {
			select {
			case <-s.ctx.Done():
			default:
				log.Error(common.NewError("transport accept error").Base(err))
				time.Sleep(time.Millisecond * 100)
			}
			return
		}

		go func(tcpConn net.Conn) {
			log.Info("tcp connection from", tcpConn.RemoteAddr())
			s.httpLock.RLock()
			if s.nextHTTP { // plaintext mode enabled
				s.httpLock.RUnlock()
				// we use real http header parser to mimic a real http server
				rewindConn := common.NewRewindConn(tcpConn)
				rewindConn.SetBufferSize(512)
				defer rewindConn.StopBuffering()

				r := bufio.NewReader(rewindConn)
				httpReq, err := http.ReadRequest(r)
				rewindConn.Rewind()
				rewindConn.StopBuffering()
				if err != nil {
					// this is not a http request, pass it to trojan protocol layer for further inspection
					s.connChan <- &Conn{
						Conn: rewindConn,
					}
				} else {
					// this is a http request, pass it to websocket protocol layer
					log.Debug("plaintext http request: ", httpReq)
					s.wsChan <- &Conn{
						Conn: rewindConn,
					}
				}
			} else {
				s.httpLock.RUnlock()
				s.connChan <- &Conn{
					Conn: tcpConn,
				}
			}
		}(tcpConn)
	}
}

```

该函数名为`func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error)`，它接收一个名为`overlay`的传输通道参数，并返回一个传输通道连接和可能的错误。

函数体中包含以下语句：


	// 如果传输通道参数为WEBSOCKET或HTTP，则获取服务器HTTP锁，设置nextHTTP为true，然后尝试连接到传输通道。
	if overlay != nil && (overlay.Name() == "WEBSOCKET" || overlay.Name() == "HTTP") {
		s.httpLock.Lock()
		s.nextHTTP = true
		s.httpLock.Unlock()
		select {
			case conn := <-s.wsChan:
				return conn, nil
				case <-s.ctx.Done():
					return nil, common.NewError("transport server closed")
				}
			}
		}
	}
	select {
		case conn := <-s.connChan:
			return conn, nil
		case <-s.ctx.Done():
			return nil, common.NewError("transport server closed")
		}
	}


函数的作用是处理服务器接受连接到传输通道的过程。首先，它检查给定的传输通道参数是否为WEBSOCKET或HTTP。如果是，它尝试获取服务器HTTP锁，设置`nextHTTP`为`true`，然后连接到传输通道。如果连接成功，它返回传输通道连接对象，否则返回一个错误。


```go
func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
	// TODO fix import cycle
	if overlay != nil && (overlay.Name() == "WEBSOCKET" || overlay.Name() == "HTTP") {
		s.httpLock.Lock()
		s.nextHTTP = true
		s.httpLock.Unlock()
		select {
		case conn := <-s.wsChan:
			return conn, nil
		case <-s.ctx.Done():
			return nil, common.NewError("transport server closed")
		}
	}
	select {
	case conn := <-s.connChan:
		return conn, nil
	case <-s.ctx.Done():
		return nil, common.NewError("transport server closed")
	}
}

```

This is a Go program that sets up an embedded HTTP server using the Jetty HTTP server library and the go-toml-plugin HTTP plugin.

The program creates a configuration file (`cfg.toml`) that sets up the basic settings for the server, such as the transport plugin to use (` Jetty8`), the server's hostname and port, and the IP address for the DNS server.

The program then creates a function called `createServer` that takes the configuration file as input and returns a struct containing the暴露 server settings and a function to start the server.

The `createServer` function creates an instance of the `Server` struct, which has several fields:

* `tcpListener`: a `net.Listen` function that creates a TCP listener on the specified IP address and port, and starts listening for incoming connections.
* `cmd`: an `exec.Command` function that runs the Jetty HTTP server command with the specified arguments and environment variables.
* `ctx`: a context that allows the cancel() method to cancel the operation.
* `connChan`: a channel that is used to accept incoming connections.
* `wsChan`: a channel that is used to accept incoming WebSocket connections.

The `Server` struct also has a `acceptLoop` method that runs in a loop and accepts incoming connections by calling the `accept` method on the `tcpListener` and creating a new `tcp.连接` object.

The program also has a `wsDial`方法，用于在ws端口建立dial连接。

The `createServer` function returns a server object, which has a `Start` method that starts the server by running the `acceptLoop` method on the `tcpListener` and the `cmd` function.


```go
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	panic("not supported")
}

// NewServer creates a transport layer server
func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	listenAddress := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)

	var cmd *exec.Cmd
	if cfg.TransportPlugin.Enabled {
		log.Warn("transport server will use plugin and work in plain text mode")
		switch cfg.TransportPlugin.Type {
		case "shadowsocks":
			trojanHost := "127.0.0.1"
			trojanPort := common.PickPort("tcp", trojanHost)
			cfg.TransportPlugin.Env = append(
				cfg.TransportPlugin.Env,
				"SS_REMOTE_HOST="+cfg.LocalHost,
				"SS_REMOTE_PORT="+strconv.FormatInt(int64(cfg.LocalPort), 10),
				"SS_LOCAL_HOST="+trojanHost,
				"SS_LOCAL_PORT="+strconv.FormatInt(int64(trojanPort), 10),
				"SS_PLUGIN_OPTIONS="+cfg.TransportPlugin.Option,
			)

			cfg.LocalHost = trojanHost
			cfg.LocalPort = trojanPort
			listenAddress = tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
			log.Debug("new listen address", listenAddress)
			log.Debug("plugin env", cfg.TransportPlugin.Env)

			cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
			cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
			cmd.Stdout = os.Stdout
			cmd.Stderr = os.Stdout
			cmd.Start()
		case "other":
			cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
			cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
			cmd.Stdout = os.Stdout
			cmd.Stderr = os.Stdout
			cmd.Start()
		case "plaintext":
			// do nothing
		default:
			return nil, common.NewError("invalid plugin type: " + cfg.TransportPlugin.Type)
		}
	}
	tcpListener, err := net.Listen("tcp", listenAddress.String())
	if err != nil {
		return nil, err
	}

	ctx, cancel := context.WithCancel(ctx)
	server := &Server{
		tcpListener: tcpListener,
		cmd:         cmd,
		ctx:         ctx,
		cancel:      cancel,
		connChan:    make(chan tunnel.Conn, 32),
		wsChan:      make(chan tunnel.Conn, 32),
	}
	go server.acceptLoop()
	return server, nil
}

```