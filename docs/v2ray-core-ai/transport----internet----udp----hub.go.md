# `v2ray-core\transport\internet\udp\hub.go`

```
package udp

import (
    "context"

    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/net"
    "v2ray.com/core/common/protocol/udp"
    "v2ray.com/core/transport/internet"
)

type HubOption func(h *Hub)

func HubCapacity(capacity int) HubOption {
    return func(h *Hub) {
        h.capacity = capacity
    }
}

func HubReceiveOriginalDestination(r bool) HubOption {
    return func(h *Hub) {
        h.recvOrigDest = r
    }
}

type Hub struct {
    conn         *net.UDPConn  // UDP连接对象
    cache        chan *udp.Packet  // 缓存通道，用于存储UDP数据包
    capacity     int  // 缓存通道的容量
    recvOrigDest bool  // 是否接收原始目标地址
}

func ListenUDP(ctx context.Context, address net.Address, port net.Port, streamSettings *internet.MemoryStreamConfig, options ...HubOption) (*Hub, error) {
    hub := &Hub{
        capacity:     256,  // 默认缓存通道容量为256
        recvOrigDest: false,  // 默认不接收原始目标地址
    }
    for _, opt := range options {
        opt(hub)  // 应用选项函数
    }

    var sockopt *internet.SocketConfig
    if streamSettings != nil {
        sockopt = streamSettings.SocketSettings
    }
    if sockopt != nil && sockopt.ReceiveOriginalDestAddress {
        hub.recvOrigDest = true  // 如果配置中设置了接收原始目标地址，则设置为true
    }

    udpConn, err := internet.ListenSystemPacket(ctx, &net.UDPAddr{
        IP:   address.IP(),
        Port: int(port),
    }, sockopt)  // 监听UDP连接
    if err != nil {
        return nil, err
    }
    newError("listening UDP on ", address, ":", port).WriteToLog()  // 记录监听UDP的地址和端口
    hub.conn = udpConn.(*net.UDPConn)  // 将UDP连接对象赋值给hub对象
    hub.cache = make(chan *udp.Packet, hub.capacity)  // 初始化缓存通道

    go hub.start()  // 启动处理UDP数据包的goroutine
    return hub, nil
}

// Close implements net.Listener.
func (h *Hub) Close() error {
    h.conn.Close()  // 关闭UDP连接
    return nil
}

func (h *Hub) WriteTo(payload []byte, dest net.Destination) (int, error) {
    return h.conn.WriteToUDP(payload, &net.UDPAddr{
        IP:   dest.Address.IP(),
        Port: int(dest.Port),
    })  // 向目标地址写入UDP数据包
}

func (h *Hub) start() {
    c := h.cache  // 获取缓存通道
    defer close(c)  // 在函数退出时关闭缓存通道

    oobBytes := make([]byte, 256)  // 创建256字节的oob数据
}
    # 无限循环，接收 UDP 消息并处理
    for {
        # 创建一个新的缓冲区
        buffer := buf.New()
        # 定义变量 noob 和 addr
        var noob int
        var addr *net.UDPAddr
        # 分配足够大小的原始字节切片
        rawBytes := buffer.Extend(buf.Size)

        # 从 UDP 连接中读取消息和控制信息
        n, noob, _, addr, err := ReadUDPMsg(h.conn, rawBytes, oobBytes)
        # 如果出现错误，记录错误信息并退出循环
        if err != nil {
            newError("failed to read UDP msg").Base(err).WriteToLog()
            buffer.Release()
            break
        }
        # 调整缓冲区大小以匹配读取的字节数
        buffer.Resize(0, int32(n))

        # 如果缓冲区为空，释放缓冲区并继续下一次循环
        if buffer.IsEmpty() {
            buffer.Release()
            continue
        }

        # 创建 UDP 数据包对象
        payload := &udp.Packet{
            Payload: buffer,
            Source:  net.UDPDestination(net.IPAddress(addr.IP), net.Port(addr.Port)),
        }
        # 如果需要接收原始目的地信息并且控制信息长度大于 0
        if h.recvOrigDest && noob > 0 {
            # 从控制信息中获取原始目的地信息
            payload.Target = RetrieveOriginalDest(oobBytes[:noob])
            # 如果原始目的地信息有效，记录信息到调试日志中
            if payload.Target.IsValid() {
                newError("UDP original destination: ", payload.Target).AtDebug().WriteToLog()
            } else {
                # 如果无法读取原始目的地信息，记录错误信息到日志中
                newError("failed to read UDP original destination").WriteToLog()
            }
        }

        # 尝试将数据包发送到通道 c 中
        select {
        case c <- payload:
        default:
            # 如果发送失败，释放缓冲区并清空数据包的载荷
            buffer.Release()
            payload.Payload = nil
        }

    }
// Addr 实现了 net.Listener 接口，返回当前 Hub 的本地地址
func (h *Hub) Addr() net.Addr {
    // 返回当前 Hub 的连接的本地地址
    return h.conn.LocalAddr()
}

// Receive 返回一个只读的通道，用于接收 UDP 数据包
func (h *Hub) Receive() <-chan *udp.Packet {
    // 返回 Hub 的缓存通道，用于接收 UDP 数据包
    return h.cache
}
```