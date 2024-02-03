# `trojan-go\tunnel\tproxy\tcp.go`

```go
//go:build linux
// +build linux

package tproxy

import (
    "fmt"  // 导入 fmt 包，用于格式化输出
    "net"  // 导入 net 包，用于网络操作
    "os"   // 导入 os 包，用于操作系统功能
    "syscall"  // 导入 syscall 包，用于底层系统调用
    "unsafe"   // 导入 unsafe 包，用于底层数据类型操作
)

// Listener 描述了一个带有 Linux IP_TRANSPARENT 选项的 TCP 监听器
type Listener struct {
    base net.Listener  // 基础的网络监听器
}

// Accept 等待并返回监听器的下一个连接
//
// 这个命令包装了 Listener 的 AcceptTProxy 方法
func (listener *Listener) Accept() (net.Conn, error) {
    tcpConn, err := listener.base.(*net.TCPListener).AcceptTCP()  // 接受 TCP 连接
    if err != nil {
        return nil, err
    }

    return tcpConn, nil
}

// Addr 返回监听器正在接受连接的网络地址
func (listener *Listener) Addr() net.Addr {
    return listener.base.Addr()  // 返回监听器的地址
}

// Close 将关闭监听器，不再接受任何连接
func (listener *Listener) Close() error {
    return listener.base.Close()  // 关闭监听器
}

// ListenTCP 将构建一个带有 Linux IP_TRANSPARENT 选项的新 TCP 监听器
func ListenTCP(network string, laddr *net.TCPAddr) (net.Listener, error) {
    listener, err := net.ListenTCP(network, laddr)  // 监听 TCP 连接
    if err != nil {
        return nil, err
    }

    fileDescriptorSource, err := listener.File()  // 获取文件描述符
    if err != nil {
        return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("get file descriptor: %s", err)}  // 返回获取文件描述符错误
    }
    defer fileDescriptorSource.Close()  // 延迟关闭文件描述符

    if err = syscall.SetsockoptInt(int(fileDescriptorSource.Fd()), syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
        return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}  // 返回设置套接字选项错误
    }

    return &Listener{listener}, nil  // 返回监听器
}

const (
    IP6T_SO_ORIGINAL_DST = 80  // 定义 IP6T_SO_ORIGINAL_DST 常量
    SO_ORIGINAL_DST      = 80  // 定义 SO_ORIGINAL_DST 常量
)

// getOriginalTCPDest 从原始目标地址检索原始目标 TCP 地址
// 获取原始的 TCP 目的地址，用于处理 NAT 连接。目前只支持使用 DNAT/REDIRECT 的 Linux iptables。
// 对于其他操作系统，将返回 conn.LocalAddr()。
//
// 注意，此函数仅在内核中加载了 nf_conntrack_ipv4 和/或 nf_conntrack_ipv6 时才有效。
func getOriginalTCPDest(conn *net.TCPConn) (*net.TCPAddr, error) {
    // 获取文件描述符
    f, err := conn.File()
    if err != nil {
        return nil, err
    }
    defer f.Close()

    fd := int(f.Fd())
    // 恢复为非阻塞模式
    // 参考 http://stackoverflow.com/a/28968431/1493661
    if err = syscall.SetNonblock(fd, true); err != nil {
        return nil, os.NewSyscallError("setnonblock", err)
    }

    v6 := conn.LocalAddr().(*net.TCPAddr).IP.To4() == nil
    if v6 {
        var addr syscall.RawSockaddrInet6
        var len uint32
        len = uint32(unsafe.Sizeof(addr))
        // 获取 IPv6 的原始目的地址
        err = getsockopt(fd, syscall.IPPROTO_IPV6, IP6T_SO_ORIGINAL_DST,
            unsafe.Pointer(&addr), &len)
        if err != nil {
            return nil, os.NewSyscallError("getsockopt", err)
        }
        ip := make([]byte, 16)
        for i, b := range addr.Addr {
            ip[i] = b
        }
        pb := *(*[2]byte)(unsafe.Pointer(&addr.Port))
        return &net.TCPAddr{
            IP:   ip,
            Port: int(pb[0])*256 + int(pb[1]),
        }, nil
    }

    // IPv4
    var addr syscall.RawSockaddrInet4
    var len uint32
    len = uint32(unsafe.Sizeof(addr))
    // 获取 IPv4 的原始目的地址
    err = getsockopt(fd, syscall.IPPROTO_IP, SO_ORIGINAL_DST,
        unsafe.Pointer(&addr), &len)
    if err != nil {
        return nil, os.NewSyscallError("getsockopt", err)
    }
    ip := make([]byte, 4)
    for i, b := range addr.Addr {
        ip[i] = b
    }
    pb := *(*[2]byte)(unsafe.Pointer(&addr.Port))
    return &net.TCPAddr{
        IP:   ip,
        Port: int(pb[0])*256 + int(pb[1]),
    }, nil
}
```