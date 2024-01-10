# `v2ray-core\infra\conf\trojan.go`

```
package conf

import (
    "encoding/json" // 导入用于 JSON 编解码的包
    "runtime" // 导入用于运行时信息的包
    "strconv" // 导入用于字符串转换的包
    "syscall" // 导入系统调用的包

    "github.com/golang/protobuf/proto" // nolint: staticcheck

    "v2ray.com/core/common/net" // 导入 V2Ray 的网络包
    "v2ray.com/core/common/protocol" // 导入 V2Ray 的协议包
    "v2ray.com/core/common/serial" // 导入 V2Ray 的序列化包
    "v2ray.com/core/proxy/trojan" // 导入 Trojan 代理包
)

// TrojanServerTarget is configuration of a single trojan server
type TrojanServerTarget struct {
    Address  *Address `json:"address"` // Trojan 服务器的地址
    Port     uint16   `json:"port"` // Trojan 服务器的端口
    Password string   `json:"password"` // Trojan 服务器的密码
    Email    string   `json:"email"` // Trojan 服务器的邮箱
    Level    byte     `json:"level"` // Trojan 服务器的级别
}

// TrojanClientConfig is configuration of trojan servers
type TrojanClientConfig struct {
    Servers []*TrojanServerTarget `json:"servers"` // Trojan 客户端的服务器配置
}

// Build implements Buildable
func (c *TrojanClientConfig) Build() (proto.Message, error) {
    config := new(trojan.ClientConfig) // 创建 Trojan 客户端配置对象

    if len(c.Servers) == 0 { // 如果服务器列表为空
        return nil, newError("0 Trojan server configured.") // 返回错误信息
    }

    serverSpecs := make([]*protocol.ServerEndpoint, len(c.Servers)) // 创建服务器端点列表
    for idx, rec := range c.Servers { // 遍历服务器列表
        if rec.Address == nil { // 如果服务器地址为空
            return nil, newError("Trojan server address is not set.") // 返回错误信息
        }
        if rec.Port == 0 { // 如果端口为0
            return nil, newError("Invalid Trojan port.") // 返回错误信息
        }
        if rec.Password == "" { // 如果密码为空
            return nil, newError("Trojan password is not specified.") // 返回错误信息
        }
        account := &trojan.Account{ // 创建 Trojan 账户对象
            Password: rec.Password, // 设置密码
        }
        trojan := &protocol.ServerEndpoint{ // 创建 Trojan 服务器端点对象
            Address: rec.Address.Build(), // 设置地址
            Port:    uint32(rec.Port), // 设置端口
            User: []*protocol.User{ // 创建用户列表
                {
                    Level:   uint32(rec.Level), // 设置级别
                    Email:   rec.Email, // 设置邮箱
                    Account: serial.ToTypedMessage(account), // 设置账户
                },
            },
        }

        serverSpecs[idx] = trojan // 将服务器端点对象添加到列表中
    }

    config.Server = serverSpecs // 设置配置对象的服务器列表

    return config, nil // 返回配置对象和空错误
}

// TrojanInboundFallback is fallback configuration
// TrojanInboundFallback 结构体定义，包含了传入连接的回退配置信息
type TrojanInboundFallback struct {
    Alpn string          `json:"alpn"`  // 传输层协议名称
    Path string          `json:"path"`  // 路径
    Type string          `json:"type"`  // 类型
    Dest json.RawMessage `json:"dest"`  // 目标地址
    Xver uint64          `json:"xver"`  // 版本号
}

// TrojanUserConfig 结构体定义，包含了用户配置信息
type TrojanUserConfig struct {
    Password string `json:"password"`  // 用户密码
    Level    byte   `json:"level"`     // 用户级别
    Email    string `json:"email"`     // 用户邮箱
}

// TrojanServerConfig 结构体定义，包含了传入连接的服务器配置信息
type TrojanServerConfig struct {
    Clients   []*TrojanUserConfig      `json:"clients"`    // 用户配置列表
    Fallback  json.RawMessage          `json:"fallback"`   // 回退配置
    Fallbacks []*TrojanInboundFallback `json:"fallbacks"`  // 回退配置列表
}

// Build 实现了 Buildable 接口
func (c *TrojanServerConfig) Build() (proto.Message, error) {
    config := new(trojan.ServerConfig)  // 创建 Trojan 服务器配置对象

    if len(c.Clients) == 0 {  // 如果用户配置列表为空
        return nil, newError("No trojan user settings.")  // 返回错误信息
    }

    config.Users = make([]*protocol.User, len(c.Clients))  // 初始化用户列表

    for idx, rawUser := range c.Clients {  // 遍历用户配置列表
        user := new(protocol.User)  // 创建用户对象
        account := &trojan.Account{  // 创建 Trojan 账户对象
            Password: rawUser.Password,  // 设置密码
        }

        user.Email = rawUser.Email  // 设置邮箱
        user.Level = uint32(rawUser.Level)  // 设置用户级别
        user.Account = serial.ToTypedMessage(account)  // 设置账户信息
        config.Users[idx] = user  // 将用户对象添加到用户列表中
    }

    if c.Fallback != nil {  // 如果存在回退配置
        return nil, newError(`Trojan settings: please use "fallbacks":[{}] instead of "fallback":{}`)  // 返回错误信息
    }

    for _, fb := range c.Fallbacks {  // 遍历回退配置列表
        var i uint16  // 定义整型变量
        var s string  // 定义字符串变量
        if err := json.Unmarshal(fb.Dest, &i); err == nil {  // 尝试解析目标地址为整型
            s = strconv.Itoa(int(i))  // 转换为字符串
        } else {
            _ = json.Unmarshal(fb.Dest, &s)  // 解析目标地址为字符串
        }
        config.Fallbacks = append(config.Fallbacks, &trojan.Fallback{  // 将回退配置添加到服务器配置中
            Alpn: fb.Alpn,  // 设置传输层协议名称
            Path: fb.Path,  // 设置路径
            Type: fb.Type,  // 设置类型
            Dest: s,        // 设置目标地址
            Xver: fb.Xver,  // 设置版本号
        })
    }
}
    // 遍历配置中的所有回退选项
    for _, fb := range config.Fallbacks {
        /*
            // 如果回退选项的协议是 h2 并且路径不为空，则返回错误
            if fb.Alpn == "h2" && fb.Path != "" {
                return nil, newError(`Trojan fallbacks: "alpn":"h2" doesn't support "path"`)
            }
        */
        // 如果回退选项的路径不为空并且第一个字符不是斜杠，则返回错误
        if fb.Path != "" && fb.Path[0] != '/' {
            return nil, newError(`Trojan fallbacks: "path" must be empty or start with "/"`)
        }
        // 如果回退选项的类型为空并且目标不为空
        if fb.Type == "" && fb.Dest != "" {
            // 如果目标是 "serve-ws-none"，则类型为 "serve"，否则根据目标的不同情况设置类型
            if fb.Dest == "serve-ws-none" {
                fb.Type = "serve"
            } else {
                switch fb.Dest[0] {
                case '@', '/':
                    fb.Type = "unix"
                    // 如果目标以 '@' 开头且长度大于1且第二个字符也是 '@' 并且运行时环境是 Linux，则处理目标地址
                    if fb.Dest[0] == '@' && len(fb.Dest) > 1 && fb.Dest[1] == '@' && runtime.GOOS == "linux" {
                        fullAddr := make([]byte, len(syscall.RawSockaddrUnix{}.Path)) // may need padding to work in front of haproxy
                        copy(fullAddr, fb.Dest[1:])
                        fb.Dest = string(fullAddr)
                    }
                default:
                    // 如果目标是数字，则设置类型为 "tcp"，否则根据目标是否包含端口号设置类型
                    if _, err := strconv.Atoi(fb.Dest); err == nil {
                        fb.Dest = "127.0.0.1:" + fb.Dest
                    }
                    if _, _, err := net.SplitHostPort(fb.Dest); err == nil {
                        fb.Type = "tcp"
                    }
                }
            }
        }
        // 如果回退选项的类型为空，则返回错误
        if fb.Type == "" {
            return nil, newError(`Trojan fallbacks: please fill in a valid value for every "dest"`)
        }
        // 如果回退选项的 PROXY 协议版本大于2，则返回错误
        if fb.Xver > 2 {
            return nil, newError(`Trojan fallbacks: invalid PROXY protocol version, "xver" only accepts 0, 1, 2`)
        }
    }

    // 返回配置和空错误
    return config, nil
# 闭合前面的函数定义
```