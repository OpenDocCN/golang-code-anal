# `trojan-go\tunnel\http\server.go`

```
package http

import (
    "bufio" // 导入 bufio 包，提供了缓冲 I/O 的功能
    "context" // 导入 context 包，提供了跟踪请求的上下文
    "fmt" // 导入 fmt 包，提供了格式化 I/O 的功能
    "io" // 导入 io 包，提供了基本的 I/O 接口
    "io/ioutil" // 导入 ioutil 包，提供了 I/O 实用工具函数
    "net" // 导入 net 包，提供了基本的网络 I/O 接口
    "net/http" // 导入 http 包，提供了 HTTP 客户端和服务端的实现
    "strings" // 导入 strings 包，提供了操作字符串的函数

    "github.com/p4gefau1t/trojan-go/common" // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/log" // 导入自定义包 log
    "github.com/p4gefau1t/trojan-go/tunnel" // 导入自定义包 tunnel
)

type ConnectConn struct {
    net.Conn // 嵌入 net.Conn 接口
    metadata *tunnel.Metadata // 定义 metadata 指针字段
}

func (c *ConnectConn) Metadata() *tunnel.Metadata {
    return c.metadata // 返回 metadata 指针
}

type OtherConn struct {
    net.Conn // 嵌入 net.Conn 接口
    metadata   *tunnel.Metadata // 定义 metadata 指针字段
    reqReader  *io.PipeReader // 定义 reqReader 指针字段
    respWriter *io.PipeWriter // 定义 respWriter 指针字段
    ctx        context.Context // 定义 ctx 字段
    cancel     context.CancelFunc // 定义 cancel 字段
}

func (c *OtherConn) Metadata() *tunnel.Metadata {
    return c.metadata // 返回 metadata 指针
}

func (c *OtherConn) Read(p []byte) (int, error) {
    n, err := c.reqReader.Read(p) // 从 reqReader 中读取数据到 p 中
    if err == io.EOF { // 如果读取到文件末尾
        if n != 0 { // 如果读取的字节数不为 0
            panic("non zero") // 抛出 panic 异常
        }
        for range c.ctx.Done() { // 循环等待 ctx 的 Done 信号
            return 0, common.NewError("http conn closed") // 返回错误信息
        }
    }
    return n, err // 返回读取的字节数和错误信息
}

func (c *OtherConn) Write(p []byte) (int, error) {
    return c.respWriter.Write(p) // 将 p 中的数据写入 respWriter
}

func (c *OtherConn) Close() error {
    c.cancel() // 调用 cancel 函数
    c.reqReader.Close() // 关闭 reqReader
    c.respWriter.Close() // 关闭 respWriter
    return nil // 返回空错误
}

type Server struct {
    underlay tunnel.Server // 定义 underlay 字段
    connChan chan tunnel.Conn // 定义 connChan 通道
    ctx      context.Context // 定义 ctx 字段
    cancel   context.CancelFunc // 定义 cancel 字段
}

func (s *Server) acceptLoop() {
    // 实现接收循环的逻辑
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    select {
    case conn := <-s.connChan: // 从 connChan 中接收连接
        return conn, nil // 返回连接和空错误
    case <-s.ctx.Done(): // 如果 ctx 发送了 Done 信号
        return nil, common.NewError("http server closed") // 返回错误信息
    }
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    <-s.ctx.Done() // 等待 ctx 发送 Done 信号
    return nil, common.NewError("http server closed") // 返回错误信息
}

func (s *Server) Close() error {
    s.cancel() // 调用 cancel 函数
    return s.underlay.Close() // 返回 underlay 的 Close 方法的结果
}

func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    ctx, cancel := context.WithCancel(ctx) // 创建一个新的上下文，并返回取消函数
    // 创建一个新的服务器对象，设置其底层连接、连接通道、上下文和取消函数
    server := &Server{
        underlay: underlay,
        connChan: make(chan tunnel.Conn, 32),
        ctx:      ctx,
        cancel:   cancel,
    }
    // 启动服务器的接受连接循环
    go server.acceptLoop()
    // 返回创建的服务器对象和空错误
    return server, nil
# 闭合前面的函数定义
```