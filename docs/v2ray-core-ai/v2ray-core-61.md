# v2ray-core源码解析 61

# `transport/internet/connection.go`

这段代码定义了一个名为`StatCounterConnection`的结构体，它代表了一个使用`v2ray.com`作为代理的Internet连接。这个结构体包含一个`net.Conn`类型的`Connection`接口成员，该成员表示一个网络连接。同时，它还包含两个`stats.Counter`类型的成员变量：`ReadCounter`和`WriteCounter`。

`ReadCounter`统计了统计请求中从代理服务器接收到的数据量，`WriteCounter`统计了统计发送出去的数据量。这些统计信息可以帮助您了解与代理服务器建立通信时的数据传输情况。

这个包可能用于统计使用代理服务器时数据的接收和发送情况。


```go
package internet

import (
	"net"

	"v2ray.com/core/features/stats"
)

type Connection interface {
	net.Conn
}

type StatCouterConnection struct {
	Connection
	ReadCounter  stats.Counter
	WriteCounter stats.Counter
}

```

这两函数的作用是作为StatCounterConnection类的成员函数，用于读取和写入数据到连接上。

具体来说，Read函数接收一个字节切片([]byte)，然后向连接上的数据源进行读取操作。如果连接上存在数据源，那么将连接上的数据源的读取计数器Add(int64(nBytes))，其中nBytes是读取的字节数，err表示读取过程中是否出错。最后返回实际读取的字节数，err表示错误情况。

Write函数与Read函数类似，只不过是向连接上的数据源写入数据。如果连接上有数据源，那么将连接上的数据源的写入计数器Add(int64(nBytes))，其中nBytes是写入的字节数，err表示写入过程中是否出错。最后返回实际写入的字节数，err表示错误情况。


```go
func (c *StatCouterConnection) Read(b []byte) (int, error) {
	nBytes, err := c.Connection.Read(b)
	if c.ReadCounter != nil {
		c.ReadCounter.Add(int64(nBytes))
	}

	return nBytes, err
}

func (c *StatCouterConnection) Write(b []byte) (int, error) {
	nBytes, err := c.Connection.Write(b)
	if c.WriteCounter != nil {
		c.WriteCounter.Add(int64(nBytes))
	}
	return nBytes, err
}

```

# `transport/internet/dialer.go`

这段代码定义了一个名为"internet"的包，其中包含一个名为"Dialer"的接口类型。这个接口类型的一个名为"Dial"的方法用于拨打电话给指定目标，另一个名为"Address"的静态方法返回该接口的地址，有时需要从环境中获取它。

具体来说，这段代码定义了一个"Dialer"接口，它会在连接上下文(Context)中调用"Dial"方法进行拨打电话。然后，它还定义了一个名为"Session"的接口，用于在线程中保存和检索会话信息。

在"internet"包的帮助下，我们可以使用这个Dialer接口来拨打电话，并使用Session接口来管理在线会话。


```go
package internet

import (
	"context"

	"v2ray.com/core/common/net"
	"v2ray.com/core/common/session"
)

// Dialer is the interface for dialing outbound connections.
type Dialer interface {
	// Dial dials a system connection to the given destination.
	Dial(ctx context.Context, destination net.Destination) (Connection, error)

	// Address returns the address used by this Dialer. Maybe nil if not known.
	Address() net.Address
}

```

这段代码定义了一个名为dialFunc的接口，表示一个用于通过网络连接到指定目的地的函数。接着定义了一个名为dialFunc的函数类型，该类型包含一个名为(ctx context.Context, dest net.Destination, streamSettings *MemoryStreamConfig)的参数，以及一个返回类型为Connection和error的函数值。

接着，该代码实现了一个名为transportDialerCache的map，用于存储注册的Dialer。当尝试注册一个Dialer时，如果该Dialer已经在map中，那么函数会返回一个错误，否则将Dialer注册到map中。

最后，该代码还实现了一个名为registerTransportDialer的函数，用于注册一个Dialer到transportDialerCache中。如果注册失败，会返回一个错误。


```go
// dialFunc is an interface to dial network connection to a specific destination.
type dialFunc func(ctx context.Context, dest net.Destination, streamSettings *MemoryStreamConfig) (Connection, error)

var (
	transportDialerCache = make(map[string]dialFunc)
)

// RegisterTransportDialer registers a Dialer with given name.
func RegisterTransportDialer(protocol string, dialer dialFunc) error {
	if _, found := transportDialerCache[protocol]; found {
		return newError(protocol, " dialer already registered").AtError()
	}
	transportDialerCache[protocol] = dialer
	return nil
}

```

该代码是一个名为`Dial`的函数，它用于通过互联网连接到指定目的地。它需要一个`net.Destination`类型的参数`dest`以及一个`MemoryStreamConfig`类型的参数`streamSettings`。函数返回一个`Connection`类型的接口和一个错误。

函数首先检查`dest`是否为`net.Network_TCP`，如果是，则执行以下操作：

1. 如果`streamSettings`为`nil`，则创建一个默认的`MemoryStreamConfig`并将其设置为`streamSettings`。
2. 查找`transportDialerCache`中的协议名称，并返回其`dialer`。
3. 如果`dialer`不为`nil`，则返回`dialer`的`ClientCall`方法，并将其与`ctx`和`dest`以及`streamSettings`一起使用。

如果`dest`不是`net.Network_TCP`，也不是`net.Network_UDP`，则返回一个错误。


```go
// Dial dials a internet connection towards the given destination.
func Dial(ctx context.Context, dest net.Destination, streamSettings *MemoryStreamConfig) (Connection, error) {
	if dest.Network == net.Network_TCP {
		if streamSettings == nil {
			s, err := ToMemoryStreamConfig(nil)
			if err != nil {
				return nil, newError("failed to create default stream settings").Base(err)
			}
			streamSettings = s
		}

		protocol := streamSettings.ProtocolName
		dialer := transportDialerCache[protocol]
		if dialer == nil {
			return nil, newError(protocol, " dialer not registered").AtError()
		}
		return dialer(ctx, dest, streamSettings)
	}

	if dest.Network == net.Network_UDP {
		udpDialer := transportDialerCache["udp"]
		if udpDialer == nil {
			return nil, newError("UDP dialer not registered").AtError()
		}
		return udpDialer(ctx, dest, streamSettings)
	}

	return nil, newError("unknown network ", dest.Network)
}

```

这段代码定义了一个名为DialSystem的函数，它使用 DialSystem 系统调用拨号网络连接。函数接受三个参数：一个上下文上下文、一个目标网络接口和一个套接字配置选项指针。函数返回一个网络连接对象和一个错误。

函数的作用是：创建一个网络连接并将其返回。它通过调用 effectiveSystemDialer.Dial 函数来实现拨号。如果目标网络接口存在，函数将使用该接口进行连接。如果目标网络接口不存在，函数将默认连接失败。函数还使用了 session.OutboundFromContext 函数从当前上下文中获取一个出站套接字。


```go
// DialSystem calls system dialer to create a network connection.
func DialSystem(ctx context.Context, dest net.Destination, sockopt *SocketConfig) (net.Conn, error) {
	var src net.Address
	if outbound := session.OutboundFromContext(ctx); outbound != nil {
		src = outbound.Gateway
	}
	return effectiveSystemDialer.Dial(ctx, src, dest, sockopt)
}

```

# `transport/internet/dialer_test.go`

这段代码是用于测试 "v2ray.com/core/transport/internet" 包的一个函数。它通过使用 Go-CPU 库的 cmp.Run 函数来运行测试。以下是代码的主要步骤：

1. 导入所需的库。
2. 定义一个名为 "TestDialWithLocalAddr" 的函数。
3. 在函数内部创建一个 tcp.Server 实例。
4. 调用 Start 函数启动服务器。
5. 创建一个目标客户端。
6. 使用 DialSystem 函数拨打目标客户端。
7. 关闭客户端并关闭服务器。
8. 打印结果并检查它们是否与期望的结果相符。如果结果不同，则函数将失败。


