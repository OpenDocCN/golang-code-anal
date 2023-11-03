# v2ray-core源码解析 3

# `app/dispatcher/errors.generated.go`

这段代码定义了一个名为 dispatcher 的包，其中包含一个名为 errPathObjHolder 的类型，以及一个名为 newError 的函数。

newError 函数接收一个或多个参数，这些参数以逗号分隔。函数在内部创建一个名为 errPathObjHolder 的类型，该类型使用了一个匿名结构体，其中包含一个空字符串成员。

接着，函数使用 errors.New 函数创建一个名为 errors 的错误对象，并使用 WithPathObj 方法将其与 errPathObjHolder 类型结合。errPathObjHolder 类型包含一个空字符串，这是表示错误路径的基准点。通过将错误对象与 errPathObjHolder 类型的空字符串相结合，可以确保错误对象包含一个错误路径，以便应用程序能够更好地处理和调试错误。

最后，函数返回错误对象，以便应用程序能够处理错误。


```go
package dispatcher

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `app/dispatcher/sniffer.go`

这段代码是一个 Go 语言中的 struct 类型，定义了一个名为 SniffResult 的接口。这个接口容许在函数内部使用，但是不能在函数外部直接使用。它定义了两个字段：Protocol 和 Domain，分别表示捕获到的协议类型和域名。

具体来说，这段代码的作用是：

1. 通过 `build` 工具构建 Go 语言项目。
2. 通过 `confonly` 选项，禁止在构建过程中输出任何内容。
3. 定义了一个名为 SniffResult 的接口，用于描述从目标主机和端口捕获到的信息。
4. 在 `dispatcher` 包中定义了如何使用 SniffResult 接口来处理从客户端发送的数据。
5. 通过 `v2ray.com/core/common/protocol/bittorrent`、`v2ray.com/core/common/protocol/http` 和 `v2ray.com/core/common/protocol/tls` 包，实现了与目标主机和端口通信并捕获数据的功能。


```go
// +build !confonly

package dispatcher

import (
	"v2ray.com/core/common"
	"v2ray.com/core/common/protocol/bittorrent"
	"v2ray.com/core/common/protocol/http"
	"v2ray.com/core/common/protocol/tls"
)

type SniffResult interface {
	Protocol() string
	Domain() string
}

```

该代码定义了一个名为`protocolSniffer`的函数类型，它接受一个字节切片`[]byte`作为参数，并返回一个`SniffResult`和一个错误类型`error`。

该代码还定义了一个名为`Sniffer`的结构体，该结构体包含一个名为`sniffer`的切片，该切片包含多个名为`protocolSniffer`的函数指针。

该代码还定义了一个名为`NewSniffer`的函数，该函数返回一个`Sniffer`实例，该实例初始化时包含多个名为`protocolSniffer`的函数指针，这些函数指针分别对应于`http.SniffHTTP`、`tls.SniffTLS`和`bittorrent.SniffBittorrent`函数。


```go
type protocolSniffer func([]byte) (SniffResult, error)

type Sniffer struct {
	sniffer []protocolSniffer
}

func NewSniffer() *Sniffer {
	return &Sniffer{
		sniffer: []protocolSniffer{
			func(b []byte) (SniffResult, error) { return http.SniffHTTP(b) },
			func(b []byte) (SniffResult, error) { return tls.SniffTLS(b) },
			func(b []byte) (SniffResult, error) { return bittorrent.SniffBittorrent(b) },
		},
	}
}

```

这段代码定义了一个名为 `Sniffer` 的接口 `Sniffer`，以及一个名为 `errUnknownContent` 的错误 `Error`。

`Sniffer` 接口表示一个函数，接收一个字节数组 `payload`，并返回一个 `SniffResult` 类型的数据，它是通过调用传递给 `Sniffer` 的另一个接口 `Sniffer` 函数得到的，如果失败则返回 `Error`。

`errUnknownContent` 错误在 `Sniffer` 函数内部被创建，它输出一个字符串 "unknown content"。

整个函数的作用是：在 `Sniffer` 函数中处理可能返回的 `SniffResult` 和可能产生的错误。

对于 `Sniffer` 函数，首先检查 `sniffer` 数组是否为空，如果是，则清空 `pendingSniffer` 数组，然后检查 `sniffer` 数组中的每个 `Sniffer` 函数是否失败并返回了错误。如果是成功完成的，将 `Sniffer` 函数的结果存储在 `pendingSniffer` 数组中，并跳过该 `Sniffer` 函数。

接下来，遍历 `pendingSniffer` 数组，检查是否有一个元素成功完成了 `Sniffer` 函数并返回了结果。如果是，那么将 `Sniffer` 函数的结果存储在 `pendingSniffer` 数组中，跳过该 `Sniffer` 函数，并返回 `NoContent` 和一个错误 `Error`。

如果遍历完 `pendingSniffer` 数组都没有找到成功完成的 `Sniffer` 函数，那么执行 `errUnknownContent` 错误，它将输出 "unknown content"。


```go
var errUnknownContent = newError("unknown content")

func (s *Sniffer) Sniff(payload []byte) (SniffResult, error) {
	var pendingSniffer []protocolSniffer
	for _, s := range s.sniffer {
		result, err := s(payload)
		if err == common.ErrNoClue {
			pendingSniffer = append(pendingSniffer, s)
			continue
		}

		if err == nil && result != nil {
			return result, nil
		}
	}

	if len(pendingSniffer) > 0 {
		s.sniffer = pendingSniffer
		return nil, common.ErrNoClue
	}

	return nil, errUnknownContent
}

```

# `app/dispatcher/stats.go`

这段代码定义了一个名为SizeStatWriter的结构体，其中包含一个计数器和一个写入缓冲区的字段。

SizeStatWriter的作用是创建一个用于记录并发连接数量规模的统计器，并将该统计器的计数器和写入缓冲区一起设置。具体来说，它会在v2ray.com/core/中的一个名为dispatcher的包中进行实现，通过使用go build命令进行构建，并在构建时将以下-confonly选项添加到源代码中，以避免在生产环境中输出调试信息。

通过SizeStatWriter，可以方便地收集和统计并发连接的数量，并将其存储在一个缓冲区中。这种统计器可以帮助开发人员了解他们的应用程序在处理并发连接方面的性能和瓶颈。


```go
// +build !confonly

package dispatcher

import (
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/features/stats"
)

type SizeStatWriter struct {
	Counter stats.Counter
	Writer  buf.Writer
}

```

该代码实现了一个名为SizeStatWriter的函数式接口，它用于在控制台上打印缓冲区中的多个字节，并确保写入操作正确以确保缓冲区不会溢出。

具体来说，以下是代码中定义的三个函数：

1. `func (w *SizeStatWriter) WriteMultiBuffer(mb buf.MultiBuffer) error`：该函数接收一个名为mb的缓冲区作为参数，然后将其写入w的写入缓冲区中。函数返回一个error类型的变量，其中包含一个描述写入操作是否成功的信息。如果写入操作成功，则返回 nil；如果写入操作失败，则返回一个相应的错误。

2. `func (w *SizeStatWriter) Close() error`：该函数接收一个名为w的SizeStatWriter对象作为参数，然后关闭它。函数返回一个error类型的变量，其中包含一个描述关闭操作是否成功的信息。如果关闭操作成功，则返回 nil；如果关闭操作失败，则返回一个相应的错误。

3. `func (w *SizeStatWriter) Interrupt()`：该函数是一个通用的断开函数，用于通知SizeStatWriter的写入操作已经结束，应该停止写入并从断开状态恢复。函数没有任何实际操作，因为它只是将w的状态设置为阻塞，而不是向函数提供任何新的信息或知识。


```go
func (w *SizeStatWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	w.Counter.Add(int64(mb.Len()))
	return w.Writer.WriteMultiBuffer(mb)
}

func (w *SizeStatWriter) Close() error {
	return common.Close(w.Writer)
}

func (w *SizeStatWriter) Interrupt() {
	common.Interrupt(w.Writer)
}

```

# `app/dispatcher/stats_test.go`

这段代码是一个名为 "dispatcher_test" 的包，用于测试 "dispatcher" 组件的行为。

它导入了两个包： "v2ray.com/core/app/dispatcher" 和 "v2ray.com/core/common"，其中 "dispatcher" 包中包含一些与测试相关的函数和变量。

接下来，定义了一个名为 "TestCounter" 的类型，它是一个 "int64" 类型的变量。

在 "dispatcher" 包中的 "main.go" 函数中，创建了一个 "dispatcher" 实例并将其指定了 "TestCounter" 类型的 "c" 变量，这样 "c" 变量就可以访问 "dispatcher" 实例中的某些实现了 "TestCounter" 类型的函数和变量了。

最后，通过 "c.Value" 函数获取并返回 "TestCounter" 类型实例的 "Value" 函数的返回值，这样就可以测试 "dispatcher" 包中实现 "TestCounter" 类型实例的行为了。


```go
package dispatcher_test

import (
	"testing"

	. "v2ray.com/core/app/dispatcher"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
)

type TestCounter int64

func (c *TestCounter) Value() int64 {
	return int64(*c)
}

```

这段代码定义了两个函数，一个是用来增加一个整型变量v，另一个是用来设置整型变量v的值。这两个函数的实现都在函数内部进行。

函数内部首先将传入的整型变量v与传入的整型变量*c相加，然后将结果存储回原来的整型变量*c。这样做是为了确保对原始的整型变量进行操作，而不是修改它。

函数内部还有一行代码，用于将上面得到的整型变量v的值存储回传入的整型变量*c。

这两个函数的实现都在同一个函数中，因此它们的参数都是从同一个位置获取的。


```go
func (c *TestCounter) Add(v int64) int64 {
	x := int64(*c) + v
	*c = TestCounter(x)
	return x
}

func (c *TestCounter) Set(v int64) int64 {
	*c = TestCounter(v)
	return v
}

