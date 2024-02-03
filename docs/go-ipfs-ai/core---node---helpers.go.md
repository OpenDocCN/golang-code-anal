# `kubo\core\node\helpers.go`

```go
package node

import (
    "context"

    "github.com/jbenet/goprocess"
    "github.com/pkg/errors"
    "go.uber.org/fx"
)

type lcProcess struct {
    fx.In

    LC   fx.Lifecycle
    Proc goprocess.Process
}

// Append wraps ProcessFunc into a goprocess, and appends it to the lifecycle
func (lp *lcProcess) Append(f goprocess.ProcessFunc) {
    // Hooks are guaranteed to run in sequence. If a hook fails to start, its
    // OnStop won't be executed.
    var proc goprocess.Process

    lp.LC.Append(fx.Hook{
        OnStart: func(ctx context.Context) error {
            proc = lp.Proc.Go(f)  // 将 ProcessFunc 包装成 goprocess，并启动
            return nil
        },
        OnStop: func(ctx context.Context) error {
            if proc == nil { // Theoretically this shouldn't ever happen
                return errors.New("lcProcess: proc was nil")  // 如果 proc 为 nil，则返回错误
            }

            return proc.Close() // todo: respect ctx, somehow  // 关闭 goprocess
        },
    })
}

func maybeProvide(opt interface{}, enable bool) fx.Option {
    if enable {
        return fx.Provide(opt)  // 如果 enable 为 true，则提供 opt
    }
    return fx.Options()  // 否则返回空的 fx.Option
}

// nolint unused
func maybeInvoke(opt interface{}, enable bool) fx.Option {
    if enable {
        return fx.Invoke(opt)  // 如果 enable 为 true，则调用 opt
    }
    return fx.Options()  // 否则返回空的 fx.Option
}

// baseProcess creates a goprocess which is closed when the lifecycle signals it to stop
func baseProcess(lc fx.Lifecycle) goprocess.Process {
    p := goprocess.WithParent(goprocess.Background())  // 创建一个具有父进程的 goprocess
    lc.Append(fx.Hook{
        OnStop: func(_ context.Context) error {
            return p.Close()  // 当生命周期信号要求关闭时，关闭 goprocess
        },
    })
    return p
}
```