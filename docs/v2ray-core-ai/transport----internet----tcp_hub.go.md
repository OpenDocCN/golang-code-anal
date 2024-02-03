# `v2ray-core\transport\internet\tcp_hub.go`

```go
package internet

import (
    "context"  // 导入 context 包，用于处理上下文
    "v2ray.com/core/common/net"  // 导入 v2ray 网络包
)

var (
    transportListenerCache = make(map[string]ListenFunc)  // 创建一个存储 ListenFunc 的 map，用于缓存传输监听器
)

func RegisterTransportListener(protocol string, listener ListenFunc) error {
    if _, found := transportListenerCache[protocol]; found {  // 如果传输监听器已经注册，则返回错误
        return newError(protocol, " listener already registered.").AtError()
    }
    transportListenerCache[protocol] = listener  // 将传输监听器注册到缓存中
    return nil
}

type ConnHandler func(Connection)  // 定义 ConnHandler 类型，用于处理连接

type ListenFunc func(ctx context.Context, address net.Address, port net.Port, settings *MemoryStreamConfig, handler ConnHandler) (Listener, error)  // 定义 ListenFunc 类型，用于监听连接

type Listener interface {
    Close() error  // 定义 Listener 接口，包含关闭和获取地址的方法
    Addr() net.Addr
}

func ListenTCP(ctx context.Context, address net.Address, port net.Port, settings *MemoryStreamConfig, handler ConnHandler) (Listener, error) {
    if settings == nil {  // 如果设置为空，则创建默认的流设置
        s, err := ToMemoryStreamConfig(nil)
        if err != nil {
            return nil, newError("failed to create default stream settings").Base(err)
        }
        settings = s
    }

    if address.Family().IsDomain() && address.Domain() == "localhost" {  // 如果地址是域名且为 localhost，则使用本地 IP 地址
        address = net.LocalHostIP
    }

    if address.Family().IsDomain() {  // 如果地址是域名，则不允许监听
        return nil, newError("domain address is not allowed for listening: ", address.Domain())
    }

    protocol := settings.ProtocolName  // 获取协议名称
    listenFunc := transportListenerCache[protocol]  // 从缓存中获取传输监听器
    if listenFunc == nil {  // 如果传输监听器未注册，则返回错误
        return nil, newError(protocol, " listener not registered.").AtError()
    }
    listener, err := listenFunc(ctx, address, port, settings, handler)  // 调用传输监听器监听连接
    if err != nil {
        return nil, newError("failed to listen on address: ", address, ":", port).Base(err)  // 如果监听失败，则返回错误
    }
    return listener, nil  // 返回监听器和空错误
}

// ListenSystem listens on a local address for incoming TCP connections.
//
// v2ray:api:beta
func ListenSystem(ctx context.Context, addr net.Addr, sockopt *SocketConfig) (net.Listener, error) {
    return effectiveListener.Listen(ctx, addr, sockopt)  // 监听本地地址的 TCP 连接
}
// ListenSystemPacket函数在本地地址上监听传入的UDP连接。
//
// v2ray:api:beta
func ListenSystemPacket(ctx context.Context, addr net.Addr, sockopt *SocketConfig) (net.PacketConn, error) {
    // 调用effectiveListener的ListenPacket方法，监听指定地址的UDP连接，并返回net.PacketConn和error
    return effectiveListener.ListenPacket(ctx, addr, sockopt)
}
```