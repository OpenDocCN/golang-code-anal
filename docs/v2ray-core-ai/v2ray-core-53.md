# v2ray-core源码解析 53

# `proxy/vmess/encoding/commands.go`

这段代码定义了一个名为“encoding”的包，其中定义了一些错误类型的变量，以及一些导入的函数和变量。

具体来说，这段代码以下几种作用：

1. 导入了多个与编码相关的包，包括：encoding/binary、io、net、protocol、serial、uuid等，以便在整个项目中都能够使用。

2. 定义了三个错误类型变量，分别是：ErrCommandTypeMismatch、ErrUnknownCommand、ErrCommandTooLarge。这些错误类型用于在编码过程中检测到一些预定义的错误情况，如命令类型不匹配、未知的命令或命令过长等。

3. 在程序运行时，如果检测到某个命令的类型与其期望类型不匹配，则会抛出 ErrCommandTypeMismatch 类型的错误；如果检测到某个命令是未知的，则会抛出 ErrUnknownCommand 类型的错误；如果检测到某个命令太长，则会抛出 ErrCommandTooLarge 类型的错误。


```go
package encoding

import (
	"encoding/binary"
	"io"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/common/uuid"
)

var (
	ErrCommandTypeMismatch = newError("Command type mismatch.")
	ErrUnknownCommand      = newError("Unknown command.")
	ErrCommandTooLarge     = newError("Command too large.")
)

```

该函数名为 `MarshalCommand`，其作用是执行给定命令并将结果写入到一个 `io.Writer` 类型的变量中。

函数接收两个参数：一个 `interface{}` 类型的 `command` 和一个 `io.Writer` 类型的 `writer`。首先，函数检查 `command` 是否为 `nil`，如果是，则返回一个名为 `ErrUnknownCommand` 的错误。然后，函数根据给定 `interface{}` 类型来创建一个 `CommandFactory` 类型的变量 `factory`，并将其赋值为 `CommandSwitchAccountFactory` 类型的实例。

接着，函数创建一个长度为 `cmdID` 大小的缓冲区 `buffer`，并将其用于存储命令的数据。然后，函数调用 `factory.Marshal` 函数将 `command` 类型执行，并将结果存储到缓冲区中。

接下来，函数根据需要调整缓冲区的长度以确保不超过 255 个字节，然后使用 `writer.Write` 函数将缓冲区中的数据写入到 `writer` 类型中。最后，函数使用 `common.Must2` 和 `common.Must2` 函数来确保所有 `Write` 函数的参数都正确设置，并返回一个 `nil` 表示成功。


```go
func MarshalCommand(command interface{}, writer io.Writer) error {
	if command == nil {
		return ErrUnknownCommand
	}

	var cmdID byte
	var factory CommandFactory
	switch command.(type) {
	case *protocol.CommandSwitchAccount:
		factory = new(CommandSwitchAccountFactory)
		cmdID = 1
	default:
		return ErrUnknownCommand
	}

	buffer := buf.New()
	defer buffer.Release()

	err := factory.Marshal(command, buffer)
	if err != nil {
		return err
	}

	auth := Authenticate(buffer.Bytes())
	length := buffer.Len() + 4
	if length > 255 {
		return ErrCommandTooLarge
	}

	common.Must2(writer.Write([]byte{cmdID, byte(length), byte(auth >> 24), byte(auth >> 16), byte(auth >> 8), byte(auth)}))
	common.Must2(writer.Write(buffer.Bytes()))
	return nil
}

```

该函数的作用是验证给定的命令（cmdID）和数据是否符合预期，并返回相应的结果。

具体来说，函数首先检查给定的数据是否符合长度为4的要求。如果不符合，函数会返回一个错误（error）并指出问题的原因。

然后，函数会尝试从给定的数据中提取出与期望的auth（认证）相关的字节。如果提取出的字节与期望的auth不匹配，函数会返回一个错误并指出问题的原因。

接下来，函数会根据给定的cmdID来创建一个适当的CommandFactory实例，并使用该实例的Unmarshal函数对数据中从命令ID为1开始到数据末尾的字节进行反序列化。

总结起来，该函数的主要目的是确保给定的命令和数据符合预期，并返回相应的结果。


```go
func UnmarshalCommand(cmdID byte, data []byte) (protocol.ResponseCommand, error) {
	if len(data) <= 4 {
		return nil, newError("insufficient length")
	}
	expectedAuth := Authenticate(data[4:])
	actualAuth := binary.BigEndian.Uint32(data[:4])
	if expectedAuth != actualAuth {
		return nil, newError("invalid auth")
	}

	var factory CommandFactory
	switch cmdID {
	case 1:
		factory = new(CommandSwitchAccountFactory)
	default:
		return nil, ErrUnknownCommand
	}
	return factory.Unmarshal(data[4:])
}

```

这段代码定义了一个名为`CommandFactory`的接口`Marshal`方法和一个名为`CommandSwitchAccountFactory`的结构体。

`CommandFactory`接口的`Marshal`方法接受一个`command`类型和一个`writer`类型为`io.Writer`的函数。该函数将`command`和`writer`参数进行序列化和反序列化。

`CommandSwitchAccountFactory`结构体实现了`CommandFactory`接口，并重写了`Marshal`方法。在`Marshal`方法中，首先检查`command`是否为`*protocol.CommandSwitchAccount`类型，如果不是，则返回错误。如果是，则将主机名存储在`hostStr`字段中，并将其作为`writer`参数的写入内容的一部分。然后，将主机名的长度存储在`len(hostStr)`中，并将其作为`writer`参数的写入内容的一部分。接着，将客户端的端口号存储在`portStr`字段中，并将其作为`writer`参数的写入内容的一部分。然后，将ID的字节数组存储在`idBytes`中，并将其作为`writer`参数的写入内容的一部分。接着，将等级字段存储在`levelStr`中，并将其作为`writer`参数的写入内容的一部分。最后，写入命令的有效最小值。

通过这种方式，`CommandFactory`和`CommandSwitchAccountFactory`可以确保在序列化和反序列化`command`和`writer`参数时，其值按照正确的格式进行传输。


```go
type CommandFactory interface {
	Marshal(command interface{}, writer io.Writer) error
	Unmarshal(data []byte) (interface{}, error)
}

type CommandSwitchAccountFactory struct {
}

func (f *CommandSwitchAccountFactory) Marshal(command interface{}, writer io.Writer) error {
	cmd, ok := command.(*protocol.CommandSwitchAccount)
	if !ok {
		return ErrCommandTypeMismatch
	}

	hostStr := ""
	if cmd.Host != nil {
		hostStr = cmd.Host.String()
	}
	common.Must2(writer.Write([]byte{byte(len(hostStr))}))

	if len(hostStr) > 0 {
		common.Must2(writer.Write([]byte(hostStr)))
	}

	common.Must2(serial.WriteUint16(writer, cmd.Port.Value()))

	idBytes := cmd.ID.Bytes()
	common.Must2(writer.Write(idBytes))
	common.Must2(serial.WriteUint16(writer, cmd.AlterIds))
	common.Must2(writer.Write([]byte{byte(cmd.Level)}))

	common.Must2(writer.Write([]byte{cmd.ValidMin}))
	return nil
}

```

这段代码定义了一个名为"func"的函数，它接受一个名为"f"的参数以及一个字节切片"data"。函数的作用是解码数据并返回两个值，第一个值是一个接口{}，第二个值是错误。

具体来说，函数首先创建一个名为"cmd"的新命令交换器账户"protocol.CommandSwitchAccount"，然后检查数据的长度。如果数据长度为0，函数将返回一个名为"insufficient length"的错误。如果数据长度不为0，函数将解析数据并设置命令交换器账户的"Host"字段为数据中的IP地址，设置命令交换器账户的"Port"字段为数据中的端口号，设置命令交换器账户的ID字段为数据中的UUID，设置命令交换器账户的"AlterIds"字段为数据中的二进制数据，并设置命令交换器账户的"Level"字段为数据中的整数，设置命令交换器账户的"ValidMin"字段为数据中的时间戳。最后，函数将创建的命令交换器账户返回给调用者。如果函数在解析数据时遇到错误，它将返回一个名为"insufficient length"的错误。


```go
func (f *CommandSwitchAccountFactory) Unmarshal(data []byte) (interface{}, error) {
	cmd := new(protocol.CommandSwitchAccount)
	if len(data) == 0 {
		return nil, newError("insufficient length.")
	}
	lenHost := int(data[0])
	if len(data) < lenHost+1 {
		return nil, newError("insufficient length.")
	}
	if lenHost > 0 {
		cmd.Host = net.ParseAddress(string(data[1 : 1+lenHost]))
	}
	portStart := 1 + lenHost
	if len(data) < portStart+2 {
		return nil, newError("insufficient length.")
	}
	cmd.Port = net.PortFromBytes(data[portStart : portStart+2])
	idStart := portStart + 2
	if len(data) < idStart+16 {
		return nil, newError("insufficient length.")
	}
	cmd.ID, _ = uuid.ParseBytes(data[idStart : idStart+16])
	alterIDStart := idStart + 16
	if len(data) < alterIDStart+2 {
		return nil, newError("insufficient length.")
	}
	cmd.AlterIds = binary.BigEndian.Uint16(data[alterIDStart : alterIDStart+2])
	levelStart := alterIDStart + 2
	if len(data) < levelStart+1 {
		return nil, newError("insufficient length.")
	}
	cmd.Level = uint32(data[levelStart])
	timeStart := levelStart + 1
	if len(data) < timeStart {
		return nil, newError("insufficient length.")
	}
	cmd.ValidMin = data[timeStart]
	return cmd, nil
}

```

# `proxy/vmess/encoding/commands_test.go`

该代码的主要作用是测试 "switch account" 功能，具体解释如下：

1. 定义了一个名为 "sa" 的变量，它是一个 "CommandSwitchAccount" 类型的数据结构，包含了该账户的端口、ID、更改ID数量级和最小允许等级等信息。

