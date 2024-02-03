# `v2ray-core\testing\scenarios\dokodemo_test.go`

```go
package scenarios

import (
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "golang.org/x/sync/errgroup"  // 导入错误组包

    "v2ray.com/core"  // 导入 V2Ray 核心包
    "v2ray.com/core/app/log"  // 导入日志包
    "v2ray.com/core/app/proxyman"  // 导入代理管理包
    "v2ray.com/core/common"  // 导入通用包
    clog "v2ray.com/core/common/log"  // 导入通用日志包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/serial"  // 导入序列化包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
    "v2ray.com/core/proxy/dokodemo"  // 导入 dokodemo 代理包
    "v2ray.com/core/proxy/freedom"  // 导入自由代理包
    "v2ray.com/core/proxy/vmess"  // 导入 VMess 代理包
    "v2ray.com/core/proxy/vmess/inbound"  // 导入入站 VMess 代理包
    "v2ray.com/core/proxy/vmess/outbound"  // 导入出站 VMess 代理包
    "v2ray.com/core/testing/servers/tcp"  // 导入 TCP 服务器测试包
    "v2ray.com/core/testing/servers/udp"  // 导入 UDP 服务器测试包
)

func TestDokodemoTCP(t *testing.T) {
    tcpServer := tcp.Server{  // 创建 TCP 服务器对象
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    dest, err := tcpServer.Start()  // 启动 TCP 服务器，获取目标地址和错误信息
    common.Must(err)  // 如果有错误，立即终止程序
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    userID := protocol.NewID(uuid.New())  // 生成新的用户 ID
    serverPort := tcp.PickPort()  // 选择一个可用的端口
    # 创建服务器配置对象
    serverConfig := &core.Config{
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
                    PortRange: net.SinglePortRange(serverPort),  # 设置端口范围为服务器端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机IP
                }),
                # 设置代理配置，包括用户配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            # 设置用户账号配置，包括用户ID
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id: userID.String(),  # 设置用户ID为字符串形式的用户ID
                            }),
                        },
                    },
                }),
            },
        },
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置，包括自由代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    # 为客户端选择一个端口
    clientPort := uint32(tcp.PickPort())
    # 设置客户端端口范围
    clientPortRange := uint32(5)
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
                    PortRange: &net.PortRange{From: clientPort, To: clientPort + clientPortRange},  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                }),
                // 设置代理配置，包括目标地址、端口和网络列表
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),  // 设置目标地址
                    Port:    uint32(dest.Port),  // 设置目标端口
                    NetworkList: &net.NetworkList{  // 设置网络列表为仅支持TCP
                        Network: []net.Network{net.Network_TCP},
                    },
                }),
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置，包括服务器端点地址、端口和用户账号
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  // 设置服务器端点地址为本地主机IP
                            Port:    uint32(serverPort),  // 设置服务器端点端口
                            User: []*protocol.User{  // 设置用户账号信息
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        Id: userID.String(),  // 设置用户ID
                                    }),
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
    # 使用循环遍历客户端端口范围内的每一个端口
    for port := clientPort; port <= clientPort+clientPortRange; port++ {
        # 调用 testTCPConn 函数测试 TCP 连接，传入端口号、缓冲区大小和超时时间
        if err := testTCPConn(net.Port(port), 1024, time.Second*2)(); err != nil {
            # 如果测试出错，输出错误信息
            t.Error(err)
        }
    }
func TestDokodemoUDP(t *testing.T) {
    // 创建一个 UDP 服务器对象，设置消息处理器为 xor
    udpServer := udp.Server{
        MsgProcessor: xor,
    }
    // 启动 UDP 服务器，获取目标地址和可能的错误
    dest, err := udpServer.Start()
    // 如果有错误发生，立即终止程序
    common.Must(err)
    // 延迟关闭 UDP 服务器连接
    defer udpServer.Close()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 配置服务器端的参数
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器参数，监听指定端口
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理参数，包括用户账号和 ID
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
                // 设置出站代理参数为 freedom.Config
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个可用的客户端端口
    clientPort := uint32(tcp.PickPort())
    // 设置客户端端口范围
    clientPortRange := uint32(5)
}
    # 创建客户端配置对象
    clientConfig := &core.Config{
        # 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                # 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: &net.PortRange{From: clientPort, To: clientPort + clientPortRange},
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                # 设置代理配置
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    Address: net.NewIPOrDomain(dest.Address),
                    Port:    uint32(dest.Port),
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_UDP},
                    },
                }),
            },
        },
        # 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 设置代理配置
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

    # 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    common.Must(err)
    # 延迟关闭所有服务器
    defer CloseAllServers(servers)

    # 创建错误组
    var errg errgroup.Group
    # 循环测试 UDP 连接
    for port := clientPort; port <= clientPort+clientPortRange; port++ {
        errg.Go(testUDPConn(net.Port(port), 1024, time.Second*5))
    }
    # 等待所有测试完成
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
# 闭合前面的函数定义
```