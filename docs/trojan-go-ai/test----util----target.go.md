# `trojan-go\test\util\target.go`

```
package util

import (
    "crypto/rand"  // 导入加密随机数生成包
    "fmt"  // 导入格式化包
    "io"  // 导入输入输出包
    "io/ioutil"  // 导入输入输出工具包
    "net"  // 导入网络包
    "net/http"  // 导入HTTP包
    "sync"  // 导入同步包
    "time"  // 导入时间包

    "golang.org/x/net/websocket"  // 导入websocket包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义common包
    "github.com/p4gefau1t/trojan-go/log"  // 导入自定义log包
)

var (
    HTTPAddr string  // 定义HTTP地址变量
    HTTPPort string  // 定义HTTP端口变量
)

func runHelloHTTPServer() {
    httpHello := func(w http.ResponseWriter, req *http.Request) {  // 定义HTTP处理函数
        w.Write([]byte("HelloWorld"))  // 向客户端写入"HelloWorld"
    }

    wsConfig, err := websocket.NewConfig("wss://127.0.0.1/websocket", "https://127.0.0.1")  // 创建websocket配置
    common.Must(err)  // 检查错误
    wsServer := websocket.Server{  // 创建websocket服务器
        Config: *wsConfig,  // 设置配置
        Handler: func(conn *websocket.Conn) {  // 定义处理函数
            conn.Write([]byte("HelloWorld"))  // 向客户端写入"HelloWorld"
        },
        Handshake: func(wsConfig *websocket.Config, httpRequest *http.Request) error {  // 定义握手函数
            log.Debug("websocket url", httpRequest.URL, "origin", httpRequest.Header.Get("Origin"))  // 记录调试信息
            return nil  // 返回空错误
        },
    }
    mux := &http.ServeMux{}  // 创建HTTP多路复用器
    mux.HandleFunc("/", httpHello)  // 注册HTTP处理函数
    mux.HandleFunc("/websocket", wsServer.ServeHTTP)  // 注册websocket处理函数
    HTTPAddr = GetTestAddr()  // 获取测试地址
    _, HTTPPort, _ = net.SplitHostPort(HTTPAddr)  // 分割地址和端口
    server := http.Server{  // 创建HTTP服务器
        Addr:    HTTPAddr,  // 设置地址
        Handler: mux,  // 设置处理器
    }
    go server.ListenAndServe()  // 启动HTTP服务器
    time.Sleep(time.Second * 1)  // 等待1秒
    fmt.Println("http test server listening on", HTTPAddr)  // 打印信息
    wg.Done()  // 完成等待
}

var (
    EchoAddr string  // 定义Echo地址变量
    EchoPort int  // 定义Echo端口变量
)

func runTCPEchoServer() {
    listener, err := net.Listen("tcp", EchoAddr)  // 监听TCP连接
    common.Must(err)  // 检查错误
    wg.Done()  // 完成等待
}
    # 创建一个匿名的 goroutine
    go func() {
        # 延迟关闭监听器，确保在函数返回时关闭
        defer listener.Close()
        # 无限循环，接受连接
        for {
            # 接受连接
            conn, err := listener.Accept()
            # 如果出现错误，返回
            if err != nil {
                return
            }
            # 创建一个匿名的 goroutine 处理连接
            go func(conn net.Conn) {
                # 延迟关闭连接，确保在函数返回时关闭
                defer conn.Close()
                # 无限循环，读取数据
                for {
                    # 创建一个 2048 字节的缓冲区
                    buf := make([]byte, 2048)
                    # 设置连接的读取超时时间为 5 秒
                    conn.SetDeadline(time.Now().Add(time.Second * 5))
                    # 读取数据到缓冲区
                    n, err := conn.Read(buf)
                    # 重置连接的读取超时时间
                    conn.SetDeadline(time.Time{})
                    # 如果出现错误，返回
                    if err != nil {
                        return
                    }
                    # 将读取到的数据写回到连接
                    _, err = conn.Write(buf[0:n])
                    # 如果出现错误，返回
                    if err != nil {
                        return
                    }
                }
            }(conn)
        }
    }()
}

func runUDPEchoServer() {
    // 监听 UDP 连接
    conn, err := net.ListenPacket("udp", EchoAddr)
    // 检查错误
    common.Must(err)
    // 减少等待组计数
    wg.Done()
    // 启动 goroutine
    go func() {
        for {
            // 创建缓冲区
            buf := make([]byte, 1024*8)
            // 从连接中读取数据
            n, addr, err := conn.ReadFrom(buf)
            // 检查错误
            if err != nil {
                return
            }
            // 记录日志
            log.Info("Echo from", addr)
            // 将数据写回到地址
            conn.WriteTo(buf[0:n], addr)
        }
    }()
}

func GeneratePayload(length int) []byte {
    // 创建指定长度的字节数组
    buf := make([]byte, length)
    // 从随机源中读取数据填充到字节数组
    io.ReadFull(rand.Reader, buf)
    // 返回字节数组
    return buf
}

var (
    BlackHoleAddr string
    BlackHolePort int
)

func runTCPBlackHoleServer() {
    // 监听 TCP 连接
    listener, err := net.Listen("tcp", BlackHoleAddr)
    // 检查错误
    common.Must(err)
    // 减少等待组计数
    wg.Done()
    // 启动 goroutine
    go func() {
        // 延迟关闭监听器
        defer listener.Close()
        for {
            // 接受连接
            conn, err := listener.Accept()
            // 检查错误
            if err != nil {
                return
            }
            // 启动 goroutine
            go func(conn net.Conn) {
                // 将所有数据拷贝到 ioutil.Discard，然后关闭连接
                io.Copy(ioutil.Discard, conn)
                conn.Close()
            }(conn)
        }
    }()
}

func runUDPBlackHoleServer() {
    // 监听 UDP 连接
    conn, err := net.ListenPacket("udp", BlackHoleAddr)
    // 检查错误
    common.Must(err)
    // 减少等待组计数
    wg.Done()
    // 启动 goroutine
    go func() {
        // 延迟关闭连接
        defer conn.Close()
        // 创建缓冲区
        buf := make([]byte, 1024*8)
        for {
            // 从连接中读取数据
            _, _, err := conn.ReadFrom(buf)
            // 检查错误
            if err != nil {
                return
            }
        }
    }()
}

var wg = sync.WaitGroup{}

func init() {
    // 增加等待组计数
    wg.Add(5)
    // 运行 HTTP 服务器
    runHelloHTTPServer()

    // 选择可用的端口并设置地址
    EchoPort = common.PickPort("tcp", "127.0.0.1")
    EchoAddr = fmt.Sprintf("127.0.0.1:%d", EchoPort)

    // 选择可用的端口并设置地址
    BlackHolePort = common.PickPort("tcp", "127.0.0.1")
    BlackHoleAddr = fmt.Sprintf("127.0.0.1:%d", BlackHolePort)

    // 运行 TCP 回显服务器
    runTCPEchoServer()
    // 运行 UDP 回显服务器
    runUDPEchoServer()

    // 运行 TCP 黑洞服务器
    runTCPBlackHoleServer()
    // 运行 UDP 黑洞服务器
    runUDPBlackHoleServer()

    // 等待所有 goroutine 完成
    wg.Wait()
}
```