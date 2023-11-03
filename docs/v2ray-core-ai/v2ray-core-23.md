# v2ray-core源码解析 23

# `common/mux/errors.generated.go`

这段代码定义了一个名为 `mux` 的包，其中包含了一个名为 `errPathObjHolder` 的类型，以及一个名为 `newError` 的函数。

`errPathObjHolder` 是一个匿名类型，其中包含一个空括号 `{}`，这个类型被称为 "errPathObjHolder"，用于表示一个对象，这个对象中包含一个 `errors.Error` 类型的类型和一个包含 `values...interface{}` 的字面量。这个类型表示一个 "错误路径对象"，其中包含一个错误对象和一个包含多个 `interface{}` 的字面量，这个字面量指定了这个错误对象的一些元数据，如错误类型、错误消息和错误堆栈等。

`newError` 函数接收多个参数，这些参数中包含一个 `error` 类型和一个包含 `values...interface{}` 的字面量。这个函数创建一个新的 `errors.Error` 类型实例，其中包含一个包含 `values...interface{}` 的字面量，以及一个指向 `errPathObjHolder` 类型的引用，这个引用用于获取错误对象的元数据。然后，这个函数使用 `WithPathObj` 方法将错误对象的元数据与 `errPathObjHolder` 类型的对象合并，这样就可以在错误对象上设置错误路径对象。最后，这个函数返回一个新的 `errors.Error` 类型实例，其中包含一个指向 `errPathObjHolder` 类型的引用，以及原始的错误消息和错误堆栈。


```go
package mux

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `common/mux/frame.go`

这段代码定义了一个名为 "mux" 的包，其中定义了一个名为 "SessionStatus" 的字节类型变量。

"SessionStatus" 类型表示一个会话的状态，可以是已连接、待连接、正在交互、已结束等状态。具体来说，这个字节的值为以下人体可见的 ASCII 字符：

- P: 表示已连接
- I: 表示正在交互
- E: 表示已结束
- X: 表示未知或无效状态

这个包可能是一个用于网络会话的库，根据 ASCII 字符的编码，这个包可能用于 HTTP/HTTPS 连接，也可能是用于其他用途的会话管理库。


```go
package mux

import (
	"encoding/binary"
	"io"

	"v2ray.com/core/common"
	"v2ray.com/core/common/bitmask"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
)

type SessionStatus byte

```

这段代码定义了一个结构体 `TargetNetwork`，该结构体有两个字节，分别表示目标网络是使用 TCP 协议还是 UDP 协议。

接着，定义了一个名为 `SessionStatus` 的枚举类型，包含四个具体的枚举值，分别表示网络会话的状态，分别为 `SessionStatusNew`、`SessionStatusKeep`、`SessionStatusEnd` 和 `SessionStatusKeepAlive`。每个状态位的含义如下：

- `0x01`：会话已创建，但未连接到服务器
- `0x02`：正在保持与服务器的连接，但未发送或接收数据
- `0x03`：已经关闭会话，或服务器已释放连接资源
- `0x04`：保持连接，正在尝试重新建立或重新连接服务器

然后，定义了一个名为 `OptionData` 的枚举类型，包含两个字节，具体是 `bitmask.Byte` 类型。接着，定义了一个 `TargetNetwork` 类型，该类型包含两个字节，具体是 `TargetNetworkTCP` 或 `TargetNetworkUDP`。

最后，定义了一个名为 `UserOption` 的结构体，包含一个 `OptionData` 字段和一个 `TargetNetwork` 字段，具体是 `OptionData` 和 `TargetNetwork` 的组合。


```go
const (
	SessionStatusNew       SessionStatus = 0x01
	SessionStatusKeep      SessionStatus = 0x02
	SessionStatusEnd       SessionStatus = 0x03
	SessionStatusKeepAlive SessionStatus = 0x04
)

const (
	OptionData  bitmask.Byte = 0x01
	OptionError bitmask.Byte = 0x02
)

type TargetNetwork byte

const (
	TargetNetworkTCP TargetNetwork = 0x01
	TargetNetworkUDP TargetNetwork = 0x02
)

```

这段代码的作用是创建一个新的地址解析器，用于解析IPv4和IPv6地址。它允许您将IPv4和IPv6地址解析为相应的协议地址，并返回其下一个跳转到该地址的传输层协议（如TCP或UDP）。

具体来说，这段代码通过使用`protocol.NewAddressParser`函数创建一个新的地址解析器。这个函数接收四个参数：

1. `protocol.AddressFamilyByte`函数表示地址家族，它将下一跳的地址转换为相应的协议类型。第一个参数表示IPv4地址，第二个参数表示IPv6地址。
2. `net.AddressFamilyIPv4`和`net.AddressFamilyIPv6`函数分别表示IPv4和IPv6地址的地址家族。
3. `protocol.PortThenAddress`函数表示协议类型（如TCP或UDP）。
4. `()`是一个空括号，用于将上述参数组合在一起。

创建地址解析器后，您可以使用它来解析IPv4和IPv6地址，并获取下一跳地址。这使得您可以根据需要调整网络传输中的数据包，如更改IP头部或重新组装数据包。


```go
var addrParser = protocol.NewAddressParser(
	protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv4), net.AddressFamilyIPv4),
	protocol.AddressFamilyByte(byte(protocol.AddressTypeDomain), net.AddressFamilyDomain),
	protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv6), net.AddressFamilyIPv6),
	protocol.PortThenAddress(),
)

/*
Frame format
2 bytes - length
2 bytes - session id
1 bytes - status
1 bytes - option

1 byte - network
```

这段代码定义了一个名为FrameMetadata的结构体，用于表示数据帧的元数据。这个结构体包含以下字段：

- Target：目标IP地址或网络套接字
- Address：目标地址
- Option：选项数据(如源IP地址，校验和等)
- SessionID：会话ID，用于标识发送数据帧的会话
- SessionStatus：会话状态，用于指示数据帧的发送状态

数据帧的发送需要通过一个传输缓冲区(buf)来实现，这个缓冲区需要先扩展大小以包含元数据长度，然后将元数据按照正确的格式编码，最后将数据帧发送出去。

具体实现中，首先定义了一个FrameMetadata结构体，其中包含了要发送的数据帧的各种元数据。然后，定义了一个WriteTo函数，用于将数据帧发送出去。函数接收一个传输缓冲区(b)，然后将数据帧中的各种元数据按照格式编码后，逐一发送出去。

发送数据帧需要先获取目标IP地址或网络套接字，然后建立目标网络的TCP或UDP套接字。接着，解析目标地址的地址套接字，并发送出去。如果发送过程中出现错误，函数会返回错误。

整个函数的实现要点包括：

1. 定义了FrameMetadata结构体，以及WriteTo函数，用于将数据帧发送出去。
2. 创建了一个缓冲区(b)，并将其扩展到适当的大小，以包含元数据长度。
3. 解析目标地址的地址套接字，并将其发送出去。
4. 发送数据帧中的选项数据，包括会话ID和选项数据。
5. 将数据帧发送出去。


```go
2 bytes - port
n bytes - address

*/

type FrameMetadata struct {
	Target        net.Destination
	SessionID     uint16
	Option        bitmask.Byte
	SessionStatus SessionStatus
}

func (f FrameMetadata) WriteTo(b *buf.Buffer) error {
	lenBytes := b.Extend(2)

	len0 := b.Len()
	sessionBytes := b.Extend(2)
	binary.BigEndian.PutUint16(sessionBytes, f.SessionID)

	common.Must(b.WriteByte(byte(f.SessionStatus)))
	common.Must(b.WriteByte(byte(f.Option)))

	if f.SessionStatus == SessionStatusNew {
		switch f.Target.Network {
		case net.Network_TCP:
			common.Must(b.WriteByte(byte(TargetNetworkTCP)))
		case net.Network_UDP:
			common.Must(b.WriteByte(byte(TargetNetworkUDP)))
		}

		if err := addrParser.WriteAddressPort(b, f.Target.Address, f.Target.Port); err != nil {
			return err
		}
	}

	len1 := b.Len()
	binary.BigEndian.PutUint16(lenBytes, uint16(len1-len0))
	return nil
}

