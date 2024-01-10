# `v2ray-core\testing\scenarios\policy_test.go`

```
package scenarios

import (
    "io"
    "testing"
    "time"

    "golang.org/x/sync/errgroup"

    "v2ray.com/core"
    "v2ray.com/core/app/log"
    "v2ray.com/core/app/policy"
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
)

// startQuickClosingTCPServer 启动一个快速关闭的 TCP 服务器
func startQuickClosingTCPServer() (net.Listener, error) {
    // 在随机端口上监听 TCP 连接
    listener, err := net.Listen("tcp", "127.0.0.1:0")
    if err != nil {
        return nil, err
    }
    // 启动一个 goroutine 处理连接
    go func() {
        for {
            // 接受连接
            conn, err := listener.Accept()
            if err != nil {
                break
            }
            // 读取数据并立即关闭连接
            b := make([]byte, 1024)
            conn.Read(b)
            conn.Close()
        }
    }()
    return listener, nil
}

// TestVMessClosing 测试 VMess 连接关闭
func TestVMessClosing(t *testing.T) {
    // 启动快速关闭的 TCP 服务器
    tcpServer, err := startQuickClosingTCPServer()
    common.Must(err)
    defer tcpServer.Close()

    // 获取 TCP 服务器的目标地址
    dest := net.DestinationFromAddr(tcpServer.Addr())

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置应用配置
        App: []*serial.TypedMessage{
            // 将策略配置转换为 TypedMessage
            serial.ToTypedMessage(&policy.Config{
                // 设置级别对应的策略
                Level: map[uint32]*policy.Policy{
                    0: {
                        // 设置超时策略
                        Timeout: &policy.Policy_Timeout{
                            UplinkOnly:   &policy.Second{Value: 0},
                            DownlinkOnly: &policy.Second{Value: 0},
                        },
                    },
                },
            }),
        },
        // 设置入站处理器配置
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
                            // 设置用户账号
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),
                                AlterId: 64,
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

    // 选择客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 检查错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != io.EOF {
        // 输出错误信息
        t.Error(err)
    }
func TestZeroBuffer(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址
    dest, err := tcpServer.Start()
    // 检查错误，如果有错误则终止程序
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置
    serverConfig := &core.Config{
        // 应用配置
        App: []*serial.TypedMessage{
            // 序列化并转换为消息类型的策略配置
            serial.ToTypedMessage(&policy.Config{
                // 配置等级映射到策略对象
                Level: map[uint32]*policy.Policy{
                    0: {
                        // 超时配置
                        Timeout: &policy.Policy_Timeout{
                            UplinkOnly:   &policy.Second{Value: 0},
                            DownlinkOnly: &policy.Second{Value: 0},
                        },
                        // 缓冲配置
                        Buffer: &policy.Policy_Buffer{
                            Connection: 0,
                        },
                    },
                },
            }),
        },
        // 入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 接收器设置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 代理设置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 用户账户设置
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),
                                AlterId: 64,
                            }),
                        },
                    },
                }),
            },
        },
        // 出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 代理设置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择一个客户端端口
    clientPort := tcp.PickPort()
}
    # 创建一个客户端配置对象
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
                # 设置代理配置，包括服务器端点地址、端口和用户账户信息
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            Address: net.NewIPOrDomain(net.LocalHostIP),  # 设置服务器端点地址为本地主机IP
                            Port:    uint32(serverPort),  # 设置服务器端点端口为服务器端口
                            User: []*protocol.User{  # 设置用户账户信息，包括ID、AlterId和安全配置
                                {
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        Id:      userID.String(),  # 设置用户ID
                                        AlterId: 64,  # 设置AlterId为64
                                        SecuritySettings: &protocol.SecurityConfig{  # 设置安全配置，使用AES128_GCM加密
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
    // 使用给定的服务器配置和客户端配置初始化服务器，返回服务器列表和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 必须处理错误
    common.Must(err)
    // 延迟关闭所有服务器连接
    defer CloseAllServers(servers)

    // 创建一个错误组
    var errg errgroup.Group
    // 循环创建并发测试TCP连接
    for i := 0; i < 10; i++ {
        // 将测试TCP连接的错误添加到错误组
        errg.Go(testTCPConn(clientPort, 10240*1024, time.Second*20))
    }
    // 等待所有并发操作完成，如果有错误则记录到测试错误中
    if err := errg.Wait(); err != nil {
        t.Error(err)
    }
# 闭合前面的函数定义
```