# `v2ray-core\proxy\dns\dns_test.go`

```
package dns_test

import (
    "strconv"  // 导入 strconv 包，用于字符串和数字之间的转换
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"     // 导入 time 包，用于处理时间相关的操作

    "github.com/google/go-cmp/cmp"  // 导入 cmp 包，用于比较两个值是否相等
    "github.com/miekg/dns"          // 导入 dns 包，用于处理 DNS 相关的操作

    "v2ray.com/core"                // 导入 core 包
    "v2ray.com/core/app/dispatcher"  // 导入 dispatcher 包，用于流量分发
    dnsapp "v2ray.com/core/app/dns"  // 导入 dnsapp 包，用于处理 DNS 相关的应用
    "v2ray.com/core/app/policy"     // 导入 policy 包，用于处理策略相关的操作
    "v2ray.com/core/app/proxyman"   // 导入 proxyman 包，用于管理代理相关的操作
    _ "v2ray.com/core/app/proxyman/inbound"  // 导入 inbound 包
    _ "v2ray.com/core/app/proxyman/outbound"  // 导入 outbound 包
    "v2ray.com/core/common"          // 导入 common 包，用于处理通用的操作
    "v2ray.com/core/common/net"      // 导入 net 包，用于处理网络相关的操作
    "v2ray.com/core/common/serial"   // 导入 serial 包，用于处理序列化相关的操作
    dns_proxy "v2ray.com/core/proxy/dns"  // 导入 dns_proxy 包，用于处理 DNS 代理相关的操作
    "v2ray.com/core/proxy/dokodemo"  // 导入 dokodemo 包
    "v2ray.com/core/testing/servers/tcp"  // 导入 tcp 包，用于测试 TCP 服务器
    "v2ray.com/core/testing/servers/udp"  // 导入 udp 包，用于测试 UDP 服务器
)

type staticHandler struct {
}

func (*staticHandler) ServeDNS(w dns.ResponseWriter, r *dns.Msg) {
    ans := new(dns.Msg)  // 创建一个新的 DNS 消息
    ans.Id = r.Id  // 设置消息的 ID

    var clientIP net.IP  // 声明一个变量用于存储客户端 IP 地址

    opt := r.IsEdns0()  // 检查消息是否包含 EDNS0 扩展
    if opt != nil {  // 如果包含 EDNS0 扩展
        for _, o := range opt.Option {  // 遍历扩展选项
            if o.Option() == dns.EDNS0SUBNET {  // 如果是子网选项
                subnet := o.(*dns.EDNS0_SUBNET)  // 将选项转换为子网选项
                clientIP = subnet.Address  // 获取客户端 IP 地址
            }
        }
    }
    # 遍历 DNS 请求中的每个问题
    for _, q := range r.Question {
        # 如果问题的域名是 "google.com." 并且查询类型是 TypeA
        if q.Name == "google.com." && q.Qtype == dns.TypeA {
            # 如果客户端 IP 为空
            if clientIP == nil {
                # 创建一个新的资源记录并添加到回答部分
                rr, _ := dns.NewRR("google.com. IN A 8.8.8.8")
                ans.Answer = append(ans.Answer, rr)
            } else {
                # 创建一个新的资源记录并添加到回答部分
                rr, _ := dns.NewRR("google.com. IN A 8.8.4.4")
                ans.Answer = append(ans.Answer, rr)
            }
        } 
        # 如果问题的域名是 "facebook.com." 并且查询类型是 TypeA
        else if q.Name == "facebook.com." && q.Qtype == dns.TypeA {
            # 创建一个新的资源记录并添加到回答部分
            rr, _ := dns.NewRR("facebook.com. IN A 9.9.9.9")
            ans.Answer = append(ans.Answer, rr)
        } 
        # 如果问题的域名是 "ipv6.google.com." 并且查询类型是 TypeA
        else if q.Name == "ipv6.google.com." && q.Qtype == dns.TypeA {
            # 创建一个新的资源记录并添加到回答部分
            rr, err := dns.NewRR("ipv6.google.com. IN A 8.8.8.7")
            common.Must(err)
            ans.Answer = append(ans.Answer, rr)
        } 
        # 如果问题的域名是 "ipv6.google.com." 并且查询类型是 TypeAAAA
        else if q.Name == "ipv6.google.com." && q.Qtype == dns.TypeAAAA {
            # 创建一个新的资源记录并添加到回答部分
            rr, err := dns.NewRR("ipv6.google.com. IN AAAA 2001:4860:4860::8888")
            common.Must(err)
            ans.Answer = append(ans.Answer, rr)
        } 
        # 如果问题的域名是 "notexist.google.com." 并且查询类型是 TypeAAAA
        else if q.Name == "notexist.google.com." && q.Qtype == dns.TypeAAAA {
            # 设置消息头的响应码为 RcodeNameError
            ans.MsgHdr.Rcode = dns.RcodeNameError
        }
    }
    # 将回答部分写入响应
    w.WriteMsg(ans)
// 测试 UDP DNS 隧道的功能
func TestUDPDNSTunnel(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 创建一个 DNS 服务器对象
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),  // 设置服务器地址
        Net:     "udp",  // 设置网络类型为 UDP
        Handler: &staticHandler{},  // 设置处理程序为静态处理程序
        UDPSize: 1200,  // 设置 UDP 数据包大小
    }
    // 延迟关闭 DNS 服务器
    defer dnsServer.Shutdown()

    // 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    // 等待一秒钟
    time.Sleep(time.Second)

    // 选择另一个可用的端口作为服务器端口
    serverPort := udp.PickPort()
    // 创建配置对象
    config := &core.Config{
        App: []*serial.TypedMessage{
            // 添加 DNS 应用的配置
            serial.ToTypedMessage(&dnsapp.Config{
                NameServers: []*net.Endpoint{
                    {
                        Network: net.Network_UDP,  // 设置网络类型为 UDP
                        Address: &net.IPOrDomain{  // 设置 IP 或域名
                            Address: &net.IPOrDomain_Ip{  // 设置 IP 地址
                                Ip: []byte{127, 0, 0, 1},  // 设置 IP 地址为 127.0.0.1
                            },
                        },
                        Port: uint32(port),  // 设置端口号
                    },
                },
            }),
            serial.ToTypedMessage(&dispatcher.Config{}),  // 添加调度器的配置
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),  // 添加代理管理器的出站配置
            serial.ToTypedMessage(&proxyman.InboundConfig{}),  // 添加代理管理器的入站配置
            serial.ToTypedMessage(&policy.Config{}),  // 添加策略的配置
        },
        Inbound: []*core.InboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{  // 设置代理设置为虚拟入站配置
                    Address:  net.NewIPOrDomain(net.LocalHostIP),  // 设置地址为本地主机 IP
                    Port:     uint32(port),  // 设置端口号
                    Networks: []net.Network{net.Network_UDP},  // 设置网络类型为 UDP
                }),
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{  // 设置接收器配置
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址为本地主机 IP
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&dns_proxy.Config{}),  // 设置代理设置为 DNS 代理配置
            },
        },
    }

    // 创建新的 V2Ray 实例
    v, err := core.New(config)
    common.Must(err)  // 检查错误
    common.Must(v.Start())  // 启动 V2Ray 实例
}
    // 延迟关闭连接
    defer v.Close()

    // 创建新的 DNS 消息对象
    {
        m1 := new(dns.Msg)
        // 生成随机的消息 ID
        m1.Id = dns.Id()
        // 设置递归查询标志
        m1.RecursionDesired = true
        // 创建包含一个问题的问题数组
        m1.Question = make([]dns.Question, 1)
        // 设置问题数组的第一个问题
        m1.Question[0] = dns.Question{Name: "google.com.", Qtype: dns.TypeA, Qclass: dns.ClassINET}

        // 创建 DNS 客户端对象
        c := new(dns.Client)
        // 发送 DNS 查询并接收响应
        in, _, err := c.Exchange(m1, "127.0.0.1:"+strconv.Itoa(int(serverPort)))
        // 检查错误并处理
        common.Must(err)

        // 检查响应中回答的数量
        if len(in.Answer) != 1 {
            t.Fatal("len(answer): ", len(in.Answer))
        }

        // 尝试将回答转换为 A 记录
        rr, ok := in.Answer[0].(*dns.A)
        // 检查转换是否成功
        if !ok {
            t.Fatal("not A record")
        }
        // 检查回答是否符合预期
        if r := cmp.Diff(rr.A[:], net.IP{8, 8, 8, 8}); r != "" {
            t.Error(r)
        }
    }

    // 创建新的 DNS 消息对象
    {
        m1 := new(dns.Msg)
        // 生成随机的消息 ID
        m1.Id = dns.Id()
        // 设置递归查询标志
        m1.RecursionDesired = true
        // 创建包含一个问题的问题数组
        m1.Question = make([]dns.Question, 1)
        // 设置问题数组的第一个问题
        m1.Question[0] = dns.Question{Name: "ipv4only.google.com.", Qtype: dns.TypeAAAA, Qclass: dns.ClassINET}

        // 创建 DNS 客户端对象
        c := new(dns.Client)
        // 设置查询超时时间
        c.Timeout = 10 * time.Second
        // 发送 DNS 查询并接收响应
        in, _, err := c.Exchange(m1, "127.0.0.1:"+strconv.Itoa(int(serverPort)))
        // 检查错误并处理
        common.Must(err)

        // 检查响应中回答的数量
        if len(in.Answer) != 0 {
            t.Fatal("len(answer): ", len(in.Answer))
        }
    }

    // 创建新的 DNS 消息对象
    {
        m1 := new(dns.Msg)
        // 生成随机的消息 ID
        m1.Id = dns.Id()
        // 设置递归查询标志
        m1.RecursionDesired = true
        // 创建包含一个问题的问题数组
        m1.Question = make([]dns.Question, 1)
        // 设置问题数组的第一个问题
        m1.Question[0] = dns.Question{Name: "notexist.google.com.", Qtype: dns.TypeAAAA, Qclass: dns.ClassINET}

        // 创建 DNS 客户端对象
        c := new(dns.Client)
        // 发送 DNS 查询并接收响应
        in, _, err := c.Exchange(m1, "127.0.0.1:"+strconv.Itoa(int(serverPort)))
        // 检查错误并处理
        common.Must(err)

        // 检查响应中的响应码是否为 NameError
        if in.Rcode != dns.RcodeNameError {
            t.Error("expected NameError, but got ", in.Rcode)
        }
    }
}
# 定义一个名为 TestTCPDNSTunnel 的测试函数
func TestTCPDNSTunnel(t *testing.T) {
    # 选择一个可用的 UDP 端口
    port := udp.PickPort()

    # 创建一个 DNS 服务器对象
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),  # 设置服务器地址和端口
        Net:     "udp",  # 设置网络类型为 UDP
        Handler: &staticHandler{},  # 设置处理程序为 staticHandler
    }
    # 延迟关闭 DNS 服务器
    defer dnsServer.Shutdown()

    # 在一个新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    # 等待一秒钟
    time.Sleep(time.Second)

    # 选择一个可用的 TCP 端口
    serverPort := tcp.PickPort()
    # 创建一个配置对象
    config := &core.Config{
        App: []*serial.TypedMessage{
            # 将 DNS 应用的配置信息转换为 TypedMessage
            serial.ToTypedMessage(&dnsapp.Config{
                NameServer: []*dnsapp.NameServer{
                    {
                        Address: &net.Endpoint{
                            Network: net.Network_UDP,  # 设置网络类型为 UDP
                            Address: &net.IPOrDomain{
                                Address: &net.IPOrDomain_Ip{  # 设置 IP 地址
                                    Ip: []byte{127, 0, 0, 1},  # 设置具体的 IP 地址
                                },
                            },
                            Port: uint32(port),  # 设置端口号
                        },
                    },
                },
            }),
            serial.ToTypedMessage(&dispatcher.Config{}),  # 将 dispatcher 的配置信息转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),  # 将 proxyman 的出站配置信息转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.InboundConfig{}),  # 将 proxyman 的入站配置信息转换为 TypedMessage
            serial.ToTypedMessage(&policy.Config{}),  # 将 policy 的配置信息转换为 TypedMessage
        },
        Inbound: []*core.InboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{  # 将 dokodemo 的配置信息转换为 TypedMessage
                    Address:  net.NewIPOrDomain(net.LocalHostIP),  # 设置地址为本地主机 IP
                    Port:     uint32(port),  # 设置端口号
                    Networks: []net.Network{net.Network_TCP},  # 设置网络类型为 TCP
                }),
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{  # 将 proxyman 的接收器配置信息转换为 TypedMessage
                    PortRange: net.SinglePortRange(serverPort),  # 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  # 设置监听地址为本地主机 IP
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&dns_proxy.Config{}),  # 将 DNS 代理的配置信息转换为 TypedMessage
            },
        },
    }

    // 创建一个新的核心对象，根据配置信息
    v, err := core.New(config)
    // 检查错误，如果有错误则终止程序
    common.Must(err)
    // 启动核心对象
    common.Must(v.Start())
    // 延迟关闭核心对象
    defer v.Close()

    // 创建一个新的 DNS 消息对象
    m1 := new(dns.Msg)
    // 设置消息的 ID
    m1.Id = dns.Id()
    // 设置消息的递归查询标志
    m1.RecursionDesired = true
    // 创建一个包含一个问题的问题数组
    m1.Question = make([]dns.Question, 1)
    // 设置问题数组中的问题
    m1.Question[0] = dns.Question{Name: "google.com.", Qtype: dns.TypeA, Qclass: dns.ClassINET}

    // 创建一个 DNS 客户端对象
    c := &dns.Client{
        Net: "tcp",
    }
    // 发送 DNS 消息到指定的服务器地址，并接收响应
    in, _, err := c.Exchange(m1, "127.0.0.1:"+serverPort.String())
    // 检查错误，如果有错误则终止程序
    common.Must(err)

    // 检查响应中的回答数量是否为1，如果不是则终止程序
    if len(in.Answer) != 1 {
        t.Fatal("len(answer): ", len(in.Answer))
    }

    // 将响应中的第一个回答转换为 A 记录类型
    rr, ok := in.Answer[0].(*dns.A)
    // 如果转换失败则终止程序
    if !ok {
        t.Fatal("not A record")
    }
    // 检查 A 记录的 IP 地址是否为 8.8.8.8，如果不是则输出差异信息
    if r := cmp.Diff(rr.A[:], net.IP{8, 8, 8, 8}); r != "" {
        t.Error(r)
    }
# 定义一个测试函数，用于测试 UDP 到 TCP 的 DNS 隧道
func TestUDP2TCPDNSTunnel(t *testing.T) {
    # 选择一个可用的端口
    port := tcp.PickPort()

    # 创建一个 DNS 服务器对象
    dnsServer := dns.Server{
        Addr:    "127.0.0.1:" + port.String(),  # 设置服务器地址和端口
        Net:     "tcp",  # 设置网络类型为 TCP
        Handler: &staticHandler{},  # 设置处理程序为静态处理程序
    }
    # 延迟关闭 DNS 服务器
    defer dnsServer.Shutdown()

    # 在新的 goroutine 中启动 DNS 服务器
    go dnsServer.ListenAndServe()
    # 等待一秒钟
    time.Sleep(time.Second)

    # 选择另一个可用的端口作为服务器端口
    serverPort := tcp.PickPort()
    // 创建一个 core.Config 结构体的指针，并初始化其字段
    config := &core.Config{
        // 设置 App 字段为包含多个 TypedMessage 的切片
        App: []*serial.TypedMessage{
            // 将 dnsapp.Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&dnsapp.Config{
                // 设置 NameServer 字段为包含一个 NameServer 结构体的切片
                NameServer: []*dnsapp.NameServer{
                    {
                        // 设置 Address 字段为 net.Endpoint 结构体
                        Address: &net.Endpoint{
                            // 设置 Network 字段为 net.Network_UDP
                            Network: net.Network_UDP,
                            // 设置 Address 字段为 net.IPOrDomain 结构体
                            Address: &net.IPOrDomain{
                                // 设置 Address 字段为包含本地 IP 地址的 net.IPOrDomain_Ip 结构体
                                Address: &net.IPOrDomain_Ip{
                                    Ip: []byte{127, 0, 0, 1},
                                },
                            },
                            // 设置 Port 字段为指定的端口号
                            Port: uint32(port),
                        },
                    },
                },
            }),
            // 将 dispatcher.Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&dispatcher.Config{}),
            // 将 proxyman.OutboundConfig 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
            // 将 proxyman.InboundConfig 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&proxyman.InboundConfig{}),
            // 将 policy.Config 结构体转换为 TypedMessage，并添加到切片中
            serial.ToTypedMessage(&policy.Config{}),
        },
        // 设置 Inbound 字段为包含一个 InboundHandlerConfig 结构体的切片
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置 ProxySettings 字段为 dokodemo.Config 结构体转换后的 TypedMessage
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    // 设置 Address 字段为本地 IP 地址和指定端口的 net.Endpoint 结构体
                    Address:  net.NewIPOrDomain(net.LocalHostIP),
                    Port:     uint32(port),
                    // 设置 Networks 字段为包含 net.Network_TCP 的切片
                    Networks: []net.Network{net.Network_TCP},
                }),
                // 设置 ReceiverSettings 字段为 proxyman.ReceiverConfig 结构体转换后的 TypedMessage
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置 PortRange 字段为指定的端口范围
                    PortRange: net.SinglePortRange(serverPort),
                    // 设置 Listen 字段为本地 IP 地址的 net.IPOrDomain 结构体
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
            },
        },
        // 设置 Outbound 字段为包含一个 OutboundHandlerConfig 结构体的切片
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置 ProxySettings 字段为 dns_proxy.Config 结构体转换后的 TypedMessage
                ProxySettings: serial.ToTypedMessage(&dns_proxy.Config{
                    // 设置 Server 字段为 net.Endpoint 结构体
                    Server: &net.Endpoint{
                        // 设置 Network 字段为 net.Network_TCP
                        Network: net.Network_TCP,
                    },
                }),
            },
        },
    }

    // 使用配置创建一个新的 core 实例
    v, err := core.New(config)
    // 检查错误并处理
    common.Must(err)
    // 启动 core 实例
    common.Must(v.Start())
    // 延迟关闭 core 实例
    defer v.Close()

    // 创建一个新的 DNS 消息
    m1 := new(dns.Msg)
    // 设置消息的 ID
    m1.Id = dns.Id()
    // 设置消息的递归查询标志为 true
    m1.RecursionDesired = true
    # 创建一个包含一个 DNS 问题的切片
    m1.Question = make([]dns.Question, 1)
    # 设置切片中第一个元素的值为指定的 DNS 问题
    m1.Question[0] = dns.Question{Name: "google.com.", Qtype: dns.TypeA, Qclass: dns.ClassINET}

    # 创建一个 DNS 客户端对象
    c := &dns.Client{
        Net: "tcp",
    }
    # 向指定的 DNS 服务器发送查询请求，并接收响应
    in, _, err := c.Exchange(m1, "127.0.0.1:"+serverPort.String())
    # 检查是否有错误发生，如果有则必须处理
    common.Must(err)

    # 检查响应中回答的数量是否为1，如果不是则输出错误信息并终止测试
    if len(in.Answer) != 1 {
        t.Fatal("len(answer): ", len(in.Answer))
    }

    # 尝试将响应中的第一个回答转换为 DNS A 记录类型，如果失败则输出错误信息并终止测试
    rr, ok := in.Answer[0].(*dns.A)
    if !ok {
        t.Fatal("not A record")
    }
    # 检查 A 记录的值是否与预期的 IP 地址相匹配，如果不匹配则输出差异信息
    if r := cmp.Diff(rr.A[:], net.IP{8, 8, 8, 8}); r != "" {
        t.Error(r)
    }
# 闭合前面的函数定义
```