# `v2ray-core\transport\internet\xtls\xtls.go`

```
// +build !confonly

package xtls

import (
    xtls "github.com/xtls/go"  // 导入 xtls 包，别名为 xtls

    "v2ray.com/core/common/buf"  // 导入 buf 包
    "v2ray.com/core/common/net"  // 导入 net 包
)

//go:generate go run v2ray.com/core/common/errors/errorgen  // 使用 go:generate 命令生成错误处理代码

var (
    _ buf.Writer = (*Conn)(nil)  // 确保 Conn 类型实现了 buf.Writer 接口
)

type Conn struct {
    *xtls.Conn  // 包含 xtls.Conn 类型的匿名字段
}

func (c *Conn) WriteMultiBuffer(mb buf.MultiBuffer) error {
    mb = buf.Compact(mb)  // 压缩多缓冲区
    mb, err := buf.WriteMultiBuffer(c, mb)  // 将多缓冲区写入连接
    buf.ReleaseMulti(mb)  // 释放多缓冲区
    return err  // 返回错误
}

func (c *Conn) HandshakeAddress() net.Address {
    if err := c.Handshake(); err != nil {  // 进行握手，如果出错则返回空
        return nil
    }
    state := c.ConnectionState()  // 获取连接状态
    if state.ServerName == "" {  // 如果服务器名称为空，则返回空
        return nil
    }
    return net.ParseAddress(state.ServerName)  // 解析服务器名称为网络地址
}

// Client initiates a XTLS client handshake on the given connection.
func Client(c net.Conn, config *xtls.Config) net.Conn {
    xtlsConn := xtls.Client(c, config)  // 在给定连接上初始化 XTLS 客户端握手
    return &Conn{Conn: xtlsConn}  // 返回包含 xtlsConn 的 Conn 类型
}

// Server initiates a XTLS server handshake on the given connection.
func Server(c net.Conn, config *xtls.Config) net.Conn {
    xtlsConn := xtls.Server(c, config)  // 在给定连接上初始化 XTLS 服务器握手
    return &Conn{Conn: xtlsConn}  // 返回包含 xtlsConn 的 Conn 类型
}
```