# `v2ray-core\testing\scenarios\shadowsocks_test.go`

```
package scenarios

import (
    "crypto/rand"  // 导入加密随机数生成包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "github.com/google/go-cmp/cmp"  // 导入 Google 的比较包
    "golang.org/x/sync/errgroup"  // 导入错误组包

    "v2ray.com/core"  // 导入 V2Ray 核心包
    "v2ray.com/core/app/log"  // 导入日志应用包
    "v2ray.com/core/app/proxyman"  // 导入代理管理应用包
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/errors"  // 导入错误包
    clog "v2ray.com/core/common/log"  // 导入通用日志包
    "v2ray.com/core/common/net"  // 导入网络通用包
    "v2ray.com/core/common/protocol"  // 导入协议通用包
    "v2ray.com/core/common/serial"  // 导入序列化通用包
    "v2ray.com/core/proxy/dokodemo"  // 导入 dokodemo 代理包
    "v2ray.com/core/proxy/freedom"  // 导入 freedom 代理包
    "v2ray.com/core/proxy/shadowsocks"  // 导入 shadowsocks 代理包
    "v2ray.com/core/testing/servers/tcp"  // 导入 TCP 服务器测试包
    "v2ray.com/core/testing/servers/udp"  // 导入 UDP 服务器测试包
)

func TestShadowsocksAES256TCP(t *testing.T) {
    tcpServer := tcp.Server{  // 创建 TCP 服务器对象
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    dest, err := tcpServer.Start()  // 启动 TCP 服务器，获取目标地址和错误信息
    common.Must(err)  // 如果有错误，立即终止程序
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    account := serial.ToTypedMessage(&shadowsocks.Account{  // 创建 shadowsocks 账户对象
        Password:   "shadowsocks-password",  // 设置密码
        CipherType: shadowsocks.CipherType_AES_256_CFB,  // 设置加密类型
    })

    serverPort := tcp.PickPort()  // 选择一个可用的 TCP 端口
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   // 设置错误日志类型为控制台输出
            }),
        },
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围为服务器端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,  // 设置用户账户
                        Level:   1,        // 设置用户级别为1
                    },
                    Network: []net.Network{net.Network_TCP},  // 设置网络类型为TCP
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 使用自由配置
            },
        },
    }

    // 为客户端选择一个可用端口
    clientPort := tcp.PickPort()
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   // 设置错误日志类型为控制台输出
            }),
        },
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  // 设置目标地址
                    Port:    uint32(dest.Port),  // 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP},  // 设置网络类型为TCP
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  // 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  // 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: account,  // 设置用户账户信息
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
    common.Must(err)  // 检查错误
    defer CloseAllServers(servers)  // 延迟关闭所有服务器

    // 创建错误组
    var errg errgroup.Group
    // 循环进行测试TCP连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))  // 测试TCP连接
    }
    // 等待所有测试完成
    if err := errg.Wait(); err != nil {
        t.Fatal(err)  // 如果有错误则输出错误信息
    }
func TestShadowsocksAES128UDP(t *testing.T) {
    // 创建一个 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器并返回目标地址和可能的错误
    dest, err := udpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 创建一个 Shadowsocks 账户对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",
        CipherType: shadowsocks.CipherType_AES_128_CFB,
    })

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建一个核心配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage 对象
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,
                        Level:   1,
                    },
                    Network: []net.Network{net.Network_UDP},
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

    // 选择一个可用的客户端端口
    clientPort := tcp.PickPort()
}
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   // 设置错误日志类型为控制台输出
            }),
        },
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  // 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  // 设置目标地址
                    Port:    uint32(dest.Port),  // 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_UDP},  // 设置网络类型为UDP
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  // 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  // 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: account,  // 设置用户账户信息
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
    common.Must(err)  // 检查错误
    defer CloseAllServers(servers)  // 延迟关闭所有服务器

    var errg errgroup.Group  // 创建错误组
    # 循环10次，每次执行以下操作
    for i := 0; i < 10; i++ {
        # 使用 errgroup.Go 方法并发执行以下函数
        errg.Go(func() error {
            # 使用 UDP 协议连接到指定 IP 和端口
            conn, err := net.DialUDP("udp", nil, &net.UDPAddr{
                IP:   []byte{127, 0, 0, 1},
                Port: int(clientPort),
            })
            # 如果连接出现错误，则返回错误
            if err != nil {
                return err
            }
            # 延迟关闭连接
            defer conn.Close()

            # 创建一个长度为1024的字节切片
            payload := make([]byte, 1024)
            # 读取随机字节到 payload 中
            common.Must2(rand.Read(payload))

            # 向连接中写入 payload 数据
            nBytes, err := conn.Write([]byte(payload))
            # 如果写入出现错误，则返回错误
            if err != nil {
                return err
            }
            # 如果实际写入的字节数不等于 payload 的长度，则返回错误
            if nBytes != len(payload) {
                return errors.New("expect ", len(payload), " written, but actually ", nBytes)
            }

            # 从连接中读取数据，超时时间为5秒，最大读取字节数为1024
            response := readFrom(conn, time.Second*5, 1024)
            # 如果读取的数据与 payload 经过异或运算的结果不一致，则返回错误
            if r := cmp.Diff(response, xor(payload)); r != "" {
                return errors.New(r)
            }
            # 返回空错误，表示操作成功
            return nil
        })
    }

    # 等待并发操作完成，如果出现错误则终止测试
    if err := errg.Wait(); err != nil {
        t.Fatal(err)
    }
func TestShadowsocksChacha20TCP(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 必须处理错误
    common.Must(err)

    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个 Shadowsocks 账号对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",
        CipherType: shadowsocks.CipherType_CHACHA20_IETF,
    })

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 创建日志配置对象
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        Inbound: []*core.InboundHandlerConfig{
            {
                // 创建接收器配置对象
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 创建代理配置对象
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,
                        Level:   1,
                    },
                    Network: []net.Network{net.Network_TCP},
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 创建自由代理配置对象
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置应用程序配置
        App: []*serial.TypedMessage{
            # 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,  # 设置错误日志类型为控制台输出
            }),
        },
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置目标地址
                    Port:    uint32(dest.Port),  # 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP},  # 设置网络类型为TCP
                    },
                }),
            },
        },
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  # 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: account,  # 设置用户账户信息
                                },
                            },
                        },
                    },
                }),
            },
        },
    }

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)  # 检查错误
    defer CloseAllServers(servers)  # 延迟关闭所有服务器

    # 创建错误组
    var errg errgroup.Group
    # 循环进行测试TCP连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*40))  # 进行TCP连接测试
    }

    # 等待所有任务完成
    if err := errg.Wait(); err != nil {
        t.Error(err)  # 输出错误信息
    }
func TestShadowsocksChacha20Poly1305TCP(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 TCP 服务器并返回目标地址
    dest, err := tcpServer.Start()
    common.Must(err)  // 检查错误
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    // 创建一个 Shadowsocks 账号对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",  // 设置密码
        CipherType: shadowsocks.CipherType_CHACHA20_POLY1305,  // 设置加密类型
    })

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
                // 设置代理配置为 Shadowsocks 服务器配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,  // 设置账号
                        Level:   1,  // 设置级别
                    },
                    Network: []net.Network{net.Network_TCP},  // 设置网络类型为 TCP
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置为自由配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
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
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括服务器地址、端口和用户信息
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                            User: []*protocol.User{
                                {
                                    Account: account,
                                },
                            },
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

    # 创建错误组
    var errg errgroup.Group
    # 循环创建并发测试 TCP 连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }
    # 等待所有任务完成
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestShadowsocksAES256GCMTCP(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个 Shadowsocks 账户对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",
        CipherType: shadowsocks.CipherType_AES_256_GCM,
    })

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage 对象
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,
                        Level:   1,
                    },
                    Network: []net.Network{net.Network_TCP},
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

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置应用配置，包括日志配置
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        # 设置入站配置，包括接收器和代理配置
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
        # 设置出站配置，包括代理配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                            User: []*protocol.User{
                                {
                                    Account: account,
                                },
                            },
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

    # 创建错误组
    var errg errgroup.Group
    # 循环创建并发测试 TCP 连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }

    # 等待所有并发操作完成
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestShadowsocksAES128GCMUDP(t *testing.T) {
    // 创建一个 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器并获取目标地址和可能的错误
    dest, err := udpServer.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 创建一个 Shadowsocks 账户对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",
        CipherType: shadowsocks.CipherType_AES_128_GCM,
    })

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage 对象
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,
                        Level:   1,
                    },
                    Network: []net.Network{net.Network_UDP},
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

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置应用程序配置
        App: []*serial.TypedMessage{
            # 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   # 设置错误日志类型为控制台输出
            }),
        },
        # 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置目标地址
                    Port:    uint32(dest.Port),  # 设置目标端口
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_UDP},  # 设置网络类型为UDP
                    },
                }),
            },
        },
        # 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  # 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: account,  # 设置用户账户信息
                                },
                            },
                        },
                    },
                }),
            },
        },
    }

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)  # 检查错误
    defer CloseAllServers(servers)  # 延迟关闭所有服务器

    # 创建错误组
    var errg errgroup.Group
    # 循环进行UDP连接测试
    for i := 0; i < 10; i++ {
        errg.Go(testUDPConn(clientPort, 1024, time.Second*5))  # 启动UDP连接测试
    }
    # 等待所有测试完成
    if err := errg.Wait(); err != nil {
        t.Error(err)  # 输出错误信息
    }
func TestShadowsocksAES128GCMUDPMux(t *testing.T) {
    // 创建一个 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    // 启动 UDP 服务器并获取目标地址
    dest, err := udpServer.Start()
    common.Must(err)  // 必须处理错误
    defer udpServer.Close()  // 延迟关闭 UDP 服务器

    // 创建一个 Shadowsocks 账号对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",  // 设置密码
        CipherType: shadowsocks.CipherType_AES_128_GCM,  // 设置加密类型为 AES-128-GCM
    })

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试
                ErrorLogType:  log.LogType_Console,  // 设置错误日志类型为控制台输出
            }),
        },
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围为单一端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机
                }),
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,  // 设置账号信息
                        Level:   1,  // 设置级别为 1
                    },
                    Network: []net.Network{net.Network_TCP},  // 设置网络类型为 TCP
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置自由代理配置
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    // 创建客户端配置对象
    clientConfig := &core.Config{
        // 设置应用程序配置，包括日志配置
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  // 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   // 设置错误日志类型为控制台输出
            }),
        },
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  // 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                }),
                // 设置代理配置，包括目标地址、端口和网络列表
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  // 设置目标地址
                    Port:    uint32(dest.Port),  // 设置目标端口
                    NetworkList: &net.NetworkList{  // 设置网络列表为UDP
                        Network: []net.Network{net.Network_UDP},
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置发送器配置，包括多路复用配置
                SenderSettings: serial.ToTypedMessage(&proxyman.SenderConfig{
                    MultiplexSettings: &proxyman.MultiplexingConfig{
                        Enabled:     true,  // 启用多路复用
                        Concurrency: 8,     // 设置并发数为8
                    },
                }),
                // 设置代理配置，包括服务器端点和用户信息
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  // 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  // 设置服务器端口为指定端口
                            User: []*protocol.User{  // 设置用户信息
                                {
                                    Account: account,  // 设置账户信息
                                },
                            },
                        },
                    },
                }),
            },
        },
    }

    // 初始化服务器配置并返回服务器列表
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 检查错误并处理
    common.Must(err)
    // 延迟关闭所有服务器连接
    defer CloseAllServers(servers)
    # 创建一个错误组，用于并发执行多个测试UDP连接的任务
    var errg errgroup.Group
    # 循环10次，每次执行testUDPConn函数并将其加入错误组
    for i := 0; i < 10; i++ {
        errg.Go(testUDPConn(clientPort, 1024, time.Second*5))
    }
    # 等待所有任务完成，并检查是否有错误发生
    if err := errg.Wait(); err != nil {
        # 如果有错误发生，则在测试中输出错误信息
        t.Error(err)
    }
func TestShadowsocksNone(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址和错误信息
    dest, err := tcpServer.Start()
    // 必须处理错误
    common.Must(err)

    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个 Shadowsocks 账户对象
    account := serial.ToTypedMessage(&shadowsocks.Account{
        Password:   "shadowsocks-password",
        CipherType: shadowsocks.CipherType_NONE,
    })

    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置为 Shadowsocks 服务器配置
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ServerConfig{
                    User: &protocol.User{
                        Account: account,
                        Level:   1,
                    },
                    Network: []net.Network{net.Network_TCP},
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置为自由代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
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
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括服务器地址、端口和用户信息
                ProxySettings: serial.ToTypedMessage(&shadowsocks.ClientConfig{
                    Server: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                            User: []*protocol.User{
                                {
                                    Account: account,
                                },
                            },
                        },
                    },
                }),
            },
        },
    }

    # 初始化服务器配置并返回服务器对象列表
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    # 检查错误并处理
    common.Must(err)

    # 延迟关闭所有服务器连接
    defer CloseAllServers(servers)

    # 创建错误组
    var errg errgroup.Group
    # 循环创建并发测试 TCP 连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }

    # 等待所有测试完成并检查错误
    if err := errg.Wait(); err != nil {
        t.Fatal(err)
    }
# 闭合函数定义的大括号
```