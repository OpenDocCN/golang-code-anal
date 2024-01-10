# `trojan-go\tunnel\transport\transport_test.go`

```
package transport

import (
    "context"  // 导入上下文包
    "net"  // 导入网络包
    "sync"  // 导入同步包
    "testing"  // 导入测试包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自定义包
)

func TestTransport(t *testing.T) {
    serverCfg := &Config{  // 定义服务器配置
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  common.PickPort("tcp", "127.0.0.1"),  // 选择本地端口
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: common.PickPort("tcp", "127.0.0.1"),  // 选择远程端口
    }
    clientCfg := &Config{  // 定义客户端配置
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  common.PickPort("tcp", "127.0.0.1"),  // 选择本地端口
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: serverCfg.LocalPort,  // 使用服务器配置的本地端口
    }
    freedomCfg := &freedom.Config{}  // 自由配置
    sctx := config.WithConfig(context.Background(), Name, serverCfg)  // 使用服务器配置创建上下文
    cctx := config.WithConfig(context.Background(), Name, clientCfg)  // 使用客户端配置创建上下文
    cctx = config.WithConfig(cctx, freedom.Name, freedomCfg)  // 在客户端配置上下文中添加自由配置

    s, err := NewServer(sctx, nil)  // 创建服务器
    common.Must(err)  // 检查错误
    c, err := NewClient(cctx, nil)  // 创建客户端
    common.Must(err)  // 检查错误

    wg := sync.WaitGroup{}  // 创建同步等待组
    wg.Add(1)  // 添加一个等待
    var conn1, conn2 net.Conn  // 定义两个网络连接
    go func() {  // 启动一个 goroutine
        conn2, err = s.AcceptConn(nil)  // 接受服务器连接
        common.Must(err)  // 检查错误
        wg.Done()  // 完成一个等待
    }()
    conn1, err = c.DialConn(nil, nil)  // 客户端拨号连接
    common.Must(err)  // 检查错误

    common.Must2(conn1.Write([]byte("12345678\r\n")))  // 写入数据到连接1
    wg.Wait()  // 等待所有 goroutine 完成
    buf := [10]byte{}  // 创建一个长度为10的字节数组
    conn2.Read(buf[:])  // 从连接2读取数据到缓冲区
    if !util.CheckConn(conn1, conn2) {  // 检查连接1和连接2是否一致
        t.Fail()  // 测试失败
    }
    s.Close()  // 关闭服务器
    c.Close()  // 关闭客户端
}

func TestClientPlugin(t *testing.T) {
    clientCfg := &Config{  // 定义客户端配置
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  common.PickPort("tcp", "127.0.0.1"),  // 选择本地端口
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: 12345,  // 远程端口
        TransportPlugin: TransportPluginConfig{  // 传输插件配置
            Enabled: true,  // 启用
            Type:    "shadowsocks",  // 类型为shadowsocks
            Command: "echo $SS_REMOTE_PORT",  // 命令
            Option:  "",  // 选项
            Arg:     nil,  // 参数
            Env:     nil,  // 环境变量
        },
    }
    # 使用config.WithConfig将clientCfg配置添加到上下文中，并返回新的上下文
    ctx := config.WithConfig(context.Background(), Name, clientCfg)
    # 创建一个freedom.Config类型的指针变量freedomCfg
    freedomCfg := &freedom.Config{}
    # 使用config.WithConfig将freedomCfg配置添加到上下文中，并返回新的上下文
    ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)
    # 使用NewClient函数创建一个客户端c，并将ctx作为参数传入
    c, err := NewClient(ctx, nil)
    # 检查错误，如果有错误则抛出panic
    common.Must(err)
    # 关闭客户端c
    c.Close()
func TestServerPlugin(t *testing.T) {
    // 创建一个配置对象，设置本地主机和端口，远程主机和端口，以及传输插件的配置
    cfg := &Config{
        LocalHost:  "127.0.0.1",
        LocalPort:  common.PickPort("tcp", "127.0.0.1"),
        RemoteHost: "127.0.0.1",
        RemotePort: 12345,
        TransportPlugin: TransportPluginConfig{
            Enabled: true,
            Type:    "shadowsocks",
            Command: "echo $SS_REMOTE_PORT",
            Option:  "",
            Arg:     nil,
            Env:     nil,
        },
    }
    // 在上下文中设置配置对象
    ctx := config.WithConfig(context.Background(), Name, cfg)
    // 创建一个自由配置对象
    freedomCfg := &freedom.Config{}
    // 在上下文中设置自由配置对象
    ctx = config.WithConfig(ctx, freedom.Name, freedomCfg)
    // 创建一个新的服务器对象
    s, err := NewServer(ctx, nil)
    // 必须处理错误
    common.Must(err)
    // 关闭服务器
    s.Close()
}
```