# trojan-go源码解析 8

# `test/util/target.go`

这段代码是一个 Go 语言编写的 packages.util 包中的函数。该包中包含了一些与util相关的函数，包括从网络请求获取数据、创建 WebSocket 连接、加密数据、生成随机数等。

具体来说，该代码包括以下几个函数：

- net.get：从网络获取数据，包括 HTTP 和 WebSocket 请求。
- http.Client.Could：判断 HTTP 请求是否成功，并返回 HTTP 状态码。
- ioutil.WriteFile：将内容写入文件。
- log.Logger：创建一个日志logger，可以输出到控制台、日志文件等。
- random.UUID：生成一个随机 UUID。
- time.Now：获取当前时间。
- golang.org/x/net/websocket：通过 WebSocket 连接与其他服务器进行通信。
- trojan.core.Trojan：创建一个 Trojan 对象。
- trojan.util.Console：创建一个简单的控制台工具函数。


```go
package util

import (
	"crypto/rand"
	"fmt"
	"io"
	"io/ioutil"
	"net"
	"net/http"
	"sync"
	"time"

	"golang.org/x/net/websocket"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/log"
)

```

该代码使用了Go语言的websocket包实现了简单的HTTP服务器和客户端。主要步骤如下：

1. 定义了两个变量HTTPAddr和HTTPPort，用于存储HTTP服务器监听的地址和端口。

2. 定义了一个名为runHelloHTTPServer的函数，用于启动HTTP服务器。

3. 在函数内部定义了一个名为httpHello的函数，用于处理HTTP请求。该函数接收一个http.ResponseWriter和一个http.Request对象，并写入"HelloWorld"字符串到响应输出流中。

4. 在httpHello函数内部，使用websocket.NewConfig函数创建一个HTTP-Websocket转换的配置对象，并使用websocket.Server构造一个HTTP服务器。

5. 使用websocket.Server的Config和Handler函数配置服务器，并实现Server的逻辑。这里的逻辑包括：验证传入的配置对象，创建一个Websocket服务器实例并处理客户端请求，将"HelloWorld"字符串写入响应输出流中。

6. 创建一个带有handshake逻辑的Websocket服务器实例，并将其与上面创建的HTTP服务器连接。

7. 创建一个HTTP服务器，并将其与上面创建的Websocket服务器连接，实现HTTP请求和响应的映射。

8. 通过调用HTTPAddr和HTTPPort来设置HTTP服务器监听的地址和端口，然后启动服务器并等待客户端连接。

9. 在启动服务器后，等待一段时间（这里设置为1秒）后，打印"http test server listening on" HTTPAddr。

10. 最后，调用wg.Done函数来等待HTTP服务器关闭。


```go
var (
	HTTPAddr string
	HTTPPort string
)

func runHelloHTTPServer() {
	httpHello := func(w http.ResponseWriter, req *http.Request) {
		w.Write([]byte("HelloWorld"))
	}

	wsConfig, err := websocket.NewConfig("wss://127.0.0.1/websocket", "https://127.0.0.1")
	common.Must(err)
	wsServer := websocket.Server{
		Config: *wsConfig,
		Handler: func(conn *websocket.Conn) {
			conn.Write([]byte("HelloWorld"))
		},
		Handshake: func(wsConfig *websocket.Config, httpRequest *http.Request) error {
			log.Debug("websocket url", httpRequest.URL, "origin", httpRequest.Header.Get("Origin"))
			return nil
		},
	}
	mux := &http.ServeMux{}
	mux.HandleFunc("/", httpHello)
	mux.HandleFunc("/websocket", wsServer.ServeHTTP)
	HTTPAddr = GetTestAddr()
	_, HTTPPort, _ = net.SplitHostPort(HTTPAddr)
	server := http.Server{
		Addr:    HTTPAddr,
		Handler: mux,
	}
	go server.ListenAndServe()
	time.Sleep(time.Second * 1) // wait for http server
	fmt.Println("http test server listening on", HTTPAddr)
	wg.Done()
}

```

这段代码定义了一个名为`runTCPEchoServer`的函数，用于运行一个基于TCP的echo服务器。函数接受一个`EchoAddr`参数和一个`EchoPort`参数，分别表示服务器监听的地址和端口号。

函数内部使用`net.Listen`函数创建一个TCP服务器连接器。然后使用`var`声明两个字符串类型的变量`EchoAddr`和`EchoPort`，分别存储服务器监听的地址和端口号。

接下来，函数使用一个`for`循环，持续接受客户端连接。在循环内部，使用一个`while`循环，持续处理连接。在循环处理过程中，使用`conn.Accept`方法接受一个TCP连接，然后使用`go`关键字结合一个闭包函数，将处理连接的职责转交给一个内部函数。这个内部函数使用一个`for`循环，持续读取客户端发送的数据包，然后使用`conn.Write`方法发送一个数据包给客户端。如果出现任何错误，比如客户端关闭或网络连接中断，函数会返回。

最后，函数内部的一个`defer`关键字声明一个函数，在函数内部关闭客户端连接。


```go
var (
	EchoAddr string
	EchoPort int
)

func runTCPEchoServer() {
	listener, err := net.Listen("tcp", EchoAddr)
	common.Must(err)
	wg.Done()
	go func() {
		defer listener.Close()
		for {
			conn, err := listener.Accept()
			if err != nil {
				return
			}
			go func(conn net.Conn) {
				defer conn.Close()
				for {
					buf := make([]byte, 2048)
					conn.SetDeadline(time.Now().Add(time.Second * 5))
					n, err := conn.Read(buf)
					conn.SetDeadline(time.Time{})
					if err != nil {
						return
					}
					_, err = conn.Write(buf[0:n])
					if err != nil {
						return
					}
				}
			}(conn)
		}
	}()
}

```

该代码实现了一个 UDP Echo 服务器，其目的是监听来自客户端的 UDP 数据包并返回一个重复的数据包。

具体来说，代码创建了一个名为 "runUDPEchoServer" 的函数，该函数通过调用 "net.ListenPacket" 函数来监听来自客户端的 UDP 数据包。然后，它创建了一个名为 "conn" 的 UDP 套接字，通过调用 "net.ListenPacket" 函数来监听来自客户端的 UDP 数据包。

接下来，代码使用了一个名为 "wg.Done" 的 goroutine，该 goroutine 会等待客户端套接字关闭并执行一个函数 "func()"，该函数将重复发送从客户端套接字读取的数据包到客户端套接字中。

最后，代码使用了一个 for 循环来不断从客户端套接字中读取数据包，并使用 "conn.WriteTo" 函数将数据包发送回客户端套接字中，然后使用 "buf" 变量中的数据来更新客户端套接字。循环的每次读取都使用 "log.Info" 函数记录客户端的地址。

因此，该代码的作用是创建一个 UDP Echo 服务器，接收客户端发送的 UDP 数据包，并重复发送一个数据包到客户端。


```go
func runUDPEchoServer() {
	conn, err := net.ListenPacket("udp", EchoAddr)
	common.Must(err)
	wg.Done()
	go func() {
		for {
			buf := make([]byte, 1024*8)
			n, addr, err := conn.ReadFrom(buf)
			if err != nil {
				return
			}
			log.Info("Echo from", addr)
			conn.WriteTo(buf[0:n], addr)
		}
	}()
}

```

这段代码定义了一个名为func GeneratePayload的函数，该函数接收一个整数参数length，返回一个长度为length的byte数组。函数内部使用了make函数创建了一个长度为length的byte数组，并使用io.ReadFull函数从随机读写器中读取整数length的byte数据并将其赋值给数组。

该代码还定义了一个名为runTCPBlackHoleServer的函数。该函数使用net.Listen函数以TCP协议在黑寡妇漏洞地址（BlackHoleAddr）上启动一个服务器。然后，该函数使用一个while循环来等待客户端连接。当客户端连接时，函数内部使用一个close函数来关闭客户端与服务器的连接。然后函数内部使用io.Copy函数将从客户端套接字中读取的bytes字节数据复制到客户端的缓冲区。

