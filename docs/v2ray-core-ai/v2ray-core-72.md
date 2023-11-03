# v2ray-core源码解析 72

# `transport/internet/tls/config_windows.go`

这段代码是一个 Go 语言编写的 TLS 库，用于生成 TLS 证书池。它的作用是加载一个自定义的证书颁发机构(CA)，以便在应用程序中使用。

具体来说，代码分为以下几个部分：

1. `// +build windows`：这是一个预处理指令，用于指定生成 Windows 版本的 TLS 证书池。
2. `// +build !confonly`：这也是一个预处理指令，用于指定生成只包含所需证书的 TLS 证书池，避免不必要的警告和错误。
3. `package tls`：声明了该 TLS 库的包名。
4. `import "crypto/x509"`：导入 `crypto/x509` 包，该包提供了用于生成和处理 X.509 证书的函数和类型。
5. `func (c *Config) getCertPool() (*x509.CertPool, error) {`：该函数的参数 `c` 是一个指向 `Config` 类型对象的指针，用于获取配置中指定的证书颁发机构。函数实现了获取证书池并返回错误信息的方法。
6. `if c.DisableSystemRoot {`：该代码块判断了 `c` 是否启用了系统根。如果启用了系统根，则执行以下操作：
  
  // loadSelfCertPool
  c.loadSelfCertPool()
  
  如果未启用系统根，则跳过该代码块，执行下面的操作：
  
  // loadSelfCertPool
  c.loadSelfCertPool()
  
  7. `return nil, nil`：该代码块返回一个空 `CertPool` 类型和一个空 `Error` 类型。
  
函数 `getCertPool` 接收一个指向 `Config` 类型对象的指针 `c`，并在不启用系统根的情况下从本地加载证书颁发机构。如果启用了系统根，则先加载系统根证书，然后从本地加载证书颁发机构。最后，函数返回一个指向 `CertPool` 类型对象的指针，如果没有指定具体的证书颁发机构，则返回 `nil`。


```go
// +build windows
// +build !confonly

package tls

import "crypto/x509"

func (c *Config) getCertPool() (*x509.CertPool, error) {
	if c.DisableSystemRoot {
		return c.loadSelfCertPool()
	}

	return nil, nil
}

```

# `transport/internet/tls/errors.generated.go`

这段代码定义了一个名为"tls"的包，它导入了自称为"v2ray.com/core/common/errors"的包，可能是用于在"tls"包中使用"errors"和"pathobj"函数的库。

该代码中定义了一个名为"errPathObjHolder"的结构体，它包含一个空的"pathobj"字段。

该代码中定义了一个名为"newError"的函数，该函数接收一个或多个参数，这些参数以"...interface{}"的形式传递。该函数使用传递的参数创建一个代表"errors.Error"类型的新错误对象，并使用"WithPathObj"方法将其路径设置为传递的路径对象。最后，该函数返回新错误对象。

该代码中没有明确说明"tls"包的任何其他函数或变量的作用，因此无法提供有关如何使用该包的更具体说明。


```go
package tls

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `transport/internet/tls/tls.go`

这段代码是一个Go语言编写的用于构建TLS证书片的命令行工具。它的作用是生成一个TLS证书片，然后将其保存到指定的文件路径。

具体来说，代码中使用了两个相关的库：`tls`和`v2ray.com/core/common/errors/errorgen`。

首先，它导入了`tls`库，这是用于实现TLS/SSL协议的Go库，它提供了创建和处理TLS连接的函数和类型。

其次，它导入了`v2ray.com/core/common/buf`库，这是一个用于传输数据的库，它提供了缓冲缓冲区的操作。

接着，它定义了一个名为`buildTlsCertificate`的函数，该函数接受一个`*buf.Writer`类型的参数，表示用于写入证书片的字节缓冲区。

然后，它创建了一个名为`confOnly`的选项，这个选项表示仅在创建证书片时输出配置信息，而不是在每次写入文件时都输出。

接下来，它导入了`io/ioutil`库，以用于处理文件字节切片。

最后，它使用`os`包的`Exit`函数来设置默认的退出码，以便在命令行中运行该工具时，如果出现错误，能够正确退出。

综上所述，这段代码的作用是生成一个TLS证书片，并将其保存到指定的文件路径。


```go
// +build !confonly

package tls

import (
	"crypto/tls"

	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
)

//go:generate go run v2ray.com/core/common/errors/errorgen

var (
	_ buf.Writer = (*Conn)(nil)
)

```

此代码定义了一个名为Conn的结构体，该结构体包含一个tls.Conn类型的指针变量。

该代码使用*tls.Conn指针变量c，通过c的WriteMultiBuffer函数，将一个MultiBuffer类型的缓冲区mb发送到tls.Conn类型的连接上。

该函数首先将mb缓冲区中的数据进行压缩，然后使用buf.WriteMultiBuffer函数将mb发送到tls.Conn类型的连接上。最后，使用buf.ReleaseMulti函数释放mb，并返回任何错误。

该代码还定义了一个名为HandshakeAddress的函数，该函数使用c的Handshake函数对连接进行初始化。如果Handshake函数执行成功，该函数返回连接的地址，否则返回 nil。

该代码还定义了一个名为ServerName的函数，该函数使用c的ConnectionState函数获取服务器名称。如果服务器名称没有设置，该函数返回 nil。


```go
type Conn struct {
	*tls.Conn
}

func (c *Conn) WriteMultiBuffer(mb buf.MultiBuffer) error {
	mb = buf.Compact(mb)
	mb, err := buf.WriteMultiBuffer(c, mb)
	buf.ReleaseMulti(mb)
	return err
}

func (c *Conn) HandshakeAddress() net.Address {
	if err := c.Handshake(); err != nil {
		return nil
	}
	state := c.ConnectionState()
	if state.ServerName == "" {
		return nil
	}
	return net.ParseAddress(state.ServerName)
}

```

这段代码定义了两个函数，第一个函数名为 Client，作用是发起一个 TLS 客户端握手，在给定的连接上建立一个 TLS 连接，然后返回一个含有该连接的net.Conn类型的变量。第二个函数名为 copyConfig，作用是复制一个 tls.Config 类型的参数，并返回一个 utls.Config 类型的变量，该变量包含了与给定 tls.Config 参数中包含的 Next 字段相同配置。


```go
// Client initiates a TLS client handshake on the given connection.
func Client(c net.Conn, config *tls.Config) net.Conn {
	tlsConn := tls.Client(c, config)
	return &Conn{Conn: tlsConn}
}

/*
func copyConfig(c *tls.Config) *utls.Config {
	return &utls.Config{
		NextProtos:         c.NextProtos,
		ServerName:         c.ServerName,
		InsecureSkipVerify: c.InsecureSkipVerify,
		MinVersion:         utls.VersionTLS12,
		MaxVersion:         utls.VersionTLS12,
	}
}

```

这两函数的作用如下：

1. `func UClient(c net.Conn, config *tls.Config) net.Conn` 是一个用户级服务器函数，它接收一个客户端连接和一个 TLS 配置对象。它首先将传入的 TLS 配置对象复制一份，然后使用 `utls.Client` 函数建立一个 TLS 客户端连接。最后，它返回一个指向客户端连接的客户端连接对象。

2. `func Server(c net.Conn, config *tls.Config) net.Conn` 是一个服务器级函数，它接收一个客户端连接和一个 TLS 配置对象。它首先创建一个 TLS 服务器连接对象，然后使用 `tls.Server` 函数将服务器端开始。最后，它返回一个指向服务器端连接的服务器连接对象。


```go
func UClient(c net.Conn, config *tls.Config) net.Conn {
	uConfig := copyConfig(config)
	return utls.Client(c, uConfig)
}
*/

