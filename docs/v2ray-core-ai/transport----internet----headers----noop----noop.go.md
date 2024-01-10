# `v2ray-core\transport\internet\headers\noop\noop.go`

```
package noop

import (
    "context"  // 导入 context 包
    "net"      // 导入 net 包

    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
)

type NoOpHeader struct{}  // 定义 NoOpHeader 结构体

func (NoOpHeader) Size() int32 {  // 实现 Size 方法，返回整数
    return 0
}

// Serialize implements PacketHeader.
func (NoOpHeader) Serialize([]byte) {}  // 实现 Serialize 方法，接收字节切片参数

func NewNoOpHeader(context.Context, interface{}) (interface{}, error) {  // 定义 NewNoOpHeader 函数，接收 context 和 interface 参数
    return NoOpHeader{}, nil  // 返回 NoOpHeader 结构体实例和 nil 错误
}

type NoOpConnectionHeader struct{}  // 定义 NoOpConnectionHeader 结构体

func (NoOpConnectionHeader) Client(conn net.Conn) net.Conn {  // 实现 Client 方法，接收 net.Conn 参数，返回 net.Conn
    return conn  // 返回传入的连接
}

func (NoOpConnectionHeader) Server(conn net.Conn) net.Conn {  // 实现 Server 方法，接收 net.Conn 参数，返回 net.Conn
    return conn  // 返回传入的连接
}

func NewNoOpConnectionHeader(context.Context, interface{}) (interface{}, error) {  // 定义 NewNoOpConnectionHeader 函数，接收 context 和 interface 参数
    return NoOpConnectionHeader{}, nil  // 返回 NoOpConnectionHeader 结构体实例和 nil 错误
}

func init() {  // 初始化函数
    common.Must(common.RegisterConfig((*Config)(nil), NewNoOpHeader))  // 注册 Config 类型和 NewNoOpHeader 函数
    common.Must(common.RegisterConfig((*ConnectionConfig)(nil), NewNoOpConnectionHeader))  // 注册 ConnectionConfig 类型和 NewNoOpConnectionHeader 函数
}
```