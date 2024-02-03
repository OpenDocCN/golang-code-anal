# `trojan-go\tunnel\simplesocks\simplesocks_test.go`

```go
package simplesocks

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "testing"  // 导入测试包

    "github.com/p4gefau1t/trojan-go/common"  // 导入trojan-go的common包
    "github.com/p4gefau1t/trojan-go/config"  // 导入trojan-go的config包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入trojan-go的test/util包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入trojan-go的tunnel包
    "github.com/p4gefau1t/trojan-go/tunnel/freedom"  // 导入trojan-go的tunnel/freedom包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入trojan-go的tunnel/transport包
)

func TestSimpleSocks(t *testing.T) {
    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的端口
    transportConfig := &transport.Config{  // 创建transport配置
        LocalHost:  "127.0.0.1",  // 本地主机地址
        LocalPort:  port,  // 本地端口
        RemoteHost: "127.0.0.1",  // 远程主机地址
        RemotePort: port,  // 远程端口
    }
    ctx := config.WithConfig(context.Background(), transport.Name, transportConfig)  // 使用transport配置创建上下文
    ctx = config.WithConfig(ctx, freedom.Name, &freedom.Config{})  // 使用freedom配置更新上下文
    tcpClient, err := transport.NewClient(ctx, nil)  // 创建transport客户端
    common.Must(err)  // 检查错误
    tcpServer, err := transport.NewServer(ctx, nil)  // 创建transport服务器
    common.Must(err)  // 检查错误

    c, err := NewClient(ctx, tcpClient)  // 创建简单socks客户端
    common.Must(err)  // 检查错误
    s, err := NewServer(ctx, tcpServer)  // 创建简单socks服务器
    common.Must(err)  // 检查错误

    conn1, err := c.DialConn(&tunnel.Address{  // 通过简单socks客户端拨号连接
        DomainName:  "www.baidu.com",  // 目标域名
        AddressType: tunnel.DomainName,  // 地址类型
        Port:        443,  // 端口
    }, nil)  // 无额外参数
    common.Must(err)  // 检查错误
    defer conn1.Close()  // 延迟关闭连接
    conn1.Write(util.GeneratePayload(1024))  // 写入数据
    conn2, err := s.AcceptConn(nil)  // 接受连接
    common.Must(err)  // 检查错误
    defer conn2.Close()  // 延迟关闭连接
    buf := [1024]byte{}  // 创建缓冲区
    common.Must2(conn2.Read(buf[:]))  // 读取数据
    if !util.CheckConn(conn1, conn2) {  // 检查连接
        t.Fail()  // 测试失败
    }

    packet1, err := c.DialPacket(nil)  // 通过简单socks客户端拨号数据包
    common.Must(err)  // 检查错误
    packet1.WriteWithMetadata([]byte("12345678"), &tunnel.Metadata{  // 写入数据和元数据
        Address: &tunnel.Address{  // 地址信息
            DomainName:  "test.com",  // 目标域名
            AddressType: tunnel.DomainName,  // 地址类型
            Port:        443,  // 端口
        },
    })
    defer packet1.Close()  // 延迟关闭数据包
    packet2, err := s.AcceptPacket(nil)  // 接受数据包
    common.Must(err)  // 检查错误
    defer packet2.Close()  // 延迟关闭数据包
    _, m, err := packet2.ReadWithMetadata(buf[:])  // 读取数据和元数据
    common.Must(err)  // 检查错误
    fmt.Println(m)  // 打印元数据
}
    # 如果包不完整，即 packet1 和 packet2 不匹配，则测试失败
    if !util.CheckPacketOverConn(packet1, packet2) {
        # 测试失败
        t.Fail()
    }
    # 关闭服务器连接
    s.Close()
    # 关闭客户端连接
    c.Close()
# 闭合前面的函数定义
```