# `v2ray-core\transport\internet\quic\hub.go`

```
// +build !confonly

package quic

import (
    "context"
    "time"

    "github.com/lucas-clemente/quic-go"
    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol/tls/cert"
    "v2ray.com/core/common/signal/done"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
)

// Listener is an internet.Listener that listens for TCP connections.
type Listener struct {
    rawConn  *sysConn
    listener quic.Listener
    done     *done.Instance
    addConn  internet.ConnHandler
}

func (l *Listener) acceptStreams(session quic.Session) {
    for {
        stream, err := session.AcceptStream(context.Background())
        if err != nil {
            // 如果无法接受流，则记录错误并根据情况进行处理
            newError("failed to accept stream").Base(err).WriteToLog()
            select {
            case <-session.Context().Done():
                return
            case <-l.done.Wait():
                if err := session.CloseWithError(0, ""); err != nil {
                    newError("failed to close session").Base(err).WriteToLog()
                }
                return
            default:
                time.Sleep(time.Second)
                continue
            }
        }

        // 创建一个新的连接对象，并将其添加到连接处理器中
        conn := &interConn{
            stream: stream,
            local:  session.LocalAddr(),
            remote: session.RemoteAddr(),
        }

        l.addConn(conn)
    }

}

func (l *Listener) keepAccepting() {
    for {
        conn, err := l.listener.Accept(context.Background())
        if err != nil {
            // 如果无法接受 QUIC 会话，则记录错误并根据情况进行处理
            newError("failed to accept QUIC sessions").Base(err).WriteToLog()
            if l.done.Done() {
                break
            }
            time.Sleep(time.Second)
            continue
        }
        // 启动一个新的 goroutine 处理接受到的会话
        go l.acceptStreams(conn)
    }
}

// Addr implements internet.Listener.Addr.
func (l *Listener) Addr() net.Addr {
    return l.listener.Addr()
}

// Close implements internet.Listener.Close.
func (l *Listener) Close() error {
    // 关闭监听器
    l.done.Close()
    # 关闭监听器
    l.listener.Close()
    # 关闭原始连接
    l.rawConn.Close()
    # 返回空值
    return nil
// Listen函数基于配置创建一个新的Listener
func Listen(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, handler internet.ConnHandler) (internet.Listener, error) {
    // 如果地址是域名类型，则不允许监听quic
    if address.Family().IsDomain() {
        return nil, newError("domain address is not allows for listening quic")
    }

    // 根据流设置创建TLS配置
    tlsConfig := tls.ConfigFromStreamSettings(streamSettings)
    if tlsConfig == nil {
        // 如果TLS配置为空，则使用默认配置
        tlsConfig = &tls.Config{
            Certificate: []*tls.Certificate{tls.ParseCertificate(cert.MustGenerate(nil, cert.DNSNames(internalDomain), cert.CommonName(internalDomain)))},
        }
    }

    // 获取协议配置
    config := streamSettings.ProtocolSettings.(*Config)
    // 使用系统包监听UDP地址
    rawConn, err := internet.ListenSystemPacket(context.Background(), &net.UDPAddr{
        IP:   address.IP(),
        Port: int(port),
    }, streamSettings.SocketSettings)

    if err != nil {
        return nil, err
    }

    // 配置quic参数
    quicConfig := &quic.Config{
        ConnectionIDLength:    12,
        HandshakeTimeout:      time.Second * 8,
        MaxIdleTimeout:        time.Second * 45,
        MaxIncomingStreams:    32,
        MaxIncomingUniStreams: -1,
    }

    // 将系统连接包装成quic连接
    conn, err := wrapSysConn(rawConn, config)
    if err != nil {
        conn.Close()
        return nil, err
    }

    // 使用quic监听器监听连接
    qListener, err := quic.Listen(conn, tlsConfig.GetTLSConfig(), quicConfig)
    if err != nil {
        conn.Close()
        return nil, err
    }

    // 创建Listener对象
    listener := &Listener{
        done:     done.New(),
        rawConn:  conn,
        listener: qListener,
        addConn:  handler,
    }

    // 启动接受连接的goroutine
    go listener.keepAccepting()

    return listener, nil
}

// 在init函数中注册传输监听器
func init() {
    common.Must(internet.RegisterTransportListener(protocolName, Listen))
}
```