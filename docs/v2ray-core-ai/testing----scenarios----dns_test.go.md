# `v2ray-core\testing\scenarios\dns_test.go`

```go
# 导入所需的包
package scenarios

import (
    "fmt"  # 导入 fmt 包，用于格式化输出
    "testing"  # 导入 testing 包，用于编写测试函数
    "time"  # 导入 time 包，用于处理时间相关操作

    xproxy "golang.org/x/net/proxy"  # 导入 x/net/proxy 包的别名 xproxy
    "v2ray.com/core"  # 导入 v2ray 核心包
    "v2ray.com/core/app/dns"  # 导入 DNS 应用包
    "v2ray.com/core/app/proxyman"  # 导入代理管理应用包
    "v2ray.com/core/app/router"  # 导入路由应用包
    "v2ray.com/core/common"  # 导入通用包
    "v2ray.com/core/common/net"  # 导入网络通用包
    "v2ray.com/core/common/serial"  # 导入序列化通用包
    "v2ray.com/core/proxy/blackhole"  # 导入黑洞代理包
    "v2ray.com/core/proxy/freedom"  # 导入自由代理包
    "v2ray.com/core/proxy/socks"  # 导入 SOCKS 代理包
    "v2ray.com/core/testing/servers/tcp"  # 导入 TCP 服务器测试包
)

# 定义测试函数 TestResolveIP
func TestResolveIP(t *testing.T) {
    # 创建一个 TCP 服务器对象 tcpServer
    tcpServer := tcp.Server{
        MsgProcessor: xor,  # 设置消息处理器为 xor
    }
    # 启动 TCP 服务器，获取目标地址和错误信息
    dest, err := tcpServer.Start()
    common.Must(err)  # 如果有错误发生，则终止程序
    defer tcpServer.Close()  # 在函数返回前关闭 TCP 服务器

    # 选择一个可用的端口号并赋值给 serverPort
    serverPort := tcp.PickPort()
    // 创建一个服务器配置对象
    serverConfig := &core.Config{
        // 设置应用程序配置
        App: []*serial.TypedMessage{
            // 将 DNS 配置转换为 TypedMessage
            serial.ToTypedMessage(&dns.Config{
                // 设置主机名到 IP 或域名的映射
                Hosts: map[string]*net.IPOrDomain{
                    "google.com": net.NewIPOrDomain(dest.Address),
                },
            }),
            // 将路由配置转换为 TypedMessage
            serial.ToTypedMessage(&router.Config{
                // 设置域名策略为 IPIfNonMatch
                DomainStrategy: router.Config_IpIfNonMatch,
                // 设置路由规则
                Rule: []*router.RoutingRule{
                    {
                        // 设置 CIDR 地址
                        Cidr: []*router.CIDR{
                            {
                                Ip:     []byte{127, 0, 0, 0},
                                Prefix: 8,
                            },
                        },
                        // 设置目标标签为 "direct"
                        TargetTag: &router.RoutingRule_Tag{
                            Tag: "direct",
                        },
                    },
                },
            }),
        },
        // 设置入站处理程序配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置端口范围
                    PortRange: net.SinglePortRange(serverPort),
                    // 设置监听地址为本地主机
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&socks.ServerConfig{
                    // 设置认证类型为无认证
                    AuthType: socks.AuthType_NO_AUTH,
                    // 设置账户映射
                    Accounts: map[string]string{
                        "Test Account": "Test Password",
                    },
                    // 设置地址为本地主机
                    Address:    net.NewIPOrDomain(net.LocalHostIP),
                    // 禁用 UDP
                    UdpEnabled: false,
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
                // 设置标签为 "direct"
                Tag: "direct",
                // 设置代理配置为自由
                ProxySettings: serial.ToTypedMessage(&freedom.Config{
                    // 设置域名策略为 USE_IP
                    DomainStrategy: freedom.Config_USE_IP,
                }),
            },
        },
    }
    // 初始化服务器配置并返回服务器列表和错误信息
    servers, err := InitializeServerConfigs(serverConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭所有服务器连接
    defer CloseAllServers(servers)

    {
        // 创建一个不需要认证的拨号器，并返回拨号器和错误信息
        noAuthDialer, err := xproxy.SOCKS5("tcp", net.TCPDestination(net.LocalHostIP, serverPort).NetAddr(), nil, xproxy.Direct)
        // 如果有错误发生，则必须处理
        common.Must(err)
        // 使用拨号器连接到指定地址，并返回连接和错误信息
        conn, err := noAuthDialer.Dial("tcp", fmt.Sprintf("google.com:%d", dest.Port))
        // 如果有错误发生，则必须处理
        common.Must(err)
        // 延迟关闭连接
        defer conn.Close()

        // 测试连接的读写能力，如果失败则记录错误
        if err := testTCPConn2(conn, 1024, time.Second*5)(); err != nil {
            t.Error(err)
        }
    }
# 闭合前面的函数定义
```