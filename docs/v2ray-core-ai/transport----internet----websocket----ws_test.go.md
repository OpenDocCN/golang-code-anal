# `v2ray-core\transport\internet\websocket\ws_test.go`

```go
package websocket_test

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "runtime"  // 导入运行时包，用于访问运行时的信息
    "testing"  // 导入测试包，用于编写和运行测试函数
    "time"     // 导入时间包，用于处理时间相关的操作

    "v2ray.com/core/common"  // 导入通用包，包含一些常用的函数和结构体
    "v2ray.com/core/common/net"  // 导入网络包，用于处理网络相关的操作
    "v2ray.com/core/common/protocol/tls/cert"  // 导入 TLS 证书包，用于处理 TLS 相关的操作
    "v2ray.com/core/transport/internet"  // 导入网络传输包，用于处理网络传输相关的操作
    "v2ray.com/core/transport/internet/tls"  // 导入 TLS 包，用于处理 TLS 相关的操作
    . "v2ray.com/core/transport/internet/websocket"  // 导入 WebSocket 包，用于处理 WebSocket 相关的操作
)

func Test_listenWSAndDial(t *testing.T) {
    listen, err := ListenWS(context.Background(), net.LocalHostIP, 13146, &internet.MemoryStreamConfig{
        ProtocolName: "websocket",  // 设置协议名称为 WebSocket
        ProtocolSettings: &Config{  // 设置协议配置
            Path: "ws",  // 设置路径为 "ws"
        },
    }, func(conn internet.Connection) {  // 定义回调函数，处理连接
        go func(c internet.Connection) {  // 启动一个新的 goroutine 处理连接
            defer c.Close()  // 在函数返回时关闭连接

            var b [1024]byte  // 定义一个长度为 1024 的字节数组
            _, err := c.Read(b[:])  // 从连接中读取数据到字节数组中
            if err != nil {  // 如果读取出错
                return  // 退出函数
            }

            common.Must2(c.Write([]byte("Response")))  // 向连接中写入响应数据
        }(conn)  // 调用回调函数处理连接
    })
    common.Must(err)  // 检查错误并处理

    ctx := context.Background()  // 创建一个新的上下文
    streamSettings := &internet.MemoryStreamConfig{  // 设置流配置
        ProtocolName:     "websocket",  // 设置协议名称为 WebSocket
        ProtocolSettings: &Config{Path: "ws"},  // 设置协议配置的路径为 "ws"
    }
    conn, err := Dial(ctx, net.TCPDestination(net.DomainAddress("localhost"), 13146), streamSettings)  // 拨号连接到指定地址和端口
    common.Must(err)  // 检查错误并处理
    _, err = conn.Write([]byte("Test connection 1"))  // 向连接中写入数据
    common.Must(err)  // 检查错误并处理

    var b [1024]byte  // 定义一个长度为 1024 的字节数组
    n, err := conn.Read(b[:])  // 从连接中读取数据到字节数组中
    common.Must(err)  // 检查错误并处理
    if string(b[:n]) != "Response" {  // 如果读取的响应数据不符合预期
        t.Error("response: ", string(b[:n]))  // 输出错误信息
    }

    common.Must(conn.Close())  // 关闭连接
    <-time.After(time.Second * 5)  // 等待 5 秒钟
    conn, err = Dial(ctx, net.TCPDestination(net.DomainAddress("localhost"), 13146), streamSettings)  // 再次拨号连接到指定地址和端口
    common.Must(err)  // 检查错误并处理
    _, err = conn.Write([]byte("Test connection 2"))  // 向连接中写入数据
    common.Must(err)  // 检查错误并处理
    n, err = conn.Read(b[:])  // 从连接中读取数据到字节数组中
    common.Must(err)  // 检查错误并处理
    if string(b[:n]) != "Response" {  // 如果读取的响应数据不符合预期
        t.Error("response: ", string(b[:n]))  // 输出错误信息
    }
    common.Must(conn.Close())  // 关闭连接

    common.Must(listen.Close())  // 关闭监听
}

func TestDialWithRemoteAddr(t *testing.T) {
    // 在本地IP地址和端口13148上监听WebSocket连接，使用内存流配置
    listen, err := ListenWS(context.Background(), net.LocalHostIP, 13148, &internet.MemoryStreamConfig{
        ProtocolName: "websocket",
        ProtocolSettings: &Config{
            Path: "ws",
        },
    }, func(conn internet.Connection) {
        // 在新连接上启动一个goroutine
        go func(c internet.Connection) {
            defer c.Close()

            // 读取连接上的数据到字节数组中
            var b [1024]byte
            _, err := c.Read(b[:])
            // 如果发生错误，则返回
            if err != nil {
                return
            }

            // 向连接写入响应数据
            _, err = c.Write([]byte("Response"))
            common.Must(err)
        }(conn)
    })
    // 如果发生错误，则终止程序
    common.Must(err)

    // 连接到本地主机的13148端口，使用内存流配置
    conn, err := Dial(context.Background(), net.TCPDestination(net.DomainAddress("localhost"), 13148), &internet.MemoryStreamConfig{
        ProtocolName:     "websocket",
        ProtocolSettings: &Config{Path: "ws", Header: []*Header{{Key: "X-Forwarded-For", Value: "1.1.1.1"}}},
    })

    // 如果发生错误，则终止程序
    common.Must(err)
    // 向连接写入测试连接数据
    _, err = conn.Write([]byte("Test connection 1"))
    // 如果发生错误，则终止程序
    common.Must(err)

    // 从连接读取数据到字节数组中
    var b [1024]byte
    n, err := conn.Read(b[:])
    // 如果发生错误，则终止程序
    common.Must(err)
    // 如果读取的数据不是"Response"，则输出错误信息
    if string(b[:n]) != "Response" {
        t.Error("response: ", string(b[:n]))
    }

    // 关闭监听
    common.Must(listen.Close())
func Test_listenWSAndDial_TLS(t *testing.T) {
    // 如果运行时架构为arm64，则返回，不执行后续代码
    if runtime.GOARCH == "arm64" {
        return
    }

    // 记录开始时间
    start := time.Now()

    // 设置内存流配置
    streamSettings := &internet.MemoryStreamConfig{
        ProtocolName: "websocket", // 设置协议名称为websocket
        ProtocolSettings: &Config{ // 设置协议配置
            Path: "wss", // 设置路径为wss
        },
        SecurityType: "tls", // 设置安全类型为tls
        SecuritySettings: &tls.Config{ // 设置安全配置
            AllowInsecure: true, // 允许不安全连接
            Certificate:   []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil, cert.CommonName("localhost")))}, // 设置证书
        },
    }
    // 监听本地IP的13143端口，使用streamSettings配置，处理连接的回调函数
    listen, err := ListenWS(context.Background(), net.LocalHostIP, 13143, streamSettings, func(conn internet.Connection) {
        go func() {
            _ = conn.Close() // 关闭连接
        }()
    })
    common.Must(err) // 检查错误
    defer listen.Close() // 延迟关闭监听

    // 拨号连接到本地的13143端口，使用streamSettings配置
    conn, err := Dial(context.Background(), net.TCPDestination(net.DomainAddress("localhost"), 13143), streamSettings)
    common.Must(err) // 检查错误
    _ = conn.Close() // 关闭连接

    // 记录结束时间
    end := time.Now()
    // 如果结束时间在开始时间之后5秒以上，则输出错误信息
    if !end.Before(start.Add(time.Second * 5)) {
        t.Error("end: ", end, " start: ", start)
    }
}
```