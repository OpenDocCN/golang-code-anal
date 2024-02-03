# `v2ray-core\transport\pipe\pipe.go`

```go
package pipe

import (
    "context"

    "v2ray.com/core/common/signal"
    "v2ray.com/core/common/signal/done"
    "v2ray.com/core/features/policy"
)

// Option for creating new Pipes.
type Option func(*pipeOption)

// WithoutSizeLimit returns an Option for Pipe to have no size limit.
func WithoutSizeLimit() Option {
    return func(opt *pipeOption) {
        opt.limit = -1
    }
}

// WithSizeLimit returns an Option for Pipe to have the given size limit.
func WithSizeLimit(limit int32) Option {
    return func(opt *pipeOption) {
        opt.limit = limit
    }
}

// DiscardOverflow returns an Option for Pipe to discard writes if full.
func DiscardOverflow() Option {
    return func(opt *pipeOption) {
        opt.discardOverflow = true
    }
}

// OptionsFromContext returns a list of Options from context.
func OptionsFromContext(ctx context.Context) []Option {
    var opt []Option

    // 从上下文中获取缓冲策略
    bp := policy.BufferPolicyFromContext(ctx)
    // 如果每个连接的缓冲大小大于等于0，则设置为有限制的缓冲大小
    if bp.PerConnection >= 0 {
        opt = append(opt, WithSizeLimit(bp.PerConnection))
    } else {
        // 否则设置为无限制的缓冲大小
        opt = append(opt, WithoutSizeLimit())
    }

    return opt
}

// New creates a new Reader and Writer that connects to each other.
func New(opts ...Option) (*Reader, *Writer) {
    // 创建一个新的管道对象
    p := &pipe{
        readSignal:  signal.NewNotifier(),
        writeSignal: signal.NewNotifier(),
        done:        done.New(),
        option: pipeOption{
            limit: -1,
        },
    }

    // 遍历所有的选项，并应用到管道对象上
    for _, opt := range opts {
        opt(&(p.option))
    }

    // 返回新创建的 Reader 和 Writer 对象
    return &Reader{
            pipe: p,
        }, &Writer{
            pipe: p,
        }
}
```