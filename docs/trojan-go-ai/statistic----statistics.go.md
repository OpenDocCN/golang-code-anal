# `trojan-go\statistic\statistics.go`

```go
package statistic

import (
    "context" // 导入上下文包
    "io" // 导入输入输出包
    "strings" // 导入字符串处理包
    "sync" // 导入同步包

    "github.com/p4gefau1t/trojan-go/common" // 导入自定义包
    "github.com/p4gefau1t/trojan-go/log" // 导入日志包
)

// TrafficMeter 接口定义
type TrafficMeter interface {
    io.Closer // TrafficMeter 接口继承 io.Closer 接口
    Hash() string // 返回哈希值的方法
    AddTraffic(sent, recv int) // 添加流量的方法
    GetTraffic() (sent, recv uint64) // 获取流量的方法
    SetTraffic(sent, recv uint64) // 设置流量的方法
    ResetTraffic() (sent, recv uint64) // 重置流量的方法
    GetSpeed() (sent, recv uint64) // 获取速度的方法
    GetSpeedLimit() (sent, recv int) // 获取速度限制的方法
    SetSpeedLimit(sent, recv int) // 设置速度限制的方法
}

// IPRecorder 接口定义
type IPRecorder interface {
    AddIP(string) bool // 添加 IP 地址的方法
    DelIP(string) bool // 删除 IP 地址的方法
    GetIP() int // 获取 IP 地址数量的方法
    SetIPLimit(int) // 设置 IP 地址限制的方法
    GetIPLimit() int // 获取 IP 地址限制的方法
}

// User 接口定义，继承 TrafficMeter 和 IPRecorder 接口
type User interface {
    TrafficMeter
    IPRecorder
}

// Authenticator 接口定义，继承 io.Closer 接口
type Authenticator interface {
    io.Closer // Authenticator 接口继承 io.Closer 接口
    AuthUser(hash string) (valid bool, user User) // 鉴权用户的方法
    AddUser(hash string) error // 添加用户的方法
    DelUser(hash string) error // 删除用户的方法
    ListUsers() []User // 列出用户的方法
}

// Creator 函数类型定义
type Creator func(ctx context.Context) (Authenticator, error)

var (
    createdAuthLock sync.Mutex // 创建互斥锁
    authCreators    = make(map[string]Creator) // 创建 Authenticator 创建函数的映射
    createdAuth     = make(map[context.Context]Authenticator) // 创建 Authenticator 实例的映射
)

// RegisterAuthenticatorCreator 函数，注册 Authenticator 创建函数
func RegisterAuthenticatorCreator(name string, creator Creator) {
    authCreators[name] = creator // 将创建函数注册到映射中
}

// NewAuthenticator 函数，根据名称和上下文创建 Authenticator 实例
func NewAuthenticator(ctx context.Context, name string) (Authenticator, error) {
    // 为每个上下文分配一个唯一的 Authenticator
    createdAuthLock.Lock() // 避免并发地读写映射
    defer createdAuthLock.Unlock()
    if auth, found := createdAuth[ctx]; found {
        log.Debug("authenticator has been created:", name) // 记录日志
        return auth, nil
    }
    creator, found := authCreators[strings.ToUpper(name)] // 获取对应名称的创建函数
    if !found {
        return nil, common.NewError("auth driver name " + name + " not found") // 返回错误信息
    }
    auth, err := creator(ctx) // 调用创建函数创建 Authenticator 实例
    if err != nil {
        return nil, err // 返回错误信息
    }
    createdAuth[ctx] = auth // 将创建的 Authenticator 实例存储到映射中
    return auth, err // 返回 Authenticator 实例和错误信息
}
```