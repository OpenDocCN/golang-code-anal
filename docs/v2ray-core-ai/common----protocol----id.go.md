# `v2ray-core\common\protocol\id.go`

```go
package protocol

import (
    "crypto/hmac"  // 导入加密哈希消息认证码包
    "crypto/md5"   // 导入 MD5 加密算法包
    "hash"         // 导入哈希函数包

    "v2ray.com/core/common"  // 导入通用工具包
    "v2ray.com/core/common/uuid"  // 导入 UUID 包
)

const (
    IDBytesLen = 16  // 定义 ID 字节长度为 16
)

type IDHash func(key []byte) hash.Hash  // 定义 IDHash 类型为哈希函数

func DefaultIDHash(key []byte) hash.Hash {
    return hmac.New(md5.New, key)  // 返回使用 MD5 哈希算法和指定密钥的 HMAC 对象
}

// The ID of en entity, in the form of a UUID.
type ID struct {
    uuid   uuid.UUID  // 定义 ID 结构体包含 UUID
    cmdKey [IDBytesLen]byte  // 定义 ID 结构体包含长度为 16 的字节数组
}

// Equals returns true if this ID equals to the other one.
func (id *ID) Equals(another *ID) bool {
    return id.uuid.Equals(&(another.uuid))  // 如果当前 ID 的 UUID 等于另一个 ID 的 UUID，则返回 true
}

func (id *ID) Bytes() []byte {
    return id.uuid.Bytes()  // 返回 ID 的 UUID 字节表示
}

func (id *ID) String() string {
    return id.uuid.String()  // 返回 ID 的 UUID 字符串表示
}

func (id *ID) UUID() uuid.UUID {
    return id.uuid  // 返回 ID 的 UUID
}

func (id ID) CmdKey() []byte {
    return id.cmdKey[:]  // 返回 ID 的 cmdKey 字节数组
}

// NewID returns an ID with given UUID.
func NewID(uuid uuid.UUID) *ID {
    id := &ID{uuid: uuid}  // 创建一个包含给定 UUID 的 ID 对象
    md5hash := md5.New()  // 创建一个 MD5 哈希对象
    common.Must2(md5hash.Write(uuid.Bytes())  // 将 UUID 的字节表示写入哈希对象
    common.Must2(md5hash.Write([]byte("c48619fe-8f02-49e0-b9e9-edf763e17e21")))  // 将指定字符串写入哈希对象
    md5hash.Sum(id.cmdKey[:0])  // 计算哈希值并存储到 cmdKey 中
    return id  // 返回创建的 ID 对象
}

func nextID(u *uuid.UUID) uuid.UUID {
    md5hash := md5.New()  // 创建一个 MD5 哈希对象
    common.Must2(md5hash.Write(u.Bytes())  // 将 UUID 的字节表示写入哈希对象
    common.Must2(md5hash.Write([]byte("16167dc8-16b6-4e6d-b8bb-65dd68113a81")))  // 将指定字符串写入哈希对象
    var newid uuid.UUID  // 声明一个新的 UUID 对象
    for {
        md5hash.Sum(newid[:0])  // 计算哈希值并存储到 newid 中
        if !newid.Equals(u) {  // 如果 newid 不等于原始 UUID，则返回 newid
            return newid
        }
        common.Must2(md5hash.Write([]byte("533eff8a-4113-4b10-b5ce-0f5d76b98cd2")))  // 将指定字符串写入哈希对象
    }
}

func NewAlterIDs(primary *ID, alterIDCount uint16) []*ID {
    alterIDs := make([]*ID, alterIDCount)  // 创建一个包含指定数量 ID 对象的切片
    prevID := primary.UUID()  // 获取主 ID 的 UUID
    for idx := range alterIDs {
        newid := nextID(&prevID)  // 获取下一个 ID 的 UUID
        alterIDs[idx] = NewID(newid)  // 创建一个包含新 UUID 的 ID 对象
        prevID = newid  // 更新 prevID 为新的 UUID
    }
    return alterIDs  // 返回创建的 ID 对象切片
}
```