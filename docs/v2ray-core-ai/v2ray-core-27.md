# v2ray-core源码解析 27

# `common/protocol/context.go`

这段代码定义了一个名为"protocol"的包，其中包含了一些类型和常量。

定义了一个名为"key"的类型，并将其赋值为iota，这样它就成为了一个自动增长的全局变量。

定义了一个名为"requestKey"的常量，并将其赋值为iota，与上面同名的类型变量。

最后，定义了一个名为"ContextWithRequestHeader"的函数，它接收一个上下文上下文和一个请求头。它将请求头添加到上下文上下文中，并返回它。


```go
package protocol

import (
	"context"
)

type key int

const (
	requestKey key = iota
)

func ContextWithRequestHeader(ctx context.Context, request *RequestHeader) context.Context {
	return context.WithValue(ctx, requestKey, request)
}

```

该函数`RequestHeaderFromContext`从一个名为`ctx`的上下文上下文对象中获取请求头(`RequestHeader`类型)。如果上下文上下文中的`requestHeader`字段是`nil`，那么函数返回`nil`。如果`request`字段是一个有效的`RequestHeader`实例，那么函数返回该实例。

简单来说，该函数的作用是获取请求头并返回，前提是上下文上下文中的`requestHeader`字段存在。


```go
func RequestHeaderFromContext(ctx context.Context) *RequestHeader {
	request := ctx.Value(requestKey)
	if request == nil {
		return nil
	}
	return request.(*RequestHeader)
}

```

# `common/protocol/errors.generated.go`

这段代码定义了一个名为 "protocol" 的包，其中包含以下内容：

1. 导入了 "v2ray.com/core/common/errors" 包，可能是用于导入一些与错误处理相关的函数或类型。
2. 定义了一个名为 "errPathObjHolder" 的结构体，该结构体包含一个空的字符串对象 "errPathObjHolder{}"，用于表示错误路径对象的初始化值。
3. 定义了一个名为 "newError" 的函数，该函数接收多个参数，这些参数可能是用于错误信息的值，如整数、字符串等。该函数返回一个名为 "errors.Error" 的类型，该类型代表一个带有错误信息对象和一个包含错误路径对象的错误对象。该函数还使用 "WithPathObj" 方法对错误对象进行初始化，其中 "errPathObjHolder" 字段用于存储错误路径对象。这个初始化需要在调用函数后执行，因此 "WithPathObj" 方法中的 "errPathObjHolder" 字段必须是有效的。
4. 没有其他说明或定义，因此该代码片段的作用和使用方式无法确定。


```go
package protocol

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `common/protocol/headers.go`

这段代码定义了一个名为 "protocol" 的包，其中包含了一些与请求相关的常量以及一个名为 "RequestCommand" 的自定义命令类型。

具体来说，这段代码定义了一个名为 "RequestCommand" 的类型，它包含一个字节数组 "command"，这个 "command" 有三个枚举类型：RequestCommandTCP、RequestCommandUDP 和 RequestCommandMux。每个枚举类型都有一个对应的请求命令编号，分别为 0x01、0x02 和 0x03。

然后，代码定义了一个名为 "RequestCommandTCP" 的常量，它的值为 0x01，表示使用 TCP 协议发送请求。同样地，代码定义了一个名为 "RequestCommandUDP" 的常量，它的值为 0x02，表示使用 UDP 协议发送请求。最后，代码定义了一个名为 "RequestCommandMux" 的常量，它的值为 0x03，表示使用多路复用协议发送请求。

这段代码的作用是定义了一个请求命令类型 "RequestCommand"，以及三个常量 "RequestCommandTCP"、"RequestCommandUDP" 和 "RequestCommandMux"，它们分别表示使用 TCP、UDP 和多路复用协议发送请求。


```go
package protocol

import (
	"runtime"

	"v2ray.com/core/common/bitmask"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/uuid"
)

// RequestCommand is a custom command in a proxy request.
type RequestCommand byte

const (
	RequestCommandTCP = RequestCommand(0x01)
	RequestCommandUDP = RequestCommand(0x02)
	RequestCommandMux = RequestCommand(0x03)
)

```

这段代码定义了一个名为func的函数，它接受一个名为c的参数，并返回一个名为TransferType的整数。

函数的实现是使用一个switch语句，根据传入的c参数的类型来确定TransferType的类型。

具体来说，如果c参数是RequestCommandTCP或RequestCommandMux，那么函数将返回TransferTypeStream类型的值；如果c参数是RequestCommandUDP，那么函数将返回TransferTypePacket类型的值。如果c参数的类型不是TCP、MUX或UDP之一，那么函数将返回TransferTypeStream类型的值。

函数还定义了一个名为RequestOptionChunkStream的常量，它表示请求负载是否分成块。另外，还定义了名为RequestOptionConnectionReuse、RequestOptionChunkMasking和RequestOptionGlobalPadding的常量。


```go
func (c RequestCommand) TransferType() TransferType {
	switch c {
	case RequestCommandTCP, RequestCommandMux:
		return TransferTypeStream
	case RequestCommandUDP:
		return TransferTypePacket
	default:
		return TransferTypeStream
	}
}

const (
	// RequestOptionChunkStream indicates request payload is chunked. Each chunk consists of length, authentication and payload.
	RequestOptionChunkStream bitmask.Byte = 0x01

	// RequestOptionConnectionReuse indicates client side expects to reuse the connection.
	RequestOptionConnectionReuse bitmask.Byte = 0x02

	RequestOptionChunkMasking bitmask.Byte = 0x04

	RequestOptionGlobalPadding bitmask.Byte = 0x08
)

```

这段代码定义了一个名为 RequestHeader 的结构体，用于表示请求头信息。该结构体包含以下字段：

* Version: 请求头版本号，设置为 0 表示未完成。
* Command: 请求命令，可以设置为 RequestCommandUDP 或 RequestCommandTCP。
* Option: 选项字段，可以设置为 bitmask.Byte 类型的字段，用于标记其他选项。
* Security: 安全类型，可以设置为 SecurityTypeUnknown、SecurityTypeBasic、SecurityTypeSquid 等。
* Port: 目标端口。
* Address: 目标主机。
* User: 用于携带请求的用户代理内存缓冲区。

另外，还包含一个名为 Destination 的函数，用于设置请求的的目标端口。如果请求命令是 RequestCommandUDP，则将目标设置为指定的 IP 地址和端口；否则将目标设置为指定的 IP 地址。


```go
type RequestHeader struct {
	Version  byte
	Command  RequestCommand
	Option   bitmask.Byte
	Security SecurityType
	Port     net.Port
	Address  net.Address
	User     *MemoryUser
}

func (h *RequestHeader) Destination() net.Destination {
	if h.Command == RequestCommandUDP {
		return net.UDPDestination(h.Address, h.Port)
	}
	return net.TCPDestination(h.Address, h.Port)
}

```

这段代码定义了一个名为 ResponseHeader 的结构体，用于表示 HTTP 请求头中的选项。

ResponseHeader 包含一个名为 Option 的 bitmask.Byte 类型的字段，用于表示是否允许应用某些选项，以及一个名为 Command 的 ResponseCommand 类型的字段，用于表示请求的操作类型。

同时，还定义了一个名为 CommandSwitchAccount 的 struct 类型，用于表示用于账户控制命令行开关的配置信息，包括主机、端口、ID 和命令级别等。


```go
const (
	ResponseOptionConnectionReuse bitmask.Byte = 0x01
)

type ResponseCommand interface{}

type ResponseHeader struct {
	Option  bitmask.Byte
	Command ResponseCommand
}

type CommandSwitchAccount struct {
	Host     net.Address
	Port     net.Port
	ID       uuid.UUID
	Level    uint32
	AlterIds uint16
	ValidMin byte
}

