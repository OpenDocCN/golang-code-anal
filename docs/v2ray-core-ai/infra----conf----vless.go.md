# `v2ray-core\infra\conf\vless.go`

```go
package conf

import (
    "encoding/json" // 导入 JSON 编解码包
    "runtime" // 导入 runtime 包，用于访问运行时信息
    "strconv" // 导入 strconv 包，用于字符串和基本数据类型之间的转换
    "syscall" // 导入 syscall 包，提供了操作系统底层接口的函数

    "github.com/golang/protobuf/proto" // 导入 protobuf 包

    "v2ray.com/core/common/net" // 导入网络相关的通用包
    "v2ray.com/core/common/protocol" // 导入协议相关的通用包
    "v2ray.com/core/common/serial" // 导入序列化相关的通用包
    "v2ray.com/core/proxy/vless" // 导入 VLess 代理包
    "v2ray.com/core/proxy/vless/inbound" // 导入 VLess 入站代理包
    "v2ray.com/core/proxy/vless/outbound" // 导入 VLess 出站代理包
)

type VLessInboundFallback struct {
    Alpn string          `json:"alpn"` // 定义 VLess 入站回退的协议名称
    Path string          `json:"path"` // 定义 VLess 入站回退的路径
    Type string          `json:"type"` // 定义 VLess 入站回退的类型
    Dest json.RawMessage `json:"dest"` // 定义 VLess 入站回退的目标
    Xver uint64          `json:"xver"` // 定义 VLess 入站回退的版本号
}

type VLessInboundConfig struct {
    Clients    []json.RawMessage       `json:"clients"` // 定义 VLess 入站的客户端列表
    Decryption string                  `json:"decryption"` // 定义 VLess 入站的解密方式
    Fallback   json.RawMessage         `json:"fallback"` // 定义 VLess 入站的回退配置
    Fallbacks  []*VLessInboundFallback `json:"fallbacks"` // 定义 VLess 入站的多个回退配置
}

// Build implements Buildable
func (c *VLessInboundConfig) Build() (proto.Message, error) {
    // 创建一个新的入站配置对象
    config := new(inbound.Config)

    if len(c.Clients) == 0 {
        // 如果客户端列表为空，则返回错误
        //return nil, newError(`VLESS settings: "clients" is empty`)
    }
    // 初始化入站配置的客户端列表
    config.Clients = make([]*protocol.User, len(c.Clients))
    # 遍历 VLESS 客户端列表
    for idx, rawUser := range c.Clients:
        # 创建一个新的 User 对象
        user := new(protocol.User)
        # 将原始用户数据解析为 User 对象
        if err := json.Unmarshal(rawUser, user); err != nil:
            return nil, newError(`VLESS clients: invalid user`).Base(err)
        
        # 创建一个新的 VLESS 账户对象
        account := new(vless.Account)
        # 将原始用户数据解析为 VLESS 账户对象
        if err := json.Unmarshal(rawUser, account); err != nil:
            return nil, newError(`VLESS clients: invalid user`).Base(err)

        # 检查账户的流量类型是否合法
        switch account.Flow:
            case "", "xtls-rprx-origin", "xtls-rprx-direct":
            default:
                return nil, newError(`VLESS clients: "flow" doesn't support "` + account.Flow + `" in this version`)
        
        # 检查账户的加密方式是否为空
        if account.Encryption != "":
            return nil, newError(`VLESS clients: "encryption" should not in inbound settings`)
        
        # 将账户对象转换为 TypedMessage 并赋值给用户对象的账户属性
        user.Account = serial.ToTypedMessage(account)
        # 将用户对象添加到配置的客户端列表中
        config.Clients[idx] = user

    # 检查解密方式是否为 "none"
    if c.Decryption != "none":
        return nil, newError(`VLESS settings: please add/set "decryption":"none" to every settings`)
    # 将解密方式赋值给配置的解密属性
    config.Decryption = c.Decryption

    # 检查是否存在回退设置
    if c.Fallback != nil:
        return nil, newError(`VLESS settings: please use "fallbacks":[{}] instead of "fallback":{}`)
    # 遍历回退设置列表
    for _, fb := range c.Fallbacks:
        # 定义变量 i 和 s
        var i uint16
        var s string
        # 尝试将目标解析为 uint16 类型，如果成功则转换为字符串，否则直接解析为字符串
        if err := json.Unmarshal(fb.Dest, &i); err == nil:
            s = strconv.Itoa(int(i))
        else:
            _ = json.Unmarshal(fb.Dest, &s)
        # 将回退设置添加到配置的回退列表中
        config.Fallbacks = append(config.Fallbacks, &inbound.Fallback{
            Alpn: fb.Alpn,
            Path: fb.Path,
            Type: fb.Type,
            Dest: s,
            Xver: fb.Xver,
        })
    }
    // 遍历配置中的所有回退选项
    for _, fb := range config.Fallbacks {
        /*
            // 如果回退选项的协议是 h2 并且路径不为空，则返回错误
            if fb.Alpn == "h2" && fb.Path != "" {
                return nil, newError(`VLESS fallbacks: "alpn":"h2" doesn't support "path"`)
            }
        */
        // 如果回退选项的路径不为空并且不以斜杠开头，则返回错误
        if fb.Path != "" && fb.Path[0] != '/' {
            return nil, newError(`VLESS fallbacks: "path" must be empty or start with "/"`)
        }
        // 如果回退选项的类型为空并且目标不为空
        if fb.Type == "" && fb.Dest != "" {
            // 如果目标是 "serve-ws-none"，则类型为 "serve"，否则根据目标的不同情况确定类型
            if fb.Dest == "serve-ws-none" {
                fb.Type = "serve"
            } else {
                switch fb.Dest[0] {
                // 如果目标以 '@' 或 '/' 开头，则类型为 "unix"，并根据操作系统和目标的不同情况进行处理
                case '@', '/':
                    fb.Type = "unix"
                    if fb.Dest[0] == '@' && len(fb.Dest) > 1 && fb.Dest[1] == '@' && runtime.GOOS == "linux" {
                        fullAddr := make([]byte, len(syscall.RawSockaddrUnix{}.Path)) // may need padding to work in front of haproxy
                        copy(fullAddr, fb.Dest[1:])
                        fb.Dest = string(fullAddr)
                    }
                // 如果目标是数字，则类型为 "tcp"，如果目标是 IP 地址和端口，则类型为 "tcp"
                default:
                    if _, err := strconv.Atoi(fb.Dest); err == nil {
                        fb.Dest = "127.0.0.1:" + fb.Dest
                    }
                    if _, _, err := net.SplitHostPort(fb.Dest); err == nil {
                        fb.Type = "tcp"
                    }
                }
            }
        }
        // 如果回退选项的类型仍然为空，则返回错误
        if fb.Type == "" {
            return nil, newError(`VLESS fallbacks: please fill in a valid value for every "dest"`)
        }
        // 如果回退选项的 PROXY 协议版本大于 2，则返回错误
        if fb.Xver > 2 {
            return nil, newError(`VLESS fallbacks: invalid PROXY protocol version, "xver" only accepts 0, 1, 2`)
        }
    }

    // 返回配置和空错误
    return config, nil
