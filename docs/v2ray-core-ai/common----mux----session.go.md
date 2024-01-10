# `v2ray-core\common\mux\session.go`

```
package mux

import (
    "sync"

    "v2ray.com/core/common"
    "v2ray.com/core/common/buf"
    "v2ray.com/core/common/protocol"
)

type SessionManager struct {
    sync.RWMutex
    sessions map[uint16]*Session  // 用于存储会话的映射表
    count    uint16                // 会话计数
    closed   bool                  // 标记会话管理器是否已关闭
}

func NewSessionManager() *SessionManager {
    return &SessionManager{
        count:    0,  // 初始化会话计数为0
        sessions: make(map[uint16]*Session, 16),  // 初始化会话映射表
    }
}

func (m *SessionManager) Closed() bool {
    m.RLock()  // 加读锁
    defer m.RUnlock()  // 在函数返回前释放读锁

    return m.closed  // 返回会话管理器的关闭状态
}

func (m *SessionManager) Size() int {
    m.RLock()  // 加读锁
    defer m.RUnlock()  // 在函数返回前释放读锁

    return len(m.sessions)  // 返回会话映射表的大小
}

func (m *SessionManager) Count() int {
    m.RLock()  // 加读锁
    defer m.RUnlock()  // 在函数返回前释放读锁

    return int(m.count)  // 返回会话计数
}

func (m *SessionManager) Allocate() *Session {
    m.Lock()  // 加写锁
    defer m.Unlock()  // 在函数返回前释放写锁

    if m.closed {
        return nil  // 如果会话管理器已关闭，则返回nil
    }

    m.count++  // 会话计数加1
    s := &Session{
        ID:     m.count,  // 分配会话ID
        parent: m,  // 设置会话的父级为当前会话管理器
    }
    m.sessions[s.ID] = s  // 将会话添加到会话映射表中
    return s  // 返回分配的会话
}

func (m *SessionManager) Add(s *Session) {
    m.Lock()  // 加写锁
    defer m.Unlock()  // 在函数返回前释放写锁

    if m.closed {
        return  // 如果会话管理器已关闭，则直接返回
    }

    m.count++  // 会话计数加1
    m.sessions[s.ID] = s  // 将会话添加到会话映射表中
}

func (m *SessionManager) Remove(id uint16) {
    m.Lock()  // 加写锁
    defer m.Unlock()  // 在函数返回前释放写锁

    if m.closed {
        return  // 如果会话管理器已关闭，则直接返回
    }

    delete(m.sessions, id)  // 从会话映射表中删除指定ID的会话

    if len(m.sessions) == 0 {
        m.sessions = make(map[uint16]*Session, 16)  // 如果会话映射表为空，重新初始化
    }
}

func (m *SessionManager) Get(id uint16) (*Session, bool) {
    m.RLock()  // 加读锁
    defer m.RUnlock()  // 在函数返回前释放读锁

    if m.closed {
        return nil, false  // 如果会话管理器已关闭，则返回nil和false
    }

    s, found := m.sessions[id]  // 从会话映射表中获取指定ID的会话
    return s, found  // 返回获取的会话和是否找到的标志
}

func (m *SessionManager) CloseIfNoSession() bool {
    m.Lock()  // 加写锁
    defer m.Unlock()  // 在函数返回前释放写锁

    if m.closed {
        return true  // 如果会话管理器已关闭，则直接返回true
    }

    if len(m.sessions) != 0 {
        return false  // 如果会话映射表不为空，则返回false
    }

    m.closed = true  // 设置会话管理器为已关闭状态
    return true  // 返回true
}

func (m *SessionManager) Close() error {
    m.Lock()  // 加写锁
    defer m.Unlock()  // 在函数返回前释放写锁

    if m.closed {
        return nil  // 如果会话管理器已关闭，则直接返回nil
    }

    m.closed = true  // 设置会话管理器为已关闭状态
}
    // 遍历 m.sessions 切片中的每个元素，使用下划线 _ 忽略索引值
    for _, s := range m.sessions {
        // 关闭 s.input，忽略错误检查
        common.Close(s.input)  // nolint: errcheck
        // 关闭 s.output，忽略错误检查
        common.Close(s.output) // nolint: errcheck
    }

    // 将 m.sessions 切片置空
    m.sessions = nil
    // 返回空值
    return nil
// Session 表示 Mux 连接中的客户端连接
type Session struct {
    input        buf.Reader      // 用于读取数据的输入缓冲区
    output       buf.Writer      // 用于写入数据的输出缓冲区
    parent       *SessionManager // 指向 SessionManager 的指针，表示父级连接
    ID           uint16          // 会话的唯一标识符
    transferType protocol.TransferType // 数据传输类型
}

// Close 关闭与此会话关联的所有资源
func (s *Session) Close() error {
    common.Close(s.output) // nolint: errcheck  // 关闭输出缓冲区，忽略错误检查
    common.Close(s.input)  // nolint: errcheck  // 关闭输入缓冲区，忽略错误检查
    s.parent.Remove(s.ID)  // 从父级连接中移除当前会话
    return nil
}

// NewReader 根据此会话的传输类型创建一个基于 buf.BufferedReader 的 buf.Reader
func (s *Session) NewReader(reader *buf.BufferedReader) buf.Reader {
    if s.transferType == protocol.TransferTypeStream {
        return NewStreamReader(reader)  // 如果传输类型为流式传输，则创建一个新的流式读取器
    }
    return NewPacketReader(reader)  // 否则创建一个新的数据包读取器
}
```