```

这两段代码定义了两个函数：

1. `func (sc *SecurityConfig) GetSecurityType() SecurityType`：该函数接收一个 `SecurityConfig` 类型的参数 `sc`，并返回一个 `SecurityType` 类型的变量。函数的作用是获取 `sc` 所表示的 `SecurityConfig` 的安全类型。函数的实现使用了两个条件分支，如果 `sc` 为 `nil` 或者 `sc.Type` 为 `SecurityType_AUTO`，则返回 `SecurityType_AES128_GCM`；否则，根据所支持的架构返回不同的安全类型。

2. `func isDomainTooLong(domain string) bool`：该函数接收一个 `domain` 类型的参数，并返回一个布尔值。函数的作用是判断给定的 `domain` 是否太长。函数实现了一个简单的判断，如果 `domain` 长度超过 256 个字符，则返回 `true`，否则返回 `false`。


```go
func (sc *SecurityConfig) GetSecurityType() SecurityType {
	if sc == nil || sc.Type == SecurityType_AUTO {
		if runtime.GOARCH == "amd64" || runtime.GOARCH == "s390x" || runtime.GOARCH == "arm64" {
			return SecurityType_AES128_GCM
		}
		return SecurityType_CHACHA20_POLY1305
	}
	return sc.Type
}

func isDomainTooLong(domain string) bool {
	return len(domain) > 256
}

```

# `common/protocol/headers.pb.go`

这段代码定义了一个名为"protocol"的包，其中包含了一些定义了通用协议的接口和相关的定义。以下是这段代码的主要作用：

1. 定义了两个接口：proto.Message和proto.Message骨折件。其中，proto.Message用于定义消息类型的接口，而proto.MessageBinary为定义二进制消息类型的接口。

2. 注册了这两个接口的类型信息。注册的方式是通过反射(reflect)库来实现的，通过在类型上添加类型后缀(.proto)来标识这些接口。

3. 实现了一个名为"sync"的同步类。该类的实现使用了sync.Chan，创建了一个无缓冲的通道，可以在通道上发送和接收数据。

4. 通过使用protoreflect库，实现了对protoc generated的类型信息的后置(getter)机制。这使得可以很方便地使用go types工具来生成go.垃圾收集器不能处理的reflect.接口类型。

这段代码定义了一个用于定义通用协议的包，包含了对消息类型定义的接口和实现，以及用于同步数据的无缓冲通道。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: common/protocol/headers.proto

package protocol

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码的作用是验证 Hessian 分发包是否足够安全。具体来说：

1. 验证运行时/protoimpl 是否足够安全：确保使用了最新版本的 Hessian 分发包，版本为 20 - protoimpl.MinVersion。
2. 验证 Hessian 分发包是否足够安全：确保使用了不早于 protoimpl.MaxVersion 20 的最新版本的 Hessian 分发包。
3. 验证 protobuf 包是否足够安全：执行一个编译时检验，检验足够安全的 version 4 是否存在于 Hessian 分发包中。

这里的作用域在函数内部，所以不会输出函数体外部的 Hessian 分发包。


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

type SecurityType int32

const (
	SecurityType_UNKNOWN           SecurityType = 0
	SecurityType_LEGACY            SecurityType = 1
	SecurityType_AUTO              SecurityType = 2
	SecurityType_AES128_GCM        SecurityType = 3
	SecurityType_CHACHA20_POLY1305 SecurityType = 4
	SecurityType_NONE              SecurityType = 5
)

```

这段代码定义了一个枚举类型 SecurityType，并创建了两个变量 SecurityType_name 和 SecurityType_value。

SecurityType_name 是一个包含 5 个元素的 map[int32]string 类型，它定义了每个 SecurityType 的名称。其中，每个元素的值为 0、1、2 或 3，分别表示相应的 SecurityType 类型。

SecurityType_value 是一个包含 5 个元素的 map[string]int32 类型，它定义了每个 SecurityType 类型的默认值。其中，每个元素的值为 0、1、2 或 3，分别表示相应的 SecurityType 类型。

通过这两个 map 类型，可以方便地根据需要选择 SecurityType 类型，并获取对应的默认值。


```go
// Enum value maps for SecurityType.
var (
	SecurityType_name = map[int32]string{
		0: "UNKNOWN",
		1: "LEGACY",
		2: "AUTO",
		3: "AES128_GCM",
		4: "CHACHA20_POLY1305",
		5: "NONE",
	}
	SecurityType_value = map[string]int32{
		"UNKNOWN":           0,
		"LEGACY":            1,
		"AUTO":              2,
		"AES128_GCM":        3,
		"CHACHA20_POLY1305": 4,
		"NONE":              5,
	}
)

```

这段代码定义了一个名为 "SecurityType" 的枚举类型，并实现了两个函数：

1. "Enum()"，返回一个指向 "SecurityType" 类型的指针。函数创建了一个新的 "SecurityType" 类型的实例，将其赋值为传入的参数 "x"，然后返回这个实例。

2. "String()"，返回一个字符串类型的安全类型。函数实现了 "protoimpl.X.EnumStringOf" 和 "protoreflect.EnumNumber" 函数，用于将 "SecurityType" 类型转换为字符串类型。具体地，函数将 "x" 传递给 "protoimpl.X.EnumStringOf" 函数，然后将其结果与 "protoreflect.EnumNumber" 函数中的 "x" 字面量类型进行绑定，最后返回生成的字符串。

3. "Descriptor()"，返回一个 "protoreflect.EnumDescriptor" 类型，该类型定义了 "SecurityType" 枚举类型的元数据。函数返回了 file_common_protocol_headers_proto_enumTypes[0] 类型定义的 Descriptor() 函数的实现，因此返回了一个 "protoreflect.EnumDescriptor" 类型，其中包含了枚举类型名称、类型标识符、默认值等信息。

4. "Type()"，返回一个 "protoreflect.EnumType" 类型，该类型定义了 "SecurityType" 枚举类型的成员变量。函数返回了 "file_common_protocol_headers_proto_enumTypes[0]" 类型定义的 Descriptor() 函数的实现，因此返回了一个 "protoreflect.EnumType" 类型，其中包含了枚举类型名称、类型标识符、默认值等信息。


```go
func (x SecurityType) Enum() *SecurityType {
	p := new(SecurityType)
	*p = x
	return p
}

func (x SecurityType) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (SecurityType) Descriptor() protoreflect.EnumDescriptor {
	return file_common_protocol_headers_proto_enumTypes[0].Descriptor()
}

func (SecurityType) Type() protoreflect.EnumType {
	return &file_common_protocol_headers_proto_enumTypes[0]
}

```

这段代码定义了两个函数，以及一个结构体类型。

第一个函数名为 `func (x SecurityType) Number() protoreflect.EnumNumber`，它接收一个 `x` 类型的参数，并返回一个名为 `Number` 的 `protoreflect.EnumNumber` 类型的值。函数的作用是获取 `x` 类型的对象的安全性类型，并返回一个与该类型相对应的枚举类型。

第二个函数名为 `func (SecurityType) EnumDescriptor() ([]byte, []int)`，它接收一个 `SecurityType` 类型的参数，并返回一个名为 `EnumDescriptor` 的接口类型，该类型包含一个 `desc` 字段和一个 `fields` 字段的指针。函数的作用是在不安全的情况下获取与 `SecurityType` 相关的元数据，并返回一个字节切片和一个整数切片，用于指定 `SecurityType` 对应的字段名称和类型。

第三个结构体类型名为 `SecurityConfig`，它包含了一个名为 `state` 的字段，它代表了一个 `protoimpl.MessageState` 类型的消息状态，用于保存系统当前的安全状态。其次，`sizeCache` 字段是一个 `protoimpl.SizeCache` 类型的消息，它用于保存消息的大致大小，以避免因消息过长而引发的内存问题。最后，`unknownFields` 字段是一个 `protoimpl.UnknownFields` 类型的消息，它用于保存定义时无法获取的元数据。

函数的作用是将接收到的 `SecurityType` 对象转换为 `Number` 类型，获取与 `SecurityType` 相关的元数据，并返回一个字节切片和一个整数切片，用于指定 `SecurityType` 对应的字段名称和类型。


```go
func (x SecurityType) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

// Deprecated: Use SecurityType.Descriptor instead.
func (SecurityType) EnumDescriptor() ([]byte, []int) {
	return file_common_protocol_headers_proto_rawDescGZIP(), []int{0}
}

type SecurityConfig struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Type SecurityType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.common.protocol.SecurityType" json:"type,omitempty"`
}