```go
package internet_test

import (
	"context"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/testing/servers/tcp"
	. "v2ray.com/core/transport/internet"
)

func TestDialWithLocalAddr(t *testing.T) {
	server := &tcp.Server{}
	dest, err := server.Start()
	common.Must(err)
	defer server.Close()

	conn, err := DialSystem(context.Background(), net.TCPDestination(net.LocalHostIP, dest.Port), nil)
	common.Must(err)
	if r := cmp.Diff(conn.RemoteAddr().String(), "127.0.0.1:"+dest.Port.String()); r != "" {
		t.Error(r)
	}
	conn.Close()
}

```

# `transport/internet/errors.generated.go`

这段代码定义了一个名为"internet"的包，它导入了来自"v2ray.com/core/common/errors"的"errors"和"errPathObjHolder"类型。

在内部类型"errPathObjHolder"中，没有做出任何实际的计算或定义变量。

在函数"newError"中，它接收一个或多个参数，并将它们添加到一个名为"values"的匿名包中。然后，它使用内置的"errors.New"函数来创建一个表示给定值的错误。最后，它使用"errPathObjHolder"类型来设置错误对象的有效路径。

具体来说，"newError"函数将创建一个名为"internet.errors"的错误类型，该类型继承自"errors.Error"，它包含一个名为"err"的函数，用于创建一个表示给定值的错误对象，以及一个名为"errPathObjHolder"的类型，它包含一个匿名类型"errPathObjHolder{}"。

通过调用"newError"函数，可以创建一个表示给定错误信息的错误对象，该对象可以使用"err"函数来设置错误对象中包含的错误信息，同时使用"errPathObjHolder"类型来设置错误对象的路径和对象。


```go
package internet

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `transport/internet/header.go`

这段代码定义了一个名为"internet"的包，它通过导入两个相关的实现了"PacketHeader"接口的类型来实现网络数据包的创建和发送。

具体来说，它实现了以下功能：

1. 定义了一个名为"PacketHeader"的接口，它提供了一些用于包装网络数据包的属性和方法。

2. 实现了一个名为"CreatePacketHeader"的函数类型，该类型接受一个"config"参数，然后返回一个"PacketHeader"实例和可能的错误。函数创建了一个新的"PacketHeader"实例，如果该实例可以是"PacketHeader"类型的目标，函数将返回该实例，否则返回一个错误。

3. 通过使用"CreateObject"函数，将"PacketHeader"的配置参数包装成一个"context.Background"上下文中的对象，然后将该对象传递给" common.CreateObject"函数，它将返回一个实现了"PacketHeader"接口的"PacketHeader"实例或错误。

4. 通过使用"net"包的"net.http"类型，创建一个HTTP请求并将数据包发送到指定的目标，然后处理接收到的数据包。

因此，这段代码的主要目的是创建一个用于创建和发送网络数据包的函数，该函数可以根据传入的配置参数返回一个"PacketHeader"实例或者一个错误。


```go
package internet

import (
	"context"
	"net"

	"v2ray.com/core/common"
)

type PacketHeader interface {
	Size() int32
	Serialize([]byte)
}

func CreatePacketHeader(config interface{}) (PacketHeader, error) {
	header, err := common.CreateObject(context.Background(), config)
	if err != nil {
		return nil, err
	}
	if h, ok := header.(PacketHeader); ok {
		return h, nil
	}
	return nil, newError("not a packet header")
}

```

这段代码定义了一个名为 ConnectionAuthenticator 的接口类型，该接口有两个方法：Client 和 Server。同时，该接口实现了一个名为 CreateConnectionAuthenticator 的函数，用于创建连接认证器实例。

函数接收一个 ConnectionConfig 类型的参数。在函数内部，首先创建一个名为 auth 的对象。然后，使用 common.CreateObject 函数，将 auth 对象与传入的 ConnectionConfig 进行绑定。如果创建过程中出现错误，函数将返回 nil 和错误信息。

接着，函数调用自己的 CreateObject 函数，该函数将创建的 auth 对象作为参数传入，并返回其创建的 ConnectionAuthenticator 类型。如果 auth 对象是 ConnectionAuthenticator 类型，函数将直接返回该对象。否则，函数将返回一个名为 nil 的错误对象。


```go
type ConnectionAuthenticator interface {
	Client(net.Conn) net.Conn
	Server(net.Conn) net.Conn
}

func CreateConnectionAuthenticator(config interface{}) (ConnectionAuthenticator, error) {
	auth, err := common.CreateObject(context.Background(), config)
	if err != nil {
		return nil, err
	}
	if a, ok := auth.(ConnectionAuthenticator); ok {
		return a, nil
	}
	return nil, newError("not a ConnectionAuthenticator")
}

```

# `transport/internet/header_test.go`

这段代码是关于测试 Internet 传输协议中不同负载均衡头部的实现。它主要的作用是测试。当前的功能是输出所有创建的负载头部，测试所有可用的头部，然后根据需要打印出来。


```go
package internet_test

import (
	"testing"

	"v2ray.com/core/common"
	. "v2ray.com/core/transport/internet"
	"v2ray.com/core/transport/internet/headers/noop"
	"v2ray.com/core/transport/internet/headers/srtp"
	"v2ray.com/core/transport/internet/headers/utp"
	"v2ray.com/core/transport/internet/headers/wechat"
	"v2ray.com/core/transport/internet/headers/wireguard"
)

func TestAllHeadersLoadable(t *testing.T) {
	testCases := []struct {
		Input interface{}
		Size  int32
	}{
		{
			Input: new(noop.Config),
			Size:  0,
		},
		{
			Input: new(srtp.Config),
			Size:  4,
		},
		{
			Input: new(utp.Config),
			Size:  4,
		},
		{
			Input: new(wechat.VideoConfig),
			Size:  13,
		},
		{
			Input: new(wireguard.WireguardConfig),
			Size:  4,
		},
	}

	for _, testCase := range testCases {
		header, err := CreatePacketHeader(testCase.Input)
		common.Must(err)
		if header.Size() != testCase.Size {
			t.Error("expected size ", testCase.Size, " but got ", header.Size())
		}
	}
}

```

# `transport/internet/internet.go`

这段代码是一个 Go 语言 package，名为 "internet"，它定义了一个名为 "errorgen" 的函数，使用了 "go:generate" 指令，用于生成对应文件结构的 Go 代码。通过调用这个函数，可以生成对应文件结构的 Go 代码，从而方便对项目进行打包、部署等操作。


```go
package internet

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `transport/internet/memory_settings.go`

这段代码定义了一个名为`MemoryStreamConfig`的结构体，它表示了`Protobuf`文档中的`StreamConfig`。

`MemoryStreamConfig`结构体包含以下字段：

- `ProtocolName`：Protobuf协议的名称。
- `ProtocolSettings`：Protobuf协议的设置。
- `SecurityType`：安全套接字设置。
- `SecuritySettings`：安全套接字设置。
- `SocketSettings`：套接字设置。

该`ToMemoryStreamConfig`函数接受一个`StreamConfig`作为输入参数，并返回一个内存中的`MemoryStreamConfig`结构体。如果输入参数`StreamConfig`为空，或者尝试获取无效的设置，函数将返回一个`nil`的`MemoryStreamConfig`和一个错误。

如果输入参数`StreamConfig`不为空，函数首先尝试获取有效的传输设置，然后将其转换为`MemoryStreamConfig`结构体。如果已经设置安全套接字，函数将根据需要填充`SecuritySettings`字段。最后，函数将返回`MemoryStreamConfig`结构体。


```go
package internet

// MemoryStreamConfig is a parsed form of StreamConfig. This is used to reduce number of Protobuf parsing.
type MemoryStreamConfig struct {
	ProtocolName     string
	ProtocolSettings interface{}
	SecurityType     string
	SecuritySettings interface{}
	SocketSettings   *SocketConfig
}

// ToMemoryStreamConfig converts a StreamConfig to MemoryStreamConfig. It returns a default non-nil MemoryStreamConfig for nil input.
func ToMemoryStreamConfig(s *StreamConfig) (*MemoryStreamConfig, error) {
	ets, err := s.GetEffectiveTransportSettings()
	if err != nil {
		return nil, err
	}

	mss := &MemoryStreamConfig{
		ProtocolName:     s.GetEffectiveProtocol(),
		ProtocolSettings: ets,
	}

	if s != nil {
		mss.SocketSettings = s.SocketSettings
	}

	if s != nil && s.HasSecuritySettings() {
		ess, err := s.GetEffectiveSecuritySettings()
		if err != nil {
			return nil, err
		}
		mss.SecurityType = s.SecurityType
		mss.SecuritySettings = ess
	}

	return mss, nil
}

```

