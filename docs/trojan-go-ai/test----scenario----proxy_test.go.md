# `trojan-go\test\scenario\proxy_test.go`

```go
package scenario

import (
    "bytes"  // 导入 bytes 包，提供对字节切片的操作
    "fmt"  // 导入 fmt 包，提供格式化 I/O 的函数
    "net"  // 导入 net 包，提供对网络和 socket 编程的支持
    "net/http"  // 导入 net/http 包，提供 HTTP 客户端和服务端的实现
    _ "net/http/pprof"  // 匿名导入 net/http/pprof 包，用于性能分析
    "os"  // 导入 os 包，提供对操作系统功能的访问
    "sync"  // 导入 sync 包，提供并发编程的支持
    "testing"  // 导入 testing 包，提供单元测试的支持
    "time"  // 导入 time 包，提供时间和定时器的功能

    netproxy "golang.org/x/net/proxy"  // 导入 golang.org/x/net/proxy 包，提供代理支持

    _ "github.com/p4gefau1t/trojan-go/api"  // 匿名导入 github.com/p4gefau1t/trojan-go/api 包
    _ "github.com/p4gefau1t/trojan-go/api/service"  // 匿名导入 github.com/p4gefau1t/trojan-go/api/service 包
    "github.com/p4gefau1t/trojan-go/common"  // 导入 github.com/p4gefau1t/trojan-go/common 包
    _ "github.com/p4gefau1t/trojan-go/log/golog"  // 匿名导入 github.com/p4gefau1t/trojan-go/log/golog 包
    "github.com/p4gefau1t/trojan-go/proxy"  // 导入 github.com/p4gefau1t/trojan-go/proxy 包
    _ "github.com/p4gefau1t/trojan-go/proxy/client"  // 匿名导入 github.com/p4gefau1t/trojan-go/proxy/client 包
    _ "github.com/p4gefau1t/trojan-go/proxy/forward"  // 匿名导入 github.com/p4gefau1t/trojan-go/proxy/forward 包
    _ "github.com/p4gefau1t/trojan-go/proxy/nat"  // 匿名导入 github.com/p4gefau1t/trojan-go/proxy/nat 包
    _ "github.com/p4gefau1t/trojan-go/proxy/server"  // 匿名导入 github.com/p4gefau1t/trojan-go/proxy/server 包
    _ "github.com/p4gefau1t/trojan-go/statistic/memory"  // 匿名导入 github.com/p4gefau1t/trojan-go/statistic/memory 包
    "github.com/p4gefau1t/trojan-go/test/util"  // 导入 github.com/p4gefau1t/trojan-go/test/util 包
)

// test key and cert

var cert = `
-----BEGIN CERTIFICATE-----
MIIC5TCCAc2gAwIBAgIJAJqNVe6g/10vMA0GCSqGSIb3DQEBCwUAMBQxEjAQBgNV
BAMMCWxvY2FsaG9zdDAeFw0yMTA5MTQwNjE1MTFaFw0yNjA5MTMwNjE1MTFaMBQx
EjAQBgNVBAMMCWxvY2FsaG9zdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoC
ggEBAK7bupJ8tmHM3shQ/7N730jzpRsXdNiBxq/Jxx8j+vB3AcxuP5bjXQZqS6YR
5W5vrfLlegtq1E/mmaI3Ht0RfIlzev04Dua9PWmIQJD801nEPknbfgCLXDh+pYr2
sfg8mUh3LjGtrxyH+nmbTjWg7iWSKohmZ8nUDcX94Llo5FxibMAz8OsAwOmUueCH
jP3XswZYHEy+OOP3K0ZEiJy0f5T6ZXk9OWYuPN4VQKJx1qrc9KzZtSPHwqVdkGUi
ase9tOPA4aMutzt0btgW7h7UrvG6C1c/Rr1BxdiYq1EQ+yypnAlyToVQSNbo67zz
wGQk4GeruIkOgJOLdooN/HjhbHMCAwEAAaM6MDgwFAYDVR0RBA0wC4IJbG9jYWxo
b3N0MAsGA1UdDwQEAwIHgDATBgNVHSUEDDAKBggrBgEFBQcDATANBgkqhkiG9w0B
AQsFAAOCAQEASsBzHHYiWDDiBVWUEwVZAduTrslTLNOxG0QHBKsHWIlz/3QlhQil
ywb3OhfMTUR1dMGY5Iq5432QiCHO4IMCOv7tDIkgb4Bc3v/3CRlBlnurtAmUfNJ6
pTRSlK4AjWpGHAEEd/8aCaOE86hMP8WDht8MkJTRrQqpJ1HeDISoKt9nepHOIsj+
I2zLZZtw0pg7FuR4MzWuqOt071iRS46Pupryb3ZEGIWNz5iLrDQod5Iz2ZGSRGqE
rB8idX0mlj5AHRRanVR3PAes+eApsW9JvYG/ImuCOs+ZsukY614zQZdR+SyFm85G
4NICyeQsmiypNHHgw+xZmGqZg65bXNGoyg==
-----END CERTIFICATE-----
`  // 证书内容

var key = `
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCu27qSfLZhzN7I
UP+ze99I86UbF3TYgcavyccfI/rwdwHMbj+W410GakumEeVub63y5XoLatRP5pmi
```  // 私钥内容
# 初始化函数，用于写入服务器证书和私钥文件
func init() {
    # 将证书内容写入 server.crt 文件，权限设置为 777
    os.WriteFile("server.crt", []byte(cert), 0o777)
    # 将私钥内容写入 server.key 文件，权限设置为 777
    os.WriteFile("server.key", []byte(key), 0o777)
}

# 检查客户端和服务器的连接
# clientData: 客户端数据
# serverData: 服务器数据
# socksPort: SOCKS 端口号
# 返回值：连接是否正常的布尔值
func CheckClientServer(clientData, serverData string, socksPort int) (ok bool) {
    # 从服务器数据创建代理对象
    server, err := proxy.NewProxyFromConfigData([]byte(serverData), false)
    # 检查错误
    common.Must(err)
    # 启动服务器代理
    go server.Run()

    # 从客户端数据创建代理对象
    client, err := proxy.NewProxyFromConfigData([]byte(clientData), false)
    # 检查错误
    common.Must(err)
    # 启动客户端代理
    go client.Run()
    // 休眠2秒
    time.Sleep(time.Second * 2)
    // 使用 SOCKS5 协议建立代理连接
    dialer, err := netproxy.SOCKS5("tcp", fmt.Sprintf("127.0.0.1:%d", socksPort), nil, netproxy.Direct)
    // 检查错误
    common.Must(err)

    // 设置标志为 true
    ok = true
    // 定义常量 num 为 100
    const num = 100
    // 创建一个等待组
    wg := sync.WaitGroup{}
    // 将等待组的计数器设置为 num
    wg.Add(num)
    // 循环 num 次
    for i := 0; i < num; i++ {
        // 启动一个 goroutine
        go func() {
            // 定义常量 payloadSize 为 1024
            const payloadSize = 1024
            // 生成指定大小的随机数据
            payload := util.GeneratePayload(payloadSize)
            // 创建一个大小为 payloadSize 的字节数组
            buf := [payloadSize]byte{}

            // 通过代理连接到指定的地址
            conn, err := dialer.Dial("tcp", util.EchoAddr)
            // 检查错误
            common.Must(err)

            // 将数据写入连接
            common.Must2(conn.Write(payload))
            // 从连接中读取数据到 buf
            common.Must2(conn.Read(buf[:]))

            // 检查发送和接收的数据是否一致，如果不一致则将标志设置为 false
            if !bytes.Equal(payload, buf[:]) {
                ok = false
            }
            // 关闭连接
            conn.Close()
            // 减少等待组的计数器
            wg.Done()
        }()
    }
    // 等待所有 goroutine 完成
    wg.Wait()
    // 关闭 client 连接
    client.Close()
    // 关闭 server 连接
    server.Close()
    // 返回
    return
}
# 定义测试函数，测试客户端和服务器之间的WebSocket子树
func TestClientServerWebsocketSubTree(t *testing.T) {
    # 为服务器选择一个端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    # 为Socks代理选择一个端口
    socksPort := common.PickPort("tcp", "127.0.0.1")
    # 客户端配置数据
    clientData := fmt.Sprintf(`
    run-type: client
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %d
    password:
        - password
    ssl:
        verify: false
        fingerprint: firefox
        sni: localhost
    websocket:
        enabled: true
        path: /ws
        host: somedomainname.com
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    mux:
        enabled: true
    `, socksPort, serverPort)
    # 服务器配置数据
    serverData := fmt.Sprintf(`
    run-type: server
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %s
    disable-http-check: true
    password:
        - password
    ssl:
        verify-hostname: false
        key: server.key
        cert: server.crt
        sni: localhost
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    websocket:
        enabled: true
        path: /ws
        host: 127.0.0.1
    `, serverPort, util.HTTPPort)

    # 检查客户端和服务器之间的连接是否正常
    if !CheckClientServer(clientData, serverData, socksPort) {
        t.Fail()
    }
}

# 定义测试函数，测试客户端和服务器之间的Trojan子树
func TestClientServerTrojanSubTree(t *testing.T) {
    # 为服务器选择一个端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    # 为Socks代理选择一个端口
    socksPort := common.PickPort("tcp", "127.0.0.1")
    # 客户端配置数据
    clientData := fmt.Sprintf(`
    run-type: client
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %d
    password:
        - password
    ssl:
        verify: false
        fingerprint: firefox
        sni: localhost
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    mux:
        enabled: true
    `, socksPort, serverPort)
    # 服务器配置数据
    serverData := fmt.Sprintf(`
    run-type: server
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %s
    disable-http-check: true
    password:
        - password
    ssl:
        verify-hostname: false
        key: server.key
        cert: server.crt
        sni: localhost
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    ```go
// 定义测试函数，用于检测客户端和服务器之间的通信是否正常
func TestWebsocketDetection(t *testing.T) {
    // 为服务器和客户端分配端口号
    serverPort := common.PickPort("tcp", "127.0.0.1")
    socksPort := common.PickPort("tcp", "127.0.0.1")

    // 为客户端生成配置数据
    clientData := fmt.Sprintf(`
    run-type: client
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %d
    password:
        - password
    ssl:
        verify: false
        fingerprint: firefox
        sni: localhost
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    mux:
        enabled: true
    `, socksPort, serverPort)
    
    // 为服务器生成配置数据
    serverData := fmt.Sprintf(`
    run-type: server
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %s
    disable-http-check: true
    password:
        - password
    ssl:
        verify-hostname: false
        key: server.key
        cert: server.crt
        sni: localhost
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    websocket:
        enabled: true
        path: /ws
        hostname: 127.0.0.1
    `, serverPort, util.HTTPPort)

    // 检查客户端和服务器之间的通信是否正常
    if !CheckClientServer(clientData, serverData, socksPort) {
        t.Fail()
    }
}
    # 使用的加密方法是AEAD_CHACHA20_POLY1305
    method: AEAD_CHACHA20_POLY1305
    # 使用的密码是12345678
    password: 12345678
func TestForward(t *testing.T) {
    // 为服务器和客户端分配端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    clientPort := common.PickPort("tcp", "127.0.0.1")
    // 从util.EchoAddr中分割出目标端口
    _, targetPort, _ := net.SplitHostPort(util.EchoAddr)
    // 生成客户端配置数据
    clientData := fmt.Sprintf(`
    run-type: forward
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %d
    target-addr: 127.0.0.1
    target-port: %s
    password:
        - password
    ssl:
        verify: false
        fingerprint: firefox
        sni: localhost
    websocket:
        enabled: true
        path: /ws
        hostname: 127.0.0.1
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    mux:
        enabled: true
    `, clientPort, serverPort, targetPort)
    // 启动客户端代理
    go func() {
        proxy, err := proxy.NewProxyFromConfigData([]byte(clientData), false)
        common.Must(err)
        common.Must(proxy.Run())
    }()

    // 生成服务器配置数据
    serverData := fmt.Sprintf(`
    run-type: server
    local-addr: 127.0.0.1
    local-port: %d
    remote-addr: 127.0.0.1
    remote-port: %s
    disable-http-check: true
    password:
        - password
    ssl:
        verify-hostname: false
        key: server.key
        cert: server.crt
        sni: "localhost"
    websocket:
        enabled: true
        path: /ws
        hostname: 127.0.0.1
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    `, serverPort, util.HTTPPort)
    // 启动服务器代理
    go func() {
        proxy, err := proxy.NewProxyFromConfigData([]byte(serverData), false)
        common.Must(err)
        common.Must(proxy.Run())
    }()

    // 等待2秒
    time.Sleep(time.Second * 2)

    // 生成payload并初始化缓冲区
    payload := util.GeneratePayload(1024)
    buf := [1024]byte{}

    // 连接到客户端端口
    conn, err := net.Dial("tcp", fmt.Sprintf("127.0.0.1:%d", clientPort))
    common.Must(err)

    // 写入payload并读取响应
    common.Must2(conn.Write(payload))
    common.Must2(conn.Read(buf[:]))

    // 检查payload和响应是否相等
    if !bytes.Equal(payload, buf[:]) {
        t.Fail()
    }
}
    // 使用UDP协议监听数据包
    packet, err := net.ListenPacket("udp", "")
    // 检查错误
    common.Must(err)
    // 将数据包写入指定地址
    common.Must2(packet.WriteTo(payload, &net.UDPAddr{
        IP:   net.ParseIP("127.0.0.1"),
        Port: clientPort,
    }))
    // 从数据包中读取数据并存入buf中
    _, _, err = packet.ReadFrom(buf[:])
    // 检查错误
    common.Must(err)
    // 检查payload和buf中的数据是否相等，如果不相等则测试失败
    if !bytes.Equal(payload, buf[:]) {
        t.Fail()
    }
}
// 测试内存泄漏
func TestLeak(t *testing.T) {
    // 选择一个可用的端口作为服务器端口
    serverPort := common.PickPort("tcp", "127.0.0.1")
    // 选择一个可用的端口作为 SOCKS 代理端口
    socksPort := common.PickPort("tcp", "127.0.0.1")
    // 生成客户端配置数据
    clientData := fmt.Sprintf(`
    // 设置客户端运行类型为 client
    run-type: client
    // 设置本地地址
    local-addr: 127.0.0.1
    // 设置本地端口
    local-port: %d
    // 设置远程地址
    remote-addr: 127.0.0.1
    // 设置远程端口
    remote-port: %d
    // 设置日志级别
    log-level: 0
    // 设置密码
    password:
        - password
    // 设置 SSL 配置
    ssl:
        verify: false
        fingerprint: firefox
        sni: localhost
    // 设置 Shadowsocks 配置
    shadowsocks:
        enabled: true
        method: AEAD_CHACHA20_POLY1305
        password: 12345678
    // 设置 Mux 配置
    mux:
        enabled: true
    // 设置 API 配置
    api:
        enabled: true
        api-port: 0
    `, socksPort, serverPort)
    // 根据客户端配置数据创建代理客户端
    client, err := proxy.NewProxyFromConfigData([]byte(clientData), false)
    common.Must(err)
    // 启动客户端
    go client.Run()
    // 等待 3 秒
    time.Sleep(time.Second * 3)
    // 关闭客户端
    client.Close()
    // 再次等待 3 秒
    time.Sleep(time.Second * 3)
    // 启动性能分析服务器
    // http.ListenAndServe("localhost:6060", nil)
}

// 单线程基准测试
func SingleThreadBenchmark(clientData, serverData string, socksPort int) {
    // 根据服务器配置数据创建代理服务器
    server, err := proxy.NewProxyFromConfigData([]byte(clientData), false)
    common.Must(err)
    // 启动服务器
    go server.Run()

    // 根据客户端配置数据创建代理客户端
    client, err := proxy.NewProxyFromConfigData([]byte(serverData), false)
    common.Must(err)
    // 启动客户端
    go client.Run()

    // 等待 2 秒
    time.Sleep(time.Second * 2)
    // 创建 SOCKS5 代理拨号器
    dialer, err := netproxy.SOCKS5("tcp", fmt.Sprintf("127.0.0.1:%d", socksPort), nil, netproxy.Direct)
    common.Must(err)

    // 设置循环次数
    const num = 100
    // 创建等待组
    wg := sync.WaitGroup{}
    wg.Add(num)
    // 设置负载大小
    const payloadSize = 1024 * 1024 * 1024
    // 生成负载数据
    payload := util.GeneratePayload(payloadSize)

    for i := 0; i < 100; i++ {
        // 通过拨号器连接到黑洞地址
        conn, err := dialer.Dial("tcp", util.BlackHoleAddr)
        common.Must(err)

        // 记录开始时间
        t1 := time.Now()
        // 写入负载数据
        common.Must2(conn.Write(payload))
        // 记录结束时间
        t2 := time.Now()

        // 计算传输速度
        speed := float64(payloadSize) / (float64(t2.Sub(t1).Nanoseconds()) / float64(time.Second))
        fmt.Printf("speed: %f Gbps\n", speed/1024/1024/1024)

        // 关闭连接
        conn.Close()
    }
    // 关闭客户端
    client.Close()
    // 关闭服务器
    server.Close()
}

// 客户端服务器基准测试
func BenchmarkClientServer(b *testing.B) {
    // 启动性能分析服务器
    go func() {
        fmt.Println(http.ListenAndServe("localhost:6060", nil))
    }()
}
    # 为服务器选择一个可用的端口号
    serverPort := common.PickPort("tcp", "127.0.0.1")
    # 为 SOCKS 代理选择一个可用的端口号
    socksPort := common.PickPort("tcp", "127.0.0.1")
    # 格式化客户端数据
    clientData := fmt.Sprintf(`
