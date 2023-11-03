# v2ray-core源码解析 48

# `proxy/trojan/errors.generated.go`

这段代码定义了一个名为"trojan"的包，其中包含一个名为"errPathObjHolder"的结构体，以及一个名为"newError"的函数。

函数的参数为多个任意类型的切片，通过...切片参数传递给函数使用。函数内部创建一个名为"errPathObjHolder"的匿名结构体，类型为"time.Error"。

接着函数内部创建一个名为"values"的切片，使用循环将切片中的每个元素值复制到"errPathObjHolder"结构体中，结果保存到"errPathObjHolder"结构体中的"values"切片。

最后，使用函数内部创建的"errPathObjHolder"结构体创建一个带有错误堆栈信息的错误对象，该对象使用"WithPathObj"方法设置错误路径对象，该对象在函数内部使用"values"切片获取错误堆栈信息。

通过上述结构体，函数可以创建一个不同类型的错误对象，并设置错误堆栈信息，使得调试更加方便。


```go
package trojan

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `proxy/trojan/protocol.go`

这段代码是一个Trojan毒液木马，其作用是进行地址解析，以便在传输数据时隐藏真实的IP地址。

具体来说，代码首先定义了一个名为crlf的缓冲区，用于在传输结束后输出一个换行符。然后，代码定义了一个名为addrParser的地址解析器，该解析器用于解析目标地址。

addrParser使用协议.NewAddressParser函数创建了一个新的地址解析器，该函数将一个IPv4地址和一个IPv6地址作为参数。在这里，地址家族Byte(0x01, net.AddressFamilyIPv4)表示IPv4地址，地址家族Byte(0x04, net.AddressFamilyIPv6)表示IPv6地址。

protocol.AddressFamilyByte(0x03, net.AddressFamilyDomain)表示查询服务器的域名，用于获取服务器地址。由于没有提供具体的域名，因此该地址将不会被查询。

最后，代码创建了一个名为缓冲区的变量，用于存储crlf缓冲区中的数据。


```go
package trojan

import (
	"encoding/binary"
	"io"

	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
)

var (
	crlf = []byte{'\r', '\n'}

	addrParser = protocol.NewAddressParser(
		protocol.AddressFamilyByte(0x01, net.AddressFamilyIPv4),   // nolint: gomnd
		protocol.AddressFamilyByte(0x04, net.AddressFamilyIPv6),   // nolint: gomnd
		protocol.AddressFamilyByte(0x03, net.AddressFamilyDomain), // nolint: gomnd
	)
)

```

这段代码定义了一个名为`ConnWriter`的结构体，它表示在网络套接字上以TCP协议进行通信时使用的连接写入器。

首先，定义了一些常量，包括最大数据传输量为8192字节。

然后，定义了两个变量，一个字节类型的变量`commandTCP`，其值为1，表示使用TCP协议发送数据；另一个字节类型的变量`commandUDP`，其值为3，表示使用UDP协议发送数据。

接着，定义了一个`ConnWriter`类型的变量，其中`io.Writer`表示该变量是一个字节切片，用于向网络套接字写入数据；`Target`是一个`net.Destination`类型的变量，表示目标IP地址；`Account`是一个`MemoryAccount`类型的变量，表示用于记录已经发送的数据的长度；`headerSent`是一个布尔类型的变量，表示是否已经发送了TCP协议的头部。

最后，该`ConnWriter`类型的变量可以被赋值，例如：

conn := new(ConnWriter)
conn.Target = "127.0.0.1"
conn.Account = &MemoryAccount{1024}
conn.headerSent = false

这个`ConnWriter`实例将创建一个TCP连接写入器，向目标地址127.0.0.1发送数据，并将已经发送的数据记录在`Account`中。同时，`headerSent`变量将设置为`false`，表示还没有发送TCP协议的头部。


```go
const (
	maxLength = 8192

	commandTCP byte = 1
	commandUDP byte = 3
)

// ConnWriter is TCP Connection Writer Wrapper for trojan protocol
type ConnWriter struct {
	io.Writer
	Target     net.Destination
	Account    *MemoryAccount
	headerSent bool
}

```

这两段代码定义了两个名为 `Write` 和 `WriteMultiBuffer` 的函数，旨在实现网络连接中的数据传输。

`Write` 函数的实现包括两个步骤。第一步，如果连接的头部没有发送，那么需要先调用 `c.writeHeader` 函数来发送请求头部。如果这个步骤有错误，那么错误对象将被抛出并返回。第二步，调用 `c.Writer.Write` 函数将数据写入到 `c.Writer` 对象中。

`WriteMultiBuffer` 函数的实现包括一个循环，用于将缓冲区中的多个数据包组合成一个数据包并写入到 `c.Writer` 对象中。在组合数据包之前，它确保缓冲区中的数据不是空的，并且如果必要，将调用 `c.writeHeader` 函数来发送请求头部。

这两个函数都是使用 `io.Writer` 和 `buf.Writer` 接口实现的，因此可以组合多个字节数据和多个缓冲区数据来发送数据。


```go
// Write implements io.Writer
func (c *ConnWriter) Write(p []byte) (n int, err error) {
	if !c.headerSent {
		if err := c.writeHeader(); err != nil {
			return 0, newError("failed to write request header").Base(err)
		}
	}

	return c.Writer.Write(p)
}

// WriteMultiBuffer implements buf.Writer
func (c *ConnWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	defer buf.ReleaseMulti(mb)

	for _, b := range mb {
		if !b.IsEmpty() {
			if _, err := c.Write(b.Bytes()); err != nil {
				return err
			}
		}
	}

	return nil
}

```

这段代码的作用是定义了一个名为`func`的函数，接收一个名为`c`的指针参数`ConnWriter`，并对其进行写入。函数内部使用了一个名为`buf`的缓冲区，用来存储要写入的数据，并使用`Release`方法将缓冲区使用完毕时释放。

函数首先根据传入的`c`对象的目标网络类型，判断需要使用TCP协议还是UDP协议进行写入。如果目标网络类型是UDP，则执行以下操作：

1. 将`command`字段设置为`commandTCP`，如果是TCP，则执行以下操作：

	1. 如果缓冲区已经被分配，则先将其写入，再尝试写入`crlf`字符，最后写入`command`字段。

	2. 如果写入`crlf`或`command`字段时出现错误，则返回错误。

	3. 如果缓冲区已经被写入，则执行以下操作：

		1. 将`command`字段写入，并尝试写入`crlf`字符，最后写入`crlf`字符。

		2. 如果写入`crlf`或`command`字段时出现错误，则返回错误。

	3. 尝试将`c.Writer`对象写入缓冲区中的数据，如果写入成功，则说明已经发送了`header`，函数返回一个`error`类型。


```go
func (c *ConnWriter) writeHeader() error {
	buffer := buf.StackNew()
	defer buffer.Release()

	command := commandTCP
	if c.Target.Network == net.Network_UDP {
		command = commandUDP
	}

	if _, err := buffer.Write(c.Account.Key); err != nil {
		return err
	}
	if _, err := buffer.Write(crlf); err != nil {
		return err
	}
	if err := buffer.WriteByte(command); err != nil {
		return err
	}
	if err := addrParser.WriteAddressPort(&buffer, c.Target.Address, c.Target.Port); err != nil {
		return err
	}
	if _, err := buffer.Write(crlf); err != nil {
		return err
	}

	_, err := c.Writer.Write(buffer.Bytes())
	if err == nil {
		c.headerSent = true
	}

	return err
}

