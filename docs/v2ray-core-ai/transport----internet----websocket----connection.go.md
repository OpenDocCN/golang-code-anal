# `v2ray-core\transport\internet\websocket\connection.go`

```
// +build !confonly
// 设置构建标记，表示该文件不仅仅是配置文件

package websocket
// 声明包名为websocket

import (
    "io"
    "net"
    "time"

    "github.com/gorilla/websocket"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/errors"
    "v2ray.com/core/common/serial"
)
// 导入所需的包

var (
    _ buf.Writer = (*connection)(nil)
)
// 声明一个变量，用于实现buf.Writer接口

// connection is a wrapper for net.Conn over WebSocket connection.
type connection struct {
    conn       *websocket.Conn
    reader     io.Reader
    remoteAddr net.Addr
}
// 定义connection结构体，用于封装WebSocket连接的net.Conn

func newConnection(conn *websocket.Conn, remoteAddr net.Addr) *connection {
    return &connection{
        conn:       conn,
        remoteAddr: remoteAddr,
    }
}
// 定义一个新的连接函数，返回一个connection对象

// Read implements net.Conn.Read()
func (c *connection) Read(b []byte) (int, error) {
    for {
        reader, err := c.getReader()
        if err != nil {
            return 0, err
        }

        nBytes, err := reader.Read(b)
        if errors.Cause(err) == io.EOF {
            c.reader = nil
            continue
        }
        return nBytes, err
    }
}
// 实现net.Conn.Read()方法，用于读取数据

func (c *connection) getReader() (io.Reader, error) {
    if c.reader != nil {
        return c.reader, nil
    }

    _, reader, err := c.conn.NextReader()
    if err != nil {
        return nil, err
    }
    c.reader = reader
    return reader, nil
}
// 获取读取器，用于读取数据

// Write implements io.Writer.
func (c *connection) Write(b []byte) (int, error) {
    if err := c.conn.WriteMessage(websocket.BinaryMessage, b); err != nil {
        return 0, err
    }
    return len(b), nil
}
// 实现io.Writer接口的Write方法，用于写入数据

func (c *connection) WriteMultiBuffer(mb buf.MultiBuffer) error {
    mb = buf.Compact(mb)
    mb, err := buf.WriteMultiBuffer(c, mb)
    buf.ReleaseMulti(mb)
    return err
}
// 写入多个缓冲区的数据

func (c *connection) Close() error {
    var errors []interface{}
    if err := c.conn.WriteControl(websocket.CloseMessage, websocket.FormatCloseMessage(websocket.CloseNormalClosure, ""), time.Now().Add(time.Second*5)); err != nil {
        errors = append(errors, err)
    }
    if err := c.conn.Close(); err != nil {
        errors = append(errors, err)
    }
}
// 关闭连接
    # 如果错误列表的长度大于0
    if len(errors) > 0 {
        # 返回一个新的错误，包含"failed to close connection"的消息，并将errors列表连接起来作为基础错误信息
        return newError("failed to close connection").Base(newError(serial.Concat(errors...)))
    }
    # 如果没有错误，则返回空
    return nil
# 返回连接的本地地址
func (c *connection) LocalAddr() net.Addr {
    return c.conn.LocalAddr()
}

# 返回连接的远程地址
func (c *connection) RemoteAddr() net.Addr {
    return c.remoteAddr
}

# 设置连接的读写截止时间
func (c *connection) SetDeadline(t time.Time) error {
    # 设置读取截止时间，如果出错则返回错误
    if err := c.SetReadDeadline(t); err != nil {
        return err
    }
    # 设置写入截止时间
    return c.SetWriteDeadline(t)
}

# 设置连接的读取截止时间
func (c *connection) SetReadDeadline(t time.Time) error {
    return c.conn.SetReadDeadline(t)
}

# 设置连接的写入截止时间
func (c *connection) SetWriteDeadline(t time.Time) error {
    return c.conn.SetWriteDeadline(t)
}
```