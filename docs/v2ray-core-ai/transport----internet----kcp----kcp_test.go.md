# `v2ray-core\transport\internet\kcp\kcp_test.go`

```
package kcp_test

import (
    "context"  // 上下文包，用于控制请求的取消、超时等
    "crypto/rand"  // 加密随机数生成包
    "io"  // 输入输出包
    "testing"  // 测试包
    "time"  // 时间包

    "github.com/google/go-cmp/cmp"  // Google 的比较包
    "golang.org/x/sync/errgroup"  // Go 语言的错误组包

    "v2ray.com/core/common"  // V2Ray 核心通用包
    "v2ray.com/core/common/errors"  // V2Ray 核心错误包
    "v2ray.com/core/common/net"  // V2Ray 核心网络包
    "v2ray.com/core/transport/internet"  // V2Ray 核心传输层网络包
    . "v2ray.com/core/transport/internet/kcp"  // 导入 KCP 传输层网络包
)

func TestDialAndListen(t *testing.T) {
    listerner, err := NewListener(context.Background(), net.LocalHostIP, net.Port(0), &internet.MemoryStreamConfig{
        ProtocolName:     "mkcp",  // 设置协议名称为 mkcp
        ProtocolSettings: &Config{},  // 设置协议配置为默认配置
    }, func(conn internet.Connection) {  // 定义连接处理函数
        go func(c internet.Connection) {  // 启动一个协程处理连接
            payload := make([]byte, 4096)  // 创建一个 4096 字节大小的数据缓冲区
            for {  // 循环读取数据
                nBytes, err := c.Read(payload)  // 从连接中读取数据
                if err != nil {  // 如果读取出错
                    break  // 退出循环
                }
                for idx, b := range payload[:nBytes] {  // 遍历读取的数据
                    payload[idx] = b ^ 'c'  // 对每个字节进行异或运算
                }
                c.Write(payload[:nBytes])  // 将处理后的数据写回连接
            }
            c.Close()  // 关闭连接
        }(conn)  // 传入连接处理函数
    })
    common.Must(err)  // 检查错误
    defer listerner.Close()  // 延迟关闭监听器

    port := net.Port(listerner.Addr().(*net.UDPAddr).Port)  // 获取监听器的端口号

    var errg errgroup.Group  // 定义错误组
    # 循环10次，每次执行以下操作
    for i := 0; i < 10; i++ {
        # 使用 errgroup.Go 方法并发执行以下函数
        errg.Go(func() error {
            # 使用 DialKCP 方法建立客户端连接
            clientConn, err := DialKCP(context.Background(), net.UDPDestination(net.LocalHostIP, port), &internet.MemoryStreamConfig{
                ProtocolName:     "mkcp",
                ProtocolSettings: &Config{},
            })
            # 如果建立连接出现错误，则返回该错误
            if err != nil {
                return err
            }
            # 延迟关闭客户端连接
            defer clientConn.Close()

            # 创建客户端发送的数据
            clientSend := make([]byte, 1024*1024)
            rand.Read(clientSend)
            # 并发执行客户端连接的写操作
            go clientConn.Write(clientSend)

            # 创建客户端接收的数据
            clientReceived := make([]byte, 1024*1024)
            # 读取客户端连接的数据
            common.Must2(io.ReadFull(clientConn, clientReceived))

            # 创建客户端期望接收的数据
            clientExpected := make([]byte, 1024*1024)
            # 对客户端发送的数据进行异或操作，得到期望接收的数据
            for idx, b := range clientSend {
                clientExpected[idx] = b ^ 'c'
            }
            # 比较客户端接收的数据和期望接收的数据，如果不相等则返回错误
            if r := cmp.Diff(clientReceived, clientExpected); r != "" {
                return errors.New(r)
            }
            # 返回空值表示没有错误
            return nil
        })
    }

    # 等待并发操作完成，如果有错误则终止测试
    if err := errg.Wait(); err != nil {
        t.Fatal(err)
    }

    # 循环检查监听器的活动连接数，最多循环60次
    for i := 0; i < 60 && listerner.ActiveConnections() > 0; i++ {
        # 暂停500毫秒
        time.Sleep(500 * time.Millisecond)
    }
    # 检查最终的活动连接数，如果不为0则输出错误信息
    if v := listerner.ActiveConnections(); v != 0 {
        t.Error("active connections: ", v)
    }
# 闭合前面的函数定义
```