2. 创建了一个 "buf" 类型的缓冲区，并使用 "MarshalCommand" 函数将 "sa" 类型数据编码为字节数组，以便将该数据存储到缓冲区中。

3. 使用 "UnmarshalCommand" 函数将编码后的数据存储到缓冲区的第二个元素，并从该缓冲区中提取数据，然后使用 "CommandSwitchAccount" 类型的数据结构来存储该数据。

4. 接下来，定义了一个 "sa2" 变量，与 "sa" 变量中存储的数据进行比较，以验证 "switch account" 功能是否正常工作。

5. 最后，使用 "cmp" 库中的 "Diff" 函数来比较 "sa2" 和 "sa" 变量之间的差异，如果两个变量之间的差异为空字符串，则表示 "switch account" 功能正常。否则，函数将返回一个包含比较结果的错误消息。


```go
package encoding_test

import (
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/uuid"
	. "v2ray.com/core/proxy/vmess/encoding"
)

func TestSwitchAccount(t *testing.T) {
	sa := &protocol.CommandSwitchAccount{
		Port:     1234,
		ID:       uuid.New(),
		AlterIds: 1024,
		Level:    128,
		ValidMin: 16,
	}

	buffer := buf.New()
	common.Must(MarshalCommand(sa, buffer))

	cmd, err := UnmarshalCommand(1, buffer.BytesFrom(2))
	common.Must(err)

	sa2, ok := cmd.(*protocol.CommandSwitchAccount)
	if !ok {
		t.Fatal("failed to convert command to CommandSwitchAccount")
	}
	if r := cmp.Diff(sa2, sa); r != "" {
		t.Error(r)
	}
}

```

# `proxy/vmess/encoding/encoding.go`

这段代码定义了一个名为"encoding"的包，其中包含了以下几行内容：

1. 导入两个名为"v2ray.com/core/common/net"和"v2ray.com/core/common/protocol"的包。

2. 定义了一个名为"Version"的常量，其值为1。

3. 定义了一个名为"addrParser"的变量，它是一个名为"protocol.NewAddressParser"的函数的新地址解析器。该函数的第一个参数是一个字节数组，其中包含协议地址类型，第二个参数是一个IPv4地址家族和一个IPv6地址家族，第三个参数是一个字符串，表示地址类型为域名，第四个参数是一个字符串，表示地址类型为IPv6，第五个参数是一个字符串，表示端口号。

6. 以上函数可以用来创建一个地址解析器，用于解析IPv4和IPv6地址，并将解析到的地址添加到"v2ray.com/core/common/net"包中的"地址"函数中。


```go
package encoding

import (
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
)

//go:generate go run v2ray.com/core/common/errors/errorgen

const (
	Version = byte(1)
)

var addrParser = protocol.NewAddressParser(
	protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv4), net.AddressFamilyIPv4),
	protocol.AddressFamilyByte(byte(protocol.AddressTypeDomain), net.AddressFamilyDomain),
	protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv6), net.AddressFamilyIPv6),
	protocol.PortThenAddress(),
)

```

# `proxy/vmess/encoding/encoding_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 v2ray 代理中的 encoding 功能。它包括以下主要部分：

1. 导入所需的依赖：

	* go-cmp/cmp：用于比较 JSON 数据结构。
	* testing：用于测试框架。
	* v2ray.com/core/common：包含一些通用的工具函数和变量。
	* v2ray.com/core/common/buf：包含一些通用的字符串操作。
	* v2ray.com/core/common/net：包含一些通用的网络操作。
	* v2ray.com/core/common/protocol：包含一些通用的协议操作。
	* v2ray.com/core/proxy/vmess：包含一个 v2ray 代理的 VMESS 接口。
	* v2ray.com/core/proxy/vmess/encoding：一个编码器 VMESS 接口，这个接口与上面包含的 v2ray.com/core/proxy/vmess 中的 VMESS 接口一起实现对 JSON 数据进行编码和解码。
	* "github.com/google/go-cmp/cmp"：Go 标准库中的用于比较 JSON 数据结构的原子类型。

2. 定义一个名为 "test\_encoding" 的函数，它接收一个 VMESS 连接和一个 JSON 数据切片：

	* 通过 `vmess.Dial` 方法建立一个 VMESS 连接。
	* 通过 `vmess.Listen` 方法监听这个连接，然后等待新的客户端连接。
	* 通过 `vmess.SendMessage` 方法发送一个 JSON 数据切片到客户端。
	* 通过 `vmess.ReceiveMessage` 方法接收来自客户端的 JSON 数据切片。
	* 通过一系列的数据操作，对接收到的 JSON 数据切片进行编码和解码。
	* 最后，通过 `cmp.Dif` 函数比较原始 JSON 数据和编码后的 JSON 数据之间的差异，并输出结果。

3. 在 "test\_encoding" 的函数中，定义了一些通用的函数：

	* `createtest连接`：用于创建一个用于测试的 VMESS 连接。
	* `createJSON数据切片`：用于创建一个 JSON 数据切片。
	* `create编码器`：用于创建一个编码器 VMESS 接口。
	* `create解码器`：用于创建一个解码器 VMESS 接口。
	* `测试`：用于测试 "createJSON数据切片" 和 "create编码器" 函数。

4. 包含一些用于测试的函数：

	* `test\_create编码器`：测试 "create编码器" 函数。
	* `test\_create解码器`：测试 "create解码器" 函数。
	* `test\_连接`：测试建立 VMESS 连接的方法。
	* `test\_send消息`：测试发送 JSON 数据切片到客户端的方法。
	* `test\_接收消息`：测试接收 JSON 数据切片的方法。
	* `test\_编码和解码`：测试对 JSON 数据进行编码和解码的方法。


```go
package encoding_test

import (
	"context"
	"testing"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/uuid"
	"v2ray.com/core/proxy/vmess"
	. "v2ray.com/core/proxy/vmess/encoding"
)

```

This is a Go test that checks the serialization of a vmess account request. It uses the gRPC-Go library to interact with the vmess server.

The test creates a vmess user, an account, and a server session. It then sets the user's account to the account and creates a request header and another buffer with the same request header. It then sends the request header to the server and decodes it back to the original account.

It uses the `cmp` package to compare the original request header and the decoded request header. If they are not the same, the test panics with an error message. If the server can't decode the request, the test panics with a different error message.


```go
func toAccount(a *vmess.Account) protocol.Account {
	account, err := a.AsAccount()
	common.Must(err)
	return account
}

func TestRequestSerialization(t *testing.T) {
	user := &protocol.MemoryUser{
		Level: 0,
		Email: "test@v2ray.com",
	}
	id := uuid.New()
	account := &vmess.Account{
		Id:      id.String(),
		AlterId: 0,
	}
	user.Account = toAccount(account)

	expectedRequest := &protocol.RequestHeader{
		Version:  1,
		User:     user,
		Command:  protocol.RequestCommandTCP,
		Address:  net.DomainAddress("www.v2ray.com"),
		Port:     net.Port(443),
		Security: protocol.SecurityType_AES128_GCM,
	}

	buffer := buf.New()
	client := NewClientSession(true, protocol.DefaultIDHash, context.TODO())
	common.Must(client.EncodeRequestHeader(expectedRequest, buffer))

	buffer2 := buf.New()
	buffer2.Write(buffer.Bytes())

	sessionHistory := NewSessionHistory()
	defer common.Close(sessionHistory)

	userValidator := vmess.NewTimedUserValidator(protocol.DefaultIDHash)
	userValidator.Add(user)
	defer common.Close(userValidator)

	server := NewServerSession(userValidator, sessionHistory)
	actualRequest, err := server.DecodeRequestHeader(buffer)
	common.Must(err)

	if r := cmp.Diff(actualRequest, expectedRequest, cmp.AllowUnexported(protocol.ID{})); r != "" {
		t.Error(r)
	}

	_, err = server.DecodeRequestHeader(buffer2)
	// anti replay attack
	if err == nil {
		t.Error("nil error")
	}
}

```

这段代码是一个名为“TestInvalidRequest”的函数，属于“testing”包，用于测试是否正确地处理了当用户发送一个无效请求时服务器应该如何响应。该函数使用了*testing.T，这意味着它使用隐式断言的方式来测试函数。

以下是函数的步骤：

1. 创建一个名为“user”的内存用户对象，该用户的级别为0，电子邮件地址为“test@v2ray.com”。

2. 使用uuid.New()生成一个唯一的学生ID。

3. 创建一个名为“account”的虚拟专用网络（VMess）账户对象。该账户的ID属性设置为生成的学生ID，且没有 alterId，说明该账户是新生用户。

4. 将用户对象的“account”设置为VMess中的“toAccount”函数的输出，即将账户映射到一个虚拟专用网络中。

5. 创建一个预期请求，该请求的格式如下：

	* 版本为1
	* 用户对象使用当前用户的ID
	* 命令为100
	* 地址为“www.v2ray.com”
	* 端口为443
	* 加密类型为AES128-GCM

	* 缓冲区用于存储请求数据

	* 创建一个名为“buffer”的缓冲区，并使用buf.New()方法创建

	* 使用NewClientSession(true, protocol.DefaultIDHash, context.TODO())创建一个客户端会话，并使用该会话的encodeRequestHeader函数将请求编码为字节数组。

	* 使用NewSessionHistory()创建一个会话历史记录，使用common.Close()关闭会话历史记录。

	* 创建一个名为“userValidator”的虚拟专用网络（VMess）用户验证器，并使用protocol.DefaultIDHash对其进行初始化。
	* 使用Add对用户对象进行添加，并使用userValidator.Add对用户进行添加。
	* 使用NewServerSession(userValidator, sessionHistory)创建一个服务器会话，并使用该会话的decodeRequestHeader函数将请求从字节数组中解码为结构体。
	* 如果解码成功，则使用t.Error()函数输出“nil error”，否则函数不会做任何错误处理，可能会导致程序崩溃或泄漏资源。


