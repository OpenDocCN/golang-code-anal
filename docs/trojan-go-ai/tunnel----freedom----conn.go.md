# `trojan-go\tunnel\freedom\conn.go`

```
package freedom

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "net"    // 导入 net 包，提供了基本的网络功能

    "github.com/txthinking/socks5"  // 导入 socks5 包，实现 SOCKS5 代理协议

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包，包含了一些通用的函数和常量
    "github.com/p4gefau1t/trojan-go/log"  // 导入 log 包，用于日志记录
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 tunnel 包，用于处理数据传输
)

const MaxPacketSize = 1024 * 8  // 定义最大数据包大小为 1024 * 8

type Conn struct {
    net.Conn  // 定义 Conn 结构体，包含 net.Conn 接口
}

func (c *Conn) Metadata() *tunnel.Metadata {
    return nil  // Conn 结构体的 Metadata 方法，返回空指针
}

type PacketConn struct {
    *net.UDPConn  // 定义 PacketConn 结构体，包含 net.UDPConn 指针
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
    return c.WriteTo(p, m.Address)  // PacketConn 结构体的 WriteWithMetadata 方法，将数据写入 UDP 连接
}

func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
    n, addr, err := c.ReadFrom(p)  // 从 UDP 连接中读取数据
    if err != nil {
        return 0, nil, err
    }
    address, err := tunnel.NewAddressFromAddr("udp", addr.String())  // 从地址字符串创建新的地址对象
    common.Must(err)  // 检查错误
    metadata := &tunnel.Metadata{  // 创建元数据对象
        Address: address,  // 设置地址
    }
    return n, metadata, nil  // 返回读取的数据长度和元数据对象
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error) {
    if udpAddr, ok := addr.(*net.UDPAddr); ok {  // 检查地址类型是否为 UDP 地址
        return c.WriteToUDP(p, udpAddr)  // 如果是 UDP 地址，直接写入数据
    }
    ip, err := addr.(*tunnel.Address).ResolveIP()  // 解析地址对应的 IP
    if err != nil {
        return 0, err
    }
    udpAddr := &net.UDPAddr{  // 创建新的 UDP 地址
        IP:   ip,  // 设置 IP
        Port: addr.(*tunnel.Address).Port,  // 设置端口
    }
    return c.WriteToUDP(p, udpAddr)  // 写入数据到新的 UDP 地址
}

type SocksPacketConn struct {
    net.PacketConn  // 定义 SocksPacketConn 结构体，包含 net.PacketConn 接口
    socksAddr   *net.UDPAddr  // 定义 socksAddr，表示 SOCKS5 代理地址
    socksClient *socks5.Client  // 定义 socksClient，表示 SOCKS5 客户端
}

func (c *SocksPacketConn) WriteWithMetadata(payload []byte, metadata *tunnel.Metadata) (int, error) {
    buf := bytes.NewBuffer(make([]byte, 0, MaxPacketSize))  // 创建一个缓冲区
    buf.Write([]byte{0, 0, 0}) // RSV, FRAG  // 写入特定字节
    common.Must(metadata.Address.WriteTo(buf))  // 将元数据地址写入缓冲区
    buf.Write(payload)  // 将数据写入缓冲区
    _, err := c.PacketConn.WriteTo(buf.Bytes(), c.socksAddr)  // 将缓冲区的数据写入到 SOCKS5 代理地址
    if err != nil {
        return 0, err
    }
    log.Debug("sent udp packet to " + c.socksAddr.String() + " with metadata " + metadata.String())  // 记录日志
    return len(payload), nil  // 返回写入的数据长度
}

func (c *SocksPacketConn) ReadWithMetadata(payload []byte) (int, *tunnel.Metadata, error) {
    // 未完待续
}
    # 创建一个长度为 MaxPacketSize 的字节切片
    buf := make([]byte, MaxPacketSize)
    # 从连接中读取数据到 buf 中，并返回读取的字节数、来源地址和可能的错误
    n, from, err := c.PacketConn.ReadFrom(buf)
    # 如果有错误发生，则返回 0、nil 和错误信息
    if err != nil {
        return 0, nil, err
    }
    # 记录接收到的 UDP 数据包的来源地址
    log.Debug("recv udp packet from " + from.String())
    # 创建一个新的地址对象
    addr := new(tunnel.Address)
    # 从 buf 中的第 3 个字节开始到 n 字节的数据读取到 r 中
    r := bytes.NewBuffer(buf[3:n])
    # 从 r 中读取地址信息到 addr 中，如果出现错误则返回错误信息
    if err := addr.ReadFrom(r); err != nil {
        return 0, nil, common.NewError("socks5 failed to parse addr in the packet").Base(err)
    }
    # 从 r 中读取数据到 payload 中，并返回读取的字节数和可能的错误
    length, err := r.Read(payload)
    # 如果有错误发生，则返回 0、nil 和错误信息
    if err != nil {
        return 0, nil, err
    }
    # 返回读取的数据长度、包含地址信息的元数据对象和 nil
    return length, &tunnel.Metadata{
        Address: addr,
    }, nil
# 关闭 SocksPacketConn 对象，关闭其内部的 socksClient 对象
func (c *SocksPacketConn) Close() error:
    c.socksClient.Close()  # 关闭 socksClient 对象
    return c.PacketConn.Close()  # 关闭 PacketConn 对象并返回错误信息
```