# `v2ray-core\testing\scenarios\transport_test.go`

```
package scenarios

import (
    "os"  // 导入操作系统相关的包
    "runtime"  // 导入运行时相关的包
    "testing"  // 导入测试相关的包
    "time"  // 导入时间相关的包

    "golang.org/x/sync/errgroup"  // 导入错误组相关的包

    "v2ray.com/core"  // 导入 V2Ray 核心包
    "v2ray.com/core/app/log"  // 导入日志相关的包
    "v2ray.com/core/app/proxyman"  // 导入代理管理相关的包
    "v2ray.com/core/common"  // 导入通用相关的包
    clog "v2ray.com/core/common/log"  // 导入通用日志相关的包
    "v2ray.com/core/common/net"  // 导入网络相关的包
    "v2ray.com/core/common/protocol"  // 导入协议相关的包
    "v2ray.com/core/common/serial"  // 导入序列化相关的包
    "v2ray.com/core/common/uuid"  // 导入 UUID 相关的包
    "v2ray.com/core/proxy/dokodemo"  // 导入 dokodemo 代理相关的包
    "v2ray.com/core/proxy/freedom"  // 导入 freedom 代理相关的包
    "v2ray.com/core/proxy/vmess"  // 导入 vmess 代理相关的包
    "v2ray.com/core/proxy/vmess/inbound"  // 导入 vmess 入站代理相关的包
    "v2ray.com/core/proxy/vmess/outbound"  // 导入 vmess 出站代理相关的包
    "v2ray.com/core/testing/servers/tcp"  // 导入 TCP 服务器相关的包
    "v2ray.com/core/transport/internet"  // 导入网络传输相关的包
    "v2ray.com/core/transport/internet/domainsocket"  // 导入域套接字相关的包
    "v2ray.com/core/transport/internet/headers/http"  // 导入 HTTP 头相关的包
    "v2ray.com/core/transport/internet/headers/wechat"  // 导入微信相关的包
    "v2ray.com/core/transport/internet/quic"  // 导入 QUIC 相关的包
    tcptransport "v2ray.com/core/transport/internet/tcp"  // 导入 TCP 传输相关的包
)

func TestHttpConnectionHeader(t *testing.T) {
    tcpServer := tcp.Server{  // 创建一个 TCP 服务器对象
        MsgProcessor: xor,  // 设置消息处理器为 xor
    }
    dest, err := tcpServer.Start()  // 启动 TCP 服务器，获取目标地址和错误信息
    common.Must(err)  // 如果有错误发生，则必须处理
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    userID := protocol.NewID(uuid.New())  // 创建一个新的 UUID 作为用户 ID
    serverPort := tcp.PickPort()  // 选择一个可用的 TCP 端口
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
                        // 设置传输配置
                        TransportSettings: []*internet.TransportConfig{
                            {
                                // 设置传输协议为 TCP
                                Protocol: internet.TransportProtocol_TCP,
                                // 设置传输配置
                                Settings: serial.ToTypedMessage(&tcptransport.Config{
                                    // 设置头部配置为 HTTP
                                    HeaderSettings: serial.ToTypedMessage(&http.Config{}),
                                }),
                            },
                        },
                    },
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            // 设置用户账号
                            Account: serial.ToTypedMessage(&vmess.Account{
                                // 设置用户ID
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

    // 选择客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 检查错误
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 测试 TCP 连接
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
        // 输出错误信息
        t.Error(err)
    }
func TestDomainSocket(t *testing.T) {
    // 如果运行环境是 Windows，则跳过测试并返回
    if runtime.GOOS == "windows" {
        t.Skip("Not supported on windows")
        return
    }
    // 创建一个 TCP 服务器对象，并设置消息处理器为 xor
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，并获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 定义域套接字路径，并删除已存在的文件
    const dsPath = "/tmp/ds_scenario"
    os.Remove(dsPath)

    // 创建用户 ID 对象
    userID := protocol.NewID(uuid.New())
    // 选择服务器端口
    serverPort := tcp.PickPort()
    // 配置服务器端参数
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器参数
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                    // 设置流配置为域套接字
                    StreamSettings: &internet.StreamConfig{
                        Protocol: internet.TransportProtocol_DomainSocket,
                        TransportSettings: []*internet.TransportConfig{
                            {
                                Protocol: internet.TransportProtocol_DomainSocket,
                                // 设置域套接字配置的路径
                                Settings: serial.ToTypedMessage(&domainsocket.Config{
                                    Path: dsPath,
                                }),
                            },
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
                // 设置代理配置为自由模式
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 选择客户端端口
    clientPort := tcp.PickPort()
    // 初始化服务器配置并获取服务器对象和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
}
    # 确保错误不为空，如果有错误则会触发 panic
    common.Must(err)
    # 延迟执行 CloseAllServers 函数，确保在函数返回前关闭所有服务器
    defer CloseAllServers(servers)

    # 如果测试 TCP 连接出现错误
    if err := testTCPConn(clientPort, 1024, time.Second*2)(); err != nil {
        # 输出错误信息
        t.Error(err)
    }
# 测试 VMess 协议的 QUIC 功能
func TestVMessQuic(t *testing.T):
    # 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    # 启动 TCP 服务器，并返回目标地址和错误信息
    dest, err := tcpServer.Start()
    # 必须处理错误，如果有错误则抛出异常
    common.Must(err)
    # 延迟关闭 TCP 服务器
    defer tcpServer.Close()
    
    # 创建一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    # 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
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
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围为服务器端口
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机IP
                    StreamSettings: &internet.StreamConfig{
                        ProtocolName: "quic",  // 设置协议名称为 QUIC
                        TransportSettings: []*internet.TransportConfig{
                            {
                                ProtocolName: "quic",  // 设置传输协议名称为 QUIC
                                Settings: serial.ToTypedMessage(&quic.Config{
                                    Header: serial.ToTypedMessage(&wechat.VideoConfig{}),  // 设置头部配置为视频配置
                                    Security: &protocol.SecurityConfig{
                                        Type: protocol.SecurityType_NONE,  // 设置安全类型为无加密
                                    },
                                }),
                            },
                        },
                    },
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&inbound.Config{
                    User: []*protocol.User{
                        {
                            Account: serial.ToTypedMessage(&vmess.Account{
                                Id:      userID.String(),  // 设置用户ID
                                AlterId: 64,  // 设置变换ID为64
                            }),
                        },
                    },
                }),
            },
        },
        // 设置出站处理程序配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置代理配置为自由配置
            },
        },
    }

    // 为客户端选择一个端口
    clientPort := tcp.PickPort()
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
# 闭合前面的函数定义
```