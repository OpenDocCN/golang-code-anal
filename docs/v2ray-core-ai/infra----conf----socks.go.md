# `v2ray-core\infra\conf\socks.go`

```go
package conf

import (
    "encoding/json"

    "github.com/golang/protobuf/proto"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/proxy/socks"
)

type SocksAccount struct {
    Username string `json:"user"`  // 定义 SOCKS 账户的用户名，用于 JSON 序列化
    Password string `json:"pass"`  // 定义 SOCKS 账户的密码，用于 JSON 序列化
}

func (v *SocksAccount) Build() *socks.Account {
    return &socks.Account{
        Username: v.Username,  // 构建 SOCKS 账户对象的用户名
        Password: v.Password,  // 构建 SOCKS 账户对象的密码
    }
}

const (
    AuthMethodNoAuth   = "noauth"    // 定义不需要认证的认证方法
    AuthMethodUserPass = "password"  // 定义用户名密码认证的认证方法
)

type SocksServerConfig struct {
    AuthMethod string          `json:"auth"`  // 定义 SOCKS 服务器配置的认证方法，用于 JSON 序列化
    Accounts   []*SocksAccount `json:"accounts"`  // 定义 SOCKS 服务器配置的账户列表，用于 JSON 序列化
    UDP        bool            `json:"udp"`  // 定义是否启用 UDP，用于 JSON 序列化
    Host       *Address        `json:"ip"`  // 定义 SOCKS 服务器配置的主机地址，用于 JSON 序列化
    Timeout    uint32          `json:"timeout"`  // 定义超时时间，用于 JSON 序列化
    UserLevel  uint32          `json:"userLevel"`  // 定义用户级别，用于 JSON 序列化
}

func (v *SocksServerConfig) Build() (proto.Message, error) {
    config := new(socks.ServerConfig)  // 创建一个新的 SOCKS 服务器配置对象
    switch v.AuthMethod {
    case AuthMethodNoAuth:
        config.AuthType = socks.AuthType_NO_AUTH  // 设置认证类型为不需要认证
    case AuthMethodUserPass:
        config.AuthType = socks.AuthType_PASSWORD  // 设置认证类型为用户名密码认证
    default:
        //newError("unknown socks auth method: ", v.AuthMethod, ". Default to noauth.").AtWarning().WriteToLog()
        config.AuthType = socks.AuthType_NO_AUTH  // 默认设置认证类型为不需要认证
    }

    if len(v.Accounts) > 0 {
        config.Accounts = make(map[string]string, len(v.Accounts))  // 创建一个账户映射
        for _, account := range v.Accounts {
            config.Accounts[account.Username] = account.Password  // 将账户的用户名和密码添加到账户映射中
        }
    }

    config.UdpEnabled = v.UDP  // 设置是否启用 UDP
    if v.Host != nil {
        config.Address = v.Host.Build()  // 构建主机地址
    }

    config.Timeout = v.Timeout  // 设置超时时间
    config.UserLevel = v.UserLevel  // 设置用户级别
    return config, nil  // 返回构建好的 SOCKS 服务器配置对象
}

type SocksRemoteConfig struct {
    Address *Address          `json:"address"`  // 定义远程 SOCKS 配置的地址，用于 JSON 序列化
    Port    uint16            `json:"port"`  // 定义远程 SOCKS 配置的端口，用于 JSON 序列化
    Users   []json.RawMessage `json:"users"`  // 定义远程 SOCKS 配置的用户列表，用于 JSON 序列化
}
type SocksClientConfig struct {
    Servers []*SocksRemoteConfig `json:"servers"`  // 定义 SOCKS 客户端配置的服务器列表，用于 JSON 序列化
}
func (v *SocksClientConfig) Build() (proto.Message, error) {
    // 创建一个新的 SOCKS 客户端配置对象
    config := new(socks.ClientConfig)
    // 为服务器配置数组分配内存空间
    config.Server = make([]*protocol.ServerEndpoint, len(v.Servers))
    // 遍历服务器配置数组
    for idx, serverConfig := range v.Servers {
        // 创建一个新的协议服务器端点对象
        server := &protocol.ServerEndpoint{
            // 设置服务器地址
            Address: serverConfig.Address.Build(),
            // 设置服务器端口
            Port:    uint32(serverConfig.Port),
        }
        // 遍历服务器配置中的用户数组
        for _, rawUser := range serverConfig.Users {
            // 创建一个新的协议用户对象
            user := new(protocol.User)
            // 将原始用户数据解析为协议用户对象
            if err := json.Unmarshal(rawUser, user); err != nil {
                // 如果解析失败，则返回错误
                return nil, newError("failed to parse Socks user").Base(err).AtError()
            }
            // 创建一个新的 SOCKS 账户对象
            account := new(SocksAccount)
            // 将原始用户数据解析为 SOCKS 账户对象
            if err := json.Unmarshal(rawUser, account); err != nil {
                // 如果解析失败，则返回错误
                return nil, newError("failed to parse socks account").Base(err).AtError()
            }
            // 将账户对象转换为协议消息并赋值给用户对象的账户字段
            user.Account = serial.ToTypedMessage(account.Build())
            // 将用户对象添加到服务器对象的用户数组中
            server.User = append(server.User, user)
        }
        // 将服务器对象添加到配置对象的服务器数组中
        config.Server[idx] = server
    }
    // 返回配置对象和空错误
    return config, nil
}
```