# `transport/internet/sockopt.go`

这两段代码定义了两个名为`isTCPSocket`和`isUDPSocket`的函数，用于判断给定的网络串是否为TCP或UDP类型。

`isTCPSocket`函数接受一个字符串参数（网络串），返回一个布尔值（`true`或`false`）。在网络串"tcp"、"tcp4"或"tcp6"时，函数返回`true`，否则返回`false`。

`isUDPSocket`函数与`isTCPSocket`类似，但接受了不同的网络串。它也返回一个布尔值，`true`或`false`。在网络串"udp"、"udp4"或"udp6"时，函数返回`true`，否则返回`false`。


```go
package internet

func isTCPSocket(network string) bool {
	switch network {
	case "tcp", "tcp4", "tcp6":
		return true
	default:
		return false
	}
}

func isUDPSocket(network string) bool {
	switch network {
	case "udp", "udp4", "udp6":
		return true
	default:
		return false
	}
}

```

# `transport/internet/sockopt_darwin.go`

这段代码定义了一个名为`internet`的包，其中包含了一些与TCP套接字相关的常量。具体来说，这些常量定义了TCP fast open选项的值，包括`TCP_FASTOPEN`、`TCP_FASTOPEN_SERVER`和`TCP_FASTOPEN_CLIENT`。这些常量的值可以用来设置TCP连接的快速打开选项，从而加快TCP连接的建立速度。


```go
package internet

import (
	"syscall"
)

const (
	// TCP_FASTOPEN is the socket option on darwin for TCP fast open.
	TCP_FASTOPEN = 0x105
	// TCP_FASTOPEN_SERVER is the value to enable TCP fast open on darwin for server connections.
	TCP_FASTOPEN_SERVER = 0x01
	// TCP_FASTOPEN_CLIENT is the value to enable TCP fast open on darwin for client connections.
	TCP_FASTOPEN_CLIENT = 0x02
)

```

此函数的作用是设置入站套接字（TCP或UDP）的选项，并返回一个错误。

具体来说，它进行以下操作：

1. 如果网络是TCP，它检查`config.Tfo`的值是否为`SocketConfig_Enable`。如果是，它会执行以下操作：
   a. 如果已经开启了TCP Fast Open，它将调用`syscall.SetsockoptInt`函数，设置`int(fd)`为TCP协议的传输类型，设置选项为`TCP_FASTOPEN`，设置选项为`TCP_FASTOPEN_CLIENT`。
   b. 如果已经关闭了TCP Fast Open，它将调用`syscall.SetsockoptInt`函数，设置`int(fd)`为TCP协议的传输类型，设置选项为`TCP_FASTOPEN`，设置选项为`0`。

2. 如果网络是UDP，它将执行以下操作：

   a. 如果已经开启了UDP选项，它将调用`syscall.SetsockoptInt`函数，设置`int(fd)`为UDP协议的传输类型，设置选项为`ADV_OPTION`，设置选项为`4`，表示启用二进制编码。

  


```go
func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, TCP_FASTOPEN_CLIENT); err != nil {
				return err
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, 0); err != nil {
				return err
			}
		}
	}

	return nil
}

```

该函数`applyInboundSocketOptions`的作用是管理服务器接收端的Socket选项。它接收三个参数：`network`表示网络类型，`fd`表示套接字文件描述符，`config`是一个指向`SocketConfig`结构体的指针。

函数首先检查给定的网络类型是否为TCP套接字类型。如果是，函数将尝试调用`syscall.SetsockoptInt`函数来配置套接字。如果函数成功，它将在配置中设置`SocketConfig_Enable`，并且如果之前已经存在的话则不会设置`SocketConfig_Disable`。

如果给定的网络类型不是TCP套接字类型，则函数的行为将取决于`config`中`Tfo`字段的值。如果是`SocketConfig_Enable`，则函数将在套接字文件描述符上启用快速打开。如果是`SocketConfig_Disable`，则函数将在套接字文件描述符上禁用快速打开。


```go
func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, TCP_FASTOPEN_SERVER); err != nil {
				return err
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, 0); err != nil {
				return err
			}
		}
	}

	return nil
}

```

这三段代码都是针对网络套接字（socket）使用的函数，具体解释如下：

1. `bindAddr`函数的作用是绑定套接字到指定的端口上，并返回一个错误。它接受一个套接字描述符（fd）、一个指向字节数组的指针（address）和一个端口号（port）。函数首先检查输入参数是否为有效的端口号，然后将套接字和地址绑定到指定的端口上，如果成功，则返回`nil`表示无错误。

2. `setReuseAddr`函数的作用是设置套接字可重用的地址。它接受一个套接字描述符（fd）。函数执行的操作是：复制原始的地址到一个新的缓冲区，然后将原始地址和新的缓冲区复制到一个指针上，最后将指针返回。如果调用此函数成功，则套接字将保持可重用。

3. `setReusePort`函数的作用是设置套接字可重用的端口号。它接受一个套接字描述符（fd）和一个端口号（port）。函数执行的操作是：将端口号复制到一个新的缓冲区，然后将原始端口号和新的缓冲区复制到一个指针上，最后将指针返回。如果调用此函数成功，则套接字将保持可重用。


```go
func bindAddr(fd uintptr, address []byte, port uint32) error {
	return nil
}

func setReuseAddr(fd uintptr) error {
	return nil
}

func setReusePort(fd uintptr) error {
	return nil
}

```

# `transport/internet/sockopt_freebsd.go`

这段代码的作用是定义了一个名为`internet`的包，然后在`net`包中定义了一些与网络相关的函数。

具体来说，这些函数用于实现网络套接字输出操作。函数的参数包括套接字类型、发送数据缓冲区、发送缓冲区大小等。函数的实现包括对套接字进行相应的操作，如设置套接字为输出模式、将发送数据写入套接字缓冲区等。

通过这些函数，可以实现向远程主机发送数据并接收响应的功能。


```go
package internet

import (
	"encoding/binary"
	"net"
	"os"
	"syscall"
	"unsafe"

	"golang.org/x/sys/unix"
)

const (
	sysPFINOUT     = 0x0
	sysPFIN        = 0x1
	sysPFOUT       = 0x2
	sysPFFWD       = 0x3
	sysDIOCNATLOOK = 0xc04c4417
)

```

这是一个定义了 `pfiocNatlook` 结构的变量，它是一个 `struct` 类型，每个 `pfiocNatlook` 类型的变量都具有相同的字节数组 `Saddr`、`Daddr`、`Rsaddr` 和 `Rdaddr`，以及 `Sport`、`Dport`、`Rsport` 和 `Rdport` 字段，这些字段都是该结构中所有字段的总称。

`Af` 和 `Proto` 字段定义了网络协议，它们确定了数据如何传输。具体来说，`Af` 字段定义了使用的协议类型，而 `Proto` 字段定义了该协议的具体版本。

`Direction` 字段定义了数据从源地址流向目的地址的方向，它是一个 8 位二进制数，具体值可以在文档中查找。

`Pad` 字段是一个固定长度的字节数组，用于填充 `pfiocNatlook` 结构中的一些字段，以使其在内存中占据正确的长度。




```go
type pfiocNatlook struct {
	Saddr     [16]byte /* pf_addr */
	Daddr     [16]byte /* pf_addr */
	Rsaddr    [16]byte /* pf_addr */
	Rdaddr    [16]byte /* pf_addr */
	Sport     uint16
	Dport     uint16
	Rsport    uint16
	Rdport    uint16
	Af        uint8
	Proto     uint8
	Direction uint8
	Pad       [1]byte
}

```