```

这段代码的作用是实现了一个名为 `Unmarshal` 的函数，该函数接收一个 `io.Reader` 类型的参数，用于从给定的数据源中读取 `FrameMetadata` 类型数据。它将数据解码为 `FrameMetadata` 类型，并返回给调用者。

具体来说，这段代码实现了一个 `Unmarshal` 函数，该函数从给定的数据源中读取字节数据。首先，它使用 `serial.ReadUint16` 函数从数据源中读取一个 16 字节的金属读取器。然后，它检查金属读取器中的数据长度是否超过 512 个字节，如果是，它将返回一个错误并停止执行。

接下来，它创建了一个名为 `b` 的缓冲区并从数据源中读取字节数据。然后，它使用 `b.ReadFullFrom` 函数将字节数据解码为 `FrameMetadata` 类型。最后，它使用 `f.UnmarshalFromBuffer` 函数将 `FrameMetadata` 类型数据解码为 `FrameMetadata` 类型，并返回给调用者。

总之，这段代码实现了一个 `Unmarshal` 函数，用于从给定的数据源中读取 `FrameMetadata` 类型数据，并将其解码为 `FrameMetadata` 类型。


```go
// Unmarshal reads FrameMetadata from the given reader.
func (f *FrameMetadata) Unmarshal(reader io.Reader) error {
	metaLen, err := serial.ReadUint16(reader)
	if err != nil {
		return err
	}
	if metaLen > 512 {
		return newError("invalid metalen ", metaLen).AtError()
	}

	b := buf.New()
	defer b.Release()

	if _, err := b.ReadFullFrom(reader, int32(metaLen)); err != nil {
		return err
	}
	return f.UnmarshalFromBuffer(b)
}

```

这段代码的作用是读取一个FrameMetadata类型的数据，并将其从指定的缓冲区中解码。以下是代码的功能解释：

1. 函数参数：一个代表FrameMetadata类型的指针b和一个代表缓冲区缓冲王的指针b作为参数。
2. 如果缓冲区的长度小于4，函数会返回一个错误，因为必须要有4个字节的数据才能解码FrameMetadata。
3. 从缓冲区中读取前4个字节，并将其转换为FrameMetadata结构。
4. 然后根据SessionStatus的值来设置后续操作，包括目标网络类型和地址。
5. 如果SessionStatus为SessionStatusNew，那么从缓冲区中继续读取8个字节，然后使用addrParser函数解析出目标地址和端口，最后设置Target字段为相应的网络类型。
6. 如果在函数中遇到错误，返回相应的错误信息。


```go
// UnmarshalFromBuffer reads a FrameMetadata from the given buffer.
// Visible for testing only.
func (f *FrameMetadata) UnmarshalFromBuffer(b *buf.Buffer) error {
	if b.Len() < 4 {
		return newError("insufficient buffer: ", b.Len())
	}

	f.SessionID = binary.BigEndian.Uint16(b.BytesTo(2))
	f.SessionStatus = SessionStatus(b.Byte(2))
	f.Option = bitmask.Byte(b.Byte(3))
	f.Target.Network = net.Network_Unknown

	if f.SessionStatus == SessionStatusNew {
		if b.Len() < 8 {
			return newError("insufficient buffer: ", b.Len())
		}
		network := TargetNetwork(b.Byte(4))
		b.Advance(5)

		addr, port, err := addrParser.ReadAddressPort(nil, b)
		if err != nil {
			return newError("failed to parse address and port").Base(err)
		}

		switch network {
		case TargetNetworkTCP:
			f.Target = net.TCPDestination(addr, port)
		case TargetNetworkUDP:
			f.Target = net.UDPDestination(addr, port)
		default:
			return newError("unknown network type: ", network)
		}
	}

	return nil
}

```

# `common/mux/frame_test.go`

这段代码是针对mux_test包进行测试的Benchmark框架，用于测试框架中的FrameWrite函数。

具体来说，这段代码会创建一个mux.FrameMetadata对象，该对象包含了将要发送的数据帧的信息。然后，会创建一个buf.New()的缓冲区和一个用于存储数据帧的writer。接下来，使用一个循环来将mux.FrameMetadata对象中的数据写入缓冲区，并清空writer。最后，将数据帧发送到目标服务器。

这段代码的作用是作为mux_test包的测试框架的一部分，用于测试FrameWrite函数的正确性。具体测试包括：将数据帧写入缓冲区，清空writer，然后发送数据帧到目标服务器并验证数据是否正确发送与接收。


```go
package mux_test

import (
	"testing"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/mux"
	"v2ray.com/core/common/net"
)

func BenchmarkFrameWrite(b *testing.B) {
	frame := mux.FrameMetadata{
		Target:        net.TCPDestination(net.DomainAddress("www.v2ray.com"), net.Port(80)),
		SessionID:     1,
		SessionStatus: mux.SessionStatusNew,
	}
	writer := buf.New()
	defer writer.Release()

	for i := 0; i < b.N; i++ {
		common.Must(frame.WriteTo(writer))
		writer.Clear()
	}
}

```

# `common/mux/mux.go`

这段代码是一个 Go 语言的开源库 `mux` 的部分源代码。它包含了一个名为 `errordir` 的函数，该函数可能会生成一个错误目录。函数使用了 `go generate` 命令来生成文件。同时，该函数使用了 `v2ray.com/core/common/errors/errorgen` 导入了一个名为 `errorgen` 的错误自定义枚举类型。


```go
package mux

//go:generate go run v2ray.com/core/common/errors/errorgen

