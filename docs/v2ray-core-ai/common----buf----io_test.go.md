# `v2ray-core\common\buf\io_test.go`

```go
package buf_test

import (
    "crypto/tls"  // 导入加密传输层协议包
    "io"  // 导入输入输出包
    "testing"  // 导入测试包

    . "v2ray.com/core/common/buf"  // 导入v2ray的缓冲区包
    "v2ray.com/core/common/net"  // 导入v2ray的网络包
    "v2ray.com/core/testing/servers/tcp"  // 导入v2ray的TCP服务器测试包
)

func TestWriterCreation(t *testing.T) {
    tcpServer := tcp.Server{}  // 创建TCP服务器对象
    dest, err := tcpServer.Start()  // 启动TCP服务器
    if err != nil {
        t.Fatal("failed to start tcp server: ", err)  // 如果启动失败，输出错误信息
    }
    defer tcpServer.Close()  // 延迟关闭TCP服务器

    conn, err := net.Dial("tcp", dest.NetAddr())  // 连接到目标地址
    if err != nil {
        t.Fatal("failed to dial a TCP connection: ", err)  // 如果连接失败，输出错误信息
    }
    defer conn.Close()  // 延迟关闭连接

    {
        writer := NewWriter(conn)  // 创建一个新的写入器
        if _, ok := writer.(*BufferToBytesWriter); !ok {  // 检查写入器是否为BufferToBytesWriter类型
            t.Fatal("writer is not a BufferToBytesWriter")  // 如果不是，输出错误信息
        }

        writer2 := NewWriter(writer.(io.Writer))  // 创建另一个写入器
        if writer2 != writer {  // 检查两个写入器是否相同
            t.Fatal("writer is not reused")  // 如果不是，输出错误信息
        }
    }

    tlsConn := tls.Client(conn, &tls.Config{  // 创建TLS连接
        InsecureSkipVerify: true,  // 跳过证书验证
    })
    defer tlsConn.Close()  // 延迟关闭TLS连接

    {
        writer := NewWriter(tlsConn)  // 创建一个新的写入器
        if _, ok := writer.(*SequentialWriter); !ok {  // 检查写入器是否为SequentialWriter类型
            t.Fatal("writer is not a SequentialWriter")  // 如果不是，输出错误信息
        }
    }
}
```