这段代码定义了一个名为`ioctl`的函数，以及一个名为`pfiocNatlook`的变量。函数接受一个`uintptr`类型的参数`s`表示串口号，一个`int`类型的参数`ioc`表示IOC_MSGUI_REQISTAT，以及一个`[]byte`类型的参数`b`表示需要发送给系统调用的数据。

函数的作用是调用`syscall.Syscall`函数，通过参数`s`、`ioc`和`uintptr(unsafe.Pointer(&b[0]))`将`ioc_msggui_reqstat`发送到系统调用者，并返回一个包含多个字节数据的缓冲区`b`中。如果发送错误或者返回值为非零，则返回一个错误。

`pfiocNatlook`类型的变量`nl`被定义为一个指向`pfiocNatlook`类型的指针。该类型的定义没有给出具体的实现，只是声明了一个名为`rdPort`的函数，返回一个表示从`RDPORT`结构体中读取的端口号。


```go
const (
	sizeofPfiocNatlook = 0x4c
	soReUsePort        = 0x00000200
	soReUsePortLB      = 0x00010000
)

func ioctl(s uintptr, ioc int, b []byte) error {
	if _, _, errno := syscall.Syscall(syscall.SYS_IOCTL, s, uintptr(ioc), uintptr(unsafe.Pointer(&b[0]))); errno != 0 {
		return error(errno)
	}
	return nil
}
func (nl *pfiocNatlook) rdPort() int {
	return int(binary.BigEndian.Uint16((*[2]byte)(unsafe.Pointer(&nl.Rdport))[:]))
}

```

This is a function that creates a TCP-to-UDP packet socket and binds it to a specific IP address and port. It then sets various options for the socket, such as the flow control and send buffer sizes, and binds the socket to the specified IP address and port. If the IP address is specified using a URL, the function parses the IP address from the URL and copies it to the IP field of the packet socket. If the IP address is given as an IPv6 address, the function parses the address from the given string and copies it to the IP field of the packet socket. The function also creates a UDP socket and binds it to the specified IP address and port, sets the options for the socket, and binds it to the specified IP address and port. If any errors occur during the creation or binding of the socket, the function returns an error.


```go
func (nl *pfiocNatlook) setPort(remote, local int) {
	binary.BigEndian.PutUint16((*[2]byte)(unsafe.Pointer(&nl.Sport))[:], uint16(remote))
	binary.BigEndian.PutUint16((*[2]byte)(unsafe.Pointer(&nl.Dport))[:], uint16(local))
}

// OriginalDst uses ioctl to read original destination from /dev/pf
func OriginalDst(la, ra net.Addr) (net.IP, int, error) {
	f, err := os.Open("/dev/pf")
	if err != nil {
		return net.IP{}, -1, newError("failed to open device /dev/pf").Base(err)
	}
	defer f.Close()
	fd := f.Fd()
	b := make([]byte, sizeofPfiocNatlook)
	nl := (*pfiocNatlook)(unsafe.Pointer(&b[0]))
	var raIP, laIP net.IP
	var raPort, laPort int
	switch la.(type) {
	case *net.TCPAddr:
		raIP = ra.(*net.TCPAddr).IP
		laIP = la.(*net.TCPAddr).IP
		raPort = ra.(*net.TCPAddr).Port
		laPort = la.(*net.TCPAddr).Port
		nl.Proto = syscall.IPPROTO_TCP
	case *net.UDPAddr:
		raIP = ra.(*net.UDPAddr).IP
		laIP = la.(*net.UDPAddr).IP
		raPort = ra.(*net.UDPAddr).Port
		laPort = la.(*net.UDPAddr).Port
		nl.Proto = syscall.IPPROTO_UDP
	}
	if raIP.To4() != nil {
		if laIP.IsUnspecified() {
			laIP = net.ParseIP("127.0.0.1")
		}
		copy(nl.Saddr[:net.IPv4len], raIP.To4())
		copy(nl.Daddr[:net.IPv4len], laIP.To4())
		nl.Af = syscall.AF_INET
	}
	if raIP.To16() != nil && raIP.To4() == nil {
		if laIP.IsUnspecified() {
			laIP = net.ParseIP("::1")
		}
		copy(nl.Saddr[:], raIP)
		copy(nl.Daddr[:], laIP)
		nl.Af = syscall.AF_INET6
	}
	nl.setPort(raPort, laPort)
	ioc := uintptr(sysDIOCNATLOOK)
	for _, dir := range []byte{sysPFOUT, sysPFIN} {
		nl.Direction = dir
		err = ioctl(fd, int(ioc), b)
		if err == nil || err != syscall.ENOENT {
			break
		}
	}
	if err != nil {
		return net.IP{}, -1, os.NewSyscallError("ioctl", err)
	}

	odPort := nl.rdPort()
	var odIP net.IP
	switch nl.Af {
	case syscall.AF_INET:
		odIP = make(net.IP, net.IPv4len)
		copy(odIP, nl.Rdaddr[:net.IPv4len])
	case syscall.AF_INET6:
		odIP = make(net.IP, net.IPv6len)
		copy(odIP, nl.Rdaddr[:])
	}
	return odIP, odPort, nil
}

```

This is a Go function that sets the options for a socket. It takes a single parameter, `config`, which is a struct that specifies the options to set.

The options that the function can set are:

* `SO_REUSEADDR`
* `SO_CLOEXEC`
* `SO_LOKEN`
* `SO_PEER_DELAY`
* `SO_RCOMMAND`
* `SO_RECORD`
* `SO_W忙碌
* `SO_USER_COOKIE`
* `TFO`

The function uses the `syscall` package to communicate with the operating system and the `net` package to handle tasks related to the IP addresses and the IPV6 address.

The function first checks if the socket is a TCP or UDP socket and sets the appropriate options based on the `TFO` configuration. If the `TFO` is enabled, the function sets the options for the outbound IPV6 connection if it is enabled.

It also sets the options for the TCP connection, including the `SO_REUSEADDR`, `SO_CLOEXEC`, `SO_LOKEN`, `SO_PEER_DELAY`, `SO_RCOMMAND`, `SO_RECORD`, and `SO_W忙碌` options.

Finally, it sets the `SO_USER_COOKIE` option if it is enabled and returns the `nil` if any errors occurred.


```go
func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
	if config.Mark != 0 {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_USER_COOKIE, int(config.Mark)); err != nil {
			return newError("failed to set SO_USER_COOKIE").Base(err)
		}
	}

	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 1); err != nil {
				return newError("failed to set TCP_FASTOPEN_CONNECT=1").Base(err)
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 0); err != nil {
				return newError("failed to set TCP_FASTOPEN_CONNECT=0").Base(err)
			}
		}
	}

	if config.Tproxy.IsEnabled() {
		ip, _, _ := net.SplitHostPort(address)
		if net.ParseIP(ip).To4() != nil {
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IP, syscall.IP_BINDANY, 1); err != nil {
				return newError("failed to set outbound IP_BINDANY").Base(err)
			}
		} else {
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IPV6, syscall.IPV6_BINDANY, 1); err != nil {
				return newError("failed to set outbound IPV6_BINDANY").Base(err)
			}
		}
	}
	return nil
}

```

这段代码是一个名为`applyInboundSocketOptions`的函数，它接受三个参数：

1. `network`：表示网络类型，如TCP或UDP。
2. `fd`：是一个文件描述符，用于套接字操作。
3. `config`：是一个`SocketConfig`结构体，包含了设置套接字选项的配置信息。

函数的作用是设置套接字选项，以使套接字支持特定的功能。以下是函数的详细解释：

1. 检查 `config` 是否为空。如果是，函数将返回一个名为`newError`的错误。
2. 如果 `network` 是TCP，函数根据 `config.Tfo` 的值来设置套接字选项。
3. 如果 `config.Tproxy` 启用了代理，函数根据 `config.Tfo` 的值来设置套接字选项。
4. 如果 `config.Mark` 是1，函数将尝试设置 `SO_USER_COOKIE`，但失败时会返回一个错误。
5. 如果 `config.Tfo` 是TCP_FASTOPEN，函数将尝试设置 `TCP_FASTOPEN`，但失败时会返回一个错误。
6. 如果 `config.Tproxy.IsEnabled()` 是启用，函数将尝试设置 `IPV6_BINDANY`，但失败时会返回一个错误。
7. 如果所有设置都成功，函数返回`nil`，表示操作成功。


```go
func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
	if config.Mark != 0 {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_USER_COOKIE, int(config.Mark)); err != nil {
			return newError("failed to set SO_USER_COOKIE").Base(err)
		}
	}
	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 1); err != nil {
				return newError("failed to set TCP_FASTOPEN=1").Base(err)
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 0); err != nil {
				return newError("failed to set TCP_FASTOPEN=0").Base(err)
			}
		}
	}

	if config.Tproxy.IsEnabled() {
		if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IPV6, syscall.IPV6_BINDANY, 1); err != nil {
			if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IP, syscall.IP_BINDANY, 1); err != nil {
				return newError("failed to set inbound IP_BINDANY").Base(err)
			}
		}
	}

	return nil
}

