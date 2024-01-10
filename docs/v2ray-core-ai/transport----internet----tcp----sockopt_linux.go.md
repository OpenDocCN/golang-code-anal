# `v2ray-core\transport\internet\tcp\sockopt_linux.go`

```
// 仅在 Linux 系统下编译，且不是仅用于配置
// 定义了一个名为 tcp 的包
package tcp

// 导入所需的包
import (
    "syscall"

    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
)

// 定义常量 SO_ORIGINAL_DST 的值为 80
const SO_ORIGINAL_DST = 80

// 获取原始目标地址
func GetOriginalDestination(conn internet.Connection) (net.Destination, error) {
    // 将连接转换为 syscall.Conn 类型
    sysrawconn, f := conn.(syscall.Conn)
    if !f {
        return net.Destination{}, newError("unable to get syscall.Conn")
    }
    // 获取原始连接
    rawConn, err := sysrawconn.SyscallConn()
    if err != nil {
        return net.Destination{}, newError("failed to get sys fd").Base(err)
    }
    // 定义目标地址变量
    var dest net.Destination
    // 控制连接
    err = rawConn.Control(func(fd uintptr) {
        // 获取原始目标地址的 IPv6Mreq 结构
        addr, err := syscall.GetsockoptIPv6Mreq(int(fd), syscall.IPPROTO_IP, SO_ORIGINAL_DST)
        if err != nil {
            newError("failed to call getsockopt").Base(err).WriteToLog()
            return
        }
        // 解析 IP 和端口
        ip := net.IPAddress(addr.Multiaddr[4:8])
        port := uint16(addr.Multiaddr[2])<<8 + uint16(addr.Multiaddr[3])
        // 构建 TCP 目标地址
        dest = net.TCPDestination(ip, net.Port(port))
    })
    if err != nil {
        return net.Destination{}, newError("failed to control connection").Base(err)
    }
    // 检查目标地址是否有效
    if !dest.IsValid() {
        return net.Destination{}, newError("failed to call getsockopt")
    }
    // 返回目标地址和 nil 错误
    return dest, nil
}
```