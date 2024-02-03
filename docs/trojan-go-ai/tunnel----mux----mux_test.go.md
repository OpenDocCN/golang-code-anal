# `trojan-go\tunnel\mux\mux_test.go`

```go
package mux

import (
    "context"  // 导入上下文包
    "testing"  // 导入测试包

    "github.com/p4gefau1t/trojan-go/common"  // 导入公共包
    "github.com/p4gefau1t/trojan-go/config"  // 导入配置包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入测试工具包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入自由隧道包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入传输包
)

func TestMux(t *testing.T) {
    muxCfg := &Config{  // 创建配置对象
        Mux: MuxConfig{  // 设置多路复用配置
            Enabled:     true,  // 启用多路复用
            Concurrency: 8,  // 设置并发数为8
            IdleTimeout: 60,  // 设置空闲超时时间为60秒
        },
    }
    ctx := config.WithConfig(context.Background(), Name, muxCfg)  // 使用配置对象创建上下文

    port := common.PickPort("tcp", "127.0.0.1")  // 选择可用端口
    transportConfig := &transport.Config{  // 创建传输配置对象
        LocalHost:  "127.0.0.1",  // 设置本地主机
        LocalPort:  port,  // 设置本地端口
        RemoteHost: "127.0.0.1",  // 设置远程主机
        RemotePort: port,  // 设置远程端口
    }
    ctx = config.WithConfig(ctx, transport.Name, transportConfig)  // 使用传输配置对象更新上下文
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})  // 使用自由隧道配置对象更新上下文

    tcpClient, err := transport.NewClient(ctx, nil)  // 创建传输客户端
    common.Must(err)  // 检查错误
    tcpServer, err := transport.NewServer(ctx, nil)  // 创建传输服务器
    common.Must(err)  // 检查错误

    common.Must(err)  // 检查错误

    muxTunnel := Tunnel{}  // 创建多路复用隧道对象
    muxClient, _ := muxTunnel.NewClient(ctx, tcpClient)  // 创建多路复用客户端
    muxServer, _ := muxTunnel.NewServer(ctx, tcpServer)  // 创建多路复用服务器

    conn1, err := muxClient.DialConn(nil, nil)  // 通过多路复用客户端建立连接
    common.Must2(conn1.Write(util.GeneratePayload(1024)))  // 写入数据
    common.Must(err)  // 检查错误
    buf := [1024]byte{}  // 创建缓冲区
    conn2, err := muxServer.AcceptConn(nil)  // 接受连接
    common.Must(err)  // 检查错误
    common.Must2(conn2.Read(buf[:]))  // 读取数据
    if !util.CheckConn(conn1, conn2) {  // 检查连接是否正常
        t.Fail()  // 测试失败
    }
    conn1.Close()  // 关闭连接
    conn2.Close()  // 关闭连接
    muxClient.Close()  // 关闭多路复用客户端
    muxServer.Close()  // 关闭多路复用服务器
}
```