```go
func TestInvalidRequest(t *testing.T) {
	user := &protocol.MemoryUser{
		Level: 0,
		Email: "test@v2ray.com",
	}
	id := uuid.New()
	account := &vmess.Account{
		Id:      id.String(),
		AlterId: 0,
	}
	user.Account = toAccount(account)

	expectedRequest := &protocol.RequestHeader{
		Version:  1,
		User:     user,
		Command:  protocol.RequestCommand(100),
		Address:  net.DomainAddress("www.v2ray.com"),
		Port:     net.Port(443),
		Security: protocol.SecurityType_AES128_GCM,
	}

	buffer := buf.New()
	client := NewClientSession(true, protocol.DefaultIDHash, context.TODO())
	common.Must(client.EncodeRequestHeader(expectedRequest, buffer))

	buffer2 := buf.New()
	buffer2.Write(buffer.Bytes())

	sessionHistory := NewSessionHistory()
	defer common.Close(sessionHistory)

	userValidator := vmess.NewTimedUserValidator(protocol.DefaultIDHash)
	userValidator.Add(user)
	defer common.Close(userValidator)

	server := NewServerSession(userValidator, sessionHistory)
	_, err := server.DecodeRequestHeader(buffer)
	if err == nil {
		t.Error("nil error")
	}
}

```

这段代码是一个名为`TestMuxRequest`的测试函数，用于测试是否正确地实现了MuxRequest的功能。

该函数的作用如下：

1. 创建一个名为`user`的内存用户，设置其等级为0，电子邮件为`test@v2ray.com`。
2. 创建一个名为`account`的虚拟机消息账户，设置其ID为`toAccount`，并将账户设置为通过`toAccount`函数得到的虚拟机消息账户。
3. 将虚拟机消息账户设置为用户`user`的账户，即将`user.Account`设置为`toAccount`函数得到的虚拟机消息账户。
4. 创建一个预期请求`expectedRequest`，该请求包含用户`user`，虚拟机消息账户`account`，命令为`protocol.RequestCommandMux`，安全措施为`protocol.SecurityType_AES128_GCM`，目标地址为`net.DomainAddress("v1.mux.cool")`。
5. 创建一个缓冲区`buffer`和一个名为`client`的客户端会话，并使用新创建的客户端会话发送`expectedRequest`到服务器。
6. 创建一个名为`buffer2`的缓冲区，并使用客户端会话从服务器接收返回的`expectedRequest`。
7. 创建一个名为`sessionHistory`的会话历史记录，使用客户端会话从服务器接收和存储`expectedRequest`。
8. 创建一个名为`userValidator`的虚拟机消息验证器，使用用户ID为`protocol.DefaultIDHash`。
9. 使用上面创建的验证器将用户`user`添加到会话历史记录中。
10. 使用客户端会话创建一个服务器会话，使用上面创建的验证器接收来自客户端的`expectedRequest`，并使用上面创建的会话历史记录来处理请求。
11. 如果返回的`expectedRequest`与`expectedRequest`不同，则测试失败。


```go
func TestMuxRequest(t *testing.T) {
	user := &protocol.MemoryUser{
		Level: 0,
		Email: "test@v2ray.com",
	}
	id := uuid.New()
	account := &vmess.Account{
		Id:      id.String(),
		AlterId: 0,
	}
	user.Account = toAccount(account)

	expectedRequest := &protocol.RequestHeader{
		Version:  1,
		User:     user,
		Command:  protocol.RequestCommandMux,
		Security: protocol.SecurityType_AES128_GCM,
		Address:  net.DomainAddress("v1.mux.cool"),
	}

	buffer := buf.New()
	client := NewClientSession(true, protocol.DefaultIDHash, context.TODO())
	common.Must(client.EncodeRequestHeader(expectedRequest, buffer))

	buffer2 := buf.New()
	buffer2.Write(buffer.Bytes())

	sessionHistory := NewSessionHistory()
	defer common.Close(sessionHistory)

	userValidator := vmess.NewTimedUserValidator(protocol.DefaultIDHash)
	userValidator.Add(user)
	defer common.Close(userValidator)

	server := NewServerSession(userValidator, sessionHistory)
	actualRequest, err := server.DecodeRequestHeader(buffer)
	common.Must(err)

	if r := cmp.Diff(actualRequest, expectedRequest, cmp.AllowUnexported(protocol.ID{})); r != "" {
		t.Error(r)
	}
}

```

# `proxy/vmess/encoding/errors.generated.go`

这段代码定义了一个名为 "encoding" 的包，其中包含了一个名为 "errPathObjHolder" 的结构体。

该结构体包含一个名为 "errPathObjHolder" 的类型，其成员为空。

另外，该段代码还定义了一个名为 "newError" 的函数，该函数接收多个参数，这些参数可以是任意类型的对象。

函数内部创建了一个名为 "errPathObjHolder" 的新的 "errPathObjHolder" 实例，然后使用 "errors.New" 函数创建一个新的错误对象，该对象使用 "errPathObjHolder" 实例的 "withPathObj" 方法获取一个包含错误路径和异常对象的 "errPathObjHolder" 实例。最后，将错误对象和 "errPathObjHolder" 实例的上下文信息一起返回。


```go
package encoding

import "v2ray.com/core/common/errors"

type errPathObjHolder struct{}

func newError(values ...interface{}) *errors.Error {
	return errors.New(values...).WithPathObj(errPathObjHolder{})
}

```

# `proxy/vmess/encoding/server.go`

这段代码是一个 Go 语言编写的库，名为 "encoding"。它提供了多种密码学哈希算法，包括 AES、MD5 和 SHA256。同时，它还提供了几种编码方式，如字节编码和 Binary 编码。

具体来说，这段代码实现了一些常用的密码学哈希算法，包括 AES-128、AES-192 和 AES-256。对于每个算法，它都会生成一个 256 字节的哈希值，并且使用了不同的初始向量(IV)来提高哈希的性能。

此外，这段代码还提供了一个 HTTP 代理中间件，用于在 V2Ray 代理中传递加密消息。同时，它还提供了一个名为 "vmess" 的接口，用于在 V2Ray 代理中实现代理加密。


```go
package encoding

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/md5"
	"crypto/sha256"
	"encoding/binary"
	"hash/fnv"
	"io"
	"io/ioutil"
	"sync"
	"time"

	"golang.org/x/crypto/chacha20poly1305"
	"v2ray.com/core/common"
	"v2ray.com/core/common/bitmask"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/crypto"
	"v2ray.com/core/common/dice"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/task"
	"v2ray.com/core/proxy/vmess"
	vmessaead "v2ray.com/core/proxy/vmess/aead"
)

```

此代码定义了一个名为 "sessionId" 的结构体，该结构体包含一个用户名[16]字节，一个密钥[16]字节和一个非密码[16]字节。

该代码还定义了一个名为 "SessionHistory" 的结构体，该结构体包含一个共享的 "并发写入多路复用锁"（RWMutex）、一个缓存（map[sessionId]time.Time）和一个定时任务（task.Periodic）。

该代码最后定义了一个名为 "NewSessionHistory" 的函数，该函数返回一个名为 "SessionHistory" 的 pointer，初始化时创建一个空的 SessionHistory 对象并设置其初始值。


```go
type sessionId struct {
	user  [16]byte
	key   [16]byte
	nonce [16]byte
}

// SessionHistory keeps track of historical session ids, to prevent replay attacks.
type SessionHistory struct {
	sync.RWMutex
	cache map[sessionId]time.Time
	task  *task.Periodic
}

// NewSessionHistory creates a new SessionHistory object.
func NewSessionHistory() *SessionHistory {
	h := &SessionHistory{
		cache: make(map[sessionId]time.Time, 128),
	}
	h.task = &task.Periodic{
		Interval: time.Second * 30,
		Execute:  h.removeExpiredEntries,
	}
	return h
}

```

这段代码定义了一个名为 Close 的接口，它实现了 common.Closable.

函数 h.Close() 接收一个 SessionHistory 类型的 h 变量，它尝试关闭当前会话并返回一个错误。

函数 h.addIfNotExits() 接收一个 sessionId 参数。它先尝试从缓存中获取会话信息，如果已经过期则返回 false。然后它尝试在缓存中添加一个新的会话，并使用当前时间戳加上每分钟 3 度的毫秒来保证会话不会过期。最后，它调用了一个名为 mustStart 的函数，它确保会话成功启动。

这段代码的主要目的是实现了一个会话历史功能，管理和维护会话信息，并确保会话不会过期。


```go
// Close implements common.Closable.
func (h *SessionHistory) Close() error {
	return h.task.Close()
}

func (h *SessionHistory) addIfNotExits(session sessionId) bool {
	h.Lock()

	if expire, found := h.cache[session]; found && expire.After(time.Now()) {
		h.Unlock()
		return false
	}

	h.cache[session] = time.Now().Add(time.Minute * 3)
	h.Unlock()
	common.Must(h.task.Start())
	return true
}

```

此代码是一个函数，名为`removeExpiredEntries()`，它接受一个参数`h`，代表一个`SessionHistory`类型的切片。

函数的作用是移除过期的`Session`，其中`Session`是一个`map[sessionId]time.Time`类型的键值对，代表一个`SessionId`类型的值。

具体实现过程如下：

1. 首先获取`SessionHistory`的锁，以确保在遍历过程中只有一个实例被访问。
2. 遍历`SessionHistory`中的所有`Session`，对于每个`Session`，获取它的`expire`时间，如果当前时间`now`早于`expire`，则将其从`SessionHistory`的`cache`中删除。
3. 如果`SessionHistory`的`cache`中仍然存在`Session`，则将其`expire`时间与当前时间`now`比较，如果当前时间`now`早于`expire`，则在`cache`中删除相应的`Session`。
4. 如果`SessionHistory`的`cache`中仍然不存在`Session`，则创建一个包含128个键值对的`map[sessionId]time.Time`类型的键值对，并将当前时间`now`存储到键中。
5. 返回`nil`表示操作成功。

