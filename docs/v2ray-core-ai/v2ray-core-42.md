# v2ray-core源码解析 42

# `proxy/dokodemo/dokodemo.go`

这段代码是一个 Go 语言编写的命令行工具，它用于生成一个名为 "dokodemo.go" 的二进制文件。这个二进制文件是干什么用的呢？通过 `build` 命令编译出来，然后在 `confonly` 标志下仅输出二进制文件，这样就可以避免在生产环境中误下同名的源代码。

具体来说，这个工具包含以下步骤：

1. 定义了一个名为 `dokodemo` 的包，其中定义了一些用 `package` 关键字声明的函数、变量和类型。
2. 引入了一些用 `import` 语句导入的外部库，包括：
	* v2ray.com/core/common/errors/errorgen，用于引入一个名为 `errorgen` 的函数，该函数可以生成 V2Ray 错误日志。
	* v2ray.com/core/common/signal，用于引入一个名为 `Signal` 的类型，该类型可以用于产生信号，并传递给一些自定义的信号函数。
	* v2ray.com/core/common/net，用于引入一个名为 `net` 的包，该包可以用于网络通信。
	* v2ray.com/core/features/policy，用于引入一个名为 `policy` 的包，该包可以用于生成策略，并应用到会话上。
	* v2ray.com/core/features/routing，用于引入一个名为 `routing` 的包，该包可以用于生成路由。
	* v2ray.com/core/transport/internet，用于引入一个名为 `internet` 的包，该包可以用于生成 HTTP/HTTPS 请求。
3. 定义了一个名为 `build` 的函数，该函数执行以下操作：
	* 使用 `+build` 标志编译 Go 代码。
	* 使用 `!confonly` 标志，这样就可以仅输出二进制文件，而不输出源代码。


```go
// +build !confonly

package dokodemo

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"
	"sync/atomic"
	"time"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/log"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal"
	"v2ray.com/core/common/task"
	"v2ray.com/core/features/policy"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport/internet"
)

```

该代码定义了一个名为“init”的函数，用于初始化DokodemoDoor的配置。函数的实现包括以下步骤：

1. 注册配置器函数以允许函数使用Config类型的参数。
2. 创建一个DokodemoDoor实例，并使用该实例的“Init”方法进行初始化。
3. 如果初始化成功，则返回DokodemoDoor实例和 nil 作为结果，否则返回DokodemoDoor实例和错误。

该函数的作用是协调DokodemoDoor的初始化和配置，确保实现了一个易于使用的接口，同时允许用户在配置器函数中指定DokodemoDoor的初始化和配置。


```go
func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		d := new(DokodemoDoor)
		err := core.RequireFeatures(ctx, func(pm policy.Manager) error {
			return d.Init(config.(*Config), pm, session.SockoptFromContext(ctx))
		})
		return d, err
	}))
}

type DokodemoDoor struct {
	policyManager policy.Manager
	config        *Config
	address       net.Address
	port          net.Port
	sockopt       *session.Sockopt
}

```

这段代码定义了一个名为DokodemoDoor的实例，用于管理Dokodemo门的行为。它实现了两个函数：Init和Network。

Init函数接收三个参数：一个Config实例，一个PolicyManager实例和一个Sockopt实例。函数首先检查Config实例中指定的网络列表是否为空，如果是，则返回一个错误。然后，函数创建一个DokodemoDoor实例，设置其配置、地址和端口，并设置其PolicyManager实例和Sockopt实例。最后，函数返回一个 nil 表示一切顺利。

Network函数实现在Inbound代理的接口中，用于设置Dokodemo门的网络代理设置。函数接收两个参数：一个Config实例和一个PolicyManager实例。函数首先检查Config实例中指定的网络列表是否为空，如果是，则返回一个错误。然后，函数创建一个包含Networks的Config实例，并将它设置为当前Dokodemo门的配置。接下来，函数创建一个包含当前网络的PolicyManager实例，并将其设置为Dokodemo门的PolicyManager实例。最后，函数使用sockopt参数设置Dokodemo门的套接字选项，并返回一个 nil 表示一切顺利。


```go
// Init initializes the DokodemoDoor instance with necessary parameters.
func (d *DokodemoDoor) Init(config *Config, pm policy.Manager, sockopt *session.Sockopt) error {
	if (config.NetworkList == nil || len(config.NetworkList.Network) == 0) && len(config.Networks) == 0 {
		return newError("no network specified")
	}
	d.config = config
	d.address = config.GetPredefinedAddress()
	d.port = net.Port(config.Port)
	d.policyManager = pm
	d.sockopt = sockopt

	return nil
}

// Network implements proxy.Inbound.
```

这两函数的作用如下：

1. `func (d *DokodemoDoor) Network() []net.Network`：该函数接收一个 `DokodemoDoor` 类型的参数 `d`，并返回一个只包含当前 `DokodemoDoor` 实例配置中的网络的 `net.Network` 切片。

2. `func (d *DokodemoDoor) policy() policy.Session`：该函数接收一个 `DokodemoDoor` 类型的参数 `d`，并返回一个属于用户级别的策略实例。

函数 `Network`：

1. 如果 `DokodemoDoor` 实例配置中的网络列表不为空，则直接返回。

2. 如果 `DokodemoDoor` 实例配置中的网络列表为空，并且 `DokodemoDoor` 实例配置中有一个名为 `NetworkList` 的配置项，则返回该配置项中的第一个网络。

函数 `policy`：

1. 首先，设置一个 `config` 变量为 `d` 实例的配置，然后设置 `policyManager` 实例的计数器为 `config.UserLevel`。

2. 如果 `DokodemoDoor` 实例配置中的 `timeout` 设置不等于0，则设置 `Timeouts.ConnectionIdle` 为 `time.Duration(timeout * time.Second)`。设置 `timeout` 为0的 `UserLevel` 将设置为0。

3. 返回策略管理器实例 `p`。


```go
func (d *DokodemoDoor) Network() []net.Network {
	if len(d.config.Networks) > 0 {
		return d.config.Networks
	}

	return d.config.NetworkList.Network
}

func (d *DokodemoDoor) policy() policy.Session {
	config := d.config
	p := d.policyManager.ForLevel(config.UserLevel)
	if config.Timeout > 0 && config.UserLevel == 0 {
		p.Timeouts.ConnectionIdle = time.Duration(config.Timeout) * time.Second
	}
	return p
}

```

This is a Go program that implements the TPROXY protocol for remote procedure calls (RPCs). The program includes a TCP proxy for receiving and sending requests to a remote system using the Internet-enabled side of the TPROXY protocol.

The program has the following components:

1. A TCP connection to the remote system, established using the internet-enabled side of the TPROXY protocol.
2. A buffer for storing incoming requests from the remote system.
3. A buffer for storing the response from the remote system.
4. A function for receiving and sending requests to the remote system using the buffer stored in step 2.
5. A function for receiving and sending responses from the remote system using the buffer stored in step 3.
6. A function for running the TPROXY protocol.
7. A function for closing the TCP connection to the remote system.