这段代码的主要目的是创建一个黑寡妇漏洞服务器，让攻击者可以使用该漏洞来窃取敏感数据。


```go
func GeneratePayload(length int) []byte {
	buf := make([]byte, length)
	io.ReadFull(rand.Reader, buf)
	return buf
}

var (
	BlackHoleAddr string
	BlackHolePort int
)

func runTCPBlackHoleServer() {
	listener, err := net.Listen("tcp", BlackHoleAddr)
	common.Must(err)
	wg.Done()
	go func() {
		defer listener.Close()
		for {
			conn, err := listener.Accept()
			if err != nil {
				return
			}
			go func(conn net.Conn) {
				io.Copy(ioutil.Discard, conn)
				conn.Close()
			}(conn)
		}
	}()
}

```

该函数的目的是创建一个UDP套接字并绑定到BlackHole地址，然后使用该套接字监听来自客户端的UDP数据包。

具体来说，它首先通过调用`net.ListenPacket`函数来创建一个UDP套接字，并指定连接类型为UDP，端口号为9000，以及目标地址为(-8000, 9000)。然后它调用`net.MustListen`函数来确保套接字连接成功。

接下来，该函数创建一个缓冲区`buf`，其大小为1024字节，用于存储从客户端收到的数据包。然后它使用一个循环来反复从套接字中读取数据包，并将其存储在`buf`中。

最后，该函数内部定义了一个内部函数`go func()`，该函数会在`buf`缓冲区中放置一个1024字节的缓冲区，然后使用`conn.ReadFrom`函数从套接字中读取数据包并将其存储在缓冲区中。这样做是为了防止`conn.ReadFrom`函数立即关闭套接字连接，导致未完成的读取数据包丢失。


```go
func runUDPBlackHoleServer() {
	conn, err := net.ListenPacket("udp", BlackHoleAddr)
	common.Must(err)
	wg.Done()
	go func() {
		defer conn.Close()
		buf := make([]byte, 1024*8)
		for {
			_, _, err := conn.ReadFrom(buf)
			if err != nil {
				return
			}
		}
	}()
}

```

该代码使用了Go语言中的并发编程库——sync。它创建了一个名为"wg"的同步等待组，用于等待一系列函数的完成。

在函数内部，首先创建一个空等待组，然后调用一个名为"init"的函数，该函数中添加了5个工作负载(job)，并调用了两个名为"runHelloHTTPServer"和"runTCPEchoServer"的函数。

接下来，使用"common.PickPort"函数选择一个TCP端口(端口号为127.0.0.1)，并将其用于设置一个名为"EchoPort"的变量，另一个名为"EchoAddr"的变量则使用该端口号和数字127.0.0.1来创建一个指向IP地址的指针。

然后，它选择一个TCP端口(端口号为127.0.0.1)，并将其设置为名为"BlackHolePort"的变量，另一个名为"BlackHoleAddr"的变量则使用该端口号和数字127.0.0.1来创建一个指向IP地址的指针。

接下来，使用"runTCPEchoServer"函数运行一个TCPecho服务器。然后，使用"runUDPEchoServer"函数运行一个UDPecho服务器。

最后，它使用"runTCPBlackHoleServer"函数运行一个TCP Blackhole服务器。然后，使用"runUDPBlackHoleServer"函数运行一个UDP Blackhole服务器。

最后，使用"wg.Wait"函数等待所有工作负载的完成，然后立即返回。


```go
var wg = sync.WaitGroup{}

func init() {
	wg.Add(5)
	runHelloHTTPServer()

	EchoPort = common.PickPort("tcp", "127.0.0.1")
	EchoAddr = fmt.Sprintf("127.0.0.1:%d", EchoPort)

	BlackHolePort = common.PickPort("tcp", "127.0.0.1")
	BlackHoleAddr = fmt.Sprintf("127.0.0.1:%d", BlackHolePort)

	runTCPEchoServer()
	runUDPEchoServer()

	runTCPBlackHoleServer()
	runUDPBlackHoleServer()

	wg.Wait()
}

```

# `test/util/util.go`

这段代码定义了一个名为`util`的包，其中包含了一些函数。

1. `CheckConn`函数用于检查两个网络连接是否正常工作，返回一个布尔值。该函数接收两个`net.Conn`对象作为参数，并在内部创建了两个缓冲区`payload1`和`payload2`，然后发送这两个缓冲区的内容。接着，该函数创建了一个名为`wg`的`sync.WaitGroup`，并让该组中的两个函数分别向网络连接发送数据。最后，该函数检查两个内存缓冲区是否与预期相同，并返回一个布尔值。

2. `fmt.Printf`函数用于将一个字符串格式化并输出到标准输出。该函数接收一个字符串作为参数，并将其打印到标准输出(通常是终端)。

3. `net.Listen`函数用于创建一个TCP套接字并绑定到指定端口上，返回一个`net.Listen`实例。该函数接收一个端口号作为参数，并在内部创建一个套接字并绑定到该端口上。

4. `bytes.Buffer`函数用于创建一个字节缓冲区，并允许对缓冲区进行修改。该函数没有提供任何可输出的接口，因此通常不会与其他函数组合使用。

5. `crypto.rand`函数用于从标准随机数生成一个随机整数。该函数返回一个随机整数。

6. `sync.WaitGroup`函数用于等待一组函数完成。该函数接收一个函数作为参数，并在内部创建了一个`sync.WaitGroup`。接着，该函数调用其中一个函数，并传入该函数的名称作为参数。最后，该函数返回一个`bool`表示所有函数是否都已完成。


```go
package util

import (
	"bytes"
	"crypto/rand"
	"fmt"
	"net"
	"sync"

	"github.com/p4gefau1t/trojan-go/common"
)

// CheckConn checks if two netConn were connected and work properly
func CheckConn(a net.Conn, b net.Conn) bool {
	payload1 := make([]byte, 1024)
	payload2 := make([]byte, 1024)

	result1 := make([]byte, 1024)
	result2 := make([]byte, 1024)

	rand.Reader.Read(payload1)
	rand.Reader.Read(payload2)

	wg := sync.WaitGroup{}
	wg.Add(2)

	go func() {
		a.Write(payload1)
		a.Read(result2)
		wg.Done()
	}()

	go func() {
		b.Read(result1)
		b.Write(payload2)
		wg.Done()
	}()

	wg.Wait()

	return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
}

```

该代码定义了一个名为`CheckPacketOverConn`的函数，用于检查两个通过连接进行的数据包连接是否正常工作。函数接受两个参数，一个是`net.PacketConn`类型的变量`a`，另一个是`net.PacketConn`类型的变量`b`。

函数首先创建一个`net.UDPAddr`类型的变量`addr`，指定了源IP为`127.0.0.1`，并指定了一个TCP端口。然后，函数创建了两个长度为1024的切片`payload1`和`payload2`，用于存储数据。

接下来，函数分别向`a`和`b`发送数据包。为了确保数据包能够正常到达，函数首先调用了一个名为`common.PickPort`的函数，选择一个TCP端口（127.0.0.1），然后将该端口作为`addr`的IP地址。接着，函数调用`common.Must2`函数向`a`发送数据包，并将其写入`payload1`。然后，函数再次调用`common.Must2`函数向`b`发送数据包，并将其写入`result2`。

最后，函数使用`bytes.Equal`函数比较`payload1`和`result1`，以及`payload2`和`result2`是否相等。如果它们相等，那么函数返回`true`，表示数据包连接正常工作。否则，函数返回`false`。