由于对`SessionHistory`的`cache`进行删除操作，可能会导致`Session`的ID重复，从而覆盖之前的值。因此，在删除`Session`时，需要判断`Session`是否存在于`SessionHistory`的`cache`中，以避免覆盖之前的值。


```go
func (h *SessionHistory) removeExpiredEntries() error {
	now := time.Now()

	h.Lock()
	defer h.Unlock()

	if len(h.cache) == 0 {
		return newError("nothing to do")
	}

	for session, expire := range h.cache {
		if expire.Before(now) {
			delete(h.cache, session)
		}
	}

	if len(h.cache) == 0 {
		h.cache = make(map[sessionId]time.Time, 128)
	}

	return nil
}

```

这段代码定义了一个名为ServerSession的 struct 类型，用于在VMess服务器上保存会话信息。

该结构体包含以下字段：

- `userValidator`：一个指向`vmess.TimedUserValidator`类型的指针，用于验证通过的用户信息。
- `sessionHistory`：一个指向`SessionHistory`类型的指针，用于保存会话相关的信息，例如登录时间、会话状态、会话类型等。
- `requestBodyKey`：一个16字节的数组，用于保存请求主体相关的键值数据，例如用户ID、用户密码等。
- `requestBodyIV`：一个16字节的数组，用于保存请求主体IV，例如用户密码、签名等。
- `responseBodyKey`：一个16字节的数组，用于保存响应主体相关的键值数据，例如用户ID、状态信息等。
- `responseBodyIV`：一个16字节的数组，用于保存响应主体IV，例如用户ID、状态信息等。
- `responseWriter`：一个输出写入的函数，用于将ResponseBody写入到`responseBodyKey`所指向的内存中。
- `responseHeader`：一个用于设置HTTP请求头的函数，用于设置请求头中的信息，例如设备类型、会话ID等。
- `isAEADRequest`：一个布尔类型的变量，表示是否是一个AEAD请求会话。
- `isAEADForced`：一个布尔类型的变量，表示是否是一个强制性的AEAD会话。

该结构体可以用于定义一个服务器端的会话，通过`userValidator`验证用户身份、`sessionHistory`记录会话相关的信息、`requestBodyKey`和`requestBodyIV`保存请求相关的数据、`responseBodyKey`和`responseBodyIV`保存响应相关的数据，并通过`responseWriter`将响应数据写入到指定的位置，`responseHeader`用于设置请求头中的信息。


```go
// ServerSession keeps information for a session in VMess server.
type ServerSession struct {
	userValidator   *vmess.TimedUserValidator
	sessionHistory  *SessionHistory
	requestBodyKey  [16]byte
	requestBodyIV   [16]byte
	responseBodyKey [16]byte
	responseBodyIV  [16]byte
	responseWriter  io.Writer
	responseHeader  byte

	isAEADRequest bool

	isAEADForced bool
}

```

这段代码定义了一个名为NewServerSession的函数，它接受两个参数，一个是用户验证器(UserValidator)，另一个是会话历史记录(SessionHistory)，然后返回一个指向ServerSession的指针。

函数的作用是创建一个新的ServerSession实例，这个实例并不拥有UserValidator，也就是说，这个UserValidator仍然是有效的，可以在以后的使用中继续被使用。

函数中还有一个名为parseSecurityType的函数，它接受一个字节(即8位)，然后返回一个表示安全类型(SecurityType)的协议.安全类型由协议定义，用于指定服务器的安全性。

parseSecurityType函数的逻辑是，根据传入的b字节，它会尝试从"/安全类型名称"数组中查找与b相等的元素，如果没有找到，就返回协议定义的未定义类型。如果找到了与b相等的元素，就返回该元素对应的SecurityType类型。

最后，函数返回一个新的ServerSession实例，用于指定新的服务器会话及其会话历史记录。


```go
// NewServerSession creates a new ServerSession, using the given UserValidator.
// The ServerSession instance doesn't take ownership of the validator.
func NewServerSession(validator *vmess.TimedUserValidator, sessionHistory *SessionHistory) *ServerSession {
	return &ServerSession{
		userValidator:  validator,
		sessionHistory: sessionHistory,
	}
}

func parseSecurityType(b byte) protocol.SecurityType {
	if _, f := protocol.SecurityType_name[int32(b)]; f {
		st := protocol.SecurityType(b)
		// For backward compatibility.
		if st == protocol.SecurityType_UNKNOWN {
			st = protocol.SecurityType_LEGACY
		}
		return st
	}
	return protocol.SecurityType_UNKNOWN
}

```

This is a Go function that performs a function to validate an Ethernet海报 (RFC 6282) containing an Ethernet address, also known as a system account or group account. It checks if the Ethernet address is valid and also performs a checksum calculation.

The function takes an Ethernet address as input, but it is assumed that the address is in the format of a memory-typed IPv4 address. The function first checks if the input address is a valid IPv4 address by using the `s.isAddressValid` function. If the address is not a valid IPv4 address, it will raise an error.

The function then performs a checksum calculation using the `fnv1a.New32a()` and `fnv1a.Sum32()` functions. The function then compares the calculated checksum with the expected checksum. If the calculated checksum is different from the expected checksum, the function will raise an error.

Finally, the function checks if the input address is an Ethernet address with the specified security type (e.g., `protocol.SecurityType_UNKNOWN`, `protocol.SecurityType_AUTO`, etc.). If the security type is `protocol.SecurityType_UNKNOWN`, the function will raise an error.

The function returns an error if the input address is not a valid IPv4 address or if the checksum calculation fails.


```go
// DecodeRequestHeader decodes and returns (if successful) a RequestHeader from an input stream.
func (s *ServerSession) DecodeRequestHeader(reader io.Reader) (*protocol.RequestHeader, error) {
	buffer := buf.New()
	behaviorRand := dice.NewDeterministicDice(int64(s.userValidator.GetBehaviorSeed()))
	BaseDrainSize := behaviorRand.Roll(3266)
	RandDrainMax := behaviorRand.Roll(64) + 1
	RandDrainRolled := dice.Roll(RandDrainMax)
	DrainSize := BaseDrainSize + 16 + 38 + RandDrainRolled
	readSizeRemain := DrainSize

	drainConnection := func(e error) error {
		//We read a deterministic generated length of data before closing the connection to offset padding read pattern
		readSizeRemain -= int(buffer.Len())
		if readSizeRemain > 0 {
			err := s.DrainConnN(reader, readSizeRemain)
			if err != nil {
				return newError("failed to drain connection DrainSize = ", BaseDrainSize, " ", RandDrainMax, " ", RandDrainRolled).Base(err).Base(e)
			}
			return newError("connection drained DrainSize = ", BaseDrainSize, " ", RandDrainMax, " ", RandDrainRolled).Base(e)
		}
		return e
	}

	defer func() {
		buffer.Release()
	}()

	if _, err := buffer.ReadFullFrom(reader, protocol.IDBytesLen); err != nil {
		return nil, newError("failed to read request header").Base(err)
	}

	var decryptor io.Reader
	var vmessAccount *vmess.MemoryAccount

	user, foundAEAD, errorAEAD := s.userValidator.GetAEAD(buffer.Bytes())

	var fixedSizeAuthID [16]byte
	copy(fixedSizeAuthID[:], buffer.Bytes())

	if foundAEAD {
		vmessAccount = user.Account.(*vmess.MemoryAccount)
		var fixedSizeCmdKey [16]byte
		copy(fixedSizeCmdKey[:], vmessAccount.ID.CmdKey())
		aeadData, shouldDrain, errorReason, bytesRead := vmessaead.OpenVMessAEADHeader(fixedSizeCmdKey, fixedSizeAuthID, reader)
		if errorReason != nil {
			if shouldDrain {
				readSizeRemain -= bytesRead
				return nil, drainConnection(newError("AEAD read failed").Base(errorReason))
			} else {
				return nil, drainConnection(newError("AEAD read failed, drain skiped").Base(errorReason))
			}
		}
		decryptor = bytes.NewReader(aeadData)
		s.isAEADRequest = true
	} else if !s.isAEADForced && errorAEAD == vmessaead.ErrNotFound {
		userLegacy, timestamp, valid, userValidationError := s.userValidator.Get(buffer.Bytes())
		if !valid || userValidationError != nil {
			return nil, drainConnection(newError("invalid user").Base(userValidationError))
		}
		user = userLegacy
		iv := hashTimestamp(md5.New(), timestamp)
		vmessAccount = userLegacy.Account.(*vmess.MemoryAccount)

		aesStream := crypto.NewAesDecryptionStream(vmessAccount.ID.CmdKey(), iv[:])
		decryptor = crypto.NewCryptionReader(aesStream, reader)
	} else {
		return nil, drainConnection(newError("invalid user").Base(errorAEAD))
	}

	readSizeRemain -= int(buffer.Len())
	buffer.Clear()
	if _, err := buffer.ReadFullFrom(decryptor, 38); err != nil {
		return nil, newError("failed to read request header").Base(err)
	}

	request := &protocol.RequestHeader{
		User:    user,
		Version: buffer.Byte(0),
	}

	copy(s.requestBodyIV[:], buffer.BytesRange(1, 17))   // 16 bytes
	copy(s.requestBodyKey[:], buffer.BytesRange(17, 33)) // 16 bytes
	var sid sessionId
	copy(sid.user[:], vmessAccount.ID.Bytes())
	sid.key = s.requestBodyKey
	sid.nonce = s.requestBodyIV
	if !s.sessionHistory.addIfNotExits(sid) {
		if !s.isAEADRequest {
			drainErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
			if drainErr != nil {
				return nil, drainConnection(newError("duplicated session id, possibly under replay attack, and failed to taint userHash").Base(drainErr))
			}
			return nil, drainConnection(newError("duplicated session id, possibly under replay attack, userHash tainted"))
		} else {
			return nil, newError("duplicated session id, possibly under replay attack, but this is a AEAD request")
		}

	}

	s.responseHeader = buffer.Byte(33)             // 1 byte
	request.Option = bitmask.Byte(buffer.Byte(34)) // 1 byte
	padingLen := int(buffer.Byte(35) >> 4)
	request.Security = parseSecurityType(buffer.Byte(35) & 0x0F)
	// 1 bytes reserved
	request.Command = protocol.RequestCommand(buffer.Byte(37))

	switch request.Command {
	case protocol.RequestCommandMux:
		request.Address = net.DomainAddress("v1.mux.cool")
		request.Port = 0
	case protocol.RequestCommandTCP, protocol.RequestCommandUDP:
		if addr, port, err := addrParser.ReadAddressPort(buffer, decryptor); err == nil {
			request.Address = addr
			request.Port = port
		}
	}

	if padingLen > 0 {
		if _, err := buffer.ReadFullFrom(decryptor, int32(padingLen)); err != nil {
			if !s.isAEADRequest {
				burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
				if burnErr != nil {
					return nil, newError("failed to read padding, failed to taint userHash").Base(burnErr).Base(err)
				}
				return nil, newError("failed to read padding, userHash tainted").Base(err)
			}
			return nil, newError("failed to read padding").Base(err)
		}
	}

	if _, err := buffer.ReadFullFrom(decryptor, 4); err != nil {
		if !s.isAEADRequest {
			burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
			if burnErr != nil {
				return nil, newError("failed to read checksum, failed to taint userHash").Base(burnErr).Base(err)
			}
			return nil, newError("failed to read checksum, userHash tainted").Base(err)
		}
		return nil, newError("failed to read checksum").Base(err)
	}

	fnv1a := fnv.New32a()
	common.Must2(fnv1a.Write(buffer.BytesTo(-4)))
	actualHash := fnv1a.Sum32()
	expectedHash := binary.BigEndian.Uint32(buffer.BytesFrom(-4))

	if actualHash != expectedHash {
		if !s.isAEADRequest {
			Autherr := newError("invalid auth, legacy userHash tainted")
			burnErr := s.userValidator.BurnTaintFuse(fixedSizeAuthID[:])
			if burnErr != nil {
				Autherr = newError("invalid auth, can't taint legacy userHash").Base(burnErr)
			}
			//It is possible that we are under attack described in https://github.com/v2ray/v2ray-core/issues/2523
			return nil, drainConnection(Autherr)
		} else {
			return nil, newError("invalid auth, but this is a AEAD request")
		}

	}

	if request.Address == nil {
		return nil, newError("invalid remote address")
	}

	if request.Security == protocol.SecurityType_UNKNOWN || request.Security == protocol.SecurityType_AUTO {
		return nil, newError("unknown security type: ", request.Security)
	}

	return request, nil
}

```

