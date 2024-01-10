# `v2ray-core\infra\conf\vmess.go`

```
package conf

import (
    "encoding/json"
    "strings"

    "github.com/golang/protobuf/proto"

    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/proxy/vmess"
    "v2ray.com/core/proxy/vmess/inbound"
    "v2ray.com/core/proxy/vmess/outbound"
)

type VMessAccount struct {
    ID       string `json:"id"` // VMess 账户的 ID
    AlterIds uint16 `json:"alterId"` // VMess 账户的 AlterId
    Security string `json:"security"` // VMess 账户的安全设置
}

// Build implements Buildable
func (a *VMessAccount) Build() *vmess.Account {
    var st protocol.SecurityType
    switch strings.ToLower(a.Security) {
    case "aes-128-gcm":
        st = protocol.SecurityType_AES128_GCM
    case "chacha20-poly1305":
        st = protocol.SecurityType_CHACHA20_POLY1305
    case "auto":
        st = protocol.SecurityType_AUTO
    case "none":
        st = protocol.SecurityType_NONE
    default:
        st = protocol.SecurityType_AUTO
    }
    return &vmess.Account{
        Id:      a.ID, // 设置账户的 ID
        AlterId: uint32(a.AlterIds), // 设置账户的 AlterId
        SecuritySettings: &protocol.SecurityConfig{ // 设置账户的安全配置
            Type: st, // 设置安全类型
        },
    }
}

type VMessDetourConfig struct {
    ToTag string `json:"to"` // VMess 重定向配置的目标标签
}

// Build implements Buildable
func (c *VMessDetourConfig) Build() *inbound.DetourConfig {
    return &inbound.DetourConfig{
        To: c.ToTag, // 设置重定向配置的目标标签
    }
}

type FeaturesConfig struct {
    Detour *VMessDetourConfig `json:"detour"` // VMess 的特性配置，包括重定向配置
}

type VMessDefaultConfig struct {
    AlterIDs uint16 `json:"alterId"` // VMess 默认配置的 AlterId
    Level    byte   `json:"level"` // VMess 默认配置的级别
}

// Build implements Buildable
func (c *VMessDefaultConfig) Build() *inbound.DefaultConfig {
    config := new(inbound.DefaultConfig)
    config.AlterId = uint32(c.AlterIDs) // 设置默认配置的 AlterId
    if config.AlterId == 0 {
        config.AlterId = 32 // 如果默认配置的 AlterId 为 0，则设置为 32
    }
    config.Level = uint32(c.Level) // 设置默认配置的级别
    return config
}

type VMessInboundConfig struct {
    Users        []json.RawMessage   `json:"clients"` // VMess 传入连接的用户列表
    Features     *FeaturesConfig     `json:"features"` // VMess 的特性配置
    Defaults     *VMessDefaultConfig `json:"default"` // VMess 的默认配置
    # 定义一个名为VMessDetourConfig的指针类型的变量，用于存储Detour配置信息，该变量将被序列化为json的detour字段
    DetourConfig *VMessDetourConfig  `json:"detour"`
    # 定义一个布尔类型的变量，用于表示是否禁用不安全的加密算法，该变量将被序列化为json的disableInsecureEncryption字段
    SecureOnly   bool                `json:"disableInsecureEncryption"`
// 实现 Buildable 接口
func (c *VMessInboundConfig) Build() (proto.Message, error) {
    // 创建一个入站配置对象
    config := &inbound.Config{
        SecureEncryptionOnly: c.SecureOnly,
    }

    // 如果存在默认配置，则构建默认配置
    if c.Defaults != nil {
        config.Default = c.Defaults.Build()
    }

    // 如果存在 DetourConfig，则构建 Detour 配置
    if c.DetourConfig != nil {
        config.Detour = c.DetourConfig.Build()
    } else if c.Features != nil && c.Features.Detour != nil {
        config.Detour = c.Features.Detour.Build()
    }

    // 创建用户数组，遍历用户配置并构建用户对象
    config.User = make([]*protocol.User, len(c.Users))
    for idx, rawData := range c.Users {
        user := new(protocol.User)
        // 解析用户配置为 User 对象
        if err := json.Unmarshal(rawData, user); err != nil {
            return nil, newError("invalid VMess user").Base(err)
        }
        account := new(VMessAccount)
        // 解析用户配置为 VMessAccount 对象
        if err := json.Unmarshal(rawData, account); err != nil {
            return nil, newError("invalid VMess user").Base(err)
        }
        // 将 VMessAccount 转换为 TypedMessage，并赋值给 User 的 Account 字段
        user.Account = serial.ToTypedMessage(account.Build())
        config.User[idx] = user
    }

    // 返回构建好的配置对象和 nil 错误
    return config, nil
}

// VMessOutboundTarget 定义
type VMessOutboundTarget struct {
    Address *Address          `json:"address"`
    Port    uint16            `json:"port"`
    Users   []json.RawMessage `json:"users"`
}

// VMessOutboundConfig 定义
type VMessOutboundConfig struct {
    Receivers []*VMessOutboundTarget `json:"vnext"`
}

// 实现 Buildable 接口
func (c *VMessOutboundConfig) Build() (proto.Message, error) {
    // 创建一个出站配置对象
    config := new(outbound.Config)

    // 如果没有配置 VMess 接收者，则返回错误
    if len(c.Receivers) == 0 {
        return nil, newError("0 VMess receiver configured")
    }
    // 创建服务器端点数组，遍历接收者配置并构建服务器端点对象
    serverSpecs := make([]*protocol.ServerEndpoint, len(c.Receivers));
    # 遍历接收者列表，获取索引和接收者信息
    for idx, rec := range c.Receivers {
        # 如果接收者用户列表为空，则返回错误
        if len(rec.Users) == 0 {
            return nil, newError("0 user configured for VMess outbound")
        }
        # 如果接收者地址为空，则返回错误
        if rec.Address == nil {
            return nil, newError("address is not set in VMess outbound config")
        }
        # 创建服务器端点对象，设置地址和端口
        spec := &protocol.ServerEndpoint{
            Address: rec.Address.Build(),
            Port:    uint32(rec.Port),
        }
        # 遍历接收者用户列表，解析用户信息并添加到服务器端点对象中
        for _, rawUser := range rec.Users {
            user := new(protocol.User)
            # 解析用户信息，如果出错则返回错误
            if err := json.Unmarshal(rawUser, user); err != nil {
                return nil, newError("invalid VMess user").Base(err)
            }
            account := new(VMessAccount)
            # 解析用户账户信息，如果出错则返回错误
            if err := json.Unmarshal(rawUser, account); err != nil {
                return nil, newError("invalid VMess user").Base(err)
            }
            # 将用户账户信息转换为类型化的消息并添加到服务器端点对象中
            user.Account = serial.ToTypedMessage(account.Build())
            spec.User = append(spec.User, user)
        }
        # 将服务器端点对象添加到服务器配置列表中
        serverSpecs[idx] = spec
    }
    # 将服务器配置列表设置为配置对象的接收者
    config.Receiver = serverSpecs
    # 返回配置对象和空错误
    return config, nil
# 闭合前面的函数定义
```