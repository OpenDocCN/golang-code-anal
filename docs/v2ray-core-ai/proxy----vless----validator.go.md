# `v2ray-core\proxy\vless\validator.go`

```
// +build !confonly
// 声明当前文件不仅仅是配置文件

package vless
// 声明包名为vless

import (
    "strings"
    "sync"

    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/uuid"
)
// 导入所需的包

type Validator struct {
    // Considering email's usage here, map + sync.Mutex/RWMutex may have better performance.
    email sync.Map
    users sync.Map
}
// 定义结构体Validator，包含email和users两个sync.Map类型的字段

func (v *Validator) Add(u *protocol.MemoryUser) error {
    // 添加用户到Validator中
    if u.Email != "" {
        // 如果用户的邮箱不为空
        _, loaded := v.email.LoadOrStore(strings.ToLower(u.Email), u)
        // 将用户的邮箱转换为小写，存储到email中
        if loaded {
            return newError("User ", u.Email, " already exists.")
            // 如果邮箱已存在，则返回错误信息
        }
    }
    v.users.Store(u.Account.(*MemoryAccount).ID.UUID(), u)
    // 存储用户到users中
    return nil
}

func (v *Validator) Del(e string) error {
    // 从Validator中删除用户
    if e == "" {
        return newError("Email must not be empty.")
        // 如果邮箱为空，则返回错误信息
    }
    le := strings.ToLower(e)
    u, _ := v.email.Load(le)
    // 将邮箱转换为小写，从email中加载用户
    if u == nil {
        return newError("User ", e, " not found.")
        // 如果用户不存在，则返回错误信息
    }
    v.email.Delete(le)
    v.users.Delete(u.(*protocol.MemoryUser).Account.(*MemoryAccount).ID.UUID())
    // 从email和users中删除用户
    return nil
}

func (v *Validator) Get(id uuid.UUID) *protocol.MemoryUser {
    // 通过ID获取用户
    u, _ := v.users.Load(id)
    // 从users中加载用户
    if u != nil {
        return u.(*protocol.MemoryUser)
        // 如果用户存在，则返回用户信息
    }
    return nil
    // 如果用户不存在，则返回nil
}
```