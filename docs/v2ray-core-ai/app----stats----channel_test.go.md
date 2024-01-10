# `v2ray-core\app\stats\channel_test.go`

```
package stats_test

import (
    "context" // 上下文包，用于传递请求范围的数据
    "fmt" // 格式化包，用于格式化输出
    "testing" // 测试包，用于编写单元测试
    "time" // 时间包，用于处理时间相关操作

    . "v2ray.com/core/app/stats" // 导入 stats 包并使用 . 符号，表示在后续代码中可以直接使用 stats 包里的函数和变量
    "v2ray.com/core/common" // 导入 common 包
    "v2ray.com/core/features/stats" // 导入 stats 包里的 features 子包
)

func TestStatsChannel(t *testing.T) {
    // At most 2 subscribers could be registered
    c := NewChannel(&ChannelConfig{SubscriberLimit: 2, Blocking: true}) // 创建一个新的 Channel 对象，设置订阅者数量上限为 2，启用阻塞模式

    a, err := stats.SubscribeRunnableChannel(c) // 通过 stats 包里的 SubscribeRunnableChannel 函数订阅一个可运行的 Channel
    common.Must(err) // 如果 err 不为 nil，则触发 panic

    if !c.Running() { // 如果 Channel 不处于运行状态
        t.Fatal("unexpected failure in running channel after first subscription") // 输出错误信息并终止测试
    }

    b, err := c.Subscribe() // 通过 Channel 的 Subscribe 方法订阅
    common.Must(err) // 如果 err 不为 nil，则触发 panic

    // Test that third subscriber is forbidden
    _, err = c.Subscribe() // 尝试第三次订阅
    if err == nil { // 如果没有出现错误
        t.Fatal("unexpected successful subscription") // 输出错误信息并终止测试
    }
    t.Log("expected error: ", err) // 输出预期的错误信息

    stopCh := make(chan struct{}) // 创建一个无缓冲的结构体类型的通道
    errCh := make(chan string) // 创建一个字符串类型的通道

    go func() { // 启动一个 goroutine
        c.Publish(context.Background(), 1) // 向 Channel 发布数据
        c.Publish(context.Background(), 2) // 向 Channel 发布数据
        c.Publish(context.Background(), "3") // 向 Channel 发布数据
        c.Publish(context.Background(), []int{4}) // 向 Channel 发布数据
        stopCh <- struct{}{} // 向 stopCh 通道发送数据
    }()

    go func() { // 启动另一个 goroutine
        if v, ok := (<-a).(int); !ok || v != 1 { // 从 a 通道接收数据，并进行类型断言和比较
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 1) // 向 errCh 通道发送错误信息
        }
        // ...（后续类似操作省略）
        stopCh <- struct{}{} // 向 stopCh 通道发送数据
    }()
    // 启动一个匿名的 goroutine
    go func() {
        // 从通道 b 中接收一个值，并判断是否为整数类型的 1
        if v, ok := (<-b).(int); !ok || v != 1 {
            // 如果接收到的值不是整数类型的 1，则向 errCh 发送错误信息
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 1)
        }
        // 从通道 b 中接收一个值，并判断是否为整数类型的 2
        if v, ok := (<-b).(int); !ok || v != 2 {
            // 如果接收到的值不是整数类型的 2，则向 errCh 发送错误信息
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 2)
        }
        // 从通道 b 中接收一个值，并判断是否为字符串类型的 "3"
        if v, ok := (<-b).(string); !ok || v != "3" {
            // 如果接收到的值不是字符串类型的 "3"，则向 errCh 发送错误信息
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", "3")
        }
        // 从通道 b 中接收一个值，并判断是否为整数数组类型，且第一个元素为 4
        if v, ok := (<-b).([]int); !ok || v[0] != 4 {
            // 如果接收到的值不是整数数组类型，或者第一个元素不是 4，则向 errCh 发送错误信息
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", []int{4})
        }
        // 向 stopCh 发送空结构体，表示停止信号
        stopCh <- struct{}{}
    }()

    // 设置一个 2 秒的超时时间
    timeout := time.After(2 * time.Second)
    // 循环 3 次
    for i := 0; i < 3; i++ {
        // 使用 select 语句处理通道的接收和超时情况
        select {
        case <-timeout:
            // 如果超时，则输出测试超时的错误信息
            t.Fatal("Test timeout after 2s")
        case e := <-errCh:
            // 如果从 errCh 中接收到错误信息，则输出错误信息
            t.Fatal(e)
        case <-stopCh:
            // 如果从 stopCh 中接收到停止信号，则继续下一次循环
        }
    }

    // 测试取消订阅通道的行为
    common.Must(c.Unsubscribe(b))

    // 测试最后一个订阅者是否会使用 `UnsubscribeClosableChannel` 关闭通道
    common.Must(stats.UnsubscribeClosableChannel(c, a))
    // 如果通道仍在运行，则输出意外的通道运行状态错误信息
    if c.Running() {
        t.Fatal("unexpected running channel after unsubscribing the last subscriber")
    }
func TestStatsChannelUnsubcribe(t *testing.T) {
    // 创建一个新的通道，并设置为阻塞模式
    c := NewChannel(&ChannelConfig{Blocking: true})
    // 确保通道已经启动
    common.Must(c.Start())
    // 延迟关闭通道，确保测试结束后关闭通道
    defer c.Close()

    // 订阅通道，获取订阅者 a 和可能的错误
    a, err := c.Subscribe()
    // 确保订阅成功
    common.Must(err)
    // 延迟取消订阅 a
    defer c.Unsubscribe(a)

    // 再次订阅通道，获取订阅者 b 和可能的错误
    b, err := c.Subscribe()
    // 确保订阅成功
    common.Must(err)

    // 创建暂停通道
    pauseCh := make(chan struct{})
    // 创建停止通道
    stopCh := make(chan struct{})
    // 创建错误通道
    errCh := make(chan string)

    {
        // 定义变量 aSet 和 bSet，并初始化为 false
        var aSet, bSet bool
        // 遍历通道的订阅者列表
        for _, s := range c.Subscribers() {
            // 如果订阅者是 a，则将 aSet 设置为 true
            if s == a {
                aSet = true
            }
            // 如果订阅者是 b，则将 bSet 设置为 true
            if s == b {
                bSet = true
            }
        }
        // 如果 aSet 和 bSet 都不为 true，则输出错误信息
        if !(aSet && bSet) {
            t.Fatal("unexpected subscribers: ", c.Subscribers())
        }
    }

    // 启动一个 goroutine，用于阻塞发布消息
    go func() { // Blocking publish
        // 发布消息 1，并等待暂停通道的信号
        c.Publish(context.Background(), 1)
        <-pauseCh // Wait for `b` goroutine to resume sending message
        // 发布消息 2
        c.Publish(context.Background(), 2)
    }()

    // 启动一个 goroutine，用于接收消息
    go func() {
        // 从订阅者 a 接收消息，并检查是否符合预期
        if v, ok := (<-a).(int); !ok || v != 1 {
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 1)
        }
        // 从订阅者 a 接收消息，并检查是否符合预期
        if v, ok := (<-a).(int); !ok || v != 2 {
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 2)
        }
    }()
    // 启动一个匿名的 goroutine
    go func() {
        // 从通道 b 中接收一个值，并判断是否为整数类型和值是否为 1
        if v, ok := (<-b).(int); !ok || v != 1 {
            // 如果不是整数类型或者值不为 1，则向错误通道发送错误信息
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 1)
        }
        // 在发布暂停期间取消订阅 `b`
        c.Unsubscribe(b)
        { // 测试 `b` 是否不在订阅者列表中
            var aSet, bSet bool
            for _, s := range c.Subscribers() {
                if s == a {
                    aSet = true
                }
                if s == b {
                    bSet = true
                }
            }
            // 如果 `a` 在订阅者列表中，而 `b` 不在，则向错误通道发送错误信息
            if !(aSet && !bSet) {
                errCh <- fmt.Sprint("unexpected subscribers: ", c.Subscribers())
            }
        }
        // 恢复发布进度
        close(pauseCh)
        // 测试 `b` 既不关闭也不能接收任何数据
        select {
        case v, ok := <-b:
            if ok {
                // 如果接收到数据，则向错误通道发送错误信息
                errCh <- fmt.Sprint("unexpected data received: ", v)
            } else {
                // 如果通道关闭，则向错误通道发送错误信息
                errCh <- fmt.Sprint("unexpected closed channel: ", b)
            }
        default:
        }
        // 关闭停止通道
        close(stopCh)
    }()

    // 选择不同的 case 进行处理
    select {
    case <-time.After(2 * time.Second):
        // 如果 2 秒后仍未收到信号，则测试超时
        t.Fatal("Test timeout after 2s")
    case e := <-errCh:
        // 如果收到错误信息，则测试失败
        t.Fatal(e)
    case <-stopCh:
        // 如果收到停止信号，则继续执行
    }
}
// 测试阻塞情况下的统计通道
func TestStatsChannelBlocking(t *testing.T) {
    // 不使用缓冲区以创建阻塞场景
    c := NewChannel(&ChannelConfig{BufferSize: 0, Blocking: true})
    common.Must(c.Start())
    defer c.Close()

    a, err := c.Subscribe()
    common.Must(err)
    defer c.Unsubscribe(a)

    pauseCh := make(chan struct{})
    stopCh := make(chan struct{})
    errCh := make(chan string)

    ctx, cancel := context.WithCancel(context.Background())

    // 测试阻塞通道发布
    go func() {
        // 没有订阅者接收的虚拟消息，将阻塞广播 goroutine
        c.Publish(context.Background(), nil)

        <-pauseCh

        // 在这里应该被阻塞，因为上一条消息没有被清除且缓冲区已满
        c.Publish(context.Background(), nil)

        pauseCh <- struct{}{}

        // 在这里仍然应该被阻塞
        c.Publish(ctx, nil)

        // 检查发布是否完成，因为上下文已被取消
        select {
        case <-ctx.Done():
            if ctx.Err() != context.Canceled {
                errCh <- fmt.Sprint("unexpected error: ", ctx.Err())
            }
        default:
            errCh <- "unexpected non-blocked publishing"
        }
        close(stopCh)
    }()

    go func() {
        pauseCh <- struct{}{}

        select {
        case <-pauseCh:
            errCh <- "unexpected non-blocked publishing"
        case <-time.After(100 * time.Millisecond):
        }

        // 接收第一条发布的消息
        <-a

        select {
        case <-pauseCh:
        case <-time.After(100 * time.Millisecond):
            errCh <- "unexpected blocking publishing"
        }

        // 手动取消上下文以结束发布
        cancel()
    }()

    select {
    case <-time.After(2 * time.Second):
        t.Fatal("Test timeout after 2s")
    case e := <-errCh:
        t.Fatal(e)
    case <-stopCh:
    }
}

func TestStatsChannelNonBlocking(t *testing.T) {
    // 创建一个不使用缓冲区的通道，以创建阻塞场景
    c := NewChannel(&ChannelConfig{BufferSize: 0, Blocking: false})
    // 强制启动通道
    common.Must(c.Start())
    // 延迟关闭通道
    defer c.Close()

    // 订阅通道，获取订阅者和错误信息
    a, err := c.Subscribe()
    common.Must(err)
    // 延迟取消订阅
    defer c.Unsubscribe(a)

    // 创建暂停、停止和错误通道
    pauseCh := make(chan struct{})
    stopCh := make(chan struct{})
    errCh := make(chan string)

    // 创建上下文和取消函数
    ctx, cancel := context.WithCancel(context.Background())

    // 测试阻塞通道发布
    go func() {
        // 发布空消息
        c.Publish(context.Background(), nil)
        c.Publish(context.Background(), nil)
        // 发送暂停信号
        pauseCh <- struct{}{}
        <-pauseCh
        // 发布消息并检查是否因为上下文取消而完成
        c.Publish(ctx, nil)
        c.Publish(ctx, nil)
        select {
        case <-ctx.Done():
            if ctx.Err() != context.Canceled {
                errCh <- fmt.Sprint("unexpected error: ", ctx.Err())
            }
        case <-time.After(100 * time.Millisecond):
            errCh <- "unexpected non-cancelled publishing"
        }
    }()

    go func() {
        // 检查即使没有订阅者接收消息，发布也不会阻塞
        select {
        case <-pauseCh:
        case <-time.After(100 * time.Millisecond):
            errCh <- "unexpected blocking publishing"
        }

        // 接收第一和第二个发布的消息
        <-a
        <-a

        // 发送暂停信号
        pauseCh <- struct{}{}

        // 手动取消上下文以结束发布
        cancel()

        // 检查第三和第四个发布的消息是否被取消并且无法接收
        <-time.After(100 * time.Millisecond)
        select {
        case <-a:
            errCh <- "unexpected non-cancelled publishing"
        default:
        }
        select {
        case <-a:
            errCh <- "unexpected non-cancelled publishing"
        default:
        }
        close(stopCh)
    }()

    // 检查测试是否超时
    select {
    case <-time.After(2 * time.Second):
        t.Fatal("Test timeout after 2s")
    # 从错误通道errCh中接收错误信息，并赋值给变量e，如果没有接收到信息则阻塞
    case e := <-errCh:
        # 如果接收到错误信息，则使用t.Fatal()方法输出错误信息并终止测试
        t.Fatal(e)
    # 从停止通道stopCh中接收信息，如果没有接收到信息则阻塞
    case <-stopCh:
    }
func TestStatsChannelConcurrency(t *testing.T) {
    // 创建一个不带缓冲区的通道，以创建阻塞场景
    c := NewChannel(&ChannelConfig{BufferSize: 0, Blocking: true})
    common.Must(c.Start()) // 确保通道已启动
    defer c.Close() // 在函数返回前关闭通道

    a, err := c.Subscribe() // 订阅通道，返回订阅者和错误
    common.Must(err) // 确保没有错误发生
    defer c.Unsubscribe(a) // 在函数返回前取消订阅

    b, err := c.Subscribe() // 再次订阅通道，返回订阅者和错误
    common.Must(err) // 确保没有错误发生
    defer c.Unsubscribe(b) // 在函数返回前取消订阅

    stopCh := make(chan struct{}) // 创建一个停止信号的通道
    errCh := make(chan string) // 创建一个错误信息的通道

    go func() { // 启动一个 goroutine，用于阻塞发布
        c.Publish(context.Background(), 1) // 发布数据 1 到通道
        c.Publish(context.Background(), 2) // 发布数据 2 到通道
    }()

    go func() { // 启动一个 goroutine，用于接收数据
        if v, ok := (<-a).(int); !ok || v != 1 { // 从订阅者 a 接收数据，并检查是否符合预期
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 1) // 如果接收到的数据不符合预期，向错误通道发送错误信息
        }
        if v, ok := (<-a).(int); !ok || v != 2 { // 从订阅者 a 接收数据，并检查是否符合预期
            errCh <- fmt.Sprint("unexpected receiving: ", v, ", wanted ", 2) // 如果接收到的数据不符合预期，向错误通道发送错误信息
        }
    }()
    // 启动一个匿名的 goroutine
    go func() {
        // 阻塞 `b` 一段时间，以确保源通道正在尝试向 `b` 发送消息
        <-time.After(25 * time.Millisecond)
        // 这会导致并发场景：在尝试向其发送消息时取消订阅 `b`
        c.Unsubscribe(b)
        // 测试 `b` 没有关闭且仍然可以接收数据 1：
        // 因为取消订阅不会影响正在进行的发送消息过程
        select {
        case v, ok := <-b:
            if v1, ok1 := v.(int); !(ok && ok1 && v1 == 1) {
                errCh <- fmt.Sprint("unexpected failure in receiving data: ", 1)
            }
        default:
            errCh <- fmt.Sprint("unexpected block from receiving data: ", 1)
        }
        // 测试 `b` 没有关闭但无法接收数据 2：
        // 因为在新一轮的消息传递中，`b` 已经被取消订阅
        select {
        case v, ok := <-b:
            if ok {
                errCh <- fmt.Sprint("unexpected receiving: ", v)
            } else {
                errCh <- fmt.Sprint("unexpected closing of channel")
            }
        default:
        }
        // 关闭 stopCh 通道
        close(stopCh)
    }()

    // 选择监听多个通道的情况
    select {
    case <-time.After(2 * time.Second):
        t.Fatal("Test timeout after 2s")
    case e := <-errCh:
        t.Fatal(e)
    case <-stopCh:
    }
# 闭合前面的函数定义
```