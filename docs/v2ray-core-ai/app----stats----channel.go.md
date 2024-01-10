# `v2ray-core\app\stats\channel.go`

```
// +build !confonly  // 标记此文件不仅仅是配置文件

package stats  // 声明 stats 包

import (  // 导入所需的包
    "context"  // 上下文包
    "sync"  // 同步包

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
)

// Channel is an implementation of stats.Channel.  // Channel 是 stats.Channel 的实现
type Channel struct {  // 定义 Channel 结构体
    channel     chan channelMessage  // 定义 channel 属性为通道类型
    subscribers []chan interface{}  // 定义 subscribers 属性为通道接口类型的切片

    // Synchronization components  // 同步组件
    access sync.RWMutex  // 定义 access 属性为读写锁
    closed chan struct{}  // 定义 closed 属性为结构体通道

    // Channel options  // 通道选项
    blocking   bool // Set blocking state if channel buffer reaches limit  // 如果通道缓冲区达到限制，则设置阻塞状态
    bufferSize int  // Set to 0 as no buffering  // 设置为 0 表示没有缓冲
    subsLimit  int  // Set to 0 as no subscriber limit  // 设置为 0 表示没有订阅者限制
}

// NewChannel creates an instance of Statistics Channel.  // NewChannel 创建一个统计通道的实例
func NewChannel(config *ChannelConfig) *Channel {  // 定义 NewChannel 函数，传入配置参数，返回 Channel 实例
    return &Channel{  // 返回 Channel 实例
        channel:    make(chan channelMessage, config.BufferSize),  // 创建带有指定缓冲区大小的通道
        subsLimit:  int(config.SubscriberLimit),  // 设置订阅者限制
        bufferSize: int(config.BufferSize),  // 设置缓冲区大小
        blocking:   config.Blocking,  // 设置阻塞状态
    }
}

// Subscribers implements stats.Channel.  // Subscribers 实现了 stats.Channel
func (c *Channel) Subscribers() []chan interface{} {  // 定义 Subscribers 方法，返回订阅者切片
    c.access.RLock()  // 加读锁
    defer c.access.RUnlock()  // 延迟解锁
    return c.subscribers  // 返回订阅者切片
}

// Subscribe implements stats.Channel.  // Subscribe 实现了 stats.Channel
func (c *Channel) Subscribe() (chan interface{}, error) {  // 定义 Subscribe 方法，返回通道接口和错误
    c.access.Lock()  // 加锁
    defer c.access.Unlock()  // 延迟解锁
    if c.subsLimit > 0 && len(c.subscribers) >= c.subsLimit {  // 如果订阅者限制大于 0 且订阅者数量大于等于订阅者限制
        return nil, newError("Number of subscribers has reached limit")  // 返回空和错误信息
    }
    subscriber := make(chan interface{}, c.bufferSize)  // 创建带有指定缓冲区大小的通道接口
    c.subscribers = append(c.subscribers, subscriber)  // 将订阅者添加到订阅者切片
    return subscriber, nil  // 返回订阅者和空错误
}

// Unsubscribe implements stats.Channel.  // Unsubscribe 实现了 stats.Channel
func (c *Channel) Unsubscribe(subscriber chan interface{}) error {  // 定义 Unsubscribe 方法，传入订阅者通道接口，返回错误
    c.access.Lock()  // 加锁
    defer c.access.Unlock()  // 延迟解锁
    for i, s := range c.subscribers {  // 遍历订阅者切片
        if s == subscriber {  // 如果当前订阅者等于传入的订阅者
            // Copy to new memory block to prevent modifying original data  // 复制到新的内存块以防止修改原始数据
            subscribers := make([]chan interface{}, len(c.subscribers)-1)  // 创建新的订阅者切片
            copy(subscribers[:i], c.subscribers[:i])  // 复制前半部分
            copy(subscribers[i:], c.subscribers[i+1:])  // 复制后半部分
            c.subscribers = subscribers  // 更新订阅者切片
        }
    }
    # 返回空值
    return nil
// Publish 实现了 stats.Channel 接口，用于发布消息到通道
func (c *Channel) Publish(ctx context.Context, msg interface{}) {
    select { // 如果通道已关闭，则提前退出
    case <-c.closed:
        return
    default:
        pub := channelMessage{context: ctx, message: msg}
        if c.blocking {
            pub.publish(c.channel)
        } else {
            pub.publishNonBlocking(c.channel)
        }
    }
}

// Running 返回通道是否正在运行
func (c *Channel) Running() bool {
    select {
    case <-c.closed: // 通道已关闭
    default: // 通道正在运行或未初始化
        if c.closed != nil { // 通道已初始化
            return true
        }
    }
    return false
}

// Start 实现了 common.Runnable 接口，用于启动通道
func (c *Channel) Start() error {
    c.access.Lock()
    defer c.access.Unlock()
    if !c.Running() {
        c.closed = make(chan struct{}) // 重置关闭信号
        go func() {
            for {
                select {
                case pub := <-c.channel: // 接收到发布的消息
                    for _, sub := range c.Subscribers() { // 并发安全地获取订阅者
                        if c.blocking {
                            pub.broadcast(sub)
                        } else {
                            pub.broadcastNonBlocking(sub)
                        }
                    }
                case <-c.closed: // 通道已关闭
                    for _, sub := range c.Subscribers() { // 移除所有订阅者
                        common.Must(c.Unsubscribe(sub))
                        close(sub)
                    }
                    return
                }
            }
        }()
    }
    return nil
}

// Close 实现了 common.Closable 接口，用于关闭通道
func (c *Channel) Close() error {
    c.access.Lock()
    defer c.access.Unlock()
    if c.Running() {
        close(c.closed) // 发送关闭信号
    }
    return nil
}

// channelMessage 是具有可靠传递的已发布消息
// 定义一个结构体 channelMessage，包含一个上下文和一个消息
type channelMessage struct {
    context context.Context
    message interface{}
}

// 发布消息到通道，当上下文被提前取消时，消息被丢弃
func (c channelMessage) publish(publisher chan channelMessage) {
    // 使用 select 语句，如果可以将消息发送到通道，则发送；否则等待上下文被取消
    select {
    case publisher <- c:
    case <-c.context.Done():
    }
}

// 非阻塞地发布消息到通道，如果通道已满则创建另一个 goroutine 继续发送消息
func (c channelMessage) publishNonBlocking(publisher chan channelMessage) {
    // 使用 select 语句，如果可以将消息发送到通道，则发送；否则创建另一个 goroutine 继续发送消息
    select {
    case publisher <- c:
    default: // 创建另一个 goroutine 继续发送消息
        go c.publish(publisher)
    }
}

// 广播消息到订阅者通道
func (c channelMessage) broadcast(subscriber chan interface{}) {
    // 使用 select 语句，如果可以将消息发送到订阅者通道，则发送；否则等待上下文被取消
    select {
    case subscriber <- c.message:
    case <-c.context.Done():
    }
}

// 非阻塞地广播消息到订阅者通道，如果通道已满则创建另一个 goroutine 继续发送消息
func (c channelMessage) broadcastNonBlocking(subscriber chan interface{}) {
    // 使用 select 语句，如果可以将消息发送到订阅者通道，则发送；否则创建另一个 goroutine 继续发送消息
    select {
    case subscriber <- c.message:
    default: // 创建另一个 goroutine 继续发送消息
        go c.broadcast(subscriber)
    }
}
```