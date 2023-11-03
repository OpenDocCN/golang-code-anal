# v2ray-core源码解析 56

# `testing/scenarios/common_coverage.go`

这段代码是一个名为"BuildV2Ray"的函数，主要作用是编译并运行一个名为"buildcoverage"的二进制文件。"buildcoverage"文件的目的是为了生成测试用例。

具体来说，这段代码实现以下步骤：

1. 首先，函数创建了一个名为"scenarios"的包。
2. 在包中定义了一个名为"BuildV2Ray"的函数。
3. 在"BuildV2Ray"函数中，使用"genTestBinaryPath"函数生成一个测试二进制文件的路径。
4. 接着，函数检查测试二进制文件是否存在。如果不存在，函数返回一个 nil 错误。
5. 然后，创建一个执行命令的 "cmd" 变量，包含 "go" 命令、"test" 命令和一些参数，如 "coverage"、"coveragemain" 和 "v2ray.com/core/" 等。
6. 将 "testBinaryPath" 和 "GetSourcePath" 参数传递给 "cmd" 变量。
7. 最后，函数运行 "cmd" 生成的二进制文件并返回结果。


```go
// +build coverage

package scenarios

import (
	"bytes"
	"os"
	"os/exec"

	"v2ray.com/core/common/uuid"
)

func BuildV2Ray() error {
	genTestBinaryPath()
	if _, err := os.Stat(testBinaryPath); err == nil {
		return nil
	}

	cmd := exec.Command("go", "test", "-tags", "coverage coveragemain", "-coverpkg", "v2ray.com/core/...", "-c", "-o", testBinaryPath, GetSourcePath())
	return cmd.Run()
}

```

这段代码定义了一个名为 `RunV2RayProtobuf` 的函数，其接收一个字节数组 `config` 作为参数，并返回一个执行 `exec.Cmd` 的结果。

函数首先通过 `genTestBinaryPath` 函数生成一个测试二进制文件，并将该文件路径存储在名为 `config_gen` 的变量中。

接下来，函数创建了一个目录，并将该目录命名为 `cov_dir`，用于存储校验和。然后，函数生成一个随机的校验和 `randomID`，并将其存储在名为 `profile` 的变量中。

接着，函数使用 `exec.Command` 函数运行名为 `testBinaryPath` 的测试二进制文件，并传递以下参数：

- `-config=stdin`：从标准输入读取 `config` 字节数组。
- `-format=pb`：将测试二进制文件格式化为 protobuf 格式。
- `-test.run`：运行测试二进制文件以产生测试报告。
- `-test.coverprofile`：运行测试二进货以生成覆盖报告。
- `profile`：设置测试二进分的输出目录为 `cov_dir`。
- `-test.outputdir`：设置测试报告和覆盖报告的输出目录为 `cov_dir`。

最后，函数返回一个执行 `exec.Cmd` 的结果，该结果包含前面生成的 `proc`，即将 `config` 字节数组通过 `proc.Stdin`、 `proc.Stderr` 和 `proc.Stdout` 传递给 `testBinaryPath` 函数。


```go
func RunV2RayProtobuf(config []byte) *exec.Cmd {
	genTestBinaryPath()

	covDir := os.Getenv("V2RAY_COV")
	os.MkdirAll(covDir, os.ModeDir)
	randomID := uuid.New()
	profile := randomID.String() + ".out"
	proc := exec.Command(testBinaryPath, "-config=stdin:", "-format=pb", "-test.run", "TestRunMainForCoverage", "-test.coverprofile", profile, "-test.outputdir", covDir)
	proc.Stdin = bytes.NewBuffer(config)
	proc.Stderr = os.Stderr
	proc.Stdout = os.Stdout

	return proc
}

```

# `testing/scenarios/common_regular.go`

这段代码是一个 Go 语言编写的函数，名为 `BuildV2Ray`，旨在执行以下操作：

1. 生成测试二进制文件。如果该文件已存在，则不执行任何操作；如果已不存在，将构建 V2Ray 并将其保存到指定路径。
2. 创建一个名为 `testBinaryPath` 的文件，如果该文件不存在，从当前工作目录开始创建一个新文件。
3. 打印 "Building V2Ray into path (%s)" 的消息，其中 `%s` 是 `testBinaryPath` 变量的引用，即测试二进制文件的保存路径。
4. 使用 `go build` 命令构建 Go 应用程序，使用 `-o` 选项指定生成的二进制文件名，即 `testBinaryPath` 的副本。同时使用 `-sourcepath` 选项指定应用程序的源代码目录。
5. 打印 "Building V2Ray into path (%s)\n" 的消息，其中 `%s` 是 `testBinaryPath` 变量的引用，即测试二进制文件的保存路径。


```go
// +build !coverage

package scenarios

import (
	"bytes"
	"fmt"
	"os"
	"os/exec"
)

func BuildV2Ray() error {
	genTestBinaryPath()
	if _, err := os.Stat(testBinaryPath); err == nil {
		return nil
	}

	fmt.Printf("Building V2Ray into path (%s)\n", testBinaryPath)
	cmd := exec.Command("go", "build", "-o="+testBinaryPath, GetSourcePath())
	return cmd.Run()
}

```

这段代码定义了一个名为"RunV2RayProtobuf"的函数，其作用是运行一个使用Go标准库编写的名为"test"二进制的实验。

函数接收一个字节数组表示的配置参数，并将其传递给"testBinaryPath"函数。

函数创建一个执行二进制文件的命令，通过 standardinput 参数读取传入的配置参数，通过 stdout 参数将结果输出到标准输出，通过 stderr 参数将错误输出到标准错误。

函数返回一个执行二进制文件的命令对象（exec.Cmd）。


```go
func RunV2RayProtobuf(config []byte) *exec.Cmd {
	genTestBinaryPath()
	proc := exec.Command(testBinaryPath, "-config=stdin:", "-format=pb")
	proc.Stdin = bytes.NewBuffer(config)
	proc.Stderr = os.Stderr
	proc.Stdout = os.Stdout

	return proc
}

```

