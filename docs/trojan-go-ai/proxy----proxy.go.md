# `trojan-go\proxy\proxy.go`

```go
package proxy

import (
    "context"  // 上下文包，用于控制请求的取消、超时和截止
    "io"  // 提供了基本的接口类型和函数，用于读写数据
    "math/rand"  // 生成伪随机数
    "net"  // 提供了基本的网络接口
    "os"  // 提供了操作系统功能
    "strings"  // 提供了操作字符串的函数

    "github.com/p4gefau1t/trojan-go/common"  // 引入自定义的 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 引入自定义的 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 引入自定义的 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 引入自定义的 tunnel 包
)

const Name = "PROXY"  // 定义常量 Name 为 "PROXY"

const (
    MaxPacketSize = 1024 * 8  // 定义常量 MaxPacketSize 为 1024 * 8
)

// Proxy relay connections and packets
type Proxy struct {
    sources []tunnel.Server  // 定义 sources 为 tunnel.Server 类型的切片
    sink    tunnel.Client  // 定义 sink 为 tunnel.Client 类型
    ctx     context.Context  // 定义 ctx 为 context.Context 类型
    cancel  context.CancelFunc  // 定义 cancel 为 context.CancelFunc 类型
}

func (p *Proxy) Run() error {
    p.relayConnLoop()  // 调用 relayConnLoop 方法
    p.relayPacketLoop()  // 调用 relayPacketLoop 方法
    <-p.ctx.Done()  // 从 ctx.Done() 通道中接收数据
    return nil  // 返回空值
}

func (p *Proxy) Close() error {
    p.cancel()  // 调用 cancel 方法
    p.sink.Close()  // 调用 sink 的 Close 方法
    for _, source := range p.sources {  // 遍历 sources 切片
        source.Close()  // 调用 source 的 Close 方法
    }
    return nil  // 返回空值
}

func (p *Proxy) relayConnLoop() {
    # 遍历 p.sources 切片中的每个元素，使用下划线 _ 忽略索引值
    for _, source := range p.sources {
        # 启动一个 goroutine 处理每个 source 对象
        go func(source tunnel.Server) {
            # 无限循环，接受传入的连接
            for {
                # 接受传入的连接，如果有错误则返回错误信息
                inbound, err := source.AcceptConn(nil)
                if err != nil {
                    # 如果有错误，检查是否上下文已经结束，如果是则退出循环
                    select {
                    case <-p.ctx.Done():
                        log.Debug("exiting")
                        return
                    default:
                    }
                    # 记录错误信息并继续下一次循环
                    log.Error(common.NewError("failed to accept connection").Base(err))
                    continue
                }
                # 启动一个 goroutine 处理传入的连接
                go func(inbound tunnel.Conn) {
                    # 在函数结束时关闭传入的连接
                    defer inbound.Close()
                    # 使用传入连接的地址进行拨号连接
                    outbound, err := p.sink.DialConn(inbound.Metadata().Address, nil)
                    if err != nil {
                        # 如果有错误，记录错误信息并返回
                        log.Error(common.NewError("proxy failed to dial connection").Base(err))
                        return
                    }
                    # 在函数结束时关闭拨号连接
                    defer outbound.Close()
                    # 创建一个错误通道，用于传递拷贝连接时的错误
                    errChan := make(chan error, 2)
                    # 定义一个函数用于拷贝连接数据
                    copyConn := func(a, b net.Conn) {
                        _, err := io.Copy(a, b)
                        errChan <- err
                    }
                    # 启动两个 goroutine 分别拷贝连接数据
                    go copyConn(inbound, outbound)
                    go copyConn(outbound, inbound)
                    # 选择处理错误或者上下文结束的情况
                    select {
                    case err = <-errChan:
                        if err != nil {
                            # 如果有错误，记录错误信息
                            log.Error(err)
                        }
                    case <-p.ctx.Done():
                        # 如果上下文结束，记录信息并返回
                        log.Debug("shutting down conn relay")
                        return
                    }
                    # 记录连接中继结束的信息
                    log.Debug("conn relay ends")
                }(inbound)
            }
        }(source)
    }
}

func (p *Proxy) relayPacketLoop() {
    // 实现代理的数据传输循环
}

func NewProxy(ctx context.Context, cancel context.CancelFunc, sources []tunnel.Server, sink tunnel.Client) *Proxy {
    // 创建一个新的代理对象
    return &Proxy{
        sources: sources,
        sink:    sink,
        ctx:     ctx,
        cancel:  cancel,
    }
}

type Creator func(ctx context.Context) (*Proxy, error)

var creators = make(map[string]Creator)

func RegisterProxyCreator(name string, creator Creator) {
    // 注册代理创建函数
    creators[name] = creator
}

func NewProxyFromConfigData(data []byte, isJSON bool) (*Proxy, error) {
    // 为每个代理实例创建一个唯一的上下文，以避免重复的认证器
    ctx := context.WithValue(context.Background(), Name+"_ID", rand.Int())
    var err error
    if isJSON {
        ctx, err = config.WithJSONConfig(ctx, data)
        if err != nil {
            return nil, err
        }
    } else {
        ctx, err = config.WithYAMLConfig(ctx, data)
        if err != nil {
            return nil, err
        }
    }
    cfg := config.FromContext(ctx, Name).(*Config)
    create, ok := creators[strings.ToUpper(cfg.RunType)]
    if !ok {
        return nil, common.NewError("unknown proxy type: " + cfg.RunType)
    }
    log.SetLogLevel(log.LogLevel(cfg.LogLevel))
    if cfg.LogFile != "" {
        file, err := os.OpenFile(cfg.LogFile, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0o644)
        if err != nil {
            return nil, common.NewError("failed to open log file").Base(err)
        }
        log.SetOutput(file)
    }
    return create(ctx)
}
```