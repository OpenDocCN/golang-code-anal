# `v2ray-core\app\commander\commander.go`

```
// +build !confonly
// 定义了一个不是 confonly 的构建标签

package commander
// 定义了 commander 包

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用 go:generate 命令生成错误处理代码

import (
    "context"
    "net"
    "sync"

    "google.golang.org/grpc"

    "v2ray.com/core"
    "v2ray.com/core/common"
    "v2ray.com/core/common/signal/done"
    "v2ray.com/core/features/outbound"
)

// Commander is a V2Ray feature that provides gRPC methods to external clients.
// Commander 是一个 V2Ray 功能，提供给外部客户端使用的 gRPC 方法
type Commander struct {
    sync.Mutex
    server   *grpc.Server
    services []Service
    ohm      outbound.Manager
    tag      string
}

// NewCommander creates a new Commander based on the given config.
// NewCommander 根据给定的配置创建一个新的 Commander
func NewCommander(ctx context.Context, config *Config) (*Commander, error) {
    c := &Commander{
        tag: config.Tag,
    }

    common.Must(core.RequireFeatures(ctx, func(om outbound.Manager) {
        c.ohm = om
    }))

    for _, rawConfig := range config.Service {
        config, err := rawConfig.GetInstance()
        if err != nil {
            return nil, err
        }
        rawService, err := common.CreateObject(ctx, config)
        if err != nil {
            return nil, err
        }
        service, ok := rawService.(Service)
        if !ok {
            return nil, newError("not a Service.")
        }
        c.services = append(c.services, service)
    }

    return c, nil
}

// Type implements common.HasType.
// Type 实现了 common.HasType 接口
func (c *Commander) Type() interface{} {
    return (*Commander)(nil)
}

// Start implements common.Runnable.
// Start 实现了 common.Runnable 接口
func (c *Commander) Start() error {
    c.Lock()
    c.server = grpc.NewServer()
    for _, service := range c.services {
        service.Register(c.server)
    }
    c.Unlock()

    listener := &OutboundListener{
        buffer: make(chan net.Conn, 4),
        done:   done.New(),
    }

    go func() {
        if err := c.server.Serve(listener); err != nil {
            newError("failed to start grpc server").Base(err).AtError().WriteToLog()
        }
    }()
    # 如果尝试移除现有的处理程序时发生错误
    if err := c.ohm.RemoveHandler(context.Background(), c.tag); err != nil:
        # 写入日志记录错误信息
        newError("failed to remove existing handler").WriteToLog()
    
    # 添加处理程序到 ohm 对象中
    return c.ohm.AddHandler(context.Background(), &Outbound{
        tag:      c.tag,  # 设置处理程序的标签
        listener: listener,  # 设置处理程序的监听器
    })
// Close 方法实现了 common.Closable 接口
func (c *Commander) Close() error {
    // 加锁
    c.Lock()
    // 延迟解锁
    defer c.Unlock()

    // 如果服务器不为空，则停止服务器并将其置空
    if c.server != nil {
        c.server.Stop()
        c.server = nil
    }

    // 返回空错误
    return nil
}

// init 函数用于注册配置并初始化 Commander 对象
func init() {
    // 注册配置并初始化 Commander 对象
    common.Must(common.RegisterConfig((*Config)(nil), func(ctx context.Context, cfg interface{}) (interface{}, error) {
        return NewCommander(ctx, cfg.(*Config))
    }))
}
```