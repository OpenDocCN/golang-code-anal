# `kubo\core\node\helpers\helpers.go`

```go
package helpers

import (
    "context"  // 导入 context 包

    "go.uber.org/fx"  // 导入 go.uber.org/fx 包
)

type MetricsCtx context.Context  // 定义 MetricsCtx 类型为 context.Context

// LifecycleCtx creates a context which will be cancelled when lifecycle stops
//
// This is a hack which we need because most of our services use contexts in a
// wrong way
func LifecycleCtx(mctx MetricsCtx, lc fx.Lifecycle) context.Context {
    ctx, cancel := context.WithCancel(mctx)  // 使用 mctx 创建一个可取消的 context
    lc.Append(fx.Hook{  // 在 fx.Lifecycle 上添加一个 Hook
        OnStop: func(_ context.Context) error {  // 在停止时执行的函数
            cancel()  // 取消 context
            return nil  // 返回 nil
        },
    })
    return ctx  // 返回创建的 context
}
```