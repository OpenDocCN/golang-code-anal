# `v2ray-core\proxy\vless\encoding\encoding.go`

```go
// +build !confonly
// 声明当前文件不仅仅是配置文件

package encoding
// 声明当前文件属于 encoding 包

//go:generate go run v2ray.com/core/common/errors/errorgen
// 使用 go:generate 命令生成错误处理代码

import (
    "io"
    // 导入 io 包

    "v2ray.com/core/common/buf"
    // 导入 v2ray.com/core/common/buf 包
    "v2ray.com/core/common/net"
    // 导入 v2ray.com/core/common/net 包
    "v2ray.com/core/common/protocol"
    // 导入 v2ray.com/core/common/protocol 包
    "v2ray.com/core/proxy/vless"
    // 导入 v2ray.com/core/proxy/vless 包
)

const (
    Version = byte(0)
    // 声明常量 Version 为字节类型的 0
)

var addrParser = protocol.NewAddressParser(
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv4), net.AddressFamilyIPv4),
    protocol.AddressFamilyByte(byte(protocol.AddressTypeDomain), net.AddressFamilyDomain),
    protocol.AddressFamilyByte(byte(protocol.AddressTypeIPv6), net.AddressFamilyIPv6),
    protocol.PortThenAddress(),
)
// 创建地址解析器 addrParser

// EncodeRequestHeader writes encoded request header into the given writer.
// EncodeRequestHeader 函数将编码后的请求头写入给定的写入器
func EncodeRequestHeader(writer io.Writer, request *protocol.RequestHeader, requestAddons *Addons) error {
    // 定义 EncodeRequestHeader 函数，参数为 writer、request 和 requestAddons，返回错误

    buffer := buf.StackNew()
    // 创建一个新的缓冲区 buffer
    defer buffer.Release()
    // 在函数返回前释放缓冲区

    if err := buffer.WriteByte(request.Version); err != nil {
        return newError("failed to write request version").Base(err)
    }
    // 如果写入请求版本号出错，则返回错误

    if _, err := buffer.Write(request.User.Account.(*vless.MemoryAccount).ID.Bytes()); err != nil {
        return newError("failed to write request user id").Base(err)
    }
    // 如果写入请求用户 ID 出错，则返回错误

    if err := EncodeHeaderAddons(&buffer, requestAddons); err != nil {
        return newError("failed to encode request header addons").Base(err)
    }
    // 如果编码请求头附加信息出错，则返回错误

    if err := buffer.WriteByte(byte(request.Command)); err != nil {
        return newError("failed to write request command").Base(err)
    }
    // 如果写入请求命令出错，则返回错误

    if request.Command != protocol.RequestCommandMux {
        if err := addrParser.WriteAddressPort(&buffer, request.Address, request.Port); err != nil {
            return newError("failed to write request address and port").Base(err)
        }
    }
    // 如果请求命令不是 Mux，则写入请求地址和端口

    if _, err := writer.Write(buffer.Bytes()); err != nil {
        return newError("failed to write request header").Base(err)
    }
    // 如果写入请求头出错，则返回错误

    return nil
    // 返回空值
}

// DecodeRequestHeader decodes and returns (if successful) a RequestHeader from an input stream.
// DecodeRequestHeader 函数解码并从输入流中返回（如果成功）一个 RequestHeader
func DecodeRequestHeader(isfb bool, first *buf.Buffer, reader io.Reader, validator *vless.Validator) (*protocol.RequestHeader, *Addons, error, bool) {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 在函数返回时释放缓冲区
    defer buffer.Release()

    // 创建一个新的请求头对象
    request := new(protocol.RequestHeader)

    // 如果是第一个字节
    if isfb {
        // 从第一个字节中读取版本号
        request.Version = first.Byte(0)
    } else {
        // 从读取器中读取一个字节到缓冲区
        if _, err := buffer.ReadFullFrom(reader, 1); err != nil {
            // 如果读取失败，返回错误
            return nil, nil, newError("failed to read request version").Base(err), false
        }
        // 从缓冲区中获取版本号
        request.Version = buffer.Byte(0)
    }

    // 根据版本号进行不同的处理
    switch request.Version {
    # 根据不同的 case 值进行不同的处理
    case 0:
        # 定义一个长度为 16 的字节数组 id
        var id [16]byte

        # 如果是fb请求，则将请求的第一个字节复制到 id 中
        if isfb {
            copy(id[:], first.BytesRange(1, 17))
        } else {
            # 清空缓冲区
            buffer.Clear()
            # 从 reader 中读取 16 个字节到 buffer 中
            if _, err := buffer.ReadFullFrom(reader, 16); err != nil {
                return nil, nil, newError("failed to read request user id").Base(err), false
            }
            # 将 buffer 中的内容复制到 id 中
            copy(id[:], buffer.Bytes())
        }

        # 根据 id 获取用户信息
        if request.User = validator.Get(id); request.User == nil {
            return nil, nil, newError("invalid request user id"), isfb
        }

        # 如果是fb请求，则将 first 的读取位置向前移动 17 个字节
        if isfb {
            first.Advance(17)
        }

        # 解码请求头附加信息
        requestAddons, err := DecodeHeaderAddons(&buffer, reader)
        if err != nil {
            return nil, nil, newError("failed to decode request header addons").Base(err), false
        }

        # 清空缓冲区
        buffer.Clear()
        # 从 reader 中读取 1 个字节到 buffer 中
        if _, err := buffer.ReadFullFrom(reader, 1); err != nil {
            return nil, nil, newError("failed to read request command").Base(err), false
        }

        # 将 buffer 中的第一个字节转换为请求命令
        request.Command = protocol.RequestCommand(buffer.Byte(0))
        switch request.Command {
        case protocol.RequestCommandMux:
            # 设置请求地址和端口
            request.Address = net.DomainAddress("v1.mux.cool")
            request.Port = 0
        case protocol.RequestCommandTCP, protocol.RequestCommandUDP:
            # 从 buffer 和 reader 中读取地址和端口信息
            if addr, port, err := addrParser.ReadAddressPort(&buffer, reader); err == nil {
                request.Address = addr
                request.Port = port
            }
        }

        # 如果请求地址为空，则返回错误
        if request.Address == nil {
            return nil, nil, newError("invalid request address"), false
        }

        # 返回请求对象、附加信息、空错误和 false
        return request, requestAddons, nil, false

    default:
        # 返回空请求对象、空附加信息、错误信息和 isfb
        return nil, nil, newError("invalid request version"), isfb
    }
// EncodeResponseHeader 将编码后的响应头写入给定的写入器
func EncodeResponseHeader(writer io.Writer, request *protocol.RequestHeader, responseAddons *Addons) error {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 在函数返回时释放缓冲区
    defer buffer.Release()
    // 写入响应版本号到缓冲区
    if err := buffer.WriteByte(request.Version); err != nil {
        return newError("failed to write response version").Base(err)
    }
    // 编码响应头附加信息到缓冲区
    if err := EncodeHeaderAddons(&buffer, responseAddons); err != nil {
        return newError("failed to encode response header addons").Base(err)
    }
    // 将缓冲区的内容写入到给定的写入器
    if _, err := writer.Write(buffer.Bytes()); err != nil {
        return newError("failed to write response header").Base(err)
    }
    // 返回空值
    return nil
}

// DecodeResponseHeader 从输入流解码并返回（如果成功）一个 ResponseHeader
func DecodeResponseHeader(reader io.Reader, request *protocol.RequestHeader) (*Addons, error) {
    // 创建一个缓冲区
    buffer := buf.StackNew()
    // 在函数返回时释放缓冲区
    defer buffer.Release()
    // 从输入流中读取一个字节到缓冲区
    if _, err := buffer.ReadFullFrom(reader, 1); err != nil {
        return nil, newError("failed to read response version").Base(err)
    }
    // 检查读取的响应版本号是否与请求版本号相匹配
    if buffer.Byte(0) != request.Version {
        return nil, newError("unexpected response version. Expecting ", int(request.Version), " but actually ", int(buffer.Byte(0)))
    }
    // 从输入流中解码响应头附加信息
    responseAddons, err := DecodeHeaderAddons(&buffer, reader)
    if err != nil {
        return nil, newError("failed to decode response header addons").Base(err)
    }
    // 返回解码后的响应头附加信息
    return responseAddons, nil
}
```