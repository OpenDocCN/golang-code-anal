# `kubo\p2p\stream.go`

```go
package p2p

import (
    "io"
    "sync"

    ifconnmgr "github.com/libp2p/go-libp2p/core/connmgr"
    net "github.com/libp2p/go-libp2p/core/network"
    peer "github.com/libp2p/go-libp2p/core/peer"
    protocol "github.com/libp2p/go-libp2p/core/protocol"
    ma "github.com/multiformats/go-multiaddr"
    manet "github.com/multiformats/go-multiaddr/net"
)

const cmgrTag = "stream-fwd"

// Stream holds information on active incoming and outgoing p2p streams.
type Stream struct {
    id uint64  // 用于标识流的唯一ID

    Protocol protocol.ID  // 流的协议ID

    OriginAddr ma.Multiaddr  // 流的原始地址
    TargetAddr ma.Multiaddr  // 流的目标地址
    peer       peer.ID  // 流的对端节点ID

    Local  manet.Conn  // 本地连接
    Remote net.Stream  // 远程连接

    Registry *StreamRegistry  // 流所属的注册表
}

// close stream endpoints and deregister it.
func (s *Stream) close() {
    s.Registry.Close(s)  // 关闭流的端点并从注册表中注销
}

// reset closes stream endpoints and deregisters it.
func (s *Stream) reset() {
    s.Registry.Reset(s)  // 重置流的端点并从注册表中注销
}

func (s *Stream) startStreaming() {
    go func() {
        _, err := io.Copy(s.Local, s.Remote)  // 将远程流的数据拷贝到本地流
        if err != nil {
            s.reset()  // 如果出现错误，则重置流
        } else {
            s.close()  // 否则关闭流
        }
    }()

    go func() {
        _, err := io.Copy(s.Remote, s.Local)  // 将本地流的数据拷贝到远程流
        if err != nil {
            s.reset()  // 如果出现错误，则重置流
        } else {
            s.close()  // 否则关闭流
        }
    }()
}

// StreamRegistry is a collection of active incoming and outgoing proto app streams.
type StreamRegistry struct {
    sync.Mutex  // 互斥锁

    Streams map[uint64]*Stream  // 存储流的映射表
    conns   map[peer.ID]int  // 存储节点ID和连接数的映射表
    nextID  uint64  // 下一个流的ID

    ifconnmgr.ConnManager  // 连接管理器
}

// Register registers a stream to the registry.
func (r *StreamRegistry) Register(streamInfo *Stream) {
    r.Lock()  // 加锁
    defer r.Unlock()  // 延迟解锁

    r.ConnManager.TagPeer(streamInfo.peer, cmgrTag, 20)  // 给对端节点打标签
    r.conns[streamInfo.peer]++  // 对端节点的连接数加一

    streamInfo.id = r.nextID  // 设置流的ID
    r.Streams[r.nextID] = streamInfo  // 将流添加到映射表中
    r.nextID++  // 下一个流的ID加一

    streamInfo.startStreaming()  // 开始流的数据传输
}

// Deregister deregisters stream from the registry.
func (r *StreamRegistry) Deregister(streamID uint64) {
    r.Lock()  // 加锁
    defer r.Unlock()  // 延迟解锁
    # 从 r.Streams 中获取指定 streamID 对应的流对象 s，同时返回是否存在的标志 ok
    s, ok := r.Streams[streamID]
    # 如果不存在对应的流对象，则直接返回
    if !ok {
        return
    }
    # 获取流对象 s 的对等节点 peer
    p := s.peer
    # 减少对等节点 p 的连接数
    r.conns[p]--
    # 如果对等节点 p 的连接数小于 1，则删除对应的连接记录，并从连接管理器中取消对等节点的标记
    if r.conns[p] < 1 {
        delete(r.conns, p)
        r.ConnManager.UntagPeer(p, cmgrTag)
    }

    # 删除 r.Streams 中指定的 streamID 对应的流对象
    delete(r.Streams, streamID)
// 关闭流端点并注销它
func (r *StreamRegistry) Close(s *Stream) {
    // 关闭本地端点
    _ = s.Local.Close()
    // 关闭远程端点
    _ = s.Remote.Close()
    // 从注册表中注销流
    s.Registry.Deregister(s.id)
}

// 重置关闭流端点并注销它
func (r *StreamRegistry) Reset(s *Stream) {
    // 关闭本地端点
    _ = s.Local.Close()
    // 重置远程端点
    _ = s.Remote.Reset()
    // 从注册表中注销流
    s.Registry.Deregister(s.id)
}
```