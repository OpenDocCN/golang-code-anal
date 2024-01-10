# `trojan-go\tunnel\dokodemo\tunnel.go`

```
package dokodemo

import (
    "context"  // 导入上下文包
    "github.com/p4gefau1t/trojan-go/common"  // 导入公共包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入隧道包
)

const Name = "DOKODEMO"  // 定义常量 Name 为 "DOKODEMO"

type Tunnel struct{ tunnel.Tunnel }  // 定义类型 Tunnel，继承自隧道类型

func (*Tunnel) Name() string {  // 定义方法 Name，返回字符串类型
    return Name  // 返回常量 Name
}

func (*Tunnel) NewServer(ctx context.Context, underlay tunnel.Server) (tunnel.Server, error) {  // 定义方法 NewServer，接收上下文和隧道服务器对象，返回隧道服务器对象和错误
    return NewServer(ctx, underlay)  // 返回新的服务器对象和错误
}

func (*Tunnel) NewClient(ctx context.Context, underlay tunnel.Client) (tunnel.Client, error) {  // 定义方法 NewClient，接收上下文和隧道客户端对象，返回隧道客户端对象和错误
    return nil, common.NewError("not supported")  // 返回空对象和错误信息
}

func init() {  // 初始化函数
    tunnel.RegisterTunnel(Name, &Tunnel{})  // 注册隧道
}
```