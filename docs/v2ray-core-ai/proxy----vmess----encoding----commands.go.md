# `v2ray-core\proxy\vmess\encoding\commands.go`

```
package encoding

import (
    "encoding/binary"  // 导入二进制编码包
    "io"  // 导入输入输出包

    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/buf"  // 导入缓冲区包
    "v2ray.com/core/common/net"  // 导入网络包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/serial"  // 导入序列化包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
)

var (
    ErrCommandTypeMismatch = newError("Command type mismatch.")  // 定义命令类型不匹配错误
    ErrUnknownCommand      = newError("Unknown command.")  // 定义未知命令错误
    ErrCommandTooLarge     = newError("Command too large.")  // 定义命令过大错误
)

func MarshalCommand(command interface{}, writer io.Writer) error {
    if command == nil {
        return ErrUnknownCommand  // 如果命令为空，则返回未知命令错误
    }

    var cmdID byte  // 定义命令 ID
    var factory CommandFactory  // 定义命令工厂接口
    switch command.(type) {  // 根据命令类型进行判断
    case *protocol.CommandSwitchAccount:  // 如果是 CommandSwitchAccount 类型的命令
        factory = new(CommandSwitchAccountFactory)  // 创建 CommandSwitchAccountFactory 工厂
        cmdID = 1  // 设置命令 ID 为 1
    default:
        return ErrUnknownCommand  // 其他类型的命令返回未知命令错误
    }

    buffer := buf.New()  // 创建新的缓冲区
    defer buffer.Release()  // 在函数返回前释放缓冲区

    err := factory.Marshal(command, buffer)  // 使用工厂将命令序列化到缓冲区
    if err != nil {
        return err  // 如果序列化过程中出错，则返回错误
    }

    auth := Authenticate(buffer.Bytes())  // 对缓冲区内容进行认证
    length := buffer.Len() + 4  // 计算长度
    if length > 255 {
        return ErrCommandTooLarge  // 如果长度超过 255，则返回命令过大错误
    }

    common.Must2(writer.Write([]byte{cmdID, byte(length), byte(auth >> 24), byte(auth >> 16), byte(auth >> 8), byte(auth)}))  // 写入命令 ID 和长度
    common.Must2(writer.Write(buffer.Bytes()))  // 写入缓冲区内容
    return nil  // 返回空
}

func UnmarshalCommand(cmdID byte, data []byte) (protocol.ResponseCommand, error) {
    if len(data) <= 4 {
        return nil, newError("insufficient length")  // 如果数据长度不足 4，则返回长度不足错误
    }
    expectedAuth := Authenticate(data[4:])  // 对数据进行认证
    actualAuth := binary.BigEndian.Uint32(data[:4])  // 从数据中读取实际认证值
    if expectedAuth != actualAuth {
        return nil, newError("invalid auth")  // 如果认证失败，则返回无效认证错误
    }

    var factory CommandFactory  // 定义命令工厂接口
    switch cmdID {  // 根据命令 ID 进行判断
    case 1:  // 如果是命令 ID 为 1
        factory = new(CommandSwitchAccountFactory)  // 创建 CommandSwitchAccountFactory 工厂
    default:
        return nil, ErrUnknownCommand  // 其他命令 ID 返回未知命令错误
    }
    return factory.Unmarshal(data[4:])  // 使用工厂将数据反序列化为命令
}

type CommandFactory interface {
    Marshal(command interface{}, writer io.Writer) error  // 定义序列化命令的接口
    Unmarshal(data []byte) (interface{}, error)  // 定义反序列化命令的接口
}
type CommandSwitchAccountFactory struct {
}

func (f *CommandSwitchAccountFactory) Marshal(command interface{}, writer io.Writer) error {
    // 将命令转换为 CommandSwitchAccount 类型
    cmd, ok := command.(*protocol.CommandSwitchAccount)
    if !ok {
        return ErrCommandTypeMismatch
    }

    // 将主机地址转换为字符串
    hostStr := ""
    if cmd.Host != nil {
        hostStr = cmd.Host.String()
    }
    // 写入主机地址字符串的长度
    common.Must2(writer.Write([]byte{byte(len(hostStr))}))

    // 如果主机地址字符串长度大于 0，则写入主机地址字符串
    if len(hostStr) > 0 {
        common.Must2(writer.Write([]byte(hostStr)))
    }

    // 写入端口号
    common.Must2(serial.WriteUint16(writer, cmd.Port.Value()))

    // 将 ID 转换为字节并写入
    idBytes := cmd.ID.Bytes()
    common.Must2(writer.Write(idBytes))

    // 写入 AlterIds
    common.Must2(serial.WriteUint16(writer, cmd.AlterIds))

    // 写入 Level
    common.Must2(writer.Write([]byte{byte(cmd.Level)}))

    // 写入 ValidMin
    common.Must2(writer.Write([]byte{cmd.ValidMin}))
    return nil
}

func (f *CommandSwitchAccountFactory) Unmarshal(data []byte) (interface{}, error) {
    // 创建 CommandSwitchAccount 对象
    cmd := new(protocol.CommandSwitchAccount)
    if len(data) == 0 {
        return nil, newError("insufficient length.")
    }
    // 读取主机地址字符串的长度
    lenHost := int(data[0])
    if len(data) < lenHost+1 {
        return nil, newError("insufficient length.")
    }
    // 如果主机地址字符串长度大于 0，则解析主机地址
    if lenHost > 0 {
        cmd.Host = net.ParseAddress(string(data[1 : 1+lenHost]))
    }
    // 读取端口号
    portStart := 1 + lenHost
    if len(data) < portStart+2 {
        return nil, newError("insufficient length.")
    }
    cmd.Port = net.PortFromBytes(data[portStart : portStart+2])
    // 读取 ID
    idStart := portStart + 2
    if len(data) < idStart+16 {
        return nil, newError("insufficient length.")
    }
    cmd.ID, _ = uuid.ParseBytes(data[idStart : idStart+16])
    // 读取 AlterIds
    alterIDStart := idStart + 16
    if len(data) < alterIDStart+2 {
        return nil, newError("insufficient length.")
    }
    cmd.AlterIds = binary.BigEndian.Uint16(data[alterIDStart : alterIDStart+2])
    // 读取 Level
    levelStart := alterIDStart + 2
    if len(data) < levelStart+1 {
        return nil, newError("insufficient length.")
    }
    cmd.Level = uint32(data[levelStart])
    // 读取时间
    timeStart := levelStart + 1
}
    # 如果数据长度小于timeStart，则返回空和新的错误信息
    if len(data) < timeStart:
        return nil, newError("insufficient length.")
    # 将数据中从timeStart位置开始的值赋给cmd.ValidMin
    cmd.ValidMin = data[timeStart]
    # 返回cmd和空的错误信息
    return cmd, nil
# 闭合前面的函数定义
```