```go
// CheckPacketOverConn checks if two PacketConn streaming over a connection work properly
func CheckPacketOverConn(a, b net.PacketConn) bool {
	port := common.PickPort("tcp", "127.0.0.1")
	addr := &net.UDPAddr{
		IP:   net.ParseIP("127.0.0.1"),
		Port: port,
	}

	payload1 := make([]byte, 1024)
	payload2 := make([]byte, 1024)

	result1 := make([]byte, 1024)
	result2 := make([]byte, 1024)

	rand.Reader.Read(payload1)
	rand.Reader.Read(payload2)

	common.Must2(a.WriteTo(payload1, addr))
	_, addr1, err := b.ReadFrom(result1)
	common.Must(err)
	if addr1.String() != addr.String() {
		return false
	}

	common.Must2(a.WriteTo(payload2, addr))
	_, addr2, err := b.ReadFrom(result2)
	common.Must(err)
	if addr2.String() != addr.String() {
		return false
	}

	return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
}

```

该函数 `CheckPacket` 的作用是检查数据包是否有效，并返回一个布尔值。它接收两个参数 `a` 和 `b`，这两个参数分别是一个网络套接字（net.PacketConn）和一个套接字流（net.Packet）。

函数内部首先创建了两个长度为1024的切片 `payload1` 和 `payload2`，用于存储数据。然后，它调用了两个随机数生成器 `rand.Reader`，分别从 `a` 和 `b` 套接字流中读取数据，并将这些数据存储到 `payload1` 和 `payload2` 中。

接着，函数使用 `a.WriteTo` 函数将数据 `payload1` 写入到 `a` 套接字流中，并将 `a.LocalAddr()` 发送给 `b` 套接字流。然后，函数使用 `b.ReadFrom` 函数从 `b` 套接字流中读取数据，并将其存储到 `result1` 变量中。

接下来，函数再次使用 `b.WriteTo` 函数将数据 `payload2` 写入到 `b` 套接字流中，并将 `a.LocalAddr()` 发送给 `b` 套接字流。然后，函数使用 `a.ReadFrom` 函数从 `a` 套接字流中读取数据，并将其存储到 `result2` 变量中。

最后，函数比较 `payload1` 和 `result1`，以及 `payload2` 和 `result2` 是否相等。如果它们相等，则函数返回 `true`，表示数据包有效；否则返回 `false`。


```go
func CheckPacket(a, b net.PacketConn) bool {
	payload1 := make([]byte, 1024)
	payload2 := make([]byte, 1024)

	result1 := make([]byte, 1024)
	result2 := make([]byte, 1024)

	rand.Reader.Read(payload1)
	rand.Reader.Read(payload2)

	_, err := a.WriteTo(payload1, b.LocalAddr())
	common.Must(err)
	_, _, err = b.ReadFrom(result1)
	common.Must(err)

	_, err = b.WriteTo(payload2, a.LocalAddr())
	common.Must(err)
	_, _, err = a.ReadFrom(result2)
	common.Must(err)

	return bytes.Equal(payload1, result1) && bytes.Equal(payload2, result2)
}

```

这段代码定义了一个名为 `GetTestAddr` 的函数，返回一个字符串类型的变量，用于存储测试目的的 IP 地址。函数的实现包括以下步骤：

1. 调用 `common.PickPort` 函数，这个函数的作用是选择一个 TCP 或 UDP 端口，同时指定一个 IP 地址（或使用默认的 IP 地址 127.0.0.1）。
2. 将选择到的端口号作为参数传入 `fmt.Sprintf` 函数，表示将选定的 IP 地址格式化为类似于 "127.0.0.1:<端口号>" 的字符串格式。
3. 将 `fmt.Sprintf` 函数的返回值作为最终的结果返回，这个结果将包含一个字符串，其中 `%d` 表示端口号，`<端口号>` 部分则是一个格式化后的字符串，用于与 IP 地址一起显示。


```go
func GetTestAddr() string {
	port := common.PickPort("tcp", "127.0.0.1")
	return fmt.Sprintf("127.0.0.1:%d", port)
}

```

# `tunnel/metadata.go`

这段代码定义了一个名为 "tunnel" 的包，其中定义了一个名为 "Command" 的字节类型变量。此外，代码还导入了一些外部的包，包括 "bytes"、"encoding/binary"、"fmt"、"net" 和 "strconv"。

接下来，代码中定义了一个 "Command" 类型的变量，但并没有对其进行初始化。然后，代码使用 "fmt.Printf" 函数输出了一些字符串，这些字符串可能是用于调试目的的。

最后，代码使用 "net" 包中的 "net.Listen" 函数创建了一个 TCP 监听端口为 ":3128"，并在监听端口上绑定了一个 "Command" 类型的变量。由于 "fmt.Printf" 函数没有提供任何输入或输出，因此无法知道 "Command" 变量被赋值为多少。


```go
package tunnel

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"io"
	"net"
	"strconv"

	"github.com/p4gefau1t/trojan-go/common"
)

type Command byte

```

这段代码定义了一个名为 Metadata 的结构体，其中包含一个命令类型字段 Metadata 和一个指向 Address 的指针类型字段 Metadata。

Metadata 结构体定义了一个 ReadFrom 方法，该方法接受一个 io.Reader 类型的参数，用于从该 reader 中读取 Metadata 结构体中的数据。

在 Metadata 的 ReadFrom 方法中，首先创建一个 byte 缓冲区并从 io.Reader 类型参数中读取数据，然后解码并获取命令类型字段的长度。接下来，创建一个 Address 类型的新实例，并从 io.Reader 类型参数中读取数据。最后，将解码后的命令类型字段赋给 Metadata 结构体的 Metadata 字段，并将解码后的 Address 实例赋给 Metadata 结构体的 Metadata 字段。

如果从 io.Reader 类型参数中读取数据时出现错误，则返回一个错误，否则将 nil 作为结果返回。


```go
type Metadata struct {
	Command
	*Address
}

func (r *Metadata) ReadFrom(rr io.Reader) error {
	byteBuf := [1]byte{}
	_, err := io.ReadFull(rr, byteBuf[:])
	if err != nil {
		return err
	}
	r.Command = Command(byteBuf[0])
	r.Address = new(Address)
	err = r.Address.ReadFrom(rr)
	if err != nil {
		return common.NewError("failed to marshal address").Base(err)
	}
	return nil
}

```

此代码定义了两个函数，分别是`func (r *Metadata) WriteTo`和`func (r *Metadata) Network`。

`func (r *Metadata) WriteTo`函数接收一个名为`w`的`io.Writer`类型和一个字符数组`r`作为参数。函数的作用是将`r`的`Command`字段写入到`w`中，并在写入前尝试从`r`的`Address`字段中写入数据。如果写入过程中出现错误，则返回该错误。最后，函数使用了`tcp`作为默认的网络类型，并尝试从`w`中写入数据。

`func (r *Metadata) Network`函数返回`r.Address.Network()`，它返回`r.Address`的`Network()`方法返回的类型。这个函数没有做任何处理，它只是返回了`r.Address`的网络类型。


```go
func (r *Metadata) WriteTo(w io.Writer) error {
	buf := bytes.NewBuffer(make([]byte, 0, 64))
	buf.WriteByte(byte(r.Command))
	if err := r.Address.WriteTo(buf); err != nil {
		return err
	}
	// use tcp by default
	r.Address.NetworkType = "tcp"
	_, err := w.Write(buf.Bytes())
	return err
}

func (r *Metadata) Network() string {
	return r.Address.Network()
}

```

这段代码定义了一个名为"Metadata"的类型，它包含一个名为"r"的指针变量。接着，定义了一个名为"AddressType"的枚举类型，它包含了IPv4、DomainName和IPv6三个选项。然后，定义了一个名为"Address"的结构体，其中包含一个名为"DomainName"的类型为"string"的变量、一个名为"Port"的类型为"int"的变量、一个名为"NetworkType"的类型为"string"的变量和一个名为"net.IP"的类型为"IPv4"的变量和一个名为"AddressType"的类型为"constant"的变量。接着，定义了一个名为"func"的函数，该函数接收一个名为"r"的指针变量和一个名为"String"的类型，并返回一个字符串类型的变量，该变量表示"r"的"Address"的"String()"形式。最后，在函数体中，通过解引用"r"指向的"Address"的"AddressType"字段，使用了"r.Address.String()"的表达式来获取"r.Address"的"String()"形式的地址字符串，并将其赋值给传入给"String()"的参数，最终返回该地址字符串。