```go
type hasHandshakeAddress interface {
	HandshakeAddress() net.Address
}

// Process implements proxy.Inbound.
func (d *DokodemoDoor) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error {
	newError("processing connection from: ", conn.RemoteAddr()).AtDebug().WriteToLog(session.ExportIDToError(ctx))
	dest := net.Destination{
		Network: network,
		Address: d.address,
		Port:    d.port,
	}

	destinationOverridden := false
	if d.config.FollowRedirect {
		if outbound := session.OutboundFromContext(ctx); outbound != nil && outbound.Target.IsValid() {
			dest = outbound.Target
			destinationOverridden = true
		} else if handshake, ok := conn.(hasHandshakeAddress); ok {
			addr := handshake.HandshakeAddress()
			if addr != nil {
				dest.Address = addr
				destinationOverridden = true
			}
		}
	}
	if !dest.IsValid() || dest.Address == nil {
		return newError("unable to get destination")
	}

	if inbound := session.InboundFromContext(ctx); inbound != nil {
		inbound.User = &protocol.MemoryUser{
			Level: d.config.UserLevel,
		}
	}

	ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
		From:   conn.RemoteAddr(),
		To:     dest,
		Status: log.AccessAccepted,
		Reason: "",
	})
	newError("received request for ", conn.RemoteAddr()).WriteToLog(session.ExportIDToError(ctx))

	plcy := d.policy()
	ctx, cancel := context.WithCancel(ctx)
	timer := signal.CancelAfterInactivity(ctx, cancel, plcy.Timeouts.ConnectionIdle)

	ctx = policy.ContextWithBufferPolicy(ctx, plcy.Buffer)
	link, err := dispatcher.Dispatch(ctx, dest)
	if err != nil {
		return newError("failed to dispatch request").Base(err)
	}

	requestCount := int32(1)
	requestDone := func() error {
		defer func() {
			if atomic.AddInt32(&requestCount, -1) == 0 {
				timer.SetTimeout(plcy.Timeouts.DownlinkOnly)
			}
		}()

		var reader buf.Reader
		if dest.Network == net.Network_UDP {
			reader = buf.NewPacketReader(conn)
		} else {
			reader = buf.NewReader(conn)
		}
		if err := buf.Copy(reader, link.Writer, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to transport request").Base(err)
		}

		return nil
	}

	tproxyRequest := func() error {
		return nil
	}

	var writer buf.Writer
	if network == net.Network_TCP {
		writer = buf.NewWriter(conn)
	} else {
		//if we are in TPROXY mode, use linux's udp forging functionality
		if !destinationOverridden {
			writer = &buf.SequentialWriter{Writer: conn}
		} else {
			sockopt := &internet.SocketConfig{
				Tproxy: internet.SocketConfig_TProxy,
			}
			if dest.Address.Family().IsIP() {
				sockopt.BindAddress = dest.Address.IP()
				sockopt.BindPort = uint32(dest.Port)
			}
			if d.sockopt != nil {
				sockopt.Mark = d.sockopt.Mark
			}
			tConn, err := internet.DialSystem(ctx, net.DestinationFromAddr(conn.RemoteAddr()), sockopt)
			if err != nil {
				return err
			}
			defer tConn.Close()

			writer = &buf.SequentialWriter{Writer: tConn}
			tReader := buf.NewPacketReader(tConn)
			requestCount++
			tproxyRequest = func() error {
				defer func() {
					if atomic.AddInt32(&requestCount, -1) == 0 {
						timer.SetTimeout(plcy.Timeouts.DownlinkOnly)
					}
				}()
				if err := buf.Copy(tReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
					return newError("failed to transport request (TPROXY conn)").Base(err)
				}
				return nil
			}
		}
	}

	responseDone := func() error {
		defer timer.SetTimeout(plcy.Timeouts.UplinkOnly)

		if err := buf.Copy(link.Reader, writer, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to transport response").Base(err)
		}
		return nil
	}

	if err := task.Run(ctx, task.OnSuccess(func() error {
		return task.Run(ctx, requestDone, tproxyRequest)
	}, task.Close(link.Writer)), responseDone); err != nil {
		common.Interrupt(link.Reader)
		common.Interrupt(link.Writer)
		return newError("connection ends").Base(err)
	}

	return nil
}

```

# `proxy/dokodemo/errors.generated.go`

这段代码定义了一个名为 `errPathObjHolder` 的结构体，它包含一个空白的对象 `{}`，代表了 `errPathObjHolder` 的类型。

该结构体有一个名为 `newError` 的函数，该函数接收多个参数，这些参数可以是 `interface{}` 类型的任意数量。该函数返回一个 errors.Error 类型的对象，它包含一个 `WithPathObj` 方法，该方法接收一个 errPathObjHolder 类型的对象和一个错误消息，用于将错误消息与错误路径关联起来。

函数的实现原理是，首先创建一个空的 errPathObjHolder 对象，然后使用 `newError` 函数创建一个带有多個接口类型的参数的错误消息，最后将错误消息与 errPathObjHolder 对象关联起来，并返回该错误消息。

通过调用 `errPathObjHolder.WithPathObj` 方法，可以方便地将错误消息与错误路径关联起来，使得错误消息更容易地被调试和追踪。


```go
package dokodemo

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `proxy/freedom/config.go`

这段代码定义了一个名为 `useIP` 的函数，接收一个指向 `Config` 类型对象的引用 `c`，并返回一个布尔值。函数的作用是判断 `c` 是否使用了 IP 地址。

函数的实现基于 `c` 的 `DomainStrategy` 字段。如果 `c` 的 `DomainStrategy` 是 `Config_USE_IP`，那么函数返回 `true`，否则返回 `false`。如果 `c` 的 `DomainStrategy` 是 `Config_USE_IP4`，那么函数也返回 `true`，否则返回 `false`。如果 `c` 的 `DomainStrategy` 是 `Config_USE_IP6`，那么函数同样返回 `true`，否则返回 `false`。

总结一下，这段代码判断 `c` 是否使用了 IP 地址，并返回相应的结果。


```go
package freedom

func (c *Config) useIP() bool {
	return c.DomainStrategy == Config_USE_IP || c.DomainStrategy == Config_USE_IP4 || c.DomainStrategy == Config_USE_IP6
}

```

# `proxy/freedom/config.pb.go`

这段代码定义了一个名为`freedom`的包，其中包含一些定义了`config.proto`接口的类型和函数。

具体来说，这个包定义了一个名为`Config`的接口，它包含一个`IDatabase`字段，该字段指定了用于存储配置的数据库。然后，这个包定义了一个名为`RootConfig`的类型，该类型实现了`Config`接口，包含了`IDatabase`字段，并提供了初始化数据库的默认选项。最后，这个包定义了一些函数，用于将`IDatabase`字段中存储的配置信息转换为`v2ray.com/core/common/protocol`协议的请求和响应消息。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: proxy/freedom/config.proto

package freedom

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	protocol "v2ray.com/core/common/protocol"
)

```

这段代码是一个使用 Rust 的 const 注解，定义了一个名为 Config 的 struct 类型，以及一个名为 DomainStrategy 的常量类型。

具体来说，这段代码验证了两个依赖项是否足够新。第一个依赖项是一个名为 _ 的常量，用于验证生成的代码是否与预期的保护模式兼容。第二个依赖项是一个名为 _ 的常量，用于验证是否使用了兼容的运行时和 proto 库版本。

对于 Config 类型，该结构类型定义了一个域策略（即 0、1、2 或 3）以及一个名为 is_ip4 和 is_ip6 的成员变量。域策略表示如何使用特定的域名，IS_IP4 和 IS_IP6 分别表示是否使用 IP4 和 IP6 协议。

最后，该代码块使用 protoimpl.EnforceVersion 函数来验证生成的代码是否足够新。如果生成的代码过时，将会抛出一个错误。


```go
const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

// This is a compile-time assertion that a sufficiently up-to-date version
// of the legacy proto package is being used.
const _ = proto.ProtoPackageIsVersion4

type Config_DomainStrategy int32

const (
	Config_AS_IS   Config_DomainStrategy = 0
	Config_USE_IP  Config_DomainStrategy = 1
	Config_USE_IP4 Config_DomainStrategy = 2
	Config_USE_IP6 Config_DomainStrategy = 3
)

```

这段代码定义了一个名为 Config_DomainStrategy 的枚举类型，该类型定义了三个枚举值映射，分别对应 AS_IS、USE_IP 和 USE_IP4/USE_IP6 三种不同的域名解析策略。每个映射都有一个对应的整数值，用于在代码中使用哈希表进行查找。

具体来说，这段代码对于每个 Config_DomainStrategy 枚举类型，创建了一个名为 Config_DomainStrategy_name 的哈希表，该哈希表包含一个包含四个键值对的键：integer32 类型和四个整数值的值。这四个键分别对应 AS_IS、USE_IP 和 USE_IP4/USE_IP6 三种不同的域名解析策略，它们之间的键值对如下：

- AS_IS：0
- USE_IP：1
- USE_IP4：2
- USE_IP6：3

然后，代码又定义了一个名为 Config_DomainStrategy_value 的哈希表，该哈希表包含一个包含六个键值对的键：string 和六个整数值的值。这六个键分别对应上面四个键值对中对应的 AS_IS、USE_IP 和 USE_IP4/USE_IP6 三种不同的域名解析策略，它们之间的键值对如下：

- AS_IS：<AS_IS 的整数值>
- USE_IP：<USE_IP 的整数值>
- USE_IP4：<USE_IP4 的整数值>
- USE_IP6：<USE_IP6 的整数值>

