# `trojan-go\tunnel\shadowsocks\server.go`

```go
package shadowsocks

import (
    "context"  // 导入上下文包
    "net"  // 导入网络包

    "github.com/shadowsocks/go-shadowsocks2/core"  // 导入 shadowsocks 核心包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 公共包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 trojan-go 配置包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 trojan-go 日志包
    "github.com/p4gefau1t/trojan-go/redirector"  // 导入 trojan-go 重定向包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 隧道包
)

type Server struct {
    core.Cipher  // shadowsocks 核心包中的 Cipher 结构
    *redirector.Redirector  // redirector 包中的 Redirector 结构
    underlay  tunnel.Server  // 隧道服务器
    redirAddr net.Addr  // 网络地址
}

func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := s.underlay.AcceptConn(&Tunnel{})  // 接受连接
    if err != nil {
        return nil, common.NewError("shadowsocks failed to accept connection from underlying tunnel").Base(err)  // 返回错误信息
    }
    rewindConn := common.NewRewindConn(conn)  // 创建新的回滚连接
    rewindConn.SetBufferSize(1024)  // 设置缓冲区大小
    defer rewindConn.StopBuffering()  // 延迟停止缓冲

    // 尝试从连接中读取数据
    buf := [1024]byte{}
    testConn := s.Cipher.StreamConn(rewindConn)  // 使用 Cipher 结构创建流连接
    if _, err := testConn.Read(buf[:]); err != nil {
        // 我们受到攻击
        log.Error(common.NewError("shadowsocks failed to decrypt").Base(err))  // 记录错误日志
        rewindConn.Rewind()  // 回滚连接
        rewindConn.StopBuffering()  // 停止缓冲
        s.Redirect(&redirector.Redirection{  // 重定向连接
            RedirectTo:  s.redirAddr,  // 重定向地址
            InboundConn: rewindConn,  // 入站连接
        })
        return nil, common.NewError("invalid aead payload")  // 返回错误信息
    }
    rewindConn.Rewind()  // 回滚连接
    rewindConn.StopBuffering()  // 停止缓冲

    return &Conn{  // 返回连接
        aeadConn: s.Cipher.StreamConn(rewindConn),  // 使用 Cipher 结构创建流连接
        Conn:     conn,  // 连接
    }, nil
}

func (s *Server) AcceptPacket(t tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")  // 抛出异常，不支持该操作
}

func (s *Server) Close() error {
    return s.underlay.Close()  // 关闭隧道服务器
}

func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    cfg := config.FromContext(ctx, Name).(*Config)  // 从上下文中获取配置信息
    cipher, err := core.PickCipher(cfg.Shadowsocks.Method, nil, cfg.Shadowsocks.Password)  // 选择加密方法
    # 如果发生错误，则返回空值和错误信息
    if err != nil:
        return nil, common.NewError("invalid shadowsocks cipher").Base(err)
    # 如果远程主机地址为空，则返回空值和错误信息
    if cfg.RemoteHost == "":
        return nil, common.NewError("invalid shadowsocks redirection address")
    # 如果远程端口为0，则返回空值和错误信息
    if cfg.RemotePort == 0:
        return nil, common.NewError("invalid shadowsocks redirection port")
    # 记录调试信息，表示 shadowsocks 客户端已创建
    log.Debug("shadowsocks client created")
    # 返回一个包含特定属性的 Server 对象和空值
    return &Server{
        underlay:   underlay,
        Cipher:     cipher,
        Redirector: redirector.NewRedirector(ctx),
        redirAddr:  tunnel.NewAddressFromHostPort("tcp", cfg.RemoteHost, cfg.RemotePort),
    }, nil
# 闭合前面的函数定义
```