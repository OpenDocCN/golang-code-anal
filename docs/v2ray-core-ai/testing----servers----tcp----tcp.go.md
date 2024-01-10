# `v2ray-core\testing\servers\tcp\tcp.go`

```
package tcp

import (
    "context"  // 引入 context 包，用于处理请求的上下文
    "fmt"  // 引入 fmt 包，用于格式化输出
    "io"  // 引入 io 包，用于实现 I/O 操作

    "v2ray.com/core/common/buf"  // 引入 v2ray.com/core/common/buf 包，用于处理缓冲区
    "v2ray.com/core/common/net"  // 引入 v2ray.com/core/common/net 包，用于处理网络相关操作
    "v2ray.com/core/common/task"  // 引入 v2ray.com/core/common/task 包，用于处理任务
    "v2ray.com/core/transport/internet"  // 引入 v2ray.com/core/transport/internet 包，用于处理网络传输
    "v2ray.com/core/transport/pipe"  // 引入 v2ray.com/core/transport/pipe 包，用于处理管道通信
)

type Server struct {
    Port         net.Port  // 服务器监听的端口号
    MsgProcessor func(msg []byte) []byte  // 消息处理函数
    ShouldClose  bool  // 是否关闭服务器的标志
    SendFirst    []byte  // 首次发送的数据
    Listen       net.Address  // 服务器监听的地址
    listener     net.Listener  // 网络监听器
}

func (server *Server) Start() (net.Destination, error) {
    return server.StartContext(context.Background(), nil)  // 启动服务器，使用默认的上下文
}

func (server *Server) StartContext(ctx context.Context, sockopt *internet.SocketConfig) (net.Destination, error) {
    listenerAddr := server.Listen  // 获取服务器监听的地址
    if listenerAddr == nil {
        listenerAddr = net.LocalHostIP  // 如果监听地址为空，则使用本地主机地址
    }
    listener, err := internet.ListenSystem(ctx, &net.TCPAddr{
        IP:   listenerAddr.IP(),  // 设置监听的 IP 地址
        Port: int(server.Port),  // 设置监听的端口号
    }, sockopt)  // 使用系统默认的网络监听方式
    if err != nil {
        return net.Destination{}, err  // 如果出错，返回空的目标地址和错误信息
    }

    localAddr := listener.Addr().(*net.TCPAddr)  // 获取监听器的本地地址
    server.Port = net.Port(localAddr.Port)  // 更新服务器的端口号
    server.listener = listener  // 更新服务器的监听器
    go server.acceptConnections(listener.(*net.TCPListener))  // 异步接受连接请求

    return net.TCPDestination(net.IPAddress(localAddr.IP), net.Port(localAddr.Port)), nil  // 返回 TCP 目标地址和空的错误信息
}

func (server *Server) acceptConnections(listener *net.TCPListener) {
    for {
        conn, err := listener.Accept()  // 接受连接请求
        if err != nil {
            fmt.Printf("Failed accept TCP connection: %v\n", err)  // 如果出错，打印错误信息
            return  // 返回
        }

        go server.handleConnection(conn)  // 异步处理连接
    }
}

func (server *Server) handleConnection(conn net.Conn) {
    if len(server.SendFirst) > 0 {  // 如果首次发送的数据长度大于 0
        conn.Write(server.SendFirst)  // 写入首次发送的数据
    }

    pReader, pWriter := pipe.New(pipe.WithoutSizeLimit())  // 创建管道读写器
    // 使用任务包装函数来执行两个并发的操作，一个是读取数据并处理，一个是写入数据
    err := task.Run(context.Background(), func() error {
        // 延迟关闭写入器，确保在函数返回前关闭
        defer pWriter.Close() // nolint: errcheck

        // 无限循环读取数据
        for {
            // 创建一个新的缓冲区
            b := buf.New()
            // 从连接中读取数据到缓冲区
            if _, err := b.ReadFrom(conn); err != nil {
                // 如果读取到文件末尾，返回 nil
                if err == io.EOF {
                    return nil
                }
                // 如果出现其他错误，返回错误
                return err
            }
            // 复制缓冲区的数据，并使用服务器消息处理器处理
            copy(b.Bytes(), server.MsgProcessor(b.Bytes()))
            // 将处理后的数据写入到写入器
            if err := pWriter.WriteMultiBuffer(buf.MultiBuffer{b}); err != nil {
                return err
            }
        }
    }, func() error {
        // 延迟中断读取器，确保在函数返回前中断
        defer pReader.Interrupt()

        // 创建一个新的连接写入器
        w := buf.NewWriter(conn)
        // 无限循环读取多重缓冲区
        for {
            // 从读取器中读取多重缓冲区和错误
            mb, err := pReader.ReadMultiBuffer()
            if err != nil {
                // 如果读取到文件末尾，返回 nil
                if err == io.EOF {
                    return nil
                }
                // 如果出现其他错误，返回错误
                return err
            }
            // 将多重缓冲区写入到连接写入器
            if err := w.WriteMultiBuffer(mb); err != nil {
                return err
            }
        }
    })

    // 如果发生错误，打印错误信息
    if err != nil {
        fmt.Println("failed to transfer data: ", err.Error())
    }

    // 关闭连接
    conn.Close() // nolint: errcheck
# 关闭服务器的方法
func (server *Server) Close() error:
    # 调用listener的Close方法关闭服务器的监听器，并返回可能的错误
    return server.listener.Close()
```