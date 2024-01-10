# `v2ray-core\transport\internet\udp\dialer.go`

```
package udp

import (
    "context"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
)

func init() {
    // 注册传输层拨号器，使用指定的协议名称
    common.Must(internet.RegisterTransportDialer(protocolName,
        // 定义传输层拨号器函数
        func(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
            var sockopt *internet.SocketConfig
            // 如果流设置不为空，则使用流设置中的套接字配置
            if streamSettings != nil {
                sockopt = streamSettings.SocketSettings
            }
            // 使用系统默认方式拨号，获取连接
            conn, err := internet.DialSystem(ctx, dest, sockopt)
            // 如果出现错误，则返回空连接和错误
            if err != nil {
                return nil, err
            }
            // TODO: 处理拨号器选项
            // 返回连接和空错误
            return internet.Connection(conn), nil
        }))
}
```