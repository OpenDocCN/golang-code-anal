# `v2ray-core\transport\internet\quic\quic_test.go`

```
package quic_test

import (
    "context"  // 引入上下文包
    "crypto/rand"  // 引入加密随机数生成包
    "testing"  // 引入测试包
    "time"  // 引入时间包

    "github.com/google/go-cmp/cmp"  // 引入 Google 的比较包

    "v2ray.com/core/common"  // 引入 V2Ray 核心通用包
    "v2ray.com/core/common/buf"  // 引入 V2Ray 核心缓冲区包
    "v2ray.com/core/common/net"  // 引入 V2Ray 核心网络包
    "v2ray.com/core/common/protocol"  // 引入 V2Ray 核心协议包
    "v2ray.com/core/common/protocol/tls/cert"  // 引入 V2Ray 核心 TLS 证书包
    "v2ray.com/core/common/serial"  // 引入 V2Ray 核心序列化包
    "v2ray.com/core/testing/servers/udp"  // 引入 V2Ray 核心 UDP 服务器测试包
    "v2ray.com/core/transport/internet"  // 引入 V2Ray 核心互联网传输包
    "v2ray.com/core/transport/internet/headers/wireguard"  // 引入 V2Ray 核心 WireGuard 头部包
    "v2ray.com/core/transport/internet/quic"  // 引入 V2Ray 核心 QUIC 传输包
    "v2ray.com/core/transport/internet/tls"  // 引入 V2Ray 核心 TLS 传输包
)

func TestQuicConnection(t *testing.T) {
    port := udp.PickPort()  // 选择一个 UDP 端口

    listener, err := quic.Listen(context.Background(), net.LocalHostIP, port, &internet.MemoryStreamConfig{  // 监听 QUIC 连接
        ProtocolName:     "quic",  // 设置协议名称为 QUIC
        ProtocolSettings: &quic.Config{},  // 设置 QUIC 配置
        SecurityType:     "tls",  // 设置安全类型为 TLS
        SecuritySettings: &tls.Config{  // 设置 TLS 配置
            Certificate: []*tls.Certificate{  // 设置证书
                tls.ParseCertificate(  // 解析证书
                    cert.MustGenerate(nil,  // 生成证书
                        cert.DNSNames("www.v2fly.org"),  // 设置域名
                    ),
                ),
            },
        },
    }, func(conn internet.Connection) {  // 处理连接的回调函数
        go func() {  // 启动一个新的 goroutine
            defer conn.Close()  // 延迟关闭连接

            b := buf.New()  // 创建一个新的缓冲区
            defer b.Release()  // 延迟释放缓冲区

            for {  // 循环处理数据
                b.Clear()  // 清空缓冲区
                if _, err := b.ReadFrom(conn); err != nil {  // 从连接中读取数据
                    return  // 如果出错则返回
                }
                common.Must2(conn.Write(b.Bytes()))  // 写入数据到连接
            }
        }()
    })
    common.Must(err)  // 检查错误

    defer listener.Close()  // 延迟关闭监听器

    time.Sleep(time.Second)  // 休眠一秒钟

    dctx := context.Background()  // 创建一个新的上下文
    conn, err := quic.Dial(dctx, net.TCPDestination(net.LocalHostIP, port), &internet.MemoryStreamConfig{  // 拨号连接
        ProtocolName:     "quic",  // 设置协议名称为 QUIC
        ProtocolSettings: &quic.Config{},  // 设置 QUIC 配置
        SecurityType:     "tls",  // 设置安全类型为 TLS
        SecuritySettings: &tls.Config{  // 设置 TLS 配置
            ServerName:    "www.v2fly.org",  // 设置服务器名称
            AllowInsecure: true,  // 允许不安全连接
        },
    })
    # 确保错误为nil，如果不是nil则会触发panic
    common.Must(err)
    # 延迟执行conn的关闭操作，确保在函数返回前关闭连接
    defer conn.Close()

    # 定义常量N为1024
    const N = 1024
    # 创建一个长度为N的字节切片b1
    b1 := make([]byte, N)
    # 从随机源中读取随机数据填充b1
    common.Must2(rand.Read(b1))
    # 创建一个新的字节缓冲区b2
    b2 := buf.New()

    # 将b1中的数据写入到conn连接中
    common.Must2(conn.Write(b1))

    # 清空b2中的数据
    b2.Clear()
    # 从conn连接中读取N个字节填充b2
    common.Must2(b2.ReadFullFrom(conn, N))
    # 比较b2中的数据和b1中的数据是否相同，如果不同则输出错误信息
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }

    # 将b1中的数据写入到conn连接中
    common.Must2(conn.Write(b1))

    # 清空b2中的数据
    b2.Clear()
    # 从conn连接中读取N个字节填充b2
    common.Must2(b2.ReadFullFrom(conn, N))
    # 比较b2中的数据和b1中的数据是否相同，如果不同则输出错误信息
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }
func TestQuicConnectionWithoutTLS(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 监听指定端口的 QUIC 连接
    listener, err := quic.Listen(context.Background(), net.LocalHostIP, port, &internet.MemoryStreamConfig{
        ProtocolName:     "quic",
        ProtocolSettings: &quic.Config{},
    }, func(conn internet.Connection) {
        // 在新连接上创建一个 goroutine 处理数据传输
        go func() {
            defer conn.Close()

            // 创建一个缓冲区
            b := buf.New()
            defer b.Release()

            for {
                // 清空缓冲区
                b.Clear()
                // 从连接中读取数据到缓冲区
                if _, err := b.ReadFrom(conn); err != nil {
                    return
                }
                // 将读取的数据写回连接
                common.Must2(conn.Write(b.Bytes()))
            }
        }()
    })
    // 检查错误
    common.Must(err)

    // 延迟关闭监听器
    defer listener.Close()

    // 等待一秒钟
    time.Sleep(time.Second)

    // 创建一个 QUIC 连接
    dctx := context.Background()
    conn, err := quic.Dial(dctx, net.TCPDestination(net.LocalHostIP, port), &internet.MemoryStreamConfig{
        ProtocolName:     "quic",
        ProtocolSettings: &quic.Config{},
    })
    // 检查错误
    common.Must(err)
    // 延迟关闭连接
    defer conn.Close()

    // 定义数据大小
    const N = 1024
    // 创建一个指定大小的字节数组
    b1 := make([]byte, N)
    // 生成随机数据填充到字节数组
    common.Must2(rand.Read(b1))
    // 创建一个缓冲区
    b2 := buf.New()

    // 将数据写入连接
    common.Must2(conn.Write(b1))

    // 清空缓冲区
    b2.Clear()
    // 从连接中读取指定大小的数据到缓冲区
    common.Must2(b2.ReadFullFrom(conn, N))
    // 检查读取的数据是否与原始数据一致
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }

    // 再次将数据写入连接
    common.Must2(conn.Write(b1))

    // 清空缓冲区
    b2.Clear()
    // 从连接中读取指定大小的数据到缓冲区
    common.Must2(b2.ReadFullFrom(conn, N))
    // 检查读取的数据是否与原始数据一致
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }
}

func TestQuicConnectionAuthHeader(t *testing.T) {
    // 选择一个可用的端口
    port := udp.PickPort()

    // 监听指定端口的带认证头的 QUIC 连接
    listener, err := quic.Listen(context.Background(), net.LocalHostIP, port, &internet.MemoryStreamConfig{
        ProtocolName: "quic",
        ProtocolSettings: &quic.Config{
            Header: serial.ToTypedMessage(&wireguard.WireguardConfig{}),
            Key:    "abcd",
            Security: &protocol.SecurityConfig{
                Type: protocol.SecurityType_AES128_GCM,
            },
        },
    }, func(conn internet.Connection) {
        // 在新的 goroutine 中处理连接
        go func() {
            // 延迟关闭连接
            defer conn.Close()

            // 创建一个新的缓冲区
            b := buf.New()
            // 延迟释放缓冲区
            defer b.Release()

            // 循环读取连接中的数据
            for {
                // 清空缓冲区
                b.Clear()
                // 从连接中读取数据到缓冲区
                if _, err := b.ReadFrom(conn); err != nil {
                    return
                }
                // 必须写入连接中的数据
                common.Must2(conn.Write(b.Bytes()))
            }
        }()
    })
    // 必须处理错误
    common.Must(err)

    // 延迟关闭监听器
    defer listener.Close()

    // 等待一秒钟
    time.Sleep(time.Second)

    // 创建一个新的上下文
    dctx := context.Background()
    // 使用 QUIC 协议进行连接
    conn, err := quic.Dial(dctx, net.TCPDestination(net.LocalHostIP, port), &internet.MemoryStreamConfig{
        ProtocolName: "quic",
        ProtocolSettings: &quic.Config{
            Header: serial.ToTypedMessage(&wireguard.WireguardConfig{}),
            Key:    "abcd",
            Security: &protocol.SecurityConfig{
                Type: protocol.SecurityType_AES128_GCM,
            },
        },
    })
    // 必须处理错误
    common.Must(err)
    // 延迟关闭连接
    defer conn.Close()

    // 定义常量 N
    const N = 1024
    // 创建一个大小为 N 的字节数组
    b1 := make([]byte, N)
    // 从随机源中读取 N 个字节到 b1 中
    common.Must2(rand.Read(b1))
    // 创建一个新的缓冲区
    b2 := buf.New()

    // 必须写入连接中的数据
    common.Must2(conn.Write(b1))

    // 清空缓冲区
    b2.Clear()
    // 从连接中读取 N 个字节到缓冲区中
    common.Must2(b2.ReadFullFrom(conn, N))
    // 比较两个字节数组是否相等，如果不相等则输出错误信息
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }

    // 必须写入连接中的数据
    common.Must2(conn.Write(b1))

    // 清空缓冲区
    b2.Clear()
    // 从连接中读取 N 个字节到缓冲区中
    common.Must2(b2.ReadFullFrom(conn, N))
    // 比较两个字节数组是否相等，如果不相等则输出错误信息
    if r := cmp.Diff(b2.Bytes(), b1); r != "" {
        t.Error(r)
    }
# 闭合前面的函数定义
```