```

# `common/mux/mux_test.go`

这段代码是一个Mux测试框架的包，其中包含了导入、函数声明以及一些常量定义。

1. `import ( "io" "testing" )`: 导入了一些 testing 包以及 io 包。

2. `import ( "github.com/google/go-cmp/cmp" )`: 导入了一个名为 go-cmp 的库，它是一个用于测试的依赖库。

3. `import "v2ray.com/core/common/buf" "v2ray.com/core/common/mux" "v2ray.com/core/common/net" "v2ray.com/core/common/protocol" "v2ray.com/core/transport/pipe" )`: 导入了一些 common 包以及 pipe 包。

4. `package mux_test`: 定义了该包的名称。

5. `import "mux_test.test" "mux_test.test/event" "mux_test.test/transport" "mux_test.test/stream" "mux_test.test/sub" "mux_test.test/text" "mux_test.test/values"`: 导入了一些 test 包以及 event、transport、stream、sub、text 和 values 包。

6. `func Test穿越隧道建立连接(t *testing.T) {`: 定义了一个名为 Test 穿越隧道建立连接 的函数，用于测试 mux 测试框架的功能。

7. `test_connect *.example.com:8888`: 定义了一个测试用例，用于测试 mux 测试框架穿越隧道建立连接的功能，该测试用例调用了 Test 穿越隧道建立连接 函数，并传入了 "*.example.com:8888"。

8. `test_send_data *"测试数据"`: 定义了一个测试用例，用于测试 mux 测试框架发送数据的功能，该测试用例调用了 Test 发送数据 函数，并传入了 "测试数据"。

9. `test_send_empty_data *"\0"`: 定义了一个测试用例，用于测试 mux 测试框架发送空数据的功能，该测试用例调用了 Test 发送数据 函数，并传入了 "\0"。

10. `test_flush *"测试目标"`: 定义了一个测试用例，用于测试 mux 测试框架flush 函数的功能，该测试用例调用了 Test flush 函数，并传入了 "测试目标"。

11. `test_run *"测试目标"`: 定义了一个测试用例，用于测试 mux 测试框架run 函数的功能，该测试用例调用了 Test 运行 函数，并传入了 "测试目标"。

12. `test_run_with_options *"-R指定端口号：指定机器码"`: 定义了一个测试用例，用于测试 mux 测试框架run 函数的带选项功能，该测试用例调用了 Test 运行 函数，并传入了 "-R指定端口号：指定机器码"。

13. `test_run_with_ip *"测试目标：测试目标IP"`: 定义了一个测试用例，用于测试 mux 测试框架run 函数的带 IP 选项功能，该测试用例调用了 Test 运行 函数，并传入了 "测试目标：测试目标IP"。

14. `test_transport_ pipe`: 定义了一个测试用例，用于测试 mux 测试框架transport 和 pipe 组合函数的功能，该测试用例调用了 Test 穿越隧道建立连接 函数。


```go
package mux_test

import (
	"io"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	. "v2ray.com/core/common/mux"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/transport/pipe"
)

```

这段代码定义了一个名为 `readAll` 的函数，它接受一个名为 `reader` 的输入参数和一个名为 `buf` 的输出参数 `buf.MultiBuffer`。它的作用是读取输入参数 `reader` 中的所有内容，并将其存储在输出参数 `buf` 的 `MultiBuffer` 字段中。

函数的实现采用了以下几个步骤：

1. 初始化输出参数 `mb` 为 `buf.MultiBuffer`。
2. 使用 `for` 循环，每次读取输入参数 `reader` 中的一个 `MultiBuffer` 类型的数据。
3. 如果读取到 `EOF` 错误，循环就结束，不需要再读取数据。
4. 如果读取过程中遇到错误，返回 `nil` 和错误信息，使得函数能够正常退出。
5. 返回 `mb` 和 `nil`，分别代表读取成功和没有错误。

由于 `MultiBuffer` 是 `buf` 包中的一个 `MultiBuffer` 类型，所以函数 `readAll` 的实现主要关注如何从输入参数 `reader` 中读取数据，并将其存储到输出参数 `buf` 的 `MultiBuffer` 字段中。


```go
func readAll(reader buf.Reader) (buf.MultiBuffer, error) {
	var mb buf.MultiBuffer
	for {
		b, err := reader.ReadMultiBuffer()
		if err == io.EOF {
			break
		}
		if err != nil {
			return nil, err
		}
		mb = append(mb, b...)
	}
	return mb, nil
}

```

This appears to be a function written in the Go programming language that compares two FramesMetadata objects. The FramesMetadata object is a data structure that represents the metadata of a video stream, including information about the session (ID), status, and option.

The function takes two parameters, a reader and a writer, and writes the data from the reader to the writer. The comparison is based on whether the data from the writer is present in the data from the reader. If the data from the writer is present, the function will compare the data from the writer to the data from the reader and will print an error if there are any differences. If the data from the writer is not present, the function will compare the data from the reader to the default data from the reader and will print an error if there are any differences.

The function uses the compareMeasure video framework, which is designed to compare two identical data structures. The compareMeasure framework provides a macarons operation ( comprehension operator) to compare two strings, a seek operation to compare two slices, and a transposition operation to compare two maps.


```go
func TestReaderWriter(t *testing.T) {
	pReader, pWriter := pipe.New(pipe.WithSizeLimit(1024))

	dest := net.TCPDestination(net.DomainAddress("v2ray.com"), 80)
	writer := NewWriter(1, dest, pWriter, protocol.TransferTypeStream)

	dest2 := net.TCPDestination(net.LocalHostIP, 443)
	writer2 := NewWriter(2, dest2, pWriter, protocol.TransferTypeStream)

	dest3 := net.TCPDestination(net.LocalHostIPv6, 18374)
	writer3 := NewWriter(3, dest3, pWriter, protocol.TransferTypeStream)

	writePayload := func(writer *Writer, payload ...byte) error {
		b := buf.New()
		b.Write(payload)
		return writer.WriteMultiBuffer(buf.MultiBuffer{b})
	}

	common.Must(writePayload(writer, 'a', 'b', 'c', 'd'))
	common.Must(writePayload(writer2))

	common.Must(writePayload(writer, 'e', 'f', 'g', 'h'))
	common.Must(writePayload(writer3, 'x'))

	writer.Close()
	writer3.Close()

	common.Must(writePayload(writer2, 'y'))
	writer2.Close()

	bytesReader := &buf.BufferedReader{Reader: pReader}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     1,
			SessionStatus: SessionStatusNew,
			Target:        dest,
			Option:        OptionData,
		}); r != "" {
			t.Error("metadata: ", r)
		}

		data, err := readAll(NewStreamReader(bytesReader))
		common.Must(err)
		if s := data.String(); s != "abcd" {
			t.Error("data: ", s)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionStatus: SessionStatusNew,
			SessionID:     2,
			Option:        0,
			Target:        dest2,
		}); r != "" {
			t.Error("meta: ", r)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     1,
			SessionStatus: SessionStatusKeep,
			Option:        1,
		}); r != "" {
			t.Error("meta: ", r)
		}

		data, err := readAll(NewStreamReader(bytesReader))
		common.Must(err)
		if s := data.String(); s != "efgh" {
			t.Error("data: ", s)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     3,
			SessionStatus: SessionStatusNew,
			Option:        1,
			Target:        dest3,
		}); r != "" {
			t.Error("meta: ", r)
		}

		data, err := readAll(NewStreamReader(bytesReader))
		common.Must(err)
		if s := data.String(); s != "x" {
			t.Error("data: ", s)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     1,
			SessionStatus: SessionStatusEnd,
			Option:        0,
		}); r != "" {
			t.Error("meta: ", r)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     3,
			SessionStatus: SessionStatusEnd,
			Option:        0,
		}); r != "" {
			t.Error("meta: ", r)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     2,
			SessionStatus: SessionStatusKeep,
			Option:        1,
		}); r != "" {
			t.Error("meta: ", r)
		}

		data, err := readAll(NewStreamReader(bytesReader))
		common.Must(err)
		if s := data.String(); s != "y" {
			t.Error("data: ", s)
		}
	}

	{
		var meta FrameMetadata
		common.Must(meta.Unmarshal(bytesReader))
		if r := cmp.Diff(meta, FrameMetadata{
			SessionID:     2,
			SessionStatus: SessionStatusEnd,
			Option:        0,
		}); r != "" {
			t.Error("meta: ", r)
		}
	}

	pWriter.Close()

	{
		var meta FrameMetadata
		err := meta.Unmarshal(bytesReader)
		if err == nil {
			t.Error("nil error")
		}
	}
}

```

# `common/mux/reader.go`

这段代码定义了一个名为`PacketReader`的结构体，表示一个读取Mux包数据的方式。

首先，它导入了三个`io`包，分别用于输入数据。

接着，它定义了一个名为`PacketReader`的`PacketReader`结构体，它包含一个`io.Reader`类型的字段`reader`和一个布尔类型的字段`eof`，表示是否读取结束。

然后，它定义了一个`PacketReader`的构造函数，通过`reader`字段传递一个`io.Reader`，并初始化`eof`为`false`。

最后，它没有做其他事情，直接返回了一个`PacketReader`类型的变量。


```go
package mux

import (
	"io"

	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/crypto"
	"v2ray.com/core/common/serial"
)

// PacketReader is an io.Reader that reads whole chunk of Mux frames every time.
type PacketReader struct {
	reader io.Reader
	eof    bool
}

```

这段代码定义了一个名为NewPacketReader的函数，它创建一个新的PacketReader。函数接受一个io.Reader类型的参数，表示一个可以读取数据的字符串。函数返回一个指向PacketReader类型的引用，其中包含一个reader和一个布尔值表示是否已经读取到数据的结尾。

函数内部还有一个名为ReadMultiBuffer的函数，它实现了buf.Reader接口。这个函数接受一个PacketReader类型的参数，表示一个已经创建好的PacketReader实例。函数首先检查reader是否已经读取到数据结尾，如果是，则返回一个nil值表示出错。否则，函数读取数据并创建一个buf.MultiBuffer类型的缓冲区，并返回它。最后，函数设置reader.eof为true，表示已经读取到了数据结尾。

整个函数的作用是创建一个可以读取数据PacketReader实例，并提供了一个方法ReadMultiBuffer，用于从io.Reader中读取数据并创建buf.MultiBuffer类型的缓冲区。


```go
// NewPacketReader creates a new PacketReader.
func NewPacketReader(reader io.Reader) *PacketReader {
	return &PacketReader{
		reader: reader,
		eof:    false,
	}
}

// ReadMultiBuffer implements buf.Reader.
func (r *PacketReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	if r.eof {
		return nil, io.EOF
	}

	size, err := serial.ReadUint16(r.reader)
	if err != nil {
		return nil, err
	}

	if size > buf.Size {
		return nil, newError("packet size too large: ", size)
	}

	b := buf.New()
	if _, err := b.ReadFullFrom(r.reader, int32(size)); err != nil {
		b.Release()
		return nil, err
	}
	r.eof = true
	return buf.MultiBuffer{b}, nil
}

