# `v2ray-core\infra\conf\api.go`

```go
package conf

import (
    "strings"  // 导入字符串操作包

    "v2ray.com/core/app/commander"  // 导入v2ray的命令包
    loggerservice "v2ray.com/core/app/log/command"  // 导入日志服务包
    handlerservice "v2ray.com/core/app/proxyman/command"  // 导入代理管理服务包
    statsservice "v2ray.com/core/app/stats/command"  // 导入统计服务包
    "v2ray.com/core/common/serial"  // 导入v2ray的序列化包
)

type ApiConfig struct {
    Tag      string   `json:"tag"`  // API配置结构体，包含标签和服务列表
    Services []string `json:"services"`  // 服务列表
}

func (c *ApiConfig) Build() (*commander.Config, error) {
    if c.Tag == "" {  // 如果标签为空
        return nil, newError("Api tag can't be empty.")  // 返回错误信息
    }

    services := make([]*serial.TypedMessage, 0, 16)  // 创建一个长度为0，容量为16的序列化消息切片
    for _, s := range c.Services {  // 遍历服务列表
        switch strings.ToLower(s) {  // 将服务名转换为小写后进行匹配
        case "handlerservice":  // 如果是代理管理服务
            services = append(services, serial.ToTypedMessage(&handlerservice.Config{}))  // 将代理管理服务配置转换为序列化消息并添加到服务列表中
        case "loggerservice":  // 如果是日志服务
            services = append(services, serial.ToTypedMessage(&loggerservice.Config{}))  // 将日志服务配置转换为序列化消息并添加到服务列表中
        case "statsservice":  // 如果是统计服务
            services = append(services, serial.ToTypedMessage(&statsservice.Config{}))  // 将统计服务配置转换为序列化消息并添加到服务列表中
        }
    }

    return &commander.Config{  // 返回命令配置对象
        Tag:     c.Tag,  // 设置标签
        Service: services,  // 设置服务列表
    }, nil
}
```