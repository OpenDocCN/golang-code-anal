# v2ray-core源码解析 1

# `functions_test.go`

这段代码是一个 Go 语言编写的测试套件，用于测试 Core 组件的功能。它定义了一个名为 "core_test" 的包，并且包含以下函数和类型：

1. 导入了一些外部的库：
	* "context"
	* "crypto/rand"
	* "io"
	* "testing"
	* "time"
	* "github.com/golang/protobuf/proto"
	* "github.com/google/go-cmp/cmp"
	* "v2ray.com/core"
	* "v2ray.com/core/app/dispatcher"
	* "v2ray.com/core/app/proxyman"
	* "v2ray.com/core/common"
	* "v2ray.com/core/common/net"
	* "v2ray.com/core/common/serial"
	* "v2ray.com/core/proxy/freedom"
	* "v2ray.com/core/testing/servers/tcp"
	* "v2ray.com/core/testing/servers/udp"
2. 通过 Iterator 设计一个名为 "test_core_dispatcher" 的测试函数：
	* "test_core_dispatcher" 函数接收一个 "dispatcher.Context" 作为上下文，然后使用 Protobuf 编解码器来解析测试数据，并调用 "Dispatcher" 类型的 "dispatcher" 字段，以实现一个简单的测试。

这个测试套件可能是为了测试 Core 的功能而编写的，但是它并不包含所有必要的测试用例，因此需要由其他测试套件来进行补充。


```go
package core_test

import (
	"context"
	"crypto/rand"
	"io"
	"testing"
	"time"

	"github.com/golang/protobuf/proto"
	"github.com/google/go-cmp/cmp"

	"v2ray.com/core"
	"v2ray.com/core/app/dispatcher"
	"v2ray.com/core/app/proxyman"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/freedom"
	"v2ray.com/core/testing/servers/tcp"
	"v2ray.com/core/testing/servers/udp"
)

```

这两段代码都是实现了XOR操作，其作用是生成与输入参数b相同长度的结果数组。

在函数实现中，首先在make函数中创建一个与输入参数b相同长度的新数组r。然后使用for循环遍历输入参数b中的每一个元素，并将当前元素与字符'c'进行异或运算，得到一个新的二进制位并将其存储到数组r中对应的位置。最后，将r生成的结果数组返回。

而第二个函数xor2的实现与第一个函数相似，只是将字符'd'替换为了第二个参数b中的字符'e'。


```go
func xor(b []byte) []byte {
	r := make([]byte, len(b))
	for i, v := range b {
		r[i] = v ^ 'c'
	}
	return r
}

func xor2(b []byte) []byte {
	r := make([]byte, len(b))
	for i, v := range b {
		r[i] = v ^ 'd'
	}
	return r
}

```

这段代码是一个 Go 语言中的测试函数，名为 "TestV2RayDial"。它使用 go- tortcp 库实现了一个 TCP 服务器到 V2Ray 的代理握手过程。

具体来说，这段代码创建了一个 TCP 服务器，通过它来连接到 V2Ray，然后测试代理握手的正确性。这个测试函数使用了以下几个主要部分：

1. 创建一个 TCP 服务器实例，设置了服务器接收到的消息处理器为 xor 函数，这个函数会将接收到的消息的最后一个字节翻转过来。
2. 创建一个 V2Ray 的代理配置，包括设置代理服务器的目标地址、端口、加密套接字等等。
3. 创建一个 TCP 服务器配置，包括设置服务器的目标地址、端口、协议类型等等，这个配置使用了 Go 语言中 core-v2 库中的配置类。
4. 调用 tcpServer.Start() 函数来启动服务器，并返回服务器实例和错误信息。
5. 创建一个核心的实例，使用这个实例来处理来自客户端的请求，然后将请求转发给代理服务器，再将代理服务器返回的消息接收回来，检查消息是否正确。
6. 在客户端连接到服务器之后，使用 io.ReadFull 函数从服务器接收一系列数据，然后使用 cmp-diff 函数检查客户端收到的数据和代理服务器返回的数据是否一致。

总的来说，这段代码实现了一个简单的 TCP 代理握手过程，可以用于测试 V2Ray 代理是否可以正常工作。


```go
func TestV2RayDial(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.InboundConfig{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	cfgBytes, err := proto.Marshal(config)
	common.Must(err)

	server, err := core.StartInstance("protobuf", cfgBytes)
	common.Must(err)
	defer server.Close()

	conn, err := core.Dial(context.Background(), server, dest)
	common.Must(err)
	defer conn.Close()

	const size = 10240 * 1024
	payload := make([]byte, size)
	common.Must2(rand.Read(payload))

	if _, err := conn.Write(payload); err != nil {
		t.Fatal(err)
	}

	receive := make([]byte, size)
	if _, err := io.ReadFull(conn, receive); err != nil {
		t.Fatal("failed to read all response: ", err)
	}

	if r := cmp.Diff(xor(receive), payload); r != "" {
		t.Error(r)
	}
}

```

This is a Go program that configures a Go proxy server to use the Google Cloud中提供的 Cloud-Native XOR secret serializer.

The program creates a Go proxy server by defining the configuration settings of the server. These settings include the address and port of the server, the XOR secret to be used for the serialization and deserialization of messages, and the serialization settings for incoming and outgoing messages.

The program then configures the incoming and outgoing connections of the server. The incoming connections are configured with the address and port of the proxy server and the XOR secret to be used for the serialization and deserialization of incoming messages. The outgoing connections are configured with the address and port of the proxy server, the XOR secret to be used for the serialization and deserialization of outgoing messages, and the payload size for the XOR secret.

Finally, the program starts the Go proxy server using the `core.StartInstance` method from the `core` package.

The program also includes a test that simulates a connection to the proxy server and sends a series of incoming and outgoing messages. The incoming messages are expected to include the XOR secret, and the outgoing messages are expected to include the same XOR secret and a payload of two times the size of the incoming message. The test uses the `t.Fat` function from the `testing` package to check for any errors in the process and the `common.Must` function from the `common` package to check that the XOR secret is not an empty string.


```go
func TestV2RayDialUDPConn(t *testing.T) {
	udpServer := udp.Server{
		MsgProcessor: xor,
	}
	dest, err := udpServer.Start()
	common.Must(err)
	defer udpServer.Close()

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.InboundConfig{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	cfgBytes, err := proto.Marshal(config)
	common.Must(err)

	server, err := core.StartInstance("protobuf", cfgBytes)
	common.Must(err)
	defer server.Close()

	conn, err := core.Dial(context.Background(), server, dest)
	common.Must(err)
	defer conn.Close()

	const size = 1024
	payload := make([]byte, size)
	common.Must2(rand.Read(payload))

	for i := 0; i < 2; i++ {
		if _, err := conn.Write(payload); err != nil {
			t.Fatal(err)
		}
	}

	time.Sleep(time.Millisecond * 500)

	receive := make([]byte, size*2)
	for i := 0; i < 2; i++ {
		n, err := conn.Read(receive)
		if err != nil {
			t.Fatal("expect no error, but got ", err)
		}
		if n != size {
			t.Fatal("expect read size ", size, " but got ", n)
		}

		if r := cmp.Diff(xor(receive[:n]), payload); r != "" {
			t.Fatal(r)
		}
	}
}

```

This is a Go program that performs a TCP connection test using the Protocol Buffers protocol. It performs two connection tests, one with a remote host defined by the `server` field of the configuration file, and the other with a remote host defined by the `server` field of the configuration file.

The program first creates a connection to the remote host using the `core.StartInstance` function, which starts a new gRPC server and returns a connection to it. It then sends a packet of data to the remote host using the `core.DialUDP` function, which establishes a connection to the remote host and sends a packet to it.

The program then performs a test to verify that the connection is successful. It sends a packet of data to the remote host using the `core.DialUDP` function, and then reads a response back from the remote host using the `core.ReadFrom` function. It then compares the response to the packet to ensure that they match, and reports any differences if they are not the same.

The program also performs a test to verify that the connection is not successful. It sends a packet of data to the remote host using the `core.DialUDP` function, and then reads a response back from the remote host using the `core.ReadFrom` function. It then reads the response again using the `core.DialUDP` function, and compares the two responses to ensure that they match. If they do not match, the program reports the difference.