```

这段代码定义了一个名为 PacketWriter 的 UDP 连接写入者，用于在 Trojan 协议中进行数据传输。

PacketWriter 结构体包含一个 io.Writer 类型的成员和一个目标网络地址类型的成员 Target。

PacketWriter 的 WriteMultiBuffer 方法实现了 buf.Writer 接口，该接口表示包装了一个缓冲区并实现了多个数据包的写入。

WriteMultiBuffer 方法的实现中，首先创建一个长度为 maxLength 的缓冲区 b。然后，使用 isEmpty() 方法判断缓冲区是否为空，如果不是，则执行以下操作：

1. 从目标网络地址类型类型的成员 Target 中的网络地址开始，创建一个长度为 length 的缓冲区，将其中的数据包写入该缓冲区。

2. 如果创建缓冲区或写入数据包的过程中出现错误，则返回错误并释放缓冲区。

3. 最后，释放多缓冲区并返回。

该代码的作用是实现了一个 UDP 连接写入者，用于在 Trojan 协议中进行数据传输，可以将多个数据包打包并发送到目标地址。


```go
// PacketWriter UDP Connection Writer Wrapper for trojan protocol
type PacketWriter struct {
	io.Writer
	Target net.Destination
}

// WriteMultiBuffer implements buf.Writer
func (w *PacketWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
	b := make([]byte, maxLength)
	for !mb.IsEmpty() {
		var length int
		mb, length = buf.SplitBytes(mb, b)
		if _, err := w.writePacket(b[:length], w.Target); err != nil {
			buf.ReleaseMulti(mb)
			return err
		}
	}

	return nil
}

```

这段代码定义了一个名为`WriteMultiBufferWithMetadata`的函数，该函数接受一个名为`mb`的传输包`MultiBuffer`和一个目标网络`net.Destination`作为参数。

函数的作用是向指定的目标网络发送一个带数据的消息，该消息使用一个名为`mb`的`MultiBuffer`进行传输。在函数内部，首先创建一个长度为`maxLength`的缓冲区`b`，然后使用`mb.IsEmpty()`判断`MultiBuffer`是否为空，如果不是，则执行以下操作：

1. 从`mb`中分离出数据长度`length`，并从`MultiBuffer`中读取数据，使用`buf.SplitBytes`函数将数据和缓冲区的索引关联起来。
2. 如果`w.writePacket`函数在写入数据时出现错误，则使用`buf.ReleaseMulti`函数释放`MultiBuffer`，并返回错误。
3. 如果没有错误，则继续在缓冲区中添加数据，使用`w.writePacket`函数将数据写入目标网络，目标网络为`dest`。

最后，函数返回一个`nil`表示没有错误。


```go
// WriteMultiBufferWithMetadata writes udp packet with destination specified
func (w *PacketWriter) WriteMultiBufferWithMetadata(mb buf.MultiBuffer, dest net.Destination) error {
	b := make([]byte, maxLength)
	for !mb.IsEmpty() {
		var length int
		mb, length = buf.SplitBytes(mb, b)
		if _, err := w.writePacket(b[:length], dest); err != nil {
			buf.ReleaseMulti(mb)
			return err
		}
	}

	return nil
}

```

这段代码定义了一个名为`func`的函数，接受一个名为`w`的包写入器和一个目标网络字节组`dest`作为参数。函数的作用是接收一个字节数组`payload`，将其中的数据写入到目标网络字节组`dest`中，并返回写入成功所需的时间（如果出错则返回-1）。

函数的实现可以分为以下几个步骤：

1. 创建一个名为`buffer`的缓冲区，用于存储输入的数据，并将其大小设置为输入的字节数组长度。
2. 创建一个长度为2的字符数组`lengthBuf`，并使用`putUint16`函数将其设置为输入字节数组长度。
3. 如果出现错误，使用`WriteAddressPort`函数将目标IP地址和端口写入到缓冲区中，然后返回。
4. 如果出现错误，使用`Write`函数将`lengthBuf`中的数据写入到缓冲区中，然后调用包写入器`w`继续写入数据。
5. 如果出现错误，返回-1表示函数失败。

函数的实现可以帮助您向目标网络写入数据，并确保在包写入器接收到数据之前将其写入到缓冲区中。


```go
func (w *PacketWriter) writePacket(payload []byte, dest net.Destination) (int, error) { // nolint: unparam
	buffer := buf.StackNew()
	defer buffer.Release()

	length := len(payload)
	lengthBuf := [2]byte{}
	binary.BigEndian.PutUint16(lengthBuf[:], uint16(length))
	if err := addrParser.WriteAddressPort(&buffer, dest.Address, dest.Port); err != nil {
		return 0, err
	}
	if _, err := buffer.Write(lengthBuf[:]); err != nil {
		return 0, err
	}
	if _, err := buffer.Write(crlf); err != nil {
		return 0, err
	}
	if _, err := buffer.Write(payload); err != nil {
		return 0, err
	}
	_, err := w.Write(buffer.Bytes())
	if err != nil {
		return 0, err
	}

	return length, nil
}

```

该代码定义了一个名为ConnReader的TCP连接读取器类，用于解析Trojan协议中的数据包头部信息。

在构造函数中，首先从目标网络中读取一个TCP连接的地址和端口号，并将其存储在ConnReader的target和port变量中。然后读取一个TCP数据报的头部信息，其中包括协议头和数据部分。如果数据部分包含的 crlf 标记和数据部分不匹配，则会抛出一个新的错误。如果解析头部数据成功，则将解析好的目标地址和端口号存储在ConnReader的target和port变量中，并设置其headerParsed标志。

在ParseHeader函数中，首先读取一个包含Trojan协议头数据的字节切片，并解码出其中的 crlf 标记、协议头和数据部分。然后根据解码出的数据部分中的命令标记和网络类型，判断出数据报的传输协议，并将其存储在网络变量中。接着，从解码出的数据部分中提取出目标地址和端口号，并将其存储在ConnReader的target变量中。最后，如果解析头部数据成功，则设置其headerParsed标志。

在trojan协议中，头部信息非常重要，可以提供对数据包进行传输时所必需的地址和端口信息，以及用于标识数据包的内容类型。通过读取和解析这些头部信息，可以正确地传递数据包，并做出相应的处理。


```go
// ConnReader is TCP Connection Reader Wrapper for trojan protocol
type ConnReader struct {
	io.Reader
	Target       net.Destination
	headerParsed bool
}

// ParseHeader parses the trojan protocol header
func (c *ConnReader) ParseHeader() error {
	var crlf [2]byte
	var command [1]byte
	var hash [56]byte
	if _, err := io.ReadFull(c.Reader, hash[:]); err != nil {
		return newError("failed to read user hash").Base(err)
	}

	if _, err := io.ReadFull(c.Reader, crlf[:]); err != nil {
		return newError("failed to read crlf").Base(err)
	}

	if _, err := io.ReadFull(c.Reader, command[:]); err != nil {
		return newError("failed to read command").Base(err)
	}

	network := net.Network_TCP
	if command[0] == commandUDP {
		network = net.Network_UDP
	}

	addr, port, err := addrParser.ReadAddressPort(nil, c.Reader)
	if err != nil {
		return newError("failed to read address and port").Base(err)
	}
	c.Target = net.Destination{Network: network, Address: addr, Port: port}

	if _, err := io.ReadFull(c.Reader, crlf[:]); err != nil {
		return newError("failed to read crlf").Base(err)
	}

	c.headerParsed = true
	return nil
}

```

这两段代码定义了两个名为 `Read` 和 `ReadMultiBuffer` 的函数，属于 `io.Reader` 和 `buf.Reader` 接口的实现。

`Read` 函数的实现读取一个字节数组 `p` 中的数据，并返回读取的字节数和可能的错误。函数在开始时检查 `c` 对象对应的 `headerParsed` 字段是否已设置。如果未设置，函数将调用 `c.ParseHeader()` 函数来设置 `headerParsed` 字段。如果设置过程中出现错误，函数将返回 0 和错误对象。否则，函数将返回读取的字节数。

`ReadMultiBuffer` 函数的实现创建一个名为 `b` 的缓冲区 `MultiBuffer`，并返回一个指向 `MultiBuffer` 对象的引用以及可能的错误。函数读取数据时，每次读取一个 `byte` 并将其添加到 `b` 缓冲区中。然后，函数返回一个名为 `MultiBuffer` 对象的引用，该对象允许 `ReadMultiBuffer` 函数复用多个 `byte` 读取操作。

这两段代码的主要目的是让 `Read` 和 `ReadMultiBuffer` 函数可以读取来自网络流或其他数据源的 `io.Reader` 数据，并按照要求返回数据读取结果或错误信息。


```go
// Read implements io.Reader
func (c *ConnReader) Read(p []byte) (int, error) {
	if !c.headerParsed {
		if err := c.ParseHeader(); err != nil {
			return 0, err
		}
	}

	return c.Reader.Read(p)
}

