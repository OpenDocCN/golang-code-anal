# `trojan-go\proxy\forward\forward.go`

```
package forward

import (
    "context"  // 导入上下文包，用于处理请求的上下文信息

    "github.com/p4gefau1t/trojan-go/config"  // 导入配置包，用于读取和处理配置信息
    "github.com/p4gefau1t/trojan-go/proxy"  // 导入代理包，用于创建代理
    "github.com/p4gefau1t/trojan-go/proxy/client"  // 导入客户端包，用于创建客户端
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入隧道包，用于创建和管理隧道
    "github.com/p4gefau1t/trojan-go/tunnel/dokodemo"  // 导入dokodemo包，用于创建dokodemo隧道
)

const Name = "FORWARD"  // 定义常量Name为"FORWARD"

func init() {
    proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {  // 注册代理创建函数
        cfg := config.FromContext(ctx, Name).(*client.Config)  // 从上下文中获取配置信息
        ctx, cancel := context.WithCancel(ctx)  // 创建一个带有取消函数的上下文
        serverStack := []string{dokodemo.Name}  // 创建服务器堆栈
        clientStack := client.GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, cfg.Router.Enabled)  // 生成客户端树
        c, err := proxy.CreateClientStack(ctx, clientStack)  // 创建客户端堆栈
        if err != nil {
            cancel()  // 取消操作
            return nil, err  // 返回错误
        }
        s, err := proxy.CreateServerStack(ctx, serverStack)  // 创建服务器堆栈
        if err != nil {
            cancel()  // 取消操作
            return nil, err  // 返回错误
        }
        return proxy.NewProxy(ctx, cancel, []tunnel.Server{s}, c), nil  // 返回新代理
    })
}

func init() {
    config.RegisterConfigCreator(Name, func() interface{} {  // 注册配置创建函数
        return new(client.Config)  // 返回一个新的客户端配置
    })
}
```