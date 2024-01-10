# `v2ray-core\transport\internet\headers\wechat\wechat.go`

```
package wechat

import (
    "context"
    "encoding/binary"

    "v2ray.com/core/common"
    "v2ray.com/core/common/dice"
)

type VideoChat struct {
    sn uint32
}

func (vc *VideoChat) Size() int32 {
    return 13
}

// Serialize implements PacketHeader.
func (vc *VideoChat) Serialize(b []byte) {
    vc.sn++  // 递增序列号
    b[0] = 0xa1  // 设置字节流中的第一个字节为 0xa1
    b[1] = 0x08  // 设置字节流中的第二个字节为 0x08
    binary.BigEndian.PutUint32(b[2:], vc.sn) // 将序列号写入字节流的第3到第6个字节
    b[6] = 0x00  // 设置字节流中的第七个字节为 0x00
    b[7] = 0x10  // 设置字节流中的第八个字节为 0x10
    b[8] = 0x11  // 设置字节流中的第九个字节为 0x11
    b[9] = 0x18  // 设置字节流中的第十个字节为 0x18
    b[10] = 0x30  // 设置字节流中的第十一个字节为 0x30
    b[11] = 0x22  // 设置字节流中的第十二个字节为 0x22
    b[12] = 0x30  // 设置字节流中的第十三个字节为 0x30
}

// NewVideoChat returns a new VideoChat instance based on given config.
func NewVideoChat(ctx context.Context, config interface{}) (interface{}, error) {
    return &VideoChat{
        sn: uint32(dice.RollUint16()),  // 根据给定的配置生成一个新的 VideoChat 实例，其中 sn 值为一个随机生成的 16 位无符号整数
    }, nil
}

func init() {
    common.Must(common.RegisterConfig((*VideoConfig)(nil), NewVideoChat))  // 注册 VideoConfig 类型的配置和 NewVideoChat 函数
}
```