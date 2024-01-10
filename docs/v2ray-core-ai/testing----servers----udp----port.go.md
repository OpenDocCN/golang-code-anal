# `v2ray-core\testing\servers\udp\port.go`

```
// 导入 UDP 包
package udp

// 导入必要的包
import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
)

// PickPort 函数返回系统中未使用的 UDP 端口。返回的端口高度可能未被使用，但不保证。
func PickPort() net.Port {
    // 监听 UDP4 协议的端口
    conn, err := net.ListenUDP("udp4", &net.UDPAddr{
        IP:   net.LocalHostIP.IP(),  // 使用本地主机 IP 地址
        Port: 0,  // 端口号为 0，表示系统自动分配
    })
    // 如果出现错误，立即终止程序
    common.Must(err)
    // 延迟关闭连接
    defer conn.Close()

    // 获取本地地址
    addr := conn.LocalAddr().(*net.UDPAddr)
    // 返回端口号
    return net.Port(addr.Port)
}
```