// Server initiates a TLS server handshake on the given connection.
func Server(c net.Conn, config *tls.Config) net.Conn {
	tlsConn := tls.Server(c, config)
	return &Conn{Conn: tlsConn}
}

```

# `transport/internet/udp/config.go`

这段代码是一个 UDP 包的初始化函数，其主要作用是注册 UDP 协议的配置创建器。

具体来说，代码首先导入 "udp" 和 "v2ray.com/core/common" 两个包。然后，通过调用 "internet.RegisterProtocolConfigCreator" 函数，传递一个名为 "protocolName" 的参数，作为 UDP 协议的名称。

接着，在匿名函数中返回一个新创建的 "Config" 类型的变量，代表一个 UDP 协议的配置。最后，通过调用 "internet.RegisterProtocol" 函数，将刚刚创建的配置注册到 UDP 协议中，使得 "v2ray.com/core/transport/internet" 包可以使用该配置。


```go
package udp

import (
	"v2ray.com/core/common"
	"v2ray.com/core/transport/internet"
)

func init() {
	common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
		return new(Config)
	}))
}

```

# `transport/internet/udp/config.pb.go`

这段代码定义了一个名为 "udp" 的包，其作用是定义 UDP 协议的接口。具体来说，它实现了两个相关的接口：proto.UdpField和proto.UdpMessage。

首先，它导入了来自 "transport/internet/udp/config.proto" 的 proto.proto 文件，并定义了 versions 字段，以便在构建时根据需要选择不同的版本。

然后，它定义了 UdpField 和 UdpMessage 两个接口，分别用于在协议定义中声明字段和消息类型。这些接口都继承自 reflect.Struct和sync.Once，用于实现反射和保证线程安全。

最后，该代码将所有定义都保存到了一个名为 "udp" 的包中，并导入了 "github.com/golang/protobuf/proto" 和 "google.golang.org/protobuf/reflect/protoreflect" 两个依赖库。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: transport/internet/udp/config.proto

package udp

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码是一个Go语言中的const类型，它包含两个变量，分别用于确保两个Go语言版本的`proto`包的兼容性和使用较旧版本的`proto`包。

首先，它定义了一个名为`_`的变量，用于设置两个条件：

1. 确保生成的代码版本与`protoimpl.MinVersion`和`protoimpl.MaxVersion`的兼容性。

2. 确保`runtime/protoimpl`包的版本与`protoimpl.MinVersion`和`protoimpl.MaxVersion`的兼容性。

然后，它定义了一个名为`_`的变量，用于检查`proto.ProtoPackageIsVersion4`是否为true，如果为true，则表明运行时正在使用一个足够旧的`proto`包。

接下来，定义了一个名为`Config`的结构体类型，其中包含一个`state`字段，一个`sizeCache`字段和一个`unknownFields`字段。

这个结构体类型没有任何方法，但可以被用来创建一个`Config`类型的变量，比如：

go
config := Config{
	state:         protoimpl.MessageState(protoimpl.Empty),
	sizeCache:     protoimpl.SizeCache(0),
	unknownFields: protoimpl.UnknownFields(0),
}


注意，这个配置是编译时检查的，不是运行时检查的，所以你可能不需要在运行时创建一个这么具体的`Config`实例。


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

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields
}

```

这段代码定义了两个函数：

1. `func (x *Config) Reset()` 函数用于重置配置对象 `x`，即将其设置为 `Config{}`。

2. `func (x *Config) String()` 函数用于将配置对象 `x` 转换为字符串形式，根据 `protoimpl.X.MessageStringOf(x)` 实现的 Unsafe 风格，这个函数会忽略 `Config` 类型中的 `*` 前缀，所以它实际上会输出 `Config{}`。

3. `func (*Config) ProtoMessage()` 函数用于将配置对象 `x` 转换为 `protoimpl.X.Message` 类型，这个函数会生成一个 `Message` 接口的实现，其中包括了 `msgTypes` 和 `messageString` 字段。

根据 `func (x *Config) Reset()` 函数的实现，可以看出它是一个空函数，因为它只是将 `x` 对象中的值设置为 `Config{}`。

根据 `func (x *Config) String()` 函数的实现，可以看出它将 `x` 对象中的值转换为字符串形式，并返回这个字符串。

根据 `func (*Config) ProtoMessage()` 函数的实现，可以看出它返回的是一个 `Message` 接口的实现，这个接口定义了 `msgTypes` 和 `messageString` 字段。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_transport_internet_udp_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个`Config`类型的参数`x`，并返回其`Descriptor`类型。这两个函数分别实现了`file_transport_internet_udp_config_proto_descriptor`和`file_transport_internet_udp_config_proto_message_descriptor`接口。

具体来说，第一个函数`Descriptor`接收一个`Config`类型的`x`，返回其`Descriptor`类型的字节切片和对应的索引数组。其实现原理是：

1. 获取`file_transport_internet_udp_config_proto_rawDescGZIP`类型在`file_transport_internet_udp_config_proto_descriptor.go`文件中的定义。
2. 如果`x`不等于`nil`，则创建一个`file_transport_internet_udp_config_proto_message_descriptor`类型的`ms`对象，并将其设置为`x`的`MessageOf`方法的返回值。
3. 如果`x`等于`nil`，则直接返回`file_transport_internet_udp_config_proto_rawDescGZIP`类型的字节切片，其中包含`file_transport_internet_udp_config_proto_descriptor.go`中定义的`Descriptor`类型字节码。

第二个函数`ProtoReflect`则接收一个`Config`类型的`x`，返回其`Proteusereflect`方法的返回值，即`file_transport_internet_udp_config_proto_message_descriptor`类型。其实现原理与第一个函数类似，但使用了`message_descriptor.go`文件中定义的`Descriptor`类型，而不是`file_transport_internet_udp_config_proto_descriptor.go`文件中的`Descriptor`类型。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_transport_internet_udp_config_proto_msgTypes[0]
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
	return file_transport_internet_udp_config_proto_rawDescGZIP(), []int{0}
}

```

It appears that this is a list of integers. Could you provide more context or what you would like to know about this data?



```go
var File_transport_internet_udp_config_proto protoreflect.FileDescriptor

