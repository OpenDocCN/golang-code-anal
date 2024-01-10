# `v2ray-core\transport\internet\tcp\dialer.go`

```
// +build !confonly

package tcp

import (
    "context"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
    "v2ray.com/core/transport/internet/xtls"
)

// Dial dials a new TCP connection to the given destination.
func Dial(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
    // 记录正在拨号的 TCP 连接目标地址
    newError("dialing TCP to ", dest).WriteToLog(session.ExportIDToError(ctx))
    // 使用系统默认设置拨号到目标地址
    conn, err := internet.DialSystem(ctx, dest, streamSettings.SocketSettings)
    if err != nil {
        return nil, err
    }

    // 如果存在 TLS 配置，则使用 TLS 进行加密
    if config := tls.ConfigFromStreamSettings(streamSettings); config != nil {
        tlsConfig := config.GetTLSConfig(tls.WithDestination(dest))
        conn = tls.Client(conn, tlsConfig)
    } else if config := xtls.ConfigFromStreamSettings(streamSettings); config != nil {
        xtlsConfig := config.GetXTLSConfig(xtls.WithDestination(dest))
        conn = xtls.Client(conn, xtlsConfig)
    }

    // 获取 TCP 设置
    tcpSettings := streamSettings.ProtocolSettings.(*Config)
    if tcpSettings.HeaderSettings != nil {
        // 获取头部设置实例
        headerConfig, err := tcpSettings.HeaderSettings.GetInstance()
        if err != nil {
            return nil, newError("failed to get header settings").Base(err).AtError()
        }
        // 创建连接认证器
        auth, err := internet.CreateConnectionAuthenticator(headerConfig)
        if err != nil {
            return nil, newError("failed to create header authenticator").Base(err).AtError()
        }
        // 对连接进行认证
        conn = auth.Client(conn)
    }
    // 返回连接
    return internet.Connection(conn), nil
}

// 在程序初始化时注册 TCP 传输拨号器
func init() {
    common.Must(internet.RegisterTransportDialer(protocolName, Dial))
}
```