```

这段代码定义了一个名为`NewStreamReader`的函数，它接受一个名为`reader`的`buf.BufferedReader`类型的参数，并返回一个名为`NewStreamReader`的函数，该函数将`reader`作为参数传递给`crypto.NewChunkStreamReaderWithChunkCount`函数，并将第二个参数设置为`1`。

具体来说，这段代码的作用是创建一个新的`StreamReader`，并将其分配给参数`reader`，该参数是一个`buf.BufferedReader`类型，它可以在读取数据时自动分配内存。

`crypto.NewChunkStreamReaderWithChunkCount`函数用于创建一个新的`ChunkStreamReader`，该函数的第一个参数是一个`PlainChunkSizeParser`类型的`WithChunkCount`选项，它指定了在每次读取数据时需要读取多少个连续的`Chunk`。第二个参数是一个`buf.Reader`类型的`reader`，它指定了要读取的数据的来源。

最终，`NewStreamReader`函数返回一个新的`buf.Reader`类型，该类型将`reader`作为参数传递给`crypto.NewChunkStreamReaderWithChunkCount`函数，并将返回值赋给`reader`，从而允许调用者使用`NewStreamReader`函数来读取数据。


```go
// NewStreamReader creates a new StreamReader.
func NewStreamReader(reader *buf.BufferedReader) buf.Reader {
	return crypto.NewChunkStreamReaderWithChunkCount(crypto.PlainChunkSizeParser{}, reader, 1)
}

```

# `common/mux/server.go`

这段代码是一个名为 "mux" 的包，它定义了一些用于与 V2Ray 服务器建立连接的函数和变量。

首先，它导入了 "v2ray.com/core" 包，其中包括一些用于连接到服务器的函数和变量。

然后，它导入了 "io" 包，以及 "package mux" 声明，这是一种将 "mux" 包的数据存储到 "io" 包中的方式。

接下来，它导入了 "v2ray.com/core/common/buf" 包，其中包括一些缓冲区操作的函数和变量。

然后，它导入了 "v2ray.com/core/common/errors" 包，其中包括一些错误处理的方法和变量。

接着，它导入了 "v2ray.com/core/common/log" 包，其中包括一些将错误信息输出到 "log" 函数中的方法。

然后，它导入了 "v2ray.com/core/common/net" 包，其中包括一些网络相关的函数和变量。

接着，它导入了 "v2ray.com/core/common/protocol" 包，其中包括一些协议相关的函数和变量。

然后，它导入了 "v2ray.com/core/common/session" 包，其中包括一些会话相关的函数和变量。

接着，它导入了 "v2ray.com/core/features/routing" 包，其中包括一些路由相关的函数和变量。

然后，它导入了 "v2ray.com/core/transport" 包，其中包括一些传输相关的函数和变量。

接着，它导入了 "v2ray.com/core/transport/pipe" 包，其中包括一些管道相关的函数和变量。

最后，它定义了一个名为 "connect" 的函数，它使用 "session" 函数创建一个会话，然后使用 "connect" 函数将连接建立起来。

另外，它定义了一个名为 "disconnect" 的函数，它使用 "session" 函数创建一个会话，然后使用 "disconnect" 函数断开连接。


```go
package mux

import (
	"context"
	"io"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/log"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/session"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport"
	"v2ray.com/core/transport/pipe"
)

```

这段代码定义了一个名为Server的结构体，其中包含一个名为dispatcher的引用，以及一个名为NewServer的函数。

NewServer函数的目的是创建一个新的Server实例。它首先创建一个Server类型的变量s，然后调用RequireFeatures函数来设置其dispatcher为d。最后，它返回s。

另外，还定义了一个名为Type的函数，该函数返回Server类型的dispatcher类型。通过这个函数，可以像使用普通类型一样使用Server类型的变量。


```go
type Server struct {
	dispatcher routing.Dispatcher
}

// NewServer creates a new mux.Server.
func NewServer(ctx context.Context) *Server {
	s := &Server{}
	core.RequireFeatures(ctx, func(d routing.Dispatcher) {
		s.dispatcher = d
	})
	return s
}

// Type implements common.HasType.
func (s *Server) Type() interface{} {
	return s.dispatcher.Type()
}

```

这段代码定义了一个名为 Dispatch 的函数，该函数实现了 routing.Dispatcher。

该函数接收两个参数：一个上下文上下文和一个目标 URI。如果目标 URI 与服务器使用的 Mux 库中的 coolAddress 不匹配，则该函数将调用服务器中实现了 Dispatcher 的函数，并将其返回。如果匹配，则创建一个名为 pipe 的通道对象，并使用上下文上下文中的选项设置其读写模式。然后，使用上下文上下文中的选项创建一个新服务器的worker实例，该实例的 reader用于接收 downlinkReader 中的数据并将其传递给 downlinkWriter，而 writer 用于将 uplinkWriter 中的数据发送到 uplinkReader。最后，函数返回一个指向实现了 Dispatcher 的函数的指针，如果可能返回 nil。


```go
// Dispatch implements routing.Dispatcher
func (s *Server) Dispatch(ctx context.Context, dest net.Destination) (*transport.Link, error) {
	if dest.Address != muxCoolAddress {
		return s.dispatcher.Dispatch(ctx, dest)
	}

	opts := pipe.OptionsFromContext(ctx)
	uplinkReader, uplinkWriter := pipe.New(opts...)
	downlinkReader, downlinkWriter := pipe.New(opts...)

	_, err := NewServerWorker(ctx, s.dispatcher, &transport.Link{
		Reader: uplinkReader,
		Writer: downlinkWriter,
	})
	if err != nil {
		return nil, err
	}

	return &transport.Link{Reader: downlinkReader, Writer: uplinkWriter}, nil
}

```

以上代码定义了一个名为ServerWorker的类，该类实现了 common.Runnable 和 common.Closable 接口。

具体来说，该代码创建了一个名为ServerWorker的实例，并实现了以下两个方法：

1. Start()：该方法用于启动服务器工作进程。它的实现中没有做任何实际的逻辑，只是返回了一个nil值，表示运行成功。

2. Close()：该方法用于关闭服务器工作进程。它的实现中也没有做任何实际的逻辑，只是返回了一个nil值，表示关闭成功。

该类还包含一个名为dispatcher的成员变量，该变量是一个实现了 routing.Dispatcher 的类型，用于处理客户端请求的分发。还包含一个名为link的成员变量，该变量是一个实现了 transport.Link 的类型，用于在客户端和服务器之间传递数据。还包含一个名为sessionManager的成员变量，该变量是一个实现了 SessionManager 的类型，用于管理服务器会话。


```go
// Start implements common.Runnable.
func (s *Server) Start() error {
	return nil
}

// Close implements common.Closable.
func (s *Server) Close() error {
	return nil
}

type ServerWorker struct {
	dispatcher     routing.Dispatcher
	link           *transport.Link
	sessionManager *SessionManager
}

```

该代码定义了一个名为 `NewServerWorker` 的函数，用于创建一个服务器工作者实例并将其提交给 `routing.Dispatcher`。

函数接受三个参数：

- `ctx`：上下文信息，用于在函数内部调用 `worker.run` 函数。
- `d`：一个 `routing.Dispatcher` 类型，用于将 `link` 参数传递给 `worker.run` 函数。
- `link`：一个 `transport.Link` 类型，用于将服务器工作者实例返回给 `NewServerWorker` 函数。

函数内部创建了一个 `ServerWorker` 类型实例，并将其 `dispatcher` 字段设置为传递给 `worker.run` 函数的 `d` 参数，将 `link` 字段设置为传递给 `worker.run` 函数的 `link` 参数。

接下来，函数内部创建了一个 `SessionManager` 类型的实例，并将其设置为 `dispatcher` 字段的 `SessionManager` 字段。

最后，函数内部调用 `worker.run` 函数，并返回服务器工作者实例和调用该函数的错误。如果出现错误，函数将返回一个非空 `error` 类型。


```go
func NewServerWorker(ctx context.Context, d routing.Dispatcher, link *transport.Link) (*ServerWorker, error) {
	worker := &ServerWorker{
		dispatcher:     d,
		link:           link,
		sessionManager: NewSessionManager(),
	}
	go worker.run(ctx)
	return worker, nil
}

func handle(ctx context.Context, s *Session, output buf.Writer) {
	writer := NewResponseWriter(s.ID, output, s.transferType)
	if err := buf.Copy(s.input, writer); err != nil {
		newError("session ", s.ID, " ends.").Base(err).WriteToLog(session.ExportIDToError(ctx))
		writer.hasError = true
	}

	writer.Close()
	s.Close()
}

