# `trojan-go\tunnel\mux\server.go`

```
package mux

import (
    "context"  // 导入 context 包，用于处理上下文
    "github.com/xtaci/smux"  // 导入 smux 包，用于处理多路复用

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 log 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 tunnel 包
)

// Server is a smux server
type Server struct {
    underlay tunnel.Server  // 定义一个 underlay 字段，类型为 tunnel.Server
    connChan chan tunnel.Conn  // 定义一个 connChan 字段，类型为通道，用于传输 tunnel.Conn 类型的数据
    ctx      context.Context  // 定义一个 ctx 字段，类型为 context.Context
    cancel   context.CancelFunc  // 定义一个 cancel 字段，类型为 context.CancelFunc
}

func (s *Server) acceptConnWorker() {
    for {  // 无限循环
        conn, err := s.underlay.AcceptConn(&Tunnel{})  // 接受连接，并返回连接和错误
        if err != nil {  // 如果有错误
            log.Debug(err)  // 记录错误信息
            select {  // 选择
            case <-s.ctx.Done():  // 如果 ctx 被取消
                return  // 结束循环
            default:  // 默认情况
            }
            continue  // 继续下一次循环
        }
        go func(conn tunnel.Conn) {  // 启动一个新的 goroutine
            smuxConfig := smux.DefaultConfig()  // 创建一个默认的 smux 配置
            // smuxConfig.KeepAliveDisabled = true  // 禁用保持连接
            smuxSession, err := smux.Server(conn, smuxConfig)  // 创建一个 smux 会话
            if err != nil {  // 如果有错误
                log.Error(err)  // 记录错误信息
                return  // 结束当前 goroutine
            }
            go func(session *smux.Session, conn tunnel.Conn) {  // 启动一个新的 goroutine
                defer session.Close()  // 延迟关闭会话
                defer conn.Close()  // 延迟关闭连接
                for {  // 无限循环
                    stream, err := session.AcceptStream()  // 接受数据流
                    if err != nil {  // 如果有错误
                        log.Error(err)  // 记录错误信息
                        return  // 结束当前 goroutine
                    }
                    select {  // 选择
                    case s.connChan <- &Conn{  // 将数据发送到 connChan
                        rwc:  stream,  // 设置 rwc 字段为 stream
                        Conn: conn,  // 设置 Conn 字段为 conn
                    }:
                    case <-s.ctx.Done():  // 如果 ctx 被取消
                        log.Debug("exiting")  // 记录调试信息
                        return  // 结束当前 goroutine
                    }
                }
            }(smuxSession, conn)
        }(conn)
    }
}

func (s *Server) AcceptConn(tunnel.Tunnel) (tunnel.Conn, error) {
    select {  // 选择
    case conn := <-s.connChan:  // 从 connChan 中接收数据
        return conn, nil  // 返回接收到的数据和空错误
    case <-s.ctx.Done():  // 如果 ctx 被取消
        return nil, common.NewError("mux server closed")  // 返回空值和 mux server closed 错误
    }
}

func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    # 抛出一个错误，表示不支持当前操作
    panic("not supported")
# 关闭服务器，取消上下文并关闭底层连接
func (s *Server) Close() error:
    # 取消上下文
    s.cancel()
    # 关闭底层连接
    return s.underlay.Close()

# 创建新的服务器实例
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error):
    # 使用传入的上下文创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 创建服务器实例，设置底层连接、上下文、取消函数和连接通道
    server := &Server{
        underlay: underlay,
        ctx:      ctx,
        cancel:   cancel,
        connChan: make(chan tunnel.Conn, 32),
    }
    # 启动接受连接的工作线程
    go server.acceptConnWorker()
    # 输出调试信息
    log.Debug("mux server created")
    # 返回创建的服务器实例和空错误
    return server, nil
```