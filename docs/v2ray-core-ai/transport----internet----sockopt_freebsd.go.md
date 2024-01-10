# `v2ray-core\transport\internet\sockopt_freebsd.go`

```
package internet

import (
    "encoding/binary"  // 导入二进制编码包
    "net"  // 导入网络包
    "os"  // 导入操作系统包
    "syscall"  // 导入系统调用包
    "unsafe"  // 导入不安全的包

    "golang.org/x/sys/unix"  // 导入 Unix 系统包
)

const (
    sysPFINOUT     = 0x0  // 定义常量 sysPFINOUT
    sysPFIN        = 0x1  // 定义常量 sysPFIN
    sysPFOUT       = 0x2  // 定义常量 sysPFOUT
    sysPFFWD       = 0x3  // 定义常量 sysPFFWD
    sysDIOCNATLOOK = 0xc04c4417  // 定义常量 sysDIOCNATLOOK
)

type pfiocNatlook struct {
    Saddr     [16]byte /* pf_addr */  // 定义 pfiocNatlook 结构体的 Saddr 字段
    Daddr     [16]byte /* pf_addr */  // 定义 pfiocNatlook 结构体的 Daddr 字段
    Rsaddr    [16]byte /* pf_addr */  // 定义 pfiocNatlook 结构体的 Rsaddr 字段
    Rdaddr    [16]byte /* pf_addr */  // 定义 pfiocNatlook 结构体的 Rdaddr 字段
    Sport     uint16  // 定义 pfiocNatlook 结构体的 Sport 字段
    Dport     uint16  // 定义 pfiocNatlook 结构体的 Dport 字段
    Rsport    uint16  // 定义 pfiocNatlook 结构体的 Rsport 字段
    Rdport    uint16  // 定义 pfiocNatlook 结构体的 Rdport 字段
    Af        uint8  // 定义 pfiocNatlook 结构体的 Af 字段
    Proto     uint8  // 定义 pfiocNatlook 结构体的 Proto 字段
    Direction uint8  // 定义 pfiocNatlook 结构体的 Direction 字段
    Pad       [1]byte  // 定义 pfiocNatlook 结构体的 Pad 字段
}

const (
    sizeofPfiocNatlook = 0x4c  // 定义常量 sizeofPfiocNatlook
    soReUsePort        = 0x00000200  // 定义常量 soReUsePort
    soReUsePortLB      = 0x00010000  // 定义常量 soReUsePortLB
)

func ioctl(s uintptr, ioc int, b []byte) error {
    if _, _, errno := syscall.Syscall(syscall.SYS_IOCTL, s, uintptr(ioc), uintptr(unsafe.Pointer(&b[0]))); errno != 0 {
        return error(errno)  // 如果系统调用出错，则返回错误
    }
    return nil  // 否则返回空
}
func (nl *pfiocNatlook) rdPort() int {
    return int(binary.BigEndian.Uint16((*[2]byte)(unsafe.Pointer(&nl.Rdport))[:]))  // 读取端口号
}

func (nl *pfiocNatlook) setPort(remote, local int) {
    binary.BigEndian.PutUint16((*[2]byte)(unsafe.Pointer(&nl.Sport))[:], uint16(remote))  // 设置远程端口号
    binary.BigEndian.PutUint16((*[2]byte)(unsafe.Pointer(&nl.Dport))[:], uint16(local))  // 设置本地端口号
}

// OriginalDst uses ioctl to read original destination from /dev/pf
func OriginalDst(la, ra net.Addr) (net.IP, int, error) {
    f, err := os.Open("/dev/pf")  // 打开 /dev/pf 设备文件
    if err != nil {
        return net.IP{}, -1, newError("failed to open device /dev/pf").Base(err)  // 如果打开失败，则返回错误
    }
    defer f.Close()  // 延迟关闭文件
    fd := f.Fd()  // 获取文件描述符
    b := make([]byte, sizeofPfiocNatlook)  // 创建指定大小的字节切片
    nl := (*pfiocNatlook)(unsafe.Pointer(&b[0]))  // 转换为 pfiocNatlook 结构体指针
    var raIP, laIP net.IP  // 声明远程 IP 和本地 IP
    var raPort, laPort int  // 声明远程端口和本地端口
    switch la.(type) {
    case *net.TCPAddr:
        raIP = ra.(*net.TCPAddr).IP  // 获取远程 TCP 地址的 IP
        laIP = la.(*net.TCPAddr).IP  // 获取本地 TCP 地址的 IP
        raPort = ra.(*net.TCPAddr).Port  // 获取远程 TCP 地址的端口
        laPort = la.(*net.TCPAddr).Port  // 获取本地 TCP 地址的端口
        nl.Proto = syscall.IPPROTO_TCP  // 设置协议为 TCP
    # 判断 ra 的类型是否为 net.UDPAddr
    case *net.UDPAddr:
        # 将 ra 转换为 net.UDPAddr 类型，并获取其 IP 地址
        raIP = ra.(*net.UDPAddr).IP
        # 将 la 转换为 net.UDPAddr 类型，并获取其 IP 地址
        laIP = la.(*net.UDPAddr).IP
        # 将 ra 转换为 net.UDPAddr 类型，并获取其端口号
        raPort = ra.(*net.UDPAddr).Port
        # 将 la 转换为 net.UDPAddr 类型，并获取其端口号
        laPort = la.(*net.UDPAddr).Port
        # 设置网络层协议为 UDP
        nl.Proto = syscall.IPPROTO_UDP
    }
    # 如果 raIP 是 IPv4 地址
    if raIP.To4() != nil:
        # 如果 laIP 是未指定的地址，则设置为本地回环地址
        if laIP.IsUnspecified():
            laIP = net.ParseIP("127.0.0.1")
        # 将 raIP 的 IPv4 地址复制到 nl 的源地址中
        copy(nl.Saddr[:net.IPv4len], raIP.To4())
        # 将 laIP 的 IPv4 地址复制到 nl 的目的地址中
        copy(nl.Daddr[:net.IPv4len], laIP.To4())
        # 设置地址族为 IPv4
        nl.Af = syscall.AF_INET
    # 如果 raIP 是 IPv6 地址且不是 IPv4 地址
    if raIP.To16() != nil && raIP.To4() == nil:
        # 如果 laIP 是未指定的地址，则设置为本地回环地址
        if laIP.IsUnspecified():
            laIP = net.ParseIP("::1")
        # 将 raIP 的 IPv6 地址复制到 nl 的源地址中
        copy(nl.Saddr[:], raIP)
        # 将 laIP 的 IPv6 地址复制到 nl 的目的地址中
        copy(nl.Daddr[:], laIP)
        # 设置地址族为 IPv6
        nl.Af = syscall.AF_INET6
    # 设置端口号
    nl.setPort(raPort, laPort)
    # 设置 ioctl 命令
    ioc := uintptr(sysDIOCNATLOOK)
    # 遍历方向数组
    for _, dir := range []byte{sysPFOUT, sysPFIN}:
        # 设置方向
        nl.Direction = dir
        # 调用 ioctl 函数
        err = ioctl(fd, int(ioc), b)
        # 如果没有错误或者错误不是 ENOENT，则跳出循环
        if err == nil || err != syscall.ENOENT:
            break
    }
    # 如果有错误，则返回错误信息
    if err != nil:
        return net.IP{}, -1, os.NewSyscallError("ioctl", err)

    # 读取目的端口号
    odPort := nl.rdPort()
    var odIP net.IP
    # 根据地址族设置目的 IP 地址
    switch nl.Af:
    case syscall.AF_INET:
        odIP = make(net.IP, net.IPv4len)
        copy(odIP, nl.Rdaddr[:net.IPv4len])
    case syscall.AF_INET6:
        odIP = make(net.IP, net.IPv6len)
        copy(odIP, nl.Rdaddr[:])
    }
    # 返回目的 IP 地址和端口号
    return odIP, odPort, nil
}
// 为出站套接字应用选项
func applyOutboundSocketOptions(network string, address string, fd uintptr, config *SocketConfig) error {
    // 如果配置了标记，则设置 SO_USER_COOKIE 选项
    if config.Mark != 0 {
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_USER_COOKIE, int(config.Mark)); err != nil {
            return newError("failed to set SO_USER_COOKIE").Base(err)
        }
    }

    // 如果是 TCP 套接字
    if isTCPSocket(network) {
        // 根据配置的 TFO 设置 TCP_FASTOPEN 选项
        switch config.Tfo {
        case SocketConfig_Enable:
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 1); err != nil {
                return newError("failed to set TCP_FASTOPEN_CONNECT=1").Base(err)
            }
        case SocketConfig_Disable:
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 0); err != nil {
                return newError("failed to set TCP_FASTOPEN_CONNECT=0").Base(err)
            }
        }
    }

    // 如果配置了 Tproxy
    if config.Tproxy.IsEnabled() {
        // 解析地址中的 IP
        ip, _, _ := net.SplitHostPort(address)
        // 如果是 IPv4 地址，则设置 IP_BINDANY 选项
        if net.ParseIP(ip).To4() != nil {
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IP, syscall.IP_BINDANY, 1); err != nil {
                return newError("failed to set outbound IP_BINDANY").Base(err)
            }
        } else {
            // 如果是 IPv6 地址，则设置 IPV6_BINDANY 选项
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IPV6, syscall.IPV6_BINDANY, 1); err != nil {
                return newError("failed to set outbound IPV6_BINDANY").Base(err)
            }
        }
    }
    return nil
}

// 为入站套接字应用选项
func applyInboundSocketOptions(network string, fd uintptr, config *SocketConfig) error {
    // 如果配置了标记，则设置 SO_USER_COOKIE 选项
    if config.Mark != 0 {
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_USER_COOKIE, int(config.Mark)); err != nil {
            return newError("failed to set SO_USER_COOKIE").Base(err)
        }
    }
    # 如果网络是TCP套接字
    if isTCPSocket(network) {
        # 根据配置的TFO选项设置TCP_FASTOPEN选项为1或0
        switch config.Tfo {
        case SocketConfig_Enable:
            # 设置TCP_FASTOPEN选项为1
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 1); err != nil {
                return newError("failed to set TCP_FASTOPEN=1").Base(err)
            }
        case SocketConfig_Disable:
            # 设置TCP_FASTOPEN选项为0
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_TCP, unix.TCP_FASTOPEN, 0); err != nil {
                return newError("failed to set TCP_FASTOPEN=0").Base(err)
            }
        }
    }

    # 如果配置中启用了Tproxy
    if config.Tproxy.IsEnabled() {
        # 设置IPv6套接字绑定任意地址
        if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IPV6, syscall.IPV6_BINDANY, 1); err != nil {
            # 如果设置IPv6失败，则设置IPv4套接字绑定任意地址
            if err := syscall.SetsockoptInt(int(fd), syscall.IPPROTO_IP, syscall.IP_BINDANY, 1); err != nil {
                return newError("failed to set inbound IP_BINDANY").Base(err)
            }
        }
    }

    # 返回空值
    return nil
# 绑定地址和端口
func bindAddr(fd uintptr, ip []byte, port uint32) error:
    # 设置地址重用
    setReuseAddr(fd)
    # 设置端口重用
    setReusePort(fd)

    # 定义套接字地址
    var sockaddr syscall.Sockaddr

    # 根据 IP 地址长度进行不同的处理
    switch len(ip):
        # IPv4 地址
        case net.IPv4len:
            # 创建 IPv4 套接字地址对象
            a4 := &syscall.SockaddrInet4{
                Port: int(port),
            }
            # 将 IP 地址拷贝到套接字地址对象中
            copy(a4.Addr[:], ip)
            sockaddr = a4
        # IPv6 地址
        case net.IPv6len:
            # 创建 IPv6 套接字地址对象
            a6 := &syscall.SockaddrInet6{
                Port: int(port),
            }
            # 将 IP 地址拷贝到套接字地址对象中
            copy(a6.Addr[:], ip)
            sockaddr = a6
        # 其他情况
        default:
            return newError("unexpected length of ip")
    
    # 绑定套接字地址和文件描述符
    return syscall.Bind(int(fd), sockaddr)

# 设置地址重用
func setReuseAddr(fd uintptr) error:
    # 设置套接字选项为地址重用
    if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil:
        return newError("failed to set SO_REUSEADDR").Base(err).AtWarning()
    return nil

# 设置端口重用
func setReusePort(fd uintptr) error:
    # 设置套接字选项为端口重用
    if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, soReUsePortLB, 1); err != nil:
        # 如果设置端口重用失败，则尝试设置另一种端口重用选项
        if err := syscall.SetsockoptInt(int(fd), syscall.SOL_SOCKET, soReUsePort, 1); err != nil:
            return newError("failed to set SO_REUSEPORT").Base(err).AtWarning()
    return nil
```