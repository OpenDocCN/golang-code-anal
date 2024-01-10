# `v2ray-core\transport\internet\system_listener.go`

```
package internet

import (
    "context"
    "syscall"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
)

// 定义一个默认的监听器
var (
    effectiveListener = DefaultListener{}
)

// 控制器函数类型
type controller func(network, address string, fd uintptr) error

// 默认监听器结构
type DefaultListener struct {
    controllers []controller
}

// 获取控制函数
func getControlFunc(ctx context.Context, sockopt *SocketConfig, controllers []controller) func(network, address string, c syscall.RawConn) error {
    return func(network, address string, c syscall.RawConn) error {
        return c.Control(func(fd uintptr) {
            // 如果有套接字选项，则应用到传入连接
            if sockopt != nil {
                if err := applyInboundSocketOptions(network, fd, sockopt); err != nil {
                    newError("failed to apply socket options to incoming connection").Base(err).WriteToLog(session.ExportIDToError(ctx))
                }
            }

            // 设置重用端口
            setReusePort(fd)

            // 遍历控制器列表，对传入连接进行操作
            for _, controller := range controllers {
                if err := controller(network, address, fd); err != nil {
                    newError("failed to apply external controller").Base(err).WriteToLog(session.ExportIDToError(ctx))
                }
            }
        })
    }
}

// 监听函数
func (dl *DefaultListener) Listen(ctx context.Context, addr net.Addr, sockopt *SocketConfig) (net.Listener, error) {
    var lc net.ListenConfig

    // 设置控制函数
    lc.Control = getControlFunc(ctx, sockopt, dl.controllers)

    return lc.Listen(ctx, addr.Network(), addr.String())
}

// 监听数据包函数
func (dl *DefaultListener) ListenPacket(ctx context.Context, addr net.Addr, sockopt *SocketConfig) (net.PacketConn, error) {
    var lc net.ListenConfig

    // 设置控制函数
    lc.Control = getControlFunc(ctx, sockopt, dl.controllers)

    return lc.ListenPacket(ctx, addr.Network(), addr.String())
}

// 注册监听器控制器，用于在文件描述符被使用之前对其进行操作
//
// v2ray:api:beta
# 注册监听器控制器函数
func RegisterListenerController(controller func(network, address string, fd uintptr) error) error:
    # 如果控制器函数为空，返回新的错误信息
    if controller == nil:
        return newError("nil listener controller")
    
    # 将控制器函数添加到有效监听器的控制器列表中
    effectiveListener.controllers = append(effectiveListener.controllers, controller)
    # 返回空值
    return nil
```