This is a Go function that wraps a Crypto洲height认证器（Authenticator） for different security types. It checks if the request has a specific security type and, if it does, it creates an appropriate authenticator

The function takes a `crypto.RequestBody` object as input and returns an `crypto.AuthenticationReader` object as output. The `AuthenticationReader` is used to read the entire request body, so that the security


```go
// DecodeRequestBody returns Reader from which caller can fetch decrypted body.
func (s *ServerSession) DecodeRequestBody(request *protocol.RequestHeader, reader io.Reader) buf.Reader {
	var sizeParser crypto.ChunkSizeDecoder = crypto.PlainChunkSizeParser{}
	if request.Option.Has(protocol.RequestOptionChunkMasking) {
		sizeParser = NewShakeSizeParser(s.requestBodyIV[:])
	}
	var padding crypto.PaddingLengthGenerator
	if request.Option.Has(protocol.RequestOptionGlobalPadding) {
		padding = sizeParser.(crypto.PaddingLengthGenerator)
	}

	switch request.Security {
	case protocol.SecurityType_NONE:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			if request.Command.TransferType() == protocol.TransferTypeStream {
				return crypto.NewChunkStreamReader(sizeParser, reader)
			}

			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(NoOpAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationReader(auth, sizeParser, reader, protocol.TransferTypePacket, padding)
		}

		return buf.NewReader(reader)
	case protocol.SecurityType_LEGACY:
		aesStream := crypto.NewAesDecryptionStream(s.requestBodyKey[:], s.requestBodyIV[:])
		cryptionReader := crypto.NewCryptionReader(aesStream, reader)
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(FnvAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationReader(auth, sizeParser, cryptionReader, request.Command.TransferType(), padding)
		}

		return buf.NewReader(cryptionReader)
	case protocol.SecurityType_AES128_GCM:
		aead := crypto.NewAesGcm(s.requestBodyKey[:])

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(s.requestBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
	case protocol.SecurityType_CHACHA20_POLY1305:
		aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(s.requestBodyKey[:]))

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(s.requestBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationReader(auth, sizeParser, reader, request.Command.TransferType(), padding)
	default:
		panic("Unknown security type.")
	}
}

```

This is a Go program that implements the AES encryption and decryption of an incoming message using the EM布拉心存发型式。

The program takes in a header buffer and an encryption key as input parameters. The header buffer must be larger than the encryption key to avoid a syntax error.

The program first encrypts the header buffer using the AES encryption method and specifies the encryption algorithm, key size, and the initialization vector (IV).

Then, it decrypts the encrypted header buffer using the AES decryption method and specifies the encryption algorithm, key size, and the initialization vector (IV).

Finally, it copies the decrypted header buffer to the output writer.

The program uses the "io" package to copy the header buffer to the output writer.


```go
// EncodeResponseHeader writes encoded response header into the given writer.
func (s *ServerSession) EncodeResponseHeader(header *protocol.ResponseHeader, writer io.Writer) {
	var encryptionWriter io.Writer
	if !s.isAEADRequest {
		s.responseBodyKey = md5.Sum(s.requestBodyKey[:])
		s.responseBodyIV = md5.Sum(s.requestBodyIV[:])
	} else {
		BodyKey := sha256.Sum256(s.requestBodyKey[:])
		copy(s.responseBodyKey[:], BodyKey[:16])
		BodyIV := sha256.Sum256(s.requestBodyIV[:])
		copy(s.responseBodyIV[:], BodyIV[:16])
	}

	aesStream := crypto.NewAesEncryptionStream(s.responseBodyKey[:], s.responseBodyIV[:])
	encryptionWriter = crypto.NewCryptionWriter(aesStream, writer)
	s.responseWriter = encryptionWriter

	aeadEncryptedHeaderBuffer := bytes.NewBuffer(nil)

	if s.isAEADRequest {
		encryptionWriter = aeadEncryptedHeaderBuffer
	}

	common.Must2(encryptionWriter.Write([]byte{s.responseHeader, byte(header.Option)}))
	err := MarshalCommand(header.Command, encryptionWriter)
	if err != nil {
		common.Must2(encryptionWriter.Write([]byte{0x00, 0x00}))
	}

	if s.isAEADRequest {

		aeadResponseHeaderLengthEncryptionKey := vmessaead.KDF16(s.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderLenKey)
		aeadResponseHeaderLengthEncryptionIV := vmessaead.KDF(s.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderLenIV)[:12]

		aeadResponseHeaderLengthEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderLengthEncryptionKey)).(cipher.Block)
		aeadResponseHeaderLengthEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderLengthEncryptionKeyAESBlock)).(cipher.AEAD)

		aeadResponseHeaderLengthEncryptionBuffer := bytes.NewBuffer(nil)

		decryptedResponseHeaderLengthBinaryDeserializeBuffer := uint16(aeadEncryptedHeaderBuffer.Len())

		common.Must(binary.Write(aeadResponseHeaderLengthEncryptionBuffer, binary.BigEndian, decryptedResponseHeaderLengthBinaryDeserializeBuffer))

		AEADEncryptedLength := aeadResponseHeaderLengthEncryptionAEAD.Seal(nil, aeadResponseHeaderLengthEncryptionIV, aeadResponseHeaderLengthEncryptionBuffer.Bytes(), nil)
		common.Must2(io.Copy(writer, bytes.NewReader(AEADEncryptedLength)))

		aeadResponseHeaderPayloadEncryptionKey := vmessaead.KDF16(s.responseBodyKey[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadKey)
		aeadResponseHeaderPayloadEncryptionIV := vmessaead.KDF(s.responseBodyIV[:], vmessaead.KDFSaltConst_AEADRespHeaderPayloadIV)[:12]

		aeadResponseHeaderPayloadEncryptionKeyAESBlock := common.Must2(aes.NewCipher(aeadResponseHeaderPayloadEncryptionKey)).(cipher.Block)
		aeadResponseHeaderPayloadEncryptionAEAD := common.Must2(cipher.NewGCM(aeadResponseHeaderPayloadEncryptionKeyAESBlock)).(cipher.AEAD)

		aeadEncryptedHeaderPayload := aeadResponseHeaderPayloadEncryptionAEAD.Seal(nil, aeadResponseHeaderPayloadEncryptionIV, aeadEncryptedHeaderBuffer.Bytes(), nil)
		common.Must2(io.Copy(writer, bytes.NewReader(aeadEncryptedHeaderPayload)))
	}
}

```

This function appears to implement the authentication process for a security transaction using the provided protocol. It checks the security type of the request and creates an appropriate authentication writer based on that type.

The function takes a request object and an optional `responseBody` key. If the `responseBody` key is present, it is passed to the `GenerateChunkNonce` function to generate a nonce for the AES128GCM algorithm. The nonce is then passed to the `NewAesGcm` function to create an AES128GCM authentication object.

If the `responseBody` key is not present, the function defaults to `NewAesGcm` for the AES128GCM algorithm.

The function returns a `crypto.AuthenticationWriter` object, which represents the writer of the response body.

Note that the AES128GCM algorithm is only supported for encryption and decryption, not for signing or data integrity protection.