# `testing/scenarios/dns_test.go`

这段代码定义了一个名为 "scenarios" 的包。它导入了以下外置件：

- "fmt"：用于格式化 I/O 的包。
- "testing"：用于测试的包。
- "time"：用于时间处理的包。

它导入了以下内部实现：

- "xproxy"：用于代理的库。
- "v2ray.com/core": 用于 V2Ray 代理服务的库。
- "v2ray.com/core/app/dns": 用于解析 DNS 查询的库。
- "v2ray.com/core/app/proxyman": 用于管理代理服务的库。
- "v2ray.com/core/app/router": 用于路由代理的库。
- "v2ray.com/core/common": 用于通用功能的开源库。
- "v2ray.com/core/common/net": 用于网络功能的开源库。
- "v2ray.com/core/common/serial": 用于串口通信的库。
- "v2ray.com/core/proxy/blackhole": 用于黑洞代理的库。
- "v2ray.com/core/proxy/freedom": 用于自由的代理服务端库。
- "v2ray.com/core/proxy/socks": 用于 Socket 代理的库。
- "v2ray.com/core/testing/servers/tcp": 用于测试服务器 TCP 代理服务的库。

这段代码的目的是提供在 Go 语言中编写测试的示例。它演示了如何使用 Go 语言编写测试来验证 V2Ray 代理服务的功能。


```go
package scenarios

import (
	"fmt"
	"testing"
	"time"

	xproxy "golang.org/x/net/proxy"
	"v2ray.com/core"
	"v2ray.com/core/app/dns"
	"v2ray.com/core/app/proxyman"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/blackhole"
	"v2ray.com/core/proxy/freedom"
	"v2ray.com/core/proxy/socks"
	"v2ray.com/core/testing/servers/tcp"
)

```

This appears to be a Go program that sets up a series of network connections to test email delivery.

The program sets up a server to listen on a specified port (the "serverPort" parameter), and starts listening for incoming connections. When a connection is accepted, the program performs a connection to a website with the specified "serverPort".

The program also sets up a number of SOCKS5 proxy connections to allow for encrypted communication with the server. These connections are configured to use the "blackhole.Config" settings, which specify that authentication should not be required and that the DNS server for the proxy should be used if one is available.

Finally, the program sets up a connection to a website using the "test.ai" bot to verify that the email delivery system is functioning properly.

I hope this helps! Let me know if you have any questions.


```go
func TestResolveIP(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dns.Config{
				Hosts: map[string]*net.IPOrDomain{
					"google.com": net.NewIPOrDomain(dest.Address),
				},
			}),
			serial.ToTypedMessage(&router.Config{
				DomainStrategy: router.Config_IpIfNonMatch,
				Rule: []*router.RoutingRule{
					{
						Cidr: []*router.CIDR{
							{
								Ip:     []byte{127, 0, 0, 0},
								Prefix: 8,
							},
						},
						TargetTag: &router.RoutingRule_Tag{
							Tag: "direct",
						},
					},
				},
			}),
		},
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
					AuthType: socks.AuthType_NO_AUTH,
					Accounts: map[string]string{
						"Test Account": "Test Password",
					},
					Address:    net.NewIPOrDomain(net.LocalHostIP),
					UdpEnabled: false,
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&blackhole.Config{}),
			},
			{
				Tag: "direct",
				ProxySettings: serial.ToTypedMessage(&freedom.Config{
					DomainStrategy: freedom.Config_USE_IP,
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		noAuthDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, serverPort).NetAddr(), nil, xproxy.Direct)
		common.Must(err)
		conn, err := noAuthDialer.Dial("tcp", fmt.Sprintf("google.com:%d", dest.Port))
		common.Must(err)
		defer conn.Close()

		if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
			t.Error(err)
		}
	}
}

```

# `testing/scenarios/dokodemo_test.go`

这段代码定义了一个名为 "scenarios" 的包。它从 "scores" 包中导入了一些函数和类型。

然后，它导入了 "testing" 包中的 "ErrorGroup" 类型。

接下来，它导入了 "time" 包中的 "time" 类型。

然后，它导入了 "v2ray.com/core" 包中的 "log" 类型。

接着，它导入了 "v2ray.com/core/app/log" 类型。

然后，它导入了 "v2ray.com/core/app/proxyman" 类型。

接着，它导入了 "v2ray.com/core/common" 包中的 "log"、"net" 和 "protocol" 类型。

然后，它导入了 "v2ray.com/core/common/serial" 类型。

接着，它导入了 "v2ray.com/core/common/uuid" 类型。

然后，它导入了 "v2ray.com/core/proxy/dokodemo" 类型。

接着，它导入了 "v2ray.com/core/proxy/freedom" 类型。

然后，它导入了 "v2ray.com/core/proxy/vmess" 类型，从 "v2ray.com/core/proxy/vmess/inbound" 和 "v2ray.com/core/proxy/vmess/outbound" 中选择 "inbound"。

接下来，它导入了 "v2ray.com/core/proxy/vmess/inbound" 类型，从 "v2ray.com/core/proxy/vmess/inbound" 中选择 "outbound"。

然后，它导入了 "v2ray.com/core/testing/servers/tcp" 类型。

接着，它导入了 "v2ray.com/core/testing/servers/udp" 类型。

最后，它导入了若干个 "ErrorGroup" 类型的实例，并为每个实例的每个函数添加了一个 "Scenario" 名称，例如 "test_scenario01"、"test_scenario02" 等。


