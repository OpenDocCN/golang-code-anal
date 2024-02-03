# `v2ray-core\testing\servers\http\http.go`

```go
package tcp

import (
    "net/http"

    "v2ray.com/core/common/net"
)

type Server struct {
    Port        net.Port               // 服务器端口
    PathHandler map[string]http.HandlerFunc  // 路径到处理函数的映射
    server      *http.Server           // HTTP 服务器对象
}

func (s *Server) ServeHTTP(resp http.ResponseWriter, req *http.Request) {
    if req.URL.Path == "/" {
        resp.Header().Set("Content-Type", "text/plain; charset=utf-8")  // 设置响应头的内容类型
        resp.WriteHeader(http.StatusOK)  // 设置响应状态码为 200
        resp.Write([]byte("Home"))  // 向响应中写入数据
        return
    }

    handler, found := s.PathHandler[req.URL.Path]  // 获取请求路径对应的处理函数
    if found {
        handler(resp, req)  // 调用对应的处理函数处理请求
    }
}

func (s *Server) Start() (net.Destination, error) {
    s.server = &http.Server{  // 创建 HTTP 服务器对象
        Addr:    "127.0.0.1:" + s.Port.String(),  // 设置服务器地址和端口
        Handler: s,  // 设置处理器为当前 Server 对象
    }
    go s.server.ListenAndServe()  // 异步启动 HTTP 服务器
    return net.TCPDestination(net.LocalHostIP, net.Port(s.Port)), nil  // 返回服务器的目标地址和端口
}

func (s *Server) Close() error {
    return s.server.Close()  // 关闭服务器
}
```