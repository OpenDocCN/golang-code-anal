# `v2ray-core\common\protocol\server_picker.go`

```
package protocol

import (
    "sync"
)

type ServerList struct {
    sync.RWMutex  // 读写锁，用于保护 servers 列表的并发读写
    servers []*ServerSpec  // 服务器列表
}

func NewServerList() *ServerList {
    return &ServerList{}  // 创建并返回一个新的服务器列表对象
}

func (sl *ServerList) AddServer(server *ServerSpec) {
    sl.Lock()  // 获取写锁
    defer sl.Unlock()  // 在函数返回时释放写锁

    sl.servers = append(sl.servers, server)  // 将新的服务器添加到服务器列表中
}

func (sl *ServerList) Size() uint32 {
    sl.RLock()  // 获取读锁
    defer sl.RUnlock()  // 在函数返回时释放读锁

    return uint32(len(sl.servers))  // 返回服务器列表的长度
}

func (sl *ServerList) GetServer(idx uint32) *ServerSpec {
    sl.Lock()  // 获取写锁
    defer sl.Unlock()  // 在函数返回时释放写锁

    for {
        if idx >= uint32(len(sl.servers)) {  // 如果索引超出服务器列表长度，则返回空
            return nil
        }

        server := sl.servers[idx]  // 获取指定索引处的服务器
        if !server.IsValid() {  // 如果服务器无效
            sl.removeServer(idx)  // 从服务器列表中移除该服务器
            continue  // 继续循环查找下一个有效的服务器
        }

        return server  // 返回有效的服务器
    }
}

func (sl *ServerList) removeServer(idx uint32) {
    n := len(sl.servers)  // 获取服务器列表的长度
    sl.servers[idx] = sl.servers[n-1]  // 将指定索引处的服务器替换为最后一个服务器
    sl.servers = sl.servers[:n-1]  // 移除最后一个服务器，实现删除指定索引处的服务器
}

type ServerPicker interface {
    PickServer() *ServerSpec
}

type RoundRobinServerPicker struct {
    sync.Mutex  // 互斥锁，用于保护 serverlist 和 nextIndex 的并发访问
    serverlist *ServerList  // 服务器列表
    nextIndex  uint32  // 下一个要选择的服务器索引
}

func NewRoundRobinServerPicker(serverlist *ServerList) *RoundRobinServerPicker {
    return &RoundRobinServerPicker{  // 创建并返回一个新的轮询服务器选择器对象
        serverlist: serverlist,  // 设置服务器列表
        nextIndex:  0,  // 设置下一个要选择的服务器索引为 0
    }
}

func (p *RoundRobinServerPicker) PickServer() *ServerSpec {
    p.Lock()  // 获取互斥锁
    defer p.Unlock()  // 在函数返回时释放互斥锁

    next := p.nextIndex  // 获取下一个要选择的服务器索引
    server := p.serverlist.GetServer(next)  // 根据索引选择服务器
    if server == nil {  // 如果选择的服务器为空
        next = 0  // 重置索引为 0
        server = p.serverlist.GetServer(0)  // 重新选择服务器
    }
    next++  // 更新下一个要选择的服务器索引
    if next >= p.serverlist.Size() {  // 如果索引超出服务器列表长度
        next = 0  // 重置索引为 0
    }
    p.nextIndex = next  // 更新下一个要选择的服务器索引

    return server  // 返回选择的服务器
}
```