```

该函数的作用是创建一个套接字并绑定到指定的文件描述符(fd)，然后设置套接字可重用，即可以重复使用已经绑定的套接字。

函数的第一个参数fd是一个uintptr类型的整数，表示文件描述符(例如，一个文件 descriptor的计数值)。第二个参数ip是一个长度为net.IPv4len或net.IPv6len的切片，指定套接字使用的IP地址。第三个参数port是一个uint32类型的整数，指定套接字使用的端口号。

函数首先调用setReuseAddr和setReusePort函数，设置套接字可重用和可重入。接下来，定义一个syscall.Sockaddr类型的变量sockaddr，用于存储套接字地址信息。

然后，根据ip的类型设置sockaddr的类型。如果ip是net.IPv4len类型，则设置套接字使用的是IPv4地址，如果ip是net.IPv6len类型，则设置套接字使用的是IPv6地址。在设置套接字地址信息之后，使用syscall.Bind函数将套接字绑定到fd上，并返回结果。如果设置过程中出现错误，函数将返回一个错误结果。


```go
func bindAddr(fd uintptr, ip []byte, port uint32) error {
	setReuseAddr(fd)
	setReusePort(fd)

	var sockaddr syscall.Sockaddr

	switch len(ip) {
	case net.IPv4len:
		a4 := &syscall.SockaddrInet4{
			Port: int(port),
		}
		copy(a4.Addr[:], ip)
		sockaddr = a4
	case net.IPv6len:
		a6 := &syscall.SockaddrInet6{
			Port: int(port),
		}
		copy(a6.Addr[:], ip)
		sockaddr = a6
	default:
		return newError("unexpected length of ip")
	}

	return syscall.Bind(int(fd), sockaddr)
}

```

这两个函数都是用于在 Linux 系统中设置套接字的 reuse addr 和 reuse port 选项。通过调用系统调用 `syscall.SetsockoptInt()`，我们可以设置套接字的 reuse addr 和 reuse port 选项。如果设置失败，函数将返回一个错误。

`setReuseAddr()` 函数用于设置服务器套接字的 reuse addr 选项。如果此函数在调用 `syscall.SetsockoptInt()` 时出现错误，将抛出一个警告并返回一个错误。

`setReusePort()` 函数用于设置客户端套接数的 reuse port 选项。如果此函数在调用 `syscall.SetsockoptInt()` 时出现错误，将抛出一个警告并返回一个错误。

注意：`setReuseAddr()` 和 `setReusePort()` 函数都使用 `syscall.SetsockoptInt()` 调用。这些函数的第二个参数 `syscall.SOL_SOCKET` 和 `syscall.SO_REUSEADDR` 都表示套接字的套接字类型。第三个参数 `soReUsePortLB` 和 `soReUsePort` 分别表示服务器套接字的任用模式和客户端套接数的模式。


```go
func setReuseAddr(fd uintptr) error {
	if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil {
		return newError("failed to set SO_REUSEADDR").Base(err).AtWarning()
	}
	return nil
}

func setReusePort(fd uintptr) error {
	if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, soReUsePortLB, 1); err != nil {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, soReUsePort, 1); err != nil {
			return newError("failed to set SO_REUSEPORT").Base(err).AtWarning()
		}
	}
	return nil
}

```

# `transport/internet/sockopt_linux.go`

这段代码定义了一个名为`internet`的包，其中包含以下几个函数：

1. `TCP_FASTOPEN`是一个与TCP连接相关的函数，用于开启TCP连接的快速打开模式。
2. `TCP_FASTOPEN_CONNECT`是`TCP_FASTOPEN`函数的别名，用于加速TCP连接的打开。
3. `net`包中的一个名为`netcall`的函数，用于向目标IP地址发送一个TCP数据报。
4. `syscall`包中的一个名为`syscall_宏定义`的函数，用于获取当前操作系统调用号对应的函数地址。
5. `unix`包中的一个名为`TCP_port`的函数，用于获取TCP连接的本地端口号。


```go
package internet

import (
	"net"
	"syscall"

	"golang.org/x/sys/unix"
)

const (
	// For incoming connections.
	TCP_FASTOPEN = 23
	// For out-going connections.
	TCP_FASTOPEN_CONNECT = 30
)

```

该函数接受一个文件描述符(fd)、一个IP地址数组(ip)和一个端口号(port)，并返回一个错误。它的作用是确保文件描述符被正确绑定到服务器套接字上，并允许服务器套接字监听来自客户端的连接请求。

函数的实现包括以下步骤：

1. 调用setReuseAddr和setReusePort函数，确保文件描述符可以被绑定到服务器套接字上。

2. 创建一个syscall.Sockaddr对象，该对象表示服务器套接字。然后使用switch语句检查ip的长度，并根据其类型设置相应的 sockaddr 对象。

3. 使用syscall.Bind函数，将文件描述符与服务器套接字绑定。

4. 返回一个错误，以便在需要时进行抛出。


```go
func bindAddr(fd uintptr, ip []byte, port uint32) error {
	setReuseAddr(fd)
	setReusePort(fd)

	var sockaddr syscall.Sockaddr

	switch len(ip) {
	case net.IPv4len:
		a4 := &syscall.SockaddrInet4{
			Port: int(port),
		}
		copy(a4.Addr[:], ip)
		sockaddr = a4
	case net.IPv6len:
		a6 := &syscall.SockaddrInet6{
			Port: int(port),
		}
		copy(a6.Addr[:], ip)
		sockaddr = a6
	default:
		return newError("unexpected length of ip")
	}

	return syscall.Bind(int(fd), sockaddr)
}

```

这段代码是一个名为`applyOutboundSocketOptions`的函数，它接受一个网络名（network）和一个目标地址（address），并返回一个错误对象（err）表示操作是否成功。

函数的作用如下：

1. 如果`config`参数中`Mark`字段为1，那么先尝试调用`syscall.SetsockoptInt`函数设置套接字标记（SO_MARK），如果失败则返回一个错误。

2. 如果网络名是TCP套接字，那么检查`config.Tfo`字段，如果为`SocketConfig_Enable`，则尝试调用`syscall.SetsockoptInt`函数设置套接字快速打开连接（TCP_FASTOPEN_CONNECT），如果失败则返回一个错误。

3. 如果`config.Tproxy.IsEnabled()`成立，则尝试调用`syscall.SetsockoptInt`函数设置IP传输协议（IP_TRANSPARENT），如果失败则返回一个错误。

函数的实现主要围绕着设置套接字的标记、快速打开连接以及IP传输协议。通过调用`syscall.SetsockoptInt`函数，可以设置套接字的标记、快速打开连接以及IP传输协议，从而实现对TCP套接字的配置。


```go
func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
	if config.Mark != 0 {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK, int(config.Mark)); err != nil {
			return newError("failed to set SO_MARK").Base(err)
		}
	}

	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN_CONNECT, 1); err != nil {
				return newError("failed to set TCP_FASTOPEN_CONNECT=1").Base(err)
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN_CONNECT, 0); err != nil {
				return newError("failed to set TCP_FASTOPEN_CONNECT=0").Base(err)
			}
		}
	}

	if config.Tproxy.IsEnabled() {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
			return newError("failed to set IP_TRANSPARENT").Base(err)
		}
	}

	return nil
}

