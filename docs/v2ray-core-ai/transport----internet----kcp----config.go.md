# `v2ray-core\transport\internet\kcp\config.go`

```go
// +build !confonly

// 声明包名为 kcp
package kcp

// 导入所需的包
import (
    "crypto/cipher"
    "v2ray.com/core/common"
    "v2ray.com/core/transport/internet"
)

// 声明常量 protocolName 为 "mkcp"
const protocolName = "mkcp"

// GetMTUValue 返回 MTU 设置的值
func (c *Config) GetMTUValue() uint32 {
    // 如果配置为空或者 Mtu 为空，则返回默认值 1350
    if c == nil || c.Mtu == nil {
        return 1350
    }
    return c.Mtu.Value
}

// GetTTIValue 返回 TTI 设置的值
func (c *Config) GetTTIValue() uint32 {
    // 如果配置为空或者 Tti 为空，则返回默认值 50
    if c == nil || c.Tti == nil {
        return 50
    }
    return c.Tti.Value
}

// GetUplinkCapacityValue 返回上行容量设置的值
func (c *Config) GetUplinkCapacityValue() uint32 {
    // 如果配置为空或者 UplinkCapacity 为空，则返回默认值 5
    if c == nil || c.UplinkCapacity == nil {
        return 5
    }
    return c.UplinkCapacity.Value
}

// GetDownlinkCapacityValue 返回下行容量设置的值
func (c *Config) GetDownlinkCapacityValue() uint32 {
    // 如果配置为空或者 DownlinkCapacity 为空，则返回默认值 20
    if c == nil || c.DownlinkCapacity == nil {
        return 20
    }
    return c.DownlinkCapacity.Value
}

// GetWriteBufferSize 返回写缓冲区的大小
func (c *Config) GetWriteBufferSize() uint32 {
    // 如果配置为空或者 WriteBuffer 为空，则返回默认值 2 * 1024 * 1024
    if c == nil || c.WriteBuffer == nil {
        return 2 * 1024 * 1024
    }
    return c.WriteBuffer.Size
}

// GetReadBufferSize 返回读缓冲区的大小
func (c *Config) GetReadBufferSize() uint32 {
    // 如果配置为空或者 ReadBuffer 为空，则返回默认值 2 * 1024 * 1024
    if c == nil || c.ReadBuffer == nil {
        return 2 * 1024 * 1024
    }
    return c.ReadBuffer.Size
}

// GetSecurity 返回安全设置
func (c *Config) GetSecurity() (cipher.AEAD, error) {
    // 如果 Seed 不为空，则返回基于 Seed 的 AEAD 加密算法
    if c.Seed != nil {
        return NewAEADAESGCMBasedOnSeed(c.Seed.Seed), nil
    }
    // 否则返回简单的认证器
    return NewSimpleAuthenticator(), nil
}

// GetPackerHeader 返回数据包头部设置
func (c *Config) GetPackerHeader() (internet.PacketHeader, error) {
    // 如果 HeaderConfig 不为空，则创建数据包头部
    if c.HeaderConfig != nil {
        rawConfig, err := c.HeaderConfig.GetInstance()
        if err != nil {
            return nil, err
        }
        return internet.CreatePacketHeader(rawConfig)
    }
    // 否则返回空
    return nil, nil
}
# 获取发送数据的在途大小，单位为字节
func (c *Config) GetSendingInFlightSize() uint32 {
    # 根据配置获取上行容量值，并转换为字节
    size := c.GetUplinkCapacityValue() * 1024 * 1024 / c.GetMTUValue() / (1000 / c.GetTTIValue())
    # 如果大小小于8字节，则设置为8字节
    if size < 8 {
        size = 8
    }
    # 返回计算后的大小
    return size
}

# 获取发送数据的缓冲区大小，单位为字节
func (c *Config) GetSendingBufferSize() uint32 {
    # 返回写入缓冲区大小除以最大传输单元的值
    return c.GetWriteBufferSize() / c.GetMTUValue()
}

# 获取接收数据的在途大小，单位为字节
func (c *Config) GetReceivingInFlightSize() uint32 {
    # 根据配置获取下行容量值，并转换为字节
    size := c.GetDownlinkCapacityValue() * 1024 * 1024 / c.GetMTUValue() / (1000 / c.GetTTIValue())
    # 如果大小小于8字节，则设置为8字节
    if size < 8 {
        size = 8
    }
    # 返回计算后的大小
    return size
}

# 获取接收数据的缓冲区大小，单位为字节
func (c *Config) GetReceivingBufferSize() uint32 {
    # 返回读取缓冲区大小除以最大传输单元的值
    return c.GetReadBufferSize() / c.GetMTUValue()
}

# 初始化函数，注册协议配置创建器
func init() {
    # 注册协议配置创建器，使用指定的协议名称和匿名函数创建配置对象
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        return new(Config)
    }))
}
```