这样，通过哈希表的键值对，可以实现了 Config_DomainStrategy 枚举类型的值映射。


```go
// Enum value maps for Config_DomainStrategy.
var (
	Config_DomainStrategy_name = map[int32]string{
		0: "AS_IS",
		1: "USE_IP",
		2: "USE_IP4",
		3: "USE_IP6",
	}
	Config_DomainStrategy_value = map[string]int32{
		"AS_IS":   0,
		"USE_IP":  1,
		"USE_IP4": 2,
		"USE_IP6": 3,
	}
)

```

这是一段使用Go编程语言编写的函数类型定义（函数指针）代码。它定义了一个名为`Config_DomainStrategy`的函数类型，该类型有一个`*Config_DomainStrategy`类型的参数，以及一个返回类型为`*Config_DomainStrategy`的函数指针。

具体来说，这段代码定义了一个名为`func`的函数，它接收一个名为`x`的参数，并返回一个名为`p`的函数指针。函数指针`p`被赋值为`x`，这意味着`x`的值将被复制并赋给`p`，因此`p`引用了`x`的值。

此外，这段代码定义了两个函数：`String`和`Descriptor`。`String`函数接收一个名为`x`的参数，并返回一个字符串类型的`*Config_DomainStrategy`类型的值，该值表示`x`的`*Config_DomainStrategy`类型的实例。`Descriptor`函数返回一个指向`*Config_DomainStrategy`类型实例的`Descriptor`字段，其中包含了该函数定义时指定的`File_Proxy_Freedom_Config_p`结构体类型的字段。

最后，这段代码还定义了一个名为`*Config_DomainStrategy`的函数指针类型，该类型代表了一个`File_Proxy_Freedom_Config_p`结构体类型的实例。


```go
func (x Config_DomainStrategy) Enum() *Config_DomainStrategy {
	p := new(Config_DomainStrategy)
	*p = x
	return p
}

func (x Config_DomainStrategy) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (Config_DomainStrategy) Descriptor() protoreflect.EnumDescriptor {
	return file_proxy_freedom_config_proto_enumTypes[0].Descriptor()
}

func (Config_DomainStrategy) Type() protoreflect.EnumType {
	return &file_proxy_freedom_config_proto_enumTypes[0]
}

```

这段代码定义了两个函数，以及一个结构体类型 DestinationOverride。

第一个函数 func(x Config_DomainStrategy) Number() protoreflect.EnumNumber {
 return protoreflect.EnumNumber(x)
} 接收一个名为 Config_DomainStrategy 的接口类型的参数 x，并返回一个名为 Number 的 enum 类型的值，该 enum 返回一个 int 类型的值。

第二个函数 func(Config_DomainStrategy) protoreflect.EnumDescriptor() ([]byte, []int) {
 return file_proxy_freedom_config_proto_rawDescGZIP(), []int{1, 0}
} 同样接收一个名为 Config_DomainStrategy 的接口类型的参数，并返回一个名为 EnumDescriptor 的 struct 类型的值，该 struct 包含一个字节数组和两个整数类型的字段，其中整数类型表示 Enum 中的值。函数的第二个参数是两个值，一个字节数组，包含 Enum 中所有元素的值，另一个整数类型，表示 Enum 中第一个元素的索引。

最后，定义了一个名为 DestinationOverride 的结构体类型，该结构体包含一个指向协议.ServerEndpoint类型的 field，该字段表示该目的地是否要缓存数据。


```go
func (x Config_DomainStrategy) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

// Deprecated: Use Config_DomainStrategy.Descriptor instead.
func (Config_DomainStrategy) EnumDescriptor() ([]byte, []int) {
	return file_proxy_freedom_config_proto_rawDescGZIP(), []int{1, 0}
}

type DestinationOverride struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Server *protocol.ServerEndpoint `protobuf:"bytes,1,opt,name=server,proto3" json:"server,omitempty"`
}

```

这段代码定义了两个函数，以及一个接口 DestinationOverride 的结构体类型。我会分别解释这两个函数的作用，然后解释一下 DestinationOverride 的接口。

