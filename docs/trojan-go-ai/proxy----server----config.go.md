# `trojan-go\proxy\server\config.go`

```go
// 导入 server 包
package server

// 导入必要的包
import (
    "github.com/p4gefau1t/trojan-go/config"  // 导入配置包
    "github.com/p4gefau1t/trojan-go/proxy/client"  // 导入代理客户端包
)

// 初始化函数
func init() {
    // 注册配置创建器，使用 Name 作为标识，返回一个代理客户端配置对象
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(client.Config)
    })
}
```