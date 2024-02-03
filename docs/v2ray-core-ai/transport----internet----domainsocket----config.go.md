# `v2ray-core\transport\internet\domainsocket\config.go`

```go
// +build !confonly

// 声明 domainsocket 包，引入必要的依赖包
package domainsocket

import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
)

// 定义协议名称为 domainsocket
const protocolName = "domainsocket"
// 定义 Unix 域套接字路径的长度
const sizeofSunPath = 108

// 获取 Unix 域套接字地址
func (c *Config) GetUnixAddr() (*net.UnixAddr, error) {
    // 获取配置中的路径
    path := c.Path
    // 如果路径为空，则返回错误
    if path == "" {
        return nil, newError("empty domain socket path")
    }
    // 如果是抽象套接字并且路径不是以 @ 开头，则在路径前加上 @
    if c.Abstract && path[0] != '@' {
        path = "@" + path
    }
    // 如果是抽象套接字并且需要填充，则将路径转换为指定长度的字节数组
    if c.Abstract && c.Padding {
        raw := []byte(path)
        addr := make([]byte, sizeofSunPath)
        for i, c := range raw {
            addr[i] = c
        }
        path = string(addr)
    }
    // 返回 Unix 域套接字地址
    return &net.UnixAddr{
        Name: path,
        Net:  "unix",
    }, nil
}

// 初始化函数，注册 domainsocket 协议的配置创建器
func init() {
    common.Must(internet.RegisterProtocolConfigCreator(protocolName, func() interface{} {
        return new(Config)
    }))
}
```