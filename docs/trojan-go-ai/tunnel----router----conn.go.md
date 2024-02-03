# `trojan-go\tunnel\router\conn.go`

```go
package router

import (
    "context"  // 导入上下文包
    "io"  // 导入输入输出包
    "net"  // 导入网络包

    "github.com/p4gefau1t/trojan-go/common"  // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log"  // 导入日志包
    "github.com/p4gefau1t/trojan-go/tunnel"  // 导入隧道包
)

type packetInfo struct {
    src     *tunnel.Metadata  // 数据包来源信息
    payload []byte  // 数据包内容
}

type PacketConn struct {
    proxy tunnel.PacketConn  // 代理隧道连接
    net.PacketConn  // 网络包连接
    packetChan chan *packetInfo  // 数据包通道
    *Client  // 客户端信息
    ctx    context.Context  // 上下文
    cancel context.CancelFunc  // 取消函数
}

func (c *PacketConn) packetLoop() {
    go func() {  // 启动协程
        for {
            buf := make([]byte, MaxPacketSize)  // 创建指定大小的字节缓冲区
            n, addr, err := c.proxy.ReadWithMetadata(buf)  // 从代理隧道读取数据和元数据
            if err != nil {  // 如果发生错误
                select {
                case <-c.ctx.Done():  // 如果上下文已经结束
                    return  // 退出循环
                default:
                    log.Error("router packetConn error", err)  // 记录错误日志
                    continue  // 继续下一次循环
                }
            }
            c.packetChan <- &packetInfo{  // 将数据包信息发送到通道
                src:     addr,  // 设置数据包来源信息
                payload: buf[:n],  // 设置数据包内容
            }
        }
    }()
    for {
        buf := make([]byte, MaxPacketSize)  // 创建指定大小的字节缓冲区
        n, addr, err := c.PacketConn.ReadFrom(buf)  // 从网络包连接读取数据和地址
        if err != nil {  // 如果发生错误
            select {
            case <-c.ctx.Done():  // 如果上下文已经结束
                return  // 退出循环
            default:
                log.Error("router packetConn error", err)  // 记录错误日志
                continue  // 继续下一次循环
            }
        }
        address, _ := tunnel.NewAddressFromAddr("udp", addr.String())  // 创建地址信息
        c.packetChan <- &packetInfo{  // 将数据包信息发送到通道
            src: &tunnel.Metadata{  // 设置数据包来源信息
                Address: address,  // 设置地址信息
            },
            payload: buf[:n],  // 设置数据包内容
        }
    }
}

func (c *PacketConn) Close() error {
    c.cancel()  // 取消函数
    c.proxy.Close()  // 关闭代理隧道连接
    return c.PacketConn.Close()  // 返回网络包连接的关闭结果
}

func (c *PacketConn) ReadFrom(p []byte) (n int, addr net.Addr, err error) {
    panic("implement me")  // 未实现的方法
}

func (c *PacketConn) WriteTo(p []byte, addr net.Addr) (n int, err error) {
    panic("implement me")  // 未实现的方法
}

func (c *PacketConn) WriteWithMetadata(p []byte, m *tunnel.Metadata) (int, error) {
    # 根据地址选择路由策略
    policy := c.Route(m.Address)
    # 根据不同的策略进行处理
    switch policy {
    # 如果是代理策略，则使用代理写入数据并返回结果
    case Proxy:
        return c.proxy.WriteWithMetadata(p, m)
    # 如果是阻止策略，则返回错误信息
    case Block:
        return 0, common.NewError("router blocked address (udp): " + m.Address.String())
    # 如果是绕过策略，则解析地址并写入数据
    case Bypass:
        ip, err := m.Address.ResolveIP()
        # 如果解析地址出错，则返回错误信息
        if err != nil {
            return 0, common.NewError("router failed to resolve udp address").Base(err)
        }
        # 否则，使用解析后的 IP 地址和端口写入数据
        return c.PacketConn.WriteTo(p, &net.UDPAddr{
            IP:   ip,
            Port: m.Address.Port,
        })
    # 如果是未知策略，则抛出异常
    default:
        panic("unknown policy")
    }
# 从PacketConn中读取数据和元数据
func (c *PacketConn) ReadWithMetadata(p []byte) (int, *tunnel.Metadata, error) {
    # 通过select语句监听多个channel，一旦其中一个channel准备好，就执行对应的case
    select {
        # 从packetChan中读取数据
        case info := <-c.packetChan:
            # 将info.payload中的数据复制到p中
            n := copy(p, info.payload)
            # 返回复制的字节数，数据来源的地址，和nil错误
            return n, info.src, nil
        # 如果c.ctx.Done() channel关闭，表示上下文已经结束
        case <-c.ctx.Done():
            # 返回0，nil，和io.EOF错误，表示读取结束
            return 0, nil, io.EOF
    }
}
```