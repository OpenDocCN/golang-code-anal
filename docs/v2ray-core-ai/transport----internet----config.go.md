# `v2ray-core\transport\internet\config.go`

```
package internet

import (
    "v2ray.com/core/common/serial"
    "v2ray.com/core/features"
)

type ConfigCreator func() interface{}

var (
    globalTransportConfigCreatorCache = make(map[string]ConfigCreator)  // 全局传输配置创建器缓存
    globalTransportSettings           []*TransportConfig  // 全局传输设置
)

const unknownProtocol = "unknown"  // 未知协议常量

func transportProtocolToString(protocol TransportProtocol) string {
    switch protocol {
    case TransportProtocol_TCP:
        return "tcp"
    case TransportProtocol_UDP:
        return "udp"
    case TransportProtocol_HTTP:
        return "http"
    case TransportProtocol_MKCP:
        return "mkcp"
    case TransportProtocol_WebSocket:
        return "websocket"
    case TransportProtocol_DomainSocket:
        return "domainsocket"
    default:
        return unknownProtocol
    }
}

func RegisterProtocolConfigCreator(name string, creator ConfigCreator) error {
    if _, found := globalTransportConfigCreatorCache[name]; found {
        return newError("protocol ", name, " is already registered").AtError()  // 如果协议已经注册，则返回错误
    }
    globalTransportConfigCreatorCache[name] = creator  // 将协议配置创建器添加到全局缓存中
    return nil
}

func CreateTransportConfig(name string) (interface{}, error) {
    creator, ok := globalTransportConfigCreatorCache[name]
    if !ok {
        return nil, newError("unknown transport protocol: ", name)  // 如果协议未知，则返回错误
    }
    return creator(), nil  // 调用协议配置创建器创建配置
}

func (c *TransportConfig) GetTypedSettings() (interface{}, error) {
    return c.Settings.GetInstance()  // 获取类型化的设置
}

func (c *TransportConfig) GetUnifiedProtocolName() string {
    if len(c.ProtocolName) > 0 {
        return c.ProtocolName  // 如果协议名不为空，则返回协议名
    }

    return transportProtocolToString(c.Protocol)  // 否则返回协议对应的字符串
}

func (c *StreamConfig) GetEffectiveProtocol() string {
    if c == nil {
        return "tcp"  // 如果配置为空，则默认返回 TCP
    }

    if len(c.ProtocolName) > 0 {
        return c.ProtocolName  // 如果协议名不为空，则返回协议名
    }

    return transportProtocolToString(c.Protocol)  // 否则返回协议对应的字符串
}

func (c *StreamConfig) GetEffectiveTransportSettings() (interface{}, error) {
    protocol := c.GetEffectiveProtocol()  // 获取有效的协议
    # 返回给定协议的传输设置
    return c.GetTransportSettingsFor(protocol)
# 获取指定协议的传输设置，返回传输设置的接口和可能的错误
func (c *StreamConfig) GetTransportSettingsFor(protocol string) (interface{}, error) {
    # 如果 StreamConfig 对象不为空
    if c != nil {
        # 遍历 TransportSettings 切片
        for _, settings := range c.TransportSettings {
            # 如果设置的统一协议名称与指定协议相同
            if settings.GetUnifiedProtocolName() == protocol {
                # 返回设置的类型化设置
                return settings.GetTypedSettings()
            }
        }
    }

    # 遍历全局传输设置切片
    for _, settings := range globalTransportSettings {
        # 如果设置的统一协议名称与指定协议相同
        if settings.GetUnifiedProtocolName() == protocol {
            # 返回设置的类型化设置
            return settings.GetTypedSettings()
        }
    }

    # 创建指定协议的传输配置
    return CreateTransportConfig(protocol)
}

# 获取有效的安全设置，返回安全设置的接口和可能的错误
func (c *StreamConfig) GetEffectiveSecuritySettings() (interface{}, error) {
    # 遍历安全设置切片
    for _, settings := range c.SecuritySettings {
        # 如果设置的类型与 StreamConfig 的安全类型相同
        if settings.Type == c.SecurityType {
            # 返回设置的实例
            return settings.GetInstance()
        }
    }
    # 返回安全类型的实例
    return serial.GetInstance(c.SecurityType)
}

# 判断是否存在安全设置，返回布尔值
func (c *StreamConfig) HasSecuritySettings() bool {
    # 返回安全类型的长度是否大于 0
    return len(c.SecurityType) > 0
}

# 应用全局传输设置，返回可能的错误
func ApplyGlobalTransportSettings(settings []*TransportConfig) error {
    # 打印已弃用的特性警告
    features.PrintDeprecatedFeatureWarning("global transport settings")
    # 设置全局传输设置
    globalTransportSettings = settings
    # 返回 nil
    return nil
}

# 判断是否存在标签，返回布尔值
func (c *ProxyConfig) HasTag() bool {
    # 返回 ProxyConfig 对象是否不为空且标签长度大于 0
    return c != nil && len(c.Tag) > 0
}

# 判断 TProxyMode 是否启用，返回布尔值
func (m SocketConfig_TProxyMode) IsEnabled() bool {
    # 返回 TProxyMode 是否不等于 Off
    return m != SocketConfig_Off
}
```