```

This is a function that sets the socket options for a given file descriptor (fd) on a Linux system. It takes in a single argument, which is a file descriptor with information about the socket, and returns either a newly created error object if any errors occurred or the original error object if the operation was successful.

The function first sets theSO_MARK bit, which stands for set options and mark, using the value of the configure.Mark field. It then sets the TCP flow option using theSO_TFO type field, and either enables or disables it.

If the socket is a TCP socket, the function sets the TCP fast open option using theSO_FTPO type field, and sets themark bit, which is an optional binary mark to indicate the socket's mark state.

If the socket is an UDP socket, the function sets the receive original destination address option using theSO_REUSEADDR type field and sets theIPV6RECEIVER type field, and sets theIPV6RECEIVER type field, if the socket is an IPv6 socket.

The function returns either a newly created error object if any errors occurred or the original error object if the operation was successful.


```go
func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
	if config.Mark != 0 {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK, int(config.Mark)); err != nil {
			return newError("failed to set SO_MARK").Base(err)
		}
	}
	if isTCPSocket(network) {
		switch config.Tfo {
		case SocketConfig_Enable:
			if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN, 1); err != nil {
				return newError("failed to set TCP_FASTOPEN=1").Base(err)
			}
		case SocketConfig_Disable:
			if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN, 0); err != nil {
				return newError("failed to set TCP_FASTOPEN=0").Base(err)
			}
		}
	}

	if config.Tproxy.IsEnabled() {
		if err := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
			return newError("failed to set IP_TRANSPARENT").Base(err)
		}
	}

	if config.ReceiveOriginalDestAddress && isUDPSocket(network) {
		err1 := syscall.SetsockoptInt(int(fd), syscall.SOL_IPV6, unix.IPV6_RECVORIGDSTADDR, 1)
		err2 := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_RECVORIGDSTADDR, 1)
		if err1 != nil && err2 != nil {
			return err1
		}
	}

	return nil
}

```

这两个函数都是用于在网络套接字中设置 reuse 地址和端口的选项。通过调用 syscall.SetsockoptInt 和 syscall.SetsockoptInt，我们可以设置套接字中 TCP 或 UDP 协议栈的 reuse 选项。

具体来说，这两个函数分别尝试设置套接字中 TCP 或 UDP 协议栈的 reuse 选项。如果设置成功，则返回 nil，否则返回一个自定义错误。


```go
func setReuseAddr(fd uintptr) error {
	if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil {
		return newError("failed to set SO_REUSEADDR").Base(err).AtWarning()
	}
	return nil
}

func setReusePort(fd uintptr) error {
	if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, unix.SO_REUSEPORT, 1); err != nil {
		return newError("failed to set SO_REUSEPORT").Base(err).AtWarning()
	}
	return nil
}

```

# `transport/internet/sockopt_linux_test.go`

这段代码是关于测试网络连线的，主要是通过使用v2ray.com的tcp服务器来进行测试。代码的作用是测试服务器接受一个标记为1的tcp连接，并使用该连接进行通信。

具体来说，代码中首先创建一个tcp服务器，使用默认的系统调用dialer.Dial来连接到v2ray.com的tcp服务器。然后使用服务器提供的msg_processor函数来处理接收到的tcp数据包，该函数会将数据包直接返回，不会对数据包进行进一步的处理。

接下来，代码使用DefaultSystemDialer默认的连接器来连接到服务器，并使用syscall.GetsockoptInt函数获取一个基于标记的socket。然后，代码使用该socket进行控制调用，包括获取套接字标记、发送消息等。如果接收到的是标记为1的连接，代码会打印出标记并退出测试，否则会进行一系列的测试来验证服务器是否正常工作。


```go
package internet_test

import (
	"context"
	"syscall"
	"testing"

	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/testing/servers/tcp"
	. "v2ray.com/core/transport/internet"
)

func TestSockOptMark(t *testing.T) {
	t.Skip("requires CAP_NET_ADMIN")

	tcpServer := tcp.Server{
		MsgProcessor: func(b []byte) []byte {
			return b
		},
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	const mark = 1
	dialer := DefaultSystemDialer{}
	conn, err := dialer.Dial(context.Background(), nil, dest, &SocketConfig{Mark: mark})
	common.Must(err)
	defer conn.Close()

	rawConn, err := conn.(*net.TCPConn).SyscallConn()
	common.Must(err)
	err = rawConn.Control(func(fd uintptr) {
		m, err := syscall.GetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK)
		common.Must(err)
		if mark != m {
			t.Fatal("unexpected connection mark", m, " want ", mark)
		}
	})
	common.Must(err)
}

```

# `transport/internet/sockopt_other.go`

这段代码定义了三个函数：`applyOutboundSocketOptions`、`applyInboundSocketOptions` 和 `bindAddr`。它们都属于名为 `internet` 的包。

这些函数的主要作用是设置网络套接字的选项，包括设置网络字符串、IP地址和端口号等。以下是这些函数的一些详细信息：

1. `applyOutboundSocketOptions`：设置网络套接字的出站选项。它的参数包括：网络字符串、目标IP地址和套接字文件描述符（FD）。

2. `applyInboundSocketOptions`：设置网络套接字的入站选项。它的参数包括：网络字符串、目标套接字文件描述符和套接字配置结构体。

3. `bindAddr`：设置套接字的绑定地址和端口号。它的参数包括：套接字文件描述符（FD）、目标IP地址和端口号。


```go
// +build js dragonfly netbsd openbsd solaris

package internet

func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
	return nil
}

func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
	return nil
}

func bindAddr(fd uintptr, ip []byte, port uint32) error {
	return nil
}

```

这两个函数的目的是在使用操作系统套接字时，设置一个套接字的可重用地址。

在 Linux 系统中，每个套接字都有自己的独立 IP 地址和一个端口号。当一个应用程序需要通过网络与另一个应用程序通信时，它需要在本地操作系统中查找可用的套接字。如果找到了一个可用的套接字，并且该套接字的端口号与当前正在使用的端口号相同，那么应用程序就可以继续使用该套接字，而不是创建一个新的套接字。

这两个函数分别尝试在使用 `fd` 变量表示的套接字上设置可重用地址和端口号。如果设置成功，则返回 `nil`；否则返回一个错误。


```go
func setReuseAddr(fd uintptr) error {
	return nil
}

func setReusePort(fd uintptr) error {
	return nil
}

```

# `transport/internet/sockopt_test.go`

这段代码是一个用于测试TCP协议的函数，它演示了如何使用Go标准库中的`testing`包来测试TCP客户端和服务器之间的通信。

具体来说，这段代码创建了一个TCP服务器，通过`tcp.Server`函数启动了一个TCP端点，接收方处理消息的方式是通过`tcp.MsgProcessor`函数，这个函数接收一个字节数组并返回，所以它不会对接收到的消息进行进一步的处理。然后，它通过一个闭包函数启动了TCP服务器并返回一个客户端的`SocketConfig`结构体，这个结构体指定了服务器如何与客户端通信，包括端口号、超时时间和启用TCP套接字等功能。

在客户端，它使用Go标准库中的`DefaultSystemDialer`函数来选择一个系统调用，通过调用`tcp.Dial`函数来建立一个TCP连接，并把一个字节数组发送到服务器。然后，通过`conn.Write`函数发送一个字节数组到服务器，通过`conn.Close`函数关闭TCP连接。

最后，该函数创建一个缓冲区`b`，并使用`cmp.Diff`函数比较接收到的字节数组和预期字节数组之间的差异，如果两个字节数组之间存在差异，则函数会打印出来并返回。


```go
package internet_test

import (
	"context"
	"testing"

	"github.com/google/go-cmp/cmp"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/testing/servers/tcp"
	. "v2ray.com/core/transport/internet"
)