```go
package scenarios

import (
	"testing"
	"time"

	"golang.org/x/sync/errgroup"

	"v2ray.com/core"
	"v2ray.com/core/app/log"
	"v2ray.com/core/app/proxyman"
	"v2ray.com/core/common"
	clog "v2ray.com/core/common/log"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/common/uuid"
	"v2ray.com/core/proxy/dokodemo"
	"v2ray.com/core/proxy/freedom"
	"v2ray.com/core/proxy/vmess"
	"v2ray.com/core/proxy/vmess/inbound"
	"v2ray.com/core/proxy/vmess/outbound"
	"v2ray.com/core/testing/servers/tcp"
	"v2ray.com/core/testing/servers/udp"
)

```

This appears to be a Go program that sets up a testbed to connect to a remote server using the virtual private overlay protocol (VMess). The program creates a server that listens for incoming connections on a specified port, and a client that connects to the server and sends a request to a remote endpoint with a specified port number.

The server listens for incoming connections using the `net.NewIPOrDomain` function, which creates a new IP address or domain name and returns it. The server also sets up a proxy with a specified receiver endpoint, which will forward incoming connections to the specified receiver endpoint. The `dokodemo.Config` struct defines the connection settings for the client, including the network list and the `Protocol` and `User` settings.

The program initializes the server and client connections, and then iterates through all client ports up to the specified client port range, and sends a test connection request to the remote server using the `testTCPConn` function. If the connection is successful, the program prints a message indicating that the connection was successful.


```go
func TestDokodemoTCP(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	userID := protocol.NewID(uuid.New())
	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&log.Config{
				ErrorLogLevel: clog.Severity_Debug,
				ErrorLogType:  log.LogType_Console,
			}),
		},
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: userID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	clientPort := uint32(tcp.PickPort())
	clientPortRange := uint32(5)
	clientConfig := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&log.Config{
				ErrorLogLevel: clog.Severity_Debug,
				ErrorLogType:  log.LogType_Console,
			}),
		},
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: &net.PortRange{From: clientPort, To: clientPort + clientPortRange},
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(serverPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: userID.String(),
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig, clientConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	for port := clientPort; port <= clientPort+clientPortRange; port++ {
		if err := testTCPConn(net.Port(port), 1024, time.Second*2)(); err != nil {
			t.Error(err)
		}
	}
}

```

This is a Go program that initializes a WireMaker proxy server. The program creates a server for incoming connections and handles incoming client connections.

The program first sets up a proxy with a specified receiver, which will be the IP address of a server that will receive data from the client. The receiver is then configured to accept incoming connections with the specified port (1024 by default) and a timeout of 5 seconds.

The program then sets up a WireMaker client to connect to the server. This is followed by a loop that connects to the server using the specified IP address and port, and then sends data to the server. The client will timeout after 5 seconds if it does not receive a response.

The program also sets up a test that runs the client for a specified number of ports (up to 1023 by default).

Finally, the program initializes the server by creating a slice of server configurations, each with its own receiver and a list of outbound handlers. These configurations are then added to the errgroup, which will run the server. The program also sets up a function that runs the server and closes all servers after the initialization is complete.


```go
func TestDokodemoUDP(t *testing.T) {
	udpServer := udp.Server{
		MsgProcessor: xor,
	}
	dest, err := udpServer.Start()
	common.Must(err)
	defer udpServer.Close()

	userID := protocol.NewID(uuid.New())
	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: userID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	clientPort := uint32(tcp.PickPort())
	clientPortRange := uint32(5)
	clientConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: &net.PortRange{From: clientPort, To: clientPort + clientPortRange},
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_UDP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(serverPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: userID.String(),
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig, clientConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	var errg errgroup.Group
	for port := clientPort; port <= clientPort+clientPortRange; port++ {
		errg.Go(testUDPConn(net.Port(port), 1024, time.Second*5))
	}
	if err := errg.Wait(); err != nil {
		t.Error(err)
	}
}

```

# `testing/scenarios/feature_test.go`

这段代码定义了一个名为 "scenarios" 的包。它下面定义了一些函数和变量，以及一些常量。

下面是一些重要的函数和变量：

- 函数：
 - "package scenarios": 这个函数的作用是导入 "scenarios" 包。
 - "import (": 这个部分的作用是导入其他包。
 - "net/http": 导入 "net/http" 包。
 - "v2ray.com/core": 导入 "v2ray.com/core" 包。
 - "v2ray.com/core/dispatcher": 导入 "v2ray.com/core/app/dispatcher" 包。
 - "v2ray.com/core/app/log": 导入 "v2ray.com/core/app/log" 包。
 - "v2ray.com/core/app/proxyman": 导入 "v2ray.com/core/app/proxyman" 包。
 - "v2ray.com/core/app/proxyman/inbound": 导入 "v2ray.com/core/app/proxyman/inbound" 包。
 - "v2ray.com/core/app/proxyman/outbound": 导入 "v2ray.com/core/app/proxyman/outbound" 包。
 - "v2ray.com/core/app/router": 导入 "v2ray.com/core/app/router" 包。
 - "v2ray.com/core/common": 导入 "v2ray.com/core/common" 包。
 - "v2ray.com/core/common/net": 导入 "v2ray.com/core/common/net" 包。
 - "v2ray.com/core/common/protocol": 导入 "v2ray.com/core/common/protocol" 包。
 - "v2ray.com/core/common/serial": 导入 "v2ray.com/core/common/serial" 包。
 - "v2ray.com/core/common/uuid": 导入 "v2ray.com/core/common/uuid" 包。
 - "v2ray.com/core/proxy/blackhole": 导入 "v2ray.com/core/proxy/blackhole" 包。
 - "v2ray.com/core/proxy/dokodemo": 导入 "v2ray.com/core/proxy/dokodemo" 包。
 - "v2ray.com/core/proxy/freedom": 导入 "v2ray.com/core/proxy/freedom" 包。
 - "v2ray.com/core/proxy/vmess": 导入 "v2ray.com/core/proxy/vmess" 包。
 - "v2ray.com/core/proxy/vmess/inbound": 导入 "v2ray.com/core/proxy/vmess/inbound" 包。
 - "v2ray.com/core/proxy/vmess/outbound": 导入 "v2ray.com/core/proxy/vmess/outbound" 包。
 - "v2ray.com/core/testing/servers/tcp": 导入 "v2ray.com/core/testing/servers/tcp" 包。
 - "v2ray.com/core/testing/servers/udp": 导入 "


