# `kubo\p2p\remote.go`

```go
package p2p

import (
    "context"
    "fmt"

    net "github.com/libp2p/go-libp2p/core/network"
    "github.com/libp2p/go-libp2p/core/protocol"
    ma "github.com/multiformats/go-multiaddr"
    manet "github.com/multiformats/go-multiaddr/net"
)

var maPrefix = "/" + ma.ProtocolWithCode(ma.P_IPFS).Name + "/"

// remoteListener accepts libp2p streams and proxies them to a manet host.
type remoteListener struct {
    p2p *P2P  // 保存对 P2P 对象的引用

    // Application proto identifier.
    proto protocol.ID  // 保存协议标识符

    // Address to proxy the incoming connections to
    addr ma.Multiaddr  // 保存要代理传入连接的地址

    // reportRemote if set to true makes the handler send '<base58 remote peerid>\n'
    // to target before any data is forwarded
    reportRemote bool  // 如果设置为 true，则在转发任何数据之前，处理程序会向目标发送 '<base58 remote peerid>\n'

}

// ForwardRemote creates new p2p listener.
func (p2p *P2P) ForwardRemote(ctx context.Context, proto protocol.ID, addr ma.Multiaddr, reportRemote bool) (Listener, error) {
    listener := &remoteListener{
        p2p: p2p,  // 初始化 remoteListener 结构体

        proto: proto,  // 设置协议标识符
        addr:  addr,  // 设置地址

        reportRemote: reportRemote,  // 设置 reportRemote 标志
    }

    if err := p2p.ListenersP2P.Register(listener); err != nil {  // 将 listener 注册到 ListenersP2P 中
        return nil, err
    }

    return listener, nil  // 返回 listener 对象
}

func (l *remoteListener) handleStream(remote net.Stream) {
    local, err := manet.Dial(l.addr)  // 使用 l.addr 连接到本地地址
    if err != nil {
        _ = remote.Reset()  // 如果出错，重置远程流
        return
    }

    peer := remote.Conn().RemotePeer()  // 获取远程对等点的标识符

    if l.reportRemote {  // 如果 reportRemote 为 true
        if _, err := fmt.Fprintf(local, "%s\n", peer); err != nil {  // 向本地发送远程对等点的标识符
            _ = remote.Reset()  // 如果出错，重置远程流
            return
        }
    }

    peerMa, err := ma.NewMultiaddr(maPrefix + peer.String())  // 创建包含远程对等点标识符的多地址
    if err != nil {
        _ = remote.Reset()  // 如果出错，重置远程流
        return
    }

    stream := &Stream{  // 创建新的流对象
        Protocol: l.proto,  // 设置协议标识符
        OriginAddr: peerMa,  // 设置源地址
        TargetAddr: l.addr,  // 设置目标地址
        peer:       peer,  // 设置对等点标识符
        Local:  local,  // 设置本地流
        Remote: remote,  // 设置远程流
        Registry: l.p2p.Streams,  // 设置流注册表
    }

    l.p2p.Streams.Register(stream)  // 将流注册到 Streams 中
}

func (l *remoteListener) Protocol() protocol.ID {
    return l.proto  // 返回协议标识符
}
# 返回远程监听器的地址
func (l *remoteListener) ListenAddress() ma.Multiaddr {
    # 创建包含P2P标识的多地址
    addr, err := ma.NewMultiaddr(maPrefix + l.p2p.identity.String())
    # 如果创建多地址时出现错误，抛出异常
    if err != nil {
        panic(err)
    }
    # 返回创建的多地址
    return addr
}

# 返回远程监听器的目标地址
func (l *remoteListener) TargetAddress() ma.Multiaddr {
    # 返回远程监听器的地址
    return l.addr
}

# 关闭远程监听器
func (l *remoteListener) close() {}

# 返回远程监听器的协议ID
func (l *remoteListener) key() protocol.ID {
    # 返回远程监听器的协议ID
    return l.proto
}
```