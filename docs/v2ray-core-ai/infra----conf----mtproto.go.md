# `v2ray-core\infra\conf\mtproto.go`

```go
package conf

import (
    "encoding/hex"  // 导入编码/解码十六进制的包
    "encoding/json"  // 导入 JSON 编解码的包

    "github.com/golang/protobuf/proto"  // 导入 protobuf 包

    "v2ray.com/core/common/protocol"  // 导入 v2ray 核心通用协议包
    "v2ray.com/core/common/serial"  // 导入 v2ray 核心通用序列化包
    "v2ray.com/core/proxy/mtproto"  // 导入 v2ray 核心代理 MTProto 包
)

type MTProtoAccount struct {
    Secret string `json:"secret"`  // MTProto 账户的密钥
}

// Build implements Buildable
func (a *MTProtoAccount) Build() (*mtproto.Account, error) {
    if len(a.Secret) != 32 {  // 如果密钥长度不等于32
        return nil, newError("MTProto secret must have 32 chars")  // 返回错误信息
    }
    secret, err := hex.DecodeString(a.Secret)  // 将密钥解码为字节流
    if err != nil {  // 如果解码出错
        return nil, newError("failed to decode secret: ", a.Secret).Base(err)  // 返回错误信息
    }
    return &mtproto.Account{  // 返回 MTProto 账户对象
        Secret: secret,  // 设置密钥
    }, nil
}

type MTProtoServerConfig struct {
    Users []json.RawMessage `json:"users"`  // MTProto 服务器配置的用户列表
}

func (c *MTProtoServerConfig) Build() (proto.Message, error) {
    config := &mtproto.ServerConfig{}  // 创建 MTProto 服务器配置对象

    if len(c.Users) == 0 {  // 如果用户列表为空
        return nil, newError("zero MTProto users configured.")  // 返回错误信息
    }
    config.User = make([]*protocol.User, len(c.Users))  // 初始化用户列表
    for idx, rawData := range c.Users {  // 遍历用户列表
        user := new(protocol.User)  // 创建用户对象
        if err := json.Unmarshal(rawData, user); err != nil {  // 解析用户数据
            return nil, newError("invalid MTProto user").Base(err)  // 返回错误信息
        }
        account := new(MTProtoAccount)  // 创建 MTProto 账户对象
        if err := json.Unmarshal(rawData, account); err != nil {  // 解析账户数据
            return nil, newError("invalid MTProto user").Base(err)  // 返回错误信息
        }
        accountProto, err := account.Build()  // 构建账户对象
        if err != nil {  // 如果构建出错
            return nil, newError("failed to parse MTProto user").Base(err)  // 返回错误信息
        }
        user.Account = serial.ToTypedMessage(accountProto)  // 设置用户的账户信息
        config.User[idx] = user  // 将用户添加到配置中
    }

    return config, nil  // 返回配置对象
}

type MTProtoClientConfig struct {
}

func (c *MTProtoClientConfig) Build() (proto.Message, error) {
    config := new(mtproto.ClientConfig)  // 创建 MTProto 客户端配置对象
    return config, nil  // 返回配置对象
}
```