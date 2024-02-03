# `trojan-go\proxy\nat\nat.go`

```go
//go:build linux
// +build linux

// 定义包名为nat
package nat

// 导入所需的包
import (
    "context"
    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/proxy"
    "github.com/p4gefau1t/trojan-go/proxy/client"
    "github.com/p4gefau1t/trojan-go/tunnel"
    "github.com/p4gefau1t/trojan-go/tunnel/tproxy"
)

// 定义常量Name为"NAT"
const Name = "NAT"

// 初始化函数，注册NAT代理的创建函数
func init() {
    proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
        // 从上下文中获取NAT配置
        cfg := config.FromContext(ctx, Name).(*client.Config)
        // 如果启用了路由器，则返回错误
        if cfg.Router.Enabled {
            return nil, common.NewError("router is not allowed in nat mode")
        }
        // 创建新的上下文和取消函数
        ctx, cancel := context.WithCancel(ctx)
        // 定义服务器栈
        serverStack := []string{tproxy.Name}
        // 生成客户端栈
        clientStack := client.GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, false)
        // 创建客户端栈
        c, err := proxy.CreateClientStack(ctx, clientStack)
        if err != nil {
            cancel()
            return nil, err
        }
        // 创建服务器栈
        s, err := proxy.CreateServerStack(ctx, serverStack)
        if err != nil {
            cancel()
            return nil, err
        }
        // 返回新的代理对象
        return proxy.NewProxy(ctx, cancel, []tunnel.Server{s}, c), nil
    })
}

// 初始化函数，注册NAT配置的创建函数
func init() {
    config.RegisterConfigCreator(Name, func() interface{} {
        return new(client.Config)
    })
}
```