1. func (x *DestinationOverride) Reset() {
这个函数的作用是重置 destination 对象 x，将 x 设置为空的 DestinationOverride 类型。这里涉及到了一个名为 x 的 pointer，通过这个 pointer 调用 Reset() 函数，将 x 的值设为 DestinationOverride{}，也就是一个空的 DestinationOverride 对象。

2. func (x *DestinationOverride) String() string {
这个函数的作用是将 DestinationOverride 类型中的 x 转换为字符串，并返回这个字符串。通过这个函数，可以方便地将 DestinationOverride 对象作为字符串返回。

3. func (x *DestinationOverride) ProtoMessage() {}
这个函数的作用是定义一个名为 x 的 DestinationOverride 类型接口，以便在与其他类型进行交互时使用。通过这个函数，可以定义一个接口，使得所有实现这个接口的类型具有相同的结构体。

现在，让我们通过以下代码使用这些函数：
protobuf
package example

import (
	"fmt"
)

type DestinationOverride struct {
	* DestinationOverride{}
}

type DestinationOverrideImpl struct {
	*DestinationOverride{}
}

func (x *DestinationOverrideImpl) Reset() {
	*x = DestinationOverride{}
}

func (x *DestinationOverrideImpl) String() string {
	return "An empty DestinationOverride object"
}

func (x *DestinationOverrideImpl) ProtoMessage() *DestinationOverride {
	return &DestinationOverride{}
}

在这段代码中，我们定义了一个名为 Example 的包，其中包含了一些函数和类型。我们定义了一个名为 DestinationOverride 的结构体类型，并在其中定义了两个函数：Reset() 和 String()。通过这些函数，我们可以实现与 DestinationOverride 类型相关的接口，从而在代码中更方便地使用它们。


```go
func (x *DestinationOverride) Reset() {
	*x = DestinationOverride{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_freedom_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *DestinationOverride) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*DestinationOverride) ProtoMessage() {}

```

这段代码定义了两个函数，分别是func和Descriptor。

func函数接收一个名为x的*DestinationOverride类型的参数，并返回一个指向protoreflect.Message类型对象的指针。函数的作用是在不违反安全性的情况下，根据传入的x对象的消息类型信息，返回相应的消息类型对象的指针。

Descriptor函数返回一个包含描述DestinationOverride类型对象的序列化字段名称和序列化字段类型的字节切片，用于指定将来的DeserializationContext。函数的作用是返回一个描述DestinationOverride类型对象的摘要信息，以便在将来的序列化和反序列化过程中使用。

这两函数一起描述了一个file_proxy_freedom_config_proto_文件的函数，用于在Go 2.0中对旧依赖的protoreflection库进行支持。在Go 3.0中，建议使用更安全的DestinationOverride类型，而不使用这段代码定义的函数。


```go
func (x *DestinationOverride) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_freedom_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use DestinationOverride.ProtoReflect.Descriptor instead.
func (*DestinationOverride) Descriptor() ([]byte, []int) {
	return file_proxy_freedom_config_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了一个名为 Config 的结构体类型，用于配置相关参数。

该结构体类型包含以下字段：

* state: 该字段是一个 protobuf 的类型，定义了配置文件的 state 字段类型，用于指示该配置文件是否已存在，目前还没有实现。
* sizeCache: 该字段是一个 protobuf 的类型，定义了配置文件的大小缓存策略，用于提高文件大小缓存的效率。
* unknownFields: 该字段是一个 protobuf 的类型，定义了未知字段，用于存储无法解析的配置文件字段。

该函数函数接收一个名为 x 的参数，该参数是一个指向 DestinationOverride 类型对象的指针。该函数的作用是通过 x 访问并返回一个 DestinationOverride 类型对象的 GetServer() *protocol.ServerEndpoint {...}


```go
func (x *DestinationOverride) GetServer() *protocol.ServerEndpoint {
	if x != nil {
		return x.Server
	}
	return nil
}

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	DomainStrategy Config_DomainStrategy `protobuf:"varint,1,opt,name=domain_strategy,json=domainStrategy,proto3,enum=v2ray.core.proxy.freedom.Config_DomainStrategy" json:"domain_strategy,omitempty"`
	// Deprecated: Do not use.
	Timeout             uint32               `protobuf:"varint,2,opt,name=timeout,proto3" json:"timeout,omitempty"`
	DestinationOverride *DestinationOverride `protobuf:"bytes,3,opt,name=destination_override,json=destinationOverride,proto3" json:"destination_override,omitempty"`
	UserLevel           uint32               `protobuf:"varint,4,opt,name=user_level,json=userLevel,proto3" json:"user_level,omitempty"`
}

```

这段代码定义了两个函数：

1. `Reset()`函数接收一个指向`Config`类型的`x`参数，并将其赋值为`Config{}`。然后，函数检查`protoimpl.UnsafeEnabled`是否为真，如果是，则执行以下操作：

  - 创建一个指向`file_proxy_freedom_config_proto_msgTypes`类型对象的`mi`变量。
  - 创建一个指向`x`类型对象的`ms`变量。
  - 将`mi`和`ms`指向的变量存储为`x`类型对象的`MessageInfo`字段。

2. `String()`函数接收一个指向`Config`类型的`x`参数，并返回`protoimpl.X.MessageStringOf(x)`的返回值。

3. `ProtoMessage()`函数用于将`Config`类型对象转换为字节序列，但没有实现任何具体的接口，因此它返回一个空括号`{}`。这个空括号表示该函数不会执行任何操作，因此返回的类型没有任何实际意义。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_freedom_config_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个`Config`类型的参数`x`，并返回`protoreflect.Message`类型的结果。

函数`ProtoReflect`接收`x`并返回一个`protoreflect.Message`类型的结果。在函数内部，首先检查`protoimpl.UnsafeEnabled`是否为`true`，如果是，则执行以下操作：

1. 从`file_proxy_freedom_config_proto_msgTypes`数组中获取第二个元素（因为 `*Config` 类型中只有一个元素，索引为1）。
2. 如果`x`不等于`nil`，则执行以下操作：
  1. 从`file_proxy_freedom_config_proto_msgTypes`数组中获取对应的`MessageType`类型。
  2. 如果`MessageStateOf`函数返回的结果为`nil`，则执行以下操作：
      1. 创建一个新的`MessageInfo`结构体，其中包含`MessageType`字段和`MessageId`字段（如果有）。
      2. 将新的`MessageInfo`结构体存储为新的`MessageStateOf`函数的输入参数。
      3. 返回新的`MessageStateOf`函数的输出结果。

函数`Descriptor`接收`Config`类型参数`x`并返回两个字节切片，分别是`file_proxy_freedom_config_proto_rawDescGZIP`和`descriptor`字段的值。函数内部直接使用`file_proxy_freedom_config_proto_rawDescGZIP`切片，然后创建一个新的字节切片并将`descriptor`字段的值存储到其中。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_freedom_config_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Config.ProtoReflect.Descriptor instead.
func (*Config) Descriptor() ([]byte, []int) {
	return file_proxy_freedom_config_proto_rawDescGZIP(), []int{1}
}

```

这两函数定义在了一个名为`func`的函数内部。它们的作用是获取`Config`类型的变量`x`的某些属性的值。

具体来说，这两函数分别返回以下两个函数的返回值：

1. `GetDomainStrategy()`函数返回一个名为`Config_DomainStrategy`的类型，它是由`x`指向的`Config`类型中的`DomainStrategy`字段的内容。

2. `GetTimeout()`函数返回一个名为`uint32`的类型，它是由`x`指向的`Config`类型中的`Timeout`字段的内容。

这两个函数在函数内部被调用了多次，因此在函数外部可能已经被废弃了。


```go
func (x *Config) GetDomainStrategy() Config_DomainStrategy {
	if x != nil {
		return x.DomainStrategy
	}
	return Config_AS_IS
}

// Deprecated: Do not use.
func (x *Config) GetTimeout() uint32 {
	if x != nil {
		return x.Timeout
	}
	return 0
}

```

此代码定义了两个函数，分别接收一个Config类型的参数x，并返回一个DestinationOverride类型的值。

第一个函数func (x *Config) GetDestinationOverride() *DestinationOverride {
   if x != nil {
       return x.DestinationOverride
   }
   return nil
} 作用于一个Config类型的参数x，如果x不为 nil，则返回x的DestinationOverride，否则返回 nil。

第二个函数func (x *Config) GetUserLevel() uint32 {
   if x != nil {
       return x.UserLevel
   }
   return 0
} 作用于一个Config类型的参数x，如果x不为 nil，则返回x的UserLevel，否则返回 0。

另外，定义了一个名为File_proxy_freedom_config_proto的protoreflect.FileDescriptor，但并没有使用它。


```go
func (x *Config) GetDestinationOverride() *DestinationOverride {
	if x != nil {
		return x.DestinationOverride
	}
	return nil
}

func (x *Config) GetUserLevel() uint32 {
	if x != nil {
		return x.UserLevel
	}
	return 0
}

var File_proxy_freedom_config_proto protoreflect.FileDescriptor

```

0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x6d, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x64, 0x6f, 0x6d, 0x50,
0x01, 0x5a, 0x1c, 0x76,
0x32, 0x72, 0x61, 0x79,
0x2e, 0x63, 0x6f, 0x6d,
0x2f, 0x63, 0x6f, 0x72,
0x65, 0x2f, 0x70, 0x72,
0x6f, 0x78, 0x79, 0x2e,
0x43, 0x6f, 0x72, 0x65,
0x65, 0x64, 0x6f, 0x6d,
0x62, 0x06, 0x70, 0x72,
0x6f, 0x74, 0x6f, 0x33,
0x50, 0x72, 0x6f, 0x78,
0x79, 0x2e, 0x46,
0x72, 0x65, 0x65,
0x64, 0x6f, 0x6d,
0x62, 0x06, 0x70,
0x72, 0x6f, 0x74,
0x6f, 0x6d, 0x50,
0x01, 0x5a, 0x1c,
0x76,


```go
var file_proxy_freedom_config_proto_rawDesc = []byte{
	0x0a, 0x1a, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x2f,
	0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x18, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66,
	0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x1a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70,
	0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x73,
	0x70, 0x65, 0x63, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x59, 0x0a, 0x13, 0x44, 0x65, 0x73,
	0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x4f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65,
	0x12, 0x42, 0x0a, 0x06, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b,
	0x32, 0x2a, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65,
	0x72, 0x76, 0x65, 0x72, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x06, 0x73, 0x65,
	0x72, 0x76, 0x65, 0x72, 0x22, 0xc4, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
	0x58, 0x0a, 0x0f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x5f, 0x73, 0x74, 0x72, 0x61, 0x74, 0x65,
	0x67, 0x79, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x2f, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79,
	0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65,
	0x64, 0x6f, 0x6d, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69,
	0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x52, 0x0e, 0x64, 0x6f, 0x6d, 0x61, 0x69,
	0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x1c, 0x0a, 0x07, 0x74, 0x69, 0x6d,
	0x65, 0x6f, 0x75, 0x74, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x42, 0x02, 0x18, 0x01, 0x52, 0x07,
	0x74, 0x69, 0x6d, 0x65, 0x6f, 0x75, 0x74, 0x12, 0x60, 0x0a, 0x14, 0x64, 0x65, 0x73, 0x74, 0x69,
	0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x6f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x18,
	0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
	0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d,
	0x2e, 0x44, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x4f, 0x76, 0x65, 0x72,
	0x72, 0x69, 0x64, 0x65, 0x52, 0x13, 0x64, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f,
	0x6e, 0x4f, 0x76, 0x65, 0x72, 0x72, 0x69, 0x64, 0x65, 0x12, 0x1d, 0x0a, 0x0a, 0x75, 0x73, 0x65,
	0x72, 0x5f, 0x6c, 0x65, 0x76, 0x65, 0x6c, 0x18, 0x04, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x09, 0x75,
	0x73, 0x65, 0x72, 0x4c, 0x65, 0x76, 0x65, 0x6c, 0x22, 0x41, 0x0a, 0x0e, 0x44, 0x6f, 0x6d, 0x61,
	0x69, 0x6e, 0x53, 0x74, 0x72, 0x61, 0x74, 0x65, 0x67, 0x79, 0x12, 0x09, 0x0a, 0x05, 0x41, 0x53,
	0x5f, 0x49, 0x53, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x10,
	0x01, 0x12, 0x0b, 0x0a, 0x07, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x34, 0x10, 0x02, 0x12, 0x0b,
	0x0a, 0x07, 0x55, 0x53, 0x45, 0x5f, 0x49, 0x50, 0x36, 0x10, 0x03, 0x42, 0x59, 0x0a, 0x1c, 0x63,
	0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72,
	0x6f, 0x78, 0x79, 0x2e, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x50, 0x01, 0x5a, 0x1c, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70, 0x72,
	0x6f, 0x78, 0x79, 0x2f, 0x66, 0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0xaa, 0x02, 0x18, 0x56, 0x32,
	0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x46,
	0x72, 0x65, 0x65, 0x64, 0x6f, 0x6d, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

这段代码定义了一个名为file_proxy_freedom_config_proto_rawDescOnce的变量，其作用是确保一个名为file_proxy_freedom_config_proto_rawDesc的静态类型的变量在每次调用file_proxy_freedom_config_proto_rawDescGZIP函数时都被创建一次。

file_proxy_freedom_config_proto_rawDescGZIP函数接收一个任意类型的数据，并将其压缩为字节切片并返回。其实现依赖于一个名为file_proxy_freedom_config_proto_rawDescOnce的变量，该变量确保在每次调用file_proxy_freedom_config_proto_rawDescGZIP函数时，该变量都会被正确地创建并初始化。

file_proxy_freedom_config_proto_enumTypes变量表示一个名为file_proxy_freedom_config_proto_enumTypes的静态类型变量，其中包含多个名为file_proxy_freedom_config_proto_rawDesc的枚举类型。

file_proxy_freedom_config_proto_msgTypes变量表示一个名为file_proxy_freedom_config_proto_msgTypes的静态类型变量，其中包含两个名为file_proxy_freedom_config_proto_msg类型的消息类型。

file_proxy_freedom_config_proto_goTypes变量表示一个名为file_proxy_freedom_config_proto_goTypes的静态类型变量，其中包含一个名为v2ray.core.proxy.freedom.Config的类型，该类型包含一个名为DomainStrategy的类型，该类型包含一个名为DestinationOverride的类型，该类型包含一个名为Config的类型，该类型包含一个名为protocol.ServerEndpoint的类型。


```go
var (
	file_proxy_freedom_config_proto_rawDescOnce sync.Once
	file_proxy_freedom_config_proto_rawDescData = file_proxy_freedom_config_proto_rawDesc
)

func file_proxy_freedom_config_proto_rawDescGZIP() []byte {
	file_proxy_freedom_config_proto_rawDescOnce.Do(func() {
		file_proxy_freedom_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_freedom_config_proto_rawDescData)
	})
	return file_proxy_freedom_config_proto_rawDescData
}

