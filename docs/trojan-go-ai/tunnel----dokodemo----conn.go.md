# `trojan-go\tunnel\dokodemo\conn.go`

```
package dokodemo

import (
    "context"
    "net"

    "github.com/p4gefau1t/trojan-go/common"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

const MaxPacketSize = 1024 * 8

type Conn struct {
    net.Conn
    src            *tunnel.Address  // 定义 Conn 结构体中的源地址
    targetMetadata *tunnel.Metadata  // 定义 Conn 结构体中的目标元数据
}

func (c *Conn) Metadata() *tunnel.Metadata {
    return c.targetMetadata  // 返回 Conn 结构体中的目标元数据
}

// PacketConn receive packet info from the packet dispatcher
type PacketConn struct {
    net.PacketConn
    metadata *tunnel.Metadata  // 定义 PacketConn 结构体中的元数据
    input    chan []byte  // 定义 PacketConn 结构体中的输入通道
    output   chan []byte  // 定义 PacketConn 结构体中的输出通道
    src      net.Addr  // 定义 PacketConn 结构体中的源地址
    ctx      context.Context  // 定义 PacketConn 结构体中的上下文
    cancel   context.CancelFunc  // 定义 PacketConn 结构体中的取消函数
}

func (c *PacketConn) Close() error {
    c.cancel()  // 调用取消函数
    // don't close the underlying udp socket
    return nil  // 返回空值
}

func (c *PacketConn) ReadFrom(p []byte) (int, net.Addr, error) {
    return c.ReadWithMetadata(p)  // 调用 ReadWithMetadata 方法
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (int, error) {
    address, err := tunnel.NewAddressFromAddr("udp", addr.String())  // 从地址创建新的地址
    if err != nil {
        return 0, err  // 如果出错，返回错误
    }
    return c.WriteWithMetadata(p, &tunnel.Metadata{
        Address: address,  // 返回使用元数据的写入结果
    })
}

func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
    select {
    case payload := <-c.input:  // 从输入通道中接收数据
        n := copy(p, payload)  // 将接收到的数据复制到指定的字节切片中
        return n, c.metadata, nil  // 返回复制的字节数和元数据
    case <-c.ctx.Done():  // 如果上下文被取消
        return 0, nil, common.NewError("dokodemo packet conn closed")  // 返回错误信息
    }
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
    select {
    case c.output <- p:  // 将数据写入输出通道
        return len(p), nil  // 返回写入的字节数和空值
    case <-c.ctx.Done():  // 如果上下文被取消
        return 0, common.NewError("dokodemo packet conn failed to write")  // 返回错误信息
    }
}
```