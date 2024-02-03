# `kubo\thirdparty\notifier\notifier_test.go`

```go
package notifier

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "sync" // 导入 sync 包，用于实现同步操作
    "testing" // 导入 testing 包，用于编写测试函数
    "time" // 导入 time 包，用于处理时间相关操作
)

// test data structures.
type Router struct {
    queue    chan Packet // 定义一个通道，用于存储 Packet 类型的数据
    notifier Notifier // 定义一个 Notifier 接口类型的字段
}

type Packet struct{} // 定义一个空结构体 Packet

type RouterNotifiee interface {
    Enqueued(*Router, Packet) // 定义 RouterNotifiee 接口，包含 Enqueued 方法
    Forwarded(*Router, Packet) // 定义 RouterNotifiee 接口，包含 Forwarded 方法
    Dropped(*Router, Packet) // 定义 RouterNotifiee 接口，包含 Dropped 方法
}

func (r *Router) Notify(n RouterNotifiee) {
    r.notifier.Notify(n) // 调用 notifier 接口的 Notify 方法
}

func (r *Router) StopNotify(n RouterNotifiee) {
    r.notifier.StopNotify(n) // 调用 notifier 接口的 StopNotify 方法
}

func (r *Router) notifyAll(notify func(n RouterNotifiee)) {
    r.notifier.NotifyAll(func(n Notifiee) {
        notify(n.(RouterNotifiee)) // 调用 notifier 接口的 NotifyAll 方法，并将 Notifiee 转换为 RouterNotifiee 类型
    })
}

func (r *Router) Receive(p Packet) {
    select {
    case r.queue <- p: // 将 p 发送到队列中，表示 enqueued
        r.notifyAll(func(n RouterNotifiee) {
            n.Enqueued(r, p) // 调用所有注册的 RouterNotifiee 接口的 Enqueued 方法
        })

    default: // 如果队列已满，表示 drop
        r.notifyAll(func(n RouterNotifiee) {
            n.Dropped(r, p) // 调用所有注册的 RouterNotifiee 接口的 Dropped 方法
        })
    }
}

func (r *Router) Forward() {
    p := <-r.queue // 从队列中取出数据
    r.notifyAll(func(n RouterNotifiee) {
        n.Forwarded(r, p) // 调用所有注册的 RouterNotifiee 接口的 Forwarded 方法
    })
}

type Metrics struct {
    enqueued  int // 已入队数量
    forwarded int // 已转发数量
    dropped   int // 已丢弃数量
    received  chan struct{} // 接收通道
    sync.Mutex // 互斥锁
}

func (m *Metrics) Enqueued(*Router, Packet) {
    m.Lock() // 加锁
    m.enqueued++ // 入队数量加一
    m.Unlock() // 解锁
    if m.received != nil {
        m.received <- struct{}{} // 如果接收通道不为空，则向通道发送数据
    }
}

func (m *Metrics) Forwarded(*Router, Packet) {
    m.Lock() // 加锁
    m.forwarded++ // 转发数量加一
    m.Unlock() // 解锁
    if m.received != nil {
        m.received <- struct{}{} // 如果接收通道不为空，则向通道发送数据
    }
}

func (m *Metrics) Dropped(*Router, Packet) {
    m.Lock() // 加锁
    m.dropped++ // 丢弃数量加一
    m.Unlock() // 解锁
    if m.received != nil {
        m.received <- struct{}{} // 如果接收通道不为空，则向通道发送数据
    }
}

func (m *Metrics) String() string {
    m.Lock() // 加锁
    defer m.Unlock() // 延迟解锁
    return fmt.Sprintf("%d enqueued, %d forwarded, %d in queue, %d dropped",
        m.enqueued, m.forwarded, m.enqueued-m.forwarded, m.dropped) // 格式化输出已入队、已转发、队列中数量、已丢弃的数据
}

func TestNotifies(t *testing.T) {
    m := Metrics{received: make(chan struct{})} // 创建 Metrics 结构体实例，初始化接收通道
    r := Router{queue: make(chan Packet, 10)} // 创建 Router 结构体实例，初始化队列通道
    r.Notify(&m) // 注册 Metrics 结构体实例到 Router 实例的通知列表
}
    # 循环10次，每次向 r 发送一个空的 Packet 对象
    for i := 0; i < 10; i++ {
        r.Receive(Packet{})
        # 从 m.received 通道接收数据
        <-m.received
        # 如果 m.enqueued 不等于 (1 + i)，则输出错误信息
        if m.enqueued != (1 + i) {
            t.Error("not notifying correctly", m.enqueued, 1+i)
        }

    }

    # 再次循环10次，每次向 r 发送一个空的 Packet 对象
    for i := 0; i < 10; i++ {
        r.Receive(Packet{})
        # 从 m.received 通道接收数据
        <-m.received
        # 如果 m.enqueued 不等于 10，则输出错误信息
        if m.enqueued != 10 {
            t.Error("not notifying correctly", m.enqueued, 10)
        }
        # 如果 m.dropped 不等于 (1 + i)，则输出错误信息
        if m.dropped != (1 + i) {
            t.Error("not notifying correctly", m.dropped, 1+i)
        }
    }
func TestStopsNotifying(t *testing.T) {
    // 创建 Metrics 结构体对象，并初始化 received 通道
    m := Metrics{received: make(chan struct{})}
    // 创建 Router 结构体对象，并初始化 queue 通道
    r := Router{queue: make(chan Packet, 10)}
    // 通知 Router 对象监听 Metrics 对象
    r.Notify(&m)

    // 循环 5 次
    for i := 0; i < 5; i++ {
        // 向 Router 对象发送 Packet
        r.Receive(Packet{})
        // 从 received 通道接收数据
        <-m.received
        // 检查 enqueued 是否等于 (1 + i)，如果不等则输出错误信息
        if m.enqueued != (1 + i) {
            t.Error("not notifying correctly")
        }
    }

    // 停止通知 Router 对象监听 Metrics 对象
    r.StopNotify(&m)

    // 再次循环 5 次
    for i := 0; i < 5; i++ {
        // 向 Router 对象发送 Packet
        r.Receive(Packet{})
        // 从 received 通道接收数据，如果能接收到数据则输出错误信息
        select {
        case <-m.received:
            t.Error("did not stop notifying")
        default:
        }
        // 检查 enqueued 是否等于 5，如果不等则输出错误信息
        if m.enqueued != 5 {
            t.Error("did not stop notifying")
        }
    }
}

func TestThreadsafe(t *testing.T) {
    // 定义 N 为 1000
    N := 1000
    // 创建 Router 结构体对象，并初始化 queue 通道
    r := Router{queue: make(chan Packet, 10)}
    // 创建三个 Metrics 结构体对象，并初始化 received 通道
    m1 := Metrics{received: make(chan struct{})}
    m2 := Metrics{received: make(chan struct{})}
    m3 := Metrics{received: make(chan struct{})}
    // 通知 Router 对象监听三个 Metrics 对象
    r.Notify(&m1)
    r.Notify(&m2)
    r.Notify(&m3)

    // 定义变量 n 和 WaitGroup 对象 wg
    var n int
    var wg sync.WaitGroup
    // 循环 N 次
    for i := 0; i < N; i++ {
        n++
        wg.Add(1)
        // 启动一个 goroutine，向 Router 对象发送 Packet
        go func() {
            defer wg.Done()
            r.Receive(Packet{})
        }()

        // 如果 i 除以 3 的余数为 0
        if i%3 == 0 {
            n++
            wg.Add(1)
            // 启动一个 goroutine，执行 Router 对象的 Forward 方法
            go func() {
                defer wg.Done()
                r.Forward()
            }()
        }
    }

    // 清空队列
    for i := 0; i < (n * 3); i++ {
        // 从 m1.received、m2.received、m3.received 三个通道中接收数据
        select {
        case <-m1.received:
        case <-m2.received:
        case <-m3.received:
        }
    }

    // 等待所有 goroutine 完成
    wg.Wait()

    // 输出 m1、m2、m3 的统计信息
    t.Log("m1", m1.String())
    t.Log("m2", m2.String())
    t.Log("m3", m3.String())

    // 检查 m1、m2、m3 的统计信息是否一致，如果不一致则输出错误信息
    if m1.String() != m2.String() || m2.String() != m3.String() {
        t.Error("counts disagree")
    }
}

type highwatermark struct {
    mu    sync.Mutex
    mark  int
    limit int
    errs  chan error
}

func (m *highwatermark) incr() {
    // 加锁
    m.mu.Lock()
    // mark 加一
    m.mark++
    // 解锁
    # 如果标记值大于限制值
    if m.mark > m.limit {
        # 将错误信息发送到错误通道
        m.errs <- fmt.Errorf("went over limit: %d/%d", m.mark, m.limit)
    }
    # 释放互斥锁
    m.mu.Unlock()
// 减少高水位标记的值
func (m *highwatermark) decr() {
    // 加锁，确保并发安全
    m.mu.Lock()
    // 标记值减一
    m.mark--
    // 如果标记值小于0，向错误通道发送错误信息
    if m.mark < 0 {
        m.errs <- fmt.Errorf("went under zero: %d/%d", m.mark, m.limit)
    }
    // 解锁
    m.mu.Unlock()
}

// 测试有限速率
func TestLimited(t *testing.T) {
    // 设置超时时间
    timeout := 10 * time.Second // huge timeout.
    // 设置限制值
    limit := 9

    // 创建高水位标记对象
    hwm := highwatermark{limit: limit, errs: make(chan error, 100)}
    // 创建有限速率对象
    n := RateLimited(limit) // will stop after 3 rounds
    // 发送通知
    n.Notify(1)
    n.Notify(2)
    n.Notify(3)

    // 创建通道
    entr := make(chan struct{})
    exit := make(chan struct{})
    done := make(chan struct{})
    // 启动协程
    go func() {
        for i := 0; i < 10; i++ {
            // 发送通知给所有观察者
            n.NotifyAll(func(e Notifiee) {
                // 增加高水位标记的值
                hwm.incr()
                // 发送进入通道
                entr <- struct{}{}
                // 等待退出通道
                <-exit // wait
                // 减少高水位标记的值
                hwm.decr()
            })
        }
        // 发送完成通道
        done <- struct{}{}
    }()

    for i := 0; i < 30; {
        select {
        case <-entr:
            // 继续让尽可能多的进入
            continue // let as many enter as possible
        case <-time.After(1 * time.Millisecond):
        }

        // 让一个退出
        select {
        case <-entr:
            // 在出现时间问题的情况下继续
            continue // in case of timing issues.
        case exit <- struct{}{}:
        case <-time.After(timeout):
            t.Error("got stuck")
        }
        i++
    }

    select {
    case <-done: // 两部分完成
    case <-time.After(timeout):
        t.Error("did not finish")
    }

    // 关闭错误通道
    close(hwm.errs)
    // 遍历错误通道，输出错误信息
    for err := range hwm.errs {
        t.Error(err)
    }
}
```