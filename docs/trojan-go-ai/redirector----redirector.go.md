# `trojan-go\redirector\redirector.go`

```
package redirector

import (
    "context"  // 上下文包，用于控制请求的生命周期
    "io"  // 输入输出包，提供了基本的输入输出功能
    "net"  // 网络包，提供了基本的网络操作函数
    "reflect"  // 反射包，提供了运行时操作结构体、接口和函数等

    "github.com/p4gefau1t/trojan-go/common"  // 引入第三方库
    "github.com/p4gefau1t/trojan-go/log"  // 引入第三方库
)

type Dial func(net.Addr) (net.Conn, error)  // 定义一个函数类型 Dial，用于建立网络连接

func defaultDial(addr net.Addr) (net.Conn, error) {  // 默认的建立网络连接函数
    return net.Dial("tcp", addr.String())  // 使用 TCP 协议建立网络连接
}

type Redirection struct {  // 定义重定向结构体
    Dial  // 函数类型 Dial
    RedirectTo  net.Addr  // 重定向目标地址
    InboundConn net.Conn  // 入站连接
}

type Redirector struct {  // 定义重定向器结构体
    ctx             context.Context  // 上下文
    redirectionChan chan *Redirection  // 重定向通道
}

func (r *Redirector) Redirect(redirection *Redirection) {  // 重定向方法
    select {  // 选择通道操作
    case r.redirectionChan <- redirection:  // 将重定向请求发送到通道
        log.Debug("redirect request")  // 记录调试信息
    case <-r.ctx.Done():  // 当上下文结束时
        log.Debug("exiting")  // 记录调试信息
    }
}

func (r *Redirector) worker() {  // 工作方法
    for {
        select {
        case redirection := <-r.redirectionChan:  # 从重定向通道中接收重定向对象
            handle := func(redirection *Redirection) {  # 定义处理重定向的函数
                if redirection.InboundConn == nil || reflect.ValueOf(redirection.InboundConn).IsNil() {  # 检查入站连接是否为空
                    log.Error("nil inbound conn")  # 记录错误日志
                    return  # 结束处理函数
                }
                defer redirection.InboundConn.Close()  # 延迟关闭入站连接
                if redirection.RedirectTo == nil || reflect.ValueOf(redirection.RedirectTo).IsNil() {  # 检查重定向地址是否为空
                    log.Error("nil redirection addr")  # 记录错误日志
                    return  # 结束处理函数
                }
                if redirection.Dial == nil:  # 检查拨号函数是否为空
                    redirection.Dial = defaultDial  # 设置默认拨号函数
                log.Warn("redirecting connection from", redirection.InboundConn.RemoteAddr(), "to", redirection.RedirectTo.String())  # 记录警告日志，显示重定向连接信息
                outboundConn, err := redirection.Dial(redirection.RedirectTo)  # 拨号到重定向地址
                if err != nil:  # 如果出现错误
                    log.Error(common.NewError("failed to redirect to target address").Base(err))  # 记录错误日志
                    return  # 结束处理函数
                }
                defer outboundConn.Close()  # 延迟关闭出站连接
                errChan := make(chan error, 2)  # 创建错误通道
                copyConn := func(a, b net.Conn) {  # 定义复制连接的函数
                    _, err := io.Copy(a, b)  # 复制连接数据
                    errChan <- err  # 将错误信息发送到错误通道
                }
                go copyConn(outboundConn, redirection.InboundConn)  # 启动复制连接的协程
                go copyConn(redirection.InboundConn, outboundConn)  # 启动复制连接的协程
                select {
                case err := <-errChan:  # 从错误通道中接收错误信息
                    if err != nil {  # 如果出现错误
                        log.Error(common.NewError("failed to redirect").Base(err))  # 记录错误日志
                    }
                    log.Info("redirection done")  # 记录信息日志，显示重定向完成
                case <-r.ctx.Done():  # 如果上下文被取消
                    log.Debug("exiting")  # 记录调试日志，显示退出信息
                    return  # 结束处理函数
                }
            }
            go handle(redirection)  # 启动处理重定向的协程
        case <-r.ctx.Done():  # 如果上下文被取消
            log.Debug("shutting down redirector")  # 记录调试日志，显示关闭重定向器
            return  # 结束循环
        }
    }
# 创建一个新的重定向器对象，传入上下文参数
func NewRedirector(ctx context.Context) *Redirector {
    # 创建一个重定向器对象，并初始化上下文和重定向通道
    r := &Redirector{
        ctx:             ctx,
        redirectionChan: make(chan *Redirection, 64),
    }
    # 启动一个新的 goroutine 来处理重定向任务
    go r.worker()
    # 返回创建的重定向器对象
    return r
}
```