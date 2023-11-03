# v2ray-core源码解析 24

# `common/net/address.go`

这段代码定义了几个常量，以及一个IPv4地址类型和一个IPv6地址类型。

常量包括：
- LocalHostIP:IPv4地址，表示本地主机的IP地址。
- AnyIP:IPv4地址，表示任何IP地址。
- LocalHostDomain：域名，表示本地主机的域名。
- LocalHostIPv6:IPv6地址，表示本地主机的IPv6地址。
- AnyIPv6:IPv6地址，表示任何IP地址。

IPv4地址类型包括：
- IPAddress:IPv4地址类型，用于存储IPv4地址。

IPv6地址类型包括：
- IPAddress:IPv6地址类型，用于存储IPv6地址。

这些常量和类型以及IPv4地址类型和IPv6地址类型中的IP地址类型都使用了从net包中导入的命名。


```go
package net

import (
	"bytes"
	"net"
	"strings"
)

var (
	// LocalHostIP is a constant value for localhost IP in IPv4.
	LocalHostIP = IPAddress([]byte{127, 0, 0, 1})

	// AnyIP is a constant value for any IP in IPv4.
	AnyIP = IPAddress([]byte{0, 0, 0, 0})

	// LocalHostDomain is a constant value for localhost domain.
	LocalHostDomain = DomainAddress("localhost")

	// LocalHostIPv6 is a constant value for localhost IP in IPv6.
	LocalHostIPv6 = IPAddress([]byte{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1})

	// AnyIPv6 is a constant value for any IP in IPv6.
	AnyIPv6 = IPAddress([]byte{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0})
)

```

这段代码定义了一个名为AddressFamily的枚举类型，共有三个枚举值：AddressFamilyIPv4、AddressFamilyIPv6和AddressFamilyDomain。每个枚举值对应一个二进制位，用于表示是否是IPv4、IPv6或域名地址。

在const语句中，定义了三个const变量，分别对应AddressFamilyIPv4、AddressFamilyIPv6和AddressFamilyDomain，它们的值分别为AddressFamily(0)、AddressFamily(1)和AddressFamily(2)。

接着，定义了一个常量常数组，包含三个元素，分别是一个字节(char)类型的变量，分别对应AddressFamilyIPv4、AddressFamilyIPv6和AddressFamilyDomain。

然后，定义了一个名为IsIPv4的函数，返回值为true，如果当前的AddressFamily是IPv4，则返回true，否则返回false。

最后，在函数内部，对AddressFamily类型进行枚举，根据当前的AddressFamily值，返回相应的类型说明，例如，如果当前的AddressFamily是AddressFamilyIPv4，则返回"IPv4"。


```go
// AddressFamily is the type of address.
type AddressFamily byte

const (
	// AddressFamilyIPv4 represents address as IPv4
	AddressFamilyIPv4 = AddressFamily(0)

	// AddressFamilyIPv6 represents address as IPv6
	AddressFamilyIPv6 = AddressFamily(1)

	// AddressFamilyDomain represents address as Domain
	AddressFamilyDomain = AddressFamily(2)
)

// IsIPv4 returns true if current AddressFamily is IPv4.
```

这段代码定义了三个名为IsIPv4、IsIPv6和IsDomain的函数，它们都接受一个名为af的参数，并返回一个布尔值。

函数的作用是判断给定的地址族（AddressFamily）是否为IPv4或IPv6，或者是否为Domain。IPv4和IPv6分别使用不同的IP协议，而Domain则表示一个域名。


```go
func (af AddressFamily) IsIPv4() bool {
	return af == AddressFamilyIPv4
}

// IsIPv6 returns true if current AddressFamily is IPv6.
func (af AddressFamily) IsIPv6() bool {
	return af == AddressFamilyIPv6
}

// IsIP returns true if current AddressFamily is IPv6 or IPv4.
func (af AddressFamily) IsIP() bool {
	return af == AddressFamilyIPv4 || af == AddressFamilyIPv6
}

// IsDomain returns true if current AddressFamily is Domain.
```

这段代码定义了一个名为`isAlphaNumeric`的函数，其接收一个字节（即一个八位字符）作为参数，并返回一个布尔值，表示输入的字符是否只包含字母数字，不包括下划线、美元符等其他字符。

接下来，给出了一个名为`func`的函数，该函数接收一个名为`af`的参数，代表一个地址家族（如IPv4地址家族或IPv6地址家族），并返回一个布尔值，表示传入的地址是否属于所指定的地址家族。

另外，还定义了一个名为`Address`的接口类型，该接口类型包含一个`IP`字段和一个`Domain`字段，分别表示一个网络地址和该地址所属的域名。该接口类型还包含一个名为`Family`字段的类型，该字段可以用来判断一个地址是否属于某个特定的地址家族。

最后，给出了一个名为`isDomain`的函数，该函数接收一个名为`af`的参数，并使用地址家族`af`来判断传入的地址是否属于指定的地址家族。


```go
func (af AddressFamily) IsDomain() bool {
	return af == AddressFamilyDomain
}

// Address represents a network address to be communicated with. It may be an IP address or domain
// address, not both. This interface doesn't resolve IP address for a given domain.
type Address interface {
	IP() net.IP     // IP of this Address
	Domain() string // Domain of this Address
	Family() AddressFamily

	String() string // String representation of this Address
}

func isAlphaNum(c byte) bool {
	return (c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z')
}

```

这段代码定义了一个名为 `ParseAddress` 的函数，它接受一个字符串参数 `addr`。函数的作用是解析 `addr` 这个字符串，将其转换为一个 `Address` 类型的结果。

具体来说，这段代码对传入的 `addr` 字符串进行以下处理：

1. 对 IPv6 地址进行处理，将其按照 `"[2001:4860:0:2001::68]"` 的形式进行解析，如果字符串中包含 `"[2001:4860:0:2001::68]"`，则去除开头的 `"` 和结束的 `]`，否则不做处理。
2. 对 IPv4 地址进行处理，如果字符串中包含 `"[2001:4860:0:2001::68]"`，则将其转换为 `IPAddr` 类型，并返回。否则，如果字符串中包含 `"[2001:4860:0:2001::68]"` 和 `"::68"`，则将其去除开头的 `"` 和结束的 `:`，将字符串左半部分转换为 `"[2001:4860:0:2001::68]"` 的形式，再将其转换为 `IPAddr` 类型，并返回。
3. 如果字符串中不包含 IPv6 或 IPv4 地址，则将其转换为 `"-"` 和 `"-"` 两个空字符串，并返回。

函数的返回类型为 `Address`，即根据输入的 `addr` 字符串返回一个 IP 地址或域名地址。


```go
// ParseAddress parses a string into an Address. The return value will be an IPAddress when
// the string is in the form of IPv4 or IPv6 address, or a DomainAddress otherwise.
func ParseAddress(addr string) Address {
	// Handle IPv6 address in form as "[2001:4860:0:2001::68]"
	lenAddr := len(addr)
	if lenAddr > 0 && addr[0] == '[' && addr[lenAddr-1] == ']' {
		addr = addr[1 : lenAddr-1]
		lenAddr -= 2
	}

	if lenAddr > 0 && (!isAlphaNum(addr[0]) || !isAlphaNum(addr[len(addr)-1])) {
		addr = strings.TrimSpace(addr)
	}

	ip := net.ParseIP(addr)
	if ip != nil {
		return IPAddress(ip)
	}
	return DomainAddress(addr)
}

```

这段代码定义了一个名为`IPAddress`的函数，它接收一个字节数组`ip`作为参数，并返回一个`Address`类型的变量`addr`。

`IPAddress`函数根据`ip`字节数组长度，使用不同的网络接口类型进行转换，并返回相应的`Address`类型变量。

