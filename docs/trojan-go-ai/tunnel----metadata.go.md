# `trojan-go\tunnel\metadata.go`

```
package tunnel

import (
    "bytes"  // 导入 bytes 包，用于操作字节流
    "encoding/binary"  // 导入 binary 包，用于处理二进制数据的编解码
    "fmt"  // 导入 fmt 包，用于格式化输出
    "io"  // 导入 io 包，用于进行输入输出操作
    "net"  // 导入 net 包，用于网络相关操作
    "strconv"  // 导入 strconv 包，用于字符串转换
    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包

)

type Command byte  // 定义 Command 类型为 byte

type Metadata struct {  // 定义 Metadata 结构体
    Command  // Metadata 结构体包含 Command 类型的字段
    *Address  // Metadata 结构体包含 Address 指针类型的字段
}

func (r *Metadata) ReadFrom(rr io.Reader) error {  // Metadata 结构体的 ReadFrom 方法
    byteBuf := [1]byte{}  // 创建长度为 1 的字节数组
    _, err := io.ReadFull(rr, byteBuf[:])  // 从输入流中读取数据到字节数组
    if err != nil {  // 如果读取出错
        return err  // 返回错误
    }
    r.Command = Command(byteBuf[0])  // 将字节数组转换为 Command 类型并赋值给 Metadata 结构体的 Command 字段
    r.Address = new(Address)  // 创建一个 Address 对象并赋值给 Metadata 结构体的 Address 字段
    err = r.Address.ReadFrom(rr)  // 调用 Address 结构体的 ReadFrom 方法
    if err != nil {  // 如果出错
        return common.NewError("failed to marshal address").Base(err)  // 返回自定义错误
    }
    return nil  // 返回空
}

func (r *Metadata) WriteTo(w io.Writer) error {  // Metadata 结构体的 WriteTo 方法
    buf := bytes.NewBuffer(make([]byte, 0, 64))  // 创建一个容量为 64 的字节缓冲区
    buf.WriteByte(byte(r.Command))  // 将 Command 类型转换为字节并写入缓冲区
    if err := r.Address.WriteTo(buf); err != nil {  // 调用 Address 结构体的 WriteTo 方法
        return err  // 返回错误
    }
    // use tcp by default
    r.Address.NetworkType = "tcp"  // 默认使用 tcp 网络类型
    _, err := w.Write(buf.Bytes())  // 将缓冲区的内容写入输出流
    return err  // 返回错误
}

func (r *Metadata) Network() string {  // Metadata 结构体的 Network 方法
    return r.Address.Network()  // 返回 Address 结构体的 Network 方法的结果
}

func (r *Metadata) String() string {  // Metadata 结构体的 String 方法
    return r.Address.String()  // 返回 Address 结构体的 String 方法的结果
}

type AddressType byte  // 定义 AddressType 类型为 byte

const (
    IPv4       AddressType = 1  // 定义 IPv4 常量
    DomainName AddressType = 3  // 定义 DomainName 常量
    IPv6       AddressType = 4  // 定义 IPv6 常量
)

type Address struct {  // 定义 Address 结构体
    DomainName  string  // 域名
    Port        int  // 端口
    NetworkType string  // 网络类型
    net.IP  // IP 地址
    AddressType  // 地址类型
}

func (a *Address) String() string {  // Address 结构体的 String 方法
    switch a.AddressType {  // 根据地址类型进行判断
    case IPv4:  // 如果是 IPv4
        return fmt.Sprintf("%s:%d", a.IP.String(), a.Port)  // 返回格式化后的 IPv4 地址和端口
    case IPv6:  // 如果是 IPv6
        return fmt.Sprintf("[%s]:%d", a.IP.String(), a.Port)  // 返回格式化后的 IPv6 地址和端口
    case DomainName:  // 如果是域名
        return fmt.Sprintf("%s:%d", a.DomainName, a.Port)  // 返回格式化后的域名和端口
    default:  // 默认情况
        return "INVALID_ADDRESS_TYPE"  // 返回无效地址类型
    }
}

func (a *Address) Network() string {  // Address 结构体的 Network 方法
    return a.NetworkType  // 返回网络类型
}

func (a *Address) ResolveIP() (net.IP, error) {  // Address 结构体的 ResolveIP 方法
    if a.AddressType == IPv4 || a.AddressType == IPv6 {  // 如果是 IPv4 或 IPv6
        return a.IP, nil  // 直接返回 IP 地址
    }
    if a.IP != nil {  // 如果 IP 地址不为空
        return a.IP, nil  // 直接返回 IP 地址
    }
    addr, err := net.ResolveIPAddr("ip", a.DomainName)  // 解析域名为 IP 地址
    # 如果错误不为空，返回空和错误
    if err != nil:
        return nil, err
    # 将地址的 IP 赋值给变量 a.IP
    a.IP = addr.IP
    # 返回地址的 IP 和空错误
    return addr.IP, nil
}

// 从地址字符串创建新的地址对象
func NewAddressFromAddr(network string, addr string) (*Address, error) {
    // 分割地址字符串，获取主机和端口
    host, portStr, err := net.SplitHostPort(addr)
    if err != nil {
        return nil, err
    }
    // 将端口字符串转换为整数
    port, err := strconv.ParseInt(portStr, 10, 32)
    common.Must(err)
    // 调用函数创建新的地址对象
    return NewAddressFromHostPort(network, host, int(port)), nil
}

// 从主机和端口创建新的地址对象
func NewAddressFromHostPort(network string, host string, port int) *Address {
    // 如果主机是 IP 地址
    if ip := net.ParseIP(host); ip != nil {
        // 如果是 IPv4 地址
        if ip.To4() != nil {
            return &Address{
                IP:          ip,
                Port:        port,
                AddressType: IPv4,
                NetworkType: network,
            }
        }
        // 如果是 IPv6 地址
        return &Address{
            IP:          ip,
            Port:        port,
            AddressType: IPv6,
            NetworkType: network,
        }
    }
    // 如果主机是域名
    return &Address{
        DomainName:  host,
        Port:        port,
        AddressType: DomainName,
        NetworkType: network,
    }
}

// 从输入流中读取地址信息
func (a *Address) ReadFrom(r io.Reader) error {
    byteBuf := [1]byte{}
    // 从输入流中读取一个字节
    _, err := io.ReadFull(r, byteBuf[:])
    if err != nil {
        return common.NewError("unable to read ATYP").Base(err)
    }
    // 将读取的字节转换为地址类型
    a.AddressType = AddressType(byteBuf[0])
    switch a.AddressType {
    case IPv4:
        var buf [6]byte
        // 从输入流中读取 IPv4 地址和端口
        _, err := io.ReadFull(r, buf[:])
        if err != nil {
            return common.NewError("failed to read IPv4").Base(err)
        }
        a.IP = buf[0:4]
        a.Port = int(binary.BigEndian.Uint16(buf[4:6]))
    case IPv6:
        var buf [18]byte
        // 从输入流中读取 IPv6 地址和端口
        _, err := io.ReadFull(r, buf[:])
        if err != nil {
            return common.NewError("failed to read IPv6").Base(err)
        }
        a.IP = buf[0:16]
        a.Port = int(binary.BigEndian.Uint16(buf[16:18]))
    # 如果地址类型是域名
    case DomainName:
        # 从输入流中读取一个字节到 byteBuf 中
        _, err := io.ReadFull(r, byteBuf[:])
        # 读取的字节表示域名的长度
        length := byteBuf[0]
        # 如果读取出错，返回错误信息
        if err != nil {
            return common.NewError("failed to read domain name length")
        }
        # 创建一个长度为 length+2 的字节数组
        buf := make([]byte, length+2)
        # 从输入流中读取 length+2 个字节到 buf 中
        _, err = io.ReadFull(r, buf)
        # 如果读取出错，返回错误信息
        if err != nil {
            return common.NewError("failed to read domain name")
        }
        # 有些浏览器有时会将 IP 作为域名使用
        host := buf[0:length]
        # 如果能够解析为 IP 地址
        if ip := net.ParseIP(string(host)); ip != nil {
            # 将 IP 地址存入地址对象
            a.IP = ip
            # 如果是 IPv4 地址，设置地址类型为 IPv4
            if ip.To4() != nil {
                a.AddressType = IPv4
            } else {
                # 否则设置地址类型为 IPv6
                a.AddressType = IPv6
            }
        } else {
            # 否则将域名存入地址对象
            a.DomainName = string(host)
        }
        # 从 buf 中读取端口号并存入地址对象
        a.Port = int(binary.BigEndian.Uint16(buf[length : length+2]))
    # 如果地址类型不是域名，返回错误信息
    default:
        return common.NewError("invalid ATYP " + strconv.FormatInt(int64(a.AddressType), 10))
    }
    # 返回空，表示没有错误发生
    return nil
# 将地址信息写入到指定的写入器中，返回可能出现的错误
func (a *Address) WriteTo(w io.Writer) error {
    # 写入地址类型的字节数据到写入器中
    _, err := w.Write([]byte{byte(a.AddressType)})
    # 如果出现错误，返回错误
    if err != nil {
        return err
    }
    # 根据地址类型进行不同的处理
    switch a.AddressType {
    # 如果是域名类型
    case DomainName:
        # 写入域名长度的字节数据到写入器中
        w.Write([]byte{byte(len(a.DomainName))})
        # 再写入域名数据到写入器中
        _, err = w.Write([]byte(a.DomainName))
    # 如果是 IPv4 类型
    case IPv4:
        # 将 IPv4 地址转换为 4 字节数据并写入到写入器中
        _, err = w.Write(a.IP.To4())
    # 如果是 IPv6 类型
    case IPv6:
        # 将 IPv6 地址转换为 16 字节数据并写入到写入器中
        _, err = w.Write(a.IP.To16())
    # 如果是其他类型
    default:
        # 返回一个包含错误信息的错误对象
        return common.NewError("invalid ATYP " + strconv.FormatInt(int64(a.AddressType), 10))
    }
    # 如果出现错误，返回错误
    if err != nil {
        return err
    }
    # 将端口号转换为 2 字节数据并写入到写入器中
    port := [2]byte{}
    binary.BigEndian.PutUint16(port[:], uint16(a.Port))
    _, err = w.Write(port[:])
    return err
}
```