```

这段代码定义了两个函数：

1. `Reset()`函数接收一个 `*SecurityConfig` 类型的参数，并将其赋值为 `SecurityConfig{}`。然后，代码检查 `protoimpl.UnsafeEnabled` 是否为 `true`，如果是，则执行以下操作：

 - 创建一个空的 `*SecurityConfig` 类型，并将其赋值给 `*x`。
 - 如果 `protoimpl.UnsafeEnabled` 为 `true`，则执行以下操作：

   - 从 `file_common_protocol_headers_proto_msgTypes` 数组中获取第一个元素，并将其存储为 `*mi`。
   - 从 `protoimpl.X` 类型中获取 `MessageStateOf` 函数的返回值，并将其存储为 `ms`。
   - 将 `*mi` 和 `ms` 存储的消息信息存储为 `mi`。

2. `String()` 函数返回一个字符串表示 `*SecurityConfig` 类型的对象。

3. `ProtoMessage()` 函数返回一个消息类型 `*SecurityConfig` 类型的接口，以便在代码中使用 `ProtocolMessage` 类型的函数或类型。


```go
func (x *SecurityConfig) Reset() {
	*x = SecurityConfig{}
	if protoimpl.UnsafeEnabled {
		mi := &file_common_protocol_headers_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *SecurityConfig) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*SecurityConfig) ProtoMessage() {}

