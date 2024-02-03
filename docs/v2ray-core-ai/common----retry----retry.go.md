# `v2ray-core\common\retry\retry.go`

```go
package retry // import "v2ray.com/core/common/retry"

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "time"
)

var (
    ErrRetryFailed = newError("all retry attempts failed") // 定义一个全局变量，表示所有重试尝试均失败
)

// Strategy is a way to retry on a specific function.
type Strategy interface {
    // On performs a retry on a specific function, until it doesn't return any error.
    On(func() error) error // 定义一个接口，包含一个执行重试的方法
}

type retryer struct {
    totalAttempt int
    nextDelay    func() uint32
}

// On implements Strategy.On.
func (r *retryer) On(method func() error) error {
    attempt := 0
    accumulatedError := make([]error, 0, r.totalAttempt)
    for attempt < r.totalAttempt {
        err := method()
        if err == nil {
            return nil
        }
        numErrors := len(accumulatedError)
        if numErrors == 0 || err.Error() != accumulatedError[numErrors-1].Error() {
            accumulatedError = append(accumulatedError, err)
        }
        delay := r.nextDelay()
        time.Sleep(time.Duration(delay) * time.Millisecond)
        attempt++
    }
    return newError(accumulatedError).Base(ErrRetryFailed) // 返回一个包含所有错误的新错误，作为重试失败的基础
}

// Timed returns a retry strategy with fixed interval.
func Timed(attempts int, delay uint32) Strategy {
    return &retryer{
        totalAttempt: attempts,
        nextDelay: func() uint32 {
            return delay
        },
    }
}

func ExponentialBackoff(attempts int, delay uint32) Strategy {
    nextDelay := uint32(0)
    return &retryer{
        totalAttempt: attempts,
        nextDelay: func() uint32 {
            r := nextDelay
            nextDelay += delay
            return r
        },
    }
}
```