具体来说，如果`ip`字节数组长度属于`net.IPv4len`类型，则创建一个`ipv4Address`变量，其值为`[4]byte{ip[0], ip[1], ip[2], ip[3]}`。如果`ip`字节数组长度属于`net.IPv6len`类型，则需要判断`ip`字节数组是否以`0xff`开头，如果是，则创建一个`ipv6Address`变量，其值为`[16]byte{ip[12:16]}`。如果`ip`字节数组长度既不属于`net.IPv4len`类型，也不属于`net.IPv6len`类型，则会输出一个错误信息，并返回`nil`。


```go
var bytes0 = []byte{0, 0, 0, 0, 0, 0, 0, 0, 0, 0}

// IPAddress creates an Address with given IP.
func IPAddress(ip []byte) Address {
	switch len(ip) {
	case net.IPv4len:
		var addr ipv4Address = [4]byte{ip[0], ip[1], ip[2], ip[3]}
		return addr
	case net.IPv6len:
		if bytes.Equal(ip[:10], bytes0) && ip[10] == 0xff && ip[11] == 0xff {
			return IPAddress(ip[12:16])
		}
		var addr ipv6Address = [16]byte{
			ip[0], ip[1], ip[2], ip[3],
			ip[4], ip[5], ip[6], ip[7],
			ip[8], ip[9], ip[10], ip[11],
			ip[12], ip[13], ip[14], ip[15],
		}
		return addr
	default:
		newError("invalid IP format: ", ip).AtError().WriteToLog()
		return nil
	}
}

```

这段代码定义了一个名为DomainAddress的结构体，该结构体创建了一个具有指定域名的新地址。

定义了一个名为ipv4Address的类型，该类型表示一个4字节长的IPv4地址。

接着定义了两个名为ipv4Address的结构体方法，一个名为(a ipv4Address)的切片类型方法，通过该方法访问IPv4地址的IP部分，另一个名为(a ipv4Address)的域名方法，通过该方法获取域名部分。

最后，该代码实现了一个名为DomainAddress的函数，该函数接收一个域名参数，返回一个创建好的DomainAddress类型。


```go
// DomainAddress creates an Address with given domain.
func DomainAddress(domain string) Address {
	return domainAddress(domain)
}

type ipv4Address [4]byte

func (a ipv4Address) IP() net.IP {
	return net.IP(a[:])
}

func (ipv4Address) Domain() string {
	panic("Calling Domain() on an IPv4Address.")
}

```

这段代码定义了两个函数，一个是名为"ipv4Address"，另一个是名为"ipv6Address"。这两个函数的作用分别是返回IPv4地址的"AddressFamily"类型和IPv6地址的"IP"类型。

另外，还定义了一个名为"ipv6Address"的"ipv6Address"类型，它的长度为16字节。

最后，定义了一个名为"func"的函数，这个函数接收一个"ipv4Address"类型的参数，然后返回一个名为"AddressFamily"的类型，该类型包含一个IPv4地址的"AddressFamily"类型。函数接收一个"ipv4Address"类型的参数，然后返回一个字符串类型的"IPv4"类型，该类型包含了IPv4地址的"String"类型。函数接收一个"ipv6Address"类型的参数，然后返回一个名为"net.IP"的"IP"类型，该类型包含了IPv6地址的"String"类型。函数接收一个"ipv6Address"类型的参数，然后返回一个字符串类型的"Domain"类型，该类型使用了"ipv6Address"类型中包含的"IP"类型。


```go
func (ipv4Address) Family() AddressFamily {
	return AddressFamilyIPv4
}

func (a ipv4Address) String() string {
	return a.IP().String()
}

type ipv6Address [16]byte

func (a ipv6Address) IP() net.IP {
	return net.IP(a[:])
}

func (ipv6Address) Domain() string {
	panic("Calling Domain() on an IPv6Address.")
}

```

这段代码定义了两个函数，以及一个枚举类型。

函数1 "func (ipv6Address) Family() AddressFamily {
	return AddressFamilyIPv6
}"，接收一个IPv6地址参数，返回一个地址家族，其中 AddressFamilyIPv6 是IPv6地址所属的地址家族。函数返回的 AddressFamily 是 AddressFamilyIPv6 的别名，用于将IPv6地址转换为地址家族名称。

函数2 "func (a ipv6Address) String() string {
	return "[" + a.IP().String() + "]"
}"，接收一个IPv6地址参数，返回它的字符串表示形式。函数内部调用 a.IP() 函数获取IPv6地址的IP部分，然后将其转换为字符串并返回。

枚举类型 "type domainAddress string"，定义了一个名为 "domainAddress" 的枚举类型，它包含一个名为 "string" 的类型。

函数3 "func (domainAddress) IP() net.IP {
	panic("Calling IP() on a DomainAddress.")
}"，接收一个名为 "domainAddress" 的域地址参数，返回一个网络IPv6地址。函数内部引发一个 "panic" 函数，以确保在非 IPv6 地址条件下，函数能够正确运行。因为域地址并不是一个IPv6地址，所以在尝试获取其IP部分时引发错误。

函数4 "func (a domainAddress) Domain() string {
	return string(a)
}"，接收一个名为 "domainAddress" 的域地址参数，返回其域标识符。函数内部将传入的域地址作为字符串返回。


```go
func (ipv6Address) Family() AddressFamily {
	return AddressFamilyIPv6
}

func (a ipv6Address) String() string {
	return "[" + a.IP().String() + "]"
}

type domainAddress string

func (domainAddress) IP() net.IP {
	panic("Calling IP() on a DomainAddress.")
}

func (a domainAddress) Domain() string {
	return string(a)
}

```

该代码定义了两个函数，分别接收一个 `domainAddress` 参数，并返回 `AddressFamily` 类型的函数值。

第一个函数 `func (domainAddress) Family() AddressFamily` 接收一个 `domainAddress` 参数，然后将其转换为 `AddressFamily` 类型并返回。

第二个函数 `func (a domainAddress) String() string` 接收一个 `domainAddress` 参数，然后将其转换为字符串并返回。

另外，该代码还定义了一个名为 `AsAddress` 的函数，接收一个 `IPOrDomain` 类型的参数，将其转换为 `Address` 类型的函数值，并返回。

最后，该代码还定义了一个名为 `Net_IPOrDomain` 的类型，该类型代表 `IPOrDomain` 类型，通过 `address.Ip` 和 `address.Domain` 字段提供 IP 地址或域名解析功能。


```go
func (domainAddress) Family() AddressFamily {
	return AddressFamilyDomain
}

func (a domainAddress) String() string {
	return a.Domain()
}

// AsAddress translates IPOrDomain to Address.
func (d *IPOrDomain) AsAddress() Address {
	if d == nil {
		return nil
	}
	switch addr := d.Address.(type) {
	case *IPOrDomain_Ip:
		return IPAddress(addr.Ip)
	case *IPOrDomain_Domain:
		return DomainAddress(addr.Domain)
	}
	panic("Common|Net: Invalid address.")
}

```

这段代码定义了一个名为 `NewIPOrDomain` 的函数，它接受一个 `Address` 参数，并将其转换为相应的 IPOrDomain 结构体。

IPOrDomain 是一个用于将 IP 地址转换为域名或 IP 地址的第三方库中的一种类型。

函数内部，根据传入的 `addr.Family()` 函数值，使用 switch 语句来判断 `addr.Family()` 属于哪个地址家族。如果是 `AddressFamilyDomain`，则创建一个 IPOrDomain 对象，该对象包含一个指向域名的指针；如果是 `AddressFamilyIPv4` 或 `AddressFamilyIPv6`，则创建一个 IPOrDomain 对象，该对象包含一个指向 IP 地址的指针。如果无法确定 `addr.Family()` 的值，函数会输出一个错误。

