# `v2ray-core\transport\internet\http\dialer.go`

```
// +build !confonly

package http

import (
    "context"
    gotls "crypto/tls"
    "net/http"
    "net/url"
    "sync"

    "golang.org/x/net/http2"
    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
    "v2ray.com/core/transport/pipe"
)

var (
    globalDialerMap    map[net.Destination]*http.Client  // 全局变量，存储目的地到 HTTP 客户端的映射
    globalDialerAccess sync.Mutex  // 全局互斥锁，用于保护全局变量的并发访问
)

func getHTTPClient(ctx context.Context, dest net.Destination, tlsSettings *tls.Config) (*http.Client, error) {
    globalDialerAccess.Lock()  // 获取全局互斥锁
    defer globalDialerAccess.Unlock()  // 在函数返回前释放全局互斥锁

    if globalDialerMap == nil {  // 如果全局映射为空
        globalDialerMap = make(map[net.Destination]*http.Client)  // 创建一个新的映射
    }

    if client, found := globalDialerMap[dest]; found {  // 如果目的地对应的客户端已存在
        return client, nil  // 直接返回该客户端
    }
    // 创建一个 HTTP/2 传输对象
    transport := &http2.Transport{
        // 自定义 DialTLS 函数，用于建立 TLS 连接
        DialTLS: func(network string, addr string, tlsConfig *gotls.Config) (net.Conn, error) {
            // 解析地址和端口
            rawHost, rawPort, err := net.SplitHostPort(addr)
            if err != nil {
                return nil, err
            }
            // 如果端口为空，则默认为 443
            if len(rawPort) == 0 {
                rawPort = "443"
            }
            // 解析端口
            port, err := net.PortFromString(rawPort)
            if err != nil {
                return nil, err
            }
            // 解析地址
            address := net.ParseAddress(rawHost)

            // 使用系统默认的网络拨号器建立连接
            pconn, err := internet.DialSystem(context.Background(), net.TCPDestination(address, port), nil)
            if err != nil {
                return nil, err
            }

            // 使用 TLS 配置创建 TLS 客户端
            cn := gotls.Client(pconn, tlsConfig)
            // 进行 TLS 握手
            if err := cn.Handshake(); err != nil {
                return nil, err
            }
            // 如果不跳过证书验证
            if !tlsConfig.InsecureSkipVerify {
                // 验证服务器的主机名
                if err := cn.VerifyHostname(tlsConfig.ServerName); err != nil {
                    return nil, err
                }
            }
            // 获取连接状态
            state := cn.ConnectionState()
            // 检查协商的协议是否为 HTTP/2
            if p := state.NegotiatedProtocol; p != http2.NextProtoTLS {
                return nil, newError("http2: unexpected ALPN protocol " + p + "; want q" + http2.NextProtoTLS).AtError()
            }
            // 检查是否协商的协议是双向的
            if !state.NegotiatedProtocolIsMutual {
                return nil, newError("http2: could not negotiate protocol mutually").AtError()
            }
            // 返回 TLS 连接
            return cn, nil
        },
        // 设置 TLS 客户端配置
        TLSClientConfig: tlsSettings.GetTLSConfig(tls.WithDestination(dest)),
    }

    // 创建一个 HTTP 客户端
    client := &http.Client{
        Transport: transport,
    }

    // 将客户端与目标地址关联存储到全局映射中
    globalDialerMap[dest] = client
    // 返回创建的客户端
    return client, nil
// Dial函数拨号到给定目的地的新TCP连接。
func Dial(ctx context.Context, dest net.Destination, streamSettings *internet.MemoryStreamConfig) (internet.Connection, error) {
    // 从流设置中获取HTTP设置
    httpSettings := streamSettings.ProtocolSettings.(*Config)
    // 从流设置中获取TLS配置
    tlsConfig := tls.ConfigFromStreamSettings(streamSettings)
    // 如果TLS配置为空，则返回错误
    if tlsConfig == nil {
        return nil, newError("TLS must be enabled for http transport.").AtWarning()
    }
    // 获取HTTP客户端
    client, err := getHTTPClient(ctx, dest, tlsConfig)
    if err != nil {
        return nil, err
    }

    // 从上下文中获取选项
    opts := pipe.OptionsFromContext(ctx)
    // 创建新的管道读写器
    preader, pwriter := pipe.New(opts...)
    // 创建缓冲读取器
    breader := &buf.BufferedReader{Reader: preader}
    // 创建HTTP请求
    request := &http.Request{
        Method: "PUT",
        Host:   httpSettings.getRandomHost(),
        Body:   breader,
        URL: &url.URL{
            Scheme: "https",
            Host:   dest.NetAddr(),
            Path:   httpSettings.getNormalizedPath(),
        },
        Proto:      "HTTP/2",
        ProtoMajor: 2,
        ProtoMinor: 0,
        Header:     make(http.Header),
    }
    // 禁用服务器的任何压缩方法
    request.Header.Set("Accept-Encoding", "identity")

    // 发送HTTP请求并获取响应
    response, err := client.Do(request)
    if err != nil {
        return nil, newError("failed to dial to ", dest).Base(err).AtWarning()
    }
    // 如果响应状态码不是200，则返回错误
    if response.StatusCode != 200 {
        return nil, newError("unexpected status", response.StatusCode).AtWarning()
    }

    // 创建新的连接并返回
    bwriter := buf.NewBufferedWriter(pwriter)
    common.Must(bwriter.SetBuffered(false))
    return net.NewConnection(
        net.ConnectionOutput(response.Body),
        net.ConnectionInput(bwriter),
        net.ConnectionOnClose(common.ChainedClosable{breader, bwriter, response.Body}),
    ), nil
}

// 初始化函数
func init() {
    // 注册传输拨号器
    common.Must(internet.RegisterTransportDialer(protocolName, Dial))
}
```