# `trojan-go\statistic\memory\memory.go`

```go
package memory

import (
    "context"  // 导入 context 包
    "sync"  // 导入 sync 包
    "sync/atomic"  // 导入 sync/atomic 包
    "time"  // 导入 time 包

    "golang.org/x/time/rate"  // 导入 rate 包

    "github.com/p4gefau1t/trojan-go/common"  // 导入 common 包
    "github.com/p4gefau1t/trojan-go/config"  // 导入 config 包
    "github.com/p4gefau1t/trojan-go/log"  // 导入 log 包
    "github.com/p4gefau1t/trojan-go/statistic"  // 导入 statistic 包
)

const Name = "MEMORY"  // 定义常量 Name 为 "MEMORY"

type User struct {
    // WARNING: do not change the order of these fields.
    // 64-bit fields that use `sync/atomic` package functions
    // must be 64-bit aligned on 32-bit systems.
    // Reference: https://github.com/golang/go/issues/599
    // Solution: https://github.com/golang/go/issues/11891#issuecomment-433623786
    sent      uint64  // 定义 uint64 类型的 sent 字段
    recv      uint64  // 定义 uint64 类型的 recv 字段
    lastSent  uint64  // 定义 uint64 类型的 lastSent 字段
    lastRecv  uint64  // 定义 uint64 类型的 lastRecv 字段
    sendSpeed uint64  // 定义 uint64 类型的 sendSpeed 字段
    recvSpeed uint64  // 定义 uint64 类型的 recvSpeed 字段

    hash        string  // 定义 string 类型的 hash 字段
    ipTable     sync.Map  // 定义 sync.Map 类型的 ipTable 字段
    ipNum       int32  // 定义 int32 类型的 ipNum 字段
    maxIPNum    int  // 定义 int 类型的 maxIPNum 字段
    limiterLock sync.RWMutex  // 定义 sync.RWMutex 类型的 limiterLock 字段
    sendLimiter *rate.Limiter  // 定义 *rate.Limiter 类型的 sendLimiter 字段
    recvLimiter *rate.Limiter  // 定义 *rate.Limiter 类型的 recvLimiter 字段
    ctx         context.Context  // 定义 context.Context 类型的 ctx 字段
    cancel      context.CancelFunc  // 定义 context.CancelFunc 类型的 cancel 字段
}

func (u *User) Close() error {
    u.ResetTraffic()  // 调用 ResetTraffic 方法
    u.cancel()  // 调用 cancel 方法
    return nil  // 返回空值
}

func (u *User) AddIP(ip string) bool {
    if u.maxIPNum <= 0 {  // 如果 maxIPNum 小于等于 0
        return true  // 返回 true
    }
    _, found := u.ipTable.Load(ip)  // 从 ipTable 中加载 ip
    if found {  // 如果找到
        return true  // 返回 true
    }
    if int(u.ipNum)+1 > u.maxIPNum {  // 如果 ipNum 加 1 后大于 maxIPNum
        return false  // 返回 false
    }
    u.ipTable.Store(ip, true)  // 将 ip 存储到 ipTable 中
    atomic.AddInt32(&u.ipNum, 1)  // 使用原子操作将 1 加到 ipNum 中
    return true  // 返回 true
}

func (u *User) DelIP(ip string) bool {
    if u.maxIPNum <= 0 {  // 如果 maxIPNum 小于等于 0
        return true  // 返回 true
    }
    _, found := u.ipTable.Load(ip)  // 从 ipTable 中加载 ip
    if !found {  // 如果未找到
        return false  // 返回 false
    }
    u.ipTable.Delete(ip)  // 从 ipTable 中删除 ip
    atomic.AddInt32(&u.ipNum, -1)  // 使用原子操作将 -1 加到 ipNum 中
    return true  // 返回 true
}

func (u *User) GetIP() int {
    return int(u.ipNum)  // 返回 ipNum 的整数值
}

func (u *User) SetIPLimit(n int) {
    u.maxIPNum = n  // 设置 maxIPNum 为 n
}

func (u *User) GetIPLimit() int {
    return u.maxIPNum  // 返回 maxIPNum 的值
}

func (u *User) AddTraffic(sent, recv int) {
    u.limiterLock.RLock()  // 读取锁定 limiterLock
    defer u.limiterLock.RUnlock()  // 延迟解锁 limiterLock
    # 如果发送限制器不为空并且已发送数据大于等于0
    if u.sendLimiter != nil && sent >= 0:
        # 等待发送限制器释放指定数量的资源
        u.sendLimiter.WaitN(u.ctx, sent)
    # 如果发送限制器为空并且接收数据大于等于0
    elif u.recvLimiter != nil && recv >= 0:
        # 等待接收限制器释放指定数量的资源
        u.recvLimiter.WaitN(u.ctx, recv)
    # 原子操作，将已发送数据加上指定数量
    atomic.AddUint64(&u.sent, uint64(sent))
    # 原子操作，将已接收数据加上指定数量
    atomic.AddUint64(&u.recv, uint64(recv))
}
// 设置用户的发送和接收速率限制
func (u *User) SetSpeedLimit(send, recv int) {
    // 加锁，确保并发安全
    u.limiterLock.Lock()
    // 在函数返回时解锁
    defer u.limiterLock.Unlock()

    // 如果发送速率小于等于0，则将发送限制器设置为nil
    if send <= 0 {
        u.sendLimiter = nil
    } else {
        // 否则根据发送速率创建发送限制器
        u.sendLimiter = rate.NewLimiter(rate.Limit(send), send*2)
    }
    // 如果接收速率小于等于0，则将接收限制器设置为nil
    if recv <= 0 {
        u.recvLimiter = nil
    } else {
        // 否则根据接收速率创建接收限制器
        u.recvLimiter = rate.NewLimiter(rate.Limit(recv), recv*2)
    }
}

// 获取用户的速率限制
func (u *User) GetSpeedLimit() (send, recv int) {
    // 加读锁，确保并发安全
    u.limiterLock.RLock()
    // 在函数返回时解锁
    defer u.limiterLock.RUnlock()

    // 如果发送限制器不为nil，则获取发送速率限制
    if u.sendLimiter != nil {
        send = int(u.sendLimiter.Limit())
    }
    // 如果接收限制器不为nil，则获取接收速率限制
    if u.recvLimiter != nil {
        recv = int(u.recvLimiter.Limit())
    }
    return
}

// 返回用户的哈希值
func (u *User) Hash() string {
    return u.hash
}

// 设置用户的流量发送和接收量
func (u *User) SetTraffic(send, recv uint64) {
    // 原子性地设置发送和接收量
    atomic.StoreUint64(&u.sent, send)
    atomic.StoreUint64(&u.recv, recv)
}

// 获取用户的流量发送和接收量
func (u *User) GetTraffic() (uint64, uint64) {
    return atomic.LoadUint64(&u.sent), atomic.LoadUint64(&u.recv)
}

// 重置用户的流量发送和接收量
func (u *User) ResetTraffic() (uint64, uint64) {
    // 原子性地交换并重置发送和接收量
    sent := atomic.SwapUint64(&u.sent, 0)
    recv := atomic.SwapUint64(&u.recv, 0)
    // 重置最后发送和接收量
    atomic.StoreUint64(&u.lastSent, 0)
    atomic.StoreUint64(&u.lastRecv, 0)
    return sent, recv
}

// 更新用户的速率
func (u *User) speedUpdater() {
    // 创建定时器，每秒触发一次
    ticker := time.NewTicker(time.Second)
    for {
        select {
        case <-u.ctx.Done():
            return
        case <-ticker.C:
            // 获取当前流量发送和接收量
            sent, recv := u.GetTraffic()
            // 计算并存储发送和接收速率
            atomic.StoreUint64(&u.sendSpeed, sent-u.lastSent)
            atomic.StoreUint64(&u.recvSpeed, recv-u.lastRecv)
            // 更新最后发送和接收量
            atomic.StoreUint64(&u.lastSent, sent)
            atomic.StoreUint64(&u.lastRecv, recv)
        }
    }
}

// 获取用户的发送和接收速率
func (u *User) GetSpeed() (uint64, uint64) {
    return atomic.LoadUint64(&u.sendSpeed), atomic.LoadUint64(&u.recvSpeed)
}

// 用户认证器结构体
type Authenticator struct {
    users sync.Map
    ctx   context.Context
}

// 认证用户，返回是否找到用户和用户对象
func (a *Authenticator) AuthUser(hash string) (bool, statistic.User) {
    if user, found := a.users.Load(hash); found {
        return true, user.(*User)
    # 返回布尔值 false 和空值 nil
    }
    return false, nil
# 向 Authenticator 中添加用户，使用用户的哈希值作为参数
func (a *Authenticator) AddUser(hash string) error {
    # 检查用户是否已存在，如果存在则返回错误
    if _, found := a.users.Load(hash); found {
        return common.NewError("hash " + hash + " is already exist")
    }
    # 创建一个带有取消函数的上下文
    ctx, cancel := context.WithCancel(a.ctx)
    # 创建一个 User 对象，并初始化其属性
    meter := &User{
        hash:   hash,
        ctx:    ctx,
        cancel: cancel,
    }
    # 启动用户速度更新的协程
    go meter.speedUpdater()
    # 将用户对象存储到 Authenticator 中
    a.users.Store(hash, meter)
    return nil
}

# 从 Authenticator 中删除用户，使用用户的哈希值作为参数
func (a *Authenticator) DelUser(hash string) error {
    # 从 Authenticator 中加载用户对象
    meter, found := a.users.Load(hash)
    # 如果用户不存在，则返回错误
    if !found {
        return common.NewError("hash " + hash + " not found")
    }
    # 关闭用户对象
    meter.(*User).Close()
    # 从 Authenticator 中删除用户对象
    a.users.Delete(hash)
    return nil
}

# 列出 Authenticator 中的所有用户
func (a *Authenticator) ListUsers() []statistic.User {
    # 创建一个空的用户列表
    result := make([]statistic.User, 0)
    # 遍历 Authenticator 中的所有用户，并将其添加到列表中
    a.users.Range(func(k, v interface{}) bool {
        result = append(result, v.(*User))
        return true
    })
    return result
}

# 关闭 Authenticator
func (a *Authenticator) Close() error {
    return nil
}

# 创建一个新的 Authenticator 对象
func NewAuthenticator(ctx context.Context) (statistic.Authenticator, error) {
    # 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    # 创建一个 Authenticator 对象
    u := &Authenticator{
        ctx: ctx,
    }
    # 遍历配置中的密码列表，为每个密码创建一个用户并添加到 Authenticator 中
    for _, password := range cfg.Passwords {
        hash := common.SHA224String(password)
        u.AddUser(hash)
    }
    # 输出日志信息
    log.Debug("memory authenticator created")
    return u, nil
}

# 在初始化时注册 Authenticator 的创建函数
func init() {
    statistic.RegisterAuthenticatorCreator(Name, NewAuthenticator)
}
```