# `v2ray-core\infra\conf\http.go`

```go
package conf

import (
    "encoding/json"  // 导入 JSON 编解码包
    "github.com/golang/protobuf/proto"  // 导入 protobuf 包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/serial"  // 导入序列化包
    "v2ray.com/core/proxy/http"  // 导入 HTTP 代理包
)

type HttpAccount struct {
    Username string `json:"user"`  // 定义 HTTP 账号的用户名，使用 JSON 标签
    Password string `json:"pass"`  // 定义 HTTP 账号的密码，使用 JSON 标签
}

func (v *HttpAccount) Build() *http.Account {
    return &http.Account{  // 构建并返回 HTTP 账号对象
        Username: v.Username,  // 设置账号对象的用户名
        Password: v.Password,  // 设置账号对象的密码
    }
}

type HttpServerConfig struct {
    Timeout     uint32         `json:"timeout"`  // 定义 HTTP 服务器配置的超时时间，使用 JSON 标签
    Accounts    []*HttpAccount `json:"accounts"`  // 定义 HTTP 服务器配置的账号列表，使用 JSON 标签
    Transparent bool           `json:"allowTransparent"`  // 定义 HTTP 服务器配置的透明代理是否允许，使用 JSON 标签
    UserLevel   uint32         `json:"userLevel"`  // 定义 HTTP 服务器配置的用户级别，使用 JSON 标签
}

func (c *HttpServerConfig) Build() (proto.Message, error) {
    config := &http.ServerConfig{  // 构建 HTTP 服务器配置对象
        Timeout:          c.Timeout,  // 设置超时时间
        AllowTransparent: c.Transparent,  // 设置透明代理是否允许
        UserLevel:        c.UserLevel,  // 设置用户级别
    }

    if len(c.Accounts) > 0 {  // 如果账号列表不为空
        config.Accounts = make(map[string]string)  // 创建账号映射
        for _, account := range c.Accounts {  // 遍历账号列表
            config.Accounts[account.Username] = account.Password  // 将账号和密码添加到账号映射中
        }
    }

    return config, nil  // 返回配置对象和空错误
}

type HttpRemoteConfig struct {
    Address *Address          `json:"address"`  // 定义 HTTP 远程配置的地址，使用 JSON 标签
    Port    uint16            `json:"port"`  // 定义 HTTP 远程配置的端口，使用 JSON 标签
    Users   []json.RawMessage `json:"users"`  // 定义 HTTP 远程配置的用户列表，使用 JSON 标签
}
type HttpClientConfig struct {
    Servers []*HttpRemoteConfig `json:"servers"`  // 定义 HTTP 客户端配置的服务器列表，使用 JSON 标签
}

func (v *HttpClientConfig) Build() (proto.Message, error) {
    config := new(http.ClientConfig)  // 创建 HTTP 客户端配置对象
    config.Server = make([]*protocol.ServerEndpoint, len(v.Servers))  // 创建服务器端点列表
    # 遍历服务器配置列表，获取索引和配置信息
    for idx, serverConfig := range v.Servers {
        # 创建服务器端点对象，并设置地址和端口
        server := &protocol.ServerEndpoint{
            Address: serverConfig.Address.Build(),
            Port:    uint32(serverConfig.Port),
        }
        # 遍历服务器配置中的用户列表
        for _, rawUser := range serverConfig.Users {
            # 创建用户对象
            user := new(protocol.User)
            # 解析 JSON 格式的用户数据到用户对象，如果失败则返回错误
            if err := json.Unmarshal(rawUser, user); err != nil {
                return nil, newError("failed to parse HTTP user").Base(err).AtError()
            }
            # 创建 HTTP 账户对象
            account := new(HttpAccount)
            # 解析 JSON 格式的用户数据到账户对象，如果失败则返回错误
            if err := json.Unmarshal(rawUser, account); err != nil {
                return nil, newError("failed to parse HTTP account").Base(err).AtError()
            }
            # 将账户对象转换为 TypedMessage 类型，并赋值给用户对象的账户属性
            user.Account = serial.ToTypedMessage(account.Build())
            # 将用户对象添加到服务器端点对象的用户列表中
            server.User = append(server.User, user)
        }
        # 将服务器端点对象添加到配置对象的服务器列表中
        config.Server[idx] = server
    }
    # 返回配置对象和空错误
    return config, nil
# 闭合前面的函数定义
```