# `v2ray-core\proxy\socks\config.go`

```
// +build !confonly

// 导入 protocol 包
package socks

import "v2ray.com/core/common/protocol"

// 判断当前账户是否与另一个账户相等
func (a *Account) Equals(another protocol.Account) bool {
    // 尝试将另一个账户转换为当前类型的账户
    if account, ok := another.(*Account); ok {
        // 比较用户名是否相等
        return a.Username == account.Username
    }
    return false
}

// 将当前账户转换为 protocol.Account 接口
func (a *Account) AsAccount() (protocol.Account, error) {
    // 返回当前账户和空错误
    return a, nil
}

// 检查服务器配置中是否存在指定用户名和密码的账户
func (c *ServerConfig) HasAccount(username, password string) bool {
    // 如果账户列表为空，则返回 false
    if c.Accounts == nil {
        return false
    }
    // 在账户列表中查找指定用户名对应的密码
    storedPassed, found := c.Accounts[username]
    // 如果未找到指定用户名的密码，则返回 false
    if !found {
        return false
    }
    // 比较存储的密码与输入的密码是否相等
    return storedPassed == password
}
```