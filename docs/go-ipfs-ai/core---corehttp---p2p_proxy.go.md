# `kubo\core\corehttp\p2p_proxy.go`

```
package corehttp

import (
    "fmt" // 导入 fmt 包，用于格式化输出
    "net" // 导入 net 包，用于网络操作
    "net/http" // 导入 net/http 包，用于 HTTP 协议操作
    "net/http/httputil" // 导入 net/http/httputil 包，用于 HTTP 请求和响应的转储
    "net/url" // 导入 net/url 包，用于 URL 操作
    "strings" // 导入 strings 包，用于字符串操作

    core "github.com/ipfs/kubo/core" // 导入 core 包，用于 IPFS 相关操作
    peer "github.com/libp2p/go-libp2p/core/peer" // 导入 peer 包，用于 libp2p 的 peer 操作

    p2phttp "github.com/libp2p/go-libp2p-http" // 导入 p2phttp 包，用于 libp2p 的 HTTP 操作
    protocol "github.com/libp2p/go-libp2p/core/protocol" // 导入 protocol 包，用于 libp2p 的协议操作
)

// P2PProxyOption is an endpoint for proxying a HTTP request to another ipfs peer
// P2PProxyOption 是将 HTTP 请求代理到另一个 IPFS peer 的端点
func P2PProxyOption() ServeOption {
    return func(ipfsNode *core.IpfsNode, _ net.Listener, mux *http.ServeMux) (*http.ServeMux, error) {
        mux.HandleFunc("/p2p/", func(w http.ResponseWriter, request *http.Request) {
            // parse request
            // 解析请求
            parsedRequest, err := parseRequest(request)
            if err != nil {
                handleError(w, "failed to parse request", err, 400) // 处理错误
                return
            }

            request.Host = "" // Let URL's Host take precedence.
            request.URL.Path = parsedRequest.httpPath
            target, err := url.Parse(fmt.Sprintf("libp2p://%s", parsedRequest.target)) // 解析目标 URL
            if err != nil {
                handleError(w, "failed to parse url", err, 400) // 处理错误
                return
            }

            rt := p2phttp.NewTransport(ipfsNode.PeerHost, p2phttp.ProtocolOption(parsedRequest.name)) // 创建 libp2p HTTP 传输
            proxy := httputil.NewSingleHostReverseProxy(target) // 创建反向代理
            proxy.Transport = rt
            proxy.ServeHTTP(w, request) // 代理 HTTP 请求
        })
        return mux, nil
    }
}

type proxyRequest struct {
    target   string
    name     protocol.ID
    httpPath string // path to send to the proxy-host
}

// from the url path parse the peer-ID, name and http path
// /p2p/$peer_id/http/$http_path
// or
// /p2p/$peer_id/x/$protocol/http/$http_path
// 从 URL 路径解析 peer-ID、name 和 http 路径
// /p2p/$peer_id/http/$http_path
// 或
// /p2p/$peer_id/x/$protocol/http/$http_path
func parseRequest(request *http.Request) (*proxyRequest, error) {
    path := request.URL.Path // 获取请求的 URL 路径

    split := strings.SplitN(path, "/", 5) // 使用 "/" 分割 URL 路径，最多分割成 5 部分
    if len(split) < 5 {
        return nil, fmt.Errorf("invalid request path '%s'", path) // 返回错误信息
    }
    # 使用 peer.Decode 方法解析 split[2]，如果出现错误则返回错误信息
    if _, err := peer.Decode(split[2]); err != nil {
        return nil, fmt.Errorf("invalid request path '%s'", path)
    }

    # 如果 split[3] 等于 "http"，则返回一个 proxyRequest 对象和 nil
    if split[3] == "http" {
        return &proxyRequest{split[2], protocol.ID("/http"), split[4]}, nil
    }

    # 使用 strings.SplitN 方法将 path 按 "/" 分割成最多 7 个部分
    split = strings.SplitN(path, "/", 7)
    # 如果分割后的部分数量小于 7，或者 split[3] 不等于 "x"，或者 split[5] 不等于 "http"，则返回错误信息
    if len(split) < 7 || split[3] != "x" || split[5] != "http" {
        return nil, fmt.Errorf("invalid request path '%s'", path)
    }

    # 返回一个 proxyRequest 对象和 nil，其中 protocol.ID("/x/" + split[4] + "/http") 作为 protocol.ID 的参数
    return &proxyRequest{split[2], protocol.ID("/x/" + split[4] + "/http"), split[6]}, nil
# 定义一个处理错误的函数，接受一个 http.ResponseWriter 对象、错误消息、错误对象和状态码作为参数
func handleError(w http.ResponseWriter, msg string, err error, code int) {
    # 使用 http.Error 函数向客户端返回一个错误消息，消息内容为格式化后的错误消息和错误对象，状态码为指定的状态码
    http.Error(w, fmt.Sprintf("%s: %s", msg, err), code)
}
```