```go
package scenarios

import (
	"context"
	"io/ioutil"
	"net/http"
	"net/url"
	"testing"
	"time"

	xproxy "golang.org/x/net/proxy"
	"v2ray.com/core"
	"v2ray.com/core/app/dispatcher"
	"v2ray.com/core/app/log"
	"v2ray.com/core/app/proxyman"
	_ "v2ray.com/core/app/proxyman/inbound"
	_ "v2ray.com/core/app/proxyman/outbound"
	"v2ray.com/core/app/router"
	"v2ray.com/core/common"
	clog "v2ray.com/core/common/log"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/protocol"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/common/uuid"
	"v2ray.com/core/proxy/blackhole"
	"v2ray.com/core/proxy/dokodemo"
	"v2ray.com/core/proxy/freedom"
	v2http "v2ray.com/core/proxy/http"
	"v2ray.com/core/proxy/socks"
	"v2ray.com/core/proxy/vmess"
	"v2ray.com/core/proxy/vmess/inbound"
	"v2ray.com/core/proxy/vmess/outbound"
	"v2ray.com/core/testing/servers/tcp"
	"v2ray.com/core/testing/servers/udp"
	"v2ray.com/core/transport/internet"
)

```

This is a Go program that creates a CoreDNS server. It defines a handler for incoming requests and an outbound handler for processing domains.

The program first initializes a set of server configurations and closes all servers when done. It then creates a connection to the server and reads a response from the incoming request.

The incoming handler is defined as a function that receives a CoreDNS request and returns a response. It creates an outgoing connection to the server and reads a response from the incoming request. It compares the response with the expected "send first" response and logs an error if the response is not what is expected.

The outbound handler is defined as a function that receives a list of domains to process and a timeout value. It creates an outgoing connection to the server and reads a response from the incoming request for each domain. It compares the response with the expected send first response for each domain and logs an error if the response is not what is expected.

The program uses the `net` package to create TCP connections and the `core` package to create CoreDNS messages.


```go
func TestPassiveConnection(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
		SendFirst:    []byte("send first"),
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	conn, err := net.DialTCP("tcp", nil, &net.TCPAddr{
		IP:   []byte{127, 0, 0, 1},
		Port: int(serverPort),
	})
	common.Must(err)

	{
		response := make([]byte, 1024)
		nBytes, err := conn.Read(response)
		common.Must(err)
		if string(response[:nBytes]) != "send first" {
			t.Error("unexpected first response: ", string(response[:nBytes]))
		}
	}

	if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}
}

```

This is a Go program that appears to be testing a simple HTTP server using the Go-RPC library.

The program has a main function that initializes a server, connects a client to the server, and sends a request to the server using the HTTP request method.

The server is initialized with a configuration file that specifies the server endpoints and the connection settings for the server. The server listens for incoming connections on a specified port and uses the connection settings to establish a TCP connection with each client.

The connection settings for the server include the代理选项，即使用代理服务器。通过测试TCP连通性，如果服务器接收到连接请求，它将尝试与客户端建立一个TCP连接。如果连接建立成功，客户端就可以向代理服务器发送请求，并将服务器的响应接收过来。

此外，服务器也监听来自客户端的账户信息，以便在客户端建立账户并登录服务器时进行验证。


```go
func TestProxy(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverUserID := protocol.NewID(uuid.New())
	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: serverUserID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	proxyUserID := protocol.NewID(uuid.New())
	proxyPort := tcp.PickPort()
	proxyConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(proxyPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: proxyUserID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	clientPort := tcp.PickPort()
	clientConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(clientPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(serverPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: serverUserID.String(),
									}),
								},
							},
						},
					},
				}),
				SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
					ProxySettings: &internet.ProxyConfig{
						Tag: "proxy",
					},
				}),
			},
			{
				Tag: "proxy",
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(proxyPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: proxyUserID.String(),
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig, proxyConfig, clientConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	if err := testTCPConn(clientPort, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}
}

```

This is a Go program that initializes a server to handle incoming connections. It appears to be using the ProxyMan library to handle the connection and the Go-TCP/Go-TCP-JSON library to handle the actual TCP connections.

The program is creating a server that listens for incoming connections on port 1024 by creating a proxyman.ServerEndpoint struct and passing it to the ProxySettings of the SenderSettings struct. This tells the server to use the proxyman library to forward the incoming connections to the receiver, which is specified by the Receiver setting of the SenderSettings struct.

The SenderSettings struct is then configured with the StreamSettings of the internet.StreamConfig struct, which specifies that the receiver should be configured to receive streams.

The connection is established using the testTCPConn function which creates a client that sends a connection to the server, waits for a response, and sends a stream of data to the server.

It is important to note that this server is not secure and should not be used for production purposes. The connection initialization is using the default credentials provided by the Go-TCP library, which may expose sensitive information.


```go
func TestProxyOverKCP(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverUserID := protocol.NewID(uuid.New())
	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
					StreamSettings: &internet.StreamConfig{
						Protocol: internet.TransportProtocol_MKCP,
					},
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: serverUserID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	proxyUserID := protocol.NewID(uuid.New())
	proxyPort := tcp.PickPort()
	proxyConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(proxyPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id: proxyUserID.String(),
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
				SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
					StreamSettings: &internet.StreamConfig{
						Protocol: internet.TransportProtocol_MKCP,
					},
				}),
			},
		},
	}

	clientPort := tcp.PickPort()
	clientConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(clientPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(serverPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: serverUserID.String(),
									}),
								},
							},
						},
					},
				}),
				SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
					ProxySettings: &internet.ProxyConfig{
						Tag: "proxy",
					},
					StreamSettings: &internet.StreamConfig{
						Protocol: internet.TransportProtocol_MKCP,
					},
				}),
			},
			{
				Tag: "proxy",
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(proxyPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id: proxyUserID.String(),
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig, proxyConfig, clientConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	if err := testTCPConn(clientPort, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}
}

```