```go
func TestV2RayDialUDP(t *testing.T) {
	udpServer1 := udp.Server{
		MsgProcessor: xor,
	}
	dest1, err := udpServer1.Start()
	common.Must(err)
	defer udpServer1.Close()

	udpServer2 := udp.Server{
		MsgProcessor: xor2,
	}
	dest2, err := udpServer2.Start()
	common.Must(err)
	defer udpServer2.Close()

	config := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.InboundConfig{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	cfgBytes, err := proto.Marshal(config)
	common.Must(err)

	server, err := core.StartInstance("protobuf", cfgBytes)
	common.Must(err)
	defer server.Close()

	conn, err := core.DialUDP(context.Background(), server)
	common.Must(err)
	defer conn.Close()

	const size = 1024
	{
		payload := make([]byte, size)
		common.Must2(rand.Read(payload))

		if _, err := conn.WriteTo(payload, &net.UDPAddr{
			IP:   dest1.Address.IP(),
			Port: int(dest1.Port),
		}); err != nil {
			t.Fatal(err)
		}

		receive := make([]byte, size)
		if _, _, err := conn.ReadFrom(receive); err != nil {
			t.Fatal(err)
		}

		if r := cmp.Diff(xor(receive), payload); r != "" {
			t.Error(r)
		}
	}

	{
		payload := make([]byte, size)
		common.Must2(rand.Read(payload))

		if _, err := conn.WriteTo(payload, &net.UDPAddr{
			IP:   dest2.Address.IP(),
			Port: int(dest2.Port),
		}); err != nil {
			t.Fatal(err)
		}

		receive := make([]byte, size)
		if _, _, err := conn.ReadFrom(receive); err != nil {
			t.Fatal(err)
		}

		if r := cmp.Diff(xor2(receive), payload); r != "" {
			t.Error(r)
		}
	}
}

```

# `mocks.go`

这段代码定义了一个名为"Core"的包，其中包含多个用于模拟和测试的函数。

具体来说，这些函数使用了Go标准库中的"mockgen"工具，以生成模拟和测试代码。生成的代码包括：

- "io.go"：一个以"io"为前缀的函数，模拟了I/O操作，如读写文件、网络请求等。
- "log.go"：一个以"log"为前缀的函数，模拟了日志记录，如记录v2ray.com/core/common/log中的信息。
- "mux.go"：一个以"mux"为前缀的函数，模拟了v2ray.com/core/common/mux中的客户端/服务器通信。
- "dns.go"：一个以"dns"为前缀的函数，模拟了DNS查询和响应。
- "outbound.go"：一个以"outbound"为前缀的函数，模拟了v2ray.com/core/features/outbound中的客户端和服务器通信。
- "proxy.go"：一个以"proxy"为前缀的函数，模拟了一个代理服务器，允许客户端通过代理连接到v2ray.com/core/features/outbound。


```go
package core

//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/io.go -mock_names Reader=Reader,Writer=Writer io Reader,Writer
//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/log.go -mock_names Handler=LogHandler v2ray.com/core/common/log Handler
//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/mux.go -mock_names ClientWorkerFactory=MuxClientWorkerFactory v2ray.com/core/common/mux ClientWorkerFactory
//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/dns.go -mock_names Client=DNSClient v2ray.com/core/features/dns Client
//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/outbound.go -mock_names Manager=OutboundManager,HandlerSelector=OutboundHandlerSelector v2ray.com/core/features/outbound Manager,HandlerSelector
//go:generate go run github.com/golang/mock/mockgen -package mocks -destination testing/mocks/proxy.go -mock_names Inbound=ProxyInbound,Outbound=ProxyOutbound v2ray.com/core/proxy Inbound,Outbound

```

# `proto.go`

这段代码的作用是生成一些Go语言可执行文件和源代码文件，用于在项目中使用Google Protobuf格式的数据交互消息（.proto）文件。

具体来说，代码会使用三个不同的工具生成这些文件：

1. `protoc-gen-gofast`：这是一个用于生成高效、快速、小型Go语言可执行文件的工具。它通过从给定的Protoc源代码文件生成pb.go文件，从而简化了在Go项目中使用.proto文件的过程。
2. `go install`：这个命令用于安装Go语言的特定版本，以及相关的依赖项（如protoc-gen-go-grpc和github.com/gogo/protobuf/protoc-gen-gofast）。
3. `./infra/vprotogen/main.go`：这是一个位于项目根目录下的文件，它可能是包含一些自动生成的Java类和定义，用于处理与.proto文件关联的自动生成的Java代码。


```go
package core

//go:generate go install -v google.golang.org/protobuf/cmd/protoc-gen-go
//go:generate go install -v google.golang.org/grpc/cmd/protoc-gen-go-grpc
//go:generate go install -v github.com/gogo/protobuf/protoc-gen-gofast
//go:generate go run ./infra/vprotogen/main.go

import "path/filepath"

// ProtoFilesUsingProtocGenGoFast is the map of Proto files
// that use `protoc-gen-gofast` to generate pb.go files
var ProtoFilesUsingProtocGenGoFast = map[string]bool{"proxy/vless/encoding/addons.proto": true}

// ProtocMap is the map of paths to `protoc` binary excutable files of specific platform
var ProtocMap = map[string]string{
	"windows": filepath.Join(".dev", "protoc", "windows", "protoc.exe"),
	"darwin":  filepath.Join(".dev", "protoc", "macos", "protoc"),
	"linux":   filepath.Join(".dev", "protoc", "linux", "protoc"),
}

```

# Move To https://github.com/v2fly/v2ray-core

***

# Project V

[![GitHub Test Badge][1]][2] [![codecov.io][3]][4] [![GoDoc][5]][6] [![codebeat][7]][8] [![Downloads][9]][10] [![Downloads][11]][12]

[1]: https://github.com/v2fly/v2ray-core/workflows/Test/badge.svg "GitHub Test Badge"
[2]: https://github.com/v2fly/v2ray-core/actions "GitHub Actions Page"
[3]: https://codecov.io/gh/v2fly/v2ray-core/branch/master/graph/badge.svg?branch=master "Coverage Badge"
[4]: https://codecov.io/gh/v2fly/v2ray-core?branch=master "Codecov Status"
[5]: https://godoc.org/v2ray.com/core?status.svg "GoDoc Badge"
[6]: https://godoc.org/v2ray.com/core "GoDoc"
[7]: https://goreportcard.com/badge/github.com/v2fly/v2ray-core "Goreportcard Badge"
[8]: https://goreportcard.com/report/github.com/v2fly/v2ray-core "Goreportcard Result"
[9]: https://img.shields.io/github/downloads/v2ray/v2ray-core/total.svg "v2ray/v2ray-core downloads count"
[10]: https://github.com/v2ray/v2ray-core/releases "v2ray/v2ray-core release page"
[11]: https://img.shields.io/github/downloads/v2fly/v2ray-core/total.svg "v2fly/v2ray-core downloads count"
[12]: https://github.com/v2fly/v2ray-core/releases "v2fly/v2ray-core release page"