函数的具体实现将根据传递的 `addr` 参数，将其转换为相应的 IPOrDomain 结构体，这样就可以将 IP 地址转换为域名或 IP 地址了。


```go
// NewIPOrDomain translates Address to IPOrDomain
func NewIPOrDomain(addr Address) *IPOrDomain {
	switch addr.Family() {
	case AddressFamilyDomain:
		return &IPOrDomain{
			Address: &IPOrDomain_Domain{
				Domain: addr.Domain(),
			},
		}
	case AddressFamilyIPv4, AddressFamilyIPv6:
		return &IPOrDomain{
			Address: &IPOrDomain_Ip{
				Ip: addr.IP(),
			},
		}
	default:
		panic("Unknown Address type.")
	}
}

```

# `common/net/address.pb.go`

这段代码定义了一个名为 "net" 的包，其中包含一个名为 "address" 的接口类型。这个接口类型定义了一个名为 "net" 的接口，它包含一个名为 "address" 的字段。

同时，该代码定义了一个名为 "addressImpl" 的接口类型，它的作用是实现名为 "address" 的接口。通过实现该接口，可以方便地使用由 "addressImpl" 定义的方法来访问 "address" 接口的属性和方法。

该代码还定义了一个名为 "netImpl" 的接口类型，它的作用是实现名为 "net" 的接口。通过实现该接口，可以方便地使用由 "netImpl" 定义的方法来访问 "net" 接口的属性和方法。

最后，该代码还定义了一个名为 "addressType" 的类型，它的作用是支持 "addressImpl" 和 "netImpl" 引用的任何值，并定义了一个名为 "address" 的字段，它的类型为 "addressImpl" 或 "netImpl"。

通过使用这些接口和类型，可以定义一个 "address" 类型的实例，并使用 "addressImpl" 或 "netImpl" 来访问该实例。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: common/net/address.proto

package net

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码定义了一个名为`IPOrDomain`的结构体类型，以及一个名为`address`的属性。

首先，代码检查当前使用的`proto`包的版本是否足够更新。如果版本过低，代码会执行一个名为`_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)`的函数，这个函数会强制使用`protoimpl`包中定义的最新的`version`。同样，如果版本过高，代码会执行一个名为`_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)`的函数，这个函数会强制使用`protoimpl`包中定义的最旧版本的`version`。

接下来，代码会检查当前使用的`proto`包的版本是否足够支持`address`属性的定义。如果当前版本不支持`address`属性，代码会执行一个名为`const _ = proto.ProtoPackageIsVersion4`的函数，这个函数会检查当前使用的`proto`包是否使用了`proto4`格式的定义。如果当前版本不支持`address`属性，那么这个函数会输出一个警告信息。

最后，代码会创建一个名为`IPOrDomain`的结构体类型的实例，并设置其`address`属性为`IPOrDomain_Address`类型。


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

// Address of a network host. It may be either an IP address or a domain
// address.
type IPOrDomain struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// Types that are assignable to Address:
	//	*IPOrDomain_Ip
	//	*IPOrDomain_Domain
	Address isIPOrDomain_Address `protobuf_oneof:"address"`
}

```

这段代码定义了两个函数，以及一个名为IPOrDomain的类型。这两个函数分别用于设置IPOrDomain类型的参数x，并将其转换为IPOrDomain{}类型。此外，如果定义了"protoimpl.UnsafeEnabled"为真，则会创建一个名为mi的FileCommonNetAddressProtocolMessage类型的指针，并将x的地址存储为该指针的内存位置。最后，通过调用IPOrDomain的String()函数，可以返回一个字符串表示IPOrDomain类型。


```go
func (x *IPOrDomain) Reset() {
	*x = IPOrDomain{}
	if protoimpl.UnsafeEnabled {
		mi := &file_common_net_address_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *IPOrDomain) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*IPOrDomain) ProtoMessage() {}