// ReadMultiBuffer implements buf.Reader
func (c *ConnReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	b := buf.New()
	_, err := b.ReadFrom(c)
	return buf.MultiBuffer{b}, err
}

```

这段代码定义了一个名为 PacketPayload 的结构体，该结构体表示一个目标为给定 UDP 目标的 UDP 数据包的携带部分。

该结构体定义了一个名为 Target 的成员，它表示目标 UDP 目标地址，以及一个名为 Buffer 的成员，它是一个缓冲区缓冲区中的多字节数据。

该 PacketPayload 结构体定义了一个名为 PacketReader 的类，该类实现了一个 UDP 连接读取器，该类使用 trojan 协议进行通信。

该 PacketReader 类包含一个名为 io.Reader 的成员，该成员表示一个可以读取多字节数据的 I/O Reader。

该 PacketReader 类实现了一个名为 ReadMultiBuffer 的方法，该方法使用 io.Reader 从缓冲区中读取多字节数据，并返回一个名为 p 的缓冲区和一个名为 err 的错误。如果读取过程中出现错误，该错误对象将被传递给返回值。

该 PacketReader 类还实现了一个名为 NewPacketReader 的方法，该方法接受一个 PacketPayload 结构体作为其构造函数的参数，并返回一个 PacketReader 实例。


```go
// PacketPayload combines udp payload and destination
type PacketPayload struct {
	Target net.Destination
	Buffer buf.MultiBuffer
}

// PacketReader is UDP Connection Reader Wrapper for trojan protocol
type PacketReader struct {
	io.Reader
}

// ReadMultiBuffer implements buf.Reader
func (r *PacketReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
	p, err := r.ReadMultiBufferWithMetadata()
	if p != nil {
		return p.Buffer, err
	}
	return nil, err
}

```

这段代码的作用是读取一个带有元数据的UDP数据包。它的主要步骤如下：

1. 读取目标IP地址和端口。
2. 读取数据包的长度。
3. 如果数据包长度小于设定的最大长度，将数据包长度设为当前长度，然后继续读取。
4. 读取数据包内的数据。
5. 如果遇到连续的空闲字节，则表示数据包结束，释放内存并返回一个错误。
6. 如果没有错误，就返回一个包含目标UDP地址和数据的结构体，以及一个表示数据包长度的整数。


```go
// ReadMultiBufferWithMetadata reads udp packet with destination
func (r *PacketReader) ReadMultiBufferWithMetadata() (*PacketPayload, error) {
	addr, port, err := addrParser.ReadAddressPort(nil, r)
	if err != nil {
		return nil, newError("failed to read address and port").Base(err)
	}

	var lengthBuf [2]byte
	if _, err := io.ReadFull(r, lengthBuf[:]); err != nil {
		return nil, newError("failed to read payload length").Base(err)
	}

	remain := int(binary.BigEndian.Uint16(lengthBuf[:]))
	if remain > maxLength {
		return nil, newError("oversize payload")
	}

	var crlf [2]byte
	if _, err := io.ReadFull(r, crlf[:]); err != nil {
		return nil, newError("failed to read crlf").Base(err)
	}

	dest := net.UDPDestination(addr, port)
	var mb buf.MultiBuffer
	for remain > 0 {
		length := buf.Size
		if remain < length {
			length = remain
		}

		b := buf.New()
		mb = append(mb, b)
		n, err := b.ReadFullFrom(r, int32(length))
		if err != nil {
			buf.ReleaseMulti(mb)
			return nil, newError("failed to read payload").Base(err)
		}

		remain -= int(n)
	}

	return &PacketPayload{Target: dest, Buffer: mb}, nil
}

```

# `proxy/trojan/protocol_test.go`

这段代码定义了一个名为 "trojan" 的包，该包包含了一些用于测试 Trojan 代理的函数和变量。下面是对该包中的一些主要函数的的解释：

1. "toAccount" 函数将一个 "Account" 类型的对象转换为 "protocol.Account" 类型的对象。这个函数的作用是将 "Account" 对象转换为可以用于 "protocol.Account" 类型接口的 "protocol.Account" 对象。

2. "testing" 包定义了 "accountTest" 函数。这个函数的作用是用于测试 "toAccount" 函数的正确性。具体来说，它创建了一个 "Account" 对象，然后使用 "toAccount" 函数将其转换为 "protocol.Account""，最后测试 "protocol.Account" 是否与 "Account" 相等。

3. "v2ray.com/core/common/net" 包定义了一个名为 "testnet" 的网络接口。这个包的作用是在 "v2ray.com/core/proxy/trojan" 包的测试中使用。它提供了一个用于测试网络通信的测试网络，可以用于测试代理之间的通信。

4. "v2ray.com/core/common/protocol" 包定义了一个名为 "testprotocol" 的测试协议。这个包的作用是用于测试 "v2ray.com/core/proxy/trojan" 包的代理是否能够正确地解析和发送 "testprotocol" 协议的数据。

5. "v2ray.com/core/proxy/trojan" 包定义了一个名为 "Trojan" 的代理类。这个包的作用是用于测试 "v2ray.com/core/proxy/trojan" 包的代理是否能够正确地创建、发送和接收 "v2ray.com/core/testprotocol" 协议的数据。


```go
package trojan_test

import (
	"testing"

	"github.com/google/go-cmp/cmp"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	. "v2ray.com/core/proxy/trojan"
)

func toAccount(a *Account) protocol.Account {
	account, err := a.AsAccount()
	common.Must(err)
	return account
}

```

该代码的主要目的是测试一个名为 "TestTCPRequest" 的函数，该函数接受一个测试开关 (t *testing.T)。该函数的作用如下：

1. 创建一个名为 "user" 的内存用户对象，该用户对象包含一个电子邮件地址和一个密码，分别传递给测试用例中的 "user" 和 "password" 变量。
2. 创建一个字符串 "test string"，并将其字节化成一个缓冲缓冲区 (buf)。
3. 创建一个缓冲区 (data) 和一个缓冲区 (buffer)，分别用于数据和读取数据。
4. 调用 "ToAccount" 函数创建一个名为 "Account" 的内存对象，并将其传递给 "user.Account" 类型的参数。
5. 调用 "Write" 函数将 "test string" 缓冲区中的数据写入缓冲区 (data)。
6. 调用 "MultiBuffer" 函数将数据缓冲区 (data) 中的数据拆分成多个小缓冲区，每个小缓冲区都有相同的数量的数据。
7. 创建一个目标连接 (destination)，该连接的网络为网络_TCP，地址为本地主机 IP 地址，端口为 1234。
8. 创建一个 "Writer" 类型的连接写入器 (writer)，该写入器将数据缓冲区 (data) 中的数据写入目标连接 (destination)，并将目标连接 (destination) 作为其目标。将 "user.Account" 内存对象中的 Account 类型作为连接写入器 (writer) 的 Account 参数传递给 writer。
9. 创建一个 "Reader" 类型的连接读取器 (reader)，该读取器从缓冲区 (data) 中读取数据。
10. 调用 "ReadHeader" 函数读取目标连接 (destination) 的头信息并将其存储在 "decodedData" 变量中。
11. 如果 "decodedData" 和 "destination" 的头信息不相等，则打印错误消息。
12. 打印 "data" 缓冲区中的数据。
13. 创建一个名为 "cmp" 的比较器 (cmp)，并将其类型设置为 "test驾驶"。
14. 调用 "Diff" 函数比较 "decodedData" 和 "destination" 的字节顺序，并将结果存储在 "r" 变量中。
15. 由于 "decodedData" 和 "destination" 的字节顺序相等，因此打印错误消息。


```go
func TestTCPRequest(t *testing.T) {
	user := &protocol.MemoryUser{
		Email: "love@v2ray.com",
		Account: toAccount(&Account{
			Password: "password",
		}),
	}
	payload := []byte("test string")
	data := buf.New()
	common.Must2(data.Write(payload))

	buffer := buf.New()
	defer buffer.Release()

	destination := net.Destination{Network: net.Network_TCP, Address: net.LocalHostIP, Port: 1234}
	writer := &ConnWriter{Writer: buffer, Target: destination, Account: user.Account.(*MemoryAccount)}
	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{data}))

	reader := &ConnReader{Reader: buffer}
	common.Must(reader.ParseHeader())

	if r := cmp.Diff(reader.Target, destination); r != "" {
		t.Error("destination: ", r)
	}

	decodedData, err := reader.ReadMultiBuffer()
	common.Must(err)
	if r := cmp.Diff(decodedData[0].Bytes(), payload); r != "" {
		t.Error("data: ", r)
	}
}

