# `v2ray-core\proxy\trojan\config.go`

```
package trojan

import (
    "crypto/sha256"  // 导入加密算法包
    "encoding/hex"   // 导入十六进制编码包
    fmt "fmt"         // 导入格式化包

    "v2ray.com/core/common"  // 导入通用包
    "v2ray.com/core/common/protocol"  // 导入协议包
)

// MemoryAccount is an account type converted from Account.
type MemoryAccount struct {
    Password string  // 账户密码
    Key      []byte  // 密钥
}

// AsAccount implements protocol.AsAccount.
func (a *Account) AsAccount() (protocol.Account, error) {
    password := a.GetPassword()  // 获取密码
    key := hexSha224(password)    // 对密码进行 SHA224 加密
    return &MemoryAccount{
        Password: password,  // 返回包含密码和密钥的 MemoryAccount 对象
        Key:      key,
    }, nil
}

// Equals implements protocol.Account.Equals().
func (a *MemoryAccount) Equals(another protocol.Account) bool {
    if account, ok := another.(*MemoryAccount); ok {  // 判断是否为 MemoryAccount 类型
        return a.Password == account.Password  // 比较密码是否相等
    }
    return false
}

func hexSha224(password string) []byte {
    buf := make([]byte, 56)  // 创建长度为 56 的字节切片
    hash := sha256.New224()  // 创建 SHA224 加密对象
    common.Must2(hash.Write([]byte(password)))  // 对密码进行 SHA224 加密
    hex.Encode(buf, hash.Sum(nil))  // 将加密结果转换为十六进制编码
    return buf  // 返回加密后的结果
}

func hexString(data []byte) string {
    str := ""  // 初始化空字符串
    for _, v := range data {  // 遍历字节切片
        str += fmt.Sprintf("%02x", v)  // 将每个字节转换为十六进制并拼接到字符串中
    }
    return str  // 返回十六进制字符串
}
```