func TestStatsWriter(t *testing.T) {
	var c TestCounter
	writer := &SizeStatWriter{
		Counter: &c,
		Writer:  buf.Discard,
	}

	mb := buf.MergeBytes(nil, []byte("abcd"))
	common.Must(writer.WriteMultiBuffer(mb))

	mb = buf.MergeBytes(nil, []byte("efg"))
	common.Must(writer.WriteMultiBuffer(mb))

	if c.Value() != 7 {
		t.Fatal("unexpected counter value. want 7, but got ", c.Value())
	}
}

```

# `app/dns/config.pb.go`

这段代码定义了一个名为 "dns" 的包，其中包括了配置相关的内容。具体来说，它导入了以下依赖：

- protoc-gen-go：一个用于生成 Go 代码的 Protocol Buffer 兼容的工具，用于将 Go 代码编译成 protoc 语言的规范。
- protoc：一个用于生成 Protocol Buffer 规范的工具，它能够将 Go 代码编译成字节码，然后在编译生成的文件中定义了各种类型、接口、常量等，用于定义数据结构和代码结构。
- source：一个无法访问的文件，可能是某个空的源代码文件或者是由于缺少依赖文件而无法生成代码的文件。

另外，它还导入了以下 packages:

- app/dns/config：定义了 DNS 配置的类型和字段，以及将 Go 代码编译成 config.proto 规范的工具。
- router：定义了路由相关的类型和字段，以及将 Go 代码编译成 router.proto 规范的工具。
- net：定义了网络相关的类型和字段，以及将 Go 代码编译成 net.proto 规范的工具。
- synchronization：定义了同步相关的类型和字段，以及与并发和锁相关的方法。

最后，还定义了一些常量和函数。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: app/dns/config.proto

package dns

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	router "v2ray.com/core/app/router"
	net "v2ray.com/core/common/net"
)

```

这段代码是一个 Rust 的 const 变量，它包含两个检验代码是否足够兼容的函数，然后定义了一个 `DomainMatchingType` 类型。最后，它输出一个布尔值，表示该代码是否使用了足够兼容的包来编译。

具体来说，第一个函数 `_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)` 检查当前使用的 `protoimpl` 包是否包含了足够兼容的 `proto` 定义。如果 `protoimpl` 包的版本低于 `protoimpl.MinVersion`，函数将强制使用最新版本的 `protoimpl` 包。

第二个函数 `_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)` 检查当前使用的 `protoimpl` 包是否包含了足够兼容的 `proto` 定义。如果 `protoimpl` 包的版本高于 `protoimpl.MaxVersion-20`，函数将强制使用最新版本的 `protoimpl` 包。

然后定义了一个 `DomainMatchingType` 类型，它定义了 `DomainMatchingType` 的不同枚举类型。最后，它使用 `proto.PyroEnabled` 函数来检查当前使用的 `proto` 定义是否足够兼容，如果足够兼容，则输出 `true`，否则输出 `false`。


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

type DomainMatchingType int32

const (
	DomainMatchingType_Full      DomainMatchingType = 0
	DomainMatchingType_Subdomain DomainMatchingType = 1
	DomainMatchingType_Keyword   DomainMatchingType = 2
	DomainMatchingType_Regex     DomainMatchingType = 3
)

```

这段代码定义了一个枚举类型 DomainMatchingType，它有四个枚举值：Full、Subdomain、Keyword 和 Regex。每个枚举值都对应一个整型变量 DomainMatchingType_value，该变量类型为 int32 表示 32 位整数。此外，还定义了一个名为 DomainMatchingType_name 的字符串类型变量，它包含了每个枚举值的名称。


```go
// Enum value maps for DomainMatchingType.
var (
	DomainMatchingType_name = map[int32]string{
		0: "Full",
		1: "Subdomain",
		2: "Keyword",
		3: "Regex",
	}
	DomainMatchingType_value = map[string]int32{
		"Full":      0,
		"Subdomain": 1,
		"Keyword":   2,
		"Regex":     3,
	}
)

```

该代码定义了一个名为“DomainMatchingType”的枚举类型。该枚举类型有一个名为“x”的成员变量，其类型被定义为“DomainMatchingType”。

此外，该代码还定义了一个名为“Enum”的函数，该函数返回一个指向“DomainMatchingType”类型的指针。

接下来，该代码定义了一个名为“String”的函数，该函数将“DomainMatchingType”类型的成员变量“x”包装起来，然后使用“protoimpl.X.EnumStringOf”接口将该成员变量的前缀“x”与数字“1”组合，并将结果存储在“x”类型的指针中，然后返回这个指针。

最后，该代码定义了一个名为“Descriptor”的函数，该函数返回一个指向“DomainMatchingType”类型的描述符，使用“file_app_dns_config_proto_enumTypes[0].Descriptor()”接口将类型“DomainMatchingType”包装起来。

该代码还定义了一个名为“Type”的函数，该函数返回一个指向“DomainMatchingType”类型的枚举类型，使用“file_app_dns_config_proto_enumTypes[0]”接口将类型“DomainMatchingType”包装起来。


```go
func (x DomainMatchingType) Enum() *DomainMatchingType {
	p := new(DomainMatchingType)
	*p = x
	return p
}

func (x DomainMatchingType) String() string {
	return protoimpl.X.EnumStringOf(x.Descriptor(), protoreflect.EnumNumber(x))
}

func (DomainMatchingType) Descriptor() protoreflect.EnumDescriptor {
	return file_app_dns_config_proto_enumTypes[0].Descriptor()
}

func (DomainMatchingType) Type() protoreflect.EnumType {
	return &file_app_dns_config_proto_enumTypes[0]
}

```

这段代码定义了两个函数，函数一是将一个名为 `x` 的 `DomainMatchingType` 类型的参数返回一个 `protoreflect.EnumNumber` 类型的值，函数二是返回一个 `DomainMatchingType` 类型的 `enumDescriptor` 字段，它的值是一个二进制字节数组和两个整数的组合。

函数一的作用是返回一个与 `DomainMatchingType` 相关的 `protoreflect.EnumNumber` 类型的值，这个值使用 `x` 参数作为参数进行传递，然后使用 `protoreflect.EnumNumber` 的方法进行返回。

函数二的作用是返回一个 `DomainMatchingType` 类型的 `enumDescriptor` 字段，它包含一个二进制字节数组，该字节数组包含了一个 `NameServer` 类型的实例，然后使用了 `json:` 标签将这个实例的 `address`、`prioritized_domain` 和 `geoip` 字段映射到了 JSON 字段中。


```go
func (x DomainMatchingType) Number() protoreflect.EnumNumber {
	return protoreflect.EnumNumber(x)
}

// Deprecated: Use DomainMatchingType.Descriptor instead.
func (DomainMatchingType) EnumDescriptor() ([]byte, []int) {
	return file_app_dns_config_proto_rawDescGZIP(), []int{0}
}

type NameServer struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Address           *net.Endpoint                `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	PrioritizedDomain []*NameServer_PriorityDomain `protobuf:"bytes,2,rep,name=prioritized_domain,json=prioritizedDomain,proto3" json:"prioritized_domain,omitempty"`
	Geoip             []*router.GeoIP              `protobuf:"bytes,3,rep,name=geoip,proto3" json:"geoip,omitempty"`
	OriginalRules     []*NameServer_OriginalRule   `protobuf:"bytes,4,rep,name=original_rules,json=originalRules,proto3" json:"original_rules,omitempty"`
}

```

这段代码定义了两个函数：

1. `func (x *NameServer) Reset()`：该函数的作用是重置 `x` 的值，具体实现如下：

 - 创建一个 `NameServer` 类型的 `x` 并将其赋值为空字符串 `""`。
 - 如果 `protoimpl.UnsafeEnabled` 是 `true`，则执行以下操作：
   - 创建一个 `file_app_dns_config_proto_msgTypes` 类型的 `mi` 变量。
   - 创建一个 `protoimpl.X` 类型的 `ms` 变量，并将 `mi` 存储为 `ms` 的值。
   - 将 `ms` 存储为 `x` 的指针变量 `x`。

2. `func (x *NameServer) String()`：该函数的作用是返回 `x` 的字符串表示，具体实现如下：

 - 根据 `protoimpl.X` 类型中的 `String` 函数，创建一个字符串 `s`。
 - 由于 `x` 是一个 `NameServer` 类型，因此将 `x` 的 `String` 类型设置为 `file_app_dns_config_proto_String`。

这两个函数都在尝试通过 `x` 实现 `file_app_dns_config_proto` 中的 `NameServer` 类型，从而实现字符串操作。


```go
func (x *NameServer) Reset() {
	*x = NameServer{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *NameServer) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*NameServer) ProtoMessage() {}

```

这段代码定义了一个名为“func”的函数，接收一个名为“x”的参数，并返回一个名为“protoreflect.Message”的类型。

函数的作用是：

1. 如果传入的参数“x”不为空，则执行以下操作：
a. 获取文件“file_app_dns_config_proto_msgTypes”中对应类型“file_app_dns_config_proto_msgTypes.NameServer”的指针（也就是一个指向“file_app_dns_config_proto_msgTypes.NameServer”类型实例的指针）。
b. 如果使用了“file_app_dns_config_proto_unsafe_enabled”参数，则执行以下操作：
   i. 获取“file_app_dns_config_proto_x”类型的实例。
   ii. 如果“file_app_dns_config_proto_x”不为空，执行以下操作：
       a. 获取“file_app_dns_config_proto_msgTypes”中对应类型“file_app_dns_config_proto_msgTypes.NameServer”的指针（也就是一个指向“file_app_dns_config_proto_msgTypes.NameServer”类型实例的指针）。
       b. 如果“file_app_dns_config_proto_x”的“MessageStateOf”函数返回的是一个有效的“file_app_dns_config_proto_msgTypes.NameServer”实例，则执行以下操作：
           a. 调用“MessageInfo”的“LoadMessageInfo”函数，获取到“file_app_dns_config_proto_x”中的消息信息。
           b. 如果消息信息为空，则执行以下操作：
               i. 将“file_app_dns_config_proto_x”中的消息信息设置为当前“file_app_dns_config_proto_msgTypes.NameServer”类型实例的“MessageOf”函数返回的“file_app_dns_config_proto_msgTypes.NameServer”实例。
               ii. 调用“MessageInfo”的“StoreMessageInfo”函数，将消息信息保存到“file_app_dns_config_proto_msgTypes.NameServer”实例中。
           c. 返回“file_app_dns_config_proto_x”中的消息信息。
           d. 如果“file_app_dns_config_proto_x”的“MessageOf”函数返回的“file_app_dns_config_proto_msgTypes.NameServer”实例为空，则直接返回“file_app_dns_config_proto_msgTypes.NameServer”。
