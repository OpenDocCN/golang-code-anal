# `kubo\p2p\local.go`

```go
package p2p

import (
    "context"
    "time"

    tec "github.com/jbenet/go-temp-err-catcher"
    net "github.com/libp2p/go-libp2p/core/network"
    "github.com/libp2p/go-libp2p/core/peer"
    "github.com/libp2p/go-libp2p/core/protocol"
    ma "github.com/multiformats/go-multiaddr"
    manet "github.com/multiformats/go-multiaddr/net"
)

// localListener manet streams and proxies them to libp2p services.
type localListener struct {
    ctx context.Context  // 上下文对象
    p2p *P2P  // P2P 对象指针

    proto protocol.ID  // 协议 ID
    laddr ma.Multiaddr  // 多地址对象
    peer  peer.ID  // 对等节点 ID

    listener manet.Listener  // manet 监听器对象
}

// ForwardLocal creates new P2P stream to a remote listener.
func (p2p *P2P) ForwardLocal(ctx context.Context, peer peer.ID, proto protocol.ID, bindAddr ma.Multiaddr) (Listener, error) {
    listener := &localListener{  // 创建 localListener 对象
        ctx:   ctx,  // 设置上下文对象
        p2p:   p2p,  // 设置 P2P 对象指针
        proto: proto,  // 设置协议 ID
        peer:  peer,  // 设置对等节点 ID
    }

    maListener, err := manet.Listen(bindAddr)  // 监听指定地址
    if err != nil {
        return nil, err  // 如果出错，返回错误
    }

    listener.listener = maListener  // 设置监听器对象
    listener.laddr = maListener.Multiaddr()  // 设置多地址对象

    if err := p2p.ListenersLocal.Register(listener); err != nil {  // 注册监听器对象
        return nil, err  // 如果出错，返回错误
    }

    go listener.acceptConns()  // 异步接受连接

    return listener, nil  // 返回监听器对象和空错误
}

func (l *localListener) dial(ctx context.Context) (net.Stream, error) {
    cctx, cancel := context.WithTimeout(ctx, time.Second*30)  // 创建带超时的上下文对象
    defer cancel()  // 延迟取消上下文

    return l.p2p.peerHost.NewStream(cctx, l.peer, l.proto)  // 创建新的流对象
}

func (l *localListener) acceptConns() {
    for {
        local, err := l.listener.Accept()  // 接受连接
        if err != nil {
            if tec.ErrIsTemporary(err) {  // 如果是临时错误
                continue  // 继续循环
            }
            return  // 返回
        }

        go l.setupStream(local)  // 异步设置流对象
    }
}

func (l *localListener) setupStream(local manet.Conn) {
    remote, err := l.dial(l.ctx)  // 拨号连接
    if err != nil {
        local.Close()  // 关闭本地连接
        log.Warnf("failed to dial to remote %s/%s", l.peer, l.proto)  // 记录警告日志
        return  // 返回
    }
}
    // 创建一个新的流对象，并初始化其属性
    stream := &Stream{
        Protocol: l.proto, // 设置流对象的协议属性为 l.proto

        OriginAddr: local.RemoteMultiaddr(), // 设置流对象的 OriginAddr 属性为本地的远程多地址
        TargetAddr: l.TargetAddress(), // 设置流对象的 TargetAddr 属性为 l 的目标地址
        peer:       l.peer, // 设置流对象的 peer 属性为 l 的对等节点

        Local:  local, // 设置流对象的 Local 属性为本地
        Remote: remote, // 设置流对象的 Remote 属性为远程

        Registry: l.p2p.Streams, // 设置流对象的 Registry 属性为 l.p2p.Streams
    }

    // 将新创建的流对象注册到流对象的注册表中
    l.p2p.Streams.Register(stream)
# 关闭本地监听器
func (l *localListener) close() {
    l.listener.Close()
}

# 返回本地监听器的协议ID
func (l *localListener) Protocol() protocol.ID {
    return l.proto
}

# 返回本地监听器的监听地址
func (l *localListener) ListenAddress() ma.Multiaddr {
    return l.laddr
}

# 返回本地监听器的目标地址
func (l *localListener) TargetAddress() {
    # 通过拼接前缀和对等节点的字符串形式创建多地址
    addr, err := ma.NewMultiaddr(maPrefix + l.peer.String())
    if err != nil {
        panic(err)
    }
    return addr
}

# 返回本地监听器的键值
func (l *localListener) key() protocol.ID {
    return protocol.ID(l.ListenAddress().String())
}
```