```

这段代码是一个用于测试 UDP 请求的 Go 语言函数，名为 "TestUDPRequest"。它的作用是模拟一个 UDP 请求的过程，并对请求的数据进行测试。

具体来说，这段代码创建了一个 UDP 客户端（即的用户代理）和一个 UDP 服务器，然后向服务器发送一个包含请求数据的数据包。接下来，这段代码会处理服务器接收到这个数据包并返回的结果，包括对请求数据解码和比较，以验证请求是否被正确接收和解析。

概括来说，这段代码的主要作用是测试 UDP 请求的功能和正确性，以便在实际应用中确保系统的稳定性。


```go
func TestUDPRequest(t *testing.T) {
	user := &protocol.MemoryUser{
		Email: "love@v2ray.com",
		Account: toAccount(&Account{
			Password: "password",
		}),
	}
	payload := []byte("test string")
	data := buf.New()
	common.Must2(data.Write(payload))

	buffer := buf.New()
	defer buffer.Release()

	destination := net.Destination{Network: net.Network_UDP, Address: net.LocalHostIP, Port: 1234}
	writer := &PacketWriter{Writer: &ConnWriter{Writer: buffer, Target: destination, Account: user.Account.(*MemoryAccount)}, Target: destination}
	common.Must(writer.WriteMultiBuffer(buf.MultiBuffer{data}))

	connReader := &ConnReader{Reader: buffer}
	common.Must(connReader.ParseHeader())

	packetReader := &PacketReader{Reader: connReader}
	p, err := packetReader.ReadMultiBufferWithMetadata()
	common.Must(err)

	if p.Buffer.IsEmpty() {
		t.Error("no request data")
	}

	if r := cmp.Diff(p.Target, destination); r != "" {
		t.Error("destination: ", r)
	}

	mb, decoded := buf.SplitFirst(p.Buffer)
	buf.ReleaseMulti(mb)

	if r := cmp.Diff(decoded.Bytes(), payload); r != "" {
		t.Error("data: ", r)
	}
}

```

# `proxy/trojan/server.go`

这段代码是一个 Go 语言编写的 build 任务。它通过 `build` 命令构建一个新版的 V2Ray 框架。在这个过程中，它将会执行以下操作：

1. 将 `.confonly` 标志设置为 `true`，这意味着这个 build 任务仅在 `.confonly` 文件夹下进行，而不会修改原始代码或者向输出目录输出任何内容。
2. 构建 V2Ray 框架的一些依赖项，如 crypto/tls、io、strconv、time、v2ray.com/core 等。
3. 设置 UDP 协议为 v2ray.com/core/common/protocol/udp。
4. 设置 HTTP/1.1 头部以启用 HTTP 头部。
5. 设置 V2Ray 的一些配置选项，如在失败时重试多少次，超时时间等。
6. 设置 V2Ray 的一些核心服务的房间 ID，以便在构建过程中下载预设。
7. 下载并设置 OpenSSL TLS 证书。
8. 使用 Go 标准库中的 `context`、`crypto/tls`、`io`、`strconv`、`time`、`v2ray.com/core/common/net`、`v2ray.com/core/common/retry`、`v2ray.com/core/common/session`、`v2ray.com/core/common/signal`、`v2ray.com/core/common/task` 和 `v2ray.com/core/features/policy`、`v2ray.com/core/features/routing`。


```go
// +build !confonly

package trojan

import (
	"context"
	"crypto/tls"
	"io"
	"strconv"
	"time"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/errors"
	"v2ray.com/core/common/log"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	udp_proto "v2ray.com/core/common/protocol/udp"
	"v2ray.com/core/common/retry"
	"v2ray.com/core/common/session"
	"v2ray.com/core/common/signal"
	"v2ray.com/core/common/task"
	"v2ray.com/core/features/policy"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/transport/internet"
	"v2ray.com/core/transport/internet/udp"
	"v2ray.com/core/transport/internet/xtls"
)

```

This is a Go language implementation that creates a new inbound trojan handler with a specified fallback policy, and adds the fallback policy to the map of fallbacks.

It first checks if the given fallback policy is set, and if not, it creates a new fallback map, the same way as the `map[string]map[string]*Fallback` is defined.

Then, it gets the current validator and adds the user to it, if any, and makes sure the user has been added.

Finally, it initializes the server with the fallback policy and the validator, and returns the server. If any errors occur, it will return an error and log it.


```go
func init() {
	common.Must(common.RegisterConfig((*ServerConfig)(nil), func(ctx context.Context, config interface{}) (interface{}, error) { // nolint: lll
		return NewServer(ctx, config.(*ServerConfig))
	}))
}

// Server is an inbound connection handler that handles messages in trojan protocol.
type Server struct {
	policyManager policy.Manager
	validator     *Validator
	fallbacks     map[string]map[string]*Fallback // or nil
}

// NewServer creates a new trojan inbound handler.
func NewServer(ctx context.Context, config *ServerConfig) (*Server, error) {
	validator := new(Validator)
	for _, user := range config.Users {
		u, err := user.ToMemoryUser()
		if err != nil {
			return nil, newError("failed to get trojan user").Base(err).AtError()
		}

		if err := validator.Add(u); err != nil {
			return nil, newError("failed to add user").Base(err).AtError()
		}
	}

	v := core.MustFromContext(ctx)
	server := &Server{
		policyManager: v.GetFeature(policy.ManagerType()).(policy.Manager),
		validator:     validator,
	}

	if config.Fallbacks != nil {
		server.fallbacks = make(map[string]map[string]*Fallback)
		for _, fb := range config.Fallbacks {
			if server.fallbacks[fb.Alpn] == nil {
				server.fallbacks[fb.Alpn] = make(map[string]*Fallback)
			}
			server.fallbacks[fb.Alpn][fb.Path] = fb
		}
		if server.fallbacks[""] != nil {
			for alpn, pfb := range server.fallbacks {
				if alpn != "" { // && alpn != "h2" {
					for path, fb := range server.fallbacks[""] {
						if pfb[path] == nil {
							pfb[path] = fb
						}
					}
				}
			}
		}
	}

	return server, nil
}

