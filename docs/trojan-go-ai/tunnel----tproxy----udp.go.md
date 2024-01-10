# `trojan-go\tunnel\tproxy\udp.go`

```
// 根据操作系统构建条件编译标记，指定只在 Linux 系统下编译
// 导入 tproxy 包
package tproxy

// 导入所需的包
import (
    "bytes"  // 导入 bytes 包，用于操作二进制数据
    "encoding/binary"  // 导入 binary 包，用于处理二进制数据的编解码
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "net"  // 导入 net 包，用于网络编程
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "strconv"  // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "syscall"  // 导入 syscall 包，提供了操作系统底层接口的访问
    "unsafe"  // 导入 unsafe 包，提供了 Go 语言中访问未导出字段的能力
)

// ListenUDP 函数将构建一个带有 Linux IP_TRANSPARENT 选项设置的新 UDP 监听器套接字
func ListenUDP(network string, laddr *net.UDPAddr) (*net.UDPConn, error) {
    // 使用给定的网络类型和本地地址创建一个 UDP 监听器
    listener, err := net.ListenUDP(network, laddr)
    if err != nil {
        return nil, err
    }

    // 获取监听器的文件描述符
    fileDescriptorSource, err := listener.File()
    if err != nil {
        return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("get file descriptor: %s", err)}
    }
    defer fileDescriptorSource.Close()  // 延迟关闭文件描述符

    // 获取文件描述符的整数值
    fileDescriptor := int(fileDescriptorSource.Fd())
    // 设置文件描述符的选项为 IP_TRANSPARENT
    if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
        return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}
    }

    // 设置文件描述符的选项为 IP_RECVORIGDSTADDR
    if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_RECVORIGDSTADDR, 1); err != nil {
        return nil, &net.OpError{Op: "listen", Net: network, Source: nil, Addr: laddr, Err: fmt.Errorf("set socket option: IP_RECVORIGDSTADDR: %s", err)}
    }

    // 返回监听器和空错误
    return listener, nil
}

// ReadFromUDP 从连接中读取一个 UDP 数据包，将有效载荷复制到 b 中
// 返回复制到 b 中的字节数以及数据包的返回地址
//
// 还会读取带外数据，以便识别和解析原始目标地址
func ReadFromUDP(conn *net.UDPConn, b []byte) (int, *net.UDPAddr, *net.UDPAddr, error) {
    oob := make([]byte, 1024)  // 创建一个长度为 1024 的字节切片
    // 从连接中读取 UDP 数据包和带外数据
    n, oobn, _, addr, err := conn.ReadMsgUDP(b, oob)
    if err != nil {
        return 0, nil, nil, err
    }

    // 解析带外数据中的套接字控制消息
    msgs, err := syscall.ParseSocketControlMessage(oob[:oobn])
    # 如果发生错误，则返回错误信息
    if err != nil:
        return 0, nil, nil, fmt.Errorf("parsing socket control message: %s", err)
    
    # 声明 originalDst 变量，用于存储原始目标地址
    var originalDst *net.UDPAddr
    
    # 遍历消息列表
    for _, msg := range msgs:
        # 检查消息头部的 Level 和 Type 是否符合条件
        if (msg.Header.Level == syscall.SOL_IP || msg.Header.Level == syscall.SOL_IPV6) && msg.Header.Type == syscall.IP_RECVORIGDSTADDR:
            # 声明 originalDstRaw 变量，用于存储原始目标地址的原始数据
            originalDstRaw := &syscall.RawSockaddrInet4{}
            # 从消息数据中读取原始目标地址的原始数据，并进行解析
            if err = binary.Read(bytes.NewReader(msg.Data), binary.LittleEndian, originalDstRaw); err != nil:
                return 0, nil, nil, fmt.Errorf("reading original destination address: %s", err)
            
            # 根据原始地址的协议类型进行不同的处理
            switch originalDstRaw.Family:
                # 如果是 IPv4 地址
                case syscall.AF_INET:
                    # 将原始数据转换为 IPv4 地址和端口
                    pp := (*syscall.RawSockaddrInet4)(unsafe.Pointer(originalDstRaw))
                    p := (*[2]byte)(unsafe.Pointer(&pp.Port))
                    originalDst = &net.UDPAddr{
                        IP:   net.IPv4(pp.Addr[0], pp.Addr[1], pp.Addr[2], pp.Addr[3]),
                        Port: int(p[0])<<8 + int(p[1]),
                    }
                
                # 如果是 IPv6 地址
                case syscall.AF_INET6:
                    # 将原始数据转换为 IPv6 地址和端口
                    pp := (*syscall.RawSockaddrInet6)(unsafe.Pointer(originalDstRaw))
                    p := (*[2]byte)(unsafe.Pointer(&pp.Port))
                    originalDst = &net.UDPAddr{
                        IP:   net.IP(pp.Addr[:]),
                        Port: int(p[0])<<8 + int(p[1]),
                        Zone: strconv.Itoa(int(pp.Scope_id)),
                    }
                
                # 如果是其他类型的地址
                default:
                    return 0, nil, nil, fmt.Errorf("original destination is an unsupported network family")
    
    # 如果未能获取到原始目标地址，则返回错误信息
    if originalDst == nil:
        return 0, nil, nil, fmt.Errorf("unable to obtain original destination: %s", err)
    
    # 返回结果
    return n, addr, originalDst, nil
// DialUDP连接到网络net上的远程地址raddr，net必须是"udp"、"udp4"或"udp6"。如果laddr不是nil，则用作连接的本地地址。
func DialUDP(network string, laddr *net.UDPAddr, raddr *net.UDPAddr) (*net.UDPConn, error) {
    // 将UDP地址转换为套接字地址
    remoteSocketAddress, err := udpAddrToSocketAddr(raddr)
    if err != nil {
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("build destination socket address: %s", err)}
    }

    // 将UDP地址转换为套接字地址
    localSocketAddress, err := udpAddrToSocketAddr(laddr)
    if err != nil {
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("build local socket address: %s", err)}
    }

    // 打开一个套接字
    fileDescriptor, err := syscall.Socket(udpAddrFamily(network, laddr, raddr), syscall.SOCK_DGRAM, 0)
    if err != nil {
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket open: %s", err)}
    }

    // 设置套接字选项SO_REUSEADDR
    if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1); err != nil {
        syscall.Close(fileDescriptor)
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("set socket option: SO_REUSEADDR: %s", err)}
    }

    // 设置套接字选项IP_TRANSPARENT
    if err = syscall.SetsockoptInt(fileDescriptor, syscall.SOL_IP, syscall.IP_TRANSPARENT, 1); err != nil {
        syscall.Close(fileDescriptor)
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("set socket option: IP_TRANSPARENT: %s", err)}
    }

    // 将套接字绑定到本地地址
    if err = syscall.Bind(fileDescriptor, localSocketAddress); err != nil {
        syscall.Close(fileDescriptor)
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket bind: %s", err)}
    }

    // 将套接字连接到远程地址
    if err = syscall.Connect(fileDescriptor, remoteSocketAddress); err != nil {
        syscall.Close(fileDescriptor)
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("socket connect: %s", err)}
    }

    // 创建一个新的文件描述符
    fdFile := os.NewFile(uintptr(fileDescriptor), fmt.Sprintf("net-udp-dial-%s", raddr.String()))
    // 延迟关闭文件描述符
    defer fdFile.Close()

    // 从文件描述符创建一个UDP连接
    remoteConn, err := net.FileConn(fdFile)
    # 如果发生错误
    if err != nil:
        # 关闭文件描述符
        syscall.Close(fileDescriptor)
        # 返回空和网络操作错误，包括操作和错误信息
        return nil, &net.OpError{Op: "dial", Err: fmt.Errorf("convert file descriptor to connection: %s", err)}
    
    # 返回远程连接和空
    return remoteConn.(*net.UDPConn), nil
// udpAddToSockerAddr将UDPAddr转换为Sockaddr，可用于连接和绑定套接字
func udpAddrToSocketAddr(addr *net.UDPAddr) (syscall.Sockaddr, error) {
    switch {
    case addr.IP.To4() != nil: // 如果IP地址是IPv4
        ip := [4]byte{}
        copy(ip[:], addr.IP.To4()) // 将IPv4地址复制到数组中

        return &syscall.SockaddrInet4{Addr: ip, Port: addr.Port}, nil // 返回IPv4的Sockaddr

    default: // 如果IP地址是IPv6
        ip := [16]byte{}
        copy(ip[:], addr.IP.To16()) // 将IPv6地址复制到数组中

        zoneID, err := strconv.ParseUint(addr.Zone, 10, 32) // 解析区域ID
        if err != nil {
            return nil, err
        }

        return &syscall.SockaddrInet6{Addr: ip, Port: addr.Port, ZoneId: uint32(zoneID)}, nil // 返回IPv6的Sockaddr
    }
}

// udpAddrFamily将尝试根据网络和UDP地址来确定地址族
func udpAddrFamily(net string, laddr, raddr *net.UDPAddr) int {
    switch net[len(net)-1] { // 获取网络的最后一个字符
    case '4': // 如果是IPv4
        return syscall.AF_INET
    case '6': // 如果是IPv6
        return syscall.AF_INET6
    }

    if (laddr == nil || laddr.IP.To4() != nil) && (raddr == nil || laddr.IP.To4() != nil) { // 如果本地地址是IPv4并且远程地址是IPv4
        return syscall.AF_INET // 返回IPv4地址族
    }
    return syscall.AF_INET6 // 默认返回IPv6地址族
}
```