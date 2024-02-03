# `trojan-go\tunnel\tls\server.go`

```go
package tls

import (
    "bufio"  // 用于提供带缓冲的 I/O 操作
    "bytes"  // 提供对字节切片的操作
    "context"  // 提供跟踪请求的上下文
    "crypto/tls"  // 提供了 TLS/SSL 加密通信协议的实现
    "crypto/x509"  // 提供了 X.509 证书的解析
    "encoding/pem"  // 用于对 PEM 格式的数据进行编码和解码
    "io"  // 提供了基本的 I/O 接口
    "io/ioutil"  // 提供了一些 I/O 实用函数
    "net"  // 提供了用于网络通信的基本接口
    "net/http"  // 提供了 HTTP 客户端和服务器的实现
    "os"  // 提供了操作系统函数
    "strings"  // 提供了操作字符串的函数
    "sync"  // 提供了并发编程的基本同步原语
    "sync/atomic"  // 提供了底层的原子级内存操作
    "time"  // 提供了时间的函数和类型

    "github.com/p4gefau1t/trojan-go/common"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/config"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/redirector"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/tls/fingerprint"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/transport"  // 引入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel/websocket"  // 引入自定义包
)

// Server is a tls server
type Server struct {
    fallbackAddress    *tunnel.Address  // 用于指定备用地址
    verifySNI          bool  // 用于指定是否验证 SNI
    sni                string  // 用于指定 SNI
    alpn               []string  // 用于指定 ALPN
    PreferServerCipher bool  // 用于指定是否优先使用服务器密码
    keyPair            []tls.Certificate  // 用于存储 TLS 证书
    keyPairLock        sync.RWMutex  // 用于保护 TLS 证书的读写锁
    httpResp           []byte  // 用于存储 HTTP 响应
    cipherSuite        []uint16  // 用于存储密码套件
    sessionTicket      bool  // 用于指定是否启用会话票据
    curve              []tls.CurveID  // 用于存储曲线 ID
    keyLogger          io.WriteCloser  // 用于记录密钥日志
    connChan           chan tunnel.Conn  // 用于传输连接的通道
    wsChan             chan tunnel.Conn  // 用于传输 WebSocket 连接的通道
    redir              *redirector.Redirector  // 用于重定向的对象
    ctx                context.Context  // 上下文对象
    cancel             context.CancelFunc  // 用于取消上下文的函数
    underlay           tunnel.Server  // 用于底层通道的服务器
    nextHTTP           int32  // 用于存储下一个 HTTP 请求的标识
    portOverrider      map[string]int  // 用于重写端口的映射
}

func (s *Server) Close() error {
    s.cancel()  // 取消上下文
    if s.keyLogger != nil {
        s.keyLogger.Close()  // 关闭密钥日志
    }
    return s.underlay.Close()  // 关闭底层通道
}

func isDomainNameMatched(pattern string, domainName string) bool {
    if strings.HasPrefix(pattern, "*.") {  // 判断是否为通配符域名
        suffix := pattern[2:]  // 获取通配符后缀
        domainPrefixLen := len(domainName) - len(suffix) - 1  // 计算域名前缀长度
        return strings.HasSuffix(domainName, suffix) && domainPrefixLen > 0 && !strings.Contains(domainName[:domainPrefixLen], ".")  // 判断域名是否匹配
    }
    return pattern == domainName  // 判断域名是否匹配
}

func (s *Server) acceptLoop() {
    // 实现 TLS 服务器的接受循环
}
# 接受连接并返回通道，如果是 websocket 则返回 websocket 连接，否则返回 trojan 连接
func (s *Server) AcceptConn(overlay tunnel.Tunnel) (tunnel.Conn, error) {
    # 如果是 websocket 连接，则设置下一个协议为 HTTP，并记录日志
    if _, ok := overlay.(*websocket.Tunnel); ok {
        atomic.StoreInt32(&s.nextHTTP, 1)
        log.Debug("next proto http")
        # 选择处理 websocket 连接的通道
        select {
        case conn := <-s.wsChan:
            return conn, nil
        case <-s.ctx.Done():
            return nil, common.NewError("transport server closed")
        }
    }
    # 如果是 trojan 连接，则选择处理 trojan 连接的通道
    select {
    case conn := <-s.connChan:
        return conn, nil
    case <-s.ctx.Done():
        return nil, common.NewError("transport server closed")
    }
}

# 接受数据包并返回数据包通道，但不支持该操作，会触发 panic
func (s *Server) AcceptPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")
}

# 检查密钥对循环，定期检查密钥和证书文件是否发生变化
func (s *Server) checkKeyPairLoop(checkRate time.Duration, keyPath string, certPath string, password string) {
    # 初始化上次密钥和证书的字节流
    var lastKeyBytes, lastCertBytes []byte
    # 创建定时器
    ticker := time.NewTicker(checkRate)
    # 无限循环，用于检查证书
    for {
        # 打印日志，表示正在检查证书
        log.Debug("checking cert...")
        # 读取密钥文件内容
        keyBytes, err := ioutil.ReadFile(keyPath)
        # 如果读取密钥文件出错
        if err != nil {
            # 打印错误日志，并继续下一次循环
            log.Error(common.NewError("tls failed to check key").Base(err))
            continue
        }
        # 读取证书文件内容
        certBytes, err := ioutil.ReadFile(certPath)
        # 如果读取证书文件出错
        if err != nil {
            # 打印错误日志，并继续下一次循环
            log.Error(common.NewError("tls failed to check cert").Base(err))
            continue
        }
        # 如果密钥内容与上次的不一样，或者证书内容与上次的不一样
        if !bytes.Equal(keyBytes, lastKeyBytes) || !bytes.Equal(lastCertBytes, certBytes) {
            # 打印日志，表示检测到新的密钥对
            log.Info("new key pair detected")
            # 加载新的密钥对
            keyPair, err := loadKeyPair(keyPath, certPath, password)
            # 如果加载密钥对出错
            if err != nil {
                # 打印错误日志，并继续下一次循环
                log.Error(common.NewError("tls failed to load new key pair").Base(err))
                continue
            }
            # 加锁，更新密钥对
            s.keyPairLock.Lock()
            s.keyPair = []tls.Certificate{*keyPair}
            s.keyPairLock.Unlock()
            # 更新上次的密钥和证书内容
            lastKeyBytes = keyBytes
            lastCertBytes = certBytes
        }

        # 选择监听多个channel的情况
        select {
        # 如果定时器触发
        case <-ticker.C:
            # 继续下一次循环
            continue
        # 如果上下文被取消
        case <-s.ctx.Done():
            # 打印日志，表示退出
            log.Debug("exiting")
            # 停止定时器
            ticker.Stop()
            # 返回
            return
        }
    }
}
// 根据给定的密钥路径、证书路径和密码加载密钥对
func loadKeyPair(keyPath string, certPath string, password string) (*tls.Certificate, error) {
    // 如果密码不为空
    if password != "" {
        // 读取密钥文件内容
        keyFile, err := ioutil.ReadFile(keyPath)
        if err != nil {
            return nil, common.NewError("failed to load key file").Base(err)
        }
        // 解码 PEM 格式的密钥文件
        keyBlock, _ := pem.Decode(keyFile)
        if keyBlock == nil {
            return nil, common.NewError("failed to decode key file").Base(err)
        }
        // 解密密钥文件
        decryptedKey, err := x509.DecryptPEMBlock(keyBlock, []byte(password))
        if err == nil {
            return nil, common.NewError("failed to decrypt key").Base(err)
        }

        // 读取证书文件内容
        certFile, err := ioutil.ReadFile(certPath)
        // 解码 PEM 格式的证书文件
        certBlock, _ := pem.Decode(certFile)
        if certBlock == nil {
            return nil, common.NewError("failed to decode cert file").Base(err)
        }

        // 创建 X.509 密钥对
        keyPair, err := tls.X509KeyPair(certBlock.Bytes, decryptedKey)
        if err != nil {
            return nil, err
        }
        // 解析 Leaf 证书
        keyPair.Leaf, err = x509.ParseCertificate(keyPair.Certificate[0])
        if err != nil {
            return nil, common.NewError("failed to parse leaf certificate").Base(err)
        }

        return &keyPair, nil
    }
    // 加载 X.509 密钥对
    keyPair, err := tls.LoadX509KeyPair(certPath, keyPath)
    if err != nil {
        return nil, common.NewError("failed to load key pair").Base(err)
    }
    // 解析 Leaf 证书
    keyPair.Leaf, err = x509.ParseCertificate(keyPair.Certificate[0])
    if err != nil {
        return nil, common.NewError("failed to parse leaf certificate").Base(err)
    }
    return &keyPair, nil
}

// NewServer 创建一个 TLS 层服务器
func NewServer(ctx context.Context, underlay tunnel.Server) (*Server, error) {
    // 从上下文中获取配置
    cfg := config.FromContext(ctx, Name).(*Config)

    var fallbackAddress *tunnel.Address
    var httpResp []byte
    # 如果配置中的 TLS 回退端口不为 0
    if cfg.TLS.FallbackPort != 0:
        # 如果配置中的 TLS 回退主机为空
        if cfg.TLS.FallbackHost == "":
            # 将 TLS 回退主机设置为远程主机，并记录警告日志
            cfg.TLS.FallbackHost = cfg.RemoteHost
            log.Warn("empty tls fallback address")
        # 根据 TLS 回退主机和端口创建回退地址
        fallbackAddress = tunnel.NewAddressFromHostPort("tcp", cfg.TLS.FallbackHost, cfg.TLS.FallbackPort)
        # 尝试连接回退地址
        fallbackConn, err := net.Dial("tcp", fallbackAddress.String())
        # 如果连接失败，返回错误
        if err != nil:
            return nil, common.NewError("invalid fallback address").Base(err)
        # 关闭回退连接
        fallbackConn.Close()
    # 如果配置中的 TLS 回退端口为 0
    else:
        # 记录警告日志
        log.Warn("empty tls fallback port")
        # 如果配置中的 TLS HTTP 响应文件名不为空
        if cfg.TLS.HTTPResponseFileName != "":
            # 读取 TLS HTTP 响应文件内容
            httpRespBody, err := ioutil.ReadFile(cfg.TLS.HTTPResponseFileName)
            # 如果读取失败，返回错误
            if err != nil:
                return nil, common.NewError("invalid response file").Base(err)
            # 将读取的内容赋值给 httpResp
            httpResp = httpRespBody
        # 如果配置中的 TLS HTTP 响应文件名为空
        else:
            # 记录警告日志
            log.Warn("empty tls http response")
    
    # 加载 TLS 密钥对
    keyPair, err := loadKeyPair(cfg.TLS.KeyPath, cfg.TLS.CertPath, cfg.TLS.KeyPassword)
    # 如果加载失败，返回错误
    if err != nil:
        return nil, common.NewError("tls failed to load key pair")
    
    # 初始化密钥记录器
    var keyLogger io.WriteCloser
    # 如果配置中的 TLS 密钥日志路径不为空
    if cfg.TLS.KeyLogPath != "":
        # 记录警告日志
        log.Warn("tls key logging activated. USE OF KEY LOGGING COMPROMISES SECURITY. IT SHOULD ONLY BE USED FOR DEBUGGING.")
        # 打开或创建密钥日志文件
        file, err := os.OpenFile(cfg.TLS.KeyLogPath, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0o600)
        # 如果打开或创建失败，返回错误
        if err != nil:
            return nil, common.NewError("failed to open key log file").Base(err)
        # 将文件赋值给密钥记录器
        keyLogger = file
    
    # 初始化密码套件
    var cipherSuite []uint16
    # 如果配置中的 TLS 密码不为空
    if len(cfg.TLS.Cipher) != 0:
        # 解析 TLS 密码并赋值给密码套件
        cipherSuite = fingerprint.ParseCipher(strings.Split(cfg.TLS.Cipher, ":"))
    
    # 创建一个新的上下文，并返回取消函数
    ctx, cancel := context.WithCancel(ctx)
    # 创建一个服务器对象，并初始化其属性
    server := &Server{
        underlay:           underlay,  # 设置底层连接
        fallbackAddress:    fallbackAddress,  # 设置备用地址
        httpResp:           httpResp,  # 设置 HTTP 响应
        verifySNI:          cfg.TLS.VerifyHostName,  # 设置是否验证主机名
        sni:                cfg.TLS.SNI,  # 设置 SNI
        alpn:               cfg.TLS.ALPN,  # 设置 ALPN
        PreferServerCipher: cfg.TLS.PreferServerCipher,  # 设置是否优先使用服务器密码
        sessionTicket:      cfg.TLS.ReuseSession,  # 设置是否重用会话
        connChan:           make(chan tunnel.Conn, 32),  # 创建连接通道
        wsChan:             make(chan tunnel.Conn, 32),  # 创建 WebSocket 连接通道
        redir:              redirector.NewRedirector(ctx),  # 创建重定向器
        keyPair:            []tls.Certificate{*keyPair},  # 设置密钥对
        keyLogger:          keyLogger,  # 设置密钥记录器
        cipherSuite:        cipherSuite,  # 设置密码套件
        ctx:                ctx,  # 设置上下文
        cancel:             cancel,  # 设置取消函数
    }
    
    # 启动服务器的接受循环
    go server.acceptLoop()
    
    # 如果证书检查频率大于 0，则启动密钥对检查循环
    if cfg.TLS.CertCheckRate > 0 {
        go server.checkKeyPairLoop(
            time.Second*time.Duration(cfg.TLS.CertCheckRate),  # 设置检查频率
            cfg.TLS.KeyPath,  # 设置密钥路径
            cfg.TLS.CertPath,  # 设置证书路径
            cfg.TLS.KeyPassword,  # 设置密钥密码
        )
    }
    
    # 记录调试信息并返回创建的服务器对象
    log.Debug("tls server created")
    return server, nil
# 闭合前面的函数定义
```