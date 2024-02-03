# `trojan-go\tunnel\socks\socks_test.go`

```go
package socks_test

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "context"  // 导入 context 包，用于跟踪请求的上下文
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io/ioutil"  // 导入 ioutil 包，用于读取文件内容
    "net"  // 导入 net 包，提供了用于网络通信的基本接口
    "sync"  // 导入 sync 包，提供了同步原语的基本操作

    "github.com/txthinking/socks5"  // 导入 socks5 包，实现了 SOCKS5 代理协议
    "golang.org/x/net/proxy"  // 导入 proxy 包，用于实现代理

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包，包含了一些通用的函数和常量
    "github.com/p4gefau1t/trojan-go/config"  // 导入 config 包，用于配置管理
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入 util 包，用于测试工具
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 tunnel 包，用于实现隧道
    "github.com/p4gefau1t/trojan-go/tunnel/adapter"  // 导入 adapter 包，用于适配器
    "github.com/p4gefau1t/trojan-go/tunnel/socks"  // 导入 socks 包，用于实现 SOCKS5 代理
)

func TestSocks(t *testing.T) {
    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的端口
    ctx := config.WithConfig(context.Background(), adapter.Name, &adapter.Config{  // 使用适配器配置创建上下文
        LocalHost: "127.0.0.1",  // 设置本地主机地址
        LocalPort: port,  // 设置本地端口
    })
    ctx = config.WithConfig(ctx, socks.Name, &socks.Config{  // 使用 SOCKS 配置更新上下文
        LocalHost: "127.0.0.1",  // 设置本地主机地址
        LocalPort: port,  // 设置本地端口
    })
    tcpServer, err := adapter.NewServer(ctx, nil)  // 创建新的适配器服务器
    common.Must(err)  // 检查错误
    addr := tunnel.NewAddressFromHostPort("tcp", "127.0.0.1", port)  // 创建新的地址
    s, err := socks.NewServer(ctx, tcpServer)  // 创建新的 SOCKS 服务器
    common.Must(err)  // 检查错误
    socksClient, err := proxy.SOCKS5("tcp", addr.String(), nil, proxy.Direct)  // 创建新的 SOCKS5 客户端
    common.Must(err)  // 检查错误
    var conn1, conn2 net.Conn  // 声明两个网络连接
    wg := sync.WaitGroup{}  // 创建同步等待组
    wg.Add(2)  // 添加两个等待

    time.Sleep(time.Second * 2)  // 休眠 2 秒
    go func() {
        conn2, err = s.AcceptConn(nil)  // 接受连接
        common.Must(err)  // 检查错误
        wg.Done()  // 完成一个等待
    }()

    time.Sleep(time.Second * 1)  // 休眠 1 秒
    go func() {
        conn1, err = socksClient.Dial("tcp", util.EchoAddr)  // 使用 SOCKS5 客户端连接到指定地址
        common.Must(err)  // 检查错误
        wg.Done()  // 完成一个等待
    }()

    wg.Wait()  // 等待所有协程完成
    if !util.CheckConn(conn1, conn2) {  // 检查连接是否成功
        t.Fail()  // 测试失败
    }
    fmt.Println(conn2.(tunnel.Conn).Metadata())  // 打印连接的元数据信息

    udpConn, err := net.ListenPacket("udp", ":0")  // 监听 UDP 连接
    common.Must(err)  // 检查错误

    addr = &tunnel.Address{  // 创建新的地址
        AddressType: tunnel.DomainName,  // 设置地址类型为域名
        DomainName:  "google.com",  // 设置域名
        Port:        12345,  // 设置端口
    }

    payload := util.GeneratePayload(1024)  // 生成指定大小的有效载荷
    buf := bytes.NewBuffer(make([]byte, 0, 4096))  // 创建新的字节缓冲区
    buf.Write([]byte{0, 0, 0}) // RSV, FRAG  // 写入指定字节到缓冲区
    # 将地址写入缓冲区
    common.Must(addr.WriteTo(buf))
    # 将有效载荷写入缓冲区
    buf.Write(payload)

    # 将缓冲区的字节发送到指定的 UDP 地址
    udpConn.WriteTo(buf.Bytes(), &net.UDPAddr{
        IP:   net.ParseIP("127.0.0.1"),
        Port: port,
    })

    # 接受一个数据包
    packet, err := s.AcceptPacket(nil)
    common.Must(err)
    # 创建一个接收缓冲区
    recvBuf := make([]byte, 4096)
    # 从数据包中读取数据和元数据
    n, m, err := packet.ReadWithMetadata(recvBuf)
    common.Must(err)
    # 检查元数据的域名、端口和数据长度，以及接收到的数据是否与有效载荷相等
    if m.DomainName != "google.com" || m.Port != 12345 || n != 1024 || !(bytes.Equal(recvBuf[:n], payload)) {
        t.Fail()
    }

    # 生成新的有效载荷
    payload = util.GeneratePayload(1024)
    # 向数据包写入数据和元数据
    _, err = packet.WriteWithMetadata(payload, &tunnel.Metadata{
        Address: &tunnel.Address{
            AddressType: tunnel.IPv4,
            IP:          net.ParseIP("123.123.234.234"),
            Port:        12345,
        },
    })
    common.Must(err)

    # 从 UDP 连接中读取数据到接收缓冲区
    _, _, err = udpConn.ReadFrom(recvBuf)
    common.Must(err)

    # 创建一个新的读取器
    r := bytes.NewReader(recvBuf)
    header := [3]byte{}
    # 从读取器中读取头部数据
    r.Read(header[:])
    # 创建一个新的地址对象
    addr = new(tunnel.Address)
    # 从读取器中读取地址数据
    common.Must(addr.ReadFrom(r))
    # 检查读取的地址是否与预期相等
    if addr.IP.String() != "123.123.234.234" || addr.Port != 12345 {
        t.Fail()
    }

    # 从读取器中读取剩余的数据
    recvBuf, err = ioutil.ReadAll(r)
    common.Must(err)

    # 检查接收到的数据是否与有效载荷相等
    if bytes.Equal(recvBuf, payload) {
        t.Fail()
    }
    # 关闭数据包
    packet.Close()
    # 关闭 UDP 连接
    udpConn.Close()

    # 创建一个新的 SOCKS5 客户端
    c, _ := socks5.NewClient(fmt.Sprintf("127.0.0.1:%d", port), "", "", 0, 0)

    # 通过 SOCKS5 客户端拨号到指定的地址和协议
    conn, err := c.Dial("udp", util.EchoAddr)
    common.Must(err)

    # 生成新的有效载荷
    payload = util.GeneratePayload(4096)
    # 创建一个新的接收缓冲区
    recvBuf = make([]byte, 4096)

    # 向连接中写入有效载荷
    conn.Write(payload)

    # 接受一个新的数据包
    newPacket, err := s.AcceptPacket(nil)
    common.Must(err)

    # 从数据包中读取数据和元数据
    _, m, err = newPacket.ReadWithMetadata(recvBuf)
    common.Must(err)
    # 检查元数据的字符串表示和接收到的数据是否与预期相等
    if m.String() != util.EchoAddr || !bytes.Equal(recvBuf, payload) {
        t.Fail()
    }

    # 关闭数据包
    s.Close()
# 闭合前面的函数定义
```