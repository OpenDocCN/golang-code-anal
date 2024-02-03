# `v2ray-core\functions.go`

```go
// +build !confonly
// 标记此文件不仅仅是配置文件

package core
// 导入所需的包
import (
    "bytes"
    "context"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/features/routing"
    "v2ray.com/core/transport/internet/udp"
)

// CreateObject 根据给定的 V2Ray 实例和配置创建一个新对象。V2Ray 实例可以为 nil。
func CreateObject(v *Instance, config interface{}) (interface{}, error) {
    ctx := v.ctx
    if v != nil {
        ctx = context.WithValue(ctx, v2rayKey, v)
    }
    return common.CreateObject(ctx, config)
}

// StartInstance 使用给定的序列化配置启动一个新的 V2Ray 实例。
// 默认情况下，V2Ray 仅支持 protobuf 格式的配置，即 configFormat = "protobuf"。调用者需要加载其他包以添加 JSON 支持。
//
// v2ray:api:stable
func StartInstance(configFormat string, configBytes []byte) (*Instance, error) {
    config, err := LoadConfig(configFormat, "", bytes.NewReader(configBytes))
    if err != nil {
        return nil, err
    }
    instance, err := New(config)
    if err != nil {
        return nil, err
    }
    if err := instance.Start(); err != nil {
        return nil, err
    }
    return instance, nil
}

// Dial 为上游调用者提供了一个通过 V2Ray 创建 net.Conn 的简单方法。
// 它将请求分派到给定 V2Ray 实例的目标。
// 由于它在代理上下文中，返回的 net.Conn 中的 LocalAddr() 和 RemoteAddr() 不会显示用于通信的真实地址。
//
// v2ray:api:stable
func Dial(ctx context.Context, v *Instance, dest net.Destination) (net.Conn, error) {
    dispatcher := v.GetFeature(routing.DispatcherType())
    if dispatcher == nil {
        return nil, newError("routing.Dispatcher is not registered in V2Ray core")
    }
    r, err := dispatcher.(routing.Dispatcher).Dispatch(ctx, dest)
    if err != nil {
        return nil, err
    }
    var readerOpt net.ConnectionOption
    # 如果目标网络是 TCP，则使用 TCP 连接输出多路复用
    if dest.Network == net.Network_TCP:
        readerOpt = net.ConnectionOutputMulti(r.Reader)
    # 如果目标网络不是 TCP，则使用 UDP 连接输出多路复用
    else:
        readerOpt = net.ConnectionOutputMultiUDP(r.Reader)
    # 返回一个新的连接对象，输入为多路复用的连接写入对象，输出为根据目标网络选择的多路复用连接读取对象
    return net.NewConnection(net.ConnectionInputMulti(r.Writer), readerOpt), nil
// DialUDP提供了通过V2Ray实例与远程服务器交换UDP数据包的方法。
// 由于它在代理上下文中，返回的PacketConn中的LocalAddr()将不显示真实地址。
//
// TODO: 未实现SetDeadline() / SetReadDeadline() / SetWriteDeadline()。
//
// v2ray:api:beta
func DialUDP(ctx context.Context, v *Instance) (net.PacketConn, error) {
    // 从V2Ray实例中获取路由分发器
    dispatcher := v.GetFeature(routing.DispatcherType())
    // 如果分发器为空，则返回错误
    if dispatcher == nil {
        return nil, newError("routing.Dispatcher is not registered in V2Ray core")
    }
    // 调用udp包中的DialDispatcher方法，传入上下文和路由分发器，返回PacketConn和错误
    return udp.DialDispatcher(ctx, dispatcher.(routing.Dispatcher))
}
```