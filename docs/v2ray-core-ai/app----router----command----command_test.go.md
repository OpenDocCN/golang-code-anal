# `v2ray-core\app\router\command\command_test.go`

```go
package command_test

import (
    "context" // 引入上下文包
    "testing" // 引入测试包
    "time" // 引入时间包

    "github.com/golang/mock/gomock" // 引入 mock 包
    "github.com/google/go-cmp/cmp" // 引入比较包
    "github.com/google/go-cmp/cmp/cmpopts" // 引入比较包的选项
    "google.golang.org/grpc" // 引入 gRPC 包
    "google.golang.org/grpc/test/bufconn" // 引入 gRPC 测试包
    "v2ray.com/core/app/router" // 引入路由包
    . "v2ray.com/core/app/router/command" // 引入路由命令包
    "v2ray.com/core/app/stats" // 引入统计包
    "v2ray.com/core/common" // 引入通用包
    "v2ray.com/core/common/net" // 引入网络包
    "v2ray.com/core/features/routing" // 引入路由特性包
    "v2ray.com/core/testing/mocks" // 引入测试模拟包
)

func TestServiceSubscribeRoutingStats(t *testing.T) {
    c := stats.NewChannel(&stats.ChannelConfig{ // 创建一个新的统计通道
        SubscriberLimit: 1, // 设置订阅者限制
        BufferSize:      0, // 设置缓冲区大小
        Blocking:        true, // 设置阻塞
    })
    common.Must(c.Start()) // 必须启动统计通道
    defer c.Close() // 延迟关闭统计通道

    lis := bufconn.Listen(1024 * 1024) // 创建一个缓冲连接监听器
    bufDialer := func(context.Context, string) (net.Conn, error) { // 定义一个缓冲拨号函数
        return lis.Dial() // 返回监听器的拨号连接
    }

    testCases := []*RoutingContext{ // 创建路由上下文测试用例数组
        {InboundTag: "in", OutboundTag: "out"}, // 设置入站标签和出站标签
        {TargetIPs: [][]byte{{1, 2, 3, 4}}, TargetPort: 8080, OutboundTag: "out"}, // 设置目标 IP、端口和出站标签
        {TargetDomain: "example.com", TargetPort: 443, OutboundTag: "out"}, // 设置目标域名、端口和出站标签
        {SourcePort: 9999, TargetPort: 9999, OutboundTag: "out"}, // 设置源端口、目标端口和出站标签
        {Network: net.Network_UDP, OutboundGroupTags: []string{"outergroup", "innergroup"}, OutboundTag: "out"}, // 设置网络类型、出站组标签和出站标签
        {Protocol: "bittorrent", OutboundTag: "blocked"}, // 设置协议和出站标签
        {User: "example@v2fly.org", OutboundTag: "out"}, // 设置用户和出站标签
        {SourceIPs: [][]byte{{127, 0, 0, 1}}, Attributes: map[string]string{"attr": "value"}, OutboundTag: "out"}, // 设置源 IP、属性和出站标签
    }
    errCh := make(chan error) // 创建一个错误通道
    nextPub := make(chan struct{}) // 创建一个结构体通道

    // Server goroutine
    go func() { // 启动服务器协程
        server := grpc.NewServer() // 创建 gRPC 服务器
        RegisterRoutingServiceServer(server, NewRoutingServer(nil, c)) // 注册路由服务
        errCh <- server.Serve(lis) // 服务器监听连接
    }()

    // Publisher goroutine
    # 创建一个匿名的 goroutine
    go func() {
        # 定义一个发布测试用例的函数
        publishTestCases := func() error {
            # 使用 WithTimeout 创建一个带有超时的上下文
            ctx, cancel := context.WithTimeout(context.Background(), time.Second)
            # 延迟取消上下文
            defer cancel()
            # 循环直到路由统计通道中有一个订阅者
            for {
                if len(c.Subscribers()) > 0 {
                    break
                }
                # 如果上下文出错，则返回错误
                if ctx.Err() != nil {
                    return ctx.Err()
                }
            }
            # 遍历测试用例，发布到路由
            for _, tc := range testCases {
                c.Publish(context.Background(), AsRoutingRoute(tc))
                # 等待一段时间
                time.Sleep(time.Millisecond)
            }
            return nil
        }

        # 调用发布测试用例的函数，并处理可能出现的错误
        if err := publishTestCases(); err != nil {
            errCh <- err
        }

        # 等待下一轮发布
        <-nextPub

        # 再次调用发布测试用例的函数，并处理可能出现的错误
        if err := publishTestCases(); err != nil {
            errCh <- err
        }
    }()

    # 客户端 goroutine

    # 等待所有 goroutine 完成
    select {
    case <-time.After(2 * time.Second):
        t.Fatal("Test timeout after 2s")
    case err := <-errCh:
        if err != nil {
            t.Fatal(err)
        }
    }
func TestSerivceTestRoute(t *testing.T) {
    // 创建一个具有指定配置的统计通道
    c := stats.NewChannel(&stats.ChannelConfig{
        SubscriberLimit: 1,
        BufferSize:      16,
        Blocking:        true,
    })
    // 必须启动通道
    common.Must(c.Start())
    // 延迟关闭通道
    defer c.Close()

    // 创建一个新的路由器
    r := new(router.Router)
    // 创建一个新的模拟控制器
    mockCtl := gomock.NewController(t)
    // 延迟结束模拟控制器
    defer mockCtl.Finish()
    // 初始化路由器，设置路由规则
    common.Must(r.Init(&router.Config{
        Rule: []*router.RoutingRule{
            {
                InboundTag: []string{"in"},
                TargetTag:  &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                Protocol:  []string{"bittorrent"},
                TargetTag: &router.RoutingRule_Tag{Tag: "blocked"},
            },
            {
                PortList:  &net.PortList{Range: []*net.PortRange{{From: 8080, To: 8080}}},
                TargetTag: &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                SourcePortList: &net.PortList{Range: []*net.PortRange{{From: 9999, To: 9999}}},
                TargetTag:      &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                Domain:    []*router.Domain{{Type: router.Domain_Domain, Value: "com"}},
                TargetTag: &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                SourceGeoip: []*router.GeoIP{{CountryCode: "private", Cidr: []*router.CIDR{{Ip: []byte{127, 0, 0, 0}, Prefix: 8}}}},
                TargetTag:   &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                UserEmail: []string{"example@v2fly.org"},
                TargetTag: &router.RoutingRule_Tag{Tag: "out"},
            },
            {
                Networks:  []net.Network{net.Network_UDP, net.Network_TCP},
                TargetTag: &router.RoutingRule_Tag{Tag: "out"},
            },
        },
    }, mocks.NewDNSClient(mockCtl), mocks.NewOutboundManager(mockCtl)))

    // 在本地创建一个缓冲连接
    lis := bufconn.Listen(1024 * 1024)
}
    // 定义一个函数类型的变量bufDialer，该函数接受一个上下文和一个字符串参数，返回一个网络连接和一个错误
    bufDialer := func(context.Context, string) (net.Conn, error) {
        return lis.Dial()
    }

    // 创建一个用于传递错误信息的通道
    errCh := make(chan error)

    // 服务器协程
    go func() {
        // 创建一个 gRPC 服务器
        server := grpc.NewServer()
        // 注册路由服务
        RegisterRoutingServiceServer(server, NewRoutingServer(r, c))
        // 启动服务器并将错误信息发送到errCh通道
        errCh <- server.Serve(lis)
    }()

    // 客户端协程
    }()

    // 等待协程完成
    select {
    // 等待2秒后超时
    case <-time.After(2 * time.Second):
        t.Fatal("Test timeout after 2s")
    // 从errCh通道接收错误信息
    case err := <-errCh:
        // 如果有错误则输出错误信息
        if err != nil {
            t.Fatal(err)
        }
    }
# 闭合前面的函数定义
```