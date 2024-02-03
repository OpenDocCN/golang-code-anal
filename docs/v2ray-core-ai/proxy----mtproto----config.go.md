# `v2ray-core\proxy\mtproto\config.go`

```go
// 在 mtproto 包中导入 protocol 包
import (
    "v2ray.com/core/common/protocol"
)

// 定义 Account 结构体的 Equals 方法，用于比较两个账户是否相等
func (a *Account) Equals(another protocol.Account) bool {
    // 将 another 转换为 *Account 类型，并判断是否成功
    aa, ok := another.(*Account)
    if !ok {
        return false
    }

    // 比较两个账户的 Secret 字段长度是否相等
    if len(a.Secret) != len(aa.Secret) {
        return false
    }

    // 遍历比较两个账户的 Secret 字段的值
    for i, v := range a.Secret {
        if v != aa.Secret[i] {
            return false
        }
    }

    // 如果以上条件都满足，则返回 true
    return true
}
```