```

This is a Go function that handles the receiving of a TCP connection from a client, performs any necessary authentication and passes the first message of the buffer to the `handleConnection` function, which will handle the logic of the connection and/or the sending of messages.

This function takes in a connection context, a policy for authentication, the `user` object representing the incoming connection, a `BufferedReader` and a `PacketWriter` for handling the message that will be sent to the `handleConnection` function.

It is first checking if a `Facebook` policy is set, if it is not set it will return a new error, if it is set it will first check if the `shouldFallback` is true or not, if it is not set it will return a new error, if it is set it will return the result of the `s.fallback` function.

Then it will check if the `shouldFallback` is set or not, if it is not set it will return a new error, if it is set it will first check the current user level, then it will check the `clientManager` and then it will check the `shouldUseAPFB` if it is set or not.

It will then create a `BufferedReader` and a `PacketWriter` and set the writer to the connection and the writer to the `clientManager` and the base on the writer, it will then read the message from the `BufferedReader` and pass it to the `handleConnection` function.


```go
// Network implements proxy.Inbound.Network().
func (s *Server) Network() []net.Network {
	return []net.Network{net.Network_TCP}
}

// Process implements proxy.Inbound.Process().
func (s *Server) Process(ctx context.Context, network net.Network, conn internet.Connection, dispatcher routing.Dispatcher) error { // nolint: funlen,lll

	sid := session.ExportIDToError(ctx)

	iConn := conn
	if statConn, ok := iConn.(*internet.StatCouterConnection); ok {
		iConn = statConn.Connection
	}

	sessionPolicy := s.policyManager.ForLevel(0)
	if err := conn.SetReadDeadline(time.Now().Add(sessionPolicy.Timeouts.Handshake)); err != nil {
		return newError("unable to set read deadline").Base(err).AtWarning()
	}

	first := buf.New()
	defer first.Release()

	firstLen, err := first.ReadFrom(conn)
	if err != nil {
		return newError("failed to read first request").Base(err)
	}
	newError("firstLen = ", firstLen).AtInfo().WriteToLog(sid)

	bufferedReader := &buf.BufferedReader{
		Reader: buf.NewReader(conn),
		Buffer: buf.MultiBuffer{first},
	}

	var user *protocol.MemoryUser

	apfb := s.fallbacks
	isfb := apfb != nil

	shouldFallback := false
	if firstLen < 58 || first.Byte(56) != '\r' { // nolint: gomnd
		// invalid protocol
		err = newError("not trojan protocol")
		log.Record(&log.AccessMessage{
			From:   conn.RemoteAddr(),
			To:     "",
			Status: log.AccessRejected,
			Reason: err,
		})

		shouldFallback = true
	} else {
		user = s.validator.Get(hexString(first.BytesTo(56))) // nolint: gomnd
		if user == nil {
			// invalid user, let's fallback
			err = newError("not a valid user")
			log.Record(&log.AccessMessage{
				From:   conn.RemoteAddr(),
				To:     "",
				Status: log.AccessRejected,
				Reason: err,
			})

			shouldFallback = true
		}
	}

	if isfb && shouldFallback {
		return s.fallback(ctx, sid, err, sessionPolicy, conn, iConn, apfb, first, firstLen, bufferedReader)
	} else if shouldFallback {
		return newError("invalid protocol or invalid user")
	}

	clientReader := &ConnReader{Reader: bufferedReader}
	if err := clientReader.ParseHeader(); err != nil {
		log.Record(&log.AccessMessage{
			From:   conn.RemoteAddr(),
			To:     "",
			Status: log.AccessRejected,
			Reason: err,
		})
		return newError("failed to create request from: ", conn.RemoteAddr()).Base(err)
	}

	destination := clientReader.Target
	if err := conn.SetReadDeadline(time.Time{}); err != nil {
		return newError("unable to set read deadline").Base(err).AtWarning()
	}

	inbound := session.InboundFromContext(ctx)
	if inbound == nil {
		panic("no inbound metadata")
	}
	inbound.User = user
	sessionPolicy = s.policyManager.ForLevel(user.Level)

	if destination.Network == net.Network_UDP { // handle udp request
		return s.handleUDPPayload(ctx, &PacketReader{Reader: clientReader}, &PacketWriter{Writer: conn}, dispatcher)
	}

	// handle tcp request

	ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
		From:   conn.RemoteAddr(),
		To:     destination,
		Status: log.AccessAccepted,
		Reason: "",
		Email:  user.Email,
	})

	newError("received request for ", destination).WriteToLog(sid)
	return s.handleConnection(ctx, sessionPolicy, destination, clientReader, buf.NewWriter(conn), dispatcher)
}

```

这段代码是一个UDP协议的Server功能，负责处理接收到的UDP数据包。具体来说，它实现了以下功能：

1. 创建一个UDP服务器实例，并将其存储在变量s中。
2. 定义一个名为handleUDPPayload的函数，接收一个UDP数据包、一个PacketReader和一个PacketWriter作为参数。
3. 在函数内部，使用udp.NewDispatcher创建一个UDP dispatcher，并使用该dispatcher函数来处理接收到的数据包。
4. 在dispatcher中，定义一个名为<-clientWriter.WriteMultiBufferWithMetadata的函数，接收一个MultiBuffer和一个UDP数据包作为参数。
5. 在handleUDPPayload函数中，使用for循环来接收和处理UDP数据包。
6. 对于每个UDP数据包，先通过clientReader.ReadMultiBufferWithMetadata读取数据包并设置其Payload和Source。
7. 然后，使用inbound.Session.InboundFromContext获取当前会话的上下文，并通过该上下文执行getter和setter操作，获取和设置客户端的ID。
8. 在处理UDP数据包的过程中，如果当前会话已经结束（通过ctx.Done()函数返回），则返回nil。否则，读取并处理数据包中的所有数据，然后将处理结果写入日志。
9. 最后，定义一个名为newError的函数，接收一个错误消息字符串和一个上下文作为参数，并输出错误信息。


```go
func (s *Server) handleUDPPayload(ctx context.Context, clientReader *PacketReader, clientWriter *PacketWriter, dispatcher routing.Dispatcher) error { // nolint: lll
	udpServer := udp.NewDispatcher(dispatcher, func(ctx context.Context, packet *udp_proto.Packet) {
		common.Must(clientWriter.WriteMultiBufferWithMetadata(buf.MultiBuffer{packet.Payload}, packet.Source))
	})

	inbound := session.InboundFromContext(ctx)
	user := inbound.User

	for {
		select {
		case <-ctx.Done():
			return nil
		default:
			p, err := clientReader.ReadMultiBufferWithMetadata()
			if err != nil {
				if errors.Cause(err) != io.EOF {
					return newError("unexpected EOF").Base(err)
				}
				return nil
			}

			ctx = log.ContextWithAccessMessage(ctx, &log.AccessMessage{
				From:   inbound.Source,
				To:     p.Target,
				Status: log.AccessAccepted,
				Reason: "",
				Email:  user.Email,
			})
			newError("tunnelling request to ", p.Target).WriteToLog(session.ExportIDToError(ctx))

			for _, b := range p.Buffer {
				udpServer.Dispatch(ctx, p.Target, b)
			}
		}
	}
}

```

该函数`handleConnection`的作用是处理客户端与服务器之间的连接。它接受一个`Server`类型的参数，用于表示服务器对象。

以下是函数的主要步骤：

1. 设置一个定时器，以防止客户端在连接不在活性状态时发送请求。
2. 通过调用`dispatcher.Dispatch`函数将客户端请求路由到服务器，并返回一个已经处理过的`link.Writer`。
3. 创建一个名为`requestDone`的函数，用于处理客户端请求。它使用`buf.Copy`函数将客户端缓冲区中的数据复制到服务器输出缓冲区，并使用`link.Writer`写入客户端请求数据。
4. 创建一个名为`responseDone`的函数，用于处理服务器响应。它使用`buf.Copy`函数将服务器输入缓冲区中的数据复制到客户端写入缓冲区，并使用`link.Reader`从客户端读取响应数据。
5. 创建一个名为`task.Run`的函数，用于运行`requestDonePost`和`responseDone`两个任务。如果一个任务失败，它将调用`requestDonePost`和`responseDone`中的错误处理函数，并尝试使用`common.Must.Interrupt`和`common.Must.Interrupt`函数来挂起和唤醒客户端和 server。
6. 如果所有任务都成功完成，函数将返回一个`nil`表示成功。否则，函数将返回一个错误。


```go
func (s *Server) handleConnection(ctx context.Context, sessionPolicy policy.Session,
	destination net.Destination,
	clientReader buf.Reader,
	clientWriter buf.Writer, dispatcher routing.Dispatcher) error {
	ctx, cancel := context.WithCancel(ctx)
	timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
	ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)

	link, err := dispatcher.Dispatch(ctx, destination)
	if err != nil {
		return newError("failed to dispatch request to ", destination).Base(err)
	}

	requestDone := func() error {
		defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)

		if err := buf.Copy(clientReader, link.Writer, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to transfer request").Base(err)
		}
		return nil
	}

	responseDone := func() error {
		defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)

		if err := buf.Copy(link.Reader, clientWriter, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to write response").Base(err)
		}
		return nil
	}

	var requestDonePost = task.OnSuccess(requestDone, task.Close(link.Writer))
	if err := task.Run(ctx, requestDonePost, responseDone); err != nil {
		common.Must(common.Interrupt(link.Reader))
		common.Must(common.Interrupt(link.Writer))
		return newError("connection ends").Base(err)
	}

	return nil
}

