# `v2ray-core\testing\scenarios\tls_test.go`

```
# 导入所需的包
package scenarios

import (
    "crypto/x509"  # 导入加密证书相关的包
    "runtime"  # 导入运行时相关的包
    "testing"  # 导入测试相关的包
    "time"  # 导入时间相关的包

    "golang.org/x/sync/errgroup"  # 导入错误组相关的包

    "v2ray.com/core"  # 导入 V2Ray 核心包
    "v2ray.com/core/app/proxyman"  # 导入代理管理相关的包
    "v2ray.com/core/common"  # 导入通用功能相关的包
    "v2ray.com/core/common/net"  # 导入网络相关的包
    "v2ray.com/core/common/protocol"  # 导入协议相关的包
    "v2ray.com/core/common/protocol/tls/cert"  # 导入 TLS 证书相关的包
    "v2ray.com/core/common/serial"  # 导入序列化相关的包
    "v2ray.com/core/common/uuid"  # 导入 UUID 相关的包
    "v2ray.com/core/proxy/dokodemo"  # 导入 dokodemo 代理相关的包
    "v2ray.com/core/proxy/freedom"  # 导入 freedom 代理相关的包
    "v2ray.com/core/proxy/vmess"  # 导入 vmess 代理相关的包
    "v2ray.com/core/proxy/vmess/inbound"  # 导入 vmess 入站代理相关的包
    "v2ray.com/core/proxy/vmess/outbound"  # 导入 vmess 出站代理相关的包
    "v2ray.com/core/testing/servers/tcp"  # 导入 TCP 服务器测试相关的包
    "v2ray.com/core/testing/servers/udp"  # 导入 UDP 服务器测试相关的包
    "v2ray.com/core/transport/internet"  # 导入网络传输相关的包
    "v2ray.com/core/transport/internet/http"  # 导入 HTTP 传输相关的包
    "v2ray.com/core/transport/internet/tls"  # 导入 TLS 传输相关的包
    "v2ray.com/core/transport/internet/websocket"  # 导入 WebSocket 传输相关的包
)

# 定义测试函数 TestSimpleTLSConnection
func TestSimpleTLSConnection(t *testing.T) {
    # 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,  # 设置消息处理器为 xor
    }
    # 启动 TCP 服务器，获取目标地址和错误信息
    dest, err := tcpServer.Start()
    common.Must(err)  # 如果有错误发生，则终止程序
    defer tcpServer.Close()  # 在函数返回时关闭 TCP 服务器

    # 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    # 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围
                    PortRange: net.SinglePortRange(serverPort),
                    // 设置监听地址
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    // 设置流配置
                    StreamSettings: &internet.StreamConfig{
                        // 设置安全类型为 TLS
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        // 设置安全设置
                        SecuritySettings: []*serial.TypedMessage{
                            // 将 TLS 配置转换为类型化消息
                            serial.ToTypedMessage(&tls.Config{
                                // 设置证书
                                Certificate: []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil))},
                            }),
                        },
                    },
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    // 设置用户配置
                    User: []*protocol.User{
                        {
                            // 设置账户信息
                            Account: serial.ToTypedMessage(&vmess.Account{
                                // 设置用户 ID
                                Id: userID.String(),
                            }),
                        },
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }
    
    // 为客户端选择一个端口
    clientPort := tcp.PickPort()
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置入站代理处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置，包括目标地址、端口和网络类型
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),
                    Port:    uint32(dest.Port),
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        # 设置出站代理处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括服务器地址、端口和用户信息
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
                # 设置发送器配置，包括流设置和安全设置
                SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
                    StreamSettings: &internet.StreamConfig{
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        SecuritySettings: []*serial.TypedMessage{
                            serial.ToTypedMessage(&tls.Config{
                                AllowInsecure: true,
                            }),
                        },
                    },
                }),
            },
        },
    }
    // 使用 serverConfig 和 clientConfig 初始化服务器配置，返回服务器列表和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 在函数返回前关闭所有服务器连接
    defer CloseAllServers(servers)

    // 测试客户端端口是否可以建立 TCP 连接，超时时间为 2 秒
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
        // 如果连接测试失败，输出错误信息并终止测试
        t.Fatal(err)
    }
func TestAutoIssuingCertificate(t *testing.T) {
    // 检查操作系统是否为 Windows，如果是则不支持，直接返回
    if runtime.GOOS == "windows" {
        // Not supported on Windows yet.
        return
    }

    // 检查操作系统架构是否为 arm64，如果是则直接返回
    if runtime.GOARCH == "arm64" {
        return
    }

    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 检查是否有错误发生，如果有则抛出异常
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 生成 CA 证书，设置为根证书，包含数字签名、密钥加密和证书签名的密钥用途
    caCert, err := cert.Generate(nil, cert.Authority(true), cert.KeyUsage(x509.KeyUsageDigitalSignature|x509.KeyUsageKeyEncipherment|x509.KeyUsageCertSign))
    // 检查是否有错误发生，如果有则抛出异常
    common.Must(err)
    // 将 CA 证书转换为 PEM 格式的证书和密钥
    certPEM, keyPEM := caCert.ToPEM()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
}
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围
                    PortRange: net.SinglePortRange(serverPort),
                    // 设置监听地址
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    // 设置流配置
                    StreamSettings: &internet.StreamConfig{
                        // 设置安全类型为 TLS
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        // 设置安全设置
                        SecuritySettings: []*serial.TypedMessage{
                            // 将 TLS 配置转换为类型化消息
                            serial.ToTypedMessage(&tls.Config{
                                // 设置证书
                                Certificate: []*tls.Certificate{{
                                    Certificate: certPEM,
                                    Key:         keyPEM,
                                    Usage:       tls.Certificate_AUTHORITY_ISSUE,
                                }},
                            }),
                        },
                    },
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    // 设置用户配置
                    User: []*protocol.User{
                        {
                            // 设置账户配置
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id: userID.String(),
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
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }
    
    // 选择客户端端口
    clientPort := tcp.PickPort()
    
    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 检查错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)
    
    // 循环进行测试 TCP 连接
    for i := 0; i < 10; i++ {
        // 如果测试 TCP 连接出错，则输出错误信息
        if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
            t.Error(err)
        }
    }
// 测试使用 KCP 协议进行 TLS 传输
func TestTLSOverKCP(t *testing.T) {
    // 创建一个 TCP 服务器对象，并指定消息处理器为 xor
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，并获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := udp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 配置接收器设置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    // 配置流设置
                    StreamSettings: &internet.StreamConfig{
                        Protocol:     internet.TransportProtocol_MKCP,
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        SecuritySettings: []*serial.TypedMessage{
                            // 配置 TLS 安全设置
                            serial.ToTypedMessage(&tls.Config{
                                Certificate: []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil))},
                            }),
                        },
                    },
                }),
                // 配置代理设置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 配置用户账户
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
                // 配置代理设置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置和客户端配置，并获取服务器对象和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 必须处理错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
        t.Error(err)
    }
}

// 测试使用 WebSocket 进行 TLS 传输
func TestTLSOverWebSocket(t *testing.T) {
    // 创建一个 TCP 服务器实例，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    StreamSettings: &internet.StreamConfig{
                        Protocol:     internet.TransportProtocol_WebSocket,
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        SecuritySettings: []*serial.TypedMessage{
                            // 设置 TLS 配置
                            serial.ToTypedMessage(&tls.Config{
                                Certificate: []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil))},
                            }),
                        },
                    },
                }),
                // 设置代理配置
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
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置和客户端配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 创建一个错误组
    var errg errgroup.Group
    // 循环测试 TCP 连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }
    # 如果 errg.Wait() 返回的错误不为空，则执行 t.Error(err)
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
# 定义一个名为TestHTTP2的测试函数
func TestHTTP2(t *testing.T) {
    # 创建一个TCP服务器对象，设置消息处理器为xor
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    # 启动TCP服务器，并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    # 如果有错误发生，则必须处理错误
    common.Must(err)
    # 延迟关闭TCP服务器，确保在函数返回前关闭
    defer tcpServer.Close()

    # 创建一个新的用户ID
    userID := protocol.NewID(uuid.New())
    # 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
}
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围
                    PortRange: net.SinglePortRange(serverPort),
                    // 设置监听地址
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    // 设置流配置
                    StreamSettings: &internet.StreamConfig{
                        // 设置协议为 HTTP
                        Protocol: internet.TransportProtocol_HTTP,
                        // 设置传输配置
                        TransportSettings: []*internet.TransportConfig{
                            {
                                // 设置协议为 HTTP
                                Protocol: internet.TransportProtocol_HTTP,
                                // 设置 HTTP 配置
                                Settings: serial.ToTypedMessage(&http.Config{
                                    // 设置主机
                                    Host: []string{"v2ray.com"},
                                    // 设置路径
                                    Path: "/testpath",
                                }),
                            },
                        },
                        // 设置安全类型为 TLS
                        SecurityType: serial.GetMessageType(&tls.Config{}),
                        // 设置安全配置
                        SecuritySettings: []*serial.TypedMessage{
                            serial.ToTypedMessage(&tls.Config{
                                // 设置证书
                                Certificate: []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil))},
                            }),
                        },
                    },
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    // 设置用户配置
                    User: []*protocol.User{
                        {
                            // 设置账户配置
                            Account: serial.ToTypedMessage(&vmess.Account{
                                // 设置用户 ID
                                Id: userID.String(),
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
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置并返回服务器对象
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    # 确保错误不为空，如果为空则触发 panic
    common.Must(err)
    # 延迟关闭所有服务器连接
    defer CloseAllServers(servers)

    # 创建错误组
    var errg errgroup.Group
    # 循环10次
    for i := 0; i < 10; i++ {
        # 向错误组中添加并发执行的测试TCP连接任务
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*40))
    }
    # 等待所有任务完成并返回错误
    if err := errg.Wait(); err != nil {
        # 如果有错误则在测试中输出错误信息
        t.Error(err)
    }
# 闭合前面的函数定义
```