var file_transport_internet_udp_config_proto_rawDesc = []byte{
	0x0a, 0x23, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2f, 0x75, 0x64, 0x70, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e,
	0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x21, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
	0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2e, 0x75, 0x64, 0x70, 0x22, 0x08, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66,
	0x69, 0x67, 0x42, 0x74, 0x0a, 0x25, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
	0x63, 0x6f, 0x72, 0x65, 0x2e, 0x74, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x69,
	0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74, 0x2e, 0x75, 0x64, 0x70, 0x50, 0x01, 0x5a, 0x25, 0x76,
	0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x74, 0x72,
	0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2f, 0x69, 0x6e, 0x74, 0x65, 0x72, 0x6e, 0x65, 0x74,
	0x2f, 0x75, 0x64, 0x70, 0xaa, 0x02, 0x21, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72,
	0x65, 0x2e, 0x54, 0x72, 0x61, 0x6e, 0x73, 0x70, 0x6f, 0x72, 0x74, 0x2e, 0x49, 0x6e, 0x74, 0x65,
	0x72, 0x6e, 0x65, 0x74, 0x2e, 0x55, 0x64, 0x70, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

此代码定义了一个名为file_transport_internet_udp_config_proto的函数，其作用是压缩GZIP格式的file_transport_internet_udp_config_proto_rawDescData。

函数的实现分为两步：

1. 第一步，使用Once类型确保file_transport_internet_udp_config_proto_rawDescOnce只被调用一次，即在函数内部创建并返回file_transport_internet_udp_config_proto_rawDescOnce所代表的值，然后调用一个Do函数来执行实际的压缩操作。
2. 第二步，创建一个名为file_transport_internet_udp_config_proto_rawDescData的var变量，该变量存储了compressGZIP操作后压缩后的file_transport_internet_udp_config_proto_rawDescData。

函数的返回值是一个压缩后的byte数组，它包含了file_transport_internet_udp_config_proto_rawDescData的值。


```go
var (
	file_transport_internet_udp_config_proto_rawDescOnce sync.Once
	file_transport_internet_udp_config_proto_rawDescData = file_transport_internet_udp_config_proto_rawDesc
)

func file_transport_internet_udp_config_proto_rawDescGZIP() []byte {
	file_transport_internet_udp_config_proto_rawDescOnce.Do(func() {
		file_transport_internet_udp_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_transport_internet_udp_config_proto_rawDescData)
	})
	return file_transport_internet_udp_config_proto_rawDescData
}

var file_transport_internet_udp_config_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_transport_internet_udp_config_proto_goTypes = []interface{}{
	(*Config)(nil), // 0: v2ray.core.transport.internet.udp.Config
}
```

This is a function definition for the `file_transport_internet_udp_config_proto_init()` function. It is used to initialize the `file_transport_internet_udp_config_proto` message type, which is a part of the Google Protocol Buffers specification, and can be used to serialize and deserialize data in binary format.

The function uses the `File_transport_internet_udp_config_proto_msgTypes` and `file_transport_internet_udp_config_proto_goTypes` variables to define the message types and their prototypes, respectively. It also defines the `out` parameter of the `File_transport_internet_udp_config_proto_init()` function, which is the message type to be serialized or deserialized.

Finally, the function uses the `File_transport_internet_udp_config_proto_rawDesc` and `file_transport_internet_udp_config_proto_goTypes` variables to define the initialized message type and its generated code, respectively.


```go
var file_transport_internet_udp_config_proto_depIdxs = []int32{
	0, // [0:0] is the sub-list for method output_type
	0, // [0:0] is the sub-list for method input_type
	0, // [0:0] is the sub-list for extension type_name
	0, // [0:0] is the sub-list for extension extendee
	0, // [0:0] is the sub-list for field type_name
}

func init() { file_transport_internet_udp_config_proto_init() }
func file_transport_internet_udp_config_proto_init() {
	if File_transport_internet_udp_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_transport_internet_udp_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
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
			RawDescriptor: file_transport_internet_udp_config_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_transport_internet_udp_config_proto_goTypes,
		DependencyIndexes: file_transport_internet_udp_config_proto_depIdxs,
		MessageInfos:      file_transport_internet_udp_config_proto_msgTypes,
	}.Build()
	File_transport_internet_udp_config_proto = out.File
	file_transport_internet_udp_config_proto_rawDesc = nil
	file_transport_internet_udp_config_proto_goTypes = nil
	file_transport_internet_udp_config_proto_depIdxs = nil
}

```

# `transport/internet/udp/dialer.go`

这段代码是一个UDP协议的包，包含了一些通用的功能，以及注册了一个名为"udp"的传输协议。下面是具体的实现细节：

1. `init()`函数是一个初始化函数，它在创建通用网络上下文和Internet传输协议实例之前执行。

2. `internet.RegisterTransportDialer`函数是一个注册传输协议对函数，它接收一个协议名称、一个函数作为下一跳服务器、一个流设置选项作为参数。这个函数返回一个Internet传输协议实例和一个错误。

3. `internet.DialSystem`函数是一个用于建立UDP连接的函数，它接收一个上下文、一个目标IP地址和一个套接字设置选项作为参数。这个函数使用系统调用拨打目标服务器，并返回一个Internet连接实例和一个错误。

4. `internet.Connection`函数是一个返回一个Internet连接实例的函数，它接收一个已经建立好的Internet连接实例作为参数。

5. `internet.TransportDialer`是一个实现了TransportDialer接口的函数，它提供了一个下一跳服务器和套接字设置选项。

6. `tcp`包提供了对TCP套接字的功能，包括TCP连接的建立、数据传输和关闭等。


```go
package udp

import (
	"context"

	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

func init() {
	common.Must(internet.RegisterTransportDialer(protocolName,
		func(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
			var sockopt *internet.SocketConfig
			if streamSettings != nil {
				sockopt = streamSettings.SocketSettings
			}
			conn, err := internet.DialSystem(ctx, dest, sockopt)
			if err != nil {
				return nil, err
			}
			// TODO: handle dialer options
			return internet.Connection(conn), nil
		}))
}

```

# `transport/internet/udp/dispatcher.go`

这段代码定义了一个名为 "udp" 的 UDP 包，包含了用于实现 UDP 网络传输的一些基本功能。

首先，它导入了几个必要的库，包括 "udp"、"sync" 和 "time"，分别用于 UDP 包的传输、同步和时间相关的操作。

接下来，它定义了一个名为 "done" 的函数，它是 "v2ray.com/core/common/signal" 包中的一个名为 "done" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "DONE" 的变量。

然后，它定义了一个名为 "udpCtx" 的函数，它是 "v2ray.com/core/common/net" 包中的一个名为 "BeginUDPTransmission" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "BeginUDPTransmission" 的函数。

接下来，它定义了一个名为 "udpFinish" 的函数，它是 "v2ray.com/core/common/net" 包中的一个名为 "EndUDPTransmission" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "EndUDPTransmission" 的函数。

然后，它定义了一个名为 "udpSend" 的函数，它是 "udp" 包中的一个名为 "send" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "send" 的函数。

接下来，它定义了一个名为 "udpReceive" 的函数，它是 "udp" 包中的一个名为 "ReceiveUDPData" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "ReceiveUDPData" 的函数。

然后，它定义了一个名为 "udpRemoteIP" 的函数，它是 "v2ray.com/core/common/net" 包中的一个名为 "GetRemoteIP" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "GetRemoteIP" 的函数。

接下来，它定义了一个名为 "udpDstPort" 的函数，它是 "udp" 包中的一个名为 "GetDstPort" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "GetDstPort" 的函数。

然后，它定义了一个名为 "udpUDPEncoder" 的函数，它是 "v2ray.com/core/features/python3/transport" 包中的一个名为 "UdpEncoder" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "UdpEncoder" 的函数。

接下来，它定义了一个名为 "udpUDPDecoder" 的函数，它是 "v2ray.com/core/features/python3/transport" 包中的一个名为 "UdpDecoder" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "UdpDecoder" 的函数。

然后，它定义了一个名为 "udpUDPTransport" 的函数，它是 "v2ray.com/core/features/python3/transport" 包中的一个名为 "UdpTransport" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "UdpTransport" 的函数。

最后，它定义了一个名为 "udpDead" 的函数，它是 "v2ray.com/core/common/signal" 包中的一个名为 "done" 的函数的别名，这个别名在这里被用作 "udp" 包中的一个名为 "DONE" 的函数。


```go
package udp

import (
	"context"
	"io"
	"sync"
	"time"

	"v2ray.com/core/common/signal/done"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/udp"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport"
)

```

这段代码定义了一个名为`ResponseCallback`的函数类型，它接受一个名为`ctx`的上下文参考和一个名为`packet`的UDP数据包作为参数。

接下来，定义了一个名为`connEntry`的结构体，它包含一个名为`link`的指针，一个名为`timer`的信号变量和一个名为`cancel`的取消函数。

接着，定义了一个名为`Dispatcher`的抽象结构体，它包含一个名为`sync.RWMutex`的读写互斥锁，一个名为`conns`的 map，一个名为`dispatcher`的函数指针和一个名为`callback`的响应回调函数。`conns`的键是`net.Destination`类型的网络目的地，`dispatcher`是一个名为`routing.Dispatcher`的函数指针，`callback`是一个名为`ResponseCallback`的函数类型。


```go
type ResponseCallback func(ctx context.Context, packet *udp.Packet)

type connEntry struct {
	link   *transport.Link
	timer  signal.ActivityUpdater
	cancel context.CancelFunc
}

type Dispatcher struct {
	sync.RWMutex
	conns      map[net.Destination]*connEntry
	dispatcher routing.Dispatcher
	callback   ResponseCallback
}

```

这段代码定义了一个名为NewDispatcher的函数，它接受两个参数，一个是dispatcher，另一个是ResponseCallback，用于创建一个新的dispatcher实例。

函数返回一个指向Dispatcher类型的引用，该Dispatcher类型包含一个连接到Dispatcher的map，以及一个用于处理响应结果的回调函数。

NewDispatcher函数的实现包括以下步骤：

1. 创建一个名为conns的map，用于存储每个网络目标的连接信息。
2. 创建一个名为Dispatcher的dispatcher实例，并将其赋值给传入的dispatcher参数。
3. 创建一个名为callback的回调函数，该函数在连接到新的目标时被调用。
4. 返回一个新的Dispatcher实例，其中包含conns、dispatcher和callback参数。

函数的具体实现如下：

func NewDispatcher(dispatcher routed.Dispatcher, callback ResponseCallback) *Dispatcher {
	return &Dispatcher{
		conns:      make(map[net.Destination]*connEntry),
		dispatcher: dispatcher,
		callback:   callback,
	}
}

func (v *Dispatcher) RemoveRay(dest net.Destination) {
	v.Lock()
	defer v.Unlock()
	if conn, found := v.conns[dest]; found {
		common.Close(conn.link.Reader)
		common.Close(conn.link.Writer)
		delete(v.conns, dest)
	}
}

NewDispatcher函数创建了一个新的Dispatcher实例，并实现了一个RemoveRay方法，用于移除连接到指定dest目标的连接。

Dispatcher实例包含一个map，其中key是每个网络目标的名称，value是一个指向Connection信息类型的引用。当连接到一个新的目标时，函数调用RemoveRay方法来处理该目标。

RemoveRay方法的实现包括：

1. 获取当前Dispatcher实例的conns map，并获取dest目标的连接信息。
2. 关闭dest目标的读写流，并删除conns map中对应dest目标的键值对。
3. 返回dispatcher实例，表示已经对dest目标进行了处理。


```go
func NewDispatcher(dispatcher routing.Dispatcher, callback ResponseCallback) *Dispatcher {
	return &Dispatcher{
		conns:      make(map[net.Destination]*connEntry),
		dispatcher: dispatcher,
		callback:   callback,
	}
}

func (v *Dispatcher) RemoveRay(dest net.Destination) {
	v.Lock()
	defer v.Unlock()
	if conn, found := v.conns[dest]; found {
		common.Close(conn.link.Reader)
		common.Close(conn.link.Writer)
		delete(v.conns, dest)
	}
}

```

该函数的作用是获取入站 ray，并返回一个 *connEntry 类型的数据结构。

具体来说，该函数首先对传入的 `v` 参数进行锁操作，然后进行一个判断操作，如果 `v` 缓存中已经存在与目标 `dest` 对应的 `entry`，则返回该 `entry`，否则会创建一个新的错误并输出。

接着，该函数使用 `context.WithCancel` 取消当前操作的取消信号，并使用 `v.RemoveRay` 函数移除目标 `dest` 对应的 ray，然后设置一个定时器，在定时器超时时执行 `removeRay` 函数。

然后，该函数使用 `v.dispatcher.Dispatch` 函数将目标 `dest` 对应的 ray 发送出库，并返回一个 `entry` 数据结构。

最后，该函数使用 `go handleInput` 函数，将入站 ray 与传入的 `entry` 进行连接，并回调传递给 `handleInput` 的回调函数，即传入的 `entry` 的 callback 函数。


```go
func (v *Dispatcher) getInboundRay(ctx context.Context, dest net.Destination) *connEntry {
	v.Lock()
	defer v.Unlock()

	if entry, found := v.conns[dest]; found {
		return entry
	}

	newError("establishing new connection for ", dest).WriteToLog()

	ctx, cancel := context.WithCancel(ctx)
	removeRay := func() {
		cancel()
		v.RemoveRay(dest)
	}
	timer := signal.CancelAfterInactivity(ctx, removeRay, time.Second*4)
	link, _ := v.dispatcher.Dispatch(ctx, dest)
	entry := &connEntry{
		link:   link,
		timer:  timer,
		cancel: removeRay,
	}
	v.conns[dest] = entry
	go handleInput(ctx, entry, dest, v.callback)
	return entry
}

```

这是一段 Go 语言中的函数指针类型，表示了一个名为 `func` 的函数，它接收一个名为 `v` 的 UDP 数据报代理 `Dispatcher` 作为第一个参数，以及一个名为 `ctx` 的上下文对象、一个名为 `destination` 的网络目的地和一个名为 `payload` 的 UDP 数据报缓冲区作为第二个和第三个参数。

函数的作用是在 UDP 数据报发送到指定网络目的地时执行。首先，它将调用 `v.getInboundRay` 函数来获取进入该 UDP 数据报代理的连接，然后使用该连接的 `link.Writer` 字段写入数据到指定目的地。如果写入数据成功，则函数将返回。否则，函数将输出错误信息并尝试取消连接。


```go
func (v *Dispatcher) Dispatch(ctx context.Context, destination net.Destination, payload *buf.Buffer) {
	// TODO: Add user to destString
	newError("dispatch request to: ", destination).AtDebug().WriteToLog(session.ExportIDToError(ctx))

	conn := v.getInboundRay(ctx, destination)
	outputStream := conn.link.Writer
	if outputStream != nil {
		if err := outputStream.WriteMultiBuffer(buf.MultiBuffer{payload}); err != nil {
			newError("failed to write first UDP payload").Base(err).WriteToLog(session.ExportIDToError(ctx))
			conn.cancel()
			return
		}
	}
}

```

该函数`handleInput`用于处理用户数据报(UDP)输入。它接受一个`conn.Context`上下文、一个`conn.entry`连接entry、一个`net.Destination`目标IP地址和一个`ResponseCallback`响应回调函数作为参数。

首先，它使用一个`defer`关键字来延迟函数中的取消操作，即在函数返回前通知连接上下文已经取消。

然后，它读取并解析输入数据为一个多缓冲的`input.Reader`。如果遇到任何错误，它使用`newError`函数创建一个错误并将错误信息附加到`err`参数，然后将错误信息写入日志。

接下来，它循环等待输入数据的可用，使用`select`语句检查是否已准备好数据。如果已经准备好，它读取数据并更新计时器。然后，它循环遍历输入数据缓冲区中的所有数据，将数据传递给`ResponseCallback`函数作为请求的负载。

最后，如果输入数据准备好，它使用`conn.entry.cancel()`取消连接操作，以确保连接已经断开。


```go
func handleInput(ctx context.Context, conn *connEntry, dest net.Destination, callback ResponseCallback) {
	defer conn.cancel()

	input := conn.link.Reader
	timer := conn.timer

	for {
		select {
		case <-ctx.Done():
			return
		default:
		}

		mb, err := input.ReadMultiBuffer()
		if err != nil {
			newError("failed to handle UDP input").Base(err).WriteToLog(session.ExportIDToError(ctx))
			return
		}
		timer.Update()
		for _, b := range mb {
			callback(ctx, &udp.Packet{
				Payload: b,
				Source:  dest,
			})
		}
	}
}

```

该代码定义了一个名为 "dispatcherConn" 的结构体，该结构体包含一个名为 "dispatcher" 的成员变量，一个名为 "cache" 的成员变量和一个名为 "done" 的成员变量。

该代码还定义了一个名为 "DialDispatcher" 的函数，该函数接受一个名为 "ctx" 的上下文参数和一个名为 "dispatcher" 的路由器 "dispatcher" 参数。该函数将根据 "dispatcher" 创建一个名为 "c" 的 "dispatcherConn" 实例，并设置 "c.cache" 和 "c.done" 成员变量的值。然后，该函数将调用名为 "dispatcher" 的 "NewDispatcher" 函数和一个名为 "c" 的 "callback" 函数，并将 "dispatcher" 和 "c" 作为参数传递给 "DialDispatcher" 函数。最后，该函数返回一个名为 "c" 的 "dispatcherconn" 实例和一个名为 "nil" 的 "error" 类型的参数。

换句话说，该代码定义了一个名为 "dispatcherConn" 的结构体，用于管理 UDP 包的发送和接收。它通过创建一个 "done.Instance" 类型的 "done" 实例来跟踪是否已经完成数据包的发送或接收。同时，它还创建了一个 "cache" 类型的 "dispatcher" 实例，用于缓存已经接收到的数据包。通过调用 "DialDispatcher" 函数，可以设置 "dispatcher" 的路由器和 "callback" 函数，从而实现发送和接收 UDP 包的功能。


```go
type dispatcherConn struct {
	dispatcher *Dispatcher
	cache      chan *udp.Packet
	done       *done.Instance
}

func DialDispatcher(ctx context.Context, dispatcher routing.Dispatcher) (net.PacketConn, error) {
	c := &dispatcherConn{
		cache: make(chan *udp.Packet, 16),
		done:  done.New(),
	}

	d := NewDispatcher(dispatcher, c.callback)
	c.dispatcher = d
	return c, nil
}

```

这两函数是实现UDP数据报的传输和接收。

函数1：
kotlin
func (c *dispatcherConn) callback(ctx context.Context, packet *udp.Packet) {
	select {
	case <-c.done.Wait():
		packet.Payload.Release()
		return
	case c.cache <- packet:
			packet.Payload.Release()
			return
	}
}


这个函数接收一个UDP数据报，并根据给定的上下文、包装为<br>c.cache，然后按照给定方式处理接收到的数据报。

函数2：
kotlin
func (c *dispatcherConn) ReadFrom(p []byte) (int, net.Addr, error) {
	select {
	case <-c.done.Wait():
		return 0, nil, io.EOF
	case packet := <-c.cache:
		n := copy(p, packet.Payload.Bytes())
		return n, &net.UDPAddr{
			IP:   packet.Source.Address.IP(),
			Port: int(packet.Source.Port),
		}, nil
	}
}


函数2的作用是读取一个UDP数据报中的数据，并返回数据的长度、目的IP地址和错误。

这两个函数都是使用select语句来处理异步数据和保证通知，当有新数据到达时，它们将在适当的上下文中处理数据，并在完成后通知结果。


```go
func (c *dispatcherConn) callback(ctx context.Context, packet *udp.Packet) {
	select {
	case <-c.done.Wait():
		packet.Payload.Release()
		return
	case c.cache <- packet:
	default:
		packet.Payload.Release()
		return
	}
}

func (c *dispatcherConn) ReadFrom(p []byte) (int, net.Addr, error) {
	select {
	case <-c.done.Wait():
		return 0, nil, io.EOF
	case packet := <-c.cache:
		n := copy(p, packet.Payload.Bytes())
		return n, &net.UDPAddr{
			IP:   packet.Source.Address.IP(),
			Port: int(packet.Source.Port),
		}, nil
	}
}

```

这两段代码是 Go 中实现了 HTTP 库 `net/http` 和 `io/ioutil` 的函数。它们定义了一个名为 `func` 的函数，接收一个名为 `c` 的 pointer 类型参数，代表一个 HTTP 连接上下文。

这两段代码的主要作用是向 HTTP 服务器发送数据并获取响应。函数 `WriteTo` 接收一个字节数组 `p` 和一个目标客户端的 IP 地址 `addr`。它创建了一个名为 `buffer` 的缓冲区，将字节数组 `p` 中的所有元素复制到 `buffer` 中，并使用 `buffer.Size` 和 `copy` 函数将字节数组 `p` 中的元素扩展到 `buffer` 缓冲区的最大大小。然后，它创建一个名为 `ctx` 的上下文并使用 `c.dispatcher.Dispatch` 函数将数据发送到目标客户端 `addr`。最后，它返回数据发送的数量和错误。

函数 `Close` 是一个 HTTP 连接上下文的关闭操作，它返回一个错误。


```go
func (c *dispatcherConn) WriteTo(p []byte, addr net.Addr) (int, error) {
	buffer := buf.New()
	raw := buffer.Extend(buf.Size)
	n := copy(raw, p)
	buffer.Resize(0, int32(n))

	ctx := context.Background()
	c.dispatcher.Dispatch(ctx, net.DestinationFromAddr(addr), buffer)
	return n, nil
}

func (c *dispatcherConn) Close() error {
	return c.done.Close()
}

```

这是一组函数定义，作用于一个名为dispatcherConn的上下文对象上。

func localAddr() *net.UDPAddr {
	return &net.UDPAddr{
		IP:   []byte{0, 0, 0, 0},
		Port: 0,
	}
}

func setDeadline(t time.Time) error {
	return nil
}

func setReadDeadline(t time.Time) error {
	return nil
}

这些函数的主要作用是调节 dispatcherConn 对象的读写超时设置，包括设置 deadline 时间和设置允许的 IP 地址范围。

函数 `localAddr()` 返回一个网络套接字 UDP 套接字地址，其中 IP 地址设置为 `nil` 表示使用默认的 IP 地址(0.0.0.0)，端口设置为 0。

函数 `setDeadline(t,)` 接受一个 `time.Time` 类型的参数 `t`，设置一个 deadlines，用于关闭当前套接字。函数的返回值是错误的。

函数 `setReadDeadline(t,)` 接受一个 `time.Time` 类型的参数 `t`，设置一个读取 deadline，用于限制从客户端的请求时间。函数的返回值是错误的。


```go
func (c *dispatcherConn) LocalAddr() net.Addr {
	return &net.UDPAddr{
		IP:   []byte{0, 0, 0, 0},
		Port: 0,
	}
}

func (c *dispatcherConn) SetDeadline(t time.Time) error {
	return nil
}

func (c *dispatcherConn) SetReadDeadline(t time.Time) error {
	return nil
}

```

此函数名为`func`，定义了一个接收器`dispatcherConn`，以及一个名为`SetWriteDeadline`的函数，接受一个名为`t`的时间`time.Time`参数。

函数的作用是：接收一个`dispatcherConn`对象和一个`time.Time`参数，将其赋值给`dispatcherConn.WriteDeadline`，即设置一个写入超时时间，若写入超时，则会返回一个非空错误。


```go
func (c *dispatcherConn) SetWriteDeadline(t time.Time) error {
	return nil
}

```

# `transport/internet/udp/dispatcher_test.go`

这段代码是一个用于测试UDP协议的UDP_TEST包。它主要包含以下几部分：

1. 导入所需的包：
	* "context"（用于创建测试上下文）
	* "sync/atomic"（用于原子操作）
	* "testing"（用于用于测试）
	* "time"（用于获取当前时间）
	* "v2ray.com/core/common"（UDP协议的相关包）
	* "v2ray.com/core/common/buf"（用于创建缓冲区）
	* "v2ray.com/core/common/net"（用于网络操作）
	* "v2ray.com/core/common/protocol/udp"（UDP协议的相关包）
	* "v2ray.com/core/features/routing"（用于路由选择）
	* "v2ray.com/core/transport"（用于传输数据）
	* "v2ray.com/core/transport/internet/udp"（用于互联网传输）
	* "v2ray.com/core/transport/pipe"（用于管道传输）
2. 定义测试函数：
go
func TestMain(ctx context.Context) {
	// 创建一个测试路由器
	router, err := routing.NewRouter()
	if err != nil {
		panic(err)
	}

	// 创建一个测试路由器
	testRouter, err := router.CreateRouter(router, 0)
	if err != nil {
		panic(err)
	}

	// 创建一个测试套接字
	testSocket, err := net.ListenIP("udp", ":5000")
	if err != nil {
		panic(err)
	}

	// 创建一个定时器，定期向套接字发送数据
	ticker, err := time.NewTimer(2 * time.Second)
	if err != nil {
		panic(err)
	}

	// 设置定时器，每秒发送一次数据
	go func() {
		for {
			select {
			case <-ticker.C:
				// 从缓冲区获取数据
					buf, err := testRouter.GetRoutedFrame("test_frame")
					if err == nil {
							// 发送数据
							err = testSocket.WriteTo(buf)
							if err == nil {
								}
							}
						}
							buf = nil
							ticker.Unselect()
					}
					case <-buf.Done:
						ticker.Unselect()
						break
					}
				}
			}
		}
	}()

	// 设置超时时间
	t.Sig等待直到 <-testSocket.Done()
	t.SigSet()

	// 启动路由器
	r, err := router.Start(testSocket.String(), "udp")
	if err != nil {
		panic(err)
	}

	// 等待数据连接
	<-testSocket.Wait()

	// 发送数据
	err = testRouter.CreateRouter(r, 0).Send("test_frame")
	if err == nil {
		t.Log("数据发送成功")
	} else {
		t.Fatalf("数据发送失败： %v", err)
	}

	// 关闭套接字
	err = testSocket.Close()
	if err != nil {
		t.Fatalf("关闭套接字失败： %v", err)
	}

	// 关闭路由器
	err = r.Stop()
	if err == nil {
		t.Log("路由器停止成功")
	} else {
		t.Fatalf("路由器停止失败： %v", err)
	}
}

