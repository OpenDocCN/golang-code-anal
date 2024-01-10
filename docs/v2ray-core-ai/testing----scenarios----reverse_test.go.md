# `v2ray-core\testing\scenarios\reverse_test.go`

```
package scenarios

import (
    "testing"
    "time"

    "golang.org/x/sync/errgroup"

    "v2ray.com/core"
    "v2ray.com/core/app/log"
    "v2ray.com/core/app/policy"
    "v2ray.com/core/app/proxyman"
    "v2ray.com/core/app/reverse"
    "v2ray.com/core/app/router"
    "v2ray.com/core/common"
    clog "v2ray.com/core/common/log"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/serial"
    "v2ray.com/core/common/uuid"
    "v2ray.com/core/proxy/blackhole"
    "v2ray.com/core/proxy/dokodemo"
    "v2ray.com/core/proxy/freedom"
    "v2ray.com/core/proxy/vmess"
    "v2ray.com/core/proxy/vmess/inbound"
    "v2ray.com/core/proxy/vmess/outbound"
    "v2ray.com/core/testing/servers/tcp"
)

func TestReverseProxy(t *testing.T) {
    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)

    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个外部端口
    externalPort := tcp.PickPort()
    // 选择一个反向代理端口
    reversePort := tcp.PickPort()

    // 选择一个客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置，获取服务器和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)

    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    // 创建一个错误组
    var errg errgroup.Group
    // 循环 32 次，每次执行测试 TCP 连接
    for i := 0; i < 32; i++ {
        errg.Go(testTCPConn(externalPort, 10240*1024, time.Second*40))
    }

    // 等待错误组中的所有操作完成，获取可能的错误
    if err := errg.Wait(); err != nil {
        // 如果有错误发生，则测试失败
        t.Fatal(err)
    }
}

func TestReverseProxyLongRunning(t *testing.T) {
    // 创建一个 TCP 服务器对象，使用 xor 作为消息处理器
    tcpServer := tcp.Server{
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器，获取目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)

    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 生成一个新的用户 ID
    userID := protocol.NewID(uuid.New())
    // 选择一个外部端口
    externalPort := tcp.PickPort()
    // 选择一个反向代理端口
    reversePort := tcp.PickPort()

    // 选择一个客户端端口
    clientPort := tcp.PickPort()

    // 初始化服务器配置，获取服务器和可能的错误
    servers, err := InitializeServerConfigs(serverConfig, clientConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)

    // 延迟关闭所有服务器
    defer CloseAllServers(servers)
}
    # 循环执行4096次
    for i := 0; i < 4096; i++ {
        # 调用testTCPConn函数，传入externalPort, 1024, time.Second*20作为参数，如果返回错误则输出错误信息
        if err := testTCPConn(externalPort, 1024, time.Second*20)(); err != nil {
            t.Error(err)
        }
    }
# 闭合前面的函数定义
```