var file_proxy_freedom_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_proxy_freedom_config_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
var file_proxy_freedom_config_proto_goTypes = []interface{}{
	(Config_DomainStrategy)(0),      // 0: v2ray.core.proxy.freedom.Config.DomainStrategy
	(*DestinationOverride)(nil),     // 1: v2ray.core.proxy.freedom.DestinationOverride
	(*Config)(nil),                  // 2: v2ray.core.proxy.freedom.Config
	(*protocol.ServerEndpoint)(nil), // 3: v2ray.core.common.protocol.ServerEndpoint
}
```

This is a Go file that exports some protobuf messages that define the Config message struct. The Config message struct represents the state, size, and unknown fields of a Proxy. The file includes message imports from the same package as the message types, as well as defining the default export for the struct field.

The file also defines a struct field of type `x` for clarity and does not use it in the message definitions.

The `file_proxy_freedom_config_proto_msgTypes` and `file_proxy_freedom_config_proto_fieldDrop` macro constants are defined to customize the message exported field and the field fallback in case of zero-widthened braces.

The `file_proxy_freedom_config_proto_structFieldName` macro defines the name of the struct field for Config messages exported by the file.


```go
var file_proxy_freedom_config_proto_depIdxs = []int32{
	3, // 0: v2ray.core.proxy.freedom.DestinationOverride.server:type_name -> v2ray.core.common.protocol.ServerEndpoint
	0, // 1: v2ray.core.proxy.freedom.Config.domain_strategy:type_name -> v2ray.core.proxy.freedom.Config.DomainStrategy
	1, // 2: v2ray.core.proxy.freedom.Config.destination_override:type_name -> v2ray.core.proxy.freedom.DestinationOverride
	3, // [3:3] is the sub-list for method output_type
	3, // [3:3] is the sub-list for method input_type
	3, // [3:3] is the sub-list for extension type_name
	3, // [3:3] is the sub-list for extension extendee
	0, // [0:3] is the sub-list for field type_name
}

func init() { file_proxy_freedom_config_proto_init() }
func file_proxy_freedom_config_proto_init() {
	if File_proxy_freedom_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_proxy_freedom_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*DestinationOverride); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_proxy_freedom_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Config); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_proxy_freedom_config_proto_rawDesc,
			NumEnums:      1,
			NumMessages:   2,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_proxy_freedom_config_proto_goTypes,
		DependencyIndexes: file_proxy_freedom_config_proto_depIdxs,
		EnumInfos:         file_proxy_freedom_config_proto_enumTypes,
		MessageInfos:      file_proxy_freedom_config_proto_msgTypes,
	}.Build()
	File_proxy_freedom_config_proto = out.File
	file_proxy_freedom_config_proto_rawDesc = nil
	file_proxy_freedom_config_proto_goTypes = nil
	file_proxy_freedom_config_proto_depIdxs = nil
}

```

# `proxy/freedom/errors.generated.go`

这段代码定义了一个名为“errPathObjHolder”的结构体，它包含一个空白的 object。这个结构体的一个方法是名为“newError”的函数，这个函数接受多个参数，包括一个或多个对象（也被称为“values”）。函数返回一个名为“errors.Error”的类型的新错误对象，这个对象包含一个包含有关错误信息的“withPathObj”方法。

通过使用这个“errPathObjHolder”结构体，你可以创建一个包含错误信息的对象，通过调用“newError”函数，你可以为这个对象添加错误信息。这个函数使用了“v2ray.com/core/common/errors”包中的一个名为“errors”的函数，它用来创建一个错误对象。


```go
package freedom

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `proxy/freedom/freedom.go`

这段代码是一个 Go 语言编写的 build 包，其中包含了压缩源代码的功能。首先，它使用了一个名为 "!confonly" 的标志，这意味着这个 build 包仅在confonly模式下执行，这种模式下会清除任何生成的依赖关系，编译时会忽略任何外部依赖。接下来，它通过导入 " freedom" 包，并定义了一些常用函数和变量来自 "v2ray.com/core/common" 包。然后，它导入了 "v2ray.com/core/transport" 和 "v2ray.com/core/transport/internet" 包，这两个包是构成 V2Ray 核心网络和服务的核心部分。最后，通过调用 "v2ray.com/core/features/policy" 和 "v2ray.com/core/features/dns" 包中的函数，设置 V2Ray 的策略和 DNS 服务，实现网络访问的鉴权和安全。


```go
// +build !confonly

package freedom

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"
	"time"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/dice"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/retry"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal"
	"v2ray.com/core/common/task"
	"v2ray.com/core/features/dns"
	"v2ray.com/core/features/policy"
	"v2ray.com/core/transport"
	"v2ray.com/core/transport/internet"
)

```

该代码定义了一个名为"init"的函数，该函数接受一个名为"ctx"的上下文对象和一个名为"config"的接口类型参数。函数内部创建一个名为"h"的新的"Handler"实例，并尝试使用"core.RegisterConfig"函数将一个名为"nil"的Config对象注册到上下文里。如果注册失败，则函数返回一个非空错误。

接着，函数创建了一个名为"h"的"Handler"实例，并尝试使用"core.RequireFeatures"函数将一个名为"pm"的policy.Manager和一个名为"d"的dns.Client和一个名为"config"的接口类型参数注册到上下文中。如果注册失败，则函数返回一个非空错误。

最后，函数返回注册成功时创建的"h"实例，以及注册失败时返回的nil。


```go
func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		h := new(Handler)
		if err := core.RequireFeatures(ctx, func(pm policy.Manager, d dns.Client) error {
			return h.Init(config.(*Config), pm, d)
		}); err != nil {
			return nil, err
		}
		return h, nil
	}))
}

// Handler handles Freedom connections.
type Handler struct {
	policyManager policy.Manager
	dns           dns.Client
	config        *Config
}

```

