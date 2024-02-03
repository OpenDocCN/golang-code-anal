# `trojan-go\tunnel\tls\client.go`

```go
package tls

import (
    "context"
    "crypto/tls"
    "crypto/x509"
    "encoding/pem"
    "io"
    "io/ioutil"
    "strings"

    utls "github.com/refraction-networking/utls"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/config"
    "github.com/p4gefau1t/trojan-go/log"
    "github.com/p4gefau1t/trojan-go/tunnel"
    "github.com/p4gefau1t/trojan-go/tunnel/tls/fingerprint"
    "github.com/p4gefau1t/trojan-go/tunnel/transport"
)

// Client is a tls client
type Client struct {
    verify        bool        // 是否验证服务器证书
    sni           string      // 服务器名称指示（Server Name Indication）
    ca            *x509.CertPool   // 证书授权
    cipher        []uint16    // 加密套件
    sessionTicket bool        // 是否启用会话票据
    reuseSession  bool        // 是否重用会话
    fingerprint   string      // 指纹
    helloID       utls.ClientHelloID   // 客户端 Hello ID
    keyLogger     io.WriteCloser   // 密钥记录器
    underlay      tunnel.Client   // 底层通道
}

func (c *Client) Close() error {
    if c.keyLogger != nil {
        c.keyLogger.Close()   // 关闭密钥记录器
    }
    return c.underlay.Close()   // 关闭底层通道
}

func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")   // 不支持的操作
}

func (c *Client) DialConn(_ *tunnel.Address, overlay tunnel.Tunnel) (tunnel.Conn, error) {
    conn, err := c.underlay.DialConn(nil, &Tunnel{})   // 使用底层通道建立连接
    if err != nil {
        return nil, common.NewError("tls failed to dial conn").Base(err)   // 返回连接错误
    }

    if c.fingerprint != "" {
        // utls fingerprint
        tlsConn := utls.UClient(conn, &utls.Config{
            RootCAs:            c.ca,   // 设置根证书
            ServerName:         c.sni,  // 设置服务器名称指示
            InsecureSkipVerify: !c.verify,   // 设置是否跳过验证
            KeyLogWriter:       c.keyLogger,   // 设置密钥记录器
        }, c.helloID)
        if err := tlsConn.Handshake(); err != nil {
            return nil, common.NewError("tls failed to handshake with remote server").Base(err)   // 返回握手错误
        }
        return &transport.Conn{
            Conn: tlsConn,   // 返回传输连接
        }, nil
    }
    // golang default tls library
}
    # 使用传入的连接和TLS配置创建一个TLS连接
    tlsConn := tls.Client(conn, &tls.Config{
        # 设置是否跳过证书验证
        InsecureSkipVerify:     !c.verify,
        # 设置服务器名称
        ServerName:             c.sni,
        # 设置根证书
        RootCAs:                c.ca,
        # 设置密钥日志记录器
        KeyLogWriter:           c.keyLogger,
        # 设置密码套件
        CipherSuites:           c.cipher,
        # 设置是否禁用会话票据
        SessionTicketsDisabled: !c.sessionTicket,
    })
    # 进行TLS握手
    err = tlsConn.Handshake()
    # 如果握手失败，则返回错误
    if err != nil {
        return nil, common.NewError("tls failed to handshake with remote server").Base(err)
    }
    # 返回TLS连接
    return &transport.Conn{
        Conn: tlsConn,
    }, nil
// NewClient函数用于创建一个TLS客户端
func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)

    // 初始化客户端helloID
    helloID := utls.ClientHelloID{}
    // 如果配置中指定了TLS指纹
    if cfg.TLS.Fingerprint != "" {
        // 根据不同的指纹类型设置helloID
        switch cfg.TLS.Fingerprint {
        case "firefox":
            helloID = utls.HelloFirefox_Auto
        case "chrome":
            helloID = utls.HelloChrome_Auto
        case "ios":
            helloID = utls.HelloIOS_Auto
        default:
            // 如果指纹类型无效，则返回错误
            return nil, common.NewError("invalid fingerprint " + cfg.TLS.Fingerprint)
        }
        // 输出应用了的TLS指纹信息
        log.Info("tls fingerprint", cfg.TLS.Fingerprint, "applied")
    }

    // 如果配置中未指定SNI，则使用远程主机作为SNI
    if cfg.TLS.SNI == "" {
        cfg.TLS.SNI = cfg.RemoteHost
        // 输出警告信息，SNI未指定
        log.Warn("tls sni is unspecified")
    }

    // 创建客户端对象并初始化
    client := &Client{
        underlay:      underlay,
        verify:        cfg.TLS.Verify,
        sni:           cfg.TLS.SNI,
        cipher:        fingerprint.ParseCipher(strings.Split(cfg.TLS.Cipher, ":")),
        sessionTicket: cfg.TLS.ReuseSession,
        fingerprint:   cfg.TLS.Fingerprint,
        helloID:       helloID,
    }
}
    // 如果配置中指定了证书路径
    if cfg.TLS.CertPath != "" {
        // 读取证书文件内容
        caCertByte, err := ioutil.ReadFile(cfg.TLS.CertPath)
        // 如果读取出错，返回错误信息
        if err != nil {
            return nil, common.NewError("failed to load cert file").Base(err)
        }
        // 创建证书池
        client.ca = x509.NewCertPool()
        // 将证书内容添加到证书池中
        ok := client.ca.AppendCertsFromPEM(caCertByte)
        // 如果添加失败，输出警告信息
        if !ok {
            log.Warn("invalid cert list")
        }
        // 输出信息，表示正在使用自定义证书
        log.Info("using custom cert")

        // 打印证书信息
        pemCerts := caCertByte
        for len(pemCerts) > 0 {
            var block *pem.Block
            // 解析证书内容
            block, pemCerts = pem.Decode(pemCerts)
            if block == nil {
                break
            }
            // 检查证书类型和头部信息
            if block.Type != "CERTIFICATE" || len(block.Headers) != 0 {
                continue
            }
            // 解析证书并输出发行者和主题信息
            cert, err := x509.ParseCertificate(block.Bytes)
            if err != nil {
                continue
            }
            log.Trace("issuer:", cert.Issuer, "subject:", cert.Subject)
        }
    }

    // 如果配置中未指定证书路径
    if cfg.TLS.CertPath == "" {
        // 输出信息，表示未指定证书，使用默认证书列表
        log.Info("cert is unspecified, using default ca list")
    }

    // 输出调试信息，表示 TLS 客户端已创建
    log.Debug("tls client created")
    // 返回创建的 TLS 客户端和空值
    return client, nil
# 闭合前面的函数定义
```