2. 如果传入的参数“x”为空，则执行以下操作：
a. 返回“file_app_dns_config_proto_rawDescGZIP”和空列表“[]int”。




```go
func (x *NameServer) ProtoReflect() protoreflect.Message {
	mi := &file_app_dns_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use NameServer.ProtoReflect.Descriptor instead.
func (*NameServer) Descriptor() ([]byte, []int) {
	return file_app_dns_config_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了三个名为`func`的函数，它们接收一个名为`x`的`NameServer`类型的参数并返回相应的结果。

第一个函数`GetAddress() *net.Endpoint`接收一个`NameServer`类型的参数，并返回一个`net.Endpoint`类型的引用。如果参数`x`不为空，则返回`x`的`Address`字段；否则，返回一个空的`Endpoint`结构体。

第二个函数`GetPrioritizedDomain() []*NameServer_PriorityDomain`接收一个`NameServer`类型的参数，并返回一个包含`NameServer_PriorityDomain`类型的数组。如果参数`x`不为空，则返回`x`的`PrioritizedDomain`字段；否则，返回一个空的`PriorityDomain`数组。

第三个函数`GetGeoip() []*router.GeoIP`接收一个`NameServer`类型的参数，并返回一个包含`router.GeoIP`类型的数组。如果参数`x`不为空，则返回`x`的`Geoip`字段；否则，返回一个空的`GeoIP`数组。


```go
func (x *NameServer) GetAddress() *net.Endpoint {
	if x != nil {
		return x.Address
	}
	return nil
}

func (x *NameServer) GetPrioritizedDomain() []*NameServer_PriorityDomain {
	if x != nil {
		return x.PrioritizedDomain
	}
	return nil
}

func (x *NameServer) GetGeoip() []*router.GeoIP {
	if x != nil {
		return x.Geoip
	}
	return nil
}

```

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	//Nameservers used by this DNS. Only traditional UDP servers are
	//support at the moment. A special value 'localhost' as a domain
	//address can be set to use DNS on local system.
	//
	//Deprecated: Do not use.
	//
	NameServers []*net.Endpoint `protobuf:"bytes,1,rep,name=NameServers,proto3" json:"NameServers,omitempty"`
	//NameServer list used by this DNS client.
	NameServer []*NameServer `protobuf:"bytes,5,rep,name=name_server,json=nameServer,proto3" json:"name_server,omitempty"`
	//Static hosts. Domain to IP.
	//Deprecated. Use static_hosts.
	//
	//Deprecated: Do not use.
	//
	//External endpoint for a DNS client to connect to an authoritative
	//DNS server.
	//
	//External endpoint for a DNS client to connect to an authoritative
	//DNS server. This endpoint will be used if a)
	//a) We don't have a preferred DNS server and b) Our connection to the
	//DNS server is not working.
	//
	//External endpoint for a DNS client to connect to an authoritative
	//DNS server. This endpoint will be used if a)
	//b) We have a preferred DNS server and b) Our connection to the
	//DNS server is working.
	//
	ClientIp    []byte                `protobuf:"bytes,3,opt,name=client_ip,json=clientIp,proto3" json:"client_ip,omitempty"`
	StaticHosts []*Config_HostMapping `protobuf:"bytes,4,rep,name=static_hosts,json=staticHosts,proto3" json:"static_hosts,omitempty"`
	//Tag is the inbound tag of DNS client.
	Tag string `protobuf:"bytes,6,opt,name=tag,proto3" json:"tag,omitempty"`
}


```go
func (x *NameServer) GetOriginalRules() []*NameServer_OriginalRule {
	if x != nil {
		return x.OriginalRules
	}
	return nil
}

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// Nameservers used by this DNS. Only traditional UDP servers are support at
	// the moment. A special value 'localhost' as a domain address can be set to
	// use DNS on local system.
	//
	// Deprecated: Do not use.
	NameServers []*net.Endpoint `protobuf:"bytes,1,rep,name=NameServers,proto3" json:"NameServers,omitempty"`
	// NameServer list used by this DNS client.
	NameServer []*NameServer `protobuf:"bytes,5,rep,name=name_server,json=nameServer,proto3" json:"name_server,omitempty"`
	// Static hosts. Domain to IP.
	// Deprecated. Use static_hosts.
	//
	// Deprecated: Do not use.
	Hosts map[string]*net.IPOrDomain `protobuf:"bytes,2,rep,name=Hosts,proto3" json:"Hosts,omitempty" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
	// Client IP for EDNS client subnet. Must be 4 bytes (IPv4) or 16 bytes
	// (IPv6).
	ClientIp    []byte                `protobuf:"bytes,3,opt,name=client_ip,json=clientIp,proto3" json:"client_ip,omitempty"`
	StaticHosts []*Config_HostMapping `protobuf:"bytes,4,rep,name=static_hosts,json=staticHosts,proto3" json:"static_hosts,omitempty"`
	// Tag is the inbound tag of DNS client.
	Tag string `protobuf:"bytes,6,opt,name=tag,proto3" json:"tag,omitempty"`
}

```

这段代码定义了两个函数，以及一个接口 `Config`。通过观察代码，我们可以看到以下几个关键点：

1. 函数 `Reset()`：该函数接收一个 `Config` 类型的参数 `x`，并将其赋值为 `Config{}`。然后判断 `protoimpl.UnsafeEnabled` 是否为真，如果是，则执行以下操作：

  - 创建一个 `file_app_dns_config_proto_msgTypes[1]` 的类型别名 `mi`。
  - 创建一个指向 `Config` 类型实例的 `protoimpl.X` 类型的指针 `x`。
  - 将 `mi` 和 `x` 都赋值为 `*Config()`，即 `*x`。
  - 如果 `protoimpl.UnsafeEnabled` 为真，则执行以下操作：

    - 创建一个空的 `MessageInfo` 类型的实例，并将其设置为 `mi`。
    - 将 `ms` 设置为 `(*x).MessageStateOf<file_app_dns_config_proto_msgTypes[1]>()`，即 `(*x).MessageInfo`。
    - 将 `ms`.StoreMessageInfo` 的实现设置为 `mi`。

2. 函数 `String()`：与 `Reset()` 函数类似，只是返回了一个字符串，而不是 `Config` 类型。

3. 函数 `ProtoMessage()`：该函数没有实际的作用，只是定义了一个 `ProtoMessage` 接口。

4. `Config` 接口：该接口没有实现，因为它只是一个占位符，接收的参数 `Config` 类型没有具体的实现。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，函数一是接收一个`Config`类型的参数`x`，并返回一个指向`protoreflect.Message`类型对象的`protoReflect`函数，函数二是获取`Config`类型的实例的`descriptor`字段，并返回它的字节数组和长度。

函数一的作用是：当`Config`类型的实例`x`被创建时，调用`protoimpl.X.MessageStateOf`和`protoimpl.Pointer(x)`方法，根据创建时的`LoadMessageInfo`方法来判断是否需要初始化`descriptor`字段，如果`LoadMessageInfo`为`nil`，则执行`ms.StoreMessageInfo(mi)`，然后调用`ms.LoadMessageInfo()`，返回`ms`，最后返回`ms`的`MessageOf`方法，这个方法将返回`descriptor`字段的字节数组和长度。

函数二的作用是：返回`Config`类型实例的`descriptor`字段的字节数组和长度，如果在函数一中`descriptor`字段没有被初始化，将会执行初始化操作。函数二同样适用于那些已经创建好的`Config`实例，它们将会返回其`descriptor`字段的字节数组和长度。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_app_dns_config_proto_msgTypes[1]
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
	return file_app_dns_config_proto_rawDescGZIP(), []int{1}
}

```

这两行代码是Go语言中的函数指针类型，定义了一个名为“Config”的名为“x”的变量，并对其进行了类型转换，具体类型转换为“*Config”。

这两行代码中，第一行使用了 deprecated 标签，表示该代码已经被废止了，不应该在生产环境中使用。

第二行代码中，定义了一个名为“GetNameServers”的函数，接收一个名为“Config”的指针变量作为参数，返回一个名为“NameServers”的 slice 类型。函数的实现中，先判断“Config”是否已经赋值，如果已经赋值，就返回“Config”中“NameServers”的值；否则返回一个空 slice。

第三行代码中，定义了一个名为“GetNameServer”的函数，与第二行代码中的“GetNameServers”类似，也是接收一个名为“Config”的指针变量作为参数，返回一个名为“NameServer”的 slice 类型。函数的实现中，同样先判断“Config”是否已经赋值，如果已经赋值，就返回“Config”中“NameServer”的值；否则返回一个空 slice。


```go
// Deprecated: Do not use.
func (x *Config) GetNameServers() []*net.Endpoint {
	if x != nil {
		return x.NameServers
	}
	return nil
}

func (x *Config) GetNameServer() []*NameServer {
	if x != nil {
		return x.NameServer
	}
	return nil
}

```

这段代码定义了两个函数，分别名为 `GetHosts` 和 `GetClientIp`。这两个函数接收一个名为 `x` 的 `*Config` 类型的参数。