这段代码定义了一个名为 `Handler` 的上下文处理程序，它可以将必要的参数初始化并返回一个指向 `policy.Session` 类型的指针。

具体来说，代码中定义了一个名为 `Init` 的函数，该函数接受三个参数：`config`、`pm` 和 `d`，分别表示上下文中的配置、策略管理器和 DNS 客户端。函数内部使用了这三个参数，创建了一个 `h` 变量，并将其初始化。

接着，定义了一个名为 `policy` 的函数，该函数使用了上面创建的 `h` 变量，以及从 `pm` 和 `dns.Client` 中获取的上下文信息和策略经理。函数内部根据 `h.config.UserLevel` 是否为 0 来设置不同时间的策略。

最后，通过 `policyManager.ForLevel` 方法获取了一个名为 `p` 的策略会话，并将其返回。


```go
// Init initializes the Handler with necessary parameters.
func (h *Handler) Init(config *Config, pm policy.Manager, d dns.Client) error {
	h.config = config
	h.policyManager = pm
	h.dns = d

	return nil
}

func (h *Handler) policy() policy.Session {
	p := h.policyManager.ForLevel(h.config.UserLevel)
	if h.config.Timeout > 0 && h.config.UserLevel == 0 {
		p.Timeouts.ConnectionIdle = time.Duration(h.config.Timeout) * time.Second
	}
	return p
}

```

该函数`resolveIP`的作用是解决将IPv4或IPv6地址解析为域名的问题。它首先检查`h.config.DomainStrategy`设置为`Config_USE_IP4`或`Config_USE_IP6`时，它是否正在使用IPv4或IPv6 DNS查找。如果是，函数内部将直接尝试使用`h.dns.(dns.IPv4Lookup)`或`h.dns.(dns.IPv6Lookup)`函数获取IPv4或IPv6地址。

如果`h.config.DomainStrategy`设置为`Config_USE_IP4`或`Config_USE_IP6`，并且`localAddr`参数不为空且`localAddr.Family().IsIPv4()`或`localAddr.Family().IsIPv6()`为真，那么函数将尝试使用`h.dns.(dns.IPv4Lookup)`或`h.dns.(dns.IPv6Lookup)`函数获取IPv4或IPv6地址。

如果解析IPv4或IPv6地址成功，函数将返回该地址。否则，函数将返回`nil`表示失败。


```go
func (h *Handler) resolveIP(ctx context.Context, domain string, localAddr net.Address) net.Address {
	var lookupFunc func(string) ([]net.IP, error) = h.dns.LookupIP

	if h.config.DomainStrategy == Config_USE_IP4 || (localAddr != nil && localAddr.Family().IsIPv4()) {
		if lookupIPv4, ok := h.dns.(dns.IPv4Lookup); ok {
			lookupFunc = lookupIPv4.LookupIPv4
		}
	} else if h.config.DomainStrategy == Config_USE_IP6 || (localAddr != nil && localAddr.Family().IsIPv6()) {
		if lookupIPv6, ok := h.dns.(dns.IPv6Lookup); ok {
			lookupFunc = lookupIPv6.LookupIPv6
		}
	}

	ips, err := lookupFunc(domain)
	if err != nil {
		newError("failed to get IP address for domain ", domain).Base(err).WriteToLog(session.ExportIDToError(ctx))
	}
	if len(ips) == 0 {
		return nil
	}
	return net.IPAddress(ips[dice.Roll(len(ips))])
}

```

This is a Go function that connects to a remote server using the GoDialer library. It is used to handle the connection lifecycle of a service that requires an active connection to the server.

The function takes a connection context (`ctx`) and a destination URL, which is used to establish the connection. The function returns a failure error if an error occurs during the connection process or a timeout occurs.

The connection is established using the `dialer.Dial` method, which raw waits for the connection to be established. If an error occurs during the dialing process, the connection is closed.

Once the connection is established, a `plcy` policy is created to handle the connection handling. This policy sets up a timeout for the upload and download of data, which is controlled by the `dialer.Timeouts.UplinkOnly` and `dialer.Timeouts.DownlinkOnly` parameters.

The connection is also followed by a timer that resets after an inactivity timeout, which is controlled by the `dialer.Timeouts.ConnectionIdle`.

Finally, the connection is closed by calling the `buf.Copy` method to copy the input data to the `buf.Writer` and writing it to the `buf.Reader`.

The function uses the `task.Run` method to run the `requestDone` and `responseDone` functions asynchronously. The `responseDone` function is called with the `output` parameter, which is written to in the event of a successful response.

The connection is established using the `dialer.Dial` method, which raw waits for the connection to be established. If an error occurs during the dialing process, the connection is closed. Once the connection is established, a `plcy` policy is created to handle the connection handling. This policy sets up a timeout for the upload and download of data, which is controlled by the `dialer.Timeouts.UplinkOnly` and `dialer.Timeouts.DownlinkOnly` parameters.


```go
func isValidAddress(addr *net.IPOrDomain) bool {
	if addr == nil {
		return false
	}

	a := addr.AsAddress()
	return a != net.AnyIP
}

// Process implements proxy.Outbound.
func (h *Handler) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
	outbound := session.OutboundFromContext(ctx)
	if outbound == nil || !outbound.Target.IsValid() {
		return newError("target not specified.")
	}
	destination := outbound.Target
	if h.config.DestinationOverride != nil {
		server := h.config.DestinationOverride.Server
		if isValidAddress(server.Address) {
			destination.Address = server.Address.AsAddress()
		}
		if server.Port != 0 {
			destination.Port = net.Port(server.Port)
		}
	}
	newError("opening connection to ", destination).WriteToLog(session.ExportIDToError(ctx))

	input := link.Reader
	output := link.Writer

	var conn internet.Connection
	err := retry.ExponentialBackoff(5, 100).On(func() error {
		dialDest := destination
		if h.config.useIP() && dialDest.Address.Family().IsDomain() {
			ip := h.resolveIP(ctx, dialDest.Address.Domain(), dialer.Address())
			if ip != nil {
				dialDest = net.Destination{
					Network: dialDest.Network,
					Address: ip,
					Port:    dialDest.Port,
				}
				newError("dialing to to ", dialDest).WriteToLog(session.ExportIDToError(ctx))
			}
		}

		rawConn, err := dialer.Dial(ctx, dialDest)
		if err != nil {
			return err
		}
		conn = rawConn
		return nil
	})
	if err != nil {
		return newError("failed to open connection to ", destination).Base(err)
	}
	defer conn.Close() // nolint: errcheck

	plcy := h.policy()
	ctx, cancel := context.WithCancel(ctx)
	timer := signal.CancelAfterInactivity(ctx, cancel, plcy.Timeouts.ConnectionIdle)

	requestDone := func() error {
		defer timer.SetTimeout(plcy.Timeouts.DownlinkOnly)

		var writer buf.Writer
		if destination.Network == net.Network_TCP {
			writer = buf.NewWriter(conn)
		} else {
			writer = &buf.SequentialWriter{Writer: conn}
		}

		if err := buf.Copy(input, writer, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to process request").Base(err)
		}

		return nil
	}

	responseDone := func() error {
		defer timer.SetTimeout(plcy.Timeouts.UplinkOnly)

		var reader buf.Reader
		if destination.Network == net.Network_TCP {
			reader = buf.NewReader(conn)
		} else {
			reader = buf.NewPacketReader(conn)
		}
		if err := buf.Copy(reader, output, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to process response").Base(err)
		}

		return nil
	}

	if err := task.Run(ctx, requestDone, task.OnSuccess(responseDone, task.Close(output))); err != nil {
		return newError("connection ends").Base(err)
	}

	return nil
}

```

# `proxy/http/client.go`

这段代码是一个 Go 语言编写的 HTTP 服务器，使用了 net/http 和 golang.org/x/net/http2 库，通过使用 !confonly 标志来创建一个只读的文件，防止修改代码。

具体来说，代码实现了一个 HTTP/1.1 服务器，可以处理 HTTP/1.1 和 HTTP/2 请求，允许客户端连接并发送请求。服务器包括以下组件：

1. HTTP 请求处理：通过使用 v2ray.com/core/common/buf 和 v2ray.com/core/common/bytespool 库来处理请求数据和内存缓冲池。

2. HTTP 连接管理：通过使用 v2ray.com/core/common/session 和 v2ray.com/core/common/signal 库来实现 HTTP 连接的创建、管理和关闭。

