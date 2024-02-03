# `v2ray-core\app\stats\stats_test.go`

```go
package stats_test

import (
    "context" // 引入上下文包
    "testing" // 引入测试包
    "time" // 引入时间包

    . "v2ray.com/core/app/stats" // 引入统计包
    "v2ray.com/core/common" // 引入公共包
    "v2ray.com/core/features/stats" // 引入统计特性包
)

func TestInterface(t *testing.T) {
    _ = (stats.Manager)(new(Manager)) // 测试 Manager 接口
}

func TestStatsChannelRunnable(t *testing.T) {
    raw, err := common.CreateObject(context.Background(), &Config{}) // 创建配置对象
    common.Must(err) // 检查错误

    m := raw.(stats.Manager) // 类型断言为 Manager 接口

    ch1, err := m.RegisterChannel("test.channel.1") // 注册通道1
    c1 := ch1.(*Channel) // 类型断言为 Channel
    common.Must(err) // 检查错误

    if c1.Running() { // 检查通道1是否在运行
        t.Fatalf("unexpected running channel: test.channel.%d", 1) // 报告错误
    }

    common.Must(m.Start()) // 启动 Manager

    if !c1.Running() { // 检查通道1是否在运行
        t.Fatalf("unexpected non-running channel: test.channel.%d", 1) // 报告错误
    }

    ch2, err := m.RegisterChannel("test.channel.2") // 注册通道2
    c2 := ch2.(*Channel) // 类型断言为 Channel
    common.Must(err) // 检查错误

    if !c2.Running() { // 检查通道2是否在运行
        t.Fatalf("unexpected non-running channel: test.channel.%d", 2) // 报告错误
    }

    s1, err := c1.Subscribe() // 订阅通道1
    common.Must(err) // 检查错误
    common.Must(c1.Close()) // 关闭通道1

    if c1.Running() { // 检查通道1是否在运行
        t.Fatalf("unexpected running channel: test.channel.%d", 1) // 报告错误
    }

    select { // 检查所有订阅者是否在关闭的通道中关闭
    case _, ok := <-s1:
        if ok {
            t.Fatalf("unexpected non-closed subscriber in channel: test.channel.%d", 1) // 报告错误
        }
    case <-time.After(500 * time.Millisecond):
        t.Fatalf("unexpected non-closed subscriber in channel: test.channel.%d", 1) // 报告错误
    }

    if len(c1.Subscribers()) != 0 { // 检查关闭的通道中的订阅者是否为空
        t.Fatalf("unexpected non-empty subscribers in channel: test.channel.%d", 1) // 报告错误
    }

    common.Must(m.Close()) // 关闭 Manager

    if c2.Running() { // 检查通道2是否在运行
        t.Fatalf("unexpected running channel: test.channel.%d", 2) // 报告错误
    }

    ch3, err := m.RegisterChannel("test.channel.3") // 注册通道3
    c3 := ch3.(*Channel) // 类型断言为 Channel
    common.Must(err) // 检查错误

    if c3.Running() { // 检查通道3是否在运行
        t.Fatalf("unexpected running channel: test.channel.%d", 3) // 报告错误
    }

    common.Must(c3.Start()) // 启动通道3
}
    # 调用 m 对象的 UnregisterChannel 方法，注销名为 "test.channel.3" 的通道
    common.Must(m.UnregisterChannel("test.channel.3"))

    # 如果 c3 对象正在运行
    if c3.Running() { // Test that unregistering will close the channel.
        # 抛出致命错误，指出意外的运行中通道："test.channel.3"
        t.Fatalf("unexpected running channel: test.channel.%d", 3)
    }
# 闭合前面的函数定义
```