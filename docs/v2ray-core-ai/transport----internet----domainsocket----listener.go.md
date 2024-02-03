# `v2ray-core\transport\internet\domainsocket\listener.go`

```go
// 根据构建标签，排除 Windows、WASM 和 confonly 环境
// 定义 domainsocket 包
package domainsocket

// 导入所需的包
import (
    "context"
    gotls "crypto/tls"
    "os"
    "strings"

    "github.com/pires/go-proxyproto"
    goxtls "github.com/xtls/go"
    "golang.org/x/sys/unix"

    "v2ray.com/core/common"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/session"
    "v2ray.com/core/transport/internet"
    "v2ray.com/core/transport/internet/tls"
    "v2ray.com/core/transport/internet/xtls"
)

// 定义 Listener 结构体
type Listener struct {
    addr       *net.UnixAddr
    ln         net.Listener
    tlsConfig  *gotls.Config
    xtlsConfig *goxtls.Config
    config     *Config
    addConn    internet.ConnHandler
    locker     *fileLocker
}

// Listen 函数，用于监听 Unix 域套接字
func Listen(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, handler internet.ConnHandler) (internet.Listener, error) {
    // 从 streamSettings 中获取配置信息
    settings := streamSettings.ProtocolSettings.(*Config)
    // 获取 Unix 地址
    addr, err := settings.GetUnixAddr()
    if err != nil {
        return nil, err
    }

    // 在 Unix 域上监听指定地址
    unixListener, err := net.ListenUnix("unix", addr)
    if err != nil {
        return nil, newError("failed to listen domain socket").Base(err).AtWarning()
    }

    var ln *Listener
    // 如果配置中需要接受代理协议
    if settings.AcceptProxyProtocol {
        // 定义代理协议策略函数
        policyFunc := func(upstream net.Addr) (proxyproto.Policy, error) { return proxyproto.REQUIRE, nil }
        // 创建 Listener 对象，使用代理协议监听 Unix 域套接字
        ln = &Listener{
            addr:    addr,
            ln:      &proxyproto.Listener{Listener: unixListener, Policy: policyFunc},
            config:  settings,
            addConn: handler,
        }
        // 记录接受代理协议的日志
        newError("accepting PROXY protocol").AtWarning().WriteToLog(session.ExportIDToError(ctx))
    } else {
        // 创建普通的 Listener 对象，监听 Unix 域套接字
        ln = &Listener{
            addr:    addr,
            ln:      unixListener,
            config:  settings,
            addConn: handler,
        }
    }
    # 如果不是抽象的设置，则创建文件锁对象
    if !settings.Abstract {
        ln.locker = &fileLocker{
            path: settings.Path + ".lock",
        }
        # 尝试获取文件锁，如果失败则关闭 UnixListener 并返回错误
        if err := ln.locker.Acquire(); err != nil {
            unixListener.Close()
            return nil, err
        }
    }

    # 根据流设置创建 TLS 配置
    if config := tls.ConfigFromStreamSettings(streamSettings); config != nil {
        ln.tlsConfig = config.GetTLSConfig()
    }
    # 根据流设置创建 XTLS 配置
    if config := xtls.ConfigFromStreamSettings(streamSettings); config != nil {
        ln.xtlsConfig = config.GetXTLSConfig()
    }

    # 启动监听器的运行
    go ln.run()

    # 返回监听器对象和空错误
    return ln, nil
# 返回监听器的地址
func (ln *Listener) Addr() net.Addr {
    return ln.addr
}

# 关闭监听器，释放锁并关闭底层连接
func (ln *Listener) Close() error {
    # 如果存在锁对象，则释放锁
    if ln.locker != nil {
        ln.locker.Release()
    }
    # 关闭底层连接
    return ln.ln.Close()
}

# 运行监听器，接受传入的连接
func (ln *Listener) run() {
    # 循环接受传入的连接
    for {
        conn, err := ln.ln.Accept()
        # 如果出现错误
        if err != nil {
            # 如果错误信息包含 "closed"，则跳出循环
            if strings.Contains(err.Error(), "closed") {
                break
            }
            # 记录警告日志并继续循环
            newError("failed to accepted raw connections").Base(err).AtWarning().WriteToLog()
            continue
        }

        # 如果存在 TLS 配置，则使用 TLS 服务器包装连接
        if ln.tlsConfig != nil {
            conn = tls.Server(conn, ln.tlsConfig)
        } 
        # 如果存在 XTLS 配置，则使用 XTLS 服务器包装连接
        else if ln.xtlsConfig != nil {
            conn = xtls.Server(conn, ln.xtlsConfig)
        }

        # 将连接添加到监听器的连接列表中
        ln.addConn(internet.Connection(conn))
    }
}

# 文件锁对象
type fileLocker struct {
    path string
    file *os.File
}

# 获取文件锁
func (fl *fileLocker) Acquire() error {
    # 创建文件
    f, err := os.Create(fl.path)
    if err != nil {
        return err
    }
    # 对文件加锁
    if err := unix.Flock(int(f.Fd()), unix.LOCK_EX); err != nil {
        f.Close()
        return newError("failed to lock file: ", fl.path).Base(err)
    }
    fl.file = f
    return nil
}

# 释放文件锁
func (fl *fileLocker) Release() {
    # 解锁文件
    if err := unix.Flock(int(fl.file.Fd()), unix.LOCK_UN); err != nil {
        newError("failed to unlock file: ", fl.path).Base(err).WriteToLog()
    }
    # 关闭文件
    if err := fl.file.Close(); err != nil {
        newError("failed to close file: ", fl.path).Base(err).WriteToLog()
    }
    # 删除文件
    if err := os.Remove(fl.path); err != nil {
        newError("failed to remove file: ", fl.path).Base(err).WriteToLog()
    }
}

# 初始化函数
func init() {
    # 注册传输监听器
    common.Must(internet.RegisterTransportListener(protocolName, Listen))
}
```