```

这段代码定义了两个函数，一个是`func (x *SecurityConfig) ProtoReflect() protoreflect.Message`，另一个是`func (*SecurityConfig) Descriptor() ([]byte, []int)`。

第一个函数`func (x *SecurityConfig) ProtoReflect() protoreflect.Message`接收一个`*SecurityConfig`类型的参数，并返回一个`protoreflect.Message`类型的指针。这个函数的作用是在函数主体外部定义了`*SecurityConfig`类型的变量，然后使用它来创建一个`protoreflect.Message`类型的变量，将这个`protoreflect.Message`类型的变量作为参数传递给`mi.MessageOf`函数，最后返回`mi`的`MessageOf`函数返回的值。

第二个函数`func (*SecurityConfig) Descriptor() ([]byte, []int)`接收一个`*SecurityConfig`类型的参数，并返回一个字节切片和一个整数数组，这两个参数分别是`file_common_protocol_headers_proto_rawDescGZIP`类型和`[]int`类型的变量。这个函数的作用是在函数主体外部定义了`*SecurityConfig`类型的变量，然后使用它来创建一个字节切片和一个整数数组，将这个`*SecurityConfig`类型的变量作为参数传递给`descriptor`函数，最后返回这两个参数的值。

这两个函数可能是在一个定义了`file_common_protocol_headers_proto`接口的函数中使用的，`file_common_protocol_headers_proto`接口定义了一些与安全配置相关的函数和方法，这个函数通过`*SecurityConfig`类型来实现对`file_common_protocol_headers_proto`接口的实例化。


```go
func (x *SecurityConfig) ProtoReflect() protoreflect.Message {
	mi := &file_common_protocol_headers_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use SecurityConfig.ProtoReflect.Descriptor instead.
func (*SecurityConfig) Descriptor() ([]byte, []int) {
	return file_common_protocol_headers_proto_rawDescGZIP(), []int{0}
}

```

It looks like a hexadecimal representation of a JSON object.

The first line starts with a JavaScript object, with the "Object" subclass and a timestamp (in this case, 2022-12-08T15:29:53.456Z). The timestamp is followed by a "null" value.

The next block of hexadecimal values represents the properties of the object. Each property starts with a type, followed by a hexadecimal value, and then a property name. For example, the first property is "终了時間" (ending time), with a hexadecimal value of 0x6f.

It appears that the rest of the properties are not included in this representation, such as the "配布焓勢" property.



```go
func (x *SecurityConfig) GetType() SecurityType {
	if x != nil {
		return x.Type
	}
	return SecurityType_UNKNOWN
}

var File_common_protocol_headers_proto protoreflect.FileDescriptor

var file_common_protocol_headers_proto_rawDesc = []byte{
	0x0a, 0x1d, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
	0x6c, 0x2f, 0x68, 0x65, 0x61, 0x64, 0x65, 0x72, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12,
	0x1a, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d,
	0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x22, 0x4e, 0x0a, 0x0e, 0x53,
	0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x3c, 0x0a,
	0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x28, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
	0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x53, 0x65, 0x63, 0x75, 0x72, 0x69, 0x74,
	0x79, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65, 0x2a, 0x62, 0x0a, 0x0c, 0x53,
	0x65, 0x63, 0x75, 0x72, 0x69, 0x74, 0x79, 0x54, 0x79, 0x70, 0x65, 0x12, 0x0b, 0x0a, 0x07, 0x55,
	0x4e, 0x4b, 0x4e, 0x4f, 0x57, 0x4e, 0x10, 0x00, 0x12, 0x0a, 0x0a, 0x06, 0x4c, 0x45, 0x47, 0x41,
	0x43, 0x59, 0x10, 0x01, 0x12, 0x08, 0x0a, 0x04, 0x41, 0x55, 0x54, 0x4f, 0x10, 0x02, 0x12, 0x0e,
	0x0a, 0x0a, 0x41, 0x45, 0x53, 0x31, 0x32, 0x38, 0x5f, 0x47, 0x43, 0x4d, 0x10, 0x03, 0x12, 0x15,
	0x0a, 0x11, 0x43, 0x48, 0x41, 0x43, 0x48, 0x41, 0x32, 0x30, 0x5f, 0x50, 0x4f, 0x4c, 0x59, 0x31,
	0x33, 0x30, 0x35, 0x10, 0x04, 0x12, 0x08, 0x0a, 0x04, 0x4e, 0x4f, 0x4e, 0x45, 0x10, 0x05, 0x42,
	0x5f, 0x0a, 0x1e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
	0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
	0x6c, 0x50, 0x01, 0x5a, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
	0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
	0x63, 0x6f, 0x6c, 0xaa, 0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c,
	0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

此代码定义了一个名为file_common_protocol_headers_proto_rawDescOnce的变量，其类型为sync.Once。该变量确保了该变量的作用域仅在函数内部。

函数file_common_protocol_headers_proto_rawDescGZIP()返回一个字节切片，其中包含经过GZIP压缩的file_common_protocol_headers_proto_rawDescData。

file_common_protocol_headers_proto_enumTypes是一个名为file_common_protocol_headers_proto_enumTypes的var切片，其中包含一个名为SecurityType的固定类型和一个名为SecurityConfig的固定类型。

file_common_protocol_headers_proto_msgTypes是一个名为file_common_protocol_headers_proto_msgTypes的var切片，其中包含一个名为RequestAnalysisConfig的固定类型和一个名为ResponseAnalysisConfig的固定类型。

file_common_protocol_headers_proto_goTypes定义了一个名为file_common_protocol_headers_proto_goTypes的var切片，其中包含一个名为SecurityType的固定类型和一个名为SecurityConfig的固定类型。


```go
var (
	file_common_protocol_headers_proto_rawDescOnce sync.Once
	file_common_protocol_headers_proto_rawDescData = file_common_protocol_headers_proto_rawDesc
)

func file_common_protocol_headers_proto_rawDescGZIP() []byte {
	file_common_protocol_headers_proto_rawDescOnce.Do(func() {
		file_common_protocol_headers_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_protocol_headers_proto_rawDescData)
	})
	return file_common_protocol_headers_proto_rawDescData
}

var file_common_protocol_headers_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_common_protocol_headers_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_protocol_headers_proto_goTypes = []interface{}{
	(SecurityType)(0),      // 0: v2ray.core.common.protocol.SecurityType
	(*SecurityConfig)(nil), // 1: v2ray.core.common.protocol.SecurityConfig
}
```

This is a JavaScript file that defines the structure and contents of a `file_common_protocol_headers_proto` message.

The `file_common_protocol_headers_proto` message type has two fields: `state` and `sizeCache`. The `state` field is a `SecurityConfig` struct that represents the state information of the protocol header. The `sizeCache` field is a `SecurityConfigSizeCache` struct that represents the cache size information of the protocol header.

The file initializes the `file_common_protocol_headers_proto` message type with the initialized structure of the `file_common_protocol_headers_proto` message.


```go
var file_common_protocol_headers_proto_depIdxs = []int32{
	0, // 0: v2ray.core.common.protocol.SecurityConfig.type:type_name -> v2ray.core.common.protocol.SecurityType
	1, // [1:1] is the sub-list for method output_type
	1, // [1:1] is the sub-list for method input_type
	1, // [1:1] is the sub-list for extension type_name
	1, // [1:1] is the sub-list for extension extendee
	0, // [0:1] is the sub-list for field type_name
}

func init() { file_common_protocol_headers_proto_init() }
func file_common_protocol_headers_proto_init() {
	if File_common_protocol_headers_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_common_protocol_headers_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*SecurityConfig); i {
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
			RawDescriptor: file_common_protocol_headers_proto_rawDesc,
			NumEnums:      1,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_common_protocol_headers_proto_goTypes,
		DependencyIndexes: file_common_protocol_headers_proto_depIdxs,
		EnumInfos:         file_common_protocol_headers_proto_enumTypes,
		MessageInfos:      file_common_protocol_headers_proto_msgTypes,
	}.Build()
	File_common_protocol_headers_proto = out.File
	file_common_protocol_headers_proto_rawDesc = nil
	file_common_protocol_headers_proto_goTypes = nil
	file_common_protocol_headers_proto_depIdxs = nil
}

```

# `common/protocol/id.go`

这段代码定义了一个名为"protocol"的包，以及它所需的导入和常量。

它导入了两个加密哈希函数："hmac"和"md5"，以及一个名为"uuid"的哈希函数。

它定义了一个名为"IDBytesLen"的常量，表示ID字段的长度，该字段用于标识和区分不同的消息。

最后，它导入了两个名为"v2ray.com/core/common"和"v2ray.com/core/common/uuid"的包，它们可能是用于与第三方库或服务进行交互的包。


```go
package protocol

import (
	"crypto/hmac"
	"crypto/md5"
	"hash"

	"v2ray.com/core/common"
	"v2ray.com/core/common/uuid"
)

const (
	IDBytesLen = 16
)

```

这段代码定义了一个名为ID的 struct 类型，该类型包含一个 UUID 和一个命令键（一个字节数组，长度为 IDBytesLen）。

另外，还定义了一个名为 IDHash 的函数类型，该函数接受一个字节数组作为输入参数，并返回一个名为 hash.Hash 的哈希函数实现。该函数使用了一个名为 hmac.New 的实现，该实现使用 MD5 哈希算法创建一个新哈希表。

然后，定义了一个名为 DefaultIDHash 的函数实现，该函数实现了 IDHash 函数类型的哈希函数，但在这里使用了一个默认的实现，即使用哈希函数 NewIDHash 实现。

最后，定义了一个名为 ID 的常量类型，该类型包含一个 UUID 和一个命令键（一个字节数组，长度为 IDBytesLen）。以及一个名为 Equals 的函数实现，该函数比较两个 ID 类型的对象的 uuid 字段是否相等。


```go
type IDHash func(key []byte) hash.Hash

func DefaultIDHash(key []byte) hash.Hash {
	return hmac.New(md5.New, key)
}

// The ID of en entity, in the form of a UUID.
type ID struct {
	uuid   uuid.UUID
	cmdKey [IDBytesLen]byte
}

// Equals returns true if this ID equals to the other one.
func (id *ID) Equals(another *ID) bool {
	return id.uuid.Equals(&(another.uuid))
}

```

这段代码定义了三个名为ID的结构的体，并实现了四个函数：

1. `func (id *ID) Bytes() []byte`，该函数接收一个ID类型的指针，并返回一个字节数组，该数组包含ID的UUID。
2. `func (id *ID) String() string`，该函数接收一个ID类型的指针，并返回一个字符串，该字符串表示ID的UUID。
3. `func (id *ID) UUID() uuid.UUID`，该函数接收一个ID类型的指针，并返回一个UUID类型，该UUID表示ID的UUID。
4. `func (id *ID) CmdKey() []byte`，该函数接收一个ID类型的指针，并返回一个字节数组，该数组是ID的命令键。


```go
func (id *ID) Bytes() []byte {
	return id.uuid.Bytes()
}

func (id *ID) String() string {
	return id.uuid.String()
}

func (id *ID) UUID() uuid.UUID {
	return id.uuid
}

func (id ID) CmdKey() []byte {
	return id.cmdKey[:]
}

```

该代码定义了两个名为`NewID`和`nextID`的函数，它们都接受一个名为`uuid`的UUID参数。

函数`NewID`的作用是生成一个满足UUID规范的ID，并返回该ID的`*ID`类型。具体实现过程如下：

1. 创建一个名为`id`的`ID`对象，其中`uuid`字段设置为传入的`uuid`参数。

2. 创建一个名为`md5hash`的`md5`哈希函数，并使用`md5.New()`函数创建一个新的哈希函数。

3. 使用`md5hash`函数的`Write()`方法将`uuid`字节编码的哈希值和`c48619fe-8f02-49e0-b9e9-edf763e17e21`字节编码的哈希值作为参数进行哈希运算，得到一个新的哈希值作为`id`对象的`cmdKey`字段。

4. 返回`id`作为新生成的ID的`*ID`类型。

函数`nextID`的作用是在给定`u` UUID的基础上生成一个新的UUID，并返回该新UUID。具体实现过程如下：

1. 创建一个名为`newid`的新生UUID。

2. 创建一个名为`md5hash`的`md5`哈希函数，并使用`md5.New()`函数创建一个新的哈希函数。

3. 使用`md5hash`函数的`Write()`方法将`u` UUID字节编码的哈希值作为参数进行哈希运算，得到一个新的哈希值作为`newid`对象的`cmdKey`字段。

4. 循环执行以下步骤：

a. 使用`md5hash.Sum()`方法计算新生UUID的哈希值。

b. 如果新生UUID与`u` UUID不相等，则返回新生UUID。

c. 每次循环执行完毕后，将哈希值作为参数传递给下一个哈希函数，以获取下一个可能的新生UUID。

d. 循环继续执行，直到生成多个不同的哈希值，或者不再生成新的UUID为止。


```go
// NewID returns an ID with given UUID.
func NewID(uuid uuid.UUID) *ID {
	id := &ID{uuid: uuid}
	md5hash := md5.New()
	common.Must2(md5hash.Write(uuid.Bytes()))
	common.Must2(md5hash.Write([]byte("c48619fe-8f02-49e0-b9e9-edf763e17e21")))
	md5hash.Sum(id.cmdKey[:0])
	return id
}

func nextID(u *uuid.UUID) uuid.UUID {
	md5hash := md5.New()
	common.Must2(md5hash.Write(u.Bytes()))
	common.Must2(md5hash.Write([]byte("16167dc8-16b6-4e6d-b8bb-65dd68113a81")))
	var newid uuid.UUID
	for {
		md5hash.Sum(newid[:0])
		if !newid.Equals(u) {
			return newid
		}
		common.Must2(md5hash.Write([]byte("533eff8a-4113-4b10-b5ce-0f5d76b98cd2")))
	}
}

```

此代码定义了一个名为`NewAlterIDs`的函数，接受两个参数：`primary`和`alterIDCount`，函数返回一个包含`alterIDs`的数组，每个`alterIDs`都是一个指向`ID`类型的指针。

函数的主要逻辑如下：

1. 首先，函数使用`make`函数创建一个名为`alterIDs`的数组，其大小为`alterIDCount`，用于存储每个`ID`类型对象的引用。

2. 函数初始化一个名为`prevID`的`ID`类型变量，并将其初始化为调用者提供的`primary`参数的UUID。

3. 函数使用一个循环遍历`alterIDs`数组中的每个子元素。对于每个子元素，函数使用`nextID`函数获取一个随机ID，并将该ID添加到相应的`ID`对象中。然后，函数将`prevID`变量设置为新ID。

4. 函数返回`alterIDs`数组，每个`ID`类型对象都指向一个`ID`类型的对象。


```go
func NewAlterIDs(primary *ID, alterIDCount uint16) []*ID {
	alterIDs := make([]*ID, alterIDCount)
	prevID := primary.UUID()
	for idx := range alterIDs {
		newid := nextID(&prevID)
		alterIDs[idx] = NewID(newid)
		prevID = newid
	}
	return alterIDs
}

```

# `common/protocol/id_test.go`

这段代码是针对一个名为 "protocol_test" 的包进行的测试。该包包含一个名为 "TestIdEquals" 的函数。

该函数的作用是测试两个 UUID 是否相等，并验证它们是否具有相同的字符串表示。

具体来说，代码首先创建两个 UUID 对象 id1 和 id2。然后，函数使用 NewID 函数为这两个 UUID 对象分配一个测试ID。接下来，函数使用Equals 函数比较这两个 UUID 对象。如果它们不相等，函数会在测试中输出错误信息。

如果 id1 和 id2 的字符串表示相同，函数也会输出错误信息。

这段代码的目的是测试 UUID 对象是否具有正确的 UUID 格式，以及是否能够通过字符串比较它们。


```go
package protocol_test

import (
	"testing"

	. "v2ray.com/core/common/protocol"
	"v2ray.com/core/common/uuid"
)

func TestIdEquals(t *testing.T) {
	id1 := NewID(uuid.New())
	id2 := NewID(id1.UUID())

	if !id1.Equals(id2) {
		t.Error("expected id1 to equal id2, but actually not")
	}

	if id1.String() != id2.String() {
		t.Error(id1.String(), " != ", id2.String())
	}
}

```

# `common/protocol/payload.go`

这段代码定义了一个名为`protocol`的包，其中包含了一些数据类型和常量。

`TransferType`是一个字节类型的枚举类型，定义了两种不同的传输类型：`TransferTypeStream`和`TransferTypePacket`。这两种类型的具体含义可以在后面通过`具体的`实现来定义。

`AddressType`也是一个字节类型的枚举类型，定义了四种不同的地址类型：`AddressTypeIPv4`、`AddressTypeDomain`和`AddressTypeIPv6`。这些地址类型可以用来标识一个实体在网络中的位置，例如一个IP地址、一个域名或一个IPv6地址。

这段代码没有包含任何函数或变量，只是一个定义了数据类型的部分。


```go
package protocol

type TransferType byte

const (
	TransferTypeStream TransferType = 0
	TransferTypePacket TransferType = 1
)

type AddressType byte

const (
	AddressTypeIPv4   AddressType = 1
	AddressTypeDomain AddressType = 2
	AddressTypeIPv6   AddressType = 3
)

```

# `common/protocol/protocol.go`

这段代码是一个 Go 语言中的第三方库 `v2ray.com/core/common/protocol` 的导入语句。它将从该库中导入 `v2ray.com/core/common/protocol` 包，以便在程序中使用该包中的某些功能。

此外，该代码还会运行一个额外的命令 `go run v2ray.com/core/common/errors/errorgen`，该命令将使用 `go run` 命令运行一个名为 `v2ray.com/core/common/errors/errorgen` 的源代码文件。这个源代码文件可能是一个用于生成错误信息的工具，其目的是在代码中报告潜在的错误。


```go
package protocol // import "v2ray.com/core/common/protocol"

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `common/protocol/server_picker.go`

这段代码定义了一个名为`ServerList`的结构体，它包含一个`sync.RWMutex`类型的字段`servers`和一个`ServerSpec`类型的字段`sync.WaitGroup`类型的字段`waitGroup`。

`NewServerList`函数返回一个`ServerList`实例，初始化时创建一个空的字节数组作为`servers`字段的值，并将`waitGroup`设置为没有等待的组。

`func`函数是用来定义外部的接口，没有实现任何功能。


```go
package protocol

import (
	"sync"
)

type ServerList struct {
	sync.RWMutex
	servers []*ServerSpec
}

func NewServerList() *ServerList {
	return &ServerList{}
}

```

这段代码定义了一个名为`ServerList`的`ServerSpec`类型切片，以及三个函数：

1. `AddServer`函数，该函数接收一个`ServerSpec`类型的切片和一个`ServerSpec`类型的服务器，将其添加到`sl.servers`数组中。

2. `Size`函数，该函数返回`sl.servers`数组的长度。

3. `GetServer`函数，该函数接收一个整数`idx`，返回`sl.servers`数组中索引为`idx`的服务器`ServerSpec`类型。

为了确保函数的安全性，函数内部使用了`Lock`和`Unlock`操作符获取互斥锁，以确保在同一时间只有一个进程对数组进行添加、删除或修改操作。另外，函数内部还使用了`RLock`和`RUnlock`操作符获取读取锁，以确保在同一时间只有一个进程读取数组元素。


```go
func (sl *ServerList) AddServer(server *ServerSpec) {
	sl.Lock()
	defer sl.Unlock()

	sl.servers = append(sl.servers, server)
}

func (sl *ServerList) Size() uint32 {
	sl.RLock()
	defer sl.RUnlock()

	return uint32(len(sl.servers))
}

func (sl *ServerList) GetServer(idx uint32) *ServerSpec {
	sl.Lock()
	defer sl.Unlock()

	for {
		if idx >= uint32(len(sl.servers)) {
			return nil
		}

		server := sl.servers[idx]
		if !server.IsValid() {
			sl.removeServer(idx)
			continue
		}

		return server
	}
}

```

该代码定义了一个名为 func 的函数，接收一个名为 sl 的 ServerList 类型变量和一个名为 idx 的 uint32 类型参数。函数的作用是移除索引为 idx 的服务器并将其从服务器列表中删除，同时将索引为 idx 的位置替换为从服务器列表中剩余的最大的服务器的位置。

该函数定义了一个名为 RoundRobinServerPicker 的接口，该接口实现了一个随机选择服务器的方法。该接口需要一个名为 serverlist 的 ServerList 类型变量和一个名为 nextIndex 的 uint32 类型变量。

接着函数内部定义了一个名为 nextIndex 的变量，用于记录下一个服务器的位置，由于每次循环都是从服务器列表中取出下一个服务器，因此 nextIndex 的初始值应该是服务器列表的长度减一，即 n-1。

然后函数通过一系列操作，实现了从服务器列表中取出索引为 idx 的服务器并将其删除，同时将索引为 idx 的位置替换为服务器列表中剩余的最大的服务器的位置。最后将 modifiedList 赋值为 sl.servers[:n-1] 即从服务器列表中取出 n-1 个服务器，并返回该 modifiedList。


```go
func (sl *ServerList) removeServer(idx uint32) {
	n := len(sl.servers)
	sl.servers[idx] = sl.servers[n-1]
	sl.servers = sl.servers[:n-1]
}

type ServerPicker interface {
	PickServer() *ServerSpec
}

type RoundRobinServerPicker struct {
	sync.Mutex
	serverlist *ServerList
	nextIndex  uint32
}

```

该代码定义了一个名为NewRoundRobinServerPicker的函数，它接收一个名为serverlist的指向服务器列表的指针变量，并返回一个指向名为RoundRobinServerPicker的指针变量。

函数内部，首先定义了一个nextIndex变量，用于跟踪循环服务器列表中下一个要选中的服务器的索引，然后从服务器列表中获取当前服务器，并将其存储在next变量中。如果当前服务器为空，将将next变量设置为0，并将服务器列表的长度设置为0，以确保不越界访问列表中的元素。

接下来，增加next变量，以便在循环结束后，将next变量设置为0并从头开始循环。如果next变量已达到服务器列表的长度，将next变量重置为0，并循环回到始于0的位置。

最后，函数返回选中的服务器，通过调用 PickServer函数，可以访问并使用所选的服务器。


```go
func NewRoundRobinServerPicker(serverlist *ServerList) *RoundRobinServerPicker {
	return &RoundRobinServerPicker{
		serverlist: serverlist,
		nextIndex:  0,
	}
}

func (p *RoundRobinServerPicker) PickServer() *ServerSpec {
	p.Lock()
	defer p.Unlock()

	next := p.nextIndex
	server := p.serverlist.GetServer(next)
	if server == nil {
		next = 0
		server = p.serverlist.GetServer(0)
	}
	next++
	if next >= p.serverlist.Size() {
		next = 0
	}
	p.nextIndex = next

	return server
}

```

# `common/protocol/server_picker_test.go`

该代码的作用是测试一个名为"server_list"的函数，该函数使用v2ray.com.的包来创建一个服务器列表。

具体来说，该函数会创建两个服务器，并将它们添加到服务器列表中。第一个服务器将连接到本地计算机的IP地址（通过v2ray.com.中的net.TCPDestination函数指定），第二个服务器将连接到本地计算机的IP地址，并在加入服务器列表后立即加入服务器。

接下来，该函数使用list.GetServer函数获取服务器列表中的第二个服务器（索引为1），并使用该服务器上的一个名为"GetServer"的函数获取该服务器上的Destination字段。如果该服务器上的Destination字段的值不是2，则t.Error函数会输出错误信息。

在该函数的最后，该函数使用另一个名为"GetServer"的函数获取服务器列表中的第一个服务器（索引为0），并使用该服务器上的一个名为"GetServer"的函数获取该服务器上的Destination字段。如果该服务器上的Destination字段的值不是1，则t.Error函数会输出错误信息。


```go
package protocol_test

import (
	"testing"
	"time"

	"v2ray.com/core/common/net"
	. "v2ray.com/core/common/protocol"
)

func TestServerList(t *testing.T) {
	list := NewServerList()
	list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(1)), AlwaysValid()))
	if list.Size() != 1 {
		t.Error("list size: ", list.Size())
	}
	list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(2)), BeforeTime(time.Now().Add(time.Second))))
	if list.Size() != 2 {
		t.Error("list.size: ", list.Size())
	}

	server := list.GetServer(1)
	if server.Destination().Port != 2 {
		t.Error("server: ", server.Destination())
	}
	time.Sleep(2 * time.Second)
	server = list.GetServer(1)
	if server != nil {
		t.Error("server: ", server)
	}

	server = list.GetServer(0)
	if server.Destination().Port != 1 {
		t.Error("server: ", server.Destination())
	}
}

