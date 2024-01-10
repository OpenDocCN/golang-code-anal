# `v2ray-core\transport\internet\sockopt_linux_test.go`

```
package internet_test

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "syscall"  // 提供了操作系统底层接口的包
    "testing"  // 测试框架包

    "v2ray.com/core/common"  // V2Ray 核心通用包
    "v2ray.com/core/common/net"  // V2Ray 核心网络通用包
    "v2ray.com/core/testing/servers/tcp"  // V2Ray 核心测试服务器包
    . "v2ray.com/core/transport/internet"  // V2Ray 核心网络传输包
)

func TestSockOptMark(t *testing.T) {
    t.Skip("requires CAP_NET_ADMIN")  // 跳过测试，需要 CAP_NET_ADMIN 权限

    tcpServer := tcp.Server{  // 创建一个 TCP 服务器
        MsgProcessor: func(b []byte) []byte {  // 消息处理函数，返回原始消息
            return b
        },
    }
    dest, err := tcpServer.Start()  // 启动 TCP 服务器
    common.Must(err)  // 如果有错误，立即终止程序
    defer tcpServer.Close()  // 延迟关闭 TCP 服务器

    const mark = 1  // 定义常量 mark 为 1
    dialer := DefaultSystemDialer{}  // 创建默认系统拨号器
    conn, err := dialer.Dial(context.Background(), nil, dest, &SocketConfig{Mark: mark})  // 使用拨号器建立连接
    common.Must(err)  // 如果有错误，立即终止程序
    defer conn.Close()  // 延迟关闭连接

    rawConn, err := conn.(*net.TCPConn).SyscallConn()  // 获取原始连接
    common.Must(err)  // 如果有错误，立即终止程序
    err = rawConn.Control(func(fd uintptr) {  // 控制原始连接
        m, err := syscall.GetsockoptInt(int(fd), syscall.SOL_SOCKET, syscall.SO_MARK)  // 获取连接的标记
        common.Must(err)  // 如果有错误，立即终止程序
        if mark != m {  // 如果连接标记不等于预期标记
            t.Fatal("unexpected connection mark", m, " want ", mark)  // 输出错误信息并终止测试
        }
    })
    common.Must(err)  // 如果有错误，立即终止程序
}
```