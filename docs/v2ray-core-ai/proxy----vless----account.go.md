# `v2ray-core\proxy\vless\account.go`

```
// +build !confonly
// 声明当前文件不仅仅是配置文件

package vless

import (
    "v2ray.com/core/common/protocol"
    "v2ray.com/core/common/uuid"
)

// AsAccount implements protocol.Account.AsAccount().
// 实现 protocol.Account 接口的 AsAccount 方法
func (a *Account) AsAccount() (protocol.Account, error) {
    // 解析账户 ID
    id, err := uuid.ParseString(a.Id)
    if err != nil {
        return nil, newError("failed to parse ID").Base(err).AtError()
    }
    // 返回内存账户对象
    return &MemoryAccount{
        ID:         protocol.NewID(id),
        Flow:       a.Flow,       // 需要在这里解析吗？
        Encryption: a.Encryption, // 需要在这里解析吗？
    }, nil
}

// MemoryAccount is an in-memory form of VLess account.
// MemoryAccount 是 VLess 账户的内存形式
type MemoryAccount struct {
    // 账户的 ID
    ID *protocol.ID
    // 账户的流量。可能是 "xtls-rprx-origin"
    Flow string
    // 账户的加密方式。用于客户端连接，目前只接受 "none"
    Encryption string
}

// Equals implements protocol.Account.Equals().
// 实现 protocol.Account 接口的 Equals 方法
func (a *MemoryAccount) Equals(account protocol.Account) bool {
    // 将账户转换为 VLess 账户
    vlessAccount, ok := account.(*MemoryAccount)
    if !ok {
        return false
    }
    // 比较账户的 ID 是否相等
    return a.ID.Equals(vlessAccount.ID)
}
```