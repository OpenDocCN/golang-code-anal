# `v2ray-core\v2ray_test.go`

```
package core_test

import (
    "testing"

    proto "github.com/golang/protobuf/proto"  // 导入 proto 包
    . "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/app/dispatcher"  // 导入调度器包
    "v2ray.com/core/app/proxyman"  // 导入代理管理包
    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/serial"  // 导入序列化包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
    "v2ray.com/core/features/dns"  // 导入 DNS 功能包
    "v2ray.com/core/features/dns/localdns"  // 导入本地 DNS 功能包
    _ "v2ray.com/core/main/distro/all"  // 导入所有分发包
    "v2ray.com/core/proxy/dokodemo"  // 导入 dokodemo 代理包
    "v2ray.com/core/proxy/vmess"  // 导入 vmess 代理包
    "v2ray.com/core/proxy/vmess/outbound"  // 导入 vmess 出站代理包
    "v2ray.com/core/testing/servers/tcp"  // 导入 TCP 服务器测试包
)

func TestV2RayDependency(t *testing.T) {
    instance := new(Instance)  // 创建 Instance 实例

    wait := make(chan bool, 1)  // 创建一个缓冲大小为 1 的布尔类型通道
    instance.RequireFeatures(func(d dns.Client) {  // 要求实例具有 DNS 客户端功能
        if d == nil {  // 如果 DNS 客户端为空
            t.Error("expected dns client fulfilled, but actually nil")  // 输出错误信息
        }
        wait <- true  // 将 true 写入通道
    })
    instance.AddFeature(localdns.New())  // 添加本地 DNS 功能到实例
    <-wait  // 从通道中读取数据
}

func TestV2RayClose(t *testing.T) {
    port := tcp.PickPort()  // 选择一个可用端口

    userId := uuid.New()  // 生成一个新的 UUID
    // 创建一个 Config 结构体的指针，并初始化其字段
    config := &Config{
        // 初始化 App 字段为包含三个 TypedMessage 的切片
        App: []*serial.TypedMessage{
            // 将 dispatcher.Config 结构体转换为 TypedMessage
            serial.ToTypedMessage(&dispatcher.Config{}),
            // 将 proxyman.InboundConfig 结构体转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.InboundConfig{}),
            // 将 proxyman.OutboundConfig 结构体转换为 TypedMessage
            serial.ToTypedMessage(&proxyman.OutboundConfig{}),
        },
        // 初始化 Inbound 字段为包含一个 InboundHandlerConfig 结构体的切片
        Inbound: []*InboundHandlerConfig{
            {
                // 初始化 ReceiverSettings 字段为包含 ReceiverConfig 结构体的 TypedMessage
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    // 设置 PortRange 字段为单个端口范围
                    PortRange: net.SinglePortRange(port),
                    // 设置 Listen 字段为本地主机 IP 地址
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 初始化 ProxySettings 字段为包含 dokodemo.Config 结构体的 TypedMessage
                ProxySettings: serial.ToTypedMessage(&dokodemo.Config{
                    // 设置 Address 字段为本地主机 IP 地址
                    Address: net.NewIPOrDomain(net.LocalHostIP),
                    // 设置 Port 字段为 0
                    Port:    uint32(0),
                    // 设置 NetworkList 字段为包含 TCP 和 UDP 的网络列表
                    NetworkList: &net.NetworkList{
                        Network: []net.Network{net.Network_TCP, net.Network_UDP},
                    },
                }),
            },
        },
        // 初始化 Outbound 字段为包含一个 OutboundHandlerConfig 结构体的切片
        Outbound: []*OutboundHandlerConfig{
            {
                // 初始化 ProxySettings 字段为包含 outbound.Config 结构体的 TypedMessage
                ProxySettings: serial.ToTypedMessage(&outbound.Config{
                    // 设置 Receiver 字段为包含一个 ServerEndpoint 结构体的切片
                    Receiver: []*protocol.ServerEndpoint{
                        {
                            // 设置 Address 字段为本地主机 IP 地址
                            Address: net.NewIPOrDomain(net.LocalHostIP),
                            // 设置 Port 字段为 0
                            Port:    uint32(0),
                            // 设置 User 字段为包含一个 User 结构体的切片
                            User: []*protocol.User{
                                {
                                    // 设置 Account 字段为包含 vmess.Account 结构体的 TypedMessage
                                    Account: serial.ToTypedMessage(&vmess.Account{
                                        // 设置 Id 字段为 userId 的字符串表示
                                        Id: userId.String(),
                                    }),
                                },
                            },
                        },
                    },
                }),
            },
        },
    }

    // 将 config 结构体序列化为字节数组
    cfgBytes, err := proto.Marshal(config)
    // 检查序列化过程中是否出现错误
    common.Must(err)

    // 启动一个使用 "protobuf" 协议和 cfgBytes 字节数组的实例
    server, err := StartInstance("protobuf", cfgBytes)
    // 检查启动过程中是否出现错误
    common.Must(err)
    // 关闭服务器实例
    server.Close()
# 闭合前面的函数定义
```