This is a Go program that configures a proxy server to route traffic from a client to a server. The program uses the `core` and `freedom` Routes to handle outgoing traffic and `blackhole` Routes to handle incoming traffic that is considered "blocked".

The program sets up a server with port `serverPort2` and creates a proxy settings that routes traffic to a server with `dest2` as the destination. The program also sets up a route with the tag "direct" and the port `80` to handle incoming traffic from the client.

The program initializes the server configuration and opens the connection. If the connection is successful, it waits for 5 seconds before closing the connection.

The program uses the `testTCPConn` function from `testing` to test the connection. If the connection is established, an error is expected. If the connection is not established, the program uses the `t.Error` function to log an error message.


```go
func TestBlackhole(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	tcpServer2 := tcp.Server{
		MsgProcessor: xor,
	}
	dest2, err := tcpServer2.Start()
	common.Must(err)
	defer tcpServer2.Close()

	serverPort := tcp.PickPort()
	serverPort2 := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort2),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest2.Address),
					Port:    uint32(dest2.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				Tag:           "direct",
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
			{
				Tag:           "blocked",
				ProxySettings: serial.ToTypedMessage(&blackhole.Config{}),
			},
		},
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&router.Config{
				Rule: []*router.RoutingRule{
					{
						TargetTag: &router.RoutingRule_Tag{
							Tag: "blocked",
						},
						PortRange: net.SinglePortRange(dest2.Port),
					},
				},
			}),
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	if err := testTCPConn(serverPort2, 1024, time.Second*5)(); err == nil {
		t.Error("nil error")
	}
}

```

This appears to be a network setting for a Node.js application that uses the `xproxy` package to handle TCP connections.

The settings include a SOCKS5 proxy server that allows the Node.js application to forward requests to a remote server to another server using a generated SOCKS5 proxy. The proxy is set to listen on the local host IP and a specified port, and is configured to allow traffic to a remote server with the IP address `google.com` and port `80`.

The `noAuthDialer` is used to establish a connection to the remote server using the `xproxy.SOCKS5` method without authentication, and the `noAuthDialer.Dial` method to open a connection to the remote server.

The `testTCPConn2` function is used to establish a connection to a remote server using the `testTCPConn2` function, which initializes a TCP connection to port `80` on the remote server and waits for a response. If the connection is established successfully and a response is received within 5 seconds, the connection is considered successful.

Overall, this setting allows the Node.js application to establish a connection to a remote server and forward traffic to that server using a SOCKS5 proxy.


```go
func TestForward(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
					AuthType: socks.AuthType_NO_AUTH,
					Accounts: map[string]string{
						"Test Account": "Test Password",
					},
					Address:    net.NewIPOrDomain(net.LocalHostIP),
					UdpEnabled: false,
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{
					DestinationOverride: &freedom.DestinationOverride{
						Server: &protocol.ServerEndpoint{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(dest.Port),
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		noAuthDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, serverPort).NetAddr(), nil, xproxy.Direct)
		common.Must(err)
		conn, err := noAuthDialer.Dial("tcp", "google.com:80")
		common.Must(err)
		defer conn.Close()

		if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
			t.Error(err)
		}
	}
}

```

This is a Go test that tests the ability of an application with a given configuration to establish a UDP connection and send/receive data through it. It uses the `net` and `core` packages, as well as some custom code to create the `udp.Server` and `core.Config` objects.

The `TestUDPConnection` function runs the server by starting the `udp.Server` with a `msg_processor` that performs in-place message processing. It then starts a client with a randomly selected port and a client configuration that includes an outbound handler for the `dokodemo.Config` object, which is used to send data to the server.

The server is closed and the client is allowed to connect to it before the connection is closed. The server then waits for up to five seconds before closing, and then the client and server are closed.

If the connection cannot be established or the client disconnects, the test prints an error and logs one绿茶喷雾 (🍃).


```go
func TestUDPConnection(t *testing.T) {
	udpServer := udp.Server{
		MsgProcessor: xor,
	}
	dest, err := udpServer.Start()
	common.Must(err)
	defer udpServer.Close()

	clientPort := tcp.PickPort()
	clientConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(clientPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(dest.Address),
					Port:    uint32(dest.Port),
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_UDP},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(clientConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}

	time.Sleep(20 * time.Second)

	if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}
}

```

This is a Go program that sets up a configuration server to handle configuration messages that are sent by a router. It uses the Go-Auto毁灭库 to handle errors.

The program has several configuration settings:

* A router that listens for configuration messages on port 8080.
* A log configuration that writes log messages to a file specified by the `-logFile` flag.
* A server configuration that initializes the router and log configuration, and starts the router server.

The router listens for incoming configuration messages on port 8080 and routes them to the corresponding handler. The log configuration writes log messages to a file specified by the `-logFile` flag. The server configuration initializes the router server and starts it.