3. HTTP 错误处理：通过使用 v2ray.com/core/common/retry 和 v2ray.com/core/common/http2/transport/internet/tls 库来处理 HTTP 错误和重试。

4. HTTP 安全：通过使用 v2ray.com/core/common/policy 和 v2ray.com/core/common/net.ssl.TLS11 库来实现 HTTP 安全，包括 SSL/TLS11 加密和身份验证。

5. HTTP 请求拦截：通过使用 v2ray.com/core/common/net.http.Interceptor 库来实现对 HTTP 请求的拦截和修改。

6. HTTP 响应拦截：通过使用 v2ray.com/core/common/net.http.Interceptor 库来实现对 HTTP 响应的拦截和修改。

7. HTTP 流量控制：通过使用 v2ray.com/core/common/net.http.Interceptor 库来实现对 HTTP 流量进行控制和限制。

8. HTTP 静态文件服务器：通过使用 v2ray.com/core/features/http.StaticFileServer 库来实现 HTTP 静态文件服务器。

9. HTTP 自动压缩：通过使用 v2ray.com/core/features/http.Compression 库来实现 HTTP 压缩。

这段代码的作用是提供一个 HTTP 服务器，允许客户端连接并发送 HTTP 请求，实现了一个简单的 HTTP 服务功能。


```go
// +build !confonly

package http

import (
	"bufio"
	"context"
	"encoding/base64"
	"io"
	"net/http"
	"net/url"
	"sync"

	"golang.org/x/net/http2"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/bytespool"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/retry"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal"
	"v2ray.com/core/common/task"
	"v2ray.com/core/features/policy"
	"v2ray.com/core/transport"
	"v2ray.com/core/transport/internet"
	"v2ray.com/core/transport/internet/tls"
)

```

该代码定义了一个名为 Client 的结构体，其中包含一个名为 serverPicker 的协议.ServerPicker 和一个名为 policyManager 的 policy.Manager 类型的字段。

该代码定义了一个名为 h2Conn 的 struct，其中包含一个名为 rawConn 的 net.Conn 类型字段和一个名为 h2Conn 的 *http2.ClientConn类型字段。

该代码定义了一个名为 cacheH2Mutex 的 sync.Mutex，用于 cache 类型为 h2Conn 的连接。

该代码定义了一个名为 cacheH2Conns 的 map，用于存储类型为 net.Destination 的 h2Conn。


```go
type Client struct {
	serverPicker  protocol.ServerPicker
	policyManager policy.Manager
}

type h2Conn struct {
	rawConn net.Conn
	h2Conn  *http2.ClientConn
}

var (
	cachedH2Mutex sync.Mutex
	cachedH2Conns map[net.Destination]h2Conn
)

```

这段代码定义了一个名为NewClient的函数，该函数接受一个名为ctx的上下文和一个名为config的ClientConfig参数。

函数首先创建一个基于给定配置的新的HTTP客户端。然后，它遍历给定的服务器配置，并尝试从其中获取一个服务器。如果从服务器中获取任何服务器失败，那么函数将返回一个 nil客户端并抛出一个错误。如果从服务器中获取的所有服务器都是有效的，那么函数将返回一个包含Client实例和错误对象的客户端。

具体来说，这段代码实现了一个HTTP客户端，用于从给定的一组服务器中获取数据。它通过使用protocol.NewServerList()方法创建一个服务器列表，通过使用protocol.NewServerSpecFromPB()方法逐个获取服务器的信息，并将它们添加到服务器列表中。如果服务器列表为空，函数将抛出一个错误。如果服务器列表中包含服务器，函数将根据给定的客户端配置选择其中一个服务器，并创建一个Client实例，该实例使用选定的服务器作为其默认目标服务器。


```go
// NewClient create a new http client based on the given config.
func NewClient(ctx context.Context, config *ClientConfig) (*Client, error) {
	serverList := protocol.NewServerList()
	for _, rec := range config.Server {
		s, err := protocol.NewServerSpecFromPB(rec)
		if err != nil {
			return nil, newError("failed to get server spec").Base(err)
		}
		serverList.AddServer(s)
	}
	if serverList.Size() == 0 {
		return nil, newError("0 target server")
	}

	v := core.MustFromContext(ctx)
	return &Client{
		serverPicker:  protocol.NewRoundRobinServerPicker(serverList),
		policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),
	}, nil
}

```

This is a Go function that appears to handle the destruction of a TCP connection. It takes in an active connection, sets up a new HTTPS tunnel, and uses a retry strategy to establish a connection with the destination specified in the HTTPS tunnel. If the connection cannot be established within the specified timeout, the connection is closed, and the function returns an error.

The function has several fields:

* c: a context that is used to cancel the HTTPS tunnel connection.
* policyManager: a policy manager for setting connection limits.
* serverPicker: a function that is used to pick the server to use for the HTTPS tunnel.
* targetAddr: the IP address or hostname of the destination in the HTTPS tunnel.
* user: a connection user on the destination server.
* dialer: a connection planner that is used to establish the HTTPS tunnel connection.
* firstPayload: the first data payload to be sent over the HTTPS tunnel.
* retry: a retry strategy used to establish the HTTPS tunnel connection.
* exponentialBackoff: a backoff strategy used for the exponential backoff.

The function has the following methods:

* deffeerByT ByteSpool:
	This method is used to manage the TCP connection. It takes in a firstPayload data slice and calls the Free method of the ByteSpool to allow the connection to be closed.
	* deffeerByT ByteSpool:
	This method is used to manage the TCP connection. It takes in a firstPayload data slice and calls the Free method of the ByteSpool to allow the connection to be closed.
	* ifSuccess:
	This method is used to handle the success response of the HTTPS connection attempt. It takes in a function that is executed if the HTTPS connection is successful and returns an error if it fails.
	* ifError:
	This method is used to handle the error response of the HTTPS connection attempt. It takes in a function that is executed if the HTTPS connection fails and returns an error if it fails.
	* do:
	This method is used to run the function asynchronously.
	* retry:
	This method is used to establish the HTTPS tunnel connection using the exponential backoff strategy.
	* On:
	This method is used to register an event handler for the On response of the HTTPS connection attempt.
	* Off:
	This method is used to register an event handler for the Off response of the HTTPS connection attempt.
	* connect:
	This method is used to establish the HTTPS tunnel connection.
	* dest:
	This method is used to specify the destination of the HTTPS connection.
	* user:
	This method is used to specify the user that will be used for the HTTPS connection.
	* addr:
	This method is used to specify the IP address or hostname of the destination that will be used for the HTTPS connection.
	* session:
	This method is used to create a new session for the HTTPS connection.
	* destroy:
	This method is used to destroy the HTTPS connection.


```go
// Process implements proxy.Outbound.Process. We first create a socket tunnel via HTTP CONNECT method, then redirect all inbound traffic to that tunnel.
func (c *Client) Process(ctx context.Context, link *transport.Link, dialer internet.Dialer) error {
	outbound := session.OutboundFromContext(ctx)
	if outbound == nil || !outbound.Target.IsValid() {
		return newError("target not specified.")
	}
	target := outbound.Target
	targetAddr := target.NetAddr()

	if target.Network == net.Network_UDP {
		return newError("UDP is not supported by HTTP outbound")
	}

	var user *protocol.MemoryUser
	var conn internet.Connection

	mbuf, _ := link.Reader.ReadMultiBuffer()
	len := mbuf.Len()
	firstPayload := bytespool.Alloc(len)
	mbuf, _ = buf.SplitBytes(mbuf, firstPayload)
	firstPayload = firstPayload[:len]

	buf.ReleaseMulti(mbuf)
	defer bytespool.Free(firstPayload)

	if err := retry.ExponentialBackoff(5, 100).On(func() error {
		server := c.serverPicker.PickServer()
		dest := server.Destination()
		user = server.PickUser()

		netConn, err := setUpHTTPTunnel(ctx, dest, targetAddr, user, dialer, firstPayload)
		if netConn != nil {
			conn = internet.Connection(netConn)
		}
		return err
	}); err != nil {
		return newError("failed to find an available destination").Base(err)
	}

	defer func() {
		if err := conn.Close(); err != nil {
			newError("failed to closed connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
		}
	}()

	p := c.policyManager.ForLevel(0)
	if user != nil {
		p = c.policyManager.ForLevel(user.Level)
	}

	ctx, cancel := context.WithCancel(ctx)
	timer := signal.CancelAfterInactivity(ctx, cancel, p.Timeouts.ConnectionIdle)

	requestFunc := func() error {
		defer timer.SetTimeout(p.Timeouts.DownlinkOnly)
		return buf.Copy(link.Reader, buf.NewWriter(conn), buf.UpdateActivity(timer))
	}
	responseFunc := func() error {
		defer timer.SetTimeout(p.Timeouts.UplinkOnly)
		return buf.Copy(buf.NewReader(conn), link.Writer, buf.UpdateActivity(timer))
	}

	var responseDonePost = task.OnSuccess(responseFunc, task.Close(link.Writer))
	if err := task.Run(ctx, requestFunc, responseDonePost); err != nil {
		return newError("connection ends").Base(err)
	}

	return nil
}

```