```

该代码使用了 Go 标准库中的 `testing` 和 `time` 包，实现了一个关于服务器选择功能测试的函数 `TestServerPicker`。主要作用是测试一个名为 `ServerSpec` 的接口，该接口定义了服务器配置信息，包括 IP 地址、端口、时间戳等。

具体来说，这段代码的作用是模拟在一个服务器列表中，有多个服务器，每个服务器都包含一个 `ServerSpec` 配置，这个列表中的服务器会按照一定的规则循环选择一个服务器，然后测试选择出的服务器是否满足一定的要求，如选择的服务器 IP 地址、端口和时间戳等。如果选择的服务器不符合要求，测试函数会输出错误信息并暂停 2 秒钟，然后继续选择下一个服务器。

这段代码的主要目的是提供一个简单的测试框架，用于测试 `ServerSpec` 接口的实现和正确性。


```go
func TestServerPicker(t *testing.T) {
	list := NewServerList()
	list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(1)), AlwaysValid()))
	list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(2)), BeforeTime(time.Now().Add(time.Second))))
	list.AddServer(NewServerSpec(net.TCPDestination(net.LocalHostIP, net.Port(3)), BeforeTime(time.Now().Add(time.Second))))

	picker := NewRoundRobinServerPicker(list)
	server := picker.PickServer()
	if server.Destination().Port != 1 {
		t.Error("server: ", server.Destination())
	}
	server = picker.PickServer()
	if server.Destination().Port != 2 {
		t.Error("server: ", server.Destination())
	}
	server = picker.PickServer()
	if server.Destination().Port != 3 {
		t.Error("server: ", server.Destination())
	}
	server = picker.PickServer()
	if server.Destination().Port != 1 {
		t.Error("server: ", server.Destination())
	}

	time.Sleep(2 * time.Second)
	server = picker.PickServer()
	if server.Destination().Port != 1 {
		t.Error("server: ", server.Destination())
	}
	server = picker.PickServer()
	if server.Destination().Port != 1 {
		t.Error("server: ", server.Destination())
	}
}