3. 设置超时时间以等待数据连接，并发送数据。


```go
package udp_test

import (
	"context"
	"sync/atomic"
	"testing"
	"time"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/udp"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport"
	. "v2ray.com/core/transport/internet/udp"
	"v2ray.com/core/transport/pipe"
)

```

该代码定义了一个名为 TestDispatcher 的 struct 类型，它包含一个名为 OnDispatch 的函数，用于处理传入的上下文 (ctx) 和目标 IP 地址 (dest)。

在 OnDispatch 函数中，传入的上下文 (ctx) 和目标 IP 地址 (dest) 会被传递给网络中的传输层，用于在目标设备上创建一个链接。如果创建链接成功，则返回一个指向传输层的链接，否则返回一个错误。

在 Dispatch 函数中，通过调用 OnDispatch 函数来获取目标设备上的链接，然后返回它。

在 Start 函数中，如果调用 OnDispatch 函数成功，则不会输出任何错误，否则输出一个错误并关闭套接字。

在 Close 函数中，如果调用 OnDispatch 函数成功，则不会输出任何错误，否则关闭套接字并输出一个错误。


```go
type TestDispatcher struct {
	OnDispatch func(ctx context.Context, dest net.Destination) (*transport.Link, error)
}

func (d *TestDispatcher) Dispatch(ctx context.Context, dest net.Destination) (*transport.Link, error) {
	return d.OnDispatch(ctx, dest)
}

func (d *TestDispatcher) Start() error {
	return nil
}

func (d *TestDispatcher) Close() error {
	return nil
}

```

