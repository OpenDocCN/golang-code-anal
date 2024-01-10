# `v2ray-core\common\signal\done\done.go`

```
// 包名为 done
package done

// 引入 sync 包，用于实现同步操作
import (
    "sync"
)

// Instance 结构体用于表示某个操作已完成的实用工具
type Instance struct {
    access sync.Mutex  // 用于保护共享资源的互斥锁
    c      chan struct{}  // 用于通知操作完成的通道
    closed bool  // 标记操作是否已完成
}

// New 函数返回一个新的 Instance 实例
func New() *Instance {
    return &Instance{
        c: make(chan struct{}),  // 初始化通道
    }
}

// Done 方法返回 true，如果 Close() 方法被调用
func (d *Instance) Done() bool {
    select {
    case <-d.Wait():  // 等待 Wait() 方法返回的通道
        return true
    default:
        return false
    }
}

// Wait 方法返回一个用于等待操作完成的通道
func (d *Instance) Wait() <-chan struct{} {
    return d.c
}

// Close 方法标记该操作已完成。该方法可以被多次调用，但第一次调用之后的所有调用都不会对其状态产生影响
func (d *Instance) Close() error {
    d.access.Lock()  // 加锁
    defer d.access.Unlock()  // 延迟解锁

    if d.closed {  // 如果操作已完成
        return nil
    }

    d.closed = true  // 标记操作已完成
    close(d.c)  // 关闭通道，通知操作已完成

    return nil
}
```