首先，我们来看 `GetHosts` 函数。函数接收一个 `map[string]*net.IPOrDomain` 类型的变量，并返回其中的值。函数内部首先检查参数 `x` 是否为 `nil`，如果是，则直接返回 `nil`。否则，函数内部通过 `x.Hosts` 访问 `x` 所指向的 `*Config` 类型的参数的 `Hosts` 字段，并将其存储到一个 `map` 类型的变量中，最后返回这个 `map` 类型的变量。

接下来，我们来看 `GetClientIp` 函数。函数接收一个 `[]byte` 类型的变量，并返回其中的值。函数内部首先检查参数 `x` 是否为 `nil`，如果是，则直接返回 `nil`。否则，函数内部通过 `x.ClientIp` 访问 `x` 所指向的 `*Config` 类型的参数的 `ClientIp` 字段，并将其存储到一个 `[]byte` 类型的变量中，最后返回这个 `[]byte` 类型的变量。


```go
// Deprecated: Do not use.
func (x *Config) GetHosts() map[string]*net.IPOrDomain {
	if x != nil {
		return x.Hosts
	}
	return nil
}

func (x *Config) GetClientIp() []byte {
	if x != nil {
		return x.ClientIp
	}
	return nil
}

```

此代码定义了两个函数，以及一个结构体类型：NameServer_PriorityDomain。

函数 GetStaticHosts 和 GetTag 是从Config类中导出的函数，用于获取 Config 实例中的静态主机和标签。

函数 GetStaticHosts 接收一个Config实例，然后使用其中的 StaticHosts 字段，如果 StaticHosts 不为空，那么该函数返回 StaticHosts 切片。

函数 GetTag 同样接收一个Config实例，然后返回其中的 Tag 字段。

结构体类型 NameServer_PriorityDomain 在其中定义了一个域名的 Priority 字段，该字段用于指定优先级级，如果两个或多个域名解析失败，则该优先级较高的域名将保留。该结构体还定义了一个未知字段 field，用于保留无法解析的域名。


```go
func (x *Config) GetStaticHosts() []*Config_HostMapping {
	if x != nil {
		return x.StaticHosts
	}
	return nil
}

func (x *Config) GetTag() string {
	if x != nil {
		return x.Tag
	}
	return ""
}

type NameServer_PriorityDomain struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Type   DomainMatchingType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.dns.DomainMatchingType" json:"type,omitempty"`
	Domain string             `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
}

```

此代码定义了一个名为`NameServer_PriorityDomain`的接口类型，以及两个函数：`Reset()`和`String()`。

`Reset()`函数的作用是重置`x`对象的值，将其设置为`NameServer_PriorityDomain{}`类型的默认值。

`String()`函数的作用是将`x`对象转换为字符串形式，并返回该对象的`toString()`方法返回的值。

根据`NameServer_PriorityDomain`接口的定义，这些函数都执行以下操作：

1. 创建一个`NameServer_PriorityDomain`对象；
2. 如果`protoimpl.UnsafeEnabled`为`true`，则创建一个`file_app_dns_config_proto_msgTypes[2]`类型的对象，并将其存储为`x`对象的`MessageStateOf()`方法的`mi`字段；
3. 创建一个`file_app_dns_config_proto_msgTypes[2]`类型的对象，并将其存储为`x`对象的`MessageStateOf()`方法的`ms`字段；
4. 如果`protoimpl.UnsafeEnabled`为`true`，则创建一个`file_app_dns_config_proto_msgTypes[2]`类型的对象，并将其存储为`x`对象的`toString()`方法的返回值。

总而言之，此代码定义了一个`NameServer_PriorityDomain`接口类型，以及两个函数：`Reset()`和`String()`，用于创建和输出`NameServer_PriorityDomain`类型的对象。


```go
func (x *NameServer_PriorityDomain) Reset() {
	*x = NameServer_PriorityDomain{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *NameServer_PriorityDomain) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*NameServer_PriorityDomain) ProtoMessage() {}

```

此代码定义了两个函数，分别为 `func (x *NameServer_PriorityDomain) ProtoReflect() protoreflect.Message` 和 `func (*NameServer_PriorityDomain) Descriptor() ([]byte, []int)`。

1. `func (x *NameServer_PriorityDomain) ProtoReflect() protoreflect.Message` 函数接收一个 `*NameServer_PriorityDomain` 类型的参数 `x`，并返回一个 `protoreflect.Message` 类型。函数的作用是将 `x` 包装为 `protoreflect.Message` 类型，以便可以安全地使用 `protoreflect` 包中的方法。

2. `func (*NameServer_PriorityDomain) Descriptor() ([]byte, []int)` 函数接收一个 `*NameServer_PriorityDomain` 类型的 `x`，并返回其 `Descriptor` 函数的签名包含的字节数和参数数量。函数的作用是返回 `x` 的 `Descriptor` 函数的签名，以便调用者可以使用自己的代码来解析 `x` 的 `Descriptor` 函数返回的结果。

函数的实现细节如下：

1. `func (x *NameServer_PriorityDomain) ProtoReflect() protoreflect.Message` 函数首先使用 `file_app_dns_config_proto_msgTypes[2]` 获取第二个名为 `NameServer_PriorityDomain` 的方法类型，然后判断 `x` 是否为 `*NameServer_PriorityDomain` 类型的实例。如果是，函数创建一个名为 `ms` 的 `FileAppDnsConfigProtocol` 类型指针，并将其指向 `x`，然后使用 `ms.MessageStateOf(x)` 获取到 `x` 的 `MessageInfo` 字段，如果 `x` 的 `MessageInfo` 为 `nil`，则将 `nil` 存储为 `ms.MessageInfo`，否则将 `mi` 存储为 `ms.MessageInfo`。最后，函数调用 `ms.MessageOf(x)` 来获取 `x` 的 `Message` 字段，并将其存储为 `ms`，从而可以返回 `ms`。

2. `func (*NameServer_PriorityDomain) Descriptor() ([]byte, []int)` 函数创建一个名为 `descriptor` 的新函数，该函数接收一个 `*NameServer_PriorityDomain` 类型的 `x`，并返回其 `Descriptor` 函数的签名包含的字节数和参数数量。函数首先创建一个空字符数组 `result`，然后定义一个名为 `paramCount` 的整数变量，用于存储 `Descriptor` 函数的参数数量。接下来，函数使用 `file_app_dns_config_proto_rawDescGZIP()` 函数将 `x` 的 `Descriptor` 函数签名二进制编码并解码为字符串，然后使用 `strconv.Itoa()` 函数将字符串转换为整数。最后，函数将 `paramCount` 和解码后的字符串作为参数传递给 `Descriptor` 函数，并将结果存储为 `result`。


```go
func (x *NameServer_PriorityDomain) ProtoReflect() protoreflect.Message {
	mi := &file_app_dns_config_proto_msgTypes[2]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use NameServer_PriorityDomain.ProtoReflect.Descriptor instead.
func (*NameServer_PriorityDomain) Descriptor() ([]byte, []int) {
	return file_app_dns_config_proto_rawDescGZIP(), []int{0, 0}
}

```

此代码定义了两个函数，以及一个结构体类型。

第一个函数 `func (x *NameServer_PriorityDomain) GetType() DomainMatchingType` 接收一个名为 `x` 的 `NameServer_PriorityDomain` 类型的指针变量作为参数，并返回该 `x` 类型对象的 `DomainMatchingType` 字段的值。

第二个函数 `func (x *NameServer_PriorityDomain) GetDomain() string` 同样接收一个名为 `x` 的 `NameServer_PriorityDomain` 类型的指针变量作为参数，并返回该 `x` 类型对象的 `Domain` 字段的值。如果 `x` 为 `nil`，则返回一个空字符串 ` ""`。

第三个函数 `type NameServer_OriginalRule struct { ... }` 定义了一个名为 `NameServer_OriginalRule` 的结构体类型。这个结构体类型包含三个字段：`Rule`、`Size` 和 `UnknownFields`。其中，`Rule` 和 `Size` 是按照 `protobuf` 语法定义的字符串字段，`UnknownFields` 是按照 `protobuf` 语法定义的未知字段。

另外，代码中还定义了一个全局变量 `gRU器的IP地址`，用于存储 IP 地址。


```go
func (x *NameServer_PriorityDomain) GetType() DomainMatchingType {
	if x != nil {
		return x.Type
	}
	return DomainMatchingType_Full
}

func (x *NameServer_PriorityDomain) GetDomain() string {
	if x != nil {
		return x.Domain
	}
	return ""
}

type NameServer_OriginalRule struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Rule string `protobuf:"bytes,1,opt,name=rule,proto3" json:"rule,omitempty"`
	Size uint32 `protobuf:"varint,2,opt,name=size,proto3" json:"size,omitempty"`
}

```

此代码定义了一个名为`NameServer_OriginalRule`的接口类型，以及两个函数：`Reset()`和`String()`。下面是详细的解释：

1. `Reset()`函数的作用是重置`x`对象的原始值，将其设置为`NameServer_OriginalRule{}`。

2. `String()`函数的作用是将`x`对象转换为字符串表示，并返回原始值。这个函数使用了`protoimpl.X.MessageStringOf()`来实现。

3. `ProtoMessage()`函数的作用是定义`NameServer_OriginalRule`接口的`Message`类型。这个函数使用了`protoimpl.UnsafeEnabled`选项，以便在编译时允许安全的类型操作。

由于`NameServer_OriginalRule`接口的定义比较简单，没有太多可以展开的内容，因此以上代码已经足够清晰地描述了该接口的作用。


```go
func (x *NameServer_OriginalRule) Reset() {
	*x = NameServer_OriginalRule{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[3]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *NameServer_OriginalRule) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*NameServer_OriginalRule) ProtoMessage() {}

```

这段代码定义了一个名为"func"的函数，它接收一个名为"x"的整数类型的参数，并返回一个名为"protoreflect.Message"的接口类型的变量。

具体来说，这段代码实现了一个依赖函数"Descriptor"，它返回了一个描述符类型的切片，其中第一个元素是"file_app_dns_config_proto_rawDescGZIP"字节数组，第二个元素是两个整数类型的切片，分别对应于"nameServerOriginalRule"和"messageType"的序列。