```go
func TestDomainSniffing(t *testing.T) {
	sniffingPort := tcp.PickPort()
	httpPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				Tag: "snif",
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(sniffingPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
					DomainOverride: []proxyman.KnownProtocols{
						proxyman.KnownProtocols_TLS,
					},
				}),
				ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
					Address: net.NewIPOrDomain(net.LocalHostIP),
					Port:    443,
					NetworkList: &net.NetworkList{
						Network: []net.Network{net.Network_TCP},
					},
				}),
			},
			{
				Tag: "http",
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(httpPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				Tag: "redir",
				ProxySettings: serial.ToTypedMessage(&freedom.Config{
					DestinationOverride: &freedom.DestinationOverride{
						Server: &protocol.ServerEndpoint{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(sniffingPort),
						},
					},
				}),
			},
			{
				Tag:           "direct",
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&router.Config{
				Rule: []*router.RoutingRule{
					{
						TargetTag: &router.RoutingRule_Tag{
							Tag: "direct",
						},
						InboundTag: []string{"snif"},
					}, {
						TargetTag: &router.RoutingRule_Tag{
							Tag: "redir",
						},
						InboundTag: []string{"http"},
					},
				},
			}),
			serial.ToTypedMessage(&log.Config{
				ErrorLogLevel: clog.Severity_Debug,
				ErrorLogType:  log.LogType_Console,
			}),
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + httpPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		resp, err := client.Get("https://www.github.com/")
		common.Must(err)
		if resp.StatusCode != 200 {
			t.Error("unexpected status code: ", resp.StatusCode)
		}
		common.Must(resp.Write(ioutil.Discard))
	}
}

```

This appears to be a function definition for a Core回波系统中的配置选项。该系统似乎允许用户配置服务器以接收消息，并在客户端和服务器之间建立安全连接。以下是配置选项的特定部分：

go
[dependencies]
图为：xhttp、xgo、xlet、json、net、io、encoding
图为：go.route、go.setup、go.pkg.glify、go.cmd、go. coredns
图为：github.com/a铃木/go-象棋引擎、github.com/ast相遇/go-预案、github.com/coreos/go-豆腐、github.com/em蕴贝尔/go-缓存、github.com/锋芒独立/go-随机数、github.com/鬼谷子/go-签名、github.com/激荡异想/go-json-十字章、github.com/轻度用户/go-工具、github.com/long tendance/go-量热、github.com/鸣响饱听/go-发送-Websocket、github.com/ Spor旅行者/go-ws-赵云、github.com/ stx/go-sqlite3、github.com/stx/go-useast
图为：github.com/合成ature/go-cloudflare、github.com/共存计划/go-dam/表格、github.com/ioredial/go-folders、github.com/ Iknothere/go-noms、github.com/k亭根/go-应用/cache/redis、github.com/零快速/go-fslapply、github.com/罗生 apolog/go-parse-yaml、github.com/sirupsen/go-时间、github.com/ Spre和行动/go-todo-排队、github.com/ Weis Ultimately 5/go-新闻源、github.com/foremost/go-抹布吸尘器、github.com/林皓_9/go-红花生、github.com/荣幸 Incorporated/go-合并告诉他、github.com/文的杜小康/go-不定量的胶囊、github.com/ZZ射手/go-斜率网络、github.com/标注者/go-标注、github.com/模拟生产者/go-多线程、github.com/单细胞机器人/go-多线程、github.com/十五个调度的W双向千兆/go-鉴权



[dependencies]
图为：go.route、go.setup、go.pkg.glify、go.cmd、go. coredns
图为：github.com/ast相遇/go-预案、github.com/coreos/go-豆腐、github.com/em蕴贝尔/go-豆腐、github.com/轻度用户/go-工具、github.com/鬼谷子/go-签名、github.com/激荡异想/go-随机数、github.com/锋芒独立/go-歌词解析、github.com/ HRS provided/go-氢溴硫胺、github.com/ Jd King/go-判断力、github.com/烈焰灵魂/go-交易信息、github.com/南京二一/go-评阅卡、github.com/ param0/go-随机数、github.com/ silver-gate/go-手持设备、github.com/ skimp/go-压缩、github.com/ status-audio/go-静音、github.com/snowflakedb/go-mbdb、github.com/ substance-manager/go-substitute、github.com/ TheP差距/go-Aspects
图为：github.com/穿越银河系/go-You衍界、github.com/ em在三里屯/go-物价控、github.com/ 花火互鉴/go-高效独立托管、github.com/ MirrorMine/go-port-srv、github.com/ remains/go-内存管理、github.com/ rhythmgui/go-核潜艇、github.com/ map顺/go-极光、github.com/笔名南/go- rainwater、github.com/ you-compile/go-不会变质、github.com/ estyling/go- mutantgo、github.com/ G我就)/go-大红



```go
func TestDialV2Ray(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	userID := protocol.NewID(uuid.New())
	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&log.Config{
				ErrorLogLevel: clog.Severity_Debug,
				ErrorLogType:  log.LogType_Console,
			}),
		},
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&inbound.Config{
					User: []*protocol.User{
						{
							Account: serial.ToTypedMessage(&vmess.Account{
								Id:      userID.String(),
								AlterId: 64,
							}),
						},
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	clientConfig := &core.Config{
		App: []*serial.TypedMessage{
			serial.ToTypedMessage(&dispatcher.Config{}),
			serial.ToTypedMessage(&proxyman.InboundConfig{}),
			serial.ToTypedMessage(&proxyman.OutboundConfig{}),
		},
		Inbound: []*core.InboundHandlerConfig{},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&outbound.Config{
					Receiver: []*protocol.ServerEndpoint{
						{
							Address: net.NewIPOrDomain(net.LocalHostIP),
							Port:    uint32(serverPort),
							User: []*protocol.User{
								{
									Account: serial.ToTypedMessage(&vmess.Account{
										Id:      userID.String(),
										AlterId: 64,
										SecuritySettings: &protocol.SecurityConfig{
											Type: protocol.SecurityType_AES128_GCM,
										},
									}),
								},
							},
						},
					},
				}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	client, err := core.New(clientConfig)
	common.Must(err)

	conn, err := core.Dial(context.Background(), client, dest)
	common.Must(err)
	defer conn.Close()

	if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
		t.Error(err)
	}
}

```

# `testing/scenarios/http_test.go`

该代码包名为“scenarios”，包含了一些与网络和HTTP相关的接口和函数。

