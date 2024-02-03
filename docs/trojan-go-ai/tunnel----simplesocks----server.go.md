# `trojan-go\tunnel\simplesocks\server.go`

```go
package simplesocks

import (
    "context"
    "fmt"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/log"
    "github.com/p4gefau1t/trojan-go/tunnel"
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"
)

// Server is a simplesocks server
type Server struct {
    underlay   tunnel.Server  // 声明一个tunnel.Server类型的字段underlay
    connChan   chan tunnel.Conn  // 声明一个tunnel.Conn类型的通道connChan
    packetChan chan tunnel.PacketConn  // 声明一个tunnel.PacketConn类型的通道packetChan
    ctx        context.Context  // 声明一个context.Context类型的字段ctx
    cancel     context.CancelFunc  // 声明一个context.CancelFunc类型的字段cancel
}

func (s *Server) Close() error {
    s.cancel()  // 调用cancel函数
    return s.underlay.Close()  // 返回underlay字段的Close方法的结果
}

func (s *Server) acceptLoop() {
    for {
        conn, err := s.underlay.AcceptConn(&Tunnel{})  // 调用underlay字段的AcceptConn方法，传入&Tunnel{}作为参数
        if err != nil {
            log.Error(common.NewError("simplesocks failed to accept connection from underlying tunnel").Base(err))  // 记录错误日志
            select {
            case <-s.ctx.Done():  // 当ctx的Done通道关闭时
                return  // 退出循环
            default:
            }
            continue  // 继续下一次循环
        }
        metadata := new(tunnel.Metadata)  // 创建一个tunnel.Metadata类型的指针metadata
        if err := metadata.ReadFrom(conn); err != nil {  // 调用metadata的ReadFrom方法，传入conn作为参数
            log.Error(common.NewError("simplesocks server faield to read header").Base(err))  // 记录错误日志
            conn.Close()  // 关闭conn
            continue  // 继续下一次循环
        }
        switch metadata.Command {  // 根据metadata的Command字段进行分支判断
        case Connect:  // 如果Command为Connect
            s.connChan <- &Conn{  // 将&Conn{metadata: metadata, Conn: conn}发送到connChan通道
                metadata: metadata,
                Conn:     conn,
            }
        case Associate:  // 如果Command为Associate
            s.packetChan <- &PacketConn{  // 将&PacketConn{PacketConn: trojan.PacketConn{Conn: conn}}发送到packetChan通道
                PacketConn: trojan.PacketConn{
                    Conn: conn,
                },
            }
        default:  // 如果Command为其他值
            log.Error(common.NewError(fmt.Sprintf("simplesocks unknown command %d", metadata.Command)))  // 记录错误日志
            conn.Close()  // 关闭conn
        }
    }
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    select {
    case conn := <-s.connChan:  // 从connChan通道接收数据并赋值给conn
        return conn, nil  // 返回conn和nil
    case <-s.ctx.Done():  // 当ctx的Done通道关闭时
        return nil, common.NewError("simplesocks server closed")  // 返回nil和一个描述服务器关闭的错误
    }
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    select {
    # 从 s.packetChan 通道接收数据包连接，并赋值给 packetConn，如果通道没有数据则阻塞
    case packetConn := <-s.packetChan:
        # 返回接收到的数据包连接和空错误
        return packetConn, nil
    # 如果 s.ctx 被取消，则执行以下代码
    case <-s.ctx.Done():
        # 返回空和一个描述服务器关闭的错误
        return nil, common.NewError("simplesocks server closed")
    }
# 创建一个新的服务器对象
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    # 使用传入的上层服务器对象创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 创建一个服务器对象，设置其属性和通道
    server := &Server{
        underlay:   underlay,
        ctx:        ctx,
        connChan:   make(chan tunnel.Conn, 32),
        packetChan: make(chan tunnel.PacketConn, 32),
        cancel:     cancel,
    }
    # 启动一个新的 goroutine 来处理连接请求
    go server.acceptLoop()
    # 记录日志，表示简单的 SOCKS 服务器已创建
    log.Debug("simplesocks server created")
    # 返回创建的服务器对象和 nil 错误
    return server, nil
}
```