```

# `common/protocol/server_spec.go`

这段代码定义了一个名为`ValidationStrategy`的接口，以及一个名为`Protocol`的包。

具体来说，这段代码实现了一个基本的验证策略，以保证客户端连接到服务器端时，客户端发送的数据请求是有效的。客户端发送的数据会经过一个验证策略，如果验证策略返回的`IsValid()`方法返回`true`，则说明客户端发送的数据是有效的，否则会触发一个异步通知客户端进行修正。

另外，由于Dice是提供的一个基于Net的网络代理服务，因此在这段代码中，还引入了Net库。这使得代码可以更好地处理网络连接相关的操作。


```go
package protocol

import (
	"sync"
	"time"

	"v2ray.com/core/common/dice"
	"v2ray.com/core/common/net"
)

type ValidationStrategy interface {
	IsValid() bool
	Invalidate()
}

```

这段代码定义了一个名为 `alwaysValidStrategy` 的 struct 类型，它包含一个 `ValidationStrategy` 类型的成员函数 `AlwaysValid` 和一个 `timeoutValidStrategy` 的成员函数 `Invalidate`。

`ValidationStrategy` 类型表示一个验证策略，这里没有具体实现，只是一个通用的类型。

`AlwaysValid` 函数是一个验证策略，这里使用了始终有效的策略来实现。它的作用是确保所有时间都不会影响验证的准确性，即使验证的参数在时间上存在延迟。

`Invalidate` 函数是一个时间限制的验证策略，它会在指定的时间内检查验证是否仍然有效。如果验证在指定的时间内失败，策略将失效。

这里没有输出具体的代码实现，只是定义了 `alwaysValidStrategy` 类型，以及 `ValidationStrategy` 和 `timeoutValidStrategy` 的成员函数。


```go
type alwaysValidStrategy struct{}

func AlwaysValid() ValidationStrategy {
	return alwaysValidStrategy{}
}

func (alwaysValidStrategy) IsValid() bool {
	return true
}

func (alwaysValidStrategy) Invalidate() {}

type timeoutValidStrategy struct {
	until time.Time
}

```

这段代码定义了一个名为“BeforeTime”的函数，它接收一个名为“t”的时间类型为“time.Time”的参数。函数返回一个名为“timeoutValidStrategy”的“ValidationStrategy”类型的变量，它包含以下两个方法：

1. “IsValid”方法：判断当前时间是否在超时时间内，如果当前时间在超时时间内，则返回 false，否则返回 true。
2. “Invalidate”方法：使当前时间的有效性状态失效，即让当前时间的有效性状态为 false。

同时，定义了一个名为“ServerSpec”的 struct 类型，包含以下字段：

1. 一个名为“sync.RWMutex”类型的变量“mutex”，用于在多个 goroutine 访问该类型的字段时进行互斥锁。
2. 一个名为“net.Destination”类型的变量“dest”，用于存储服务器的目标 IP 地址。
3. 一个名为“[]*MemoryUser”类型的变量“users”，用于存储服务器上的用户信息。
4. 一个名为“ValidationStrategy”类型的变量“valid”，用于验证客户端请求的有效性。

最后，该函数在导出时使用了一个名为“BeforeTime”的函数，因此可以推断出该函数是用于设置服务器超时时间的行为。


```go
func BeforeTime(t time.Time) ValidationStrategy {
	return &timeoutValidStrategy{
		until: t,
	}
}

func (s *timeoutValidStrategy) IsValid() bool {
	return s.until.After(time.Now())
}

func (s *timeoutValidStrategy) Invalidate() {
	s.until = time.Time{}
}

type ServerSpec struct {
	sync.RWMutex
	dest  net.Destination
	users []*MemoryUser
	valid ValidationStrategy
}

```

这段代码定义了两个函数，函数1名为 `NewServerSpec`，函数2名为 `NewServerSpecFromPB`。

函数1 `NewServerSpec` 接收一个 `dest` 网络设备，一个 `validationStrategy` 验证策略，以及一个或多个 `内存User` 类型的参数。函数返回一个指向 `ServerSpec` 类型的引用。

函数2 `NewServerSpecFromPB` 接收一个 `内存ServerEndpoint` 类型的参数。函数首先将 `dest` 网络设备创建为 `net.TCPDestination` 类型，然后设置其 IP 地址为传入的 `address` 和端口为传入的 `port`。接下来，函数遍历传入的 `user` 参数，将其转换为 `内存User` 类型并将其添加到 `mUsers` 数组中。最后，函数调用 `NewServerSpec` 函数并传递其 `dest`、`validationStrategy` 和 `mUsers` 参数，返回一个指向 `ServerSpec` 类型的引用并将其赋值给传入的 `内存ServerEndpoint` 参数，避免错误。


```go
func NewServerSpec(dest net.Destination, valid ValidationStrategy, users ...*MemoryUser) *ServerSpec {
	return &ServerSpec{
		dest:  dest,
		users: users,
		valid: valid,
	}
}

func NewServerSpecFromPB(spec *ServerEndpoint) (*ServerSpec, error) {
	dest := net.TCPDestination(spec.Address.AsAddress(), net.Port(spec.Port))
	mUsers := make([]*MemoryUser, len(spec.User))
	for idx, u := range spec.User {
		mUser, err := u.ToMemoryUser()
		if err != nil {
			return nil, err
		}
		mUsers[idx] = mUser
	}
	return NewServerSpec(dest, AlwaysValid(), mUsers...), nil
}

