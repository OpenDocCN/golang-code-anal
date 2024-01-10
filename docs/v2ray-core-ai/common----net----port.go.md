# `v2ray-core\common\net\port.go`

```
// 定义 net 包，导入必要的包
package net

import (
    "encoding/binary"  // 导入二进制编码包
    "strconv"  // 导入字符串转换包
)

// 定义 Port 类型，表示 TCP 和 UDP 协议中的网络端口
type Port uint16

// PortFromBytes 将字节数组转换为 Port 类型，假定字节按大端顺序排列
// @unsafe 调用者必须确保字节数组至少有 2 个元素
func PortFromBytes(port []byte) Port {
    return Port(binary.BigEndian.Uint16(port))  // 使用大端顺序将字节数组转换为 Port 类型
}

// PortFromInt 将整数转换为 Port 类型
// @error 当整数不为正数或大于 65535 时返回错误
func PortFromInt(val uint32) (Port, error) {
    if val > 65535 {
        return Port(0), newError("invalid port range: ", val)  // 返回错误，表示端口范围无效
    }
    return Port(val), nil  // 返回转换后的 Port 类型和 nil 错误
}

// PortFromString 将字符串转换为 Port 类型
// @error 当字符串不是整数或整数值不是有效的端口时返回错误
func PortFromString(s string) (Port, error) {
    val, err := strconv.ParseUint(s, 10, 32)  // 将字符串解析为无符号整数
    if err != nil {
        return Port(0), newError("invalid port range: ", s)  // 返回错误，表示端口范围无效
    }
    return PortFromInt(uint32(val))  // 调用 PortFromInt 将整数转换为 Port 类型
}

// Value 返回 Port 类型对应的 uint16 值
func (p Port) Value() uint16 {
    return uint16(p)  // 返回 Port 类型对应的 uint16 值
}

// String 返回 Port 类型的字符串表示
func (p Port) String() string {
    return strconv.Itoa(int(p))  // 返回 Port 类型的字符串表示
}

// FromPort 返回 PortRange 的起始端口
func (p *PortRange) FromPort() Port {
    return Port(p.From)  // 返回 PortRange 的起始端口
}

// ToPort 返回 PortRange 的结束端口
func (p *PortRange) ToPort() Port {
    return Port(p.To)  // 返回 PortRange 的结束端口
}

// Contains 如果给定端口在 PortRange 的范围内，则返回 true
func (p *PortRange) Contains(port Port) bool {
    return p.FromPort() <= port && port <= p.ToPort()  // 判断给定端口是否在 PortRange 的范围内
}

// SinglePortRange 返回包含单个端口的 PortRange
func SinglePortRange(p Port) *PortRange {
    return &PortRange{
        From: uint32(p),  // 设置 PortRange 的起始端口为给定端口的值
        To:   uint32(p),  // 设置 PortRange 的结束端口为给定端口的值
    }
}

// MemoryPortRange 表示内存中的端口范围
type MemoryPortRange struct {
    From Port  // 起始端口
    To   Port  // 结束端口
}

// Contains 如果给定端口在 MemoryPortRange 的范围内，则返回 true
func (r MemoryPortRange) Contains(port Port) bool {
    # 检查端口号是否在指定范围内
    return r.From <= port && port <= r.To
# 定义一个名为 MemoryPortList 的自定义类型，是一个包含 MemoryPortRange 的切片
type MemoryPortList []MemoryPortRange

# 从给定的 PortList 对象转换为 MemoryPortList 对象
func PortListFromProto(l *PortList) MemoryPortList {
    # 创建一个空的 MemoryPortList 切片，预留足够的容量
    mpl := make(MemoryPortList, 0, len(l.Range))
    # 遍历 PortList 中的每个 PortRange 对象，将其转换为 MemoryPortRange 对象并添加到 MemoryPortList 中
    for _, r := range l.Range {
        mpl = append(mpl, MemoryPortRange{From: Port(r.From), To: Port(r.To)})
    }
    # 返回转换后的 MemoryPortList 对象
    return mpl
}

# 判断 MemoryPortList 是否包含指定的端口
func (mpl MemoryPortList) Contains(port Port) bool {
    # 遍历 MemoryPortList 中的每个 MemoryPortRange 对象，判断是否包含指定的端口
    for _, pr := range mpl {
        if pr.Contains(port) {
            # 如果包含，则返回 true
            return true
        }
    }
    # 如果都不包含，则返回 false
    return false
}
```