This is a unit test for a hypothetical `pipe.Node` implementation that implements the same-destination dispatching mechanism.

The `pipe.Node` is designed to handle the transfer of data between two pipe servers.

The `TestDispatcher` struct has a `OnDispatch` method that defines the behavior of the dispatcher when it is called with a `net.Destination` and a `net.Packet` respectively.

The `TestSameDestinationDispatching` struct implements the `OnDispatch` method using a closure that increments a count and returns a new `transport.Link` and an error.

The `count` variable keeps track of the number of packets received from the `uplinkReader` and sent to the `downlinkWriter`.

The count is incremented by the `OnDispatch` method.

In the `TestSameDestinationDispatching` method, a `buf.New()` is created and filled with a string "abcd".

The function `dispatcher.Dispatch` is dispatched to the `dest` by `cancel`.

The function dispatches the packet received from the `uplinkReader` to the `downlinkWriter`.

The function dispatches the packet received from the `uplinkReader` to the `downlinkWriter`.

Finally, the function waits for 5 seconds and then increments the count and dispatches the packet received from the `uplinkReader` to the `downlinkWriter`.

It should be noted that the `TestSameDestinationDispatching` is not testing the actual behavior of the `pipe.Node` but rather the behavior of the `OnDispatch` method defined by the `TestDispatcher`.


