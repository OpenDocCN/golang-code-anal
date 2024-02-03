# `v2ray-core\common\signal\timer.go`

```go
package signal

import (
    "context"
    "sync"
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/common/task"
)

type ActivityUpdater interface {
    Update()  // 定义接口，包含更新方法
}

type ActivityTimer struct {
    sync.RWMutex  // 定义同步读写锁
    updated   chan struct{}  // 用于通知更新的通道
    checkTask *task.Periodic  // 定时任务
    onTimeout func()  // 超时回调函数
}

func (t *ActivityTimer) Update() {
    select {
    case t.updated <- struct{}{}:  // 将更新通知发送到通道
    default:
    }
}

func (t *ActivityTimer) check() error {
    select {
    case <-t.updated:  // 监听更新通道
    default:
        t.finish()  // 如果没有更新，则执行结束操作
    }
    return nil
}

func (t *ActivityTimer) finish() {
    t.Lock()  // 加锁
    defer t.Unlock()  // 解锁

    if t.onTimeout != nil {
        t.onTimeout()  // 执行超时回调函数
        t.onTimeout = nil  // 清空超时回调函数
    }
    if t.checkTask != nil {
        t.checkTask.Close() // 关闭定时任务
        t.checkTask = nil
    }
}

func (t *ActivityTimer) SetTimeout(timeout time.Duration) {
    if timeout == 0 {
        t.finish()  // 如果超时时间为0，则执行结束操作
        return
    }

    checkTask := &task.Periodic{  // 创建定时任务
        Interval: timeout,
        Execute:  t.check,
    }

    t.Lock()  // 加锁

    if t.checkTask != nil {
        t.checkTask.Close() // 关闭之前的定时任务
    }
    t.checkTask = checkTask  // 设置新的定时任务
    t.Unlock()  // 解锁
    t.Update()  // 执行更新操作
    common.Must(checkTask.Start())  // 启动定时任务
}

func CancelAfterInactivity(ctx context.Context, cancel context.CancelFunc, timeout time.Duration) *ActivityTimer {
    timer := &ActivityTimer{  // 创建活动定时器对象
        updated:   make(chan struct{}, 1),  // 初始化更新通道
        onTimeout: cancel,  // 设置超时回调函数
    }
    timer.SetTimeout(timeout)  // 设置超时时间
    return timer  // 返回活动定时器对象
}
```