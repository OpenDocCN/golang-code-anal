# `v2ray-core\transport\internet\headers\srtp\srtp.go`

```
package srtp

import (
    "context"
    "encoding/binary"

    "v2ray.com/core/common"
    "v2ray.com/core/common/dice"
)

type SRTP struct {
    header uint16  // 定义 SRTP 结构体的头部字段，类型为 uint16
    number uint16  // 定义 SRTP 结构体的编号字段，类型为 uint16
}

func (*SRTP) Size() int32 {
    return 4  // 返回 SRTP 结构体的大小，类型为 int32
}

// Serialize implements PacketHeader.
func (s *SRTP) Serialize(b []byte) {
    s.number++  // 编号自增1
    binary.BigEndian.PutUint16(b, s.header)  // 将头部字段写入字节切片
    binary.BigEndian.PutUint16(b[2:], s.number)  // 将编号字段写入字节切片
}

// New returns a new SRTP instance based on the given config.
func New(ctx context.Context, config interface{}) (interface{}, error) {
    return &SRTP{  // 返回一个新的 SRTP 实例
        header: 0xB5E8,  // 头部字段赋值为 0xB5E8
        number: dice.RollUint16(),  // 编号字段赋值为随机生成的 uint16 数值
    }, nil
}

func init() {
    common.Must(common.RegisterConfig((*Config)(nil), New))  // 注册配置信息和对应的构造函数
}
```