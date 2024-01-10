# `v2ray-core\transport\internet\header.go`

```
package internet

import (
    "context"  // 导入上下文包
    "net"  // 导入网络包

    "v2ray.com/core/common"  // 导入自定义包
)

type PacketHeader interface {
    Size() int32  // 定义接口方法 Size，返回 int32 类型
    Serialize([]byte)  // 定义接口方法 Serialize，接收 []byte 类型参数
}

func CreatePacketHeader(config interface{}) (PacketHeader, error) {
    // 使用配置创建对象，并返回对象和错误
    header, err := common.CreateObject(context.Background(), config)
    if err != nil {
        return nil, err
    }
    // 判断对象是否实现了 PacketHeader 接口
    if h, ok := header.(PacketHeader); ok {
        return h, nil
    }
    return nil, newError("not a packet header")  // 返回错误信息
}

type ConnectionAuthenticator interface {
    Client(net.Conn) net.Conn  // 定义接口方法 Client，接收 net.Conn 类型参数，返回 net.Conn 类型
    Server(net.Conn) net.Conn  // 定义接口方法 Server，接收 net.Conn 类型参数，返回 net.Conn 类型
}

func CreateConnectionAuthenticator(config interface{}) (ConnectionAuthenticator, error) {
    // 使用配置创建对象，并返回对象和错误
    auth, err := common.CreateObject(context.Background(), config)
    if err != nil {
        return nil, err
    }
    // 判断对象是否实现了 ConnectionAuthenticator 接口
    if a, ok := auth.(ConnectionAuthenticator); ok {
        return a, nil
    }
    return nil, newError("not a ConnectionAuthenticator")  // 返回错误信息
}
```