# `v2ray-core\common\net\destination.go`

```
package net

import (
    "net"  // 导入 net 包
    "strings"  // 导入 strings 包
)

// Destination represents a network destination including address and protocol (tcp / udp).
type Destination struct {  // 定义 Destination 结构体
    Address Address  // 地址
    Port    Port  // 端口
    Network Network  // 网络协议
}

// DestinationFromAddr generates a Destination from a net address.
func DestinationFromAddr(addr net.Addr) Destination {  // 从 net 地址生成 Destination
    switch addr := addr.(type) {  // 根据地址类型进行判断
    case *net.TCPAddr:  // 如果是 TCP 地址
        return TCPDestination(IPAddress(addr.IP), Port(addr.Port))  // 返回 TCPDestination
    case *net.UDPAddr:  // 如果是 UDP 地址
        return UDPDestination(IPAddress(addr.IP), Port(addr.Port))  // 返回 UDPDestination
    case *net.UnixAddr:  // 如果是 Unix 地址
        // TODO: deal with Unix domain socket  // 处理 Unix 域套接字
        return TCPDestination(LocalHostIP, Port(9))  // 返回 TCPDestination
    default:  // 其他情况
        panic("Net: Unknown address type.")  // 抛出异常
    }
}

// ParseDestination converts a destination from its string presentation.
func ParseDestination(dest string) (Destination, error) {  // 解析目标地址字符串
    d := Destination{  // 创建 Destination 结构体
        Address: AnyIP,  // 地址为 AnyIP
        Port:    Port(0),  // 端口为 0
    }
    if strings.HasPrefix(dest, "tcp:") {  // 如果字符串以 "tcp:" 开头
        d.Network = Network_TCP  // 设置网络协议为 TCP
        dest = dest[4:]  // 截取字符串
    } else if strings.HasPrefix(dest, "udp:") {  // 如果字符串以 "udp:" 开头
        d.Network = Network_UDP  // 设置网络协议为 UDP
        dest = dest[4:]  // 截取字符串
    }

    hstr, pstr, err := SplitHostPort(dest)  // 分割主机和端口
    if err != nil {  // 如果有错误
        return d, err  // 返回错误
    }
    if len(hstr) > 0 {  // 如果主机字符串长度大于 0
        d.Address = ParseAddress(hstr)  // 解析主机地址
    }
    if len(pstr) > 0 {  // 如果端口字符串长度大于 0
        port, err := PortFromString(pstr)  // 将端口字符串转换为端口
        if err != nil {  // 如果有错误
            return d, err  // 返回错误
        }
        d.Port = port  // 设置端口
    }
    return d, nil  // 返回 Destination 结构体和空错误
}

// TCPDestination creates a TCP destination with given address
func TCPDestination(address Address, port Port) Destination {  // 创建具有给定地址的 TCP 目标
    return Destination{  // 返回 Destination 结构体
        Network: Network_TCP,  // 网络协议为 TCP
        Address: address,  // 设置地址
        Port:    port,  // 设置端口
    }
}

// UDPDestination creates a UDP destination with given address
func UDPDestination(address Address, port Port) Destination {  // 创建具有给定地址的 UDP 目标
    return Destination{  // 返回 Destination 结构体
        Network: Network_UDP,  // 网络协议为 UDP
        Address: address,  // 设置地址
        Port:    port,  // 设置端口
    }
}
// NetAddr返回此Destination中网络地址的字符串形式。
func (d Destination) NetAddr() string {
    return d.Address.String() + ":" + d.Port.String()
}

// String返回此Destination的字符串形式。
func (d Destination) String() string {
    prefix := "unknown:"
    switch d.Network {
    case Network_TCP:
        prefix = "tcp:"
    case Network_UDP:
        prefix = "udp:"
    }
    return prefix + d.NetAddr()
}

// IsValid如果此Destination有效，则返回true。
func (d Destination) IsValid() bool {
    return d.Network != Network_Unknown
}

// AsDestination将当前Endpoint转换为Destination。
func (p *Endpoint) AsDestination() Destination {
    return Destination{
        Network: p.Network,
        Address: p.Address.AsAddress(),
        Port:    Port(p.Port),
    }
}
```