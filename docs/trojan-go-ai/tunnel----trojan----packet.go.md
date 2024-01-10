# `trojan-go\tunnel\trojan\packet.go`

```
package trojan

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "encoding/binary"  // 导入 binary 包，用于处理二进制数据的编解码
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "net"  // 导入 net 包，提供了基本的网络功能

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义包 log
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义包 tunnel
)

type PacketConn struct {
    tunnel.Conn  // 定义 PacketConn 结构体，包含 tunnel.Conn 的所有字段和方法
}

func (c *PacketConn) ReadFrom(payload []byte) (int, net.Addr, error) {
    return c.ReadWithMetadata(payload)  // 从连接中读取数据并返回元数据
}

func (c *PacketConn) WriteTo(payload []byte, addr net.Addr) (int, error) {
    address, err := tunnel.NewAddressFromAddr("udp", addr.String())  // 从地址字符串创建新的地址对象
    if err != nil {
        return 0, err  // 如果出错，返回错误
    }
    m := &tunnel.Metadata{  // 创建元数据对象
        Address: address,  // 设置地址字段
    }
    return c.WriteWithMetadata(payload, m)  // 写入数据并附带元数据
}

func (c *PacketConn) WriteWithMetadata(payload []byte, metadata *tunnel.Metadata) (int, error) {
    packet := make([]byte, 0, MaxPacketSize)  // 创建指定大小的字节切片
    w := bytes.NewBuffer(packet)  // 创建一个新的字节缓冲区

    metadata.Address.WriteTo(w)  // 将地址写入缓冲区

    length := len(payload)  // 获取载荷长度
    lengthBuf := [2]byte{}  // 创建长度缓冲区
    crlf := [2]byte{0x0d, 0x0a}  // 创建回车换行符

    binary.BigEndian.PutUint16(lengthBuf[:], uint16(length))  // 将长度转换为大端字节序并写入缓冲区
    w.Write(lengthBuf[:])  // 将长度缓冲区写入字节缓冲区
    w.Write(crlf[:])  // 将回车换行符写入字节缓冲区
    w.Write(payload)  // 将载荷写入字节缓冲区

    _, err := c.Conn.Write(w.Bytes())  // 将字节缓冲区的内容写入连接

    log.Debug("udp packet remote", c.RemoteAddr(), "metadata", metadata, "size", length)  // 记录调试日志
    return len(payload), err  // 返回写入的载荷长度和可能的错误
}

func (c *PacketConn) ReadWithMetadata(payload []byte) (int, *tunnel.Metadata, error) {
    addr := &tunnel.Address{  // 创建地址对象
        NetworkType: "udp",  // 设置网络类型
    }
    if err := addr.ReadFrom(c.Conn); err != nil {  // 从连接中读取地址信息
        return 0, nil, common.NewError("failed to parse udp packet addr").Base(err)  // 如果出错，返回错误
    }
    lengthBuf := [2]byte{}  // 创建长度缓冲区
    if _, err := io.ReadFull(c.Conn, lengthBuf[:]); err != nil {  // 从连接中读取长度信息
        return 0, nil, common.NewError("failed to read length")  // 如果出错，返回错误
    }
    length := int(binary.BigEndian.Uint16(lengthBuf[:]))  // 将长度缓冲区的内容转换为整数

    crlf := [2]byte{}  // 创建回车换行符
    if _, err := io.ReadFull(c.Conn, crlf[:]); err != nil {  // 从连接中读取回车换行符
        return 0, nil, common.NewError("failed to read crlf")  // 如果出错，返回错误
    }
    # 如果 payload 的长度小于 length 或者 length 大于 MaxPacketSize
    if len(payload) < length || length > MaxPacketSize {
        # 从连接中丢弃剩余的数据包
        io.CopyN(ioutil.Discard, c.Conn, int64(length)) // drain the rest of the packet
        # 返回错误，表示传入的数据包大小过大
        return 0, nil, common.NewError("incoming packet size is too large")
    }

    # 从连接中读取指定长度的数据到 payload 中
    if _, err := io.ReadFull(c.Conn, payload[:length]); err != nil {
        # 返回错误，表示读取 payload 失败
        return 0, nil, common.NewError("failed to read payload")
    }

    # 记录 UDP 数据包的来源地址、元数据和大小
    log.Debug("udp packet from", c.RemoteAddr(), "metadata", addr.String(), "size", length)
    # 返回数据包大小、元数据和空错误
    return length, &tunnel.Metadata{
        Address: addr,
    }, nil
# 闭合前面的函数定义
```