```

这段代码定义了两个函数：

1. `ActiveConnections()`函数返回服务器工作区（ServerWorker）中当前活动的连接数量。函数的实现使用了`w.sessionManager.Size()`，它返回的是当前连接总数，因为每个连接都有一个独立的连接状态，所以这个函数返回的是连接总数，而不是活跃连接数。

2. `Closed()`函数返回服务器工作区（ServerWorker）当前的状态是否为关闭。函数的实现直接使用了`w.sessionManager.Closed()`，它返回了一个布尔值，表示当前连接是否关闭。

3. `handleStatusKeepAlive(meta, reader)`函数接收一个`FrameMetadata`类型的参数`meta`和一个`buf.BufferedReader`类型的参数`reader`。函数实现了一个从客户端缓冲区（buf）读取数据到服务器输出流（可能为空）的递归操作。函数首先检查`meta`是否设置了一个`OptionData`选项，如果是，那么函数使用`buf.Copy(NewStreamReader(reader), buf.Discard)`将数据从客户端缓冲区（buf）拷贝到服务器输出流（可能为空）。否则，函数返回一个`nil`表示错误。


```go
func (w *ServerWorker) ActiveConnections() uint32 {
	return uint32(w.sessionManager.Size())
}

func (w *ServerWorker) Closed() bool {
	return w.sessionManager.Closed()
}

func (w *ServerWorker) handleStatusKeepAlive(meta *FrameMetadata, reader *buf.BufferedReader) error {
	if meta.Option.Has(OptionData) {
		return buf.Copy(NewStreamReader(reader), buf.Discard)
	}
	return nil
}

```

This function handle the "status:new" event that is emitted by the ServerWorker's dispatcher. It takes in a context.Context, a FrameMetadata object, and a BufferedReader to handle the event.

It does the following:

1. Writing the error message to the log, using the current session ID, and the name of the operation that caused the error.

2. Setting up the message for the event, if the incoming request has a valid source.

3. Writing the event to the log, using the current session ID and the status of the operation (received and not expired yet).

4. Logging the event using the current session ID and the status of the operation, using the log.AccessMessage method, which writes the message to the log, and using the log.AccessAccepted reason.

5. If the incoming request has an UDP source, the function is setting the transfer type to packet, so that the function can properly handle the packet data.

6. Adding the new session to the session manager and the function's handle function, if it is not already done, using the session.


```go
func (w *ServerWorker) handleStatusNew(ctx context.Context, meta *FrameMetadata, reader *buf.BufferedReader) error {
	newError("received request for ", meta.Target).WriteToLog(session.ExportIDToError(ctx))
	{
		msg := &log.AccessMessage{
			To:     meta.Target,
			Status: log.AccessAccepted,
			Reason: "",
		}
		if inbound := session.InboundFromContext(ctx); inbound != nil && inbound.Source.IsValid() {
			msg.From = inbound.Source
			msg.Email = inbound.User.Email
		}
		ctx = log.ContextWithAccessMessage(ctx, msg)
	}
	link, err := w.dispatcher.Dispatch(ctx, meta.Target)
	if err != nil {
		if meta.Option.Has(OptionData) {
			buf.Copy(NewStreamReader(reader), buf.Discard)
		}
		return newError("failed to dispatch request.").Base(err)
	}
	s := &Session{
		input:        link.Reader,
		output:       link.Writer,
		parent:       w.sessionManager,
		ID:           meta.SessionID,
		transferType: protocol.TransferTypeStream,
	}
	if meta.Target.Network == net.Network_UDP {
		s.transferType = protocol.TransferTypePacket
	}
	w.sessionManager.Add(s)
	go handle(ctx, s, w.link.Writer)
	if !meta.Option.Has(OptionData) {
		return nil
	}

	rr := s.NewReader(reader)
	if err := buf.Copy(rr, s.output); err != nil {
		buf.Copy(rr, buf.Discard)
		common.Interrupt(s.input)
		return s.Close()
	}
	return nil
}

```

该函数`handleStatusKeep`接收一个`FrameMetadata`对象和一个`BufferedReader`对象作为参数。它的作用是处理`SessionMetadata`对象中的`Option.Get(OptionData)`方法。

如果`Option.Get(OptionData)`不存在，函数返回一个`Nil`值。否则，它尝试从`sessionManager`实例中获取具有相同`SessionID`的会话，如果失败则执行以下操作：

1. 通知远程对端会关闭当前会话。
2. 从`sessionManager`中读取新的`Reader`对象，并将其复制到`s.output`中。
3. 如果从`output`中写入数据时出现错误，函数将抛出并记录错误信息。
4. 从`Reader`对象中读取数据并将其复制到`rr`中。
5. 如果从`Reader`中读取数据时出现错误，函数将关闭输入并关闭会话。
6. 如果`input`仍然存在且没有正在读取，函数将关闭会话并返回错误信息。

函数的实现基于以下假设：

- `w`是一个`ServerWorker`实例。
- `meta`是一个`FrameMetadata`对象。
- `reader`是一个`BufferedReader`对象。
- `sessionManager`是一个实现了`SessionManager`接口的对象，用于管理会话。
- `closingWriter`是一个实现了`ResponseWriter`接口的对象，用于在客户端写入数据时通知远程对端关闭会话。
- `buf`是一个实现了`Buffer`接口的对象，用于管理输入和输出流。
- `protocol.TransferTypeStream`是一个实现了`TransferTypeStream`接口的类型，用于表示数据传输类型为流。
- ` common.Interrupt`是一个实现了`Interrupt`接口的对象，用于在`sessionManager`的`input`上中断并关闭会话。
- `s.Close`会在会话关闭时调用，确保所有输出流都已关闭。


```go
func (w *ServerWorker) handleStatusKeep(meta *FrameMetadata, reader *buf.BufferedReader) error {
	if !meta.Option.Has(OptionData) {
		return nil
	}

	s, found := w.sessionManager.Get(meta.SessionID)
	if !found {
		// Notify remote peer to close this session.
		closingWriter := NewResponseWriter(meta.SessionID, w.link.Writer, protocol.TransferTypeStream)
		closingWriter.Close()

		return buf.Copy(NewStreamReader(reader), buf.Discard)
	}

	rr := s.NewReader(reader)
	err := buf.Copy(rr, s.output)

	if err != nil && buf.IsWriteError(err) {
		newError("failed to write to downstream writer. closing session ", s.ID).Base(err).WriteToLog()

		// Notify remote peer to close this session.
		closingWriter := NewResponseWriter(meta.SessionID, w.link.Writer, protocol.TransferTypeStream)
		closingWriter.Close()

		drainErr := buf.Copy(rr, buf.Discard)
		common.Interrupt(s.input)
		s.Close()
		return drainErr
	}

	return err
}

```

这两函数接收一个Worker实例和一个BufferedReader实例，并输出分别处理了一个FrameMetadata和一个FrameMetadata，然后关闭了BufferedReader。然后，根据传入的SessionStatus，这两函数执行不同的操作。

具体来说，当SessionStatus为SessionStatusKeepAlive时，这两函数分别执行了以下操作：

- 调用handleStatusKeepAlive函数，传递一个FrameMetadata和一个BufferedReader，返回一个没有错误的结果。
- 如果前一个操作中出现了错误，则执行common.Interrupt函数来中断这些实体的输入和输出，然后关闭BufferedReader。
- 如果SessionStatus为SessionStatusEnd，则执行handleStatusEnd函数，传递一个FrameMetadata和一个BufferedReader，返回一个没有错误的结果。
- 如果SessionStatus为SessionStatusNew，则执行handleStatusNew函数，传递一个新的SessionMetadata和一个BufferedReader，如果没有错误，则返回一个没有错误的结果。如果错误，则返回一个错误。
- 如果SessionStatus为SessionStatusKeep，则这两函数执行handleStatusKeep函数，传递一个FrameMetadata和一个BufferedReader，如果没有错误，则返回一个没有错误的结果。如果错误，则返回一个错误。


```go
func (w *ServerWorker) handleStatusEnd(meta *FrameMetadata, reader *buf.BufferedReader) error {
	if s, found := w.sessionManager.Get(meta.SessionID); found {
		if meta.Option.Has(OptionError) {
			common.Interrupt(s.input)
			common.Interrupt(s.output)
		}
		s.Close()
	}
	if meta.Option.Has(OptionData) {
		return buf.Copy(NewStreamReader(reader), buf.Discard)
	}
	return nil
}