```go
func (r *Metadata) String() string {
	return r.Address.String()
}

type AddressType byte

const (
	IPv4       AddressType = 1
	DomainName AddressType = 3
	IPv6       AddressType = 4
)

type Address struct {
	DomainName  string
	Port        int
	NetworkType string
	net.IP
	AddressType
}

```

这两段代码定义了两个函数，分别接收一个`*Address`类型的参数`a`，并返回相应的网络类型字符串。

第一个函数是一个名为`String()`的函数，接收一个`*Address`类型的参数`a`，然后根据`a.AddressType`的值，执行相应的操作并返回一个字符串。根据`a.AddressType`的值，可以确定要打印的字符串中的内容，可以是IPv4地址的`IP.String()`、IPv6地址的`IP.String()`、域名名称的`DomainName`以及套筒的`NetworkType`。

第二个函数是一个名为`Network()`的函数，接收一个`*Address`类型的参数`a`，并返回`a.NetworkType`，也就是IPv4地址或IPv6地址的网络类型。


```go
func (a *Address) String() string {
	switch a.AddressType {
	case IPv4:
		return fmt.Sprintf("%s:%d", a.IP.String(), a.Port)
	case IPv6:
		return fmt.Sprintf("[%s]:%d", a.IP.String(), a.Port)
	case DomainName:
		return fmt.Sprintf("%s:%d", a.DomainName, a.Port)
	default:
		return "INVALID_ADDRESS_TYPE"
	}
}

func (a *Address) Network() string {
	return a.NetworkType
}

```

该函数接受一个名为 `a` 的 *Address 类型的参数，并返回一个网络 IP 地址，或者错误。函数首先检查 `a` 的 `AddressType` 是否为 IPv4 或 IPv6，如果是，则函数会尝试使用该地址类型来获取 IP 地址。如果已经存在一个 IP 地址，则函数会将其作为返回值。否则，函数会尝试使用 `net.ResolveIPAddr` 函数来查找 `a` 的域名，并返回其 IP 地址。如果该函数在尝试解析域名时出现错误，则返回 `nil`。


```go
func (a *Address) ResolveIP() (net.IP, error) {
	if a.AddressType == IPv4 || a.AddressType == IPv6 {
		return a.IP, nil
	}
	if a.IP != nil {
		return a.IP, nil
	}
	addr, err := net.ResolveIPAddr("ip", a.DomainName)
	if err != nil {
		return nil, err
	}
	a.IP = addr.IP
	return addr.IP, nil
}

```

该代码定义了两个函数，分别为`NewAddressFromAddr`和`NewAddressFromHostPort`。这两个函数的作用是获取一个`*Address`类型的地址，如果失败则返回一个`error`类型的值。

`NewAddressFromAddr`函数的参数为`network`和`addr`，其中`network`是一个字符串，表示网络类型，`addr`是一个字符串，表示目标地址。函数首先通过`net.SplitHostPort`函数将addr转换为`host`和`portStr`两个参数，然后通过`strconv.ParseInt`函数将portStr转换为`port`类型的整数，最后使用`NewAddressFromHostPort`函数生成一个`*Address`类型的值，并将其返回。

`NewAddressFromHostPort`函数的参数也为`network`和`host`，其中`network`是一个字符串，表示网络类型，`host`是一个字符串，表示目标地址。函数首先使用`net.ParseIP`函数将host转换为`ip`地址，然后使用`ip.To4()`函数将其转换为`ip`地址的四字节表示形式，最后通过`strconv.ParseInt`函数将其转换为`port`类型的整数。函数使用`NewAddressFromHostPort`函数生成一个`*Address`类型的值，并将其返回。

这两个函数的实现主要依赖于`net`包，通过调用`net`包中的`SplitHostPort`函数将addr分离出`host`和`port`两个参数，然后在自定义函数中对其进行处理。


```go
func NewAddressFromAddr(network string, addr string) (*Address, error) {
	host, portStr, err := net.SplitHostPort(addr)
	if err != nil {
		return nil, err
	}
	port, err := strconv.ParseInt(portStr, 10, 32)
	common.Must(err)
	return NewAddressFromHostPort(network, host, int(port)), nil
}

func NewAddressFromHostPort(network string, host string, port int) *Address {
	if ip := net.ParseIP(host); ip != nil {
		if ip.To4() != nil {
			return &Address{
				IP:          ip,
				Port:        port,
				AddressType: IPv4,
				NetworkType: network,
			}
		}
		return &Address{
			IP:          ip,
			Port:        port,
			AddressType: IPv6,
			NetworkType: network,
		}
	}
	return &Address{
		DomainName:  host,
		Port:        port,
		AddressType: DomainName,
		NetworkType: network,
	}
}

```

This is a function that reads an IPv4 or IPv6 address from a network interface and an optional port number from a network interface. It is implemented using the Go programming language and is built on top of the Go standard library's "sync" and "net" packages.

The function takes a reader as an input and returns an error if any errors occur. If an error does not occur, the function returns a common.Error object with a specific error message based on the type of network address being read.

The function first checks if the network interface is a IPv4 or IPv6 address by examining the first 4 bytes of the network interface. If the first 4 bytes are not a valid IPv4 or IPv6 address, the function returns an error.

If the network interface is a IPv4 address, the function reads the port number from the remaining bytes of the network interface. If the remaining bytes cannot be read because of an error, the function returns an error.

If the network interface is a IPv6 address, the function reads the port number from the remaining bytes of the network interface. If the remaining bytes cannot be read because of an error, the function returns an error.

If the network interface is not a valid network address, the function returns an error.

The function also checks if the network address is an IPv4 address. If the network address is an IPv4 address, the function checks if the address is a loopback address by examining the last byte of the network address. If the last byte is a 0, the function assumes that the address is a loopback address and returns no error. If the last byte is a non-0 value, the function checks if the address is a valid IPv4 address by comparing it to a valid IPv4 address. If the address is not a valid IPv4 address, the function returns an error.

If the network address is an IPv6 address, the function checks if the address is a loopback address by examining the last byte of the network address. If the last byte is a 0, the function assumes that the address is a loopback address and returns no error. If the last byte is a non-0 value, the function checks if the address is a valid IPv6 address by comparing it to a valid IPv6 address. If the address is not a valid IPv6 address, the function returns an error.

If the network address is not a valid network address, the function returns an error.


```go
func (a *Address) ReadFrom(r io.Reader) error {
	byteBuf := [1]byte{}
	_, err := io.ReadFull(r, byteBuf[:])
	if err != nil {
		return common.NewError("unable to read ATYP").Base(err)
	}
	a.AddressType = AddressType(byteBuf[0])
	switch a.AddressType {
	case IPv4:
		var buf [6]byte
		_, err := io.ReadFull(r, buf[:])
		if err != nil {
			return common.NewError("failed to read IPv4").Base(err)
		}
		a.IP = buf[0:4]
		a.Port = int(binary.BigEndian.Uint16(buf[4:6]))
	case IPv6:
		var buf [18]byte
		_, err := io.ReadFull(r, buf[:])
		if err != nil {
			return common.NewError("failed to read IPv6").Base(err)
		}
		a.IP = buf[0:16]
		a.Port = int(binary.BigEndian.Uint16(buf[16:18]))
	case DomainName:
		_, err := io.ReadFull(r, byteBuf[:])
		length := byteBuf[0]
		if err != nil {
			return common.NewError("failed to read domain name length")
		}
		buf := make([]byte, length+2)
		_, err = io.ReadFull(r, buf)
		if err != nil {
			return common.NewError("failed to read domain name")
		}
		// the fucking browser uses IP as a domain name sometimes
		host := buf[0:length]
		if ip := net.ParseIP(string(host)); ip != nil {
			a.IP = ip
			if ip.To4() != nil {
				a.AddressType = IPv4
			} else {
				a.AddressType = IPv6
			}
		} else {
			a.DomainName = string(host)
		}
		a.Port = int(binary.BigEndian.Uint16(buf[length : length+2]))
	default:
		return common.NewError("invalid ATYP " + strconv.FormatInt(int64(a.AddressType), 10))
	}
	return nil
}

```