```go
func (*TestDispatcher) Type() interface{} {
	return routing.DispatcherType()
}

func TestSameDestinationDispatching(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	uplinkReader, uplinkWriter := pipe.New(pipe.WithSizeLimit(1024))
	downlinkReader, downlinkWriter := pipe.New(pipe.WithSizeLimit(1024))

	go func() {
		for {
			data, err := uplinkReader.ReadMultiBuffer()
			if err != nil {
				break
			}
			err = downlinkWriter.WriteMultiBuffer(data)
			common.Must(err)
		}
	}()

	var count uint32
	td := &TestDispatcher{
		OnDispatch: func(ctx context.Context, dest net.Destination) (*transport.Link, error) {
			atomic.AddUint32(&count, 1)
			return &transport.Link{Reader: downlinkReader, Writer: uplinkWriter}, nil
		},
	}
	dest := net.UDPDestination(net.LocalHostIP, 53)

	b := buf.New()
	b.WriteString("abcd")

	var msgCount uint32
	dispatcher := NewDispatcher(td, func(ctx context.Context, packet *udp.Packet) {
		atomic.AddUint32(&msgCount, 1)
	})

	dispatcher.Dispatch(ctx, dest, b)
	for i := 0; i < 5; i++ {
		dispatcher.Dispatch(ctx, dest, b)
	}

	time.Sleep(time.Second)
	cancel()

	if count != 1 {
		t.Error("count: ", count)
	}
	if v := atomic.LoadUint32(&msgCount); v != 6 {
		t.Error("msgCount: ", v)
	}
}

```

