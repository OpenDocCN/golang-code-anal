# `v2ray-core\testing\scenarios\socks_test.go`

```go
package scenarios

import (
    "testing"
    "time"

    xproxy "golang.org/x/net/proxy"
    socks4 "h12.io/socks"

    "v2ray.com/core"
    "v2ray.com/core/app/proxyman"
    "v2ray.com/core/app/router"
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/proxy/blackhole"
    "v2ray.com/core/proxy/dokodemo"
    "v2ray.com/core/proxy/freedom"
    "v2ray.com/core/proxy/socks"
    "v2ray.com/core/testing/servers/tcp"
    "v2ray.com/core/testing/servers/udp"
)

func TestSocksBridgeTCP(t *testing.T) {
    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置，监听指定端口和地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置，使用密码认证，指定账号和密码，以及地址和 UDP 是否启用
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    AuthType: socks.AuthType_PASSWORD,
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
                // 设置出站代理配置，使用自由代理
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个可用的客户端端口
    clientPort := tcp.PickPort()
}
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort), // 设置端口范围
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
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP), // 设置服务器地址
                            Port:    uint32(serverPort), // 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: serial.ToTypedMessage(&socks.Account{
                                        Username: "Test Account", // 设置用户名
                                        Password: "Test Password", // 设置密码
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
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
        t.Error(err)
    }
func TestSocksBridageUDP(t *testing.T) {
    // 创建一个 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 UDP 服务器并获取目标地址和可能的错误
    dest, err := udpServer.Start()
    common.Must(err)  // 必须处理错误
    defer udpServer.Close()  // 延迟关闭 UDP 服务器

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    AuthType: socks.AuthType_PASSWORD,  // 设置认证类型为密码
                    Accounts: map[string]string{  // 设置账户和密码
                        "Test Account": "Test Password",
                    },
                    Address:    net.NewIPOrDomain(net.LocalHostIP),  // 设置地址
                    UdpEnabled: true,  // 启用 UDP
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置代理配置为自由
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort), // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP), // 设置监听地址
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address), // 设置目标地址
                    Port:    uint32(dest.Port), // 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP, net.Network_UDP}, // 设置网络列表
                    },
                }),
            },
        },
        // 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP), // 设置服务器地址
                            Port:    uint32(serverPort), // 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: serial.ToTypedMessage(&socks.Account{
                                        Username: "Test Account", // 设置用户名
                                        Password: "Test Password", // 设置密码
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
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 UDP 连接
    if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
        t.Error(err)
    }
func TestSocksBridageUDPWithRouting(t *testing.T) {
    // 创建一个 UDP 服务器对象，使用 xor 作为消息处理器
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器，获取目标地址和可能的错误
    dest, err := udpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将路由配置转换为 TypedMessage 对象
            serial.ToTypedMessage(&router.Config{
                // 设置路由规则
                Rule: []*router.RoutingRule{
                    {
                        // 设置目标标签为 "out"
                        TargetTag: &router.RoutingRule_Tag{
                            Tag: "out",
                        },
                        // 设置入站标签为 "socks"
                        InboundTag: []string{"socks"},
                    },
                },
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置标签为 "socks"
                Tag: "socks",
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围为单一端口
                    PortRange: net.SinglePortRange(serverPort),
                    // 监听本地主机 IP 地址
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    // 设置认证类型为无认证
                    AuthType:   socks.AuthType_NO_AUTH,
                    // 设置地址为本地主机 IP 地址
                    Address:    net.NewIPOrDomain(net.LocalHostIP),
                    // 启用 UDP
                    UdpEnabled: true,
                }),
            },
        },
        // 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置为黑洞
                ProxySettings: serial.ToTypedMessage(&blackhole.Config{}),
            },
            {
                // 设置标签为 "out"
                Tag:           "out",
                // 设置代理配置为自由
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个可用的客户端端口
    clientPort := tcp.PickPort()
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置，包括目标地址、端口和网络列表
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),
                    Port:    uint32(dest.Port),
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP, net.Network_UDP},
                    },
                }),
            },
        },
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括服务器地址和端口
                ProxySettings: serial.ToTypedMessage(&socks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                        },
                    },
                }),
            },
        },
    }

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    # 检查错误
    common.Must(err)
    # 延迟关闭所有服务器
    defer CloseAllServers(servers)

    # 测试 UDP 连接
    if err := testUDPConn(clientPort, 1024, time.Second*5)(); err != nil {
        # 输出错误信息
        t.Error(err)
    }
}
# 测试 SOCKS 协议的一致性
func TestSocksConformanceMod(t *testing.T) {
    # 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    # 启动 TCP 服务器并返回目标地址和错误信息
    dest, err := tcpServer.Start()
    # 必须处理错误
    common.Must(err)
    # 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    # 选择一个认证端口和一个无认证端口
    authPort := tcp.PickPort()
    noAuthPort := tcp.PickPort()
    # 创建服务器配置对象
    serverConfig := &core.Config{
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(authPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    AuthType: socks.AuthType_PASSWORD,
                    Accounts: map[string]string{
                        "Test Account": "Test Password",
                    },
                    Address:    net.NewIPOrDomain(net.LocalHostIP),
                    UdpEnabled: false,
                }),
            },
            {
                # 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(noAuthPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置
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
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }
    # 初始化服务器配置并返回服务器对象和错误信息
    servers, err := InitializeServerConfigs(serverConfig)
    # 必须处理错误
    common.Must(err)
    # 延迟关闭所有服务器
    defer CloseAllServers(servers)
    {
        // 创建一个不需要认证的 SOCKS5 拨号器
        noAuthDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, noAuthPort).NetAddr(), nil, xproxy.Direct)
        // 检查错误
        common.Must(err)
        // 通过拨号器连接目标地址
        conn, err := noAuthDialer.Dial("tcp", dest.NetAddr())
        // 检查错误
        common.Must(err)
        // 延迟关闭连接
        defer conn.Close()

        // 测试连接
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }

    {
        // 创建一个需要认证的 SOCKS5 拨号器
        authDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, authPort).NetAddr(), &xproxy.Auth{User: "Test Account", Password: "Test Password"}, xproxy.Direct)
        // 检查错误
        common.Must(err)
        // 通过拨号器连接目标地址
        conn, err := authDialer.Dial("tcp", dest.NetAddr())
        // 检查错误
        common.Must(err)
        // 延迟关闭连接
        defer conn.Close()

        // 测试连接
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }

    {
        // 创建一个 SOCKS4 拨号器
        dialer := socks4.Dial("socks4://" + net.TCPDestination(net.LocalHostIP, noAuthPort).NetAddr())
        // 通过拨号器连接目标地址
        conn, err := dialer("tcp", dest.NetAddr())
        // 检查错误
        common.Must(err)
        // 延迟关闭连接
        defer conn.Close()

        // 测试连接
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }

    {
        // 创建一个 SOCKS4 拨号器
        dialer := socks4.Dial("socks4://" + net.TCPDestination(net.LocalHostIP, noAuthPort).NetAddr())
        // 通过拨号器连接目标地址
        conn, err := dialer("tcp", net.TCPDestination(net.LocalHostIP, tcpServer.Port).NetAddr())
        // 检查错误
        common.Must(err)
        // 延迟关闭连接
        defer conn.Close()

        // 测试连接
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }
# 闭合前面的函数定义
```