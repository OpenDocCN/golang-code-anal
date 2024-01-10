# `v2ray-core\transport\internet\dialer_test.go`

```
// 定义 internet_test 包
package internet_test

// 导入所需的包
import (
    "context"
    "testing"

    "github.com/google/go-cmp/cmp"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/testing/servers/tcp"
    . "v2ray.com/core/transport/internet"
)

// 测试函数，测试 DialWithLocalAddr 方法
func TestDialWithLocalAddr(t *testing.T) {
    // 创建一个 TCP 服务器
    server := &tcp.Server{}
    // 启动服务器，获取目标地址和错误信息
    dest, err := server.Start()
    // 必须处理错误
    common.Must(err)
    // 延迟关闭服务器
    defer server.Close()

    // 使用系统默认设置拨号，获取连接和错误信息
    conn, err := DialSystem(context.Background(), net.TCPDestination(net.LocalHostIP, dest.Port), nil)
    // 必须处理错误
    common.Must(err)
    // 比较连接的远程地址是否为 "127.0.0.1:端口号"，如果不是则输出差异信息
    if r := cmp.Diff(conn.RemoteAddr().String(), "127.0.0.1:"+dest.Port.String()); r != "" {
        t.Error(r)
    }
    // 关闭连接
    conn.Close()
}
```