```go
// EncodeResponseBody returns a Writer that auto-encrypt content written by caller.
func (s *ServerSession) EncodeResponseBody(request *protocol.RequestHeader, writer io.Writer) buf.Writer {
	var sizeParser crypto.ChunkSizeEncoder = crypto.PlainChunkSizeParser{}
	if request.Option.Has(protocol.RequestOptionChunkMasking) {
		sizeParser = NewShakeSizeParser(s.responseBodyIV[:])
	}
	var padding crypto.PaddingLengthGenerator
	if request.Option.Has(protocol.RequestOptionGlobalPadding) {
		padding = sizeParser.(crypto.PaddingLengthGenerator)
	}

	switch request.Security {
	case protocol.SecurityType_NONE:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			if request.Command.TransferType() == protocol.TransferTypeStream {
				return crypto.NewChunkStreamWriter(sizeParser, writer)
			}

			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(NoOpAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationWriter(auth, sizeParser, writer, protocol.TransferTypePacket, padding)
		}

		return buf.NewWriter(writer)
	case protocol.SecurityType_LEGACY:
		if request.Option.Has(protocol.RequestOptionChunkStream) {
			auth := &crypto.AEADAuthenticator{
				AEAD:                    new(FnvAuthenticator),
				NonceGenerator:          crypto.GenerateEmptyBytes(),
				AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
			}
			return crypto.NewAuthenticationWriter(auth, sizeParser, s.responseWriter, request.Command.TransferType(), padding)
		}

		return &buf.SequentialWriter{Writer: s.responseWriter}
	case protocol.SecurityType_AES128_GCM:
		aead := crypto.NewAesGcm(s.responseBodyKey[:])

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(s.responseBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
	case protocol.SecurityType_CHACHA20_POLY1305:
		aead, _ := chacha20poly1305.New(GenerateChacha20Poly1305Key(s.responseBodyKey[:]))

		auth := &crypto.AEADAuthenticator{
			AEAD:                    aead,
			NonceGenerator:          GenerateChunkNonce(s.responseBodyIV[:], uint32(aead.NonceSize())),
			AdditionalDataGenerator: crypto.GenerateEmptyBytes(),
		}
		return crypto.NewAuthenticationWriter(auth, sizeParser, writer, request.Command.TransferType(), padding)
	default:
		panic("Unknown security type.")
	}
}

```

该函数名为 `DrainConnN`，其作用是接收一个 `ServerSession` 类型的参数 `s`，以及一个整数 `n`。它通过 `io.CopyN` 函数从 `reader` 参数中的输入流中读取 `n` 个 `int64` 类型的数据并将其复制到输出流中，然后返回任何错误。

具体来说，函数的实现分两个步骤：

1. 使用 `io.CopyN` 函数将 `reader` 中的数据按照 `n` 取整并复制到 `ioutil.Discard` 类型的输出流中。这里 `ioutil.Discard` 是一个 `io.Writer` 类型的指针，代表一个 discarding 的输出流，可以将其用于 `io.CopyN` 函数的输入和输出。
2. 返回 `err` 变量，即 `DrainConnN` 函数的返回值。

由于 `DrainConnN` 函数没有进行任何错误处理，因此在实际应用中应当对其进行适当的错误处理，以确保代码的健壮性。


```go
func (s *ServerSession) DrainConnN(reader io.Reader, n int) error {
	_, err := io.CopyN(ioutil.Discard, reader, int64(n))
	return err
}

```

# `proxy/vmess/inbound/config.go`

这段代码定义了一个名为`inbound`的包，其中包含一个名为`Config`的类型，该类型有一个名为`GetDefaultValue`的函数。

该函数的参数是一个指向`DefaultConfig`类型的指针`c`，它代表一个配置对象。函数首先检查`c`是否已经设置了一个默认值，如果没有，则返回`DefaultConfig`类型的默认设置，否则使用`c`设置的默认值。

具体来说，如果`c`还没有设置默认值，那么将创建一个新的`DefaultConfig`对象，其中`AlterId`设置为32,`Level`设置为0。如果已经设置了一个默认值，那么直接返回`c`所设置的默认值。

最后，该函数使用了`!confonly`参数，表示仅在函数执行时才写入配置文件，这样可以避免在每次启动应用程序时都需要重新配置应用程序。


```go
// +build !confonly

package inbound

// GetDefaultValue returns default settings of DefaultConfig.
func (c *Config) GetDefaultValue() *DefaultConfig {
	if c.GetDefault() == nil {
		return &DefaultConfig{
			AlterId: 32,
			Level:   0,
		}
	}
	return c.Default
}

```

# `proxy/vmess/inbound/config.pb.go`

这段代码定义了一个名为 "inbound" 的包，用于定义了在v2ray协议中的inbound功能。

首先，定义了两个protoc版本：protoc-gen-go v1.25.0和protoc v3.13.0。

然后，定义了以下message定义：

protoreflect "google.golang.org/protobuf/proto"



protoc-gen-go v1.25.0


接着，定义了以下2个函数：

func Config(config *Protocol.Config) error

func DefaultConfig() *Protocol.Config


配置函数接受一个Protocol.Config类型的参数，返回一个错误。DefaultConfig函数返回一个默认的Protocol.Config，包含一些默认的inbound配置。

接下来，定义了以下Protocol消息类型：

type Config(version int, options map[string]interface{}) *Protocol.Config

type ConfigMessage(config *Protocol.Config) *Protocol.Config


然后，定义了以下函数：

func (c *Config) Add(field *Protocol.Field) error

func (c *Config) Update(field *Protocol.Field, value *Protocol.Value) error


Add和Update函数用于向inbound配置中添加或更新标头，分别接收一个Protocol.Field和Protocol.Value类型的参数。

接着，定义了以下同步函数：

var sync.WaitGroup



func Do(ch chan<-int, int) {
   config = &Config{}
   go func() {
       res, err := ConfigFunc(ch, config)
       if err != nil {
           return err
       }
       // do something with the result
   }()
   // ...
}


最后一个函数：

func ConfigFunc(ch chan<-int, int) (int, error) {
   var config *Protocol.Config
   var configLoaded bool
   var waitGroup sync.WaitGroup
   var result int

   // Add some WaitGroup干员
   waitGroup.Add(2)
   go func() {
       for {
           try:
               select {
               case <-ch:
                   // Config loaded, return the result
                   result, err := config.ConfigLoaded()
                   if err == nil {
                       return result, nil
                   }
                   // Wait for a config value to be available
                   <-waitGroup.代办器
               }
               case "exit":
                   // WaitGroup exit
                   return
               }
           }
           time.Delay(1 * time.毫秒)
       }
   }()

   // Add your config loading code here

   // Perform config updates as needed

   return result, nil
}


这段代码最后定义了一个名为 "config" 的实参，通过 ConfigFunc() 函数可以设置inbound的配置。


```go
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.25.0
// 	protoc        v3.13.0
// source: proxy/vmess/inbound/config.proto

package inbound

import (
	proto "github.com/golang/protobuf/proto"
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
	protocol "v2ray.com/core/common/protocol"
)

```

这段代码定义了一个名为 `DetourConfig` 的结构体，用于配置 detour 工具链的版本。

首先，它检查当前的 `proto` 版本是否足够新，可以通过 `protoimpl.EnforceVersion` 函数实现。

然后，它检查运行时/`protoimpl` 是否足够新，同样可以通过 `protoimpl.EnforceVersion` 函数实现。

接着，它会检查一个编译时断言，即是否正在使用足够新版本的 `proto` 包，如果这个断言为真，则表明 Detour 工具链的版本足够新。

最后，定义了一个 `To` 字段，用于指定输入或输出协议的地址。


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

type DetourConfig struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	To string `protobuf:"bytes,1,opt,name=to,proto3" json:"to,omitempty"`
}

```

这段代码定义了两个函数，一个是`func (x *DetourConfig) Reset()`，另一个是`func (x *DetourConfig) String()`。它们的主要作用是重置`DetourConfig`类型的实例，并将其转换为`file_proxy_vmess_inbound_config_proto_msgTypes`类型的实例。下面分别解释这两个函数的作用：

1. `func (x *DetourConfig) Reset()`函数的作用是重置`DetourConfig`类型的实例。它创建了一个新的`file_proxy_vmess_inbound_config_proto_msgTypes`类型的实例，并将这个实例赋值给`x`。这个新的实例包含了一个空的`DetourConfig`，当调用这个函数时，会将其赋值给`x`，从而实现清除`DetourConfig`类型的实例的效果。

2. `func (x *DetourConfig) String()`函数的作用是将`DetourConfig`类型的实例转换为字符串表示。它首先尝试从`x`开始调用`toString()`函数，如果`x`实现了`file_proxy_vmess_inbound_config_proto_msgTypes`类型的`String()`函数，那么它将返回`file_proxy_vmess_inbound_config_proto_msgTypes`类型的实例的引用。否则，它会尝试从`x`开始递归调用`String()`函数，每次将当前`DetourConfig`实例的引用设置为`x`，并将前一个`DetourConfig`实例作为参数传递给`toString()`函数。这样做的好处是，即使`x`没有实现`file_proxy_vmess_inbound_config_proto_msgTypes`类型的`String()`函数，`toString()`函数仍然可以从`DetourConfig`实例中还原出原始的`file_proxy_vmess_inbound_config_proto_msgTypes`实例。


```go
func (x *DetourConfig) Reset() {
	*x = DetourConfig{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_vmess_inbound_config_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *DetourConfig) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*DetourConfig) ProtoMessage() {}

```

这段代码定义了两个函数，函数一是接收一个指向DetourConfig类型的参数x，并返回一个来自DetourConfig.proto的Msg类型的指针；函数二是获取一个DetourConfig类型的对象*DetourConfig，返回其Descriptor。

函数一首先检查是否启用了UnsafeEnabled，如果是，则创建一个基于x的MessageStateOf的副本，并检查LoadMessageInfo()是否为nil。如果是，则执行StorageMessageInfo(mi)操作，将MessageInfo存储为mi。最后，返回MessageOf(x)的返回值作为结果。

函数二创建一个指向DetourConfig.proto的Msg类型的指针，并返回其对应的Descriptor。函数二没有做任何实际的逻辑，它只是一个空函数，因此不产生任何的副本来作为输出。


```go
func (x *DetourConfig) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_vmess_inbound_config_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use DetourConfig.ProtoReflect.Descriptor instead.
func (*DetourConfig) Descriptor() ([]byte, []int) {
	return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{0}
}

