# `v2ray-core\transport\internet\udp\hub_freebsd.go`

```go
// +build freebsd

// 声明当前文件属于 freebsd 构建标签

package udp

import (
    "bytes"  // 导入 bytes 包，用于操作字节切片
    "encoding/gob"  // 导入 gob 包，用于编码和解码数据
    "io"  // 导入 io 包，提供了基本的 I/O 接口

    "v2ray.com/core/common/net"  // 导入 net 包，用于网络相关操作
    "v2ray.com/core/transport/internet"  // 导入 internet 包，用于网络传输相关操作
)

// RetrieveOriginalDest 从存储的本地地址和远程地址中获取原始目的地
func RetrieveOriginalDest(oob []byte) net.Destination {
    dec := gob.NewDecoder(bytes.NewBuffer(oob))  // 创建一个解码器，用于解码 oob 中的数据
    var la, ra net.UDPAddr  // 声明本地地址和远程地址变量
    dec.Decode(&la)  // 解码并存储到本地地址变量
    dec.Decode(&ra)  // 解码并存储到远程地址变量
    ip, port, err := internet.OriginalDst(&la, &ra)  // 获取原始目的地的 IP 地址和端口
    if err != nil {  // 如果出现错误
        return net.Destination{}  // 返回空的目的地对象
    }
    return net.UDPDestination(net.IPAddress(ip), net.Port(port))  // 返回 UDP 目的地对象
}

// ReadUDPMsg 存储本地地址和远程地址以供后续使用
func ReadUDPMsg(conn *net.UDPConn, payload []byte, oob []byte) (int, int, int, *net.UDPAddr, error) {
    nBytes, addr, err := conn.ReadFromUDP(payload)  // 从 UDP 连接中读取数据和远程地址
    var buf bytes.Buffer  // 创建一个字节缓冲区
    enc := gob.NewEncoder(&buf)  // 创建一个编码器，用于编码数据到缓冲区
    enc.Encode(conn.LocalAddr().(*net.UDPAddr))  // 编码本地地址到缓冲区
    enc.Encode(addr)  // 编码远程地址到缓冲区
    var reader io.Reader = &buf  // 创建一个读取器，用于从缓冲区读取数据
    noob, _ := reader.Read(oob)  // 从缓冲区读取数据到 oob 中
    return nBytes, noob, 0, addr, err  // 返回读取的字节数、oob 的长度、0、远程地址和可能的错误
}
```