# `trojan-go\proxy\server\server.go`

```go
package server

import (
    "context"  // 导入 context 包，用于处理请求的上下文

    "github.com/p4gefau1t/trojan-go/config"  // 导入配置包
    "github.com/p4gefau1t/trojan-go/proxy"  // 导入代理包
    "github.com/p4gefau1t/trojan-go/proxy/client"  // 导入客户端包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自由隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/mux"  // 导入多路复用包
    "github.com/p4gefau1t/trojan-go/tunnel/router"  // 导入路由包
    "github.com/p4gefau1t/trojan-go/tunnel/shadowsocks"  // 导入影子套接字包
    "github.com/p4gefau1t/trojan-go/tunnel/simplesocks"  // 导入简单套接字包
    "github.com/p4gefau1t/trojan-go/tunnel/tls"  // 导入 TLS 包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入传输包
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"  // 导入 Trojan 包
    "github.com/p4gefau1t/trojan-go/tunnel/websocket"  // 导入 WebSocket 包
)

const Name = "SERVER"  // 定义常量 Name 为 "SERVER"

func init() {
    # 注册代理创建函数，根据给定的名称和函数创建代理对象
    proxy.RegisterProxyCreator(Name, func(ctx context.Context) (*proxy.Proxy, error) {
        # 从上下文中获取配置信息
        cfg := config.FromContext(ctx, Name).(*client.Config)
        # 创建一个新的上下文，并返回一个取消函数
        ctx, cancel := context.WithCancel(ctx)
        # 创建一个传输服务器，并返回一个传输服务器对象和可能的错误
        transportServer, err := transport.NewServer(ctx, nil)
        # 如果创建传输服务器出现错误，则取消上下文并返回错误
        if err != nil {
            cancel()
            return nil, err
        }
        # 客户端栈默认包含 freedom 模块
        clientStack := []string{freedom.Name}
        # 如果路由器启用，则客户端栈包含 router 模块
        if cfg.Router.Enabled {
            clientStack = []string{freedom.Name, router.Name}
        }

        # 创建代理节点，设置节点名称、下一个节点、是否为终端节点、上下文和服务器对象
        root := &proxy.Node{
            Name:       transport.Name,
            Next:       make(map[string]*proxy.Node),
            IsEndpoint: false,
            Context:    ctx,
            Server:     transportServer,
        }

        # 如果传输插件未启用，则在根节点后构建 TLS 节点
        if !cfg.TransportPlugin.Enabled {
            root = root.BuildNext(tls.Name)
        }

        # 如果 Shadowsocks 启用，则在 Trojan 子树后构建 Shadowsocks 节点
        trojanSubTree := root
        if cfg.Shadowsocks.Enabled {
            trojanSubTree = trojanSubTree.BuildNext(shadowsocks.Name)
        }
        # 在 Trojan 子树后构建 Mux、SimpleSocks 节点，并将最后一个节点标记为终端节点
        trojanSubTree.BuildNext(trojan.Name).BuildNext(mux.Name).BuildNext(simplesocks.Name).IsEndpoint = true
        # 在 Trojan 子树后构建 Trojan 节点，并将该节点标记为终端节点
        trojanSubTree.BuildNext(trojan.Name).IsEndpoint = true

        # 在根节点后构建 WebSocket 子树
        wsSubTree := root.BuildNext(websocket.Name)
        # 如果 Shadowsocks 启用，则在 WebSocket 子树后构建 Shadowsocks 节点
        if cfg.Shadowsocks.Enabled {
            wsSubTree = wsSubTree.BuildNext(shadowsocks.Name)
        }
        # 在 WebSocket 子树后构建 Mux、SimpleSocks 节点，并将最后一个节点标记为终端节点
        wsSubTree.BuildNext(trojan.Name).BuildNext(mux.Name).BuildNext(simplesocks.Name).IsEndpoint = true
        # 在 WebSocket 子树后构建 Trojan 节点，并将该节点标记为终端节点
        wsSubTree.BuildNext(trojan.Name).IsEndpoint = true

        # 查找所有终端节点，并返回节点列表
        serverList := proxy.FindAllEndpoints(root)
        # 创建客户端栈，并返回客户端列表和可能的错误
        clientList, err := proxy.CreateClientStack(ctx, clientStack)
        # 如果创建客户端栈出现错误，则取消上下文并返回错误
        if err != nil {
            cancel()
            return nil, err
        }
        # 创建新的代理对象，并返回该对象和 nil 错误
        return proxy.NewProxy(ctx, cancel, serverList, clientList), nil
    })
# 闭合前面的函数定义
```