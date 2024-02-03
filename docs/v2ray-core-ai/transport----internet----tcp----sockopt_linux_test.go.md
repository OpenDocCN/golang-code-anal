# `v2ray-core\transport\internet\tcp\sockopt_linux_test.go`

```go
// +build linux
// 声明该文件只在 Linux 系统下编译

package tcp_test
// 声明包名为 tcp_test

import (
    "context"
    "strings"
    "testing"

    "v2ray.com/core/common"
    "v2ray.com/core/testing/servers/tcp"
    "v2ray.com/core/transport/internet"
    . "v2ray.com/core/transport/internet/tcp"
)
// 导入所需的包

func TestGetOriginalDestination(t *testing.T) {
    // 定义测试函数 TestGetOriginalDestination
    tcpServer := tcp.Server{}
    // 创建一个 TCP 服务器对象
    dest, err := tcpServer.Start()
    // 启动 TCP 服务器并获取目标地址和错误信息
    common.Must(err)
    // 如果有错误则抛出异常
    defer tcpServer.Close()
    // 延迟关闭 TCP 服务器

    config, err := internet.ToMemoryStreamConfig(nil)
    // 将配置转换为内存流配置
    common.Must(err)
    // 如果有错误则抛出异常
    conn, err := Dial(context.Background(), dest, config)
    // 使用目标地址和配置建立连接
    common.Must(err)
    // 如果有错误则抛出异常
    defer conn.Close()
    // 延迟关闭连接

    originalDest, err := GetOriginalDestination(conn)
    // 获取连接的原始目标地址和错误信息
    if !(dest == originalDest || strings.Contains(err.Error(), "failed to call getsockopt")) {
        // 如果目标地址不等于原始目标地址，或者错误信息中包含特定字符串
        t.Error("unexpected state")
        // 抛出测试错误
    }
}
```