func (w *ServerWorker) handleFrame(ctx context.Context, reader *buf.BufferedReader) error {
	var meta FrameMetadata
	err := meta.Unmarshal(reader)
	if err != nil {
		return newError("failed to read metadata").Base(err)
	}

	switch meta.SessionStatus {
	case SessionStatusKeepAlive:
		err = w.handleStatusKeepAlive(&meta, reader)
	case SessionStatusEnd:
		err = w.handleStatusEnd(&meta, reader)
	case SessionStatusNew:
		err = w.handleStatusNew(ctx, &meta, reader)
	case SessionStatusKeep:
		err = w.handleStatusKeep(&meta, reader)
	default:
		status := meta.SessionStatus
		return newError("unknown status: ", status).AtError()
	}

	if err != nil {
		return newError("failed to process data").Base(err)
	}
	return nil
}

```

这段代码定义了一个名为 `func` 的函数，接收一个名为 `ServerWorker` 的参数 `w`，并使用 `run` 函数将其应用于 `ctx` 上下文。函数的作用是在 `ctx` 上下文中执行操作，并在操作完成后返回。

以下是 `func` 函数的实现细节：

1. 从输入中读取数据并将其存储在 `input` 变量中，然后使用一个名为 `buf.BufferedReader` 的类从输入中逐行读取数据。
2. 为了确保在 `ctx.Done()` 事件发生时，函数能够正常返回，即使在输入还没有完成时，也要使用 `defer` 关键字来关闭 `w.sessionManager`。
3. 循环从 `input` 中读取数据，并在读取完成后，根据不同的情况采取不同的操作：
	* 如果当前操作发生错误，则执行相应的错误处理并尝试关闭 `input`。如果错误是由于 `io.EOF` 导致的，那么使用 `newError` 函数并将其添加到 `common.Interrupt` 函数中，同时使用 `common.Interrupt` 函数停止读入。
	* 如果当前操作没有发生错误，则执行 `w.handleFrame` 函数并将读取到的数据传递给它。如果在这个过程中出现错误，使用 `w.handleFrame` 函数处理错误，并检查错误是否是由于 `io.EOF` 导致的。如果是，那么使用 `newError` 函数并将其添加到 `common.Interrupt` 函数中，同时使用 `common.Interrupt` 函数停止读入。


```go
func (w *ServerWorker) run(ctx context.Context) {
	input := w.link.Reader
	reader := &buf.BufferedReader{Reader: input}

	defer w.sessionManager.Close() // nolint: errcheck

	for {
		select {
		case <-ctx.Done():
			return
		default:
			err := w.handleFrame(ctx, reader)
			if err != nil {
				if errors.Cause(err) != io.EOF {
					newError("unexpected EOF").Base(err).WriteToLog(session.ExportIDToError(ctx))
					common.Interrupt(input)
				}
				return
			}
		}
	}
}

```

# `common/mux/session.go`

这段代码定义了一个名为 `SessionManager` 的 `Session` 类型，以及一个名为 `SessionManager` 的 `SessionManager` 结构体。这个 `SessionManager` 结构体包含一个 `sync.RWMutex` 类型的 `sessionMutex`，一个 `map[uint16]*Session` 类型的 `sessions` 字段，一个 `uint16` 类型的 `count` 字段，和一个 `bool` 类型的 `closed` 字段。

`SessionManager` 的 `sync.RWMutex` 类型的 `sessionMutex` 类型表示，这个 `SessionManager` 中的每个会话都是互斥的，并且可以同时读写。`sessions` 字段是一个 `map[uint16]*Session` 类型的字段，表示每个会话对应着一个 `Session` 对象，并且存储了一个 `Session` 对象的引用。`count` 字段是一个 `uint16` 类型的字段，表示当前会话的数量。`closed` 字段是一个 `bool` 类型的字段，表示当前会话是否关闭。

`SessionManager` 的 `Session` 类型是一个简单的 `v2ray.com/core/common/protocol` 包中的 `*Session` 类型定义的。这个 `Session` 类型包含了基本的会话同步、加密、校验等功能。


```go
package mux

import (
	"sync"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/protocol"
)

type SessionManager struct {
	sync.RWMutex
	sessions map[uint16]*Session
	count    uint16
	closed   bool
}

```

这段代码定义了一个名为SessionManager的类，其作用是管理会话实例。具体来说，它实现了以下几个方法：

1. NewSessionManager：返回一个新的SessionManager对象，这个对象包含以下字段：
	* count：当前会话实例的数量
	* sessions：一个哈希 map，用于存储会话实例，键是会话ID，值是会话实例

2. Closed：返回一个布尔值，表示当前会话是否已经关闭。

3. Size：返回当前会话实例的数量。

4. RLock：对会话实例的RW锁进行锁定，直到当前会话结束。

5. RUnlock：解锁当前会话实例的RW锁。

6.其主要方法：
	* NewSessionManager：创建一个新的SessionManager对象，包含count和sessions字段。
	* Closed：返回一个布尔值，表示当前会话是否已经关闭。
	* Size：返回当前会话实例的数量。
	* RLock：对当前会话实例的RW锁进行锁定。
	* RUnlock：解锁当前会话实例的RW锁。


```go
func NewSessionManager() *SessionManager {
	return &SessionManager{
		count:    0,
		sessions: make(map[uint16]*Session, 16),
	}
}

func (m *SessionManager) Closed() bool {
	m.RLock()
	defer m.RUnlock()

	return m.closed
}

func (m *SessionManager) Size() int {
	m.RLock()
	defer m.RUnlock()

	return len(m.sessions)
}

```

这两段代码定义了两个函数，分别作用于一个名为SessionManager的接口上。

第一个函数名为Count，函数接收一个SessionManager类型的参数m，并返回m中count成员的整数类型。函数使用了两个重要的技巧：首先，使用m.RLock()方法对m进行加锁操作，确保在函数内部对m的修改不会影响其他人对m的访问；其次，在函数内部使用m.RUnlock()方法释放锁，以避免因为重复加锁而导致的资源浪费。

第二个函数名为Allocate，函数同样接收一个SessionManager类型的参数m，并返回一个指向Session类型的指针。函数使用了m.Lock()方法对m进行加锁操作，确保在函数内部对m的修改不会影响其他人对m的访问；然后使用m.Unlock()方法释放锁。函数逻辑比较简单，它仅仅是在判断m是否为空，如果是，就返回一个空的Session类型；否则，就创建一个新的Session对象，并将它加入到m.sessions映射中。最后，函数返回新创建的Session对象。


```go
func (m *SessionManager) Count() int {
	m.RLock()
	defer m.RUnlock()

	return int(m.count)
}

func (m *SessionManager) Allocate() *Session {
	m.Lock()
	defer m.Unlock()

	if m.closed {
		return nil
	}

	m.count++
	s := &Session{
		ID:     m.count,
		parent: m,
	}
	m.sessions[s.ID] = s
	return s
}

```

这两函数是`SessionManager`的私有函数，主要作用是管理会话。

`Add`函数接收一个`Session`对象`s`，首先对`m`进行锁定，确保在函数内部对`m`进行修改不会被其他函数同时访问。

然后 `m`的计数器`count`自增，同时将`s`的ID存储在`m.sessions`中。最后，如果`m`是关闭的（即已经释放），则不需要做任何操作，直接返回。

`Remove`函数与`Add`函数类似，首先对`m`进行锁定，确保在函数内部对`m`进行修改不会被其他函数同时访问。

然后从`m.sessions`中删除ID为`id`的`Session`对象。由于`m.sessions`中没有找到`id`，因此需要将`id`对应的`Session`对象从`m.sessions`中删除。

接着，由于删除了一个`Session`对象，所以需要将`m.sessions`中的键的数量减一。如果`m.sessions`为空，需要将`make`函数创建一个空字典，大小为16。

最后，函数返回，不会做任何其他操作。


```go
func (m *SessionManager) Add(s *Session) {
	m.Lock()
	defer m.Unlock()

	if m.closed {
		return
	}

	m.count++
	m.sessions[s.ID] = s
}

func (m *SessionManager) Remove(id uint16) {
	m.Lock()
	defer m.Unlock()

	if m.closed {
		return
	}

	delete(m.sessions, id)

	if len(m.sessions) == 0 {
		m.sessions = make(map[uint16]*Session, 16)
	}
}

