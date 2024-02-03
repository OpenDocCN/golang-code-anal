# `kubo\client\rpc\swarm.go`

```go
package rpc

import (
    "context"
    "time"

    iface "github.com/ipfs/kubo/core/coreiface"
    "github.com/libp2p/go-libp2p/core/network"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/libp2p/go-libp2p/core/protocol"
    "github.com/multiformats/go-multiaddr"
)

type SwarmAPI HttpApi

func (api *SwarmAPI) Connect(ctx context.Context, pi peer.AddrInfo) error {
    // 创建包含 p2p 协议和 peer ID 的多地址
    pidma, err := multiaddr.NewComponent("p2p", pi.ID.String())
    if err != nil {
        return err
    }

    // 将 peer 地址列表中的地址封装成包含 p2p 协议和 peer ID 的多地址字符串
    saddrs := make([]string, len(pi.Addrs))
    for i, addr := range pi.Addrs {
        saddrs[i] = addr.Encapsulate(pidma).String()
    }

    // 发起连接请求
    return api.core().Request("swarm/connect", saddrs...).Exec(ctx, nil)
}

func (api *SwarmAPI) Disconnect(ctx context.Context, addr multiaddr.Multiaddr) error {
    // 发起断开连接请求
    return api.core().Request("swarm/disconnect", addr.String()).Exec(ctx, nil)
}

type connInfo struct {
    addr      multiaddr.Multiaddr
    peer      peer.ID
    latency   time.Duration
    muxer     string
    direction network.Direction
    streams   []protocol.ID
}

func (c *connInfo) ID() peer.ID {
    return c.peer
}

func (c *connInfo) Address() multiaddr.Multiaddr {
    return c.addr
}

func (c *connInfo) Direction() network.Direction {
    return c.direction
}

func (c *connInfo) Latency() (time.Duration, error) {
    return c.latency, nil
}

func (c *connInfo) Streams() ([]protocol.ID, error) {
    return c.streams, nil
}

func (api *SwarmAPI) Peers(ctx context.Context) ([]iface.ConnectionInfo, error) {
    var resp struct {
        Peers []struct {
            Addr      string
            Peer      string
            Latency   string
            Muxer     string
            Direction network.Direction
            Streams   []struct {
                Protocol string
            }
        }
    }

    // 发起获取对等节点信息的请求
    err := api.core().Request("swarm/peers").
        Option("streams", true).
        Option("latency", true).
        Exec(ctx, &resp)
    if err != nil {
        return nil, err
    # 根据响应中的对等节点数量创建一个连接信息的切片
    res := make([]iface.ConnectionInfo, len(resp.Peers))
    # 遍历响应中的对等节点信息
    for i, conn := range resp.Peers {
        # 将连接延迟转换为时间间隔
        latency, _ := time.ParseDuration(conn.Latency)
        # 创建连接信息对象
        out := &connInfo{
            latency:   latency,
            muxer:     conn.Muxer,
            direction: conn.Direction,
        }

        # 解码对等节点的标识符
        out.peer, err = peer.Decode(conn.Peer)
        if err != nil {
            return nil, err
        }

        # 创建对等节点的地址
        out.addr, err = multiaddr.NewMultiaddr(conn.Addr)
        if err != nil {
            return nil, err
        }

        # 创建连接信息对象中的协议ID切片
        out.streams = make([]protocol.ID, len(conn.Streams))
        for i, p := range conn.Streams {
            out.streams[i] = protocol.ID(p.Protocol)
        }

        # 将连接信息对象添加到结果切片中
        res[i] = out
    }

    # 返回结果切片和空错误
    return res, nil
// 返回已知对等节点的地址信息，以映射形式返回对等节点ID到多地址的映射
func (api *SwarmAPI) KnownAddrs(ctx context.Context) (map[peer.ID][]multiaddr.Multiaddr, error) {
    var out struct {
        Addrs map[string][]string
    }
    // 发送请求获取已知对等节点的地址信息
    if err := api.core().Request("swarm/addrs").Exec(ctx, &out); err != nil {
        return nil, err
    }
    res := map[peer.ID][]multiaddr.Multiaddr{}
    for spid, saddrs := range out.Addrs {
        addrs := make([]multiaddr.Multiaddr, len(saddrs))

        for i, addr := range saddrs {
            a, err := multiaddr.NewMultiaddr(addr)
            if err != nil {
                return nil, err
            }
            addrs[i] = a
        }

        pid, err := peer.Decode(spid)
        if err != nil {
            return nil, err
        }

        res[pid] = addrs
    }

    return res, nil
}

// 返回本地对等节点的地址信息
func (api *SwarmAPI) LocalAddrs(ctx context.Context) ([]multiaddr.Multiaddr, error) {
    var out struct {
        Strings []string
    }

    // 发送请求获取本地对等节点的地址信息
    if err := api.core().Request("swarm/addrs/local").Exec(ctx, &out); err != nil {
        return nil, err
    }

    res := make([]multiaddr.Multiaddr, len(out.Strings))
    for i, addr := range out.Strings {
        ma, err := multiaddr.NewMultiaddr(addr)
        if err != nil {
            return nil, err
        }
        res[i] = ma
    }
    return res, nil
}

// 返回监听地址信息
func (api *SwarmAPI) ListenAddrs(ctx context.Context) ([]multiaddr.Multiaddr, error) {
    var out struct {
        Strings []string
    }

    // 发送请求获取监听地址信息
    if err := api.core().Request("swarm/addrs/listen").Exec(ctx, &out); err != nil {
        return nil, err
    }

    res := make([]multiaddr.Multiaddr, len(out.Strings))
    for i, addr := range out.Strings {
        ma, err := multiaddr.NewMultiaddr(addr)
        if err != nil {
            return nil, err
        }
        res[i] = ma
    }
    return res, nil
}

// 返回核心 HTTP API 对象
func (api *SwarmAPI) core() *HttpApi {
    return (*HttpApi)(api)
}
```