"file_app_dns_config_proto_rawDescGZIP"字节数组包含了从"file_app_dns_config_proto.go"中定义的原始规则结构体中获取的值。这个结构体在问题描述中没有提到，但是根据名称可以猜测它可能是一个自定义的结构体，用于表示"dns_config_name_server_rule"的原始规则。

函数的实现还涉及到了"protoreflect.Message"接口的"Message"字段，它表示了原始规则的结构体类型，但并不包含原始规则的具体实现。另外，函数还使用了"unused"的标识符 "unused" 作为函数名，这可能是一个警告，表示该函数在依赖定义中没有被使用过。


```go
func (x *NameServer_OriginalRule) ProtoReflect() protoreflect.Message {
	mi := &file_app_dns_config_proto_msgTypes[3]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use NameServer_OriginalRule.ProtoReflect.Descriptor instead.
func (*NameServer_OriginalRule) Descriptor() ([]byte, []int) {
	return file_app_dns_config_proto_rawDescGZIP(), []int{0, 1}
}

```

此代码定义了两个函数：GetRule 和 GetSize，以及一个结构体类型Config_HostMapping。

GetRule函数接收一个名为x的指针作为参数，并判断x是否为nil。如果是，则返回x的Rule值。否则，返回一个空字符串。

GetSize函数与GetRule函数类似，只是返回一个uint32类型的值，而不是一个string类型的值。

Config_HostMapping结构体类型定义了一个域名的状态、缓存大小以及未知字段。其中，域名的类型为DomainMatchingType，该类型可以在代码中使用。此结构体类型定义了一个域名、IP地址和代理域名，以及一个可选的代理域名。V2Ray将在IP查询时使用代理域名，但如果IP地址为空，则此字段将无效。


```go
func (x *NameServer_OriginalRule) GetRule() string {
	if x != nil {
		return x.Rule
	}
	return ""
}

func (x *NameServer_OriginalRule) GetSize() uint32 {
	if x != nil {
		return x.Size
	}
	return 0
}

type Config_HostMapping struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Type   DomainMatchingType `protobuf:"varint,1,opt,name=type,proto3,enum=v2ray.core.app.dns.DomainMatchingType" json:"type,omitempty"`
	Domain string             `protobuf:"bytes,2,opt,name=domain,proto3" json:"domain,omitempty"`
	Ip     [][]byte           `protobuf:"bytes,3,rep,name=ip,proto3" json:"ip,omitempty"`
	// ProxiedDomain indicates the mapped domain has the same IP address on this
	// domain. V2Ray will use this domain for IP queries. This field is only
	// effective if ip is empty.
	ProxiedDomain string `protobuf:"bytes,4,opt,name=proxied_domain,json=proxiedDomain,proto3" json:"proxied_domain,omitempty"`
}

```

这是一段C语言代码，定义了两个函数名为"func"的函数，以及一个名为"*Config_HostMapping"的指针类型。函数的参数分别为一个"Config_HostMapping"类型的变量"x"，以及一个空指针类型"*Config_HostMapping"。函数没有返回值，但使用了"*"（指针类型）和"?"（未定义类型）以提高代码可读性。

第一个函数"Reset"的实现如下：