```

This is a Go function that implements the PROXY protocol. It functions to send a request and receive a response from another endpoint using the PROXY protocol.

The function takes two parameters: a remote address and a local address for the endpoint, and two parameters for the request and response data. The remote address is passed as an argument to the `net.ParseIP` function to convert it to a byte array that can be written to the network.

The function sends the request data first to the remote endpoint using the `net.Write` method and the `net.ParseIP` function to convert the remote address to a byte array. Then, it sends the request data to the remote endpoint using the `net.WriteMultiBuffer` method.

If the request is successful, the function returns without returning any errors. If the request fails, the function returns a custom error with information about the failure.

The function also implements a fallback mechanism in case the request fails. In this case, the function reads the response data from the local endpoint and sends it to the remote endpoint using the `net.WriteMultiBuffer` method. This is done in a loop that runs for a period of 60 seconds, or until a new response is received.

The function uses the `buf.Copy` method to copy the request and response data between the `buf.Reader` and `buf.Writer` methods. It also uses the `task.Run` method to run the function asynchronously with the `serverWriter` and `buf.Writer` constants.

Finally, the function uses a combination of ` common.Must2` and ` common.Must2` macros to handle errors and ensure that the connection is closed properly.


```go
func (s *Server) fallback(ctx context.Context, sid errors.ExportOption, err error, sessionPolicy policy.Session, connection internet.Connection, iConn internet.Connection, apfb map[string]map[string]*Fallback, first *buf.Buffer, firstLen int64, reader buf.Reader) error { // nolint: lll
	if err := connection.SetReadDeadline(time.Time{}); err != nil {
		newError("unable to set back read deadline").Base(err).AtWarning().WriteToLog(sid)
	}
	newError("fallback starts").Base(err).AtInfo().WriteToLog(sid)

	alpn := ""
	if len(apfb) > 1 || apfb[""] == nil {
		if tlsConn, ok := iConn.(*tls.Conn); ok {
			alpn = tlsConn.ConnectionState().NegotiatedProtocol
			newError("realAlpn = " + alpn).AtInfo().WriteToLog(sid)
		} else if xtlsConn, ok := iConn.(*xtls.Conn); ok {
			alpn = xtlsConn.ConnectionState().NegotiatedProtocol
			newError("realAlpn = " + alpn).AtInfo().WriteToLog(sid)
		}
		if apfb[alpn] == nil {
			alpn = ""
		}
	}
	pfb := apfb[alpn]
	if pfb == nil {
		return newError(`failed to find the default "alpn" config`).AtWarning()
	}

	path := ""
	if len(pfb) > 1 || pfb[""] == nil {
		if firstLen >= 18 && first.Byte(4) != '*' { // not h2c
			firstBytes := first.Bytes()
			for i := 4; i <= 8; i++ { // 5 -> 9
				if firstBytes[i] == '/' && firstBytes[i-1] == ' ' {
					search := len(firstBytes)
					if search > 64 {
						search = 64 // up to about 60
					}
					for j := i + 1; j < search; j++ {
						k := firstBytes[j]
						if k == '\r' || k == '\n' { // avoid logging \r or \n
							break
						}
						if k == ' ' {
							path = string(firstBytes[i:j])
							newError("realPath = " + path).AtInfo().WriteToLog(sid)
							if pfb[path] == nil {
								path = ""
							}
							break
						}
					}
					break
				}
			}
		}
	}
	fb := pfb[path]
	if fb == nil {
		return newError(`failed to find the default "path" config`).AtWarning()
	}

	ctx, cancel := context.WithCancel(ctx)
	timer := signal.CancelAfterInactivity(ctx, cancel, sessionPolicy.Timeouts.ConnectionIdle)
	ctx = policy.ContextWithBufferPolicy(ctx, sessionPolicy.Buffer)

	var conn net.Conn
	if err := retry.ExponentialBackoff(5, 100).On(func() error {
		var dialer net.Dialer
		conn, err = dialer.DialContext(ctx, fb.Type, fb.Dest)
		if err != nil {
			return err
		}
		return nil
	}); err != nil {
		return newError("failed to dial to " + fb.Dest).Base(err).AtWarning()
	}
	defer conn.Close()

	serverReader := buf.NewReader(conn)
	serverWriter := buf.NewWriter(conn)

	postRequest := func() error {
		defer timer.SetTimeout(sessionPolicy.Timeouts.DownlinkOnly)
		if fb.Xver != 0 {
			remoteAddr, remotePort, err := net.SplitHostPort(connection.RemoteAddr().String())
			if err != nil {
				return err
			}
			localAddr, localPort, err := net.SplitHostPort(connection.LocalAddr().String())
			if err != nil {
				return err
			}
			ipv4 := true
			for i := 0; i < len(remoteAddr); i++ {
				if remoteAddr[i] == ':' {
					ipv4 = false
					break
				}
			}
			pro := buf.New()
			defer pro.Release()
			switch fb.Xver {
			case 1:
				if ipv4 {
					common.Must2(pro.Write([]byte("PROXY TCP4 " + remoteAddr + " " + localAddr + " " + remotePort + " " + localPort + "\r\n")))
				} else {
					common.Must2(pro.Write([]byte("PROXY TCP6 " + remoteAddr + " " + localAddr + " " + remotePort + " " + localPort + "\r\n")))
				}
			case 2:
				common.Must2(pro.Write([]byte("\x0D\x0A\x0D\x0A\x00\x0D\x0A\x51\x55\x49\x54\x0A\x21"))) // signature + v2 + PROXY
				if ipv4 {
					common.Must2(pro.Write([]byte("\x11\x00\x0C"))) // AF_INET + STREAM + 12 bytes
					common.Must2(pro.Write(net.ParseIP(remoteAddr).To4()))
					common.Must2(pro.Write(net.ParseIP(localAddr).To4()))
				} else {
					common.Must2(pro.Write([]byte("\x21\x00\x24"))) // AF_INET6 + STREAM + 36 bytes
					common.Must2(pro.Write(net.ParseIP(remoteAddr).To16()))
					common.Must2(pro.Write(net.ParseIP(localAddr).To16()))
				}
				p1, _ := strconv.ParseUint(remotePort, 10, 16)
				p2, _ := strconv.ParseUint(localPort, 10, 16)
				common.Must2(pro.Write([]byte{byte(p1 >> 8), byte(p1), byte(p2 >> 8), byte(p2)}))
			}
			if err := serverWriter.WriteMultiBuffer(buf.MultiBuffer{pro}); err != nil {
				return newError("failed to set PROXY protocol v", fb.Xver).Base(err).AtWarning()
			}
		}
		if err := buf.Copy(reader, serverWriter, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to fallback request payload").Base(err).AtInfo()
		}
		return nil
	}

	writer := buf.NewWriter(connection)

	getResponse := func() error {
		defer timer.SetTimeout(sessionPolicy.Timeouts.UplinkOnly)
		if err := buf.Copy(serverReader, writer, buf.UpdateActivity(timer)); err != nil {
			return newError("failed to deliver response payload").Base(err).AtInfo()
		}
		return nil
	}

	if err := task.Run(ctx, task.OnSuccess(postRequest, task.Close(serverWriter)), task.OnSuccess(getResponse, task.Close(writer))); err != nil {
		common.Must(common.Interrupt(serverReader))
		common.Must(common.Interrupt(serverWriter))
		return newError("fallback ends").Base(err).AtInfo()
	}

	return nil
}

