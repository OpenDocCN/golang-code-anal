# `v2ray-core\app\log\log.go`

```
// +build !confonly

package log

//go:generate go run v2ray.com/core/common/errors/errorgen

import (
    "context"
    "sync"

    "v2ray.com/core/common"
    "v2ray.com/core/common/log"
)

// Instance is a log.Handler that handles logs.
type Instance struct {
    sync.RWMutex
    config       *Config
    accessLogger log.Handler
    errorLogger  log.Handler
    active       bool
}

// New creates a new log.Instance based on the given config.
func New(ctx context.Context, config *Config) (*Instance, error) {
    g := &Instance{
        config: config,
        active: false,
    }
    log.RegisterHandler(g)

    // start logger instantly on inited
    // other modules would log during init
    if err := g.startInternal(); err != nil {
        return nil, err
    }

    newError("Logger started").AtDebug().WriteToLog()
    return g, nil
}

func (g *Instance) initAccessLogger() error {
    handler, err := createHandler(g.config.AccessLogType, HandlerCreatorOptions{
        Path: g.config.AccessLogPath,
    })
    if err != nil {
        return err
    }
    g.accessLogger = handler
    return nil
}

func (g *Instance) initErrorLogger() error {
    handler, err := createHandler(g.config.ErrorLogType, HandlerCreatorOptions{
        Path: g.config.ErrorLogPath,
    })
    if err != nil {
        return err
    }
    g.errorLogger = handler
    return nil
}

// Type implements common.HasType.
func (*Instance) Type() interface{} {
    return (*Instance)(nil)
}

func (g *Instance) startInternal() error {
    g.Lock()
    defer g.Unlock()

    if g.active {
        return nil
    }

    g.active = true

    if err := g.initAccessLogger(); err != nil {
        return newError("failed to initialize access logger").Base(err).AtWarning()
    }
    if err := g.initErrorLogger(); err != nil {
        return newError("failed to initialize error logger").Base(err).AtWarning()
    }

    return nil
}

// Start implements common.Runnable.Start().
func (g *Instance) Start() error {
    # 调用对象 g 的 startInternal 方法并返回结果
    return g.startInternal()
// Handle 实现了log.Handler接口。
func (g *Instance) Handle(msg log.Message) {
    g.RLock()  // 获取读锁
    defer g.RUnlock()  // 延迟释放读锁

    if !g.active {  // 如果日志实例未激活，则直接返回
        return
    }

    switch msg := msg.(type) {  // 根据消息类型进行处理
    case *log.AccessMessage:  // 如果是访问消息类型
        if g.accessLogger != nil {  // 如果访问日志记录器不为空
            g.accessLogger.Handle(msg)  // 调用访问日志记录器的Handle方法处理消息
        }
    case *log.GeneralMessage:  // 如果是一般消息类型
        if g.errorLogger != nil && msg.Severity <= g.config.ErrorLogLevel {  // 如果错误日志记录器不为空且消息的严重程度小于等于配置的错误日志级别
            g.errorLogger.Handle(msg)  // 调用错误日志记录器的Handle方法处理消息
        }
    default:  // 默认情况下
        // Swallow  // 不做任何处理
    }
}

// Close 实现了common.Closable.Close()接口。
func (g *Instance) Close() error {
    newError("Logger closing").AtDebug().WriteToLog()  // 记录日志：Logger closing

    g.Lock()  // 获取写锁
    defer g.Unlock()  // 延迟释放写锁

    if !g.active {  // 如果日志实例未激活，则直接返回
        return nil
    }

    g.active = false  // 将日志实例标记为未激活

    common.Close(g.accessLogger) // nolint: errcheck  // 关闭访问日志记录器，忽略错误检查
    g.accessLogger = nil  // 将访问日志记录器置空

    common.Close(g.errorLogger) // nolint: errcheck  // 关闭错误日志记录器，忽略错误检查
    g.errorLogger = nil  // 将错误日志记录器置空

    return nil  // 返回空错误
}

func init() {
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, config interface{}) (interface{}, error) {
        return New(ctx, config.(*Config))  // 注册配置并返回新的实例
    }))
}
```