func (x *Config_HostMapping) Reset() {
	*x = Config_HostMapping{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[5]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}


这个函数的作用是重置一个"Config_HostMapping"类型的变量"x"，将x的值设为其类型的默认值，如果`protoimpl.UnsafeEnabled`为`true`，则使用`file_app_dns_config_proto_msgTypes[5]`获取与该函数相关的消息类型，并将其存储在`ms`中，最后将`mi`存储为0，使`ms`不再引用`x`，从而达到消除引用循环的目的是。

第二个函数"String"的实现如下：


func (x *Config_HostMapping) String() string {
	return protoimpl.X.MessageStringOf(x)
}


这个函数的作用是打印一个"Config_HostMapping"类型的变量"x"的类型，与`toString()`方法的作用相同。

第三个函数名为"*Config_HostMapping"的指针类型定义了一个名为"*"（指针类型）和"?"（未定义类型）的函数，但未实现任何函数体，因此这个指针类型没有任何实际功能。


```go
func (x *Config_HostMapping) Reset() {
	*x = Config_HostMapping{}
	if protoimpl.UnsafeEnabled {
		mi := &file_app_dns_config_proto_msgTypes[5]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config_HostMapping) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config_HostMapping) ProtoMessage() {}

```

这段代码定义了一个名为func的函数，它接收一个名为x的*Config_HostMapping类型的参数，并返回一个名为protoreflect.Message的接口类型。

函数的作用是输出一个指向Config_HostMapping的接口类型的引用，并返回该接口类型的一些元数据，如名称、方法列表和序列化器。

具体来说，函数首先检查是否启用了`protoreflect.UnsafeEnabled`选项，如果是，则尝试使用x的地址作为参数传递给Message的`LoadMessageInfo`函数。如果`LoadMessageInfo`函数返回的是一个`null`值，则将x的地址存储为mi的地址，然后返回mi的地址。

否则，函数将x的地址传递给Message的`MessageOf`函数，并返回mi的地址。

接下来，函数实现了一个deprecated的函数，它返回Config_HostMapping的描述符，包括名称、方法列表和序列化器。函数使用了file_app_dns_config_proto_rawDescGZIP序列化器，该序列化器将Struct的名称和结构体中的字段名称映射到gzip编码的字节数组。


```go
func (x *Config_HostMapping) ProtoReflect() protoreflect.Message {
	mi := &file_app_dns_config_proto_msgTypes[5]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Config_HostMapping.ProtoReflect.Descriptor instead.
func (*Config_HostMapping) Descriptor() ([]byte, []int) {
	return file_app_dns_config_proto_rawDescGZIP(), []int{1, 1}
}

```

这段代码定义了三个函数，分别返回如下：

1. `func (x *Config_HostMapping) GetType() DomainMatchingType`: 该函数接收一个`Config_HostMapping`类型的参数`x`，并返回其`Type`类型的值。
2. `func (x *Config_HostMapping) GetDomain() string`: 该函数接收一个`Config_HostMapping`类型的参数`x`，并返回其`Domain`类型的值。
3. `func (x *Config_HostMapping) GetIp() [][]byte`: 该函数接收一个`Config_HostMapping`类型的参数`x`，并返回其`Ip`类型的值。


```go
func (x *Config_HostMapping) GetType() DomainMatchingType {
	if x != nil {
		return x.Type
	}
	return DomainMatchingType_Full
}

func (x *Config_HostMapping) GetDomain() string {
	if x != nil {
		return x.Domain
	}
	return ""
}

func (x *Config_HostMapping) GetIp() [][]byte {
	if x != nil {
		return x.Ip
	}
	return nil
}

```

It looks like a乘法表， with each row of the table is a sequence of multiple 8-bit values. The values in each row correspond to a single 8-bit value, and appear to be repeated according to a specific pattern.

The first row of the table is 0x10 0x03 0x42 0x47 0x0a 0x16 0x63 0x6f 0x6d 0x2e 0x76 0x32 0x72 0x61 0x79 0x2e 0x63 0x6f 0x72 0x65 2e 2e 61 70 70 2e 64 6e 73 50 01 5a 16 76 32 72 61 79 2e 63 6f 6d 2f 63 6f 72 65 2f 61 70 70 2e 61 70 2e 44 6e 73 62 06 70 72 6f 74 6f 33.


```go
func (x *Config_HostMapping) GetProxiedDomain() string {
	if x != nil {
		return x.ProxiedDomain
	}
	return ""
}

var File_app_dns_config_proto protoreflect.FileDescriptor

var file_app_dns_config_proto_rawDesc = []byte{
	0x0a, 0x14, 0x61, 0x70, 0x70, 0x2f, 0x64, 0x6e, 0x73, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67,
	0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x12, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f,
	0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x1a, 0x18, 0x63, 0x6f, 0x6d, 0x6d,
	0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x61, 0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x2e, 0x70,
	0x72, 0x6f, 0x74, 0x6f, 0x1a, 0x1c, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74,
	0x2f, 0x64, 0x65, 0x73, 0x74, 0x69, 0x6e, 0x61, 0x74, 0x69, 0x6f, 0x6e, 0x2e, 0x70, 0x72, 0x6f,
	0x74, 0x6f, 0x1a, 0x17, 0x61, 0x70, 0x70, 0x2f, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2f, 0x63,
	0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0xcb, 0x03, 0x0a, 0x0a,
	0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x12, 0x39, 0x0a, 0x07, 0x61, 0x64,
	0x64, 0x72, 0x65, 0x73, 0x73, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x1f, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
	0x6e, 0x65, 0x74, 0x2e, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x52, 0x07, 0x61, 0x64,
	0x64, 0x72, 0x65, 0x73, 0x73, 0x12, 0x5c, 0x0a, 0x12, 0x70, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74,
	0x69, 0x7a, 0x65, 0x64, 0x5f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x03, 0x28,
	0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61,
	0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65,
	0x72, 0x2e, 0x50, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x79, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e,
	0x52, 0x11, 0x70, 0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x69, 0x7a, 0x65, 0x64, 0x44, 0x6f, 0x6d,
	0x61, 0x69, 0x6e, 0x12, 0x32, 0x0a, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x18, 0x03, 0x20, 0x03,
	0x28, 0x0b, 0x32, 0x1c, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
	0x61, 0x70, 0x70, 0x2e, 0x72, 0x6f, 0x75, 0x74, 0x65, 0x72, 0x2e, 0x47, 0x65, 0x6f, 0x49, 0x50,
	0x52, 0x05, 0x67, 0x65, 0x6f, 0x69, 0x70, 0x12, 0x52, 0x0a, 0x0e, 0x6f, 0x72, 0x69, 0x67, 0x69,
	0x6e, 0x61, 0x6c, 0x5f, 0x72, 0x75, 0x6c, 0x65, 0x73, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b, 0x32,
	0x2b, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70,
	0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x2e,
	0x4f, 0x72, 0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c, 0x65, 0x52, 0x0d, 0x6f, 0x72,
	0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c, 0x65, 0x73, 0x1a, 0x64, 0x0a, 0x0e, 0x50,
	0x72, 0x69, 0x6f, 0x72, 0x69, 0x74, 0x79, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x3a, 0x0a,
	0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0e, 0x32, 0x26, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73,
	0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x69, 0x6e, 0x67, 0x54,
	0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65, 0x12, 0x16, 0x0a, 0x06, 0x64, 0x6f, 0x6d,
	0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69,
	0x6e, 0x1a, 0x36, 0x0a, 0x0c, 0x4f, 0x72, 0x69, 0x67, 0x69, 0x6e, 0x61, 0x6c, 0x52, 0x75, 0x6c,
	0x65, 0x12, 0x12, 0x0a, 0x04, 0x72, 0x75, 0x6c, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52,
	0x04, 0x72, 0x75, 0x6c, 0x65, 0x12, 0x12, 0x0a, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x18, 0x02, 0x20,
	0x01, 0x28, 0x0d, 0x52, 0x04, 0x73, 0x69, 0x7a, 0x65, 0x22, 0xc3, 0x04, 0x0a, 0x06, 0x43, 0x6f,
	0x6e, 0x66, 0x69, 0x67, 0x12, 0x45, 0x0a, 0x0b, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76,
	0x65, 0x72, 0x73, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x1f, 0x2e, 0x76, 0x32, 0x72, 0x61,
	0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
	0x74, 0x2e, 0x45, 0x6e, 0x64, 0x70, 0x6f, 0x69, 0x6e, 0x74, 0x42, 0x02, 0x18, 0x01, 0x52, 0x0b,
	0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x73, 0x12, 0x3f, 0x0a, 0x0b, 0x6e,
	0x61, 0x6d, 0x65, 0x5f, 0x73, 0x65, 0x72, 0x76, 0x65, 0x72, 0x18, 0x05, 0x20, 0x03, 0x28, 0x0b,
	0x32, 0x1e, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
	0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x4e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72,
	0x52, 0x0a, 0x6e, 0x61, 0x6d, 0x65, 0x53, 0x65, 0x72, 0x76, 0x65, 0x72, 0x12, 0x3f, 0x0a, 0x05,
	0x48, 0x6f, 0x73, 0x74, 0x73, 0x18, 0x02, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x25, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73,
	0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x48, 0x6f, 0x73, 0x74, 0x73, 0x45, 0x6e, 0x74,
	0x72, 0x79, 0x42, 0x02, 0x18, 0x01, 0x52, 0x05, 0x48, 0x6f, 0x73, 0x74, 0x73, 0x12, 0x1b, 0x0a,
	0x09, 0x63, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x5f, 0x69, 0x70, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0c,
	0x52, 0x08, 0x63, 0x6c, 0x69, 0x65, 0x6e, 0x74, 0x49, 0x70, 0x12, 0x49, 0x0a, 0x0c, 0x73, 0x74,
	0x61, 0x74, 0x69, 0x63, 0x5f, 0x68, 0x6f, 0x73, 0x74, 0x73, 0x18, 0x04, 0x20, 0x03, 0x28, 0x0b,
	0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70,
	0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x48, 0x6f, 0x73,
	0x74, 0x4d, 0x61, 0x70, 0x70, 0x69, 0x6e, 0x67, 0x52, 0x0b, 0x73, 0x74, 0x61, 0x74, 0x69, 0x63,
	0x48, 0x6f, 0x73, 0x74, 0x73, 0x12, 0x10, 0x0a, 0x03, 0x74, 0x61, 0x67, 0x18, 0x06, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x03, 0x74, 0x61, 0x67, 0x1a, 0x5b, 0x0a, 0x0a, 0x48, 0x6f, 0x73, 0x74, 0x73,
	0x45, 0x6e, 0x74, 0x72, 0x79, 0x12, 0x10, 0x0a, 0x03, 0x6b, 0x65, 0x79, 0x18, 0x01, 0x20, 0x01,
	0x28, 0x09, 0x52, 0x03, 0x6b, 0x65, 0x79, 0x12, 0x37, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65,
	0x18, 0x02, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x21, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63,
	0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65, 0x74, 0x2e, 0x49,
	0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65,
	0x3a, 0x02, 0x38, 0x01, 0x1a, 0x98, 0x01, 0x0a, 0x0b, 0x48, 0x6f, 0x73, 0x74, 0x4d, 0x61, 0x70,
	0x70, 0x69, 0x6e, 0x67, 0x12, 0x3a, 0x0a, 0x04, 0x74, 0x79, 0x70, 0x65, 0x18, 0x01, 0x20, 0x01,
	0x28, 0x0e, 0x32, 0x26, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
	0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73, 0x2e, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61,
	0x74, 0x63, 0x68, 0x69, 0x6e, 0x67, 0x54, 0x79, 0x70, 0x65, 0x52, 0x04, 0x74, 0x79, 0x70, 0x65,
	0x12, 0x16, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09,
	0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12, 0x0e, 0x0a, 0x02, 0x69, 0x70, 0x18, 0x03,
	0x20, 0x03, 0x28, 0x0c, 0x52, 0x02, 0x69, 0x70, 0x12, 0x25, 0x0a, 0x0e, 0x70, 0x72, 0x6f, 0x78,
	0x69, 0x65, 0x64, 0x5f, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x04, 0x20, 0x01, 0x28, 0x09,
	0x52, 0x0d, 0x70, 0x72, 0x6f, 0x78, 0x69, 0x65, 0x64, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x2a,
	0x45, 0x0a, 0x12, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x4d, 0x61, 0x74, 0x63, 0x68, 0x69, 0x6e,
	0x67, 0x54, 0x79, 0x70, 0x65, 0x12, 0x08, 0x0a, 0x04, 0x46, 0x75, 0x6c, 0x6c, 0x10, 0x00, 0x12,
	0x0d, 0x0a, 0x09, 0x53, 0x75, 0x62, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x10, 0x01, 0x12, 0x0b,
	0x0a, 0x07, 0x4b, 0x65, 0x79, 0x77, 0x6f, 0x72, 0x64, 0x10, 0x02, 0x12, 0x09, 0x0a, 0x05, 0x52,
	0x65, 0x67, 0x65, 0x78, 0x10, 0x03, 0x42, 0x47, 0x0a, 0x16, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x61, 0x70, 0x70, 0x2e, 0x64, 0x6e, 0x73,
	0x50, 0x01, 0x5a, 0x16, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
	0x72, 0x65, 0x2f, 0x61, 0x70, 0x70, 0x2f, 0x64, 0x6e, 0x73, 0xaa, 0x02, 0x12, 0x56, 0x32, 0x52,
	0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x41, 0x70, 0x70, 0x2e, 0x44, 0x6e, 0x73, 0x62,
	0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

FILE: api/v1alpha1/appdnsconfigproto/app_dns_config_proto_rawDesc.go


package v2ray

import (
	"fmt"
	"io/ioutil"

	"google.golang.org/protobuf/v2"
	"google.golang.org/protobuf/v2/types"
	"google.golang.org/protobuf/v2/math/蜂拥比"
	"google.golang.org/protobuf/v2/trace"
	"google.golang.org/protobuf/v2/types/code"
	"google.golang.org/protobuf/v2/x packing/golang"
	"google.golang.org/protobuf/v2/x/math/工欲善兮"
	"google.golang.org/protobuf/v2/x/math/Notify ratings"
	"google.golang.org/protobuf/v2/x/math/SuccessRate"
	"google.golang.org/protobuf/v2/x/math/WindowTerm"
	"google.golang.org/protobuf/v2/reflect/reflect"
	"google.golang.org/protobuf/v2/runtime"
	"google.golang.org/protobuf/v2/runtime/身份认证"
	"google.golang.org/protobuf/v2/runtime/ scoring"
	"google.golang.org/protobuf/v2/runtime/ status"
	"google.golang.org/protobuf/v2/runtime/ testing"
	"google.golang.org/protobuf/v2/runtime/code. Ref"
	"google.golang.org/protobuf/v2/runtime/輕量级/lightweights"
	"google.golang.org/protobuf/v2/runtime/recordpath/recordpath_rules"
	"google.golang.org/protobuf/v2/runtime/time"
	"google.golang.org/protobuf/v2/runtime/stat你可能想要


```go
var (
	file_app_dns_config_proto_rawDescOnce sync.Once
	file_app_dns_config_proto_rawDescData = file_app_dns_config_proto_rawDesc
)

func file_app_dns_config_proto_rawDescGZIP() []byte {
	file_app_dns_config_proto_rawDescOnce.Do(func() {
		file_app_dns_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_app_dns_config_proto_rawDescData)
	})
	return file_app_dns_config_proto_rawDescData
}

