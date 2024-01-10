# `v2ray-core\proxy\trojan\protocol.go`

```
package trojan

import (
    "encoding/binary"  // 导入编码二进制数据的包
    "io"  // 导入输入输出的包

    "v2ray.com/core/common/buf"  // 导入缓冲区的包
    "v2ray.com/core/common/net"  // 导入网络相关的包
    "v2ray.com/core/common/protocol"  // 导入协议相关的包
)

var (
    crlf = []byte{'\r', '\n'}  // 定义回车换行符的字节切片

    addrParser = protocol.NewAddressParser(  // 创建地址解析器
        protocol.AddressFamilyByte(0x01, net.AddressFamilyIPv4),   // nolint: gomnd
        protocol.AddressFamilyByte(0x04, net.AddressFamilyIPv6),   // nolint: gomnd
        protocol.AddressFamilyByte(0x03, net.AddressFamilyDomain), // nolint: gomnd
    )
)

const (
    maxLength = 8192  // 定义最大长度为 8192

    commandTCP byte = 1  // 定义 TCP 命令为 1
    commandUDP byte = 3  // 定义 UDP 命令为 3
)

// ConnWriter is TCP Connection Writer Wrapper for trojan protocol
type ConnWriter struct {
    io.Writer  // TCP 连接写入器的接口
    Target     net.Destination  // 目标地址
    Account    *MemoryAccount  // 内存账户
    headerSent bool  // 头部是否已发送
}

// Write implements io.Writer
func (c *ConnWriter) Write(p []byte) (n int, err error) {
    if !c.headerSent {  // 如果头部未发送
        if err := c.writeHeader(); err != nil {  // 写入头部
            return 0, newError("failed to write request header").Base(err)  // 返回错误
        }
    }

    return c.Writer.Write(p)  // 返回写入结果
}

// WriteMultiBuffer implements buf.Writer
func (c *ConnWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    defer buf.ReleaseMulti(mb)  // 延迟释放多重缓冲区

    for _, b := range mb {  // 遍历多重缓冲区
        if !b.IsEmpty() {  // 如果不为空
            if _, err := c.Write(b.Bytes()); err != nil {  // 写入字节
                return err  // 返回错误
            }
        }
    }

    return nil  // 返回空
}

func (c *ConnWriter) writeHeader() error {
    buffer := buf.StackNew()  // 创建新的缓冲区
    defer buffer.Release()  // 延迟释放缓冲区

    command := commandTCP  // 默认命令为 TCP
    if c.Target.Network == net.Network_UDP {  // 如果目标网络为 UDP
        command = commandUDP  // 命令为 UDP
    }

    if _, err := buffer.Write(c.Account.Key); err != nil {  // 写入账户密钥
        return err  // 返回错误
    }
    if _, err := buffer.Write(crlf); err != nil {  // 写入回车换行符
        return err  // 返回错误
    }
    if err := buffer.WriteByte(command); err != nil {  // 写入命令
        return err  // 返回错误
    }
    if err := addrParser.WriteAddressPort(&buffer, c.Target.Address, c.Target.Port); err != nil {  // 写入地址和端口
        return err  // 返回错误
    }
    # 如果写入换行符出现错误，则返回错误
    if _, err := buffer.Write(crlf); err != nil:
        return err
    # 将缓冲区中的字节写入到响应体中
    _, err := c.Writer.Write(buffer.Bytes())
    # 如果没有错误发生，则将标记设置为已发送响应头
    if err == nil:
        c.headerSent = true
    # 返回可能发生的错误
    return err
// PacketWriter 是用于 trojan 协议的 UDP 连接写入器包装器
type PacketWriter struct {
    io.Writer
    Target net.Destination
}

// WriteMultiBuffer 实现了 buf.Writer 接口
func (w *PacketWriter) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 创建一个长度为 maxLength 的字节切片
    b := make([]byte, maxLength)
    // 当多重缓冲区不为空时循环
    for !mb.IsEmpty() {
        var length int
        // 从多重缓冲区中分割出字节并返回长度
        mb, length = buf.SplitBytes(mb, b)
        // 调用 writePacket 方法写入数据包
        if _, err := w.writePacket(b[:length], w.Target); err != nil {
            // 释放多重缓冲区并返回错误
            buf.ReleaseMulti(mb)
            return err
        }
    }

    return nil
}

// WriteMultiBufferWithMetadata 使用指定的目标地址写入 UDP 数据包
func (w *PacketWriter) WriteMultiBufferWithMetadata(mb buf.MultiBuffer, dest net.Destination) error {
    // 创建一个长度为 maxLength 的字节切片
    b := make([]byte, maxLength)
    // 当多重缓冲区不为空时循环
    for !mb.IsEmpty() {
        var length int
        // 从多重缓冲区中分割出字节并返回长度
        mb, length = buf.SplitBytes(mb, b)
        // 调用 writePacket 方法写入数据包
        if _, err := w.writePacket(b[:length], dest); err != nil {
            // 释放多重缓冲区并返回错误
            buf.ReleaseMulti(mb)
            return err
        }
    }

    return nil
}

// writePacket 方法用于写入数据包
func (w *PacketWriter) writePacket(payload []byte, dest net.Destination) (int, error) { // nolint: unparam
    // 创建一个新的缓冲区
    buffer := buf.StackNew()
    defer buffer.Release()

    // 获取 payload 的长度并转换为大端序的字节切片
    length := len(payload)
    lengthBuf := [2]byte{}
    binary.BigEndian.PutUint16(lengthBuf[:], uint16(length))
    // 写入地址和端口信息
    if err := addrParser.WriteAddressPort(&buffer, dest.Address, dest.Port); err != nil {
        return 0, err
    }
    // 写入长度信息
    if _, err := buffer.Write(lengthBuf[:]); err != nil {
        return 0, err
    }
    // 写入 CRLF
    if _, err := buffer.Write(crlf); err != nil {
        return 0, err
    }
    // 写入 payload
    if _, err := buffer.Write(payload); err != nil {
        return 0, err
    }
    // 将缓冲区的内容写入到 PacketWriter 的目标中
    _, err := w.Write(buffer.Bytes())
    if err != nil {
        return 0, err
    }

    return length, nil
}

// ConnReader 是用于 trojan 协议的 TCP 连接读取器包装器
type ConnReader struct {
    io.Reader
    Target       net.Destination
    headerParsed bool
}

// ParseHeader 用于解析 trojan 协议的头部信息
// ParseHeader 解析连接头部信息
func (c *ConnReader) ParseHeader() error {
    // 读取用户哈希值
    var crlf [2]byte
    var command [1]byte
    var hash [56]byte
    if _, err := io.ReadFull(c.Reader, hash[:]); err != nil {
        return newError("failed to read user hash").Base(err)
    }

    // 读取回车换行符
    if _, err := io.ReadFull(c.Reader, crlf[:]); err != nil {
        return newError("failed to read crlf").Base(err)
    }

    // 读取命令
    if _, err := io.ReadFull(c.Reader, command[:]); err != nil {
        return newError("failed to read command").Base(err)
    }

    // 判断网络类型
    network := net.Network_TCP
    if command[0] == commandUDP {
        network = net.Network_UDP
    }

    // 读取地址和端口
    addr, port, err := addrParser.ReadAddressPort(nil, c.Reader)
    if err != nil {
        return newError("failed to read address and port").Base(err)
    }
    c.Target = net.Destination{Network: network, Address: addr, Port: port}

    // 读取回车换行符
    if _, err := io.ReadFull(c.Reader, crlf[:]); err != nil {
        return newError("failed to read crlf").Base(err)
    }

    // 标记头部解析完成
    c.headerParsed = true
    return nil
}

// Read 实现了io.Reader接口
func (c *ConnReader) Read(p []byte) (int, error) {
    // 如果头部未解析，则先解析头部
    if !c.headerParsed {
        if err := c.ParseHeader(); err != nil {
            return 0, err
        }
    }

    return c.Reader.Read(p)
}

// ReadMultiBuffer 实现了buf.Reader接口
func (c *ConnReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 创建新的多缓冲区
    b := buf.New()
    // 从连接中读取数据到多缓冲区
    _, err := b.ReadFrom(c)
    return buf.MultiBuffer{b}, err
}

// PacketPayload 结构体，包含UDP数据和目标地址
type PacketPayload struct {
    Target net.Destination
    Buffer buf.MultiBuffer
}

// PacketReader 是trojan协议的UDP连接读取器包装
type PacketReader struct {
    io.Reader
}

// ReadMultiBuffer 实现了buf.Reader接口
func (r *PacketReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    // 读取带有元数据的UDP数据包
    p, err := r.ReadMultiBufferWithMetadata()
    if p != nil {
        return p.Buffer, err
    }
    return nil, err
}

// ReadMultiBufferWithMetadata 读取带有目标地址的UDP数据包
func (r *PacketReader) ReadMultiBufferWithMetadata() (*PacketPayload, error) {
    // 从 PacketReader 中读取地址和端口信息
    addr, port, err := addrParser.ReadAddressPort(nil, r)
    if err != nil {
        // 如果读取失败，则返回错误信息
        return nil, newError("failed to read address and port").Base(err)
    }

    // 读取 payload 长度信息
    var lengthBuf [2]byte
    if _, err := io.ReadFull(r, lengthBuf[:]); err != nil {
        // 如果读取失败，则返回错误信息
        return nil, newError("failed to read payload length").Base(err)
    }

    // 将长度信息转换为 remain 变量
    remain := int(binary.BigEndian.Uint16(lengthBuf[:]))
    if remain > maxLength {
        // 如果长度超过最大长度，则返回错误信息
        return nil, newError("oversize payload")
    }

    // 读取 CRLF（回车换行）信息
    var crlf [2]byte
    if _, err := io.ReadFull(r, crlf[:]); err != nil {
        // 如果读取失败，则返回错误信息
        return nil, newError("failed to read crlf").Base(err)
    }

    // 创建 UDP 目标地址
    dest := net.UDPDestination(addr, port)
    // 创建多缓冲区
    var mb buf.MultiBuffer
    // 循环读取数据直到 remain 为 0
    for remain > 0 {
        length := buf.Size
        if remain < length {
            length = remain
        }

        // 创建新的缓冲区并添加到多缓冲区中
        b := buf.New()
        mb = append(mb, b)
        // 从 r 中读取指定长度的数据到缓冲区中
        n, err := b.ReadFullFrom(r, int32(length))
        if err != nil {
            // 如果读取失败，则释放多缓冲区并返回错误信息
            buf.ReleaseMulti(mb)
            return nil, newError("failed to read payload").Base(err)
        }

        // 更新 remain 变量
        remain -= int(n)
    }

    // 返回包含目标地址和多缓冲区的 PacketPayload 结构体
    return &PacketPayload{Target: dest, Buffer: mb}, nil
}
```