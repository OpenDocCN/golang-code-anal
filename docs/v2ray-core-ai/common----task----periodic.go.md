# `v2ray-core\common\task\periodic.go`

```
package task

import (
    "sync"
    "time"
)

// Periodic is a task that runs periodically.
type Periodic struct {
    // Interval of the task being run
    Interval time.Duration
    // Execute is the task function
    Execute func() error

    access  sync.Mutex
    timer   *time.Timer
    running bool
}

func (t *Periodic) hasClosed() bool {
    t.access.Lock()  // 加锁以确保并发安全
    defer t.access.Unlock()  // 在函数返回前解锁

    return !t.running  // 返回任务是否已关闭
}

func (t *Periodic) checkedExecute() error {
    if t.hasClosed() {  // 如果任务已关闭，则直接返回
        return nil
    }

    if err := t.Execute(); err != nil {  // 执行任务函数，如果出错则处理
        t.access.Lock()  // 加锁
        t.running = false  // 将任务状态设置为关闭
        t.access.Unlock()  // 解锁
        return err  // 返回错误
    }

    t.access.Lock()  // 加锁
    defer t.access.Unlock()  // 在函数返回前解锁

    if !t.running {  // 如果任务已关闭，则直接返回
        return nil
    }

    t.timer = time.AfterFunc(t.Interval, func() {  // 设置定时器，定时执行任务
        t.checkedExecute() // nolint: errcheck  // 执行任务函数，忽略错误检查
    })

    return nil
}

// Start implements common.Runnable.
func (t *Periodic) Start() error {
    t.access.Lock()  // 加锁
    if t.running {  // 如果任务已在运行，则直接返回
        t.access.Unlock()  // 解锁
        return nil
    }
    t.running = true  // 将任务状态设置为运行中
    t.access.Unlock()  // 解锁

    if err := t.checkedExecute(); err != nil {  // 执行任务函数，如果出错则处理
        t.access.Lock()  // 加锁
        t.running = false  // 将任务状态设置为关闭
        t.access.Unlock()  // 解锁
        return err  // 返回错误
    }

    return nil
}

// Close implements common.Closable.
func (t *Periodic) Close() error {
    t.access.Lock()  // 加锁
    defer t.access.Unlock()  // 在函数返回前解锁

    t.running = false  // 将任务状态设置为关闭
    if t.timer != nil {  // 如果定时器存在，则停止定时器
        t.timer.Stop()
        t.timer = nil
    }

    return nil
}
```