var file_app_dns_config_proto_enumTypes = make([]protoimpl.EnumInfo, 1)
var file_app_dns_config_proto_msgTypes = make([]protoimpl.MessageInfo, 6)
var file_app_dns_config_proto_goTypes = []interface{}{
	(DomainMatchingType)(0),           // 0: v2ray.core.app.dns.DomainMatchingType
	(*NameServer)(nil),                // 1: v2ray.core.app.dns.NameServer
	(*Config)(nil),                    // 2: v2ray.core.app.dns.Config
	(*NameServer_PriorityDomain)(nil), // 3: v2ray.core.app.dns.NameServer.PriorityDomain
	(*NameServer_OriginalRule)(nil),   // 4: v2ray.core.app.dns.NameServer.OriginalRule
	nil,                               // 5: v2ray.core.app.dns.Config.HostsEntry
	(*Config_HostMapping)(nil),        // 6: v2ray.core.app.dns.Config.HostMapping
	(*net.Endpoint)(nil),              // 7: v2ray.core.common.net.Endpoint
	(*router.GeoIP)(nil),              // 8: v2ray.core.app.router.GeoIP
	(*net.IPOrDomain)(nil),            // 9: v2ray.core.common.net.IPOrDomain
}
```

This is a list of type names for the v2ray.core.app.router.GeoIP type. It appears to be defining a set of rules for routing traffic to a specified IP or domain, based on the source and destination names of the packets.

The first field is "NameServer.geoip:type\_name", which is the v2ray.core.app.router.GeoIP type that will be used to


```go
var file_app_dns_config_proto_depIdxs = []int32{
	7,  // 0: v2ray.core.app.dns.NameServer.address:type_name -> v2ray.core.common.net.Endpoint
	3,  // 1: v2ray.core.app.dns.NameServer.prioritized_domain:type_name -> v2ray.core.app.dns.NameServer.PriorityDomain
	8,  // 2: v2ray.core.app.dns.NameServer.geoip:type_name -> v2ray.core.app.router.GeoIP
	4,  // 3: v2ray.core.app.dns.NameServer.original_rules:type_name -> v2ray.core.app.dns.NameServer.OriginalRule
	7,  // 4: v2ray.core.app.dns.Config.NameServers:type_name -> v2ray.core.common.net.Endpoint
	1,  // 5: v2ray.core.app.dns.Config.name_server:type_name -> v2ray.core.app.dns.NameServer
	5,  // 6: v2ray.core.app.dns.Config.Hosts:type_name -> v2ray.core.app.dns.Config.HostsEntry
	6,  // 7: v2ray.core.app.dns.Config.static_hosts:type_name -> v2ray.core.app.dns.Config.HostMapping
	0,  // 8: v2ray.core.app.dns.NameServer.PriorityDomain.type:type_name -> v2ray.core.app.dns.DomainMatchingType
	9,  // 9: v2ray.core.app.dns.Config.HostsEntry.value:type_name -> v2ray.core.common.net.IPOrDomain
	0,  // 10: v2ray.core.app.dns.Config.HostMapping.type:type_name -> v2ray.core.app.dns.DomainMatchingType
	11, // [11:11] is the sub-list for method output_type
	11, // [11:11] is the sub-list for method input_type
	11, // [11:11] is the sub-list for extension type_name
	11, // [11:11] is the sub-list for extension extendee
	0,  // [0:11] is the sub-list for field type_name
}

```

This code appears to be implementing a function that creates a struct that represents the state of the ConfigHostMapping object in the DNS API. The struct has several fields, including the current state of the object, the size of the cache for the host names, and a list of unknown fields.

The function also includes two function declarations. The first is the type exporter for the struct, which is used to export the struct values as a Go type. The second is the exporter for the ConfigHostMapping object, which is used to export the object values as a Go type.

It appears that the struct is used to represent the state of the ConfigHostMapping object in the DNS API. The current state of the object is represented by the field `state`, which is a boolean value. The size of the cache for the host names is represented by the field `sizeCache`, which is an int. The unknown fields are represented by the field `unknownFields`, which is a slice of不明数据类型。


```go
func init() { file_app_dns_config_proto_init() }
func file_app_dns_config_proto_init() {
	if File_app_dns_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_app_dns_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*NameServer); i {
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
		file_app_dns_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
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
		file_app_dns_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*NameServer_PriorityDomain); i {
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
		file_app_dns_config_proto_msgTypes[3].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*NameServer_OriginalRule); i {
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
		file_app_dns_config_proto_msgTypes[5].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Config_HostMapping); i {
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
			RawDescriptor: file_app_dns_config_proto_rawDesc,
			NumEnums:      1,
			NumMessages:   6,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_app_dns_config_proto_goTypes,
		DependencyIndexes: file_app_dns_config_proto_depIdxs,
		EnumInfos:         file_app_dns_config_proto_enumTypes,
		MessageInfos:      file_app_dns_config_proto_msgTypes,
	}.Build()
	File_app_dns_config_proto = out.File
	file_app_dns_config_proto_rawDesc = nil
	file_app_dns_config_proto_goTypes = nil
	file_app_dns_config_proto_depIdxs = nil
}

```

# `app/dns/dns.go`

这段代码是一个 Go 语言编写的软件包，名为 "dns"，其实现了 Go 语言标准库中的 "core.DNS" 功能。该软件包定义了一个 DNS 函数，用于创建和解析 DNS 查询。

下面是代码的概述：

1. 首先定义了一个名为 "dns" 的软件包，该软件包中包含一个名为 "core.DNS" 的函数，用于创建和解析 DNS 查询。

2. 在 "dns" 软件包的定义中，定义了一个名为 "errorgen" 的函数，它使用了 "go:generate" 指令生成了一个名为 "v2ray.com/core/common/errors/errorgen" 的函数。

3. 在 "errorgen" 函数中，使用了 "generate" 函数将一个名为 "errorgen" 的函数导出为 "v2ray.com/core/common/errors/errorgen"。

4. 在 "errorgen" 函数内部，没有定义任何函数体，因此它的作用将是导出 "errorgen" 为一个名为 "v2ray.com/core/common/errors/errorgen" 的函数。

总结一下，这段代码定义了一个名为 "dns" 的软件包，其中包含一个名为 "core.DNS" 的函数，用于创建和解析 DNS 查询。通过定义一个名为 "errorgen" 的函数，并使用 "go:generate" 指令将其导出为名为 "v2ray.com/core/common/errors/errorgen" 的函数，可以方便地在其他 Go 代码中使用这个功能。


```go
// Package dns is an implementation of core.DNS feature.
package dns

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `app/dns/dnscommon.go`

这段代码是一个 Go 语言编写的 DFS 根服务器，用于将二进制数据流量通过二层隧道传输到远程服务器。以下是对代码的详细解释：

1. `// +build !confonly`：这是 Go 语言中的 `build` 选项，用于编译时生成二进制文件。如果没有这个选项，则会生成一个名为 `dns_route.go` 的源文件。

2. `package dns`：这是定义了一个名为 `dns` 的包。

3. `import (`：这是 Go 语言中的 `import` 语句，用于导入其他包的函数、类型和变量。

4. `encoding/binary`：这是 Go 语言中的 `encoding` 和 `binary` 包，用于与二进制数据类型相关的方法。

5. `time`：这是 Go 语言中的 `time` 包，用于处理与时间相关的操作。

6. `(`、`)`：这是括号，用于分别包围前面的代码和后面的代码。

7. `package dns`：这是定义了一个名为 `dns` 的包。

8. `import (`：这是 Go 语言中的 `import` 语句，用于导入其他包的函数、类型和变量。

9. `encoding/binary`：这是 Go 语言中的 `encoding` 和 `binary` 包，用于与二进制数据类型相关的方法。

10. `time`：这是 Go 语言中的 `time` 包，用于处理与时间相关的操作。

11. `(`、`)`：这是括号，用于分别包围前面的代码和后面的代码。

12. `package dns`：这是定义了一个名为 `dns` 的包。

13. `import (`：这是 Go 语言中的 `import` 语句，用于导入其他包的函数、类型和变量。

14. `encoding/binary`：这是 Go 语言中的 `encoding` 和 `binary` 包，用于与二进制数据类型相关的方法。

15. `time`：这是 Go 语言中的 `time` 包，用于处理与时间相关的操作。

16. `(`、`)`：这是括号，用于分别包围前面的代码和后面的代码。

17. `+build !confonly`：这是 `build` 选项，用于编译时生成二进制文件。如果没有这个选项，则会生成一个名为 `dns_route.go` 的源文件。

综上所述，这段代码定义了一个名为 `dns_route` 的函数，它接收一个二进制数据流，将其传输到远程服务器，然后通过二层隧道返回。


```go
// +build !confonly

package dns

import (
	"encoding/binary"
	"time"

	"golang.org/x/net/dns/dnsmessage"
	"v2ray.com/core/common"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/net"
	dns_feature "v2ray.com/core/features/dns"
)

```

这段代码定义了一个名为Fqdn的函数，接受一个字符串类型的域名参数，并返回该域名的标准化形式。如果输入的域名已经是以点号结尾的，则直接返回该域名；否则，在域名后加上点号。

接着定义了一个名为record的结构体，该结构体包含一个IPRecord类型的A记录和一个IPRecord类型的AAAA记录。

最后，定义了一个名为IPRecord的接口，该接口定义了一个IPRecord类型的记录，包含一个IP地址和一个DNSMessage类型的记录。同时，定义了一个名为IPRecord的缓存记录类型，该类型使用了一个IPRecord类型的实例，该实例使用了一个字段存储了最近已经解决过的域名，使用了expire时间戳来记录该实例是否已经过期，以及一个RCode类型的字段来记录DNSMessage类型的记录。

该代码的作用是实现了一个IP地址解析函数，可以接受一个字符串类型的域名参数，返回一个已标准化后的域名。在该函数中，对输入的域名进行了处理，以保证其符合IP地址的规范，然后使用IPRecord类型的实例来存储解析过的域名，并使用expire时间戳来记录该域名是否过期。


```go
// Fqdn normalize domain make sure it ends with '.'
func Fqdn(domain string) string {
	if len(domain) > 0 && domain[len(domain)-1] == '.' {
		return domain
	}
	return domain + "."
}

type record struct {
	A    *IPRecord
	AAAA *IPRecord
}

// IPRecord is a cacheable item for a resolved domain
type IPRecord struct {
	ReqID  uint16
	IP     []net.Address
	Expire time.Time
	RCode  dnsmessage.RCode
}

```

该代码段定义了两个函数：一个是 `getIPs()`，另一个是 `isNewer()`。

1. `getIPs()` 函数接收一个 IPRecord 类型的参数 `r`，并返回一个 IP 地址数组和错误。函数首先检查 `r` 是否为 `nil`，如果是，则返回 `nil` 和错误 `errRecordNotFound`。然后，函数检查 `r` 的 `Expire` 字段是否小于当前时间 `time.Now()`，如果是，则返回 `nil` 和错误 `dns_feature.RCodeError(r.RCode)`。最后，函数返回 `r.IP` 和 `nil`。

