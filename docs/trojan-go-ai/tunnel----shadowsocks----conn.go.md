# `trojan-go\tunnel\shadowsocks\conn.go`

```go
package shadowsocks

import (
    "net"  // 导入网络包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入第三方包
)

type Conn struct {
    aeadConn net.Conn  // 定义一个网络连接对象
    tunnel.Conn  // 嵌入一个tunnel.Conn对象
}

func (c *Conn) Read(p []byte) (n int, err error) {
    return c.aeadConn.Read(p)  // 从aeadConn对象中读取数据
}

func (c *Conn) Write(p []byte) (n int, err error) {
    return c.aeadConn.Write(p)  // 向aeadConn对象中写入数据
}

func (c *Conn) Close() error {
    c.Conn.Close()  // 调用嵌入的Conn对象的Close方法
    return c.aeadConn.Close()  // 关闭aeadConn对象
}

func (c *Conn) Metadata() *tunnel.Metadata {
    return c.Conn.Metadata()  // 获取嵌入的Conn对象的Metadata
}
```