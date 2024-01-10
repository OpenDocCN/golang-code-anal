# `v2ray-core\transport\internet\quic\conn.go`

```
// +build !confonly

package quic

import (
    "crypto/cipher" // 导入加密解密包
    "crypto/rand" // 导入随机数生成包
    "errors" // 导入错误处理包
    "time" // 导入时间包

    "github.com/lucas-clemente/quic-go" // 导入 QUIC 协议包
    "v2ray.com/core/common" // 导入通用工具包
    "v2ray.com/core/common/buf" // 导入缓冲区包
    "v2ray.com/core/common/net" // 导入网络通信包
    "v2ray.com/core/transport/internet" // 导入网络传输包
)

type sysConn struct {
    conn   net.PacketConn // 网络数据包连接
    header internet.PacketHeader // 网络数据包头部
    auth   cipher.AEAD // 加密认证
}

func wrapSysConn(rawConn net.PacketConn, config *Config) (*sysConn, error) {
    header, err := getHeader(config) // 获取配置中的数据包头部
    if err != nil {
        return nil, err
    }
    auth, err := getAuth(config) // 获取配置中的加密认证
    if err != nil {
        return nil, err
    }
    return &sysConn{
        conn:   rawConn, // 初始化网络数据包连接
        header: header, // 初始化网络数据包头部
        auth:   auth, // 初始化加密认证
    }, nil
}

var errInvalidPacket = errors.New("invalid packet") // 定义错误类型

func (c *sysConn) readFromInternal(p []byte) (int, net.Addr, error) {
    buffer := getBuffer() // 获取缓冲区
    defer putBuffer(buffer) // 延迟释放缓冲区

    nBytes, addr, err := c.conn.ReadFrom(buffer) // 从网络数据包连接中读取数据
    if err != nil {
        return 0, nil, err
    }

    payload := buffer[:nBytes] // 获取有效载荷
    if c.header != nil { // 如果存在数据包头部
        if len(payload) <= int(c.header.Size()) { // 如果有效载荷长度小于等于数据包头部长度
            return 0, nil, errInvalidPacket // 返回错误
        }
        payload = payload[c.header.Size():] // 截取有效载荷
    }

    if c.auth == nil { // 如果不存在加密认证
        n := copy(p, payload) // 复制有效载荷到目标数组
        return n, addr, nil
    }

    if len(payload) <= c.auth.NonceSize() { // 如果有效载荷长度小于等于加密认证的 Nonce 大小
        return 0, nil, errInvalidPacket // 返回错误
    }

    nonce := payload[:c.auth.NonceSize()] // 获取 Nonce
    payload = payload[c.auth.NonceSize():] // 截取有效载荷

    p, err = c.auth.Open(p[:0], nonce, payload, nil) // 解密有效载荷
    if err != nil {
        return 0, nil, errInvalidPacket // 返回错误
    }

    return len(p), addr, nil
}

func (c *sysConn) ReadFrom(p []byte) (int, net.Addr, error) {
    if c.header == nil && c.auth == nil { // 如果不存在数据包头部和加密认证
        return c.conn.ReadFrom(p) // 直接从网络数据包连接中读取数据
    }
    # 无限循环，不断执行以下操作
    n, addr, err := c.readFromInternal(p)  # 调用 c 对象的 readFromInternal 方法，获取返回的 n, addr, err
    # 如果出现错误并且错误不是 errInvalidPacket，则返回错误
    if err != nil && err != errInvalidPacket {
        return 0, nil, err
    }
    # 如果没有错误，则返回 n, addr, nil
    if err == nil {
        return n, addr, nil
    }
func (c *sysConn) WriteTo(p []byte, addr net.Addr) (int, error) {
    // 如果连接的头部和认证信息都为空，则直接调用底层连接的WriteTo方法
    if c.header == nil && c.auth == nil {
        return c.conn.WriteTo(p, addr)
    }

    // 获取一个缓冲区
    buffer := getBuffer()
    // 在函数返回时释放缓冲区
    defer putBuffer(buffer)

    // 将payload指向buffer
    payload := buffer
    n := 0
    // 如果存在头部信息，则将头部信息序列化到payload中，并更新n的值
    if c.header != nil {
        c.header.Serialize(payload)
        n = int(c.header.Size())
    }

    // 如果认证信息为空，则将p中的数据拷贝到payload中，并更新n的值
    if c.auth == nil {
        nBytes := copy(payload[n:], p)
        n += nBytes
    } else {
        // 如果存在认证信息，则生成一个随机的nounce，并将nounce写入payload中，更新n的值
        nounce := payload[n : n+c.auth.NonceSize()]
        common.Must2(rand.Read(nounce))
        n += c.auth.NonceSize()
        // 使用认证信息对payload进行加密，并更新n的值
        pp := c.auth.Seal(payload[:n], nounce, p, nil)
        n = len(pp)
    }

    // 调用底层连接的WriteTo方法，将payload中的数据写入连接
    return c.conn.WriteTo(payload[:n], addr)
}

func (c *sysConn) Close() error {
    // 调用底层连接的Close方法
    return c.conn.Close()
}

func (c *sysConn) LocalAddr() net.Addr {
    // 返回底层连接的本地地址
    return c.conn.LocalAddr()
}

func (c *sysConn) SetDeadline(t time.Time) error {
    // 设置底层连接的deadline
    return c.conn.SetDeadline(t)
}

func (c *sysConn) SetReadDeadline(t time.Time) error {
    // 设置底层连接的读取deadline
    return c.conn.SetReadDeadline(t)
}

func (c *sysConn) SetWriteDeadline(t time.Time) error {
    // 设置底层连接的写入deadline
    return c.conn.SetWriteDeadline(t)
}

type interConn struct {
    stream quic.Stream
    local  net.Addr
    remote net.Addr
}

func (c *interConn) Read(b []byte) (int, error) {
    // 从流中读取数据到b中
    return c.stream.Read(b)
}

func (c *interConn) WriteMultiBuffer(mb buf.MultiBuffer) error {
    // 对多个缓冲区进行合并
    mb = buf.Compact(mb)
    // 将合并后的缓冲区写入连接中，并返回可能出现的错误
    mb, err := buf.WriteMultiBuffer(c, mb)
    // 释放合并后的缓冲区
    buf.ReleaseMulti(mb)
    return err
}

func (c *interConn) Write(b []byte) (int, error) {
    // 将b中的数据写入流中
    return c.stream.Write(b)
}

func (c *interConn) Close() error {
    // 关闭流
    return c.stream.Close()
}

func (c *interConn) LocalAddr() net.Addr {
    // 返回连接的本地地址
    return c.local
}

func (c *interConn) RemoteAddr() net.Addr {
    // 返回连接的远程地址
    return c.remote
}

func (c *interConn) SetDeadline(t time.Time) error {
    // 设置连接的deadline
    return c.stream.SetDeadline(t)
}

func (c *interConn) SetReadDeadline(t time.Time) error {
    // 设置连接的读取deadline
    return c.stream.SetReadDeadline(t)
}

func (c *interConn) SetWriteDeadline(t time.Time) error {
    // 设置连接的写入deadline
    return c.stream.SetWriteDeadline(t)
}
    # 设置连接的写入截止时间
    return c.stream.SetWriteDeadline(t)
# 闭合前面的函数定义
```