2. `isNewer()` 函数接收两个 IPRecord 类型的参数 `baseRec` 和 `newRec`，并返回一个布尔值，表示是否 `newRec` 是较新的实例。函数首先检查 `newRec` 是否为 `nil`，如果是，则返回 `false`。然后，函数检查 `baseRec` 是否为 `nil`，如果是，则返回 `true`。最后，函数比较 `baseRec.Expire` 和 `newRec.Expire` 的大小，如果 `baseRec.Expire` 在先，则 `newRec` 是较新的实例。


```go
func (r *IPRecord) getIPs() ([]net.Address, error) {
	if r == nil || r.Expire.Before(time.Now()) {
		return nil, errRecordNotFound
	}
	if r.RCode != dnsmessage.RCodeSuccess {
		return nil, dns_feature.RCodeError(r.RCode)
	}
	return r.IP, nil
}

func isNewer(baseRec *IPRecord, newRec *IPRecord) bool {
	if newRec == nil {
		return false
	}
	if baseRec == nil {
		return true
	}
	return baseRec.Expire.Before(newRec.Expire)
}

```

This code appears to be a Go language implementation of the EDNS0 protocol. It defines a `dnsmessage.Resource` struct to represent an options resource for EDNS0 subscriptions.

The `genEDNS0Options` function takes a client IP address and generates options for that IP address using the EDNS0 subnet provided by the IP address. The function returns a pointer to the generated resource.

The `EDNS0SUBNET` constant is defined as 0x08. It appears that this is the network address for the "Editing the DNS on cluster" option, which allows a DNS server to perform a EDNS0 subnet.

The generated resource's body contains the options for the generated IP address. The generated IP address is divided into two parts: a subnet and an offset. The subnet is generated based on the passed client IP address and the netmask specified by the `net.IPv4len` or `net.IPv6len` function, rounded up to the nearest eight-bit value. The offset is calculated as the difference between the netmask and 8, divided by 8, and appended to the generated IP address.

The generated resource also includes the `EDNS0` subnet in its options. This is defined by setting the `Code` field to `EDNS0SUBNET` and the `Data` field to the generated IP address.

Overall, this code appears to be a simple tool for generating options for an EDNS0 subscription based on a given client IP address.


```go
var (
	errRecordNotFound = errors.New("record not found")
)

type dnsRequest struct {
	reqType dnsmessage.Type
	domain  string
	start   time.Time
	expire  time.Time
	msg     *dnsmessage.Message
}

func genEDNS0Options(clientIP net.IP) *dnsmessage.Resource {
	if len(clientIP) == 0 {
		return nil
	}

	var netmask int
	var family uint16

	if len(clientIP) == 4 {
		family = 1
		netmask = 24 // 24 for IPV4, 96 for IPv6
	} else {
		family = 2
		netmask = 96
	}

	b := make([]byte, 4)
	binary.BigEndian.PutUint16(b[0:], family)
	b[2] = byte(netmask)
	b[3] = 0
	switch family {
	case 1:
		ip := clientIP.To4().Mask(net.CIDRMask(netmask, net.IPv4len*8))
		needLength := (netmask + 8 - 1) / 8 // division rounding up
		b = append(b, ip[:needLength]...)
	case 2:
		ip := clientIP.Mask(net.CIDRMask(netmask, net.IPv6len*8))
		needLength := (netmask + 8 - 1) / 8 // division rounding up
		b = append(b, ip[:needLength]...)
	}

	const EDNS0SUBNET = 0x08

	opt := new(dnsmessage.Resource)
	common.Must(opt.Header.SetEDNS0(1350, 0xfe00, true))

	opt.Body = &dnsmessage.OPTResource{
		Options: []dnsmessage.Option{
			{
				Code: EDNS0SUBNET,
				Data: b,
			},
		},
	}

	return opt
}

```

The code you provided is a DNS message generator for creating a DNS query or a DNS response message. It creates a DNS message based on the provided DNS query or response and options.

The code first checks if the IPv4 or IPv6 option is enabled. If IPv4 is enabled, the code creates a DNS message for question A (to retrieve a domain name), with the question being the DNS query. The question is created using the question name, type, and class of the DNS query. The code then generates a DNS response message with the IPv4 response.

If IPv6 is enabled, the code creates a DNS message for question AAAA (to retrieve a domain name), with the question being the DNS query. The question is created using the question name, type, and class of the DNS query. The code then generates a DNS response message with the IPv6 response.

The code then checks if any options are provided for the DNS message generation. If options are provided, the code applies them to the DNS message. Finally, the code returns the DNS messages.

Overall, the code is designed to retrieve a domain name by sending a DNS query or response, based on the provided options.


```go
func buildReqMsgs(domain string, option IPOption, reqIDGen func() uint16, reqOpts *dnsmessage.Resource) []*dnsRequest {
	qA := dnsmessage.Question{
		Name:  dnsmessage.MustNewName(domain),
		Type:  dnsmessage.TypeA,
		Class: dnsmessage.ClassINET,
	}

	qAAAA := dnsmessage.Question{
		Name:  dnsmessage.MustNewName(domain),
		Type:  dnsmessage.TypeAAAA,
		Class: dnsmessage.ClassINET,
	}

	var reqs []*dnsRequest
	now := time.Now()

	if option.IPv4Enable {
		msg := new(dnsmessage.Message)
		msg.Header.ID = reqIDGen()
		msg.Header.RecursionDesired = true
		msg.Questions = []dnsmessage.Question{qA}
		if reqOpts != nil {
			msg.Additionals = append(msg.Additionals, *reqOpts)
		}
		reqs = append(reqs, &dnsRequest{
			reqType: dnsmessage.TypeA,
			domain:  domain,
			start:   now,
			msg:     msg,
		})
	}

	if option.IPv6Enable {
		msg := new(dnsmessage.Message)
		msg.Header.ID = reqIDGen()
		msg.Header.RecursionDesired = true
		msg.Questions = []dnsmessage.Question{qAAAA}
		if reqOpts != nil {
			msg.Additionals = append(msg.Additionals, *reqOpts)
		}
		reqs = append(reqs, &dnsRequest{
			reqType: dnsmessage.TypeAAAA,
			domain:  domain,
			start:   now,
			msg:     msg,
		})
	}

	return reqs
}

```

该代码的作用是从返回的 payloads 中解析 DNS 查询响应。它首先创建一个名为 `parser` 的 `dnsmessage.Parser` 对象，然后使用 `parser.Start` 方法将 payloads 解析为 DNS 查询响应。

如果解析过程中出现错误，函数将返回 ` nil` 和相应的错误信息。如果解析成功，函数将返回一个 `IPRecord` 类型的实例，其中包含 DNS 查询响应的详细信息。

函数还有一个可选的 `now` 参数，用于在解析响应后记录当前时间。


```go
// parseResponse parse DNS answers from the returned payload
func parseResponse(payload []byte) (*IPRecord, error) {
	var parser dnsmessage.Parser
	h, err := parser.Start(payload)
	if err != nil {
		return nil, newError("failed to parse DNS response").Base(err).AtWarning()
	}
	if err := parser.SkipAllQuestions(); err != nil {
		return nil, newError("failed to skip questions in DNS response").Base(err).AtWarning()
	}

	now := time.Now()
	ipRecord := &IPRecord{
		ReqID:  h.ID,
		RCode:  h.RCode,
		Expire: now.Add(time.Second * 600),
	}

```

这段代码是一个 Go 语言中的函数，它接收一个 `parser.AnswerHeader()` 函数的返回值，解析了一个 HTTP 问答（Answer Section）的头部信息（ah.Name），并对其进行了一些处理。

具体来说，这段代码的作用如下：

1. 遍历解析套用了 `parser.AnswerHeader()` 函数的 `ah.Answer` 字段，对于每个 `ah.Answer` 字段，执行以下操作：

a. 解析问句部分（问句是 a 字段的子部分，即问句中的内容）。

b. 如果解析失败，处理错误并记录到日志中。

c. 提取问句解析后得到的 TTL（Time to Live）字段。

d. 如果当前时间距离解析出的 TTL 还过去，更新本地保存的 `ipRecord` 中的 IP 地址。

e. 如果解析出的类型是 dnsmessage.TypeA，则解析 A 记录。

f. 如果解析出的类型是 dnsmessage.TypeAAAA，则解析 AAAA 记录。

g. 如果解析出任何错误，跳过该部分并继续处理下一个。

i. 对于所有解析出的类型，添加到 `ipRecord.IP` 字段中。

j. 在处理完所有 `ah.Answer` 字段后，返回 `ipRecord` 和一个 `nil`，表示已经处理完了所有信息。


```go
L:
	for {
		ah, err := parser.AnswerHeader()
		if err != nil {
			if err != dnsmessage.ErrSectionDone {
				newError("failed to parse answer section for domain: ", ah.Name.String()).Base(err).WriteToLog()
			}
			break
		}

		ttl := ah.TTL
		if ttl == 0 {
			ttl = 600
		}
		expire := now.Add(time.Duration(ttl) * time.Second)
		if ipRecord.Expire.After(expire) {
			ipRecord.Expire = expire
		}

		switch ah.Type {
		case dnsmessage.TypeA:
			ans, err := parser.AResource()
			if err != nil {
				newError("failed to parse A record for domain: ", ah.Name).Base(err).WriteToLog()
				break L
			}
			ipRecord.IP = append(ipRecord.IP, net.IPAddress(ans.A[:]))
		case dnsmessage.TypeAAAA:
			ans, err := parser.AAAAResource()
			if err != nil {
				newError("failed to parse A record for domain: ", ah.Name).Base(err).WriteToLog()
				break L
			}
			ipRecord.IP = append(ipRecord.IP, net.IPAddress(ans.AAAA[:]))
		default:
			if err := parser.SkipAnswer(); err != nil {
				newError("failed to skip answer").Base(err).WriteToLog()
				break L
			}
			continue
		}
	}

	return ipRecord, nil
}

```