```

这两函数是`SessionManager`的封装函数，主要作用于`SessionManager`的实例变量上。

1. `Get`函数的作用是获取`SessionManager`中某个`Session`对象的引用，并返回其值。具体实现方式如下：

	1. 对`m`实例变量进行加锁操作，确保在调用`RLock`函数之后，当前函数仍可以访问`m`实例变量的引用。
	2. 使用`defer`关键字，确保在调用`RUnlock`函数之后，当前函数仍可以访问`m`实例变量。
	3. 在函数内部，首先判断`m`实例变量是否为`closed`状态。如果是，直接返回`nil`和`false`，表示没有找到匹配的`Session`对象，并且`SessionManager`也没有关闭。
	4. 如果`m`实例变量不是`closed`状态，那么就需要进一步判断`m`实例变量中`sessions`数组是否为空，如果是，表示当前`SessionManager`实例没有打开任何会话，那么需要关闭`m`实例变量，返回`false`。否则，就返回`sessions`数组中已有的`Session`对象的引用，以及判断`m`实例变量是否为`closed`状态，如果是，返回`true`。

2. `CloseIfNoSession`函数的作用是关闭`SessionManager`，具体实现方式如下：

	1. 对`m`实例变量进行加锁操作，确保在调用`Lock`函数之后，当前函数仍可以访问`m`实例变量。
	2. 使用`defer`关键字，确保在调用`Unlock`函数之后，当前函数仍可以访问`m`实例变量。
	3. 在函数内部，首先判断`m`实例变量是否为`closed`状态。如果是，直接返回`true`，表示`SessionManager`已经关闭。
	4. 如果`m`实例变量不是`closed`状态，那么就需要对`m`实例变量进行解锁操作，并且关闭`SessionManager`实例。


```go
func (m *SessionManager) Get(id uint16) (*Session, bool) {
	m.RLock()
	defer m.RUnlock()

	if m.closed {
		return nil, false
	}

	s, found := m.sessions[id]
	return s, found
}

func (m *SessionManager) CloseIfNoSession() bool {
	m.Lock()
	defer m.Unlock()

	if m.closed {
		return true
	}

	if len(m.sessions) != 0 {
		return false
	}

	m.closed = true
	return true
}

```

此代码定义了一个名为 `func` 的函数，其参数为 `m *SessionManager`，表示一个指向 `SessionManager` 类型的指针变量 `m`。函数的作用是关闭会话管理器 `m`。

具体来说，函数首先获取会话管理器 `m` 的锁，然后使用 `defer` 关键字释放锁。接下来，函数将 `m` 的 `closed` 字段设置为 `true`，表示会话管理器已经处于关闭状态。然后，函数遍历 `m` 管理的所有会话，并调用 `common.Close` 函数关闭会话的输入和输出。注意，由于使用了 `nolint: errcheck` 注解，因此在函数内部不会输出任何错误信息。

最后，函数将 `m` 的 `sessions` 字段设置为 `nil`，表示不再管理任何会话。函数本身不会返回任何错误信息，但会话管理器 `m` 在调用此函数时可能会尝试关闭会话，此时函数可能会返回非零的错误信息。


```go
func (m *SessionManager) Close() error {
	m.Lock()
	defer m.Unlock()

	if m.closed {
		return nil
	}

	m.closed = true

	for _, s := range m.sessions {
		common.Close(s.input)  // nolint: errcheck
		common.Close(s.output) // nolint: errcheck
	}

	m.sessions = nil
	return nil
}

```

这段代码定义了一个名为Session的结构体，表示一个客户端连接到服务器的Mux连接中的会话。

会话包含以下字段：

- input：一个缓冲区Reader，用于接收数据并将其发送到服务器。
- output：一个缓冲区Writer，用于将数据从服务器接收并将其发送到客户端。
- parent：一个SessionManager实例，表示会话的父会话。
- ID：会话的唯一ID。
- transferType：用于数据传输的协议TransferType类型。

此外，还包含两个方法：

- Close：关闭与会话相关的所有资源，并确保所有缓冲区都已经被关闭。
- 传递给Go标准库的common.Close函数，用于关闭会话的输出和输入缓冲区。

最后，还可以实现一个私有化closing函数，在关闭所有资源后，调用其传入的函数进行关闭。


```go
// Session represents a client connection in a Mux connection.
type Session struct {
	input        buf.Reader
	output       buf.Writer
	parent       *SessionManager
	ID           uint16
	transferType protocol.TransferType
}

// Close closes all resources associated with this session.
func (s *Session) Close() error {
	common.Close(s.output) // nolint: errcheck
	common.Close(s.input)  // nolint: errcheck
	s.parent.Remove(s.ID)
	return nil
}

```

这段代码定义了一个名为`NewReader`的函数，该函数根据当前会话的传输类型创建了一个buf.Reader对象。

首先，函数检查当前会话的传输类型是否为`protocol.TransferTypeStream`。如果是，函数将返回一个基于`buf.Reader`的`NewStreamReader`实例；如果不是，函数将返回一个基于`buf.Reader`的`NewPacketReader`实例。

换句话说，该函数根据会话的传输类型来选择如何创建buf.Reader对象，以便正确地处理数据的传输。


```go
// NewReader creates a buf.Reader based on the transfer type of this Session.
func (s *Session) NewReader(reader *buf.BufferedReader) buf.Reader {
	if s.transferType == protocol.TransferTypeStream {
		return NewStreamReader(reader)
	}
	return NewPacketReader(reader)
}

```

# `common/mux/session_test.go`

这段代码是针对一个名为 "mux_test" 的包进行的测试。该包包含一个名为 "SessionManagerAdd" 的函数。

该函数的主要作用是测试一个名为 "mux" 的会话管理器的添加功能。具体来说，该函数会创建一个 "SessionManager" 类型的实例，然后使用其中的 "Allocate" 函数分配两个会话，并将它们分别添加到管理器中。最后，会测试管理器中添加的会话的 ID 和大小是否符合预期。

由于该函数中直接使用了 "mux" 包，因此它对 "mux" 包的定义是可见的。而 "SessionManager" 和 "Add" 函数则是 "mux" 包中的具体函数，因此它们的实现对 "mux_test" 包的定义也是可见的。


```go
package mux_test

import (
	"testing"

	. "v2ray.com/core/common/mux"
)

func TestSessionManagerAdd(t *testing.T) {
	m := NewSessionManager()

	s := m.Allocate()
	if s.ID != 1 {
		t.Error("id: ", s.ID)
	}
	if m.Size() != 1 {
		t.Error("size: ", m.Size())
	}

	s = m.Allocate()
	if s.ID != 2 {
		t.Error("id: ", s.ID)
	}
	if m.Size() != 2 {
		t.Error("size: ", m.Size())
	}

	s = &Session{
		ID: 4,
	}
	m.Add(s)
	if s.ID != 4 {
		t.Error("id: ", s.ID)
	}
	if m.Size() != 3 {
		t.Error("size: ", m.Size())
	}
}

```

这段代码为名为 "TestSessionManagerClose" 的测试函数，用于测试 SessionManager 关闭功能。具体解释如下：

1. 首先，定义了一个名为 "m" 的变量，并将其类型设置为 "*testing.T"，表示这是一个测试函数。

2. 接着，定义了一个名为 "s" 的变量，并将其类型设置为 "*testing.T"。

3. 使用 NewSessionManager() 函数创建了一个名为 "m" 的会话管理器实例。

4. 使用 m.Allocate() 函数从会话管理器实例中分配了一个 "s" 会话。

5. 接下来，使用条件语句判断 m.CloseIfNoSession() 是否成功关闭会话。如果成功，则执行 t.Error() 函数并传入 "able to close"，否则执行 t.Error() 函数并传入 "not able to close"。这里，如果会话关闭失败，则会崩溃应用程序并输出错误信息。

6. 最后，使用 m.Remove() 函数从会话管理器实例中删除了 "s" 会话。

7. 再次使用条件语句判断 m.CloseIfNoSession() 是否成功关闭会话。如果成功，则不执行 t.Error() 函数，否则执行 t.Error() 函数并传入 "not able to close"。这里，如果会话关闭失败，则会再次崩溃应用程序并输出错误信息。


```go
func TestSessionManagerClose(t *testing.T) {
	m := NewSessionManager()
	s := m.Allocate()

	if m.CloseIfNoSession() {
		t.Error("able to close")
	}
	m.Remove(s.ID)
	if !m.CloseIfNoSession() {
		t.Error("not able to close")
	}
}

```

# `common/mux/writer.go`

该代码定义了一个名为 "mux" 的包，其中包括了一个名为 "Writer" 的结构体。

在 "Writer" 结构体中，定义了一些与缓冲区相关的字段，包括 "dest"(目标网络端口)、"writer"(用于写入数据到目标端口)、"id"(标识符)、"followup"(是否跟上上一个请求)、"hasError"(是否发生错误)、"transferType"(用于不同协议的传输类型)。

总的来说，这个 "Writer" 结构体用于在目标网络中写入数据，并支持多种不同的传输协议，如 HTTP、TCP 和 UDP。


```go
package mux

