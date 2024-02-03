# `v2ray-core\transport\internet\headers\utp\utp.go`

```go
package utp

import (
    "context"
    "encoding/binary"

    "v2ray.com/core/common"
    "v2ray.com/core/common/dice"
)

type UTP struct {
    header       byte      // UTP 头部的字节
    extension    byte      // UTP 扩展的字节
    connectionId uint16    // UTP 连接 ID
}

func (*UTP) Size() int32 {
    return 4   // 返回 UTP 头部的大小
}

// Serialize implements PacketHeader.
func (u *UTP) Serialize(b []byte) {
    binary.BigEndian.PutUint16(b, u.connectionId)  // 将连接 ID 转换为大端序的字节流
    b[2] = u.header   // 将头部字节写入到字节流的第三个位置
    b[3] = u.extension   // 将扩展字节写入到字节流的第四个位置
}

// New creates a new UTP header for the given config.
func New(ctx context.Context, config interface{}) (interface{}, error) {
    return &UTP{
        header:       1,   // 设置头部字节为 1
        extension:    0,   // 设置扩展字节为 0
        connectionId: dice.RollUint16(),   // 生成一个随机的连接 ID
    }, nil
}

func init() {
    common.Must(common.RegisterConfig((*Config)(nil), New))   // 注册 UTP 配置
}
```