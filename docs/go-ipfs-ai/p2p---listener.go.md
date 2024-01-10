# `kubo\p2p\listener.go`

```
package p2p

import (
    "errors"
    "sync"

    p2phost "github.com/libp2p/go-libp2p/core/host"
    net "github.com/libp2p/go-libp2p/core/network"
    "github.com/libp2p/go-libp2p/core/protocol"
    ma "github.com/multiformats/go-multiaddr"
)

// Listener listens for connections and proxies them to a target.
type Listener interface {
    Protocol() protocol.ID  // 返回协议 ID
    ListenAddress() ma.Multiaddr  // 返回监听地址
    TargetAddress() ma.Multiaddr  // 返回目标地址

    key() protocol.ID  // 返回协议 ID
    // close closes the listener. Does not affect child streams
    close()  // 关闭监听器
}

// Listeners manages a group of Listener implementations,
// checking for conflicts and optionally dispatching connections.
type Listeners struct {
    sync.RWMutex  // 读写锁

    Listeners map[protocol.ID]Listener  // 协议 ID 到 Listener 的映射
}

func newListenersLocal() *Listeners {
    return &Listeners{
        Listeners: map[protocol.ID]Listener{},  // 初始化 Listeners
    }
}

func newListenersP2P(host p2phost.Host) *Listeners {
    reg := &Listeners{
        Listeners: map[protocol.ID]Listener{},  // 初始化 Listeners
    }

    host.SetStreamHandlerMatch("/x/", func(p protocol.ID) bool {
        reg.RLock()  // 读锁
        defer reg.RUnlock()  // 解锁

        _, ok := reg.Listeners[p]  // 检查是否存在对应协议 ID 的 Listener
        return ok
    }, func(stream net.Stream) {
        reg.RLock()  // 读锁
        defer reg.RUnlock()  // 解锁

        l := reg.Listeners[stream.Protocol()]  // 获取对应协议 ID 的 Listener
        if l != nil {
            go l.(*remoteListener).handleStream(stream)  // 处理流
        }
    })

    return reg
}

// Register registers listenerInfo into this registry and starts it.
func (r *Listeners) Register(l Listener) error {
    r.Lock()  // 写锁
    defer r.Unlock()  // 解锁

    if _, ok := r.Listeners[l.key()]; ok {  // 检查是否已经注册了相同的 Listener
        return errors.New("listener already registered")  // 返回错误信息
    }

    r.Listeners[l.key()] = l  // 注册 Listener
    return nil
}

func (r *Listeners) Close(matchFunc func(listener Listener) bool) int {
    todo := make([]Listener, 0)  // 初始化待处理的 Listener 数组
    r.Lock()  // 写锁
    # 遍历 r.Listeners 中的每个监听器
    for _, l := range r.Listeners {
        # 如果监听器不符合匹配函数的条件，则跳过当前循环，继续下一个监听器
        if !matchFunc(l) {
            continue
        }

        # 如果 r.Listeners 中存在当前监听器的键值
        if _, ok := r.Listeners[l.key()]; ok {
            # 删除 r.Listeners 中的当前监听器
            delete(r.Listeners, l.key())
            # 将当前监听器添加到待处理列表中
            todo = append(todo, l)
        }
    }
    # 解锁 r 对象
    r.Unlock()

    # 遍历待处理列表中的每个监听器
    for _, l := range todo {
        # 关闭当前监听器
        l.close()
    }

    # 返回待处理列表的长度
    return len(todo)
# 闭合前面的函数定义
```