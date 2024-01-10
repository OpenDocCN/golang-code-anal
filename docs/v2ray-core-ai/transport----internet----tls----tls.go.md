# `v2ray-core\transport\internet\tls\tls.go`

```
// +build !confonly

package tls

import (
    "crypto/tls"  // 导入加密通信包

    "v2ray.com/core/common/buf"  // 导入数据缓冲包
    "v2ray.com/core/common/net"  // 导入网络通信包
)

//go:generate go run v2ray.com/core/common/errors/errorgen

var (
    _ buf.Writer = (*Conn)(nil)  // 确保 Conn 类型实现了 buf.Writer 接口
)

type Conn struct {
    *tls.Conn  // TLS 连接对象
}

func (c *Conn) WriteMultiBuffer(mb buf.MultiBuffer) error {
    mb = buf.Compact(mb)  // 压缩多个数据缓冲
    mb, err := buf.WriteMultiBuffer(c, mb)  // 将多个数据缓冲写入连接
    buf.ReleaseMulti(mb)  // 释放多个数据缓冲
    return err  // 返回错误信息
}

func (c *Conn) HandshakeAddress() net.Address {
    if err := c.Handshake(); err != nil {  // 进行 TLS 握手
        return nil  // 如果出错，返回空地址
    }
    state := c.ConnectionState()  // 获取连接状态
    if state.ServerName == "" {  // 如果服务器名称为空
        return nil  // 返回空地址
    }
    return net.ParseAddress(state.ServerName)  // 解析服务器名称为网络地址
}

// Client initiates a TLS client handshake on the given connection.
func Client(c net.Conn, config *tls.Config) net.Conn {
    tlsConn := tls.Client(c, config)  // 创建 TLS 客户端连接
    return &Conn{Conn: tlsConn}  // 返回 TLS 连接对象
}

// Server initiates a TLS server handshake on the given connection.
func Server(c net.Conn, config *tls.Config) net.Conn {
    tlsConn := tls.Server(c, config)  // 创建 TLS 服务器连接
    return &Conn{Conn: tlsConn}  // 返回 TLS 连接对象
}
```