```

# `proxy/trojan/trojan.go`

这段代码定义了一个名为 "trojan" 的包，其中包含了以下内容：


//定义了一个名为 "Seq" 的序列类型
typedef struct {
   int valid;
   int frag;
   int start;
   int size;
} Seq;

//定义了一个名为 "trojan" 的包
typedef struct {
   int version;
   Seq sequence[100];
} Trojan;


这段代码定义了一个名为 "trojan" 的包，其中包含了一个名为 "Seq" 的序列类型和一个名为 "trojan" 的结构体。这个结构体包含了一个名为 "version" 的整数类型和一个名为 "sequence" 的数组类型，数组中每个元素都是一个名为 "Seq" 的结构体。这个 "trojan" 结构体还包含一个名为 "start" 的整数类型和一个名为 "size" 的整数类型，分别用于计数序列中包含的有效字符数量和最大长度。


```go
package trojan

```

# `proxy/trojan/validator.go`

这段代码定义了一个名为"trojan"的包，它包含了一个名为"Validator"的结构体。

"Validator"结构体包含一个名为"users"的共享Map，用于存储验证用户的有效信息。

此外，它还导入了一个名为"sync"的包，它提供了用于并发执行的包。

最后，它导入了"v2ray.com/core/common/protocol"包，该包可能用于与远程桌面客户端进行通信的协议。


```go
package trojan

import (
	"sync"

	"v2ray.com/core/common/protocol"
)

// Validator stores valid trojan users
type Validator struct {
	users sync.Map
}

// Add a trojan user
func (v *Validator) Add(u *protocol.MemoryUser) error {
	user := u.Account.(*MemoryAccount)
	v.users.Store(hexString(user.Key), u)
	return nil
}

```

这段代码定义了一个名为`Validator`的接口中的一个名为`Get`的方法，该方法接受一个散列键（即密码哈希）和一个需要获取的用户ID，并返回一个指向内存中的`MemoryUser`的指针（该指针类型定义了该接口需要满足的要求）。

首先，该方法首先从名为`v.users`的内部状态中查找具有给定哈希的`User`实例。如果找到该用户，则直接返回该用户的`MemoryUser`指针，否则返回`nil`。

这里使用了Go语言中的类型转换语法，将`u`作为类型转换为`*protocol.MemoryUser`，以便在方法中可以安全地使用`MemoryUser`接口。


```go
// Get user with hashed key, nil if user doesn't exist.
func (v *Validator) Get(hash string) *protocol.MemoryUser {
	u, _ := v.users.Load(hash)
	if u != nil {
		return u.(*protocol.MemoryUser)
	}
	return nil
}

```

# `proxy/vless/account.go`

这段代码定义了一个名为 "vless" 的包，其中包含了一个名为 "AsAccount" 的函数。该函数接受一个 "Account" 类型的参数，并返回一个实现了 "protocol.Account" 和 "AsAccount" 接口的对象，以及一个可能的错误。

具体来说，该函数首先通过调用 "uuid.ParseString" 函数来尝试将传入的 "Id" 字段解析为 UUID 类型。如果解析失败，函数将返回一个空对象并记录错误信息，具体错误信息将在后面记录并输出。如果解析成功，函数将创建一个名为 "MemoryAccount" 的实现了 "protocol.Account" 和 "AsAccount" 接口的对象，其中 "ID" 字段存储了 UUID，而 "Flow" 和 "Encryption" 字段可能是需要进一步解析的参数。

最后，函数将返回 "AsAccount" 对象，以及一个可能的错误对象。


```go
// +build !confonly

package vless

import (
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/uuid"
)

// AsAccount implements protocol.Account.AsAccount().
func (a *Account) AsAccount() (protocol.Account, error) {
	id, err := uuid.ParseString(a.Id)
	if err != nil {
		return nil, newError("failed to parse ID").Base(err).AtError()
	}
	return &MemoryAccount{
		ID:         protocol.NewID(id),
		Flow:       a.Flow,       // needs parser here?
		Encryption: a.Encryption, // needs parser here?
	}, nil
}

```

这段代码定义了一个名为`MemoryAccount`的内存账单结构体。内存账单是一种轻量级的虚拟账单，通常用于在不受任何安全保护的环境中进行客户端连接。

该结构体包含以下字段：

- `ID`：账单的唯一ID，使用ProtocolID类型表示。
- `Flow`：账单的流量，可以是"xtls-rprx-origin"等。
- `Encryption`：账单的加密，目前仅接受"none"。

该结构体还重写了`Equals`方法，用于比较两个`MemoryAccount`是否相等。如果两个`MemoryAccount`的ID相等，则返回`true`，否则返回`false`。


```go
// MemoryAccount is an in-memory form of VLess account.
type MemoryAccount struct {
	// ID of the account.
	ID *protocol.ID
	// Flow of the account. May be "xtls-rprx-origin".
	Flow string
	// Encryption of the account. Used for client connections, and only accepts "none" for now.
	Encryption string
}

// Equals implements protocol.Account.Equals().
func (a *MemoryAccount) Equals(account protocol.Account) bool {
	vlessAccount, ok := account.(*MemoryAccount)
	if !ok {
		return false
	}
	return a.ID.Equals(vlessAccount.ID)
}

```

# `proxy/vless/account.pb.go`

这段代码定义了一个名为 "vless" 的包，其中包括了名为 "account" 的类型。它使用了 Google Protocol Buffers 的语法，用于生成 Go 语言中的接口或接口的实现。

首先，定义了一系列来自 "google.golang.org/protobuf/proto" 和 "github.com/golang/protobuf/proto" 的依赖。接着，定义了一个名为 "reflect" 的依赖，来自 "reflect" 包，它提供了用于在 "vless" 包中使用反射的函数和类型。然后，定义了一个名为 "sync" 的依赖，来自 "sync" 包，它提供了用于在 "vless" 包中使用同步的函数和类型。

最后，定义了一个名为 "account" 的类型，它定义了 "account" 包所需的接口，该接口可以被 "vless" 包中的其他类型实现。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: proxy/vless/account.proto

package vless

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

```

这段代码是一个Go语言中的const变量，包含两个判断，一个是检查当前生成的代码是否足够更新，另一个是检查运行时/protoimpl包是否足够更新。如果两个判断都为true，则执行以下操作：强制使用一个足够更新版本的岭协议包，然后检查该版本是否为4，如果不是4，则抛出错误。

Account结构体包含一个string类型的Id成员变量，一个string类型的Flow成员变量和一个string类型的Encryption成员变量。根据需要，这些成员变量都可以映射到Protobuf中的一个相应的字段。


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

type Account struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	// ID of the account, in the form of a UUID, e.g., "66ad4540-b58c-4ad2-9926-ea63445a9b57".
	Id string `protobuf:"bytes,1,opt,name=id,proto3" json:"id,omitempty"`
	// Flow settings. May be "xtls-rprx-origin".
	Flow string `protobuf:"bytes,2,opt,name=flow,proto3" json:"flow,omitempty"`
	// Encryption settings. Only applies to client side, and only accepts "none" for now.
	Encryption string `protobuf:"bytes,3,opt,name=encryption,proto3" json:"encryption,omitempty"`
}

