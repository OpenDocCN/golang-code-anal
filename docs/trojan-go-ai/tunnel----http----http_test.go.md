# `trojan-go\tunnel\http\http_test.go`

```go
package http

import (
    "bufio"  // 导入 bufio 包，提供了缓冲 I/O 的功能
    "context"  // 导入 context 包，用于跟踪请求的上下文
    "fmt"  // 导入 fmt 包，用于格式化输入输出
    "io/ioutil"  // 导入 ioutil 包，提供了一些 I/O 实用函数
    "net"  // 导入 net 包，提供了基本的网络功能
    "net/http"  // 导入 http 包，提供了 HTTP 客户端和服务端的实现
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，提供了时间的测量和显示功能

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/config"  // 导入自定义包 config
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入自定义包 util
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 导入自定义包 transport
)

func TestHTTP(t *testing.T) {
    port := common.PickPort("tcp", "127.0.0.1")  // 选择一个可用的端口号
    ctx := config.WithConfig(context.Background(), transport.Name, &transport.Config{  // 使用 transport 包的配置创建上下文
        LocalHost: "127.0.0.1",  // 设置本地主机地址
        LocalPort: port,  // 设置本地端口号
    })

    tcpServer, err := transport.NewServer(ctx, nil)  // 创建一个新的 transport 服务器
    common.Must(err)  // 检查错误并处理

    s, err := NewServer(ctx, tcpServer)  // 创建一个新的 HTTP 服务器
    common.Must(err)  // 检查错误并处理

    for i := 0; i < 10; i++ {  // 循环 10 次
        go func() {  // 启动一个新的 goroutine
            resp, err := http.Get(fmt.Sprintf("http://127.0.0.1:%d", port))  // 发起 HTTP GET 请求
            common.Must(err)  // 检查错误并处理
            defer resp.Body.Close()  // 延迟关闭响应体
        }()
        time.Sleep(time.Microsecond * 10)  // 休眠一段时间
        conn, err := s.AcceptConn(nil)  // 接受连接
        common.Must(err)  // 检查错误并处理
        bufReader := bufio.NewReader(bufio.NewReader(conn))  // 创建一个新的缓冲读取器
        req, err := http.ReadRequest(bufReader)  // 从缓冲读取器中读取 HTTP 请求
        common.Must(err)  // 检查错误并处理
        fmt.Println(req)  // 打印请求信息
        ioutil.ReadAll(req.Body)  // 读取请求体的所有内容
        req.Body.Close()  // 关闭请求体
        resp, err := http.Get("http://127.0.0.1:" + util.HTTPPort)  // 发起 HTTP GET 请求
        common.Must(err)  // 检查错误并处理
        defer resp.Body.Close()  // 延迟关闭响应体
        err = resp.Write(conn)  // 将响应写入连接
        common.Must(err)  // 检查错误并处理
        buf := [100]byte{}  // 创建一个长度为 100 的字节数组
        _, err = conn.Read(buf[:])  // 从连接中读取数据到字节数组
        if err == nil {  // 如果没有错误
            t.Fail()  // 测试失败
        }
        conn.Close()  // 关闭连接
    }

    req, err := http.NewRequest(http.MethodConnect, "https://google.com:443", nil)  // 创建一个新的 HTTP 请求
    common.Must(err)  // 检查错误并处理
    conn1, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", port))  // 拨号连接到指定地址和端口
    common.Must(err)  // 检查错误并处理
    go func() {  // 启动一个新的 goroutine
        common.Must(req.Write(conn1))  // 发送 HTTP 请求
    }()

    conn2, err := s.AcceptConn(nil)  // 接受连接
    common.Must(err)  // 检查错误并处理

    if conn2.Metadata().Port != 443 || conn2.Metadata().DomainName != "google.com" {  // 如果连接的端口不是 443 或者域名不是 google.com
        t.Fail()  // 测试失败
    }
}
    # 建立连接响应的字符串
    connResp := "HTTP/1.1 200 Connection established\r\n\r\n"
    # 创建一个与连接响应字符串相同长度的字节切片
    buf := make([]byte, len(connResp))
    # 从 conn1 中读取数据到 buf 中
    _, err = conn1.Read(buf)
    # 检查是否有错误发生
    common.Must(err)
    # 如果读取的数据不等于连接响应字符串，则测试失败
    if string(buf) != connResp {
        t.Fail()
    }

    # 检查两个连接是否有效
    if !util.CheckConn(conn1, conn2) {
        t.Fail()
    }

    # 关闭连接1
    conn1.Close()
    # 关闭连接2
    conn2.Close()
    # 关闭服务器
    s.Close()
# 闭合前面的函数定义
```