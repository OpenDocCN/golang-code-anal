# `kubo\core\node\libp2p\rcmgr_logging.go`

```go
package libp2p

import (
    "context"  // 导入上下文包，用于处理请求的取消和超时
    "errors"   // 导入错误包，用于处理错误信息
    "sync"     // 导入同步包，用于实现并发控制
    "time"     // 导入时间包，用于处理时间相关操作

    "github.com/benbjohnson/clock"  // 导入第三方库，用于时钟操作
    "github.com/libp2p/go-libp2p/core/network"  // 导入 libp2p 网络核心包
    "github.com/libp2p/go-libp2p/core/peer"     // 导入 libp2p 对等核心包
    "github.com/libp2p/go-libp2p/core/protocol" // 导入 libp2p 协议核心包
    rcmgr "github.com/libp2p/go-libp2p/p2p/host/resource-manager"  // 导入 libp2p 资源管理包
    ma "github.com/multiformats/go-multiaddr"  // 导入多格式多地址包
    "go.uber.org/zap"  // 导入 Uber 的日志库
)

type loggingResourceManager struct {
    clock       clock.Clock  // 时钟对象
    logger      *zap.SugaredLogger  // 日志记录器
    delegate    network.ResourceManager  // 网络资源管理器
    logInterval time.Duration  // 日志间隔时间

    mut               sync.Mutex  // 互斥锁
    limitExceededErrs map[string]int  // 超出限制的错误信息
}

type loggingScope struct {
    logger    *zap.SugaredLogger  // 日志记录器
    delegate  network.ResourceScope  // 网络资源范围
    countErrs func(error)  // 统计错误信息
}

var (
    _ network.ResourceManager    = (*loggingResourceManager)(nil)  // 网络资源管理器接口的实现
    _ rcmgr.ResourceManagerState = (*loggingResourceManager)(nil)  // 资源管理器状态接口的实现
)

func (n *loggingResourceManager) start(ctx context.Context) {
    logInterval := n.logInterval  // 获取日志间隔时间
    if logInterval == 0 {  // 如果日志间隔时间为0
        logInterval = 10 * time.Second  // 设置默认的日志间隔时间为10秒
    }
    ticker := n.clock.Ticker(logInterval)  // 创建定时器
    go func() {  // 启动协程
        defer ticker.Stop()  // 延迟关闭定时器
        for {  // 循环
            select {  // 选择
            case <-ticker.C:  // 定时器触发
                n.mut.Lock()  // 加锁
                errs := n.limitExceededErrs  // 获取超出限制的错误信息
                n.limitExceededErrs = make(map[string]int)  // 清空超出限制的错误信息

                for e, count := range errs {  // 遍历错误信息
                    n.logger.Warnf("Protected from exceeding resource limits %d times.  libp2p message: %q.", count, e)  // 记录警告日志
                }

                if len(errs) != 0 {  // 如果错误信息不为空
                    n.logger.Warnf("Learn more about potential actions to take at: https://github.com/ipfs/kubo/blob/master/docs/libp2p-resource-management.md")  // 记录警告日志
                }

                n.mut.Unlock()  // 解锁
            case <-ctx.Done():  // 上下文取消
                return  // 返回
            }
        }
    }()
}

func (n *loggingResourceManager) countErrs(err error) {
    # 如果错误是资源限制超出的错误
    if errors.Is(err, network.ErrResourceLimitExceeded) {
        # 对共享资源进行加锁
        n.mut.Lock()
        # 如果限制超出错误的映射为空，则创建一个新的映射
        if n.limitExceededErrs == nil {
            n.limitExceededErrs = make(map[string]int)
        }

        # 解包错误以获取限制范围和达到的限制类型
        eout := errors.Unwrap(err)
        # 如果解包后的错误不为空
        if eout != nil {
            # 将解包后的错误信息添加到限制超出错误的映射中，并增加计数
            n.limitExceededErrs[eout.Error()]++
        }

        # 解锁共享资源
        n.mut.Unlock()
    }
# 查看系统资源，调用委托对象的 ViewSystem 方法
func (n *loggingResourceManager) ViewSystem(f func(network.ResourceScope) error) error {
    return n.delegate.ViewSystem(f)
}

# 查看临时资源，调用委托对象的 ViewTransient 方法
func (n *loggingResourceManager) ViewTransient(f func(network.ResourceScope) error) error {
    return n.delegate.ViewTransient(func(s network.ResourceScope) error {
        return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
    })
}

# 查看服务资源，调用委托对象的 ViewService 方法
func (n *loggingResourceManager) ViewService(svc string, f func(network.ServiceScope) error) error {
    return n.delegate.ViewService(svc, func(s network.ServiceScope) error {
        return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
    })
}

# 查看协议资源，调用委托对象的 ViewProtocol 方法
func (n *loggingResourceManager) ViewProtocol(p protocol.ID, f func(network.ProtocolScope) error) error {
    return n.delegate.ViewProtocol(p, func(s network.ProtocolScope) error {
        return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
    })
}

# 查看对等节点资源，调用委托对象的 ViewPeer 方法
func (n *loggingResourceManager) ViewPeer(p peer.ID, f func(network.PeerScope) error) error {
    return n.delegate.ViewPeer(p, func(s network.PeerScope) error {
        return f(&loggingScope{logger: n.logger, delegate: s, countErrs: n.countErrs})
    })
}

# 打开连接，调用委托对象的 OpenConnection 方法
func (n *loggingResourceManager) OpenConnection(dir network.Direction, usefd bool, remote ma.Multiaddr) (network.ConnManagementScope, error) {
    connMgmtScope, err := n.delegate.OpenConnection(dir, usefd, remote)
    n.countErrs(err)
    return connMgmtScope, err
}

# 打开流，调用委托对象的 OpenStream 方法
func (n *loggingResourceManager) OpenStream(p peer.ID, dir network.Direction) (network.StreamManagementScope, error) {
    connMgmtScope, err := n.delegate.OpenStream(p, dir)
    n.countErrs(err)
    return connMgmtScope, err
}

# 关闭资源管理器，调用委托对象的 Close 方法
func (n *loggingResourceManager) Close() error {
    return n.delegate.Close()
}

# 列出服务，检查委托对象是否实现了 ResourceManagerState 接口，如果是则调用 ListServices 方法
func (n *loggingResourceManager) ListServices() []string {
    rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
    if !ok {
        return nil
    }

    return rapi.ListServices()
}
# 返回支持的协议列表
func (n *loggingResourceManager) ListProtocols() []protocol.ID {
    # 尝试将委托对象转换为资源管理器状态对象
    rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
    # 如果转换失败，则返回空列表
    if !ok {
        return nil
    }
    # 调用资源管理器状态对象的ListProtocols方法并返回结果
    return rapi.ListProtocols()
}

# 返回对等节点列表
func (n *loggingResourceManager) ListPeers() []peer.ID {
    # 尝试将委托对象转换为资源管理器状态对象
    rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
    # 如果转换失败，则返回空列表
    if !ok {
        return nil
    }
    # 调用资源管理器状态对象的ListPeers方法并返回结果
    return rapi.ListPeers()
}

# 返回资源管理器的统计信息
func (n *loggingResourceManager) Stat() rcmgr.ResourceManagerStat {
    # 尝试将委托对象转换为资源管理器状态对象
    rapi, ok := n.delegate.(rcmgr.ResourceManagerState)
    # 如果转换失败，则返回空的资源管理器统计信息对象
    if !ok {
        return rcmgr.ResourceManagerStat{}
    }
    # 调用资源管理器状态对象的Stat方法并返回结果
    return rapi.Stat()
}

# 申请内存资源
func (s *loggingScope) ReserveMemory(size int, prio uint8) error {
    # 调用委托对象的ReserveMemory方法申请内存资源
    err := s.delegate.ReserveMemory(size, prio)
    # 记录错误信息
    s.countErrs(err)
    # 返回错误信息
    return err
}

# 释放内存资源
func (s *loggingScope) ReleaseMemory(size int) {
    # 调用委托对象的ReleaseMemory方法释放内存资源
    s.delegate.ReleaseMemory(size)
}

# 返回资源范围的统计信息
func (s *loggingScope) Stat() network.ScopeStat {
    # 调用委托对象的Stat方法返回资源范围的统计信息
    return s.delegate.Stat()
}

# 开始一个资源范围的跟踪
func (s *loggingScope) BeginSpan() (network.ResourceScopeSpan, error) {
    # 调用委托对象的BeginSpan方法开始一个资源范围的跟踪
    return s.delegate.BeginSpan()
}

# 结束资源范围的跟踪
func (s *loggingScope) Done() {
    # 调用委托对象的Done方法结束资源范围的跟踪
    s.delegate.(network.ResourceScopeSpan).Done()
}

# 返回资源范围的名称
func (s *loggingScope) Name() string {
    # 调用委托对象的Name方法返回资源范围的名称
    return s.delegate.(network.ServiceScope).Name()
}

# 返回资源范围所属的协议
func (s *loggingScope) Protocol() protocol.ID {
    # 调用委托对象的Protocol方法返回资源范围所属的协议
    return s.delegate.(network.ProtocolScope).Protocol()
}

# 返回资源范围所属的对等节点
func (s *loggingScope) Peer() peer.ID {
    # 调用委托对象的Peer方法返回资源范围所属的对等节点
    return s.delegate.(network.PeerScope).Peer()
}

# 返回资源范围的对等节点范围
func (s *loggingScope) PeerScope() network.PeerScope {
    # 调用委托对象的PeerScope方法返回资源范围的对等节点范围
    return s.delegate.(network.PeerScope)
}

# 设置资源范围的对等节点
func (s *loggingScope) SetPeer(p peer.ID) error {
    # 调用委托对象的SetPeer方法设置资源范围的对等节点
    err := s.delegate.(network.ConnManagementScope).SetPeer(p)
    # 记录错误信息
    s.countErrs(err)
    # 返回错误信息
    return err
}

# 返回资源范围的协议范围
func (s *loggingScope) ProtocolScope() network.ProtocolScope {
    # 调用委托对象的ProtocolScope方法返回资源范围的协议范围
    return s.delegate.(network.ProtocolScope)
}

# 设置资源范围的协议
func (s *loggingScope) SetProtocol(proto protocol.ID) error {
    # 调用委托对象的SetProtocol方法设置资源范围的协议
    err := s.delegate.(network.StreamManagementScope).SetProtocol(proto)
    # 记录错误信息
    s.countErrs(err)
    # 返回错误信息
    return err
}
# 返回 loggingScope 结构体中 delegate 字段的 network.ServiceScope 接口
func (s *loggingScope) ServiceScope() network.ServiceScope {
    return s.delegate.(network.ServiceScope)
}

# 设置 loggingScope 结构体中 delegate 字段的 network.StreamManagementScope 接口的服务，并返回可能的错误
func (s *loggingScope) SetService(srv string) error {
    # 调用 delegate 字段的 network.StreamManagementScope 接口的 SetService 方法，并将可能的错误赋值给 err
    err := s.delegate.(network.StreamManagementScope).SetService(srv)
    # 调用 countErrs 方法，记录错误信息
    s.countErrs(err)
    # 返回可能的错误
    return err
}

# 返回 loggingScope 结构体中 delegate 字段的 rcmgr.ResourceScopeLimiter 接口的 Limit
func (s *loggingScope) Limit() rcmgr.Limit {
    return s.delegate.(rcmgr.ResourceScopeLimiter).Limit()
}

# 设置 loggingScope 结构体中 delegate 字段的 rcmgr.ResourceScopeLimiter 接口的 Limit
func (s *loggingScope) SetLimit(limit rcmgr.Limit) {
    s.delegate.(rcmgr.ResourceScopeLimiter).SetLimit(limit)
}
```