Project V is a set of network tools that help you to build your own computer network. It secures your network connections and thus protects your privacy. See [our website](https://www.v2fly.org/) for more information.

## License

[The MIT License (MIT)](https://raw.githubusercontent.com/v2fly/v2ray-core/master/LICENSE)

## Credits

This repo relies on the following third-party projects:

- In production:
  - [gorilla/websocket](https://github.com/gorilla/websocket)
  - [gRPC](https://google.golang.org/grpc)
- For testing only:
  - [miekg/dns](https://github.com/miekg/dns)
  - [h12w/socks](https://github.com/h12w/socks)


# 安全策略 Security Policy

## 受支持的版本 Supported Versions

目前 v2ray-core 项目由 [V2Fly 社区](https://github.com/v2fly) 继续提供代码维护，由于精力有限且项目复杂度较高，只维护主线代码的功能和安全性完整。原则上主页的兼容性保证继续遵循，
如有例外另行说明。

Currently v2ray-core project is maintained by [V2Fly community](https://github.com/v2fly). Feature and security guarantee may only be limited to the
master branch, though we would still try our best to follow the compatiblity claims listed on the official website.


## 汇报安全风险 Reporting a Vulnerability

使用邮箱: security |at| v2fly.org。

Report to email: security |at| v2fly.org.

GPG public key:

```go
pub   rsa4096 2020-06-02 [SC] [有效至：2022-01-02]
      E2E35E27914FB007C0D4B6DDB117BA3BE8B494A7
uid           [ 绝对 ] V2Fly Developers <dev@v2fly.org>
sub   rsa4096 2020-06-02 [E] [有效至：2022-01-02]
sub   rsa4096 2020-11-08 [S] [有效至：2022-01-02] // 用于 Debian / Ubuntu 签名


-----BEGIN PGP PUBLIC KEY BLOCK-----

mQINBF7V7pQBEACozcw4/BlPgFWaz4AdN8HKSrCDLlN+/g7m4AKZIo13fAnDh+sJ
2H4NrWNr0xxgovbco5Xw4OSSwY1BuUhnb4AmIyxbwqUQD2UADe5xD6gMBwNiJTP4
02VCHhh7DnWTeLbAsUgRotxUCxsWVvd2F08SYGfJggOVftOnG+VNnwzTOvHWFVEw
1Pv1DaY7bKSA0voACerRbAPCYqhmElAGJHYNjBMaxqCaWFJWpAFfBxkvS1FDVyZk
BsABhn6sOcGJn8EYSHUIXhWwqtkQCjBB4OOik+Jn+S2DFGyk5l1NrGRQtX8C0BYn
nc7VaxtFOp5fnJ4y0GNd4AM9KO0/Ojosi6b64l407Fj9i9OXznmZUACQw2u+VcL3
qNy768hsTmka3pXzpRHZwYcOLOEr3jGHmLOtXgQ656OjF8Xd9DJ4cB42X8iBeqTp
iQchHIdBpnu27ZbBFy09OMak+STB5zA0JmxDaC8b48mVkc0BMRXdYl7wWXJsEJf1
roAOr3RCBKiE840w0PLOTnUljfqazPYTwzs91oP+SeZjBmGOpaAh7bh5BVOpzPSE
bdA61/n01GEb5bpOKpaTi9GviF3RCbfFnLKJnBq0vHvW9BqKTVFRPAKkBGuOPBdy
8MBNY+VY/2aP3ukZUoYe8Ypl9Q7dVPRjnoWaH0sEMzftoh+3s7GSSgAylQARAQAB
tCBWMkZseSBEZXZlbG9wZXJzIDxkZXZAdjJmbHkub3JnPokCVAQTAQgAPgIbAwUL
CQgHAgYVCgkICwIEFgIDAQIeAQIXgBYhBOLjXieRT7AHwNS23bEXujvotJSnBQJf
p4leBQkC+1DKAAoJELEXujvotJSn124P/0swu9POvEQtxVlRzNh2VjAGHZ5NEDnl
pMrhfC5ryCYtlVS/kc2WwRhIRHKzr9nbamgSxUCiyLagfnIjhIvAohun49grYNzG
MZWRURiuFrCnYbD7juJTvfbzZCzJk7LPsdnqHWr8fYcOZMTOZVzQiQB2jUx2KeRm
yV8aV21Z8gMLqSGjs06a0UaRbKB0FSysTURm91/jFmiH43aG1s/LcB9/lKf5HpNl
9or6LrEOrokAwtkMSBYTqm7Dp1j+cK0iOMw2CmMqmQZkV+i6msYrQRiX/X6YufiM
wfMMSdOZOz9KG+k+C6N1swSbGeDMrJfnDUDbvrAXKhDjNgY7UBwbk69Abd7Y9aQz
/jVmrFEWt4lisBxglBot60CRUTM2boK/uQS5zBCJhemeg14F9Q/FRiUTlS8jQoeK
PWeK2lagYJS8lpJZLXkqe4xSpjCgoT0Z+lYSfTjx+T0AFF+xz5E243Lb5kDxwnR9
Y5CZt3vV6GWBYOt9MEL3pk7AnYyNT1y1KIiMyONh/Z1koUdHr4J9exllnsmAJQUa
W/j0UtVsLsvUjFv9RTr9w5p/U0J0VLIN0YOpx4wYaBEwFIa8lsL+Ey1Vphkvvjfz
uMRAHe4v+axWb1f1hVCBjtyCVyvzf+i9RTAYsBJ3MJ0C8cvvrm10N9B7MHh0JZA5
PcJSilailp1TuQINBF7V7pQBEADkQdO75smeKnmPt0/aNNlb7JDOSWW5VY0kYgx3
6Toh139JstIQ2xz0CLSGReizUFB6eR3DXmezLrmhkgN2Aq5A+hCtFAJwWKuKr1HS
usvJ1el9h0oh7IO+tF8E/gNYwWfabjPX27FGVCHR1qG7ffN51Bghrnwi1T4YC98E
R9EGU6N0Xs9DeIJL9WQPH/DF22251i/OAXkqKVGn3PNe2cBsp0yKxr9mlSyzjrha
KXokPiPcvNqlnkiDCgfiRj7c2C2Lyl9PoEiGpsNZaCZYkMPgjM0xiLenQddwRyOU
z2cLG3d8WdCTRyHSZd/YQtSi5R6AnkJEsVtUiDN5zwNFVpQlTq3jNHsVUpjFU2nK
ourTZVCCLbAC60VTdxLN6eFO0f+lS2WjyJ7uZ9SGbS6uP0jMNphH/QjVF848bWXs
1CuZty5QQY7+MTNUAhSWWntrpTkdXYqT0zUqiOc1YNnkfg3hvC4d0dbnFTfcyZnB
Sg8e7/9n6+ms75/deYgnLuA6h7pkIcflm7pUMfVKXKz5Vlc8FC9ia0UtobeKBKqi
jObfiO/zmNL0HQBeX0e8GkJrCyv6ikD8cUqsmVtgw7jdxGsV0SL5CddDnGKsc68O
pGDmkAuRqR3QtXju/4r7a8IEVveGWc3rUvddYrtqbbCNWCN0JKX13PEvbNAm+2eD
MGQtcQARAQABiQI8BBgBCAAmAhsMFiEE4uNeJ5FPsAfA1LbdsRe6O+i0lKcFAl+n
dwcFCQL7PnMACgkQsRe6O+i0lKeWfxAApopL5I9p4btmkcLIg2lkA1n+czFekbdr
2tjFKrBER4QWkyDCUE8QaVo/ECveTHmnxrTB/djW6xqPVS77PL8xOATIYTo6qU38
oTCB1T7/P2L9qI72BzcRY5f9ZPyJhCtrkvjCPzjUjw+ZIPIOgQcWgKHWnE+OyUKD
0GkVEUME3QP5S4Nr3XGrgS7oxDAmD52u7pn0mSk5WmEcLW0oGwsVdc4aDXxpX+u/
gkBZysmAuomPov7iXVosMakl+4rz30yPcrL9A81m1WAeB3PGkpaO3B++8Ql+FBCQ
OrLtPn/nnIzEuAXB1Hd8vYzxtRM2CZvhRExM7xofnhkBJOtR/ddfbJa7H5+Aruc0
4S0JIaqMCrC6tZezjTACAzrWULmZZGmrHbLrmXBuLk0huRkeIRnDzHP+DoE2UciL
3hR9EGOHX9O/dGb3bb3y11LAf7GI28ZG7So1GeoFkEOga1IJnsBnXCqwM8vbDDWq
/7aLb3/m0gT7DUfjeXKfWPJXcnaq8r4llHzDn2i6ax4Uq/brCOLj9ovVGIctZTbt
yvsFOc1bVkSuUM+pMkCtBx80/sJSB2Nu94S6osdaUlRE+jaCcqEbPd+G68Yd0Khi
CL8zF1a3dX1dpuVFTLNpXOgrviGBzXQmzFeil7mWFs0l+1XZOPz9nhmRrMn6wV3n
i4KItRSJAXy5Ag0EX6d5hQEQAMsVyLTXdybeei2nWDb5jtzzC3AtSnPWtKG4B86C
BXncaZpU43hKI3oduW2+42eM8n8KTvO11r9xv4zKATfaHBZq2hkKZdDQjuSstovr
a3hapHHknHeNVTg3yuiakKzpr6FK23W/GE1lJfhz254v9+dRV0KazWksXvpGEdgI
+6sC4Nr5bKgJVEQibyrrL0gmzlVB/oQU/W4eGvk21zmgMlHri+edBLpVtlCmn7k/
0t+2X9D1Pq2nkjMUurB9EJ1z24LMldmPOl6P7iJCx9kSUjcHrEg56q5VSZq50FAj
DeSjAqsdussI8cdstCMktE9nhizxVKFXpbXifqoYfJwCo23wFqQJpyPgQqHIT12s
GWRUa/MF6hRYg/5CyeadDmkmnKPTPjmQ2S2SFNXX7xs+dZKvIvXP30z4cpuVY8i8
chZSRNb8K0L9T0Jme7CPm28F6lvDUkNDQ1WErXZruHbOKwQOfQBdXK3nedOiUpBt
401HVlGUJSInfEb3JXU01tRqnnzI/y5z7cWCGEMEa6TeaCrMbVvl8xeAA1w/nw0y
zHz6/Pnf4TITuCH22aa7+xfgpq8gRLhUUws89mbQT+9fd8tT65+Q8xcaLCyzrLAq
zND5sVZ4/PwaYc8UNZcHjeQR7aYWI1xgr/IwY1wyDWZLbWgkk0HVxpvYdMEpJryD
AyMdABEBAAGJBHIEGAEIACYWIQTi414nkU+wB8DUtt2xF7o76LSUpwUCX6d5hQIb
AgUJAim2AAJACRCxF7o76LSUp8F0IAQZAQgAHRYhBK8FZLGpNMztuW02YbJOz+X/
ddOBBQJfp3mFAAoJELJOz+X/ddOBNKQP/0nwIC4R9gQhY53vME7VA7elIrBiSM6d
Va26a7J1nrCcpDAE7Lp0TqzrDMqyen+IL4X5QK5sKTgenYTgjppEJIQn+Wup54ix
I+YOQ8MVLfN6/3QPACWMngSPRF+UKDg4hyTCEL+/GCgTp58oXrl/YIO6Oqt5drog
w4+4ufU1/eKTb2ruGULGl9jZvFSZpLdsvJ19xJB2kC1k8GVNu7MnUL+S2pU/9kO4
5EZ/jEa1wT45zev+HdmzX5TYW6SLaI9HKHMqbQz2EHc3tRYIDaz3FE3s4VdMjqpp
e42SvkOYaguc6cXToDbzBmU+iWGlXCTHfNhxwxoUYcKZlDEkEtvSYHJOb0k9eqbT
gvMb5GjbAgqqwOBwtN3v790j8jEG+cdXR3qHcEx0bw3F2Bd18U7j946OxHLKE5Xk
2sWEG422maVrE9o1DdeTV5oDFNNPBzqfjgGBZCCKrjkpldhDOHeoDU2aFMJ7yVqw
ZwKwJ5f8fdNS13UnQVwGsZ2BsW1cox5ZGZ/C5A7mfSF1WAgJcYIw4M2JQbDn4Yuw
yqjyg53lT3OurBONbEZ7unnsLqpT9qKwZ1qCemqGRJieXXxJwl7G4gBgZbH0rBJR
6dhbyt4c2JE8MMdC65mDWneltNM6pttC/j5jCuvIlZGACZ91UuLLediJJWAlOJ+1
fBQ0m8TD6d8ZrakP/RFMLZrxh9WPaFB43sW/b1Fq2h933HQ29oSQFuXhsHsx1Vaq
HTRTcBB7kywAr9+zMYsOsk0/WnoZNGoMkUWu/gFkb6CdUcsdEumgyZ8S24VoBCHB
T1fD/8eOA5K82hwAFcKbPwuuTLtf9b9HB4/xsObfcczTeqIknzIPsGlgVz4w1c9a
StSo4iI4bCSLL+/mqiXZ+ArXJ/z4Vejl92fNLWVOlOrjkBV+AY6iAFCCsxJ1O5ud
5a5r1bUeBXd0BcQ1m/hpjawMC1y0SkIBTQCgxIQoPoxJ27hHNIN1R2nkqfY9vboQ
7O0uIHF8fmuz93xg68ZTW0JHwOw4Mz88lGibE2laHApjKWZAtF/i+LlhbnewtESL
EuGTT7gt7cSHgnBiDEIm5UJVEGeM0sMReztxy9V7glohH5DV8GpVK/GncKlsrh1K
BuEuz7IrqKlBzhsDy0SrNZpX7EzsiU1uvoA6teT4EPey8qXH+7WR9B2ad1Zc5yE3
zv4BpnWkkJp8qdYu4fdCs/mrmnBR5G1YdOAIlNWhU74Wdyq+W4HfTWMgvJHmElnZ
UvQ9RDTWnw2+3n2ATeLf9ZwW1g4/Dqh55OaLtJZo5me8vU9W+vkm34xzfVfD/mus
ljogw5eiGyj8j3lUVjYWu28l/bz0zDUueWmHhV8E8z0Cn7OhrHPpUCHx2Aep
=quYd
-----END PGP PUBLIC KEY BLOCK-----

```


# `v2ray.go`

这段代码是一个名为 "core.go" 的包的源代码。它定义了一个名为 "core" 的包，其中包含许多功能和组件，用于实现 v2ray.com 项目的核心功能。下面是对代码中的一些要点的解释：

1. `// +build !confonly`：这是一个编译选项。`+build` 表示编译时生成 `build/core.go` 文件，`!confonly` 表示不要在进入编译器选项行之前生成任何日志。

2. `package core`：这个包定义了它自己的 packageset，其中包含一些名为 "core" 的包。

3. `import (<v2ray.com/core/common.go>` `import "v2ray.com/core/common/reflect"` `import "v2ray.com/core/common/serial"` `import "v2ray.com/core/features"` `import "v2ray.com/core/features/dns"` `import "v2ray.com/core/features/dns/localdns"` `import "v2ray.com/core/features/inbound"` `import "v2ray.com/core/features/outbound"` `import "v2ray.com/core/features/policy"` `import "v2ray.com/core/features/routing"` `import "v2ray.com/core/features/stats"`

4. `package core.io`：这个包定义了 "core.io" 的 packageset，其中包含一些名为 "io" 的包。

5. `import "io/ioutil.收费"`：这个函数导入了一个名为 "ioutil.收费" 的包，它提供了文件 I/O 操作的功能。

6. `import "io/shortcut.收费"`：这个函数导入了一个名为 "shortcut" 的包，它提供了几个方便的 I/O 操作，如文件复制、移动、删除、打印等等。

7. `import "net/http.收费"`：这个函数导入了一个名为 "http.收费" 的包，它提供了 HTTP 请求的功能。

8. `import "net/url.收费"`：这个函数导入了一个名为 "url.收费" 的包，它提供了 URL 解析的功能。

9. `import "encoding.收费"`：这个函数导入了一个名为 "收费" 的包，它提供了编码功能。

10. `import "math.收费"`：这个函数导入了一个名为 "math.收费" 的包，它提供了数学计算的功能。

11. `import "container/收费"`：这个函数导入了一个名为 "container/收费" 的包，它提供了容器化功能。

12. `import "os"收费`：这个函数导入了一个名为 "os" 的包，它提供了操作系统功能的支持。

13. `import "time"收费`：这个函数导入了一个名为 "time" 的包，它提供了时间相关的功能。

14. `import "encoding.收费"`：这个函数导入了一个名为 "encoding.收费" 的包，它提供了编码功能。

15. `import "json"收费`：这个函数导入了一个名为 "收费/json" 的包，它提供了 JSON 的解析和序列化的功能。

16. `import "log"收费`：这个函数导入了一个名为 "log" 的包，它提供了日志记录的功能。

17. `import "strings"收费`：这个函数导入了一个名为 "strings" 的包，它提供了字符串操作的功能。

18. `import "path"收费`：这个函数导入了一个名为 "path" 的包，它提供了路径解析的功能。

19. `import "system"收费`：这个函数导入了一个名为 "system" 的包，它提供了操作系统功能的支持。

20. `import "v2ray.com/core/收费"`：这个函数导入了一个名为 "v2ray.com/core/收费" 的包，它提供了 v2ray.com 项目的核心功能。


```go
// +build !confonly

package core

import (
	"context"
	"reflect"
	"sync"

	"v2ray.com/core/common"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/features"
	"v2ray.com/core/features/dns"
	"v2ray.com/core/features/dns/localdns"
	"v2ray.com/core/features/inbound"
	"v2ray.com/core/features/outbound"
	"v2ray.com/core/features/policy"
	"v2ray.com/core/features/routing"
	"v2ray.com/core/features/stats"
)

```

这段代码定义了一个名为Server的接口类型，以及一个名为resolution的 struct类型。

Server接口类型定义了一个common.Runnable类型的Server实例。该接口类型的Server实例必须存在于任何时间，且只能有一个实例。

resolution结构体定义了一个包含一个或多个reflect.Type类型的deps字段和一个common.Runnable类型的callback字段。deps字段是一个接口类型，它定义了Server实例的依赖关系，即该实例需要使用的类型。callback字段是一个reflect.Type类型的接口类型，它定义了一个函数，该函数在Server实例被调用时将被执行。

通过定义这些类型，可以创建一个Server实例，该实例可以使用reflect.Type类型的deps字段来初始化依赖关系，以及一个callback函数来执行一些操作。同时，可以确保在同一时间只能有一个Server实例在运行。


```go
// Server is an instance of V2Ray. At any time, there must be at most one Server instance running.
type Server interface {
	common.Runnable
}

// ServerType returns the type of the server.
func ServerType() interface{} {
	return (*Instance)(nil)
}

type resolution struct {
	deps     []reflect.Type
	callback interface{}
}

```

该代码定义了两个函数：`getFeature` 和 `resolve`。

函数 `getFeature` 接收一个 `allFeatures` 切片和一个 `reflect.Type` 类型的参数 `t`，并返回一个 `features.Feature` 类型的对象。它遍历 `allFeatures` 切片中的每个元素，如果遍历到的元素的 `reflect.Type()` 等于传递给 `t` 的 `reflect.Type`，那么就返回该元素，否则返回 `nil`。

函数 `resolve` 接收一个 `allFeatures` 切片和一个 `reflect.Type` 类型的参数 `t`，并返回一个布尔值表示是否成功解析依赖关系，以及一个错误类型或非空 `error`。它遍历 `allFeatures` 切片中的每个元素，并使用 `getFeature` 函数获取每个元素的值。然后，它遍历依赖关系链，直到到达传递给 `resolve` 的 `reflect.Type`，并尝试将其赋值给该 `reflect.Type`。如果 `resolve` 函数能够成功解析依赖关系，它将返回 `true`，否则返回一个非空 `error`。

函数 `getFeature` 和 `resolve` 都是构建在反射和类型系统之上，用于获取或设置 `features.Feature` 类型的函数，可用于在应用程序中查询或设置依赖关系。


```go
func getFeature(allFeatures []features.Feature, t reflect.Type) features.Feature {
	for _, f := range allFeatures {
		if reflect.TypeOf(f.Type()) == t {
			return f
		}
	}
	return nil
}

func (r *resolution) resolve(allFeatures []features.Feature) (bool, error) {
	var fs []features.Feature
	for _, d := range r.deps {
		f := getFeature(allFeatures, d)
		if f == nil {
			return false, nil
		}
		fs = append(fs, f)
	}

	callback := reflect.ValueOf(r.callback)
	var input []reflect.Value
	callbackType := callback.Type()
	for i := 0; i < callbackType.NumIn(); i++ {
		pt := callbackType.In(i)
		for _, f := range fs {
			if reflect.TypeOf(f).AssignableTo(pt) {
				input = append(input, reflect.ValueOf(f))
				break
			}
		}
	}

	if len(input) != callbackType.NumIn() {
		panic("Can't get all input parameters")
	}

	var err error
	ret := callback.Call(input)
	errInterface := reflect.TypeOf((*error)(nil)).Elem()
	for i := len(ret) - 1; i >= 0; i-- {
		if ret[i].Type() == errInterface {
			v := ret[i].Interface()
			if v != nil {
				err = v.(error)
			}
			break
		}
	}

	return true, err
}

```

这是一个 Go 语言中的类，名为 `Instance`。它的作用是实例化 V2Ray 中的所有功能，并封装了一个 `Instance` 类型的结构体，该结构体包含以下字段：

go
	access       sync.Mutex
	features    []features.Feature
	featureResolutions  []resolution
	running      bool


该类包含了一个互斥锁 `access`，一个包含 `features` 字段的数组 `features`，一个包含 `featureResolutions` 字段的数组 `featureResolutions`，以及一个包含 `running` 字段的布尔值 `running`。

该类还有一个方法 `AddInboundHandler`，它接受一个 `Instance` 类型的 `server` 和一个 `InboundHandlerConfig` 类型的参数。该方法的作用是在 V2Ray 的服务器中添加一个入站处理程序，并返回是否成功。

该方法的具体实现包括：

1. 从 `server` 中获取 `inbound.ManagerType` 字段，并将其 `.()` 方法返回，作为 `inboundManager` 变量。
2. 创建一个 `CreateObject` 函数，该函数接受一个 `InboundHandlerConfig` 类型的参数，并返回一个 `rawHandler` 类型的变量。
3. 从 `rawHandler` 获取一个 `Inbound.Handler` 类型的变量，作为 `handler` 变量。
4. 从 `inboundManager` 获取一个 `.AddHandler` 方法，并将其传入服务器上下文上下文，作为参数传递。
5. 从 `handler` 获取一个 `inbound.Manager` 类型的变量，作为 `server.ctx` 上下文上下文的 `inboundManager` 字段。
6. 创建一个 `CreateObject` 函数，该函数接受一个 `InboundHandlerConfig` 类型的参数，并返回一个 `rawHandler`` 类型的变量。
7. 从 `rawHandler` 创建一个 `Inbound.Handler`` 类型的变量，作为 `handler` 字段的值。
8. 将 `handler` 字段的值作为参数传递给 `inboundManager.AddHandler` 方法，得到一个是否成功的结果。

该类的实例可以被传递给其他 Go 语言中的函数或方法，以实例化 V2Ray 的服务器功能。


```go
// Instance combines all functionalities in V2Ray.
type Instance struct {
	access             sync.Mutex
	features           []features.Feature
	featureResolutions []resolution
	running            bool

	ctx context.Context
}

func AddInboundHandler(server *Instance, config *InboundHandlerConfig) error {
	inboundManager := server.GetFeature(inbound.ManagerType()).(inbound.Manager)
	rawHandler, err := CreateObject(server, config)
	if err != nil {
		return err
	}
	handler, ok := rawHandler.(inbound.Handler)
	if !ok {
		return newError("not an InboundHandler")
	}
	if err := inboundManager.AddHandler(server.ctx, handler); err != nil {
		return err
	}
	return nil
}

```

这段代码定义了两个函数：addInboundHandlers 和 addOutboundHandler。它们的作用是确保Inbound 和 Outbound 处理器（Handler）在服务器上注册并配置正确。

函数addInboundHandlers接收一个服务器实例（Instance）和一个或多个Inbound Handler Configuration对象的切片。函数首先遍历 configs 中的所有 Inbound Handler Configuration 对象。对于每个对象，如果尝试调用AddInboundHandler 函数时出现错误，那么返回错误。否则，将 Inbound Handler 添加到服务器上。

函数addOutboundHandler与addInboundHandlers类似，但针定期望的Outbound Handler Configuration对象。如果尝试创建或AddOutbound Handler 函数时出现错误，那么返回错误。如果成功创建或添加Outbound Handler，则返回0，表示操作成功。


```go
func addInboundHandlers(server *Instance, configs []*InboundHandlerConfig) error {
	for _, inboundConfig := range configs {
		if err := AddInboundHandler(server, inboundConfig); err != nil {
			return err
		}
	}

	return nil
}

func AddOutboundHandler(server *Instance, config *OutboundHandlerConfig) error {
	outboundManager := server.GetFeature(outbound.ManagerType()).(outbound.Manager)
	rawHandler, err := CreateObject(server, config)
	if err != nil {
		return err
	}
	handler, ok := rawHandler.(outbound.Handler)
	if !ok {
		return newError("not an OutboundHandler")
	}
	if err := outboundManager.AddHandler(server.ctx, handler); err != nil {
		return err
	}
	return nil
}

```

这段代码定义了一个名为 addOutboundHandlers 的函数，其作用是添加出站接口的配置。具体来说，它接收一个 Server 实例和一个或多个 OutboundHandlerConfig 类型的参数，然后遍历所有的 OutboundHandlerConfig 配置，如果尝试添加出站接口时出现错误，则返回该错误。

函数的实现过程中，首先定义了一个名为 configs 的变量，它是一个 OutboundHandlerConfig 类型的数组。然后，遍历 configs 数组中的每一个元素，如果接收到一个 OutboundHandlerConfig 类型的参数，则将其添加到 configs 上。

最后，函数返回一个 nil 表示没有错误。


```go
func addOutboundHandlers(server *Instance, configs []*OutboundHandlerConfig) error {
	for _, outboundConfig := range configs {
		if err := AddOutboundHandler(server, outboundConfig); err != nil {
			return err
		}
	}

	return nil
}

// RequireFeatures is a helper function to require features from Instance in context.
// See Instance.RequireFeatures for more information.
func RequireFeatures(ctx context.Context, callback interface{}) error {
	v := MustFromContext(ctx)
	return v.RequireFeatures(callback)
}

```

这段代码定义了两个函数，名为`New`和`NewWithContext`。它们的作用是创建一个新的V2Ray实例。

`New`函数接收一个`Config`参数。这个函数首先创建一个`Instance`对象，这个对象将用作V2Ray实例的起始点。然后它调用一个名为`initInstanceWithConfig`的内部函数，这个函数将根据给定的配置初始化V2Ray实例。如果初始化失败，函数将返回一个`nil`值，同时记录错误。如果初始化成功，函数将返回`nil`表示成功。

`NewWithContext`函数与`New`函数类似，但它使用了`ctx`参数。这个函数需要一个`Config`和一个`ctx`参数。函数创建一个`Instance`对象，并使用`initInstanceWithConfig`函数初始化它。然后，它将返回`nil`表示成功。

这两个函数的实现都依赖于一个名为`initInstanceWithConfig`的内部函数。这个函数的实现比较复杂，因此我没有提供完整的实现。


```go
// New returns a new V2Ray instance based on given configuration.
// The instance is not started at this point.
// To ensure V2Ray instance works properly, the config must contain one Dispatcher, one InboundHandlerManager and one OutboundHandlerManager. Other features are optional.
func New(config *Config) (*Instance, error) {
	var server = &Instance{ctx: context.Background()}

	err, done := initInstanceWithConfig(config, server)
	if done {
		return nil, err
	}

	return server, nil
}

func NewWithContext(config *Config, ctx context.Context) (*Instance, error) {
	var server = &Instance{ctx: ctx}

	err, done := initInstanceWithConfig(config, server)
	if done {
		return nil, err
	}

	return server, nil
}

```

这段代码是一个名为`func initInstanceWithConfig`的函数，它接收两个参数：`config`和`server`实例。函数的作用是初始化一个服务器实例，并在初始化过程中完成一些配置设置。

以下是具体的实现步骤：

1. 检查配置参数`config`是否为空，如果是，输出一条警告信息，提示全局传输设置已过时。

2. 检查配置参数`config.Transport`是否为空，如果是，尝试使用默认的网络传输设置，并返回`true`表示配置已成功。

3. 遍历配置参数`config.App`中的所有应用程序设置，并尝试获取相应的设置实例。如果设置实例获取失败，返回`true`表示配置已成功。

4. 使用从`config.App`中获取的设置创建服务器实例。

5. 检查服务器实例是否支持指定的`features`，如果是，尝试将`feature`添加到服务器实例中。如果设置失败，返回`true`表示配置已成功。

6. 遍历`essentialFeatures`中定义的所有必需功能，检查服务器实例是否支持指定功能。如果服务器实例不支持指定功能，返回错误。

7. 如果服务器初始化过程中没有错误，尝试添加到服务器实例中的`feature`。

8. 检查服务器实例是否设置为使用默认DNS服务器，如果是，执行后续操作。

9. 检查服务器实例是否设置为使用策略管理器。如果是，尝试创建一个策略管理器实例并设置默认管理器。如果设置失败，返回错误。

10. 检查服务器实例是否设置为使用路由器。如果是，尝试创建一个路由器实例并设置默认路由器。如果设置失败，返回错误。

11. 检查服务器实例是否设置为设置统计器。如果是，尝试创建一个统计器实例。如果设置失败，返回错误。

12. 遍历`essentialFeatures`中定义的所有必需功能，检查服务器实例是否支持指定功能。如果服务器实例不支持指定功能，返回错误。

13. 如果服务器初始化过程中没有错误，返回`false`表示配置已成功。


```go
func initInstanceWithConfig(config *Config, server *Instance) (error, bool) {
	if config.Transport != nil {
		features.PrintDeprecatedFeatureWarning("global transport settings")
	}
	if err := config.Transport.Apply(); err != nil {
		return err, true
	}

	for _, appSettings := range config.App {
		settings, err := appSettings.GetInstance()
		if err != nil {
			return err, true
		}
		obj, err := CreateObject(server, settings)
		if err != nil {
			return err, true
		}
		if feature, ok := obj.(features.Feature); ok {
			if err := server.AddFeature(feature); err != nil {
				return err, true
			}
		}
	}

	essentialFeatures := []struct {
		Type     interface{}
		Instance features.Feature
	}{
		{dns.ClientType(), localdns.New()},
		{policy.ManagerType(), policy.DefaultManager{}},
		{routing.RouterType(), routing.DefaultRouter{}},
		{stats.ManagerType(), stats.NoopManager{}},
	}

	for _, f := range essentialFeatures {
		if server.GetFeature(f.Type) == nil {
			if err := server.AddFeature(f.Instance); err != nil {
				return err, true
			}
		}
	}

	if server.featureResolutions != nil {
		return newError("not all dependency are resolved."), true
	}

	if err := addInboundHandlers(server, config.Inbound); err != nil {
		return err, true
	}

	if err := addOutboundHandlers(server, config.Outbound); err != nil {
		return err, true
	}
	return nil, false
}

```

这段代码定义了一个名为Instance的V2Ray实例的类型实现，该类型实现了common.HasType接口。

func (s *Instance) Type() interface{} {
	return ServerType()
}

func (s *Instance) Close() error {
	s.access.Lock()
	defer s.access.Unlock()

	s.running = false

	var errors []interface{}
	for _, f := range s.features {
		if err := f.Close(); err != nil {
			errors = append(errors, err)
		}
	}
	if len(errors) > 0 {
		return newError("failed to close all features").Base(newError(serial.Concat(errors...)))
	}

	return nil
}


```go
// Type implements common.HasType.
func (s *Instance) Type() interface{} {
	return ServerType()
}

// Close shutdown the V2Ray instance.
func (s *Instance) Close() error {
	s.access.Lock()
	defer s.access.Unlock()

	s.running = false

	var errors []interface{}
	for _, f := range s.features {
		if err := f.Close(); err != nil {
			errors = append(errors, err)
		}
	}
	if len(errors) > 0 {
		return newError("failed to close all features").Base(newError(serial.Concat(errors...)))
	}

	return nil
}

```

这段代码定义了一个名为 "RequireFeatures" 的函数，该函数接受一个 "callback" 参数，该参数必须是一个 "Feature" 类型的接口。函数的作用是在所有 dependent features 都被注册后，调用传递给它的 "callback" 函数。

函数首先检查传递给它的 "callback" 参数是否是一个 "Function" 类型的类型，如果不是，则产生一个异常。然后，函数遍历 "callback" 类型中所有 "Feature" 类型的类型，并将它们存储在一个名为 "featureTypes" 的数组中。

接下来，函数创建一个名为 "r" 的 "Resolution" 类型的变量，该变量包含一个 "Dependencies" 字段，它类型为 "featureTypes" 数组，一个 "Callback" 字段，类型为 "callback" 类型。然后，函数调用 "resolve" 函数来解决依赖关系，并将 "r"、"callback" 和 "featureTypes" 作为参数传递给它。如果 "resolve" 函数成功，则返回，否则将 "r" 和 "featureTypes" 添加到 "s.featureResolutions" 数组中。

最后，函数返回，如果没有错误，则 "RequireFeatures" 函数的作用完成。


```go
// RequireFeatures registers a callback, which will be called when all dependent features are registered.
// The callback must be a func(). All its parameters must be features.Feature.
func (s *Instance) RequireFeatures(callback interface{}) error {
	callbackType := reflect.TypeOf(callback)
	if callbackType.Kind() != reflect.Func {
		panic("not a function")
	}

	var featureTypes []reflect.Type
	for i := 0; i < callbackType.NumIn(); i++ {
		featureTypes = append(featureTypes, reflect.PtrTo(callbackType.In(i)))
	}

	r := resolution{
		deps:     featureTypes,
		callback: callback,
	}
	if finished, err := r.resolve(s.features); finished {
		return err
	}
	s.featureResolutions = append(s.featureResolutions, r)
	return nil
}

```

这段代码定义了一个名为`AddFeature`的函数，该函数接受一个`Feature`实例作为参数，将其注册到当前实例中。

函数的具体实现包括以下几步：

1. 将新到的`Feature`实例添加到当前实例的`features`数组中。
2. 如果当前实例正在运行，检查新到的`Feature`是否可以开始，如果不能，输出错误并返回。
3. 如果当前实例的`featureResolutions`数组为空，直接返回。
4. 对于新到的`Feature`，检查其`resolve`方法是否已经完成，如果是，则执行解延迟操作，否则将`pendingResolutions`数组添加到`featureResolutions`数中。
5. 循环遍历`featureResolutions`数组，如果所有解析都已完成，则设置`AddFeature`函数的返回值为`pendingResolutions`数组，否则继续添加`pendingResolutions`数到`featureResolutions`数中。
6. 最后，如果`featureResolutions`数组仍然为空，则执行完毕，返回`nil`表示成功。如果`AddFeature`函数返回失败，则将其作为错误信息输出到日志中。


```go
// AddFeature registers a feature into current Instance.
func (s *Instance) AddFeature(feature features.Feature) error {
	s.features = append(s.features, feature)

	if s.running {
		if err := feature.Start(); err != nil {
			newError("failed to start feature").Base(err).WriteToLog()
		}
		return nil
	}

	if s.featureResolutions == nil {
		return nil
	}

	var pendingResolutions []resolution
	for _, r := range s.featureResolutions {
		finished, err := r.resolve(s.features)
		if finished && err != nil {
			return err
		}
		if !finished {
			pendingResolutions = append(pendingResolutions, r)
		}
	}
	if len(pendingResolutions) == 0 {
		s.featureResolutions = nil
	} else if len(pendingResolutions) < len(s.featureResolutions) {
		s.featureResolutions = pendingResolutions
	}

	return nil
}

```

这段代码定义了一个名为Instance的V2Ray实例的功能，用于获取指定类型的特征(也就是注册了对应类型的特征)。

如果V2Ray实例上注册了该特征类型，则返回该实例的特征中注册的该类型的第一个特征；否则返回nil。

该函数有一个参数featureType，它指定了要获取的特征类型。在函数内部，使用了一个名为getFeature的函数，该函数接收一个Instance类型的参数，并返回该实例上注册的以该类型注册的特征中的第一个特征。然后，将返回的第一个特征返回给调用者。

该函数的另一个参数是reflect.TypeOf，用于获取给定的featureType的类型，以便在函数内部根据该类型进行类型检查。


```go
// GetFeature returns a feature of the given type, or nil if such feature is not registered.
func (s *Instance) GetFeature(featureType interface{}) features.Feature {
	return getFeature(s.features, reflect.TypeOf(featureType))
}

// Start starts the V2Ray instance, including all registered features. When Start returns error, the state of the instance is unknown.
// A V2Ray instance can be started only once. Upon closing, the instance is not guaranteed to start again.
//
// v2ray:api:stable
func (s *Instance) Start() error {
	s.access.Lock()
	defer s.access.Unlock()

	s.running = true
	for _, f := range s.features {
		if err := f.Start(); err != nil {
			return err
		}
	}

	newError("V2Ray ", Version(), " started").AtWarning().WriteToLog()

	return nil
}

```

# `v2ray_test.go`

这段代码是一个 Go 语言编写的测试框架，用于测试 Core 包中的实现。它包括以下几个部分：

1. 导入相关库：通过导入 "testing" 和 "proto"，使得测试可以进行字符串和 protobuf 类型之间的交互。

2. 定义测试 protobuf 类型：通过使用 "github.com/golang/protobuf/proto" 和 "v2ray.com/core" 包中的 "Dispatcher" 和 "VMess" 类型，创建一个测试 protobuf 类型，用于测试 Core 包的实现。

3. 定义测试用例：通过定义一个名为 "test_core_dispatcher" 的测试函数，并且该函数使用了上面定义的 protobuf 类型，然后使用 Go 标准库中的 "testing" 包中的 "Run" 函数，实现了测试的目的。函数内部可以调用前面定义的 "Dispatcher" 和 "VMess" 类型的方法，对 Core 包进行测试。

4. 定义测试支持：通过定义一个名为 "test_core_dispatcher_test" 的测试函数，对 "Dispatcher" 进行测试。该函数使用 "testing" 包中的 "LocalSetup" 和 "TestFixture"，在测试开始时对 "Dispatcher" 进行初始化，并在测试完成后恢复它。

5. 定义测试函数：通过定义一个名为 "test_core_dispatcher_test" 的测试函数，该函数实现了 "Dispatcher" 的两个实现：

	* "test_core_dispatcher_test.go"：实现了 "Dispatcher" 的 "IsLocked" 方法的测试。
	* "test_core_dispatcher_test_2.go"：实现了 "Dispatcher" 的 "DispatcherWith512" 方法的测试。

	在这两个实现中，对 "Dispatcher" 进行了不同的测试，包括：

		+ 对 "Dispatcher" 是否可以创建空闲的 "Dispatcher" 进行测试。
		+ 对 "Dispatcher" 是否可以设置目标 UDP 服务器进行测试。
		+ 对 "Dispatcher" 是否可以设置目标 TCP 服务器进行测试。
		+ 对 "Dispatcher" 是否可以设置目标 HTTP 服务器进行测试。
		+ 对 "Dispatcher" 是否可以设置目标 HTTPS 服务器进行测试。
		+ 对 "Dispatcher" 是否可以设置目标 ARP 服务器进行测试。
		+ 对 "Dispatcher" 是否可以设置目标尼日墨手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标印度手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标欧洲手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标中国手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标美国手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标韩国手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标墨西哥手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标巴西手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标俄罗斯手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标日本手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标中国台湾手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标香港手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标台湾手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标新加坡手机进行测试。
		+ 对 "Dispatcher" 是否可以设置目标美国加拿大手机进行测试。


```go
package core_test

import (
	"testing"

	proto "github.com/golang/protobuf/proto"
	. "v2ray.com/core"
	"v2ray.com/core/app/dispatcher"
	"v2ray.com/core/app/proxyman"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/common/uuid"
	"v2ray.com/core/features/dns"
	"v2ray.com/core/features/dns/localdns"
	_ "v2ray.com/core/main/distro/all"
	"v2ray.com/core/proxy/dokodemo"
	"v2ray.com/core/proxy/vmess"
	"v2ray.com/core/proxy/vmess/outbound"
	"v2ray.com/core/testing/servers/tcp"
)

```

This is a Go program that starts a gRPC server using the protobuf-grpc library. The server listens for incoming connections on a specific port and for incoming requests it proxies the connection to the receiver which is defined in the received settings of the receiverConfig. It also creates an outbound handler that receives a connection from the server and proxies the connection to the receiver.

The server has a receiverSettings that defines the port range, the address of the receiver, and the network list of the receiver. The receiverSettings is defined as a serialized message that is then converted to a typed message.

The server also has a proxySettings that defines the address of the proxy server, the port number, and the network list of the proxy server. The proxySettings is also defined as a serialized message that is then converted to a typed message.

The receiver is defined as a struct that has the receiverConfig and the receiverSettings. The receiverConfig has the address of the receiver, the port number, and the network list of the receiver. The receiverSettings has the port range, the address of the receiver, and the network list of the receiver.

The outbound handler has a config that defines the proxySettings and the receiverSettings. The proxySettings has the address of the proxy server, the port number, and the network list of the proxy server. The receiverSettings has the port range, the address of the receiver, and the network list of the receiver.

The server starts by creating an instance of the gRPC server using the StartInstance function and then closes the connection to the server by calling the Close function.


```go
func TestV2RayDependency(t *testing.T) {
	instance := new(Instance)

	wait := make(chan bool, 1)
	instance.RequireFeatures(func(d dns.Client) {
		if d == nil {
			t.Error("expected dns client fulfilled, but actually nil")
		}
		wait <- true
	})
	instance.AddFeature(localdns.New())
	<-wait
}

func TestV2RayClose(t *testing.T) {
	port := tcp.PickPort()

	userId := uuid.New()
	config := &Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.InboundConfig{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
		},
		Inbound: []*InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(port),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(net.LocalHostIP),
					Port:    uint32(0),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP, net.Network_UDP},
					},
				}),
			},
		},
		Outbound: []*OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(0),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: userId.String(),
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	cfgBytes, err := proto.Marshal(config)
	common.Must(err)

	server, err := StartInstance("protobuf", cfgBytes)
	common.Must(err)
	server.Close()
}

```

# Contributor Covenant Code of Conduct

## Our Pledge

In the interest of fostering an open and welcoming environment, we as contributors and maintainers pledge to making participation in our project and our community a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

## Our Standards

Examples of behavior that contributes to creating a positive environment include:

* Using welcoming and inclusive language
* Being respectful of differing viewpoints and experiences
* Gracefully accepting constructive criticism
* Focusing on what is best for the community
* Showing empathy towards other community members

Examples of unacceptable behavior by participants include:

* The use of sexualized language or imagery and unwelcome sexual attention or advances
* Trolling, insulting/derogatory comments, and personal or political attacks
* Public or private harassment
* Publishing others' private information, such as a physical or electronic address, without explicit permission
* Other conduct which could reasonably be considered inappropriate in a professional setting

## Our Responsibilities

Project maintainers are responsible for clarifying the standards of acceptable behavior and are expected to take appropriate and fair corrective action in response to any instances of unacceptable behavior.

Project maintainers have the right and responsibility to remove, edit, or reject comments, commits, code, wiki edits, issues, and other contributions that are not aligned to this Code of Conduct, or to ban temporarily or permanently any contributor for other behaviors that they deem inappropriate, threatening, offensive, or harmful.

## Scope

This Code of Conduct applies both within project spaces and in public spaces when an individual is representing the project or its community. Examples of representing a project or community include using an official project e-mail address, posting via an official social media account, or acting as an appointed representative at an online or offline event. Representation of a project may be further defined and clarified by project maintainers.

## Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting the project team at love@v2ray.com. The project team will review and investigate all complaints, and will respond in a way that it deems appropriate to the circumstances. The project team is obligated to maintain confidentiality with regard to the reporter of an incident. Further details of specific enforcement policies may be posted separately.

Project maintainers who do not follow or enforce the Code of Conduct in good faith may face temporary or permanent repercussions as determined by other members of the project's leadership.

## Attribution

This Code of Conduct is adapted from the [Contributor Covenant][homepage], version 1.4, available at [http://contributor-covenant.org/version/1/4][version]

[homepage]: http://contributor-covenant.org
[version]: http://contributor-covenant.org/version/1/4/


如果你遇到的问题不是 V2Ray 的 bug，比如你不清楚要如何配置，请使用[Discussion](https://github.com/v2fly/discussion/issues)进行讨论。

此 Issue 会被立即关闭。

If you are not sure if your question is truely a bug in V2Ray, please discuss it [here](https://github.com/v2fly/discussion/issues) first.

This issue will be closed immediately.


Please Move to https://github.com/v2fly/v2ray-core/pulls


---
name: V2Ray 程序问题
about: "提交一个 V2Ray 的程序问题报告。"
---

除非特殊情况，请完整填写所有问题。不按模板发的 issue 将直接被关闭。
如果你遇到的问题不是 V2Ray 的 bug，比如你不清楚要如何配置，请使用[Discussion](https://github.com/v2fly/discussion/issues)进行讨论。

1) 你正在使用哪个版本的 V2Ray？（如果服务器和客户端使用了不同版本，请注明）

2) 你的使用场景是什么？比如使用 Chrome 通过 Socks/VMess 代理观看 YouTube 视频。

3) 你看到的不正常的现象是什么？（请描述具体现象，比如访问超时，TLS 证书错误等）

4) 你期待看到的正确表现是怎样的？

5) 请附上你的配置（提交 Issue 前请隐藏服务器端IP地址）。

服务器端配置:

```gojavascript
    // 在这里附上服务器端配置文件
```

客户端配置:

```gojavascript
    // 在这里附上客户端配置
```

6)  请附上出错时软件输出的错误日志。在 Linux 中，日志通常在 `/var/log/v2ray/error.log` 文件中。

服务器端错误日志:

```gojavascript
    // 在这里附上服务器端日志
```

客户端错误日志:

```gojavascript
    // 在这里附上客户端日志
```

7) 请附上访问日志。在 Linux 中，日志通常在 `/var/log/v2ray/access.log` 文件中。

```gojavascript
    // 在这里附上服务器端日志
```

8) 其它相关的配置文件（如 Nginx）和相关日志。

9) 如果 V2Ray 无法启动，请附上 `--test` 输出。

通常的命令为 `/usr/bin/v2ray/v2ray --test --config /etc/v2ray/config.json`。请按实际情况修改。

10) 如果 V2Ray 服务运行不正常，请附上 journal 日志。

通常的命令为 `journalctl -u v2ray`。

请预览一下你填的内容再提交。


---
name: Bug report
about: "Create a bug report to help us improve"
---

Please answer all the questions with enough information. All issues not following this template will be closed immediately.
If you are not sure if your question is truely a bug in V2Ray, please discuss it [here](https://github.com/v2fly/discussion/issues) first.

1) What version of V2Ray are you using (If you deploy different version on server and client, please explicitly point out)?

2) What's your scenario of using V2Ray? E.g., Watching YouTube videos in Chrome via Socks/VMess proxy.

3) What did you see? (Please describe in detail, such as timeout, fake TLS certificate etc)

