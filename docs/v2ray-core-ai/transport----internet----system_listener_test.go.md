# `v2ray-core\transport\internet\system_listener_test.go`

```go
package internet_test

import (
    "context"  // 导入上下文包
    "net"  // 导入网络包
    "testing"  // 导入测试包

    "v2ray.com/core/common"  // 导入v2ray的common包
    "v2ray.com/core/transport/internet"  // 导入v2ray的internet包
)

func TestRegisterListenerController(t *testing.T) {
    var gotFd uintptr  // 声明一个uintptr类型的变量gotFd

    common.Must(internet.RegisterListenerController(func(network string, addr string, fd uintptr) error {  // 调用RegisterListenerController注册监听器控制器
        gotFd = fd  // 将fd赋值给gotFd
        return nil  // 返回空值
    }))

    conn, err := internet.ListenSystemPacket(context.Background(), &net.UDPAddr{  // 监听系统数据包
        IP: net.IPv4zero,  // 使用IPv4的0地址
    }, nil)
    common.Must(err)  // 检查错误
    common.Must(conn.Close())  // 关闭连接

    if gotFd == 0 {  // 如果gotFd为0
        t.Error("expected none-zero fd, but actually 0")  // 输出错误信息
    }
}
```