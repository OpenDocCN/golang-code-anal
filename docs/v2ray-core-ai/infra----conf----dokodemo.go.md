# `v2ray-core\infra\conf\dokodemo.go`

```go
package conf

import (
    "github.com/golang/protobuf/proto"  // 导入 protobuf 包
    "v2ray.com/core/proxy/dokodemo"     // 导入 dokodemo 包
)

type DokodemoConfig struct {
    Host         *Address     `json:"address"`  // 定义 Host 字段，类型为 Address 指针，json 标签为 address
    PortValue    uint16       `json:"port"`     // 定义 PortValue 字段，类型为 uint16，json 标签为 port
    NetworkList  *NetworkList `json:"network"`  // 定义 NetworkList 字段，类型为 NetworkList 指针，json 标签为 network
    TimeoutValue uint32       `json:"timeout"`  // 定义 TimeoutValue 字段，类型为 uint32，json 标签为 timeout
    Redirect     bool         `json:"followRedirect"`  // 定义 Redirect 字段，类型为 bool，json 标签为 followRedirect
    UserLevel    uint32       `json:"userLevel"`  // 定义 UserLevel 字段，类型为 uint32，json 标签为 userLevel
}

func (v *DokodemoConfig) Build() (proto.Message, error) {
    config := new(dokodemo.Config)  // 创建 dokodemo.Config 对象
    if v.Host != nil {  // 如果 Host 字段不为空
        config.Address = v.Host.Build()  // 则将 Host 字段构建的地址赋值给 config.Address
    }
    config.Port = uint32(v.PortValue)  // 将 PortValue 转换为 uint32 类型后赋值给 config.Port
    config.Networks = v.NetworkList.Build()  // 将 NetworkList 字段构建的网络列表赋值给 config.Networks
    config.Timeout = v.TimeoutValue  // 将 TimeoutValue 赋值给 config.Timeout
    config.FollowRedirect = v.Redirect  // 将 Redirect 字段赋值给 config.FollowRedirect
    config.UserLevel = v.UserLevel  // 将 UserLevel 字段赋值给 config.UserLevel
    return config, nil  // 返回 config 对象和 nil 错误
}
```