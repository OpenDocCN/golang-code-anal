# `v2ray-core\testing\scenarios\feature_test.go`

```
package scenarios

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "io/ioutil"  // 用于读取文件内容的包
    "net/http"  // HTTP 协议的客户端和服务端实现
    "net/url"  // 用于解析 URL 的包
    "testing"  // Go 语言的测试框架
    "time"  // 时间处理包

    xproxy "golang.org/x/net/proxy"  // 代理包
    "v2ray.com/core"  // V2Ray 核心库
    "v2ray.com/core/app/dispatcher"  // V2Ray 调度器
    "v2ray.com/core/app/log"  // V2Ray 日志
    "v2ray.com/core/app/proxyman"  // V2Ray 代理管理
    _ "v2ray.com/core/app/proxyman/inbound"  // V2Ray 入站代理管理
    _ "v2ray.com/core/app/proxyman/outbound"  // V2Ray 出站代理管理
    "v2ray.com/core/app/router"  // V2Ray 路由
    "v2ray.com/core/common"  // V2Ray 公共功能
    clog "v2ray.com/core/common/log"  // V2Ray 日志
    "v2ray.com/core/common/net"  // V2Ray 网络功能
    "v2ray.com/core/common/protocol"  // V2Ray 协议
    "v2ray.com/core/common/serial"  // V2Ray 序列化
    "v2ray.com/core/common/uuid"  // V2Ray UUID
    "v2ray.com/core/proxy/blackhole"  // V2Ray 黑洞代理
    "v2ray.com/core/proxy/dokodemo"  // V2Ray 毒药代理
    "v2ray.com/core/proxy/freedom"  // V2Ray 自由代理
    v2http "v2ray.com/core/proxy/http"  // V2Ray HTTP 代理
    "v2ray.com/core/proxy/socks"  // V2Ray SOCKS 代理
    "v2ray.com/core/proxy/vmess"  // V2Ray VMess 代理
    "v2ray.com/core/proxy/vmess/inbound"  // V2Ray VMess 入站代理
    "v2ray.com/core/proxy/vmess/outbound"  // V2Ray VMess 出站代理
    "v2ray.com/core/testing/servers/tcp"  // V2Ray TCP 服务器
    "v2ray.com/core/testing/servers/udp"  // V2Ray UDP 服务器
    "v2ray.com/core/transport/internet"  // V2Ray 互联网传输
)

func TestPassiveConnection(t *testing.T) {
    tcpServer := tcp.Server{  // 创建一个 TCP 服务器对象
        MsgProcessor: xor,  // 消息处理器为 xor 函数
        SendFirst:    []byte("send first"),  // 发送的第一个消息内容
    }
    dest, err := tcpServer.Start()  // 启动 TCP 服务器，获取目标地址和错误信息
    common.Must(err)  // 如果有错误，立即终止程序
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    serverPort := tcp.PickPort()  // 选择一个可用的端口
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort), // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP), // 设置监听地址
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address), // 设置目标地址
                    Port:    uint32(dest.Port), // 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP}, // 设置网络类型为 TCP
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}), // 设置自由代理配置
            },
        },
    }

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 建立 TCP 连接
    conn, err := net.DialTCP("tcp", nil, &net.TCPAddr{
        IP:   []byte{127, 0, 0, 1}, // 设置目标 IP 地址
        Port: int(serverPort), // 设置目标端口
    })
    common.Must(err)

    // 读取连接响应
    {
        response := make([]byte, 1024)
        nBytes, err := conn.Read(response)
        common.Must(err)
        // 检查响应内容是否为 "send first"
        if string(response[:nBytes]) != "send first" {
            t.Error("unexpected first response: ", string(response[:nBytes]))
        }
    }

    // 测试 TCP 连接
    if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
        t.Error(err)
    }
func TestProxy(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 检查错误，如果有错误则终止程序
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个服务器用户 ID
    serverUserID := protocol.NewID(uuid.New())
    // 选择一个可用的端口作为服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置，包括用户账户信息
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id: serverUserID.String(),  // 设置用户 ID
                            }),
                        },
                    },
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置为 freedom
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 创建一个代理用户 ID
    proxyUserID := protocol.NewID(uuid.New())
    // 选择一个可用的端口作为代理端口
    proxyPort := tcp.PickPort()
}
    # 创建代理配置对象
    proxyConfig := &core.Config{
        # 设置入站代理处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(proxyPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置，包括用户账户信息
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
        # 设置出站代理处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    # 选择客户端端口
    clientPort := tcp.PickPort()

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, proxyConfig, clientConfig)
    common.Must(err)
    # 延迟关闭所有服务器
    defer CloseAllServers(servers)

    # 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*5)(); err != nil {
        t.Error(err)
    }
func TestProxyOverKCP(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 TCP 服务器并获取目标地址
    dest, err := tcpServer.Start()
    common.Must(err)  // 检查错误
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    // 生成服务器的用户 ID 和端口号
    serverUserID := protocol.NewID(uuid.New())
    serverPort := tcp.PickPort()
    // 配置服务器的参数
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器的配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址
                    StreamSettings: &internet.StreamConfig{
                        Protocol: internet.TransportProtocol_MKCP,  // 设置传输协议为 MKCP
                    },
                }),
                // 设置代理的配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id: serverUserID.String(),  // 设置用户 ID
                            }),
                        },
                    },
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置代理的配置
            },
        },
    }

    // 生成代理的用户 ID 和端口号
    proxyUserID := protocol.NewID(uuid.New())
    proxyPort := tcp.PickPort()
}
    # 创建代理配置对象
    proxyConfig := &core.Config{
        # 设置入站代理处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(proxyPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置，包括用户账户信息
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
        # 设置出站代理处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置为自由代理
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
                # 设置发送器配置，包括流设置协议为 MKCP
                SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
                    StreamSettings: &internet.StreamConfig{
                        Protocol: internet.TransportProtocol_MKCP,
                    },
                }),
            },
        },
    }

    # 选择客户端端口
    clientPort := tcp.PickPort()

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, proxyConfig, clientConfig)
    # 检查错误
    common.Must(err)
    # 延迟关闭所有服务器
    defer CloseAllServers(servers)

    # 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*5)(); err != nil {
        # 输出错误信息
        t.Error(err)
    }
func TestBlackhole(t *testing.T) {
    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，并获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建第二个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer2 := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动第二个 TCP 服务器，并获取目标地址和可能的错误
    dest2, err := tcpServer2.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭第二个 TCP 服务器
    defer tcpServer2.Close()

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 选择第二个服务器端口
    serverPort2 := tcp.PickPort()

    // 初始化服务器配置，返回服务器对象和可能的错误
    servers, err := InitializeServerConfigs(serverConfig)
    // 必须处理错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 如果测试 TCP 连接返回 nil 错误
    if err := testTCPConn(serverPort2, 1024, time.Second*5)(); err == nil {
        // 输出错误信息
        t.Error("nil error")
    }
}

func TestForward(t *testing.T) {
    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，并获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
}
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站代理配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort), // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP), // 设置监听地址
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    AuthType: socks.AuthType_NO_AUTH, // 设置认证类型为无认证
                    Accounts: map[string]string{ // 设置账号和密码
                        "Test Account": "Test Password",
                    },
                    Address:    net.NewIPOrDomain(net.LocalHostIP), // 设置代理地址
                    UdpEnabled: false, // 禁用 UDP
                },
            },
        },
        // 设置出站代理配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{
                    DestinationOverride: &freedom.DestinationOverride{
                        Server: &protocol.ServerEndpoint{
                            Address: net.NewIPOrDomain(net.LocalHostIP), // 设置服务器地址
                            Port:    uint32(dest.Port), // 设置服务器端口
                        },
                    },
                }),
            },
        },
    }
    
    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err)
    defer CloseAllServers(servers) // 延迟关闭所有服务器
    
    // 创建无认证的拨号器，连接到 google.com:80
    {
        noAuthDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, serverPort).NetAddr(), nil, xproxy.Direct)
        common.Must(err)
        conn, err := noAuthDialer.Dial("tcp", "google.com:80")
        common.Must(err)
        defer conn.Close() // 延迟关闭连接
    
        // 测试 TCP 连接
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }
// 测试 UDP 连接
func TestUDPConnection(t *testing.T) {
    // 创建 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器并获取目标地址
    dest, err := udpServer.Start()
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 选择客户端端口
    clientPort := tcp.PickPort()
    // 创建客户端配置
    clientConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
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
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(clientConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 UDP 连接
    if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
        t.Error(err)
    }

    // 等待 20 秒
    time.Sleep(20 * time.Second)

    // 再次测试 UDP 连接
    if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
        t.Error(err)
    }
}

// 测试域名嗅探
func TestDomainSniffing(t *testing.T) {
    // 选择嗅探端口
    sniffingPort := tcp.PickPort()
    // 选择 HTTP 端口
    httpPort := tcp.PickPort()
    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)
}
    {
        // 创建一个 HTTP 传输对象，设置代理为本地地址
        transport := &http.Transport{
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + httpPort.String())
            },
        }

        // 创建一个 HTTP 客户端对象，设置传输对象为上面创建的传输对象
        client := &http.Client{
            Transport: transport,
        }

        // 发送 GET 请求到指定 URL，获取响应和可能的错误
        resp, err := client.Get("https://www.github.com/")
        // 检查是否有错误发生，如果有则抛出异常
        common.Must(err)
        // 检查响应状态码是否为 200，如果不是则输出错误信息
        if resp.StatusCode != 200 {
            t.Error("unexpected status code: ", resp.StatusCode)
        }
        // 将响应内容丢弃，不保存到任何地方
        common.Must(resp.Write(ioutil.Discard))
    }
func TestDialV2Ray(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 TCP 服务器并返回目标地址和错误信息
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage 对象
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,  // 设置错误日志类型为控制台输出
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围为单一端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机 IP
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 设置用户账户信息
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),  // 设置用户 ID
                                AlterId: 64,  // 设置变换 ID
                            }),
                        },
                    },
                }),
            },
        },
        // 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 使用自由代理配置
            },
        },
    }
}
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将 dispatcher.Config 转换为 TypedMessage
            serial.ToTypedMessage(&dispatcher.Config{}),
            // 将 proxyman.InboundConfig 转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.InboundConfig{}),
            // 将 proxyman.OutboundConfig 转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{},
        // 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理设置
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    // 设置接收器配置
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            // 设置地址为本地主机IP
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            // 设置端口号为服务器端口
                            Port:    uint32(serverPort),
                            // 设置用户配置
                            User: []*protocol.User{
                                {
                                    // 设置账户配置
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        // 设置用户ID
                                        Id:      userID.String(),
                                        // 设置AlterId
                                        AlterId: 64,
                                        // 设置安全设置
                                        SecuritySettings: &protocol.SecurityConfig{
                                            // 设置安全类型为 AES128_GCM
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

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    // 检查错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 创建客户端
    client, err := core.New(clientConfig)
    // 检查错误
    common.Must(err)

    // 建立连接
    conn, err := core.Dial(context.Background(), client, dest)
    // 检查错误
    common.Must(err)
    // 延迟关闭连接
    defer conn.Close()

    // 测试TCP连接
    if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
        // 输出错误信息
        t.Error(err)
    }
# 闭合前面的函数定义
```