4) What's your expectation?

5) Please attach your configuration file (**Mask IP addresses before submit this issue**).

Server configuration:

```gojavascript
    // Please attach your server configuration here.
```

Client configuration:

```gojavascript
    // Please attach your client configuration here.
```

6) Please attach error logs, especially the bottom lines if the file is large. Error log file is usually at `/var/log/v2ray/error.log` on Linux.

Server error log:

```gojavascript
    // Please attach your server error log here.
```

Client error log:

```gojavascript
    // Please attach your client error log here.
```

7) Please attach access log. Access log is usually at '/var/log/v2ray/access.log' on Linux.

```gojavascript
    // Please attach your server access log here.
```

8) Other configurations (such as Nginx) and logs.

9) If V2Ray doesn't run, please attach output from `--test`.

The command is usually `/usr/bin/v2ray/v2ray --test --config /etc/v2ray/config.json`, but may vary according to your scenario.

10) If V2Ray service doesn't run, please attach journal log.

Usual command is `journalctl -u v2ray`.

Please review your issue before submitting.

---
name: Other
about: "其它问题请使用 https://github.com/v2fly/discussion/issues 进行讨论 / Please discuss other issues at https://github.com/v2fly/discussion/issues"
---

如果你遇到的问题不是 V2Ray 的 bug，比如你不清楚要如何配置，请使用[Discussion](https://github.com/v2fly/discussion/issues)进行讨论。

此 Issue 会被立即关闭。

If you are not sure if your question is truely a bug in V2Ray, please discuss it [here](https://github.com/v2fly/discussion/issues) first.

This issue will be closed immediately.


# `app/app.go`

这段代码定义了一个名为 "app" 的包，其中包含 V2Ray 的功能实现。这些功能可能会在运行时被启用或禁用。由于 V2Ray 是一款第三方网络加速工具，具体的功能和实现可能因不同的 V2Ray 版本而异。


```go
// Package app contains feature implementations of V2Ray. The features may be enabled during runtime.
package app

```

# `app/commander/commander.go`

这段代码是一个 Go 语言编写的构建命令，主要作用是编译命令并输出一个不包含 `.d/*` 规则的依赖文件。

具体来说，代码首先执行 `+build` 操作，这将编译当前目录下的所有 Go 源文件并输出名为 `.o` 的目标文件。接着，定义了一个名为 `!confonly` 的依赖规则，它告诉 Go 编译器在编译时不要输出任何 `.d/*` 规则的依赖文件，这样可以避免在构建过程中产生无用的文件。

在编译命令中，还通过导入一些需要的包来获得必要的功能。例如，使用了 `google.golang.org/grpc` 包来建立与外部服务器的通信管道，使用了 `v2ray.com/core` 包中的 `errorgen` 函数来生成错误日志，以及使用了 `v2ray.com/core/features/outbound` 包中的 `outbound` 功能来处理网络通信。

最后，通过 `command.use()` 函数将 `build` 命令添加到已定义的命令列表中，这样就可以使用 `build` 命令来编译依赖文件并生成可执行文件了。


```go
// +build !confonly

package commander

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
	"context"
	"net"
	"sync"

	"google.golang.org/grpc"

	"v2ray.com/core"
	"v2ray.com/core/common"
	"v2ray.com/core/common/signal/done"
	"v2ray.com/core/features/outbound"
)

```

该代码定义了一个名为 Commander 的 struct 类型，它提供了一个 gRPC 方法到外部客户端的的功能。

结构体中包含一个名为 sync.Mutex 的类型，它用于保证在某些操作中只有一个线程能够访问到 Commander 实例。

有一个名为 server 的类型，它是一个 gRPC Server。

有一个名为 services 的类型，它是一个数组，包含了通过 outbound.Manager 进行管理的服务实例。

有一个名为 ohm 的类型，它是一个 outbound.Manager 实例，用于管理 outbound 服务。

有一个名为 NewCommander 的函数，该函数接收一个配置实例和一个 outbound.Manager 实例。

该函数首先创建了一个 Commander 实例，其中包含了一个自定义的 tag，该 tag 将决定如何使用以身作则。

然后，该函数遍历给定的配置实例中的服务实例，使用 common.CreateObject 函数将每个服务实例创建为 outbound.Manager 类型。

最后，将创建好的服务实例添加到 services 数组中，并返回生成的 Commander 实例和错误。


```go
// Commander is a V2Ray feature that provides gRPC methods to external clients.
type Commander struct {
	sync.Mutex
	server   *grpc.Server
	services []Service
	ohm      outbound.Manager
	tag      string
}

// NewCommander creates a new Commander based on the given config.
func NewCommander(ctx context.Context, config *Config) (*Commander, error) {
	c := &Commander{
		tag: config.Tag,
	}

	common.Must(core.RequireFeatures(ctx, func(om outbound.Manager) {
		c.ohm = om
	}))

	for _, rawConfig := range config.Service {
		config, err := rawConfig.GetInstance()
		if err != nil {
			return nil, err
		}
		rawService, err := common.CreateObject(ctx, config)
		if err != nil {
			return nil, err
		}
		service, ok := rawService.(Service)
		if !ok {
			return nil, newError("not a Service.")
		}
		c.services = append(c.services, service)
	}

	return c, nil
}

```

该代码定义了一个名为 "Commander" 的接口，该接口实现了 common.HasType 和 common.Runnable 两个接口。

在 `Commander` 的 `Type()` 函数中，返回一个指向 "Commander" 类型的指针，这样就可以通过该指针来调用 "Commander" 提供的服务了。

`Commander` 的 `Start()` 函数会尝试启动一个 gRPC 服务器，并将所有服务注册到服务器上。然后创建一个 Outbound 类型的监听器，用于监听来自服务器的连接请求。

如果服务器启动成功并监听端口，该函数就会运行一个 gRPC 服务器循环，等待来自客户端的连接请求，并将它们转发给请求者。如果错误，函数会记录错误并输出错误信息。

另外，如果 "Commander" 的 `ohm.RemoveHandler()` 或 `ohm.AddHandler()` 函数在传递参数时出现错误，函数会记录错误并输出错误信息。


```go
// Type implements common.HasType.
func (c *Commander) Type() interface{} {
	return (*Commander)(nil)
}

// Start implements common.Runnable.
func (c *Commander) Start() error {
	c.Lock()
	c.server = grpc.NewServer()
	for _, service := range c.services {
		service.Register(c.server)
	}
	c.Unlock()

	listener := &OutboundListener{
		buffer: make(chan net.Conn, 4),
		done:   done.New(),
	}

	go func() {
		if err := c.server.Serve(listener); err != nil {
			newError("failed to start grpc server").Base(err).AtError().WriteToLog()
		}
	}()

	if err := c.ohm.RemoveHandler(context.Background(), c.tag); err != nil {
		newError("failed to remove existing handler").WriteToLog()
	}

	return c.ohm.AddHandler(context.Background(), &Outbound{
		tag:      c.tag,
		listener: listener,
	})
}

```

此代码实现了一个名为`Commander`的类，该类使用`common.Closable`实现了`Closable`接口。其`Close`方法的具体作用如下：

1. 首先，使用`c.Lock`方法对要关闭的资源进行锁定，以确保在关闭过程中不会被其他请求打扰。
2. 在锁定后，使用`c.server`获取已关联的服务器，如果服务器不为空，则调用`c.server.Stop`方法使其停止运行，并将其服务器设置为`nil`，这将释放服务器资源并关闭服务器。
3. 最后，返回一个`nil`表示成功关闭服务器。

`init`函数的作用是注册`Commander`类为指定配置下的`Closable`实现。它接受一个`Config`对象和一个`Context`参数，在创建`Commander`实例时，将使用这个配置对象来初始化服务器。


```go
// Close implements common.Closable.
func (c *Commander) Close() error {
	c.Lock()
	defer c.Unlock()

	if c.server != nil {
		c.server.Stop()
		c.server = nil
	}

	return nil
}

func init() {
	common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
		return NewCommander(ctx, cfg.(*Config))
	}))
}

```