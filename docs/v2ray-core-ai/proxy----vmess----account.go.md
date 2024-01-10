# `v2ray-core\proxy\vmess\account.go`

```
// +build !confonly

package vmess

import (
    "v2ray.com/core/common/dice"  // 导入随机数生成包
    "v2ray.com/core/common/protocol"  // 导入协议包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
)

// MemoryAccount is an in-memory form of VMess account.
type MemoryAccount struct {
    // ID is the main ID of the account.
    ID *protocol.ID  // 账户的主要 ID
    // AlterIDs are the alternative IDs of the account.
    AlterIDs []*protocol.ID  // 账户的备用 ID
    // Security type of the account. Used for client connections.
    Security protocol.SecurityType  // 账户的安全类型，用于客户端连接
}

// AnyValidID returns an ID that is either the main ID or one of the alternative IDs if any.
func (a *MemoryAccount) AnyValidID() *protocol.ID {
    if len(a.AlterIDs) == 0 {  // 如果备用 ID 为空
        return a.ID  // 返回主要 ID
    }
    return a.AlterIDs[dice.Roll(len(a.AlterIDs))]  // 返回随机选择的备用 ID
}

// Equals implements protocol.Account.
func (a *MemoryAccount) Equals(account protocol.Account) bool {
    vmessAccount, ok := account.(*MemoryAccount)  // 将账户转换为 VMess 账户
    if !ok {  // 如果转换失败
        return false  // 返回 false
    }
    // TODO: handle AlterIds difference
    return a.ID.Equals(vmessAccount.ID)  // 返回主要 ID 是否相等
}

// AsAccount implements protocol.Account.
func (a *Account) AsAccount() (protocol.Account, error) {
    id, err := uuid.ParseString(a.Id)  // 解析账户 ID
    if err != nil {  // 如果解析失败
        return nil, newError("failed to parse ID").Base(err).AtError()  // 返回错误信息
    }
    protoID := protocol.NewID(id)  // 创建新的 ID
    return &MemoryAccount{
        ID:       protoID,  // 设置主要 ID
        AlterIDs: protocol.NewAlterIDs(protoID, uint16(a.AlterId)),  // 设置备用 ID
        Security: a.SecuritySettings.GetSecurityType(),  // 设置安全类型
    }, nil  // 返回创建的 MemoryAccount 对象和 nil
}
```