# `v2ray-core\transport\internet\udp\hub_linux.go`

```go
// +build linux
// 声明该文件只在 Linux 系统下编译

package udp
// 声明包名为 udp

import (
    "syscall"
    // 导入 syscall 包，提供了操作系统底层的系统调用接口

    "golang.org/x/sys/unix"
    // 导入 golang.org/x/sys/unix 包，提供了对 Unix 系统的底层访问

    "v2ray.com/core/common/net"
    // 导入 v2ray.com/core/common/net 包，提供了网络相关的通用功能
)

func RetrieveOriginalDest(oob []byte) net.Destination {
    // 从传入的 oob 数据中解析出原始目的地地址
    msgs, err := syscall.ParseSocketControlMessage(oob)
    // 解析传入的 oob 数据，返回消息数组和错误信息
    if err != nil {
        return net.Destination{}
        // 如果解析出错，返回空的 net.Destination 对象
    }
    for _, msg := range msgs {
        // 遍历消息数组
        if msg.Header.Level == syscall.SOL_IP && msg.Header.Type == syscall.IP_RECVORIGDSTADDR {
            // 如果消息的 Level 是 SOL_IP 并且 Type 是 IP_RECVORIGDSTADDR
            ip := net.IPAddress(msg.Data[4:8])
            // 从消息数据中解析出 IP 地址
            port := net.PortFromBytes(msg.Data[2:4])
            // 从消息数据中解析出端口号
            return net.UDPDestination(ip, port)
            // 返回 UDP 目的地地址对象
        } else if msg.Header.Level == syscall.SOL_IPV6 && msg.Header.Type == unix.IPV6_RECVORIGDSTADDR {
            // 如果消息的 Level 是 SOL_IPV6 并且 Type 是 IPV6_RECVORIGDSTADDR
            ip := net.IPAddress(msg.Data[8:24])
            // 从消息数据中解析出 IPv6 地址
            port := net.PortFromBytes(msg.Data[2:4])
            // 从消息数据中解析出端口号
            return net.UDPDestination(ip, port)
            // 返回 UDP 目的地地址对象
        }
    }
    return net.Destination{}
    // 如果没有匹配的消息，返回空的 net.Destination 对象
}

func ReadUDPMsg(conn *net.UDPConn, payload []byte, oob []byte) (int, int, int, *net.UDPAddr, error) {
    // 从 UDP 连接中读取消息和 oob 数据
    return conn.ReadMsgUDP(payload, oob)
    // 调用 UDP 连接的 ReadMsgUDP 方法，返回读取的字节数、消息的长度、oob 数据的长度、消息来源地址和可能的错误
}
```