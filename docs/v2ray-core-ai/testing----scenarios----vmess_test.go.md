# `v2ray-core\testing\scenarios\vmess_test.go`

```go
package scenarios

import (
    "os" // 导入操作系统相关的包
    "testing" // 导入测试相关的包
    "time" // 导入时间相关的包

    "golang.org/x/sync/errgroup" // 导入错误组相关的包
    "v2ray.com/core" // 导入 V2Ray 核心包
    "v2ray.com/core/app/log" // 导入日志相关的包
    "v2ray.com/core/app/proxyman" // 导入代理管理相关的包
    "v2ray.com/core/common" // 导入通用相关的包
    clog "v2ray.com/core/common/log" // 导入日志相关的包并重命名为 clog
    "v2ray.com/core/common/net" // 导入网络相关的包
    "v2ray.com/core/common/protocol" // 导入协议相关的包
    "v2ray.com/core/common/serial" // 导入序列化相关的包
    "v2ray.com/core/common/uuid" // 导入 UUID 相关的包
    "v2ray.com/core/proxy/dokodemo" // 导入 dokodemo 代理相关的包
    "v2ray.com/core/proxy/freedom" // 导入 freedom 代理相关的包
    "v2ray.com/core/proxy/vmess" // 导入 vmess 代理相关的包
    "v2ray.com/core/proxy/vmess/inbound" // 导入 vmess 入站代理相关的包
    "v2ray.com/core/proxy/vmess/outbound" // 导入 vmess 出站代理相关的包
    "v2ray.com/core/testing/servers/tcp" // 导入 TCP 服务器测试相关的包
    "v2ray.com/core/testing/servers/udp" // 导入 UDP 服务器测试相关的包
    "v2ray.com/core/transport/internet" // 导入网络传输相关的包
    "v2ray.com/core/transport/internet/kcp" // 导入 KCP 相关的包
)

func TestVMessDynamicPort(t *testing.T) {
    tcpServer := tcp.Server{ // 创建一个 TCP 服务器对象
        MsgProcessor: xor, // 设置消息处理器为 xor
    }
    dest, err := tcpServer.Start() // 启动 TCP 服务器并获取目标地址和错误信息
    common.Must(err) // 如果有错误则终止程序
    defer tcpServer.Close() // 延迟关闭 TCP 服务器

    userID := protocol.NewID(uuid.New()) // 生成一个新的用户 ID
    serverPort := tcp.PickPort() // 选择一个可用的服务器端口
    }

    clientPort := tcp.PickPort() // 选择一个可用的客户端端口
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置应用程序配置，包括日志配置
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   # 设置错误日志类型为控制台输出
            }),
        },
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置，包括端口范围和监听地址
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                # 设置代理配置，包括目标地址、端口和网络列表
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置目标地址
                    Port:    uint32(dest.Port),  # 设置目标端口
                    NetworkList: &net.NetworkList{  # 设置网络列表为仅支持TCP
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括服务器端点地址、端口和用户账号
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置服务器端点地址为本地主机IP
                            Port:    uint32(serverPort),  # 设置服务器端点端口为服务器端口
                            User: []*protocol.User{  # 设置用户账号信息
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        Id: userID.String(),  # 设置用户ID
                                    }),
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

    # 循环进行TCP连接测试
    for i := 0; i < 10; i++ {
        # 测试TCP连接，检查是否有错误并输出错误信息
        if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
            t.Error(err)
        }
    }
func TestVMessGCM(t *testing.T) {
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

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
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
    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建一个名为clientConfig的指针类型的core.Config结构体
    clientConfig := &core.Config{
        # 将log.Config结构体转换为TypedMessage类型，并添加到App字段中
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志的级别为Debug
                ErrorLogType:  log.LogType_Console,   # 设置错误日志的类型为Console
            }),
        },
        # 将ReceiverConfig和ProxySettings结构体转换为TypedMessage类型，并添加到Inbound字段中
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为clientPort
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置目标地址为dest.Address
                    Port:    uint32(dest.Port),  # 设置目标端口为dest.Port
                    NetworkList: &net.NetworkList{  # 设置网络列表为仅包含TCP网络
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        # 将ProxySettings结构体转换为TypedMessage类型，并添加到Outbound字段中
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{  # 设置接收者的地址为本地主机IP和端口为serverPort
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                            User: []*protocol.User{  # 设置用户账户信息
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{  # 设置账户ID、AlterId和安全配置
                                        Id:      userID.String(),
                                        AlterId: 64,
                                        SecuritySettings: &protocol.SecurityConfig{
                                            Type: protocol.SecurityType_AES128_GCM,  # 设置安全类型为AES128_GCM
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
    // 使用给定的服务器配置和客户端配置初始化服务器，返回服务器列表和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果初始化出错，输出错误信息并终止测试
    if err != nil {
        t.Fatal("Failed to initialize all servers: ", err.Error())
    }
    // 延迟关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组
    var errg errgroup.Group
    // 循环创建10个并发任务，每个任务测试TCP连接
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*40))
    }

    // 等待所有任务完成，如果有错误则输出错误信息
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestVMessGCMReadv(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
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
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 设置用户账户配置
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),
                                AlterId: 64,
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

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建一个名为clientConfig的指针类型的core.Config结构体
    clientConfig := &core.Config{
        # 将log.Config结构体转换为TypedMessage类型，并添加到App字段中
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志级别为Debug
                ErrorLogType:  log.LogType_Console,   # 设置错误日志类型为Console
            }),
        },
        # 将ReceiverConfig和ProxySettings结构体转换为TypedMessage类型，并添加到Inbound字段中
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为clientPort
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置目标地址为dest.Address
                    Port:    uint32(dest.Port),  # 设置目标端口为dest.Port
                    NetworkList: &net.NetworkList{  # 设置网络列表为TCP
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        # 将ProxySettings结构体转换为TypedMessage类型，并添加到Outbound字段中
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{  # 设置接收者地址为本地主机IP和端口为serverPort
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            Port:    uint32(serverPort),
                            User: []*protocol.User{  # 设置用户账号信息
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{  # 设置账号ID、AlterId和安全设置
                                        Id:      userID.String(),
                                        AlterId: 64,
                                        SecuritySettings: &protocol.SecurityConfig{
                                            Type: protocol.SecurityType_AES128_GCM,  # 设置安全类型为AES128_GCM
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
    // 定义环境变量名称
    const envName = "V2RAY_BUF_READV"
    // 设置环境变量的值为 "enable"，并在函数返回时取消设置
    common.Must(os.Setenv(envName, "enable"))
    defer os.Unsetenv(envName)

    // 初始化服务器配置，如果出错则输出错误信息并终止测试
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    if err != nil {
        t.Fatal("Failed to initialize all servers: ", err.Error())
    }
    // 在函数返回时关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组，用于并发执行测试TCP连接
    var errg errgroup.Group
    for i := 0; i < 10; i++ {
        // 并发执行测试TCP连接，每个连接发送10240*1024字节数据，超时时间为40秒
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*40))
    }
    // 等待所有并发操作完成，如果有错误则输出错误信息
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestVMessGCMUDP(t *testing.T) {
    // 创建一个 UDP 服务器对象
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器并获取目标地址和可能的错误
    dest, err := udpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 配置服务器端的参数
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
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

    // 选择一个可用的客户端端口
    clientPort := tcp.PickPort()
}
    // 创建一个名为clientConfig的指针，指向core.Config结构体
    clientConfig := &core.Config{
        // 设置App字段为一个包含log.Config结构体的切片
        App: []*serial.TypedMessage{
            // 将log.Config结构体转换为TypedMessage类型，并添加到切片中
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug, // 设置ErrorLogLevel字段为Severity_Debug
                ErrorLogType:  log.LogType_Console, // 设置ErrorLogType字段为LogType_Console
            }),
        },
        // 设置Inbound字段为一个包含core.InboundHandlerConfig结构体的切片
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置ReceiverSettings字段为一个包含proxyman.ReceiverConfig结构体的TypedMessage类型
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort), // 设置PortRange字段为包含clientPort的单端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP), // 设置Listen字段为本地主机IP的IPOrDomain类型
                }),
                // 设置ProxySettings字段为一个包含dokodemo.Config结构体的TypedMessage类型
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address), // 设置Address字段为目标地址的IPOrDomain类型
                    Port:    uint32(dest.Port), // 设置Port字段为目标端口
                    NetworkList: &net.NetworkList{ // 设置NetworkList字段为包含net.Network_UDP的NetworkList结构体
                        Network: []net.Network{net.Network_UDP}, // 设置Network字段为包含net.Network_UDP的切片
                    },
                }),
            },
        },
        // 设置Outbound字段为一个包含core.OutboundHandlerConfig结构体的切片
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置ProxySettings字段为一个包含outbound.Config结构体的TypedMessage类型
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{ // 设置Receiver字段为包含protocol.ServerEndpoint结构体的切片
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP), // 设置Address字段为本地主机IP的IPOrDomain类型
                            Port:    uint32(serverPort), // 设置Port字段为服务器端口
                            User: []*protocol.User{ // 设置User字段为包含protocol.User结构体的切片
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{ // 设置Account字段为一个包含vmess.Account结构体的TypedMessage类型
                                        Id:      userID.String(), // 设置Id字段为userID的字符串形式
                                        AlterId: 64, // 设置AlterId字段为64
                                        SecuritySettings: &protocol.SecurityConfig{ // 设置SecuritySettings字段为一个包含protocol.SecurityConfig结构体的指针
                                            Type: protocol.SecurityType_AES128_GCM, // 设置Type字段为SecurityType_AES128_GCM
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
    // 使用 serverConfig 和 clientConfig 初始化服务器配置，返回服务器列表和错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 在函数返回前关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组
    var errg errgroup.Group
    // 循环创建 10 个 goroutine，每个 goroutine 测试 UDP 连接
    for i := 0; i < 10; i++ {
        errg.Go(testUDPConn(clientPort, 1024, time.Second*5))
    }
    // 等待所有 goroutine 完成，如果有错误发生，记录到测试日志中
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestVMessChacha20(t *testing.T) {
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

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 将日志配置转换为类型化消息
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围和监听地址
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            Account: serial.ToTypedMessage(&vmess.Account{
                                // 设置用户 ID 和额外 ID
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
    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建一个名为clientConfig的core.Config对象
    clientConfig := &core.Config{
        # 设置App字段为一个包含log.Config对象的数组
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置ErrorLogLevel为Debug级别
                ErrorLogType:  log.LogType_Console,   # 设置ErrorLogType为Console类型
            }),
        },
        # 设置Inbound字段为一个包含core.InboundHandlerConfig对象的数组
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置ReceiverSettings字段为一个包含proxyman.ReceiverConfig对象的TypedMessage
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置PortRange为clientPort的单端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置Listen为本地主机IP
                }),
                # 设置ProxySettings字段为一个包含dokodemo.Config对象的TypedMessage
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  # 设置Address为目标地址
                    Port:    uint32(dest.Port),  # 设置Port为目标端口
                    NetworkList: &net.NetworkList{  # 设置NetworkList为包含TCP网络的NetworkList对象
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        # 设置Outbound字段为一个包含core.OutboundHandlerConfig对象的数组
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置ProxySettings字段为一个包含outbound.Config对象的TypedMessage
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{  # 设置Receiver为包含一个ServerEndpoint对象的数组
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置Address为本地主机IP
                            Port:    uint32(serverPort),  # 设置Port为服务器端口
                            User: []*protocol.User{  # 设置User为包含一个User对象的数组
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{  # 设置Account为包含vmess.Account对象的TypedMessage
                                        Id:      userID.String(),  # 设置Id为userID的字符串表示
                                        AlterId: 64,  # 设置AlterId为64
                                        SecuritySettings: &protocol.SecurityConfig{  # 设置SecuritySettings为包含SecurityType的SecurityConfig对象
                                            Type: protocol.SecurityType_CHACHA20_POLY1305,  # 设置Type为CHACHA20_POLY1305
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
    // 使用 serverConfig 和 clientConfig 初始化服务器配置，并返回服务器列表和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 在函数返回前关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组，用于并发执行 TCP 连接测试
    var errg errgroup.Group
    // 循环创建 10 个并发任务，每个任务测试 TCP 连接
    for i := 0; i < 10; i++ {
        // 将 TCP 连接测试任务添加到错误组中
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }

    // 等待所有并发任务完成，如果有错误发生，记录错误
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
func TestVMessNone(t *testing.T) {
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

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 应用配置
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,
                ErrorLogType:  log.LogType_Console,
            }),
        },
        // 传入连接处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 用户账户配置
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),
                                AlterId: 64,
                            }),
                        },
                    },
                }),
            },
        },
        // 传出连接处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置应用程序配置，包括日志配置
        App: []*serial.TypedMessage{
            serial.ToTypedMessage(&log.Config{
                ErrorLogLevel: clog.Severity_Debug,  # 设置错误日志级别为调试级别
                ErrorLogType:  log.LogType_Console,   # 设置错误日志类型为控制台输出
            }),
        },
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(clientPort),  # 设置端口范围为客户端端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
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
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置服务器地址为本地主机IP
                            Port:    uint32(serverPort),  # 设置服务器端口
                            User: []*protocol.User{
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        Id:      userID.String(),  # 设置用户ID
                                        AlterId: 64,  # 设置AlterId为64
                                        SecuritySettings: &protocol.SecurityConfig{
                                            Type: protocol.SecurityType_NONE,  # 设置安全类型为无
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
    // 使用给定的服务器配置和客户端配置初始化服务器，返回服务器列表和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 在函数返回前关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组，用于并发执行测试TCP连接的任务
    var errg errgroup.Group
    // 循环创建10个并发任务
    for i := 0; i < 10; i++ {
        // 并发执行测试TCP连接的任务，并将任务添加到错误组中
        errg.Go(testTCPConn(clientPort, 1024*1024, time.Second*30))
    }
    // 等待所有并发任务完成，如果有错误发生，则记录错误
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
// 测试 VMessKCP 函数
func TestVMessKCP(t *testing.T) {
    // 创建 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并获取目标地址
    dest, err := tcpServer.Start()
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择服务器端口
    serverPort := udp.PickPort()
    // 创建服务器配置
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 创建日志配置
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
                    StreamSettings: &internet.StreamConfig{
                        Protocol: internet.TransportProtocol_MKCP,
                    },
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

    // 选择客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置并获取服务器对象
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 创建错误组
    var errg errgroup.Group
    // 循环进行 TCP 连接测试
    for i := 0; i < 10; i++ {
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Minute*2))
    }
    // 等待错误组中的所有操作完成
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
}

// 测试 VMessKCPLarge 函数
func TestVMessKCPLarge(t *testing.T) {
    // 创建一个 TCP 服务器实例，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := udp.PickPort()

    // 选择一个可用的客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置和客户端配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)

    // 创建一个错误组，用于并发执行 TCP 连接测试
    var errg errgroup.Group
    for i := 0; i < 2; i++ {
        // 并发执行 TCP 连接测试
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Minute*5))
    }
    // 等待所有并发操作完成，并检查是否有错误发生
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }

    // 延迟执行的匿名函数，用于在一定时间后关闭所有服务器
    defer func() {
        <-time.After(5 * time.Second)
        CloseAllServers(servers)
    }()
}
// 测试 VMessGCMMux 函数
func TestVMessGCMMux(t *testing.T) {
    // 创建 TCP 服务器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器
    dest, err := tcpServer.Start()
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置
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

    // 选择客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 循环测试
    for range "abcd" {
        var errg errgroup.Group
        for i := 0; i < 16; i++ {
            errg.Go(testTCPConn(clientPort, 10240, time.Second*20))
        }
        if err := errg.Wait(); err != nil {
            t.Fatal(err)
        }
        time.Sleep(time.Second)
    }
}

// 测试 VMessGCMMuxUDP 函数
func TestVMessGCMMuxUDP(t *testing.T) {
    // 创建 TCP 服务器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址
    dest, err := tcpServer.Start()
    // 检查错误，如果有错误则触发 panic
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建 UDP 服务器实例
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器并返回目标地址
    udpDest, err := udpServer.Start()
    // 检查错误，如果有错误则触发 panic
    common.Must(err)
    // 延迟关闭 UDP 服务器
    defer udpServer.Close()

    // 创建新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置
    serverConfig := &core.Config{
        App: []*serial.TypedMessage{
            // 将日志配置转换为 TypedMessage
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

    // 选择客户端端口
    clientPort := tcp.PickPort()
    // 选择客户端 UDP 端口
    clientUDPPort := udp.PickPort()

    // 初始化服务器配置和客户端配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 检查错误，如果有错误则触发 panic
    common.Must(err)

    // 循环测试连接
    for range "abcd" {
        // 创建错误组
        var errg errgroup.Group
        // 循环创建 TCP 和 UDP 连接测试任务
        for i := 0; i < 16; i++ {
            errg.Go(testTCPConn(clientPort, 10240, time.Second*20))
            errg.Go(testUDPConn(clientUDPPort, 1024, time.Second*10))
        }
        // 等待所有任务完成
        if err := errg.Wait(); err != nil {
            t.Error(err)
        }
        // 休眠一秒
        time.Sleep(time.Second)
    }
    # 延迟执行的匿名函数，用于在一定时间后关闭所有服务器
    defer func() {
        # 从time包中获取一个channel，5秒后向该channel发送数据
        <-time.After(5 * time.Second)
        # 调用CloseAllServers函数关闭服务器
        CloseAllServers(servers)
    }()
# 闭合前面的函数定义
```