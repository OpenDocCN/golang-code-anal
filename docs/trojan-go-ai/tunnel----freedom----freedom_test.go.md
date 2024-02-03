# `trojan-go\tunnel\freedom\freedom_test.go`

```go
package freedom

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于管理协程的上下文
    "fmt"  // 导入 fmt 包，用于格式化输出
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，用于处理时间

    "github.com/txthinking/socks5"  // 导入 socks5 包，用于实现 SOCKS5 代理

    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 的 common 包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入 trojan-go 的测试工具包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 的隧道包
)

func TestConn(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    client := &Client{  // 创建一个 Client 结构体对象
        ctx:    ctx,  // 设置上下文
        cancel: cancel,  // 设置取消函数
    }
    addr, err := tunnel.NewAddressFromAddr("tcp", util.EchoAddr)  // 从地址字符串创建一个地址对象
    common.Must(err)  // 如果 err 不为 nil，则触发 panic
    conn1, err := client.DialConn(addr, nil)  // 使用 Client 对象连接到指定地址
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    sendBuf := util.GeneratePayload(1024)  // 生成指定大小的负载数据
    recvBuf := [1024]byte{}  // 创建一个大小为 1024 的字节数组

    common.Must2(conn1.Write(sendBuf))  // 向连接中写入数据
    common.Must2(conn1.Read(recvBuf[:]))  // 从连接中读取数据

    if !bytes.Equal(sendBuf, recvBuf[:]) {  // 判断两个字节序列是否相等
        t.Fail()  // 标记测试函数为失败
    }
    client.Close()  // 关闭客户端连接
}

func TestPacket(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文
    client := &Client{  // 创建一个 Client 结构体对象
        ctx:    ctx,  // 设置上下文
        cancel: cancel,  // 设置取消函数
    }
    addr, err := tunnel.NewAddressFromAddr("udp", util.EchoAddr)  // 从地址字符串创建一个地址对象
    common.Must(err)  // 如果 err 不为 nil，则触发 panic
    conn1, err := client.DialPacket(nil)  // 使用 Client 对象连接到指定地址
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    sendBuf := util.GeneratePayload(1024)  // 生成指定大小的负载数据
    recvBuf := [1024]byte{}  // 创建一个大小为 1024 的字节数组

    common.Must2(conn1.WriteTo(sendBuf, addr))  // 向连接中写入数据并指定目标地址
    _, _, err = conn1.ReadFrom(recvBuf[:])  // 从连接中读取数据并获取来源地址
    common.Must(err)  // 如果 err 不为 nil，则触发 panic

    if !bytes.Equal(sendBuf, recvBuf[:]) {  // 判断两个字节序列是否相等
        t.Fail()  // 标记测试函数为失败
    }
}

func TestSocks(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())  // 创建一个带有取消功能的上下文

    socksAddr := tunnel.NewAddressFromHostPort("udp", "127.0.0.1", common.PickPort("udp", "127.0.0.1"))  // 从主机和端口创建一个地址对象
    client := &Client{  // 创建一个 Client 结构体对象
        ctx:          ctx,  // 设置上下文
        cancel:       cancel,  // 设置取消函数
        proxyAddr:    socksAddr,  // 设置代理地址
        forwardProxy: true,  // 设置为转发代理
        noDelay:      true,  // 设置为无延迟
    }
    target, err := tunnel.NewAddressFromAddr("tcp", util.EchoAddr)  // 从地址字符串创建一个地址对象
    common.Must(err)  // 如果 err 不为 nil，则触发 panic
    s, _ := socks5.NewClassicServer(socksAddr.String(), "127.0.0.1", "", "", 0, 0)  // 创建一个经典的 SOCKS5 代理服务器
    s.Handle = &socks5.DefaultHandle{}  // 设置代理服务器的处理程序
    go s.RunTCPServer()  // 在新的协程中运行 TCP 代理服务器
    # 运行 UDP 服务器
    go s.RunUDPServer()

    # 等待 2 秒
    time.Sleep(time.Second * 2)
    # 通过客户端连接到目标地址
    conn, err := client.DialConn(target, nil)
    common.Must(err)
    # 生成 1024 字节的数据包
    payload := util.GeneratePayload(1024)
    common.Must2(conn.Write(payload))

    # 读取接收缓冲区
    recvBuf := [1024]byte{}
    conn.Read(recvBuf[:])
    # 检查接收到的数据是否与发送的数据相等，如果不相等则测试失败
    if !bytes.Equal(recvBuf[:], payload) {
        t.Fail()
    }
    # 关闭连接
    conn.Close()

    # 通过客户端连接到目标地址并发送数据包
    packet, err := client.DialPacket(nil)
    common.Must(err)
    common.Must2(packet.WriteWithMetadata(payload, &tunnel.Metadata{
        Address: target,
    }))

    # 读取带有元数据的数据包
    recvBuf = [1024]byte{}
    n, m, err := packet.ReadWithMetadata(recvBuf[:])
    common.Must(err)

    # 检查接收到的数据长度和内容是否与发送的数据相等，如果不相等则测试失败
    if n != 1024 || !bytes.Equal(recvBuf[:], payload) {
        t.Fail()
    }

    # 打印元数据
    fmt.Println(m)
    # 关闭数据包连接
    packet.Close()
    # 关闭客户端连接
    client.Close()
# 闭合前面的函数定义
```