import (
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
)

type Writer struct {
	dest         net.Destination
	writer       buf.Writer
	id           uint16
	followup     bool
	hasError     bool
	transferType protocol.TransferType
}

```

这两函数创建了一个Writer对象，用于在 transferType 为 protocol.TransferType 时进行数据传输。

具体来说，这两函数创建的Writer对象都包含以下字段：

- id：写入ID，这是一个16位的UUID。
- dest：目标网络终点。
- writer：一个Writer缓冲区，用于存储数据。
- followup：如果设置了为 true，将在发送数据后立即发送Follow-up数据。
- transferType：用于传输的数据类型。

在这两个函数中，我们创建了一个Writer对象，并将其返回。然后，在两个函数中，我们创建了一个新的Writer对象，并将其与上面创建的Writer对象进行初始化。

对于 NewResponseWriter函数，初始化时创建的Writer对象都设置了一些额外的字段，如followup和transferType。

而对于 NewWriter函数，我们创建的Writer对象中，followup和transferType字段都被设置为0。


```go
func NewWriter(id uint16, dest net.Destination, writer buf.Writer, transferType protocol.TransferType) *Writer {
	return &Writer{
		id:           id,
		dest:         dest,
		writer:       writer,
		followup:     false,
		transferType: transferType,
	}
}

func NewResponseWriter(id uint16, writer buf.Writer, transferType protocol.TransferType) *Writer {
	return &Writer{
		id:           id,
		writer:       writer,
		followup:     true,
		transferType: transferType,
	}
}

```

此函数接收一个名为w的Writer作为参数，并返回一个FrameMetadata对象。

FrameMetadata表示一个与HTTP请求相关的元数据对象，其中包含请求的信息，如请求ID、请求目标URL等。

具体来说，函数首先创建一个名为meta的FrameMetadata对象，其中包含请求的会话ID和目标URL，接着判断w是否已经发送了后续帧，如果是，则将meta的sessionStatus设置为SessionStatusKeep，否则将sessionStatus设置为SessionStatusNew，并设置followup为true。最后，函数返回meta对象。

如果w发送了后续帧，那么函数将使用Meta的SessionStatusKeep值，表示请求仍然处于进行中，而不是新创建的SessionStatusNew值，表示请求已经完成了。


```go
func (w *Writer) getNextFrameMeta() FrameMetadata {
	meta := FrameMetadata{
		SessionID: w.id,
		Target:    w.dest,
	}

	if w.followup {
		meta.SessionStatus = SessionStatusKeep
	} else {
		w.followup = true
		meta.SessionStatus = SessionStatusNew
	}

	return meta
}

```

这两函数函数接受一个Writer作为参数，然后执行业士务的写入操作。

函数`writeMetaOnly()`接受一个Writer和一个数据缓冲区(buf)，并返回一个错误。该函数首先从Writer中读取下一帧的元数据，然后将其写入数据缓冲区。如果犯错误，函数返回。

函数`writeMetaWithFrame()`同样接受一个Writer和一个数据缓冲区(buf)，并返回一个错误。该函数首先从Writer中读取下一帧的元数据，然后将其写入数据缓冲区。然后，函数使用`writeUint16()`函数将数据缓冲区中的数据字节数乘以16并将其写入新创建的多缓冲区(mb2)。最后，函数将新创建的多缓冲区(mb2)和数据缓冲区(buf)一起传递给Writer。

这两个函数都是帮助函数，接受一个Writer和一个数据缓冲区，然后执行业务的写入操作。第一个函数将数据缓冲区设置为从Writer中读取的下一帧的元数据，然后将其写入数据缓冲区。第二个函数将数据缓冲区中的数据字节数乘以16并将其写入新创建的多缓冲区，然后将新创建的多缓冲区和数据缓冲区一起传递给Writer。


```go
func (w *Writer) writeMetaOnly() error {
	meta := w.getNextFrameMeta()
	b := buf.New()
	if err := meta.WriteTo(b); err != nil {
		return err
	}
	return w.writer.WriteMultiBuffer(buf.MultiBuffer{b})
}

func writeMetaWithFrame(writer buf.Writer, meta FrameMetadata, data buf.MultiBuffer) error {
	frame := buf.New()
	if err := meta.WriteTo(frame); err != nil {
		return err
	}
	if _, err := serial.WriteUint16(frame, uint16(data.Len())); err != nil {
		return err
	}

	mb2 := make(buf.MultiBuffer, 0, len(data)+1)
	mb2 = append(mb2, frame)
	mb2 = append(mb2, data...)
	return writer.WriteMultiBuffer(mb2)
}

```

这两段代码定义了两个函数，一个是`WriteMultiBuffer`，另一个是`WriteData`。它们都接受一个名为`mb`的`MultiBuffer`参数，并且都不返回错误。

这两个函数都在同一个`Writer`对象中执行，但是它们的实现方式不同。

`WriteMultiBuffer`函数接收一个`MultiBuffer`对象，首先检查`mb`是否为空。如果是，那么函数将直接返回，因为在这种情况下不需要写入数据。否则，函数会遍历`mb`中的所有数据，并将数据逐个写入。

`WriteData`函数也接收一个`MultiBuffer`对象，但首先会尝试使用`TransferTypeStream`类型的`Writer`对象将数据写入。如果使用的是`TransferTypeOverWrite`类型，那么函数将尝试使用`WriteMetaOnly`函数将数据写入，而不是逐个写入。

如果使用的是`TransferTypeStream`类型，那么函数将使用`WriteMetaOnly`函数将数据写入，并将`meta`设置为`getNextFrameMeta()`的输出。`getNextFrameMeta()`函数返回一个`meta`对象，其中包含下一个帧的元数据。通过设置`meta.Option.Set(OptionData)`，可以将`OptionData`设置为`data`的下一个帧的元数据。

如果以上尝试均失败，那么函数将返回一个错误。


```go
func (w *Writer) writeData(mb buf.MultiBuffer) error {
	meta := w.getNextFrameMeta()
	meta.Option.Set(OptionData)

	return writeMetaWithFrame(w.writer, meta, mb)
}

// WriteMultiBuffer implements buf.Writer.
func (w *Writer) WriteMultiBuffer(mb buf.MultiBuffer) error {
	defer buf.ReleaseMulti(mb)

	if mb.IsEmpty() {
		return w.writeMetaOnly()
	}

	for !mb.IsEmpty() {
		var chunk buf.MultiBuffer
		if w.transferType == protocol.TransferTypeStream {
			mb, chunk = buf.SplitSize(mb, 8*1024)
		} else {
			mb2, b := buf.SplitFirst(mb)
			mb = mb2
			chunk = buf.MultiBuffer{b}
		}
		if err := w.writeData(chunk); err != nil {
			return err
		}
	}

	return nil
}

```

这段代码定义了一个名为 `Close` 的函数，它属于一个名为 ` common.Closable` 的类。

这个函数接受一个名为 `w` 的整数指针，代表一个 `Writer` 对象。函数实现的作用是关闭 `Writer` 对象，即关闭数据写入。

具体来说，函数内部先创建一个名为 `meta` 的 `FrameMetadata` 对象，其中 `SessionID` 和 `SessionStatus` 字段分别表示当前会话的 ID 和状态，如果 `writer` 对象有错误，则将 `OptionError` 选项设置为 `meta.Option.Set()`。

然后，使用 `buf.New()` 创建一个缓冲区 `frame`，并将其写入 `meta` 对象中。

接着，将 `w.writer` 对象中的所有数据多缓冲区 `frame` 写入。注意，由于 `w.writer` 对象是 `Writer` 类型，所以这里使用了多缓冲区 `frame` 作为数据缓冲区，而不是 `Reader` 类型。

最后，关闭 `writer` 对象并返回。


```go
// Close implements common.Closable.
func (w *Writer) Close() error {
	meta := FrameMetadata{
		SessionID:     w.id,
		SessionStatus: SessionStatusEnd,
	}
	if w.hasError {
		meta.Option.Set(OptionError)
	}

	frame := buf.New()
	common.Must(meta.WriteTo(frame))

	w.writer.WriteMultiBuffer(buf.MultiBuffer{frame}) // nolint: errcheck
	return nil
}

```