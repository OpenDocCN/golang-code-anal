# `v2ray-core\common\signal\timer_test.go`

```go
// 声明 signal_test 包
package signal_test

// 导入所需的包
import (
    "context"
    "runtime"
    "testing"
    "time"

    . "v2ray.com/core/common/signal"
)

// 测试 ActivityTimer 函数
func TestActivityTimer(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在指定时间内无活动则取消上下文
    timer := CancelAfterInactivity(ctx, cancel, time.Second*4)
    // 休眠指定时间
    time.Sleep(time.Second * 6)
    // 检查上下文的错误状态
    if ctx.Err() == nil {
        t.Error("expected some error, but got nil")
    }
    // 保持 timer 活跃
    runtime.KeepAlive(timer)
}

// 测试 ActivityTimerUpdate 函数
func TestActivityTimerUpdate(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在指定时间内无活动则取消上下文
    timer := CancelAfterInactivity(ctx, cancel, time.Second*10)
    // 休眠指定时间
    time.Sleep(time.Second * 3)
    // 检查上下文的错误状态
    if ctx.Err() != nil {
        t.Error("expected nil, but got ", ctx.Err().Error())
    }
    // 设置新的超时时间
    timer.SetTimeout(time.Second * 1)
    // 休眠指定时间
    time.Sleep(time.Second * 2)
    // 检查上下文的错误状态
    if ctx.Err() == nil {
        t.Error("expcted some error, but got nil")
    }
    // 保持 timer 活跃
    runtime.KeepAlive(timer)
}

// 测试 ActivityTimerNonBlocking 函数
func TestActivityTimerNonBlocking(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在指定时间内无活动则取消上下文
    timer := CancelAfterInactivity(ctx, cancel, 0)
    // 休眠指定时间
    time.Sleep(time.Second * 1)
    // 检查上下文是否已完成
    select {
    case <-ctx.Done():
    default:
        t.Error("context not done")
    }
    // 设置新的超时时间
    timer.SetTimeout(0)
    timer.SetTimeout(1)
    timer.SetTimeout(2)
}

// 测试 ActivityTimerZeroTimeout 函数
func TestActivityTimerZeroTimeout(t *testing.T) {
    // 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(context.Background())
    // 在指定时间内无活动则取消上下文
    timer := CancelAfterInactivity(ctx, cancel, 0)
    // 检查上下文是否已完成
    select {
    case <-ctx.Done():
    default:
        t.Error("context not done")
    }
    // 保持 timer 活跃
    runtime.KeepAlive(timer)
}
```