# `transport/internet/udp/errors.generated.go`

这段代码定义了一个名为udp的包，其中包含一个名为errPathObjHolder的结构体，以及一个名为newError的函数。

udp包中包含的errPathObjHolder结构体是一个空的结构体类型，它没有任何字段或变量。

newError函数接收多个参数，这些参数可以是任意类型的对象。函数返回一个带有错误对象和错误路径对象的错误对象。

函数的实现比较简单，直接创建一个errPathObjHolder类型的空对象，并将它作为构造函数的参数传入，这样构造函数就可以将错误对象的错误路径与给定的错误对象关联起来。然后，在函数内部使用errors.New创建一个新的错误对象，并将values...(也就是任意多个参数)添加到构造函数的参数中，这样就可以将错误对象的错误信息与给定的错误对象关联起来。最后，使用WithPathObj函数将errPathObjHolder对象与错误对象的错误路径关联起来，这样就可以将错误对象的错误信息与给定的错误对象的错误路径关联起来。

这段代码定义了一个用于创建错误对象的函数，可以用来在程序中记录错误信息。


```go
package udp

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `transport/internet/udp/hub.go`

这段代码定义了一个名为 "udp" 的 package，其中包含了一些 UDP 相关的函数和类型。

首先，导入了 "v2ray.com/core/common/buf"、"v2ray.com/core/common/net" 和 "v2ray.com/core/common/protocol/udp" 包，分别用于缓冲区操作、网络通信和 UDP 协议。

接着，定义了一个名为 "HubOption" 的函数类型，该类型可以作为 "Hub" 类型的方法分录的参数，不过没有给出具体的实现。

然后，定义了一个名为 "HubCapacity" 的函数，该函数接收一个整数参数 "capacity"，返回一个 "HubOption" 类型的函数，该函数可以设置 Hub 的容量。实现上，该函数直接返回一个容量为 "capacity" 的匿名函数，调用该函数时需要保证传入的 Hub 对象已经初始化好了容量。

最后，通过 "import" 语句导入了 "context" 包，不过没有给出具体的实现。


```go
package udp

import (
	"context"

	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol/udp"
	"v2ray.com/core/transport/internet"
)

type HubOption func(h *Hub)

func HubCapacity(capacity int) HubOption {
	return func(h *Hub) {
		h.capacity = capacity
	}
}

```

该代码定义了一个名为HubReceiveOriginalDestination的函数类型变量hub，该函数接收一个bool类型的参数r，并返回一个 HubOption 类型的函数指针。函数指针函数执行以下操作：

1. 将hub的recvOrigDest字段设置为参数r的值，以便在函数调用时设置 Hub 的接收者 originalDestination。
2. 如果函数选项中包含一个名为HubOption的类型变量，将其设置为hub的指针。
3. 如果函数选项中包含一个名为ListenUDP的函数，将其设置为hub的capacity字段，以便设置 Hub 的接收者数量。
4. 如果函数选项中包含一个名为InternetStreamSettings的函数，将其设置为hub的options字段，以便设置 Hub 的接收者 UDP 网络套接字设置。
5. 调用 Hub 的start函数以开始监听。
6. 将 Hub 的接收者 originalDestination 设置为参数r的值，以便在接收到数据时设置为true，从而启用接收数据并将其保存到Hub的接收者缓存中。

该函数的作用是创建一个 Hub 实例并设置其参数，使其能够接收数据并将其保存到缓存中。通过将 Hub 的接收者 originalDestination 设置为参数 r 的值，可以设置为接收数据并将其保存到缓存中。该函数指针将 Hub 实例的接收者缓存容量设置为参数 Hub 的容量，以便接收数据并将其保存到缓存中。


```go
func HubReceiveOriginalDestination(r bool) HubOption {
	return func(h *Hub) {
		h.recvOrigDest = r
	}
}

type Hub struct {
	conn         *net.UDPConn
	cache        chan *udp.Packet
	capacity     int
	recvOrigDest bool
}

func ListenUDP(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, options ...HubOption) (*Hub, error) {
	hub := &Hub{
		capacity:     256,
		recvOrigDest: false,
	}
	for _, opt := range options {
		opt(hub)
	}

	var sockopt *internet.SocketConfig
	if streamSettings != nil {
		sockopt = streamSettings.SocketSettings
	}
	if sockopt != nil && sockopt.ReceiveOriginalDestAddress {
		hub.recvOrigDest = true
	}

	udpConn, err := internet.ListenSystemPacket(ctx, &net.UDPAddr{
		IP:   address.IP(),
		Port: int(port),
	}, sockopt)
	if err != nil {
		return nil, err
	}
	newError("listening UDP on ", address, ":", port).WriteToLog()
	hub.conn = udpConn.(*net.UDPConn)
	hub.cache = make(chan *udp.Packet, hub.capacity)

	go hub.start()
	return hub, nil
}

```

This function appears to be used to send a UDP packet to a specified destination using the `h` instance of a `Hub` struct.

The function takes two arguments:

* `payload`: a slice of bytes representing the data to be sent in the UDP packet.
* `dest`: a `net.Destination` object representing the destination address and port of the packet.

The function returns an error indicating whether the packet was successfully sent or an error occurred.

The function starts by initializing a local cache of UDPMessage objects using the `h.cache` instance of the `Hub` struct.

The function then enters a loop that reads incoming UDP messages from the `h.conn` instance and sends them to the specified destination.

For each incoming message, the function creates a new `udp.Packet` object with the given `payload` and the destination specified by the `dest` argument.

If the destination is an original destination, the function reads the original destination from the `h.recvOrigDest` field and sets the `Target` field of the `Packet` object to it.

If the destination is not an original destination, the function reads the UDP source address and port from the `Payload` field of the incoming message.

The function then sends the packet to the destination using the `h.conn.WriteToUDP` method, passing in the `payload` and the `Target` fields of the `Packet` object.

If the function completes successfully, it returns `nil`. If an error occurs, the function returns a custom error.


```go
// Close implements net.Listener.
func (h *Hub) Close() error {
	h.conn.Close()
	return nil
}

func (h *Hub) WriteTo(payload []byte, dest net.Destination) (int, error) {
	return h.conn.WriteToUDP(payload, &net.UDPAddr{
		IP:   dest.Address.IP(),
		Port: int(dest.Port),
	})
}

func (h *Hub) start() {
	c := h.cache
	defer close(c)

	oobBytes := make([]byte, 256)

	for {
		buffer := buf.New()
		var noob int
		var addr *net.UDPAddr
		rawBytes := buffer.Extend(buf.Size)

		n, noob, _, addr, err := ReadUDPMsg(h.conn, rawBytes, oobBytes)
		if err != nil {
			newError("failed to read UDP msg").Base(err).WriteToLog()
			buffer.Release()
			break
		}
		buffer.Resize(0, int32(n))

		if buffer.IsEmpty() {
			buffer.Release()
			continue
		}

		payload := &udp.Packet{
			Payload: buffer,
			Source:  net.UDPDestination(net.IPAddress(addr.IP), net.Port(addr.Port)),
		}
		if h.recvOrigDest && noob > 0 {
			payload.Target = RetrieveOriginalDest(oobBytes[:noob])
			if payload.Target.IsValid() {
				newError("UDP original destination: ", payload.Target).AtDebug().WriteToLog()
			} else {
				newError("failed to read UDP original destination").WriteToLog()
			}
		}

		select {
		case c <- payload:
		default:
			buffer.Release()
			payload.Payload = nil
		}

	}
}

