# `trojan-go\tunnel\socks\conn.go`

```
package socks

import (
    "context"  // 导入上下文包
    "net"      // 导入网络包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入自定义包
)

type Conn struct {
    net.Conn         // 定义 Conn 结构体，包含 net.Conn 类型的匿名字段
    metadata *tunnel.Metadata  // 定义 metadata 字段，类型为 *tunnel.Metadata
}

func (c *Conn) Metadata() *tunnel.Metadata {
    return c.metadata  // 返回 Conn 结构体中的 metadata 字段
}

type packetInfo struct {
    metadata *tunnel.Metadata  // 定义 metadata 字段，类型为 *tunnel.Metadata
    payload  []byte            // 定义 payload 字段，类型为 []byte
}

type PacketConn struct {
    net.PacketConn         // 定义 PacketConn 结构体，包含 net.PacketConn 类型的匿名字段
    input  chan *packetInfo  // 定义 input 字段，类型为 chan *packetInfo
    output chan *packetInfo  // 定义 output 字段，类型为 chan *packetInfo
    src    net.Addr          // 定义 src 字段，类型为 net.Addr
    ctx    context.Context   // 定义 ctx 字段，类型为 context.Context
    cancel context.CancelFunc  // 定义 cancel 字段，类型为 context.CancelFunc
}

func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
    panic("implement me")  // 读取数据到 p 中，返回读取的字节数、地址和错误信息
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
    panic("implement me")  // 将数据 p 写入到地址 addr，返回写入的字节数和错误信息
}

func (c *PacketConn) Close() error {
    c.cancel()  // 调用 cancel 函数
    return nil  // 返回空错误
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
    select {
    case c.output <- &packetInfo{  // 将 packetInfo 结构体指针发送到 output 通道
        metadata: m,  // 设置 metadata 字段值为 m
        payload:  p,  // 设置 payload 字段值为 p
    }:
        return len(p), nil  // 返回写入的字节数和空错误
    case <-c.ctx.Done():
        return 0, common.NewError("socks packet conn closed")  // 返回 0 和自定义错误信息
    }
}

func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
    select {
    case info := <-c.input:  // 从 input 通道接收 packetInfo 结构体
        n := copy(p, info.payload)  // 将 info.payload 复制到 p 中
        return n, info.metadata, nil  // 返回复制的字节数、metadata 和空错误
    case <-c.ctx.Done():
        return 0, nil, common.NewError("socks packet conn closed")  // 返回 0、nil 和自定义错误信息
    }
}
```