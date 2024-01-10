# `trojan-go\proxy\client\client.go`

```
package client

import (
    "context"

    "github.com/p4gefau1t/trojan-go/config"  // 导入配置包
    "github.com/p4gefau1t/trojan-go/proxy"   // 导入代理包
    "github.com/p4gefau1t/trojan-go/tunnel/adapter"  // 导入适配器包
    "github.com/p4gefau1t/trojan-go/tunnel/http"  // 导入 HTTP 包
    "github.com/p4gefau1t/trojan-go/tunnel/mux"  // 导入多路复用包
    "github.com/p4gefau1t/trojan-go/tunnel/router"  // 导入路由包
    "github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"  // 导入 Shadowsocks 包
    "github.com/p4gefau1t/trojan-go/tunnel/simplesocks"  // 导入简单 SOCKS 包
    "github.com/p4gefau1t/trojan-go/tunnel/socks"  // 导入 SOCKS 包
    "github.com/p4gefau1t/trojan-go/tunnel/tls"  // 导入 TLS 包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入传输包
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"  // 导入 Trojan 包
    "github.com/p4gefau1t/trojan-go/tunnel/websocket"  // 导入 WebSocket 包
)

const Name = "CLIENT"  // 定义常量 Name 为 "CLIENT"

// GenerateClientTree generate general outbound protocol stack
func GenerateClientTree(transportPlugin bool, muxEnabled bool, wsEnabled bool, ssEnabled bool, routerEnabled bool) []string {
    clientStack := []string{transport.Name}  // 创建一个字符串数组 clientStack，包含 transport.Name
    if !transportPlugin {  // 如果不是使用传输插件
        clientStack = append(clientStack, tls.Name)  // 将 tls.Name 添加到 clientStack 中
    }
    if wsEnabled {  // 如果启用了 WebSocket
        clientStack = append(clientStack, websocket.Name)  // 将 websocket.Name 添加到 clientStack 中
    }
    if ssEnabled {  // 如果启用了 Shadowsocks
        clientStack = append(clientStack, shadowsocks.Name)  // 将 shadowsocks.Name 添加到 clientStack 中
    }
    clientStack = append(clientStack, trojan.Name)  // 将 trojan.Name 添加到 clientStack 中
    if muxEnabled {  // 如果启用了多路复用
        clientStack = append(clientStack, []string{mux.Name, simplesocks.Name}...)  // 将 mux.Name 和 simplesocks.Name 添加到 clientStack 中
    }
    if routerEnabled {  // 如果启用了路由
        clientStack = append(clientStack, router.Name)  // 将 router.Name 添加到 clientStack 中
    }
    return clientStack  // 返回 clientStack
}

func init() {
    # 注册代理创建器，根据给定的名称和函数创建代理
    proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
        # 从上下文中获取配置信息
        cfg := config.FromContext(ctx, Name).(*Config)
        # 创建适配器服务器
        adapterServer, err := adapter.NewServer(ctx, nil)
        # 如果创建适配器服务器出现错误，则返回错误
        if err != nil {
            return nil, err
        }
        # 创建一个新的上下文，并返回一个取消函数
        ctx, cancel := context.WithCancel(ctx)

        # 创建代理节点
        root := &proxy.Node{
            Name:       adapter.Name,
            Next:       make(map[string]*proxy.Node),
            IsEndpoint: false,
            Context:    ctx,
            Server:     adapterServer,
        }

        # 设置代理节点的下一个节点为 http 和 socks，并将它们标记为终端节点
        root.BuildNext(http.Name).IsEndpoint = true
        root.BuildNext(socks.Name).IsEndpoint = true

        # 生成客户端树
        clientStack := GenerateClientTree(cfg.TransportPlugin.Enabled, cfg.Mux.Enabled, cfg.Websocket.Enabled, cfg.Shadowsocks.Enabled, cfg.Router.Enabled)
        # 创建客户端栈
        c, err := proxy.CreateClientStack(ctx, clientStack)
        # 如果创建客户端栈出现错误，则取消上下文并返回错误
        if err != nil {
            cancel()
            return nil, err
        }
        # 查找所有的终端节点
        s := proxy.FindAllEndpoints(root)
        # 创建新的代理，并返回
        return proxy.NewProxy(ctx, cancel, s, c), nil
    })
# 闭合前面的函数定义
```