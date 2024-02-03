# `trojan-go\tunnel\transport\server.go`

```go
package transport

import (
    "bufio" // 导入 bufio 包，提供了缓冲 I/O 的功能
    "context" // 导入 context 包，用于跟踪请求的上下文、取消请求等
    "net" // 导入 net 包，提供了基本的网络 I/O 接口
    "net/http" // 导入 net/http 包，提供了 HTTP 客户端和服务端的实现
    "os" // 导入 os 包，提供了操作系统函数
    "os/exec" // 导入 os/exec 包，提供了执行外部命令的函数
    "strconv" // 导入 strconv 包，提供了字符串和基本数据类型之间的转换
    "sync" // 导入 sync 包，提供了并发安全的锁和条件变量等基本同步原语
    "time" // 导入 time 包，提供了时间的显示和测量用的函数

    "github.com/p4gefau1t/trojan-go/common" // 导入自定义包 common
    "github.com/p4gefau1t/trojan-go/config" // 导入自定义包 config
    "github.com/p4gefau1t/trojan-go/log" // 导入自定义包 log
    "github.com/p4gefau1t/trojan-go/tunnel" // 导入自定义包 tunnel
)

// Server is a server of transport layer
type Server struct {
    tcpListener net.Listener // 定义一个 TCP 服务器监听器
    cmd         *exec.Cmd // 定义一个指向外部命令的指针
    connChan    chan tunnel.Conn // 定义一个通道，用于传输 tunnel.Conn 类型的数据
    wsChan      chan tunnel.Conn // 定义一个通道，用于传输 tunnel.Conn 类型的数据
    httpLock    sync.RWMutex // 定义一个读写锁
    nextHTTP    bool // 定义一个布尔类型的变量
    ctx         context.Context // 定义一个上下文对象
    cancel      context.CancelFunc // 定义一个取消函数
}

func (s *Server) Close() error {
    s.cancel() // 调用取消函数，取消上下文
    if s.cmd != nil && s.cmd.Process != nil {
        s.cmd.Process.Kill() // 如果外部命令不为空且其进程不为空，则杀死进程
    }
    return s.tcpListener.Close() // 关闭 TCP 服务器监听器
}

func (s *Server) acceptLoop() {
    // 无限循环，接受TCP连接
    for {
        // 接受TCP连接
        tcpConn, err := s.tcpListener.Accept()
        // 如果出现错误
        if err != nil {
            // 检查是否上下文已经结束
            select {
            case <-s.ctx.Done():
            default:
                // 记录错误并等待一段时间后继续
                log.Error(common.NewError("transport accept error").Base(err))
                time.Sleep(time.Millisecond * 100)
            }
            return
        }

        // 在新的goroutine中处理TCP连接
        go func(tcpConn net.Conn) {
            // 记录TCP连接来源
            log.Info("tcp connection from", tcpConn.RemoteAddr())
            // 读取HTTP锁
            s.httpLock.RLock()
            // 如果启用了明文模式
            if s.nextHTTP { // plaintext mode enabled
                // 释放HTTP锁
                s.httpLock.RUnlock()
                // 使用真实的HTTP头解析器模拟真实的HTTP服务器
                rewindConn := common.NewRewindConn(tcpConn)
                rewindConn.SetBufferSize(512)
                defer rewindConn.StopBuffering()

                r := bufio.NewReader(rewindConn)
                // 读取HTTP请求
                httpReq, err := http.ReadRequest(r)
                rewindConn.Rewind()
                rewindConn.StopBuffering()
                // 如果出现错误，将其传递到trojan协议层进行进一步检查
                if err != nil {
                    s.connChan <- &Conn{
                        Conn: rewindConn,
                    }
                } else {
                    // 如果是HTTP请求，将其传递到websocket协议层
                    log.Debug("plaintext http request: ", httpReq)
                    s.wsChan <- &Conn{
                        Conn: rewindConn,
                    }
                }
            } else {
                // 如果未启用明文模式，将TCP连接传递到连接通道
                s.httpLock.RUnlock()
                s.connChan <- &Conn{
                    Conn: tcpConn,
                }
            }
        }(tcpConn)
    }
// AcceptConn 接受连接，返回通道连接和错误信息
func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
    // TODO fix import cycle
    // 如果覆盖不为空并且覆盖名称为"WEBSOCKET"或"HTTP"
    if overlay != nil && (overlay.Name() == "WEBSOCKET" || overlay.Name() == "HTTP") {
        // 加锁以防止并发访问
        s.httpLock.Lock()
        // 设置下一个 HTTP 连接为真
        s.nextHTTP = true
        // 解锁
        s.httpLock.Unlock()
        // 选择操作，等待 WebSocket 连接
        select {
        case conn := <-s.wsChan:
            return conn, nil
        // 等待上下文结束
        case <-s.ctx.Done():
            return nil, common.NewError("transport server closed")
        }
    }
    // 选择操作，等待普通连接
    select {
    case conn := <-s.connChan:
        return conn, nil
    // 等待上下文结束
    case <-s.ctx.Done():
        return nil, common.NewError("transport server closed")
    }
}

// AcceptPacket 接受数据包连接，返回数据包连接和错误信息
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    // 抛出错误，表示不支持该操作
    panic("not supported")
}

// NewServer 创建一个传输层服务器
func NewServer(ctx context.Context, _ tunnel.Server) (*Server, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 创建监听地址
    listenAddress := tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)

    var cmd *exec.Cmd
}
    # 如果启用了传输插件
    if cfg.TransportPlugin.Enabled:
        # 输出警告信息
        log.Warn("transport server will use plugin and work in plain text mode")
        # 根据传输插件类型进行不同的处理
        switch cfg.TransportPlugin.Type:
            # 如果是 shadowsocks 类型
            case "shadowsocks":
                # 设置 trojanHost 和 trojanPort
                trojanHost := "127.0.0.1"
                trojanPort := common.PickPort("tcp", trojanHost)
                # 将相关环境变量添加到传输插件的环境变量中
                cfg.TransportPlugin.Env = append(
                    cfg.TransportPlugin.Env,
                    "SS_REMOTE_HOST="+cfg.LocalHost,
                    "SS_REMOTE_PORT="+strconv.FormatInt(int64(cfg.LocalPort), 10),
                    "SS_LOCAL_HOST="+trojanHost,
                    "SS_LOCAL_PORT="+strconv.FormatInt(int64(trojanPort), 10),
                    "SS_PLUGIN_OPTIONS="+cfg.TransportPlugin.Option,
                )

                # 更新本地主机和端口
                cfg.LocalHost = trojanHost
                cfg.LocalPort = trojanPort
                # 创建监听地址
                listenAddress = tunnel.NewAddressFromHostPort("tcp", cfg.LocalHost, cfg.LocalPort)
                # 输出调试信息
                log.Debug("new listen address", listenAddress)
                log.Debug("plugin env", cfg.TransportPlugin.Env)

                # 执行传输插件的命令
                cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
                cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
                cmd.Stdout = os.Stdout
                cmd.Stderr = os.Stdout
                cmd.Start()
            # 如果是其他类型
            case "other":
                # 执行传输插件的命令
                cmd = exec.Command(cfg.TransportPlugin.Command, cfg.TransportPlugin.Arg...)
                cmd.Env = append(cmd.Env, cfg.TransportPlugin.Env...)
                cmd.Stdout = os.Stdout
                cmd.Stderr = os.Stdout
                cmd.Start()
            # 如果是明文类型
            case "plaintext":
                # 什么都不做
                // do nothing
            # 如果是其他类型
            default:
                # 返回错误信息
                return nil, common.NewError("invalid plugin type: " + cfg.TransportPlugin.Type)
        # 创建 TCP 监听器
        tcpListener, err := net.Listen("tcp", listenAddress.String())
        # 如果出现错误，返回错误信息
        if err != nil:
            return nil, err

        # 创建上下文和取消函数
        ctx, cancel := context.WithCancel(ctx)
    # 创建一个 Server 结构体实例，并初始化其字段
    server := &Server{
        tcpListener: tcpListener,  # 使用传入的 tcpListener 初始化 tcpListener 字段
        cmd:         cmd,           # 使用传入的 cmd 初始化 cmd 字段
        ctx:         ctx,           # 使用传入的 ctx 初始化 ctx 字段
        cancel:      cancel,        # 使用传入的 cancel 初始化 cancel 字段
        connChan:    make(chan tunnel.Conn, 32),  # 创建一个容量为 32 的通道，并赋值给 connChan 字段
        wsChan:      make(chan tunnel.Conn, 32),  # 创建一个容量为 32 的通道，并赋值给 wsChan 字段
    }
    # 启动接受连接的循环
    go server.acceptLoop()
    # 返回创建的 Server 实例和空错误
    return server, nil
# 闭合前面的函数定义
```