This is a function that establishes an HTTPS connection to an HTTPS server by using the TLS protocol. It uses the `dialer` and `connectHTTP2` functions to handle the actual TLS handshake and connection建立， while also keeping track of the active HTTP/2 connections using a cache to avoid unnecessary DNS lookups.

The function takes a connection context (`ctx`) and the destination HTTPS host, port, and SSL/TLS certificate and optionally a proxy connection the SSL/TLS certificate and optionally a proxy connection.

It returns the active HTTP/2 connection, or `nil` if an error occurred.

The function first checks if the connection is to a valid HTTPS host, port, and if the SSL/TLS certificate is valid and not expired.

If the connection is valid, the function then performs the TLS handshake and establishes the active HTTP/2 connection.

If the connection is to a proxy server, the function first creates a new HTTP/2 connection and performs the TLS handshake. Then it creates a new HTTP/2 proxy connection and performs the TLS handshake with the proxy server.

Finally, the active HTTP/2 connection is either the newly established connection or the existing one, depending on whether a proxy connection was used or not.


```go
// setUpHTTPTunnel will create a socket tunnel via HTTP CONNECT method
func setUpHTTPTunnel(ctx context.Context, dest net.Destination, target string, user *protocol.MemoryUser, dialer internet.Dialer, firstPayload []byte) (net.Conn, error) {
	req := &http.Request{
		Method: http.MethodConnect,
		URL:    &url.URL{Host: target},
		Header: make(http.Header),
		Host:   target,
	}

	if user != nil && user.Account != nil {
		account := user.Account.(*Account)
		auth := account.GetUsername() + ":" + account.GetPassword()
		req.Header.Set("Proxy-Authorization", "Basic "+base64.StdEncoding.EncodeToString([]byte(auth)))
	}

	connectHTTP1 := func(rawConn net.Conn) (net.Conn, error) {
		req.Header.Set("Proxy-Connection", "Keep-Alive")

		err := req.Write(rawConn)
		if err != nil {
			rawConn.Close()
			return nil, err
		}

		if _, err := rawConn.Write(firstPayload); err != nil {
			rawConn.Close()
			return nil, err
		}

		resp, err := http.ReadResponse(bufio.NewReader(rawConn), req)
		if err != nil {
			rawConn.Close()
			return nil, err
		}

		if resp.StatusCode != http.StatusOK {
			rawConn.Close()
			return nil, newError("Proxy responded with non 200 code: " + resp.Status)
		}
		return rawConn, nil
	}

	connectHTTP2 := func(rawConn net.Conn, h2clientConn *http2.ClientConn) (net.Conn, error) {
		pr, pw := io.Pipe()
		req.Body = pr

		var pErr error
		var wg sync.WaitGroup
		wg.Add(1)

		go func() {
			_, pErr = pw.Write(firstPayload)
			wg.Done()
		}()

		resp, err := h2clientConn.RoundTrip(req)
		if err != nil {
			rawConn.Close()
			return nil, err
		}

		wg.Wait()
		if pErr != nil {
			rawConn.Close()
			return nil, pErr
		}

		if resp.StatusCode != http.StatusOK {
			rawConn.Close()
			return nil, newError("Proxy responded with non 200 code: " + resp.Status)
		}
		return newHTTP2Conn(rawConn, pw, resp.Body), nil
	}

	cachedH2Mutex.Lock()
	cachedConn, cachedConnFound := cachedH2Conns[dest]
	cachedH2Mutex.Unlock()

	if cachedConnFound {
		rc, cc := cachedConn.rawConn, cachedConn.h2Conn
		if cc.CanTakeNewRequest() {
			proxyConn, err := connectHTTP2(rc, cc)
			if err != nil {
				return nil, err
			}

			return proxyConn, nil
		}
	}

	rawConn, err := dialer.Dial(ctx, dest)
	if err != nil {
		return nil, err
	}

	iConn := rawConn
	if statConn, ok := iConn.(*internet.StatCouterConnection); ok {
		iConn = statConn.Connection
	}

	nextProto := ""
	if tlsConn, ok := iConn.(*tls.Conn); ok {
		if err := tlsConn.Handshake(); err != nil {
			rawConn.Close()
			return nil, err
		}
		nextProto = tlsConn.ConnectionState().NegotiatedProtocol
	}

	switch nextProto {
	case "", "http/1.1":
		return connectHTTP1(rawConn)
	case "h2":
		t := http2.Transport{}
		h2clientConn, err := t.NewClientConn(rawConn)
		if err != nil {
			rawConn.Close()
			return nil, err
		}

		proxyConn, err := connectHTTP2(rawConn, h2clientConn)
		if err != nil {
			rawConn.Close()
			return nil, err
		}

		cachedH2Mutex.Lock()
		if cachedH2Conns == nil {
			cachedH2Conns = make(map[net.Destination]h2Conn)
		}

		cachedH2Conns[dest] = h2Conn{
			rawConn: rawConn,
			h2Conn:  h2clientConn,
		}
		cachedH2Mutex.Unlock()

		return proxyConn, err
	default:
		return nil, newError("negotiated unsupported application layer protocol: " + nextProto)
	}
}

```

该代码定义了一个名为 newHTTP2Conn 的函数，它接受两个参数：一个 net.Conn 类型的客户端连接和两个 io.PipeWriter 类型的读/写流。第三个参数是一个 io.ReadCloser 类型的数据流，用于响应客户端发送的请求。

函数返回一个指向 http2Conn 类型的变量，它包含一个 net.Conn 类型的客户端连接、一个 io.PipeWriter 类型的数据流入连接和一个 io.ReadCloser 类型的数据流出连接。

newHTTP2Conn 函数的作用是创建一个新的 HTTP/2 连接，将客户端连接到服务器，并将请求和响应数据通过该连接传输。客户端连接的 `net.Conn` 类型表示网络连接，`io.PipeWriter` 类型表示数据流入通道的写入端口，`io.ReadCloser` 类型表示数据流出通道的读取端口。

函数中的 `Read` 和 `Write` 函数分别用于读取和写入数据。客户端发送的请求数据通过 `io.PipeWriter` 类型的数据流入连接发送，而服务器发送的响应数据则通过 `io.ReadCloser` 类型的数据流出连接接收。


```go
func newHTTP2Conn(c net.Conn, pipedReqBody *io.PipeWriter, respBody io.ReadCloser) net.Conn {
	return &http2Conn{Conn: c, in: pipedReqBody, out: respBody}
}

type http2Conn struct {
	net.Conn
	in  *io.PipeWriter
	out io.ReadCloser
}

func (h *http2Conn) Read(p []byte) (n int, err error) {
	return h.out.Read(p)
}

func (h *http2Conn) Write(p []byte) (n int, err error) {
	return h.in.Write(p)
}

```

这段代码定义了一个名为`func`的函数，其接收一个名为`h`的`http2Conn`类型的参数。

该函数的作用是关闭`h`所代表的`http2Conn`对象，并返回一个`error`类型的结果。

具体来说，函数首先关闭`h.in`所代表的`http2.Dial`连接，然后关闭`h.out`所代表的`http2.Dial`连接。这样做是为了确保所有连接都关闭，从而关闭整个`http2Conn`对象。


```go
func (h *http2Conn) Close() error {
	h.in.Close()
	return h.out.Close()
}

func init() {
	common.Must(common.RegisterConfig((*ClientConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
		return NewClient(ctx, config.(*ClientConfig))
	}))
}

```