# `v2ray-core\common\uuid\uuid.go`

```go
package uuid // import "v2ray.com/core/common/uuid"

import (
    "bytes" // 导入 bytes 包，用于操作字节
    "crypto/rand" // 导入 crypto/rand 包，用于生成随机数
    "encoding/hex" // 导入 encoding/hex 包，用于十六进制编解码

    "v2ray.com/core/common" // 导入 common 包
    "v2ray.com/core/common/errors" // 导入 errors 包
)

var (
    byteGroups = []int{8, 4, 4, 4, 12} // 定义 UUID 的字节分组
)

type UUID [16]byte // 定义 UUID 类型，长度为 16 字节

// String returns the string representation of this UUID.
func (u *UUID) String() string {
    bytes := u.Bytes() // 获取 UUID 的字节表示
    result := hex.EncodeToString(bytes[0 : byteGroups[0]/2]) // 将字节转换为十六进制字符串
    start := byteGroups[0] / 2 // 记录起始位置
    for i := 1; i < len(byteGroups); i++ { // 遍历每个字节分组
        nBytes := byteGroups[i] / 2 // 获取当前分组的字节数
        result += "-" // 添加分隔符
        result += hex.EncodeToString(bytes[start : start+nBytes]) // 将字节转换为十六进制字符串
        start += nBytes // 更新起始位置
    }
    return result // 返回 UUID 的字符串表示
}

// Bytes returns the bytes representation of this UUID.
func (u *UUID) Bytes() []byte {
    return u[:] // 返回 UUID 的字节表示
}

// Equals returns true if this UUID equals another UUID by value.
func (u *UUID) Equals(another *UUID) bool {
    if u == nil && another == nil { // 判断两个 UUID 是否都为 nil
        return true
    }
    if u == nil || another == nil { // 判断两个 UUID 是否有一个为 nil
        return false
    }
    return bytes.Equal(u.Bytes(), another.Bytes()) // 比较两个 UUID 的字节表示是否相等
}

// New creates a UUID with random value.
func New() UUID {
    var uuid UUID // 创建一个 UUID 对象
    common.Must2(rand.Read(uuid.Bytes())) // 生成随机的 UUID 值
    return uuid // 返回生成的 UUID 对象
}

// ParseBytes converts a UUID in byte form to object.
func ParseBytes(b []byte) (UUID, error) {
    var uuid UUID // 创建一个 UUID 对象
    if len(b) != 16 { // 判断字节数组长度是否为 16
        return uuid, errors.New("invalid UUID: ", b) // 返回错误信息
    }
    copy(uuid[:], b) // 将字节数组复制到 UUID 对象中
    return uuid, nil // 返回转换后的 UUID 对象
}

// ParseString converts a UUID in string form to object.
func ParseString(str string) (UUID, error) {
    var uuid UUID // 创建一个 UUID 对象

    text := []byte(str) // 将字符串转换为字节数组
    if len(text) < 32 { // 判断字节数组长度是否小于 32
        return uuid, errors.New("invalid UUID: ", str) // 返回错误信息
    }

    b := uuid.Bytes() // 获取 UUID 的字节表示

    for _, byteGroup := range byteGroups { // 遍历每个字节分组
        if text[0] == '-' { // 判断是否为分隔符
            text = text[1:] // 去除分隔符
        }

        if _, err := hex.Decode(b[:byteGroup/2], text[:byteGroup]); err != nil { // 将字符串转换为字节并进行解码
            return uuid, err // 返回错误信息
        }

        text = text[byteGroup:] // 更新字节数组位置
        b = b[byteGroup/2:] // 更新字节位置
    }
}
    # 返回 uuid 和 nil
    return uuid, nil
# 闭合前面的函数定义
```