```

这两段代码定义了两个名为"func"的函数，接收一个名为"ServerSpec"的类型为"*ServerSpec"的参数，并返回一个名为"net.Destination"的类型为"net.Destination"的变量。函数内部使用了"net"包中的"Destination"类型。

这两段代码还定义了一个名为"func"的函数，接收一个名为"ServerSpec"的类型为"*ServerSpec"的参数，并返回一个名为"bool"的类型。函数内部使用了"内存User"类型和"User"类型的嵌套遍历。具体来说，函数首先尝试获取传入的"User"类型的对象，如果当前对象与传入的"User"类型的对象的"Account"字段相等，则返回真，否则返回假。最后，函数使用了"RLock"和"RUnlock"类型的"内存User"对象的同步操作，确保在函数内部对"内存User"类型的对象进行任何修改操作时，都先获得足够的同步。


```go
func (s *ServerSpec) Destination() net.Destination {
	return s.dest
}

func (s *ServerSpec) HasUser(user *MemoryUser) bool {
	s.RLock()
	defer s.RUnlock()

	for _, u := range s.users {
		if u.Account.Equals(user.Account) {
			return true
		}
	}
	return false
}

```

这两函数一起组成了一个库中的部分功能，具体解释如下：

1. `func (s *ServerSpec) AddUser(user *MemoryUser)` 是一个名为 AddUser 的函数，接受一个名为 user 的 *MemoryUser 类型的参数。

2. `func (s *ServerSpec) PickUser() *MemoryUser` 是一个名为 PickUser 的函数，接受一个指向内存中的 ServerSpec 类型的参数 s。

AddUser 函数的作用是在 ServerSpec 中添加一个新的用户。它首先检查传入的用户是否已经存在于 ServerSpec 中，如果是，则直接返回。否则，它会尝试锁定服务器，并确保在解锁服务器之后，将新的用户添加到 ServerSpec 的 users 数组中。

PickUser 函数的作用是从 ServerSpec 的 users 数组中选择一个用户，并返回该用户的 *MemoryUser 类型。它使用RLock 函数来获取对 users 数组的只读访问权，并使用 dice.Roll 函数在 users 数组中随机选择一个元素。如果 users 数局为空，它返回 nil，否则它返回 users 数组中的第一个用户。


```go
func (s *ServerSpec) AddUser(user *MemoryUser) {
	if s.HasUser(user) {
		return
	}

	s.Lock()
	defer s.Unlock()

	s.users = append(s.users, user)
}

func (s *ServerSpec) PickUser() *MemoryUser {
	s.RLock()
	defer s.RUnlock()

	userCount := len(s.users)
	switch userCount {
	case 0:
		return nil
	case 1:
		return s.users[0]
	default:
		return s.users[dice.Roll(userCount)]
	}
}

```

这两函数的作用是管理一个ServerSpec结构体中的valid成员的合法性检查和无效性通知。

函数1：`IsValid()` 的作用是返回`s.valid`结构体的valid字段是否有效。如果`s.valid`的valid字段的值为`true`，则返回`true`，否则返回`false`。

函数2：`Invalidate()` 的作用是通知`s.valid`结构体中的valid字段无效。


```go
func (s *ServerSpec) IsValid() bool {
	return s.valid.IsValid()
}

func (s *ServerSpec) Invalidate() {
	s.valid.Invalidate()
}

```

# `common/protocol/server_spec.pb.go`

这段代码定义了一个名为"protocol"的包，其中包含用于定义Go协议的代码。具体来说，它实现了两个相关的Go语言库：protoc-gen-go和protoc。

首先，它引入了来自github.com/golang/protobuf/proto的protobuf类型定义，以及google.golang.org/protobuf/reflect/protoreflect的reflect类型定义。这些定义用于定义Go类型和接口的相应结构和类型。

然后，它导入了net包，因为Go语言中的网络编程需要使用网络接口，它通过使用net包中定义的网络相关接口来与外界通信。

最后，它定义了一些常见的类型，如sync.Tqueue和net.Tcp，用于实现并发编程和网络通信的功能。

整个package协议定义了用于定义Go协议的基本结构和类型，以及实现网络通信的功能，可以被用于编写具有相关接口的Go程序。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: common/protocol/server_spec.proto

package protocol

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	net "v2ray.com/core/common/net"
)

```

这段代码的作用是验证名为“protoimpl”的包是否足够更新。具体来说，它包括两个部分：

1. 使用 protoimpl.EnforceVersion 函数，验证“protoimpl”包的版本是否与当前最低版本兼容。如果兼容，则返回 true；如果不兼容，则返回 false。
2. 使用同样的函数，验证“runtime/protoimpl”包的版本是否与当前最高版本兼容。如果兼容，则返回 true；如果不兼容，则返回 false。

然后，它创建了一个名为“_”的常量，用于检查生成的代码是否足够更新。接下来，定义了一个名为“ServerEndpoint”的结构体，其中包含一个名为“address”的 IPv4 类型字段，用于表示服务器端点的地址，以及一个名为“port”的 Uint32 类型字段，表示服务器端点的端口。最后，定义了一个名为“unknownFields”的隐藏字段，用于存储可能需要在运行时验证的未知数据。


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

type ServerEndpoint struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Address *net.IPOrDomain `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	Port    uint32          `protobuf:"varint,2,opt,name=port,proto3" json:"port,omitempty"`
	User    []*User         `protobuf:"bytes,3,rep,name=user,proto3" json:"user,omitempty"`
}

```

这段代码定义了两个函数：

1. `func (x *ServerEndpoint) Reset()` 函数的作用是重置 `x` 的值，将 `*x` 赋值为 `ServerEndpoint{}`。然后检查 `protoimpl.UnsafeEnabled` 是否为 `true`，如果是，那么执行以下操作：

  - 创建一个指向 `ServerEndpoint` 类型的 `mi` 变量。
  - 创建一个指向 `x` 类型实例的 `ms` 变量。
  - 将 `mi` 类型中的 `*x` 存储到 `ms` 类型的 `MessageInfo` 字段中。

2. `func (x *ServerEndpoint) String()` 函数的作用是将 `x` 转换为字符串，并返回结果。

3. `func (x *ServerEndpoint) ProtoMessage()` 函数的作用是返回 `x` 的 `ProtoMessage` 类型，这个函数在定义时不会被调用，但是需要在使用时进行调用，否则不会输出任何信息。


```go
func (x *ServerEndpoint) Reset() {
	*x = ServerEndpoint{}
	if protoimpl.UnsafeEnabled {
		mi := &file_common_protocol_server_spec_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *ServerEndpoint) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*ServerEndpoint) ProtoMessage() {}

```

这段代码定义了一个名为func的函数，接收一个名为x的参数，并返回一个名为protoreflect.Message类型的变量。函数的作用是在函数内部根据传入的参数x，尝试使用或创建一个ServerEndpoint类型的对象，然后使用该对象的相关方法来获取或设置Protobuf消息类型的元数据。

具体来说，函数的实现可以分为以下几个步骤：

1. 获取文件中protoc编译器生成的.proto文件中定义的.proto消息类型，即file_common_protocol_server_spec_proto_msgTypes[0]。

2. 如果函数的输入参数x存在，并且函数使用了.UnsafeEnabled参数，那么尝试使用x指向的对象的MessageStateOf方法获取消息类型元数据。

3. 如果.UnsafeEnabled参数不启用，那么直接使用.MessageOf方法获取消息类型元数据。

4. 如果尝试获取消息类型元数据时出现错误，那么将错误的消息类型元数据存储到x指向的对象的MessageStateOf方法的内部位置。

5. 返回.MessageOf方法获取的消息类型元数据。

6. 函数还实现了一个名为descriptor的函数，接收一个ServerEndpoint类型的对象，返回该对象的描述信息。函数的实现类似于使用.FileCommonProtocolServerSpecRawDescGZIP和.MessageOf方法获取或设置消息类型元数据。