func TestTCPFastOpen(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: func(b []byte) []byte {
			return b
		},
	}
	dest, err := tcpServer.StartContext(context.Background(), &SocketConfig{Tfo: SocketConfig_Enable})
	common.Must(err)
	defer tcpServer.Close()

	ctx := context.Background()
	dialer := DefaultSystemDialer{}
	conn, err := dialer.Dial(ctx, nil, dest, &SocketConfig{
		Tfo: SocketConfig_Enable,
	})
	common.Must(err)
	defer conn.Close()

	_, err = conn.Write([]byte("abcd"))
	common.Must(err)

	b := buf.New()
	common.Must2(b.ReadFrom(conn))
	if r := cmp.Diff(b.Bytes(), []byte("abcd")); r != "" {
		t.Fatal(r)
	}
}

```

# `transport/internet/sockopt_windows.go`

这段代码定义了一个名为 `setTFO` 的函数，该函数接受一个 `SocketConfig_TCPFastOpenState` 类型的参数 `settings`。函数的作用是根据 `settings` 的值，设置 TCP 文件的快速打开状态，从而实现 TCP 文件的快速打开功能。

具体来说，函数内部首先根据 `settings` 的值选择一个正确的函数调用，然后调用 `syscall.SetsockoptInt` 函数，设置 TCP 文件的快速打开状态。如果设置成功，函数返回 `nil` 表示操作成功；否则，函数返回一个非 `nil` 值的错误，具体错误类型取决于调用 `syscall.SetsockoptInt` 函数时出现的错误。


```go
package internet

import (
	"syscall"
)

const (
	TCP_FASTOPEN = 15
)

func setTFO(fd syscall.Handle, settings SocketConfig_TCPFastOpenState) error {
	switch settings {
	case SocketConfig_Enable:
		if err := syscall.SetsockoptInt(fd, syscall.IPPROTO_TCP, TCP_FASTOPEN, 1); err != nil {
			return err
		}
	case SocketConfig_Disable:
		if err := syscall.SetsockoptInt(fd, syscall.IPPROTO_TCP, TCP_FASTOPEN, 0); err != nil {
			return err
		}
	}
	return nil
}

```

这两函数函数用于配置套接字选项，并连接到网络接口。`applyOutboundSocketOptions`用于连接到远程主机，并返回一个错误。`applyInboundSocketOptions`用于连接到本地主机，并返回一个错误。

这两个函数都在`syscall.Handle`函数的输入参数中获取套接字文件描述符（通常是一个套接字ID）。然后，它们使用`setTFO`函数来设置套接字选项。如果设置套接字选项时出现错误，函数将返回错误。

如果套接字连接到远程主机，这两个函数将使用`setTCP`函数来设置套接字选项，并返回一个错误。如果套接字连接到本地主机，这两个函数将使用`setTCP`函数来设置套接字选项，并返回一个错误。


```go
func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
	if isTCPSocket(network) {
		if err := setTFO(syscall.Handle(fd), config.Tfo); err != nil {
			return err
		}

	}

	return nil
}

func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
	if isTCPSocket(network) {
		if err := setTFO(syscall.Handle(fd), config.Tfo); err != nil {
			return err
		}
	}

	return nil
}

```

这两个函数都是用于在网络套接字中设置套接字地址和端口号的功能。

函数`bindAddr`接受一个文件描述符(fd)，一个IP地址数组`ip`和一个端口号`port`。它返回一个`error`类型的值。如果函数正常完成，它将返回`nil`；如果函数失败，它将返回一个适当的错误类型，例如`net`错误类型。

函数`setReuseAddr`接受一个文件描述符(fd)，它将存储套接字地址。它返回一个`error`类型的值。如果函数正常完成，它将返回`nil`；如果函数失败，它将返回一个适当的错误类型，例如`net`错误类型。

函数`setReusePort`接受一个文件描述符(fd)，它将存储套接字端口号。它返回一个`error`类型的值。如果函数正常完成，它将返回`nil`；如果函数失败，它将返回一个适当的错误类型，例如`net`错误类型。


```go
func bindAddr(fd uintptr, ip []byte, port uint32) error {
	return nil
}

func setReuseAddr(fd uintptr) error {
	return nil
}

func setReusePort(fd uintptr) error {
	return nil
}

```

# `transport/internet/system_dialer.go`

这段代码是一个 Go 语言 package，它的作用是创建一个名为 "internet" 的包。通过导入其他包，实现了一些与网络相关的功能。

具体来说，这段代码实现了以下功能：

1. 导入 "context"、"syscall" 和 "time" 包，以便在 "internet" 包中使用它们的功能。

2. 导入自 "v2ray.com/core/common/net"，这样我们就可以使用 "net" 包中的内容了。

3. 实现了一个名为 "DefaultSystemDialer" 的名为 "EffectiveSystemDialer" 的类型，并在 "internet" 包的 "EffectiveSystemDialer" 函数中使用它。

4. 实现了一个名为 "EffectiveSystemDialer" 的类型，并在 "internet" 包的 "DefaultSystemDialer" 函数中使用它。

5. 通过调用 "DefaultSystemDialer" 和 "EffectiveSystemDialer" 函数，实现了系统拨号连接的功能。


```go
package internet

import (
	"context"
	"syscall"
	"time"

	"v2ray.com/core/common/net"
	"v2ray.com/core/common/session"
)

var (
	effectiveSystemDialer SystemDialer = &DefaultSystemDialer{}
)

```

这段代码定义了一个名为SystemDialer的接口类型，以及一个名为DefaultSystemDialer的 struct 类型。

SystemDialer接口定义了一个名为Dial的函数，该函数接收四个参数：上下文上下文、源地址、目标地址和套接字配置。函数返回一个网络连接错误和一个错误对象。

DefaultSystemDialer结构体实现了一个SystemDialer接口，包含一个包含控制器对象的数组。

另外，还实现了一个名为resolveSrcAddr的函数，该函数接收两个参数：网络类型和源地址。函数根据网络类型将源地址转换为相应的网络地址，如果源地址为空或者网络类型为TCP，则返回一个默认的网络地址。如果网络类型为UDP，则将源地址转换为相应的网络地址。函数返回一个网络地址对象。


```go
type SystemDialer interface {
	Dial(ctx context.Context, source net.Address, destination net.Destination, sockopt *SocketConfig) (net.Conn, error)
}

type DefaultSystemDialer struct {
	controllers []controller
}

func resolveSrcAddr(network net.Network, src net.Address) net.Addr {
	if src == nil || src == net.AnyIP {
		return nil
	}

	if network == net.Network_TCP {
		return &net.TCPAddr{
			IP:   src.IP(),
			Port: 0,
		}
	}

	return &net.UDPAddr{
		IP:   src.IP(),
		Port: 0,
	}
}

```

This is a function that sets up a connection to a remote host using the provided `destAddr` and `src` network addresses. It returns a `net.Conn` if the connection was successful, or an error if one occurred.

The function first resolves the `destAddr` to a network address using the `resolveSrcAddr` function, and then sets up a `net.Dialer` to establish the connection. If the user specifies options for the connection (e.g. a timeout), the `Dialer` will use that information to control the connection.

If the user specifies a syscall.ControlledConnection object and any controllers, those will be applied to the `Dialer`. The `Control` method of the `Dialer` will be used to handle the actual connection process. This method takes a function as an argument that will be called with the `net.Conn` object as its input.

The function uses the `applyOutboundSocketOptions`, `bindAddr`, `hasBindAddr`, `net.Network_UDP`, `syscall.RawConn`, `net.Dialer`, `net.ControlledConnection`, and `syscall.RawControlledConnection` types.


```go
func hasBindAddr(sockopt *SocketConfig) bool {
	return sockopt != nil && len(sockopt.BindAddress) > 0 && sockopt.BindPort > 0
}

