# `trojan-go\tunnel\websocket\websocket_test.go`

```
package websocket

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "net"  // 导入网络包
    "strings"  // 导入字符串包
    "sync"  // 导入同步包
    "testing"  // 导入测试包
    "time"  // 导入时间包

    "golang.org/x/net/websocket"  // 导入 websocket 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 公共包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 trojan-go 配置包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入 trojan-go 测试工具包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入 trojan-go 自由包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入 trojan-go 传输包
)

func TestWebsocket(t *testing.T) {
    cfg := &Config{  // 创建 Config 结构体指针
        Websocket: WebsocketConfig{  // 设置 WebsocketConfig 结构体
            Enabled: true,  // 启用 WebSocket
            Host:    "localhost",  // 设置主机为本地主机
            Path:    "/ws",  // 设置路径为 /ws
        },
    }

    ctx := config.WithConfig(context.Background(), Name, cfg)  // 使用配置创建上下文

    port := common.PickPort("tcp", "127.0.0.1")  // 选择可用端口
    transportConfig := &transport.Config{  // 创建传输配置结构体指针
        LocalHost:  "127.0.0.1",  // 设置本地主机
        LocalPort:  port,  // 设置本地端口
        RemoteHost: "127.0.0.1",  // 设置远程主机
        RemotePort: port,  // 设置远程端口
    }
    freedomCfg := &freedom.Config{}  // 创建自由配置结构体指针
    ctx = config.WithConfig(ctx, transport.Name, transportConfig)  // 使用传输配置更新上下文
    ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)  // 使用自由配置更新上下文
    tcpClient, err := transport.NewClient(ctx, nil)  // 创建传输客户端
    common.Must(err)  // 检查错误
    tcpServer, err := transport.NewServer(ctx, nil)  // 创建传输服务器
    common.Must(err)  // 检查错误

    c, err := NewClient(ctx, tcpClient)  // 创建 WebSocket 客户端
    common.Must(err)  // 检查错误
    s, err := NewServer(ctx, tcpServer)  // 创建 WebSocket 服务器
    var conn2 tunnel.Conn  // 定义隧道连接变量
    wg := sync.WaitGroup{}  // 创建同步等待组
    wg.Add(1)  // 添加一个等待
    go func() {  // 启动协程
        conn2, err = s.AcceptConn(nil)  // 接受连接
        common.Must(err)  // 检查错误
        wg.Done()  // 完成等待
    }()
    time.Sleep(time.Second)  // 休眠一秒
    conn1, err := c.DialConn(nil, nil)  // 拨号连接
    common.Must(err)  // 检查错误
    wg.Wait()  // 等待协程完成
    if !util.CheckConn(conn1, conn2) {  // 检查连接
        t.Fail()  // 测试失败
    }

    if strings.HasPrefix(conn1.RemoteAddr().String(), "ws") {  // 检查连接地址前缀
        t.Fail()  // 测试失败
    }
    if strings.HasPrefix(conn2.RemoteAddr().String(), "ws") {  // 检查连接地址前缀
        t.Fail()  // 测试失败
    }

    conn1.Close()  // 关闭连接1
    conn2.Close()  // 关闭连接2
    s.Close()  // 关闭服务器
    c.Close()  // 关闭客户端
}

func TestRedirect(t *testing.T) {
    // 创建一个配置结构体指针，设置远程主机地址和 WebSocket 配置
    cfg := &Config{
        RemoteHost: "127.0.0.1",
        Websocket: WebsocketConfig{
            Enabled: true,
            Host:    "localhost",
            Path:    "/ws",
        },
    }
    // 从 util.HTTPPort 中读取端口号并设置到 cfg.RemotePort 中
    fmt.Sscanf(util.HTTPPort, "%d", &cfg.RemotePort)
    // 使用配置信息创建一个上下文
    ctx := config.WithConfig(context.Background(), Name, cfg)

    // 选择一个可用的端口并设置到 transportConfig 中
    port := common.PickPort("tcp", "127.0.0.1")
    transportConfig := &transport.Config{
        LocalHost: "127.0.0.1",
        LocalPort: port,
    }
    // 将 transportConfig 设置到上下文中
    ctx = config.WithConfig(ctx, transport.Name, transportConfig)
    // 使用上下文创建一个 TCP 服务器
    tcpServer, err := transport.NewServer(ctx, nil)
    common.Must(err)

    // 创建一个新的服务器实例
    s, err := NewServer(ctx, tcpServer)
    common.Must(err)

    // 启动一个 goroutine，尝试接受连接，如果成功则调用 t.Fail()
    go func() {
        _, err := s.AcceptConn(nil)
        if err == nil {
            t.Fail()
        }
    }()
    // 等待一秒钟
    time.Sleep(time.Second)
    // 使用 TCP 连接到指定地址和端口
    conn, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))
    common.Must(err)
    // 设置 WebSocket 的 URL 和来源
    url := "wss://localhost/wrong-path"
    origin := "https://localhost"
    wsConfig, err := websocket.NewConfig(url, origin)
    common.Must(err)
    // 使用给定的连接创建一个新的 WebSocket 客户端
    _, err = websocket.NewClient(wsConfig, conn)
    // 如果成功创建 WebSocket 客户端，则调用 t.Fail()
    if err == nil {
        t.Fail()
    }
    // 关闭连接
    conn.Close()

    // 关闭服务器
    s.Close()
# 闭合前面的函数定义
```