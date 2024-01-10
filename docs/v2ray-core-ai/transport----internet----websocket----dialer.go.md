# `v2ray-core\transport\internet\websocket\dialer.go`

```
// +build !confonly

package websocket

import (
    "context"
    "time"

    "github.com/gorilla/websocket"
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
)

// Dial dials a WebSocket connection to the given destination.
func Dial(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
    // 记录创建连接的日志
    newError("creating connection to ", dest).WriteToLog(session.ExportIDToError(ctx))

    // 拨号 WebSocket 连接
    conn, err := dialWebsocket(ctx, dest, streamSettings)
    if err != nil {
        return nil, newError("failed to dial WebSocket").Base(err)
    }
    return internet.Connection(conn), nil
}

func init() {
    // 注册 WebSocket 拨号器
    common.Must(internet.RegisterTransportDialer(protocolName, Dial))
}

func dialWebsocket(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (net.Conn, error) {
    // 获取 WebSocket 配置
    wsSettings := streamSettings.ProtocolSettings.(*Config)

    // 创建 WebSocket 拨号器
    dialer := &websocket.Dialer{
        NetDial: func(network, addr string) (net.Conn, error) {
            return internet.DialSystem(ctx, dest, streamSettings.SocketSettings)
        },
        ReadBufferSize:   4 * 1024,
        WriteBufferSize:  4 * 1024,
        HandshakeTimeout: time.Second * 8,
    }

    protocol := "ws"

    // 根据流设置获取 TLS 配置
    if config := tls.ConfigFromStreamSettings(streamSettings); config != nil {
        protocol = "wss"
        dialer.TLSClientConfig = config.GetTLSConfig(tls.WithDestination(dest), tls.WithNextProto("http/1.1"))
    }

    host := dest.NetAddr()
    // 根据协议和端口设置主机地址
    if (protocol == "ws" && dest.Port == 80) || (protocol == "wss" && dest.Port == 443) {
        host = dest.Address.String()
    }
    uri := protocol + "://" + host + wsSettings.GetNormalizedPath()

    // 拨号 WebSocket 连接
    conn, resp, err := dialer.Dial(uri, wsSettings.GetRequestHeader())
}
    # 如果错误不为空
    if err != nil:
        # 声明 reason 变量
        var reason string
        # 如果响应不为空，将响应状态赋给 reason
        if resp != nil:
            reason = resp.Status
        # 返回一个新的错误，包含拨号失败的原因
        return nil, newError("failed to dial to (", uri, "): ", reason).Base(err)
    
    # 返回一个新的连接和空的错误
    return newConnection(conn, conn.RemoteAddr()), nil
# 闭合前面的函数定义
```