# `trojan-go\tunnel\simplesocks\conn.go`

```
package simplesocks

import (
    "bytes"  // 导入 bytes 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 trojan-go 项目的 common 包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入 trojan-go 项目的 tunnel 包
    "github.com/p4gefau1t/trojan-go/tunnel/trojan"  // 导入 trojan-go 项目的 trojan 包
)

// Conn is a simplesocks connection
type Conn struct {
    tunnel.Conn  // 嵌入 tunnel.Conn 结构体
    metadata      *tunnel.Metadata  // 指向 tunnel.Metadata 结构体的指针
    isOutbound    bool  // 布尔类型的字段，表示是否为出站连接
    headerWritten bool  // 布尔类型的字段，表示是否已经写入头部信息
}

func (c *Conn) Metadata() *tunnel.Metadata {
    return c.metadata  // 返回连接的元数据
}

func (c *Conn) Write(payload []byte) (int, error) {
    if c.isOutbound && !c.headerWritten {  // 如果是出站连接且头部信息未写入
        buf := bytes.NewBuffer(make([]byte, 0, 4096))  // 创建一个缓冲区
        c.metadata.WriteTo(buf)  // 将元数据写入缓冲区
        buf.Write(payload)  // 将有效载荷写入缓冲区
        _, err := c.Conn.Write(buf.Bytes())  // 将缓冲区的内容写入连接
        if err != nil {
            return 0, common.NewError("failed to write simplesocks header").Base(err)  // 如果写入失败，返回错误信息
        }
        c.headerWritten = true  // 标记头部信息已经写入
        return len(payload), nil  // 返回有效载荷的长度和空错误
    }
    return c.Conn.Write(payload)  // 如果不是出站连接或者头部信息已经写入，则直接将有效载荷写入连接
}

// PacketConn is a simplesocks packet connection
// The header syntax is the same as trojan's
type PacketConn struct {
    trojan.PacketConn  // 嵌入 trojan.PacketConn 结构体
}
```