```

这段代码定义了两个函数，函数一是对IPOrDomain类型的*IPOrDomain实例进行保护函数，函数二是返回IPOrDomain的描述信息。

函数一：
kotlin
func (x *IPOrDomain) ProtoReflect() protoreflect.Message {
	mi := &file_common_net_address_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

这段代码的作用是：
1. 如果IPOrDomain类型的实例被赋值给x，那么执行以下操作：
a. 获取文件中common_net_address_proto_msgTypes数组中的第一个元素，作为MI类型指针的类型声明。
b. 如果使用了`protoimpl.UnsafeEnabled`，那么执行以下操作：
i. 尝试从x中获取IPOrDomain类型的实例，如果当前x实例是有效的，那么执行以下操作：
ii. 从x中获取IPOrDomain类型的实例，然后执行MessageStateOf函数，获取MessageStateOf的返回值，如果返回值有效的，执行以下操作：
a. 从x中获取MessageStateOf的返回值，然后执行MessageStoreOf函数，将MessageStateOf的返回值存储为x的MessageStateOf，如果之前没有设置，执行以下操作：
a. 将MessageStateOf的返回值设置为x的MessageStateOf，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果x的MessageOf被正确初始化，那么执行以下操作：
a. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从x中获取MessageOf的返回值，然后执行以下操作：
a. 如果MessageOf的返回值正确，那么执行以下操作：
a. 将MessageOf的返回值存储为mi，然后执行以下操作：
i. 从


```go
func (x *IPOrDomain) ProtoReflect() protoreflect.Message {
	mi := &file_common_net_address_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use IPOrDomain.ProtoReflect.Descriptor instead.
func (*IPOrDomain) Descriptor() ([]byte, []int) {
	return file_common_net_address_proto_rawDescGZIP(), []int{0}
}

```

该代码定义了两个函数：一个是`func (m *IPOrDomain) GetAddress() isIPOrDomain_Address`，另一个是`func (x *IPOrDomain) GetIp() []byte`。

这两个函数的作用如下：

1. `func (m *IPOrDomain) GetAddress() isIPOrDomain_Address`：该函数接收一个`IPOrDomain`类型的指针变量`m`，并返回`m`所指向对象的`Address`字段。

2. `func (x *IPOrDomain) GetIp() []byte`：该函数接收一个`IPOrDomain`类型的指针变量`x`，并返回`x`所指向对象的`Ip`字段（如果存在的话）。如果`x`所指向的对象的`Address`字段为`nil`，则返回`nil`。


```go
func (m *IPOrDomain) GetAddress() isIPOrDomain_Address {
	if m != nil {
		return m.Address
	}
	return nil
}

func (x *IPOrDomain) GetIp() []byte {
	if x, ok := x.GetAddress().(*IPOrDomain_Ip); ok {
		return x.Ip
	}
	return nil
}

func (x *IPOrDomain) GetDomain() string {
	if x, ok := x.GetAddress().(*IPOrDomain_Domain); ok {
		return x.Domain
	}
	return ""
}

```

这段代码定义了一个名为 `IPOrDomain_Ip` 的接口类型，它包含一个 `Ip` 字段和一个 `domain` 字段。

`Ip` 字段是一个 IP 地址，必须为 4 或 16 字节长。这个字段通过 `isIPOrDomain_Address()` 函数进行判断。

`domain` 字段是一个域名，必须为字符串类型。这个字段通过 `isIPOrDomain_Address()` 函数进行判断。

* `*IPOrDomain_Ip` 是一个指向 `IPOrDomain_Ip` 类型实例的指针类型。
* `isIPOrDomain_Address()` 函数用于判断 `Ip` 字段是否为 IP 地址，如果是 IP 地址，函数会返回 `true`，否则返回 `false`。
* `IPOrDomain_Ip` 类型实例中 `Ip` 字段的长度为 4 或 16，表示它可能是一个 IPv4 或 IPv6 地址。
* `domain` 字段是一个字符串，用于表示域名。


```go
type isIPOrDomain_Address interface {
	isIPOrDomain_Address()
}

type IPOrDomain_Ip struct {
	// IP address. Must by either 4 or 16 bytes.
	Ip []byte `protobuf:"bytes,1,opt,name=ip,proto3,oneof"`
}

type IPOrDomain_Domain struct {
	// Domain address.
	Domain string `protobuf:"bytes,2,opt,name=domain,proto3,oneof"`
}

func (*IPOrDomain_Ip) isIPOrDomain_Address() {}

```

0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
	0x2e, 0x6e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33, 0x2e, 0x6e, 0x65, 0x74,
	0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
	0x2e, 0x6e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33, 0x2e, 0x6e, 0x65, 0x74,
	0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
	0x2e, 0x6e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33, 0x2e, 0x6e, 0x65, 0x74,
	0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
	0x2e, 0x6e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33, 0x2e, 0x6e, 0x65, 0x74,
	0xaa, 0x02, 0x15,


```go
func (*IPOrDomain_Domain) isIPOrDomain_Address() {}

var File_common_net_address_proto protoreflect.FileDescriptor

var file_common_net_address_proto_rawDesc = []byte{
	0x0a, 0x18, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74, 0x2f, 0x61, 0x64, 0x64,
	0x72, 0x65, 0x73, 0x73, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x15, 0x76, 0x32, 0x72, 0x61,
	0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x6e, 0x65,
	0x74, 0x22, 0x43, 0x0a, 0x0a, 0x49, 0x50, 0x4f, 0x72, 0x44, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x12,
	0x10, 0x0a, 0x02, 0x69, 0x70, 0x18, 0x01, 0x20, 0x01, 0x28, 0x0c, 0x48, 0x00, 0x52, 0x02, 0x69,
	0x70, 0x12, 0x18, 0x0a, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x18, 0x02, 0x20, 0x01, 0x28,
	0x09, 0x48, 0x00, 0x52, 0x06, 0x64, 0x6f, 0x6d, 0x61, 0x69, 0x6e, 0x42, 0x09, 0x0a, 0x07, 0x61,
	0x64, 0x64, 0x72, 0x65, 0x73, 0x73, 0x42, 0x50, 0x0a, 0x19, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e,
	0x6e, 0x65, 0x74, 0x50, 0x01, 0x5a, 0x19, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d,
	0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x6e, 0x65, 0x74,
	0xaa, 0x02, 0x15, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x43, 0x6f,
	0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x4e, 0x65, 0x74, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

此代码定义了一个名为file_common_net_address_proto_rawDescGZIP的函数，其作用是将一个名为file_common_net_address_proto_rawDesc的接口类型对象的八字带GZIP压缩后的字节数组返回。

函数的实现包括两个步骤：

1. 调用protoimpl.X.CompressGZIP，对file_common_net_address_proto_rawDescData进行压缩GZIP。函数的return值类型是[]byte，即压缩后的字节数组。
2. 创建一个名为file_common_net_address_proto_msgTypes的数组，将一个名为IPOrDomain的接口类型类型指针和另一个名为v2ray.core.common.net.IPOrDomain的接口类型指针作为其元素。同时，定义一个名为file_common_net_address_proto_goTypes的数组，包含一个IPOrDomain类型的接口指针作为其元素。

这个函数的作用是提供一个将file_common_net_address_proto_rawDesc数据压缩GZIP后返回的功能。它将IPOrDomain类型的接口与v2ray.core.common.net.IPOrDomain类型的接口进行转换，以便在调用函数时可以匹配。


```go
var (
	file_common_net_address_proto_rawDescOnce sync.Once
	file_common_net_address_proto_rawDescData = file_common_net_address_proto_rawDesc
)

func file_common_net_address_proto_rawDescGZIP() []byte {
	file_common_net_address_proto_rawDescOnce.Do(func() {
		file_common_net_address_proto_rawDescData = protoimpl.X.CompressGZIP(file_common_net_address_proto_rawDescData)
	})
	return file_common_net_address_proto_rawDescData
}

var file_common_net_address_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_common_net_address_proto_goTypes = []interface{}{
	(*IPOrDomain)(nil), // 0: v2ray.core.common.net.IPOrDomain
}
```

This is a JavaScript file that implements the initialization of the `file_common_net_address_proto` message in the Go-based protobuf compiler.

The initialization of the `file_common_net_address_proto` message is done by calling the `file_common_net_address_proto_init()` function, which calls the `file_common_net_address_proto_init()` function defined in the `file_common_net_address_proto_.go` file.

The `file_common_net_address_proto_init()` function is responsible for initializing the fields of the `file_common_net_address_proto` message, including the base type information, the one-of message wonders, and the nested message structure.

The initialization of the `file_common_net_address_proto_init()` function uses the `File_common_net_address_proto_.go` file to define the message types, methods, and the number of messages and extensions.


```go
var file_common_net_address_proto_depIdxs = []int32{
	0, // [0:0] is the sub-list for method output_type
	0, // [0:0] is the sub-list for method input_type
	0, // [0:0] is the sub-list for extension type_name
	0, // [0:0] is the sub-list for extension extendee
	0, // [0:0] is the sub-list for field type_name
}

func init() { file_common_net_address_proto_init() }
func file_common_net_address_proto_init() {
	if File_common_net_address_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_common_net_address_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*IPOrDomain); i {
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
	file_common_net_address_proto_msgTypes[0].OneofWrappers = []interface{}{
		(*IPOrDomain_Ip)(nil),
		(*IPOrDomain_Domain)(nil),
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_common_net_address_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_common_net_address_proto_goTypes,
		DependencyIndexes: file_common_net_address_proto_depIdxs,
		MessageInfos:      file_common_net_address_proto_msgTypes,
	}.Build()
	File_common_net_address_proto = out.File
	file_common_net_address_proto_rawDesc = nil
	file_common_net_address_proto_goTypes = nil
	file_common_net_address_proto_depIdxs = nil
}

```

# `common/net/address_test.go`

This appears to be a network packet tracer that performs the following operations:

1. Creates a list of IP addresses and domains with the given input.
2. Transforms each IP address or domain into its corresponding address object.
3. Checks if the IP addresses are valid by using the `IsIP()` method.
4. If the IP addresses are valid, the tracer checks if the domains are present in the network.
5. If the domains are not present in the network, the tracer sets the output to `nil`.
6. The `cmp` package is used to compare the output of the tracer with the expected output.
7. The `for` loop iterates over the `testCases` slice.
8. The `


```go
package net_test

import (
	"net"
	"testing"

	"github.com/google/go-cmp/cmp"

	. "v2ray.com/core/common/net"
)

func TestAddressProperty(t *testing.T) {
	type addrProprty struct {
		IP     []byte
		Domain string
		Family AddressFamily
		String string
	}

	testCases := []struct {
		Input  Address
		Output addrProprty
	}{
		{
			Input: IPAddress([]byte{byte(1), byte(2), byte(3), byte(4)}),
			Output: addrProprty{
				IP:     []byte{byte(1), byte(2), byte(3), byte(4)},
				Family: AddressFamilyIPv4,
				String: "1.2.3.4",
			},
		},
		{
			Input: IPAddress([]byte{
				byte(1), byte(2), byte(3), byte(4),
				byte(1), byte(2), byte(3), byte(4),
				byte(1), byte(2), byte(3), byte(4),
				byte(1), byte(2), byte(3), byte(4),
			}),
			Output: addrProprty{
				IP: []byte{
					byte(1), byte(2), byte(3), byte(4),
					byte(1), byte(2), byte(3), byte(4),
					byte(1), byte(2), byte(3), byte(4),
					byte(1), byte(2), byte(3), byte(4),
				},
				Family: AddressFamilyIPv6,
				String: "[102:304:102:304:102:304:102:304]",
			},
		},
		{
			Input: IPAddress([]byte{
				byte(0), byte(0), byte(0), byte(0),
				byte(0), byte(0), byte(0), byte(0),
				byte(0), byte(0), byte(255), byte(255),
				byte(1), byte(2), byte(3), byte(4),
			}),
			Output: addrProprty{
				IP:     []byte{byte(1), byte(2), byte(3), byte(4)},
				Family: AddressFamilyIPv4,
				String: "1.2.3.4",
			},
		},
		{
			Input: DomainAddress("v2ray.com"),
			Output: addrProprty{
				Domain: "v2ray.com",
				Family: AddressFamilyDomain,
				String: "v2ray.com",
			},
		},
		{
			Input: IPAddress(net.IPv4(1, 2, 3, 4)),
			Output: addrProprty{
				IP:     []byte{byte(1), byte(2), byte(3), byte(4)},
				Family: AddressFamilyIPv4,
				String: "1.2.3.4",
			},
		},
		{
			Input: ParseAddress("[2001:4860:0:2001::68]"),
			Output: addrProprty{
				IP:     []byte{0x20, 0x01, 0x48, 0x60, 0x00, 0x00, 0x20, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x68},
				Family: AddressFamilyIPv6,
				String: "[2001:4860:0:2001::68]",
			},
		},
		{
			Input: ParseAddress("::0"),
			Output: addrProprty{
				IP:     AnyIPv6.IP(),
				Family: AddressFamilyIPv6,
				String: "[::]",
			},
		},
		{
			Input: ParseAddress("[::ffff:123.151.71.143]"),
			Output: addrProprty{
				IP:     []byte{123, 151, 71, 143},
				Family: AddressFamilyIPv4,
				String: "123.151.71.143",
			},
		},
		{
			Input: NewIPOrDomain(ParseAddress("v2ray.com")).AsAddress(),
			Output: addrProprty{
				Domain: "v2ray.com",
				Family: AddressFamilyDomain,
				String: "v2ray.com",
			},
		},
		{
			Input: NewIPOrDomain(ParseAddress("8.8.8.8")).AsAddress(),
			Output: addrProprty{
				IP:     []byte{8, 8, 8, 8},
				Family: AddressFamilyIPv4,
				String: "8.8.8.8",
			},
		},
		{
			Input: NewIPOrDomain(ParseAddress("[2001:4860:0:2001::68]")).AsAddress(),
			Output: addrProprty{
				IP:     []byte{0x20, 0x01, 0x48, 0x60, 0x00, 0x00, 0x20, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x68},
				Family: AddressFamilyIPv6,
				String: "[2001:4860:0:2001::68]",
			},
		},
	}

	for _, testCase := range testCases {
		actual := addrProprty{
			Family: testCase.Input.Family(),
			String: testCase.Input.String(),
		}
		if testCase.Input.Family().IsIP() {
			actual.IP = testCase.Input.IP()
		} else {
			actual.Domain = testCase.Input.Domain()
		}

		if r := cmp.Diff(actual, testCase.Output); r != "" {
			t.Error("for input: ", testCase.Input, ":", r)
		}
	}
}

```

这段代码定义了一个名为 `TestInvalidAddressConvertion` 的测试函数，用于测试 `ParseAddress` 函数在处理无效地址时是否抛出 panic。

具体来说，这段代码实现了一个名为 `panics` 的函数，它会接收一个函数 `f`，并返回一个布尔值 `ret`。在函数内部，它先定义了一个名为 `defer` 的局部变量，然后调用了一个名为 `f` 的函数，最后输出一个布尔值。

接着，定义了一个名为 `testCases` 的数组，其中包含多个名为 `func()` 的函数，它们都会接收一个测试函数 `testCase`，然后执行相应的 `ParseAddress` 函数。

最后，通过一个循环来遍历 `testCases`，对于每个测试函数，如果它返回的布尔值 `ret` 不为 `false`，那么说明测试函数能够正确地处理无效地址，否则就会输出一个错误信息，代码也就没有问题。


```go
func TestInvalidAddressConvertion(t *testing.T) {
	panics := func(f func()) (ret bool) {
		defer func() {
			if r := recover(); r != nil {
				ret = true
			}
		}()
		f()
		return false
	}

	testCases := []func(){
		func() { ParseAddress("8.8.8.8").Domain() },
		func() { ParseAddress("2001:4860:0:2001::68").Domain() },
		func() { ParseAddress("v2ray.com").IP() },
	}
	for idx, testCase := range testCases {
		if !panics(testCase) {
			t.Error("case ", idx, " failed")
		}
	}
}

```

这两段代码测试了Benchmark中的两个函数，分别测试了IPv4地址和IPv6地址的解析是否正确。

具体来说，第一个函数(BenchmarkParseAddressIPv4)对于每个测试用例，先使用B中的参数`8.8.8.8`获取一个IPv4地址，然后使用address.Family()方法获取该地址的家族，最后与IPv4地址的家族进行比较，如果两个家族不同，则会输出"not ipv4"`的错误信息。

第二个函数(BenchmarkParseAddressIPv6)与第一个函数类似，但是测试使用了一个IPv6地址，而不是IPv4地址。通过调用ParseAddress函数获取该地址，然后使用address.Family()方法获取该地址的家族，最后与IPv6地址的家族进行比较，如果两个家族不同，则会输出"not ipv6"`的错误信息。

Benchmark是一个测试框架，通常包含多个测试用例，每个测试用例都会运行一组测试，以验证程序的正确性。


```go
func BenchmarkParseAddressIPv4(b *testing.B) {
	for i := 0; i < b.N; i++ {
		addr := ParseAddress("8.8.8.8")
		if addr.Family() != AddressFamilyIPv4 {
			panic("not ipv4")
		}
	}
}

func BenchmarkParseAddressIPv6(b *testing.B) {
	for i := 0; i < b.N; i++ {
		addr := ParseAddress("2001:4860:0:2001::68")
		if addr.Family() != AddressFamilyIPv6 {
			panic("not ipv6")
		}
	}
}

```

该代码段是一个名为 "BenchmarkParseAddressDomain" 的函数，属于 "func" 函数。它用于测试 "ParseAddress" 函数的正确性。

函数的作用是对于 "ParseAddress" 函数的一个版本，对于每个版本的函数运行一个循环。该循环使用一个名为 "b" 的参数，它是一个 testing 包中的 "B" 类型别名，表示测试场景的数量。

在循环体内，使用 "for" 循环来运行 "ParseAddress" 函数，并将其返回的 "addr" 参数存储在名为 "addr" 的变量中。接下来，使用 "if" 语句检查 "addr.Family()" 的值是否为 "AddressFamilyDomain"，如果是，则运行 "panic" 函数并输出 "not domain" 消息，表明出现了错误。

由于该函数仅包含一个循环，因此它会在每次运行时对不同的测试场景进行测试。


```go
func BenchmarkParseAddressDomain(b *testing.B) {
	for i := 0; i < b.N; i++ {
		addr := ParseAddress("v2ray.com")
		if addr.Family() != AddressFamilyDomain {
			panic("not domain")
		}
	}
}

```

# `common/net/connection.go`

这段代码是一个 Go 语言编写的网络工具中的 build 包，定义了一个名为 "net" 的包。通过引入其他包中的函数和类型，这个 build 包实现了异步连接追踪的功能。

首先，通过 `+build` 标志，这段代码会在编译时将所有用 "+" 开头的依赖关系构建为二进制文件。

接下来，定义了一个名为 "ConnectionOption" 的函数类型，它接收一个具有 "*connection" 类型的参数。这个类型表示一个可以设置连接选项的函数指针。

然后，通过 `!confonly` 标志，这段代码只会在编译时构建二进制文件，而不在运行时。

接着，定义了 "net" 包的其他函数和类型，包括：

1. 通过 `net.Listen` 函数创建一个监听端口为 ":12345" 的服务器。
2. 通过 `net.Listen` 函数创建一个监听端口为 ":6345" 的服务器。
3. 通过 `net.Listen` 函数创建一个监听所有 IP 地址的服务器。
4. 通过 `net.Listen` 函数创建一个监听所有端口的服务器。
5. 通过 `net.Listen` 函数创建一个监听特定端口的服务器。
6. 通过 `net.Listen` 函数删除一个已经存在的服务器。
7. 通过 `net.Listen` 函数获取一个已经存在的服务器。
8. 通过 `net.Listen` 函数获取一个服务器中间的负载均衡策略。
9. 通过 `net.Listen` 函数设置服务器的中间负载均衡策略。
10. 通过 `net.Listen` 函数取消一个已经存在的服务器。

然后，通过定义的 "ConnectionOption" 函数，可以设置连接选项，例如：

1. 通过 `设置最大连接数` 选项，可以指定服务器最大连接数，默认值为 100。
2. 通过 `设置最大空闲时间` 选项，可以指定服务器最大空闲时间，默认值为 30 秒。
3. 通过 `设置最大活跃时间` 选项，可以指定服务器最大活跃时间，默认值为 60 秒。
4. 通过 `设置是否启用跟踪` 选项，可以指定是否启用跟踪用户在网络中的行为，默认值为 false。

最后，通过定义的 "net" 包中的函数，可以实现异步连接追踪的功能。例如：

1. 通过 `make` 函数创建一个连接对象，该对象可以使用 `net.Listen` 函数获取一个监听端口的服务器。
2. 通过 `addHeader` 函数向连接对象添加一个自定义的 HTTP 头，该头包含一些额外的信息，例如：
	* `User-Agent` 字段包含客户端的 UA 信息。
	* `X-Real-IP` 字段包含客户端的 IP 地址信息。
	* `X-Forwarded-For` 字段包含客户端的转发 IP 地址信息。
	* `X-Forwarded-Proto` 字段包含客户端请求的协议类型信息。
	* `Connection-Close` 字段表示客户端与服务器之间的连接关闭。
	* `Connection-Close-Date` 字段表示客户端与服务器之间的连接关闭日期和时间。
	* `Connection-Close-Cause` 字段表示客户端与服务器之间的连接关闭的原因。
	* `Connection-Close-Status` 字段表示客户端与服务器之间的连接关闭的状态信息。
	* `Content-Type` 字段包含客户端发送的请求头中包含的 Content-Type 信息。
	* `Content-Length` 字段包含客户端发送的请求头中包含的 Content-Length 信息。
	* `Parent-Origin` 字段包含客户端发送的请求头中包含的 Parent-Origin 信息。
	* `Strict-Transport-Security` 字段包含客户端发送的请求头中包含的 Strict-Transport-Security 信息。
	* `X-Requested-With` 字段包含客户端发送的请求头中包含的 X-Requested-With 信息。
	* `Create-是为其他服务创建的连接'` 字段表示客户端创建的连接是否为为其他服务创建的连接。
	* `Color` 字段包含颜色代码，用于设置连接的显示颜色。
	* `TLS-Certificate-科`字段包含服务器证书的科


```go
// +build !confonly

package net

import (
	"io"
	"net"
	"time"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/signal/done"
)

type ConnectionOption func(*connection)

```

这段代码定义了三个名为 ConnectionLocalAddr、ConnectionRemoteAddr 和 ConnectionInput 的函数，它们都接收一个名为 ConnectionOption 的参数，并返回一个名为 ConnectionOption 的函数作为结果。

ConnectionLocalAddr 和 ConnectionRemoteAddr 函数分别设置客户端的本地地址和远程地址。在客户端的连接建立后，通过 ConnectionOption 中的 c.local 和 c.remote 可以获取到客户端的本地地址和远程地址。

ConnectionInput 函数用于将客户端的输入输出到缓冲区 c 的 writer 字段上。通过 ConnectionInput 函数，客户端可以向服务器发送输入数据，并将服务器返回的输出数据存储在缓冲区 c 中。


```go
func ConnectionLocalAddr(a net.Addr) ConnectionOption {
	return func(c *connection) {
		c.local = a
	}
}

func ConnectionRemoteAddr(a net.Addr) ConnectionOption {
	return func(c *connection) {
		c.remote = a
	}
}

func ConnectionInput(writer io.Writer) ConnectionOption {
	return func(c *connection) {
		c.writer = buf.NewWriter(writer)
	}
}

```

这三段代码定义了三个名为 ConnectionInputMulti、ConnectionOutput 和 ConnectionOutputMulti 的函数，它们都接收一个名为 writer、reader 和 connection 的参数。

ConnectionInputMulti 和 ConnectionOutputMulti 都返回一个名为 ConnectionOption 的函数类型，而 ConnectionOutput 返回的是一个名为 io.Reader 的函数类型。

这些函数的作用如下：

1. ConnectionInputMulti 和 ConnectionOutputMulti：

	* `func(c *connection)`：将 c 与 writer 构建连接，并将其作为参数传递给 `c.reader`。

2. ConnectionOutput：

	* `func(c *connection)`：将 c 与 reader 构建连接，并返回一个缓冲区缓冲器（`buf.BufferedReader`）和一个缓冲区读者（`buf.NewReader`）。

3. ConnectionOutputMulti：

	* `func(c *connection)`：将 c 与 reader 构建连接，并返回一个缓冲区缓冲器（`buf.BufferedReader`）。


```go
func ConnectionInputMulti(writer buf.Writer) ConnectionOption {
	return func(c *connection) {
		c.writer = writer
	}
}

func ConnectionOutput(reader io.Reader) ConnectionOption {
	return func(c *connection) {
		c.reader = &buf.BufferedReader{Reader: buf.NewReader(reader)}
	}
}

func ConnectionOutputMulti(reader buf.Reader) ConnectionOption {
	return func(c *connection) {
		c.reader = &buf.BufferedReader{Reader: reader}
	}
}

```

这两函数是在Go语言中定义的，它们的作用如下：

1. `ConnectionOutputMultiUDP` 函数接收一个字符读取缓冲区 `buf.Reader`，然后返回一个与给定的 `ConnectionOption` 类型一起使用的函数，该函数将 `buf.Reader` 作为字符读取缓冲区，并使用 `buf.SplitFirstBytes` 函数将缓冲区中的数据分割成多路复用数据包。
2. `ConnectionOnClose` 函数接收一个 `io.Closer`，然后返回一个与给定的 `ConnectionOption` 类型一起使用的函数，该函数将 `io.Closer` 作为 `OnClose` 字段，并在关闭连接时执行该函数。

这些函数的主要目的是让 `读者` 和 `连接` 函数可以一起使用，即使它们具有不同的功能，也可以通过组合使用来实现更复杂的操作。


```go
func ConnectionOutputMultiUDP(reader buf.Reader) ConnectionOption {
	return func(c *connection) {
		c.reader = &buf.BufferedReader{
			Reader:  reader,
			Spliter: buf.SplitFirstBytes,
		}
	}
}

func ConnectionOnClose(n io.Closer) ConnectionOption {
	return func(c *connection) {
		c.onClose = n
	}
}

```

该函数`NewConnection`接受多个`ConnectionOption`作为参数，然后创建一个新的`net.Conn`实例。

该函数创建一个`net.TCPAddr`结构体，其中包含IP地址和端口号。函数的第一个参数`c`是一个`&connection`类型的变量，它被赋值为一个已经完成的`done.New()`操作的返回值。

接下来，函数遍历传递给它的`opts`数组，对每个`ConnectionOption`调用其中的`*opt`函数。这些`ConnectionOption`包括连接服务器端的地址和端口，以及客户端的地址和端口。

最后，函数返回`c`，它是一个`net.Conn`实例，可以用于创建一个TCP连接。


```go
func NewConnection(opts ...ConnectionOption) net.Conn {
	c := &connection{
		done: done.New(),
		local: &net.TCPAddr{
			IP:   []byte{0, 0, 0, 0},
			Port: 0,
		},
		remote: &net.TCPAddr{
			IP:   []byte{0, 0, 0, 0},
			Port: 0,
		},
	}

	for _, opt := range opts {
		opt(c)
	}

	return c
}

```

这是一段 Go 语言中的代码，定义了一个名为 `connection` 的结构体，表示一个网络连接的上下文。

该结构体包含以下成员变量：

* `reader`：一个指向缓冲区 `buf.BufferedReader` 的指针。
* `writer`：一个指向缓冲区 `buf.Writer` 的指针。
* `done`：一个指向 `done.Instance` 的指针。
* `onClose`：一个指向关闭操作的 `io.Closer` 的指针。
* `local`：一个指向本地地址的 `Addr` 类型的指针。
* `remote`：一个指向远程地址的 `Addr` 类型的指针。

`Read` 函数用于从缓冲区读取数据，并返回读取的字节数和错误。

`ReadMultiBuffer` 函数实现了 `buf.Reader.` 接口，用于从缓冲区读取多行数据，并返回一个缓冲区 `buf.MultiBuffer` 和错误。

`connection` 结构体还包含以下方法：

* `Write`：向缓冲区 `writer` 中写入数据。
* `Close`：关闭连接并关闭所有套接字。

这些方法的实现使得 `connection` 成为了一个可以进行数据传输的上下文。


```go
type connection struct {
	reader  *buf.BufferedReader
	writer  buf.Writer
	done    *done.Instance
	onClose io.Closer
	local   Addr
	remote  Addr
}

func (c *connection) Read(b []byte) (int, error) {
	return c.reader.Read(b)
}

// ReadMultiBuffer implements buf.Reader.
func (c *connection) ReadMultiBuffer() (buf.MultiBuffer, error) {
	return c.reader.ReadMultiBuffer()
}

```

这段代码定义了两个函数，一个是 `Write` 函数，用于将 `mb` 缓冲区的字节串写入 `net.Conn` 类型的连接对象 `c` 中，另一个是 `WriteMultiBuffer` 函数，用于将 `mb` 缓冲区的字节串写入 `net.Conn` 类型的连接对象 `c` 中。

具体来说，这两个函数的实现都涉及到以下步骤：

1. 检查 `c` 对象是否已经完成写入操作，如果是，则返回 0 和一个错误 `io.ErrClosedPipe`。
2. 创建一个名为 `mb` 的 `MultiBuffer` 对象，如果 `l` 长度比 `mb` 大，则将 `l` 分解为多个 `mb` 大小的小整数，并将这些小整数合并到 `mb` 对象中。
3. 调用 `c.writer.WriteMultiBuffer` 函数，将 `mb` 对象写入到 `c` 对象的写入缓冲区中。
4. 如果 `c` 对象已经完成写入操作，则释放 `mb` 对象并返回一个错误 `io.ErrClosedPipe`。
5. 如果 `c` 对象还没有完成写入操作，则返回 `c.writer.WriteMultiBuffer` 函数的返回值。


```go
// Write implements net.Conn.Write().
func (c *connection) Write(b []byte) (int, error) {
	if c.done.Done() {
		return 0, io.ErrClosedPipe
	}

	l := len(b)
	mb := make(buf.MultiBuffer, 0, l/buf.Size+1)
	mb = buf.MergeBytes(mb, b)
	return l, c.writer.WriteMultiBuffer(mb)
}

func (c *connection) WriteMultiBuffer(mb buf.MultiBuffer) error {
	if c.done.Done() {
		buf.ReleaseMulti(mb)
		return io.ErrClosedPipe
	}

	return c.writer.WriteMultiBuffer(mb)
}

```

这段代码定义了一个名为`Close`的函数，它属于一个名为`net.Conn`的类。

该函数的作用是关闭与客户端的连接，并尝试关闭套接字驱动程序和数据写入者。如果函数正在使用的套接字已经关闭，它将尝试调用传递给函数的`onClose`函数来关闭套接字。如果`onClose`函数不存在，那么函数将返回一个`nil`值表示成功关闭套接字。

该函数还实现了一个名为`LocalAddr`的函数，它属于一个名为`net.Conn.LocalAddr`的类。

该函数返回一个名为`net.Addr`的类型，它表示客户端的本地地址。

总的来说，该代码片段定义了一个用于关闭网络连接的函数，该函数尝试关闭套接字驱动程序和数据写入者，并允许在关闭套接字时使用传递给函数的`onClose`函数。


```go
// Close implements net.Conn.Close().
func (c *connection) Close() error {
	common.Must(c.done.Close())
	common.Interrupt(c.reader)
	common.Close(c.writer)
	if c.onClose != nil {
		return c.onClose.Close()
	}

	return nil
}

// LocalAddr implements net.Conn.LocalAddr().
func (c *connection) LocalAddr() net.Addr {
	return c.local
}

```

该代码定义了三个函数，分别名为RemoteAddr、SetDeadline和SetReadDeadline，应用于一个名为connection的传输层连接实例。

RemoteAddr函数返回一个Net.Conn.RemoteAddr类型的变量，表示连接的远程地址。

SetDeadline函数接受一个名为t的time.Time类型的参数，表示设置连接超时时间。该函数的实现较为简单，只是简单地返回一个Nil类型的错误，表示设置成功。

SetReadDeadline函数与SetDeadline函数类似，接受一个名为t的time.Time类型的参数，表示设置接收端超时时间。该函数的实现也较为简单，只是简单地返回一个Nil类型的错误，表示设置成功。


```go
// RemoteAddr implements net.Conn.RemoteAddr().
func (c *connection) RemoteAddr() net.Addr {
	return c.remote
}

// SetDeadline implements net.Conn.SetDeadline().
func (c *connection) SetDeadline(t time.Time) error {
	return nil
}

// SetReadDeadline implements net.Conn.SetReadDeadline().
func (c *connection) SetReadDeadline(t time.Time) error {
	return nil
}

```

这段代码定义了一个名为 `SetWriteDeadline` 的函数，它属于一个名为 `net.Conn` 的网络连接类，这个函数接受一个名为 `t` 的参数，表示写入数据的截止时间，返回值类型为 `error` 类型。

在这个函数中，首先定义了一个名为 `c` 的变量，它是一个指向 `connection` 类型对象的指针。然后，定义了一个名为 `SetWriteDeadline` 的函数，它接收一个名为 `t` 的参数，表示写入数据的截止时间，这个函数内部的具体实现可能由其他代码完成。

函数内部没有执行实际的代码，它返回了一个 `nil`，表示调用这个函数不会产生任何返回值。


```go
// SetWriteDeadline implements net.Conn.SetWriteDeadline().
func (c *connection) SetWriteDeadline(t time.Time) error {
	return nil
}

```

# `common/net/destination.go`

这段代码定义了一个名为`Destination`的结构体，用于表示网络中的目标地址，包括地址和协议（TCP或UDP）。

具体来说，这个结构体包含以下字段：

1. `Address`：目标网络地址的字符串表示形式。
2. `Port`：目标网络协议（TCP或UDP）的整数值。
3. `Network`：目标网络，可以是IP地址、IP4或IPTUDPack类型。

`import`语句导入了两个来自`net`包的函数：`net. DestinationFromAddr`和`strings. lowercase`。

1. `import strings. lowercase`：用于将`net.Destination`结构体中`Address`字段的值转换为小写。
2. `import net.DestinationFromAddr`：用于从给定的目标地址中生成`Destination`结构体。

从上面的解释可以看出来，这段代码的主要目的是为了创建一个`Destination`结构体，用于存储目标网络地址和协议，并使其可以被不同函数使用。


```go
package net

import (
	"net"
	"strings"
)

// Destination represents a network destination including address and protocol (tcp / udp).
type Destination struct {
	Address Address
	Port    Port
	Network Network
}

// DestinationFromAddr generates a Destination from a net address.
```

这段代码定义了一个名为 "func DestinationFromAddr" 的函数，它接收一个名为 "addr" 的参数，然后根据 "addr" 的类型来决定如何解析地址并返回相应的目的地址。

函数内部，首先定义了一个名为 "switch" 的switch语句，它接收 "addr" 参数，根据该参数的类型来执行相应的操作。

当 "addr" 是 *net.TCPAddr 时，函数将返回一个名为 "TCPDestination" 的函数，它接收一个名为 "IPAddress" 的参数，然后将收到的地址解析为 IP 地址，并返回一个名为 "TCP" 的类型。

当 "addr" 是 *net.UDPAddr 时，函数将返回一个名为 "UDPDestination" 的函数，它接收一个名为 "IPAddress" 的参数，然后将收到的地址解析为 IP 地址，并返回一个名为 "UDP" 的类型。

当 "addr" 是 *net.UnixAddr 时，函数将会处理 Unix 域套接字，但目前还没有实现 Unix 域套接字的相关代码，因此无法确定其具体行为。

如果 "addr" 无法被正确解析，函数将会崩溃并输出 "Net: Unknown address type." 错误。


```go
func DestinationFromAddr(addr net.Addr) Destination {
	switch addr := addr.(type) {
	case *net.TCPAddr:
		return TCPDestination(IPAddress(addr.IP), Port(addr.Port))
	case *net.UDPAddr:
		return UDPDestination(IPAddress(addr.IP), Port(addr.Port))
	case *net.UnixAddr:
		// TODO: deal with Unix domain socket
		return TCPDestination(LocalHostIP, Port(9))
	default:
		panic("Net: Unknown address type.")
	}
}

// ParseDestination converts a destination from its string presentation.
```

此代码定义了一个名为 "ParseDestination" 的函数，其接收一个名为 "dest" 的字符串参数作为Destination结构体类型的返回值。函数的作用是解析Destination结构体的地址和端口号，并将其存储在d结构体中。

以下是函数的实现步骤：

1. 检查给定的dest字符串是否以"tcp:"开头。如果是，则将d结构体的Network设置为Network_TCP，将dest字符串的后续部分作为destination参数。
2. 如果dest字符串不以"tcp:"开头，则继续检查是否以"udp:"开头。如果是，则将d结构体的Network设置为Network_UDP，将dest字符串的后续部分作为destination参数。
3. 如果dest字符串既不是以"tcp:"开头，也不是以"udp:"开头，则函数会尝试从给定的dest字符串中提取出主机名和端口号。如果 extraction 成功，则将d结构体的Address设置为主机名，将d结构体的Port设置为端口号。如果extraction 失败，则返回d结构体，错误则返回d结构体，错误信息则返回。
4. 最后，函数返回d结构体，如果没有错误，则返回函数。


```go
func ParseDestination(dest string) (Destination, error) {
	d := Destination{
		Address: AnyIP,
		Port:    Port(0),
	}
	if strings.HasPrefix(dest, "tcp:") {
		d.Network = Network_TCP
		dest = dest[4:]
	} else if strings.HasPrefix(dest, "udp:") {
		d.Network = Network_UDP
		dest = dest[4:]
	}

	hstr, pstr, err := SplitHostPort(dest)
	if err != nil {
		return d, err
	}
	if len(hstr) > 0 {
		d.Address = ParseAddress(hstr)
	}
	if len(pstr) > 0 {
		port, err := PortFromString(pstr)
		if err != nil {
			return d, err
		}
		d.Port = port
	}
	return d, nil
}

```

这两段代码定义了TCP和UDP Destination结构体，用于创建TCP和UDP目标的接口。这些Destination结构体包含一个IP地址和一个端口号，用于连接到远程主机。

在函数内部，根据传入的地址和端口号，创建一个适当的TCP或UDP Destination 实例。这些实例都使用Network_TCP或Network_UDP作为网络接口，并包含一个IP地址和一个端口号。

TCPDestination创建一个TCP目标实例，并使用address作为IP地址，port作为端口号。UDPDestination创建一个UDP目标实例，并使用address作为IP地址，port作为端口号。

这些函数可以被用来创建TCP和UDP目标的实例，以便在应用程序中进行网络通信。


```go
// TCPDestination creates a TCP destination with given address
func TCPDestination(address Address, port Port) Destination {
	return Destination{
		Network: Network_TCP,
		Address: address,
		Port:    port,
	}
}

// UDPDestination creates a UDP destination with given address
func UDPDestination(address Address, port Port) Destination {
	return Destination{
		Network: Network_UDP,
		Address: address,
		Port:    port,
	}
}

```

这两段代码定义了一个名为`NetAddr`的函数，它的输入参数`d`是一个`Destination`类型的变量，返回值为该`d`的`Address`字段和`Port`字段的字符串形式连接。

具体来说，这两段代码实现了一个`NetAddr`函数，它接收一个`Destination`类型的参数`d`，并返回一个字符串形式的网络地址。函数的第一个实现将`d`的`Address`字段和`Port`字段的字符串连接起来，然后将`d.Address`和`d.Port`提取出来，最后将它们连接在一起返回。第二个实现则根据`d.Network`的类型来定义一个字符串前缀，然后将`d.Address`和`d.Port`根据网络类型连接起来，最后将结果的前缀加上`d.NetAddr()`的结果，得到一个字符串形式的网络地址。


```go
// NetAddr returns the network address in this Destination in string form.
func (d Destination) NetAddr() string {
	return d.Address.String() + ":" + d.Port.String()
}

// String returns the strings form of this Destination.
func (d Destination) String() string {
	prefix := "unknown:"
	switch d.Network {
	case Network_TCP:
		prefix = "tcp:"
	case Network_UDP:
		prefix = "udp:"
	}
	return prefix + d.NetAddr()
}

```

这两段代码定义了一个名为 `IsValid` 的函数和一个名为 `AsDestination` 的函数。

`IsValid` 函数接收一个名为 `d` 的 `Destination` 类型的参数，并返回一个布尔值。判断依据是 `d.Network` 是否为 `Network_Unknown`。如果 `d.Network` 不为 `Network_Unknown`, 则返回 `true`；否则返回 `false`。

`AsDestination` 函数接收一个名为 `p` 的 `Endpoint` 类型的参数，并返回一个 `Destination` 类型的参数。转换方式是将 `p` 的 `Network` 类型赋给 `d.Network`，将 `p.Address` 的 `AsAddress` 函数返回的 `Address` 类型赋给 `d.Address`，将 `p.Port` 的值赋给 `d.Port`。这样， `p` 的 `Endpoint` 类型就转换成了 `d` 的 `Destination` 类型。

换句话说，`AsDestination` 函数的作用是将 `Endpoint` 类型的 `p` 转换为 `Destination` 类型的 `d`，以便在函数调用时使用。


```go
// IsValid returns true if this Destination is valid.
func (d Destination) IsValid() bool {
	return d.Network != Network_Unknown
}

// AsDestination converts current Endpoint into Destination.
func (p *Endpoint) AsDestination() Destination {
	return Destination{
		Network: p.Network,
		Address: p.Address.AsAddress(),
		Port:    Port(p.Port),
	}
}

```