// 结构体定义，表示 VLessOutboundVnext 对象，包含地址、端口和用户信息
type VLessOutboundVnext struct {
    Address *Address          `json:"address"`
    Port    uint16            `json:"port"`
    Users   []json.RawMessage `json:"users"`
}

// 结构体定义，表示 VLessOutboundConfig 对象，包含多个 VLessOutboundVnext 对象
type VLessOutboundConfig struct {
    Vnext []*VLessOutboundVnext `json:"vnext"`
}

// 实现 Buildable 接口的方法，用于构建 VLessOutboundConfig 对象
func (c *VLessOutboundConfig) Build() (proto.Message, error) {

    // 创建一个 outbound.Config 对象
    config := new(outbound.Config)

    // 如果 VLessOutboundConfig 中的 Vnext 数组为空，则返回错误
    if len(c.Vnext) == 0 {
        return nil, newError(`VLESS settings: "vnext" is empty`)
    }
    // 根据 VLessOutboundConfig 中的 Vnext 数组长度创建对应长度的 protocol.ServerEndpoint 数组
    config.Vnext = make([]*protocol.ServerEndpoint, len(c.Vnext))
    # 遍历 Vnext 列表，获取索引和记录
    for idx, rec := range c.Vnext {
        # 如果地址为空，则返回错误
        if rec.Address == nil {
            return nil, newError(`VLESS vnext: "address" is not set`)
        }
        # 如果用户列表为空，则返回错误
        if len(rec.Users) == 0 {
            return nil, newError(`VLESS vnext: "users" is empty`)
        }
        # 创建服务器端点对象
        spec := &protocol.ServerEndpoint{
            Address: rec.Address.Build(),  # 设置地址
            Port:    uint32(rec.Port),     # 设置端口
            User:    make([]*protocol.User, len(rec.Users)),  # 创建用户列表
        }
        # 遍历用户列表
        for idx, rawUser := range rec.Users {
            user := new(protocol.User)  # 创建用户对象
            # 解析用户 JSON 数据
            if err := json.Unmarshal(rawUser, user); err != nil {
                return nil, newError(`VLESS users: invalid user`).Base(err)  # 返回解析错误
            }
            account := new(vless.Account)  # 创建 VLESS 账户对象
            # 解析用户 JSON 数据
            if err := json.Unmarshal(rawUser, account); err != nil {
                return nil, newError(`VLESS users: invalid user`).Base(err)  # 返回解析错误
            }

            # 检查流量类型是否支持
            switch account.Flow {
            case "", "xtls-rprx-origin", "xtls-rprx-origin-udp443", "xtls-rprx-direct", "xtls-rprx-direct-udp443":
            default:
                return nil, newError(`VLESS users: "flow" doesn't support "` + account.Flow + `" in this version`)  # 返回不支持的流量类型错误
            }

            # 检查加密方式是否为 "none"
            if account.Encryption != "none" {
                return nil, newError(`VLESS users: please add/set "encryption":"none" for every user`)  # 返回加密方式错误
            }

            # 将账户对象转换为 TypedMessage，并设置给用户对象
            user.Account = serial.ToTypedMessage(account)
            spec.User[idx] = user  # 将用户对象添加到服务器端点的用户列表中
        }
        config.Vnext[idx] = spec  # 将服务器端点对象添加到配置中
    }

    return config, nil  # 返回配置和空错误
# 闭合前面的函数定义
```