# 设置运行类型为客户端
run-type: client
# 设置本地地址为 127.0.0.1
local-addr: 127.0.0.1
# 设置本地端口为变量 %d
local-port: %d
# 设置远程地址为 127.0.0.1
remote-addr: 127.0.0.1
# 设置远程端口为变量 %d
remote-port: %d
# 设置日志级别为 0
log-level: 0
# 设置密码为空
password:
    - password
# 设置 SSL 配置，关闭验证，使用指纹为 firefox，SNI 为 localhost
ssl:
    verify: false
    fingerprint: firefox
    sni: localhost
`, socksPort, serverPort)

# 生成服务器数据配置
serverData := fmt.Sprintf(`
# 设置运行类型为服务器
run-type: server
# 设置本地地址为 127.0.0.1
local-addr: 127.0.0.1
# 设置本地端口为变量 %d
local-port: %d
# 设置远程地址为 127.0.0.1
remote-addr: 127.0.0.1
# 设置远程端口为变量 %s
remote-port: %s
# 设置日志级别为 0
log-level: 0
# 禁用 HTTP 检查
disable-http-check: true
# 设置密码为空
password:
    - password
# 设置 SSL 配置，关闭主机名验证，使用服务器密钥为 server.key，证书为 server.crt，SNI 为 localhost
ssl:
    verify-hostname: false
    key: server.key
    cert: server.crt
    sni: localhost
`, serverPort, util.HTTPPort)

# 运行单线程基准测试，传入客户端数据配置、服务器数据配置和 SOCKS 端口
SingleThreadBenchmark(clientData, serverData, socksPort)
```