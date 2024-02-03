# `v2ray-core\transport\internet\sockopt_darwin.go`

```go
package internet

import (
    "syscall"
)

const (
    // TCP_FASTOPEN is the socket option on darwin for TCP fast open.
    TCP_FASTOPEN = 0x105
    // TCP_FASTOPEN_SERVER is the value to enable TCP fast open on darwin for server connections.
    TCP_FASTOPEN_SERVER = 0x01
    // TCP_FASTOPEN_CLIENT is the value to enable TCP fast open on darwin for client connections.
    TCP_FASTOPEN_CLIENT = 0x02
)

func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
    // 检查是否为 TCP 套接字
    if isTCPSocket(network) {
        // 根据配置设置 TCP fast open 选项
        switch config.Tfo {
        case SocketConfig_Enable:
            // 启用 TCP fast open 客户端选项
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, TCP_FASTOPEN_CLIENT); err != nil {
                return err
            }
        case SocketConfig_Disable:
            // 禁用 TCP fast open 选项
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, 0); err != nil {
                return err
            }
        }
    }

    return nil
}

func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
    // 检查是否为 TCP 套接字
    if isTCPSocket(network) {
        // 根据配置设置 TCP fast open 选项
        switch config.Tfo {
        case SocketConfig_Enable:
            // 启用 TCP fast open 服务器选项
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, TCP_FASTOPEN_SERVER); err != nil {
                return err
            }
        case SocketConfig_Disable:
            // 禁用 TCP fast open 选项
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, TCP_FASTOPEN, 0); err != nil {
                return err
            }
        }
    }

    return nil
}

func bindAddr(fd uintptr, address []byte, port uint32) error {
    // 绑定地址
    return nil
}

func setReuseAddr(fd uintptr) error {
    // 设置地址重用
    return nil
}

func setReusePort(fd uintptr) error {
    // 设置端口重用
    return nil
}
```