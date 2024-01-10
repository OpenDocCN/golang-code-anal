# `trojan-go\tunnel\shadowsocks\shadowsocks_test.go`

```
package shadowsocks

import (
    "context"  // 上下文包，用于传递参数、取消操作等
    "fmt"  // 格式化包，用于格式化输出
    "net"  // 网络包，提供了基本的网络功能
    "strconv"  // 字符串转换包，用于字符串和基本数据类型之间的转换
    "strings"  // 字符串包，提供了操作字符串的函数
    "sync"  // 同步包，提供了并发安全的锁和条件变量等

    "github.com/p4gefau1t/trojan-go/common"  // 引入自定义的 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 引入自定义的 config 包
    "github.com/p4gefau1t/trojan-go/test/util"  // 引入自定义的测试工具包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 引入自定义的自由隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 引入自定义的传输包
)

func TestShadowsocks(t *testing.T) {
    p, err := strconv.ParseInt(util.HTTPPort, 10, 32)  // 将 util.HTTPPort 转换为 int64 类型
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的端口
    transportConfig := &transport.Config{  // 创建传输配置对象
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  port,  // 本地端口号
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: port,  // 远程端口号
    }
    ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)  // 使用传输配置创建上下文
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})  // 使用自由隧道配置更新上下文
    tcpClient, err := transport.NewClient(ctx, nil)  // 创建传输客户端
    common.Must(err)  // 如果 err 不为 nil，则触发 panic
    tcpServer, err := transport.NewServer(ctx, nil)  // 创建传输服务器
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    cfg := &Config{  // 创建配置对象
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: int(p),  // 远程端口号
        Shadowsocks: ShadowsocksConfig{  // 创建 Shadowsocks 配置对象
            Enabled:  true,  // 启用 Shadowsocks
            Method:   "AES-128-GCM",  // 加密方法
            Password: "password",  // 密码
        },
    }
    ctx = config.WithConfig(ctx, Name, cfg)  // 使用配置更新上下文

    c, err := NewClient(ctx, tcpClient)  // 创建 Shadowsocks 客户端
    common.Must(err)  // 如果 err 不为 nil，则触发 panic
    s, err := NewServer(ctx, tcpServer)  // 创建 Shadowsocks 服务器
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    wg := sync.WaitGroup{}  // 创建同步等待组
    wg.Add(2)  // 添加两个等待
    var conn1, conn2 net.Conn  // 定义两个网络连接
    go func() {  // 启动一个 goroutine
        var err error
        conn1, err = c.DialConn(nil, nil)  // 客户端发起连接
        common.Must(err)  // 如果 err 不为 nil，则触发 panic
        conn1.Write(util.GeneratePayload(1024))  // 向连接写入数据
        wg.Done()  // 完成一个等待
    }()
    go func() {  // 启动一个 goroutine
        var err error
        conn2, err = s.AcceptConn(nil)  // 服务器接受连接
        common.Must(err)  // 如果 err 不为 nil，则触发 panic
        buf := [1024]byte{}  // 创建一个长度为 1024 的字节数组
        conn2.Read(buf[:])  // 从连接读取数据
        wg.Done()  // 完成一个等待
    }()
    wg.Wait()  // 等待所有 goroutine 完成
    if !util.CheckConn(conn1, conn2) {  // 检查连接是否正常
        t.Fail()  // 测试失败
    }
}
    // 创建一个匿名的 goroutine
    go func() {
        // 声明一个错误变量
        var err error
        // 接受一个连接，将结果赋给 conn2，并将错误赋给 err
        conn2, err = s.AcceptConn(nil)
        // 如果没有错误，则测试失败
        if err == nil {
            t.Fail()
        }
    }()

    // 测试重定向
    // 使用 tcpClient 建立一个连接，将结果赋给 conn3，并将错误赋给 err
    conn3, err := tcpClient.DialConn(nil, nil)
    // 必须处理错误
    common.Must(err)
    // 向 conn3 写入一个 1024 字节的数据包，将写入的字节数和错误赋给 n 和 err
    n, err := conn3.Write(util.GeneratePayload(1024))
    // 必须处理错误
    common.Must(err)
    // 打印写入的字节数
    fmt.Println("write:", n)
    // 声明一个长度为 1024 的字节数组 buf
    buf := [1024]byte{}
    // 从 conn3 读取数据到 buf 中，将读取的字节数和错误赋给 n 和 err
    n, err = conn3.Read(buf[:])
    // 必须处理错误
    common.Must(err)
    // 打印读取的字节数
    fmt.Println("read:", n)
    // 如果 buf 中不包含 "Bad Request" 字符串，则测试失败
    if !strings.Contains(string(buf[:n]), "Bad Request") {
        t.Fail()
    }
    // 关闭 conn1
    conn1.Close()
    // 关闭 conn3
    conn3.Close()
    // 关闭 c
    c.Close()
    // 关闭 s
    s.Close()
# 闭合前面的函数定义
```