该函数接受一个指向地址结构体的指针参数a，并将其写入一个名为w的io.Writer中。函数的目的是将地址结构体中的地址信息写入到w中，并返回一个错误。

具体来说，函数首先将一个字节数组[]byte{byte(a.AddressType)}写入到w中，然后根据a.AddressType类型来决定写入的内容类型。如果是DomainName类型，则需要写入长地址[]byte{byte(len(a.DomainName))}，然后是实际的域名[]byte{byte(a.DomainName))。如果是IPv4或IPv6类型，则需要将IPv4或IPv6地址的二进制表示法写入到w中。如果写入过程中出现错误，则返回一个错误。最后，函数将一个IPv4地址类型的字段port的值写入到w中，这个字段是用于在写入数据时指定写入的端口号的。


```go
func (a *Address) WriteTo(w io.Writer) error {
	_, err := w.Write([]byte{byte(a.AddressType)})
	if err != nil {
		return err
	}
	switch a.AddressType {
	case DomainName:
		w.Write([]byte{byte(len(a.DomainName))})
		_, err = w.Write([]byte(a.DomainName))
	case IPv4:
		_, err = w.Write(a.IP.To4())
	case IPv6:
		_, err = w.Write(a.IP.To16())
	default:
		return common.NewError("invalid ATYP " + strconv.FormatInt(int64(a.AddressType), 10))
	}
	if err != nil {
		return err
	}
	port := [2]byte{}
	binary.BigEndian.PutUint16(port[:], uint16(a.Port))
	_, err = w.Write(port[:])
	return err
}

```

# `tunnel/tunnel.go`

这段代码定义了一个名为tunnel的包，其中包含了一些用于创建和管理网络连接的函数和类型。

首先，它导入了net包和io包，因为它们在创建和管理网络连接时需要使用。然后，它定义了一个名为Conn的接口类型，它包含一个net.Conn类型和一个Metadata类型，其中Metadata类型表示连接的元数据。

接着，代码定义了一个名为tunnel包的函数名为"createGDP simulator"。这个函数使用一个名为tunnel的上下文来创建一个GDP模拟器实例，并返回该实例的NetConn类型和Metadata类型。

然后，代码定义了一个名为tunnel的函数名为"createTCP simulator"。这个函数使用一个名为tcp的上下文来创建一个TCP模拟器实例，并返回该实例的NetConn类型和Metadata类型。

最后，代码定义了一个名为tunnel的函数名为"run Simulator"。这个函数使用一个名为run的函数来运行GDP或TCP模拟器，并传递一个Context对象作为参数。如果模拟器成功建立连接并成功地运行，则代码将返回模拟器的NetConn类型和Metadata类型。


```go
package tunnel

import (
	"context"
	"io"
	"net"

	"github.com/p4gefau1t/trojan-go/common"
)

// Conn is the TCP connection in the tunnel
type Conn interface {
	net.Conn
	Metadata() *Metadata
}

```

该代码定义了三个接口类型：PacketConn、ConnDialer 和 PacketDialer。它们都表示隧道中的数据传输。

PacketConn接口定义了两个函数：WriteWithMetadata 和 ReadWithMetadata。这两个函数都接收一个字节数组和一个元数据结构作为输入，并返回数据传输的数量和错误。这些函数可以用来在隧道中传输数据，并支持元数据结构以增加传输的数据类型。

ConnDialer接口定义了一个函数 DialConn，它接收一个目标地址和一个隧道作为输入参数，并返回一个连接对象和一个错误。这个函数可以用来从隧道中建立 TCP 连接。

PacketDialer接口定义了一个函数 DialPacket，它接收一个隧道作为输入参数，并返回一个数据传输对象和一个错误。这个函数可以用来在隧道中建立 UDP 连接。


```go
// PacketConn is the UDP packet stream in the tunnel
type PacketConn interface {
	net.PacketConn
	WriteWithMetadata([]byte, *Metadata) (int, error)
	ReadWithMetadata([]byte) (int, *Metadata, error)
}

// ConnDialer creates TCP connections from the tunnel
type ConnDialer interface {
	DialConn(*Address, Tunnel) (Conn, error)
}

// PacketDialer creates UDP packet stream from the tunnel
type PacketDialer interface {
	DialPacket(Tunnel) (PacketConn, error)
}

```

这段代码定义了三个接口：ConnListener、PacketListener和Dialer，以及一个没有实现接口的类型——Tunnel。

这些接口都表示了可以处理不同类型的网络连接。

具体来说，ConnListener和PacketListener都处理TCP和UDP数据流，而Dialer则处理网络连接和数据流。

需要注意的是，这些接口都没有实现具体的实现，因此它们的实现细节可以因具体应用而异。

例如，可以使用这些接口的不同组合来创建一个TCP或UDP网络连接，或通过一个TCP或UDP隧道来建立两个网络之间的通信。


```go
// ConnListener accept TCP connections
type ConnListener interface {
	AcceptConn(Tunnel) (Conn, error)
}

// PacketListener accept UDP packet stream
// We don't have any tunnel based on packet streams, so AcceptPacket will always receive a real PacketConn
type PacketListener interface {
	AcceptPacket(Tunnel) (PacketConn, error)
}

// Dialer can dial to original server with a tunnel
type Dialer interface {
	ConnDialer
	PacketDialer
}

```

这段代码定义了三个接口：Listener、Client 和 Server，以及一个接口 ListenerCanAcceptTCPAndUDPStreamsFromTunnel。这些接口表示了网络中的不同组件，具有不同的功能和特点。

以下是每个接口的功能解释：

1. Listener

Listener接口表示一个监听 TCP 和 UDP 流道的组件。它包含两个方法：

* ConvListen: 接受一个 TCP 或 UDP  stream。
* AddConvListener: 将一个 ConvListener 添加到已有的 Listener 实例中。

1. Client

Client接口表示一个基于 stream 连接的隧道客户端组件。它包含两个方法：

* Dialer: 建立与服务器的连接。
* Close: 关闭与服务器的连接。

1. Server

Server接口表示一个基于 stream 连接的隧道服务器组件。它包含三个方法：

* Listen: 监听一个或多个 TCP 或 UDP stream。
* AddServerListener: 将一个 ServerListener 添加到已有的 Listener 实例中。
* Close: 关闭与客户端的连接。

ListenerCanAcceptTCPAndUDPStreamsFromTunnel 是用于接受来自隧道中 TCP 和 UDP stream 的组件。


```go
// Listener can accept TCP and UDP streams from a tunnel
type Listener interface {
	ConnListener
	PacketListener
}

// Client is the tunnel client based on stream connections
type Client interface {
	Dialer
	io.Closer
}

// Server is the tunnel server based on stream connections
type Server interface {
	Listener
	io.Closer
}

```

这段代码定义了一个名为Tunnel的接口类型，用于创建隧道。该接口类型包含两个方法：NewClient和NewServer。通过这些方法，可以创建并注册隧道，同时假设下层隧道了解上层隧道的运作，并且下层隧道可以透明地与上层隧道通信。

接着，定义了一个名为tunnels的Map，用于存储所有注册的隧道。最后，通过两个注册函数，注册了两个隧道，并将它们存储到tunnelsMap中。


```go
// Tunnel describes a tunnel, allowing creating a tunnel from another tunnel
// We assume that the lower tunnels know exatly how upper tunnels work, and lower tunnels is transparent for the upper tunnels
type Tunnel interface {
	Name() string
	NewClient(context.Context, Client) (Client, error)
	NewServer(context.Context, Server) (Server, error)
}

var tunnels = make(map[string]Tunnel)

// RegisterTunnel register a tunnel by tunnel name
func RegisterTunnel(name string, tunnel Tunnel) {
	tunnels[name] = tunnel
}

```

