# `kubo\core\builder.go`

```go
package core

import (
    "context" // 引入上下文包
    "fmt" // 引入格式化包
    "reflect" // 引入反射包
    "sync" // 引入同步包
    "time" // 引入时间包

    "github.com/ipfs/boxo/bootstrap" // 引入boxo/bootstrap包
    "github.com/ipfs/kubo/core/node" // 引入kubo/core/node包

    "github.com/ipfs/go-metrics-interface" // 引入go-metrics-interface包
    "go.uber.org/dig" // 引入dig包
    "go.uber.org/fx" // 引入fx包
)

// FXNodeInfo contains information useful for adding fx options.
// This is the extension point for providing more info/context to fx plugins
// to make decisions about what options to include.
type FXNodeInfo struct {
    FXOptions []fx.Option // FXNodeInfo结构体包含FXOptions字段，用于存储fx选项
}

// fxOptFunc takes in some info about the IPFS node and returns the full set of fx opts to use.
type fxOptFunc func(FXNodeInfo) ([]fx.Option, error) // 定义fxOptFunc函数类型，接收FXNodeInfo参数，返回fx选项和错误

var fxOptionFuncs []fxOptFunc // 定义fxOptionFuncs切片，存储fxOptFunc函数

// RegisterFXOptionFunc registers a function that is run before the fx app is initialized.
// Functions are invoked in the order they are registered,
// and the resulting options are passed into the next function's FXNodeInfo.
//
// Note that these are applied globally, by all invocations of NewNode.
// There are multiple places in Kubo that construct nodes, such as:
//   - Repo initialization
//   - Daemon initialization
//   - When running migrations
//   - etc.
//
// If your fx options are doing anything sophisticated, you should keep this in mind.
//
// For example, if you plug in a blockservice that disallows non-allowlisted CIDs,
// this may break migrations that fetch migration code over IPFS.
func RegisterFXOptionFunc(optFunc fxOptFunc) {
    fxOptionFuncs = append(fxOptionFuncs, optFunc) // 注册fxOptFunc函数到fxOptionFuncs切片中
}

// from https://stackoverflow.com/a/59348871
type valueContext struct {
    context.Context // 定义valueContext结构体，包含context.Context字段
}

func (valueContext) Deadline() (deadline time.Time, ok bool) { return } // 实现Deadline方法
func (valueContext) Done() <-chan struct{}                   { return nil } // 实现Done方法
func (valueContext) Err() error                              { return nil } // 实现Err方法

type BuildCfg = node.BuildCfg // Alias for compatibility until we properly refactor the constructor interface

// NewNode constructs and returns an IpfsNode using the given cfg.
func NewNode(ctx context.Context, cfg *BuildCfg) (*IpfsNode, error) {
    // 将传入的上下文保存为“生命周期”上下文
    lctx := ctx

    // 派生一个新的上下文，忽略来自生命周期上下文的取消操作
    ctx, cancel := context.WithCancel(valueContext{ctx})

    // 添加一个指标范围
    ctx = metrics.CtxScope(ctx, "ipfs")

    n := &IpfsNode{
        ctx: ctx,
    }

    opts := []fx.Option{
        node.IPFS(ctx, cfg),
        fx.NopLogger,
    }
    for _, optFunc := range fxOptionFuncs {
        var err error
        opts, err = optFunc(FXNodeInfo{FXOptions: opts})
        if err != nil {
            cancel()
            return nil, fmt.Errorf("building fx opts: %w", err)
        }
    }
    //nolint:staticcheck // https://github.com/ipfs/kubo/pull/9423#issuecomment-1341038770
    opts = append(opts, fx.Extract(n))

    app := fx.New(opts...)

    var once sync.Once
    var stopErr error
    n.stop = func() error {
        once.Do(func() {
            stopErr = app.Stop(context.Background())
            if stopErr != nil {
                log.Error("failure on stop: ", stopErr)
            }
            // 在应用程序停止之后取消上下文
            cancel()
        })
        return stopErr
    }
    n.IsOnline = cfg.Online

    go func() {
        // 如果生命周期上下文被取消，则关闭应用程序
        // 注意：我们应该通过调用`Close()`在进程上停止应用程序。但是我们目前使用上下文来管理所有事务。
        select {
        case <-lctx.Done():
            err := n.stop()
            if err != nil {
                log.Error("failure on stop: ", err)
            }
        case <-ctx.Done():
        }
    }()

    if app.Err() != nil {
        return nil, logAndUnwrapFxError(app.Err())
    }

    if err := app.Start(ctx); err != nil {
        return nil, logAndUnwrapFxError(err)
    }

    // TODO: How soon will bootstrap move to libp2p?
}
    # 如果不是在线模式，则直接返回 n 和空值
    if !cfg.Online:
        return n, nil
    # 如果是在线模式，则调用 n 的 Bootstrap 方法，并传入默认的引导配置参数
    return n, n.Bootstrap(bootstrap.DefaultBootstrapConfig)
// 记录整个 `app.Err()`，但只向用户返回最内层的错误，因为完整的错误可能非常长（因为它可以在单个字符串中暴露整个构建图）。
//
// 通过 `app.Err()` 暴露的 fx.App 错误通常包含其底层 `dig` 包中未导出的错误：
// * https://github.com/uber-go/dig/blob/5e5a20d/error.go#L82
// 这些通常会在许多层中包裹自己，以暴露错误发生在构建链的哪个位置。虽然对于需要调试的开发人员很有用，但对于只想获取 IPFS 错误并且可能在不了解整个链的情况下修复它的用户来说，可能会非常令人困惑。
// 解开所有东西并不是最好的解决方案，因为中间错误中可能包含有用的信息，主要是倒数第二个错误，它可以定位构建错误来自哪个组件，但考虑到 dig 中的所有错误都是私有的，我们只有通用的 `RootCause` API，这是我们目前能做的最好的。
func logAndUnwrapFxError(fxAppErr error) error {
    if fxAppErr == nil {
        return nil
    }

    log.Error("constructing the node: ", fxAppErr)

    err := fxAppErr
    for {
        extractedErr := dig.RootCause(err)
        // 请注意，`RootCause` 名称是误导性的，因为它只是一次解开*一个*错误层，所以我们需要不断调用它。
        if !reflect.TypeOf(extractedErr).Comparable() {
            // 一些内部错误是不可比较的（例如 `dig.errMissingTypes`，它是一个切片），我们无法进一步处理。
            break
        }
        if extractedErr == err {
            // 在上次调用中我们没有解开任何新的错误，达到了最内层的错误。
            break
        }
        err = extractedErr
    }

    return fmt.Errorf("constructing the node (see log for full detail): %w", err)
}
```