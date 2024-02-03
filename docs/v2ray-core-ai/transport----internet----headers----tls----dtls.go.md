# `v2ray-core\transport\internet\headers\tls\dtls.go`

```go
package tls

import (
    "context"

    "v2ray.com/core/common"
    "v2ray.com/core/common/dice"
)

// DTLS writes header as DTLS. See https://tools.ietf.org/html/rfc6347
type DTLS struct {
    epoch    uint16  // DTLS协议的epoch字段，占用2个字节
    length   uint16  // DTLS协议的length字段，占用2个字节
    sequence uint32  // DTLS协议的sequence字段，占用4个字节
}

// Size implements PacketHeader.
func (*DTLS) Size() int32 {
    return 1 + 2 + 2 + 6 + 2  // 返回DTLS协议头的大小
}

// Serialize implements PacketHeader.
func (d *DTLS) Serialize(b []byte) {
    b[0] = 23  // 设置应用数据类型为23
    b[1] = 254  // 设置DTLS协议的版本号
    b[2] = 253  // 设置DTLS协议的版本号
    b[3] = byte(d.epoch >> 8)  // 将epoch字段的高8位存入b[3]
    b[4] = byte(d.epoch)  // 将epoch字段的低8位存入b[4]
    b[5] = 0  // 设置为0
    b[6] = 0  // 设置为0
    b[7] = byte(d.sequence >> 24)  // 将sequence字段的高8位存入b[7]
    b[8] = byte(d.sequence >> 16)  // 将sequence字段的次高8位存入b[8]
    b[9] = byte(d.sequence >> 8)  // 将sequence字段的次低8位存入b[9]
    b[10] = byte(d.sequence)  // 将sequence字段的低8位存入b[10]
    d.sequence++  // sequence字段自增1
    b[11] = byte(d.length >> 8)  // 将length字段的高8位存入b[11]
    b[12] = byte(d.length)  // 将length字段的低8位存入b[12]
    d.length += 17  // length字段增加17
    if d.length > 100 {  // 如果length字段大于100
        d.length -= 50  // 则减去50
    }
}

// New creates a new UTP header for the given config.
func New(ctx context.Context, config interface{}) (interface{}, error) {
    return &DTLS{  // 返回一个新的DTLS对象
        epoch:    dice.RollUint16(),  // 使用dice包生成一个随机的uint16数作为epoch字段的值
        sequence: 0,  // sequence字段初始化为0
        length:   17,  // length字段初始化为17
    }, nil
}

func init() {
    common.Must(common.RegisterConfig((*PacketConfig)(nil), New))  // 注册PacketConfig类型的配置，并使用New函数进行初始化
}
```