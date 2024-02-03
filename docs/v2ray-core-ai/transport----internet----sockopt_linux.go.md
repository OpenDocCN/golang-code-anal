# `v2ray-core\transport\internet\sockopt_linux.go`

```go
package internet

import (
    "net"
    "syscall"

    "golang.org/x/sys/unix"
)

const (
    // 用于传入连接
    TCP_FASTOPEN = 23
    // 用于传出连接
    TCP_FASTOPEN_CONNECT = 30
)

func bindAddr(fd uintptr, ip []byte, port uint32) error {
    // 设置地址重用
    setReuseAddr(fd)
    // 设置端口重用
    setReusePort(fd)

    var sockaddr syscall.Sockaddr

    switch len(ip) {
    case net.IPv4len:
        a4 := &syscall.SockaddrInet4{
            Port: int(port),
        }
        copy(a4.Addr[:], ip)
        sockaddr = a4
    case net.IPv6len:
        a6 := &syscall.SockaddrInet6{
            Port: int(port),
        }
        copy(a6.Addr[:], ip)
        sockaddr = a6
    default:
        return newError("unexpected length of ip")
    }

    return syscall.Bind(int(fd), sockaddr)
}

func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
    if config.Mark != 0 {
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK, int(config.Mark)); err != nil {
            return newError("failed to set SO_MARK").Base(err)
        }
    }

    if isTCPSocket(network) {
        switch config.Tfo {
        case SocketConfig_Enable:
            if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN_CONNECT, 1); err != nil {
                return newError("failed to set TCP_FASTOPEN_CONNECT=1").Base(err)
            }
        case SocketConfig_Disable:
            if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN_CONNECT, 0); err != nil {
                return newError("failed to set TCP_FASTOPEN_CONNECT=0").Base(err)
            }
        }
    }

    if config.Tproxy.IsEnabled() {
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
            return newError("failed to set IP_TRANSPARENT").Base(err)
        }
    }

    return nil
}
# 应用入站套接字选项
func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
    # 如果配置了标记
    if config.Mark != 0 {
        # 设置套接字选项 SO_MARK
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK, int(config.Mark)); err != nil {
            return newError("failed to set SO_MARK").Base(err)
        }
    }
    # 如果是 TCP 套接字
    if isTCPSocket(network) {
        # 根据配置设置 TCP_FASTOPEN 选项
        switch config.Tfo {
        case SocketConfig_Enable:
            if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN, 1); err != nil {
                return newError("failed to set TCP_FASTOPEN=1").Base(err)
            }
        case SocketConfig_Disable:
            if err := syscall.SetsockoptInt(int(fd), syscall.SOL_TCP, TCP_FASTOPEN, 0); err != nil {
                return newError("failed to set TCP_FASTOPEN=0").Base(err)
            }
        }
    }
    # 如果配置了 Tproxy
    if config.Tproxy.IsEnabled() {
        # 设置 IP_TRANSPARENT 选项
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
            return newError("failed to set IP_TRANSPARENT").Base(err)
        }
    }
    # 如果需要接收原始目标地址并且是 UDP 套接字
    if config.ReceiveOriginalDestAddress && isUDPSocket(network) {
        # 设置 IPV6_RECVORIGDSTADDR 和 IP_RECVORIGDSTADDR 选项
        err1 := syscall.SetsockoptInt(int(fd), syscall.SOL_IPV6, unix.IPV6_RECVORIGDSTADDR, 1)
        err2 := syscall.SetsockoptInt(int(fd), syscall.SOL_IP, syscall.IP_RECVORIGDSTADDR, 1)
        if err1 != nil && err2 != nil {
            return err1
        }
    }
    # 返回空
    return nil
}

# 设置 SO_REUSEADDR 选项
func setReuseAddr(fd uintptr) error {
    if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil {
        return newError("failed to set SO_REUSEADDR").Base(err).AtWarning()
    }
    return nil
}

# 设置 SO_REUSEPORT 选项
func setReusePort(fd uintptr) error {
    if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, unix.SO_REUSEPORT, 1); err != nil {
        return newError("failed to set SO_REUSEPORT").Base(err).AtWarning()
    }
    return nil
}
```