这段代码定义了一个名为 `GetTunnel` 的函数，它接收一个字符串参数 `name`。函数的作用是查找一个给定名称的隧道，如果找到了隧道，则返回该隧道，否则返回一个错误。

函数首先检查数据库中是否有该名称的隧道，如果有，则直接返回该隧道，并不会创建新的隧道。如果没有找到该名称的隧道，函数会返回一个名为 `common.NewError` 的错误。

具体实现可以分为以下几个步骤：

1. 定义一个名为 `tunnels` 的数组，用于存储各种隧道信息。
2. 定义一个名为 `name` 的字符串变量，用于指定要查找的隧道名称。
3. 使用条件语句（if语句）检查给定名称的隧道是否存在于数组中。
4. 如果存在，则返回该隧道，否则创建新的隧道并返回。
5. 函数返回一个 `Tunnel` 类型的变量，表示成功查找到的隧道，也可以使用 `nil` 表示找不到隧道的情况。
6. 在函数内部，使用 `common.NewError` 函数创建一个错误，该错误类型为 `error` 且包含一个字符串 `"unknown tunnel name"`，以便在函数返回时进行输出。


```go
func GetTunnel(name string) (Tunnel, error) {
	if t, ok := tunnels[name]; ok {
		return t, nil
	}
	return nil, common.NewError("unknown tunnel name " + name)
}

```

# `tunnel/adapter/config.go`

这段代码定义了一个名为“adapter”的包，其中包含一个名为“Config”的结构体类型。该类型包含两个字段，一个是“local_addr”类型的字符串，另一个是“local_port”类型的整数。

此外，该代码还定义了一个名为“init”的函数，该函数在函数内部创建了一个名为“adapter”的配置创建器，该函数将“Config”类型作为参数传递给其内部函数。

具体来说，上述代码将在运行时创建一个名为“adapter”的配置创建器实例，该实例可以在以后的使用中用来设置或获取本地服务器的主机名和端口号。


```go
package adapter

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
	LocalHost string `json:"local_addr" yaml:"local-addr"`
	LocalPort int    `json:"local_port" yaml:"local-port"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return new(Config)
	})
}

```

# `tunnel/adapter/server.go`

这段代码定义了一个名为 "adapter" 的包。它导入了以下的外部库：

- "context" 库：用于处理与上下文相关的操作。
- "net" 库：用于网络操作。
- "sync" 库：用于原语类型操作。
- "github.com/p4gefau1t/trojan-go" 包：用于 Trojan 代理器的统一封装。
- "github.com/p4gefau1t/trojan-go/config" 包：用于配置项的设置。
- "github.com/p4gefau1t/trojan-go/log" 包：用于日志的输出。
- "github.com/p4gefau1t/trojan-go/tunnel" 包：用于各种网络隧道技术的封装。
- "github.com/p4gefau1t/trojan-go/tunnel/freedom" 包：用于解析隧道的 Freedom 字段。
- "github.com/p4gefau1t/trojan-go/tunnel/http" 包：用于与 HTTP 隧道建立连接。
- "github.com/p4gefau1t/trojan-go/tunnel/socks" 包：用于与 SOCKS 隧道建立连接。

然后，它定义了一些常量，包括一些用于设置代理和 tunnels 的选项。

最后，它定义了一个名为 "TAdapter" 的类型，它实现了 "Adapter" 接口。它使用了一个内部 "Adapter" 类型的 "tunnel" 字段，用于存储当前正在使用的网络隧道。

这个 package 主要是用来提供 Trojan 代理器所需的基本功能。通过提供一些常见的网络隧道技术，使得开发人员可以更加方便地构建代理器，从而实现监控、日志、流量控制等功能。


```go
package adapter

import (
	"context"
	"net"
	"sync"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/log"
	"github.com/p4gefau1t/trojan-go/tunnel"
	"github.com/p4gefau1t/trojan-go/tunnel/freedom"
	"github.com/p4gefau1t/trojan-go/tunnel/http"
	"github.com/p4gefau1t/trojan-go/tunnel/socks"
)

```

这段代码定义了一个名为Server的Server struct类型，它表示一个代理服务器，用于代理客户端的连接。这个服务器可以监听TCP和UDP协议的连接，并支持SOCKS5协议的连接。

下面是具体的实现细节：

1. 该Server结构体定义了一个名为tcpListener的tcp套接字监听器，一个名为udpListener的udp套接字监听器，一个名为socksConn的通道，一个名为httpConn的通道，以及一个名为socksLock的互斥锁，一个名为nextSocks的布尔值，一个名为ctx的上下文变量和一个名为cancel的取消函数。

2. 在acceptConnLoop()函数中，该Server结构体接受一个TCP连接并尝试获取数据。如果连接出现错误，函数将退出并尝试取消任何正在运行的连接。如果连接成功，函数将执行以下操作：

  a. 创建一个名为rewindConn的新的通道连接，并设置其缓冲区大小为16个字节。

  b. 从新连接中读取数据，并将其存储在名为buf的3个字节中。

  c. 尝试重新连接，如果出现错误，将记录到日志中。

  d. 如果连接成功，将连接重新加锁，并检查所读数据是否为SOCKS5格式的数据。如果是，将连接到客户端的通道赋值给rewrittenConn，然后通知nextSocks为true。如果数据不是SOCKS5格式的数据，将通知nextSocks为false。

3. 在 acceptConnLoop()函数中，还定义了一个名为tcpListen()的函数，用于启动服务器监听TCP协议的连接。

4. 在 main()函数中，创建了一个Server实例并设置其默认连接为匿名。该Server将监听8080端口，等待来自客户端的连接。每当连接建立时，该Server将尝试从客户端获取数据。

总结起来，该代码实现了一个代理服务器，可以监听TCP和UDP协议的连接，并支持SOCKS5协议的连接。


```go
type Server struct {
	tcpListener net.Listener
	udpListener net.PacketConn
	socksConn   chan tunnel.Conn
	httpConn    chan tunnel.Conn
	socksLock   sync.RWMutex
	nextSocks   bool
	ctx         context.Context
	cancel      context.CancelFunc
}

func (s *Server) acceptConnLoop() {
	for {
		conn, err := s.tcpListener.Accept()
		if err != nil {
			select {
			case <-s.ctx.Done():
				log.Debug("exiting")
				return
			default:
				continue
			}
		}
		rewindConn := common.NewRewindConn(conn)
		rewindConn.SetBufferSize(16)
		buf := [3]byte{}
		_, err = rewindConn.Read(buf[:])
		rewindConn.Rewind()
		rewindConn.StopBuffering()
		if err != nil {
			log.Error(common.NewError("failed to detect proxy protocol type").Base(err))
			continue
		}
		s.socksLock.RLock()
		if buf[0] == 5 && s.nextSocks {
			s.socksLock.RUnlock()
			log.Debug("socks5 connection")
			s.socksConn <- &freedom.Conn{
				Conn: rewindConn,
			}
		} else {
			s.socksLock.RUnlock()
			log.Debug("http connection")
			s.httpConn <- &freedom.Conn{
				Conn: rewindConn,
			}
		}
	}
}

```

该函数名为`AcceptConn`，定义了一个名为`s`的`Server`类型的指针变量`s`，其作用是接受一个`overlay`类型的`tunnel.Tunnel`。

函数体中，首先检查`overlay`是否为`http.Tunnel`类型，如果是，则检查是否接收到了服务器`s`的`httpConn`，如果是，则返回该连接，否则返回一个`nil`。

如果不是`http.Tunnel`类型，也不是`socks.Tunnel`类型，那么会尝试获取一个`socks.Tunnel`实例，并将`nextSocks`设置为`true`，然后尝试获取服务器`s`的`socksConn`，如果是，则返回该连接，否则返回一个`nil`。

如果在以上两种情况中，任何一种情况下出现了错误，则会输出一个错误，并返回`nil`。


```go
func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
	if _, ok := overlay.(*http.Tunnel); ok {
		select {
		case conn := <-s.httpConn:
			return conn, nil
		case <-s.ctx.Done():
			return nil, common.NewError("adapter closed")
		}
	} else if _, ok := overlay.(*socks.Tunnel); ok {
		s.socksLock.Lock()
		s.nextSocks = true
		s.socksLock.Unlock()
		select {
		case conn := <-s.socksConn:
			return conn, nil
		case <-s.ctx.Done():
			return nil, common.NewError("adapter closed")
		}
	} else {
		panic("invalid overlay")
	}
}

