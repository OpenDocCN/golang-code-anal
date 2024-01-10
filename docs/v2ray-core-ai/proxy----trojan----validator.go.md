# `v2ray-core\proxy\trojan\validator.go`

```
// trojan 包声明
package trojan

// 导入必要的包
import (
    "sync"  // 同步操作包
    "v2ray.com/core/common/protocol"  // V2Ray 协议包
)

// Validator 结构体，用于存储有效的 trojan 用户
type Validator struct {
    users sync.Map  // 使用 sync.Map 存储用户信息
}

// Add 方法，用于添加 trojan 用户
func (v *Validator) Add(u *protocol.MemoryUser) error {
    user := u.Account.(*MemoryAccount)  // 获取用户的账户信息
    v.users.Store(hexString(user.Key), u)  // 将用户信息存储到 sync.Map 中
    return nil  // 返回空错误
}

// Get 方法，根据哈希值获取用户信息，如果用户不存在则返回 nil
func (v *Validator) Get(hash string) *protocol.MemoryUser {
    u, _ := v.users.Load(hash)  // 从 sync.Map 中加载用户信息
    if u != nil {  // 如果用户信息不为空
        return u.(*protocol.MemoryUser)  // 返回用户信息
    }
    return nil  // 返回空
}
```