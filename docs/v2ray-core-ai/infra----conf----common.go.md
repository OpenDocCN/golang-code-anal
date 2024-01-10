# `v2ray-core\infra\conf\common.go`

```
package conf

import (
    "encoding/json" // 导入 JSON 包，用于处理 JSON 数据
    "os" // 导入 OS 包，用于操作系统功能
    "strings" // 导入 strings 包，用于处理字符串

    "v2ray.com/core/common/net" // 导入网络包
    "v2ray.com/core/common/protocol" // 导入协议包
)

type StringList []string // 定义字符串列表类型

func NewStringList(raw []string) *StringList { // 创建新的字符串列表
    list := StringList(raw)
    return &list
}

func (v StringList) Len() int { // 获取字符串列表的长度
    return len(v)
}

func (v *StringList) UnmarshalJSON(data []byte) error { // 从 JSON 数据中解析字符串列表
    var strarray []string
    if err := json.Unmarshal(data, &strarray); err == nil { // 如果解析成功
        *v = *NewStringList(strarray) // 创建新的字符串列表
        return nil
    }

    var rawstr string
    if err := json.Unmarshal(data, &rawstr); err == nil { // 如果解析成功
        strlist := strings.Split(rawstr, ",") // 以逗号分隔字符串
        *v = *NewStringList(strlist) // 创建新的字符串列表
        return nil
    }
    return newError("unknown format of a string list: " + string(data)) // 返回错误信息
}

type Address struct { // 地址结构体
    net.Address // 网络地址
}

func (v *Address) UnmarshalJSON(data []byte) error { // 从 JSON 数据中解析地址
    var rawStr string
    if err := json.Unmarshal(data, &rawStr); err != nil { // 如果解析失败
        return newError("invalid address: ", string(data)).Base(err) // 返回错误信息
    }
    v.Address = net.ParseAddress(rawStr) // 解析地址

    return nil
}

func (v *Address) Build() *net.IPOrDomain { // 构建 IP 或域名
    return net.NewIPOrDomain(v.Address) // 创建新的 IP 或域名
}

type Network string // 网络类型

func (v Network) Build() net.Network { // 构建网络
    switch strings.ToLower(string(v)) { // 转换为小写后进行判断
    case "tcp": // 如果是 TCP
        return net.Network_TCP // 返回 TCP 网络
    case "udp": // 如果是 UDP
        return net.Network_UDP // 返回 UDP 网络
    default: // 默认情况
        return net.Network_Unknown // 返回未知网络
    }
}

type NetworkList []Network // 网络列表类型

func (v *NetworkList) UnmarshalJSON(data []byte) error { // 从 JSON 数据中解析网络列表
    var strarray []Network
    if err := json.Unmarshal(data, &strarray); err == nil { // 如果解析成功
        nl := NetworkList(strarray) // 创建新的网络列表
        *v = nl // 赋值给原始网络列表
        return nil
    }

    var rawstr Network
    if err := json.Unmarshal(data, &rawstr); err == nil { // 如果解析成功
        strlist := strings.Split(string(rawstr), ",") // 以逗号分隔字符串
        nl := make([]Network, len(strlist)) // 创建新的网络列表
        for idx, network := range strlist { // 遍历字符串列表
            nl[idx] = Network(network) // 赋值给网络列表
        }
        *v = nl // 赋值给原始网络列表
        return nil
    }
}
    # 返回一个新的错误对象，内容为未知格式的字符串列表加上数据的字符串形式
    return newError("unknown format of a string list: " + string(data))
// Build 方法用于构建 NetworkList 对象中的网络列表
func (v *NetworkList) Build() []net.Network {
    // 如果 NetworkList 为空，则返回默认的 TCP 网络
    if v == nil {
        return []net.Network{net.Network_TCP}
    }

    // 创建一个空的网络列表
    list := make([]net.Network, 0, len(*v))
    // 遍历 NetworkList 中的每个网络对象，调用其 Build 方法并添加到列表中
    for _, network := range *v {
        list = append(list, network.Build())
    }
    // 返回构建好的网络列表
    return list
}

// parseIntPort 方法用于解析字节流中的端口号
func parseIntPort(data []byte) (net.Port, error) {
    var intPort uint32
    // 解析 JSON 数据中的端口号
    err := json.Unmarshal(data, &intPort)
    if err != nil {
        return net.Port(0), err
    }
    // 将解析得到的整数端口号转换为 net.Port 类型并返回
    return net.PortFromInt(intPort)
}

// parseStringPort 方法用于解析字符串形式的端口号
func parseStringPort(s string) (net.Port, net.Port, error) {
    // 如果端口号以 "env:" 开头，则从环境变量中获取端口号
    if strings.HasPrefix(s, "env:") {
        s = s[4:]
        s = os.Getenv(s)
    }

    // 根据 "-" 分割端口范围
    pair := strings.SplitN(s, "-", 2)
    // 如果分割后的长度为 0，则表示端口范围无效，返回错误
    if len(pair) == 0 {
        return net.Port(0), net.Port(0), newError("invalid port range: ", s)
    }
    // 如果分割后的长度为 1，则表示端口范围为单个端口，解析并返回
    if len(pair) == 1 {
        port, err := net.PortFromString(pair[0])
        return port, port, err
    }

    // 解析起始端口和结束端口，并返回
    fromPort, err := net.PortFromString(pair[0])
    if err != nil {
        return net.Port(0), net.Port(0), err
    }
    toPort, err := net.PortFromString(pair[1])
    if err != nil {
        return net.Port(0), net.Port(0), err
    }
    return fromPort, toPort, nil
}

// parseJSONStringPort 方法用于解析 JSON 字符串形式的端口号
func parseJSONStringPort(data []byte) (net.Port, net.Port, error) {
    var s string
    // 解析 JSON 数据中的字符串形式的端口号
    err := json.Unmarshal(data, &s)
    if err != nil {
        return net.Port(0), net.Port(0), err
    }
    // 调用 parseStringPort 方法解析字符串形式的端口号并返回
    return parseStringPort(s)
}

// PortRange 结构体用于表示端口范围
type PortRange struct {
    From uint32
    To   uint32
}

// Build 方法用于构建 PortRange 对象
func (v *PortRange) Build() *net.PortRange {
    return &net.PortRange{
        From: v.From,
        To:   v.To,
    }
}

// UnmarshalJSON 方法用于解析 JSON 数据
func (v *PortRange) UnmarshalJSON(data []byte) error {
    // 尝试解析整数形式的端口号
    port, err := parseIntPort(data)
    if err == nil {
        v.From = uint32(port)
        v.To = uint32(port)
        return nil
    }

    // 尝试解析字符串形式的端口号
    from, to, err := parseJSONStringPort(data)
    // 返回解析结果
    # 如果 err 为 nil，则执行以下代码块
    if err == nil:
        # 将 from 转换为 uint32 类型并赋值给 v.From
        v.From = uint32(from)
        # 将 to 转换为 uint32 类型并赋值给 v.To
        v.To = uint32(to)
        # 如果起始端口大于结束端口，返回错误信息
        if v.From > v.To:
            return newError("invalid port range ", v.From, " -> ", v.To)
        # 如果以上条件都满足，返回 nil
        return nil

    # 如果 err 不为 nil，则执行以下代码块
    return newError("invalid port range: ", string(data))
// 定义了一个结构体 PortList，包含一个 PortRange 类型的切片
type PortList struct {
    Range []PortRange
}

// Build 方法用于构建一个 net.PortList 对象
func (list *PortList) Build() *net.PortList {
    portList := new(net.PortList)
    // 遍历 PortList 结构体中的 Range 切片，调用每个 PortRange 对象的 Build 方法，并将结果添加到 portList 中
    for _, r := range list.Range {
        portList.Range = append(portList.Range, r.Build())
    }
    return portList
}

// UnmarshalJSON 方法实现了 encoding/json.Unmarshaler.UnmarshalJSON 接口
func (list *PortList) UnmarshalJSON(data []byte) error {
    var listStr string
    var number uint32
    // 尝试将 JSON 数据解析为字符串
    if err := json.Unmarshal(data, &listStr); err != nil {
        // 如果解析失败，尝试将 JSON 数据解析为数字
        if err2 := json.Unmarshal(data, &number); err2 != nil {
            // 如果两次解析都失败，则返回错误信息
            return newError("invalid port: ", string(data)).Base(err2)
        }
    }
    // 将字符串按逗号分割成切片
    rangelist := strings.Split(listStr, ",")
    // 遍历切片中的每个字符串
    for _, rangeStr := range rangelist {
        // 去除字符串两端的空格
        trimmed := strings.TrimSpace(rangeStr)
        // 如果字符串长度大于 0
        if len(trimmed) > 0 {
            // 如果字符串包含 "-" 符号
            if strings.Contains(trimmed, "-") {
                // 解析字符串表示的端口范围
                from, to, err := parseStringPort(trimmed)
                // 如果解析失败，则返回错误信息
                if err != nil {
                    return newError("invalid port range: ", trimmed).Base(err)
                }
                // 将解析得到的端口范围添加到 PortList 的 Range 中
                list.Range = append(list.Range, PortRange{From: uint32(from), To: uint32(to)})
            } else {
                // 解析字符串表示的单个端口
                port, err := parseIntPort([]byte(trimmed))
                // 如果解析失败，则返回错误信息
                if err != nil {
                    return newError("invalid port: ", trimmed).Base(err)
                }
                // 将解析得到的单个端口添加到 PortList 的 Range 中
                list.Range = append(list.Range, PortRange{From: uint32(port), To: uint32(port)})
            }
        }
    }
    // 如果 number 不为 0，则将其作为单个端口添加到 PortList 的 Range 中
    if number != 0 {
        list.Range = append(list.Range, PortRange{From: uint32(number), To: uint32(number)})
    }
    return nil
}

// 定义了一个结构体 User，包含 EmailString 和 LevelByte 两个字段
type User struct {
    EmailString string `json:"email"`
    LevelByte   byte   `json:"level"`
}

// Build 方法用于构建一个 protocol.User 对象
func (v *User) Build() *protocol.User {
    return &protocol.User{
        Email: v.EmailString,
        Level: uint32(v.LevelByte),
    }
}
```