```

此代码定义了一个名为Server的的服务器类，用于监听TCP和UDP流量。

func NewServer(ctx context.Context, config *config.Server) (*Server, error) {
	ctx, cancel := context.WithCancel(ctx)
	addr := tunnel.NewAddressFromHostPort(config.LocalHost, config.LocalPort, cancel)

	tcpListener, err := net.Listen("tcp", addr.String())
	if err != nil {
		return nil, common.NewError("adapter failed to create tcp listener").Base(err)
	}
	udpListener, err := net.ListenPacket("udp", addr.String())
	if err != nil {
		return nil, common.NewError("adapter failed to create tcp listener").Base(err)
	}

	server := &Server{
		tcpListener: tcpListener,
		udpListener: udpListener,
		socksConn:   make(chan tunnel.Conn, 32),
		httpConn:    make(chan tunnel.Conn, 32),
		ctx:         ctx,
		cancel:      cancel,
	}

	go server.acceptConnLoop()

	return server, nil
}


```go
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
	return &freedom.PacketConn{
		UDPConn: s.udpListener.(*net.UDPConn),
	}, nil
}

func (s *Server) Close() error {
	s.cancel()
	s.tcpListener.Close()
	return s.udpListener.Close()
}

func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
	cfg := config.FromContext(ctx, Name).(*Config)
	var cancel context.CancelFunc
	ctx, cancel = context.WithCancel(ctx)

	addr := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
	tcpListener, err := net.Listen("tcp", addr.String())
	if err != nil {
		cancel()
		return nil, common.NewError("adapter failed to create tcp listener").Base(err)
	}
	udpListener, err := net.ListenPacket("udp", addr.String())
	if err != nil {
		cancel()
		return nil, common.NewError("adapter failed to create tcp listener").Base(err)
	}
	server := &Server{
		tcpListener: tcpListener,
		udpListener: udpListener,
		socksConn:   make(chan tunnel.Conn, 32),
		httpConn:    make(chan tunnel.Conn, 32),
		ctx:         ctx,
		cancel:      cancel,
	}
	log.Info("adapter listening on tcp/udp:", addr)
	go server.acceptConnLoop()
	return server, nil
}

```

# `tunnel/adapter/tunnel.go`

这段代码定义了一个名为"adapter"的包，其中包含一个名为"Tunnel"的接口类型。

具体来说，该包使用了一个名为"github.com/p4gefau1t/trojan-go/tunnel"的第三方库，通过该库创建了一个匿名类型的"Tunnel"结构体。

该结构体有一个名为"Name"的成员函数，返回该匿名类型的名称，即"adapter"。

此外，该包的说明文件中提到了该接口类型的默认实现，但没有定义任何名为"Tunnel"的实例变量。


```go
package adapter

import (
	"context"

	"github.com/p4gefau1t/trojan-go/tunnel"
)

const Name = "ADAPTER"

type Tunnel struct{}

func (t *Tunnel) Name() string {
	return Name
}

```

这两段代码定义了两个名为`NewClient`和`NewServer`的函数，用于创建新的隧道客户端和服务器。这两个函数接受一个名为`Tunnel`的接口类型的参数，并在函数内部使用`tunnel.Client`和`tunnel.Server`类型来返回新的客户端和服务器实例。

这两个函数的实现比较简单，只是返回一个`tunnel.Client`和一个`tunnel.Server`类型的变量，并没有对传入的参数进行更多的处理。不过，在函数内部，通过调用`panic("not supported")`来输出一条错误信息，表明目前该函数不受支持。

最后，通过调用`tunnel.RegisterTunnel`来注册该隧道的自定义设置，以便在需要时可以创建使用该隧道的客户端或服务器。


```go
func (t *Tunnel) NewClient(ctx context.Context, client tunnel.Client) (tunnel.Client, error) {
	panic("not supported")
}

func (t *Tunnel) NewServer(ctx context.Context, server tunnel.Server) (tunnel.Server, error) {
	return NewServer(ctx, server)
}

func init() {
	tunnel.RegisterTunnel(Name, &Tunnel{})
}

```

# `tunnel/dokodemo/config.go`

这段代码定义了一个名为 `Config` 的结构体类型，该类型包含以下字段：

* `LocalHost`：本地主机，使用 JSON 序列化，并使用 `json` 标签和 `yaml` 标签进行配置。
* `LocalPort`：本地端口，使用 JSON 序列化，并使用 `json` 标签和 `yaml` 标签进行配置。
* `TargetHost`：目标主机，使用 JSON 序列化，并使用 `json` 标签和 `yaml` 标签进行配置。
* `TargetPort`：目标端口，使用 JSON 序列化，并使用 `json` 标签和 `yaml` 标签进行配置。
* `UDPTimeout`：UDP 超时时间，使用 JSON 序列化，并使用 `json` 标签和 `yaml` 标签进行配置。

该代码的作用是注册一个名为 `ConfigCreator` 的函数，该函数将 `Config` 类型作为参数，并返回一个指向 `Config` 类型的引用。这个注册的函数会在程序运行时被调用，并在调用之后将 `Config` 类型的一些字段设置为默认值。

具体来说，该代码首先定义了一个 `Config` 类型，其中包含 `local_addr`、`local_port`、`target_addr` 和 `target_port` 这四个字段，分别对应于本地主机、本地端口、目标主机和目标端口的配置。然后，该类型还定义了一个名为 `udp_timeout` 的字段，用于指定 UDP 超时时间。

接着，该代码的 `init` 函数将 `Config` 类型的一些字段设置为默认值，即：

* `UDPTimeout`：设置为 60 秒。

最后，该代码注册了一个名为 `ConfigCreator` 的函数，该函数接收一个 `Config` 类型的参数，并返回一个指向 `Config` 类型的引用。该函数会在程序运行时被调用，并在调用之后将 `Config` 类型的一些字段设置为默认值。


```go
package dokodemo

import "github.com/p4gefau1t/trojan-go/config"

type Config struct {
	LocalHost  string `json:"local_addr" yaml:"local-addr"`
	LocalPort  int    `json:"local_port" yaml:"local-port"`
	TargetHost string `json:"target_addr" yaml:"target-addr"`
	TargetPort int    `json:"target_port" yaml:"target-port"`
	UDPTimeout int    `json:"udp_timeout" yaml:"udp-timeout"`
}

func init() {
	config.RegisterConfigCreator(Name, func() interface{} {
		return &Config{
			UDPTimeout: 60,
		}
	})
}

```

# `tunnel/dokodemo/conn.go`

这段代码定义了一个名为`dokodemo`的包，其中包括了以下组件：

1. 导入`net`和`github.com/p4gefau1t/trojan-go`两个包。

2. 定义了一个名为`MaxPacketSize`的常量，其值为`1024 * 8`，表示可以接收的最大数据包大小。

3. 定义了一个名为`Conn`的`tunnel.Conn`类型构造函数，其中包含了以下字段：

	- `net.Conn`类型，表示网络连接的`net`包提供的`Conn`字段。
	- `src`字段，是一个指向`tunnel.Address`类型的指针，表示数据源的地址。
	- `targetMetadata`字段，是一个指向`tunnel.Metadata`类型的指针，表示数据源的元数据。

4. 没有定义任何函数或变量，但使用了以下导入：

	- `"context"`
	- `"net"`
	- `"github.com/p4gefau1t/trojan-go"`

根据以上分析，这段代码定义了一个`tunnel.Conn`类型的变量`conn`，用于建立一个网络连接到远程主机，并从指定数据源接收数据。数据源的地址和元数据存储在`src`和`targetMetadata`字段中。最大数据包大小可以通过`MaxPacketSize`常量来设置。


```go
package dokodemo