```

这段代码定义了一个名为"func"的函数，接受一个名为"x"的协程配置参数，并返回一个字符串类型的参数"GetTo"。函数的具体实现如下：

1. 如果传递的参数"x"不为 nil，则返回该参数所对应的"To"字段的值，否则返回一个空字符串 ""。

2. 定义了一个名为"DefaultConfig"的结构体，包含三个字段：state、sizeCache 和 unknownFields。其中，state 字段是一个协议实现的接口，sizeCache 和 unknownFields 字段都是该接口的实现。

3. 在 DefaultConfig 的定义中，定义了两个字段：AlterId 和 Level。其中，AlterId 字段是一个协议实现的接口，Level 字段也是一个协议实现的接口。

4. 在函数内部，首先检查传入的参数"x"是否为 nil，如果是，则直接返回一个空字符串 ""。否则，根据上面定义的 DefaultConfig 结构体中字段的名称，将 AlterId 和 Level 的值分别赋给 x.To 和 "" 字段，最终返回这个字符串。


```go
func (x *DetourConfig) GetTo() string {
	if x != nil {
		return x.To
	}
	return ""
}

type DefaultConfig struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	AlterId uint32 `protobuf:"varint,1,opt,name=alter_id,json=alterId,proto3" json:"alter_id,omitempty"`
	Level   uint32 `protobuf:"varint,2,opt,name=level,proto3" json:"level,omitempty"`
}

```

这段代码定义了两个函数，以及一个接口 DefaultConfig 的相应类型。

第一个函数 func (x *DefaultConfig) Reset() 重置了 x 的值，使其变为一个空的 DefaultConfig 类型。

第二个函数 func (x *DefaultConfig) String() 返回了 x 的字符串表示，与 func (x *DefaultConfig) ProtoMessage() 作用相反，这个函数没有返回任何值，而是导入了 ProtoMessage() 函数。

第三个函数 func (x *DefaultConfig) ProtoMessage() 返回了一个默认的 ProtocolMessage 类型，其中 x 的类型被传递给该函数。这个函数通过 x.MessageStringOf() 实现，将在定义好的类型转换为字符串，然后将其存储在 x 的 *DefaultConfig 类型的 second印章上，通过 second印章将该 x 的值重新设置为指定的 DefaultConfig 值。


```go
func (x *DefaultConfig) Reset() {
	*x = DefaultConfig{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_vmess_inbound_config_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *DefaultConfig) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*DefaultConfig) ProtoMessage() {}

```

这段代码定义了一个名为 func 的函数，接收一个名为 x 的 *DefaultConfig 类型的参数，并返回一个名为 protoreflect.Message 的类型。

函数的作用是在接收者函数中，根据传入的 x 参数，输出一个名为 protoreflect.Message 的类型。首先，检查 x 是否为空，如果是，则执行以下操作：根据接收者函数中定义的 protoimpl.UnsafeEnabled 参数，尝试使用文件映射虚拟内存中的 *DefaultConfig 类型，并获取 *x 的消息实例。然后，如果消息实例的加载消息信息为 nil，则将消息信息设置为接收者函数中定义的 file_proxy_vmess_inbound_config_proto_msgTypes[1] 类型。最后，返回设置好的消息实例。

另外，还定义了一个名为 Descriptor 的函数，它的作用是返回一个名为 file_proxy_vmess_inbound_config_proto_rawDescGZIP 的字节切片和两个整数，分别代表消息类型和消息长度。由于函数没有实现具体的逻辑，因此它被称为 deprecated，建议使用 DefaultConfig.Descriptor 函数。


```go
func (x *DefaultConfig) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_vmess_inbound_config_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use DefaultConfig.ProtoReflect.Descriptor instead.
func (*DefaultConfig) Descriptor() ([]byte, []int) {
	return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{1}
}

```

此代码定义了一个名为 Config 的结构体类型，该类型包含了一些与服务器相关的配置项。

此代码定义了两个函数，分别是 GetAlterId 和 GetLevel，它们分别返回一个名为 alterId 和 level 的整数类型变量。

此代码定义了一个名为 DefaultConfig 的结构体类型，该类型包含了一个名为 default 的字段，该字段包含一个指向 DefaultConfig 类型的指针。

此代码定义了一个名为 Config 的结构体类型，该类型包含了一个名为 user 的字段，该字段包含一个由字节数组组成的列表，该列表的元素为名为 user 的协议类型字段。另外，该类型还包含一个名为 default 和 detour 的字段，这些字段包含指向各自对应类型字段的指针。最后，该类型还包含一个名为 secure_encryption_only 的字段，该字段包含一个布尔值，表示是否使用安全加密模式。


```go
func (x *DefaultConfig) GetAlterId() uint32 {
	if x != nil {
		return x.AlterId
	}
	return 0
}

func (x *DefaultConfig) GetLevel() uint32 {
	if x != nil {
		return x.Level
	}
	return 0
}

type Config struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	User                 []*protocol.User `protobuf:"bytes,1,rep,name=user,proto3" json:"user,omitempty"`
	Default              *DefaultConfig   `protobuf:"bytes,2,opt,name=default,proto3" json:"default,omitempty"`
	Detour               *DetourConfig    `protobuf:"bytes,3,opt,name=detour,proto3" json:"detour,omitempty"`
	SecureEncryptionOnly bool             `protobuf:"varint,4,opt,name=secure_encryption_only,json=secureEncryptionOnly,proto3" json:"secure_encryption_only,omitempty"`
}

```

这段代码定义了两个函数，一个是`Reset`函数，另一个是`String`函数。这两个函数都在尝试通过传入一个`Config`类型的参数`x`，来执行一系列操作以实现对`Config`类型的设置。

首先，`Reset`函数会将`x`的值设置为`Config{}`，这通常是一个`0`，将其作为空的`Config`类型的实例。然后，如果`protoimpl.UnsafeEnabled`为`true`，那么会尝试执行下列操作：

1. 获取`file_proxy_vmess_inbound_config_proto_msgTypes`数组中的第二个元素，它是指`protoimpl.X.MessageType`类型的别名。
2. 创建一个指向`Config`类型实例的`mi`指针。
3. 使用`ms.StoreMessageInfo`方法将`mi`的值设置为`x`的`MessageInfo`字段。

接下来，我们来看`String`函数。它试图通过返回`x`的`MessageStringOf`函数的返回值，来将`x`的`MessageStringOf`函数的返回值作为字符串返回。

最后，`ProtoMessage`函数试图通过实现`fmt.Printf`函数的方式，将`x`的`MessageStringOf`函数的返回值作为格式化字符串返回，但似乎没有实际的作用。


```go
func (x *Config) Reset() {
	*x = Config{}
	if protoimpl.UnsafeEnabled {
		mi := &file_proxy_vmess_inbound_config_proto_msgTypes[2]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Config) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Config) ProtoMessage() {}

```

这段代码定义了两个函数，分别接收一个`Config`类型的参数`x`，并返回其`Message`类型的类型指针。

第一个函数`ProtoReflect`接收一个`Config`类型的`x`，并返回一个指向`file_proxy_vmess_inbound_config_proto_msgTypes`类型中第二个元素的`Message`类型的指针。这个函数的作用是将`x`的`Message`类型信息存储到一个`file_proxy_vmess_inbound_config_proto_msgTypes`类型的结构中，以便后续在函数中进行调用。

第二个函数`Descriptor`接收一个`Config`类型的`x`，并返回其`Message`类型和两个整数类型的值。这个函数的作用是输出`x`的`Message`类型，以便用户在使用`file_proxy_vmess_inbound_config_proto`接口时知道如何使用它。在函数中，首先检查`x`是否为空，如果是，则执行`unused_resolve`安全许可证，然后尝试使用`MessageStateOf`函数获取`x`的`Message`状态。如果没有找到`MessageStateOf`函数，则执行`MessageOf`函数获取`x`的`Message`类型。最后，返回`file_proxy_vmess_inbound_config_proto_rawDescGZIP`和`[]int`分别为`Descriptor`函数的源代码中提取的两个字节数组类型。


```go
func (x *Config) ProtoReflect() protoreflect.Message {
	mi := &file_proxy_vmess_inbound_config_proto_msgTypes[2]
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
	return file_proxy_vmess_inbound_config_proto_rawDescGZIP(), []int{2}
}

```

这是一个 Go 语言中的函数指针类型。它表示了一个名为“func”的函数接收一个名为“Config”的类型参数，并返回一个名为“*protocol.User”的类型参数。

具体来说，这段代码定义了三个名为“func”的函数，分别接收一个名为“x”的参数，并返回一个名为“*protocol.User”的类型参数。这三个函数都使用了相同的“*”开头的变量名“x”，表示它们都是参数类型。如果“x”存在，则执行第一个函数，否则执行第二个函数。

在函数内部，没有做实际的计算或数据操作，只是简单地根据“x”的值返回一个或选择一个默认值。


```go
func (x *Config) GetUser() []*protocol.User {
	if x != nil {
		return x.User
	}
	return nil
}

func (x *Config) GetDefault() *DefaultConfig {
	if x != nil {
		return x.Default
	}
	return nil
}

func (x *Config) GetDetour() *DetourConfig {
	if x != nil {
		return x.Detour
	}
	return nil
}

```

0x6f, 0x75, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x22,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70,
	0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
	0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x49, 0x6e, 0x62, 0x6f,
	0x75, 0x6e, 0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33



```go
func (x *Config) GetSecureEncryptionOnly() bool {
	if x != nil {
		return x.SecureEncryptionOnly
	}
	return false
}

var File_proxy_vmess_inbound_config_proto protoreflect.FileDescriptor

