# `v2ray-core\testing\servers\tcp\port.go`

```go
// 导入所需的包
package tcp

import (
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
)

// PickPort函数返回系统中未使用的TCP端口。返回的端口高度可能未被使用，但不保证。
func PickPort() net.Port {
    // 监听TCP4协议的127.0.0.1地址和一个随机端口
    listener, err := net.Listen("tcp4", "127.0.0.1:0")
    // 如果出现错误，程序终止
    common.Must(err)
    // 延迟关闭监听器
    defer listener.Close()

    // 获取监听器的地址信息，并转换为TCP地址
    addr := listener.Addr().(*net.TCPAddr)
    // 返回TCP地址的端口号
    return net.Port(addr.Port)
}
```