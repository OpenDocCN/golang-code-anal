# `v2ray-core\common\signal\notifier.go`

```
// 定义一个名为 Notifier 的结构体，用于通知变化。变化生产者可以多次通知变化，消费者可以异步获取通知。
type Notifier struct {
    c chan struct{}  // 用于通知变化的通道
}

// NewNotifier 创建一个新的 Notifier 对象。
func NewNotifier() *Notifier {
    return &Notifier{
        c: make(chan struct{}, 1),  // 创建一个带有缓冲区大小为 1 的通道
    }
}

// Signal 用于通知变化，通常由生产者调用。该方法不会阻塞。
func (n *Notifier) Signal() {
    select {
    case n.c <- struct{}{}:  // 将空结构体发送到通道中
    default:
    }
}

// Wait 返回一个用于等待变化的通道。返回的通道永远不会被关闭。
func (n *Notifier) Wait() <-chan struct{} {
    return n.c  // 返回通知变化的通道
}
```