import (
	"context"
	"net"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/tunnel"
)

const MaxPacketSize = 1024 * 8

type Conn struct {
	net.Conn
	src            *tunnel.Address
	targetMetadata *tunnel.Metadata
}

```

这段代码定义了一个名为 PacketConn 的 struct 类型，该类型表示用于接收数据包信息的数据连接。

函数 Metadata() *tunnel.Metadata 返回一个指向隧道元数据的指针，该数据包还没有被完全接收。

该函数接收一个名为 c 的指针，该指针代表一个数据连接，然后返回该连接的目标元数据。

函数 PacketConn 接收一个包含数据包信息的数据包处理器，然后将数据包信息发送到该数据包处理器。

PacketConn 结构体包含以下字段：

* net.PacketConn：表示该连接的包装网络类型。
* metadata：该连接接收到的数据包的元数据。
* input：该连接的输入流。
* output：该连接的输出流。
* src：该连接的源地址。
* ctx：该连接所属的上下文。
* cancel：该连接使用的取消函数。

该函数创建了一个 PacketConn 类型的实例，然后使用 c 的指针将数据包信息发送到该数据处理器。


```go
func (c *Conn) Metadata() *tunnel.Metadata {
	return c.targetMetadata
}

// PacketConn receive packet info from the packet dispatcher
type PacketConn struct {
	net.PacketConn
	metadata *tunnel.Metadata
	input    chan []byte
	output   chan []byte
	src      net.Addr
	ctx      context.Context
	cancel   context.CancelFunc
}

```

此代码定义了两个函数，一个是关闭套接字并返回错误，另一个是读取数据并返回读取的UDP套接字、网络地址和错误。

函数1：`func (c *PacketConn) Close() error {
	c.cancel()
	// 不会关闭底层UDP套接字
	return nil
}`

此函数的目的是使套接字关闭，但不会关闭底层UDP套接字。函数返回一个错误，但通常情况下不会返回，因为关闭套接字不会导致任何I/O错误。

函数2：`func (c *PacketConn) ReadFrom(p []byte) (int, net.Addr, error)`

此函数的目的是从UDP套接字中读取数据并返回读取的UDP套接字、网络地址和错误。函数接收一个字节切片和一个网络地址，并返回读取的UDP套接字、网络地址和错误。

函数3：`func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error)`

此函数的目的是从UDP套接字中写入数据并返回写入的UDP套接字、错误。函数接收一个字节切片和一个网络地址，并返回写入的UDP套接字、错误和网络地址。函数通过创建一个新套接字并将套接字连接到底层UDP套接字来发送数据。


```go
func (c *PacketConn) Close() error {
	c.cancel()
	// don't close the underlying udp socket
	return nil
}

func (c *PacketConn) ReadFrom(p []byte) (int, net.Addr, error) {
	return c.ReadWithMetadata(p)
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error) {
	address, err := tunnel.NewAddressFromAddr("udp", addr.String())
	if err != nil {
		return 0, err
	}
	return c.WriteWithMetadata(p, &tunnel.Metadata{
		Address: address,
	})
}

```

这两函数函数是定义在 packetconn 类型的 P 包装器函数。

第一个函数 function(c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) 接收一个字节切片 p 作为参数，然后从 c 的输入 channel 接收数据并复制到 p 中。函数返回从 c 的输入 channel 接收的数据个数，c 的输出 channel 中的元数据，或者错误。

第二个函数 function(c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) 接收一个字节切片 p 和一个元数据结构体 m 作为参数，然后尝试将 p 写入 c 的输出 channel。函数返回从 c 的输出 channel 写入数据个数，或者错误。


```go
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
	select {
	case payload := <-c.input:
		n := copy(p, payload)
		return n, c.metadata, nil
	case <-c.ctx.Done():
		return 0, nil, common.NewError("dokodemo packet conn closed")
	}
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
	select {
	case c.output <- p:
		return len(p), nil
	case <-c.ctx.Done():
		return 0, common.NewError("dokodemo packet conn failed to write")
	}
}

```

# `tunnel/dokodemo/dokodemo_test.go`

This is a Go program that performs aUDP Echo Test. It sends two packets to a specified port, listens for incoming packets, and compares the received packets to the expected ones. If the received packets match the expected packets, the program prints a success message. If the received packets do not match the expected packets, the program prints an error message and fails the test.

The program uses the following components:

* packet1: This is the first packet sent to the specified port. It is a simple UDP packet with a single byte of data.
* packet2: This is the second packet sent to the specified port. It is also a simple UDP packet with a single byte of data, but it includes some metadata.
* net: The `net` package provides a secure way to listen for incoming packets on a networked device.
* socket: The `socket` package provides a way to create and use a network socket in Go.
* tcp: The `tcp` package provides a type of `socket` that is used for Transport Layer Security (TLS) and Transport Layer Security (TLS) 1.1 protocols.
* uuid: The `uuid` package provides a way to generate universally unique identifiers (UUIDs).
* package: The `package` package provides a way to import and use packages in Go.


```go
package dokodemo

import (
	"context"
	"fmt"
	"net"
	"sync"
	"testing"

	"github.com/p4gefau1t/trojan-go/common"
	"github.com/p4gefau1t/trojan-go/config"
	"github.com/p4gefau1t/trojan-go/test/util"
)

func TestDokodemo(t *testing.T) {
	cfg := &Config{
		LocalHost:  "127.0.0.1",
		LocalPort:  common.PickPort("tcp", "127.0.0.1"),
		TargetHost: "127.0.0.1",
		TargetPort: common.PickPort("tcp", "127.0.0.1"),
		UDPTimeout: 30,
	}
	ctx := config.WithConfig(context.Background(), Name, cfg)
	s, err := NewServer(ctx, nil)
	common.Must(err)
	conn1, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", cfg.LocalPort))
	common.Must(err)
	conn2, err := s.AcceptConn(nil)
	common.Must(err)
	if !util.CheckConn(conn1, conn2) {
		t.Fail()
	}
	conn1.Close()
	conn2.Close()

	wg := sync.WaitGroup{}
	wg.Add(1)

	packet1, err := net.ListenPacket("udp", "")
	common.Must(err)
	common.Must2(packet1.(*net.UDPConn).WriteToUDP([]byte("hello1"), &net.UDPAddr{
		IP:   net.ParseIP("127.0.0.1"),
		Port: cfg.LocalPort,
	}))
	packet2, err := s.AcceptPacket(nil)
	common.Must(err)
	buf := [100]byte{}
	n, m, err := packet2.ReadWithMetadata(buf[:])
	common.Must(err)
	if m.Address.Port != cfg.TargetPort {
		t.Fail()
	}
	if string(buf[:n]) != "hello1" {
		t.Fail()
	}
	fmt.Println(n, m, string(buf[:n]))

	if !util.CheckPacket(packet1, packet2) {
		t.Fail()
	}

	packet3, err := net.ListenPacket("udp", "")
	common.Must(err)
	common.Must2(packet3.(*net.UDPConn).WriteToUDP([]byte("hello2"), &net.UDPAddr{
		IP:   net.ParseIP("127.0.0.1"),
		Port: cfg.LocalPort,
	}))
	packet4, err := s.AcceptPacket(nil)
	common.Must(err)
	n, m, err = packet4.ReadWithMetadata(buf[:])
	common.Must(err)
	if m.Address.Port != cfg.TargetPort {
		t.Fail()
	}
	if string(buf[:n]) != "hello2" {
		t.Fail()
	}
	fmt.Println(n, m, string(buf[:n]))

	wg = sync.WaitGroup{}
	wg.Add(2)
	go func() {
		if !util.CheckPacket(packet3, packet4) {
			t.Fail()
		}
		wg.Done()
	}()
	go func() {
		if !util.CheckPacket(packet1, packet2) {
			t.Fail()
		}
		wg.Done()
	}()
	wg.Wait()
	s.Close()
}

```