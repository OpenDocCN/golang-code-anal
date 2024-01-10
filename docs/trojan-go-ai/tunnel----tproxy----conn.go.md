# `trojan-go\tunnel\tproxy\conn.go`

```
//go:build linux
// +build linux

// 定义了 tproxy 包，用于处理 Linux 下的透明代理功能
package tproxy

import (
    "context"
    "net"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

// 定义 Conn 结构体，包含 net.Conn 和 tunnel.Metadata
type Conn struct {
    net.Conn
    metadata *tunnel.Metadata
}

// 返回 Conn 结构体中的 metadata 字段
func (c *Conn) Metadata() *tunnel.Metadata {
    return c.metadata
}

// 定义 packetInfo 结构体，包含 tunnel.Metadata 和 payload
type packetInfo struct {
    metadata *tunnel.Metadata
    payload  []byte
}

// 定义 PacketConn 结构体，包含 net.PacketConn、input、output、src、ctx 和 cancel
type PacketConn struct {
    net.PacketConn
    input  chan *packetInfo
    output chan *packetInfo
    src    net.Addr
    ctx    context.Context
    cancel context.CancelFunc
}

// 从 PacketConn 中读取数据到 p 中，并返回读取的字节数、地址和错误
func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
    panic("implement me")
}

// 将 p 中的数据写入 PacketConn，并返回写入的字节数和错误
func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
    panic("implement me")
}

// 关闭 PacketConn 连接
func (c *PacketConn) Close() error {
    c.cancel()
    return nil
}

// 带有元数据的写入操作，将数据和元数据写入 PacketConn
func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
    select {
    case c.output <- &packetInfo{
        metadata: m,
        payload:  p,
    }:
        return len(p), nil
    case <-c.ctx.Done():
        return 0, common.NewError("socks packet conn closed")
    }
}

// 带有元数据的读取操作，从 PacketConn 中读取数据和元数据
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
    select {
    case info := <-c.input:
        n := copy(p, info.payload)
        return n, info.metadata, nil
    case <-c.ctx.Done():
        return 0, nil, common.NewError("socks packet conn closed")
    }
}
```