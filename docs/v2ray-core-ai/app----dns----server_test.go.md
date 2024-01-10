# `v2ray-core\app\dns\server_test.go`

```
package dns_test

import (
    "testing"
    "time"

    "github.com/google/go-cmp/cmp"  // 导入 Google 的 cmp 包，用于比较数据结构
    "github.com/miekg/dns"  // 导入 miekg 的 dns 包，用于处理 DNS 相关操作

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/app/dispatcher"  // 导入 v2ray 核心中的 dispatcher 包
    . "v2ray.com/core/app/dns"  // 导入 v2ray 核心中的 dns 包，并将其命名为当前包的默认命名空间
    "v2ray.com/core/app/policy"  // 导入 v2ray 核心中的 policy 包
    "v2ray.com/core/app/proxyman"  // 导入 v2ray 核心中的 proxyman 包
    _ "v2ray.com/core/app/proxyman/outbound"  // 导入 v2ray 核心中的 proxyman/outbound 包，但不使用它
    "v2ray.com/core/app/router"  // 导入 v2ray 核心中的 router 包
    "v2ray.com/core/common"  // 导入 v2ray 核心中的 common 包
    "v2ray.com/core/common/net"  // 导入 v2ray 核心中的 common/net 包
    "v2ray.com/core/common/serial"  // 导入 v2ray 核心中的 common/serial 包
    feature_dns "v2ray.com/core/features/dns"  // 导入 v2ray 核心中的 features/dns 包，并将其命名为 feature_dns
    "v2ray.com/core/proxy/freedom"  // 导入 v2ray 核心中的 proxy/freedom 包
    "v2ray.com/core/testing/servers/udp"  // 导入 v2ray 核心中的 testing/servers/udp 包
)

type staticHandler struct {
}

func (*staticHandler) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
    ans := new(dns.Msg)  // 创建一个新的 DNS 消息
    ans.Id = r.Id  // 将新消息的 ID 设置为接收到的消息的 ID

    var clientIP net.IP  // 声明一个变量用于存储客户端 IP 地址

    opt := r.IsEdns0()  // 检查接收到的消息是否包含 EDNS0 扩展
    if opt != nil {  // 如果包含 EDNS0 扩展
        for _, o := range opt.Option {  // 遍历所有的扩展选项
            if o.Option() == dns.EDNS0SUBNET {  // 如果扩展选项是 EDNS0SUBNET
                subnet := o.(*dns.EDNS0_SUBNET)  // 将扩展选项转换为 EDNS0_SUBNET 类型
                clientIP = subnet.Address  // 获取客户端 IP 地址
            }
        }
    }

    }
    w.WriteMsg(ans)  // 将构建好的 DNS 消息写回客户端
}

func TestUDPServerSubnet(t *testing.T) {
    port := udp.PickPort()  // 选择一个可用的端口号

    dnsServer := dns.Server{  // 创建一个 DNS 服务器对象
        Addr:    "127.0.0.1:" + port.String(),  // 设置服务器地址为本地地址和选定的端口号
        Net:     "udp",  // 设置服务器网络协议为 UDP
        Handler: &staticHandler{},  // 设置服务器处理程序为 staticHandler
        UDPSize: 1200,  // 设置 UDP 数据包大小
    }

    go dnsServer.ListenAndServe()  // 在新的 goroutine 中启动 DNS 服务器
    time.Sleep(time.Second)  // 等待一秒钟，确保服务器已经启动
}
    # 创建一个 core.Config 结构体的指针，并初始化其字段
    config := &core.Config{
        # 初始化 App 字段，包含多个 serial.TypedMessage 类型的指针
        App: []*serial.TypedMessage{
            # 将 Config 结构体转换为 serial.TypedMessage 类型的指针，并添加到列表中
            serial.ToTypedMessage(&Config{
                # 初始化 NameServers 字段，包含一个 net.Endpoint 类型的指针列表
                NameServers: []*net.Endpoint{
                    {
                        # 初始化 Network 和 Address 字段
                        Network: net.Network_UDP,
                        Address: &net.IPOrDomain{
                            # 初始化 Address 字段，包含一个 net.IPOrDomain_Ip 类型的指针
                            Address: &net.IPOrDomain_Ip{
                                # 初始化 Ip 字段，包含一个字节切片
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        # 初始化 Port 字段
                        Port: uint32(port),
                    },
                },
                # 初始化 ClientIp 字段，包含一个字节切片
                ClientIp: []byte{7, 8, 9, 10},
            }),
            # 将 dispatcher.Config 结构体转换为 serial.TypedMessage 类型的指针，并添加到列表中
            serial.ToTypedMessage(&dispatcher.Config{}),
            # 将 proxyman.OutboundConfig 结构体转换为 serial.TypedMessage 类型的指针，并添加到列表中
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            # 将 policy.Config 结构体转换为 serial.TypedMessage 类型的指针，并添加到列表中
            serial.ToTypedMessage(&policy.Config{}),
        },
        # 初始化 Outbound 字段，包含一个 core.OutboundHandlerConfig 类型的指针列表
        Outbound: []*core.OutboundHandlerConfig{
            {
                # 初始化 ProxySettings 字段，包含一个 freedom.Config 结构体转换为 serial.TypedMessage 类型的指针
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    # 使用配置创建一个 core 实例
    v, err := core.New(config)
    # 检查是否有错误发生
    common.Must(err)

    # 从 core 实例中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    # 查询指定域名的 IP 地址
    ips, err := client.LookupIP("google.com")
    # 检查是否有错误发生
    if err != nil {
        t.Fatal("unexpected error: ", err)
    }

    # 检查查询到的 IP 地址是否符合预期
    if r := cmp.Diff(ips, []net.IP{{8, 8, 4, 4}}); r != "" {
        t.Fatal(r)
    }
func TestUDPServer(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器实例
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),  // 设置服务器地址
        Net:     "udp",  // 设置网络类型为 UDP
        Handler: &staticHandler{},  // 设置处理程序为静态处理程序
        UDPSize: 1200,  // 设置 UDP 数据包大小
    }

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟，确保服务器已经启动
    time.Sleep(time.Second)

    // 创建一个核心配置实例
    config := &core.Config{
        App: []*serial.TypedMessage{
            // 添加应用程序配置
            serial.ToTypedMessage(&Config{
                NameServers: []*net.Endpoint{
                    {
                        Network: net.Network_UDP,  // 设置网络类型为 UDP
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{  // 设置 IP 地址
                                Ip: []byte{127, 0, 0, 1},  // 设置 IP 地址为 127.0.0.1
                            },
                        },
                        Port: uint32(port),  // 设置端口号
                    },
                },
            }),
            // 添加其他类型的配置
            serial.ToTypedMessage(&dispatcher.Config{}),
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            serial.ToTypedMessage(&policy.Config{}),
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置代理配置为自由配置
            },
        },
    }

    // 根据配置创建一个新的实例
    v, err := core.New(config)
    common.Must(err)

    // 从实例中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 测试 DNS 查询功能
    {
        // 查询 google.com 的 IP 地址
        ips, err := client.LookupIP("google.com")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查查询结果是否符合预期
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    // 测试 DNS 查询功能
    {
        // 查询 facebook.com 的 IP 地址
        ips, err := client.LookupIP("facebook.com")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查查询结果是否符合预期
        if r := cmp.Diff(ips, []net.IP{{9, 9, 9, 9}}); r != "" {
            t.Fatal(r)
        }
    }
}
    {
        // 使用客户端查找指定域名的IP地址，如果返回的错误不为空，则测试失败
        _, err := client.LookupIP("notexist.google.com")
        if err == nil {
            t.Fatal("nil error")
        }
        // 如果返回的错误不是预期的NameError，则测试失败
        if r := feature_dns.RCodeFromError(err); r != uint16(dns.RcodeNameError) {
            t.Fatal("expected NameError, but got ", r)
        }
    }

    {
        // 将客户端转换为IPv6查找接口
        clientv6 := client.(feature_dns.IPv6Lookup)
        // 使用IPv6查找接口查找指定域名的IPv6地址，如果返回的错误不是ErrEmptyResponse，则测试失败
        ips, err := clientv6.LookupIPv6("ipv4only.google.com")
        if err != feature_dns.ErrEmptyResponse {
            t.Fatal("error: ", err)
        }
        // 如果返回的IPv6地址列表不为空，则测试失败
        if len(ips) != 0 {
            t.Fatal("ips: ", ips)
        }
    }

    // 关闭DNS服务器
    dnsServer.Shutdown()

    {
        // 使用客户端查找指定域名的IP地址，如果返回的错误不为空，则测试失败
        ips, err := client.LookupIP("google.com")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }
        // 比较返回的IP地址列表和预期的IP地址列表，如果不一致则测试失败
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }
}
# 定义测试函数TestPrioritizedDomain
func TestPrioritizedDomain(t *testing.T) {
    # 选择一个可用的端口
    port := udp.PickPort()

    # 定义DNS服务器对象
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),  # 设置服务器地址
        Net:     "udp",  # 设置网络类型为UDP
        Handler: &staticHandler{},  # 设置处理程序为staticHandler
        UDPSize: 1200,  # 设置UDP数据包大小
    }

    # 启动DNS服务器
    go dnsServer.ListenAndServe()
    # 等待1秒
    time.Sleep(time.Second)
}
    // 创建一个 core.Config 结构体的指针，并初始化其字段
    config := &core.Config{
        // 设置 App 字段为一个包含多个 TypedMessage 的切片
        App: []*serial.TypedMessage{
            // 将 Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&Config{
                // 设置 NameServers 字段为一个包含一个网络端点的切片
                NameServers: []*net.Endpoint{
                    {
                        // 设置网络类型为 UDP，地址为 127.0.0.1，端口为 9999
                        Network: net.Network_UDP,
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        Port: 9999, /* unreachable */
                    },
                },
                // 设置 NameServer 字段为一个包含一个 NameServer 结构体的切片
                NameServer: []*NameServer{
                    {
                        // 设置地址为网络类型为 UDP，地址为 127.0.0.1，端口为变量 port 的值
                        Address: &net.Endpoint{
                            Network: net.Network_UDP,
                            Address: &net.IPOrDomain{
                                Address: &net.IPOrDomain_Ip{
                                    Ip: []byte{127, 0, 0, 1},
                                },
                            },
                            Port: uint32(port),
                        },
                        // 设置 PrioritizedDomain 字段为一个包含一个 PriorityDomain 结构体的切片
                        PrioritizedDomain: []*NameServer_PriorityDomain{
                            {
                                // 设置类型为 DomainMatchingType_Full，域名为 "google.com"
                                Type:   DomainMatchingType_Full,
                                Domain: "google.com",
                            },
                        },
                    },
                },
            }),
            // 将 dispatcher.Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&dispatcher.Config{}),
            // 将 proxyman.OutboundConfig 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            // 将 policy.Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&policy.Config{}),
        },
        // 设置 Outbound 字段为一个包含一个 OutboundHandlerConfig 结构体的切片
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置 ProxySettings 字段为一个 freedom.Config 结构体转换为 TypedMessage
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 使用 config 创建一个 core 对象，并返回给 v，同时检查是否有错误发生
    v, err := core.New(config)
    common.Must(err)

    // 从 v 中获取 feature_dns.ClientType() 对应的特性，并断言为 feature_dns.Client 类型
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 获取当前时间
    startTime := time.Now()
    {
        // 使用客户端查询域名对应的 IP 地址
        ips, err := client.LookupIP("google.com")
        // 如果查询出错，输出错误信息并终止测试
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 比较查询到的 IP 地址和预期的 IP 地址是否一致，如果不一致，输出差异信息并终止测试
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    // 记录结束时间
    endTime := time.Now()
    // 如果开始时间晚于结束时间加上 2 秒，输出错误信息
    if startTime.After(endTime.Add(time.Second * 2)) {
        t.Error("DNS query doesn't finish in 2 seconds.")
    }
// 测试基于 IPv6 的 UDP 服务器
func TestUDPServerIPv6(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器实例
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(), // 设置服务器地址
        Net:     "udp", // 设置网络类型为 UDP
        Handler: &staticHandler{}, // 设置处理程序为静态处理程序
        UDPSize: 1200, // 设置 UDP 数据包大小
    }

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟，确保服务器已经启动
    time.Sleep(time.Second)

    // 创建一个配置实例
    config := &core.Config{
        App: []*serial.TypedMessage{
            // 添加应用配置
            serial.ToTypedMessage(&Config{
                NameServers: []*net.Endpoint{
                    {
                        Network: net.Network_UDP, // 设置网络类型为 UDP
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                Ip: []byte{127, 0, 0, 1}, // 设置 IP 地址
                            },
                        },
                        Port: uint32(port), // 设置端口号
                    },
                },
            }),
            // 添加其他应用配置
            serial.ToTypedMessage(&dispatcher.Config{}),
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            serial.ToTypedMessage(&policy.Config{}),
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}), // 设置代理配置为自由模式
            },
        },
    }

    // 根据配置创建一个新的实例
    v, err := core.New(config)
    common.Must(err)

    // 获取 DNS 客户端实例
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)
    client6 := client.(feature_dns.IPv6Lookup)

    // 测试 IPv6 查询
    {
        ips, err := client6.LookupIPv6("ipv6.google.com") // 查询 IPv6 地址
        if err != nil {
            t.Fatal("unexpected error: ", err) // 如果有错误则输出错误信息
        }

        // 检查查询结果是否符合预期
        if r := cmp.Diff(ips, []net.IP{{32, 1, 72, 96, 72, 96, 0, 0, 0, 0, 0, 0, 0, 0, 136, 136}}); r != "" {
            t.Fatal(r) // 如果结果不符合预期则输出差异信息
        }
    }
}

// 测试静态主机域名
func TestStaticHostDomain(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器实例
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(), // 设置服务器地址
        Net:     "udp", // 设置网络类型为 UDP
        Handler: &staticHandler{}, // 设置处理程序为静态处理程序
        UDPSize: 1200, // 设置 UDP 数据包大小
    }

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟，确保服务器已经启动
    time.Sleep(time.Second)
}
    // 创建一个 core.Config 结构体的指针，用于配置应用程序的各种参数
    config := &core.Config{
        // 设置应用程序的各种消息处理器
        App: []*serial.TypedMessage{
            // 将 Config 结构体转换为 TypedMessage 类型的消息
            serial.ToTypedMessage(&Config{
                // 设置名称服务器的地址和端口
                NameServers: []*net.Endpoint{
                    {
                        // 设置网络类型为 UDP
                        Network: net.Network_UDP,
                        // 设置地址为 127.0.0.1
                        Address: &net.IPOrDomain{
                            Address: &net.IPOrDomain_Ip{
                                Ip: []byte{127, 0, 0, 1},
                            },
                        },
                        // 设置端口号为变量 port 的值
                        Port: uint32(port),
                    },
                },
                // 设置静态主机映射
                StaticHosts: []*Config_HostMapping{
                    {
                        // 设置域名匹配类型为全匹配
                        Type:          DomainMatchingType_Full,
                        // 设置域名为 example.com
                        Domain:        "example.com",
                        // 设置代理域名为 google.com
                        ProxiedDomain: "google.com",
                    },
                },
            }),
            // 将 dispatcher.Config 结构体转换为 TypedMessage 类型的消息
            serial.ToTypedMessage(&dispatcher.Config{}),
            // 将 proxyman.OutboundConfig 结构体转换为 TypedMessage 类型的消息
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            // 将 policy.Config 结构体转换为 TypedMessage 类型的消息
            serial.ToTypedMessage(&policy.Config{}),
        },
        // 设置应用程序的出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理设置为 freedom.Config 结构体转换为 TypedMessage 类型的消息
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 根据配置创建一个新的核心实例
    v, err := core.New(config)
    // 如果创建实例时发生错误，则必须处理该错误
    common.Must(err)

    // 从核心实例中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 查询域名 example.com 对应的 IP 地址
    {
        ips, err := client.LookupIP("example.com")
        // 如果查询过程中发生错误，则输出错误信息并终止测试
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 比较查询到的 IP 地址和预期的 IP 地址是否一致，如果不一致则输出差异信息并终止测试
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    // 关闭 DNS 服务器
    dnsServer.Shutdown()
func TestIPMatch(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器实例
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),
        Net:     "udp",
        Handler: &staticHandler{},
        UDPSize: 1200,
    }

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟
    time.Sleep(time.Second)

    // 创建一个新的核心实例
    v, err := core.New(config)
    // 如果出现错误，立即终止程序
    common.Must(err)

    // 从核心实例中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 记录开始时间
    startTime := time.Now()

    {
        // 查询指定域名的 IP 地址
        ips, err := client.LookupIP("google.com")
        // 如果出现错误，终止程序并输出错误信息
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 比较查询到的 IP 地址和预期的 IP 地址是否一致
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    // 记录结束时间
    endTime := time.Now()
    // 如果 DNS 查询时间超过 2 秒，输出错误信息
    if startTime.After(endTime.Add(time.Second * 2)) {
        t.Error("DNS query doesn't finish in 2 seconds.")
    }
}

func TestLocalDomain(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器实例
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),
        Net:     "udp",
        Handler: &staticHandler{},
        UDPSize: 1200,
    }

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟
    time.Sleep(time.Second)

    // 创建一个新的核心实例
    v, err := core.New(config)
    // 如果出现错误，立即终止程序
    common.Must(err)

    // 从核心实例中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 记录开始时间
    startTime := time.Now()

    { // Will match dotless:
        // 查询指定域名的 IP 地址
        ips, err := client.LookupIP("hostname")
        // 如果出现错误，终止程序并输出错误信息
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 比较查询到的 IP 地址和预期的 IP 地址是否一致
        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
            t.Fatal(r)
        }
    }

    { // Will match domain:local
        // 查询指定域名的 IP 地址
        ips, err := client.LookupIP("hostname.local")
        // 如果出现错误，终止程序并输出错误信息
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 比较查询到的 IP 地址和预期的 IP 地址是否一致
        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
            t.Fatal(r)
        }
    }
}
    { // 匹配静态 IP
        ips, err := client.LookupIP("hostnamestatic")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 53}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配域名替换
        ips, err := client.LookupIP("hostnamealias")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配 dotless:localhost，但不期望 IP: 127.0.0.2, 127.0.0.3，然后在 dotless: 匹配
        ips, err := client.LookupIP("localhost")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 2}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配 dotless:localhost，并期望 IP: 127.0.0.2, 127.0.0.3
        ips, err := client.LookupIP("localhost-a")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 3}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配 dotless:localhost，并期望 IP: 127.0.0.2, 127.0.0.3
        ips, err := client.LookupIP("localhost-b")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 4}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配 dotless:
        ips, err := client.LookupIP("Mijia Cloud")
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        if r := cmp.Diff(ips, []net.IP{{127, 0, 0, 1}}); r != "" {
            t.Fatal(r)
        }
    }

    endTime := time.Now()
    if startTime.After(endTime.Add(time.Second * 2)) {
        t.Error("DNS query doesn't finish in 2 seconds.")
    }
func TestMultiMatchPrioritizedDomain(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),
        Net:     "udp",
        Handler: &staticHandler{},
        UDPSize: 1200,
    }

    // 在一个 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟
    time.Sleep(time.Second)

    // 创建一个新的核心对象
    v, err := core.New(config)
    // 必须处理错误
    common.Must(err)

    // 从核心对象中获取 DNS 客户端
    client := v.GetFeature(feature_dns.ClientType()).(feature_dns.Client)

    // 记录开始时间
    startTime := time.Now()

    { // 匹配服务器 1、2，并且服务器 1 返回期望的 IP 地址
        // 查询域名的 IP 地址
        ips, err := client.LookupIP("google.com")
        // 如果有错误，则报告意外错误
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查返回的 IP 地址是否符合预期
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配服务器 1、2，并且服务器 1 返回意外的 IP 地址，然后服务器 2 返回期望的 IP 地址
        // 将客户端转换为 IPv4 查询客户端
        clientv4 := client.(feature_dns.IPv4Lookup)
        // 查询域名的 IPv4 地址
        ips, err := clientv4.LookupIPv4("ipv6.google.com")
        // 如果有错误，则报告意外错误
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查返回的 IP 地址是否符合预期
        if r := cmp.Diff(ips, []net.IP{{8, 8, 8, 7}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配服务器 3、1、2，并且服务器 3 返回期望的 IP 地址
        // 查询域名的 IP 地址
        ips, err := client.LookupIP("api.google.com")
        // 如果有错误，则报告意外错误
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查返回的 IP 地址是否符合预期
        if r := cmp.Diff(ips, []net.IP{{8, 8, 7, 7}}); r != "" {
            t.Fatal(r)
        }
    }

    { // 匹配服务器 4、3、1、2，并且服务器 4 返回期望的 IP 地址
        // 查询域名的 IP 地址
        ips, err := client.LookupIP("v2.api.google.com")
        // 如果有错误，则报告意外错误
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }

        // 检查返回的 IP 地址是否符合预期
        if r := cmp.Diff(ips, []net.IP{{8, 8, 7, 8}}); r != "" {
            t.Fatal(r)
        }
    }

    // 记录结束时间
    endTime := time.Now()
    // 如果 DNS 查询时间超过 2 秒，则报告错误
    if startTime.After(endTime.Add(time.Second * 2)) {
        t.Error("DNS query doesn't finish in 2 seconds.")
    }
}
```