# `trojan-go\tunnel\trojan\trojan_test.go`

```
package trojan

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于管理请求的上下文
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io"  // 导入 io 包，提供了基本的接口和函数
    "net"  // 导入 net 包，提供了基本的网络功能
    "testing"  // 导入 testing 包，用于编写测试函数

    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go/common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 trojan-go/config 包
    "github.com/p4gefau1t/trojan-go/statistic/memory"  // 导入 trojan-go/statistic/memory 包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入 trojan-go/test/util 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go/tunnel 包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入 trojan-go/tunnel/freedom 包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入 trojan-go/tunnel/transport 包
)

func TestTrojan(t *testing.T) {
    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的端口
    transportConfig := &transport.Config{  // 创建 transport 配置对象
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  port,  // 本地端口号
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: port,  // 远程端口号
    }
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    ctx = config.WithConfig(ctx, transport.Name, transportConfig)  // 将 transport 配置添加到上下文中
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})  // 将 freedom 配置添加到上下文中
    tcpClient, err := transport.NewClient(ctx, nil)  // 创建一个新的 transport 客户端
    common.Must(err)  // 检查错误
    tcpServer, err := transport.NewServer(ctx, nil)  // 创建一个新的 transport 服务器
    common.Must(err)  // 检查错误

    serverPort := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的服务器端口
    authConfig := &memory.Config{Passwords: []string{"password"}}  // 创建 memory 配置对象
    clientConfig := &Config{  // 创建客户端配置对象
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: serverPort,  // 远程端口号
    }
    serverConfig := &Config{  // 创建服务器配置对象
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  serverPort,  // 本地端口号
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: util.EchoPort,  // 远程端口号
    }

    ctx = config.WithConfig(ctx, memory.Name, authConfig)  // 将 memory 配置添加到上下文中
    clientCtx := config.WithConfig(ctx, Name, clientConfig)  // 将客户端配置添加到上下文中
    serverCtx := config.WithConfig(ctx, Name, serverConfig)  // 将服务器配置添加到上下文中
    c, err := NewClient(clientCtx, tcpClient)  // 创建一个新的客户端
    common.Must(err)  // 检查错误
    s, err := NewServer(serverCtx, tcpServer)  // 创建一个新的服务器
    common.Must(err)  // 检查错误
    conn1, err := c.DialConn(&tunnel.Address{  // 通过客户端连接到指定地址
        DomainName:  "example.com",  // 域名
        AddressType: tunnel.DomainName,  // 地址类型
    }, nil)
    common.Must(err)  // 检查错误
    common.Must2(conn1.Write([]byte("87654321")))  // 写入数据到连接
    conn2, err := s.AcceptConn(nil)  // 接受连接
    common.Must(err)  // 检查错误
}
    // 创建一个长度为8的字节切片
    buf := [8]byte{}
    // 从conn2中读取8个字节的数据到buf中
    conn2.Read(buf[:])
    // 检查conn1和conn2之间的连接状态，如果不通则测试失败
    if !util.CheckConn(conn1, conn2) {
        t.Fail()
    }

    // 通过c对象拨号创建一个数据包
    packet1, err := c.DialPacket(nil)
    common.Must(err)
    // 向packet1写入数据和元数据
    packet1.WriteWithMetadata([]byte("12345678"), &tunnel.Metadata{
        Address: &tunnel.Address{
            DomainName:  "example.com",
            AddressType: tunnel.DomainName,
            Port:        80,
        },
    })
    // 通过s对象接受一个数据包
    packet2, err := s.AcceptPacket(nil)
    common.Must(err)

    // 从packet2中读取数据和元数据到buf中
    _, m, err := packet2.ReadWithMetadata(buf[:])
    common.Must(err)

    // 打印元数据
    fmt.Println(m)

    // 检查packet1和packet2之间的连接状态，如果不通则测试失败
    if !util.CheckPacketOverConn(packet1, packet2) {
        t.Fail()
    }

    // 重定向连接到指定的TCP地址
    conn, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))
    common.Must(err)
    // 生成1024字节的有效载荷并发送到连接中
    sendBuf := util.GeneratePayload(1024)
    recvBuf := [1024]byte{}
    common.Must2(conn.Write(sendBuf))
    // 从连接中读取1024字节的数据到recvBuf中
    common.Must2(io.ReadFull(conn, recvBuf[:]))
    // 检查发送的数据和接收的数据是否相等，如果不相等则测试失败
    if !bytes.Equal(sendBuf, recvBuf[:]) {
        fmt.Println(sendBuf)
        fmt.Println(recvBuf[:])
        t.Fail()
    }
    // 关闭连接和对象
    conn1.Close()
    conn2.Close()
    packet1.Close()
    packet2.Close()
    conn.Close()
    c.Close()
    s.Close()
    cancel()
# 闭合前面的函数定义
```