var file_proxy_vmess_inbound_config_proto_rawDesc = []byte{
	0x0a, 0x20, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2f, 0x69, 0x6e,
	0x62, 0x6f, 0x75, 0x6e, 0x64, 0x2f, 0x63, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x2e, 0x70, 0x72, 0x6f,
	0x74, 0x6f, 0x12, 0x1e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70,
	0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75,
	0x6e, 0x64, 0x1a, 0x1a, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f,
	0x63, 0x6f, 0x6c, 0x2f, 0x75, 0x73, 0x65, 0x72, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x1e,
	0x0a, 0x0c, 0x44, 0x65, 0x74, 0x6f, 0x75, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x0e,
	0x0a, 0x02, 0x74, 0x6f, 0x18, 0x01, 0x20, 0x01, 0x28, 0x09, 0x52, 0x02, 0x74, 0x6f, 0x22, 0x40,
	0x0a, 0x0d, 0x44, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12,
	0x19, 0x0a, 0x08, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x5f, 0x69, 0x64, 0x18, 0x01, 0x20, 0x01, 0x28,
	0x0d, 0x52, 0x07, 0x61, 0x6c, 0x74, 0x65, 0x72, 0x49, 0x64, 0x12, 0x14, 0x0a, 0x05, 0x6c, 0x65,
	0x76, 0x65, 0x6c, 0x18, 0x02, 0x20, 0x01, 0x28, 0x0d, 0x52, 0x05, 0x6c, 0x65, 0x76, 0x65, 0x6c,
	0x22, 0x83, 0x02, 0x0a, 0x06, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x12, 0x34, 0x0a, 0x04, 0x75,
	0x73, 0x65, 0x72, 0x18, 0x01, 0x20, 0x03, 0x28, 0x0b, 0x32, 0x20, 0x2e, 0x76, 0x32, 0x72, 0x61,
	0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x63, 0x6f, 0x6d, 0x6d, 0x6f, 0x6e, 0x2e, 0x70, 0x72,
	0x6f, 0x74, 0x6f, 0x63, 0x6f, 0x6c, 0x2e, 0x55, 0x73, 0x65, 0x72, 0x52, 0x04, 0x75, 0x73, 0x65,
	0x72, 0x12, 0x47, 0x0a, 0x07, 0x64, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x18, 0x02, 0x20, 0x01,
	0x28, 0x0b, 0x32, 0x2d, 0x2e, 0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e,
	0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f,
	0x75, 0x6e, 0x64, 0x2e, 0x44, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x43, 0x6f, 0x6e, 0x66, 0x69,
	0x67, 0x52, 0x07, 0x64, 0x65, 0x66, 0x61, 0x75, 0x6c, 0x74, 0x12, 0x44, 0x0a, 0x06, 0x64, 0x65,
	0x74, 0x6f, 0x75, 0x72, 0x18, 0x03, 0x20, 0x01, 0x28, 0x0b, 0x32, 0x2c, 0x2e, 0x76, 0x32, 0x72,
	0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76, 0x6d,
	0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x2e, 0x44, 0x65, 0x74, 0x6f,
	0x75, 0x72, 0x43, 0x6f, 0x6e, 0x66, 0x69, 0x67, 0x52, 0x06, 0x64, 0x65, 0x74, 0x6f, 0x75, 0x72,
	0x12, 0x34, 0x0a, 0x16, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x5f, 0x65, 0x6e, 0x63, 0x72, 0x79,
	0x70, 0x74, 0x69, 0x6f, 0x6e, 0x5f, 0x6f, 0x6e, 0x6c, 0x79, 0x18, 0x04, 0x20, 0x01, 0x28, 0x08,
	0x52, 0x14, 0x73, 0x65, 0x63, 0x75, 0x72, 0x65, 0x45, 0x6e, 0x63, 0x72, 0x79, 0x70, 0x74, 0x69,
	0x6f, 0x6e, 0x4f, 0x6e, 0x6c, 0x79, 0x42, 0x6b, 0x0a, 0x22, 0x63, 0x6f, 0x6d, 0x2e, 0x76, 0x32,
	0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x72, 0x65, 0x2e, 0x70, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x76,
	0x6d, 0x65, 0x73, 0x73, 0x2e, 0x69, 0x6e, 0x62, 0x6f, 0x75, 0x6e, 0x64, 0x50, 0x01, 0x5a, 0x22,
	0x76, 0x32, 0x72, 0x61, 0x79, 0x2e, 0x63, 0x6f, 0x6d, 0x2f, 0x63, 0x6f, 0x72, 0x65, 0x2f, 0x70,
	0x72, 0x6f, 0x78, 0x79, 0x2f, 0x76, 0x6d, 0x65, 0x73, 0x73, 0x2f, 0x69, 0x6e, 0x62, 0x6f, 0x75,
	0x6e, 0x64, 0xaa, 0x02, 0x1e, 0x56, 0x32, 0x52, 0x61, 0x79, 0x2e, 0x43, 0x6f, 0x72, 0x65, 0x2e,
	0x50, 0x72, 0x6f, 0x78, 0x79, 0x2e, 0x56, 0x6d, 0x65, 0x73, 0x73, 0x2e, 0x49, 0x6e, 0x62, 0x6f,
	0x75, 0x6e, 0x64, 0x62, 0x06, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x33,
}

```

这段代码定义了一个名为file_proxy_vmess_inbound_config_proto_rawDesc的变量，其作用是返回一个GZIP压缩后的二进制字节切片。

该变量的实现主要分两个步骤：

1. 定义了一个名为file_proxy_vmess_inbound_config_proto_rawDescOnce的变量，其作用是确保每次调用file_proxy_vmess_inbound_config_proto_rawDescGZIP()函数时都只执行一次。

2. 定义了一个名为file_proxy_vmess_inbound_config_proto_rawDescData的变量，该变量存储了一个名为file_proxy_vmess_inbound_config_proto_rawDesc的协议定义的实例。

3. 定义了一个名为file_proxy_vmess_inbound_config_proto_rawDescGZIP的函数，该函数使用file_proxy_vmess_inbound_config_proto_rawDescOnce中的do()函数，执行了将file_proxy_vmess_inbound_config_proto_rawDescData通过GZIP压缩后返回的义务。

4. 返回file_proxy_vmess_inbound_config_proto_rawDescData，即压缩后的二进制字节切片。


```go
var (
	file_proxy_vmess_inbound_config_proto_rawDescOnce sync.Once
	file_proxy_vmess_inbound_config_proto_rawDescData = file_proxy_vmess_inbound_config_proto_rawDesc
)

func file_proxy_vmess_inbound_config_proto_rawDescGZIP() []byte {
	file_proxy_vmess_inbound_config_proto_rawDescOnce.Do(func() {
		file_proxy_vmess_inbound_config_proto_rawDescData = protoimpl.X.CompressGZIP(file_proxy_vmess_inbound_config_proto_rawDescData)
	})
	return file_proxy_vmess_inbound_config_proto_rawDescData
}

var file_proxy_vmess_inbound_config_proto_msgTypes = make([]protoimpl.MessageInfo, 3)
var file_proxy_vmess_inbound_config_proto_goTypes = []interface{}{
	(*DetourConfig)(nil),  // 0: v2ray.core.proxy.vmess.inbound.DetourConfig
	(*DefaultConfig)(nil), // 1: v2ray.core.proxy.vmess.inbound.DefaultConfig
	(*Config)(nil),        // 2: v2ray.core.proxy.vmess.inbound.Config
	(*protocol.User)(nil), // 3: v2ray.core.common.protocol.User
}
```

This is a Go-based implementation of the `file_proxy_vmess_inbound_config_proto` message, which is part of the file_proxy_vmess package. This package is responsible for implementing the logic for configuring a proxy for incoming video streams, such as HTTP or RTSP.

The `file_proxy_vmess_inbound_config_proto` message has several fields, including a `Config` field, which is an instance of the `Config` message, and several other fields such as `state`, `sizeCache`, and `unknownFields`.

The `Exporter` field of the `file_proxy_vmess_inbound_config_proto` message is a function that returns an instance of the `Config` message. The function takes an interface{} parameter, which is cast to a `Config` instance, and an integer parameter i.

The `file_proxy_vmess_inbound_config_proto` package provides a file-based exporter for this message, which allows for easier testing and usage of the message in代码.


```go
var file_proxy_vmess_inbound_config_proto_depIdxs = []int32{
	3, // 0: v2ray.core.proxy.vmess.inbound.Config.user:type_name -> v2ray.core.common.protocol.User
	1, // 1: v2ray.core.proxy.vmess.inbound.Config.default:type_name -> v2ray.core.proxy.vmess.inbound.DefaultConfig
	0, // 2: v2ray.core.proxy.vmess.inbound.Config.detour:type_name -> v2ray.core.proxy.vmess.inbound.DetourConfig
	3, // [3:3] is the sub-list for method output_type
	3, // [3:3] is the sub-list for method input_type
	3, // [3:3] is the sub-list for extension type_name
	3, // [3:3] is the sub-list for extension extendee
	0, // [0:3] is the sub-list for field type_name
}

func init() { file_proxy_vmess_inbound_config_proto_init() }
func file_proxy_vmess_inbound_config_proto_init() {
	if File_proxy_vmess_inbound_config_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_proxy_vmess_inbound_config_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*DetourConfig); i {
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
		file_proxy_vmess_inbound_config_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*DefaultConfig); i {
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
		file_proxy_vmess_inbound_config_proto_msgTypes[2].Exporter = func(v interface{}, i int) interface{} {
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
			RawDescriptor: file_proxy_vmess_inbound_config_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   3,
			NumExtensions: 0,
			NumServices:   0,
		},
		GoTypes:           file_proxy_vmess_inbound_config_proto_goTypes,
		DependencyIndexes: file_proxy_vmess_inbound_config_proto_depIdxs,
		MessageInfos:      file_proxy_vmess_inbound_config_proto_msgTypes,
	}.Build()
	File_proxy_vmess_inbound_config_proto = out.File
	file_proxy_vmess_inbound_config_proto_rawDesc = nil
	file_proxy_vmess_inbound_config_proto_goTypes = nil
	file_proxy_vmess_inbound_config_proto_depIdxs = nil
}

```