```go
func (x *ServerEndpoint) ProtoReflect() protoreflect.Message {
	mi := &file_common_protocol_server_spec_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use ServerEndpoint.ProtoReflect.Descriptor instead.
func (*ServerEndpoint) Descriptor() ([]byte, []int) {
	return file_common_protocol_server_spec_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了一个名为`ServerEndpoint`的类型，它包含三个方法：`GetAddress`、`GetPort`和`GetUser`。

`func (x *ServerEndpoint) GetAddress() *net.IPOrDomain {
	if x != nil {
		return x.Address
	}
	return nil
}`函数接收一个`ServerEndpoint`类型的参数`x`，并返回`x`的`Address`字段。如果`x`不等于`nil`，则返回`x`的`Address`；如果`x`为`nil`，则返回`nil`。

`func (x *ServerEndpoint) GetPort() uint32 {
	if x != nil {
		return x.Port
	}
	return 0
}`函数与上面那个函数类似，但它接收一个`ServerEndpoint`类型的参数`x`，并返回`x`的`Port`字段。如果`x`不等于`nil`，则返回`x`的`Port`；如果`x`为`nil`，则返回`0`。

`func (x *ServerEndpoint) GetUser() []*User {
	if x != nil {
		return x.User
	}
	return nil
}`函数接收一个`ServerEndpoint`类型的参数`x`，并返回`x`的`User`字段。如果`x`不等于`nil`，则返回`x`的`User`；如果`x`为`nil`，则返回`nil`。


```go
func (x *ServerEndpoint) GetAddress() *net.IPOrDomain {
	if x != nil {
		return x.Address
	}
	return nil
}

func (x *ServerEndpoint) GetPort() uint32 {
	if x != nil {
		return x.Port
	}
	return 0
}

func (x *ServerEndpoint) GetUser() []*User {
	if x != nil {
		return x.User
	}
	return nil
}

```

It appears that the given data is a piece of assembly code, which is a齐凯蒂编写的一种二进制 file format used for symbolic debugging and other purposes.

The first line starts with a sequence of 15十六进制 numbers, which are likely展开成 0x6d, 0x6f, 0x6e, and so on. The next few lines follow a similar pattern, with a sequence of alternating十六进制 numbers followed by a sequence of ASCII characters. The pattern seems to be alternating between code-oriented and data-oriented sections.

The last several lines appear to be the actual machine code for some program. The first among them is 0x61, followed by several lines of 0x72 and 0x6f, which are then followed by the last 4 lines of the machine code.

It is often used for debugging programs that use mixed addresses or have non-standard memory representations.



```go
var File_common_protocol_server_spec_proto protoreflect.FileDescriptor

var file_common_protocol_server_spec_proto_rawDesc = []byte{
	0x0a, 0x21, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
	0x6c, 0x2f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x5f, 0x73, 0x70, 0x65, 0x63, 0x2e, 0x70, 0x72,
	0x6f, 0x74, 0x6f, 0x12, 0x1a, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
	0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x1a,
	0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x61, 0x64, 0x64, 0x72,
	0x65, 0x73, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x1a, 0x63, 0x6f, 0x6d, 0x6d, 0x6f,
	0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2f, 0x75, 0x73, 0x65, 0x72, 0x2e,
	0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x97, 0x01, 0x0a, 0x0e, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72,
	0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x12, 0x3b, 0x0a, 0x07, 0x61, 0x64, 0x64, 0x72,
	0x65, 0x73, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61,
	0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
	0x74, 0x2e, 0x49, 0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x07, 0x61, 0x64,
	0x64, 0x72, 0x65, 0x73, 0x73, 0x12, 0x12, 0x0a, 0x04, 0x70, 0x6f, 0x72, 0x74, 0x18, 0x02, 0x20,
	0x01, 0x28, 0x0d, 0x52, 0x04, 0x70, 0x6f, 0x72, 0x74, 0x12, 0x34, 0x0a, 0x04, 0x75, 0x73, 0x65,
	0x72, 0x18, 0x03, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
	0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74,
	0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65, 0x72, 0x42,
	0x5f, 0x0a, 0x1e, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72,
	0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f,
	0x6c, 0x50, 0x01, 0x5a, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63,
	0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
	0x63, 0x6f, 0x6c, 0xaa, 0x02, 0x1a, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65,
	0x2e, 0x43, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x50, 0x72, 0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c,
	0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

这段代码定义了一个名为file_common_protocol_server_spec_proto_rawDescOnce的变量，该变量保证在每次调用file_common_protocol_server_spec_proto_rawDescGZIP函数时都会被初始化一次。

file_common_protocol_server_spec_proto_rawDescOnce.Do函数是干函数，即函数内部执行的操作都是被返回的，而不是被直接实现的。该函数的作用是将传入的file_common_protocol_server_spec_proto_rawDescData进行GZIP压缩，并将压缩后的结果返回。

file_common_protocol_server_spec_proto_msgTypes定义了一个接收函数file_common_protocol_server_spec_proto_rawDescGZIP的messageInfo类型。这个类型告诉编译器如何正确地解析接收到的messageInfo的值。

file_common_protocol_server_spec_proto_goTypes定义了一个包含三个接收函数的类型，这些函数的接收者都是void类型，即没有返回值。这些函数分别对应于file_common_protocol_server_spec_proto_rawDescGZIP函数的输入和输出参数。

file_common_protocol_server_spec_proto_rawDescGZIP函数的作用是压缩file_common_protocol_server_spec_proto_rawDescData，并返回压缩后的结果。它使用了protoimpl.X.CompressGZIP函数来完成这个任务，这个函数将接收到的file_common_protocol_server_spec_proto_rawDescData进行GZIP压缩，并返回压缩后的结果。


```go
var (
	file_common_protocol_server_spec_proto_rawDescOnce sync.Once
	file_common_protocol_server_spec_proto_rawDescData = file_common_protocol_server_spec_proto_rawDesc
)

func file_common_protocol_server_spec_proto_rawDescGZIP() []byte {
	file_common_protocol_server_spec_proto_rawDescOnce.Do(func() {
		file_common_protocol_server_spec_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_protocol_server_spec_proto_rawDescData)
	})
	return file_common_protocol_server_spec_proto_rawDescData
}

var file_common_protocol_server_spec_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_protocol_server_spec_proto_goTypes = []interface{}{
	(*ServerEndpoint)(nil), // 0: v2ray.core.common.protocol.ServerEndpoint
	(*net.IPOrDomain)(nil), // 1: v2ray.core.common.net.IPOrDomain
	(*User)(nil),           // 2: v2ray.core.common.protocol.User
}
```

This is a Python implementation of the file\_common\_protocol\_server\_spec\_proto file. It includes the initialization of the proto file, as well as the generated types for the initialized types.

The file starts with an if statement that checks if the file\_common\_protocol\_server\_spec\_proto file has already been initialized. If it has not been initialized, it proceeds with the file\_common\_protocol\_user\_proto\_init() function, which initializes the file\_common\_protocol\_user\_proto message type.

If the file has already been initialized, it checks that the initialized types, including the generated types for the initialized types, are being used. If they are not being used, it sets the generated types for the initialized types to nil. It also sets the generated description for the file to indicate that it has one message type and one extension.

It then generates the initialized file by calling the protoimpl.TypeBuilder's build() method, passing in the generated description and the generated types.


```go
var file_common_protocol_server_spec_proto_depIdxs = []int32{
	1, // 0: v2ray.core.common.protocol.ServerEndpoint.address:type_name -> v2ray.core.common.net.IPOrDomain
	2, // 1: v2ray.core.common.protocol.ServerEndpoint.user:type_name -> v2ray.core.common.protocol.User
	2, // [2:2] is the sub-list for method output_type
	2, // [2:2] is the sub-list for method input_type
	2, // [2:2] is the sub-list for extension type_name
	2, // [2:2] is the sub-list for extension extendee
	0, // [0:2] is the sub-list for field type_name
}

func init() { file_common_protocol_server_spec_proto_init() }
func file_common_protocol_server_spec_proto_init() {
	if File_common_protocol_server_spec_proto != nil {
		return
	}
	file_common_protocol_user_proto_init()
	if !protoimpl.UnsafeEnabled {
		file_common_protocol_server_spec_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*ServerEndpoint); i {
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
			RawDescriptor: file_common_protocol_server_spec_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_common_protocol_server_spec_proto_goTypes,
		DependencyIndexes: file_common_protocol_server_spec_proto_depIdxs,
		MessageInfos:      file_common_protocol_server_spec_proto_msgTypes,
	}.Build()
	File_common_protocol_server_spec_proto = out.File
	file_common_protocol_server_spec_proto_rawDesc = nil
	file_common_protocol_server_spec_proto_goTypes = nil
	file_common_protocol_server_spec_proto_depIdxs = nil
}

```