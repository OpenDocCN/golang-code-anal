# `v2ray-core\common\buf\readv_test.go`

```go
// +build !wasm

// 声明包名为 buf_test
package buf_test

// 导入所需的包
import (
    "crypto/rand"
    "net"
    "testing"

    "github.com/google/go-cmp/cmp"

    "golang.org/x/sync/errgroup"
    "v2ray.com/core/common"
    . "v2ray.com/core/common/buf"
    "v2ray.com/core/testing/servers/tcp"
)

// 定义测试函数 TestReadvReader
func TestReadvReader(t *testing.T) {
    // 创建一个 TCP 服务器
    tcpServer := &tcp.Server{
        MsgProcessor: func(b []byte) []byte {
            return b
        },
    }
    // 启动 TCP 服务器
    dest, err := tcpServer.Start()
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close() // nolint: errcheck

    // 连接到 TCP 服务器
    conn, err := net.Dial("tcp", dest.NetAddr())
    common.Must(err)
    // 延迟关闭连接
    defer conn.Close() // nolint: errcheck

    // 定义常量 size 为 8192
    const size = 8192
    // 创建一个长度为 8192 的字节数组
    data := make([]byte, 8192)
    // 从加密安全的随机数生成器中读取随机字节并存入 data 中
    common.Must2(rand.Read(data))

    // 创建一个错误组
    var errg errgroup.Group
    // 启动一个协程，向连接中写入数据
    errg.Go(func() error {
        writer := NewWriter(conn)
        mb := MergeBytes(nil, data)

        return writer.WriteMultiBuffer(mb)
    })

    // 延迟执行函数
    defer func() {
        // 等待错误组中的任务执行完毕，如果有错误则输出错误信息
        if err := errg.Wait(); err != nil {
            t.Error(err)
        }
    }()

    // 获取原始连接的系统调用接口
    rawConn, err := conn.(*net.TCPConn).SyscallConn()
    common.Must(err)

    // 创建一个读取多缓冲区的读取器
    reader := NewReadVReader(conn, rawConn)
    var rmb MultiBuffer
    // 循环读取数据
    for {
        mb, err := reader.ReadMultiBuffer()
        if err != nil {
            t.Fatal("unexpected error: ", err)
        }
        rmb, _ = MergeMulti(rmb, mb)
        if rmb.Len() == size {
            break
        }
    }

    // 创建一个长度为 size 的字节数组 rdata
    rdata := make([]byte, size)
    // 将多缓冲区数据拆分到 rdata 中
    SplitBytes(rmb, rdata)

    // 比较原始数据和读取的数据，如果不一致则输出差异
    if r := cmp.Diff(data, rdata); r != "" {
        t.Fatal(r)
    }
}
```