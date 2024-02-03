# `v2ray-core\transport\internet\websocket\config.go`

```go
// +build !confonly

// 导入所需的包
package websocket

import (
    "net/http"

    "v2ray.com/core/common"
    "v2ray.com/core/transport/internet"
)

// 定义协议名称为 websocket
const protocolName = "websocket"

// 获取规范化的路径
func (c *Config) GetNormalizedPath() string {
    // 获取配置中的路径
    path := c.Path
    // 如果路径为空，则返回根路径
    if path == "" {
        return "/"
    }
    // 如果路径不是以斜杠开头，则在路径前加上斜杠
    if path[0] != '/' {
        return "/" + path
    }
    // 返回路径
    return path
}

// 获取请求头
func (c *Config) GetRequestHeader() http.Header {
    // 创建一个空的 HTTP 头部
    header := http.Header{}
    // 遍历配置中的头部信息，添加到 HTTP 头部中
    for _, h := range c.Header {
        header.Add(h.Key, h.Value)
    }
    // 返回 HTTP 头部
    return header
}

// 初始化函数
func init() {
    // 注册 websocket 协议的配置创建器
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        return new(Config)
    }))
}
```