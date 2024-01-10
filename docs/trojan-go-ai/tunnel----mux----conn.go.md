# `trojan-go\tunnel\mux\conn.go`

```
package mux

import (
    "io"
    "math/rand"

    "github.com/p4gefau1t/trojan-go/log"
    "github.com/p4gefau1t/trojan-go/tunnel"
)

type stickyConn struct {
    tunnel.Conn
    synQueue chan []byte  // 用于存放同步消息的通道
    finQueue chan []byte  // 用于存放结束消息的通道
}

func (c *stickyConn) stickToPayload(p []byte) []byte {
    buf := make([]byte, 0, len(p)+16)  // 创建一个初始容量为 len(p)+16 的空字节切片
    for {
        select {
        case header := <-c.synQueue:  // 从同步消息通道中取出消息
            buf = append(buf, header...)  // 将消息追加到 buf 中
        default:
            goto stick1  // 跳转到标签 stick1
        }
    }
stick1:
    buf = append(buf, p...)  // 将 p 追加到 buf 中
    for {
        select {
        case header := <-c.finQueue:  // 从结束消息通道中取出消息
            buf = append(buf, header...)  // 将消息追加到 buf 中
        default:
            goto stick2  // 跳转到标签 stick2
        }
    }
stick2:
    return buf  // 返回 buf
}

func (c *stickyConn) Close() error {
    const maxPaddingLength = 512  // 定义最大填充长度为 512
    padding := [maxPaddingLength + 8]byte{'A', 'B', 'C', 'D', 'E', 'F'}  // 创建一个长度为 maxPaddingLength+8 的填充字节数组
    buf := c.stickToPayload(nil)  // 调用 stickToPayload 方法，传入空字节切片，返回结果赋值给 buf
    c.Write(append(buf, padding[:rand.Intn(maxPaddingLength)]...))  // 将 buf 和随机长度的填充追加到一起，然后写入连接
    return c.Conn.Close()  // 调用连接的 Close 方法
}

func (c *stickyConn) Write(p []byte) (int, error) {
    if len(p) == 8 {  // 如果 p 的长度为 8
        if p[0] == 1 || p[0] == 2 {  // 如果 p 的第一个字节为 1 或 2
            switch p[1] {
            // THE CONTENT OF THE BUFFER MIGHT CHANGE
            // NEVER STORE THE POINTER TO HEADER, COPY THE HEADER INSTEAD
            case 0:
                // cmdSYN
                header := make([]byte, 8)  // 创建一个长度为 8 的字节切片
                copy(header, p)  // 将 p 复制到 header 中
                c.synQueue <- header  // 将 header 写入同步消息通道
                return 8, nil  // 返回写入的字节数和 nil
            case 1:
                // cmdFIN
                header := make([]byte, 8)  // 创建一个长度为 8 的字节切片
                copy(header, p)  // 将 p 复制到 header 中
                c.finQueue <- header  // 将 header 写入结束消息通道
                return 8, nil  // 返回写入的字节数和 nil
            }
        } else {
            log.Debug("other 8 bytes header")  // 打印调试信息
        }
    }
    _, err := c.Conn.Write(c.stickToPayload(p))  // 调用 stickToPayload 方法，传入 p，然后将结果写入连接
    return len(p), err  // 返回写入的字节数和错误
}

func newStickyConn(conn tunnel.Conn) *stickyConn {
    # 返回一个stickyConn对象，该对象包含Conn、synQueue和finQueue属性
    return &stickyConn{
        # 将传入的conn赋值给Conn属性
        Conn:     conn,
        # 创建一个最大容量为128的通道，并赋值给synQueue属性
        synQueue: make(chan []byte, 128),
        # 创建一个最大容量为128的通道，并赋值给finQueue属性
        finQueue: make(chan []byte, 128),
    }
}
// 定义 Conn 结构体，包含一个读写关闭接口和一个隧道连接
type Conn struct {
    rwc io.ReadWriteCloser
    tunnel.Conn
}

// 重写 Conn 结构体的 Read 方法，调用 rwc 的 Read 方法
func (c *Conn) Read(p []byte) (int, error) {
    return c.rwc.Read(p)
}

// 重写 Conn 结构体的 Write 方法，调用 rwc 的 Write 方法
func (c *Conn) Write(p []byte) (int, error) {
    return c.rwc.Write(p)
}

// 重写 Conn 结构体的 Close 方法，调用 rwc 的 Close 方法
func (c *Conn) Close() error {
    return c.rwc.Close()
}
```