func (d *DefaultSystemDialer) Dial(ctx context.Context, src net.Address, dest net.Destination, sockopt *SocketConfig) (net.Conn, error) {
	if dest.Network == net.Network_UDP && !hasBindAddr(sockopt) {
		srcAddr := resolveSrcAddr(net.Network_UDP, src)
		if srcAddr == nil {
			srcAddr = &net.UDPAddr{
				IP:   []byte{0, 0, 0, 0},
				Port: 0,
			}
		}
		packetConn, err := ListenSystemPacket(ctx, srcAddr, sockopt)
		if err != nil {
			return nil, err
		}
		destAddr, err := net.ResolveUDPAddr("udp", dest.NetAddr())
		if err != nil {
			return nil, err
		}
		return &packetConnWrapper{
			conn: packetConn,
			dest: destAddr,
		}, nil
	}

	dialer := &net.Dialer{
		Timeout:   time.Second * 16,
		DualStack: true,
		LocalAddr: resolveSrcAddr(dest.Network, src),
	}

	if sockopt != nil || len(d.controllers) > 0 {
		dialer.Control = func(network, address string, c syscall.RawConn) error {
			return c.Control(func(fd uintptr) {
				if sockopt != nil {
					if err := applyOutboundSocketOptions(network, address, fd, sockopt); err != nil {
						newError("failed to apply socket options").Base(err).WriteToLog(session.ExportIDToError(ctx))
					}
					if dest.Network == net.Network_UDP && hasBindAddr(sockopt) {
						if err := bindAddr(fd, sockopt.BindAddress, sockopt.BindPort); err != nil {
							newError("failed to bind source address to ", sockopt.BindAddress).Base(err).WriteToLog(session.ExportIDToError(ctx))
						}
					}
				}

				for _, ctl := range d.controllers {
					if err := ctl(network, address, fd); err != nil {
						newError("failed to apply external controller").Base(err).WriteToLog(session.ExportIDToError(ctx))
					}
				}
			})
		}
	}

	return dialer.DialContext(ctx, dest.Network.SystemString(), dest.NetAddr())
}

```

该代码定义了一个名为`packetConnWrapper`的结构体，它包含一个`net.PacketConn`类型的`conn`字段和一个`net.Addr`类型的`dest`字段。

`Close()`函数的实现是关闭套接字连接。

`LocalAddr()`函数的实现是返回套接字本地地址。

`RemoteAddr()`函数的实现是返回套接字远程地址。


```go
type packetConnWrapper struct {
	conn net.PacketConn
	dest net.Addr
}

func (c *packetConnWrapper) Close() error {
	return c.conn.Close()
}

func (c *packetConnWrapper) LocalAddr() net.Addr {
	return c.conn.LocalAddr()
}

func (c *packetConnWrapper) RemoteAddr() net.Addr {
	return c.dest
}

```

该代码定义了一个名为`packetConnWrapper`的结构体，它表示网络连接的套接字。这个结构体包含三个函数，分别为`Write`、`Read`和`SetDeadline`、`SetReadDeadline`。下面分别对这三个函数进行解释：

1. `Write`函数：

这个函数接收一个字节数组`p`，并将其写入到连接的 destination 上。函数的实现是由一个名为`c`的指针变量和一个名为`conn`的指针变量组成的。`conn`是一个`packetConnWrapper`类型的变量，它表示网络连接的套接字。`c.conn`是一个指针，指向一个`packetConnWrapper`类型的变量，它用于与远程服务器建立连接。

2. `Read`函数：

这个函数接收一个字节数组`p`，并返回其中的`n`个字节，以及一个错误。函数的实现与`Write`函数相反，它是从连接的源端读取数据。

3. `SetDeadline`函数：

这个函数接收一个`time.Time`类型的时间`t`，并将其设置为连接的超时时间。函数的实现与`SetDeadline`函数相同，它是一个`packetConnWrapper`类型的函数，用于设置连接的超时时间，以避免连接超时并导致连接失败。


```go
func (c *packetConnWrapper) Write(p []byte) (int, error) {
	return c.conn.WriteTo(p, c.dest)
}

func (c *packetConnWrapper) Read(p []byte) (int, error) {
	n, _, err := c.conn.ReadFrom(p)
	return n, err
}

func (c *packetConnWrapper) SetDeadline(t time.Time) error {
	return c.conn.SetDeadline(t)
}

func (c *packetConnWrapper) SetReadDeadline(t time.Time) error {
	return c.conn.SetReadDeadline(t)
}

```

这段代码定义了一个名为`SimpleSystemDialer`的结构体，它实现了`SystemDialerAdapter`接口。`SystemDialer`接口定义了一个`Dial`方法，用于连接到远程主机。`SimpleSystemDialer`结构体内部实现了一个`WithAdapter`方法，该方法接收一个`SystemDialerAdapter`作为参数，并将其与`SimpleSystemDialer`结构体本身关联起来。

`SetWriteDeadline`函数用于设置写入超时时间，如果设置超时时间为0，则会丢弃任何写入数据，永远不会超时。如果设置一个非零超时时间，则会立即开始尝试写入数据，但如果写入超时时间过短，则会返回一个错误。

该函数返回结果为：`func (c *packetConnWrapper) SetWriteDeadline(t time.Time) error` 如果是错误的，则返回错误，否则返回`nil`。


```go
func (c *packetConnWrapper) SetWriteDeadline(t time.Time) error {
	return c.conn.SetWriteDeadline(t)
}

type SystemDialerAdapter interface {
	Dial(network string, address string) (net.Conn, error)
}

type SimpleSystemDialer struct {
	adapter SystemDialerAdapter
}

func WithAdapter(dialer SystemDialerAdapter) SystemDialer {
	return &SimpleSystemDialer{
		adapter: dialer,
	}
}

```

该代码定义了一个名为SimpleSystemDialer的接口，以及一个名为UseAlternativeSystemDialer的函数。

函数SimpleSystemDialer的作用是在一个使用SystemDialer的上下文中，根据传入的上下文、目标地址和选项（如果有的话）与当前系统 dialer 进行连接，并返回新创建的连接的网络套接字错误。

UseAlternativeSystemDialer函数的作用是接收一个SystemDialer实例，并将其作为上下文传递给SimpleSystemDialer类型的函数。如果接收的系统Dialer为空，则设置为默认的SystemDialer。这样，函数UseAlternativeSystemDialer提供了一个途径，可以指定使用默认的系统Dialer，或者使用传入的自己的系统Dialer。


```go
func (v *SimpleSystemDialer) Dial(ctx context.Context, src net.Address, dest net.Destination, sockopt *SocketConfig) (net.Conn, error) {
	return v.adapter.Dial(dest.Network.SystemString(), dest.NetAddr())
}

// UseAlternativeSystemDialer replaces the current system dialer with a given one.
// Caller must ensure there is no race condition.
//
// v2ray:api:stable
func UseAlternativeSystemDialer(dialer SystemDialer) {
	if dialer == nil {
		effectiveSystemDialer = &DefaultSystemDialer{}
	}
	effectiveSystemDialer = dialer
}

```

此代码的作用是注册一个名为 "RegisterDialerController" 的控制器，该控制器可以用来在将来的拨号活动中对文件描述符进行操作。该控制器仅在默认拨号器有效时才起作用。

具体来说，该代码实现了一个名为 "RegisterDialerController" 的函数，它接受三个参数：

- "ctl"：一个函数类型，表示要执行的操作。
- "network"：网络连接的字符串表示，如 "tcp://example.com:8080"。
- "address"：目标地址的字符串表示，如 "127.0.0.1:80"。
- "fd"：文件描述符，以 UDP 协议的形式指定。

函数首先检查传入的 "ctl" 参数是否为空，如果是，则返回一个错误。否则，检查传入的 "effectiveSystemDialer" 是否是一个有效的系统拨号器，如果是，则使用注册的 "RegisterDialerController" 函数将 "ctl" 添加到 "effectiveSystemDialer" 的控制器列表中。如果 "effectiveSystemDialer" 不是有效的系统拨号器，则返回一个错误。

如果 "RegisterDialerController" 函数成功添加 "ctl" 到 "effectiveSystemDialer" 的控制器列表中，则返回 "nil"，表示操作成功。


```go
// RegisterDialerController adds a controller to the effective system dialer.
// The controller can be used to operate on file descriptors before they are put into use.
// It only works when effective dialer is the default dialer.
//
// v2ray:api:beta
func RegisterDialerController(ctl func(network, address string, fd uintptr) error) error {
	if ctl == nil {
		return newError("nil listener controller")
	}

	dialer, ok := effectiveSystemDialer.(*DefaultSystemDialer)
	if !ok {
		return newError("RegisterListenerController not supported in custom dialer")
	}

	dialer.controllers = append(dialer.controllers, ctl)
	return nil
}

```