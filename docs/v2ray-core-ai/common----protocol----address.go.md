# `v2ray-core\common\protocol\address.go`

```
package protocol

import (
    "io"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/serial"
)

type AddressOption func(*option)

func PortThenAddress() AddressOption {
    return func(p *option) {
        p.portFirst = true
    }
}

func AddressFamilyByte(b byte, f net.AddressFamily) AddressOption {
    if b >= 16 {
        panic("address family byte too big")
    }
    return func(p *option) {
        p.addrTypeMap[b] = f
        p.addrByteMap[f] = b
    }
}

type AddressTypeParser func(byte) byte

func WithAddressTypeParser(atp AddressTypeParser) AddressOption {
    return func(p *option) {
        p.typeParser = atp
    }
}

type AddressSerializer interface {
    ReadAddressPort(buffer *buf.Buffer, input io.Reader) (net.Address, net.Port, error)

    WriteAddressPort(writer io.Writer, addr net.Address, port net.Port) error
}

const afInvalid = 255

type option struct {
    addrTypeMap [16]net.AddressFamily  // 存储地址类型映射的数组
    addrByteMap [16]byte  // 存储地址字节映射的数组
    portFirst   bool  // 是否先读取端口
    typeParser  AddressTypeParser  // 地址类型解析器
}

// NewAddressParser creates a new AddressParser
func NewAddressParser(options ...AddressOption) AddressSerializer {
    var o option
    for i := range o.addrByteMap {
        o.addrByteMap[i] = afInvalid  // 初始化地址字节映射数组
    }
    for i := range o.addrTypeMap {
        o.addrTypeMap[i] = net.AddressFamily(afInvalid)  // 初始化地址类型映射数组
    }
    for _, opt := range options {
        opt(&o)  // 应用选项
    }

    ap := &addressParser{
        addrByteMap: o.addrByteMap,
        addrTypeMap: o.addrTypeMap,
    }

    if o.typeParser != nil {
        ap.typeParser = o.typeParser  // 设置地址类型解析器
    }

    if o.portFirst {
        return portFirstAddressParser{ap: ap}  // 如果先读取端口，则返回端口优先的地址解析器
    }

    return portLastAddressParser{ap: ap}  // 否则返回端口后的地址解析器
}

type portFirstAddressParser struct {
    ap *addressParser
}

func (p portFirstAddressParser) ReadAddressPort(buffer *buf.Buffer, input io.Reader) (net.Address, net.Port, error) {
    if buffer == nil {
        buffer = buf.New()  // 如果缓冲区为空，则创建新的缓冲区
        defer buffer.Release()  // 函数结束时释放缓冲区
    }
    // 从缓冲区和输入中读取端口号
    port, err := readPort(buffer, input)
    // 如果出现错误，返回空值和错误信息
    if err != nil {
        return nil, 0, err
    }
    // 从缓冲区和输入中读取地址
    addr, err := p.ap.readAddress(buffer, input)
    // 如果出现错误，返回空值和错误信息
    if err != nil {
        return nil, 0, err
    }
    // 返回地址、端口号和空错误信息
    return addr, port, nil
# 定义一个结构体方法，用于将地址和端口写入到指定的io.Writer中
func (p portFirstAddressParser) WriteAddressPort(writer io.Writer, addr net.Address, port net.Port) error {
    # 调用writePort函数将端口写入到指定的io.Writer中
    if err := writePort(writer, port); err != nil {
        return err
    }
    # 调用addressParser结构体的writeAddress方法将地址写入到指定的io.Writer中
    return p.ap.writeAddress(writer, addr)
}

# 定义一个结构体，包含一个addressParser类型的指针
type portLastAddressParser struct {
    ap *addressParser
}

# 定义一个结构体方法，用于从输入流中读取地址和端口
func (p portLastAddressParser) ReadAddressPort(buffer *buf.Buffer, input io.Reader) (net.Address, net.Port, error) {
    # 如果缓冲区为空，则创建一个新的缓冲区，并在函数结束时释放
    if buffer == nil {
        buffer = buf.New()
        defer buffer.Release()
    }
    # 调用addressParser结构体的readAddress方法从输入流中读取地址
    addr, err := p.ap.readAddress(buffer, input)
    if err != nil {
        return nil, 0, err
    }
    # 调用readPort函数从输入流中读取端口
    port, err := readPort(buffer, input)
    if err != nil {
        return nil, 0, err
    }
    # 返回读取到的地址和端口
    return addr, port, nil
}

# 定义一个结构体方法，用于将地址和端口写入到指定的io.Writer中
func (p portLastAddressParser) WriteAddressPort(writer io.Writer, addr net.Address, port net.Port) error {
    # 调用addressParser结构体的writeAddress方法将地址写入到指定的io.Writer中
    if err := p.ap.writeAddress(writer, addr); err != nil {
        return err
    }
    # 调用writePort函数将端口写入到指定的io.Writer中
    return writePort(writer, port)
}

# 定义一个函数，用于从缓冲区和输入流中读取端口
func readPort(b *buf.Buffer, reader io.Reader) (net.Port, error) {
    # 从输入流中读取2个字节的数据到缓冲区中
    if _, err := b.ReadFullFrom(reader, 2); err != nil {
        return 0, err
    }
    # 从缓冲区中获取端口值并返回
    return net.PortFromBytes(b.BytesFrom(-2)), nil
}

# 定义一个函数，用于将端口写入到指定的io.Writer中
func writePort(writer io.Writer, port net.Port) error {
    # 调用serial.WriteUint16函数将端口值写入到指定的io.Writer中
    return common.Error2(serial.WriteUint16(writer, port.Value()))
}

# 定义一个函数，用于判断一个字节是否可能是IP前缀
func maybeIPPrefix(b byte) bool {
    # 如果字节等于'['或者在数字0-9的范围内，则返回true，否则返回false
    return b == '[' || (b >= '0' && b <= '9')
}

# 定义一个函数，用于判断一个字符串是否是有效的域名
func isValidDomain(d string) bool {
    # 遍历字符串中的每个字符
    for _, c := range d {
        # 如果字符不在指定的范围内，则返回false
        if !((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c == '-' || c == '.' || c == '_') {
            return false
        }
    }
    # 如果所有字符都在指定的范围内，则返回true
    return true
}

# 定义一个结构体，包含地址类型映射、地址字节映射和类型解析器
type addressParser struct {
    addrTypeMap [16]net.AddressFamily
    addrByteMap [16]byte
    typeParser  AddressTypeParser
}

# 定义一个结构体方法，用于从输入流中读取地址
func (p *addressParser) readAddress(b *buf.Buffer, reader io.Reader) (net.Address, error) {
    # 从输入流中读取1个字节的数据到缓冲区中
    if _, err := b.ReadFullFrom(reader, 1); err != nil {
        return nil, err
    }
    # 获取缓冲区中的地址类型字节
    addrType := b.Byte(b.Len() - 1)
    # 如果地址类型解析器不为空，则使用解析器对地址类型进行解析
    if p.typeParser != nil:
        addrType = p.typeParser(addrType)

    # 如果地址类型大于等于16，则返回错误信息
    if addrType >= 16:
        return nil, newError("unknown address type: ", addrType)

    # 根据地址类型获取地址族
    addrFamily := p.addrTypeMap[addrType]
    # 如果地址族为无效地址族，则返回错误信息
    if addrFamily == net.AddressFamily(afInvalid):
        return nil, newError("unknown address type: ", addrType)

    # 根据地址族进行不同的处理
    switch addrFamily:
        # 对于IPv4地址族，从reader中读取4个字节，并返回IPv4地址
        case net.AddressFamilyIPv4:
            if _, err := b.ReadFullFrom(reader, 4); err != nil:
                return nil, err
            return net.IPAddress(b.BytesFrom(-4)), nil
        # 对于IPv6地址族，从reader中读取16个字节，并返回IPv6地址
        case net.AddressFamilyIPv6:
            if _, err := b.ReadFullFrom(reader, 16); err != nil:
                return nil, err
            return net.IPAddress(b.BytesFrom(-16)), nil
        # 对于域名地址族，先读取域名长度，再根据长度读取域名，并进行解析
        case net.AddressFamilyDomain:
            if _, err := b.ReadFullFrom(reader, 1); err != nil:
                return nil, err
            domainLength := int32(b.Byte(b.Len() - 1))
            if _, err := b.ReadFullFrom(reader, domainLength); err != nil:
                return nil, err
            domain := string(b.BytesFrom(-domainLength))
            # 如果域名以IP前缀开头，则解析为IP地址
            if maybeIPPrefix(domain[0]):
                addr := net.ParseAddress(domain)
                if addr.Family().IsIP():
                    return addr, nil
            # 如果域名不是有效的域名，则返回错误信息
            if !isValidDomain(domain):
                return nil, newError("invalid domain name: ", domain)
            return net.DomainAddress(domain), nil
        # 默认情况下，抛出异常
        default:
            panic("impossible case")
// 将地址写入到给定的 writer 中
func (p *addressParser) writeAddress(writer io.Writer, address net.Address) error {
    // 获取地址族对应的字节表示
    tb := p.addrByteMap[address.Family()]
    // 如果地址族无效，则返回错误
    if tb == afInvalid {
        return newError("unknown address family", address.Family())
    }

    // 根据地址族类型进行不同的处理
    switch address.Family() {
    case net.AddressFamilyIPv4, net.AddressFamilyIPv6:
        // 如果是 IPv4 或 IPv6 地址，则写入地址族字节和 IP 地址
        if _, err := writer.Write([]byte{tb}); err != nil {
            return err
        }
        if _, err := writer.Write(address.IP()); err != nil {
            return err
        }
    case net.AddressFamilyDomain:
        // 如果是域名地址，则写入地址族字节、域名长度和域名内容
        domain := address.Domain()
        // 检查域名长度是否过长
        if isDomainTooLong(domain) {
            return newError("Super long domain is not supported: ", domain)
        }

        if _, err := writer.Write([]byte{tb, byte(len(domain))}); err != nil {
            return err
        }
        if _, err := writer.Write([]byte(domain)); err != nil {
            return err
        }
    default:
        // 如果是未知的地址族类型，则触发 panic
        panic("Unknown family type.")
    }

    // 没有错误发生，返回 nil
    return nil
}
```