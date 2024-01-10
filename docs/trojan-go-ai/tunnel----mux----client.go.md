# `trojan-go\tunnel\mux\client.go`

```
package mux

import (
    "context"  // 上下文包，用于控制程序的执行流程
    "fmt"  // 格式化包，用于格式化输出
    "math/rand"  // 随机数包，用于生成随机数
    "sync"  // 同步包，用于实现并发控制
    "time"  // 时间包，用于处理时间相关操作

    "github.com/xtaci/smux"  // 第三方库，提供了基于 smux 的多路复用功能

    "github.com/p4gefau1t/trojan-go/common"  // 自定义包，包含了一些通用的函数和常量
    "github.com/p4gefau1t/trojan-go/config"  // 自定义包，包含了配置相关的函数和常量
    "github.com/p4gefau1t/trojan-go/log"  // 自定义包，包含了日志相关的函数和常量
    "github.com/p4gefau1t/trojan-go/tunnel"  // 自定义包，包含了隧道相关的函数和常量
)

type muxID uint32  // 定义了一个类型别名，用于表示多路复用的 ID

func generateMuxID() muxID {
    return muxID(rand.Uint32())  // 生成一个随机的多路复用 ID
}

type smuxClientInfo struct {
    id             muxID  // 多路复用 ID
    client         *smux.Session  // smux 会话
    lastActiveTime time.Time  // 最后活跃时间
    underlayConn   tunnel.Conn  // 底层连接
}

// Client is a smux client
type Client struct {
    clientPoolLock sync.Mutex  // 用于保护客户端池的互斥锁
    clientPool     map[muxID]*smuxClientInfo  // 客户端池，存储多路复用 ID 到客户端信息的映射
    underlay       tunnel.Client  // 底层客户端
    concurrency    int  // 并发数
    timeout        time.Duration  // 超时时间
    ctx            context.Context  // 上下文
    cancel         context.CancelFunc  // 取消函数
}

func (c *Client) Close() error {
    c.cancel()  // 取消上下文
    c.clientPoolLock.Lock()  // 加锁
    defer c.clientPoolLock.Unlock()  // 延迟解锁
    for id, info := range c.clientPool {  // 遍历客户端池
        info.client.Close()  // 关闭客户端
        log.Debug("mux client", id, "closed")  // 输出调试信息
    }
    return nil  // 返回空错误
}

func (c *Client) cleanLoop() {
    var checkDuration time.Duration  // 定义检查间隔时间
    if c.timeout <= 0 {  // 如果超时时间小于等于 0
        checkDuration = time.Second * 10  // 设置默认的检查间隔时间为 10 秒
        log.Warn("negative mux timeout")  // 输出警告信息
    } else {
        checkDuration = c.timeout / 4  // 计算检查间隔时间
    }
    log.Debug("check duration:", checkDuration.Seconds(), "s")  // 输出调试信息，显示检查间隔时间
}
    # 无限循环，监听两个 channel 的事件
    for {
        # 监听定时器事件，每隔一段时间执行一次清理操作
        select {
        case <-time.After(checkDuration):
            # 加锁，保护对 clientPool 的操作
            c.clientPoolLock.Lock()
            # 遍历 clientPool 中的每个客户端
            for id, info := range c.clientPool:
                # 如果客户端已经关闭，则关闭客户端和底层连接，并从 clientPool 中删除
                if info.client.IsClosed():
                    info.client.Close()
                    info.underlayConn.Close()
                    delete(c.clientPool, id)
                    log.Info("mux client", id, "is dead")
                # 如果客户端没有活跃的流并且超过了超时时间，则关闭客户端和底层连接，并从 clientPool 中删除
                else if info.client.NumStreams() == 0 && time.Since(info.lastActiveTime) > c.timeout:
                    info.client.Close()
                    info.underlayConn.Close()
                    delete(c.clientPool, id)
                    log.Info("mux client", id, "is closed due to inactivity")
            # 输出当前 clientPool 中的客户端数量
            log.Debug("current mux clients: ", len(c.clientPool))
            # 输出每个客户端的信息
            for id, info := range c.clientPool:
                log.Debug(fmt.Sprintf("  - %x: %d/%d", id, info.client.NumStreams(), c.concurrency))
            # 解锁
            c.clientPoolLock.Unlock()
        # 监听上下文的 Done 事件，用于优雅关闭
        case <-c.ctx.Done():
            # 输出日志，表示正在关闭 mux cleaner
            log.Debug("shutting down mux cleaner..")
            # 加锁，保护对 clientPool 的操作
            c.clientPoolLock.Lock()
            # 遍历 clientPool 中的每个客户端，关闭客户端和底层连接，并从 clientPool 中删除
            for id, info := range c.clientPool:
                info.client.Close()
                info.underlayConn.Close()
                delete(c.clientPool, id)
                log.Debug("mux client", id, "closed")
            # 解锁
            c.clientPoolLock.Unlock()
            # 退出循环
            return
        }
    }
}
// 创建新的多路复用客户端
func (c *Client) newMuxClient() (*smuxClientInfo, error) {
    // 在调用此函数时应锁定互斥锁
    id := generateMuxID()
    // 检查是否存在重复的 ID
    if _, found := c.clientPool[id]; found {
        return nil, common.NewError("duplicated id")
    }

    // 创建一个虚假地址
    fakeAddr := &tunnel.Address{
        DomainName:  "MUX_CONN",
        AddressType: tunnel.DomainName,
    }
    // 使用底层连接拨号连接
    conn, err := c.underlay.DialConn(fakeAddr, &Tunnel{})
    if err != nil {
        return nil, common.NewError("mux failed to dial").Base(err)
    }
    // 创建一个新的粘性连接
    conn = newStickyConn(conn)

    // 设置 smux 配置
    smuxConfig := smux.DefaultConfig()
    // 创建 smux 客户端
    client, _ := smux.Client(conn, smuxConfig)
    // 创建 smux 客户端信息
    info := &smuxClientInfo{
        client:         client,
        underlayConn:   conn,
        id:             id,
        lastActiveTime: time.Now(),
    }
    // 将信息添加到客户端池中
    c.clientPool[id] = info
    return info, nil
}

// 拨号连接
func (c *Client) DialConn(*tunnel.Address, tunnel.Tunnel) (tunnel.Conn, error) {
    // 创建新连接函数
    createNewConn := func(info *smuxClientInfo) (tunnel.Conn, error) {
        // 打开新的流
        rwc, err := info.client.Open()
        // 更新最后活动时间
        info.lastActiveTime = time.Now()
        if err != nil {
            // 关闭底层连接和客户端
            info.underlayConn.Close()
            info.client.Close()
            // 从客户端池中删除信息
            delete(c.clientPool, info.id)
            return nil, common.NewError("mux failed to open stream from client").Base(err)
        }
        return &Conn{
            rwc:  rwc,
            Conn: info.underlayConn,
        }, nil
    }

    // 锁定客户端池
    c.clientPoolLock.Lock()
    defer c.clientPoolLock.Unlock()
    for _, info := range c.clientPool {
        // 检查客户端是否已关闭
        if info.client.IsClosed() {
            delete(c.clientPool, info.id)
            log.Info(fmt.Sprintf("Mux client %x is closed", info.id))
            continue
        }
        // 检查客户端流的数量是否小于并发数或并发数小于等于0
        if info.client.NumStreams() < c.concurrency || c.concurrency <= 0 {
            return createNewConn(info)
        }
    }

    // 创建新的多路复用客户端
    info, err := c.newMuxClient()
    # 如果错误不为空，则返回空值和新的错误对象，错误信息为“未找到可用的多路复用客户端”，基于原始错误err
    if err != nil:
        return nil, common.NewError("no available mux client found").Base(err)
    # 调用createNewConn函数创建新的连接，并返回结果
    return createNewConn(info)
}

// DialPacket 方法用于建立与服务器的数据包连接，但当前不支持该功能，因此触发 panic
func (c *Client) DialPacket(tunnel.Tunnel) (tunnel.PacketConn, error) {
    panic("not supported")
}

// NewClient 方法用于创建一个新的客户端对象
func NewClient(ctx context.Context, underlay tunnel.Client) (*Client, error) {
    // 从上下文中获取客户端配置
    clientConfig := config.FromContext(ctx, Name).(*Config)
    // 创建一个新的上下文，并返回一个取消函数
    ctx, cancel := context.WithCancel(ctx)
    // 初始化客户端对象
    client := &Client{
        underlay:    underlay,
        concurrency: clientConfig.Mux.Concurrency,
        timeout:     time.Duration(clientConfig.Mux.IdleTimeout) * time.Second,
        ctx:         ctx,
        cancel:      cancel,
        clientPool:  make(map[muxID]*smuxClientInfo),
    }
    // 启动清理循环
    go client.cleanLoop()
    // 输出调试信息
    log.Debug("mux client created")
    // 返回客户端对象和 nil 错误
    return client, nil
}
```