```

这段代码定义了一个名为`Addr`的函数接口，该接口表示一个`net.Listener`类型的监听器。

具体来说，`h`是一个`Hub`类型的变量，它实现了`Addr`接口。

`h.conn`是一个`net.Conn`类型的变量，它用于与远程服务器建立连接。

`h.cache`是一个`map.Map`类型的变量，它存储了`h`连接到远程服务器时收到的数据缓存。

`h.Receive()`函数接收一个`<net.Packet>`类型的参数，它返回一个`<net.Packet>`类型的通道。这个通道类型表示从`h.cache`中读取的数据包。

由于`h.conn`用于与远程服务器建立连接，并且`h.cache`用于存储从远程服务器收到的数据，因此`h.Receive()`函数可以被用来监听从远程服务器接收数据包的情况。


```go
// Addr implements net.Listener.
func (h *Hub) Addr() net.Addr {
	return h.conn.LocalAddr()
}

func (h *Hub) Receive() <-chan *udp.Packet {
	return h.cache
}

```

# `transport/internet/udp/hub_freebsd.go`

这段代码定义了一个名为"udp"的包，其中包括以下函数：

1. RetrieveOriginalDest函数：该函数接收一个字节切片(oob)作为参数，然后将其解码为两个NetBIOS名字(即IP地址和端口号)，并使用v2ray.com/core/net库中的internet.OriginalDst函数来获取原始目的IP地址和端口号。如果解码或获取IP地址和端口号时出现错误，函数将返回一个Net.Destination对象，表示没有原始目的地。

2. main函数：该函数使用bytes.NewBuffer函数创建一个字节切片(oob)，然后将其传递给RetrieveOriginalDest函数。

3. byte切片中的数据被分成两段，一段用于接收IP地址和端口号，另一段用于返回结果。


```go
// +build freebsd

package udp

import (
	"bytes"
	"encoding/gob"
	"io"

	"v2ray.com/core/common/net"
	"v2ray.com/core/transport/internet"
)

// RetrieveOriginalDest from stored laddr, caddr
func RetrieveOriginalDest(oob []byte) net.Destination {
	dec := gob.NewDecoder(bytes.NewBuffer(oob))
	var la, ra net.UDPAddr
	dec.Decode(&la)
	dec.Decode(&ra)
	ip, port, err := internet.OriginalDst(&la, &ra)
	if err != nil {
		return net.Destination{}
	}
	return net.UDPDestination(net.IPAddress(ip), net.Port(port))
}

```

这段代码定义了一个名为 `ReadUDPMsg` 的函数，它接收一个 UDP 连接器（`conn`），一个字节数组（`payload`）和一个字节数组（`oob`），然后返回四个值：`nBytes`、`addr`、`err` 和一个指向 `net.UDPAddr` 类型的指针（`err`）。函数的作用是读取数据报中的目标地址（`laddr`）和源地址（`caddr`）。

函数首先通过 `conn.ReadFromUDP` 函数读取 `payload` 字节数组中的数据报数据。然后，创建一个名为 `buf` 的字节缓冲区，并将其编码为 `net.UDPAddr` 类型，用于存储目标地址。接着，使用 `reader` 作为指针，从 `oob` 字节数组中读取数据报，并将其解码为 `net.UDPMsg` 类型的数据结构。

最后，函数将返回值 `nBytes`、`noob` 和 `err` 分别表示数据报中的目标地址 `laddr`、源地址 `caddr` 和错误，而指针 `err` 指向任何错误。


```go
// ReadUDPMsg stores laddr, caddr for later use
func ReadUDPMsg(conn *net.UDPConn, payload []byte, oob []byte) (int, int, int, *net.UDPAddr, error) {
	nBytes, addr, err := conn.ReadFromUDP(payload)
	var buf bytes.Buffer
	enc := gob.NewEncoder(&buf)
	enc.Encode(conn.LocalAddr().(*net.UDPAddr))
	enc.Encode(addr)
	var reader io.Reader = &buf
	noob, _ := reader.Read(oob)
	return nBytes, noob, 0, addr, err
}

```

# `transport/internet/udp/hub_linux.go`

这段代码定义了一个名为"udp"的包，其中包含一个名为"RetrieveOriginalDest"的函数。

函数接收一个字节切片(oob)，包含一个IPv6或IPv4数据报的消息头。函数解析消息头并遍历其中的各个条目。如果消息头中包含一个IPv6或IPv4数据报的消息类型(如IP_RECEIVER_ORIGINAL_DSTADDR)，函数返回一个UDP数据报的本地发送者(即本地发送方IP和端口)。如果消息头中包含一个IPv6或IPv4数据报的消息类型，函数尝试解析该数据报的原始出度地址，并返回一个UDP数据报的本地发送者(即本地发送方IP和端口)。如果函数无法解析任何消息头，则返回一个UDP数据报的本地发送者(即本地发送方IP和端口)。

从代码中可以看出，该函数的主要作用是接收一个IPv6或IPv4数据报的消息头，并返回一个UDP数据报的本地发送者。它通过调用syscall.ParseSocketControlMessage和遍历消息头中的各个条目来解析接收到的数据报。函数使用一个变量ip和port来保存接收到的数据报的原始出度地址和本地发送者，并返回该本地发送者的IP和端口。


```go
// +build linux

package udp

import (
	"syscall"

	"golang.org/x/sys/unix"
	"v2ray.com/core/common/net"
)

func RetrieveOriginalDest(oob []byte) net.Destination {
	msgs, err := syscall.ParseSocketControlMessage(oob)
	if err != nil {
		return net.Destination{}
	}
	for _, msg := range msgs {
		if msg.Header.Level == syscall.SOL_IP && msg.Header.Type == syscall.IP_RECVORIGDSTADDR {
			ip := net.IPAddress(msg.Data[4:8])
			port := net.PortFromBytes(msg.Data[2:4])
			return net.UDPDestination(ip, port)
		} else if msg.Header.Level == syscall.SOL_IPV6 && msg.Header.Type == unix.IPV6_RECVORIGDSTADDR {
			ip := net.IPAddress(msg.Data[8:24])
			port := net.PortFromBytes(msg.Data[2:4])
			return net.UDPDestination(ip, port)
		}
	}
	return net.Destination{}
}

```

此函数的作用是读取并返回从UDP连接中接收到的新数据报。

具体来说，它接收两个参数：一个UDP连接对象(conn)和一个字节数组(payload)和一个字节数组(oob)。它通过调用conn的ReadMsgUDP函数来读取数据报，并将其存储在payload和oob中。

函数返回三个值：

- 第二个返回值(int)表示数据报中的有效负载(payload-offset)从0开始；
- 第三个返回值(int)表示 oob中的数据个数；
- 第四个返回值(int)表示新数据报中的源IP地址，如果已知的源IP地址被丢弃，则该值将为0。

最后一个参数error表示一个网络错误。


```go
func ReadUDPMsg(conn *net.UDPConn, payload []byte, oob []byte) (int, int, int, *net.UDPAddr, error) {
	return conn.ReadMsgUDP(payload, oob)
}

```