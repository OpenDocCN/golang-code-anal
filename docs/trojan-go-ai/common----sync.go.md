# `trojan-go\common\sync.go`

```
// Notifier 是一个用于通知变化的实用工具。变化生产者可以多次通知变化，而消费者可以异步获取通知。
type Notifier struct {
    c chan struct{}  // 用于通知变化的通道
}

// NewNotifier 创建一个新的 Notifier。
func NewNotifier() *Notifier {
    return &Notifier{
        c: make(chan struct{}, 1),  // 创建一个带有缓冲区大小为1的通道
    }
}

// Signal 发出变化信号，通常由生产者调用。该方法永远不会阻塞。
func (n *Notifier) Signal() {
    select {
    case n.c <- struct{}{}:  // 将空结构体发送到通道中
    default:  // 如果通道已满，则执行默认操作
    }
}

// Wait 返回一个用于等待变化的通道。返回的通道永远不会被关闭。
func (n *Notifier) Wait() <-chan struct{} {
    return n.c  // 返回通知变化的通道
}
```