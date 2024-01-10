# `kubo\core\corehttp\corehttp.go`

```
/*
Package corehttp provides utilities for the webui, gateways, and other
high-level HTTP interfaces to IPFS.
*/
package corehttp

import (
    "context" // 导入上下文包，用于控制请求的取消、超时等
    "fmt" // 导入格式化包，用于格式化输出
    "net" // 导入网络包，用于网络操作
    "net/http" // 导入 HTTP 包，用于构建 HTTP 服务器和客户端
    "time" // 导入时间包，用于处理时间相关操作

    logging "github.com/ipfs/go-log" // 导入日志包，用于记录日志
    core "github.com/ipfs/kubo/core" // 导入核心包，用于 IPFS 核心功能
    "github.com/jbenet/goprocess" // 导入进程包，用于管理进程
    periodicproc "github.com/jbenet/goprocess/periodic" // 导入周期性进程包，用于管理周期性进程
    ma "github.com/multiformats/go-multiaddr" // 导入多地址包，用于处理多格式地址
    manet "github.com/multiformats/go-multiaddr/net" // 导入多格式地址网络包，用于处理多格式地址的网络操作
)

var log = logging.Logger("core/server") // 定义日志记录器

// shutdownTimeout is the timeout after which we'll stop waiting for hung
// commands to return on shutdown.
const shutdownTimeout = 30 * time.Second // 定义关闭超时时间为 30 秒

// ServeOption registers any HTTP handlers it provides on the given mux.
// It returns the mux to expose to future options, which may be a new mux if it
// is interested in mediating requests to future options, or the same mux
// initially passed in if not.
type ServeOption func(*core.IpfsNode, net.Listener, *http.ServeMux) (*http.ServeMux, error) // 定义 ServeOption 类型，用于注册 HTTP 处理程序

// MakeHandler turns a list of ServeOptions into a http.Handler that implements
// all of the given options, in order.
func MakeHandler(n *core.IpfsNode, l net.Listener, options ...ServeOption) (http.Handler, error) {
    topMux := http.NewServeMux() // 创建顶级 ServeMux 对象
    mux := topMux // 将顶级 ServeMux 对象赋值给 mux
    for _, option := range options { // 遍历 ServeOptions 列表
        var err error // 定义错误变量
        mux, err = option(n, l, mux) // 调用 ServeOption 注册 HTTP 处理程序
        if err != nil { // 如果注册出错
            return nil, err // 返回空值和错误
        }
    }
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) { // 创建 HTTP 处理程序
        // ServeMux does not support requests with CONNECT method,
        // so we need to handle them separately
        // https://golang.org/src/net/http/request.go#L111
        if r.Method == http.MethodConnect { // 如果请求方法为 CONNECT
            w.WriteHeader(http.StatusOK) // 返回状态码 200
            return // 结束处理
        }
        topMux.ServeHTTP(w, r) // 使用顶级 ServeMux 处理请求
    })
    return handler, nil // 返回处理程序和空值
}

// ListenAndServe runs an HTTP server listening at |listeningMultiAddr| with
// ListenAndServe listens on the given multiaddr and serves the IPFS node with the given options
// The address must be provided in multiaddr format
// TODO intelligently parse address strings in other formats so long as they unambiguously map to a valid multiaddr. e.g. for convenience, ":8080" should map to "/ip4/0.0.0.0/tcp/8080".
func ListenAndServe(n *core.IpfsNode, listeningMultiAddr string, options ...ServeOption) error {
    // Convert the listeningMultiAddr string to a Multiaddr object
    addr, err := ma.NewMultiaddr(listeningMultiAddr)
    if err != nil {
        return err
    }

    // Listen on the specified multiaddr
    list, err := manet.Listen(addr)
    if err != nil {
        return err
    }

    // Get the actual multiaddr we are listening on
    addr = list.Multiaddr()
    fmt.Printf("RPC API server listening on %s\n", addr)

    // Serve the IPFS node with the specified options
    return Serve(n, manet.NetListener(list), options...)
}

// Serve accepts incoming HTTP connections on the listener and pass them to ServeOption handlers.
func Serve(node *core.IpfsNode, lis net.Listener, options ...ServeOption) error {
    // Make sure to close the listener no matter what
    defer lis.Close()

    // Create a handler for the server with the specified options
    handler, err := MakeHandler(node, lis, options...)
    if err != nil {
        return err
    }

    // Get the network address of the listener
    addr, err := manet.FromNetAddr(lis.Addr())
    if err != nil {
        return err
    }

    // Check if the node process is closing
    select {
    case <-node.Process.Closing():
        return fmt.Errorf("failed to start server, process closing")
    default:
    }

    // Create an HTTP server with the handler
    server := &http.Server{
        Handler: handler,
    }

    var serverError error
    // Start the server in a separate goroutine
    serverProc := node.Process.Go(func(p goprocess.Process) {
        serverError = server.Serve(lis)
    })

    // Wait for the server to exit
    select {
    case <-serverProc.Closed():
    // if node being closed before server exits, close server
    // 从节点进程的关闭通道接收信号，表示服务器正在关闭
    case <-node.Process.Closing():
        // 打印服务器正在终止的信息
        log.Infof("server at %s terminating...", addr)

        // 创建一个定时器，每5秒打印一次等待服务器终止的信息
        warnProc := periodicproc.Tick(5*time.Second, func(_ goprocess.Process) {
            log.Infof("waiting for server at %s to terminate...", addr)
        })

        // 这个超时本应该不必要，如果所有命令都遵守它们的上下文，但我们应该有*一些*超时。
        // 使用带有超时的上下文创建一个新的上下文
        ctx, cancel := context.WithTimeout(context.Background(), shutdownTimeout)
        // 延迟取消上下文
        defer cancel()
        // 关闭服务器，等待它终止
        err := server.Shutdown(ctx)

        // 应该已经关闭，但我们仍然需要等待它设置错误。
        // 等待服务器进程关闭
        <-serverProc.Closed()
        // 将错误赋值给服务器错误
        serverError = err

        // 关闭警告进程
        warnProc.Close()
    }

    // 打印服务器终止的信息
    log.Infof("server at %s terminated", addr)
    // 返回服务器错误
    return serverError
# 闭合前面的函数定义
```