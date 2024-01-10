# `v2ray-core\app\proxyman\outbound\handler_test.go`

```
package outbound_test

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "v2ray.com/core"  // 导入 v2ray 核心包
    "v2ray.com/core/app/policy"  // 导入策略包
    . "v2ray.com/core/app/proxyman/outbound"  // 导入代理管理包
    "v2ray.com/core/app/stats"  // 导入统计包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/serial"  // 导入序列化包
    "v2ray.com/core/features/outbound"  // 导入出站包
    "v2ray.com/core/proxy/freedom"  // 导入自由代理包
    "v2ray.com/core/transport/internet"  // 导入网络传输包
)

func TestInterfaces(t *testing.T) {
    _ = (outbound.Handler)(new(Handler))  // 测试出站处理器接口
    _ = (outbound.Manager)(new(Manager))  // 测试出站管理器接口
}

const v2rayKey core.V2rayKey = 1  // 定义 v2rayKey 常量

func TestOutboundWithoutStatCounter(t *testing.T) {
    config := &core.Config{  // 创建核心配置对象
        App: []*serial.TypedMessage{  // 配置对象中的应用列表
            serial.ToTypedMessage(&stats.Config{}),  // 统计配置
            serial.ToTypedMessage(&policy.Config{  // 策略配置
                System: &policy.SystemPolicy{  // 系统策略
                    Stats: &policy.SystemPolicy_Stats{  // 系统策略中的统计
                        InboundUplink: true,  // 入站上行统计
                    },
                },
            }),
        },
    }

    v, _ := core.New(config)  // 创建新的 v2ray 实例
    v.AddFeature((outbound.Manager)(new(Manager)))  // 添加出站管理器作为特性
    ctx := context.WithValue(context.Background(), v2rayKey, v)  // 创建带有 v2ray 实例值的上下文
    h, _ := NewHandler(ctx, &core.OutboundHandlerConfig{  // 创建新的出站处理器
        Tag:           "tag",  // 标签
        ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 代理设置
    })
    conn, _ := h.(*Handler).Dial(ctx, net.TCPDestination(net.DomainAddress("localhost"), 13146))  // 拨号连接
    _, ok := conn.(*internet.StatCouterConnection)  // 判断连接是否为统计计数器连接
    if ok {
        t.Errorf("Expected conn to not be StatCouterConnection")  // 如果是统计计数器连接，则输出错误信息
    }
}

func TestOutboundWithStatCounter(t *testing.T) {
    config := &core.Config{  // 创建核心配置对象
        App: []*serial.TypedMessage{  // 配置对象中的应用列表
            serial.ToTypedMessage(&stats.Config{}),  // 统计配置
            serial.ToTypedMessage(&policy.Config{  // 策略配置
                System: &policy.SystemPolicy{  // 系统策略
                    Stats: &policy.SystemPolicy_Stats{  // 系统策略中的统计
                        OutboundUplink:   true,  // 出站上行统计
                        OutboundDownlink: true,  // 出站下行统计
                    },
                },
            }),
        },
    }
    // 创建一个新的核心实例，使用给定的配置
    v, _ := core.New(config)
    // 将新建的 Manager 实例添加到核心实例中
    v.AddFeature((outbound.Manager)(new(Manager)))
    // 使用核心实例创建一个带有特定值的上下文
    ctx := context.WithValue(context.Background(), v2rayKey, v)
    // 使用上下文和特定的配置创建一个新的处理程序实例
    h, _ := NewHandler(ctx, &core.OutboundHandlerConfig{
        Tag:           "tag",
        ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
    })
    // 使用处理程序实例进行拨号操作，返回连接和可能的错误
    conn, _ := h.(*Handler).Dial(ctx, net.TCPDestination(net.DomainAddress("localhost"), 13146))
    // 检查连接是否为 StatCouterConnection 类型
    _, ok := conn.(*internet.StatCouterConnection)
    // 如果不是，则输出错误信息
    if !ok {
        t.Errorf("Expected conn to be StatCouterConnection")
    }
# 闭合前面的函数定义
```