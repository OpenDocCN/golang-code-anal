# `v2ray-core\common\mux\frame.go`

```go
package mux

import (
    "encoding/binary"  // 导入编码二进制数据的包
    "io"  // 导入输入输出操作的包

    "v2ray.com/core/common"  // 导入通用功能的包
    "v2ray.com/core/common/bitmask"  // 导入位掩码操作的包
    "v2ray.com/core/common/buf"  // 导入缓冲区操作的包
    "v2ray.com/core/common/net"  // 导入网络操作的包
    "v2ray.com/core/common/protocol"  // 导入协议相关的包
    "v2ray.com/core/common/serial"  // 导入序列化操作的包
)

type SessionStatus byte  // 定义会话状态类型

const (
    SessionStatusNew       SessionStatus = 0x01  // 新会话状态
    SessionStatusKeep      SessionStatus = 0x02  // 保持会话状态
    SessionStatusEnd       SessionStatus = 0x03  // 结束会话状态
    SessionStatusKeepAlive SessionStatus = 0x04  // 保持连接状态
)

const (
    OptionData  bitmask.Byte = 0x01  // 数据选项
    OptionError bitmask.Byte = 0x02  // 错误选项
)

type TargetNetwork byte  // 目标网络类型

const (
    TargetNetworkTCP TargetNetwork = 0x01  // TCP 网络
    TargetNetworkUDP TargetNetwork = 0x02  // UDP 网络
)

var addrParser = protocol.NewAddressParser(  // 创建地址解析器
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv4), net.AddressFamilyIPv4),  // IPv4地址类型
    protocol.AddressFamilyByte(byte(protocol.AddressTypeDomain), net.AddressFamilyDomain),  // 域名地址类型
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv6), net.AddressFamilyIPv6),  // IPv6地址类型
    protocol.PortThenAddress(),  // 端口然后地址
)

/*
Frame format
2 bytes - length
2 bytes - session id
1 bytes - status
1 bytes - option

1 byte - network
2 bytes - port
n bytes - address

*/

type FrameMetadata struct {  // 帧元数据结构
    Target        net.Destination  // 目标地址
    SessionID     uint16  // 会话ID
    Option        bitmask.Byte  // 选项
    SessionStatus SessionStatus  // 会话状态
}

func (f FrameMetadata) WriteTo(b *buf.Buffer) error {  // 将帧元数据写入缓冲区
    lenBytes := b.Extend(2)  // 扩展缓冲区长度为2字节

    len0 := b.Len()  // 记录当前缓冲区长度
    sessionBytes := b.Extend(2)  // 扩展缓冲区长度为2字节
    binary.BigEndian.PutUint16(sessionBytes, f.SessionID)  // 将会话ID写入缓冲区（大端序）

    common.Must(b.WriteByte(byte(f.SessionStatus)))  // 将会话状态写入缓冲区
    common.Must(b.WriteByte(byte(f.Option)))  // 将选项写入缓冲区
}
    # 如果会话状态为新建
    if f.SessionStatus == SessionStatusNew:
        # 根据目标网络类型进行不同的处理
        switch f.Target.Network:
            # 如果是 TCP 网络
            case net.Network_TCP:
                # 写入目标网络类型为 TCP
                common.Must(b.WriteByte(byte(TargetNetworkTCP)))
            # 如果是 UDP 网络
            case net.Network_UDP:
                # 写入目标网络类型为 UDP
                common.Must(b.WriteByte(byte(TargetNetworkUDP))

        # 写入目标地址和端口
        if err := addrParser.WriteAddressPort(b, f.Target.Address, f.Target.Port); err != nil:
            return err

    # 记录当前写入的字节数
    len0 := b.Len()
    # 计算当前写入的字节数与之前的差值，并将结果写入 lenBytes
    binary.BigEndian.PutUint16(lenBytes, uint16(len1-len0))
    # 返回空值
    return nil
// Unmarshal 从给定的 reader 中读取 FrameMetadata。
func (f *FrameMetadata) Unmarshal(reader io.Reader) error {
    // 读取 metaLen 和可能的错误
    metaLen, err := serial.ReadUint16(reader)
    if err != nil {
        return err
    }
    // 如果 metaLen 大于 512，则返回一个新的错误
    if metaLen > 512 {
        return newError("invalid metalen ", metaLen).AtError()
    }

    // 创建一个新的缓冲区
    b := buf.New()
    // 在函数返回时释放缓冲区
    defer b.Release()

    // 从 reader 中读取 metaLen 长度的数据到缓冲区 b
    if _, err := b.ReadFullFrom(reader, int32(metaLen)); err != nil {
        return err
    }
    // 从缓冲区 b 中解析 FrameMetadata
    return f.UnmarshalFromBuffer(b)
}

// UnmarshalFromBuffer 从给定的缓冲区中读取 FrameMetadata。
// 仅供测试使用。
func (f *FrameMetadata) UnmarshalFromBuffer(b *buf.Buffer) error {
    // 如果缓冲区长度小于 4，则返回一个新的错误
    if b.Len() < 4 {
        return newError("insufficient buffer: ", b.Len())
    }

    // 从缓冲区中读取 SessionID 和 SessionStatus
    f.SessionID = binary.BigEndian.Uint16(b.BytesTo(2))
    f.SessionStatus = SessionStatus(b.Byte(2))
    f.Option = bitmask.Byte(b.Byte(3))
    f.Target.Network = net.Network_Unknown

    // 如果 SessionStatus 为 SessionStatusNew
    if f.SessionStatus == SessionStatusNew {
        // 如果缓冲区长度小于 8，则返回一个新的错误
        if b.Len() < 8 {
            return newError("insufficient buffer: ", b.Len())
        }
        // 从缓冲区中读取网络类型和地址端口信息
        network := TargetNetwork(b.Byte(4))
        b.Advance(5)

        addr, port, err := addrParser.ReadAddressPort(nil, b)
        if err != nil {
            return newError("failed to parse address and port").Base(err)
        }

        // 根据网络类型创建对应的目标地址
        switch network {
        case TargetNetworkTCP:
            f.Target = net.TCPDestination(addr, port)
        case TargetNetworkUDP:
            f.Target = net.UDPDestination(addr, port)
        default:
            return newError("unknown network type: ", network)
        }
    }

    return nil
}
```