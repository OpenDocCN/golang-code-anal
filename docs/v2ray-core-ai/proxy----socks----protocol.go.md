# `v2ray-core\proxy\socks\protocol.go`

```
// +build !confonly

package socks

import (
    "encoding/binary"
    "io"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
)

const (
    socks5Version = 0x05
    socks4Version = 0x04

    cmdTCPConnect    = 0x01
    cmdTCPBind       = 0x02
    cmdUDPPort       = 0x03
    cmdTorResolve    = 0xF0
    cmdTorResolvePTR = 0xF1

    socks4RequestGranted  = 90
    socks4RequestRejected = 91

    authNotRequired = 0x00
    //authGssAPI           = 0x01
    authPassword         = 0x02
    authNoMatchingMethod = 0xFF

    statusSuccess       = 0x00
    statusCmdNotSupport = 0x07
)

// 创建地址解析器，用于解析不同类型的地址
var addrParser = protocol.NewAddressParser(
    protocol.AddressFamilyByte(0x01, net.AddressFamilyIPv4),
    protocol.AddressFamilyByte(0x04, net.AddressFamilyIPv6),
    protocol.AddressFamilyByte(0x03, net.AddressFamilyDomain),
)

// ServerSession 结构体，包含服务器配置和端口信息
type ServerSession struct {
    config *ServerConfig
    port   net.Port
}

// handshake4 方法，用于处理 SOCKS4 协议的握手过程
func (s *ServerSession) handshake4(cmd byte, reader io.Reader, writer io.Writer) (*protocol.RequestHeader, error) {
    // 如果需要密码认证，则拒绝 SOCKS4 请求
    if s.config.AuthType == AuthType_PASSWORD {
        writeSocks4Response(writer, socks4RequestRejected, net.AnyIP, net.Port(0)) // nolint: errcheck
        return nil, newError("socks 4 is not allowed when auth is required.")
    }

    var port net.Port
    var address net.Address

    {
        buffer := buf.StackNew()
        // 从 reader 中读取 6 个字节，分别解析出目标地址和端口
        if _, err := buffer.ReadFullFrom(reader, 6); err != nil {
            buffer.Release()
            return nil, newError("insufficient header").Base(err)
        }
        port = net.PortFromBytes(buffer.BytesRange(0, 2))
        address = net.IPAddress(buffer.BytesRange(2, 6))
        buffer.Release()
    }

    // 读取用户 ID 直到遇到空字符
    if _, err := ReadUntilNull(reader); /* user id */ err != nil {
        return nil, err
    }
    # 如果 IP 地址的第一个字节为 0x00，则表示是一个 SOCKS 4a 协议的请求
    if address.IP()[0] == 0x00 {
        # 读取直到遇到空字符为止，获取域名
        domain, err := ReadUntilNull(reader)
        if err != nil {
            # 如果读取失败，则返回错误
            return nil, newError("failed to read domain for socks 4a").Base(err)
        }
        # 根据域名创建网络地址
        address = net.DomainAddress(domain)
    }

    # 根据命令类型进行不同的处理
    switch cmd {
    case cmdTCPConnect:
        # 创建一个 TCP 连接的请求头
        request := &protocol.RequestHeader{
            Command: protocol.RequestCommandTCP,
            Address: address,
            Port:    port,
            Version: socks4Version,
        }
        # 写入 SOCKS 4 协议的响应，表示请求已经被允许
        if err := writeSocks4Response(writer, socks4RequestGranted, net.AnyIP, net.Port(0)); err != nil {
            return nil, err
        }
        # 返回请求头
        return request, nil
    default:
        # 写入 SOCKS 4 协议的响应，表示请求被拒绝
        writeSocks4Response(writer, socks4RequestRejected, net.AnyIP, net.Port(0)) // nolint: errcheck
        # 返回错误，表示不支持的命令类型
        return nil, newError("unsupported command: ", cmd)
    }
}
// auth5 方法用于进行身份验证
func (s *ServerSession) auth5(nMethod byte, reader io.Reader, writer io.Writer) (username string, err error) {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 在方法返回时释放缓冲区
    defer buffer.Release()

    // 从 reader 中读取 nMethod 字节到缓冲区中
    if _, err = buffer.ReadFullFrom(reader, int32(nMethod)); err != nil {
        return "", newError("failed to read auth methods").Base(err)
    }

    // 预期的认证方式，默认为不需要认证
    var expectedAuth byte = authNotRequired
    // 如果配置的认证类型为密码认证，则预期的认证方式为密码认证
    if s.config.AuthType == AuthType_PASSWORD {
        expectedAuth = authPassword
    }

    // 检查缓冲区中是否包含预期的认证方式
    if !hasAuthMethod(expectedAuth, buffer.BytesRange(0, int32(nMethod))) {
        // 如果不包含预期的认证方式，则写入认证响应并返回错误
        writeSocks5AuthenticationResponse(writer, socks5Version, authNoMatchingMethod) // nolint: errcheck
        return "", newError("no matching auth method")
    }

    // 写入预期的认证方式作为认证响应
    if err := writeSocks5AuthenticationResponse(writer, socks5Version, expectedAuth); err != nil {
        return "", newError("failed to write auth response").Base(err)
    }

    // 如果预期的认证方式为密码认证
    if expectedAuth == authPassword {
        // 从 reader 中读取用户名和密码
        username, password, err := ReadUsernamePassword(reader)
        if err != nil {
            return "", newError("failed to read username and password for authentication").Base(err)
        }

        // 检查用户名和密码是否有效
        if !s.config.HasAccount(username, password) {
            // 如果无效，则写入认证响应并返回错误
            writeSocks5AuthenticationResponse(writer, 0x01, 0xFF) // nolint: errcheck
            return "", newError("invalid username or password")
        }

        // 写入认证成功的响应
        if err := writeSocks5AuthenticationResponse(writer, 0x01, 0x00); err != nil {
            return "", newError("failed to write auth response").Base(err)
        }
        return username, nil
    }

    return "", nil
}

// handshake5 方法用于进行握手
func (s *ServerSession) handshake5(nMethod byte, reader io.Reader, writer io.Writer) (*protocol.RequestHeader, error) {
    var (
        username string
        err      error
    )
    // 调用 auth5 方法进行身份验证
    if username, err = s.auth5(nMethod, reader, writer); err != nil {
        return nil, err
    }

    var cmd byte
    {
        // 创建一个新的缓冲区
        buffer := buf.StackNew()
        // 从reader中读取3个字节到缓冲区中
        if _, err := buffer.ReadFullFrom(reader, 3); err != nil {
            // 释放缓冲区
            buffer.Release()
            // 返回错误信息
            return nil, newError("failed to read request").Base(err)
        }
        // 从缓冲区中读取第二个字节作为命令
        cmd = buffer.Byte(1)
        // 释放缓冲区
        buffer.Release()
    }

    // 创建一个新的请求头
    request := new(protocol.RequestHeader)
    // 如果用户名不为空，则创建一个内存用户
    if username != "" {
        request.User = &protocol.MemoryUser{Email: username}
    }
    // 根据命令类型进行不同的处理
    switch cmd {
    case cmdTCPConnect, cmdTorResolve, cmdTorResolvePTR:
        // 对于Tor的情况暂时没有解决方案，简单地将其视为连接命令
        request.Command = protocol.RequestCommandTCP
    case cmdUDPPort:
        // 如果UDP未启用，则返回错误信息
        if !s.config.UdpEnabled {
            writeSocks5Response(writer, statusCmdNotSupport, net.AnyIP, net.Port(0)) // nolint: errcheck
            return nil, newError("UDP is not enabled.")
        }
        // 设置请求命令为UDP
        request.Command = protocol.RequestCommandUDP
    case cmdTCPBind:
        // 返回不支持TCP绑定的错误信息
        writeSocks5Response(writer, statusCmdNotSupport, net.AnyIP, net.Port(0)) // nolint: errcheck
        return nil, newError("TCP bind is not supported.")
    default:
        // 返回未知命令的错误信息
        writeSocks5Response(writer, statusCmdNotSupport, net.AnyIP, net.Port(0)) // nolint: errcheck
        return nil, newError("unknown command ", cmd)
    }

    // 设置请求头的版本号
    request.Version = socks5Version

    // 从reader中读取地址和端口
    addr, port, err := addrParser.ReadAddressPort(nil, reader)
    if err != nil {
        return nil, newError("failed to read address").Base(err)
    }
    // 设置请求头的地址和端口
    request.Address = addr
    request.Port = port

    // 设置响应地址和端口的默认值
    responseAddress := net.AnyIP
    responsePort := net.Port(1717)
    // 如果请求命令为UDP，则根据配置设置响应地址和端口
    if request.Command == protocol.RequestCommandUDP {
        addr := s.config.Address.AsAddress()
        if addr == nil {
            addr = net.LocalHostIP
        }
        responseAddress = addr
        responsePort = s.port
    }
    // 写入Socks5响应
    if err := writeSocks5Response(writer, statusSuccess, responseAddress, responsePort); err != nil {
        return nil, err
    }

    // 返回请求头和空错误信息
    return request, nil
// Handshake performs a Socks4/4a/5 handshake.
func (s *ServerSession) Handshake(reader io.Reader, writer io.Writer) (*protocol.RequestHeader, error) {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 从 reader 中读取 2 个字节到缓冲区
    if _, err := buffer.ReadFullFrom(reader, 2); err != nil {
        buffer.Release()
        return nil, newError("insufficient header").Base(err)
    }

    // 读取版本号和命令类型
    version := buffer.Byte(0)
    cmd := buffer.Byte(1)
    buffer.Release()

    // 根据版本号进行不同的处理
    switch version {
    case socks4Version:
        return s.handshake4(cmd, reader, writer)
    case socks5Version:
        return s.handshake5(cmd, reader, writer)
    default:
        return nil, newError("unknown Socks version: ", version)
    }
}

// ReadUsernamePassword reads Socks 5 username/password message from the given reader.
// +----+------+----------+------+----------+
// |VER | ULEN |  UNAME   | PLEN |  PASSWD  |
// +----+------+----------+------+----------+
// | 1  |  1   | 1 to 255 |  1   | 1 to 255 |
// +----+------+----------+------+----------+
func ReadUsernamePassword(reader io.Reader) (string, string, error) {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    defer buffer.Release()

    // 从 reader 中读取 2 个字节到缓冲区
    if _, err := buffer.ReadFullFrom(reader, 2); err != nil {
        return "", "", err
    }
    // 读取用户名的长度
    nUsername := int32(buffer.Byte(1))

    // 清空缓冲区
    buffer.Clear()
    // 从 reader 中读取用户名的内容
    if _, err := buffer.ReadFullFrom(reader, nUsername); err != nil {
        return "", "", err
    }
    // 获取用户名
    username := buffer.String()

    // 清空缓冲区
    buffer.Clear()
    // 从 reader 中读取 1 个字节到缓冲区
    if _, err := buffer.ReadFullFrom(reader, 1); err != nil {
        return "", "", err
    }
    // 读取密码的长度
    nPassword := int32(buffer.Byte(0))

    // 清空缓冲区
    buffer.Clear()
    // 从 reader 中读取密码的内容
    if _, err := buffer.ReadFullFrom(reader, nPassword); err != nil {
        return "", "", err
    }
    // 获取密码
    password := buffer.String()
    return username, password, nil
}

// ReadUntilNull reads content from given reader, until a null (0x00) byte.
func ReadUntilNull(reader io.Reader) (string, error) {
    // 创建一个缓冲区
    b := buf.StackNew()
    defer b.Release()
    // 无限循环，读取数据直到条件满足或出错
    for {
        // 从输入流中读取一个字节，如果出错则返回错误
        _, err := b.ReadFullFrom(reader, 1)
        if err != nil {
            return "", err
        }
        // 检查缓冲区最后一个字节是否为0x00，如果是则调整缓冲区大小并返回字符串
        if b.Byte(b.Len()-1) == 0x00 {
            b.Resize(0, b.Len()-1)
            return b.String(), nil
        }
        // 检查缓冲区是否已满，如果是则返回缓冲区溢出错误
        if b.IsFull() {
            return "", newError("buffer overrun")
        }
    }
# 检查给定的认证方法是否在候选方法列表中
func hasAuthMethod(expectedAuth byte, authCandidates []byte) bool:
    # 遍历认证方法候选列表
    for _, a := range authCandidates:
        # 如果找到期望的认证方法，则返回 true
        if a == expectedAuth:
            return true
    # 如果没有找到期望的认证方法，则返回 false
    return false

# 写入 SOCKS5 认证响应
func writeSocks5AuthenticationResponse(writer io.Writer, version byte, auth byte) error:
    # 将版本号和认证方法写入输出流
    return buf.WriteAllBytes(writer, []byte{version, auth})

# 写入 SOCKS5 响应
func writeSocks5Response(writer io.Writer, errCode byte, address net.Address, port net.Port) error:
    # 创建缓冲区
    buffer := buf.New()
    # 在函数返回时释放缓冲区
    defer buffer.Release()
    # 写入 SOCKS5 版本号和错误码，以及保留字段
    common.Must2(buffer.Write([]byte{socks5Version, errCode, 0x00 /* reserved */}))
    # 写入地址和端口
    if err := addrParser.WriteAddressPort(buffer, address, port); err != nil:
        return err
    # 将缓冲区内容写入输出流
    return buf.WriteAllBytes(writer, buffer.Bytes())

# 写入 SOCKS4 响应
func writeSocks4Response(writer io.Writer, errCode byte, address net.Address, port net.Port) error:
    # 创建栈上的缓冲区
    buffer := buf.StackNew()
    # 在函数返回时释放缓冲区
    defer buffer.Release()
    # 写入保留字段和错误码
    common.Must(buffer.WriteByte(0x00))
    common.Must(buffer.WriteByte(errCode))
    # 写入端口号
    portBytes := buffer.Extend(2)
    binary.BigEndian.PutUint16(portBytes, port.Value())
    # 写入 IP 地址
    common.Must2(buffer.Write(address.IP()))
    # 将缓冲区内容写入输出流
    return buf.WriteAllBytes(writer, buffer.Bytes())

# 解码 UDP 数据包
func DecodeUDPPacket(packet *buf.Buffer) (*protocol.RequestHeader, error):
    # 如果数据包长度小于 5，则返回错误
    if packet.Len() < 5:
        return nil, newError("insufficient length of packet.")
    # 创建请求头对象
    request := &protocol.RequestHeader{
        Version: socks5Version,
        Command: protocol.RequestCommandUDP,
    }
    # 检查数据包是否为分片数据
    if packet.Byte(2) != 0 /* fragments */:
        return nil, newError("discarding fragmented payload.")
    # 跳过前 3 个字节
    packet.Advance(3)
    # 读取地址和端口
    addr, port, err := addrParser.ReadAddressPort(nil, packet)
    if err != nil:
        return nil, newError("failed to read UDP header").Base(err)
    # 设置请求头的地址和端口
    request.Address = addr
    request.Port = port
    return request, nil

# 编码 UDP 数据包
func EncodeUDPPacket(request *protocol.RequestHeader, data []byte) (*buf.Buffer, error):
    # 创建一个新的字节缓冲区
    b := buf.New()
    # 向字节缓冲区写入指定的字节数据
    common.Must2(b.Write([]byte{0, 0, 0 /* Fragment */}))
    # 将地址和端口信息写入字节缓冲区
    if err := addrParser.WriteAddressPort(b, request.Address, request.Port); err != nil {
        # 如果出现错误，释放字节缓冲区并返回空和错误信息
        b.Release()
        return nil, err
    }
    # 继续向字节缓冲区写入指定的数据
    common.Must2(b.Write(data))
    # 返回填充好数据的字节缓冲区和空的错误信息
    return b, nil
}



type UDPReader struct {
    reader io.Reader
}



func NewUDPReader(reader io.Reader) *UDPReader {
    return &UDPReader{reader: reader}
}



func (r *UDPReader) ReadMultiBuffer() (buf.MultiBuffer, error) {
    b := buf.New()
    if _, err := b.ReadFrom(r.reader); err != nil {
        return nil, err
    }
    if _, err := DecodeUDPPacket(b); err != nil {
        return nil, err
    }
    return buf.MultiBuffer{b}, nil
}



type UDPWriter struct {
    request *protocol.RequestHeader
    writer  io.Writer
}



func NewUDPWriter(request *protocol.RequestHeader, writer io.Writer) *UDPWriter {
    return &UDPWriter{
        request: request,
        writer:  writer,
    }
}



// Write implements io.Writer.
func (w *UDPWriter) Write(b []byte) (int, error) {
    eb, err := EncodeUDPPacket(w.request, b)
    if err != nil {
        return 0, err
    }
    defer eb.Release()
    if _, err := w.writer.Write(eb.Bytes()); err != nil {
        return 0, err
    }
    return len(b), nil
}



func ClientHandshake(request *protocol.RequestHeader, reader io.Reader, writer io.Writer) (*protocol.RequestHeader, error) {
    authByte := byte(authNotRequired)
    if request.User != nil {
        authByte = byte(authPassword)
    }

    b := buf.New()
    defer b.Release()

    common.Must2(b.Write([]byte{socks5Version, 0x01, authByte}))
    if authByte == authPassword {
        account := request.User.Account.(*Account)

        common.Must(b.WriteByte(0x01))
        common.Must(b.WriteByte(byte(len(account.Username))))
        common.Must2(b.WriteString(account.Username))
        common.Must(b.WriteByte(byte(len(account.Password))))
        common.Must2(b.WriteString(account.Password))
    }

    if err := buf.WriteAllBytes(writer, b.Bytes()); err != nil {
        return nil, err
    }

    b.Clear()
    if _, err := b.ReadFullFrom(reader, 2); err != nil {
        return nil, err
    }
}
    # 检查服务器返回的字节流的第一个字节是否与预期的 SOCKS5 版本号相符
    if b.Byte(0) != socks5Version:
        return nil, newError("unexpected server version: ", b.Byte(0)).AtWarning()
    
    # 检查服务器返回的字节流的第二个字节是否支持指定的认证方法
    if b.Byte(1) != authByte:
        return nil, newError("auth method not supported.").AtWarning()
    
    # 如果认证方法为密码认证
    if authByte == authPassword:
        # 清空字节流
        b.Clear()
        # 从读取器中读取两个字节的数据
        if _, err := b.ReadFullFrom(reader, 2); err != nil:
            return nil, err
        # 检查服务器返回的第二个字节是否为 0x00，表示接受账户
        if b.Byte(1) != 0x00:
            return nil, newError("server rejects account: ", b.Byte(1))
    
    # 清空字节流
    b.Clear()
    
    # 设置命令为 TCP 连接
    command := byte(cmdTCPConnect)
    # 如果请求命令为 UDP
    if request.Command == protocol.RequestCommandUDP:
        # 设置命令为 UDP 端口
        command = byte(cmdUDPPort)
    # 写入 SOCKS5 版本号、命令和保留字段到字节流
    common.Must2(b.Write([]byte{socks5Version, command, 0x00 /* reserved */}))
    # 将请求地址和端口写入字节流
    if err := addrParser.WriteAddressPort(b, request.Address, request.Port); err != nil:
        return nil, err
    
    # 将字节流写入写入器
    if err := buf.WriteAllBytes(writer, b.Bytes()); err != nil:
        return nil, err
    
    # 清空字节流
    b.Clear()
    # 从读取器中读取三个字节的数据
    if _, err := b.ReadFullFrom(reader, 3); err != nil:
        return nil, err
    
    # 获取响应的第二个字节
    resp := b.Byte(1)
    # 如果响应不为 0x00，表示服务器拒绝请求
    if resp != 0x00:
        return nil, newError("server rejects request: ", resp)
    
    # 清空字节流
    b.Clear()
    
    # 从读取器中读取地址和端口信息
    address, port, err := addrParser.ReadAddressPort(b, reader)
    if err != nil:
        return nil, err
    
    # 如果请求命令为 UDP
    if request.Command == protocol.RequestCommandUDP:
        # 创建 UDP 请求头部
        udpRequest := &protocol.RequestHeader{
            Version: socks5Version,
            Command: protocol.RequestCommandUDP,
            Address: address,
            Port:    port,
        }
        return udpRequest, nil
    
    return nil, nil
# 闭合前面的函数定义
```