```

这段代码定义了两个函数，以及一个接口类型和一个函数指针变量。

函数1：`func (x *Account) Reset()`，这个函数的作用是重置`x`对象的账目。具体来说，它创建了一个新的`Account`对象（因为 `*x = Account{}`），然后检查是否启用了 `protoimpl.UnsafeEnabled`，如果是，它会在内存中存储一个名为 `mi` 的内存中的指针，它是通过 `protoimpl.X.MessageStateOf(protoimpl.Pointer(x))` 获取的。这个函数的实现与账目对象的调用没有直接关系。

函数2：`func (x *Account) String()`，这个函数的作用是返回`x`对象的字符串表示。它与账目对象的调用没有直接关系。

接口类型：`protoimpl.X`，这个接口定义了 `*Account` 的类型，但没有提供任何方法。这个接口的实现将在调用方定义。

函数指针变量：`*Account`，这个变量是一个指向 `Account` 类型对象的指针。


```go
func (x *Account) Reset() {
	*x = Account{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_vless_account_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Account) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Account) ProtoMessage() {}

```

这段代码定义了两个函数，函数一是将一个指向 Account 类型对象的指针作为参数传递给 ProtoReflect() 函数，函数二是返回一个指向 Account.Descriptor 类型的指针，其中 Descriptor() 函数已经过时，建议使用 Account.protoReflect.Descriptor() 函数。

函数一的作用是：当调用函数二时，如果传入的 x 参数为 nil，则执行函数一中的 if 语句，即判断是否使用了 ProtoReflect() 函数，如果是，则执行 if 语句中的逻辑，否则直接返回文件 Proxy Vless Account 类型对象的对应 MessageOf() 函数，根据这个消息类型对象，返回对应文件 Proxy Vless Account 类型对象的对应 Descriptor() 函数的返回值。

函数二的作用是：返回一个指向 Account.Descriptor 类型对象的指针，其中 Descriptor() 函数返回了文件 Proxy Vless Account 类型对象的对应 Descriptor() 函数的返回值，也就是包含 Account 的元数据信息，如协议名称，方法列表，该函数已经过时，不建议使用。


```go
func (x *Account) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_vless_account_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Account.ProtoReflect.Descriptor instead.
func (*Account) Descriptor() ([]byte, []int) {
	return file_proxy_vless_account_proto_rawDescGZIP(), []int{0}
}

```

这是一个 Go 语言中的函数指针类型，它接收一个指向 Account 结构体的指针变量 x，并分别返回 x 的三个方法 GetId、GetFlow 和 GetEncryption 的返回值，其中 Account 结构体可能包含了一些与账号相关的信息，但这些信息在代码中并未定义。

具体来说，函数指针通过 x 指针访问账号的相关信息，如果 x 不为 nil，则直接返回 x 的三个方法返回值，否则返回一个空字符串。这种函数指针类型非常适合在函数内部切换不同的账号信息，而不需要显式地定义多个账号结构体。


```go
func (x *Account) GetId() string {
	if x != nil {
		return x.Id
	}
	return ""
}

func (x *Account) GetFlow() string {
	if x != nil {
		return x.Flow
	}
	return ""
}

func (x *Account) GetEncryption() string {
	if x != nil {
		return x.Encryption
	}
	return ""
}

```

It looks like a hexadecimal representation of a smiley face表情 😊.



```go
var File_proxy_vless_account_proto protoreflect.FileDescriptor

var file_proxy_vless_account_proto_rawDesc = []byte{
	0x0a, 0x19, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6c, 0x65, 0x73, 0x73, 0x2f, 0x61, 0x63,
	0x63, 0x6f, 0x75, 0x6e, 0x74, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x12, 0x16, 0x76, 0x32, 0x72,
	0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6c,
	0x65, 0x73, 0x73, 0x22, 0x4d, 0x0a, 0x07, 0x41, 0x63, 0x63, 0x6f, 0x75, 0x6e, 0x74, 0x12, 0x0e,
	0x0a, 0x02, 0x69, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x02, 0x69, 0x64, 0x12, 0x12,
	0x0a, 0x04, 0x66, 0x6c, 0x6f, 0x77, 0x18, 0x02, 0x20, 0x01, 0x28, 0x09, 0x52, 0x04, 0x66, 0x6c,
	0x6f, 0x77, 0x12, 0x1e, 0x0a, 0x0a, 0x65, 0x6e, 0x63, 0x72, 0x79, 0x70, 0x74, 0x69, 0x6f, 0x6e,
	0x18, 0x03, 0x20, 0x01, 0x28, 0x09, 0x52, 0x0a, 0x65, 0x6e, 0x63, 0x72, 0x79, 0x70, 0x74, 0x69,
	0x6f, 0x6e, 0x42, 0x53, 0x0a, 0x1a, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e,
	0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6c, 0x65, 0x73, 0x73,
	0x50, 0x01, 0x5a, 0x1a, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f,
	0x72, 0x65, 0x2f, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6c, 0x65, 0x73, 0x73, 0xaa, 0x02,
	0x16, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e, 0x50, 0x72, 0x6f, 0x78,
	0x79, 0x2e, 0x56, 0x6c, 0x65, 0x73, 0x73, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

这段代码定义了一个名为file_proxy_vless_account_proto_rawDesc的变量，其类型为 sync.Once(类型声明为Once，个人猜测)。该变量包含一个名为file_proxy_vless_account_proto_rawDescData的变量，其类型为文件proxy_vless_account_proto_rawDesc类型的数据。file_proxy_vless_account_proto_rawDescGZIP函数返回一个字节切片，其内容是通过CompressGZIP函数压缩后的file_proxy_vless_account_proto_rawDescData。

file_proxy_vless_account_proto_msgTypes变量是一个接口类型，其包含一个名为file_proxy_vless_account_proto_rawDesc的类型。file_proxy_vless_account_proto_goTypes变量是一个接口类型，其中包含一个名为Account的类型和一个名为*Account的类型。


```go
var (
	file_proxy_vless_account_proto_rawDescOnce sync.Once
	file_proxy_vless_account_proto_rawDescData = file_proxy_vless_account_proto_rawDesc
)

func file_proxy_vless_account_proto_rawDescGZIP() []byte {
	file_proxy_vless_account_proto_rawDescOnce.Do(func() {
		file_proxy_vless_account_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vless_account_proto_rawDescData)
	})
	return file_proxy_vless_account_proto_rawDescData
}

var file_proxy_vless_account_proto_msgTypes = make([]protoimpl.MessageInfo, 1)
var file_proxy_vless_account_proto_goTypes = []interface{}{
	(*Account)(nil), // 0: v2ray.core.proxy.vless.Account
}
```

It appears that `file_proxy_vless_account_proto` is a type definition for an Account struct that uses the `file_proxy_vless` package. The structure of the file contents and the number of message fields it defines, as well as the number of optional fields and extensions, are not specified in this file. It is possible that the `file_proxy_vless` package provides additional information about the struct and its fields. Can you provide more context or details about the struct and what you are trying to know?


```go
var file_proxy_vless_account_proto_depIdxs = []int32{
	0, // [0:0] is the sub-list for method output_type
	0, // [0:0] is the sub-list for method input_type
	0, // [0:0] is the sub-list for extension type_name
	0, // [0:0] is the sub-list for extension extendee
	0, // [0:0] is the sub-list for field type_name
}

func init() { file_proxy_vless_account_proto_init() }
func file_proxy_vless_account_proto_init() {
	if File_proxy_vless_account_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_proxy_vless_account_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Account); i {
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
			RawDescriptor: file_proxy_vless_account_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   1,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_proxy_vless_account_proto_goTypes,
		DependencyIndexes: file_proxy_vless_account_proto_depIdxs,
		MessageInfos:      file_proxy_vless_account_proto_msgTypes,
	}.Build()
	File_proxy_vless_account_proto = out.File
	file_proxy_vless_account_proto_rawDesc = nil
	file_proxy_vless_account_proto_goTypes = nil
	file_proxy_vless_account_proto_depIdxs = nil
}

```