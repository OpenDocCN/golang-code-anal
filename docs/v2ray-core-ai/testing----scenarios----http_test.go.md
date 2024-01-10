# `v2ray-core\testing\scenarios\http_test.go`

```
package scenarios

import (
    "bytes"  // 导入 bytes 包，用于操作字节
    "crypto/rand"  // 导入 crypto/rand 包，用于生成加密安全的随机数
    "io"  // 导入 io 包，提供了基本的 I/O 接口
    "io/ioutil"  // 导入 ioutil 包，提供了一些实用的 I/O 函数
    "net/http"  // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "net/url"  // 导入 net/url 包，用于 URL 解析
    "testing"  // 导入 testing 包，用于编写测试函数
    "time"  // 导入 time 包，提供了时间的测量和显示功能

    "github.com/google/go-cmp/cmp"  // 导入 github.com/google/go-cmp/cmp 包，用于比较数据结构

    "v2ray.com/core"  // 导入 v2ray.com/core 包
    "v2ray.com/core/app/proxyman"  // 导入 v2ray.com/core/app/proxyman 包
    "v2ray.com/core/common"  // 导入 v2ray.com/core/common 包
    "v2ray.com/core/common/buf"  // 导入 v2ray.com/core/common/buf 包
    "v2ray.com/core/common/net"  // 导入 v2ray.com/core/common/net 包
    "v2ray.com/core/common/serial"  // 导入 v2ray.com/core/common/serial 包
    "v2ray.com/core/proxy/freedom"  // 导入 v2ray.com/core/proxy/freedom 包
    v2http "v2ray.com/core/proxy/http"  // 导入 v2ray.com/core/proxy/http 包，并命名为 v2http
    v2httptest "v2ray.com/core/testing/servers/http"  // 导入 v2ray.com/core/testing/servers/http 包，并命名为 v2httptest
    "v2ray.com/core/testing/servers/tcp"  // 导入 v2ray.com/core/testing/servers/tcp 包
)

func TestHttpConformance(t *testing.T) {
    httpServerPort := tcp.PickPort()  // 选择一个 TCP 端口作为 HTTP 服务器端口
    httpServer := &v2httptest.Server{  // 创建一个 HTTP 服务器实例
        Port:        httpServerPort,  // 设置 HTTP 服务器的端口
        PathHandler: make(map[string]http.HandlerFunc),  // 创建一个空的路径处理器映射
    }
    _, err := httpServer.Start()  // 启动 HTTP 服务器
    common.Must(err)  // 如果出现错误，立即终止程序
    defer httpServer.Close()  // 在函数返回时关闭 HTTP 服务器

    serverPort := tcp.PickPort()  // 选择一个 TCP 端口作为服务器端口
    serverConfig := &core.Config{  // 创建一个 V2Ray 核心配置实例
        Inbound: []*core.InboundHandlerConfig{  // 设置入站处理器配置
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{  // 设置接收器配置
                    PortRange: net.SinglePortRange(serverPort),  // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),  // 设置监听地址
                }),
                ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),  // 设置代理配置
            },
        },
        Outbound: []*core.OutboundHandlerConfig{  // 设置出站处理器配置
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),  // 设置代理配置
            },
        },
    }

    servers, err := InitializeServerConfigs(serverConfig)  // 初始化服务器配置
    common.Must(err)  // 如果出现错误，立即终止程序
    defer CloseAllServers(servers)  // 在函数返回时关闭所有服务器
}
    {
        // 创建一个 HTTP 传输对象，设置代理函数，将请求重定向到本地服务器
        transport := &http.Transport{
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + serverPort.String())
            },
        }
    
        // 创建一个 HTTP 客户端对象，设置传输对象
        client := &http.Client{
            Transport: transport,
        }
    
        // 发起 GET 请求，获取响应
        resp, err := client.Get("http://127.0.0.1:" + httpServerPort.String())
        common.Must(err)
        // 检查响应状态码，如果不是 200 则终止测试
        if resp.StatusCode != 200 {
            t.Fatal("status: ", resp.StatusCode)
        }
    
        // 读取响应内容
        content, err := ioutil.ReadAll(resp.Body)
        common.Must(err)
        // 检查响应内容，如果不是 "Home" 则终止测试
        if string(content) != "Home" {
            t.Fatal("body: ", string(content))
        }
    }
func TestHttpError(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        // 定义消息处理函数，返回空字节切片
        MsgProcessor: func(msg []byte) []byte {
            return []byte{}
        },
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 在 2 秒后执行一个函数，将 TCP 服务器的 ShouldClose 属性设置为 true
    time.AfterFunc(time.Second*2, func() {
        tcpServer.ShouldClose = true
    })

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 初始化服务器配置并返回服务器对象和可能的错误
    servers, err := InitializeServerConfigs(serverConfig)
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)

    {
        // 创建一个 HTTP 传输对象
        transport := &http.Transport{
            // 设置代理函数，返回目标 URL 和可能的错误
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + serverPort.String())
            },
        }

        // 创建一个 HTTP 客户端对象
        client := &http.Client{
            // 设置传输对象
            Transport: transport,
        }

        // 发起 GET 请求并返回响应和可能的错误
        resp, err := client.Get("http://127.0.0.1:" + dest.Port.String())
        // 如果有错误发生，则必须处理
        common.Must(err)
        // 如果响应状态码不是 503，则输出错误信息
        if resp.StatusCode != 503 {
            t.Error("status: ", resp.StatusCode)
        }
    }
}

func TestHttpConnectMethod(t *testing.T) {
    // 创建一个 TCP 服务器对象
    tcpServer := tcp.Server{
        // 定义消息处理函数为 xor
        MsgProcessor: xor,
    }
    // 启动 TCP 服务器并返回目标地址和可能的错误
    dest, err := tcpServer.Start()
    // 如果有错误发生，则必须处理
    common.Must(err)
    // 延迟关闭 TCP 服务器
    defer tcpServer.Close()

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort();
    // 创建服务器配置对象
    serverConfig := &core.Config{
        // 设置入站处理器配置
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort), // 设置端口范围
                    Listen:    net.NewIPOrDomain(net.LocalHostIP), // 设置监听地址
                }),
                ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}), // 设置代理配置
            },
        },
        // 设置出站处理器配置
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}), // 设置代理配置
            },
        },
    }

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err) // 检查错误
    defer CloseAllServers(servers) // 延迟关闭所有服务器

    {
        // 创建 HTTP 传输对象
        transport := &http.Transport{
            // 设置代理函数
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + serverPort.String()) // 解析代理地址
            },
        }

        // 创建 HTTP 客户端
        client := &http.Client{
            Transport: transport, // 设置传输对象
        }

        // 创建数据载荷
        payload := make([]byte, 1024*64)
        common.Must2(rand.Read(payload)) // 生成随机数据
        req, err := http.NewRequest("Connect", "http://"+dest.NetAddr()+"/", bytes.NewReader(payload)) // 创建 HTTP 请求
        req.Header.Set("X-a", "b") // 设置请求头
        req.Header.Set("X-b", "d") // 设置请求头
        common.Must(err) // 检查错误

        // 发起 HTTP 请求
        resp, err := client.Do(req)
        common.Must(err) // 检查错误
        if resp.StatusCode != 200 {
            t.Fatal("status: ", resp.StatusCode) // 如果状态码不是 200，则输出错误信息
        }

        // 读取响应内容
        content := make([]byte, len(payload))
        common.Must2(io.ReadFull(resp.Body, content)) // 读取响应内容
        if r := cmp.Diff(content, xor(payload)); r != "" {
            t.Fatal(r) // 如果内容不符合预期，则输出错误信息
        }
    }
func TestHttpPost(t *testing.T) {
    // 选择一个可用的端口作为 HTTP 服务器的端口
    httpServerPort := tcp.PickPort()
    // 创建一个 HTTP 服务器对象
    httpServer := &v2httptest.Server{
        Port: httpServerPort,
        // 设置路径和对应的处理函数
        PathHandler: map[string]http.HandlerFunc{
            "/testpost": func(w http.ResponseWriter, r *http.Request) {
                // 读取请求体的数据并关闭请求体
                payload, err := buf.ReadAllToBytes(r.Body)
                r.Body.Close()

                // 如果读取出错，返回 500 状态码和错误信息
                if err != nil {
                    w.WriteHeader(500)
                    w.Write([]byte("Unable to read all payload"))
                    return
                }
                // 对 payload 进行异或操作
                payload = xor(payload)
                // 返回处理后的 payload
                w.Write(payload)
            },
        },
    }

    // 启动 HTTP 服务器
    _, err := httpServer.Start()
    common.Must(err)
    // 延迟关闭 HTTP 服务器
    defer httpServer.Close()

    // 选择一个可用的端口作为服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                // 设置接收器的配置
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                // 设置代理的配置
                ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{}),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                // 设置代理的配置
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)
}
    {
        // 创建一个 HTTP 传输对象，设置代理函数，将请求重定向到本地服务器
        transport := &http.Transport{
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + serverPort.String())
            },
        }
    
        // 创建一个 HTTP 客户端，设置传输对象
        client := &http.Client{
            Transport: transport,
        }
    
        // 创建一个 64KB 大小的字节数组，用于存储随机生成的数据
        payload := make([]byte, 1024*64)
        // 生成随机数据并存储到 payload 中
        common.Must2(rand.Read(payload))
    
        // 向指定 URL 发送 POST 请求，携带随机数据
        resp, err := client.Post("http://127.0.0.1:"+httpServerPort.String()+"/testpost", "application/x-www-form-urlencoded", bytes.NewReader(payload))
        // 检查是否有错误发生
        common.Must(err)
        // 检查响应状态码是否为 200
        if resp.StatusCode != 200 {
            t.Fatal("status: ", resp.StatusCode)
        }
    
        // 读取响应内容
        content, err := ioutil.ReadAll(resp.Body)
        // 检查是否有错误发生
        common.Must(err)
        // 比较响应内容和随机数据的异或结果，如果不相等则输出差异信息
        if r := cmp.Diff(content, xor(payload)); r != "" {
            t.Fatal(r)
        }
    }
}


func setProxyBasicAuth(req *http.Request, user, pass string) {
    // 设置请求的基本认证信息
    req.SetBasicAuth(user, pass)
    // 设置请求头中的代理授权信息
    req.Header.Set("Proxy-Authorization", req.Header.Get("Authorization"))
    // 删除请求头中的认证信息
    req.Header.Del("Authorization")
}

func TestHttpBasicAuth(t *testing.T) {
    // 选择一个可用的 HTTP 服务器端口
    httpServerPort := tcp.PickPort()
    // 创建一个 HTTP 服务器对象
    httpServer := &v2httptest.Server{
        Port:        httpServerPort,
        PathHandler: make(map[string]http.HandlerFunc),
    }
    // 启动 HTTP 服务器
    _, err := httpServer.Start()
    common.Must(err)
    // 延迟关闭 HTTP 服务器
    defer httpServer.Close()

    // 选择一个可用的服务器端口
    serverPort := tcp.PickPort()
    // 创建服务器配置对象
    serverConfig := &core.Config{
        Inbound: []*core.InboundHandlerConfig{
            {
                ReceiverSettings: serial.ToTypedMessage(&proxyman.ReceiverConfig{
                    PortRange: net.SinglePortRange(serverPort),
                    Listen:    net.NewIPOrDomain(net.LocalHostIP),
                }),
                ProxySettings: serial.ToTypedMessage(&v2http.ServerConfig{
                    Accounts: map[string]string{
                        "a": "b",
                    },
                }),
            },
        },
        Outbound: []*core.OutboundHandlerConfig{
            {
                ProxySettings: serial.ToTypedMessage(&freedom.Config{}),
            },
        },
    }

    // 初始化服务器配置
    servers, err := InitializeServerConfigs(serverConfig)
    common.Must(err)
    // 延迟关闭所有服务器
    defer CloseAllServers(servers)
}
    {
        // 创建一个 HTTP 传输对象，设置代理函数，将请求重定向到本地服务器
        transport := &http.Transport{
            Proxy: func(req *http.Request) (*url.URL, error) {
                return url.Parse("http://127.0.0.1:" + serverPort.String())
            },
        }

        // 创建一个 HTTP 客户端，设置传输对象
        client := &http.Client{
            Transport: transport,
        }

        {
            // 发起 GET 请求到本地服务器，检查返回状态码是否为 407
            resp, err := client.Get("http://127.0.0.1:" + httpServerPort.String())
            common.Must(err)
            if resp.StatusCode != 407 {
                t.Fatal("status: ", resp.StatusCode)
            }
        }

        {
            // 创建一个 GET 请求到本地服务器，设置代理基本认证信息，发起请求，检查返回状态码是否为 407
            req, err := http.NewRequest("GET", "http://127.0.0.1:"+httpServerPort.String(), nil)
            common.Must(err)

            setProxyBasicAuth(req, "a", "c")
            resp, err := client.Do(req)
            common.Must(err)
            if resp.StatusCode != 407 {
                t.Fatal("status: ", resp.StatusCode)
            }
        }

        {
            // 创建一个 GET 请求到本地服务器，设置代理基本认证信息，发起请求，检查返回状态码是否为 200
            req, err := http.NewRequest("GET", "http://127.0.0.1:"+httpServerPort.String(), nil)
            common.Must(err)

            setProxyBasicAuth(req, "a", "b")
            resp, err := client.Do(req)
            common.Must(err)
            if resp.StatusCode != 200 {
                t.Fatal("status: ", resp.StatusCode)
            }

            // 读取响应内容，检查是否为 "Home"
            content, err := ioutil.ReadAll(resp.Body)
            common.Must(err)
            if string(content) != "Home" {
                t.Fatal("body: ", string(content))
            }
        }
    }
# 闭合前面的函数定义
```