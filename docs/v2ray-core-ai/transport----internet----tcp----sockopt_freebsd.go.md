# `v2ray-core\transport\internet\tcp\sockopt_freebsd.go`

```
// +build freebsd
// +build !confonly
// 声明该文件只在 freebsd 系统下编译，并且不是 confonly 模式

package tcp
// 导入所需的包
import (
    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
)

// GetOriginalDestination from tcp conn
// 从 TCP 连接中获取原始目的地地址
func GetOriginalDestination(conn internet.Connection) (net.Destination, error) {
    // 获取连接的本地地址
    la := conn.LocalAddr()
    // 获取连接的远程地址
    ra := conn.RemoteAddr()
    // 获取原始目的地地址的 IP 和端口
    ip, port, err := internet.OriginalDst(la, ra)
    // 如果出现错误，返回空的目的地地址和错误信息
    if err != nil {
        return net.Destination{}, newError("failed to get destination").Base(err)
    }
    // 根据获取的 IP 和端口创建 TCP 目的地地址
    dest := net.TCPDestination(net.IPAddress(ip), net.Port(port))
    // 如果目的地地址无效，返回空的目的地地址和错误信息
    if !dest.IsValid() {
        return net.Destination{}, newError("failed to parse destination.")
    }
    // 返回目的地地址和空的错误信息
    return dest, nil
}
```