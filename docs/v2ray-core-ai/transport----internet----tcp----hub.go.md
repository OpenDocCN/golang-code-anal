# `v2ray-core\transport\internet\tcp\hub.go`

```
// +build !confonly

package tcp

import (
    "context"
    gotls "crypto/tls"
    "strings"
    "time"

    "github.com/pires/go-proxyproto"
    goxtls "github.com/xtls/go"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
    "v2ray.com/core/transport/internet/xtls"
)

// Listener is an internet.Listener that listens for TCP connections.
type Listener struct {
    listener   net.Listener          // TCP连接监听器
    tlsConfig  *gotls.Config         // TLS配置
    xtlsConfig *goxtls.Config        // XTLS配置
    authConfig internet.ConnectionAuthenticator  // 连接认证器
    config     *Config               // TCP配置
    addConn    internet.ConnHandler  // 连接处理器
}

// ListenTCP creates a new Listener based on configurations.
func ListenTCP(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, handler internet.ConnHandler) (internet.Listener, error) {
    // 在指定地址和端口上创建TCP监听器
    listener, err := internet.ListenSystem(ctx, &net.TCPAddr{
        IP:   address.IP(),
        Port: int(port),
    }, streamSettings.SocketSettings)
    if err != nil {
        return nil, newError("failed to listen TCP on", address, ":", port).Base(err)
    }
    newError("listening TCP on ", address, ":", port).WriteToLog(session.ExportIDToError(ctx))

    tcpSettings := streamSettings.ProtocolSettings.(*Config)
    var l *Listener

    if tcpSettings.AcceptProxyProtocol {
        // 设置代理协议策略函数
        policyFunc := func(upstream net.Addr) (proxyproto.Policy, error) { return proxyproto.REQUIRE, nil }
        // 如果接受代理协议，则创建带有代理协议的监听器
        l = &Listener{
            listener: &proxyproto.Listener{Listener: listener, Policy: policyFunc},
            config:   tcpSettings,
            addConn:  handler,
        }
        newError("accepting PROXY protocol").AtWarning().WriteToLog(session.ExportIDToError(ctx))
    } else {
        // 否则创建普通监听器
        l = &Listener{
            listener: listener,
            config:   tcpSettings,
            addConn:  handler,
        }
    }
    # 如果根据流设置生成了 TLS 配置，则将其赋值给 l.tlsConfig
    if config := tls.ConfigFromStreamSettings(streamSettings); config != nil {
        l.tlsConfig = config.GetTLSConfig(tls.WithNextProto("h2"))
    }
    # 如果根据流设置生成了 XTLS 配置，则将其赋值给 l.xtlsConfig
    if config := xtls.ConfigFromStreamSettings(streamSettings); config != nil {
        l.xtlsConfig = config.GetXTLSConfig(xtls.WithNextProto("h2"))
    }

    # 如果 TCP 设置中包含头部设置
    if tcpSettings.HeaderSettings != nil {
        # 获取头部设置的实例
        headerConfig, err := tcpSettings.HeaderSettings.GetInstance()
        # 如果获取实例时出现错误，则返回错误信息
        if err != nil {
            return nil, newError("invalid header settings").Base(err).AtError()
        }
        # 创建连接认证器
        auth, err := internet.CreateConnectionAuthenticator(headerConfig)
        # 如果创建认证器时出现错误，则返回错误信息
        if err != nil {
            return nil, newError("invalid header settings.").Base(err).AtError()
        }
        # 将认证配置赋值给 l.authConfig
        l.authConfig = auth
    }

    # 启动 keepAccepting 协程
    go l.keepAccepting()
    # 返回 l 和 nil
    return l, nil
// 保持接受连接的循环
func (v *Listener) keepAccepting() {
    for {
        // 接受连接
        conn, err := v.listener.Accept()
        // 如果出现错误
        if err != nil {
            // 获取错误信息
            errStr := err.Error()
            // 如果错误信息包含 "closed"，则跳出循环
            if strings.Contains(errStr, "closed") {
                break
            }
            // 记录错误日志
            newError("failed to accepted raw connections").Base(err).AtWarning().WriteToLog()
            // 如果错误信息包含 "too many"，则休眠500毫秒
            if strings.Contains(errStr, "too many") {
                time.Sleep(time.Millisecond * 500)
            }
            // 继续下一次循环
            continue
        }

        // 如果存在 TLS 配置，则使用 TLS 服务器
        if v.tlsConfig != nil {
            conn = tls.Server(conn, v.tlsConfig)
        } else if v.xtlsConfig != nil {
            // 如果存在 XTLS 配置，则使用 XTLS 服务器
            conn = xtls.Server(conn, v.xtlsConfig)
        }
        // 如果存在认证配置，则使用认证服务器
        if v.authConfig != nil {
            conn = v.authConfig.Server(conn)
        }

        // 将连接添加到连接列表中
        v.addConn(internet.Connection(conn))
    }
}

// Addr 实现了 internet.Listener.Addr 接口
func (v *Listener) Addr() net.Addr {
    // 返回监听器的地址
    return v.listener.Addr()
}

// Close 实现了 internet.Listener.Close 接口
func (v *Listener) Close() error {
    // 关闭监听器
    return v.listener.Close()
}

// 初始化函数
func init() {
    // 注册传输监听器
    common.Must(internet.RegisterTransportListener(protocolName, ListenTCP))
}
```