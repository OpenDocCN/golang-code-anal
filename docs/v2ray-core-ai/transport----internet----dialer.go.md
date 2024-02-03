# `v2ray-core\transport\internet\dialer.go`

```go
package internet

import (
    "context"

    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
)

// Dialer is the interface for dialing outbound connections.
type Dialer interface {
    // Dial dials a system connection to the given destination.
    Dial(ctx context.Context, destination net.Destination) (Connection, error)

    // Address returns the address used by this Dialer. Maybe nil if not known.
    Address() net.Address
}

// dialFunc is an interface to dial network connection to a specific destination.
type dialFunc func(ctx context.Context, dest net.Destination, streamSettings *MemoryStreamConfig) (Connection, error)

var (
    transportDialerCache = make(map[string]dialFunc)
)

// RegisterTransportDialer registers a Dialer with given name.
func RegisterTransportDialer(protocol string, dialer dialFunc) error {
    // 检查是否已经注册了指定协议的拨号器，如果已注册则返回错误
    if _, found := transportDialerCache[protocol]; found {
        return newError(protocol, " dialer already registered").AtError()
    }
    // 将指定协议的拨号器注册到缓存中
    transportDialerCache[protocol] = dialer
    return nil
}

// Dial dials a internet connection towards the given destination.
func Dial(ctx context.Context, dest net.Destination, streamSettings *MemoryStreamConfig) (Connection, error) {
    // 如果目标网络是 TCP
    if dest.Network == net.Network_TCP {
        // 如果流设置为空
        if streamSettings == nil {
            // 创建默认的流设置
            s, err := ToMemoryStreamConfig(nil)
            if err != nil {
                return nil, newError("failed to create default stream settings").Base(err)
            }
            streamSettings = s
        }

        // 获取流设置的协议名称
        protocol := streamSettings.ProtocolName
        // 获取注册的拨号器
        dialer := transportDialerCache[protocol]
        // 如果拨号器为空，则返回错误
        if dialer == nil {
            return nil, newError(protocol, " dialer not registered").AtError()
        }
        // 使用拨号器拨号连接
        return dialer(ctx, dest, streamSettings)
    }
}
    # 如果目标网络是 UDP
    if dest.Network == net.Network_UDP {
        # 从传输拨号器缓存中获取 UDP 拨号器
        udpDialer := transportDialerCache["udp"]
        # 如果未找到 UDP 拨号器，则返回错误
        if udpDialer == nil {
            return nil, newError("UDP dialer not registered").AtError()
        }
        # 使用 UDP 拨号器进行连接
        return udpDialer(ctx, dest, streamSettings)
    }

    # 如果目标网络未知，则返回错误
    return nil, newError("unknown network ", dest.Network)
// DialSystem调用系统拨号器来创建网络连接。
func DialSystem(ctx context.Context, dest net.Destination, sockopt *SocketConfig) (net.Conn, error) {
    // 声明变量src用于存储网络地址
    var src net.Address
    // 从上下文中获取出站信息，如果存在则将网关地址赋给src
    if outbound := session.OutboundFromContext(ctx); outbound != nil {
        src = outbound.Gateway
    }
    // 调用effectiveSystemDialer的Dial方法来创建网络连接，并返回连接和错误信息
    return effectiveSystemDialer.Dial(ctx, src, dest, sockopt)
}
```