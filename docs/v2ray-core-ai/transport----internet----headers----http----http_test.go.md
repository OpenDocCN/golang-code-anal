# `v2ray-core\transport\internet\headers\http\http_test.go`

```
package http_test

import (
    "bufio" // 导入 bufio 包，提供了缓冲读写功能
    "bytes" // 导入 bytes 包，提供了操作字节切片的函数
    "context" // 导入 context 包，提供了处理请求的上下文
    "crypto/rand" // 导入 crypto/rand 包，提供了加密随机数生成
    "strings" // 导入 strings 包，提供了操作字符串的函数
    "testing" // 导入 testing 包，提供了单元测试功能
    "time" // 导入 time 包，提供了时间相关的函数

    "v2ray.com/core/common" // 导入 common 包，提供了通用的功能
    "v2ray.com/core/common/buf" // 导入 buf 包，提供了缓冲区相关的功能
    "v2ray.com/core/common/net" // 导入 net 包，提供了网络相关的功能
    . "v2ray.com/core/transport/internet/headers/http" // 导入 HTTP 头部相关的功能
)

func TestReaderWriter(t *testing.T) {
    cache := buf.New() // 创建一个新的缓冲区
    b := buf.New() // 创建一个新的缓冲区
    common.Must2(b.WriteString("abcd" + ENDING)) // 向缓冲区写入字符串
    writer := NewHeaderWriter(b) // 创建一个新的 HeaderWriter 对象
    err := writer.Write(cache) // 将 HeaderWriter 对象写入缓冲区
    common.Must(err) // 检查错误
    if v := cache.Len(); v != 8 { // 检查缓冲区长度是否为 8
        t.Error("cache len: ", v) // 输出错误信息
    }
    _, err = cache.Write([]byte{'e', 'f', 'g'}) // 向缓冲区写入字节切片
    common.Must(err) // 检查错误

    reader := &HeaderReader{} // 创建一个 HeaderReader 对象
    buffer, err := reader.Read(cache) // 从缓冲区读取数据到 buffer
    if err != nil && !strings.HasPrefix(err.Error(), "malformed HTTP request") { // 检查是否有错误并且错误信息是否以指定字符串开头
        t.Error("unknown error ", err) // 输出错误信息
    }
    _ = buffer // 忽略 buffer
    /*
        if buffer.String() != "efg" {
            t.Error("buffer: ", buffer.String())
        }*/
}

func TestRequestHeader(t *testing.T) {
    auth, err := NewHttpAuthenticator(context.Background(), &Config{ // 创建一个新的 HTTP 认证对象
        Request: &RequestConfig{ // 设置请求配置
            Uri: []string{"/"}, // 设置请求的 URI
            Header: []*Header{ // 设置请求头部
                {
                    Name:  "Test", // 设置头部名称
                    Value: []string{"Value"}, // 设置头部值
                },
            },
        },
    })
    common.Must(err) // 检查错误

    cache := buf.New() // 创建一个新的缓冲区
    err = auth.GetClientWriter().Write(cache) // 使用认证对象的客户端写入数据到缓冲区
    common.Must(err) // 检查错误

    if cache.String() != "GET / HTTP/1.1\r\nTest: Value\r\n\r\n" { // 检查缓冲区内容是否符合预期
        t.Error("cache: ", cache.String()) // 输出错误信息
    }
}

func TestLongRequestHeader(t *testing.T) {
    payload := make([]byte, buf.Size+2) // 创建一个指定长度的字节切片
    common.Must2(rand.Read(payload[:buf.Size-2])) // 生成随机字节填充到指定位置
    copy(payload[buf.Size-2:], []byte(ENDING)) // 将指定字符串复制到指定位置
    payload = append(payload, []byte("abcd")...) // 将指定字符串追加到字节切片末尾

    reader := HeaderReader{} // 创建一个 HeaderReader 对象
    b, err := reader.Read(bytes.NewReader(payload)) // 从字节流中读取数据到 buffer
    # 如果 err 不为空并且错误信息不以"invalid"或"malformed"开头，则输出错误信息
    if err != nil && !(strings.HasPrefix(err.Error(), "invalid") || strings.HasPrefix(err.Error(), "malformed")) {
        t.Error("unknown error ", err)
    }
    # 将 b 赋值给匿名变量，忽略其返回值
    _ = b
    /*
        # 必须处理 err，如果出错则触发 panic
        common.Must(err)
        # 如果 b 的字符串不等于"abcd"，则输出错误信息
        if b.String() != "abcd" {
            t.Error("expect content abcd, but actually ", b.String())
        }*/
func TestConnection(t *testing.T) {
    // 创建 HTTP 认证器
    auth, err := NewHttpAuthenticator(context.Background(), &Config{
        // 配置请求
        Request: &RequestConfig{
            Method: &Method{Value: "Post"},  // 设置请求方法为 POST
            Uri:    []string{"/testpath"},   // 设置请求路径
            Header: []*Header{               // 设置请求头
                {
                    Name:  "Host",
                    Value: []string{"www.v2ray.com", "www.google.com"},  // 设置 Host 请求头
                },
                {
                    Name:  "User-Agent",
                    Value: []string{"Test-Agent"},  // 设置 User-Agent 请求头
                },
            },
        },
        // 配置响应
        Response: &ResponseConfig{
            Version: &Version{
                Value: "1.1",  // 设置 HTTP 版本
            },
            Status: &Status{
                Code:   "404",          // 设置状态码
                Reason: "Not Found",    // 设置状态原因
            },
        },
    })
    common.Must(err)  // 检查错误

    // 监听 TCP 连接
    listener, err := net.Listen("tcp", "127.0.0.1:0")
    common.Must(err)  // 检查错误

    // 启动一个 goroutine 接受连接并进行认证
    go func() {
        conn, err := listener.Accept()
        common.Must(err)  // 检查错误
        authConn := auth.Server(conn)  // 服务端认证
        b := make([]byte, 256)
        for {
            n, err := authConn.Read(b)  // 读取数据
            if err != nil {
                break
            }
            _, err = authConn.Write(b[:n])  // 写入数据
            common.Must(err)  // 检查错误
        }
    }()

    // 建立 TCP 连接并进行客户端认证
    conn, err := net.DialTCP("tcp", nil, listener.Addr().(*net.TCPAddr))
    common.Must(err)  // 检查错误

    authConn := auth.Client(conn)  // 客户端认证
    defer authConn.Close()  // 延迟关闭连接

    authConn.Write([]byte("Test payload"))  // 写入测试数据
    authConn.Write([]byte("Test payload 2"))  // 写入测试数据

    expectedResponse := "Test payloadTest payload 2"  // 期望的响应数据
    actualResponse := make([]byte, 256)  // 创建缓冲区
    deadline := time.Now().Add(time.Second * 5)  // 设置超时时间
    totalBytes := 0
    for {
        n, err := authConn.Read(actualResponse[totalBytes:])  // 读取响应数据
        common.Must(err)  // 检查错误
        totalBytes += n
        if totalBytes >= len(expectedResponse) || time.Now().After(deadline) {
            break
        }
    }
}
    # 检查实际响应的前totalBytes字节是否与期望的响应相等
    if string(actualResponse[:totalBytes]) != expectedResponse {
        # 如果不相等，则输出错误信息，包括实际响应的前totalBytes字节内容
        t.Error("response: ", string(actualResponse[:totalBytes]))
    }
}
// 测试连接无效路径
func TestConnectionInvPath(t *testing.T) {
    // 创建 HTTP 认证器
    auth, err := NewHttpAuthenticator(context.Background(), &Config{
        // 配置请求
        Request: &RequestConfig{
            Method: &Method{Value: "Post"},
            Uri:    []string{"/testpath"},
            Header: []*Header{
                {
                    Name:  "Host",
                    Value: []string{"www.v2ray.com", "www.google.com"},
                },
                {
                    Name:  "User-Agent",
                    Value: []string{"Test-Agent"},
                },
            },
        },
        // 配置响应
        Response: &ResponseConfig{
            Version: &Version{
                Value: "1.1",
            },
            Status: &Status{
                Code:   "404",
                Reason: "Not Found",
            },
        },
    })
    // 检查错误
    common.Must(err)

    // 创建 HTTP 认证器
    authR, err := NewHttpAuthenticator(context.Background(), &Config{
        // 配置请求
        Request: &RequestConfig{
            Method: &Method{Value: "Post"},
            Uri:    []string{"/testpathErr"},
            Header: []*Header{
                {
                    Name:  "Host",
                    Value: []string{"www.v2ray.com", "www.google.com"},
                },
                {
                    Name:  "User-Agent",
                    Value: []string{"Test-Agent"},
                },
            },
        },
        // 配置响应
        Response: &ResponseConfig{
            Version: &Version{
                Value: "1.1",
            },
            Status: &Status{
                Code:   "404",
                Reason: "Not Found",
            },
        },
    })
    // 检查错误
    common.Must(err)

    // 监听网络连接
    listener, err := net.Listen("tcp", "127.0.0.1:0")
    // 检查错误
    common.Must(err)
}
    # 创建一个匿名函数，用于处理接受到的连接
    go func() {
        # 接受连接，返回连接对象和错误信息
        conn, err := listener.Accept()
        # 检查错误，如果有错误则触发panic
        common.Must(err)
        # 对接受到的连接进行身份验证
        authConn := auth.Server(conn)
        # 创建一个长度为256的字节切片
        b := make([]byte, 256)
        # 循环读取数据
        for {
            # 从连接中读取数据到字节切片b中，返回读取的字节数和错误信息
            n, err := authConn.Read(b)
            # 如果有错误
            if err != nil {
                # 关闭连接
                authConn.Close()
                # 退出循环
                break
            }
            # 将读取到的数据写回连接中，返回写入的字节数和错误信息
            _, err = authConn.Write(b[:n])
            # 检查错误，如果有错误则触发panic
            common.Must(err)
        }
    }()

    # 通过TCP连接到指定地址
    conn, err := net.DialTCP("tcp", nil, listener.Addr().(*net.TCPAddr))
    # 检查错误，如果有错误则触发panic
    common.Must(err)

    # 通过认证客户端连接对象包装连接
    authConn := authR.Client(conn)
    # 延迟关闭连接
    defer authConn.Close()

    # 向连接中写入测试负载数据
    authConn.Write([]byte("Test payload"))
    authConn.Write([]byte("Test payload 2"))

    # 期望的响应数据
    expectedResponse := "Test payloadTest payload 2"
    # 创建一个长度为256的字节切片
    actualResponse := make([]byte, 256)
    # 设置超时时间
    deadline := time.Now().Add(time.Second * 5)
    # 总共读取的字节数
    totalBytes := 0
    # 循环读取数据
    for {
        # 从连接中读取数据到字节切片actualResponse中，返回读取的字节数和错误信息
        n, err := authConn.Read(actualResponse[totalBytes:])
        # 如果没有错误
        if err == nil {
            # 报告错误
            t.Error("Error Expected", err)
        } else {
            # 返回
            return
        }
        # 更新总共读取的字节数
        totalBytes += n
        # 如果总共读取的字节数大于等于期望的响应长度，或者超过了超时时间
        if totalBytes >= len(expectedResponse) || time.Now().After(deadline) {
            # 退出循环
            break
        }
    }
func TestConnectionInvReq(t *testing.T) {
    // 创建 HTTP 认证器
    auth, err := NewHttpAuthenticator(context.Background(), &Config{
        // 配置请求
        Request: &RequestConfig{
            Method: &Method{Value: "Post"},  // 设置请求方法为 Post
            Uri:    []string{"/testpath"},   // 设置请求路径
            Header: []*Header{               // 设置请求头
                {
                    Name:  "Host",
                    Value: []string{"www.v2ray.com", "www.google.com"},  // 设置 Host 头
                },
                {
                    Name:  "User-Agent",
                    Value: []string{"Test-Agent"},  // 设置 User-Agent 头
                },
            },
        },
        // 配置响应
        Response: &ResponseConfig{
            Version: &Version{
                Value: "1.1",  // 设置 HTTP 版本
            },
            Status: &Status{
                Code:   "404",          // 设置状态码
                Reason: "Not Found",    // 设置状态原因
            },
        },
    })
    common.Must(err)  // 检查错误

    // 监听 TCP 连接
    listener, err := net.Listen("tcp", "127.0.0.1:0")
    common.Must(err)  // 检查错误

    // 启动一个 goroutine 处理连接
    go func() {
        conn, err := listener.Accept()  // 接受连接
        common.Must(err)  // 检查错误
        authConn := auth.Server(conn)  // 使用认证器处理连接
        b := make([]byte, 256)
        for {
            n, err := authConn.Read(b)  // 读取数据
            if err != nil {
                authConn.Close()  // 关闭连接
                break
            }
            _, err = authConn.Write(b[:n])  // 写入数据
            common.Must(err)  // 检查错误
        }
    }()

    // 发起 TCP 连接
    conn, err := net.DialTCP("tcp", nil, listener.Addr().(*net.TCPAddr))
    common.Must(err)  // 检查错误

    conn.Write([]byte("ABCDEFGHIJKMLN\r\n\r\n"))  // 写入数据
    l, _, err := bufio.NewReader(conn).ReadLine()  // 读取行
    common.Must(err)  // 检查错误
    if !strings.HasPrefix(string(l), "HTTP/1.1 400 Bad Request") {  // 检查响应
        t.Error("Resp to non http conn", string(l))  // 输出错误信息
    }
}
```