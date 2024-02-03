# `v2ray-core\common\protocol\account.go`

```go
// 定义一个接口类型 Account，用于表示用户身份信息，用于认证
type Account interface {
    // 定义一个方法 Equals，用于比较两个 Account 对象是否相等
    Equals(Account) bool
}

// 定义一个接口类型 AsAccount，表示可以被转换为 Account 对象的对象
type AsAccount interface {
    // 定义一个方法 AsAccount，用于将当前对象转换为 Account 对象，可能会返回错误
    AsAccount() (Account, error)
}
```