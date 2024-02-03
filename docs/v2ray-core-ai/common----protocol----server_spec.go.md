# `v2ray-core\common\protocol\server_spec.go`

```go
package protocol

import (
    "sync"  // 导入 sync 包，用于实现并发安全的数据结构
    "time"  // 导入 time 包，用于处理时间相关的操作

    "v2ray.com/core/common/dice"  // 导入 v2ray.com/core/common/dice 包，用于生成随机数
    "v2ray.com/core/common/net"   // 导入 v2ray.com/core/common/net 包，用于处理网络相关操作
)

type ValidationStrategy interface {
    IsValid() bool   // 定义接口 ValidationStrategy，包含 IsValid 方法，用于判断是否有效
    Invalidate()     // 定义接口 ValidationStrategy，包含 Invalidate 方法，用于使其失效
}

type alwaysValidStrategy struct{}  // 定义 alwaysValidStrategy 结构体

func AlwaysValid() ValidationStrategy {  // 定义 AlwaysValid 函数，返回 alwaysValidStrategy 结构体
    return alwaysValidStrategy{}
}

func (alwaysValidStrategy) IsValid() bool {  // 实现 alwaysValidStrategy 结构体的 IsValid 方法
    return true
}

func (alwaysValidStrategy) Invalidate() {}  // 实现 alwaysValidStrategy 结构体的 Invalidate 方法

type timeoutValidStrategy struct {  // 定义 timeoutValidStrategy 结构体
    until time.Time  // 结构体包含一个 time.Time 类型的字段 until
}

func BeforeTime(t time.Time) ValidationStrategy {  // 定义 BeforeTime 函数，返回 timeoutValidStrategy 结构体指针
    return &timeoutValidStrategy{
        until: t,  // 初始化 timeoutValidStrategy 结构体的 until 字段
    }
}

func (s *timeoutValidStrategy) IsValid() bool {  // 实现 timeoutValidStrategy 结构体的 IsValid 方法
    return s.until.After(time.Now())  // 判断当前时间是否在 until 时间之前
}

func (s *timeoutValidStrategy) Invalidate() {  // 实现 timeoutValidStrategy 结构体的 Invalidate 方法
    s.until = time.Time{}  // 使 until 时间失效
}

type ServerSpec struct {  // 定义 ServerSpec 结构体
    sync.RWMutex  // 嵌入 sync.RWMutex 结构体，实现读写锁
    dest  net.Destination  // 结构体包含 net.Destination 类型的字段 dest
    users []*MemoryUser  // 结构体包含 []*MemoryUser 类型的字段 users
    valid ValidationStrategy  // 结构体包含 ValidationStrategy 接口类型的字段 valid
}

func NewServerSpec(dest net.Destination, valid ValidationStrategy, users ...*MemoryUser) *ServerSpec {  // 定义 NewServerSpec 函数，返回 ServerSpec 结构体指针
    return &ServerSpec{
        dest:  dest,  // 初始化 ServerSpec 结构体的 dest 字段
        users: users,  // 初始化 ServerSpec 结构体的 users 字段
        valid: valid,  // 初始化 ServerSpec 结构体的 valid 字段
    }
}

func NewServerSpecFromPB(spec *ServerEndpoint) (*ServerSpec, error) {  // 定义 NewServerSpecFromPB 函数，返回 ServerSpec 结构体指针和错误
    dest := net.TCPDestination(spec.Address.AsAddress(), net.Port(spec.Port))  // 根据 ServerEndpoint 生成 net.Destination
    mUsers := make([]*MemoryUser, len(spec.User))  // 创建 []*MemoryUser 类型的切片
    for idx, u := range spec.User {  // 遍历 ServerEndpoint 的用户
        mUser, err := u.ToMemoryUser()  // 将用户转换为 MemoryUser
        if err != nil {
            return nil, err  // 如果转换出错，返回错误
        }
        mUsers[idx] = mUser  // 将转换后的 MemoryUser 放入切片中
    }
    return NewServerSpec(dest, AlwaysValid(), mUsers...), nil  // 返回根据 ServerEndpoint 生成的 ServerSpec 结构体指针和 nil 错误
}

func (s *ServerSpec) Destination() net.Destination {  // 定义 ServerSpec 结构体的 Destination 方法，返回 net.Destination
    return s.dest  // 返回 ServerSpec 结构体的 dest 字段
}

func (s *ServerSpec) HasUser(user *MemoryUser) bool {  // 定义 ServerSpec 结构体的 HasUser 方法，判断是否存在指定用户
    s.RLock()  // 加读锁
    defer s.RUnlock()  // 延迟释放读锁

    for _, u := range s.users {  // 遍历用户列表
        if u.Account.Equals(user.Account) {  // 判断用户是否存在
            return true  // 存在则返回 true
        }
    }
    return false  // 不存在则返回 false
}

func (s *ServerSpec) AddUser(user *MemoryUser) {  // 定义 ServerSpec 结构体的 AddUser 方法，添加用户
    if s.HasUser(user) {  // 判断用户是否已存在
        return  // 如果已存在则直接返回
    }

    s.Lock()  // 加写锁
    defer s.Unlock()  // 延迟释放写锁

    s.users = append(s.users, user)  // 将用户添加到用户列表中
}
# 从服务器规格中选择一个用户，使用读锁进行保护
func (s *ServerSpec) PickUser() *MemoryUser {
    # 获取读锁
    s.RLock()
    # 延迟释放读锁
    defer s.RUnlock()

    # 获取用户数量
    userCount := len(s.users)
    # 根据用户数量进行不同的处理
    switch userCount {
    case 0:
        # 如果用户数量为0，则返回空
        return nil
    case 1:
        # 如果用户数量为1，则返回第一个用户
        return s.users[0]
    default:
        # 如果用户数量大于1，则随机选择一个用户
        return s.users[dice.Roll(userCount)]
    }
}

# 检查服务器规格是否有效
func (s *ServerSpec) IsValid() bool {
    # 调用valid对象的IsValid方法来检查服务器规格是否有效
    return s.valid.IsValid()
}

# 使服务器规格无效
func (s *ServerSpec) Invalidate() {
    # 调用valid对象的Invalidate方法来使服务器规格无效
    s.valid.Invalidate()
}
```