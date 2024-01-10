# `v2ray-core\common\retry\retry_test.go`

```
package retry_test

import (
    "testing"
    "time"

    "v2ray.com/core/common"
    "v2ray.com/core/common/errors"
    . "v2ray.com/core/common/retry"
)

var (
    errorTestOnly = errors.New("This is a fake error.")
)

func TestNoRetry(t *testing.T) {
    startTime := time.Now().Unix()
    // 使用 Timed 方法设置最大重试次数和重试间隔，然后执行传入的函数
    err := Timed(10, 100000).On(func() error {
        return nil
    })
    endTime := time.Now().Unix()

    // 如果 err 不为 nil，则触发 panic
    common.Must(err)
    // 如果 endTime 小于 startTime，则输出错误信息
    if endTime < startTime {
        t.Error("endTime < startTime: ", startTime, " -> ", endTime)
    }
}

func TestRetryOnce(t *testing.T) {
    startTime := time.Now()
    called := 0
    // 使用 Timed 方法设置最大重试次数和重试间隔，然后执行传入的函数
    err := Timed(10, 1000).On(func() error {
        if called == 0 {
            called++
            return errorTestOnly
        }
        return nil
    })
    duration := time.Since(startTime)

    // 如果 err 不为 nil，则触发 panic
    common.Must(err)
    // 如果 duration 小于 900 毫秒，则输出错误信息
    if v := int64(duration / time.Millisecond); v < 900 {
        t.Error("duration: ", v)
    }
}

func TestRetryMultiple(t *testing.T) {
    startTime := time.Now()
    called := 0
    // 使用 Timed 方法设置最大重试次数和重试间隔，然后执行传入的函数
    err := Timed(10, 1000).On(func() error {
        if called < 5 {
            called++
            return errorTestOnly
        }
        return nil
    })
    duration := time.Since(startTime)

    // 如果 err 不为 nil，则触发 panic
    common.Must(err)
    // 如果 duration 小于 4900 毫秒，则输出错误信息
    if v := int64(duration / time.Millisecond); v < 4900 {
        t.Error("duration: ", v)
    }
}

func TestRetryExhausted(t *testing.T) {
    startTime := time.Now()
    called := 0
    // 使用 Timed 方法设置最大重试次数和重试间隔，然后执行传入的函数
    err := Timed(2, 1000).On(func() error {
        called++
        return errorTestOnly
    })
    duration := time.Since(startTime)

    // 如果 err 的原因不是 ErrRetryFailed，则输出错误信息
    if errors.Cause(err) != ErrRetryFailed {
        t.Error("cause: ", err)
    }

    // 如果 duration 小于 1900 毫秒，则输出错误信息
    if v := int64(duration / time.Millisecond); v < 1900 {
        t.Error("duration: ", v)
    }
}

func TestExponentialBackoff(t *testing.T) {
    startTime := time.Now()
    called := 0
    // 使用 ExponentialBackoff 方法设置最大重试次数和初始重试间隔，然后执行传入的函数
    err := ExponentialBackoff(10, 100).On(func() error {
        called++
        return errorTestOnly
    })
    duration := time.Since(startTime)
    # 如果错误的原因不是重试失败，则输出错误信息
    if errors.Cause(err) != ErrRetryFailed:
        t.Error("cause: ", err)
    
    # 计算时间间隔的毫秒数，并将结果赋值给变量v，如果v小于4000，则输出错误信息
    if v := int64(duration / time.Millisecond); v < 4000:
        t.Error("duration: ", v)
# 闭合前面的函数定义
```