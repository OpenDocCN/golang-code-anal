# `trojan-go\tunnel\dokodemo\dokodemo_test.go`

```go
package dokodemo

import (
    "context"  // 导入上下文包
    "fmt"  // 导入格式化包
    "net"  // 导入网络包
    "sync"  // 导入同步包
    "testing"  // 导入测试包

    "github.com/p4gefau1t/trojan-go/common"  // 导入trojan-go的common包
    "github.com/p4gefau1t/trojan-go/config"  // 导入trojan-go的config包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入trojan-go的测试工具包
)

func TestDokodemo(t *testing.T) {
    cfg := &Config{  // 创建Config对象
        LocalHost:  "127.0.0.1",  // 设置本地主机地址
        LocalPort:  common.PickPort("tcp", "127.0.0.1"),  // 设置本地端口
        TargetHost: "127.0.0.1",  // 设置目标主机地址
        TargetPort: common.PickPort("tcp", "127.0.0.1"),  // 设置目标端口
        UDPTimeout: 30,  // 设置UDP超时时间
    }
    ctx := config.WithConfig(context.Background(), Name, cfg)  // 使用配置创建上下文
    s, err := NewServer(ctx, nil)  // 创建新的服务器
    common.Must(err)  // 检查错误
    conn1, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", cfg.LocalPort))  // 建立TCP连接
    common.Must(err)  // 检查错误
    conn2, err := s.AcceptConn(nil)  // 接受连接
    common.Must(err)  // 检查错误
    if !util.CheckConn(conn1, conn2) {  // 检查连接
        t.Fail()  // 测试失败
    }
    conn1.Close()  // 关闭连接
    conn2.Close()  // 关闭连接

    wg := sync.WaitGroup{}  // 创建同步等待组
    wg.Add(1)  // 添加一个等待

    packet1, err := net.ListenPacket("udp", "")  // 监听UDP数据包
    common.Must(err)  // 检查错误
    common.Must2(packet1.(*net.UDPConn).WriteToUDP([]byte("hello1"), &net.UDPAddr{  // 写入UDP数据包
        IP:   net.ParseIP("127.0.0.1"),  // 设置IP地址
        Port: cfg.LocalPort,  // 设置端口
    }))
    packet2, err := s.AcceptPacket(nil)  // 接受UDP数据包
    common.Must(err)  // 检查错误
    buf := [100]byte{}  // 创建缓冲区
    n, m, err := packet2.ReadWithMetadata(buf[:])  // 读取UDP数据包和元数据
    common.Must(err)  // 检查错误
    if m.Address.Port != cfg.TargetPort {  // 检查目标端口
        t.Fail()  // 测试失败
    }
    if string(buf[:n]) != "hello1" {  // 检查数据内容
        t.Fail()  // 测试失败
    }
    fmt.Println(n, m, string(buf[:n]))  // 打印数据长度、元数据和内容

    if !util.CheckPacket(packet1, packet2) {  // 检查UDP数据包
        t.Fail()  // 测试失败
    }

    packet3, err := net.ListenPacket("udp", "")  // 监听UDP数据包
    common.Must(err)  // 检查错误
    common.Must2(packet3.(*net.UDPConn).WriteToUDP([]byte("hello2"), &net.UDPAddr{  // 写入UDP数据包
        IP:   net.ParseIP("127.0.0.1"),  // 设置IP地址
        Port: cfg.LocalPort,  // 设置端口
    }))
    packet4, err := s.AcceptPacket(nil)  // 接受UDP数据包
    common.Must(err)  // 检查错误
    n, m, err = packet4.ReadWithMetadata(buf[:])  // 读取UDP数据包和元数据
    common.Must(err)  // 检查错误
    if m.Address.Port != cfg.TargetPort {  // 检查目标端口
        t.Fail()  // 测试失败
    }
    if string(buf[:n]) != "hello2" {  // 检查数据内容
        t.Fail()  // 测试失败
    }
    # 打印变量 n, m, 以及 buf 的前 n 个字符转换成字符串
    fmt.Println(n, m, string(buf[:n]))

    # 创建一个 WaitGroup 对象 wg
    wg = sync.WaitGroup{}
    # 增加 WaitGroup 的计数器为 2
    wg.Add(2)
    # 启动一个 goroutine
    go func() {
        # 如果 packet3 和 packet4 的校验不通过，则标记测试失败
        if !util.CheckPacket(packet3, packet4) {
            t.Fail()
        }
        # 减少 WaitGroup 的计数器
        wg.Done()
    }()
    # 启动另一个 goroutine
    go func() {
        # 如果 packet1 和 packet2 的校验不通过，则标记测试失败
        if !util.CheckPacket(packet1, packet2) {
            t.Fail()
        }
        # 减少 WaitGroup 的计数器
        wg.Done()
    }()
    # 等待所有的 goroutine 完成
    wg.Wait()
    # 关闭连接
    s.Close()
# 闭合前面的函数定义
```