1. 导入一些必要的包：
	* “bytes”：字节序列
	* “crypto/rand”：用于生成随机RSA加密密钥的库
	* “io”：输入/输出系统套接字
	* “io/ioutil”：用于处理输入/输出文件的库
	* “net/http”：用于设置HTTP请求的库
	* “net/url”：用于解析URL的库
	* “testing”：用于进行测试的包
	* “time”：用于测量时间的库
	* “github.com/google/go-cmp/cmp”：用于比较两个字符串的库
	* “v2ray.com/core”：用于与V2Ray服务器通信的库
	* “v2ray.com/core/app/proxyman”：用于设置代理的库
	* “v2ray.com/core/common”：用于全局常量的库
	* “v2ray.com/core/common/buf”：用于缓冲数据的库
	* “v2ray.com/core/common/net”：用于网络通信的库
	* “v2ray.com/core/proxy/freedom”：用于设置代理自由的库
	* “v2http”：用于设置HTTP请求的库
	* “v2httptest”：用于测试HTTP请求的库
	* “v2ray.com/core/testing/servers/http”：用于测试代理的库
	* “v2ray.com/core/testing/servers/tcp”：用于测试TCP代理的库
2. 实现了一些与HTTP和代理相关的函数：
	* 设置随机RSA加密密钥
	* 创建HTTP请求
	* 解析HTTP请求并返回结果
	* 创建TCP连接并发送数据
	* 设置代理自由
	* 通过代理发送HTTP请求
	* 通过代理接收HTTP请求
	* 循环接收和发送HTTP请求
	* 通过代理连接到V2Ray服务器
	* 通过V2Ray服务器发送代理消息
	* 通过V2Ray服务器接收代理消息
	* 下载并解码HTTP请求和响应
	* 解析JSON格式的数据
	* 打印测试结果
3. 在“scenarios”目录下创建了一个名为“test_scenarios.go”的文件，用于测试代理的行为。
4. 在“v2ray.com/core/testing/servers/http”目录下创建了一个名为“test_http.go”的文件，用于测试HTTP代理的行为。
	* 在“test_scenarios.go”中，定义了一些测试函数，如“test_proxy_echo_request”、“test_proxy_redirect_request”、“test_proxy_youtube_video_stream”等，用于测试代理的转发和异常情况。
	* 在“test_http.go”中，实现了“v2ray.com/core/testing/servers/http”中的“下载并解析HTTP请求”函数，用于下载并解析HTTP请求，以进行代理的转发和异常情况测试。


```go
package scenarios

import (
	"bytes"
	"crypto/rand"
	"io"
	"io/ioutil"
	"net/http"
	"net/url"
	"testing"
	"time"

	"github.com/google/go-cmp/cmp"

	"v2ray.com/core"
	"v2ray.com/core/app/proxyman"
	"v2ray.com/core/common"
	"v2ray.com/core/common/buf"
	"v2ray.com/core/common/net"
	"v2ray.com/core/common/serial"
	"v2ray.com/core/proxy/freedom"
	v2http "v2ray.com/core/proxy/http"
	v2httptest "v2ray.com/core/testing/servers/http"
	"v2ray.com/core/testing/servers/tcp"
)

```

This is a Go program that initializes a server using the OpenResty Go framework. Here's a high-level overview of what it does:

1. It selects an available port to use as the server's port.
2. It sets up the server's configuration, including the incoming and outgoing connections.
3. It initializes the server by connecting to a proxy server.
4. It sets up the HTTP request receiver on the proxy server and creates a new HTTP request to get the response from the server.
5. It extracts the response body of the request and prints it to the console.

The program uses several斗篷意义的高新奇技术，例如 ioutil.ReadAll 函数，该函数用于读取 HTTP 响应的内容并返回给主程序。


```go
func TestHttpConformance(t *testing.T) {
	httpServerPort := tcp.PickPort()
	httpServer := &v2httptest.Server{
		Port:        httpServerPort,
		PathHandler: make(map[string]http.HandlerFunc),
	}
	_, err := httpServer.Start()
	common.Must(err)
	defer httpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + serverPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		resp, err := client.Get("http://127.0.0.1:" + httpServerPort.String())
		common.Must(err)
		if resp.StatusCode != 200 {
			t.Fatal("status: ", resp.StatusCode)
		}

		content, err := ioutil.ReadAll(resp.Body)
		common.Must(err)
		if string(content) != "Home" {
			t.Fatal("body: ", string(content))
		}
	}
}

```

This is a Go program that starts a server using the go-core-https library.

The program starts by creating an empty slice to hold the response from the server. It then creates a receive circuit for incoming requests on port 8080 (which is the default port used by HTTP servers) and a transmit circuit for outgoing requests.

The receive circuit is configured to listen for incoming requests on the default port and to forward them to the specified receiver. The transmit circuit is configured to proxy the incoming requests to the specified destination port.

The program then initializes the server using the InitializeServerConfigs function, which takes a configuration object and returns it. This configuration object is used to configure the server's settings, such as the server port, receiver, and transmitter.

The program then starts the server by calling the Start function on the tcpServer object.

The program then waits for up to 2 seconds and then checks if the server has finished listening for incoming requests. If it has not, it will close the server and stop listening for requests.


```go
func TestHttpError(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: func(msg []byte) []byte {
			return []byte{}
		},
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	time.AfterFunc(time.Second*2, func() {
		tcpServer.ShouldClose = true
	})

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + serverPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		resp, err := client.Get("http://127.0.0.1:" + dest.Port.String())
		common.Must(err)
		if resp.StatusCode != 503 {
			t.Error("status: ", resp.StatusCode)
		}
	}
}

```

This appears to be a Go program that sets up a server using the Go-RPi. sysd system and the Jetty HTTP server. It creates a proxy server that listens on the local host IP and the server port and routes traffic to the Jetty server. It also creates an outbound HTTP client to make requests to the Jetty server.


