# `v2ray-core\testing\servers\udp\udp.go`

```go
package udp

import (
    "fmt"

    "v2ray.com/core/common/net"
)

type Server struct {
    Port         net.Port          // 服务器监听的端口号
    MsgProcessor func(msg []byte) []byte  // 处理消息的函数
    accepting    bool              // 标识服务器是否正在接受连接
    conn         *net.UDPConn       // UDP连接对象
}

func (server *Server) Start() (net.Destination, error) {
    // 监听UDP连接
    conn, err := net.ListenUDP("udp", &net.UDPAddr{
        IP:   []byte{127, 0, 0, 1},  // 监听的IP地址
        Port: int(server.Port),       // 监听的端口号
        Zone: "",                     // 区域
    })
    if err != nil {
        return net.Destination{}, err
    }
    server.Port = net.Port(conn.LocalAddr().(*net.UDPAddr).Port)  // 更新服务器端口号
    fmt.Println("UDP server started on port ", server.Port)

    server.conn = conn  // 保存UDP连接对象
    go server.handleConnection(conn)  // 启动处理连接的协程
    localAddr := conn.LocalAddr().(*net.UDPAddr)
    return net.UDPDestination(net.IPAddress(localAddr.IP), net.Port(localAddr.Port)), nil  // 返回本地地址和端口
}

func (server *Server) handleConnection(conn *net.UDPConn) {
    server.accepting = true  // 标识服务器正在接受连接
    for server.accepting {
        buffer := make([]byte, 2*1024)  // 创建缓冲区
        nBytes, addr, err := conn.ReadFromUDP(buffer)  // 从UDP连接中读取数据
        if err != nil {
            fmt.Printf("Failed to read from UDP: %v\n", err)  // 打印读取失败的错误信息
            continue
        }

        response := server.MsgProcessor(buffer[:nBytes])  // 处理接收到的消息
        if _, err := conn.WriteToUDP(response, addr); err != nil {  // 向客户端发送处理后的消息
            fmt.Println("Failed to write to UDP: ", err.Error())  // 打印发送失败的错误信息
        }
    }
}

func (server *Server) Close() error {
    server.accepting = false  // 停止接受连接
    return server.conn.Close()  // 关闭UDP连接
}
```