```go
func TestHttpConnectMethod(t *testing.T) {
	tcpServer := tcp.Server{
		MsgProcessor: xor,
	}
	dest, err := tcpServer.Start()
	common.Must(err)
	defer tcpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + serverPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		payload := make([]byte, 1024*64)
		common.Must2(rand.Read(payload))
		req, err := http.NewRequest("Connect", "http://"+dest.NetAddr()+"/", bytes.NewReader(payload))
		req.Header.Set("X-a", "b")
		req.Header.Set("X-b", "d")
		common.Must(err)

		resp, err := client.Do(req)
		common.Must(err)
		if resp.StatusCode != 200 {
			t.Fatal("status: ", resp.StatusCode)
		}

		content := make([]byte, len(payload))
		common.Must2(io.ReadFull(resp.Body, content))
		if r := cmp.Diff(content, xor(payload)); r != "" {
			t.Fatal(r)
		}
	}
}

```

This is a Go program that initializes a Server based on the Go Server/网络程序. It starts by initializing a receiver and then creates an Outbound connection to a remote server using the HTTP protocol.

The receiver is configured to listen on a specific port, and for incoming connections, it will handle incoming messages using a received message from a sender, which is a Go Server, and then send the received message back to the sender.

The Outbound connections are configured to use the HTTP protocol, and are configured to proxy the connection to the specified Server, which is defined in the Outbound settings of the ServerConfig.

It is using the net "http" package for the HTTP communication and the "freedom" package for the Go Server and the "core" package for the Go connection.


```go
func TestHttpPost(t *testing.T) {
	httpServerPort := tcp.PickPort()
	httpServer := &v2httptest.Server{
		Port: httpServerPort,
		PathHandler: map[string]http.HandlerFunc{
			"/testpost": func(w http.ResponseWriter, r *http.Request) {
				payload, err := buf.ReadAllToBytes(r.Body)
				r.Body.Close()

				if err != nil {
					w.WriteHeader(500)
					w.Write([]byte("Unable to read all payload"))
					return
				}
				payload = xor(payload)
				w.Write(payload)
			},
		},
	}

	_, err := httpServer.Start()
	common.Must(err)
	defer httpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + serverPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		payload := make([]byte, 1024*64)
		common.Must2(rand.Read(payload))

		resp, err := client.Post("http://127.0.0.1:"+httpServerPort.String()+"/testpost", "application/x-www-form-urlencoded", bytes.NewReader(payload))
		common.Must(err)
		if resp.StatusCode != 200 {
			t.Fatal("status: ", resp.StatusCode)
		}

		content, err := ioutil.ReadAll(resp.Body)
		common.Must(err)
		if r := cmp.Diff(content, xor(payload)); r != "" {
			t.Fatal(r)
		}
	}
}

```

This looks like a Python function that is testing the HTTP request made by the Echo server. The Echo server is a simple HTTP server that responds to all incoming requests with the same "Hello World" message.

The function takes in a URL and an HTTP server port number as input, and then tries to make a GET request to that URL using the specified HTTP client. If the request is successful (i.e. the HTTP status code is 200), the function reads the response body and checks its contents against a expected "Home" string. If the response body does not contain the expected "Home" string, the function will raise an error.

It is important to note that this function may not work as intended, as it is not tested to ensure that it behaves correctly in all possible scenarios.


```go
func setProxyBasicAuth(req *http.Request, user, pass string) {
	req.SetBasicAuth(user, pass)
	req.Header.Set("Proxy-Authorization", req.Header.Get("Authorization"))
	req.Header.Del("Authorization")
}

func TestHttpBasicAuth(t *testing.T) {
	httpServerPort := tcp.PickPort()
	httpServer := &v2httptest.Server{
		Port:        httpServerPort,
		PathHandler: make(map[string]http.HandlerFunc),
	}
	_, err := httpServer.Start()
	common.Must(err)
	defer httpServer.Close()

	serverPort := tcp.PickPort()
	serverConfig := &core.Config{
		Inbound: []*core.InboundHandlerConfig{
			{
				ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
					PortRange: net.SinglePortRange(serverPort),
					Listen:    net.NewIPOrDomain(net.LocalHostIP),
				}),
				ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{
					Accounts: map[string]string{
						"a": "b",
					},
				}),
			},
		},
		Outbound: []*core.OutboundHandlerConfig{
			{
				ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
			},
		},
	}

	servers, err := InitializeServerConfigs(serverConfig)
	common.Must(err)
	defer CloseAllServers(servers)

	{
		transport := &http.Transport{
			Proxy: func(req *http.Request) (*url.URL, error) {
				return url.Parse("http://127.0.0.1:" + serverPort.String())
			},
		}

		client := &http.Client{
			Transport: transport,
		}

		{
			resp, err := client.Get("http://127.0.0.1:" + httpServerPort.String())
			common.Must(err)
			if resp.StatusCode != 407 {
				t.Fatal("status: ", resp.StatusCode)
			}
		}

		{
			req, err := http.NewRequest("GET", "http://127.0.0.1:"+httpServerPort.String(), nil)
			common.Must(err)

			setProxyBasicAuth(req, "a", "c")
			resp, err := client.Do(req)
			common.Must(err)
			if resp.StatusCode != 407 {
				t.Fatal("status: ", resp.StatusCode)
			}
		}

		{
			req, err := http.NewRequest("GET", "http://127.0.0.1:"+httpServerPort.String(), nil)
			common.Must(err)

			setProxyBasicAuth(req, "a", "b")
			resp, err := client.Do(req)
			common.Must(err)
			if resp.StatusCode != 200 {
				t.Fatal("status: ", resp.StatusCode)
			}

			content, err := ioutil.ReadAll(resp.Body)
			common.Must(err)
			if string(content) != "Home" {
				t.Fatal("body: ", string(content))
			}
		}
	}
}

```